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

## 1. MA-03 — 독립 AI 모델 개발 중 Mock-first 음성 주문 통합·오류 처리 구현 (검증 완료)

**상황:** 말로의 STT·NLU·Order Engine·Adaptive TTS가 나뉘어 구현되는 동안 각 실제 모델이 완성될 때까지 기다리면 음성 주문 경로의 통합 문제를 늦게 발견할 수 있었다.

**문제:** 텍스트 주문 위주로 동작하던 서비스에 오디오 입력을 전달하는 공통 진입점과 **STT 결과·NLU JSON 검증·주문 상태·TTS 음성 및 실패 결과를 한 번에 확인할 수 있는 계약**이 필요했다. 실제 Whisper·Qwen·Melo 모델을 매번 로드해야만 검증할 수 있는 구조라면 개발과 회귀 테스트가 느려지고 실패 원인을 모듈별로 분리하기 어렵다.

**판단:** **인터페이스를 먼저 통일하고 Mock으로 전체 흐름을 검증**하되, 실모델 어댑터는 독립적으로 추가하는 구조를 채택했다. `MalloService`에 STT·NLU·Order Engine·TTS를 주입하고, 오디오 요청을 `VoiceInteractionResult(STT, Order, TTS)`로 반환한다. 세션에서 TTS Level과 직전 안내를 유지하고 실패는 코드와 재입력 안내로 처리한다.

**조치:** 개인 [PR #68](https://github.com/aihuman-7th/proj1-d/pull/68)(2026-08-18)에서 `VoicePipeline.handle_audio` → STT `transcribe` → NLU `OrderCommand[]` 검증 → 결정론적 Order Engine → Adaptive TTS `synthesize` 연결, `MalloService.handle_audio`·통합 결과 모델·`MockSTT`·테스트용 `build_mock_service`를 구현했다. 통합 테스트에는 **모델이 읽을 WAV가 아니라 `b"audio_bytes_1"` 같은 가짜 바이트**, 고정 텍스트를 주는 `MockSTT`, `MockNLU`, `MockTTS`를 사용해 '아이스 아메리카노 두 잔'의 **9,400원 Cart**와 음성 응답 메타데이터를 확인했다. 또한 **천천히→다시→기본 속도(Level 0→1→2→0)**, 빈 입력, STT·TTS 강제 실패를 검증했다.

개인 [PR #69](https://github.com/aihuman-7th/proj1-d/pull/69)에서는 실제 `MeloTTS` 및 `Qwen3-TTS` 호출 인터페이스를 갖춘 별도 교체형 어댑터를 구현했다. Melo의 지연 로딩·checkpoint 교체·Level 1/2 speed 0.85, Qwen의 `generate_custom_voice`·WAV bytes 변환을 제공했고 단위 테스트는 **실모델이 아닌 mock 모델**을 사용했다.

**결과:** 실 STT/NLU/TTS 모델 준비와 관계없이 음성 입력·주문 명령 처리·안내 생성의 **인터페이스/정상·실패 경로를 반복 검증**할 수 있게 했다. 원본 #68은 저장소 **전체 297개 테스트 통과 및 팀원 승인**, #69는 별도 브랜치 **전체 296개 테스트 통과 및 승인**으로 기록돼 있다. 서로 다른 PR의 **저장소 전체 테스트를 신규 테스트 593개로 합산하지 않는다**.

**구현 경계:** #68은 마이크 녹음, VAD, 진짜 Whisper STT·LLM NLU·실제 TTS 합성이 포함되지 않은 **Mock 통합 테스트**다. Stub confidence 0.98/가상 1,200ms는 실제 정확도·지연 지표가 아니다. #69의 Melo/Qwen 단위 테스트도 GPU 성능 검증이 아니다. #68 시점 **주문 성공 후 TTS만 실패하면 실패 결과에 새 `result.cart`가 남을 수 있어** 모든 오류에 걸쳐 Cart가 롤백된다고 주장할 수 없다. 실제 모델 비교·친절체 파인튜닝·실 E2E latency와 배포/브라우저는 각각 MA-04·MA-09·MA-10에서 별도 다룬다. #67 Whisper ONNX 구현은 윤태현, #70 NLU 데이터 도구는 류승민 작성으로 귀속한다.

근거: [Mock E2E PR #68](https://github.com/aihuman-7th/proj1-d/pull/68) · [병합 당시 VoicePipeline](https://github.com/aihuman-7th/proj1-d/blob/fad7146da7cff9f745b1a9411b9c9fdb41d5946c/src/orchestrator/voice_pipeline.py) · [E2E 테스트](https://github.com/aihuman-7th/proj1-d/blob/fad7146da7cff9f745b1a9411b9c9fdb41d5946c/tests/integration/test_end_to_end_voice_pipeline.py) · [교체형 TTS 어댑터 PR #69](https://github.com/aihuman-7th/proj1-d/pull/69) · [검증 원장 MA-03](mallo-rework-checkpoint.md).

## 2. MA-02 — LLM 이해와 실제 주문 상태를 분리한 결정론적 Order Engine (검증 완료)

**상황:** 말로는 STT로 인식한 주문 발화를 NLU가 구조화하고, 메뉴 데이터와 장바구니를 실제 주문 코드가 관리하는 Voice-first 키오스크였다. 초기 `ADD_ITEM` 기반 구현만으로는 메뉴 삭제·수량/온도 변경·주문 조회·취소 등 사용자의 수정 요구를 해결할 수 없었다.

**문제:** 모듈마다 NLU `OrderCommand` 필드와 오류 형식, 복수 메뉴 Batch 처리 기준이 달랐고, LLM이 가격·수량·Cart 상태를 직접 변경하면 잘못된 수량·메뉴·온도를 실행하거나 이미 올바른 장바구니를 손상할 수 있었다.

**판단:** 정상 NLU 출력은 **단일 명령도 배열 `OrderCommand[]`**, 실패는 별도 `OrderCommandError`로 규격화한다. 초기 명령의 **6개 공통 필드·누락값 `null`** 규칙을 정하고, **가격·총액·Confirmation Policy는 Menu DB/결정론적 Order Engine이 전담**하게 한다. 복수 `ADD_ITEM`은 **전부 검사 후 복사된 Cart에 반영**하고, 오류 시 기존 Cart를 유지한다. 당시 구현에서는 **혼합 Intent 한 번에 실행을 거부**한다.

**조치:** 개인 명의 [PR #23](https://github.com/aihuman-7th/proj1-d/pull/23)(2026-08-14)에서 메뉴 하나당 명령 하나, 배열 계약, 요청 ID·Intent·target·category·수량·온도 필드, 온도 누락과 복수 메뉴/오류 처리, NLU와 Order Engine 책임을 문서로 고정하고 PRD·TRD를 맞췄다. 이 PR은 **7개 Markdown 변경의 설계 작업**으로, JSON Schema validator 실제 구현은 별도이다.

개인 [PR #55](https://github.com/aihuman-7th/proj1-d/pull/55)(2026-08-18)에서는 `src/domain/conversation.py`의 `ConversationAgent`에 `REMOVE_ITEM`, `UPDATE_QUANTITY`, `UPDATE_OPTION`(HOT↔ICED), `QUERY_CART`, `CONFIRM_ORDER`, `CANCEL` **6개 주문 관리 Intent**를 구현했다. `ADD_ITEM`은 동일 메뉴·온도 중복 병합, 모든 명령 검증, `cart.copy()` 이후 상태 반영이 코드로 확인된다. 신규 주문 관리 Intent에는 수량 유효성·메뉴/온도 정책 검사·금액 재계산·한국어 조사 안내를 추가하고 테스트했다.

**결과:** 주문 삭제·수량 변경·온도 변경·조회·확정·취소 동작과 함께 **존재하지 않는 메뉴 삭제·0 이하 수량·미지원 온도·빈 장바구니 확정을 거부하고 기존 Cart를 유지**하는 동작을 구현했다. [PR #55](https://github.com/aihuman-7th/proj1-d/pull/55) 기록상 전체 `pytest` **70개 통과**, 팀원 **윤태현·류승민 승인** 후 병합됐다. 실제 PG 결제 구현·음성 전체 E2E·운영 사용자의 성공률을 측정한 것은 아니다.

**개인 기여·구현 연혁:** #23은 **계약 정의**, #55는 **초기 실행 코드·단위 테스트**로 구분한다. 당시 `REMOVE_ITEM/UPDATE_QUANTITY/UPDATE_OPTION`은 Batch 중 **첫 명령만 처리**했고, HOT·ICED가 함께 있는 동일 메뉴에 온도 없이 삭제/수량 변경을 요청하면 **첫 일치 항목을 선택할 수 있어** #23의 모호성 방지 목표가 완전히 구현된 것은 아니었다. 현재는 모호성 회귀 테스트와 추가 옵션·제한된 혼합 명령·10개 필드 명령 스키마 등으로 발전했으나, 그 후속 코드를 **8월 18일의 개인 #55 구현으로 소급하지 않는다**. 또한 명령 스키마에 Size 필드가 있다는 이유로 최종 서비스에서 **Size 음성 주문을 지원했다고 주장하지 않는다**.

원본: [계약 PR #23](https://github.com/aihuman-7th/proj1-d/pull/23) · [엔진 PR #55](https://github.com/aihuman-7th/proj1-d/pull/55) · [초기 병합 코드](https://github.com/aihuman-7th/proj1-d/blob/e039f6552ee82b1c61612ad00a61130e91b75310/src/domain/conversation.py) · [검증 원장 MA-02](mallo-rework-checkpoint.md).

## 3. MA-04 — 동일 20문장 TTS 평가·MeloTTS 친절체 적응 및 체크포인트 선정 (검증 완료)

**상황:** 고령 사용자의 음성 주문 키오스크에서 메뉴·수량·가격 안내를 **빠르게**, 동시에 **안정적이고 친절한 어조**로 전달할 모델이 필요했다. 처음에는 실모델 비교 근거가 부족해 기본 TTS의 자연스러움과 응답 지연을 함께 검증해야 했다.

**문제:** CPU에서 MeloTTS와 Qwen3-TTS를 실행한 결과 지연 격차가 컸고, RTX GPU에서 다시 측정해도 Qwen 계열은 긴 생성 시간이 필요했다. 친절체 Fine-tuning이 수행됐다는 사실만으로 **대기 시간 단축이나 실제 고령 사용자의 이해도 상승**을 주장할 수 없고, 최초 `G_2000`와 나중의 사용성 테스트 후보 `G_2800`도 서로 다른 체크포인트였다.

**판단:** 20개 고정 카페 안내 문장을 기준으로 **CPU 사전학습 모델 비교 → Melo 친절체 적응 → RTX 5070 Ti에서 4후보 GPU 비교 → 후속 checkpoint 선택·재검증** 순서를 나누기로 했다. 단순 '최신 모델'보다 대화형 서비스의 **합성 지연·RTF와 친절체 지향 청감**을 함께 평가한다. 학습 loss는 후보 선정 근거이지 독립적인 청취 품질 점수가 아니다.

**조치:** 개인 [PR #33](https://github.com/aihuman-7th/proj1-d/pull/33)에서 AI Hub 데이터 선정·학습 분할/누수 방지·평가/라이선스 조건을 PoC로 규격화했다. [PR #63](https://github.com/aihuman-7th/proj1-d/pull/63)에서 **동일 20문장 CPU Baseline**의 MeloTTS **1.9223초·RTF 0.3939**, Qwen3-TTS **22.3092초·RTF 3.9254**를 측정하고 재현용 합성/평가 코드를 마련했다.

[PR #71](https://github.com/aihuman-7th/proj1-d/pull/71)에서는 AI Hub 71349의 **10번 화자·30분·299개 음성/텍스트 쌍**을 전처리해 MeloTTS 사전학습 가중치에서 **2,000 step warm-start Fine-tuning**한 `G_2000.pth`를 평가했다. Qwen3-TTS 0.6B 친절체 연구/10 epoch 학습도 별도로 기록하고, **RTX 5070 Ti에서 동일 20문장·4개 모델**을 비교해 모델을 선정했다.

| GPU 모델 | 평균 전체 음성 합성 지연 | 평균 RTF |
| --- | ---: | ---: |
| MeloTTS Base | **0.1985초** | 0.0403 |
| **MeloTTS Friendly `G_2000`** | **0.2041초** | **0.0485** |
| Qwen3-TTS Base 0.6B | 6.9075초 | 1.4620 |
| Qwen3-TTS Friendly 0.6B | 28.1953초 | 1.5732 |

그 뒤 [PR #152](https://github.com/aihuman-7th/proj1-d/pull/152)에서 TensorBoard 학습 기록과 보존 checkpoint를 재검토했다. 저장되지 않은 step **2850**의 loss는 더 낮았지만 보존된 후속 checkpoint 중에는 **`G_2800`이 `G_3600`보다 Generator loss가 낮아**, 실제 사용성 테스트용 후보를 `G_2800`으로 변경했다. 동일 20문장 GPU 재합성에서 **20/20 유효 44.1kHz WAV, 평균 0.1951초·RTF 0.0428**를 확인하고 통합·평가 스크립트의 기본 체크포인트를 맞췄다.

**결과:** MeloTTS Friendly `G_2000`은 RTX GPU에서 **평균 합성 204.1ms·RTF 0.0485**로 대화형 키오스크에 적합한 지연 범위를 유지하면서 원하는 친절체 방향을 추구할 수 있어 **PoC 우선 모델**로 선정했다. Baseline **198.5ms→204.1ms(+5.6ms, 약 2.8% 느려짐)**이므로 '파인튜닝으로 합성이 빨라졌다'고 쓰지 않는다. 후속 `G_2800`은 **20문장 자동 합성/지연 재검증이 된 사용성 테스트 후보**로 고정됐다. 관련 PR은 개인 `nanocode00` 명의로 작성·병합됐으며, 당시 저장소 전체 테스트 **#63 269개 / #71 306개 / #152 550개 통과**를 각각 기록했다(신규 테스트 수나 누적 합산이 아님).

**검증·귀속 경계:** 이 숫자는 모델 한 문장의 **전체 WAV 합성 지연**이지 브라우저의 **첫 음성 재생 시작 시점**이나 음성 주문 전체 E2E가 아니다. 동일 GPU의 Melo Base/Friendly 품질 개선은 평가자 블라인드 MOS/SIM으로 증명되지 않았고, 샘플레이트 44.1kHz도 사용자 체감 명료도의 증거는 아니다. [TTS 사용성 테스트 인계](https://github.com/aihuman-7th/proj1-d/blob/main/docs/tts-usability-test-handoff.md)는 당시 **DRAFT·외부 참가자 0명·개선 전후 0건**으로 기록돼 있어 테스트 완료/고령자 이해도 개선을 주장할 수 없다. 이와 별도로 `G_3600` Level 0/1 **200회 지연 비교(PR #150)**와 마지막 서비스 **평균 0.38초·p95 0.53초·RTF 0.0894**는 서로 다른 설정/목적의 측정이므로 `G_2000` 20문장 비교와 같은 실험으로 합산하지 않는다. 모델 체크포인트의 AI Hub 라이선스도 공개 사용 확정으로 해석하지 않는다. Qwen의 후속 Double Shift 원인 진단/재학습은 최초 GPU 비교 스냅샷과 별도로 기록하며, **TTS Level 0~2 및 문장/금액 발화 변환은 MA-05**에서 검증한다.

근거: [PoC #33](https://github.com/aihuman-7th/proj1-d/pull/33) · [CPU Base #63](https://github.com/aihuman-7th/proj1-d/pull/63) · [동일 문장 GPU 4후보 #71](https://github.com/aihuman-7th/proj1-d/pull/71) · [G_2800 후보 #152](https://github.com/aihuman-7th/proj1-d/pull/152) · [TTS Worklog](https://github.com/aihuman-7th/proj1-d/blob/main/docs/worklogs/tts.md) · [MA-04 검증 원장](mallo-rework-checkpoint.md).

## 3-5. MA-05 — 한국어 수사 정규화·Adaptive TTS 3단계와 지연 비용 분석 (검증 완료)

**상황:** MeloTTS Friendly를 키오스크 안내 모델 후보로 선정했지만, 사용자가 메뉴·수량·가격을 정확히 알아듣는 데에는 **음성 합성 모델 자체뿐 아니라 입력 문장과 속도·분절 정책**도 중요했다. 명시적으로 '천천히'·'다시 말해 주세요'를 요청하면 Adaptive Level 0→1→2로 조절하는 기능을 개선해야 했다.

**문제:** `5,200원`을 '오, 이백원'으로 오독하고 비표준 pause 기호 `...`/`/`가 클릭음·왜곡 발화를 만드는 사례가 발견됐다. Level 1과 Level 2는 초기 `speed=0.85`가 동일했고 20개 주문 안내 평가 문장 중 **17개는 L1/L2 변환 텍스트까지 동일**해 단계 구별이 약했다. 속도와 쉼을 늘릴 때 **실제 생성 지연의 추가 비용**도 분리 측정되지 않았다.

**판단:** **한국어 발화 정규화와 음향 모델을 분리**하고, 주문 금액은 한자어 수사(`5,200원→오천이백원`), 수량은 고유어 수사(`3잔→세 잔`), 비표준 분절 기호는 쉼표·마침표로 치환한다. Level 0은 1.0× 기본 문장, Level 1은 **0.85×·절 경계 쉼표**, Level 2는 **0.75×·메뉴/수량/금액 단문 분절**로 차별화한다. 사용자 청음/코드 검증과 지연 실측을 구분하고, **동일 checkpoint에서 Level 0/1만 교차 반복 측정**해 정책 적용 비용을 추정한다.

**조치:** 개인 [PR #97](https://github.com/aihuman-7th/proj1-d/pull/97)(8/19)로 세 Level 비교 HTML 생성 도구와 MeloTTS `HParams.spk2id` 조회 오류 방어, 음성 산출물 경로 정리를 수행했다. 당시 Level 2 속도도 0.85×였다는 초기 한계를 [PR #116](https://github.com/aihuman-7th/proj1-d/pull/116)(8/20)에서 수정했다. `korean_normalizer.py`로 금액·수량·기호를 정규화하고, `AdaptiveTextTransformer`에서 **메뉴 추가/삭제·수량/온도 변경·확인·합계·결제 안내**까지 L1 쉼/ L2 단문 분절 패턴을 확장했다. MeloTTS Level 2 속도를 **0.75×**로 차별화하고 신규 Level v2 청음 쇼케이스·회귀 테스트를 추가했다. 당시에 **저장소 전체 테스트 509개 통과**, 팀원 윤태현 승인으로 병합됐다.

개인 [PR #150](https://github.com/aihuman-7th/proj1-d/pull/150)(8/21)에는 **MeloTTS Friendly `G_3600`·RTX 5070 Ti·동일 주문 20문장·Level별 5회·교차 순서·Level당 warm-up 1회** 측정기를 작성했다. 통계 대상 **Level 0/1 각각 100회, 총 200회 합성 성공**, 실패 시 비교를 유보하고 원인을 저장하는 테스트를 구성했다. 당시 **전체 547 tests 통과**, 동료 승인을 받았다.

**결과:** 금액 읽기·기호의 실제 청음 문제에 대응하고 **L0 1.0× / L1 0.85× / L2 0.75×**의 구별되는 문장·속도 정책을 구현했다. 200회 L0/L1 합성 실측에서 **L0 평균 169.4ms → L1 210.1ms(+40.7ms)**, **p95 307.2 → 362.0ms(+54.8ms)**, 평균 RTF **0.0383 → 0.0391**을 얻어 속도 조절에도 RTF 1.0 미만임을 확인했다. 이는 **합성 구간의 대가**이지 첫 소리 재생 지연·전체 주문 E2E 개선·외부 고령 참가자 이해도 향상이 아니다.

**귀속·검증 경계:** [#116](https://github.com/aihuman-7th/proj1-d/pull/116)의 **청음 결과는 블라인드 MOS/사용자 조사 없이 수행한 개발자 확인**이며 숫자 모든 경계값의 정확성을 보장하지 않는다. 리뷰에서 `10001` 경계 문제를 제기했지만 병합 당시 변환 구현을 정적으로 보면 `만 일`로 조합되는 구조여서 **검증하지 않은 결함으로 단정하지 않는다**. `4개→네 잔`은 **docstring 오타**이고 실제 수량 테스트는 `2개→두 개`와 일치한다. [#150](https://github.com/aihuman-7th/proj1-d/pull/150)은 **`G_3600`에서 Level 0/1만** 테스트했으므로 MA-04 선정 후보 `G_2800`의 Level별 추가 비용이나 Level 2 속도 비용으로 일반화하지 않는다. #97/#116/#150의 **391+509+547은 각 PR의 전체 테스트 개수**로, 신규 테스트 수나 누적 1,447건 성과로 합산하지 않는다.

근거: [Level 쇼케이스 #97](https://github.com/aihuman-7th/proj1-d/pull/97) · [한국어 발화 정책 수정 #116](https://github.com/aihuman-7th/proj1-d/pull/116) · [교차 지연 실측 #150](https://github.com/aihuman-7th/proj1-d/pull/150) · [MA-05 검증 원장](mallo-rework-checkpoint.md).

## 4. MA-06 — Whisper Medium QLoRA 추가 실험·STT 후보 선정·서빙 전환 (검증 완료)

**상황:** 음성 주문 모델은 방언·메뉴명·수량뿐 아니라 **잡음이 있는 카페 환경에서 주문 의도를 놓치지 않아야** 했다. STT 담당 팀원 윤태현이 기존 방언 전처리, 화자 단위 분할, Whisper Medium QLoRA·도메인 적응 모델의 초기 연구를 진행했고, 프로젝트 후반 통합 단계에서 나는 **추가 학습·후보 비교·서비스 모델 결정과 서빙 연결**을 맡았다.

**문제:** 초기 STT의 CER/WER와 synthetic domain 평가는 **카페 잡음 수준별 안정성**을 직접 보여주지 못했다. 로컬에는 Golden500 HPF·Train65·cafe-adapt·kiosk-test·speaker-split·ONNX·merged BF16 등 학습 단계/모델 계보가 다른 후보가 다수 있었으며, **동일 가중치·중복 체크포인트·다른 평가 split/decoder**를 섞으면 정확한 모델 비교가 불가능했다. 학습 중 Test를 반복 조회하면 검증의 독립성도 훼손된다.

**판단:** **팀원 초기 연구와 내 후반 재검증을 구분**하고, 재학습 시 Validation만으로 하이퍼파라미터 후보를 선택한다. 모델 후보 비교는 **동일 디코딩·데이터셋, Clean/Cafe 15·10·5dB**, corpus CER/WER·latency, 동일 adapter SHA 중복 제외와 provider별 지연을 사용한다. 실제 서빙에서는 PEFT 의존성/모델 로드 복잡도를 줄이기 위해 **BF16 merged Train65**를 기본으로, 기존 도메인 merged 및 baseline을 롤백 경로로 유지한다.

**조치:** 2026년 8월 22~26일 이전 대화의 직접 실험 기록에서 RTX 5070 Ti 기반 **Golden-500 HPF Whisper QLoRA 17회 실험을 모두 실행**, Train 350 / Validation 77을 사용하고 Test 73은 열지 않았다. 검증 기록의 한 후보는 **LR 3e-5·5 epoch·warmup 50·grad accumulation 8·seed 42에서 Validation CER 0.205863·WER 0.480094**였다. 사용자 지시대로 실행 결과 문서만 보존하고 추가 학습과 Test 재실행·모델/원음 바이너리 Git 업로드는 하지 않았다. 이 17회는 **개인 실행을 확인한 과거 대화 기록**이며, 모든 체크포인트의 공개 Git 원시 로그 검증을 완료한 것은 아니다.

그다음 **32개 STT 후보를 탐색·SHA 중복 검사**한 뒤 평가 가능한 부분집합을 **Clean-kiosk 및 Cafe 15/10/5dB 조건**에서 비교했다. 실제 서버 후보로 선택한 **`whisper-medium-qlora-train65-v1-lr2e-5-ep2-merged-bf16`**의 소음 조건별 수치는 다음과 같다.

| Train65 후반 후보 평가 | CER | WER |
| --- | ---: | ---: |
| Clean-kiosk | **0.1357** | 0.4115 |
| Cafe 15dB | 0.1512 | 0.4370 |
| Cafe 10dB | 0.1679 | 0.4695 |
| Cafe 5dB | **0.2057** | 0.5137 |

이후 개인 [PR #214](https://github.com/aihuman-7th/proj1-d/pull/214)로 `WhisperMediumSTT` 기본 선택값을 **`train65`**로 전환하고, 로컬 CUDA **BF16 모델 로드·약 1457.6MiB VRAM**을 확인했다. 구형 `qlora`·`baseline` 모델은 rollback용으로 유지하고 `benchmark_stt_candidates.py`, runtime·테스트를 추가했다. 개인 [PR #218](https://github.com/aihuman-7th/proj1-d/pull/218)에서는 Windows/WSL 경로와 legacy 데이터 위치의 호환, Whisper `attention_mask` 명시, deprecated `dtype` 인자, Validation/Test CSV 입력을 보완해 **STT 단위 테스트 전체 133개 통과**로 기록했다.

**결과:** 동일 모델을 **Clean/Cafe 환경의 잡음 증가에 따라 평가하고 운영할** 근거를 확보해 `Train65 lr=2e-5·2epoch BF16 merged`를 기본 STT로 선정했다. [8/27 최종 자산 감사](https://github.com/aihuman-7th/proj1-d/blob/main/docs/repository-cleanup-audit.md)에서는 선택한 1.52GB 모델의 SHA256 provenance, 8/26 후보 leaderboard(가혹 Cafe5 부근 **CER 0.2056/WER 0.5137**), 실서빙 모델 경로와 rollback 모델을 구분했다. **후반의 독립 STT 후보 선정·검증·서비스 전환**이라는 개인 성과가 PR/자산 감사와 이전 실험 대화에 함께 남아 있다.

**개인 기여·평가 경계:** 초기 경상도 방언 preproc 500개·Whisper base/small/medium 비교·화자 누출 수정과 초기 도메인 QLoRA/BF16 merge·ONNX 변환은 **윤태현 개인 PR #27/#31/#32/#58/#67** 등과 연결되며 내 독자 개발로 주장하지 않는다. **Golden500 17회 Validation CER 0.2059**와 **Train65 Clean CER 0.1357**은 평가 모집단이 다르므로 '학습으로 0.2059→0.1357 개선'이라 쓰지 않는다. [최종 발표용 정량 표](https://github.com/aihuman-7th/proj1-d/blob/main/docs/final-quantitative-results.md)의 **CER 0.1131/WER 0.3352·메뉴 Recall 89.5%** 역시 기존 팀 도메인 모델의 **서로 다른 평가 조건**에 귀속해 Clean/Cafe 결과의 개선율로 재사용하지 않는다. 후보 32개는 **탐색 수**이지 32개 모두를 각 noise set 300개에서 실행했다는 뜻이 아니다. 원시 후보 비교 로그 일부는 로컬/당시 대화 증거이며 공개 Git의 모든 run이 복제되지는 않았다. 온라인 카페 고령자 음성 데이터의 실제 성능 개선, 전체 주문 E2E 지연 달성 및 배포 서비스의 영속적 작동은 별도 MA-09/10 검증 대상으로 남긴다.

근거: [후반 STT 서빙·벤치마크 #214](https://github.com/aihuman-7th/proj1-d/pull/214) · [STT 평가 스크립트 개선 #218](https://github.com/aihuman-7th/proj1-d/pull/218) · [모델 자산·결과 감사](https://github.com/aihuman-7th/proj1-d/blob/main/docs/repository-cleanup-audit.md) · [STT 팀 Worklog](https://github.com/aihuman-7th/proj1-d/blob/main/docs/worklogs/stt.md) · [검증 원장 MA-06](mallo-rework-checkpoint.md).

## 5. MA-07 — Qwen3 NLU 후반 모델 평가·0.6B 경량 서비스 후보 선정 및 안전 통합 (검증 완료)

**상황:** 카페 음성 주문의 STT 텍스트를 `OrderCommand[]`·Error로 구조화할 NLU가 필요했다. 팀원 류승민이 3,000문장 데이터셋(Train 2,400/Validation 300/Test 300), 1.7B QLoRA 훈련 CLI·첫 학습/모델 평가를 주도했으며, 윤태현은 관련 stacked PR을 main에 통합했다. 나는 후반 **Qwen3-1.7B 실험 재검증·0.6B 추가 학습/sweep·실서비스 비교·CUDA/sidecar 통합**을 맡았다.

**문제:** 팀의 1.7B adapter는 공식 Test 300건에서 **Exact Match 88.67%·Schema Valid 99.67%**였지만 **Error Code Accuracy 70%**로 production gate 90%에 미달했다. 개인 실사용 challenge에서는 수량('두 잔/2잔/두 개')·복수 주문·메뉴명/STT 오인식 처리 실패가 확인됐으며, 더 큰 모델은 응답 지연과 Transformers 의존성도 부담이었다. 학습 데이터 Test 점수와 실제 사용자 주문 시나리오의 의미 정확도는 분리 검증이 필요했다.

**판단:** 동일 입력·동일 후처리 경계 안에서 모델의 **Intent·Slot·Exact·Schema·Error Code와 지연**을 함께 비교하고, 서비스는 **Qwen3-0.6B `epochs4_seed7` merged BF16 + Prompt Lookup 8 + 결정론적 guard**로 결정했다. `RuleFirst`와 Schema/메뉴/Cart 검증으로 비정상 LLM 출력을 제어하고, STT/TTS의 구형 Transformers와 충돌하지 않도록 NLU를 **별도 Python/CUDA 12.8 HTTP sidecar**로 격리했다.

**조치:** 2026년 8월 25~26일 개인 실험 대화에는 **1.7B QLoRA/LoRA 모델 파일 재검증 및 고정 Test 300건 재평가**가 남아 있다. 별도의 개인 **3 epoch/450 step, Validation best step350** 기록도 있으나 학습 원시 ledger를 공개 Git에서 재검증하지 못했으므로 제출용 대표 수치로 사용하지 않는다. 당시 **38개 실전 주문 발화**의 raw 후보 비교는 **1.7B Semantic 18.4%·Exact 11.8% / 0.6B Semantic 28.9%·Exact 26.5%**였고, 이는 실험 대화의 기록으로 **공개 PR에서 raw artifact 전체를 검증한 결과가 아님**을 함께 명시한다.

Qwen3-0.6B의 개인 8/25 **15개 overnight sweep 실험 자산**을 보존하고 **`epochs4_seed7`** 후보를 선정했다([저장소 자산 감사](https://github.com/aihuman-7th/proj1-d/blob/main/docs/repository-cleanup-audit.md)). 일반 Validation에서 이 후보는 **Exact 95.33%·Schema 99.33%**로 기록됐다. 이어 개인 [PR #209](https://github.com/aihuman-7th/proj1-d/pull/209)에 `HTTPClientNLU`/Qwen0.6B 상주 sidecar·선택형 `build_service`·WSL2 실행기와 입력/오류 계약을 연결했다. **`focused-v1` 합성 124건**에서 guard 포함 **Intent 100%·Slot 96.78%·Schema 100%·Exact 63.71%·Error Code 96.67%·Invalid Menu 0%**를 측정했고, 이때 **42/124건은 guard가 출력 변경**한 것이다. 모델 원본과 생산 merged artifact는 SHA256으로 구분하고, **PEFT 없는 target-env에서 0.6B merged 모델 load+generation 1건**도 검증했다.

| 0.6B Focused 124건 (guard 포함) | 기존 환경 | RTX 5070 Ti / CUDA12.8 목표 환경 |
| --- | ---: | ---: |
| Intent / Slot / Schema | 100% / 96.78% / 100% | 100% / 96.78% / 100% |
| Exact Match | 63.71% | 63.71% |
| Warm latency median | 857.5ms | 1,169.4ms |
| Warm latency p95 | 1,182.5ms | **1,610.3ms** |

후속 개인 [PR #214](https://github.com/aihuman-7th/proj1-d/pull/214)에서는 **Qwen0.6B·RuleFirst/guard/정규화가 결합된** 38개 주문 발화 challenge에서 **38/38 Semantic·Exact·Schema** 성공을 확인했다. GPU NLU 응답시간은 **median 1,235.7ms·p95 1,630.3ms**여서 속도 목표는 미달했다. 이 38/38은 **모델 단독 성능이 아닌 시스템 전체 처리 경로의 테스트**다.

**결과:** **1.7B 모델의 높은 공식 Test 점수에도 Error Code gate 실패와 실제 주문 표현·지연 문제를 확인**하고, 적절한 0.6B 후보와 결정론적 guard를 결합한 경량 NLU 경로를 선택했다. 최종 모델 provenance를 SHA로 고정하고 CUDA12.8 환경에서 계약 점수 유지·PEFT 없이 실행·외부 sidecar 장애에 대한 안전 응답까지 검증했다. 다만 target-env **NLU p95 1.610초**, 후속 **p95 1.630초**는 latency 목표를 달성하지 못했고, 전체 서비스 Qwen fallback E2E p95 **3.386초**도 요구 1.5초를 초과하므로 '실시간성 목표 달성'으로 쓰지 않는다.

**귀속과 측정 경계:** 팀원 류승민의 초기 1.7B 학습·공식 Test([#195](https://github.com/aihuman-7th/proj1-d/pull/195), [#206](https://github.com/aihuman-7th/proj1-d/pull/206), [#211](https://github.com/aihuman-7th/proj1-d/pull/211))과 윤태현의 3,000행 데이터 스택 통합([#205](https://github.com/aihuman-7th/proj1-d/pull/205))을 내 단독 구현으로 주장하지 않는다. 개인 재현 Test Exact **89.67%**와 팀 공식 **88.67%**는 run/adapter 차이 미확인으로 주력 수치에서 분리한다. 일반 Validation 95.33%, Focused 124건 63.71%, 38문장 초기 unguarded 28.9%와 후속 guarded **38/38**은 **모집단/제어 경계가 달라 하나의 개선율로 합산 금지**. 과거 요약의 '198/198 exact·1000/1000·mean 0.153초'는 0.6B 실모델 단독 검증을 입증하는 평가 정의/원본이 없어 이번 대표 수치에서 제외한다. 실제 유저 마이크 E2E는 MA-09/10의 별도 작업이다.

근거: [0.6B 생산/sidecar PR #209](https://github.com/aihuman-7th/proj1-d/pull/209) · [서비스 guard #214](https://github.com/aihuman-7th/proj1-d/pull/214) · [팀 1.7B Test #211](https://github.com/aihuman-7th/proj1-d/pull/211) · [모델 후보 Spec](https://github.com/aihuman-7th/proj1-d/blob/main/docs/specs/nlu-qwen06b-service-candidate.md) · [목표 환경 평가 JSON](https://github.com/aihuman-7th/proj1-d/blob/main/docs/evidence/nlu/pr209-target-env/validation-summary.json) · [MA-07 검증 원장](mallo-rework-checkpoint.md).

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
