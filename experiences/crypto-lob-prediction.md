# Crypto Microstructure Prediction — MAMBA Trading

## 한 줄 요약

Binance BTCUSDT의 tick-by-tick 체결 데이터를 수집·전처리하고, 4-layer Mamba 모델로 5초 뒤 가격 방향을 예측했습니다. **약 19분의 validation 구간을 이용한 가상 중복거래 백테스트에서는 무수수료 +23.26%가 나왔지만, 수수료를 반영하면 손실로 바뀌었습니다.** 독립 test 20만 시퀀스의 3-class accuracy는 **45.79%**로 최빈 클래스 기준 55.64%보다 낮았습니다. 이 프로젝트는 모델의 수익성 입증이 아니라 **시계열 예측·실행비용 평가 과정과 실험 설계의 한계를 확인한 경험**입니다.

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

이 문서에서는 그중 제가 담당한 것으로 팀 발표에 기재된 **MAMBA microstructure model과 데이터/백테스트 파이프라인**을 중심으로 정리합니다. Git 저장소가 발견되지 않아 Drive 원본 코드·실험 파일·당시 보고서를 근거로 사용하며 파일별 단독 저작 여부는 Git commit으로 검증하지 못했습니다. 상세 출처는 [실험·소스 근거 원장](crypto-lob-prediction-evidence-ledger.md)에 있습니다.

## 1. 초기 Upbit LOB prototype

초기에는 Upbit `KRW-BTC` order book을 WebSocket으로 직접 수집해 LOB 기반 모델을 실험했습니다.

확인되는 설정은 다음과 같습니다.

- 상위 **20-level** bid/ask 사용
- sequence length: **50**
- 미래 horizon: **5초**를 중심으로 실험
- bid/ask level의 price/size 등으로 snapshot 구성
- 시간 순서를 유지한 train/validation split

초기 Upbit prototype의 `train.py`에는 실제 Mamba 대신 GRU 기반 `DummyMamba` placeholder가 있었다는 별도 기록이 있습니다. **하지만 Drive의 [2025-11-29 Binance LOB 모델](https://drive.google.com/file/d/1JvhlZWE5xpd--zhVTlTzkLw-Qo0DN7pK/view)은 실제 `mamba_ssm.Mamba` 4개 블록을 사용한 CNN+Mamba 이진 분류 구현**이므로, 이 둘을 같은 코드라고 적지 않습니다. Binance LOB의 CNN은 bid/ask 호가창에서 상대가격·정규화 수량·imbalance를 공간적으로 인코딩합니다.

즉 개발 흐름은 다음과 같습니다.

`Upbit LOB + DummyMamba prototype (별도 기록) → Binance LOB CNN+실제 Mamba 2-class 소스 (2025-11-29) → Binance trades 4-feature+실제 Mamba 3-class/magnitude 소스 (2025-12-05경)`

이 구조·파일 시점의 차이는 확인했지만, **전환의 정확한 의사결정 이유나 두 모델의 동등 조건 성능 비교는 확보하지 못했습니다.**

## 2. Binance tick data 수집

최종 모델에서는 order book snapshot 대신 **Binance Spot BTCUSDT aggTrade**를 사용했습니다.

`collector.py`는 과거 체결 데이터를 날짜별 JSONL로 저장하도록 구현되어 있으며, 수집 구간은 다음과 같이 분리했습니다.

- train: **2025-10-21 00:00 ~ 2025-11-21 00:00 UTC**
- test: **2025-11-21 00:00 ~ 2025-11-28 00:00 UTC**

Drive에도 `trades_YYYYMMDD.jsonl` 형태의 일자별 raw data가 수백 MB 단위로 보관되어 있습니다.

전처리 metadata에는 다음 raw sample 규모가 기록되어 있습니다.

- train 원본 `N_raw`: **38,082,848**
- test 원본 `N_raw`: **9,961,021**

대용량 데이터를 한 번에 RAM에 올리지 않도록 최종 전처리/학습 파이프라인에서는 `numpy.memmap` 기반 파일을 사용했습니다. **다만 모든 raw trade를 학습·평가한 것은 아닙니다.** 코드의 `MAX_TRAIN_SEQ=100,000`, `MAX_TEST_SEQ=200,000`에 의해 **원본 각 기간의 첫 유효 sequence에서 지정 개수까지만** 추출했습니다.

| 실제 입력 집합 | 보관된 timestamp 기준 UTC 범위 | 길이 |
|---|---|---|
| 학습 80,000개 | **2025-10-22 00:00:00~01:06:20** | 약 66분 |
| Validation 20,000개 | **2025-10-22 01:06:20~01:25:23** | 약 19분 |
| 독립 Test 200,000개 | **2025-11-21 00:00:00~02:33:15** | 약 2시간 33분 |

원본 [train `time.dat`](https://drive.google.com/file/d/1RCf5yQKSiZNRGulj-gIbERf63Nb-xviJ/view), [validation `time.npy`](https://drive.google.com/file/d/1NZFZcOAXq_UgL1yDyUuYL2FV3z6UxUpC/view), [test `time.dat`](https://drive.google.com/file/d/1sOgWdsEf6Q0ze0UthqwdDz-Yh05i35WK/view)의 첫/끝 시간을 NumPy로 직접 확인한 값입니다. 따라서 **'한 달 데이터 전체를 학습했고 일주일 전체에서 독립 평가했다'는 주장은 하지 않습니다.**

## 3. Feature와 label 설계

최종 MAMBA는 각 trade에서 다음 **4개 microstructure feature**를 사용합니다.

- 이전 trade 대비 price return
- trade size
- aggressor side
- inter-trade time (`Δt`)

입력은 최근 **50개 trade sequence**이며, target은 **5초 뒤 가격 방향**입니다.

label은 고정 수익률 threshold 대신 **각 split의 전체 미래 return 분포에서 별도로 계산한 quantile**을 사용해 3-class로 구성했습니다. [보존 전처리 코드](https://drive.google.com/file/d/1uSFTg1y9f7g3qqPQ6eLRsZpPpSMDhNuL/view)에서는 train/test 각각의 전체 raw 수익률로 경계를 계산하므로, 실제 잘라낸 첫 100k/200k만으로 라벨 기준을 정한 것은 아닙니다. **train과 test에서 클래스의 수익률 경계가 다를 수 있고, 미래 분포를 알고 정한 라벨**이라는 평가상 제약이 있습니다.

- 하위 약 20%: `Down`
- 중간 약 60%: `No-Trade`
- 상위 약 20%: `Up`

전처리된 5초 dataset은 다음 규모로 저장되어 있습니다.

- train dataset: **100,000 sequences**
  - 실제 학습: 80,000
  - validation: 20,000
- independent test dataset: **200,000 sequences (11/21 약 2시간 33분의 첫 연속 구간)**
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

**학습 원본에서 나눈 validation 20,000개(약 19분)**에 대해, 수수료 정책별 **최적 threshold를 같은 validation 결과에서 선택**했습니다. 이는 out-of-sample 투자 성과가 아니라 threshold tuning 결과이고, 5초 미래 수익률의 각 신호를 **독립 왕복 거래**로 가정한 값입니다. 매수 상태가 겹치는 현실적인 포지션 제약은 이 offline 백테스트에 없습니다.

무수수료 조건에서 threshold `0.50`은:

- trades: **4,319**
- win rate: **64.37%**
- final equity: **$12,326.05**
- return: **+23.26%**

을 기록했습니다.

즉 **이 validation 자료의 선택 신호에서 양의 gross return**은 있었지만, 이 결과만으로 재현 가능한 out-of-sample trading edge가 확인됐다고 해석할 수는 없습니다. 거래 신호마다 예상 수익이 작고 횟수가 많았다는 점은 코드와 결과표에서 확인됩니다.

## 7. 수수료를 넣자 전략이 붕괴한 이유

동일한 validation 예측과 미래 수익률에 수수료를 달리 적용하자 **offline 가상 백테스트 결과가 크게 달라졌습니다.** 각 정책은 동일 샘플을 사용하지만 **각자의 최대 equity threshold를 다시 선택**했으며, 수수료 외 실제 체결 조건은 충분히 모델링하지 못했습니다.

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

즉 이 자료에서 관찰된 **거래 1회당 평균 미래 수익률이 수수료 왕복비용보다 작고, 신호별 비용 누적이 가상 성과를 크게 악화**시켰습니다. 이것은 비용 민감도 측정 결과이지 실제 동일 조건의 실계좌 PnL이나 최종 검증된 거래 성과는 아닙니다.

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

차이가 발생한 원인은 **현재 자료만으로 미확정**입니다. 수수료 정책별 snapshot·threshold·평가 기간·코드 버전 차이 가능성은 가설이며 확인된 사실이 아닙니다.

따라서 이력서에서는 이 수치를 사용하지 않고, 발표자료와 저장 CSV가 모두 일치하는 **no-fee +23.26%, Spot -99.19%, Futures market -90.44%**를 근거로 사용합니다.

## 9. independent test와 accuracy 해석

별도 test artifact에는 **200,000 samples**의 `y_true` / `y_pred`가 보관되어 있습니다.

보관된 `y_true.npy`와 `y_pred.npy`(20만 건, `allow_pickle=False`)를 다시 비교한 **test accuracy는 45.7895%**입니다. 정답의 최빈 클래스 `No-Trade`는 **111,276/200,000 = 55.638%**이므로 단순 accuracy 기준으로 이 상수 예측 baseline보다 낮습니다. 이 시험 데이터는 **2025-11-21 약 2시간 33분 연속 구간**입니다. 다만 weighted loss로 학습한 모델이므로 방향별 precision/recall·거래 예측 품질 역시 별도 살펴야 합니다.

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

이 결과는 팀 보고서에 기록된 **Testnet 계좌 관측치(B)**이고, validation 위주 offline 백테스트의 가상 복리 수익률과는 다른 실행 조건·포지션 로직의 결과입니다. [`live_trader.py`](https://drive.google.com/file/d/1WZwfaIWHcs4cFfdfEs3shSEBlhPlMas9/view)는 실제로 열린 포지션이 있으면 추가 진입을 막고 5초 후 청산하는 반면, `backtest.py`는 신호가 난 각 시점마다 독립 왕복거래로 처리합니다. 따라서 **두 성과가 정량적으로 같은 전략의 연속 검증이라고 단정하지 않습니다.**

팀 전체의 proxy/dashboard 구현을 제 개인 구현으로 주장하지 않으며, 제 핵심 기여는 MAMBA 전략의 **데이터 → 모델 → prediction → fee-aware backtest / trading 연결**입니다.

## 개발 과정에서 확인되는 변화

### 초기

- Upbit WebSocket LOB 수집
- 20-level order book sequence 구성
- CNN spatial encoder 실험
- GRU placeholder로 시계열 pipeline 먼저 검증

### 최종

- Binance Spot historical `aggTrades`로 데이터 소스 전환
- 월 단위 train / 1주 test **원본 수집 기간**을 구분하되, 실제 입력은 **train 약 66분·validation 약 19분·test 약 2시간 33분**의 최초 추출 시퀀스
- 4-feature tick sequence 구성
- quantile 3-class labeling
- 실제 `mamba_ssm.Mamba` 4-layer 적용
- direction + magnitude multi-task learning
- probability threshold sweep
- 수수료 tier별 backtest
- Binance Testnet simulation과 연결

즉 특정 모델 하나를 학습한 경험이라기보다 **데이터 정의와 target 설계부터 실거래 비용을 포함한 평가까지 전체 experiment loop를 반복한 경험**입니다.

## 이력서용 핵심 bullet

- Binance BTCUSDT 수천만 건의 원본 체결 데이터를 수집·전처리하고, **50-trade sequence와 price return·size·aggressor side·Δt의 4개 feature**를 사용한 5초 미래 방향 예측용 `numpy.memmap` 입력 구성. 실제 학습/독립 test에 사용한 연속 구간은 각각 약 66분/2시간 33분으로 별도 명시
- `d_model=128`의 **Mamba block 4개와 direction/magnitude multi-task head를 구현**하고 100k train sequence로 Down/No-Trade/Up 3-class 모델 학습
- **약 19분 validation 구간**에서 `p_up` 임계값·수수료 tier별 가상 백테스트를 구현해, 각각 최적 threshold에서 무수수료 **+23.26%**와 Spot 0.1%/side **-99.18%**의 큰 차이를 확인. 겹치는 5초 예측을 독립 거래로 보는 단순화와 선정 편향도 인지
- prediction accuracy만으로 전략을 평가하지 않고 **turnover와 transaction cost까지 포함해 실제 활용 가능성을 검증**, Binance Testnet 자동 거래 실험까지 확장

## 표현할 때 주의할 부분

다음 표현은 피합니다.

- `한 달 동안 학습한 모델을 일주일 내내 독립 테스트했다`
  - 한 달·일주일은 원본 수집 기간이고 실제 첫 시퀀스 제한으로 **학습 66분·test 2시간 33분**에 그쳤습니다.
- `백테스트에서 실제로 4,319회 거래해 +23.26% 수익을 냈다`
  - validation의 사후 threshold 선택과 겹치는 예상 수익률을 독립 거래로 합성한 결과입니다.
- `CNN + Mamba가 최종 모델이었다`
  - CNN은 초기 Upbit LOB prototype 단계이며 최종 Binance 모델은 trade feature projection + 4-layer Mamba입니다.
- `Binance LOB 단계에는 실제 Mamba가 없었다`
  - Upbit prototype의 DummyMamba와 **Binance LOB 실제 Mamba** 코드를 구별해야 합니다.
- `validation accuracy 59.51%`
  - 현재 최종 5초 checkpoint는 54.125%입니다.
- `높은 예측 정확도로 수익을 냈다`
  - independent test accuracy와 fee-aware PnL 모두 이를 지지하지 않습니다.
- `실제 투자에서 수익성을 입증했다`
  - 반대로 실제 비용을 반영했을 때 전략의 한계를 확인한 프로젝트입니다.
- 팀 전체 dashboard / proxy / 다른 AI 모델을 개인 구현으로 표현하지 않습니다.

## 근거/검증 주의

- [원본 자료·실험 결과 작업 원장](crypto-lob-prediction-evidence-ledger.md)에는 **Binance LOB vs trades 모델 코드 비교, 시퀀스 시간 범위, 백테스트 평가 가정, Testnet 연결 코드의 결함**을 증거 수준별로 정리했습니다.
- **실시간 코드의 운영 위험:** [`live_trader.py`](https://drive.google.com/file/d/1WZwfaIWHcs4cFfdfEs3shSEBlhPlMas9/view)에는 Binance API **key/secret을 그대로 콘솔에 출력하는 코드**가 남아 있어 자격 증명 노출 위험이 있습니다. 또 매도 실패 예외 이후에도 내부 `PositionState`를 초기화해 실제 계정과 동기화가 깨질 가능성이 있습니다. **이런 보안/거래 상태 불일치를 해결했다고 주장하지 않습니다.** 원본 자격정보는 본 기록에 복제하지 않습니다.
- **사후 제안이지 당시 완성한 기능은 아님:** temporal purge/embargo, 학습기간 기반 고정 label threshold, walk-forward 평가, test 고정 threshold, offline/live 동일 single-position execution model, 체결/슬리피지/시장 충격 및 계정 동기화 검증을 적용해야 실제 전략의 유효성을 더 엄격히 평가할 수 있습니다.

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
