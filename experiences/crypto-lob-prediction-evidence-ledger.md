# Crypto Microstructure Prediction (MAMBA Trading) — 원본 자료 복원 원장

> **조사일: 2026-10-28.** 본 원장은 기존 [경험 문서](crypto-lob-prediction.md)의 주장을 **보관된 Google Drive 코드·테스트 아티팩트·팀 발표/보고서**와 비교하는 근거 레이어다. 다른 프로젝트처럼 Git commit 중심으로 조사하려 했지만, 현재 GitHub 연결에서 **이 프로젝트 저장소를 확인하지 못했다**. 따라서 파일의 기록과 저자 귀속을 Git commit 수준으로 검증했다고 주장하지 않는다. `resume.md` 및 canonical 경험 문서는 현재 변경하지 않는다.

## 1. 저장소 발견 상태와 원본 소스

- **GitHub(현재 검색 범위):** `nanocode00` 소유로 접근 가능한 41개 저장소, `collaborator` 5개 및 `organization_member` 6개의 목록에서 MAMBA/암호화폐 예측 프로젝트 전용 저장소로 식별 가능한 repo 없음. 고유 파일명 `backtest_long_only_5s_fut_limit_0p02` 등 공개 코드 검색에도 관련 repo를 확인하지 못함. **팀 소유 비공개 저장소·다른 명칭의 저장소·삭제된 저장소의 부재는 증명하지 않음.**
- **원본 코드 복원 1:** [Google Drive `binance_lob`](https://drive.google.com/drive/folders/1Rm8U-zoJH-k-fnxFvopvroPxEaUHRzHR). `model.py`, `train.py`, `preprocess.py`, `features.py`, `collector.py`, `eval.py`, `backtest.py`, `live_trader.py` 등이 존재. 파일 metadata의 원본 수정 시점은 주로 **2025-11-29**; Drive 반입 시점은 2026년 1월.
- **원본 코드 복원 2:** [Google Drive `binance_trades`](https://drive.google.com/drive/folders/1yPWmW-uCDGbcb0PJanmxCnc1WotFvIKd). `model.py`, `train.py`, `preprocess.py`, `backtest.py`, `collector.py`, `eval.py`, `live_trader.py`, `live_trader_all.py`, `run_pipeline.py`, `config.py` 및 `checkpoints/`, `preprocessed/`, `val/`, `logs/` 확인. 파일 원본 수정 시점은 2025-11-29~12-08.
- **당시 팀 증거:** [`AI_TEAM3.pdf`](https://drive.google.com/file/d/1MtXa-DS2V001RNnYsnh81lfpwgeg8VVN/view), [`AI Technical Report Team3.docx`](https://docs.google.com/document/d/1-RXYj-HtgRPpMhGoXC80O-EBpg99pAyI/edit). MAMBA 담당을 개인 이름으로 표시하며, 4개 모델과 공통 Testnet/대시보드 구조를 팀 차원에서 비교한다.
- **귀속 경계:** Drive의 소스·모델은 해당 팀 프로젝트 자료임을 확인할 수 있지만 **파일별 commit author, 특정 변경의 원작성자, 정확한 버전 관리 이력은 미확정**이다. 팀 대시보드/Proxy를 개인 단독 구현으로 표기하지 않는다.

## 2. 작업 단위 CR-01 — LOB 기반 실험 → Binance trade 기반 최종 모델

**상황:** 2025년 말 BTCUSDT 마이크로스트럭처의 짧은 미래 가격 방향을 예측하는 팀 과제.

**소스 확인(A):**
- [`binance_lob/model.py`](https://drive.google.com/file/d/1JvhlZWE5xpd--zhVTlTzkLw-Qo0DN7pK/view)에 `LOBEncoder`, `MambaBlock`, `LOBMambaBinary`. Binance order book `[B,T,2,N_LEVELS,N_FEATS]`를 CNN으로 인코딩한 뒤 `mamba_ssm.Mamba`를 사용하는 **binary up/down 모델**을 정의. 현재 남은 이 파일은 **실제 Mamba 구현**이므로, 기존 경험 문서에 적힌 *이전 Upbit prototype의 GRU `DummyMamba`*와는 별도 시점/변형으로 구분할 것.
- [`binance_trades/model.py`](https://drive.google.com/file/d/1bBes1NU8BoK6aP_7mvf9TXfhEeArEAh_/view)에는 `MambaBlock`, `TradesEncoder`, `TradesMambaTernary`의 **3-class 방향 헤드 + 미래 수익률 magnitude 회귀 헤드**. 실제 `mamba_ssm.Mamba`를 사용.
- 2025년 최종 팀 보고서도 **5초 미래 방향**, tick trade 기반 `price/volume/side/inter-trade-time`과 확률 임계값 기반 거래, 수수료 tier별 평가를 설명.

**결과:** LOB 이미지형 CNN encoder + binary Mamba 변형과 trade 4-feature + ternary multi-task Mamba 최종 변형 **두 소스가 서로 다르게 존재**한다는 사실을 확인. `Upbit LOB→Binance LOB→Binance aggTrade`의 정확한 수정 시점·실험별 선택 근거는 Git 이력이 없어 복원 보류.

## 3. 작업 단위 CR-02 — 대용량 trade 전처리·학습·평가

**직접 코드:**
- [`binance_trades/preprocess.py`](https://drive.google.com/file/d/1uSFTg1y9f7g3qqPQ6eLRsZpPpSMDhNuL/view): JSONL에서 timestamp/price scan → `np.searchsorted(times,t_now+horizon)`로 미래 가격/return 산출 → 수익률의 `LABEL_Q_LO/HI` 분위수로 Down/No-Trade/Up 라벨 → 각 시점 50개 trade sequence `np.memmap` 생성. 100,000 train/validation sequence와 200,000 test sequence의 실제 `X.dat`·`y.dat`·`ret.dat` 메타 아티팩트가 [`preprocessed/5s`](https://drive.google.com/drive/folders/1GXPb_UgQ-9EveZFc5Ek1YB2Zwf6yGhGH)에 존재한다.
- **좋은 검증 습관:** train/val 정규화는 train 80% 구간의 mean/std를 사용하고, test에 동일 mean/std를 재사용하도록 구현됨.
- **추가 확인할 설계 제약:** `build_3class_labels_and_returns`의 분위수 임계값을 `run_preprocess_for_split(split=...)`마다 **train/test 별도로 계산**한다. 즉 `train label`과 `test label`의 가격 return threshold가 같다고 볼 수 없어, out-of-sample label 의미 비교 시 주의. 이를 단순 '예측 시 정답 누출'로 규정하지는 않고 **서로 다른 라벨 기준을 사용하는 평가 설계**로 기록한다.
- [`binance_trades/train.py`](https://drive.google.com/file/d/1SdED1giUjwL7r2ovHnfS9r4R6zGQsxxR/view): `MemmapDataset`, temporal train/val 경계, class-frequency weighted CrossEntropy + `0.3×MSE(magnitude)`, AdamW, `MAX_EPOCHS=50`, patience 5, validation accuracy 기준 `best_model.pt` 저장. [`checkpoints/5s/best_model.pt`](https://drive.google.com/drive/folders/1aB4te9dq6UVht0SDCAhigBjhq5OpHnim)에 실제 모델 파일이 존재. **비신뢰 pickle 로드 실행 없이 파일 메타만 확인**.
- 기존 경험 문서의 `val_acc=0.54125`·`best_epoch=17`은 **이전 자료 검토에서 기록한 값**으로 현 조사에선 체크포인트 바이너리를 실행/역직렬화해 재검증하지 않았다.

**독립 test의 직접 원본 지표(A):**
- [`val/5s/test`](https://drive.google.com/drive/folders/1OCk2D-5_y4jVaaPoOjvg_Jafh4jGIUR9)에서 `y_true.npy`·`y_pred.npy`를 다운로드하고 **NumPy `allow_pickle=False`로 직접 대조**.
- `N=200,000`, **accuracy 0.457895 = 45.7895%**. True class count: Down **47,747**, No-Trade **111,276**, Up **40,977**. 예측 count: Down **39,417**, No-Trade **116,173**, Up **44,410**.
- No-Trade만 예측하는 **단순 최빈 클래스 기준 정확도 55.638%**이므로 단순 accuracy에서 최종 Mamba의 **45.79%는 그보다 낮다**. 그러나 이 모델은 클래스 가중치를 쓰고 거래 signal을 겨냥하므로 accuracy 하나를 전체 모델 가치 판단으로 쓰지 않고 precision/recall, calibration, 거래 손익과 함께 본다.

## 4. 작업 단위 CR-03 — 수수료 민감도와 백테스트 제약

**직접 코드:** [`binance_trades/backtest.py`](https://drive.google.com/file/d/1VTL4mIjBsFTY-n8xmKN04EbqyywUlM6i/view)는 기본적으로 `val/5s/train/y_prob.npy`와 `ret.npy`를 읽고, **학습 원본에서 분리한 20,000개 validation 예측**의 `p_up >= threshold` 0.50~0.90를 sweeping한다. 각 threshold에 대해 매 신호의 `ret - (2×side_fee)`를 자본에 곱하고 마지막 equity가 최대인 threshold를 선택한다.

**Drive에 보존된 실제 CSV 원문(A):** [`val/5s/train`](https://drive.google.com/drive/folders/1BtXnKBO_p9iMIJbVHmKX08FdJyTVfqBj)의 정책별 summary CSV를 직접 비교.

| 정책 / side fee | 최선 threshold | 거래 신호 수 | 최선 final equity (초기 $10,000) | 표시 return |
|---|---:|---:|---:|---:|
| [수수료 없음](https://drive.google.com/file/d/1W2_kiErd35IUnOaC8N2v4sAgJRGS3TeH/view) | 0.50 | 4,319 | $12,326.05 | **+23.2605%** |
| [Spot 0.10%/side](https://drive.google.com/file/d/1hX3faKVx0dLhM9Z_sX2s9AwDTk-KZzs5/view) | 0.90 | 2,458 | $81.57 | **-99.1843%** |
| [Futures market 0.05%/side](https://drive.google.com/file/d/1soNnTvJfVPBTjTWoMFBsuveS4GiYzgr8/view) | 0.90 | 2,458 | $956.24 | **-90.4376%** |
| [Futures limit 0.02%/side](https://drive.google.com/file/d/1oeD1Tps6-DyW3QarLhiubxm1I1KsqxXK/view) | 0.90 | 2,458 | $4,182.96 | **-58.1704%** |

**표의 엄격한 해석:**
1. **위 백테스트는 독립 test set이 아니라 validation 구간의 threshold/fee sweep**이다. 또한 각 정책에서 **다른 최선 threshold를 선택**했으므로 no-fee 0.50과 Spot 0.90을 '동일 threshold 전략을 그대로 실행한 비교'라고 서술하면 안 된다. 같은 prediction/return sample 기반의 **수수료 정책 민감도 분석**이다.
2. 신호가 나올 때마다 **전체 자본의 독립적인 왕복 거래**로 간주하고 각 미래 5초 return을 합성해 복리 계산한다. 겹치는 예측 구간, 이미 포지션이 열린 상태, 매수/매도 체결·slippage·시장충격 제약은 코드에서 별도 포지션 상태로 모델링되지 않는다. 따라서 *현실적으로 실행 가능했던 투자 수익률*로 표현하지 않는다.
3. 최종 보고서의 Futures limit `-51.88%`와 CSV의 `-58.17%`는 **불일치**, 해당 보고서/시뮬레이션 스냅샷의 버전 근거를 찾기 전에는 CSV 결과와 합쳐 하나의 사실로 만들지 않음.
4. 정확도와 무수수료 손익이 좋아 보이는 것과 수수료·turnover로 실제 전략이 유지되는지는 다른 차원이라는 **부정적 결과의 분석 역량**이 이 프로젝트의 강점이다.

## 5. 작업 단위 CR-04 — Binance Testnet 실험

- [`binance_trades/live_trader.py`](https://drive.google.com/file/d/1WZwfaIWHcs4cFfdfEs3shSEBlhPlMas9/view)와 [`live_trader_all.py`](https://drive.google.com/file/d/1ugCaObbbO6-3uC7vonWG6rXXaS3Cr-2Q/view)가 존재. `LiveTraderTrades`의 model checkpoint·train meta 로드, stream buffer, threshold, position state, 거래 수수료/로그 구조를 확인. **API 키나 비밀값을 원장/PR에 복제하지 않음**.
- [`logs/live_trades_stats_5s_th0.78.csv`](https://drive.google.com/file/d/1H-edFprZ5WujpBUXGEAbFjcPQqyEqSlf/view)는 실행 기록 파일로 존재. 완전한 체결 장부·실제 계정 결과와 일치하는지는 별도 대조 필요.
- 최종 [팀 기술보고서](https://docs.google.com/document/d/1-RXYj-HtgRPpMhGoXC80O-EBpg99pAyI/edit)는 **2025-12-06~08 약 2.5일, 1 BTC + 10,000 USDT 시작**, MAMBA **-98.3%** 및 다른 전략 계정과 공동 React dashboard의 비교를 기술한다. **팀 보고서가 언급한 결과**와 개인 live script의 귀속·실제 체결 전량을 독립 확인했다는 주장은 구분한다.

## 6. 5단계 경험 재구성 후보

| 경험군 | 상황 → 문제 → 판단 → 조치 → 결과 (현재 확인) |
|---|---|
| **CR-01 모델 아키텍처 진화** | 짧은 trade 미세구조 예측 → LOB 구조와 trade 기반 입력의 선택 → 기존 CNN-LOB + Mamba binary 소스와 4-feature trade Mamba ternary/magnitude 소스 존재 → 전처리·구현 비교 가능 → **전환 이유·시점은 Git 부재로 아직 불명확** |
| **CR-02 대규모 데이터/모델** | 수천만 체결 데이터에서 5초 label·시퀀스 생성 → 메모리/클래스 불균형 → memmap, train/val/test, weighted loss·magnitude head → 독립 test **45.7895%** 확인, 다수 클래스 기준보다 낮음 → **정확도 성과로 과장 금지** |
| **CR-03 실행 제약 평가** | validation 구간 무수수료 시뮬레이션에선 +23.26% → 거래 빈도 대비 작은 5초 신호, fee 불리함 → 동일 샘플로 수수료별 sweep → Spot 최선 -99.18%, Futures market -90.44% → **offline validation 전략 비교, 실제 독립 수익률 아님** |
| **CR-04 팀 Testnet 시스템** | 오프라인 외 실거래 유사 상황 필요 → 모델/주문/계좌 상태 분리 → Python live trader 및 팀 Testnet·Proxy/대시보드 연결 → 팀 보고서 MAMBA -98.3% 기록 → **전체 플랫폼은 팀 공동 작업, 모델별 파일 저자 미확정** |

## 7. 남은 조사·검증

1. GitHub 프로젝트 저장소는 현재 연결된 개인/협업/조직 저장소에서 **찾지 못함**. Drive 원본 폴더·보고서를 우선 출처로 채택. 팀원 계정에 별도 비공개 Git이 있었는지는 미확인.
2. `binance_lob`와 `binance_trades` 두 코드베이스의 파일별 수정 차이/학습 목적·전환 판단을 추가 비교하고, 가장 이른 Upbit `DummyMamba` 별도 코드가 있다면 시점 구분.
3. Validation backtest의 겹치는 5초 horizon에 따른 포지션/비용 평가 왜곡 위험, label quantile 별도 계산, threshold 탐색 편향을 **설계의 검증 한계**로 명시. 재실행이 가능하다면 test set/비중복 실행 규칙으로 사후 검증하되, 현재 당시 실적과 섞지 않음.
4. 원자료 성능 숫자를 확인한 뒤 기존 [canonical 문서](crypto-lob-prediction.md)의 '독립 test', backtest 유효성, 역할 귀속을 정밀 보강. `resume.md`는 최종 선정 전 미변경.
