# Crypto Microstructure Prediction — MAMBA Trading

## 한 줄 요약

Binance BTCUSDT의 tick-by-tick 체결 데이터를 수집·전처리하고, 4-layer Mamba 모델로 5초 뒤 가격 방향을 예측했습니다. 무수수료 백테스트에서는 수익이 발생했지만 거래 빈도가 높아 실제 수수료를 반영하면 손익이 붕괴하는 것을 확인했고, 분류 정확도보다 execution cost와 turnover가 실제 활용 가능성을 좌우한다는 점을 검증한 프로젝트입니다.

## 기간 및 역할

- 자료와 코드에서 확인되는 주요 작업 기간: **2025년 10월~12월**
- Team 3 AI 투자 전략 비교 프로젝트
- 실제 AI 모델 개발은 4명이 각자 하나의 전략을 담당
  - Random Forest
  - Markov Regime Switching
  - **MAMBA — 김재훈 담당**
  - AdaBoost-LSTM
- Buy & Hold와 Random Trading을 benchmark로 함께 비교

최종 발표의 목표는 단순히 예측 정확도가 높은 모델을 만드는 것이 아니라, **실제 거래와 유사한 환경에서 전략이 usable한지 확인하는 것**이었습니다.

## 프로젝트 전체 평가 구조

팀 전체에서는 서로 다른 철학의 AI 전략을 공통 trading framework에서 비교했습니다.

- candle 기반 모델: 1시간 단위 의사결정
- MAMBA: tick-by-tick microstructure 기반 **5초 미래 방향 예측**
- offline backtest
- 거래 수수료 sensitivity test
- Binance Testnet 기반 약 2.5일 자동 거래 simulation
- 공통 dashboard에서 계좌별 수익률 비교

이 문서에서는 그중 제가 직접 담당한 **MAMBA microstructure model과 데이터/백테스트 파이프라인**을 중심으로 정리합니다.

## 1. 초기 Upbit LOB prototype

초기에는 Upbit `KRW-BTC` order book을 WebSocket으로 직접 수집해 LOB 기반 모델을 실험했습니다.

확인되는 설정은 다음과 같습니다.

- 상위 **20-level** bid/ask 사용
- sequence length: **50**
- 미래 horizon: **5초**를 중심으로 실험
- bid/ask level의 price/size 등으로 snapshot 구성
- 시간 순서를 유지한 train/validation split

초기 모델 코드는 각 LOB snapshot을 CNN으로 encoding하고 시계열 모듈로 넘기는 구조였습니다. 이 단계의 `train.py`에는 실제 Mamba 대신 GRU 기반 `DummyMamba` placeholder가 들어 있으므로, 이를 최종 Mamba 모델의 성능으로 설명하지 않습니다.

즉 개발 흐름은 다음과 같습니다.

`Upbit 20-level LOB prototype → CNN 기반 spatial encoding 실험 → Binance tick trade 기반 실제 Mamba 모델로 전환`

## 2. Binance tick data 수집

최종 모델에서는 order book snapshot 대신 **Binance Spot BTCUSDT aggTrade**를 사용했습니다.

`collector.py`는 과거 체결 데이터를 날짜별 JSONL로 저장하도록 구현되어 있으며, 수집 구간은 다음과 같이 분리했습니다.

- train: **2025-10-21 00:00 ~ 2025-11-21 00:00 UTC**
- test: **2025-11-21 00:00 ~ 2025-11-28 00:00 UTC**

Drive에도 `trades_YYYYMMDD.jsonl` 형태의 일자별 raw data가 수백 MB 단위로 보관되어 있습니다.

전처리 metadata에는 다음 raw sample 규모가 기록되어 있습니다.

- train `N_raw`: **38,082,848**
- test `N_raw`: **9,961,021**

대용량 데이터를 한 번에 RAM에 올리지 않도록 최종 전처리/학습 파이프라인에서는 `numpy.memmap` 기반 파일을 사용했습니다.

## 3. Feature와 label 설계

최종 MAMBA는 각 trade에서 다음 **4개 microstructure feature**를 사용합니다.

- 이전 trade 대비 price return
- trade size
- aggressor side
- inter-trade time (`Δt`)

입력은 최근 **50개 trade sequence**이며, target은 **5초 뒤 가격 방향**입니다.

label은 고정 수익률 threshold 대신 train return distribution의 quantile을 사용해 3-class로 구성했습니다.

- 하위 약 20%: `Down`
- 중간 약 60%: `No-Trade`
- 상위 약 20%: `Up`

전처리된 5초 dataset은 다음 규모로 저장되어 있습니다.

- train dataset: **100,000 sequences**
  - 실제 학습: 80,000
  - validation: 20,000
- independent test dataset: **200,000 sequences**
- tensor shape: `[N, 50, 4]`

## 4. 최종 Mamba model

최종 `model.py`는 `mamba_ssm.Mamba`를 직접 사용하는 구조입니다.

```text
4-dim trade feature
→ Linear projection (d_model=128)
→ MambaBlock × 4
→ LayerNorm
→ last sequence state
├─ Direction head: Down / No-Trade / Up
└─ Magnitude head: |future return| regression
```

각 Mamba block에는 residual connection과 LayerNorm을 적용했습니다.

주요 hyperparameter는 다음과 같습니다.

- `d_model = 128`
- `num_layers = 4`
- `dropout = 0.1`
- Mamba `d_state = 16`, `d_conv = 4`, `expand = 2`

방향 분류뿐 아니라 미래 return magnitude를 함께 예측하는 multi-task 구조로 확장했습니다.

## 5. 학습 방식

학습에서는 3-class 불균형을 보정하기 위해 train label count로 class weight를 계산해 weighted CrossEntropy를 사용했습니다.

전체 loss는 다음 두 항의 조합입니다.

- direction: weighted CrossEntropy
- magnitude: MSE
- total: `direction_loss + 0.3 × magnitude_loss`

그 외 설정은 다음과 같습니다.

- optimizer: AdamW
- learning rate: `1e-3`
- weight decay: `1e-3`
- batch size: `512`
- 최대 50 epochs
- early stopping patience: 5

보관된 5초 `best_model.pt`는:

- best epoch: **17**
- validation accuracy: **0.54125**

를 기록합니다.

과거 경험 메모에 있던 `val_acc=0.5951`은 현재 최종 Binance 5초 checkpoint와 일치하지 않으므로 대표 성능으로 사용하지 않습니다.

또한 label 자체가 `Down / No-Trade / Up ≈ 20/60/20` 구조이므로 이 프로젝트는 단순 classification accuracy보다 실제 거래 결과를 더 중요한 평가 기준으로 삼았습니다.

## 6. probability threshold 기반 trading

모델의 `Up` probability를 바로 매매 신호로 사용하지 않고 threshold sweep을 수행했습니다.

백테스트에서는:

- `p_up >= threshold`일 때 LONG 진입
- threshold: `0.50 ~ 0.90`
- 초기 자본: `$10,000`
- 수수료 정책별 동일 prediction 결과 비교

방식으로 signal confidence와 trading frequency의 관계를 확인했습니다.

무수수료 조건에서 threshold `0.50`은:

- trades: **4,319**
- win rate: **64.37%**
- final equity: **$12,326.05**
- return: **+23.26%**

을 기록했습니다.

즉 prediction에 trading edge가 전혀 없었던 것은 아니지만, 거래 1회당 기대 수익이 매우 작고 거래 횟수가 많다는 문제가 있었습니다.

## 7. 수수료를 넣자 전략이 붕괴한 이유

동일한 prediction 결과에 실제와 유사한 fee를 적용하자 결과가 완전히 달라졌습니다.

### no fee

- best return: **+23.26%**
- `$10,000 → $12,326`

### Binance Spot fee 0.1% per side

- round-trip fee: **0.2%**
- best stored result: **-99.18%**
- `$10,000 → 약 $81.57`

### Futures market fee 0.05% per side

- round-trip fee: **0.1%**
- best stored result: **-90.44%**
- `$10,000 → 약 $956.24`

무수수료 실험의 선택된 trade들은 평균 gross return이 대략 `0.004~0.005%` 수준인데, Spot round-trip fee는 `0.2%`였습니다.

즉 모델이 맞히는 비율이나 gross PnL만 보면 신호가 있어 보였지만, **한 번의 거래에서 얻는 edge보다 transaction cost가 훨씬 큰 high-turnover 전략**이었기 때문에 실제 손익은 빠르게 악화됐습니다.

이 프로젝트에서 가장 중요한 결과는 높은 수익률을 달성했다는 것이 아니라, 이 차이를 수치로 확인한 것입니다.

## 8. 발표자료와 artifact 사이의 수치 차이

발표자료와 기술보고서에서는 futures limit fee `0.02%` 조건을:

- `-51.88%`
- `$10,000 → $4,812`

로 제시합니다.

반면 현재 보관된 `backtest_long_only_5s_fut_limit_0p02.csv`의 best final equity는:

- 약 `$4,182.96`
- **-58.17%**

입니다.

이는 실험 iteration 또는 threshold/전략 버전 차이로 보이지만 현재 자료만으로 어느 값이 최종 authoritative result인지 확정하기 어렵습니다.

따라서 이력서에서는 이 수치를 사용하지 않고, 발표자료와 저장 CSV가 모두 일치하는 **no-fee +23.26%, Spot -99.19%, Futures market -90.44%**를 근거로 사용합니다.

## 9. independent test와 accuracy 해석

별도 test artifact에는 **200,000 samples**의 `y_true` / `y_pred`가 보관되어 있습니다.

현재 파일을 직접 비교하면 test accuracy는 약 **45.79%**입니다.

따라서 이 프로젝트를 `높은 예측 정확도를 달성한 모델`로 설명하지 않습니다. 오히려 validation/test classification metric과 실제 PnL을 함께 보면서:

- class distribution
- threshold
- trade frequency
- transaction fee
- evaluation period

에 따라 전략 성과가 얼마나 달라지는지를 확인한 경험으로 정리합니다.

## 10. Binance Testnet 실전형 검증

팀 전체 프로젝트에서는 offline backtest에서 끝내지 않고 Binance Testnet으로 약 **2.5일** 동안 자동 거래 simulation을 진행했습니다.

- 시작 자산: 1 BTC + 10,000 USDT
- 기간: 12/6 00:00 ~ 12/8 12:00
- 각 전략별 별도 계정
- Local Proxy가 Binance account state를 polling
- React dashboard에서 전략별 수익률과 보유 자산 비교

기술보고서의 Testnet 결과에서 MAMBA는 약 **-98.3%**를 기록했습니다.

이 결과 역시 5초 단위 high-frequency signal을 실제 execution 환경에 가까이 가져갔을 때 transaction cost와 trading frequency가 치명적이라는 결론과 일치했습니다.

팀 전체의 proxy/dashboard 구현을 제 개인 구현으로 주장하지 않으며, 제 핵심 기여는 MAMBA 전략의 **데이터 → 모델 → prediction → fee-aware backtest / trading 연결**입니다.

## 개발 과정에서 확인되는 변화

### 초기

- Upbit WebSocket LOB 수집
- 20-level order book sequence 구성
- CNN spatial encoder 실험
- GRU placeholder로 시계열 pipeline 먼저 검증

### 최종

- Binance Spot historical `aggTrades`로 데이터 소스 전환
- 월 단위 train / 1주 test 기간 분리
- 4-feature tick sequence 구성
- quantile 3-class labeling
- 실제 `mamba_ssm.Mamba` 4-layer 적용
- direction + magnitude multi-task learning
- probability threshold sweep
- 수수료 tier별 backtest
- Binance Testnet simulation과 연결

즉 특정 모델 하나를 학습한 경험이라기보다 **데이터 정의와 target 설계부터 실거래 비용을 포함한 평가까지 전체 experiment loop를 반복한 경험**입니다.

## 이력서용 핵심 bullet

- Binance BTCUSDT `aggTrades` 약 1개월 train / 1주 test 데이터를 수집·전처리하고, **50-trade sequence와 price return·size·aggressor side·Δt 4개 feature로 5초 뒤 방향 예측 dataset 구축**
- `d_model=128`의 **Mamba block 4개와 direction/magnitude multi-task head를 구현**하고 100k train sequence로 Down/No-Trade/Up 3-class 모델 학습
- `p_up` threshold sweep과 fee tier별 backtest를 구성해 **무수수료 +23.26%가 Spot 0.1% fee 적용 시 -99.19%로 역전되는 현상**을 확인
- prediction accuracy만으로 전략을 평가하지 않고 **turnover와 transaction cost까지 포함해 실제 활용 가능성을 검증**, Binance Testnet 자동 거래 실험까지 확장

## 표현할 때 주의할 부분

다음 표현은 피합니다.

- `CNN + Mamba가 최종 모델이었다`
  - CNN은 초기 Upbit LOB prototype 단계이며 최종 Binance 모델은 trade feature projection + 4-layer Mamba입니다.
- `validation accuracy 59.51%`
  - 현재 최종 5초 checkpoint는 54.125%입니다.
- `높은 예측 정확도로 수익을 냈다`
  - independent test accuracy와 fee-aware PnL 모두 이를 지지하지 않습니다.
- `실제 투자에서 수익성을 입증했다`
  - 반대로 실제 비용을 반영했을 때 전략의 한계를 확인한 프로젝트입니다.
- 팀 전체 dashboard / proxy / 다른 AI 모델을 개인 구현으로 표현하지 않습니다.

## 보여주는 역량

- 대용량 금융 tick data 수집 및 전처리
- `numpy.memmap` 기반 대용량 dataset 처리
- 시계열 sequence / labeling 설계
- PyTorch / Mamba SSM 모델 구현
- class imbalance 대응
- multi-task learning
- probability threshold calibration
- backtesting과 transaction cost modeling
- validation metric과 실제 KPI의 차이 분석
- 실패 결과를 숨기지 않고 원인을 수치로 분해하는 실험 태도

## 자소서 활용 포인트

- AI 모델 성능을 실제 사용 환경의 KPI로 다시 검증한 경험
- 처음의 LOB/CNN prototype에서 최종 tick/Mamba 구조로 문제 정의를 바꾼 경험
- 대용량 raw data를 직접 수집해 학습/검증 pipeline을 만든 경험
- 좋은 offline 결과가 production-like 환경에서는 무너질 수 있음을 확인한 경험
- 결과가 기대와 다를 때 accuracy를 방어하기보다 fee와 turnover를 원인으로 분해한 경험
