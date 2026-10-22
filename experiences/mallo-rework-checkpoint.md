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
| **MA-02** | OrderCommand·Validator·결정론적 Order Engine | [PR #23](https://github.com/aihuman-7th/proj1-d/pull/23), [#55](https://github.com/aihuman-7th/proj1-d/pull/55) | 실제 명령 구현·메뉴 가격 SSOT·원자성, AI 코딩 협업 | 대기 |
| **MA-03** | STT→NLU→주문→TTS 계약·VoicePipeline·Mock E2E | [PR #68](https://github.com/aihuman-7th/proj1-d/pull/68), [#69](https://github.com/aihuman-7th/proj1-d/pull/69) | Mock/Real 호출, 297 tests가 무엇을 테스트하는지 구분 | 대기 |
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
- **다음 MA-02:** 같은 원칙을 실제 `OrderCommand[]` 계약과 [Order Engine #55](https://github.com/aihuman-7th/proj1-d/pull/55)에서 어떻게 구현했는지 PR diff·테스트·원래 팀원 역할을 세분해 조사한다.

원본: [팀 README](https://github.com/aihuman-7th/proj1-d/blob/main/README.md) · [PRD](https://github.com/aihuman-7th/proj1-d/blob/main/docs/prd.md) · [TRD](https://github.com/aihuman-7th/proj1-d/blob/main/docs/trd.md) · [Task/역할](https://github.com/aihuman-7th/proj1-d/blob/main/docs/tasks.md) · [설문](https://github.com/aihuman-7th/proj1-d/blob/main/docs/survey-pipeline-update.md) · [PR #83](https://github.com/aihuman-7th/proj1-d/pull/83).

## 4. 진행 원칙

1. 각 MA 단위는 **프로젝트 원본/개인 PR+리뷰/실제 로그/대화 기억**을 별도로 확인하고 5단계를 작성한다.
2. 수치의 **baseline vs friendly, Mock vs Real, local vs public, initial survey vs final evaluation** 조건을 혼합하지 않는다.
3. 팀 전체 결과를 개인 실행 결과로 옮기지 않고, 최종 서빙 모델은 후보 실험과 분리한다.
4. 12개 후 전부 교차검증하고 **AI Agent·Backend·Voice AI·UX 직무별 핵심 3~5개**를 선정한다.
5. `experiences/README.md`와 연대기는 최종 감사 뒤 수정하고, `resume.md`는 다른 프로젝트 전체 우선순위와 함께 조정한다.
