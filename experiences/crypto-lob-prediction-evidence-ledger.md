# Crypto Microstructure Prediction (MAMBA Trading) — 원본 자료 복원 원장

> **조사일: 2026-10-28.** 보존된 Google Drive 코드·테스트 아티팩트·팀 발표/보고서와 **2026-10-28 사용자 직접 회고(C)**를 증거 수준별로 대조한다. 현재 GitHub 연결에서 프로젝트 저장소를 확인하지 못했으므로 파일별 commit 저자 귀속은 검증되지 않았다. 하지만 본인이 **MAMBA 전략 관련 전체 작업을 직접 담당했다는 회고(C)**와 팀 보고서의 전략 담당 표시(B)는 확보했다. 본 원장과 [canonical](crypto-lob-prediction.md)은 Draft PR #8에만 반영하며 `resume.md`는 변경하지 않는다.

## 1. 저장소 발견 상태와 원본 소스

- **GitHub(현재 검색 범위):** `nanocode00` 소유로 접근 가능한 41개 저장소, `collaborator` 5개 및 `organization_member` 6개의 목록에서 MAMBA/암호화폐 예측 프로젝트 전용 저장소로 식별 가능한 repo 없음. 고유 파일명 `backtest_long_only_5s_fut_limit_0p02` 등 공개 코드 검색에도 관련 repo를 확인하지 못함. **팀 소유 비공개 저장소·다른 명칭의 저장소·삭제된 저장소의 부재는 증명하지 않음.**
- **원본 코드 복원 1:** [Google Drive `binance_lob`](https://drive.google.com/drive/folders/1Rm8U-zoJH-k-fnxFvopvroPxEaUHRzHR). `model.py`, `train.py`, `preprocess.py`, `features.py`, `collector.py`, `eval.py`, `backtest.py`, `live_trader.py` 등이 존재. 파일 metadata의 원본 수정 시점은 주로 **2025-11-29**; Drive 반입 시점은 2026년 1월.
- **원본 코드 복원 2:** [Google Drive `binance_trades`](https://drive.google.com/drive/folders/1yPWmW-uCDGbcb0PJanmxCnc1WotFvIKd). `model.py`, `train.py`, `preprocess.py`, `backtest.py`, `collector.py`, `eval.py`, `live_trader.py`, `live_trader_all.py`, `run_pipeline.py`, `config.py` 및 `checkpoints/`, `preprocessed/`, `val/`, `logs/` 확인. 파일 원본 수정 시점은 2025-11-29~12-08.
- **당시 팀 증거:** [`AI_TEAM3.pdf`](https://drive.google.com/file/d/1MtXa-DS2V001RNnYsnh81lfpwgeg8VVN/view), [`AI Technical Report Team3.docx`](https://docs.google.com/document/d/1-RXYj-HtgRPpMhGoXC80O-EBpg99pAyI/edit). MAMBA 담당을 개인 이름으로 표시하며, 4개 모델과 공통 Testnet/대시보드 구조를 팀 차원에서 비교한다.
- **직접 회고(C, 2026-10-28):** **해당 MAMBA 전략의 데이터 수집·전처리, 모델·학습·평가, 백테스트, 실시간 거래 연동은 모두 본인이 구현**했다. 여러 전략에서 프록시로 전달한 결과를 **모아 보여주는 최종 웹사이트는 다른 팀원이 개발**했다. 팀 공통 프록시 서버 자체의 구현 범위는 이번 회고로 확정되지 않는다.
- **귀속 경계:** 이 직접 회고(C)는 당시 팀 자료의 담당 표기(B) 및 전략 소스(A)와 부합하지만 **파일별 commit author·원작성자·버전 계보에 대한 기계적 검증은 여전히 불가**하다. 개인 단독 전략 구현과 타인이 만든 최종 웹사이트를 분리한다.

## 2. 작업 단위 CR-01 — LOB 기반 실험 → Binance trade 기반 최종 모델

**상황:** 2025년 말 BTCUSDT 마이크로스트럭처의 짧은 미래 가격 방향을 예측하는 팀 과제.

**소스 확인(A):**
- [`binance_lob/model.py`](https://drive.google.com/file/d/1JvhlZWE5xpd--zhVTlTzkLw-Qo0DN7pK/view)에 `LOBEncoder`, `MambaBlock`, `LOBMambaBinary`. Binance order book `[B,T,2,N_LEVELS,N_FEATS]`를 CNN으로 인코딩한 뒤 `mamba_ssm.Mamba`를 사용하는 **binary up/down 모델**을 정의. 현재 남은 이 파일은 **실제 Mamba 구현**이므로, 기존 경험 문서에 적힌 *이전 Upbit prototype의 GRU `DummyMamba`*와는 별도 시점/변형으로 구분할 것.
- [`binance_trades/model.py`](https://drive.google.com/file/d/1bBes1NU8BoK6aP_7mvf9TXfhEeArEAh_/view)에는 `MambaBlock`, `TradesEncoder`, `TradesMambaTernary`의 **3-class 방향 헤드 + 미래 수익률 magnitude 회귀 헤드**. 실제 `mamba_ssm.Mamba`를 사용.
- 2025년 최종 팀 보고서도 **5초 미래 방향**, tick trade 기반 `price/volume/side/inter-trade-time`과 확률 임계값 기반 거래, 수수료 tier별 평가를 설명.

**결과(A+C):** LOB CNN encoder+binary Mamba와 Trades 4-feature+ternary multi-task Mamba가 서로 다른 코드로 존재(A). **당시 프로젝트의 LOB 수집 방식으로는 실시간 자료만 이용할 수 있어 과거 학습 데이터를 확보하기 어려웠고, 과거 데이터도 사용할 수 있는 Trades로 전환했다는 본인 회고(C)**를 확보했다. 이는 모든 거래소에서 역사적 LOB 자료가 원천적으로 없었다는 뜻이 아니다. 'LOB 성능이 낮아 모델을 바꿨다'고 해석하지 않는다. 구현 계보·구체적 변경일은 여전히 Git 이력이 없어 완전히 확인되지 않는다.

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

## 6. 상황 → 문제 → 판단 → 조치 → 결과 : 증거 원장과 판정

> Canonical [핵심 경험 4개](crypto-lob-prediction.md#핵심-경험-4개--상황--문제--판단--조치--결과)에 각 항목을 **다섯 단계로 완성**했다. 아래는 판단·실행·최종 관측의 증빙 한계를 보존하는 용도의 압축 원장이다. 특히 **보존된 코드가 선택을 증명하는 것**과 **당시 개발자가 그 선택을 하게 된 이유를 증명하는 것**을 구분한다.

### CR-01 / 경험 1 — LOB에서 Trades 모델로 변화

- **상황:** 짧은 BTCUSDT 가격 방향을 예측하는 LOB 기반 모델 개발(A/B).
- **문제:** 당시 **본인이 접근한 LOB 데이터는 실시간 수집에 국한되어 과거 훈련 자료를 충분히 확보할 수 없었음(C)**. LOB 모델이 부정확해서 폐기했다는 근거는 없음.
- **판단:** **과거 체결 데이터를 구할 수 있는 Trades로 바꿔** 학습용 과거 자료를 활용하기로 결정(C). CNN+Mamba LOB와 Linear+Mamba Trades의 서로 다른 구조는 보존 코드(A)로 확인.
- **조치:** 기존 LOB 입력 대신 50개 체결의 price return·log-volume·aggressor·log-Δt, 3-class+magnitude Mamba 파이프라인 구현(A+C).
- **결과:** 최종 Trades 모델과 test 아티팩트 존재(A), 데이터 소스 전환 당시 이유 확보(C). **동일 조건의 LOB vs Trades 성능 개선 수치는 없음**.
- **판정:** 데이터 접근성·전환 동기=C, 구현된 최종 구조=A, 개인 전략 구현 범위=C. 변경 세부 날짜/LOB 정확도 비교는 미확정.

### CR-02 / 경험 2 — 메모리 제약이 큰 체결 데이터를 입력 시퀀스로 가공

- **상황:** train 원본 약 38M, test 원본 약 10M tick-trade를 50개 시퀀스·5초 미래 수익률로 변환.
- **문제:** 중첩 시퀀스가 매우 커 실제 **메모리 문제를 겪어 개수 제한이 필요했다는 회고(C)**. 발생한 오류 로그나 최대 RAM 사용량은 확보되지 않음.
- **판단:** **메모리 문제로 10만 train-origin·20만 test 시퀀스로 제한했다는 당시 선택 이유(C)**와 상수·two-pass·`memmap` 구현(A)이 일치. 정확히 그 숫자를 정한 근거나 다른 대안 비교는 미확정.
- **조치:** raw JSONL scan, `searchsorted` 수익률·분위수 label, memmap X/y/ret/time, temporal train/val 및 test 저장(A).
- **결과:** 100k train-origin 중 **80k train(약 66분)/20k val(약 19분)**, 별도 200k test(약 2시간 33분); test 3-class accuracy **45.7895%**, 최빈 예측 baseline **55.638%**(A). 훈련 한 달/평가 일주일 전체를 사용하지 않았다는 제한은 **2026년 재검토(D)**이며 당시 이 문제를 해결했다고 기록 금지.
- **판정:** 메모리 제약으로 제한한 이유=C, 파이프라인·제한값=A, 실제 관측 데이터의 짧은 시간 범위와 평가 한계는 사후 확인(D). 특정 OOM 예외·정확한 메모리 규모는 미확정.

### CR-03 / 경험 3 — 수수료 때문에 기대 손익이 뒤집히는 문제

- **상황:** `p_up`로 LONG 진입하는 전략의 비용 민감도 실험.
- **문제:** 작은 5초 예상 수익 대비 높은 거래 빈도 및 왕복 수수료.
- **판단:** validation 확률 threshold 0.50~0.90와 fee tier를 나눠 sweep하도록 구현(A). 그 설계 선택 자체는 확인되지만, 독립 테스트 수익성 입증을 위한 계획이었다고 확대할 수 없음.
- **조치:** `val/5s/train` 20k(약 19분)의 미래 return에 각 신호별 `ret - 2*side_fee`를 계산하고, **각 정책의 최고 `final_equity`에 해당하는 threshold**를 사후 선택(A).
- **결과:** **무수수료 +23.2605%(th 0.50)**, **Spot -99.1843%(th 0.90)**, **Futures market -90.4376%(th 0.90)**라는 validation 가상 복리 결과(A). 이는 **같은 threshold로 실행한 거래 전략의 실제 성과가 아님**. 5초 중복 horizon·포지션 상태 미반영·동일 validation의 tuning/result 재사용은 **사후 한계 확인(D)**.
- **판정:** *수수료 민감도 분석*은 대표 경험 가능; *실제 수익성 실현* 성과는 불가.

### CR-04 / 경험 4 — Testnet 주문과 오프라인 평가의 분리

- **상황:** 팀은 MAMBA 포함 여러 모델을 Binance Testnet/공동 대시보드와 연결(B).
- **문제:** 모델 확률을 실제 상태·주문 처리와 연결해야 하고, offline 단순 백테스트와 중복 포지션 규칙이 달랐음(A). 당시 특정 주문 실패 사고 미확인.
- **판단:** MAMBA 전략의 학습·추론·백테스트·실시간 연결을 직접 담당했다는 회고(C), `paper/testnet/live`·포지션 재진입 금지 등 코드상 선택(A). 특정 주문 정책을 선택한 동기는 미확정.
- **조치:** `live_trader.py`의 checkpoint·실시간 trade stream·`p_up` threshold·5초 포지션 관리와 별도 `live_trader_all.py` all-in 변형 구현(A+C). **각 전략에서 프록시로 보낸 결과를 취합·시각화한 최종 사이트는 다른 팀원 개발(C)**. 프록시 자체의 구체적 소유권은 보류.
- **결과:** 팀 보고서상 MAMBA Testnet **-98.3%**(B). 정확한 실행 스크립트 revision과 개별 체결 장부 미확인. API secret 콘솔 출력·매도 실패 뒤 포지션 초기화 위험은 2026년 **정적 코드 검토(D)**이며 실제 사고/수정 완료로 과장 금지.
- **판정:** 보존 코드=A, Testnet 관측치=B, MAMBA 전략 전체 직접 담당/최종 웹사이트 타인 구현=C. 파일별 commit 저작자 및 최종 실행 revision은 미확정.

**5단계 적용 완료 기준:** 위 네 건은 모두 다섯 항목이 명시되며, **빈 판단을 그럴듯한 가상 결정으로 채우지 않음**. 원본 자료로 확인 가능한 선택과 사후 개선(D)을 분리해 [canonical 서술](crypto-lob-prediction.md#핵심-경험-4개--상황--문제--판단--조치--결과)에도 동일 반영했다.

## 7. CR-05 — Binance LOB와 Binance trades 소스 직접 비교

**source A: [`binance_lob/features.py`](https://drive.google.com/file/d/19axTVp_P6TDrt45I4qOG2tLgr5TRfvmy/view) + [`model.py`](https://drive.google.com/file/d/1JvhlZWE5xpd--zhVTlTzkLw-Qo0DN7pK/view)** (파일 수정 2025-11-29). 호가창 입력은 `[B,T,2,N_LEVELS,N_FEATS]`이며 bid/ask 각각의 (mid 대비 상대가격, side별 수량 정규화, level별 imbalance) **3개 특징**을 배치. `Conv2d(2→32→32→128)`와 `AdaptiveAvgPool2d(1,1)`을 각 시점에 적용한 뒤 `mamba_ssm.Mamba × 4`와 **2-class Down/Up 분류 head**로 연결한다. `train.py`는 `MemmapLOBDataset`, weighted CrossEntropy, Adam과 `ReduceLROnPlateau`를 사용.

**source B: [`binance_trades/features.py`](https://drive.google.com/file/d/1hAz69aJLXAsstKCqvNt9cRtE7cwt-Az_/view) + [`model.py`](https://drive.google.com/file/d/1bBes1NU8BoK6aP_7mvf9TXfhEeArEAh_/view) + [`train.py`](https://drive.google.com/file/d/1SdED1giUjwL7r2ovHnfS9r4R6zGQsxxR/view)** (원본 수정 2025-12-05경). 입력 `[B,T,4]`의 (1) 전 체결 대비 price return 2% clip (2) `log1p(qty)` (3) buy/sell aggressor ±1 (4) `log1p` inter-trade Δt 최대 10초 clip. `Linear(4→128)`→`Mamba×4`→`3-class Down/No-Trade/Up` head와 `|future_return|` magnitude head. Weighted CrossEntropy + `0.3*MSE`, AdamW와 best-val-accuracy checkpoint 선택.

| 차원 | Binance LOB 소스 A | Binance trades 소스 B |
|---|---|---|
| 데이터 | 호가창의 bid/ask 양측 snapshot | 체결 스트림 price·qty·방향·시간 |
| 공간 인코더 | 2D CNN + avg-pooling | 4→128 linear |
| 예측 목표 | 2-class 방향 | 분위수 3-class 방향 + magnitude |
| 공통 부분 | `mamba_ssm.Mamba` residual/LayerNorm, 여러 시점 memory | 동일 Mamba 계열 구현 |
| 결과 경계 | 그 시점의 유효 정확도 수치 미검증 | `5s` checkpt/val/test/거래 산출물 있음 |

**전환 이유의 증거 수준 구분:** 보존된 코드와 수정일만으로는 전환 동기를 알 수 없지만, **당시 LOB 접근 방식은 실시간 수집에 한정되어 과거 학습 데이터 확보가 어려웠고 과거 Trades로 전환했다는 회고(C)가 추가 확보**됐다. 이는 해당 과제의 데이터 확보 조건이지 보편적인 LOB 데이터 제공 불가능성이 아니다. 기존 Upbit `DummyMamba`는 별도 구버전, Binance LOB 코드의 Mamba는 실제 구현(A)이다. **성능 문제 때문에 바꿨다는 주장 금지**.

## 8. CR-06 — 실제 학습/검증에 사용된 timestamp 구간과 평가 편향

**결정적 원본 파일 직접 대조(A, 2026-10-28):**
- [`binance_trades/preprocess.py`](https://drive.google.com/file/d/1uSFTg1y9f7g3qqPQ6eLRsZpPpSMDhNuL/view)는 전체 raw 거래의 시간을 훑어 future return을 구하되 `MAX_TRAIN_SEQ=100000`/`MAX_TEST_SEQ=200000`으로 **두 번째 pass가 첫 유효 시퀀스부터 지정 건수에서 중단**한다. 따라서 '학습 데이터 수집 1개월' ≠ '모델이 실제 학습한 1개월 전체 기간'이다.
- [`preprocessed/5s/train/time.dat`](https://drive.google.com/file/d/1RCf5yQKSiZNRGulj-gIbERf63Nb-xviJ/view)은 `float64` 100,000개. **2025-10-22 00:00:00.716~01:25:23.763 UTC**. 앞 **80,000개 학습**은 00:00:00.716~01:06:20.711 (약 **66분 20초**); 뒤 **20,000개 validation**은 01:06:20.711~01:25:23.763 (약 **19분 3초**).
- [`preprocessed/5s/test/time.dat`](https://drive.google.com/file/d/1sOgWdsEf6Q0ze0UthqwdDz-Yh05i35WK/view)은 `float64` 200,000개. **2025-11-21 00:00:00.888~02:33:15.322 UTC (약 2시간 33분 14초)**. 독립 test 범위는 원본 1주 데이터에서 **앞부분만 선별한 연속 구간**.
- [`val/5s/train/time.npy`](https://drive.google.com/file/d/1NZFZcOAXq_UgL1yDyUuYL2FV3z6UxUpC/view)은 위 **20,000개 validation timestamp**와 같은 01:06:20.711~01:25:23.763 UTC 구간. 세 파일 모두 Python NumPy로 바이너리 배열을 실제 읽어 길이·첫/끝 시간·단조성을 검사했음. 재학습은 하지 않음.

**중요한 결론:** 정확도 45.7895%는 '독립 test 200,000 trade sequences'의 평가 지표지만 **11/21 약 2시간 33분 표본**이지 '일주일 전체에서 독립 테스트한 성능'으로 일반화해서는 안 됨. 백테스트 +23.2605%는 **10/22 약 19분 validation 구간**의 overlapping hypothetical 거래 합성값으로, 원자료가 한 달인 사실만으로 *1개월 수익률*로 설명하면 잘못.

**구조적 평가 제약(A):**
1. `run_preprocess_for_split`은 `train`과 `test` **각각의 raw 전체 수익률 분포에서 q20/q80**을 계산해 3-class 경계를 정한다. **train 100,000 첫 시퀀스만이 아니라 해당 split 전체 raw 가격의 미래 정보가 label threshold 정의에 반영**된다. 본문에서 '훈련 시점에 test 정답을 feature로 직접 넣었다'는 표현은 금지하되, **future distribution을 사용한 사후 라벨 기준 및 train/test 간 라벨 의미 차이**를 평가 설계 한계로 기록.
2. **Train/validation의 경계가 연속:** 마지막 train과 첫 val의 timestamp가 동일(01:06:20.711); 각 sample은 직전 50 trade window와 미래 5초 return이므로 **경계에 겹치는 feature/target 시간 범위**가 발생할 수 있다. `purge/embargo` 적용 코드가 보이지 않으며, val은 완전히 격리된 rolling-origin 검증으로 간주 불가.
3. 최선 `p_up` 임계값은 **그 validation 표본에서 평가된 총자본 증가율을 최대화**하여 사후 선택. 같은 표본의 최대 수익률을 보고하는 **threshold selection 편향**. 비용 정책별 최적 threshold가 다르므로 같은 trading policy를 동일 threshold로 비교한 것이 아님.
4. [`backtest.py`](https://drive.google.com/file/d/1VTL4mIjBsFTY-n8xmKN04EbqyywUlM6i/view)는 `mask=(p_up>=th)`인 **모든 시점**마다 `capital *= 1+ret-2*fee`하며 기존 포지션 오픈 여부 확인, 시간별 중복 5초 horizon 제거, 주문 수량·slippage·market impact·현실적인 손실 상한을 모델링하지 않는다. **실행 가능한 4,319회 체결/실현손익을 의미하지 않는다.**
5. 반대로 [`live_trader.py`](https://drive.google.com/file/d/1WZwfaIWHcs4cFfdfEs3shSEBlhPlMas9/view)는 `PositionState.is_open`이면 새 진입을 막고 `entry_ts+5초` 후 청산한다. 따라서 **offline backtest의 무조건 중복 신호 처리와 live 주문 전략은 서로 다른 포지션 로직**. 백테스트 결과를 Testnet 주문 PnL 예측값으로 취급하지 않는다.

**실제 위험한 구현 세부사항(A, 로그에 자격정보를 옮기지 않음):** `live_trader.py:create_binance_client()`는 `get_binance_credentials()` 결과의 **API key와 secret을 그대로 `print`**하는 코드가 남아 있다. 로그 유출 위험이 있으므로 **즉시 제거할 안전 결함**으로 분류. `close_position`은 sell 주문 예외 이후에도 `PositionState()`로 재설정하므로, 실패한 청산 주문에 대한 계정 포지션 상태와 내부 기록이 불일치할 가능성도 있음. 당시 실제 보안 사고나 청산 실패가 발생했다고 주장하지 않는다.

**사후 재현 개선안(D, 과거 구현/성공 아님):** 전체 기간을 날짜/거래소 상황별 고르게 표집하거나 rolling walk-forward 분할; label quantile을 오직 당시 train context로 고정; horizon+lookback 기준 purge/embargo; threshold는 별도 validation으로만 선택하고 test 전체 구간에 한 번 고정 적용; offline/live 동일한 `single-position` execution engine과 체결·수수료·slippage·최소 주문 단위·실패시 재동기화 포함; per-run 데이터 시간범위/코드 SHA/수수료/라벨 임계값 기록; 실시간 API secret은 무조건 마스킹하고 주문 후 실제 잔고와 재조정.

## 9. 남은 조사·검증 및 canonical 반영 상태

1. GitHub 프로젝트 저장소는 현재 연결된 개인/협업/조직 저장소에서 **찾지 못함**. Drive 원본 폴더·보고서를 우선 출처로 채택. 팀원 계정에 별도 비공개 Git이 있었는지는 미확인.
2. **LOB→Trades로 바꾼 판단 이유는 직접 회고(C)로 확보:** 과거 데이터 접근성 문제. 두 코드베이스의 실제 source diff/모델 구조(A)는 대조 완료. 초기 Upbit `DummyMamba` 원본 및 구체적 계보·전환 날짜는 여전히 추가 자료가 있어야 확정.
3. **시간 범위와 평가 편향 추가 확인:** 실제 train 약 66분·val 약 19분, test 약 2시간 33분. 겹치는 5초 예측·50trade window, split별 label quantile, threshold 선택 편향, offline/live 포지션 처리 차이 기록. 사후 개선은 2025년 당시 작업과 분리.
4. **Canonical v1 5단계 정리 완료:** [canonical](crypto-lob-prediction.md)에 네 경험군의 **상황·문제·판단·조치·결과**를 명시하고 사후 사실과 당시 미확정 동기 구분. `resume.md`는 최종 선정 전 미변경.
