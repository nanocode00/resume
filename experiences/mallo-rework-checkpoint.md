# 말로(MALLO) — 이력서 경험 복원·귀속 검증 원장

> 작성 시작: 2026-10-22. 원본: [aihuman-7th/proj1-d](https://github.com/aihuman-7th/proj1-d), 프로젝트 원 일정 2026.06~08.
> 작업 원칙: **상황 → 문제 → 판단 → 조치 → 결과**. PR 기록과 실제 대화의 개인 판단을 모두 확보하되, 개인 구현·팀 기여·AI 에이전트 실행을 구분한다.
> **마스터 `resume.md` 변경 금지**. 현재 `experiences/mallo.md`는 초기 복원 및 귀속 오류 교정 단계로, 확정 제출 문구가 아니다.

## 1. 원본·정체성 교정

- 원본 [README](https://github.com/aihuman-7th/proj1-d/blob/main/README.md)에 표시된 3인: **김재훈(`nanocode00`, 팀장), 윤태현(`surf7978`), 류승민(`padong47`)**.
- 기존 경험기술서에 있던 '이규헌 STT Lead / 이승훈 NLU Lead'는 **팀 README와 모순**되므로 삭제했다. `docs/tasks.md`는 **A(Adaptive TTS/Voice UX), B(NLU/Order Engine), C(STT/GUI/Integration)**이라는 **영역 단위 역할**만 명시하며, A/B/C의 실명 매핑을 이 문서만으로 추측하지 않는다.
- 팀장이라는 역할만으로 모든 모델 학습/모든 프런트 구현에 자동 귀속하지 않는다. 공동/AI 코딩 도구 사용 사실과 원본 저자·실험 수행 증거를 구분한다.
- **초기 60대 13명 설문과 최종 사용성 평가 n=13/만족 응답 92.3%는 다른 평가**. 초기 설문은 60~64세 6명·65~69세 7명, 70대 이상 0명이고, 최종 사용자 평가 대상은 고령자만으로 구성된 것이 아니다. 동일 13명을 두 번 실험했다고 자동으로 계산하지 않는다.
- PRD/TRD는 최종 범위의 **canonical**, `docs/survey-pipeline-update.md`는 2026-08-19의 초기 탐색 및 확장 제안. **UNDO·Pending/Preview Cart 구상**이 곧 최종 해당 상태 머신 구현을 의미하지 않는다. 2026-08-28 기능동결 후 실제 지원 기능·제외 범위는 최신 PRD, `docs/final-ticket-reconciliation.md`에서 판정.
- 최종 배포 구조는 정적 Vercel UI → Cloudflare Tunnel → 로컬 Gateway:8000 → STT:8001, NLU:8002, TTS:8003. **최종 생산 시스템 전체를 단독 구현**했다고 바로 단정하지 않는다.

## 2. 재구성할 12개 작업군 원장

| 작업군 | 주제 | 시작 근거 | 귀속/추가 검증 | 상태 |
| --- | --- | --- | --- | --- |
| **MA-01** | 고령 사용자 조사·문제 정의·PRD/TRD·MVP 범위 | [PR #83](https://github.com/aihuman-7th/proj1-d/pull/83), `docs/prd.md`, `docs/survey-pipeline-update.md` | 설문 실시·분석·결정·최종 배포 구현 분리 | **1차 검증 완료** |
| **MA-02** | **OrderCommand 계약·결정론적 주문 관리 구현** | 개인 [#23](https://github.com/aihuman-7th/proj1-d/pull/23) 문서·개인 [#55](https://github.com/aihuman-7th/proj1-d/pull/55) 코드/테스트·병합 시점 snapshot | **A: 설계·Intent 구현 / 팀원 2명 승인**; 최초 6필드·동일 Intent batch와 이후 10필드·복합 Intent 지원 구분 | **검증 완료** |
| **MA-03** | **VoicePipeline·Mock E2E·교체형 TTS 어댑터** | 개인 [#68](https://github.com/aihuman-7th/proj1-d/pull/68) Mock 통합·결과/오류 계약; 개인 [#69](https://github.com/aihuman-7th/proj1-d/pull/69) Melo/Qwen 실제 모델 어댑터 | **A(통합 코드·Mock 시나리오, TTS 어댑터)** / 전체 297 vs 296 테스트 각 PR 기록 구분; 실제 Whisper·VAD·브라우저 E2E 미포함 | **검증 완료** |
| **MA-04** | **친절체 TTS adaptation·동일 문장 GPU 비교·checkpoint 보수적 선택** | 개인 [#33](https://github.com/aihuman-7th/proj1-d/pull/33) PoC 정의·[#63](https://github.com/aihuman-7th/proj1-d/pull/63) CPU Baseline·[#71](https://github.com/aihuman-7th/proj1-d/pull/71) G_2000/4모델 GPU·[#152](https://github.com/aihuman-7th/proj1-d/pull/152) G_2800 검증 | **A: 학습·비교·최종 후보 판단 / T: AI 도구 보조**; CPU/GPU·G_2000/G_2800·합성 지연/첫 오디오/음질·사용자 평가 분리 | **검증 완료** |
| **MA-05** | **적응형 한국어 TTS 전처리·L0/L1/L2 차별화·Level 비용 실측** | 개인 [#97](https://github.com/aihuman-7th/proj1-d/pull/97) Level 쇼케이스/어댑터 버그, [#116](https://github.com/aihuman-7th/proj1-d/pull/116) 수사/속도/분절, [#150](https://github.com/aihuman-7th/proj1-d/pull/150) 동일 G_3600 200회 Level 실측 | **A: 오류 재현·TTS 정규화/속도 구현·교차 측정** / 청음은 비블라인드, 10001 수사 edge·G_3600≠G_2800·L2 비용 미계측 | **검증 완료** |
| **MA-06** | **Whisper QLoRA Golden500 실험 17회·32후보 Clean/Cafe 비교·Train65 BF16 프로덕션 선택** | 대화의 개인 실험/결과, [STT 모델 자산 감사](https://github.com/aihuman-7th/proj1-d/blob/main/docs/repository-cleanup-audit.md), 개인 [#214](https://github.com/aihuman-7th/proj1-d/pull/214) 서빙 전환·벤치마크, 개인 [#218](https://github.com/aihuman-7th/proj1-d/pull/218) 평가 재현성 개선 | **A: 후반 sweep·선정·runtime·평가 유틸**, **T: 윤태현의 초기 speaker split·방언 QLoRA/ONNX**; Val vs Heldout, 잡음 조건·최종 발표 도메인 수치 분리 | **검증 완료(공개 근거+과거 대화)** |
| **MA-07** | **Qwen3-1.7B 개인 재학습/재평가·Qwen3-0.6B 15후보 sweep·BF16 NLU sidecar·서비스 gate** | 기존 개인 실행 대화·개인 [#209](https://github.com/aihuman-7th/proj1-d/pull/209) target-env 124건·[#214](https://github.com/aihuman-7th/proj1-d/pull/214) 실사용 guard 38건, [모델 감사](https://github.com/aihuman-7th/proj1-d/blob/main/docs/repository-cleanup-audit.md), 팀 [#195](https://github.com/aihuman-7th/proj1-d/pull/195)·[#205](https://github.com/aihuman-7th/proj1-d/pull/205)·[#211](https://github.com/aihuman-7th/proj1-d/pull/211) | **A: 후반 실험·모델/추론 계약 선정·0.6B 통합/측정 / T: 3,000개 데이터·초기 1.7B pipeline/최종 test**; 300건 official vs 124 focused vs 38 live challenge 서로 다른 모집단·guard·지연 측정 | **검증 완료(개인 대화+공개 근거)** |
| **MA-08** | Senior-first 9:16 UI·음성/터치·lifecycle/자동 Listening | [PR #54](https://github.com/aihuman-7th/proj1-d/pull/54), [#136](https://github.com/aihuman-7th/proj1-d/pull/136), [#137](https://github.com/aihuman-7th/proj1-d/pull/137), [#225](https://github.com/aihuman-7th/proj1-d/pull/225) | UI 직접 구현·유효성/상태 회귀·브라우저 E2E 구분 | 대기 |
| **MA-09** | 다중 런타임 의존성 충돌·resident sidecar·하이브리드 배포 | [PR #215](https://github.com/aihuman-7th/proj1-d/pull/215), [#216](https://github.com/aihuman-7th/proj1-d/pull/216) | transformers 충돌·서비스 간 readiness·실제 외부 URL | 대기 |
| **MA-10** | 실제 사용자 평가·음성 E2E·지연 병목 분석 | [PR #75](https://github.com/aihuman-7th/proj1-d/pull/75), [#122](https://github.com/aihuman-7th/proj1-d/pull/122), [#222](https://github.com/aihuman-7th/proj1-d/pull/222) | Real STT/NLU + MockTTS p95 312ms와 최종 real TTS 별도 | 대기 |
| **MA-11** | 실사용 QA·안전 실패·음성/화면 동작 보강 | [PR #213](https://github.com/aihuman-7th/proj1-d/pull/213), [#226](https://github.com/aihuman-7th/proj1-d/pull/226) | 실제 마이크 시연·수동 QA·회귀 테스트 구분 | 대기 |
| **MA-12** | 소개/기술 쇼케이스·공식 동결·원본 정리 | [PR #217](https://github.com/aihuman-7th/proj1-d/pull/217), [#220](https://github.com/aihuman-7th/proj1-d/pull/220), [#224](https://github.com/aihuman-7th/proj1-d/pull/224) | 8/28 기능동결·8/30 최종 repo 정리, 데이터/모델 인계 | 대기 |

## 3-1. MA-01 — 60대 초기 설문을 제품 우선순위와 기술 경계로 번역 (1차 검증: 2026-10-22)

### 상황 → 문제 → 판단 → 조치 → 결과

1. **상황:** 기존 키오스크의 화면/메뉴/옵션 조작이 어려운 고령층에게 **Voice-first + Minimal GUI** 형태의 카페 주문 경험을 제공하려 했다. 김재훈은 [원본 README](https://github.com/aihuman-7th/proj1-d/blob/main/README.md) 기준 **3인 팀장**이었다.
2. **문제:** '어르신이니까 무조건 천천히 말하고 버튼을 없애야 한다'는 가정만으로는 적절한 제품을 설계하기 어렵다. [8/19 초기 탐색 설문](https://github.com/aihuman-7th/proj1-d/blob/main/docs/survey-pipeline-update.md)에서 60~69세 **13명** 중 **옵션 선택의 어려움 8/13**, **실수로 처음부터 재주문 7/13**, **음성 오인식 우려 6/13**, **재설명 이용 의향 10/13**이 확인됐다. 표본은 60~64세 6명, 65~69세 7명이며 대부분 동반 방문·디지털 숙련 경험이 있어 전체 고령층의 무작위 표본은 아니다.
3. **판단:** 기존 Mic→STT→Rule/LLM NLU→Validator→Order Engine→TTS/GUI 아키텍처를 전면 폐기하기보다 **메뉴 탐색·수정·오류 복구의 우선순위를 높이기로** 했다. 화면은 없애지 않고 **현재 주문 내용을 크게 보여주는 확인 수단**으로 남긴다. TTS도 항상 느리게/분절해 읽는 방식 대신 **기본 자연 발화, 사용자가 명시적으로 '다시/천천히' 요청할 때만 적응**한다. 음성 인식 실패가 주문 금액/수량을 임의로 변경하지 않도록 **LLM/NLU 판단과 Order Engine의 결정론적 Cart 변경을 분리**한다.
4. **조치:** 김재훈 명의 [PR #83](https://github.com/aihuman-7th/proj1-d/pull/83)으로 **초기 설문 결과 분석 문서와 PRD 사용자 문제·우선순위 근거, 거버넌스/평가 Worklog 연결**을 작성했다. `docs/prd.md`에 **P0 주문 경로 완결, P1 적응형 음성·제한된 복구, P2 부가 기술 최적화**라는 우선순위와 예외, `docs/trd.md`에 OrderCommand/Order Engine 책임, 세션 시작 후 자동 Listening·수동 마이크 fallback, 로컬 모델·정적 Vercel Frontend의 경계를 정리했다. 후속 PRD에서는 실제 마감 범위와 합의 내용을 우선하여 Size 음성 주문/변경, 개별 Pending Confirmation 상태 머신, 실제 PG 결제를 **MVP 범위에서 제외**했다. **설문 표본 수집·인터뷰 진행을 혼자 수행했다고는 PR만으로 확인되지 않으므로 '팀 조사 결과를 분석·제품 설계에 반영'으로 기록**한다.
5. **결과:** 설문 수치가 **누구에게 어떤 기능을 우선 제공할지**에 대한 기술 결정을 뒷받침하는 근거로 문서화됐으며, 단순 Voice-only가 아닌 **음성 우선 + 화면 확인·복구** 설계와 **P0 주문 경로 보호** 원칙으로 연결됐다. [PR #83](https://github.com/aihuman-7th/proj1-d/pull/83)의 작성자 `nanocode00`, MERGED는 확인했다. **실사용 과제 수행률 개선·주문 시간 단축·실제 Undo 상태 머신 전체 구현을 이 설문만으로 주장할 수 없다.**

### 개인 기여/팀 실행 경계와 아직 확인할 사항

- **직접 확인:** 팀장 역할, 설문→기능 우선순위 번역 문서(PR #83), PRD/TRD 계약 및 종료 후 정책 정합성.
- **미확인:** 표본 13명 개인 직접 모집/인터뷰 수, 실제 70대 이상 검증, 설문 전후 제품 사용성 인과 개선·SUS, Undo/Pending State의 운영 구현. **초기 설문 n=13**과 **프로젝트 최종 사용성 평가 n=13**은 원본·실행 조건을 확인하기 전 결합하지 않는다.
- **MA-02 검증 완료:** 후속 섹션에서 #23 문서 계약, #55 초기 결정론적 엔진 구현과 후속 기능 차이를 병합 당시 코드 기준으로 확인했다.

원본: [팀 README](https://github.com/aihuman-7th/proj1-d/blob/main/README.md) · [PRD](https://github.com/aihuman-7th/proj1-d/blob/main/docs/prd.md) · [TRD](https://github.com/aihuman-7th/proj1-d/blob/main/docs/trd.md) · [Task/역할](https://github.com/aihuman-7th/proj1-d/blob/main/docs/tasks.md) · [설문](https://github.com/aihuman-7th/proj1-d/blob/main/docs/survey-pipeline-update.md) · [PR #83](https://github.com/aihuman-7th/proj1-d/pull/83).

## 3-2. MA-02 — OrderCommand 계약과 결정론적 주문 상태 보호 (2026-10-22 검증)

### 상황 → 문제 → 판단 → 조치 → 결과

1. **상황:** 말로의 NLU는 음성 주문에서 메뉴·온도·수량을 추출하지만, 실제 메뉴 DB·단가·장바구니 상태는 일반 코드가 관리하는 카페 주문 서비스다. 초기 기능에는 `ADD_ITEM`을 중심으로 Mock Pipeline이 있었고, 사용자에게는 주문 후 **수량 변경·삭제·옵션 변경·내역 확인·확정·취소**가 필요했다.
2. **문제:** NLU·PRD·TRD 문서마다 OrderCommand의 필드·Intent·오류 구조·복수 메뉴 처리 계약이 다르면 동일 발화도 모듈마다 다르게 실행될 수 있다. LLM에 금액이나 Cart의 직접 수정 권한을 주면 잘못된 수량·존재하지 않는 메뉴·불가능한 온도·모호한 삭제 대상이 **검증 없이 주문 상태를 오염**시킬 수 있다. 당시 엔진은 추가 이외의 주문 관리 Intent를 실행하지 못했다.
3. **판단:** NLU 정상 결과는 **단일 메뉴도 `OrderCommand[]`**, 실패는 `OrderCommandError`로 나누고, 단일 메뉴 명령의 `request_id/intent/target_item_id/category_id/quantity/temperature` **최초 6개 필드를 명시**해 생략/추가 필드 처리와 null 규칙을 고정한다. **가격·총액·사용자 확인 여부는 NLU가 결정하지 않고 Menu DB·Order Engine이 결정**한다. 한 발화의 복수 `ADD_ITEM`은 같은 메뉴·온도 중복을 합치고, Batch를 **전부 검증한 후 복사본 Cart에만 실행**하며, 실패하면 기존 Cart를 반환해야 한다. 최초 구현의 범위에서는 **서로 다른 Intent 혼합 실행을 거부**한다.
4. **조치:**
   - **개인 설계 PR [#23](https://github.com/aihuman-7th/proj1-d/pull/23), MERGED 2026-08-14:** 김재훈(`nanocode00`)이 `docs/specs/order-command.md`의 정상 `OrderCommand[]`/실패 `OrderCommandError`, 6개 필드, 메뉴별 Command 분리, 동일 메뉴·온도 합산, Batch 원자성, `AMBIGUOUS_TARGET`/온도 누락 시 무변경·P0 재입력/P1 Clarification 책임 경계를 문서로 확정하고 PRD/TRD/메뉴/Tasks를 정합화. **변경 파일 7개는 전부 Markdown**이므로 #23을 런타임 Schema·Validator 구현이라고 주장하지 않는다.
   - **개인 구현 PR [#55](https://github.com/aihuman-7th/proj1-d/pull/55), MERGED 2026-08-18:** `src/domain/conversation.py`의 `ConversationAgent.execute()`에서 `REMOVE_ITEM`, `UPDATE_QUANTITY`, `UPDATE_OPTION`(당시 **HOT↔ICED만**), `QUERY_CART`, `CONFIRM_ORDER`, `CANCEL` 실행 분기를 구현했다. `_execute_add_item`은 메뉴/수량/온도 전체 검사 뒤 `new_cart = cart.copy()`에 반영했으며, 기존 Cart에서 삭제/수정도 복사한 상태만 변경하도록 구현했다. 다른 Intent 혼합은 `INVALID_COMMAND`로 거부했다. 한국어 받침에 따른 조사·수량 표현을 추가해 TTS용 안내 메시지를 반환했고 `tests/unit/test_order_engine_operations.py`에서 성공·오류·복구 상태를 검증했다.
   - **실제 리뷰·테스트:** PR #55에는 **윤태현(`surf7978`)·류승민(`Padong47`) 2명 APPROVED**가 남아 있다. PR 본문 기록 `PYTHONPATH=src python3 -m pytest`는 **70 passed**이며, 이는 저장소 테스트 당시 전체 숫자이지 '신규 테스트 70개'가 아니다. **이번 이력서 조사에서 CI/pytest를 다시 돌렸다는 주장은 하지 않는다.**
5. **결과:** 핵심 주문 관리 **6종 Intent**를 LLM 비의존 결정론적 상태 전이로 구현했으며, 존재하지 않는 항목 삭제는 `MENU_NOT_FOUND`, 0 이하 수량은 `INVALID_QUANTITY`, 미지원 온도는 `INVALID_OPTION`, 빈 Cart 확정은 `INVALID_COMMAND`로 거부했다. 기존 Cart를 보존하면서 메뉴 조회와 최종 금액 안내를 제공하도록 구성했고, 원본 PR #55에 **70개 테스트 통과·팀원 2인 승인 후 병합**의 근거가 남았다. **실제 PG 결제 연동이나 모델 음성 인식·운영 사용자 전체 주문 성공률은 이 PR의 성과가 아니다.**

### 실제 병합 시점과 이후 발전의 구별 (중요)

- **2026-08-14의 #23:** 계약 **문서** 최초 6필드 정의; `schemas/order-command.schema.json` 작성 PR이 아니며 실제 Schema runtime 검증은 관련 후속 코드의 소유권을 따로 조사해야 한다.
- **2026-08-18의 #55:** `ConversationAgent`는 초기 6종 관리 Intent 구현, `UPDATE_OPTION`의 옵션은 **온도 변경만**, 혼합 Intent 배치는 일괄 거부. `_execute_remove_item`·`_execute_update_quantity`·`_execute_update_option`은 **첫 명령 하나만 소비**했으므로, #23에서 목표로 서술한 '모든 Intent의 복수 명령 원자 실행'이 처음부터 구현된 것은 아니다. **복수 `ADD_ITEM`에 대한 유효성 일괄 검사·Cart 복사 후 반영**이 코드로 확인되는 실제 원자성이다.
- **원본 #55의 모호성 경계:** 초기 삭제/수량 변경은 **온도 미지정 시 첫 번째 matching Cart line을 선택**할 수 있어, 같은 메뉴 HOT·ICED 동시 존재 시 `AMBIGUOUS_TARGET`을 모든 경로에서 차단하지 않았다. #23에는 모호성 차단 **설계 기준**이 기록됐고, 현재 `tests/unit/test_order_engine_operations.py`에는 모호성 거부 회귀 테스트가 추가돼 있다. **현재 동작을 8/18의 초기 #55 개인 구현 성과로 소급하지 않는다.**
- **2026-10-22 조회한 최신 `main`:** 명령 스키마는 **실제 required 10개 필드**(`request_id,intent,target_item_id,category_id,quantity,temperature,query_tags,detail_fields,options,fulfillment_type`)까지 확장됐다. 현재 Spec 본문에 '**9개 필드**'라고 쓰였으나 바로 아래 표는 **10개**를 열거하는 내부 문서 불일치가 있어 최신 Schema를 기준으로 확인했다. `size`·`extra_shots`·`syrup_pumps` 등의 필드는 API 스키마에 있어도 **최종 MVP에 Size 음성 주문/변경이 포함됐다는 근거는 아니다**(PRD에서 명시 제외).
- **비교 시점 고정:** 초기 #55 병합 커밋 `e039f6552ee82b1c61612ad00a61130e91b75310`의 `src/domain/conversation.py`와 243줄짜리 `tests/unit/test_order_engine_operations.py`를 확인했다. 현재 1227줄 엔진/554줄 테스트의 추가 기능을 모두 #55 개발 성과로 합산하지 않는다.
- **직무 활용:** 'LLM을 통해 주문을 이해하되 상태 변경은 코드가 검증'이라는 **AI-제어권 분리·SSOT·불변 조건 기반 테스트**를 주력 서사로 활용한다. 의도/엔진 계약과 실제 코드 간 시간차 및 초반 모호성 누락도 질문받으면 설명 가능하도록 기록한다.

원본: [PR #23](https://github.com/aihuman-7th/proj1-d/pull/23) · [PR #55](https://github.com/aihuman-7th/proj1-d/pull/55) · [#55 병합 시점 코드](https://github.com/aihuman-7th/proj1-d/blob/e039f6552ee82b1c61612ad00a61130e91b75310/src/domain/conversation.py) · [#55 병합 시점 테스트](https://github.com/aihuman-7th/proj1-d/blob/e039f6552ee82b1c61612ad00a61130e91b75310/tests/unit/test_order_engine_operations.py) · [최신 OrderCommand 스키마](https://github.com/aihuman-7th/proj1-d/blob/main/schemas/order-command.schema.json) · [최신 주문 Spec](https://github.com/aihuman-7th/proj1-d/blob/main/docs/specs/order-command.md).

## 3-3. MA-03 — Mock-first VoicePipeline과 표준 음성 입출력 계약 구현 (2026-10-22 검증 완료)

### 상황 → 문제 → 판단 → 조치 → 결과

1. **상황:** STT, 주문 NLU, 결정론적 Order Engine, Adaptive TTS를 서로 다른 모듈과 팀 작업으로 개발 중이었다. 각 모델의 실학습·배포 완료 전에도 **음성 입력부터 주문 결과·음성 출력까지 서비스 계약**을 고정하고 검증할 필요가 있었다. 당시 `MalloService`에는 주문 텍스트 처리 흐름이 있었고, 실제 음성 입력과 종합 결과 모델의 통합이 부족했다.
2. **문제:** STT 반환을 문자열로 주는 구현과 `STTResult`로 주는 구현, NLU 오류/스키마 오류, TTS 실패가 각각 서로 다른 예외로 나타날 수 있다. 모든 실제 GPU 모델이 완성될 때까지 통합을 미루면 **모듈 경계의 오류·주문 변경·Adaptive TTS 상태 전이** 문제를 개발 후반에 한 번에 발견하게 된다. 반대로 테스트 바이트와 canned transcript를 사용한 통합 테스트를 **실제 음성 인식·TTS 품질·마이크 E2E**로 보고하면 성능을 과장한다.
3. **판단:** STT/NLU/Agent/TTS를 **Protocol/생성자 주입으로 교체 가능한 인터페이스**에 묶고, 먼저 `MockSTT`·`MockNLU`·`MockTTS`를 이용해 **결정적 통합 경로와 실패 계약**을 검증한다. 요청 단위 반환은 `VoiceInteractionResult(stt_result, order_result, tts_result)`로 통일하고, Adaptive TTS의 Level·직전 안내·리셋은 같은 `VoicePipeline`의 세션 상태로 관리한다. **진짜 MeloTTS/Qwen3-TTS 어댑터 연결은 별도 PR**로 구현해 인터페이스 교체 가능성을 확보하되, mock unit test를 실제 GPU 추론 실측으로 부르지 않는다.
4. **조치:**
   - **개인 작성 [PR #68](https://github.com/aihuman-7th/proj1-d/pull/68), MERGED 2026-08-18:** `src/orchestrator/voice_pipeline.py`의 `handle_audio(audio: bytes, cart: Cart)`에서 STT `transcribe` 결과를 문자열·`STTResult`로 수용해 표준화하고, `handle_text`를 경유한 NLU schema 검증→Order Engine `execute`→AdaptiveTextTransformer→TTS `synthesize` 경로를 연결했다. `MalloService.handle_audio` wrapper·세션 `reset_session`, `VoiceInteractionResult`(STT/Order/TTS+직렬화), `MockSTT` (canned text·실패 주입·요청 bytes 기록), `build_mock_service`를 도입했다. 빈 입력 `EMPTY_INPUT`, `STTFailure`→`UNSUPPORTED_UTTERANCE` 및 `retryable=true`, `NLUFailure`/schema→검증 오류, `TTSFailure`→`TTS_FAILED` 응답 분기를 구현했다.
   - **구체적 통합 테스트:** `tests/integration/test_end_to_end_voice_pipeline.py`에서 **가짜 바이트 `b"audio_bytes_1"`**를 넣으면 `MockSTT`가 고정 텍스트 '아이스 아메리카노 두 잔 주세요'를 반환하고, `MockNLU`→`ConversationAgent`가 **2잔·9,400원** Cart를 계산하며 `MockTTS`가 **`b"mock_audio"`**를 반환함을 확인한다. 별도 테스트는 **Level 0→1(천천히)→2(다시)→0(기본 속도)**, 빈 오디오·STT 실패·TTS 실패를 다루며, `tests/unit/test_mock_stt.py`에서 canned transcript·failure injection을 검증한다. **당시 저장소 전체 `pytest tests/` 297 passed**라는 PR 기록과, **류승민(`Padong47`) 승인**이 있다.
   - **개인 작성 [PR #69](https://github.com/aihuman-7th/proj1-d/pull/69), MERGED 2026-08-18:** `TTS` 프로토콜을 따르는 `MeloTTSAdapter`(모델 지연 로딩, `config/ckpt` 인자, speed Level0=1.0·Level1/2=0.85, 파일 합성 후 WAV bytes+latency/duration 반환)와 `Qwen3TTSAdapter`(`generate_custom_voice`, 메모리 WAV bytes 반환, `TTSResult`/예외 통일)를 작성했다. 두 모델의 **unit test는 MagicMock/fake WAV를 통한 인터페이스 테스트**라 실모델 합성·품질·추론시간의 증거가 아니다. PR 본문에는 별도 브랜치 기준 **전체 296 tests 통과**, **류승민 승인**이 기록됐다. TTS 모델 20문장 GPU 비교/파인튜닝 및 실제 서비스 기본 모델 선택은 **MA-04**에서 별도 검증한다.
5. **결과:** **Mock 기반 음성 주문 통합의 정상·실패 경로를 코드와 테스트로 고정**, 인터페이스를 유지하면서 실모델 TTS Provider로 갈아 끼울 수 있는 어댑터를 마련했다. Mock 정상 케이스에서 **아이스 아메리카노 2잔·9,400원 Cart / Level0 TTS 결과**를 검증하고, 반복/느린 말하기/기본 속도 상태 전이와 음성·TTS 실패를 구분했다. **#68 당시 전체 297 tests 통과, #69 별도 PR 296 tests 통과**(합산·신규 테스트 593개가 아님), 두 PR 모두 개인 명의·팀원 승인·병합이다. 성과의 정확한 표현은 **'독립 모델 개발 중 Mock-first STT→NLU→Cart→TTS 통합을 먼저 검증'**이며 실제 모델 end-to-end latency 수치·사용자 음성 인식률은 이 PR의 성과가 아니다.

### 증거 경계 및 후속 중복 방지

- **#68 병합 직후 코드 snapshot:** `fad7146da7cff9f745b1a9411b9c9fdb41d5946c`의 `src/orchestrator/voice_pipeline.py`, `src/app/service.py`, `tests/integration/test_end_to_end_voice_pipeline.py`를 조회했다. 테스트 input은 **유효한 WAV 파일 검사 없이 임의 bytes**, 모델은 세 개 모두 Mock이고 브라우저 `getUserMedia`/VAD/실음성 인식/실 TTS GPU latency가 없다. `MockSTT.duration_ms=1200.0`·`confidence=0.98`도 **stub 데이터**, 실제 성능 수치가 아니다. 테스트 성과 297개는 **저장소 전체 테스트**이지 `test_end_to_end_voice_pipeline.py`에 새로 297개를 추가한 것이 아니다.
- **#69 snapshot:** `076fb0c8d152a2df81c5dbf470a4719efb908362`의 Melo/Qwen adapter code 및 단위 테스트를 대조했다. Melo Level1/2 speed=0.85 제어는 adapter 구현 사항이나, **Qwen3 어댑터는 level을 결과 metadata로만 반환하며 `generate_custom_voice`에 speed를 전달하지 않는다**. 실제 네 모델 20문장 음성 benchmark와 finetuning 결과는 #63/#71 및 MA-04에서만 다룬다.
- **오류 후 상태 제한:** STT/입력/잘못된 NLU 요청 실패는 `cart.copy()`/표준 오류 반환을 확인했지만, **주문 엔진 성공 뒤 TTS 합성만 실패한 경우** #68 `_synthesize_result`는 **실패 flag로 반환하면서 이미 만든 `result.cart`를 그대로 전달**하므로 주문의 전체 트랜잭션이 롤백된다고 주장하지 않는다. 완전한 서비스 실패 복구/동시성은 후속 QA MA-11의 검증 대상이다.
- **역할 귀속:** #68/#69의 GitHub author는 `nanocode00`, 리뷰는 `Padong47`. 원래 타 팀원들이 작성한 Whisper ONNX #67(`surf7978`)이나 NLU dataset #70(`Padong47`)을 개인 구현으로 합산하지 않는다. **VoicePipeline 통합과 실제 STT/NLU 학습·서빙·웹 자동 Listening 구현은 서로 다른 작업**이다.
- **면접에 쓰는 한 문장:** "AI 모델별 완성 일정에 의존하지 않도록 STT·NLU·주문 엔진·TTS에 공통 인터페이스를 적용하고, Mock E2E로 정상 주문·상태 전이·실패 경로를 먼저 검증했습니다."

원본: [PR #68](https://github.com/aihuman-7th/proj1-d/pull/68) · [#68 병합 시점 파이프라인](https://github.com/aihuman-7th/proj1-d/blob/fad7146da7cff9f745b1a9411b9c9fdb41d5946c/src/orchestrator/voice_pipeline.py) · [Mock E2E 테스트](https://github.com/aihuman-7th/proj1-d/blob/fad7146da7cff9f745b1a9411b9c9fdb41d5946c/tests/integration/test_end_to_end_voice_pipeline.py) · [PR #69](https://github.com/aihuman-7th/proj1-d/pull/69) · [Melo adapter](https://github.com/aihuman-7th/proj1-d/blob/076fb0c8d152a2df81c5dbf470a4719efb908362/src/adapters/tts/melotts_adapter.py) · [Qwen adapter](https://github.com/aihuman-7th/proj1-d/blob/076fb0c8d152a2df81c5dbf470a4719efb908362/src/adapters/tts/qwen3_adapter.py).

## 3-4. MA-04 — 친절체 TTS 파인튜닝과 실제 응답 지연을 함께 고려한 모델 후보 선정 (2026-10-22 검증)

### 상황 → 문제 → 판단 → 조치 → 결과

1. **상황:** 말로는 고령 사용자에게 메뉴명·수량·옵션·합계 금액을 또렷하고 차분하게 안내해야 하는 **대화형 음성 키오스크**다. 최초 서비스 파이프라인에는 Mock TTS 또는 Base 모델만 있었고, 친절체 음색을 구현하려는 PoC와 모델 선택 기준이 필요했다. 음성을 **자연스럽게 읽는 성격**과 **대기 시간이 짧아야 한다는 제품 요구**를 동시에 평가해야 했다.
2. **문제:** MeloTTS, Qwen3-TTS는 모델 구조와 요구 연산량이 달라 CPU/RTX GPU 측정을 섞어 비교하면 오판할 수 있다. Qwen3-TTS의 풍부한 음색을 장점으로 가정해도 지연이 길면 실제 주문 대화가 끊긴다. 친절체 Fine-tuning이 음질을 개선했다고 평가하려면 같은 문장의 사람 청취 판단도 필요하며, **속도 증가·44.1kHz 출력·학습 loss 감소만으로 '고령자 이해도 개선'이 증명되지는 않는다**. 이어서 `G_2000`을 선정했던 PoC 기준과 사용성 테스트 시점의 `G_2800` 후보가 혼재해 모델 버전 귀속이 불분명해질 수 있었다.
3. **판단:** 동일한 **카페 주문 안내 20문장**을 사용하여 **초기 CPU Base 모델 비교**와 **GPU의 4개 모델(Melo Base/Friendly, Qwen Base/Friendly) 비교**를 별도 단계로 기록한다. RTX 5070 Ti 16GB·CUDA 12.8에서 합성 mean·median·RTF를 측정하고, **대화형 서비스에서의 실시간성**과 **친절체 지향 청음**을 함께 고려하여 MeloTTS Friendly를 PoC 우선 후보로 선택한다. **청감 개선 주장에는 블라인드 평가 수치가 없음을 표시**하고, 제어 가능한 모델 체크포인트·AI Hub 저작권/외부 공유 제한을 보존한다. 이후 사용성 테스트 대상은 **보존된 checkpoint들의 loss와 동일 20문장 재합성**으로 새로 선정하되, loss 최저가 음질 최고를 의미하지 않음을 명시한다.
4. **조치:**
   - **평가 설계 [PR #33](https://github.com/aihuman-7th/proj1-d/pull/33), 8/16 개인 작성:** 한국어 MeloTTS friendly adaptation의 AI Hub 데이터 선택, 화자·세션 분할과 누수 방지, 모델 비교/청취 기준, checkpoint/라이선스·환경/AC-01~17을 PoC 단일 문서로 규격화. 이 PR은 **Spec 문서만 변경**, 모델 학습 성공으로 계산하지 않는다.
   - **초기 사전학습 모델 CPU 측정 [PR #63](https://github.com/aihuman-7th/proj1-d/pull/63), 8/18 개인 작성:** `data/tts/evaluation/sentences.json`의 20개 카페 안내 문장으로 `eval_synthesis.py` benchmark/단위 테스트를 추가하고 모델별 WAV·summary 결과를 기록. **Melo Base 평균 1.9223초·RTF 0.3939, Qwen3 0.6B Base 평균 22.3092초·RTF 3.9254**를 측정. 두 모델은 각기 44.1kHz/24kHz이며, 이 **CPU 수치와 이후 GPU 수치를 직접 모델 간 성능 향상률로 결합하지 않는다**. PR 보고 **당시 저장소 전체 269 tests 통과**, 변경된 benchmark 테스트 269개 신규 아님.
   - **친절체 적응·GPU 비교 [PR #71](https://github.com/aihuman-7th/proj1-d/pull/71), 8/18 개인 기록:** AI Hub 71349 감성/발화스타일 음성의 **화자 10**, 30분 adaptation package **299 paired 음성·텍스트**를 Melo 공식 한국어 음소화 전처리에 통과시키고, 사전학습 `G.pth/D.pth/DUR.pth`을 `warm_start`한 MeloTTS를 **2,000 steps** 학습해 `G_2000.pth`(로컬 595MB)을 생성. Qwen3-TTS 0.6B에도 299개와 speech codes의 친절체 adaptation **10 epoch** 연구 기록이 있다. RTX 5070 Ti GPU에서 **네 모델 동일 20문장 합성**을 실행하고 브라우저용 비교 플레이어를 생성했다. `G_2000`를 **PoC 우선 모델**로 선택했다. PR #71의 실제 변경 파일은 **Tasks·Worklog·`eval_synthesis.py`·unit tests(4개)**이므로 대형 모델 checkpoint 자체나 데이터셋 원본을 Git에 게시한 것으로 오해하지 않는다. 당시 저장소 전체 **306 tests 통과**를 기록했다.
   - **G_2800 사용성 후보 확정 [PR #152](https://github.com/aihuman-7th/proj1-d/pull/152), 8/24 개인 작성:** TensorBoard를 재확인한 결과 **전체 loss 최저 step 2850=26.6747은 저장되지 않았고**, 저장한 후속 checkpoint 중 **step 2800=29.0754**가 step 3600=31.9244보다 낮아 `G_2800.pth`를 테스트용 후보로 결정. G_2800 SHA256을 기록하고 동일 T-21 20문장 GPU 재합성 **20/20 WAV·44.1kHz·평균 0.1951초·RTF 0.0428**로 자동 확인했다. 통합·데모·평가 스크립트 기본 checkpoint를 G_2800으로 맞추고 사용성 테스트 준비 문서/차단 조건을 전달했다. **외부 사용자 0명·독립 블라인드 청음 미완료**이므로 T-30 자체는 완료로 표시하지 않았다. PR 작성 당시 전체 **550 tests 통과**를 기록했다.
5. **결과:** GPU 동일 20문장 T-28에서 **Melo Base 0.1985초/RTF 0.0403**, **Melo Friendly G_2000 0.2041초/RTF 0.0485**, **Qwen Base 6.9075초/RTF 1.4620**, **Qwen Friendly 28.1953초/RTF 1.5732**. Melo Friendly의 지연은 **Base보다 +0.0056초(+2.8%)**였으나, 0.2초 수준 생성 지연을 유지하며 친절체 어조 후보를 확보해 **PoC 모델 선택**에 활용했다. 이후 별도 스냅샷 `G_2800`도 20문장 재합성에 성공해 테스트 후보 기준을 고정했다. **실측된 것은 모델의 전체 WAV 합성 latency이지 첫 음성 재생 시점, 서비스 E2E, 외부 고령 사용자의 명료도 개선 비율이 아니다.** 정확한 경력 표현은 **'TTS 후보 평가를 동일 문장·GPU 기준으로 실시하고 속도와 친절체 품질의 trade-off를 비교해 실시간 서비스 모델을 선택'**이다.

### 동일 문장 T-28 GPU 측정 요약 (PR #71·G_2000 기준)

| 모델 | 스냅샷 | 평균 합성 지연 | 평균 RTF | 경계 |
| --- | --- | ---: | ---: | --- |
| MeloTTS Base | 사전학습 | **0.1985초** | 0.0403 | 속도 기준점 |
| **MeloTTS Friendly** | **G_2000** | **0.2041초** | **0.0485** | PoC 우선 후보, 속도 개선 아님 |
| Qwen3-TTS Base 0.6B | 사전학습 | 6.9075초 | 1.4620 | 긴 생성 지연 |
| Qwen3-TTS Friendly 0.6B | epoch 4 평가 snapshot | 28.1953초 | 1.5732 | GPU 비교 스냅샷·후속 10 epoch 실험 기록과 혼동 금지 |

### 귀속·모델 버전·사용자 검증 경계

- **개인 주도 근거:** `nanocode00`의 [#33](https://github.com/aihuman-7th/proj1-d/pull/33)·[#63](https://github.com/aihuman-7th/proj1-d/pull/63)·[#71](https://github.com/aihuman-7th/proj1-d/pull/71)·[#152](https://github.com/aihuman-7th/proj1-d/pull/152)에서 설계 문서·벤치마크 코드·실험 Worklog·학습 실행/판정 기록을 확인. AI 코딩·학습 스크립트 사용은 개인 의사결정과 분리하며, **원본 사용자 음성 데이터·모델 weight 공개 기여로 주장하지 않음**.
- **초기/후속 Qwen 학습:** #71 시점의 Qwen 친절체 연구 및 4후보 스냅샷과 **8/19 추가 double-shift 버그 진단/299개 최종 재학습**은 시간·원인이 다른 후속 작업이다. PR #97의 Qwen 진단 스크립트는 MA-05의 TTS 도구/진단 확장과도 교차하므로 **동일 Qwen 실험을 중복 계상하지 않는다**.
- **`G_3600` 지연 측정:** [PR #150](https://github.com/aihuman-7th/proj1-d/pull/150)의 Level 0/1 **200회 생성(169.4ms vs 210.1ms)**은 **G_3600으로 Adaptive Level 지연을 비교한 별도 MA-05 실험**이다. #71 `G_2000` 4후보 비교나 #152 `G_2800` 사용성 후보 최종 선택과 동일 checkpoint·조건이라고 간주하지 않는다.
- **`G_2800` 후보 선정의 의미:** 저장된 checkpoint 중 최소 Generator loss라 선택했을 뿐 독립 3명 블라인드 청취·사용성 전후 비교 결과는 아직 없다. [TTS 인계 문서](https://github.com/aihuman-7th/proj1-d/blob/main/docs/tts-usability-test-handoff.md)는 **DRAFT·외부 참가자 0명·개선 전후 0건**, 사용자용 재생 중단/5초 이상 대기 안내·first audio playback 계측 부족을 기록한다. **사용자 테스트가 완료됐다거나 노인 대상 친절도 개선이 통계적으로 입증됐다고 쓰지 않는다**.
- **측정 지표 구분:** CPU 초기 1.9223초/22.3092초, #71 GPU 0.1985·0.2041·6.9075·28.1953초, #152 G_2800 0.1951초는 **서로 다른 장치/학습 checkpoint/비교 목적**. 마지막 서비스 runtime mean 0.38s/p95 0.53s/RTF 0.0894는 **별도 측정**이므로 #71 대비 인과 향상률을 계산하지 않는다. `44.1kHz`는 sampling rate이고 사용자 측정 음질 등급이 아니다. 합성 mean 0.204초도 **브라우저 first_audio_start**가 아니다.
- **라이선스:** AI Hub 원본·학습 checkpoint 공개 범위는 문서에서 별도 확인 대상으로 두었으며, 외부 인계용 공개 모델 파일이 Git에 있다는 주장은 하지 않는다.

원본: [PoC 설계 #33](https://github.com/aihuman-7th/proj1-d/pull/33) · [CPU Base #63](https://github.com/aihuman-7th/proj1-d/pull/63) · [친절체 4후보 GPU #71](https://github.com/aihuman-7th/proj1-d/pull/71) · [G_2800 재검증 #152](https://github.com/aihuman-7th/proj1-d/pull/152) · [TTS Worklog](https://github.com/aihuman-7th/proj1-d/blob/main/docs/worklogs/tts.md) · [TTS 인계](https://github.com/aihuman-7th/proj1-d/blob/main/docs/tts-usability-test-handoff.md) · [동일 20문장](https://github.com/aihuman-7th/proj1-d/blob/main/data/tts/evaluation/sentences.json).

## 3-5. MA-05 — 고령 사용자 주문 안내의 한국어 오독·분절 왜곡 수정과 Adaptive TTS 지연 비용 분리 (2026-10-22 검증)

### 상황 → 문제 → 판단 → 조치 → 결과

1. **상황:** TTS 친절체 모델을 선정해도 음성 키오스크는 메뉴·수량·금액을 정확히 말해야 한다. 세션별 사용자가 **'천천히'·'다시'·'보통 속도'**를 명시적으로 요청할 때 **Adaptive Level 0(자연 발화), Level 1(느린 발화+휴지기), Level 2(더 느린 발화+짧은 문장)**으로 전환하는 서비스를 구축하고 있었다. [MA-04](#3-4-ma-04--친절체-tts-파인튜닝과-실제-응답-지연을-함께-고려한-모델-후보-선정-2026-10-22-검증)에서 모델 전체 WAV 생성 속도는 확인했으나, 적응형 모드에 따른 사용자 안내의 정확성·분절 및 **추가 합성 지연**은 독립 문제였다.
2. **문제:** 실제 MeloTTS 청음에서 **`5,200원`이 '오, 이백원'**처럼 잘못 읽혀 금액 전달 정확성에 영향을 주었고, 기존 pause 마커 **`...`/`/`**는 클릭음·왜곡 발음을 유발했다. 기존 Level 1과 2는 **둘 다 `speed=0.85`**이며 고정 주문 평가 문장 **20개 중 17개가 L1/L2 텍스트 100% 동일**해서 '더 천천히·더 명료하게' 요청을 했음에도 단계 구분이 약했다. 또한 L1의 속도를 낮추고 쉼표를 늘리면 음성 길이와 합성 시간이 증가할 수 있는데, 모델 비교 T-28의 4종 지연 표만으로 **Level 적용의 추가 비용**은 평가할 수 없었다.
3. **판단:** 모델 재학습보다 먼저 **TTS 전용 결정론적 한국어 수사 정규화 + 표준 문장부호 + Level별 문장 구조/속도 정책**을 분리한다. 가격은 한자어 수사, 물품 수량은 단위 명사에 맞는 고유어 수사로 교정하고 비표준 `...`, `/`, `|`, `~`를 정상 쉼표·마침표로 치환한다. **L0=1.00x·L1=0.85x 쉼표·L2=0.75x 독립 단문 분절**로 텍스트·속도를 동시에 차별화한다. 체감 품질 청음과 재현 가능한 지연 검증은 다른 증거로 두고, **동일 모델·체크포인트·문장/환경·warm-up·반복·교차 실행**으로 L0/L1 합성 구간만 측정한다. L2 지연은 따로 측정되지 않았으므로 L1 결과에서 추론하지 않는다.
4. **조치:**
   - **개인 [PR #97](https://github.com/aihuman-7th/proj1-d/pull/97), MERGED 2026-08-19:** `scripts/generate_adaptive_levels_player.py`로 동일 안내문 L0(1x)/L1(0.85x 쉼)/L2(**당시에는 역시 0.85x**, 단문 강조)의 비교 청음 HTML을 생성했다. `src/adapters/tts/melotts_adapter.py`에서 `HParams.spk2id`가 `dict`/객체·None일 때 발생하는 attribute 접근 실패를 방어하고, standalone 플레이어 출력·원시 WAV 관리 경로를 정리했다. Qwen 진단·실험 스크립트도 등록했으나 **Qwen 모델 연구·재학습 결과를 이 Level 기능 성과로 재계상하지 않는다**. 당시 전체 **391 passed / 5 skipped**, 윤태현(`surf7978`) 승인·병합.
   - **개인 [PR #116](https://github.com/aihuman-7th/proj1-d/pull/116), MERGED 2026-08-20:** `src/domain/korean_normalizer.py`에 **`5,200원→오천이백원`, `10,400원→만 사백원`, `15,400원→만 오천사백원`, `3잔→세 잔`, `2개→두 개`**, 비표준 발화/분절 기호 정제를 구현했다. `AdaptiveTextTransformer`는 공통 정규화 후 **L0 기존 자연 문장, L1 절 경계 쉼표 삽입, L2 메뉴·수량·금액·삭제·변경·확정 등을 별도 단문으로 분리**하도록 주문 템플릿 범위를 넓혔다. `MeloTTSAdapter`를 **L0 1.0 / L1 0.85 / L2 0.75**로 조정하고 v2 브라우저 Level 쇼케이스를 생성했다. 코드·단위·Mock 통합 테스트 12개 파일을 함께 수정하고 **20개 평가 안내문의 L1/L2 출력 분리**를 확인했다. 전체 **509/509 passed**는 PR 본문과 동료 실제 재실행 리뷰에 일치. 직접 청음으로 금액 발음·click noise 완화를 확인했지만 **블라인드 MOS나 사용성 개선율 수치는 없음**. 윤태현 승인. 리뷰어는 **`10001` 변환 경계 우려**와 `4개→네 잔`이라는 **docstring 오타**를 비차단 의견으로 남겼다. 다만 **병합 당시 실제 소스를 정적으로 확인하면 `10001`의 하위 1·상위 만 chunk를 조합해 `만 일`이 되는 구조**여서, 리뷰 의견만으로 `10001` 결함이 실제 재현됐다고 단정하지 않는다. 별도의 런타임 실행 검증은 하지 않았고 범용 숫자 변환의 모든 경계값 정확도를 주장하지 않는다. PR **변경량 998줄**은 당시 400줄 권장 가이드 초과로, 승인자는 이번 1회 한정 허용 후 분리 요구를 남겼다.
   - **개인 [PR #150](https://github.com/aihuman-7th/proj1-d/pull/150), MERGED 2026-08-21:** `docs/specs/adaptive-tts-latency.md`에 Level 추가 비용을 독립 AC로 정의하고 `src/adapters/tts/level_latency.py` **Level 0/1 교차 벤치마크**를 작성했다. **MeloTTS Friendly `G_3600.pth`·speaker 10·RTX 5070 Ti 16GB·Python 3.11.9·CUDA 12.8**에서 **T-21 20문장 × Level당 5회 = 200 유효 합성**을 실행, 두 Level당 1회 별도 warm-up·문장/회차 교차 순서·nearest-rank p95·합성 실패 결과 보존을 적용했다. `tests/unit/test_tts_level_latency.py` 4건은 200회·교차 순서·평균/비율/p95·오류 결과 보존을 검증한다. **200/200 성공**, 저장소 전체 **547 passed**, 윤태현 승인·병합. `G_3600`은 MA-04에서 사용성 테스트 대상이 된 `G_2800`과 **다른 체크포인트**다.
5. **결과:** [#116](https://github.com/aihuman-7th/proj1-d/pull/116)에서 메뉴·수량·가격 발화를 결정적 한국어 텍스트로 바꾸고 **L1 쉼/0.85x vs L2 단문/0.75x**를 구현해 기존 17/20 출력 동일 문제를 수정했다. 코드와 청음 사례로 확인된 기능 개선이며 **고령 사용자 이해도 상승을 외부 참가자 평가로 입증한 것은 아니다**. [#150](https://github.com/aihuman-7th/proj1-d/pull/150)에서 **L0 평균 169.4ms/p95 307.2ms → L1 평균 210.1ms/p95 362.0ms**, **추가 합성 지연 평균 +40.7ms / p95 +54.8ms**를 측정했다. 평균 RTF는 **0.0383→0.0391(+0.0008, 약 2.1%)**로 둘 다 1 미만을 유지했다. 이 **200회 측정은 Level 0/1 합성 구간**이며 Audio Browser 재생 지연·실서비스 첫 소리 도착·L2 지연·장기 사용성은 측정하지 않았다.

### 기술적으로 중요한 측정 경계

| 지표 (G_3600 / RTX 5070 Ti, 20문장 각 5회, 200회) | Level 0 | Level 1 | 차이 |
| --- | ---: | ---: | ---: |
| 평균 WAV 합성 지연 | 169.4ms | 210.1ms | +40.7ms |
| p95 WAV 합성 지연 | 307.2ms | 362.0ms | +54.8ms |
| 평균 RTF | 0.0383 | 0.0391 | +0.0008 |

- **MA-04와 고유 성과 분리:** #71 `G_2000` 4개 모델 20문장 latency 비교는 **모델 선택**의 근거, #152 `G_2800` 20문장 재합성은 **사용성 테스트 후보 재검증**, #150 `G_3600` 200회 L0/L1 교차 실험은 **Adaptive 정책 적용 비용**. 동일 모델·체크포인트의 개선 전후로 합쳐 '속도가 204→169ms 개선' 같은 지표를 만들지 않는다.
- **`G_3600`은 선정 모델이 아님:** 선택된 사용성 checkpoint는 loss 근거로 `G_2800`이며, #150에서는 **조건을 고정한 별도의 이전 샘플**을 사용했다. L2 `0.75x` 코드는 검증됐어도 L2 GPU 200회 측정은 하지 않았다.
- **금액·수사 정규화 한계:** 5,200/10,400/15,400원 등 실제 메뉴 도메인 규칙과 범용 10001·마이너스 금액/큰 수·복합 카운터의 정확성은 다른 문제. 리뷰어가 `sino_number_to_korean(10001)`에 관한 우려를 제기했지만 **병합 당시 구현을 보면 `만 일` 생성 구조**여서 이를 재현된 결함으로 옮기지 않는다. `4개→네 잔`은 docstring의 오타(실제 수량 변환 테스트는 `2개→두 개`)다. 그럼에도 전체 범용 수사 엔진의 100% 정확도는 주장하지 않는다. **정상 카페 가격과 주문 수량에 대한 테스트/청음 검증**만 이 항목의 성과다.
- **실제 품질 근거:** 개인/팀 청음은 **블라인드 외부 MOS 점수가 아니다**. [#116](https://github.com/aihuman-7th/proj1-d/pull/116)의 509 테스트는 저장소 **전체 테스트**이고 #97의 391, #150의 547과 더해 신규 1,447개 테스트로 세지 않는다.
- **엔진 간 적용 범위:** `AdaptiveTextTransformer.transform` 단계에서 공통 정규화를 호출하므로 **Melo/Qwen 등 TTS 어댑터 앞에서 한국어 수사가 변환**될 수 있다. Melo 어댑터의 추가 normalize 호출은 텍스트 정규화가 멱등이 되도록 설계돼 있으나 실모델 Qwen·모든 금액 발화에 대해 100% 청음 테스트를 수행한 것은 아니다.
- **사용자 제어 결론:** 명시적 `REQUEST_REPEAT`, `REQUEST_SLOWER`, `REQUEST_DEFAULT_SPEED`는 같은 세션에서 L0→L1→L2→L0로 전환하도록 별도 MA-03 Mock 검증과 연결. 무조건 고령자에게 느린 TTS나 연령 추정 기반 단계 상승을 적용했다고 쓰지 않는다.

원본: [Level 비교/쇼케이스 #97](https://github.com/aihuman-7th/proj1-d/pull/97) · [한국어 수사/Level 3단계 수정 #116](https://github.com/aihuman-7th/proj1-d/pull/116) · [동일 조건 L0/L1 latency #150](https://github.com/aihuman-7th/proj1-d/pull/150) · [#116 병합 당시 normalizer](https://github.com/aihuman-7th/proj1-d/blob/cecb52aa2fe3805c58816d957f88da0668173ae4/src/domain/korean_normalizer.py) · [L0/L1 측정 도구](https://github.com/aihuman-7th/proj1-d/blob/feebdd817c4a2c2c45870b13ea348448bb60673a/src/adapters/tts/level_latency.py) · [TTS Worklog](https://github.com/aihuman-7th/proj1-d/blob/main/docs/worklogs/tts.md).

---

## 3-6. MA-06 — Whisper Medium QLoRA 후반 재검증·잡음 강건성 비교와 Train65 서빙 후보 선정 (2026-10-22 복원)

### 상황 → 문제 → 판단 → 조치 → 결과

1. **상황:** 말로 STT는 한국어 방언·고령 사용자 발화·메뉴/수량/옵션을 인식해야 했지만, 실제 카페 소음에서도 안정적으로 주문을 추출할 수 있는지 검증되지 않았다. STT 파트 팀원 **윤태현(`surf7978`)**이 Whisper small/medium, 경상도 방언 500개 전처리·화자 단위 Train/Validation/Test split, Medium QLoRA·도메인 TTS 음성 학습 및 BF16 merge를 수행했다([#27](https://github.com/aihuman-7th/proj1-d/pull/27), [#31](https://github.com/aihuman-7th/proj1-d/pull/31), [#32](https://github.com/aihuman-7th/proj1-d/pull/32), [#58](https://github.com/aihuman-7th/proj1-d/pull/58)). 팀장 김재훈은 **후반 통합·추가 모델 실험·서빙 후보 검증**을 별개로 맡았다.
2. **문제:** 기존 방언 데이터 위주 평가나 합성 도메인 506개 테스트만으로는 실제 카페 소음 15/10/5dB에서의 **성능 저하 곡선**을 알기 어렵다. 초기 STT worklog에는 파일 단위 split에서 동일 화자 누출(train∩validation 18명, train∩test 28명)이 발견돼 화자 단위로 재분할한 이력이 있어, **이전 split/시험의 CER와 새로운 후보의 숫자를 그대로 비교할 위험**이 있다. 로컬에 Golden500·Train65·cafe-adapt·speaker-split·kiosk-test·fast-exp·ONNX/merged 모델이 공존해 **동일 어댑터 SHA 중복·오래된 checkpoint 선택·미사용 smoke 실행·GPU/CPU 실행 공급자 차이**까지 후보 선택을 왜곡할 수 있었다.
3. **판단:** 이전 팀원 학습 성과는 **출발점/비교군**으로 보존하고, 별도 학습 검증·후보 전수 평가를 실행하기로 했다. **Train/Val은 학습과 선택에 사용, Test는 봉인**하며, Golden500 HPF의 LR/epoch/warmup sweep은 별도 Validation 비교로 기록한다. 후보 전체 평가는 **SHA 중복 검사·top-level 최종 adapter 우선·미완료/백업 제외·동일 decoding·corpus-level CER/WER와 소음별 지연/키워드 성능**을 기준으로 만들고, 같은 모델과 Clean·Cafe 15/10/5dB 음원의 **별도 평가 조건**을 대조해 생산 모델 후보를 고른다. 배포에서는 **BF16 merged**를 직접 로드해 PEFT 의존성을 제거하고, 기존 도메인 QLoRA·baseline은 롤백 경로로 남긴다.
4. **조치:**
   - **개인 추가 학습/검증 (2026-08-22~26 당시 대화 직접 수행 기록, 공개 PR 아닌 로컬 실험):** RTX 5070 Ti에서 Golden-500 HPF **17개 QLoRA 실험**을 수행해 **17/17 완료** 보고. 이 실험은 Train 350 / Validation 77 / **Test 73 사용 금지**로 진행했으며, 가장 유리한 검증 기록 중 **LR 3e-5·5 epochs·warmup 50·gradient accumulation 8·seed 42**의 Validation **CER 0.205863·WER 0.480094**, checkpoint-176이 있다. 방언 관련 초기 350/77/73 speaker-level split 기록과 이 실험을 **동일 인물의 '초기 STT 전 기간 학습'으로 합산하지 않는다**. 사용자 지시에 따라 17개 실험의 기록만 남기고 추가 학습·Test 73 재실행·원음/체크포인트의 공개 커밋을 하지 않았다. **이 17회 실험의 원시 ledger/체크포인트는 공개 Git PR에서 재조회되지 않으므로 '대화/로컬 실행 보고' 등급으로 표시**한다.
   - **개인 후반 후보 비교 (2026-08-26 대화 + 공개 스펙/감사):** Train65, cafe-adapt, Golden500 HPF, kiosk-test, speaker-split, fast-exp 및 이전 팀 도메인 merged, ONNX 비교군 등 **32개 후보를 탐색/중복 진단**했다. Clean-kiosk **300개**, Cafe SNR **15·10·5dB 조건별 300개**(비교 가능한 후보별 스코프 구분)에서 0 오류로 비교한 과거 보고가 있다. `scripts/benchmark_stt_candidates.py`와 [후보 벤치마크 Spec](https://github.com/aihuman-7th/proj1-d/blob/main/docs/specs/stt-candidate-benchmark.md)은 **동일 SHA256 adapter 1회, `smoke`·`checkpoint_backups` 제외, adapter top-level 우선·checkpoint fallback, 모델별 unload, provider별 latency rank 분리, 생성 조건 기록**을 정의·구현했다. 후반 실험은 모델을 직접 학습·평가하는 실행이었지만 **32개 전체를 각 노이즈 300건 모두 실행했다고 서술하지 않는다**(탐색/중복 검사를 거친 부분집합 평가).
   - **선택한 모델:** `whisper-medium-qlora-train65-v1-lr2e-5-ep2-merged-bf16`(Train65, LR 2e-5, **2 epochs**)을 최종 후보로 선정. Clean/Cafe 조건 평가에서 **Clean CER 0.135667/WER 0.411471**, **Cafe15 CER 0.151167/WER 0.437032**, **Cafe10 CER 0.167943/WER 0.469451**, **Cafe5 CER 0.205689/WER 0.513716**으로 소음 증가에 따른 열화를 수치화했다. 이전 대화 평가 기록에선 **Cafe 평균 p95 452.8ms**, 다른 아카이브의 최종 leaderboard는 **Cafe5 근접 CER 0.2056/WER 0.5137 및 median 238ms/p95 386ms**를 인용하므로 **동일 구간/통계가 아니면 수치를 섞지 않는다**. 비교군 `train65-cafe-adapt-v1`은 일부 조건(15dB CER 0.150802)에서 근소하게 좋았으나 전체 조건과 운영 복잡성을 함께 고려해 Train65을 주 후보, cafe-adapt를 예비 후보, 팀 인계 `domain/merged-bf16`을 롤백 후보로 남겼다. [8/27 자산 실사](https://github.com/aihuman-7th/proj1-d/blob/main/docs/repository-cleanup-audit.md)는 **1.52GB BF16 safetensors/SHA256 `c0e432a4...`**, 로컬 benchmark `artifacts/stt-benchmark/20260826-131107/leaderboard.csv` 및 프로덕션 모델로 사용 기록을 확인한다. 공개 저장소에는 대형 모델 가중치가 Git 추적되지 않는다.
   - **개인 구현 [PR #214](https://github.com/aihuman-7th/proj1-d/pull/214), 2026-08-26:** `WhisperMediumSTT` 기본 variant를 **`train65` merged BF16**로 변경하고 기존 `qlora`(팀 도메인 merged)·`baseline` 롤백 경로를 남겼다. 실제 **CUDA BF16 로드**와 **VRAM 약 1457.6MiB**를 PR에서 보고했고, 메뉴/STT 인식 보정·실서비스 STT 후보 benchmark 스크립트/단위 테스트를 함께 추가했다. 이 PR의 NLU challenge **38/38 semantic pass**와 NLU p95 1630.3ms는 **NLU 관련 측정**이며 이 STT 검증의 인식 정확도·속도로 재활용하지 않는다. PR 메타데이터/변경 코드로 직접 개인 기여가 확인된다.
   - **개인 평가 재현성 수정 [PR #218](https://github.com/aihuman-7th/proj1-d/pull/218), 2026-08-28:** 기존 STT 평가용 `compare_model_sizes.py`·`baseline_qlora_compare.py`에서 **Windows 절대경로↔Linux/WSL·레거시 경로 매핑**, Whisper processor의 **attention_mask 명시**, deprecated `torch_dtype→dtype`, dataset CSV 인자, checkpoint/merged fallback 경로를 보완하고 단위 테스트를 추가했다. 당시 **STT 범위 133 tests 통과**. 이 PR은 **모델 훈련·서빙 모델 변경이 아니라 검증 도구의 재현성/호환성 개선**이다.
5. **결과:** 팀에서 받은 모델을 그대로 서빙하는 대신 **17회 QLoRA Validation 실험과 32후보 탐색·Clean/Cafe 소음 비교**로 별도 후보 선정 근거를 만들고, `Train65 2-epoch merged BF16`을 기본 STT로 적용했다. [PR #214](https://github.com/aihuman-7th/proj1-d/pull/214)에서 CUDA 로드·메모리, 8/27 자산 감사를 통해 서비스 기본 경로 및 rollback provenance를 확인했다. 후보의 **Cafe5 CER 약 0.2057 / WER 약 0.5137**과 소음별 열화 곡선은 **특정 heldout 합성/소음 조건**의 결과다. 성과를 **'후반 STT 추가 학습·평가 하네스를 직접 운용해 음성 주문 조건별 후보를 선택하고 프로덕션 기본 STT를 안전하게 교체'**로 표현하고, 고령 실사용자 음성 전반의 인식률 개선 또는 E2E 목표 달성이라고 쓰지 않는다.

### 8/26 Train65 Clean/Cafe 비교 (후반 실험 원장·대화)

| 대상 / 동일 평가 조건의 차이 | CER | WER |
| --- | ---: | ---: |
| Clean-kiosk | 0.135667 | 0.411471 |
| Cafe 15dB | 0.151167 | 0.437032 |
| Cafe 10dB | 0.167943 | 0.469451 |
| Cafe 5dB | 0.205689 | 0.513716 |

### 개인 귀속 및 숫자 중복 방지

- **기존 STT 담당의 기록:** 초기 small/medium 비교·방언 preproc 500개·동일 화자 중복 검출 후 350/77/73의 화자 분리 학습셋(**Train 350개·Validation 77개·Test 73개**)·QLoRA lr-sweep 및 도메인 merged 작업은 `surf7978` 작성 [#27](https://github.com/aihuman-7th/proj1-d/pull/27)·[#31](https://github.com/aihuman-7th/proj1-d/pull/31)·[#32](https://github.com/aihuman-7th/proj1-d/pull/32)·[#58](https://github.com/aihuman-7th/proj1-d/pull/58)과 [STT Worklog](https://github.com/aihuman-7th/proj1-d/blob/main/docs/worklogs/stt.md)에 귀속. 사용자 개인의 **8/22~26 17회 추가 실험/32후보 비교**와 별도로 기재한다.
- **최종 정량 보고서와 다른 시험:** [`docs/final-quantitative-results.md`](https://github.com/aihuman-7th/proj1-d/blob/main/docs/final-quantitative-results.md)의 **CER 0.1131/WER 0.3352·메뉴 89.5%·수량 99.3%·옵션 86.3%·latency 0.505s**는 **팀 도메인/주문 합성 평가 기록**으로 명시돼 있으며, 이 섹션의 **Train65 Clean/Cafe300 heldout .1357/.2057 CER**와 다른 모집단·checkpoint·설정이다. 서로 비교해 인과 개선율을 산출할 수 없다. 발표 표에는 기존 domain merged를 `최종/서비스 후보`로 표시하는 **버전 정합성 불일치**가 있으므로, 실제 **8/27 inventory·#214 코드 기본값**을 배포 경로의 직접 증거로 우선하고 정량 발표 문서의 내용은 별도 통계로 보존한다.
- **Validation/Test 봉인:** Golden-500 17회에는 **Train350/Val77**, Test73은 미사용. Golden500 Best Val CER 0.205863와 Train65 Clean CER 0.135667은 **다른 학습/평가 분포**라 수치 우위를 서로 비교할 수 없다.
- **모델 선정/serving 경계:** #214의 BF16 로드 확인은 **해당 환경에서 로컬 로딩·테스트가 성공**한 증거이며, Cloudflare/Vercel의 브라우저-마이크 전체 E2E 성공·real SNR 실제 카페 운영 개선은 MA-09/10에서 따로 검증한다. Model weights·AIHub 원본은 Git 미추적(라이선스/재현성 제약).
- **재현성 증거:** 후반 대화의 실험 결과·32후보/소음 조건은 강한 직접 실행 보고지만 **원시 ledger가 공개 Git에 모두 보존되지는 않음**. 공개 [8/27 최종 자산 감사](https://github.com/aihuman-7th/proj1-d/blob/main/docs/repository-cleanup-audit.md) 및 후보 평가 Spec·#214/218 코드/PR로 모델 ID·서빙·평가 체계를 재확인했다. 워크로그에 작성자가 미표기된 초기 항목을 내 작업으로 옮기지 않는다.
- **적합 직무:** 이 경험은 **ML 평가/MLOps·AI 시스템 엔지니어**에게 '다중 후보의 공정한 비교, 실험 범위/데이터 누수 통제, 모델 provenance/서빙 rollback 설계'로 강조한다. MA-07 NLU QLoRA 후반 추가 학습과 통합 MA-09, 최종 Real STT+NLU+TTS MA-10 수치는 여기서 중복 집계하지 않는다.

원본: [STT Worklog](https://github.com/aihuman-7th/proj1-d/blob/main/docs/worklogs/stt.md) · [초기 팀 STT #27](https://github.com/aihuman-7th/proj1-d/pull/27) · [#31](https://github.com/aihuman-7th/proj1-d/pull/31) · [#32](https://github.com/aihuman-7th/proj1-d/pull/32) · [#58](https://github.com/aihuman-7th/proj1-d/pull/58) · [후반 STT/서비스 #214](https://github.com/aihuman-7th/proj1-d/pull/214) · [평가 도구 #218](https://github.com/aihuman-7th/proj1-d/pull/218) · [벤치마크 Spec](https://github.com/aihuman-7th/proj1-d/blob/main/docs/specs/stt-candidate-benchmark.md) · [프로덕션/모델 SHA 감사](https://github.com/aihuman-7th/proj1-d/blob/main/docs/repository-cleanup-audit.md) · [발표용 최종 수치](https://github.com/aihuman-7th/proj1-d/blob/main/docs/final-quantitative-results.md). **골든 17회 추가 실험·32후보 전체 비교의 상세 수치는 사용자가 함께 확인한 이전 대화에서 보완했다.**

---

## 3-7. MA-07 — Qwen NLU 개인 후반 학습·다층 평가와 0.6B 서비스 후보의 안전한 통합 (2026-10-23 검증)

### 상황 → 문제 → 판단 → 조치 → 결과

1. **상황:** 말로의 음성 주문 서비스는 STT 전사문과 현재 Cart를 **NLU `OrderCommand[]`/Error**로 해석하고 결정론적 Order Engine에 넘겨야 한다. NLU 파트 류승민(`Padong47`)의 3,000문장(Train 2,400/Validation 300/Test 300) 데이터·Qwen3-1.7B QLoRA 설계/학습/최종 Test와 팀원 윤태현의 대형 stacked PR integration이 선행됐다([#195](https://github.com/aihuman-7th/proj1-d/pull/195), [#205](https://github.com/aihuman-7th/proj1-d/pull/205), [#206](https://github.com/aihuman-7th/proj1-d/pull/206), [#211](https://github.com/aihuman-7th/proj1-d/pull/211)). 팀장 김재훈은 **후반 로컬 1.7B 재실험·Qwen3-0.6B 모델 탐색·실서비스 안전/지연 검증·GPU 런타임 분리**를 직접 진행했다.
2. **문제:** 학습된 1.7B는 공식 Test 300건에서 **Exact 88.67%·Intent 95.38%·Slot 97.20%·Schema 99.67%**를 보였으나 [#211](https://github.com/aihuman-7th/proj1-d/pull/211)의 **Error Code Accuracy 70%(게이트 요구 90% 미달)**여서 운영 기본 모델로 승격할 수 없었다. 실제 카페 주문 문장에서는 숫자 표현(두/2잔/두 개/한 잔), 복수 메뉴, 메뉴 별칭·STT 오인식, 자기정정, 온도 미지정 시 처리가 불안정했다. **테스트셋의 JSON/Exact 점수가 높아도 실제 사용자 발화에서 안전한 명령을 내는지 보장하지 못했고**, 1.7B 추론 지연·Qwen Transformers와 STT/TTS용 의존성 충돌도 문제였다.
3. **판단:** **(a) 고정 Official Test(학습 결과 보고), (b) 모델 후보의 실제 주문 `challenge`(Generalization gap), (c) 서비스 후보 `focused-v1` 124건(계약 스트레스+guard), (d) 데스크톱 target-env 124건(동일 model/조건 호환성), (e) RuleFirst+deterministic guard 포함 생산 38문장 smoke(최종 앱 경로)**를 엄격하게 나눈다. 1.7B의 공식 Test 성능을 0.6B의 focused 124건과 한 표에서 직접 개선율로 비교하지 않는다. **정확도·schema 안정성·지연·실제 주문 문장 실패**를 함께 보고 더 작은 Qwen3-0.6B `epochs4_seed7` merged BF16+Prompt Lookup 8+deterministic guard로 생산 후보를 축소한다. LLM이 추출해도 잘못된 JSON/메뉴·수량은 **검증기/RuleFirst/Order Engine**에서 차단하고, 서로 다른 transformers를 요구하는 STT/TTS와 Qwen은 **HTTP NLU sidecar 별도 Python environment**에 격리한다.
4. **조치:**
   - **팀 기존 기반/명확한 귀속:** [#195](https://github.com/aihuman-7th/proj1-d/pull/195) 1.7B QLoRA Spec·[#197](https://github.com/aihuman-7th/proj1-d/pull/197) 학습 CLI·[#206](https://github.com/aihuman-7th/proj1-d/pull/206) 1.7B 런타임/스키마 변환·[#211](https://github.com/aihuman-7th/proj1-d/pull/211) 공식 Test 300건(88.67% Exact·70% Error code) 구현/실행은 **류승민**, 3,000행 데이터·stack을 main에 실제 통합한 [#205](https://github.com/aihuman-7th/proj1-d/pull/205)는 **윤태현**의 PR이다. 개인 학습 경험이라고 중복 귀속하지 않는다.
   - **개인 후반 1.7B 재현 (2026-08-25~26 이전 대화 증거):** 본인 GPU에서 Qwen3-1.7B QLoRA/LoRA 모델 번들과 Test 300건을 다시 검토했다. 이전 `experiences/mallo.md`의 개별 **3 epoch / 450 steps, Validation best step 350·Exact 91.67%·Intent 96.54%·Slot 96.10%·Schema 98.33%**는 **별도 Validation/run 초안**으로 기록돼 있고 원시 ledger 미공개라 조건·수치 재검증 전에는 주 대표 지표가 아니다. 고정 Test 300 재현 기록은 **Exact 89.67%·Intent 95.77%·Slot 97.58%·Schema 99.67%, warm 약 2초**라는 개인 대화 보고가 있지만, #211 팀 공식 Test와 **adapter/평가 스냅샷**이 다를 수 있으므로 별도 기록한다. 자체 real-use challenge **38건**의 unguarded 1.7B 결과는 **Exact 11.8%·Semantic 18.4%·multi-item 0%·quantity 35.3%**로 기록됐고, 같은 당시 조건의 0.6B 비교는 **Exact 26.5%·Semantic 28.9%·multi-item 20%·quantity 50%**였다. 이 수치는 **본인 대화 실행 보고**, 관련 raw predictions 재조회 전이므로 공개 PR 검증 수치와 등급을 구분한다.
   - **개인 0.6B 추가 학습·선정:** 2026-08-25의 `honorbear-20260825-021532-qwen3-0p6b-overnight`의 **15개 sweep 실험 자산**이 [8/27 자산 감사](https://github.com/aihuman-7th/proj1-d/blob/main/docs/repository-cleanup-audit.md)에 기록돼 있다. `epochs4_seed7`(Qwen/Qwen3-0.6B, LoRA r16/alpha32, ep4, seed7)을 BF16 merge해 서비스 후보로 선정했다. 일반 Validation 기록은 **Exact 95.33%·Schema 99.33%·Warm median 약 822ms/p95 약 1317ms**이며, 이는 아래 **focused-v1 별도 셋**이 아니다. `model.safetensors` SHA256 `2d72ee3dde4546a3409bdb433596be9f54039914309b8e2a13ed2500b7e4abe1`과 adapter/merged 출처를 [Qwen0.6B 서비스 후보 Spec](https://github.com/aihuman-7th/proj1-d/blob/main/docs/specs/nlu-qwen06b-service-candidate.md)에 보존했다. **15 sweep 폴더는 보존 목록**이고, 15개 모두 완전한 동일 조건 평가 실행 성공했다고 추정하지 않는다.
   - **서비스 후보 focused 124건 + 목표 환경 재검증 [개인 PR #209](https://github.com/aihuman-7th/proj1-d/pull/209):** Qwen3-0.6B `epochs4_seed7` merged BF16 + Dynamic Cache + Prompt Lookup 8 + deterministic guard의 합성 계약 스트레스 `focused-v1` **124건**에서 **Intent 100%·Slot 96.7787%·Exact 63.7097%·Error code 96.6667%·Multi-command order 100%·Schema 100%·Invalid menu 0%**, **guard가 42/124개 출력 변경**을 확인했다. 기존 계측 **median 857.5ms/p95 1182.5ms**와 달리 **RTX 5070 Ti·CUDA12.8·torch 2.11.0+cu128·transformers 5.15.0** target-env 재측정은 **median 1169.4ms/p95 1610.3ms**로 지연 **1.2초 p95 목표 미달**임을 보고했다. 이는 실제 LLM을 로드·생성한 벤치마크지만 전체 마이크/STT/TTS 앱 E2E는 아니다. 이 PR에서 `HTTPClientNLU`·NLU sidecar·`build_service(nlu=mock/qwen06b/http)`·WSL2 데스크톱 실행기를 구현하고 서로 다른 Python 의존성을 분리, merged 모델에 **PEFT 미설치** 상태 1건 생성 smoke·비정상 sidecar 실패 시 내부 URL/path 미노출 오류 처리를 확인했다.
   - **실사용 계약 보강 [개인 PR #214](https://github.com/aihuman-7th/proj1-d/pull/214):** `Qwen3-0.6B` 입력 정규화와 **deterministic guard/RuleFirst**를 보완해 음성 STT 오인식 별칭·숫자·복수 메뉴·기존 Cart 추가·온도 누락 등의 문제를 줄였다. **기존 38문장 challenge에서 RuleFirst+guard 포함 서비스 전체 계약은 38/38 Semantic·Exact·Schema·Quantity 등 통과**, 측정된 NLU live GPU latency는 **median 1235.7ms/p95 1630.3ms**(p95 1.5초 기준도 초과). **초기 unguarded 0.6B 38문장 28.9%**와 후속 **guarded 전체 38/38**는 **파이프라인 구성이 달라 순수 fine-tuning 정확도 28.9%→100%의 인과 개선이라 쓰지 않는다**. MA-09 서비스 통합 경로 [#215](https://github.com/aihuman-7th/proj1-d/pull/215)와 최종 사용자 레벨 성능 [#222](https://github.com/aihuman-7th/proj1-d/pull/222)은 별도 작업군으로 기록한다.
5. **결과:** 1.7B 모델의 높은 공식 Test Exact **88.67%에도 Error Code 70% gate 실패**와 느린 추론·실제 주문 표현 회귀가 있었음을 근거로, **작은 0.6B + BF16 merge + deterministic guard + 별도 sidecar** 구조를 선택·통합했다. 0.6B의 동일 124건 Focused 계약에서 **Intent 100%, Slot 96.78%, Schema 100%**, 38문장 서비스 smoke **38/38**을 기록하며 주문 명령의 의미·계약을 우선한 품질 결정 사례를 남겼다. 하지만 **target-env NLU p95 1.610초/후속 p95 1.630초는 1.2/1.5초 목표에 미달**했고, 실제 E2E Qwen fallback은 [#222](https://github.com/aihuman-7th/proj1-d/pull/222)에서 **p95 3.386초**로 전체 1.5초 요구를 달성하지 못했다. 따라서 '모델을 경량화해 전체 주문 응답이 실시간 요구를 충족'이라고 주장하지 않고, **'실제 주문 경로에서 안전한 NLU 선택·검증과 시스템 통합, 미달 latency를 정량 노출'**로 표현한다.

### 평가 모집단과 성과 귀속을 섞지 않는 기준

| 단계 | 모델/시스템 경계 | 건수·핵심 결과 | 사용 방법 |
| --- | --- | --- | --- |
| 팀 공식 Test | 1.7B 최종 보정 adapter / Guard 포함 여부 별도 | **300건**, Exact **88.67%**, Error Code **70%** | 팀원 #211의 gate 실패 결과; 개인 재현·후속 real-use와 분리 |
| 개인 재현·실전 발화 | 1.7B/0.6B 모델 후보를 주문 challenge에서 직접 비교(당시 대화) | **38문장**, 1.7B semantic **18.4%**, 0.6B semantic **28.9%** | Raw 개인 실행 결과, 공개 artifact 완전 미확인 |
| 일반 Validation | 0.6B epochs4_seed7 (서비스 후보 선택) | Exact **95.33%**, schema **99.33%** | 모델 선정 단계, focused 실험과 합산 금지 |
| Focused contract-stress | 0.6B merged + PromptLookup8 + guard | **124건**, Intent **100%**, Slot **96.78%**, Schema **100%**, Exact **63.71%** | 서비스 계약 스트레스, 실제 고령 사용자 일반화 아님 |
| Target env 124 재검증 | 동일 0.6B, RTX5070Ti/CUDA12.8 | **124건**, warm p95 **1.610초** (기존 1.182초) | 동일 의미/스키마 지표 재현·지연 regression |
| Production real-use challenge | 0.6B+RuleFirst/normalizer/guard 통합 | **38/38** pass, GPU NLU p95 **1.630초** | 앱 보정 기능 검증, 순수 0.6B 성능 100% 아님 |
| E2E Real Voice fallback | 실제 STT·Hybrid NLU·TTS·브라우저 | **30턴**, Qwen fallback p95 **3.386초** | **MA-10** 고유 제품 E2E 성과, MA-07 모델 평가로 이중 계상 금지 |

### 귀속/수치/운영 주장 제한

- **1.7B 데이터·학습 파이프라인과 초기 최종 Test:** 류승민(`Padong47`)의 [#195](https://github.com/aihuman-7th/proj1-d/pull/195)·[#197](https://github.com/aihuman-7th/proj1-d/pull/197)·[#206](https://github.com/aihuman-7th/proj1-d/pull/206)·[#211](https://github.com/aihuman-7th/proj1-d/pull/211) 및 팀원 윤태현(`surf7978`) [#205](https://github.com/aihuman-7th/proj1-d/pull/205)의 main 정합화. 이 팀의 성과를 '내가 3,000개 수집·학습 pipeline 전체를 최초 개발'이라고 쓰지 않음.
- **최종 서비스 후보**는 **Qwen3-0.6B epochs4_seed7 merged BF16 + Prompt Lookup8 + deterministic guard**이며, 1.7B는 reference/demo. 최종 production의 규칙 기반 RuleFirst는 비정형 발화에 대한 **NLU fallback 경로**와 결합; '모든 turn에서 0.6B LLM을 호출해 100% 정답'은 허위다.
- **#209 124건의 42개 guard 변경** 때문에 100% Intent/Schema를 파인튜닝 모델의 raw 출력 자체 품질로 주장 불가. **0.6B 일반 Validation 95.33% Exact**와 **Focused 63.71% Exact**는 다른 셋/후처리 조건.
- **지연 실패 기록 필수:** #209 목표 환경 p95 **1.610s**는 1.2s target 위반, #214 NLU p95 **1.630s**도 p95 1.5s 목표 초과, #222 fallback E2E p95 **3.386s**는 전체 E2E requirement 1.5s 실패. MA-09/10에서 병목 추적/제품 실패 보고로 분리.
- **지표 provenance:** 개인 대화의 1.7B Test 재평가 **Exact 89.67%**와 팀 [#211](https://github.com/aihuman-7th/proj1-d/pull/211) 공식 Test **88.67%**는 버전·실행 환경/판정 차이를 확인해야 하므로 둘 다 남기되 **이력서 주력 수치는 공식 88.67%, gate 실패 70%**를 사용한다. 기존 경험기술서의 Validation best step350/91.67%도 원본 실험 확인 전 대표치로 쓰지 않는다.
- **원시 아티팩트**: 0.6B 모델 원본 SHA/124건 focused 실행 및 PR#209 target summary는 Git 문서로 재현 가능하나, 독립 개인 38 real-use 초기 비교와 1.7B 재학습 checkpoint, 15개 모든 sweep의 완료 로그는 **대화/로컬 자산 기록**에만 근거가 있다. 모델 가중치·원본 Test JSONL은 Git 추적 대상이 아님.
- **직무 활용:** AI Model Engineer/MLOps에는 '작은 모델이 실제 주문 환경에서 더 유리해도 계약·지연 미달을 노출하면서 guard를 겹친 선택과 실환경 비교', Backend/Agent 시스템에는 'RuleFirst/HTTP sidecar/PEFT 분리·비정상 응답 누출 방지·계약 게이트' 중심으로 제시.

원본: [1.7B 평가 Spec](https://github.com/aihuman-7th/proj1-d/blob/main/docs/specs/nlu-qwen3-qlora-finetuning.md) · [1.7B 최종 Test #211](https://github.com/aihuman-7th/proj1-d/pull/211) · [0.6B 서비스/Sidecar #209](https://github.com/aihuman-7th/proj1-d/pull/209) · [0.6B 서비스 후보 Spec](https://github.com/aihuman-7th/proj1-d/blob/main/docs/specs/nlu-qwen06b-service-candidate.md) · [Focus124 Guarded Summary](https://github.com/aihuman-7th/proj1-d/blob/main/docs/evidence/nlu/focus3-20260825-115728/qwen06b-promptlookup8-guarded.summary.json) · [대상 환경 Summary](https://github.com/aihuman-7th/proj1-d/blob/main/docs/evidence/nlu/pr209-target-env/validation-summary.json) · [모델 자산 감사](https://github.com/aihuman-7th/proj1-d/blob/main/docs/repository-cleanup-audit.md) · [실사용 guard #214](https://github.com/aihuman-7th/proj1-d/pull/214).

---

## 4. 진행 원칙

1. 각 MA 단위는 **프로젝트 원본/개인 PR+리뷰/실제 로그/대화 기억**을 별도로 확인하고 5단계를 작성한다.
2. 수치의 **baseline vs friendly, Mock vs Real, local vs public, initial survey vs final evaluation** 조건을 혼합하지 않는다.
3. 팀 전체 결과를 개인 실행 결과로 옮기지 않고, 최종 서빙 모델은 후보 실험과 분리한다.
4. 12개 후 전부 교차검증하고 **AI Agent·Backend·Voice AI·UX 직무별 핵심 3~5개**를 선정한다.
5. `experiences/README.md`와 연대기는 최종 감사 뒤 수정하고, `resume.md`는 다른 프로젝트 전체 우선순위와 함께 조정한다.
