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
| **MA-04** | MeloTTS·Qwen3-TTS baseline, 친절체 학습과 모델 선정 | [PR #63](https://github.com/aihuman-7th/proj1-d/pull/63), [#71](https://github.com/aihuman-7th/proj1-d/pull/71), [#152](https://github.com/aihuman-7th/proj1-d/pull/152) | G_2000 비교와 G_2800 운영 벤치마크 서로 다른 설정 | 대기 |
| **MA-05** | Adaptive TTS Level 0~2·한국어 수사 정규화·쇼케이스 | [PR #97](https://github.com/aihuman-7th/proj1-d/pull/97), [#116](https://github.com/aihuman-7th/proj1-d/pull/116), [#150](https://github.com/aihuman-7th/proj1-d/pull/150) | 청음 체감 개선 vs 계산된 RTF·출력 호환성 | 대기 |
| **MA-06** | Whisper Medium STT QLoRA 후보 탐색·Train65·서빙 모델 | [PR #218](https://github.com/aihuman-7th/proj1-d/pull/218), STT worklogs·후보 평가 | STT 담당/개인 추가 학습 구분, CER/WER·speaker split | 대기 |
| **MA-07** | NLU 모델 추가 학습·생산 Qwen 0.6B runtime | [PR #209](https://github.com/aihuman-7th/proj1-d/pull/209), [#214](https://github.com/aihuman-7th/proj1-d/pull/214) | 개인 Qwen 1.7B 추가 실험 vs 팀 0.6B production | 대기 |
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

## 4. 진행 원칙

1. 각 MA 단위는 **프로젝트 원본/개인 PR+리뷰/실제 로그/대화 기억**을 별도로 확인하고 5단계를 작성한다.
2. 수치의 **baseline vs friendly, Mock vs Real, local vs public, initial survey vs final evaluation** 조건을 혼합하지 않는다.
3. 팀 전체 결과를 개인 실행 결과로 옮기지 않고, 최종 서빙 모델은 후보 실험과 분리한다.
4. 12개 후 전부 교차검증하고 **AI Agent·Backend·Voice AI·UX 직무별 핵심 3~5개**를 선정한다.
5. `experiences/README.md`와 연대기는 최종 감사 뒤 수정하고, `resume.md`는 다른 프로젝트 전체 우선순위와 함께 조정한다.
