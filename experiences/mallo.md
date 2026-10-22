# Mallo — Senior-Friendly Voice Ordering Kiosk

> **말로 경험 복원 재검증 중 (2026-10-22).** 기존 경험기술서의 팀원 실명 귀속 오류를 교정하고 원본 12개 작업군·개인 PR·최종 실험 조건을 재검토하는 단계입니다. 기존의 임시 역할·속도·모델 성능 수치는 상세 검증 완료 전까지 채용 문장으로 단정하지 않습니다. [진행 원장](mallo-rework-checkpoint.md).

## 채용용 경험 요약

고령 사용자의 키오스크 주문 부담을 줄이기 위해 만든 3인 팀 음성 주문 서비스입니다. Team Lead / Product & Integration으로 PRD/TRD와 공통 contract, deterministic Order Engine, Senior-first UI, TTS fine-tuning, STT/NLU 추가 튜닝, multi-model runtime과 배포까지 제품 전체 흐름을 연결했습니다.

- 기간: **2026.06 ~ 2026.08**
- 팀 규모: **3인**
- 역할: **Team Lead / Product & Integration**
- **원본 팀 구성:** 김재훈(`nanocode00`, 팀장), 윤태현(`surf7978`), 류승민(`padong47`). [README](https://github.com/aihuman-7th/proj1-d/blob/main/README.md) 기준. 기존 기록의 '이규헌·이승훈'은 원본 저장소와 불일치하여 교정
- **업무 구분:** [tasks.md](https://github.com/aihuman-7th/proj1-d/blob/main/docs/tasks.md)에 A(Adaptive TTS/Voice UX), B(Local NLU/Order Engine), C(STT/Audio/GUI/Integration) 영역이 있지만, 이 표만으로 **A/B/C를 개별 팀원 이름에 연결하지 않음**. 본인은 팀장으로 파트 간 통합 및 개인 명의 구현·실험을 수행
- 프로젝트 상태: 2026-08-28 feature freeze, 2026-08-30 최종 정리 후 종료/동결
- 기술: Whisper, Qwen, MeloTTS, QLoRA, FastAPI, Python sidecar, Vercel, Cloudflare Tunnel

공식 STT/NLU Lead는 별도로 있었으므로 각 파트 전체를 개인 기여로 주장하지 않습니다. 다만 프로젝트 후반 통합 병목을 해결하기 위해 Whisper/Qwen 학습 자산을 직접 인수해 추가 fine-tuning, candidate sweep, checkpoint 평가와 production 연결을 수행했습니다.

### 상황 → 문제 → 판단 → 조치 → 결과

**상황**  
음성으로 메뉴 추가·수정·삭제·확인·결제를 진행하고, 다시 듣기·천천히 듣기·직원 호출을 함께 제공하는 senior-friendly voice-first kiosk를 3인 팀으로 개발했습니다.

**문제**  
STT/NLU/TTS 정확도만 높여서는 제품이 완성되지 않았습니다. TTS 후보마다 생성시간이 수십 배 차이 났고, MeloTTS와 Qwen NLU는 `transformers` dependency가 충돌했으며, LLM이 주문 상태를 직접 변경하게 하면 Cart 계산/검증을 안정적으로 보장하기 어려웠습니다. 통합 과정에서는 화면 구조 자체도 고령 사용자의 부담으로 남았습니다.

**판단**  
1. 주문 상태는 LLM이 아니라 **deterministic Order Engine을 SSOT**로 관리한다.
2. 모델은 정확도 하나가 아니라 **latency + 품질 + deployment cost**를 함께 보고 선택한다.
3. dependency가 다른 STT/NLU/TTS는 **resident sidecar + separate venv**로 분리한다.
4. 음성 모델과 UI를 별개 문제로 보고 **Senior-first UI**도 함께 수정한다.

**조치**  
PRD/TRD와 NLU command schema, Validator, Cart/Order Engine, TTS contract를 정의하고 Mock E2E를 먼저 구성했습니다. `54131e8` 커밋에서 직접 `REMOVE_ITEM`, `UPDATE_QUANTITY`, `UPDATE_OPTION`, `QUERY_CART`, `CONFIRM_ORDER`, `CANCEL`을 deterministic Order Engine에 구현했습니다. AI Hub 친절체 데이터로 MeloTTS를 fine-tuning하고 Qwen3-TTS와 동일 20문장 benchmark를 수행했습니다. 프로젝트 후반에는 Whisper Medium QLoRA candidate sweep과 Qwen3-1.7B QLoRA 3-epoch 학습도 직접 수행했습니다. 최종 runtime은 Gateway/STT/NLU/TTS를 `8000/8001/8002/8003` resident process로 분리하고 health/warm-up launcher로 관리했습니다.

**결과**  
동일 20문장 GPU benchmark에서 **Melo Base 198.5ms / Melo Friendly 204.1ms / Qwen Base 6.91s / Qwen Friendly 28.20s**를 측정해 **MeloTTS Friendly를 최종 선택**했습니다. Fine-tuning으로 평균 지연은 약 **5.6ms(+2.8%)** 늘었지만 0.2초 수준의 interactive latency를 유지하면서 친절체 억양을 적용했습니다. 이후 `G_2800` production TTS benchmark는 **mean 0.38s / p95 0.53s / RTF 0.0894**였습니다. Vercel + Cloudflare Tunnel + local FastAPI로 배포했고 real STT+NLU + MockTTS 50-turn isolated E2E에서 **p95 312.2ms**를 측정해 STT가 주 latency bottleneck임을 확인했습니다.

## 핵심 수치

| 항목 | 결과 | 의미 |
|---|---:|---|
| 팀 규모 | 3인 | Team Lead / Product & Integration |
| MeloTTS Baseline, 20문장 GPU | **198.5ms mean, RTF 0.0403** | 빠르지만 어조 단조 |
| MeloTTS Friendly `G_2000`, 20문장 GPU | **204.1ms mean, RTF 0.0485** | 최종 모델 후보로 선정 |
| Qwen3-TTS Baseline, 20문장 GPU | **6.9075s mean** | interactive serving에 느림 |
| Qwen3-TTS Friendly, 20문장 GPU | **28.1953s mean** | fine-tuned tone 후보, latency 과다 |
| Production MeloTTS `G_2800` | **mean 0.38s / p95 0.53s / RTF 0.0894** | 최종 runtime 별도 benchmark |
| NLU source benchmark | **198/198 exact match** | final NLU evaluation |
| NLU legacy batch | **1000/1000** | schema parse error 0 |
| isolated 50-turn E2E | **p50 253.9ms / p95 312.2ms** | real STT + real NLU + MockTTS |
| 사용자 평가 | **n=13, 만족 응답 92.3%** | 참가자 전원이 고령자는 아님 |

## 0. 고령 사용자 조사에서 Voice-first + Minimal GUI 우선순위로 — MA-01 (1차 검증)

**상황 → 문제 → 판단 → 조치 → 결과**

**상황:** 음성 주문 키오스크가 고령 사용자에게 실제로 어떤 도움을 줄지 정리하기 위해 초기 60대 응답자 **13명(60~64세 6명·65~69세 7명)**의 키오스크 이용 경험을 확인했습니다.

**문제:** 제품을 Voice-only 또는 무조건 느린 분절형 TTS로 설계하면, 사용자가 겪는 문제인 **옵션 선택의 어려움(8/13), 실수 후 처음부터 다시 주문(7/13), 음성 오인식 불안(6/13), 다시 설명받고 싶은 요구(10/13)**를 충분히 해결하지 못합니다. 표본은 디지털 사용 경험이 높은 60대에 치우쳐 전체 고령층으로 일반화하기 어렵습니다.

**판단:** 기존 STT→NLU→Validator→Order Engine 구조는 유지하면서, **음성으로 탐색 부담을 낮추고 현재 주문의 음성·화면 확인 및 수정·재설명 복구**에 우선순위를 둡니다. 고령 사용자의 상태를 연령/침묵으로 추정하지 않고 **명시적 다시 듣기·천천히 듣기 요청에만 TTS를 적응**시킵니다. LLM이 Cart 금액이나 상태를 직접 변경하지 않게 Order Engine을 SSOT로 둡니다.

**조치:** 개인 명의 [PR #83](https://github.com/aihuman-7th/proj1-d/pull/83)으로 팀이 수집한 초기 설문 분석 결과를 [PRD](https://github.com/aihuman-7th/proj1-d/blob/main/docs/prd.md)의 사용자 문제·P0/P1/P2 우선순위 및 [평가 기록](https://github.com/aihuman-7th/proj1-d/blob/main/docs/worklogs/evaluation.md)에 연결했습니다. 팀 인터뷰 원본의 모집·조사 전 과정을 직접 수행했다고 PR로 증명되는 것은 아닙니다. 초기 [설문 변경안](https://github.com/aihuman-7th/proj1-d/blob/main/docs/survey-pipeline-update.md)의 **UNDO/Pending/Preview**는 기능 제안이며, 최종 MVP에서는 **ORDER_REVIEW의 명시적 주문 확인을 유지하고 별도 Pending Confirmation 상태 머신과 Size 음성 지원은 제외**했습니다.

**결과:** 제품이 막연한 음성 대체가 아니라 **'복잡한 메뉴 탐색은 줄이고, 잘못 알아들어도 화면에서 바로 확인·수정하는 Voice-first + Minimal GUI'** 방향으로 구체화됐습니다. 다만 이 단계의 성과는 **설문 기반 제품 정의와 요구사항 우선순위 확정**이지 실제 주문 시간 단축·사용성 개선의 인과적 검증이 아닙니다. **초기 60대 설문 n=13**과 별도로 기록된 **최종 사용성 평가 n=13 / 만족 응답 92.3%**는 목적·참가자 구성이 다른 평가이므로 합치지 않습니다.

상세: [MA-01 경험 복원 원장](mallo-rework-checkpoint.md) · [설문 반영 PR #83](https://github.com/aihuman-7th/proj1-d/pull/83).

## 1. 공통 contract와 Mock E2E부터 구성

실제 모델이 완성될 때까지 기다리지 않고 STT/NLU/TTS adapter와 Mock implementation을 공통 interface에 맞춰 먼저 연결했습니다.

핵심 contract:

- audio/STT result
- NLU `OrderCommand`
- Validator의 허용/거부 규칙
- Cart / Order Engine state
- ErrorResponse
- TTS request/response

이 구조 덕분에 각 파트가 독립적으로 모델을 바꿔도 전체 주문 flow는 유지할 수 있었습니다.

## 2. MA-02 — LLM 이해와 실제 주문 상태를 분리한 결정론적 Order Engine (검증 완료)

**상황:** 말로는 STT로 인식한 주문 발화를 NLU가 구조화하고, 메뉴 데이터와 장바구니를 실제 주문 코드가 관리하는 Voice-first 키오스크였다. 초기 `ADD_ITEM` 기반 구현만으로는 메뉴 삭제·수량/온도 변경·주문 조회·취소 등 사용자의 수정 요구를 해결할 수 없었다.

**문제:** 모듈마다 NLU `OrderCommand` 필드와 오류 형식, 복수 메뉴 Batch 처리 기준이 달랐고, LLM이 가격·수량·Cart 상태를 직접 변경하면 잘못된 수량·메뉴·온도를 실행하거나 이미 올바른 장바구니를 손상할 수 있었다.

**판단:** 정상 NLU 출력은 **단일 명령도 배열 `OrderCommand[]`**, 실패는 별도 `OrderCommandError`로 규격화한다. 초기 명령의 **6개 공통 필드·누락값 `null`** 규칙을 정하고, **가격·총액·Confirmation Policy는 Menu DB/결정론적 Order Engine이 전담**하게 한다. 복수 `ADD_ITEM`은 **전부 검사 후 복사된 Cart에 반영**하고, 오류 시 기존 Cart를 유지한다. 당시 구현에서는 **혼합 Intent 한 번에 실행을 거부**한다.

**조치:** 개인 명의 [PR #23](https://github.com/aihuman-7th/proj1-d/pull/23)(2026-08-14)에서 메뉴 하나당 명령 하나, 배열 계약, 요청 ID·Intent·target·category·수량·온도 필드, 온도 누락과 복수 메뉴/오류 처리, NLU와 Order Engine 책임을 문서로 고정하고 PRD·TRD를 맞췄다. 이 PR은 **7개 Markdown 변경의 설계 작업**으로, JSON Schema validator 실제 구현은 별도이다.

개인 [PR #55](https://github.com/aihuman-7th/proj1-d/pull/55)(2026-08-18)에서는 `src/domain/conversation.py`의 `ConversationAgent`에 `REMOVE_ITEM`, `UPDATE_QUANTITY`, `UPDATE_OPTION`(HOT↔ICED), `QUERY_CART`, `CONFIRM_ORDER`, `CANCEL` **6개 주문 관리 Intent**를 구현했다. `ADD_ITEM`은 동일 메뉴·온도 중복 병합, 모든 명령 검증, `cart.copy()` 이후 상태 반영이 코드로 확인된다. 신규 주문 관리 Intent에는 수량 유효성·메뉴/온도 정책 검사·금액 재계산·한국어 조사 안내를 추가하고 테스트했다.

**결과:** 주문 삭제·수량 변경·온도 변경·조회·확정·취소 동작과 함께 **존재하지 않는 메뉴 삭제·0 이하 수량·미지원 온도·빈 장바구니 확정을 거부하고 기존 Cart를 유지**하는 동작을 구현했다. [PR #55](https://github.com/aihuman-7th/proj1-d/pull/55) 기록상 전체 `pytest` **70개 통과**, 팀원 **윤태현·류승민 승인** 후 병합됐다. 실제 PG 결제 구현·음성 전체 E2E·운영 사용자의 성공률을 측정한 것은 아니다.

**개인 기여·구현 연혁:** #23은 **계약 정의**, #55는 **초기 실행 코드·단위 테스트**로 구분한다. 당시 `REMOVE_ITEM/UPDATE_QUANTITY/UPDATE_OPTION`은 Batch 중 **첫 명령만 처리**했고, HOT·ICED가 함께 있는 동일 메뉴에 온도 없이 삭제/수량 변경을 요청하면 **첫 일치 항목을 선택할 수 있어** #23의 모호성 방지 목표가 완전히 구현된 것은 아니었다. 현재는 모호성 회귀 테스트와 추가 옵션·제한된 혼합 명령·10개 필드 명령 스키마 등으로 발전했으나, 그 후속 코드를 **8월 18일의 개인 #55 구현으로 소급하지 않는다**. 또한 명령 스키마에 Size 필드가 있다는 이유로 최종 서비스에서 **Size 음성 주문을 지원했다고 주장하지 않는다**.

원본: [계약 PR #23](https://github.com/aihuman-7th/proj1-d/pull/23) · [엔진 PR #55](https://github.com/aihuman-7th/proj1-d/pull/55) · [초기 병합 코드](https://github.com/aihuman-7th/proj1-d/blob/e039f6552ee82b1c61612ad00a61130e91b75310/src/domain/conversation.py) · [검증 원장 MA-02](mallo-rework-checkpoint.md).

## 3. TTS 모델 비교와 fine-tuning

### 초기 baseline 비교

20개 카페 주문 문장을 CPU 환경에서 동일하게 합성한 초기 benchmark:

- MeloTTS Baseline: **mean 1.92s, RTF 0.3939**
- Qwen3-TTS 0.6B Baseline: **mean 22.31s, RTF 3.9254**

초기 단계부터 MeloTTS가 interactive serving에 훨씬 유리했습니다.

### AI Hub 친절체 adaptation

AI Hub 71349 감성/발화스타일 데이터의 10번 화자를 이용해 약 30분 adaptation package를 만들고 MeloTTS Korean VITS2를 fine-tuning했습니다.

`G_2000` 실험:

- paired audio-text: 299개
- batch size: 4
- lr: 1e-4
- pretrained: `myshell-ai/MeloTTS-Korean`
- 2,000 steps까지 학습

Qwen3-TTS 0.6B도 비교용으로 10 epoch SFT를 진행해 `calm_friendly` voice를 구성했습니다.

### 동일 20문장 GPU 비교

| Model | Mean latency | Mean RTF | 판단 |
|---|---:|---:|---|
| MeloTTS Baseline | 0.1985s | 0.0403 | 매우 빠름, baseline tone |
| **MeloTTS Friendly** | **0.2041s** | **0.0485** | **선정** |
| Qwen3-TTS Baseline | 6.9075s | 1.4620 | real-time 부적합 |
| Qwen3-TTS Friendly | 28.1953s | 1.5732 | tone adaptation은 됐지만 latency 과다 |

따라서 `최신/큰 모델`이라는 이유로 Qwen을 선택하지 않고, **친절체 tone을 적용하면서도 0.2초 수준 생성 지연을 유지한 MeloTTS Friendly**를 선택했습니다.

### fine-tuning 전후를 어떻게 표현할 것인가

동일 GPU/20문장 controlled comparison에서:

- Melo Baseline: 198.5ms
- Melo Friendly: 204.1ms
- latency 차이: **+5.6ms, 약 +2.8%**

즉 fine-tuning의 성과를 `속도 개선`이라고 표현하지 않습니다. 속도는 거의 유지하면서 친절한 tone을 적용한 trade-off입니다. 음질/친절성은 청음 비교와 showcase로 검증했지만 정량 MOS 같은 수치는 확보하지 않았습니다.

이후 학습을 더 진행해 최종 production에는 `G_2800.pth`를 사용했습니다. 최종 정량 문서의 production benchmark는 mean **0.38s**, p95 **0.53s**, RTF **0.0894**입니다. `G_2000` 20문장 GPU 비교와 `G_2800` production benchmark는 측정 목적/환경이 다른 별도 결과이므로 직접 성능 향상률로 비교하지 않습니다.

## 4. STT fine-tuning/선정에도 직접 참여

공식 STT Lead는 별도로 있었지만 후반 통합 단계에서 학습 자산을 직접 받아 추가 실험과 production selection을 수행했습니다.

직접 작업 기록:

- Whisper Medium QLoRA Train65 continuation smoke
- clean 16 + cafe 16 = 32 sample
- cafe SNR 15 / 10 / 5dB contract 확인
- 1 step, batch 8 학습
- adapter tensor **288 / 288 변경 확인**으로 실제 update 검증
- Train65, cafe-adapt, golden500 HPF, kiosk-test, speaker-split, fast-exp artifact sweep
- candidate benchmark 후 Train65 merged BF16을 production 기본 runtime으로 전환
- baseline/QLoRA rollback path 유지

후반 candidate sweep에서 Train65 `lr=2e-5, ep=2`는 clean CER **0.135667**, cafe5 CER **0.205689**로 안정적인 후보였고, cafe-adapt는 cafe15 CER **0.150802**로 특정 노이즈 조건에서 근소하게 우세했습니다.

저장소 전체 STT worklog에는 더 이른 단계의 QLoRA 학습, speaker-level split, 도메인 TTS data 추가학습도 기록되어 있으나 이 전체 파이프라인을 개인 기여로 주장하지 않습니다.

## 5. NLU QLoRA 학습에도 직접 참여

공식 NLU Lead가 데이터셋/학습 stack을 주도했지만 통합 직전 Qwen3-1.7B QLoRA run을 직접 수행했습니다.

- 3 epochs / 450 steps
- validation best checkpoint: step 350
- Exact Match: **91.67%**
- Intent Accuracy: **96.54%**
- Slot Accuracy: **96.10%**
- Schema Valid: **98.33%**

최종 production은 더 작은 Qwen3-0.6B 계열을 사용했고 final benchmark는:

- source benchmark: **198 / 198 exact match**
- legacy batch: **1000 / 1000**
- schema parse error: **0**
- source benchmark mean latency: **0.153s**
- median latency: **0.035s**

따라서 `Qwen3-1.7B를 최종 production model로 사용했다`고 표현하지 않습니다.

## 6. dependency 충돌을 sidecar로 분리

후반 runtime에서 실제 문제가 된 부분은 모델별 Python dependency였습니다.

`275929306...` 커밋에는 다음 이유가 명시되어 있습니다.

- STT/TTS runtime: `transformers 4.27.4` 필요
- Qwen NLU: `transformers 5.15.0` 필요
- 동일 venv로 통합하지 않고 separate venv + resident HTTP sidecar 사용

최종 port 구조:

- Gateway/FastAPI: `127.0.0.1:8000`
- STT sidecar: `127.0.0.1:8001`
- NLU sidecar: `127.0.0.1:8002`
- TTS sidecar: `127.0.0.1:8003`

통합 launcher는 각 child process를 시작하고 `/health` readiness를 확인한 뒤 실제 inference warm-up을 수행하고 마지막에 gateway를 시작합니다.

모델을 매 요청마다 load하지 않고 resident process로 유지해 model loading cost도 요청 path에서 제거했습니다.

## 7. Senior-first UI/UX pivot

모델 지연을 개선하는 과정에서 음성 인식이 잘 되더라도 화면 자체가 복잡하면 고령 사용자의 주문 부담이 남는다는 문제를 확인했습니다.

반영한 항목:

- Voice Overlay
- 큰 글자/고대비 화면
- 접근성 footer 2×2
- 다시 듣기
- 천천히 듣기 / speed level
- 직원 호출
- ORDER / REVIEW / PAYMENT 단계 정리
- 결제 UI 단순화
- express/one-shot voice payment
- microphone session state 연결

따라서 프로젝트 후반에는 `모델 성능`과 `사용 흐름`을 별도 문제로 보고 동시에 수정했습니다.

## 8. Hybrid deployment

구조:

`Vercel Static Frontend → Cloudflare Tunnel → Local FastAPI Gateway → STT/NLU/TTS Sidecars`

- Web UI: Vercel
- inference: CUDA가 있는 local Windows/WSL
- API: FastAPI gateway
- public connection: Cloudflare Tunnel

대형 model artifact를 Vercel에 올리지 않고 정적 UI만 hosting하고, GPU inference는 local runtime에서 수행했습니다.

## 9. E2E latency breakdown

최종 isolated 50-turn voice pipeline:

- p50: **253.9ms**
- p95: **312.2ms**
- STT median: 약 **188.5ms**
- NLU median: 약 **15.6ms**
- NLU latency 비중: 약 **6.2%**

이 결과로 해당 test 구성에서는 NLU보다 STT가 주 latency bottleneck임을 확인했습니다.

주의: 이 run은 **real STT + real NLU + MockTTS**입니다. 따라서 `실제 TTS 생성까지 포함한 전체 E2E p95 312.2ms`라고 말하지 않습니다. 실제 TTS는 별도 production benchmark를 사용합니다.

## 10. Showcase와 사용자 평가

### STT showcase

직접 녹음한 한국어 5개 발화로 실제 inference를 검증했습니다. 일반 주문 외에 존댓말 어미/머뭇거림도 포함했습니다.

### TTS showcase

- MeloTTS baseline
- fine-tuned checkpoint
- speed Level 0 / 1 / 2

를 같은 문장으로 비교할 수 있는 web showcase를 구성했습니다.

### 사용자 평가

최종 정량 문서:

- n=13
- 만족 응답 **92.3%**

다만 실제 참가자에는 20~40대가 섞여 있어 `고령자 13명에게 92.3%`라고 표현하지 않습니다.

## 11. 프로젝트 종료와 artifact cleanup

feature freeze 이후 production에 필요한 artifact와 학습/실험 자산을 분리했습니다.

cleanup audit 당시:

- WSL workspace 약 **213.83GB**
- checkpoint/output 약 **176GB**
- dataset extract 약 **28GB**

최종 production set:

- STT runtime model
- NLU merged model
- TTS `G_2800.pth` + `config.json`
- runtime source/tests/docs
- showcase assets

raw dataset, intermediate checkpoint, cache, 불필요한 experiment output은 runtime repo와 분리해 프로젝트를 동결했습니다.

## Git과 작업 기록으로 확인되는 흐름

### 제품/아키텍처

- PRD/TRD
- Mock E2E
- common schema/error contract
- `54131e8`: deterministic Order Engine operations — **author nanocode00**

### TTS

- `2d75463`: MeloTTS/Qwen3 baseline 20문장 benchmark
- `b3bd337`: Melo friendly 2,000-step fine-tuning + 4-model GPU benchmark
- adaptive TTS/showcase
- 최종 `G_2800` production checkpoint

### 통합

- `2759293`: Qwen NLU separate cu128 sidecar / venv 분리
- STT/NLU/TTS resident services
- readiness / warm-up launcher
- Vercel + Cloudflare + local runtime

### 종료

- final quantitative results
- showcase/intro portal
- artifact/repository cleanup
- feature freeze

## 결과

- 3인 팀 Team Lead로 요구사항/contract부터 모델 통합·배포·동결까지 제품 전체 흐름 연결
- LLM의 Cart 직접 변경 대신 deterministic Order Engine을 SSOT로 분리
- Melo/Qwen 4-model latency benchmark 후 **MeloTTS Friendly 선택**
- MeloTTS fine-tuning 전후 controlled latency **198.5ms → 204.1ms**, tone adaptation을 위해 +2.8% latency trade-off 수용
- Whisper QLoRA candidate 재검증과 Qwen3-1.7B QLoRA 직접 학습
- dependency 충돌을 separate venv + sidecar architecture로 해결
- final production TTS **mean 0.38s / p95 0.53s / RTF 0.0894**
- isolated E2E **p95 312.2ms**, STT bottleneck 확인
- hybrid deployment와 repository cleanup 후 프로젝트 종료

## 이력서용 핵심 bullet

- 3인 팀 Team Lead로 PRD/TRD와 AI command contract를 정의하고, **LLM과 Cart 상태 변경을 분리한 deterministic Order Engine**을 직접 구현해 STT→NLU→주문→TTS E2E flow 구성
- AI Hub 친절체로 MeloTTS를 fine-tuning하고 4개 TTS 후보를 동일 20문장에서 비교해 **Melo Base 198.5ms / Friendly 204.1ms vs Qwen 6.91~28.20s**를 근거로 MeloTTS Friendly 선정
- Whisper Medium QLoRA candidate sweep과 Qwen3-1.7B QLoRA 학습까지 직접 수행하고, `transformers` dependency 충돌을 **Gateway 8000 + STT/NLU/TTS 8001~8003 resident sidecar**와 별도 venv로 분리
- Vercel + Cloudflare Tunnel + local FastAPI hybrid deployment 후 real STT/NLU 기반 50-turn isolated E2E **p95 312.2ms**를 측정해 STT bottleneck 식별

## 면접에서 주의할 표현

- STT/NLU 전체 학습 파이프라인을 혼자 담당했다고 주장하지 않음
- Qwen3-1.7B가 최종 production NLU라고 표현하지 않음
- TTS fine-tuning이 latency를 개선했다고 표현하지 않음. controlled benchmark는 198.5ms → 204.1ms
- Friendly tone improvement를 MOS 같은 정량 품질 향상으로 표현하지 않음
- 312.2ms에 real TTS synthesis까지 포함됐다고 표현하지 않음
- 사용자 평가 13명이 전부 고령자였다고 표현하지 않음
