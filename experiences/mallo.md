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
- 기술: Whisper, Qwen, MeloTTS, QLoRA, **Python ThreadingHTTPServer Gateway**, HTTP sidecar, Vercel, Cloudflare Tunnel

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
동일 20문장 GPU benchmark에서 **Melo Base 198.5ms / Melo Friendly 204.1ms / Qwen Base 6.91s / Qwen Friendly 28.20s**를 측정해 **MeloTTS Friendly를 최종 선택**했습니다. Fine-tuning으로 평균 지연은 약 **5.6ms(+2.8%)** 늘었지만 0.2초 수준의 interactive latency를 유지하면서 친절체 억양을 적용했습니다. 이후 `G_2800` production TTS benchmark는 **mean 0.38s / p95 0.53s / RTF 0.0894**였습니다. Vercel + Cloudflare Named Tunnel + 로컬 **Python ThreadingHTTPServer Gateway**로 실제 외부 STT/NLU/TTS 호출을 검증했습니다. 별도의 **Real Warm 50턴 최종 E2E는 p95 3.338초, 1.5초 초과 30/50턴**으로 성능 Requirement 미달이었고, Interpret network RTT가 주된 지연 구간이었습니다([PR #222](https://github.com/aihuman-7th/proj1-d/pull/222)). 이전 **p95 312.2ms**는 실 TTS·브라우저 첫 재생까지 포함하지 않은 별도 제한 조건 측정이므로 최종 성공 수치로 사용하지 않습니다.

## 핵심 수치

| 항목 | 결과 | 의미 |
|---|---:|---|
| 팀 규모 | 3인 | Team Lead / Product & Integration |
| MeloTTS Baseline, 20문장 GPU | **198.5ms mean, RTF 0.0403** | 빠르지만 어조 단조 |
| MeloTTS Friendly `G_2000`, 20문장 GPU | **204.1ms mean, RTF 0.0485** | 최종 모델 후보로 선정 |
| Qwen3-TTS Baseline, 20문장 GPU | **6.9075s mean** | interactive serving에 느림 |
| Qwen3-TTS Friendly, 20문장 GPU | **28.1953s mean** | fine-tuned tone 후보, latency 과다 |
| Production MeloTTS `G_2800` | **mean 0.38s / p95 0.53s / RTF 0.0894** | 최종 runtime 별도 benchmark |
| NLU focused contract 124건 | **Intent 100%, Slot 96.78%, Schema 100%** | 0.6B + deterministic guard 포함, target-env p95 1.610초 |
| 최종 Real E2E Warm 50턴 | **p95 3,338ms / 30건 1.5초 초과** | 실제 STT/NLU/TTS + 브라우저 Audio.onplay, Requirement 미달 |
| 사용자 평가 | **n=13, 만족 응답 92.3%** | 참가자 전원이 고령자는 아님 |

## 0. 고령 사용자 조사에서 Voice-first + Minimal GUI 우선순위로 — MA-01 (1차 검증)

**상황 → 문제 → 판단 → 조치 → 결과**

**상황:** 음성 주문 키오스크가 고령 사용자에게 실제로 어떤 도움을 줄지 정리하기 위해 초기 60대 응답자 **13명(60~64세 6명·65~69세 7명)**의 키오스크 이용 경험을 확인했습니다.

**문제:** 제품을 Voice-only 또는 무조건 느린 분절형 TTS로 설계하면, 사용자가 겪는 문제인 **옵션 선택의 어려움(8/13), 실수 후 처음부터 다시 주문(7/13), 음성 오인식 불안(6/13), 다시 설명받고 싶은 요구(10/13)**를 충분히 해결하지 못합니다. 표본은 디지털 사용 경험이 높은 60대에 치우쳐 전체 고령층으로 일반화하기 어렵습니다.

**판단:** 기존 STT→NLU→Validator→Order Engine 구조는 유지하면서, **음성으로 탐색 부담을 낮추고 현재 주문의 음성·화면 확인 및 수정·재설명 복구**에 우선순위를 둡니다. 초기 정책에서는 고령 사용자의 상태를 연령/침묵으로 추정하지 않고 **명시적 다시 듣기·천천히 듣기 요청에만 TTS를 적응**시켰습니다. **후기 구현(PR #225)에는 성공적으로 해석된 대화 4턴 내 명시적 추가 설명 요청 2회를 신호로 한 단계 감속하는 정책이 추가**됐으며, 이는 초기 MA-01 판단과 시점을 구분합니다. LLM이 Cart 금액이나 상태를 직접 변경하지 않게 Order Engine을 SSOT로 둡니다.

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

## 6. MA-09 — 다중 AI 모델 런타임 격리와 Vercel·Cloudflare 하이브리드 서비스 배포 (검증 완료)

**상황:** 말로의 실제 사용자 주문을 처리하려면 **Whisper Train65 STT·Qwen3-0.6B NLU·MeloTTS G_2800** 모델을 하나의 음성 주문 API로 연결해야 했어. 정적 키오스크 화면은 Vercel에 제공하되, GPU 모델과 가중치는 로컬 Windows/WSL2 환경에서 추론하는 구조가 필요했어.

**문제:** 서로 다른 모델의 실행 환경이 충돌했어. **MeloTTS 런타임 `transformers 4.27.4` vs Qwen NLU `5.15.0`**였고, Windows에서는 STT와 TTS를 같은 프로세스에 적재했을 때 **cuDNN native crash**가 보고됐어. 또한 신설 환경에서 MeloTTS PyPI 설치 경로·`jsonschema`·WebM/Opus용 PyAV 누락으로 서버가 시작되지 않았고, 모델 readiness 이전 호출·초기 GPU 적재·여러 브라우저가 주문 상태를 공유할 위험·터널 비밀 정보 관리도 해결해야 했어.

**판단:** 대형 GPU 모델을 Vercel에 올리지 않고, **정적 프런트엔드→HTTPS Cloudflare Named Tunnel→로컬 HTTP Gateway→모델 Sidecar**로 분리했어. 서비스는 네 프로세스이되 **`.venvs/runtime`(Gateway·STT·TTS, `transformers 4.27.4`)과 `.venvs/nlu`(Qwen3-0.6B, 5.15.0)의 두 패키지 환경**으로 운영하고, Gateway `127.0.0.1:8000`, STT `:8001`, NLU `:8002`, TTS `:8003`를 모두 loopback에 bind했어. 사용자는 Gateway만 호출하고, **`/health` readiness·`/warmup` 실추론 성공 후 노출**하며, 사용자 세션별 Cart·TTS·주문 상태는 격리하도록 설계했어.

**조치:** STT Sidecar의 최초 코드와 HTTP adapter는 팀원 윤태현의 [PR #204](https://github.com/aihuman-7th/proj1-d/pull/204)에 출처를 두었어. 해당 PR은 개별적으로 main에 병합된 상태가 아니며, 이후 통합 브랜치에 수용된 작업으로 구분했어. 브라우저 WebM/Opus를 PyAV로 디코딩하는 폴백도 윤태현 [PR #165](https://github.com/aihuman-7th/proj1-d/pull/165) 구현이야.

개인 [PR #209](https://github.com/aihuman-7th/proj1-d/pull/209)에서는 **`HTTPClientNLU` 및 Qwen3-0.6B 상주 Sidecar**를 연결하고 CUDA 12.8·PyTorch2.11·Transformers5.15 환경에서 **PEFT 없는 merged BF16 모델 load+1회 generation smoke**를 확인했어. `build_service`의 Mock/직접 Qwen/HTTP NLU 선택과 장애 시 내부 socket·URL·CUDA 정보를 사용자 응답/TTS에 노출하지 않는 복구 계약을 구현했어. 정확도/124건 focused 평가·NLU p95 미달은 **MA-07 모델 선정**의 성과로 따로 집계했어.

개인 [PR #212](https://github.com/aihuman-7th/proj1-d/pull/212)에서는 동작 확인된 MeloTTS upstream git commit을 설치 기준으로 고정하고 `jsonschema`, `av` 의존성을 추가해 **새 CUDA12.8 Runtime에서 서버 startup**을 재현했어. 개인 [PR #215](https://github.com/aihuman-7th/proj1-d/pull/215)에서는 전체 통합선과 런처·Gateway를 main에 반영했어. `src/app/launcher.py`는 Sidecar 포트 선확인→각 프로세스 `/health` 확인→**STT/NLU/TTS 실제 추론 `/warmup`→Gateway 시작→Cloudflare Tunnel 연결**, 종료·child 실패 시 프로세스 일괄 정리를 수행해. `src/app/server.py`의 `SessionManager`는 **`X-Mallo-Session-ID`별 Cart·Conversation·TTS 상태**를 나누고 같은 세션 요청을 lock으로 직렬 처리하며, 웹의 `sessionStorage` UUID·request ID를 함께 연결해. Vercel 웹은 `web/api-url.js`에서 HTTPS Gateway URL만 사용하며 내부 모델 Sidecar에 직접 접속하지 않아.

**결과:** **서로 충돌하던 모델을 네 상주 프로세스·두 가상환경의 HTTP 구성으로 연결**하고, Vercel·Cloudflare Named Tunnel 경로를 통해 당시 **실제 외부 STT/NLU/TTS 호출과 다중 세션 격리를 수동으로 검증**했어. [원본 README](https://github.com/aihuman-7th/proj1-d/blob/main/README.md)에는 배포 프런트 주소 `https://mallo-kiosk.vercel.app/`가 기록돼 있어. #215에서 frontend/launcher/gateway/session 로컬 테스트와 코드 구문·diff 검증을 보고했지만 GitHub Actions는 사용량 제한으로 실행되지 않았어. **이번 조사로 서비스가 현재 온라인인지 확인한 것은 아니며**, 최종 Real E2E 50턴은 [PR #222](https://github.com/aihuman-7th/proj1-d/pull/222)에서 **warm p95 3.338초·50턴 중 30턴이 1.5초 초과**로 성능 요구 미달이었어. 따라서 '배포에 성공'과 '실시간 지연 요구 충족'을 구분해 적었어.

**개인·팀 기여 및 기술 정확성:** 팀원 윤태현의 STT Sidecar/Windows cuDNN 문제 회피·WebM 디코더를 개인 직접 구현으로 주장하지 않았어. **Sidecar 세 개가 서로 독립 프로세스**라는 것과 **세 개가 모두 별도 Python 가상환경**이라는 것은 다르므로, 최종 런타임의 **두 가상환경**을 그대로 적었어. Gateway는 과거 요약처럼 **FastAPI가 아니라 Python `BaseHTTPRequestHandler`·`ThreadingHTTPServer` 기반 HTTP 서버**야. Docker/Kubernetes 운영, 원격 클라우드 GPU, 무중단/SLA, 현재 외부 접속 상태와 실제 서비스 E2E 1.2/1.5초 달성을 추가로 주장하지 않아. 기존 MA-06은 STT 모델/검증, MA-07은 NLU 모델/guard, MA-08은 브라우저 입력·Cart race이며, 이 MA-09의 독립 성과는 **실행 환경 격리·프로세스 supervision·세션 격리·공개 배포**야.

근거: [NLU Sidecar #209](https://github.com/aihuman-7th/proj1-d/pull/209) · [runtime 호환 #212](https://github.com/aihuman-7th/proj1-d/pull/212) · [전체 production 통합 #215](https://github.com/aihuman-7th/proj1-d/pull/215) · [팀 STT Sidecar #204](https://github.com/aihuman-7th/proj1-d/pull/204) · [Multi-service Gateway Spec](https://github.com/aihuman-7th/proj1-d/blob/main/docs/specs/multi-service-gateway.md) · [실제 런처](https://github.com/aihuman-7th/proj1-d/blob/main/src/app/launcher.py) · [Gateway 서버](https://github.com/aihuman-7th/proj1-d/blob/main/src/app/server.py) · [MA-09 검증 원장](mallo-rework-checkpoint.md).

## 7. MA-08 — Senior-first 9:16 UI·음성/터치 분리와 비동기 Cart 경합 해결 (검증 완료)

**상황:** 고령 사용자에게 음성 주문을 제공하는 말로의 초기 GUI는 카페 녹색·크림색 테마, 대형 Tap-to-talk, 주문 자막·장바구니·TTS 속도 조절로 시작했어. 이후 메뉴 선택·주문 변경·매장/포장·결제 등 사용자 흐름이 늘어나면서 **단일 화면에서 여러 단계의 9:16 세로형 키오스크**로 재설계할 필요가 생겼어.

**문제:** 기존 화면은 상태·주문 결과·다음 행동의 정보 위계가 불명확했고, 마이크 듣기/처리 중인 Overlay와 TTS 안내가 섞이면 사용자가 실제 주문 내역을 확인하기 어려웠어. 특히 **연속 터치·결제 화면 이동·마이크 권한 승인 지연·서버 재연결**이 겹치면 오래된 비동기 결과가 새 주문 상태나 장바구니를 덮어쓸 수 있고, TTS 합성 완료를 기다리는 동안 UI 반영도 늦어졌어.

**판단:** **서버의 결정론적 Cart snapshot은 주문 데이터의 단일 기준**으로 두고, 시스템 안내 음성·Response Bar·마이크는 **별도 presentation lifecycle**로 분리했어. 터치는 NLU 없이 **정확한 `OrderCommand`를 `/api/commands`에 직접 전달**, 음성은 **Mic→STT→NLU→Order Engine** 경로로 유지했어. 화면은 **WELCOME·SERVICE_MODE·ORDER·ORDER_REVIEW·OPTION_CHANGE·PAYMENT·COMPLETE**로 역할을 명확히 나누고, 마이크 권한/녹음 응답은 **epoch**, 서버 장바구니 갱신은 **order session + mutation sequence**로 서로 다른 유효성 규칙을 적용했어.

**조치:** 개인 [PR #54](https://github.com/aihuman-7th/proj1-d/pull/54)에서 초기 Voice-first GUI와 `SessionStatus`·`SessionState`를 구현하고, 고대비·대형 글씨·시스템 자막·장바구니·TTS 보조 버튼 등 초기 인터페이스를 갖췄어(**당시 저장소 전체 65 tests pass**).

이후 개인 [#159](https://github.com/aihuman-7th/proj1-d/pull/159)·[#160](https://github.com/aihuman-7th/proj1-d/pull/160)·[#161](https://github.com/aihuman-7th/proj1-d/pull/161)·[#136](https://github.com/aihuman-7th/proj1-d/pull/136)·[#162](https://github.com/aihuman-7th/proj1-d/pull/162)·[#163](https://github.com/aihuman-7th/proj1-d/pull/163)·[#164](https://github.com/aihuman-7th/proj1-d/pull/164)·[#137](https://github.com/aihuman-7th/proj1-d/pull/137)의 순서로 **9:16(1080×1920 기준) Senior-first 개편**을 진행했어. 한 번에 비대해졌던 UI 변경을 **foundation/layout·portrait/context·HTML shell·상태 머신·frontend 테스트·typography polish·UX 동작**으로 나눠 코드 리뷰 범위를 줄였어. ORDER에는 누적 주문·가로 메뉴 탭·장바구니 미니 요약, ORDER_REVIEW에는 **스크롤 가능 품목 목록·고정 총액·명시적 주문 확정**을 배치했어. 듣기/처리 중 Voice Overlay를 주문 화면 위에 겹치게 하고, 시스템 안내는 별도 Response Bar/TTS 하이라이트로 표시했어. **다시 듣기·말하기 속도·직원 호출·처음으로의 2×2 접근성 Footer와 중앙 확대 마이크**도 정리했어.

[PR #162](https://github.com/aihuman-7th/proj1-d/pull/162)/[#163](https://github.com/aihuman-7th/proj1-d/pull/163)에서는 **화면 이탈 뒤 늦게 도착한 마이크 권한 허용**으로 녹음이 재개되지 않게 `voiceLifecycleEpoch`를 대조하고 사용하지 않는 media tracks를 종료했어. `MediaRecorder` 생성 실패 때도 획득한 stream을 정리하는 회귀 테스트를 추가했어. 개인 [PR #213](https://github.com/aihuman-7th/proj1-d/pull/213)에서는 **터치 `/api/commands` vs 음성 `/api/audio/stt`→`/api/audio/interpret`**로 입력 경계를 분리하고, STT 전사 내용을 NLU 처리 중 Overlay에 먼저 표시했어. 빠른 재요청 시 이전 TTS replay를 중단하고 오래된 playback callback이 새 안내 상태를 바꾸지 못하도록 세대 토큰/취소 처리를 보강했어. PR 기록상 **frontend 7개·Python 관련 28개 로컬 테스트와 일부 실브라우저 수동 확인**이 있어.

후반 개인 [PR #225](https://github.com/aihuman-7th/proj1-d/pull/225)에서는 **`orderSessionGeneration`와 `mutationSequence`로 최신 서버 Cart만 적용**하는 `commitCartSnapshot`을 구현하고 화면·장바구니를 **TTS 합성/재생 완료와 독립적으로 즉시 갱신**했어. `LOCAL_TOUCH`에서 서버로 다시 넘기는 handoff 도중 추가 터치가 발생하면 **stale 원격 응답을 폐기**하고 최신 로컬 Cart로 후속 재연결하도록 했어. Response Bar와 TTS는 별도 message stream 상태로 관리했으며, 네트워크가 불안정해도 터치 화면의 정해진 안내를 먼저 보여주도록 구성했어. 이 변경은 **선별 frontend 6개 테스트, 관련 Python 475개 테스트** 통과로 기록됐어.

**결과:** 고령 사용자에게 **현재 주문·다음 행동·음성 상태를 구분해서 보여주는 세로형 화면**을 완성하고, **마이크·터치·서버 Cart·TTS의 서로 다른 비동기 생명주기**를 분리했어. 화면 전환 중 stale 마이크 권한 응답, 빠른 터치에 따른 장바구니 덮어쓰기, TTS 재생 때문에 주문 화면이 지연되는 문제를 코드와 회귀 테스트로 방어했어. 다만 **고령 사용자 대상 과제 성공률/인지 시간 개선·WCAG AAA 인증·모든 브라우저/장치에서의 장시간 안정성**은 이 PR들로 입증한 결과가 아니야. #225의 자동 테스트에 **실제 브라우저 재생/마이크 E2E는 포함되지 않았어.**

**개인·팀 기여 및 버전 경계:** 초기 주문 확정/결제/리셋의 서버 연동은 **윤태현 [PR #103](https://github.com/aihuman-7th/proj1-d/pull/103)**이며 실제 PG 결제가 아닌 가상 화면이야. 브라우저 WebM/Opus 음성 디코딩 폴백은 **윤태현 [PR #165](https://github.com/aihuman-7th/proj1-d/pull/165)**의 구현이고, 내 기여는 Senior-first 프런트 구조와 그 위의 클라이언트 상태/회귀 보강이야. **초기 MA-01·05의 '명시적인 천천히/다시 요청' 정책은 당시 버전의 결정**이었고, 후기 [#225](https://github.com/aihuman-7th/proj1-d/pull/225)에는 최근 4개의 **성공적으로 해석된** 사용자 turn에서 `REQUEST_CLARIFICATION`이 2회 발생하면 1단계 감속하는 정책이 추가됐어. STT 실패·침묵·연령 추정으로 속도를 변경하지 않는다는 경계는 유지했어. [#226](https://github.com/aihuman-7th/proj1-d/pull/226)의 후기 Adaptive TTS chunk-gap 값은 L0 **1.0×/0ms**, L1 **0.85×/280ms**, L2 **0.75×/480ms**이며 모델 지연과 사용자 이해도 효과는 **MA-04/05**의 별도 실험과 구분했어.

**직무 활용:** HCI·프런트엔드에는 **9:16 정보 위계·접근성·Voice Overlay·터치/음성 멀티모달 상태 설계**, AI/백엔드 시스템에는 **Cart snapshot의 SSOT·session/sequence 기반 비동기 race 회피·TTS nonblocking UI·마이크 lifecycle 복구**를 중심으로 제시할 수 있어.

원본: [초기 GUI #54](https://github.com/aihuman-7th/proj1-d/pull/54) · [Senior-first #136](https://github.com/aihuman-7th/proj1-d/pull/136)·[#137](https://github.com/aihuman-7th/proj1-d/pull/137) · [mic lifecycle #162](https://github.com/aihuman-7th/proj1-d/pull/162)·[#163](https://github.com/aihuman-7th/proj1-d/pull/163) · [입력 경계 #213](https://github.com/aihuman-7th/proj1-d/pull/213) · [경합 방지 #225](https://github.com/aihuman-7th/proj1-d/pull/225) · [MA-08 검증 원장](mallo-rework-checkpoint.md).

## 8. Hybrid deployment

**MA-09에서 아키텍처·배포 근거를 통합 정리했어.** 정적 Vercel UI→Cloudflare Named Tunnel→**Python `ThreadingHTTPServer` Gateway**→STT/NLU/TTS sidecar의 구조이며, **FastAPI가 아니고 모델 가중치는 로컬 CUDA GPU에 상주**해. 외부 접속은 프로젝트 당시 검증했고 현재 온라인 여부는 확인하지 않았어.

## 9. MA-10 — 실제 브라우저 음성 주문 E2E 지연 측정 및 RuleFirst/Qwen 병목 분석 (검증 완료)

**상황:** STT·NLU·Order Engine·TTS를 Sidecar/Gateway로 결합한 뒤, 고령 사용자의 주문 발화 종료부터 **실제 첫 음성 안내가 재생되기까지** 지연이 제품 기준을 충족하는지 판정해야 했어. 기준은 **Warm E2E Target p95 ≤1.2초 / Requirement p95 ≤1.5초**, 50턴에서 1.5초 초과 최대 2건·실패 0건이었어.

**문제:** 초기 p95 **312.2ms**는 MockTTS를 포함한 제한 조건이었고, 과거 36턴 지표에도 MockNLU/실제 브라우저 `Audio.onplay` 부재 문제가 있었어. 최종 Real 50턴 첫 진단에도 **같은 세션에 장바구니 변경이 누적되는 실험 오염**이 있어 유효한 개인 주문 결과로 판정하기 어려웠어. 전체 지연만 보면 STT/TTS·NLU fallback·브라우저 어느 곳이 원인인지 구분할 수 없었어.

**판단:** 기존 Mock/제한 조건 수치와 실제 E2E를 명확히 나누고, **Chromium VAD 입력 종료(`vad-speech-end`)→HTML5 Audio의 실제 재생 시작(`Audio.onplay`)**을 Real E2E 경계로 정의했어. 실제 Whisper Train65·RuleFirst/Qwen3-0.6B·Order Engine·MeloTTS G_2800·서버 Sidecar를 모두 통과하도록 했어. **첫 Warm-up 3턴은 통계에서 제외**, Cold Start 3회는 별도 평가하며, **매 측정 턴 고유 `X-Mallo-Session-ID`·Cart/fulfillment reset**으로 상태 오염을 차단하고, 경로/단계별 RTT를 기록하기로 했어.

**조치:** 개인 [PR #75](https://github.com/aihuman-7th/proj1-d/pull/75)에서 사용자 주문 과제·E2E/TCR/TCT/ERR/SUS/MOS 지표·세션 로그 스키마를 정의했어. [PR #122](https://github.com/aihuman-7th/proj1-d/pull/122)에서 **10개 문장×Warm 5회=50턴**, nearest-rank p95, Cold Start 3회 및 1.2/1.5초 판정 기준을 측정 계약으로 고정하고 초기 36회는 참고용으로 분리했어. 당시 MockNLU·실제 first_audio_start 부재 측정기를 `MOCK_E2E`로 재분류한 코드는 **윤태현 [#183](https://github.com/aihuman-7th/proj1-d/pull/183) 작업**으로 별도 귀속했어.

최종 개인 [PR #222](https://github.com/aihuman-7th/proj1-d/pull/222)(측정일 2026-08-28)에서 최초 동일 세션 오염 결과를 제외하고 **Turn별 새 세션·reset·비측정 옵션 변경 사전 조건**을 적용한 **Real Warm 50턴과 Cold 3회 원시 CSV**를 저장했어. 환경은 **RTX 5070 Ti 16GB, WSL2 Ubuntu 24.04.4, CUDA12.8, Python 3.11.9, Chromium 152**였어. 입력 데이터는 실제 고령자 50명 발화가 아니라 **공개 STT Showcase 고정 음원 5종×10회**, 원래 10종×5회 계약에서 바뀌었으므로 **protocol deviation**으로 명시하고 Target/Requirement나 p95 계산 방식은 변경하지 않았어.

| 최종 실측 | Mean | Median | p95 |
| --- | ---: | ---: | ---: |
| **Real Warm E2E 전체 50턴** | **1,915.6ms** | **2,166.5ms** | **3,338ms** |
| **RuleFirst 경로 20턴** | 729.1ms | 709ms | **967ms** |
| **Qwen fallback 경로 30턴** | 2,706.5ms | 2,668.5ms | **3,386ms** |
| Interpret HTTP roundtrip(모델 처리 포함) | 1,397.2ms | 1,695ms | 2,641ms |
| STT HTTP roundtrip(모델 처리 포함) | 495.1ms | 480ms | 693ms |
| Interpret 응답 후 실제 Audio.onplay | 20.5ms | 17ms | 32ms |

Cold Start 별도 3회는 **8,804 / 7,274 / 7,035ms(평균 7,704.3ms)**였어. 이 시간에는 **Sidecar 시작·GPU 가중치 적재·CUDA context/readiness·첫 요청/응답**이 모두 포함돼 있어 순수 Whisper 추론 시간이 아니야.

**결과:** 공개 CSV 50행을 재집계해 **Warm E2E p95 3.338초·1.5초 초과 30/50턴·Playback Failure 0/50**이 일치함을 확인하고, **기존 1.5초 Requirement 미달**을 투명하게 기록했어. 반면 **RuleFirst 20턴 p95 967ms는 초기 1.2초 Target을 충족**했으며, **Qwen fallback 30턴 p95 3,386ms**가 전체 tail latency의 주된 원인이었어. `Interpret network RTT`라는 로그 필드는 **순수 인터넷 왕복이 아니라 HTTP 내부 NLU 추론·처리를 포함**하기 때문에, 단순 회선 문제로 단정하지 않고 **Qwen 처리/안전한 RuleFirst 커버리지 확장**을 후속 최적화 후보로 선정했어.

**한계 및 개인 기여:** 50턴 모두 음성 파일은 재생됐지만 **단일 턴 주문/과제 완료는 20/50**이었어. 실패 분류 중 `stt-03`은 복수 메뉴 온도 누락으로 **정상 clarification을 반환했으나 한 번에 주문 완료는 못 한 경우**, `stt-04`는 '샷 추가'를 옵션 변경 단서로 처리하지 않은 **안전 가드 미지원 사례**, `stt-05`는 '아이스 카페라떼'가 '아이스티 카페라떼'로 전사된 **STT 오인식**이었어. 즉 **Order/Task 30건 미완료=모델 충돌·음성 재생 실패 30건**은 아니야. #222에서 GitHub로 확인되는 기여는 **원시 evidence와 실험 조건·구간별 실패/지연 분석·최종 판정 정합화**이며, 물리 장비에서 모든 실험 조작을 독자 수행했다고 PR author만으로 단정하지 않았어. **실제 고령 사용자 50명 대상 품질 검증도 아님**을 명시했어. 처음 312.2ms(MockTTS)와 최종 3.338초를 직접 성능 개선/악화율로 비교하지 않고, **측정 조건이 다른 실험**으로 보존했어. #222는 모델·런타임 코드 변경이 없는 **평가/문서 PR**이므로 병목을 실제 개선 완료한 것으로 주장하지 않았어.

근거: [프로토콜 #75](https://github.com/aihuman-7th/proj1-d/pull/75) · [측정 Spec #122](https://github.com/aihuman-7th/proj1-d/pull/122) · [최종 Real 결과 #222](https://github.com/aihuman-7th/proj1-d/pull/222) · [50턴 원시 CSV](https://github.com/aihuman-7th/proj1-d/blob/main/docs/evidence/e2e/i08-real-e2e-warm-50-turns-isolated.csv) · [Cold 3회 원시 CSV](https://github.com/aihuman-7th/proj1-d/blob/main/docs/evidence/e2e/i08-real-e2e-cold-start-3-runs.csv) · [계측 원장 MA-10](mallo-rework-checkpoint.md).

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
- 최종 **Real Warm 50턴 E2E p95 3.338초 / 30턴이 1.5초 초과**. Interpret network RTT 병목, Requirement 미달([PR #222](https://github.com/aihuman-7th/proj1-d/pull/222))
- hybrid deployment와 repository cleanup 후 프로젝트 종료

## 이력서용 핵심 bullet

- 3인 팀 Team Lead로 PRD/TRD와 AI command contract를 정의하고, **LLM과 Cart 상태 변경을 분리한 deterministic Order Engine**을 직접 구현해 STT→NLU→주문→TTS E2E flow 구성
- AI Hub 친절체로 MeloTTS를 fine-tuning하고 4개 TTS 후보를 동일 20문장에서 비교해 **Melo Base 198.5ms / Friendly 204.1ms vs Qwen 6.91~28.20s**를 근거로 MeloTTS Friendly 선정
- Whisper Medium QLoRA candidate sweep과 Qwen3-1.7B QLoRA 학습까지 직접 수행하고, `transformers` dependency 충돌을 **Gateway 8000 + STT/NLU/TTS 8001~8003 resident sidecar**와 별도 venv로 분리
- Vercel·Cloudflare Named Tunnel·로컬 **ThreadingHTTPServer Gateway**로 STT/NLU/TTS 전체 외부 호출을 검증하고, 실제 브라우저 첫 재생까지 포함한 **Real Warm 50턴 E2E p95 3.338초**를 측정해 목표 1.5초 **미달** 및 Interpret network RTT 병목 규명

## 면접에서 주의할 표현

- STT/NLU 전체 학습 파이프라인을 혼자 담당했다고 주장하지 않음
- Qwen3-1.7B가 최종 production NLU라고 표현하지 않음
- TTS fine-tuning이 latency를 개선했다고 표현하지 않음. controlled benchmark는 198.5ms → 204.1ms
- Friendly tone improvement를 MOS 같은 정량 품질 향상으로 표현하지 않음
- 별도 제한 조건의 312.2ms(MockTTS 포함)와 최종 Real Warm 50턴 p95 3.338초를 같은 E2E 결과로 비교하거나 목표 달성으로 주장하지 않음
- 사용자 평가 13명이 전부 고령자였다고 표현하지 않음
