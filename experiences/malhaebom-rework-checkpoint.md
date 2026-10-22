# 말해봄 Mk.2 경험 복원 체크포인트

> 2026-10-21 1차 조사. 허밍블럭스 재정리와 같은 **근거 수집 → 개인 작업 단위 복원 → 누락 역검증 → 경험 원본 확정 → 포트폴리오 선별** 절차를 따른다.
>
> 마스터 `resume.md`는 수정하지 않는다. 현재 `malhaebom.md`는 **채용 제출용 최종 확정본이 아니라 근거·귀속을 계속 검증할 Draft**다.

## 1. 조사 범위와 현 상태

- 원천 저장소: [aihuman-7th/proj2-3](https://github.com/aihuman-7th/proj2-3), 로컬 C2C 등록 경로 `~/proj2-3`.
- 확인 시점: 2026-10-21, 로컬 브랜치 `main`, 작업 트리 clean, HEAD `489b1bf`(2026-10-12 병합 이력). GitHub 연결의 직접 저장소 조회는 권한/연결 범위로 404였으나, 등록된 로컬 프로젝트 읽기와 C2C PR 목록·Git history는 접근 가능했다.
- 프로젝트 범위: 2026-09-18~10-08; 10-10~12의 배포·소스 동기화/마무리 변경은 별도 기록. 팀원 3인(곽호준, 김재훈, 서정현).
- 우선 교차확인 문서: `README.md`, `docs/PROJECT_OVERVIEW.md`, `docs/CURRENT_STATE.md`, `docs/TARGET_STATE.md`, `docs/REQUIREMENTS.md`, `docs/CONSTRAINTS.md`, `docs/AGENT_REPORT.md`, `EVAL_REPORT.md`, `docs/stt-finetuning-lineage.md`.
- 주의: docs의 과거 `TARGET_STATE` 체크박스와 날짜별 Worklog의 미완료 표현은 현재 구현 완료/미완료의 단독 증거가 아니다. `CURRENT_STATE`와 날짜별 실행 증거를 대조한다.

## 2. 기여 귀속 증거 수준

- **A — 개인 직접 확인:** 작업기록에 김재훈 명시 + 구체적 수행 내용/테스트 또는 사람이 직접 작성한 검토표.
- **B — 귀속 교차검증 대기:** 프로젝트 Git diff·실험 기록은 있으나 실행자·설계자·리뷰어의 구분이 미확정. 당시 개인 작업 대화/로그는 추가 비교가 필요.
- **T — 팀 결과:** 저장소 또는 운영 실험에서 결과는 확인되지만 개인 단독 기여는 아님.
- **U — 미확인:** 원본 로그/실험 환경/품질 게이트 등이 없거나 자료 간 충돌.

다른 팀원의 최초 구현, AI 코딩 에이전트가 작성한 변경, 김재훈의 요구사항 정의/평가/리뷰/통합/병합을 서로 다른 역할로 기록한다. 모든 merge commit을 개인 직접 코딩 성과로 계산하지 않는다.

## 3. 작업 단위 원장 — 조사 단계

| ID | 작업 | 현재 증거 | 귀속 | 다음 확인 |
| --- | --- | --- | --- | --- |
| MB-01 | 문제 정의·경쟁 비교·프로젝트 범위 고정 | PR #59·#60, 9/23 Worklog, D2, 10/1 Worklog | **A(문서 작업 지시·검토)/T(기획 원형)** | **1차 확인 완료** — 공개자료 조사와 직접 사용은 분리; 초기 기획 주도권은 추가 미확인 |
| MB-02 | Agent의 Prompt·Skill·MCP·서버 정책 분리 | PR #119의 직접 인라인 리뷰, PR #116의 3차 수정요청, PR #123 신뢰 경계 문서; 실제 코드 및 테스트 | **A(검토·신뢰 경계 문서) / T(주요 코드 구현)** | **검증 완료** — 핵심 구현 PR 작성은 팀원; 검증 계약 결함 지적과 문서화는 개인 귀속 |
| MB-03 | OpenRouter 다양성 탐색·Final Four·서비스 후보 선정 | 사용자 커밋 PR #36/#37/#40/#41/#47, 팀원 리뷰, 실험·서비스 E2E Worklog | **A(실험 구현·운영 및 후보 선정 흐름) / T(기존 평가 기반)** | **검증 완료** — Codex/Antigravity 보조 개발을 명시; 계약·비용·E2E 근거의 모집단 분리 |
| MB-04 | 합성 평가셋 30건의 정답 사람 검토·정정 | reviewer-assignment, 9/30 offline regrade | **A** | 검토표·수정 반영 커밋 연결만 추가 |
| MB-05 | Prompt/Skill A/B 및 회귀 대응 | 10/2 30건, 10/6 Skill smoke | T/B | 직접 만든 harness/실험 실행과 팀 검토·프롬프트 작성 분리 |
| MB-06 | 로그인 사용자 공고 원문 분석 E2E | 10/1 14:00 김재훈 Worklog, 토큰 0600 검증 | **A** | 해당 PR 번호·diff, 반복 재현 여부 확인 |
| MB-07 | FastMCP stdio/HTTP, /api/agent, DB 저장 | AGENT_REPORT, CURRENT_STATE, 서버 코드, PR | T/B | stdio 도구 기여/통합 검증과 구현 코드 책임 구분 |
| MB-08 | Whisper Medium 50k → B0 → C2 학습 | stt-finetuning-lineage, 50k/C2 실험 | T/B | 본인 수행 train configs/로그/checkpoint SHA, 운영 모델 선택 |
| MB-09 | 실제 마이크/STT·NLU·Supabase 통합 E2E | checkpoint, EVAL_REPORT, 일부 worklog | T/B | 인증 사용자 부분 E2E와 운영 전체 E2E 분리 |
| MB-10 | Langfuse Prompt/trace·운영 관측 | AGENT_REPORT, 10/8~10/9 운영 증거 | T/B | 직접 운영 trace 확인/배포와 팀 공통 관측 분리 |
| MB-11 | UI/UX 개선과 소개 페이지·배포 | 10/10 README, 운영 비교 PR, 별도 showcase 저장소 | T/B | 개인 제안·목업·별도 showcase 설계와 팀 프론트 구현 구분 |
| MB-12 | D1~D12 체크포인트·PR·CI 협업/종료 | 10/1 개인 Worklog, PR/CI 기록 | A/B | 직접 처리한 리뷰·브랜치·테스트·병합 대표 3~5건 선정 |

## 3-1. MB-01 — 문제 정의·경쟁 조사·제품 범위 결정 (1차 검증 완료: 2026-10-21)

### 근거 연결

- [PR #59 — 문제정의서·차별화 전략](https://github.com/aihuman-7th/proj2-3/pull/59): **2026-09-23, 작성 계정 nanocode00 / 커밋 작성자 JaeHun Kim**. 첫 커밋에 `docs/product/problem-definition.md`, `docs/product/differentiation-strategy.md`, Worklog 신설; 다음 커밋에서 활성 기준·4상태 계약·canonical 관계 정합화. **코드/서비스 동작 변경은 없음**. 사용자 요청에 따라 AI가 Markdown 원문 작성·정합화 후 PR 생성했으므로 Git author만으로 수작업 작성이라고 주장하지 않는다.
- [PR #60 — 현황·문서 정합화](https://github.com/aihuman-7th/proj2-3/pull/60): 같은 날 기존 README/CURRENT_STATE/와이어프레임의 과거·현재 표현 충돌을 정리. 작성 계정 nanocode00; 제품의 새로운 기능 구현이 아니다.
- [PR #50 — GitHub 기술 증거 기반 공고 핏 Spec·경쟁 비교](https://github.com/aihuman-7th/proj2-3/pull/50): **SJeongHyeon 명의 원 커밋**, `docs/competitor-comparison.md`와 `docs/specs/github-evidence-job-fit.md`를 생성. MB-01에 연관된 팀원 조사로 보되, 김재훈의 단독 원고·기능 설계로 귀속하지 않는다.
- `docs/PROJECT_OVERVIEW.md`, `docs/DECISIONS.md`(2026-09-16 대상·질문·RAG 제외·검증 경계), `docs/product/problem-definition.md`, `docs/product/differentiation-strategy.md`, `docs/research/COMPETITOR_RESEARCH.md`(기획서 공개자료 요약), `docs/worklogs/2026-09-23_0930_product-definition-and-differentiation.md`.
- `docs/worklogs/2026-09-30_checkpoint-schedule-rewrite.md` 및 `docs/worklogs/2026-10-01_instructor-confirmed-checklist-items.md`: **경쟁 서비스 3곳 직접 체험 결과·출력·스크린샷을 확인하지 못했고, 강사 별도 확인에 따라 증빙 추적을 종료**했다. 제출 기준 완료와 실제 사용 실측을 혼동하지 않는다.

### 상황 → 문제 → 판단 → 조치 → 결과

1. **상황:** 기존 말해봄의 녹음·STT 자산을 이어받아 2주 내 구현·검증 가능한 신입 개발자 면접 코칭 에이전트를 정해야 했다. 초기 기획은 팀 산출물로 보관되어 있으며 개인 단독 최초 기획을 입증할 자료는 아직 없다.
2. **문제:** 공개 경쟁 서비스도 공고 맞춤 질문·음성 면접·피드백·꼬리질문·재연습 기능을 제공하므로, 기능 목록 자체로 차별성을 주장하기 어렵다. 또한 자동 공고 분석 같은 확장 기능이 현재 구현·필수 MVP에 포함된 것처럼 보일 수 있었고, 문서별 `undetermined` 상태·활성 기준 표현이 어긋났다.
3. **판단:** 핵심 사용자를 **신입 백엔드 취업준비생**, 질문을 **프로젝트 문제 해결 경험**으로 좁히고, 공고 요구사항 1~2개·실제 전사문·현재 답변 인용을 기반으로 누락 설명을 찾는 코칭과 **동일 기준 재답변 비교**에 집중한다. **사용자가 말하지 않은 경험 생성 금지, 미언급과 미수행 구분, LLM 의미 해석 vs 서버 권한·검증·상태 변경 분리**를 안전 경계로 둔다.
4. **조치:** 9/23 사용자 요청으로 AI와 협력해 문제정의서와 차별화 전략을 **서로 다른 목적의 문서**로 추가하고 PR #59에 제출했다. 원래 문서의 3상태·C/R/A/D/V 일괄 적용 오해를 질문별 활성 기준·4상태로 정정했으며, PR #60에서 현재 기능·목표·역사 기록의 충돌을 분리했다. 경쟁 비교는 공개 서비스 설명을 기준으로 작성했다.
5. **결과:** `사용자/과업/판단 경계/LLM-서버 역할/성공 기준`이 분리된 제품 문서 체계를 구축했다. 이후 10/1 실제 공고 분석 구현 근거를 반영하고, 10/4에는 향후 공고 탐색·직무 확장을 **일괄 금지한 문구만 정리**했다. 초기 2주 검증 초점이 곧 장기 제품의 영구 기능 금지는 아니라는 버전 경계를 남겼다. 이 문서 작성 자체를 **사용자 3~5명 인터뷰 성공, 경쟁사 실사용 3곳 완료, 사용자 개선 효과 실증**으로 서술하지 않는다.

### 개인 기여와 팀 결과 분리

- **개인 귀속 가능:** 9/23 문제정의서·차별화 문서 작성 **요청·범위 선택·검토·PR 기반 제출 관리**, 현재 MVP와 차후 자동분석의 경계 확정 및 문서 정합성 수정 요청. Git 작성자 계정 `nanocode00` 및 당시 사용자-에이전트 작업 흐름으로 확인된다. 생성 Markdown 초안·정합화 작업은 AI 지원이었으므로 ‘본문 전체 직접 수기 작성’으로 설명하지 않는다.
- **팀 전체/미확정:** 프로젝트 최초 문제 발견·기획 원형, 초기 타깃·제약 도출을 혼자 이끌었다는 주장, 경쟁 서비스 기능의 실제 직접 사용과 품질 평가, 최종 제품 기능 전체 설계. PR #50의 별도 GitHub·공고 증거 기반 기술 설계는 팀원 기여.
- **제품 지표와 한계:** 문제정의서에 나온 초기 3~5명 인터뷰, 80% 행동 가능성 등은 **사전 성공 가설**이지 실제 측정 완료 결과가 아니다. 제품의 비교우위·실사용 개선율을 의미하는 데이터는 이 작업군에서 확보되지 않았다.

### 채용 활용 판단

**강조 정도: 보조 사례.** 엔지니어링 관점의 ‘사용자 문제 → 좁은 범위 → 모델/서버 신뢰 경계 → 검증 조건’ 설정 역량을 보여 주지만, 단독 주력 경험으로는 제품 효과 실측과 개인 코드 귀속이 약하다. MB-02(Agent 신뢰 경계), MB-05(평가·회귀)와 연결해 **왜 모델 제어·평가 정책을 그렇게 만들었는가**라는 배경으로 활용한다.

**1차 판정:** 문서화·정합화에 대한 개인 실행/의사결정 **확인**; 초기 프로젝트 기획 원형/경쟁사 실사용을 개인 성과로 표현하는 것은 **보류**. 추가 복원은 PR #59/#60 및 위 Worklog를 이미 근거로 연결했으므로 MB-02로 이동.

---

## 3-2. MB-02 — Prompt·Skill·MCP·서버 정책 분리 (검증 완료: 2026-10-21)


### 5단계 경험 서사 — 상황 → 문제 → 판단 → 조치 → 결과

1. **상황:** 말해봄은 답변·공고·면접 대화 등 여러 입력을 LLM에 연결하고 있었고, Prompt 역할 지시와 도메인 Skill 규칙을 분리하며 MCP 조회·서버 검증의 신뢰 경계를 명확히 해야 했다. 핵심 코드 개발은 팀원과 AI 코딩 도구를 포함한 공동 작업이었다.
2. **문제:** 규칙을 파일별로 분리하더라도 잘못된 템플릿 슬롯이 system prompt에 그대로 남을 수 있었다. 별도의 D6 검증 하네스는 실제 API에 없는 필드·헤더 때문에 정상 호출을 실패로 간주하거나, HTTP 200만으로 DB 저장 성공을 주장해 **거짓 음성·거짓 양성**을 만들 위험이 있었다.
3. **판단:** 신뢰할 수 있는 것은 서버가 고정한 시스템 규칙·권한·검증 계약이며, Prompt/Skill·MCP 도구 결과·LLM 출력을 무비판적으로 성공 근거로 사용할 수 없다. 모델 입력의 변수 검증을 실패 우선으로 설계하고, **HTTP 성공·출력 계약 통과·실제 DB 저장**은 각각 별개의 검증 조건으로 다뤄야 한다.
4. **조치:** 개인 계정 `nanocode00`으로 팀원 구현 [PR #119](https://github.com/aihuman-7th/proj2-3/pull/119)의 `{{SKILL_TEXT}}`·`{{skill_text }}` 우회 사례를 정확히 짚어 미지원 슬롯 거절과 회귀 테스트를 요청했다. [PR #116](https://github.com/aihuman-7th/proj2-3/pull/116)에서는 테스트 계약 오류를 세 차례 수정 요청해 검증 범위를 바로잡았고, [PR #123](https://github.com/aihuman-7th/proj2-3/pull/123)에 사용자·외부 원문·MCP·LLM 출력의 신뢰 경계를 문서화했다.
5. **결과:** #119의 템플릿 검증 패치 및 **PR 브랜치 기준 526개 백엔드 테스트 통과 기록**, #116의 검증 목표를 **Agent HTTP 계약 확인**으로 정정하고 DB persistence는 **별도 수동 SQL 확인**으로 분리한 결과를 확인했다. **결함 발견·수정 요청·검증 기준 정립은 개인 기여**, 실제 분리·MCP 하네스 주요 코드 구현은 팀원 기여다. 이를 운영 전체 품질·보안성 입증으로 확대하지 않는다.


### 구현 구조의 사실 관계 — 팀 전체 시스템

- [AGENT_REPORT 4·5장](https://github.com/aihuman-7th/proj2-3/blob/main/docs/AGENT_REPORT.md)과 코드 대조: `prompts/`는 모델의 역할·출력 형식·입력 취급, `skills/`는 면접 유형별 기준·근거 인용·보류 규칙, `backend/app/services/coaching_runtime.py`는 provider 요청·재시도·관측 연결, 서버 출력 검증은 `backend/services/llm.py` 및 Pydantic 계약, `mcp_server/server.py`는 권한 제한 읽기 도구.
- `backend/app/services/prompt_templates.py`가 프롬프트 파일을 읽을 때 폴더 경계·빈 템플릿을 검사하고 `{{skill_text}}` 슬롯 정확히 1개 및 알려진 변수만 허용해 **삽입된 Skill 본문을 재치환하지 않음**을 현재 코드에서 확인했다.
- `backend/app/services/coaching_prompt.py`는 신규 버전의 템플릿·Skill 조합과 v3/v4/v4.1 재현 빌더를 분리한다. 템플릿과 과거 Skill snapshot을 함께 보존해야 완전 재현 가능하다는 한계까지 문서화됨.
- `mcp_context_gateway.py`가 내부 stdio FastMCP의 `get_question`, `get_practice_attempt`, `get_job_context` 세 도구를 통해 서버가 허용한 문맥을 읽으며, 별도 HTTP `/mcp`는 네 개의 읽기 도구와 인증·capability scope를 제공한다. **임의 tool을 자유 선택하는 에이전트/쓰기 가능한 MCP와 혼동 금지.**
- `docs/TRUST_BOUNDARY.md`: 사용자 입력·외부 공고·MCP 데이터·LLM 응답을 시스템 지시와 분리하고 출처, 사용자 소유권, 허용 ID, 원문 quote를 서버에서 확인한다. Langfuse에는 비식별 metadata만 전달한다. 실패·재시도·계약 검증을 코칭의 의미 품질 보증과 구분한다.

### PR 및 리뷰로 확인한 개인 기여

1. **PR #119 — Prompt·Skill 분리 구현은 곽호준 작성, 김재훈은 구체적 결함을 리뷰로 발견.** [PR](https://github.com/aihuman-7th/proj2-3/pull/119) · [직접 리뷰](https://github.com/aihuman-7th/proj2-3/pull/119#pullrequestreview-5359976796) · [인라인](https://github.com/aihuman-7th/proj2-3/pull/119#discussion_r4139564132).
   - 2026-09-30 `nanocode00`이 정규식 `[a-z_]+`에 맞지 않는 **`{{SKILL_TEXT}}`·`{{skill_text }}` 같은 오타 슬롯이 검증을 우회해 모델 system prompt에 남는 문제**를 특정하고, 모든 미지원·미완성 슬롯을 거절하는 검증과 회귀 테스트를 요청했다.
   - 후속 커밋이 parser를 전체 `{{...}}` 검증으로 보강. PR 기록에는 분리 경계 10개·백엔드 전체 526개 통과가 남았다. 이 테스트 실행/패치 적용은 팀 구현 기록이며 **김재훈이 직접 코드를 수정했다고 주장하지 않는다**.
   - 초기 코멘트 후 승인에서는 미실행 Docker/실제 Langfuse 검증을 통과 근거로 사용하지 않도록 명시하고 후속 이슈 #121을 언급했다. 승인 자체는 리뷰 이후 패치 구현의 소유권과 다르다.
2. **PR #116 — MCP/Agent D6 증거 하네스는 서정현 작성, 김재훈은 3차에 걸쳐 검증 계약 결함을 지적.** [PR](https://github.com/aihuman-7th/proj2-3/pull/116) · [리뷰 내역](https://github.com/aihuman-7th/proj2-3/pull/116).
   - 첫 검토에서 실제 `CoachingResponse`에 없는 `coaching_run_id`·`mcp_evidence/mcp_tools`와 미제공 `X-Trace-Id`를 하네스가 전제해 **정상 200 응답도 실패로 판단**할 수 있음을 지적했다.
   - 후속 검토에서 반대로 `HTTP 200 + ok/needs_more_info`만으로는 **DB `coaching_runs` 저장 완료·해당 revision 연결을 보장하지 못한다**는 거짓 양성 위험과 체크포인트 중복 상태를 지적했다.
   - 구현 담당이 PR 목적을 인증 Agent 응답 계약으로 좁히고 DB 저장은 별도 수동 SQL 증거로 분리한 뒤 승인했다. **테스트의 주장과 실제 관측 범위를 일치시킨 품질 리뷰**로 개인 기여 확인.
3. **PR #123 — 신뢰 경계 문서와 공식 체크포인트 증거 연결은 김재훈 계정 커밋.** [PR](https://github.com/aihuman-7th/proj2-3/pull/123) 첫 커밋에 `docs/TRUST_BOUNDARY.md`, `docs/research/DOMAIN_SOURCE_REGISTER.md`, 체크포인트 정합화가 포함. 후속에 D4 사람 검토 커밋 일부가 revert·분리됐으므로 **이 PR 자체를 D4 검토 완료의 근거로 재사용하지 않는다**. 신뢰 경계 명시와 출처/이용조건 정리는 개인 작업으로 기록 가능.
4. **PR #142 — stdio MCP 장애시 `503 + mcp_error + retryable` 코드·테스트는 서정현 작성, 김재훈이 검증 후 승인.** [PR](https://github.com/aihuman-7th/proj2-3/pull/142) 리뷰에서 서버·클라이언트 `mcp_error` 타입 불일치를 후속 수정 대상으로 지적. 코딩 소유권은 팀원.
5. 별도 D6 실행/권한 검증 코드도 [PR #117](https://github.com/aihuman-7th/proj2-3/pull/117), 저장 검증은 [PR #122](https://github.com/aihuman-7th/proj2-3/pull/122)에서 **서정현 작성** 확인. [9/21 코칭 런타임 보강 Worklog](https://github.com/aihuman-7th/proj2-3/blob/main/docs/worklogs/2026-09-21_0934_coaching-runtime-hardening.md)는 작업자 Codex 표기, 당시 로컬 243 tests 성공·운영 추가 검증 보류. 커밋/PR 개인 귀속은 이 문서만으로 확정하지 않는다.

### 검증 수준과 채용용 해석

- 확인된 구조/테스트는 **소스·로컬 단위/통합 테스트 근거**. Prompt·Skill 분리 PR에서는 Python 3.13 격리 환경에서 **525개, 리뷰 수정 뒤 526개 백엔드 테스트** 기록; 같은 원 작업 폴더의 547개는 미커밋 변경이 더 섞인 환경이라 PR 수치로 혼용하지 않는다.
- 이것만으로 **운영 배포/실제 사용자 E2E·Langfuse production·코칭 의미 품질 향상**이 확인됐다고 주장하지 않는다. 나중의 운영 확인이나 Prompt A/B 결과는 MB-09/MB-10/MB-05에서 별도 추적.
- **개인 기여 강도:** 코어 구현 전반의 단독 개발자는 아니지만 **Prompt 계약 검증 허점 식별, 잘못된 E2E pass/fail 기준 바로잡기, 서버/모델/외부 도구 신뢰 경계 정리**가 강한 개인 경험으로 귀속됨. 단순 PR 승인 수보다 *무슨 문제가 어떻게 잘못 검증되고 있었으며 어떤 기준으로 수정시켰는지*가 채용 활용 가치가 높다.

**1차 판정:** MB-02는 **개인 기술 리뷰·검증 판단·신뢰 경계 문서화 A 확인**, **Prompt/Skill/MCP/서버 코어 구현 T(팀 소유)**. 검증 완료. MB-03에서는 모델 선정 하네스 구현/실행 귀속을 새로 조사한다.

---

## 3-3. MB-03 — LLM 후보 평가와 기본 모델 선정 (검증 완료: 2026-10-22)

### 누가 무엇을 했는가 — Git 커밋 및 리뷰 귀속

| 근거 PR | 커밋 작성자·내용 | 개인 귀속 판단 |
| --- | --- | --- |
| [#19 초기 모델 평가 하네스](https://github.com/aihuman-7th/proj2-3/pull/19) | **SJeongHyeon**: 기존 `run_model_comparison` 기반·실험 평가 하네스 | **팀원 구현**, 내 단독 제작으로 주장 금지 |
| [#36 다양성 평가 core](https://github.com/aihuman-7th/proj2-3/pull/36) | **nanocode00**: `evals/openrouter_diversity.py`, 관련 테스트 | **내 작업 흐름 귀속 확인** |
| [#37 다양성 평가 CLI/Spec](https://github.com/aihuman-7th/proj2-3/pull/37) | **nanocode00**: `run_openrouter_diversity.py`, AC·재현 문서 | **내 작업 흐름 귀속 확인** |
| [#40 Final Four 재현](https://github.com/aihuman-7th/proj2-3/pull/40) | **nanocode00**: 4종 runner, blind 자료 생성·집계 CLI | **내 작업 흐름 귀속 확인** |
| [#41 평가 이력·선정 근거](https://github.com/aihuman-7th/proj2-3/pull/41) | **nanocode00**: `EVAL_REPORT`, `DECISIONS`, `CURRENT_STATE` 정합화 | **내 판단·재현 기록 정리** |
| [#43 모델 채팅 probe](https://github.com/aihuman-7th/proj2-3/pull/43), [#45 확장 평가](https://github.com/aihuman-7th/proj2-3/pull/45) | **SJeongHyeon**: 별도 모델 probe·평가 확장 | **팀원 기여**, 구분 |
| [#47 두 후보 코칭 서비스 통합](https://github.com/aihuman-7th/proj2-3/pull/47) | **nanocode00**: 서비스 하네스, 모델별 API 경로 테스트, OpenRouter 호환성 보완·결과 비교·기본 환경 설정 | **내 작업 흐름 귀속 확인**, 코드/실험에는 Codex·Antigravity 보조를 사용 |

**커밋 계정의 의미:** #36/#37/#40/#41/#47이 사용자 명의이며, 문서상 9/21 다양성 및 1차 서비스 실험의 작업자는 Codex, 같은 날 Docker·로컬 Supabase 실험은 Antigravity다. 따라서 **AI 코딩 도구를 사용해 평가 조건·실험·수정·PR을 주도적으로 수행한 경험**으로 기술한다. 본인이 모든 라인을 수작업으로 작성했다고 주장하지 않는다.

### 5단계 경험 서사 — 상황 → 문제 → 판단 → 조치 → 결과

1. **상황:** 말해봄의 초기 평가 기반과 여러 모델 실험은 존재했지만, 접근 가능한 외부 모델을 폭넓게 비교하고 실서비스 기본 모델을 선택하는 **연결된 재현 절차**는 부족했다. 기존 코칭 계약·평가셋은 서정현 등 팀원이 구축한 토대였다.
2. **문제:** OpenRouter에서는 모델별 실제 호출 가능 여부와 structured output 호환성, 지연·비용이 달랐다. 30건 독립 평가의 성공이 FastAPI·MCP·DB 경로의 성공을 보장하지 않았고, 두 서비스 실험의 서로 다른 결과와 `.env.example`의 불일치는 기본 모델 결정을 왜곡할 수 있었다.
3. **판단:** 단일 품질 점수나 최저 호출비용 대신 **사전 통과 게이트 → 동일 조건 계약·지연·비용 재측정 → 실제 코칭 경로 검증**으로 좁혀야 한다. 계약 준수와 의미 품질, 비용 확인값과 실제 총액, 독립 하네스와 Docker 통합 실행의 표본은 분리해 판단한다.
4. **조치:** Codex·Antigravity를 활용한 내 PR [#36](https://github.com/aihuman-7th/proj2-3/pull/36)·[#37](https://github.com/aihuman-7th/proj2-3/pull/37)·[#40](https://github.com/aihuman-7th/proj2-3/pull/40)·[#41](https://github.com/aihuman-7th/proj2-3/pull/41)·[#47](https://github.com/aihuman-7th/proj2-3/pull/47)로 **40모델 사전검증·22모델 660건 본평가**, Final Four **120건 별도 재실행**, 후보 2종 서비스 경로 **정상/경계/실패 유도 각 3건** 확인을 수행했다. `temperature=0` 파라미터 충돌에 따른 OpenRouter 404를 분석해 model-default로 맞추고, 제한된 JSON code fence만 보정했다. 팀원 리뷰에 따라 실행별 결과와 기본 환경 설정도 정합화했다.
5. **결과:** Final Four에서 Flash Lite는 **30/30 계약 준수·p50 3.41초**, Luna는 **29/30·p50 9.36초**. Docker·로컬 Supabase·MCP/DB 통합 경로에서 Flash Lite **3/3·p50 3.44초**, Luna **초기 1/3·code fence 보정 후 2/3**을 확인해 **Flash Lite를 기본 후보로 선정하고 `.env.example`·README에 반영**했다. 이는 합성·로컬 표본의 안정성 근거이지, 최종 운영 사용자 의미 품질 100% 검증은 아니다.

### 실험·결정의 상세 근거

1. **문제:** 모델 선택을 출력 계약 하나나 정성적 선호만으로 결정할 수 없고, OpenRouter 모델의 실제 접근 가능성·스키마 호환성·비용·지연·서비스 통합 안정성이 서로 다르다. 과거 6종 평가·Basic/Premium 실패 복구 실험은 **다른 목적과 조건**이므로 직접 한 표로 합치지 않는다.
2. **실험 설계:** 기존 Dataset v2 합성 30건·Prompt v3·고정 Skill/criteria·strict JSON Schema/서버 validator를 재사용. 인증된 모델 목록에서 text→text 및 `response_format` 지원 모델을 추리고 developer 다양성에 따라 선정한 뒤, 정상/경계/실패 유도 **3건 preflight → PASS 모델만 30건 본평가**를 진행. 재시도 상한·atomic JSON 저장·중단 후 재개·모델 기본 생성 설정과 `--live` 없는 dry-run을 둠.
3. **OpenRouter diversity 결과:** 283개 발견, 214개 적격. 1차 **30모델 preflight → 15 PASS → 450건**, 이어서 신규 10모델 preflight → 7 PASS → 210건. 누적 **40개 preflight / 22개 PASS / 22×30=660건 본평가**, 알려진 provider 비용 합계 **$10.2759357145**, 비용 없는 호출 **80개**. 따라서 $10.28은 *관측된 비용의 합*이지 청구 최종 총액이 아니다. 사람 semantic 평가는 **미수행(null)**.
4. **Final Four 별도 재측정(2026-09-21):** 같은 시점 고정 Dataset·Prompt·Schema·model-default 생성 설정으로 4×30=**120건을 새로 실행**. Flash Lite **30/30**, p50 **3.41s**, $0.10706534; Gemini 3.8 Flash **30/30**, **12.60s**, $0.34932900; Luna **29/30**, **9.36s**, $0.03914948; Terra **10/30**, **1.18s**, 비용 **≥$0.10570740**. Terra 20건은 provider/schema/integration 단계 실패로 raw/usage가 없어 의미상 실패나 알려진 비용 0으로 해석 금지. 1차 후보는 Flash Lite와 Luna. 사람 blind semantic 검토는 준비만 했고 이 단계에서 완료하지 않음.
5. **실서비스 호출 경로 분리 검증:** 독립 FastAPI harness에서는 정상·경계·실패 유도 각 1건에서 두 모델 모두 3/3 계약 통과(Flash Lite p50 3.37s, Luna 13.41s). 다음 **Docker+로컬 Supabase·stdio MCP·DB 경로**에서 Flash Lite **3/3**, p50 **3.44s**, 알려진 비용 $0.01132, 무재시도; Luna는 최초 **1/3**, JSON code fence 보수적 unwrap 후 **2/3**. 실제 사용자 테스트 계정으로 WAV/STT→answer revision→코칭→`coaching_runs` 확인도 기록되어 있으나 **표본 3건 모델 비교는 합성 고정 케이스**다.
6. **통합 오류 분석과 결정:** 최초 독립 하네스의 `temperature=0`/OpenRouter `require_parameters=true` 조합에서 두 모델 HTTP 404가 나와, 서비스 호출도 `temperature`를 생략해 model-default로 맞춤. Luna의 코드펜스 JSON은 보수적 파서로 일부 복구했지만 스키마 필드 누락 사례는 여전히 실패. Flash Lite는 비교 시점 **속도·계약 준수·실제 API 통과**가 안정적이므로 기본 코칭 모델로 결정하고 `.env.example`·README에 반영함.
7. **동료 리뷰로 결과/설정 정정:** [PR #47 리뷰](https://github.com/aihuman-7th/proj2-3/pull/47)에서 **서정현이 독립 harness Luna 3/3과 Docker E2E Luna 2/3을 섞어 기술한 불일치**를 지적해 날짜·버전별 두 실험을 별도 기록하도록 수정 요청. 이어 `.env.example`의 기본값이 여전히 다른 모델이어서 선정 문구와 실제 실행 설정이 어긋난다는 문제를 지적했고, 기본값·README 변경 후 승인. **팀원 지적을 반영해 재현성과 배포 구성 일치성을 확보한 협업 경험**으로 기술하는 것이 맞다.

### 핵심 기술 해석과 주장 경계

- 실험 단위: 40모델 preflight, 660건 diversity, 별도 120건 Final Four, 3×2건 서비스 smoke는 **독립 목적과 일부 중복/재실행**이므로 780건을 단일 동일조건 실험으로 표현하지 않는다.
- **계약 준수 30/30 ≠ 사용자 코칭 의미 품질 100%**. Final Four blind semantic 결과 미확정, 실제 3~5명 사용자 효과 미검증, p95(3건)는 통계적으로 불안정.
- `COACHING_LLM_MODEL` 기본값을 후보로 선택·설정한 것과 이후 **운영 배포의 전체 사용자 성능**은 다르다. 코칭 모델의 최종 운영 품질 승격까지 이 작업군 성과로 확대하지 않는다.
- 다양성 단계의 최종 **$10.2759는 비용 미확인 80호출**이 있어 하한 성격. Final Four Terra 비용도 실패 호출 사용량 누락으로 하한이다.
- **채용 활용 등급: 주력 사례.** 여러 모델의 이름이 아니라 **후보 탐색→사전 검증→조건 고정 재측정→실제 API 검증→설정 충돌 해결→채택**이라는 의사결정·시스템 평가 설계 능력, 자동화/재현성 확보를 강조한다.

**검증 판정:** MB-03은 **개인 수행·의사결정·코딩 에이전트 활용·PR 책임 A**, 초기 평가 기반·후속 probe는 **팀 작업 T**. 다음 MB-04는 이미 사람 ground truth 원본·재채점 증거가 확보돼 있으나 PR 단위 작성·사람 검토 귀속까지 별도 심화한다.

참고: [모델 다양성](https://github.com/aihuman-7th/proj2-3/blob/main/docs/experiments/2026-09-21_openrouter-model-diversity.md) · [Final Four](https://github.com/aihuman-7th/proj2-3/blob/main/docs/experiments/2026-09-21_openrouter-final-four.md) · [서비스 후보 비교](https://github.com/aihuman-7th/proj2-3/blob/main/docs/experiments/2026-09-21_service-candidate-integration.md) · [로컬 E2E Worklog](https://github.com/aihuman-7th/proj2-3/blob/main/docs/worklogs/2026-09-21_2315_local-supabase-candidate-integration-e2e.md) · [평가 총괄](https://github.com/aihuman-7th/proj2-3/blob/main/EVAL_REPORT.md).

---

## 4. 확정된 1차 사실

### MB-04 — 평가 정답 검토

- 9/30 김재훈이 30개 expected label·근거 모두 검토, 29개 유지, boundary-17의 결과 검증을 `partial → undetermined`.
- 동일 저장 출력 90개를 offline 재채점: contract 90/90 유지; 전체 label 316/360 → 313/360(86.9%), V 77/90 → 74/90(82.2%). 새 모델 호출이 아니라 정답 수정 효과.
- 단일 검토자·inter-rater 없음.
- [사람 검토표](https://github.com/aihuman-7th/proj2-3/blob/main/evals/reviewer-assignment.md) · [재채점](https://github.com/aihuman-7th/proj2-3/blob/main/docs/experiments/2026-09-30_d4-boundary17-offline-regrade.md).

### MB-06 — 공고 원문 분석 E2E

- 10/1 Worklog 작성자 김재훈/ChatGPT. JWT 0600 파일 전달 지원과 보안 테스트 3개 추가.
- API 동일 순서 실행: 추천 3개, 요구사항 5개, quote validation PASS, 저장 확인, 요구사항 2개 선택, session 생성.
- PlayMCP 인증 503 시 다른 정상 지원 provider 경로로 실행; 배포 환경·브라우저 클릭 E2E는 미검증.
- [작업기록](https://github.com/aihuman-7th/proj2-3/blob/main/docs/worklogs/2026-10-01_1400_job-source-authenticated-e2e.md).

## 5. 실험 수치 — 조건을 합치지 말 것

| 날짜/목적 | 핵심 수치 | 해석 경계 |
| --- | --- | --- |
| 9/21 Final Four | Flash Lite 계약 30/30, p50 3.41s, 30건 $0.10706534; Luna 29/30, p50 9.36s, $0.03914948 | 합성 30건, 모델 기본 생성 설정·OpenRouter 경로; 사람 semantic 우위 아님 |
| 9/21 Docker/Supabase 후보 서비스 경로 | Flash Lite 3/3, Luna 최초 1/3→파서 보정 후 2/3 | 대표 3건, 실제 회원의 브라우저 녹음 E2E와 다름 |
| 9/30 ground truth 수정 | offline label 86.9%, V 82.2%; contract 90/90 동일 | 저장 출력 재채점, 새 추론 없음 |
| 10/2 동일 Skill Prompt v4/v5 | 계약 각 30/30, 종합 자동 통과 22/30→20/30, 비용 $0.19638110→$0.17027067 | v5 비용 절감에도 판정 회귀, 출시 승인 아님 |
| 10/6 배포 이미지 반복 provider 평가 | 계약 90/90, 자동 통과 57/90 | 30×3 합성 모델 평가; 실제 사용자 전체 E2E 아님 |
| STT C2 기술 질문 8건 | 발화형 정규화 WER 20.11%→11.73% | 부분집합, B0 vs C2; 기존 도메인 retention 악화·운영 교체 미확정 |

## 6. 남은 검증과 다음 진행 지점

1. **저장소 PR 작성자와 Git diff를 작업군별로 역추적.** 우선 모델 평가·검증 관련 PR, 9/30 사람 평가 반영 PR, 10/1 인증 E2E, STT 학습/변환 기록. AI 사용에 의한 커밋 author와 실제 의사결정·운영자 역할은 구분.
2. **MB-03/05/08을 개인 기여로 확정할 근거 수집.** 코드만 존재하는 부분은 팀 결과에 둔다.
3. **MB-09~12의 누락 역검증.** 사용자의 UI 제안·평가/검증·PR 운영과 팀원 프론트 구현·운영 배포를 혼동하지 않는다.
4. **사례 선별.** 모델 평가/선정, 신뢰 계약·검증, 사람 ground truth, STT 실험, 인증 E2E 중 직무별 상위 3~5개 선택.
5. **전면 근거 검증 뒤** `experiences/malhaebom.md` 채용 요약을 확정하고 `experiences/README.md`, `career-timeline.md`를 업데이트. 마스터 `resume.md`는 나머지 프로젝트 점검 뒤 변경.

### 보수적으로 유지할 미확인 사항

- UI 전체, Agent 전체, RAG/LangGraph(사용하지 않음), 모든 모델 파인튜닝을 개인 단독 구현한 것으로 서술 금지.
- 계약 100%와 코칭 의미 품질 100%, 실제 사용자 만족도, 운영 전체 E2E 성공을 동의어로 사용 금지.
- C2를 무조건 기존 모든 음성에 우수하거나 production STT로 승격했다고 쓰지 않음.
- 새 프롬프트 v5의 형식 30/30을 품질 개선 성공으로 표현 금지.
- 프로젝트 공식 마감(10/8) 후 PR 병합/소스 동기화를 원래 프로젝트 기간의 새 기능 개발로 중복 계상하지 않음.
