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
| MB-04 | 합성 평가셋 30건의 정답 사람 검토·정정 | **PR #126**·dataset v2·reviewer-assignment·9/30 offline regrade | **A(직접 검토·정정·재채점 확인)** | **검증 완료** — 30건 사람 검토와 저장 출력 90건 재채점 분리, PR #125 폐기·#123 revert 기록 확인 |
| MB-05 | Prompt/Skill A/B 및 회귀 대응 | PR #155 리뷰, #185(공동 PR의 본인 작성 1·5~9), #189, #215; 10/2 30건·10/6 Skill smoke·10/8 D11 3건 | **A(단일 Skill A/B·검증 기준·결과 기록·추가 실험 정리) / R(10/2 팀원 A/B 리뷰)** | **검증 완료** — 변경 효과보다 회귀·미검증 범위의 정확한 판정 중점 |
| MB-06 | 로그인 사용자 공고 원문 분석 E2E | **개인 PR #149**, 로컬 redacted 증거·10/1 Worklog·팀 리뷰; 별도 팀원 PR #167·#186 | **A(인증 API E2E 도구·실측·장애 분리) / T(운영 로그 문서화·Windows 보강)** | **검증 완료** — PlayMCP 직접/브라우저 UI/운영 동일 요청 입증은 미확인 |
| MB-07 | FastMCP stdio/HTTP, /api/agent, DB 저장 | **개인 PR #112·#114·#115·#118 / 개인 리뷰 #122**, 팀원 PR #39·#116·#117·#172 | **A(통합 테스트·인증 사용자 검증·저장 거짓 양성 리뷰) / T(핵심 Agent·MCP 코어)** | **검증 완료** — fixture·로컬 JWT live·Cloud trace·DB 행 검증·운영 배포의 수준 구분 |
| MB-08 | Whisper Medium 50k → B0 → C2 학습 | **개인 PR #23~34·#51~56·#69~73·#146**, 50k/clean/71592/실사용 모델×beam 실험·리뷰 | **A(개인 주도 학습·데이터/평가 파이프라인·상대적 모델 선택)** | **검증 완료** — legacy adapter 단절, sealed test, retention 회귀, 단일 화자, 운영 기본 미교체 |
| MB-09 | **WAV 업로드·STT/VAD·전사·코칭·비교 로컬 E2E 및 분석 지연** | 개인 PR #47·#104·#106·#115; 9/21·9/28·9/29 Worklog | **A(서비스 경로 검증·벤치마크·회귀 판정) / T(제품 핵심 STT·UI 기능)** | **검증 완료** — 마이크 실녹음·Hosted Supabase 운영 전체 E2E 미확인, MB-03/07 결과 중복 집계 금지 |
| MB-10 | Langfuse Prompt·Skill 실행 버전·Trace·운영 관측 | **개인 PR #101·#114·#162·#216 / 개인 리뷰 #221**, 9/28~10/9 실측·worklog | **A(실험 export·버전 정합성·운영 trace 검증) / R(#221 요청-ID 연계 리뷰)** | **검증 완료** — 10/8 관리형 Prompt @4 실행 연결, 10/9 운영 HTTP ID 연결은 별도 실행; release/rollback 미검증 |
| MB-11 | UI/UX 개선 검토·정적 쇼케이스 제작·소개 통합 | **개인 PR #214, 개인 리뷰 #211, 개인 showcase 저장소**; 팀원 #240·#241·#251; 10/7~10/10 worklogs | **A(정적 소개 구현·콘텐츠/에셋 검증) / R(UI 코드 결함 발견·승인) / T(앱 통합·후속 운영)** | **검증 완료** — 목업/실제 UI/별도 Pages/운영 앱 통합 및 10/8 마감 전후를 분리 |
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

## 3-4. MB-04 — 합성 30건 사람 검토·Ground Truth 교정 (검증 완료: 2026-10-22)

### 5단계 경험 서사 — 상황 → 문제 → 판단 → 조치 → 결과

1. **상황:** 말해봄의 `evals/dataset.v2.jsonl`에는 정상 12·경계 10·실패 유도 8의 **합성 평가 30건**이 있었지만, D4 체크포인트의 필수 사항인 **사례별 실제 사람 검토자와 최종 판정 근거**가 확정되지 않아 provenance가 `independent_review_pending`이었다. 과거 모델 출력 품질 평가와 **정답표 검토**는 목적이 다르다.
2. **문제:** `coach-boundary-17`의 답변은 “첫 프로젝트에서는 캐시, 두 번째 프로젝트에서는 로그, 인덱스도 추가, 결과적으로 성능 개선”처럼 **프로젝트·조치와 결과가 연결되지 않는다**. 결과가 언급됐다는 이유만으로 V(`result_verification`)의 기대값을 `partial`로 두면 모델이 **귀속 불명확성을 판단하지 못해도 정답**으로 계산될 수 있다. 같은 상위 V 상태라도 '검사 행위만 있음'과 '결과 주장만 있음'은 오류 원인이 다르다.
3. **판단:** **결과가 전혀 언급되지 않음(`not_observed`)**과 **결과 정보는 있으나 어느 행동의 결과인지 확인 불가(`undetermined`)**를 분리해야 한다. 사람 검토로 수정한 **Ground Truth만 적용해 기존 출력 재채점**해야 새 추론이나 Prompt 회귀와 섞이지 않는다. V 보조지표는 **사람 검토 메타데이터**일 뿐 기존 모델의 4상태 계약·자동 품질 점수에 추가하지 않는다.
4. **조치:** 김재훈이 2026-09-30 **30건 전체의 원문·C/A/V expected 및 판정 근거를 직접 검토**, 29건을 유지하고 `coach-boundary-17`의 V를 **`partial → undetermined`**로 확정했다. `evals/reviewer-assignment.md`에 사례별 판정자·근거·상태를 기록하고 V 원인을 `verified / verification_only / result_only / omitted / explicitly_unverified / ambiguous_target / conflicting_results` **7종**으로 분류했다. Dataset·기대 행렬·provenance를 고치고, **9/22 Gemini Flash-Lite / Prompt v4.1 / 30건×3회 = 90개 최종 저장 출력**을 **provider 재호출 없이** offline 재채점했다.
5. **결과:** 세 번의 case17 출력 V가 모두 `partial`이었으므로 새 정답 적용 후 전체 label 일치가 **316/360(87.8%) → 313/360(86.9%)**, V가 **77/90(85.6%) → 74/90(82.2%)**로 정정됐다. **최종 계약 통과 90/90, evidence accuracy 96%, ID reference 100%, 비용 $0.292, 원본 호출 총 96건**은 기존 측정 그대로다. **실험 결과가 새로 악화된 것이 아니라 정확한 정답표로 동일 출력을 다시 평가한 결과**다.

### 기여·PR·검증 경계

- [PR #126 — D4 human review split](https://github.com/aihuman-7th/proj2-3/pull/126)은 **JaeHun Kim / nanocode00** 명의 2개 커밋으로 Dataset 30건 최종 검토 결과 및 `EVAL_REPORT` offline regrade를 연결했고 **MERGED**. 동료 `SJeongHyeon`이 case17 데이터셋·테스트 기대 행렬·재채점 보고서를 대조하고 **D4 관련 8개 테스트, diff check, PR Policy·Secret Scan 통과**를 확인해 승인했다. 이 PR은 개인 작업 귀속의 중심 증거다.
- [PR #123 체크포인트](https://github.com/aihuman-7th/proj2-3/pull/123)에 D4 검토/expected 정정 작업이 일시 포함됐으나 최종적으로 **두 커밋을 revert**해 체크포인트만 분리·병합했다. 원래 [PR #125](https://github.com/aihuman-7th/proj2-3/pull/125)는 **CLOSED, unmerged**. **최종 유효 기여는 #126**을 기준으로 추적해야 한다. 중복 커밋을 독립 성과로 계산하지 않는다.
- 실제 사람 검토 **30개 사례 ≠ 90개 모델 출력 전수 사람 검토**. 90개는 저장된 모델의 출력이고 자동 스코어러로 다시 채점했다. 96 provider calls는 90 최종 execution 및 contract repair를 포함한 **과거 실행 기록**이며 이번 교정에서 새 과금/API 호출은 하지 않았다.
- **단일 검토자**: 다른 검토자와 합의하거나 inter-rater agreement를 측정한 것처럼 쓰지 않는다. Ground Truth의 1개 수정은 모든 평가 정책을 독립 검증했다는 의미도 아니고, 보조 7분류는 별도 모델 예측 지표가 아니다.
- **이력서 가치: 주력 보완 사례.** '점수를 개선했다'가 아니라 **정답표 오류를 검출하고, 인과 귀속과 관찰/미관찰을 구분하며, 결과 재측정에서 입력·모델 호출 통제를 지켰다**는 신뢰성 있는 평가 엔지니어링 판단 사례로 제시한다.

원본: [D4 사람 검토표](https://github.com/aihuman-7th/proj2-3/blob/main/evals/reviewer-assignment.md) · [D4 Worklog](https://github.com/aihuman-7th/proj2-3/blob/main/docs/worklogs/2026-09-30_d4-human-review.md) · [offline 재채점](https://github.com/aihuman-7th/proj2-3/blob/main/docs/experiments/2026-09-30_d4-boundary17-offline-regrade.md) · [EVAL_REPORT §3-2](https://github.com/aihuman-7th/proj2-3/blob/main/EVAL_REPORT.md).

---

## 3-5. MB-05 — Prompt/Skill A/B·회귀 판정 및 후보 승격 보류 (검증 완료: 2026-10-22)

### 5단계 경험 서사 — 상황 → 문제 → 판단 → 조치 → 결과

1. **상황:** 말해봄은 구조화 코칭 출력의 계약 준수·의미 판정·인용 근거·비용·지연을 모두 관리해야 했다. 영문 Prompt v4→v5, 단일 Skill 규칙, D11 인젝션 방어는 **서로 다른 실험**이며, Prompt·Skill 변경 및 기준 검증에는 팀원과 AI 코딩 도구의 작업도 포함돼 있었다.
2. **문제:** 모델 출력이 JSON·Pydantic 계약을 통과하더라도 **개인 행동, 결과 확인, 질문 적합성에 대한 의미 판정**이 회귀할 수 있었다. 일부 사례 개선이나 비용 절감만 보면 부작용을 놓친다. 또한 단일 Skill 규칙 A/B의 최초 자동 게이트는 목표 `problem_context`만 확인해 `own_action`·`result_verification` 개선을 놓칠 수 있었고, 합산점수만 비교하면 한 기준의 회귀를 다른 기준 개선이 가릴 수 있었다.
3. **판단:** 변경을 채택하려면 **동일 모델·Skill·Dataset·schema·생성 설정에서 비교**하고, 결과 계약·종합 자동 통과·기준별 C/A/V와 인용·비용·지연을 함께 확인해야 한다. 단일 Skill 규칙은 **최소 한 target criterion 개선 + 각각의 target criterion 무회귀**를 자동 KEEP 후보 최소조건으로 두고, 사람 검토까지 통과해야 최종 KEEP한다. 값이 나빠지면 확대 실험을 조기 중단하며, 재시도 횟수/설정이 다른 인젝션 실험을 공정한 방어 성공률로 계산하지 않는다.
4. **조치:** 
   - **10/2 동일 Skill 30건 Prompt 비교:** `coaching-v5-skill-owned`과 이전 영문 v4를 비교하는 코드·실험 보고서는 **곽호준 작성 [PR #155](https://github.com/aihuman-7th/proj2-3/pull/155)**. 김재훈은 [직접 PR 리뷰](https://github.com/aihuman-7th/proj2-3/pull/155)에서 조건·자동 회귀·미완료 사람/운영 검증 표기를 확인하고 승인했다. **원시 결과는 당시 검토 환경에서 독립 재계산하지 못했음**을 리뷰에 남겼다. 이를 김재훈 단독 60콜 실행으로 귀속하지 않는다.
   - **10/6 단일 Skill 규칙 라운드:** **[PR #185](https://github.com/aihuman-7th/proj2-3/pull/185)**의 9개 커밋 중 **김재훈이 초기 단일 Skill 규칙 A/B runner/테스트 준비(1번), target C/A/V 전체 집계(5번), criterion별 무회귀 게이트(6번), 최신 main 정합화(7~9번)**를 작성했고 **2~4번은 서정현 작성**. 서정현의 수정 요청으로 목표를 1개 기준에서 전체 C/A/V로 확대하고, 평균 개선에 가려진 개별 회귀를 차단했다. **[PR #189](https://github.com/aihuman-7th/proj2-3/pull/189)**는 **김재훈 단독 커밋**으로 실제 `coach-boundary-19` baseline/candidate 각 1회 smoke 결과를 기록했다.
   - **10/8 신규 D11 인젝션 3건:** [PR #215](https://github.com/aihuman-7th/proj2-3/pull/215)는 **김재훈 계정**에서 가변 재시도·오류를 포함한 Prompt v4/v5 실험 기록과 `run_d11_v5_extra_once.py`를 보존했다. 이전 실패 02·03을 각각 **추가 1회만** 호출하고 기존 raw를 덮어쓰지 않았다. 공식 독립 평가자 없이 기존 **단일 검토자 승인 정답**을 사용한 점을 유지한다.
5. **결과:**
   - **10/2 30건×2:** 계약 양측 **30/30**이나 종합 자동 통과 **22/30→20/30**, intent/C/A/V 판정 **116/120→107/120**, p95 **4.621→4.622초**, 알려진 비용 **$0.19638110→$0.17027067**. 약 **13.3% 비용 감소에도 회귀**가 발생하므로 v5의 **품질 개선 승격을 DISCARD/보류**로 기록. **그 시점 운영 기본 변경·롤백 조치는 아님.** 작성/호출은 PR #155 팀원 담당, 개인은 해당 결과와 근거를 검토한 역할.
   - **10/6 Skill 1문장 후보 smoke:** 계약 양쪽 **1/1**, target criterion 일치 **2/3→1/3**, label accuracy **75%→50%**, 기록 비용 **$0.0120988**. 목표인 `problem_context`은 그대로 불일치했고 오히려 직접 행동 `own_action`이 `met→partial`로 후퇴. 자동 게이트 **`DISCARD_CANDIDATE`**가 작동해 계획한 **전체 30건×2회 확대 실행은 하지 않음**. 사람 검토 후 최종 폐기 확정은 별도 단계로 남음.
   - **10/8 D11 3건 인젝션:** 초기/1회 보정 시점은 v4 계약·기대판정 **3/3**, v5 **1/3**; 02·03을 각각 세 번째로 추가 호출해 **사례별 마지막 유효 시도 기준** v5 계약 **3/3**, 기대 intent/C/A/V 전체 일치 **2/3**. v5 02는 `own_action=met`으로 사람 기대 `partial`과 다름. **최초 성공률과 최종 유효 응답률을 섞지 않으며**, 재시도 횟수·timeout이 달라 독립 사람 확인/공식 Dataset 승격/방어 효과 확정/운영 배포 검증을 주장하지 않는다.

### 기여 귀속·재현성 및 미확인 범위

- **확인된 개인 코드·실험 책임:** 단일 규칙 A/B gate 및 검증 기준 강화([#185](https://github.com/aihuman-7th/proj2-3/pull/185)), 목표 회귀 실측·확대 실행 보류([#189](https://github.com/aihuman-7th/proj2-3/pull/189)), D11 신규 3건 추가 호출 결과/재현 runner 문서화([#215](https://github.com/aihuman-7th/proj2-3/pull/215)).
- **팀원 귀속:** 10/2 60호출 영문 v4/v5 측정 구현/기록 [#155](https://github.com/aihuman-7th/proj2-3/pull/155) **곽호준**; [#156](https://github.com/aihuman-7th/proj2-3/pull/156) Gemini structured schema·평가 후속 수정, [#158](https://github.com/aihuman-7th/proj2-3/pull/158) JSON 절단 관측, [#160](https://github.com/aihuman-7th/proj2-3/pull/160) 4096-token 기록, [#161](https://github.com/aihuman-7th/proj2-3/pull/161) 사람 리뷰 pack 준비는 **서정현** 작성. 김재훈은 이 PR들을 **구현 수준·CI·판정 한계에 따라 리뷰/승인**했으나 코드를 단독 구현했다고 주장하지 않는다.
- **다른 조건 분리:** 10/2 페어 비교 **max 2048·timeout 45s·계약 30/30**, 별도 10/2 v5 2048·60s **29/30**, 별도 v5 4096·60s **30/30**은 다른 실행. 10/6 당시 배포 이미지 Prompt v4·Skill v6 기반 **30×3·90/90 계약·57/90 자동 통과** 역시 10/2 Prompt v4/v5 교체 효과 측정이 아님.
- **채용 활용 판단:** **개인 수행 A — 사전 정의한 합격 조건이 회귀를 가리는 문제를 발견/수정하고 목표 smoke의 퇴행을 확인해 추가 실험 비용을 통제**한 사례가 가장 강함. 10/2 v5 회귀 결과는 팀 작업을 검토한 사례로 분리. 신규 인젝션 3건은 독립 사람 검토 미완료로 잠정 증거에만 사용하며 '방어 개선 성공' 금지.

원본: [10/2 Prompt 비교](https://github.com/aihuman-7th/proj2-3/blob/main/docs/experiments/2026-10-02_latest-english-v5-comparison.md) · [10/6 단일 규칙 smoke](https://github.com/aihuman-7th/proj2-3/blob/main/docs/worklogs/2026-10-06_single-skill-rule-offtopic-cav.md) · [10/8 D11 A/B](https://github.com/aihuman-7th/proj2-3/blob/main/docs/experiments/2026-10-08_d11-injection-prompt-ab-provisional.md) · [10/8 추가 재시도](https://github.com/aihuman-7th/proj2-3/blob/main/docs/worklogs/2026-10-08_d11-v5-extra-retries.md) · [EVAL_REPORT](https://github.com/aihuman-7th/proj2-3/blob/main/EVAL_REPORT.md).

---

## 3-6. MB-06 — 로그인 사용자 공고 원문 분석 인증 API E2E (검증 완료: 2026-10-22)

### 5단계 경험 서사 — 상황 → 문제 → 판단 → 조치 → 결과

1. **상황:** 공고 원문 조회·LLM 요구사항 추출·서버 quote 검증·사용자별 공고 저장 기능과 개별 테스트는 존재했지만, 강사 피드백의 **공고 선택 → 원문 조회 → 요구사항 추출 → 원문 인용 검증 → 요구사항 선택**을 *실제 로그인 사용자 권한*으로 연속 실행한 근거가 없었다. 비대화형 실행기 초안은 Supabase access JWT 전달 방법도 없어 실행을 완료하지 못했다.
2. **문제:** 로컬 시험 도중 **초기 외부 NCS API 503**, 이후 **PlayMCP 인증 503**, API 재생성 뒤 **기존 사용자 JWT 401**이 연달아 발생했다. 각각의 실패 지점을 분리하지 않으면 공고 원문 분석 자체의 결함인지, 외부 검색 provider 접근 문제인지, 로그인 토큰 수명 문제인지 구별하기 어렵다. 또한 JWT·외부 공고 원문·사용자별 UUID를 증거에 노출할 위험이 있었다.
3. **판단:** 전반적인 제품 기능 성공과 **PlayMCP 자체의 인증 성공은 다른 검증 조건**이다. 서버가 이미 지원하는 `JOB_SEARCH_PROVIDER=web` 복귀 경로를 사용하면 PlayMCP 장애 시에도 **인증 → 추천 → 원문 검증 → 요구사항 저장 → 세션 생성**이라는 핵심 기능을 검증할 수 있다. 실제 브라우저 로그인 JWT로 **프런트와 동일한 API 호출 순서**를 검증하되, 브라우저 클릭 UI/운영 PlayMCP E2E와 혼동하지 않는다. 비밀 토큰은 Git ignored 파일로 보관하고 그룹·다른 사용자 읽기 권한을 거부한다.
4. **조치:** **[PR #149](https://github.com/aihuman-7th/proj2-3/pull/149)**의 `backend/tests/run_job_source_e2e.py`를 포함한 **JaeHun Kim/nanocode00 단일 작성 커밋**에서 비대화형 JWT 전달을 추가하고 환경변수 우선·POSIX `chmod 600` 파일 허용·`0644` 거부 **3개 단위 테스트**를 작성했다. 실제 로그인 세션의 JWT로 `http://localhost:8000` 로컬 Compose API에 NCS → 공고 추천 → 사람인 공고 선택 → 원문 재검증 및 LLM 요구사항 추출 → 서버 source quote validation → `private_job` 저장/사용자 목록 조회 → 요구사항 2개 선택 → `preset-04` 연습 세션 생성 순서로 실행했다. NCS 장애는 재시도, PlayMCP 장애는 **기존 web provider fallback**, 재시작 후 401은 **갱신된 로그인 JWT**로 해결했다. 저장 증거는 JWT·원문·공고 전체 URL·사용자 ID·저장 UUID 없이 redacted 집계만 보존했다.
5. **결과:** `auth=ok`, `ncs_option=skill-server`, `recommendation_count=3`, `source_review=ok`, `requirement_count=5`, `quote_validation_gate=passed`, `authenticated_job_list_contains_selected=true`, `selected_requirement_count=2`, `practice_session_created=true`를 **1회 로컬 인증 API E2E 성공 run**으로 기록했다. [PR #149](https://github.com/aihuman-7th/proj2-3/pull/149)는 팀원 **SJeongHyeon 승인**, diff check·CI 4개 통과 후 병합됐다. **당시 PlayMCP 직접 호출은 인증 503으로 실패**했고 성공 run은 web provider였다. **실제 운영 배포/브라우저 UI 클릭 E2E/PlayMCP 전체 경로를 해당 실험으로 입증하지 않았다.**

### 후속 근거·개인 귀속 경계

- **개인 주력 기여:** 인증 API E2E runner 및 비밀 토큰 권한 경계, 실패를 provider·JWT·기능 동작으로 나눠 분석하고 기존 fallback으로 기능적 증거를 완성한 판단, 기밀을 제외한 재현 가능 결과 정리. 주 작성 커밋·리뷰 증거는 **[#149](https://github.com/aihuman-7th/proj2-3/pull/149)**.
- **팀원 후속 작업:** **곽호준 작성 [PR #167](https://github.com/aihuman-7th/proj2-3/pull/167)**은 저장된 운영 HTTP 로그(추천 200, 분석 201, 세션 201 등)를 별도 정리하고 기존 로컬 원본을 보존했다. 새로운 운영 사용자 E2E를 실행한 것이 아니라는 한계를 기록했다. **서정현 작성 [PR #186](https://github.com/aihuman-7th/proj2-3/pull/186)**은 Windows에서 POSIX 파일 권한에 의존한 JWT 파일 fallback을 막고 환경변수를 사용하도록 보완했다. 김재훈은 코드·테스트·CI를 확인하고 승인했지만 Windows 보완 코드는 팀원 귀속이다.
- **후속 사용자 수동 보고의 별도 성격:** 2026-10-02 실제 운영 브라우저에서 로그인 후 공고 검색·원문 분석·1/2차 비교가 성공했다는 사용자 확인 기록은 있다. 다만 당시 동일 요청의 **PlayMCP 도구 호출 로그→추천→원문 분석**을 이어주는 request ID/trace와 회복 원인은 확보되지 않아 10/1 로컬 **web fallback API E2E의 성공 조건에 합산하지 않는다**.
- **증거 수위:** 초기 PlayMCP 503과 NCS 503은 서로 다른 외부 경로/시도의 실패다. redacted 실행 기록의 `trace_ids_present=false`는 각 API 성공 여부와 별도 관측성 항목이며 추적 로그가 없음을 곧 응답 실패라고 주장하지 않는다. API 연속 호출 1회 성공은 다수 사용자 부하/검색 정확도·추출 의미 품질·배포 안정성을 입증하지 않는다.
- **채용용 서사:** '공고 분석 시스템 전체 구현'보다 **기존 기능의 인증 연속 경로를 자동화하고, 외부 MCP 인증 장애·토큰 만료를 분리해 정상 지원 fallback으로 E2E를 검증하면서 민감정보가 남지 않도록 한 경험**이 가장 강하다.

근거: [PR #149](https://github.com/aihuman-7th/proj2-3/pull/149) · [10/1 Local evidence](https://github.com/aihuman-7th/proj2-3/blob/main/docs/evidence/job-source-analysis-local-e2e-2026-10-01.md) · [10/1 Worklog](https://github.com/aihuman-7th/proj2-3/blob/main/docs/worklogs/2026-10-01_1400_job-source-authenticated-e2e.md) · [PR #167](https://github.com/aihuman-7th/proj2-3/pull/167) · [PR #186](https://github.com/aihuman-7th/proj2-3/pull/186).

---

## 3-7. MB-07 — MCP·Agent·DB 통합 검증과 거짓 양성 제거 (검증 완료: 2026-10-22)

### 5단계 경험 서사 — 상황 → 문제 → 판단 → 조치 → 결과

1. **상황:** 말해봄은 `POST /api/agent`에서 로그인 사용자와 질문·답변 revision·공고 snapshot을 식별하고, 서버의 **FastMCP stdio 읽기 도구**(`get_question`, `get_practice_attempt`, `get_job_context`)로 허용 문맥을 읽어 LLM 코칭→스키마·인용·ID 검증→`coaching_runs` 저장을 수행하도록 발전했다. 별도의 HTTP `/mcp`는 읽기 도구와 capability token scope를 제공했다. **Agent/MCP의 주요 구현은 팀 코드**로, 검증 범위를 자신의 전체 코어 구현이라고 주장하지 않는다.
2. **문제:** 개별 API·stdio·HTTP MCP 테스트가 있어도 이 결과만으로 **인증된 실제 사용자가 음성을 분석하고, Agent가 올바른 MCP 문맥을 읽고, 유효한 코칭 출력을 DB에 새로 저장했는지** 확인할 수 없었다. 테스트용 MCP 오류를 '실제 운영 실패'로 오인할 수도 있었고, `needs_more_info` 같은 유효한 코칭 terminal status를 `ok`가 아니라고 실패 판정할 위험이 있었다. DB 검증기가 단지 **같은 revision/snapshot의 기존 `coaching_runs` 행이 있음**만 확인하면 현재 POST의 저장 실패를 성공으로 오판하는 심각한 **stale-row false positive**가 있었다.
3. **판단:** 검증 증거를 **(a) fixture 기반 MCP 호출·권한 경계, (b) 로컬 인증 사용자 Agent E2E, (c) Langfuse provider/validation/trace, (d) DB에 이번 호출로 새 행이 저장됐는지**로 분리해야 한다. 정상 입력에도 설명 근거가 모자라면 `needs_more_info`는 실패가 아니라 **정상적인 근거 기반 보류**다. 저장 성공은 HTTP 200이 아니라 **요청 전후의 새 행 ID + 동일 revision/snapshot + 최종 API status 일치**로 검증해야 한다. 운영 배포·실제 MCP 서버 인증과 로컬 fixture를 동일 성과로 묶지 않는다.
4. **조치:**
   - **[#112 — D6 MCP 경계](https://github.com/aihuman-7th/proj2-3/pull/112):** 김재훈 명의 최초 구현 커밋으로 HTTP `/mcp`의 scope 밖 질문 ID 거부 및 `/api/agent`→실제 **stdio FastMCP subprocess 3도구** 호출·ID 일치·gateway 오류→503 정규화 테스트를 추가했다. 이후 서정현 작성 보정 커밋으로 **fixture 검증은 운영 E2E가 아니므로 체크포인트 완료를 부분 완료로 수정**했다.
   - **[#114 — 인증 사용자 Langfuse 증거](https://github.com/aihuman-7th/proj2-3/pull/114):** 김재훈 작성 커밋으로 로그인 사용자 WAV/STT→Agent→MCP→provider→validator→DB→Langfuse 흐름에서 **network_error 시도 1/2, 오류 복구, 성공 trace**의 관측을 기록했다. 출력 validator 성공 후 503으로 실패한 별도 사례의 원인은 **로컬 Supabase에서 `coaching_runs.criteria_version` migration 누락**이었고, migration 반영 후 재실행 성공을 기록했다. DB 오류가 기존 `provider_error`로 뭉개져 보일 수 있다는 진단 한계도 남겼다.
   - **[#115 — 실제 사용자 3종 검증](https://github.com/aihuman-7th/proj2-3/pull/115):** 김재훈 명의 4개 커밋으로 **normal / boundary / injection의 실제 로컬 브라우저 인증 흐름**을 테스트하고, 정상 입력의 `needs_more_info`를 유효 상태로 인정하도록 `D6_CASE_ID`별 허용 HTTP·terminal status와 runner exit 판정/회귀 테스트를 연결했다. 동료 리뷰에서 발견된 **PR·Worklog·체크포인트 간 1차→2차 비교 완료 상태 불일치**도 정정했다.
   - **[#118 — 재현 가능한 자동화](https://github.com/aihuman-7th/proj2-3/pull/118):** 김재훈 작성으로 인증 사용자 JWT를 숨김 입력/환경변수로 받고, 완료된 **사용자가 명시 선택한 테스트용 analysis job**에 합성 immutable answer revision을 생성해 정상·경계·인젝션 각각 `/api/agent`를 검증하는 runner 및 안내 문서를 구축했다. 자체 소스·mock 테스트/CI 검증과 **이 runner가 실제 사용자에서 반복 실행됐다는 증거는 분리**한다.
   - **[#122 — DB 저장 검증 코드 리뷰](https://github.com/aihuman-7th/proj2-3/pull/122):** 서정현 작성 verifier가 마지막 matching row만 조회해 **과거 저장 행을 이번 POST 성공으로 착각**할 수 있는 결함과 **DB status 불일치 미검출**을 김재훈이 `CHANGES_REQUESTED`로 지적했다. 후속 커밋은 POST 이전의 마지막 run ID를 확보하고 POST 이후 **다른 새 ID** 및 API/DB status 일치를 검사하도록 수정했고, 김재훈이 focused **3개 테스트와 compile 확인 뒤 승인**했다.
5. **결과:** [MCP D6 Worklog](https://github.com/aihuman-7th/proj2-3/blob/main/docs/worklogs/2026-09-29_1133_mcp-d6-verification.md)에 **HTTP MCP 11개, Coaching runtime 24개, Agent API 12개 테스트 통과**가 기록됐다. 로그인 사용자의 **normal·boundary·injection 세 코칭 사례는 각 `needs_more_info`로 처리되며 테스트 의도에 맞는 결과**였고, 1차 저장·코칭→재답변 저장·코칭→비교 화면까지 로컬 인증 흐름을 기록했다. Langfuse에서는 실제 provider의 controlled network 실패·1회 retry 및 migration 반영 이후 성공 trace의 비용·토큰·validation 관측을 확보했다. DB verifier가 **이전 실행의 행을 새 저장 성공으로 잘못 인정하는 회귀**를 막는 규칙이 반영됐다. **하지만 운영 환경의 실제 MCP tool→동일 호출 DB 저장을 끝까지 잇는 통합 trace, 사용자 전체 E2E 성공률, 사람 semantic 코칭 품질**은 이 근거로 확정하지 않는다.

### PR 귀속과 검증 성격의 경계

- **개인 직접 작업:** [#112](https://github.com/aihuman-7th/proj2-3/pull/112) **첫 3커밋**의 MCP 호출/차단·오류 테스트 및 CI secret fingerprint 후처리, [#114](https://github.com/aihuman-7th/proj2-3/pull/114) Langfuse 인증 사용자 실측 증거, [#115](https://github.com/aihuman-7th/proj2-3/pull/115) 실제 UI 3종 및 1차/2차 비교, [#118](https://github.com/aihuman-7th/proj2-3/pull/118) 인증 Agent 자동화 runner. 초기 **[#122](https://github.com/aihuman-7th/proj2-3/pull/122)** 직접 리뷰의 false-positive 결함 발견·변경 요청·재검토 승인도 중요하다. AI 코딩 도구의 실제 코드 작성을 포함한 작업 흐름이므로 Git author만으로 모든 코드를 수작업으로 작성했다고 주장하지 않는다.
- **팀 코어 및 별도 구현:** MCP 문맥 접근, Agent runtime, DB 비교 및 HTTP MCP 기능 다수는 **곽호준 작성 [#39](https://github.com/aihuman-7th/proj2-3/pull/39)**를 포함한 팀 기여. 서정현은 [#112](https://github.com/aihuman-7th/proj2-3/pull/112) 마지막 평가 근거 교정, [#113](https://github.com/aihuman-7th/proj2-3/pull/113) 초기 synthetic/live harness, [#116](https://github.com/aihuman-7th/proj2-3/pull/116) D6 verifier, [#117](https://github.com/aihuman-7th/proj2-3/pull/117) MCP 증거 검증, [#122](https://github.com/aihuman-7th/proj2-3/pull/122) DB 저장 verifier 구현, [#172](https://github.com/aihuman-7th/proj2-3/pull/172) D9 MCP 실패 검증의 커밋 작성자다. 김재훈의 리뷰를 팀원 구현으로 잘못 귀속하지 않는다.
- **증거 주의:** `HTTP /mcp 200`에서 `result.isError=true`는 capability 거절이 정상적으로 동작한 예다. 실제 stdio subprocess로 간 fixture는 모델 제공자·실제 Supabase JWT·운영 MCP 성공까지 검증하지 않는다. 초기 MCP 오류는 일반 `provider_error`로 매핑됐고 후속 D9에서는 `mcp_error`로 분리됐다. 서로 다른 시점 계약을 동시에 현재 상태로 주장하지 않는다.
- **로컬 DB migration:** `criteria_version` 컬럼 누락은 성공한 provider 응답 이후 **persistence 단계 실패**였고, migration 후 `needs_more_info` 성공 trace를 얻었다. **DB 저장 확인은 별도 수동 SQL 조회 증거와 자동 verifier를 구별**하고, #122의 PR 테스트 3개가 운영 DB 영속성을 재측정한 것처럼 쓰지 않는다.
- **채용 활용:** 핵심 구현 전체 소유보다 **(1) MCP/Agent 경계 통합 검증, (2) 유효한 `needs_more_info` 상태 판정 교정, (3) 신규 저장 행 여부를 구분하지 못한 테스트의 false positive 제거**를 개인 판단·조치로 제시한다. 이 중 #122의 변경 요청은 단순 ‘테스트 작성’보다 실패 가능성 식별과 명확한 검증 계약의 기술 깊이가 높다.

원본: [MCP 통합 근거](https://github.com/aihuman-7th/proj2-3/blob/main/docs/worklogs/2026-09-29_1133_mcp-d6-verification.md) · [인증 사용자 Langfuse](https://github.com/aihuman-7th/proj2-3/blob/main/docs/worklogs/2026-09-29_1430_langfuse-authenticated-e2e.md) · [D6 실제 3종](https://github.com/aihuman-7th/proj2-3/blob/main/docs/worklogs/2026-09-29_1632_d6-agent-live-e2e.md) · [D6 검증 기록](https://github.com/aihuman-7th/proj2-3/blob/main/docs/worklogs/2026-09-30_0930_d6-runtime-evidence.md) · [DB verifier](https://github.com/aihuman-7th/proj2-3/blob/main/backend/tests/verify_d6_coaching_completion.py) · [AGENT_REPORT](https://github.com/aihuman-7th/proj2-3/blob/main/docs/AGENT_REPORT.md).

---

## 3-8. MB-08 — Whisper Medium 50k QLoRA 재학습·B0 정제·C2 면접 도메인 적응 (검증 완료: 2026-10-22)

### 5단계 경험 서사 — 상황 → 문제 → 판단 → 조치 → 결과

1. **상황:** 1차 프로젝트의 부산·경상도 음성 6k LoRA 기록과 CTranslate2 추론 모델은 남아 있었지만, **추가 학습용 `adapter_model.safetensors` 가중치는 인계에서 누락**됐다. 원본 adapter를 복원하지 못했으므로 legacy 6k→50k를 **같은 모델을 이어 학습한 것으로 기록할 수 없었다**. Whisper Medium 기반 신규 STT 학습과 실제 면접 도메인 검증이 필요했다.
2. **문제:** 기존 quick-eval 소스 `DKSR20000058`는 새 train에 **181개 segment**가 중복돼 공정한 holdout으로 사용할 수 없었고, 50k 학습 텍스트에는 `#이름#`·`&company-name...&` 같은 익명화 placeholder가 약 4% 섞여 실제 음성과 정답 label 불일치 위험이 있었다. 새 **AI Hub 71592 ICT/New 장문 면접 답변은 그대로 Whisper 30초 학습 입력에 쓰기 어려워** segment 분할이 필요했다. 면접형 새 도메인에 적응할수록 기존 AI Hub 119 발화의 **catastrophic forgetting**이 생길 수 있었다. 단일 화자의 대본 녹음 성능만으로 운영 기본 모델을 정하기도 어려웠다.
3. **판단:** 누락 adapter를 기다리거나 기존 추론 CT2를 새 LoRA 원본처럼 사용할 수 없으므로 **`openai/whisper-medium`에서 독립된 50k QLoRA 계보**로 다시 시작한다. train/validation과 source가 겹치지 않는 **고정 500개 holdout**, 정제 후 별도 **Test B 500개**, 신규 71592 validation/기존 119 retention/봉인된 최종 test를 **서로 다른 목적**으로 관리한다. C1(71592 only)·C2(71592 80% + clean 119 replay 20%)는 동일 설정으로 비교하고, 새 도메인 validation이 유사하면 기존 도메인 성능 감소가 덜한 쪽을 선택하되 **최종 test 공개 전 선택 모델 ID·adapter SHA·CT2/validation/retention fingerprint를 고정**한다. 모델 채택은 기준별 성능·다화자 일반화·서빙 추론 비교 후 별도로 결정한다.
4. **조치:**
   - **50k 데이터·학습 pipeline:** 김재훈 작성 stacked [PR #23~34](https://github.com/aihuman-7th/proj2-3/pull/34)에 holdout/Test B·clean manifest 준비, dataset validation, STT metrics, `train_whisper_qlora.py` QLoRA 학습기와 shared helpers, CT2 평가·overnight job runner·운영 정책을 구현했다. 50,000 train/5,000 validation, NF4 4-bit·LoRA `q_proj/v_proj` rank16/alpha32, RTX 5070 Ti에서 effective batch32·2 epoch로 실행했다. **3,126 optimizer step** 완료, validation CER 기준 **checkpoint-3000**을 선택하고 CT2 float16으로 변환해 기존 추론 CT2와 **source-disjoint holdout 500개**에서 비교했다.
   - **B0 clean continuation:** placeholder 포함 사례만 원본 manifest와 별도로 제외해 **48,000 train/4,802 validation**을 만들고, 앞서 이용한 Test A를 제외한 **독립 Test B 500개**를 고정했다. 50k 재학습 CT2 대비 별도 B0를 같은 조건으로 평가했다. 이 성능 평가를 50k의 다른 holdout 결과와 직접 빼서 개선폭이라고 하지 않는다.
   - **71592 domain adaptation:** 김재훈 작성 [PR #51~56](https://github.com/aihuman-7th/proj2-3/pull/56)에서 persona 준비·retention·heldout·AI Hub 자료 획득과 **B0 CT2를 타임스탬프 정렬 기준으로 사용해 공식 전사문을 label로 보존한 분할**, C1/C2 학습·평가를 구성했다. 5,840 요청 중 **5,808 source record 수용 / 32개 거부**, **27,056 clips**, 실학습 신규 **24,556개**, 20% historical replay **6,139개(합계 30,695)**를 사용했다. C1/C2는 **LR 1e-5·0.5 epoch·seed45**, 선택은 71592 validation + 119 retention으로 수행했다. 팀원의 [PR #51](https://github.com/aihuman-7th/proj2-3/pull/51) 리뷰에서 과도하게 넓은 `--overwrite` 재귀 삭제 대상을 **`data/stt` 아래 생성 dataset 직접 자식으로 제한**하고 영구 fail-closed 회귀 테스트를 복원했다. [PR #56](https://github.com/aihuman-7th/proj2-3/pull/56)에서는 **문자열 boolean만으로 봉인 test를 실행하는 위험**을 지적받아 `configs/stt/71592_c2_selection.json`에 **선택 ID·adapter SHA·CT2·validation·retention fingerprint**를 기록하고 최종 test manifest 생성보다 먼저 검증해 변조·누락 시 거부하도록 수정했다.
   - **실사용 평가 및 운영 후보 판정:** 개인 PR [#69](https://github.com/aihuman-7th/proj2-3/pull/69)로 1인 대본 녹음 30건의 **raw/발화형 정규화 CER·WER 및 stress 구간**을 분리하고 실제 오인식이 정규화에 의해 숨겨지지 않도록 테스트했다. [#70](https://github.com/aihuman-7th/proj2-3/pull/70)에는 30건 오디오·대본의 **canonical SHA 고정과 override 제거**, [#71](https://github.com/aihuman-7th/proj2-3/pull/71)에는 immutable large-v3-turbo 모델 fingerprint와 평가 데이터 fingerprint를 포함한 **모델 3종 × beam1/3/5 총 9개 CUDA 비교**, 변경 입력을 `--resume`이 잘못 재사용하지 못하도록 보호하는 리뷰 수정을 반영했다. [#72~73](https://github.com/aihuman-7th/proj2-3/pull/73)의 hint 및 동적 initial_prompt는 **별도 서빙 추론 개선 계보**로 분리했다. [#146](https://github.com/aihuman-7th/proj2-3/pull/146)으로 재학습 전·후 비교 근거를 종합 정리했다.
5. **결과:**
   - **50k 재학습, 동일 holdout 500:** legacy CT2 대비 CER **9.1435→8.2708%**(**−0.8727%p**)이나 WER **16.1931→16.2937%**(**+0.1006%p**). 문자 edit는 901→815, word edit는 644→648. WER 미세 악화에는 한국어 공백 경계 차이도 포함됐다.
   - **B0 clean, 독립 Test B 500:** 50k 대비 CER **5.6038→5.3738%**(**−0.2300%p**), WER **13.5489→13.1213%**(**−0.4276%p**), 두 지표 개선.
   - **C1 vs C2, 신규 71592 validation:** C1 CER/WER **2.9298/9.5594%**, C2 **2.9290/9.5510%**, 사실상 비슷함. **기존 119 retention 4,802**에서 B0 WER **15.3344%**, C1 **17.1378%**, C2 **16.8062%**라 **C2가 C1보다 덜 잊지만 B0보다 악화**. 정량적 허용 회귀 threshold를 사전 확정한 것은 아니므로 **실험 조건 내 상대적 C2 선택**이다. C2 선택을 동결하고 최종 71592 **봉인 test 1,233건 CER 2.5876%, WER 9.7740%**를 보고했다. **동일 test의 B0 값은 없어 B0→C2 개선율 계산 불가**. 70:30 replay는 실행하지 않았고 봉인 test를 후속 튜닝에 재사용하지 않는다.
   - **단일 화자 대본 30건:** B0→C2 전체 raw CER **16.03→15.14%** 개선, raw WER **19.59→20.91%** 악화. 숫자/영문 stress 6건 제외한 **24건 발화형 정규화** WER **11.90→10.71%** 개선(기술 질문 8건 **20.11→11.73%**), 그러나 전체/기존 도메인 일반화는 불가능. **모델 3종×beam 9조합** 비교에서는 **large-v3-turbo beam1**이 30건의 비-stress core 정규화 CER/WER **3.29/6.63%**, **RTF 0.020**, p50 0.258초로 B0/C2보다 더 유리했으므로 서빙 추론의 다음 **실험 기준선**으로 선택했다. **C2가 제품 기본 STT로 승격된 것은 아니다.**

### 개인 기여·모델 계보와 주장 경계

- **Git 커밋 귀속 A:** 개인 [PR #23~34](https://github.com/aihuman-7th/proj2-3/pull/34) retrain/clean 데이터·평가·훈련·밤샘 실행, [#51~56](https://github.com/aihuman-7th/proj2-3/pull/56) persona/71592 획득·segmentation·C1/C2 훈련/retention/봉인 test, [#69~73](https://github.com/aihuman-7th/proj2-3/pull/73) 실제 녹음 평가·모델/beam·힌트, [#146](https://github.com/aihuman-7th/proj2-3/pull/146) 원본 비교 통합. 사용자 명의 Worklog에는 **김재훈/ChatGPT**, 학습 실험에는 GPU 실행 및 체크포인트·평가 결과가 별도 보존되어 **개인 주도 실험**으로 표현 가능하나 Git author만으로 모든 줄 수작업 작성·모든 밤샘 실행 단독 조작을 의미하지 않는다.
- **동료 검토와 개선:** PR #51/#56의 **SJeongHyeon** 수정 요청(삭제 범위/봉인 test 검증), #70/#71의 canonical devset·모델/재개 fingerprint 보강은 김재훈 명의 후속 커밋에서 수정해 승인·병합됐다. 구현과 리뷰의 기여를 서로 구분한다.
- **평가셋 계층과 성능 해석:** 50k holdout 500, B0 clean Test B 500, 71592 validation/봉인 test 1,233, 기존 119 retention 4,802, 한 화자 녹음 30은 서로 다른 모집단. 원본 medium→legacy 6k→50k→C2의 단일 고정 test 개선율은 **확인 불가**. 71592는 **면접/ICT 내용 적응**이며 검증된 지역 출신 metadata가 없어 이를 **경상도 화자 개선**으로 주장하지 않는다. 71592 split은 source-record disjoint이나 글로벌 speaker-disjoint 입증이 없다.
- **독립성 주의:** 최종 71592 test는 선택 뒤 열렸고 **후속 tuning 금지**. 실사용 30건은 한 화자 대본 낭독이며 devset으로 모델/beam을 골랐으므로 최종 acceptance·다화자 성능 근거가 아니다. large-v3-turbo/beam/initial_prompt 작업은 **가중치 파인튜닝이 아닌 inference optimization**이고 C2/기존 서비스 모델의 실제 배포 전환을 뜻하지 않는다.
- **채용 활용:** 데이터 누수·placeholder를 발견하고 고정 holdout 및 추가 학습 계보를 새로 수립, replay 기반 망각 트레이드오프를 **절대 성능과 구분해 선택**, 봉인 test·모델/데이터 provenance를 리뷰에 따라 강화한 **실험 설계·재현성·기술 판단** 사례로 제시한다.

원본: [STT 계보 종합](https://github.com/aihuman-7th/proj2-3/blob/main/docs/stt-finetuning-lineage.md) · [50k retrain](https://github.com/aihuman-7th/proj2-3/blob/main/docs/experiments/stt/2026-09-19_whisper-medium-50k-retrain.md) · [B0 clean](https://github.com/aihuman-7th/proj2-3/blob/main/docs/experiments/stt/2026-09-21_clean-continuation-lr2e-5.md) · [71592 C1/C2](https://github.com/aihuman-7th/proj2-3/blob/main/docs/experiments/stt/2026-09-22_aihub71592-domain-adaptation.md) · [1인 30건](https://github.com/aihuman-7th/proj2-3/blob/main/docs/experiments/stt/2026-09-27_real-user-recording-eval.md) · [모델×beam 9조합](https://github.com/aihuman-7th/proj2-3/blob/main/docs/experiments/stt/2026-09-28_real-user-model-beam-matrix.md) · [9/22 개인 Worklog](https://github.com/aihuman-7th/proj2-3/blob/main/docs/worklogs/2026-09-22_stt-persona-71592-continuation.md).

---

## 3-9. MB-09 — 로그인 사용자 WAV/STT·전사·코칭·비교 및 서비스 지연 진단 (검증 완료: 2026-10-22)

### 5단계 경험 서사 — 상황 → 문제 → 판단 → 조치 → 결과

1. **상황:** 말해봄은 프런트 연습 화면에서 공고·질문을 선택하고 음성을 입력해 **비동기 분석 작업 생성 → VAD/STT → 전사문 확정 → Agent 코칭 → `coaching_runs` 저장 → 1차·2차 비교**를 보여주는 제품으로 통합 중이었다. 개별 STT/Agent 단위 기능만으로 실제 사용자의 연속 경로와 체감 지연을 알기 어렵고, 인증·로컬 Supabase 연계도 필요했다.
2. **문제:** 로컬 Docker/ Supabase 통합 초기에 **service-role credential 구성이 잘못돼 DB 접근 401**, 경로 검증 중 별도 시점에 **Storage schema 및 `coaching_runs.criteria_version` migration 누락**이 발생했다. 모델 출력/HTTP 200만 보면 전사 확정·실제 DB 저장·비교 렌더링 누락을 놓칠 수 있다. 또한 **STT 단일 추론 속도**는 음성 업로드·작업 생성·polling·VAD·DB 저장까지 포함한 **사용자 체감시간**과 다르며, Cloud Run의 CPU 수나 이미지 크기를 줄여도 운영 저장소 성능이 좋아졌다고 단정할 수 없다.
3. **판단:** (a) **실제 로그인 JWT를 사용한 로컬 제품 기능 E2E**, (b) **WAV를 사용한 1차 코칭→재답변 저장·비교**, (c) **/analysis-jobs 업로드→terminal 서비스 지연 benchmark**, (d) **Hosted Supabase/RLS/Storage 포함 배포 E2E**를 구분해 근거를 모아야 한다. **실제 마이크를 켜서 연속 녹음했다는 검증 근거는 없으므로 WAV 업로드 E2E로 정확히 기재**한다. 1차/2차에서 같은 WAV를 사용하면 말하기 속도·침묵 지표가 그대로인 것이 예상 결과이며, 내용 판정 개선을 **발화 능력 향상**으로 과장하지 않는다. Cloud Run mock storage benchmark 결과를 운영 사용자 DB의 성능으로 취급하지 않는다.
4. **조치:**
   - **9/21 로컬 인증 통합:** 김재훈 계정 **[PR #47](https://github.com/aihuman-7th/proj2-3/pull/47)**와 Antigravity 작업 [기록](https://github.com/aihuman-7th/proj2-3/blob/main/docs/worklogs/2026-09-21_2315_local-supabase-candidate-integration-e2e.md)에서 팀원 **[#42](https://github.com/aihuman-7th/proj2-3/pull/42)**의 로컬 Supabase Docker 인증 수정사항을 결합했다. 프런트 `:18080`와 API `:8000`에서 **실제 로컬 사용자 가입/로그인→공고 목록→연습 세션→7초 음성 `POST /analysis-jobs` 업로드·Faster-Whisper/Silero VAD 처리→전사 확정 revision→`POST /api/agent`→stdio MCP·실제 OpenRouter→validator→로컬 Supabase `coaching_runs` 저장·응답**을 확인했다. 인증 환경을 수정한 **#42 원 코드는 곽호준 작성**이며, #47의 주된 개인 변경은 서비스 후보 통합 runner/설정·실험 증거라는 점을 분리한다. 같은 실행의 후보 2종×3건 API 모델 평가는 **MB-03에서만 독립 성과로 집계**한다.
   - **9/29 실제 로그인 UI 재답변 확인:** 김재훈 개인 **[#115](https://github.com/aihuman-7th/proj2-3/pull/115)**에서 기존 평가용 WAV를 업로드하고 전사 확인 화면에서 **normal/boundary/injection** 입력을 각각 확정해 브라우저 Network의 Agent payload/status와 결과 렌더링을 확인했다. normal 답변에 부족했던 **개인 행동·행동 선택 이유·공고 연결**을 보완한 2차 답변을 제출한 뒤 **1차·2차 내용 비교 카드**에 개선된 criterion 상태와 `여전히 부족한 설명` 안내가 표시되는지 확인했다. **같은 WAV를 1·2차에 재사용**했기 때문에 전달력 지표(319음절/분·긴 침묵 0·반복 0)는 같다고 기록했다. [#115](https://github.com/aihuman-7th/proj2-3/pull/115) 리뷰로 **유효한 `needs_more_info`도 정상 완료로 인정하는 case별 판정과 Worklog/PR 상태 불일치**를 수정했다. 이 Agent 정책 테스트 보정은 **MB-07에도 근거로 사용되므로 중복 성과로 계상하지 않는다**.
   - **9/28 분석 E2E 병목 측정:** 김재훈 작성 **[#104](https://github.com/aihuman-7th/proj2-3/pull/104)**에서 인증된 `POST /analysis-jobs` 업로드→HTTP 202→terminal polling, warmup 분리, **p50/p95·STT RTF**, 작업/Storage 정리, 내부 **VAD·hint·STT·전달력·persistence 구간 timing**을 수집하는 runner·실험 문서를 추가했다. 이후 **[#106](https://github.com/aihuman-7th/proj2-3/pull/106)**에서 Cloud Run용 CPU-only PyTorch/Whisper pre-baked 이미지 최적화와 추가 리뷰의 **immutable 모델 revision + image label 고정**을 반영했다. 사용자의 오디오·토큰·공고·전사 원문은 로그에 남기지 않았다.
5. **결과:**
   - **로컬 제품 경로:** 9/21 **로컬 인증 사용자·7초 업로드·STT/VAD·확정 revision·Agent·DB 저장**의 연속 성공을 확인했다. 9/29 별도 입력에서는 **normal/boundary/injection 3건**에서 `needs_more_info` 기반의 근거 보류 코칭을 확인했고, **1차 저장→코칭→2차 저장→코칭→비교 화면**을 재현했다. 두 날짜의 실행은 **같은 하나의 운용 요청/연속 trace가 아니며** 서로 다른 재현 근거로 보존한다.
   - **서비스 지연:** 로컬 Docker **CPU/int8, 측정 10회**에서 음성 업로드→terminal **E2E p50 3.670초/p95 3.825초**, 그중 STT p50 **3.370초/p95 3.573초**. CPU Cloud Run benchmark(전용 mock/local storage)에서는 **2/4/8 vCPU E2E p50 21.082/10.717/7.988초**, STT p50 **20.697/10.281/7.596초**로 vCPU 확장에 따른 수익 감소를 확인했다.
   - **이미지 최적화:** Cloud Run 비교 시 기존 이미지 **3.444GB→1.934GB**, Cloud Build **589.1→266.6초**, 4 vCPU rollout 후 첫 요청 E2E **35.792→17.249초** 기록. 그러나 앞의 크기/지연값은 **모델 revision 고정 전 이미지를 측정**했고, 이후 pinned revision 이미지의 재빌드·동일 조건 재측정은 확인되지 않았다. 이 시간은 **scale-to-zero cold start**가 아닌 rollout 직후 첫 요청이다.
   - **범위 미완료:** **실제 마이크 녹음부터의 제품 전체 E2E**, hosted Supabase의 운영 JWT/RLS/Storage까지 포함한 **동일 배포 사용자 전체 흐름**, 다중 사용자 부하·독립 사람 의미 품질은 확정하지 않는다. 9/28 Cloud Run 수치는 mock/local 저장소이며, **전역 STT 모델 또는 프로덕션 구성의 최종 승격 근거가 아니다**.

### 개인 작업 귀속과 MB-07/08 중복 방지

- **개인 A:** #47(로컬 후보 서비스 통합·실행 증거, Antigravity 협업), #115(로그인 UI에서 WAV/STT부터 2차 비교 실측·상태 정합성), #104(서비스 레벨 분석 지연 관측 설계·측정), #106(CPU 배포 진단 이미지/모델 revision 보강). 장시간 외부 작업과 실제 실행은 **작업 기록 기반**으로, PR Git author만으로 모든 실행을 수작업 단독 수행했다고 주장하지 않는다.
- **팀 기여:** 로컬 Supabase 인증 #42는 **곽호준**, 후속 [#116](https://github.com/aihuman-7th/proj2-3/pull/116)/[#117](https://github.com/aihuman-7th/proj2-3/pull/117)의 `analysis_jobs`·MCP·`coaching_runs` 상세 SQL 검증 코드는 **서정현**. 프런트/음성 파이프라인의 주 구현 전체가 김재훈 독자 소유라는 근거는 없다.
- **다른 작업군과 분리:** 모델 2종×3건 계약·지연·비용 선택은 **MB-03**, Agent/MCP 실패·`needs_more_info`·저장 검증 false positive는 **MB-07**, 50k/B0/C2·STT 1인 개발셋 모델 비교는 **MB-08**. **MB-09 고유 핵심 성과는 WAV 기반 사용자 연속 흐름과 서비스 upload→terminal 전체 지연·CPU 운영 비용 진단**이다.
- **채용용 서사:** “음성 AI 모델 전체 구현”보다 **이미 구현된 STT·VAD·Agent·Supabase의 사용 경로를 인증 상태에서 실제 검증하고, 체감 지연을 stage별로 분해해 리소스 증설·이미지 최적화의 유효성과 한계를 결정한 경험**으로 제시한다.

원본: [9/21 local E2E](https://github.com/aihuman-7th/proj2-3/blob/main/docs/worklogs/2026-09-21_2315_local-supabase-candidate-integration-e2e.md) · [9/29 live 1차·2차](https://github.com/aihuman-7th/proj2-3/blob/main/docs/worklogs/2026-09-29_1632_d6-agent-live-e2e.md) · [9/28 STT service benchmark](https://github.com/aihuman-7th/proj2-3/blob/main/docs/experiments/stt/2026-09-28_analysis-e2e-benchmark.md) · [9/28 Worklog](https://github.com/aihuman-7th/proj2-3/blob/main/docs/worklogs/2026-09-28_analysis-e2e-benchmark.md) · [10/2 근거 경계](https://github.com/aihuman-7th/proj2-3/blob/main/docs/worklogs/2026-10-02_stt-checklist-evidence-refresh.md).

---

## 3-10. MB-10 — Langfuse 실제 실행 버전 정합성과 운영 Trace 증거 검증 (검증 완료: 2026-10-22)

### 5단계 경험 서사 — 상황 → 문제 → 판단 → 조치 → 결과

1. **상황:** 말해봄은 코칭 요청의 `coaching.analyze`·`coaching.provider`·`coaching.output_validation`을 Langfuse Japan Cloud에서 기록하고, 모델·Prompt·Skill·재시도·토큰·비용을 관측하도록 개발 중이었다. 과거 모델 비교 자료는 로컬 `comparison.json`에 존재했고, 개발/운영별 Langfuse SDK·Prompt label·실제 Skill 버전이 서로 달라 재현과 비교가 어려웠다.
2. **문제:** (a) 이미 실행한 합성 실험을 Langfuse에 등록하려고 **provider를 재호출하거나 import 소요시간·비용을 원래 평가 수치와 혼동**할 위험, (b) `interview-coach-v6.1-domain-rules` 카탈로그와 관측 기본값 `interview-coach-v6` 불일치, 관리형 Prompt 선택과 무관하게 **DB가 stale 환경변수를 읽어 실제 사용하지 않은 버전을 저장하는 문제**, (c) API HTTP 200, Langfuse 유효한 JSON span, 관리형 Prompt `production` label만으로 **동일 운영 요청의 Cloud Run revision·DB 저장·실제 Skill bundle·응답 의미 품질**까지 입증했다고 과장할 위험이 있었다. 과거 코칭의 401 `auth_error`는 모델 미호출 사례인데도 운영 코칭 성공 trace로 혼동될 수 있었다.
3. **판단:** **실제 런타임이 선택한 버전을 단일 사실 근거(source of truth)**로 관측과 DB에 전달하고, 과거 레코드는 근거 없이 소급 보정하지 않는다. 기존 합성 실험의 Langfuse 이전은 **원래 평가 attempt·score·latency/cost를 metadata로 기록하는 dry-run-first import**, 실제 API 호출은 별도다. 관측 증거는 **로컬 인증 사용자 / 운영 Langfuse 같은 trace 내부 / 실제 사이트의 HTTP header ↔ Langfuse ↔ Cloud Run 동일 ID**로 단계화한다. HTTP 401·422의 올바른 차단과 실제 생성 성공을 나누고, 관리형 Prompt의 코드 `v4/v5`·Langfuse 숫자 `@4`·mutable `production` label을 구분한다. 운영 release/rollback은 **승인된 전환·복구 실측**이 있어야 완료 처리한다.
4. **조치:**
   - **[#101 — 평가 결과 import 도구](https://github.com/aihuman-7th/proj2-3/pull/101):** 김재훈 작성 `evals/export_langfuse_experiments.py`에서 원래 `comparison.json`의 **execution별 마지막 contract attempt**, candidate×repeat별 별도 run, synthetic Dataset item만 허용, `recorded_latency_ms`/`recorded_cost_usd` 및 `provider_recalled=false`, 결정적 run name·기본 dry-run/`--apply` 명시를 구현했다. SDK 4.15.1 포함 관련 **35개 tests**, mock 2후보×2반복×30건 **120 offline execution → 4개 run 계획 dry-run**을 확인했다. **원격 Langfuse `--apply`를 실행하거나 실제 실험을 업로드한 것은 아님**.
   - **[#114 — 로컬 실제 provider·실패/retry 관측](https://github.com/aihuman-7th/proj2-3/pull/114):** 실제 로그인 사용자의 기존 WAV/STT→Agent/MCP→OpenRouter→validator→DB 경로에서 network endpoint를 고의로 차단해 `network_error` **transport attempt 1 재시도 승인, attempt 2 중단**을 기록했다. 실제 provider 성공·출력 검증 `valid=true`에도 DB `coaching_runs.criteria_version` migration 누락으로 503이 발생한 **persistence 오류를 provider 오류와 분리해 분석**했고, migration 반영 뒤 정상 `needs_more_info` trace를 확인했다. 실측 수치는 **14,560 tokens, 약 $0.007367**(한 성공 trace); 응답 원문·JWT는 Git·Langfuse 증거 문서에서 제외했다. 장애 검증 경로 자체는 MB-07에도 연결되므로 결과를 중복 산정하지 않는다.
   - **[#162 — Prompt/Skill 실제 실행 버전 기록 정합성](https://github.com/aihuman-7th/proj2-3/pull/162):** 김재훈 작성으로 관측의 기본 Skill 버전을 **실제 카탈로그에서** 가져오고, 모델 코칭 서비스가 **그 요청에서 선택한 managed Prompt 및 Skill 버전**을 `coaching_runs` 저장 함수에 명시 인자로 전달하게 했다. 요청 관측도 코칭 서비스 버전을 우선하고 구버전 환경변수에 의해 덮어쓰지 않도록 구성했다. Langfuse SDK 4.15.1 환경의 **70 tests 통과**, 오래된 SDK 2.60.10 환경의 관련 실패는 환경 계약 차이로 별도 기록했다. 기존 DB 행 소급 수정·모델 정책 변경·운영 배포/Prompt `production` label 변경은 하지 않았다.
   - **[#216 — 10/8 관리형 Prompt 운영 관측](https://github.com/aihuman-7th/proj2-3/pull/216):** 사용자 제공 **Langfuse Cloud 캡처와 읽기 전용 조회**를 김재훈 명의 커밋으로 교차 검증했다. 별도 reinforced 관리형 Prompt의 **숫자 버전 @4, `production`·`latest` label**을 확인하고 10/8 **12:38:46 KST의 `coaching.provider`** generation을 `Linked Generations`와 **같은 trace ID**로 대조했다. **Gemini 3.5 Flash-Lite, 약 8.19초, 24,266 tokens, $0.006809, HTTP 200·no retry** 및 같은 트리의 `coaching.output_validation` **`valid=true`/`error_count=0`**을 기록했다. Skill `interview-coach-v6`은 **관측 표시값**이고 실제 코드/번들 해시나 운영 revision을 증명하지 못함을 명시했다. 이 PR은 **운영 설정 변경 없는 읽기 전용 확인 문서**이며 나중에 MERGED 됐다.
   - **[#221 — 10/9 운영 HTTP ↔ Trace 연결 개인 리뷰](https://github.com/aihuman-7th/proj2-3/pull/221):** **곽호준 작성 구현**에서 각 요청 ContextVar·`X-Trace-Id`/`X-Request-Id` 헤더 및 CORS 노출·`http.request` span·민감 본문 제외·동시성을 검토했다. 김재훈은 linkage/coaching API **28 tests 및 제공된 실제 200/422 증거 JSON**을 대조해 **APPROVED**. **CI job은 당시 결제/spending 한도로 시작 전 차단**돼 GitHub Actions/gitleaks 통과를 주장하지 않았고, 리뷰에서 새로운 운영 요청을 직접 재실행하거나 PR을 병합/배포하지 않았다고 명시했다.
5. **결과:**
   - **개인 구현:** 과거 실험 metadata의 Langfuse import 경계를 코드로 강제하고 **실제 선택한 Prompt·Skill 버전이 관측·DB에 전파되는 정합성 보장 코드와 회귀 테스트**를 마련했다. 실험 import는 **dry-run 성공**, 실제 Langfuse 업로드/새 비용 0.
   - **로컬 실제 인증 사용자 trace:** provider 네트워크 오류의 **2개 transport attempt(1회 재시도 후 실패)**, provider 정상·출력 검증 성공에도 DB migration 문제로 저장 실패한 사례, migration 이후 실제 `needs_more_info` 성공을 구분했다.
   - **10/8 운영 Prompt/코칭 관측:** 관리형 reinforced Prompt **@4 `production`** 및 같은 trace의 **provider·output validation PASS** 확인. 그 당시의 **HTTP 요청↔Cloud Run revision/DB 행의 동일 ID 직접 증거는 없음**.
   - **10/9 팀 운영 연결 구현의 결과 검토:** 실제 사이트의 준비 채팅 **HTTP 200**은 `X-Trace-Id`=`X-Request-Id`=Langfuse trace ID=Cloud Run request ID, **모델 generation 1건**. 필수 입력 누락 **HTTP 422**도 같은 ID가 연결되고 **generation 0건**. 이는 [10/9 증거](https://github.com/aihuman-7th/proj2-3/blob/main/docs/evidence/request-trace-linkage-2026-10-09.md) 및 동료 #221의 성과이고, 김재훈은 해당 코드·기존 운영 증거를 리뷰한 사람이다. **임시 인증 세션 주입을 사용해 OAuth 로그인 UI는 직접 검증하지 않았음.**
   - **남은 미확인:** 생산 Prompt @4의 **release gate·전환 과정·rollback 실증**, 10/8 코칭 trace의 동일 요청 Cloud Run revision·실제 Skill bundle fingerprint·`coaching_runs` DB 행, 같은 요청 PlayMCP, 최신 코칭 출력 사람 평가, 실제 마이크부터 전체 E2E. 10/9 준비 채팅 trace는 **코칭 `/api/agent` 전체·같은 Prompt 사용의 증거가 아님**.

### 개인 기여/팀 귀속 및 중복 방지

- **직접 작성 A:** [#101](https://github.com/aihuman-7th/proj2-3/pull/101) Langfuse Experiment import, [#114](https://github.com/aihuman-7th/proj2-3/pull/114) 인증 사용자 Langfuse 실측 정리, [#162](https://github.com/aihuman-7th/proj2-3/pull/162) effective Prompt/Skill provenance 코드, [#216](https://github.com/aihuman-7th/proj2-3/pull/216) 운영 @4 screenshot + read-only evidence. AI 코딩 도구/팀원 리뷰 협업이 있어 모든 코드를 수작업 단독 작성했다고 주장하지 않는다.
- **리뷰 R:** [#221](https://github.com/aihuman-7th/proj2-3/pull/221)은 곽호준 구현. 김재훈은 ContextVar·민감데이터 경계·표본 200/422 ID 연계 검증·CI 실패 공지를 포함한 승인 리뷰 담당. **Cloud Run 배포 직접 작업/원 구현으로 귀속하지 않는다.**
- **타 작업군과의 관계:** LLM 후보 점수/비용 선택은 MB-03, Prompt v4/v5 자동 품질 회귀는 MB-05, 로컬 Agent/MCP/DB 저장 failure 원인·상태 검증은 MB-07, WAV/STT부터 사용자 1·2차 비교는 MB-09. **MB-10 독립 중심은 observability의 정확한 실행 버전 기록·기존 실험 전시의 정합성·운영 trace 증거 수준 구분**이다.
- **문서/운영 시점:** 10/6 이후 후속 기능·병합된 PR 상태를 10/8 공식 마감 전에 직접 완료한 운영 변경으로 소급하지 않는다. Langfuse `production` 라벨 존재가 **본인이 직접 승격했다는 뜻은 아니며**, Prompt 숫자 버전과 코드 명칭을 동일 버전으로 간주하지 않는다.
- **채용 활용:** 평가 엔지니어링/AI 백엔드 관점에서 **버전 불일치(카탈로그 vs 관측/DB)의 원인 발견과 수정**, **재시도·DB 오류 트리 관측을 통한 원인 분리**, **HTTP response-to-trace cross-system 상관관계 검증**을 실제 판단·조치로 제시하되 운영 release/rollback 자동화를 구현했다고 쓰지 않는다.

원본: [Langfuse import 실험](https://github.com/aihuman-7th/proj2-3/blob/main/docs/worklogs/2026-09-28_1540_langfuse-evaluation-export.md) · [인증 사용자 관측](https://github.com/aihuman-7th/proj2-3/blob/main/docs/worklogs/2026-09-29_1430_langfuse-authenticated-e2e.md) · [Prompt/Skill 버전 정합성](https://github.com/aihuman-7th/proj2-3/blob/main/docs/worklogs/2026-10-02_coaching-version-provenance.md) · [10/8 운영 Prompt @4](https://github.com/aihuman-7th/proj2-3/blob/main/docs/worklogs/2026-10-08_d10-production-readonly-evidence.md) · [10/9 운영 직접 ID 연계](https://github.com/aihuman-7th/proj2-3/blob/main/docs/evidence/request-trace-linkage-2026-10-09.md) · [10/9 작업 기록](https://github.com/aihuman-7th/proj2-3/blob/main/docs/worklogs/2026-10-09_request-trace-linkage-production.md).

---

## 3-11. MB-11 — 제품 UI 사용성 검증과 독립 쇼케이스 설계·구현 (검증 완료: 2026-10-22)

### 5단계 경험 서사 — 상황 → 문제 → 판단 → 조치 → 결과

1. **상황:** 말해봄은 음성 면접 연습·전사 확인·코칭·재답변·기록 등을 React/Vite 기반으로 제공하고 있었지만, 외부 방문자가 제품 목적과 실제 사용 흐름을 이해하기 위한 별도 소개 경로가 약했다. 2026-10-07에는 기존 연습 화면의 라이트/다크 테마, 시작 화면, 녹음·전사 진행 상태의 시각적 일관성이 개선 대상이었다. 사용자 측에서 랜딩 개선안을 **11페이지 PDF 목업**으로 구체화해 소개 사이트 제작의 입력으로 제공했다.
2. **문제:** (a) 다크 모드 도입 과정에서 **`--app-background`·`--surface-raised`·`--border-subtle` 등 미정의 CSS 변수**, 기록/연습 화면의 고정 라이트 색상, 전사 입력창의 **어두운 배경에 어두운 텍스트가 남는 CSS specificity/cascade 충돌**이 있었다. (b) 랜딩 기획 목업·실제 제품 프런트 화면·현재 배포 동작을 혼동할 수 있고, 실제 모델 품질·데이터 보관 기간이 검증되지 않은 상태에서 홍보 카피로 확정할 위험이 있었다. (c) 별도 홍보 사이트를 기존 프런트에 즉시 결합하면 인증 라우팅/서비스 진입이 변하고 마감 전후 작업의 책임 범위가 불명확해질 수 있었다.
3. **판단:** 코칭 화면의 기능·API/DB 상태 전이는 보존하면서 **테마를 semantic token으로 통일하고 실제 전사 편집기 대비까지 검토**해야 한다. 랜딩 PDF를 출발점으로 **서비스 소개와 개발·평가 근거 설명을 분리한 독립 정적 사이트**를 만들어 CI/build 의존성을 줄이고, 기존 브랜드 자산을 재사용하면서 **기획용 UI와 운영 중 UI를 명확히 표시**한다. 확인하지 않은 성능·보관기간은 공개 카피에서 제외한다. 사이트 소스 제작/개인 별도 Pages/기존 운영 앱 진입 화면 결합은 **각각 독립 산출물·시점**으로 구분한다.
4. **조치:**
   - **10/7 UI 리뷰 [PR #211](https://github.com/aihuman-7th/proj2-3/pull/211):** 사용자의 메인·코칭 UI 개선 요청을 바탕으로 **김재훈 명의 `CHANGES_REQUESTED` 2회 + `APPROVED` 1회**가 남았다. CSS 변수 정의 누락으로 카드가 투명해지는 문제, 라이트 고정 CSS 때문에 공통 테마 전환 후 연습·기록·계정 일부가 어긋나는 문제, `.transcript-editor` 선택자 우선순위로 다크 모드에서 입력 글자가 잘 보이지 않는 문제를 각각 코드 줄·CSS 선택자로 지적했다. 동료/코딩 에이전트의 수정 커밋을 재검토하고 **normal 15.29:1, disabled 8.46:1** 대비를 확인했다. **실행 테스트·빌드는 Windows npm/WSL UNC 오류로 개인 환경에서 재실행하지 못했음**을 명시했고, 작성자 Worklog **181 tests 및 build PASS**는 별도 근거로만 기록했다. 실제 마이크·배포 E2E도 미실행.
   - **10/8 독립 정적 소개 사이트 개인 작성 [PR #214](https://github.com/aihuman-7th/proj2-3/pull/214):** 김재훈 명의 **3개 커밋**으로 `site/index.html` 서비스 소개와 `site/project/index.html` 개발·평가 소개를 구성하고, 기존 제품의 로고·마스코트·녹음/전사 코치 이미지·데모 4장·배경 SVG를 WebP 등으로 최적화해 복제, **보라색 브랜드 톤, Light/Dark/System·반응형 메뉴·FAQ·실서비스 CTA·콘텐츠 확인 체크리스트**를 적용했다. 의존성 없는 HTML/CSS/JS와 `site/scripts/check_site.py` 검사기를 추가했으며 최종 정적 검사 **서비스 18 IDs/31 links·assets, 프로젝트 7 IDs/30 links·assets**, 두 페이지 로컬 HTTP 200·MIME 및 이미지 시그니처·중복 앵커 검증을 통과했다. 실제 서비스 주소와 GitHub 원본 프로젝트 링크를 설정하고, **프로젝트 기간 2026-09-18~10-08**을 확인해 기록했다. **24시간 보관은 실제 삭제 정책 미확인이라 사이트 공개 확정문구에서 제외**했다.
   - **개인 쇼케이스 저장소:** [`nanocode00/malhaebom-showcase`](https://github.com/nanocode00/malhaebom-showcase)에 별도 정적 게시 루트(`README.md`, `index.html`, `project/`, `assets/`, `scripts/`)가 존재함을 **GitHub 연결 파일에서 확인**했다. README는 GitHub Pages `main / root` 게시 설정과 예상 URL을 안내하고 서비스는 `malhaebom.xyz`로 연결한다. **현재 Pages 응답/배포 시각을 본 조사에서 직접 점검한 것은 아니므로, 파일 존재와 운영 공개 실측을 동일시하지 않는다.**
   - **10/10 운영 소개 동선 통합의 팀원 귀속:** [PR #240](https://github.com/aihuman-7th/proj2-3/pull/240)에서 **곽호준**이 정적 소개 소스/자산을 이관했고, [PR #241](https://github.com/aihuman-7th/proj2-3/pull/241)에서 같은 작성자가 `/` 소개·`/app/` 서비스 홈·`/app/practice` 연습·OAuth 복귀·레거시 경로 호환을 구현했다. 원래 쇼케이스 HTML과 이관 자산에 **원본 SHA manifest**를 남겼고 **프런트 234 tests/build·경로 스모크**가 기록됐다. 10/10 당시 CI는 GitHub billing/spending limit 때문에 job을 시작하지 못해 로컬 증거와 구별했다. [PR #251](https://github.com/aihuman-7th/proj2-3/pull/251)의 최신 소개 콘텐츠/운영 화면 갱신도 **곽호준** 작성. 이 통합·후속 작업은 **공식 프로젝트 종료 10/8 이후**의 팀 작업이다.
5. **결과:** 제품 UI 리뷰에서 **미정의 토큰·고정 색상·전사문 cascade 세 문제를 구체적으로 발견해 변경 요청하고, 대비 개선을 확인한 뒤 승인**했다. 독립 소개 사이트는 **서비스 소개·프로젝트 소개 2개 정적 페이지, 브랜드 에셋/시각 데모, 반응형·다중 테마, 정적 검사와 로컬 HTTP 검증**을 갖춘 개인 커밋 산출물로 보존됐다. 사용자 제공 기획용 화면을 실제 서비스 캡처로, 시험 데이터·보관 정책을 검증된 운영 성과로 소개하지 않도록 구분했다. 별도 개인 GitHub 저장소에도 게시 소스가 존재한다. 이후 곽호준 작성 통합 PR로 소개→로그인(필요 시)→연습 진입 경로가 프런트에 추가됐으나 **원본 소스 제작과 통합/배포는 같은 개인 작업으로 중복 계상하지 않는다**.

### 기여·시점·검증 경계

- **A — 직접 구현/기획:** [#214](https://github.com/aihuman-7th/proj2-3/pull/214) 독립 정적 소개 사이트·기획 목업의 웹 구성·브랜드 원본 자산 적용·공개 전 근거 검증, [개인 showcase 저장소](https://github.com/nanocode00/malhaebom-showcase) 소스.
- **R — 직접 코드 리뷰:** [#211](https://github.com/aihuman-7th/proj2-3/pull/211) CSS 정의/명암/입력 상태 리뷰와 변경 요청·최종 승인. **제품 ThemeProvider/홈·코칭 화면 코어 코드를 김재훈이 단독 구현했다고 서술하지 않음**.
- **T — 팀원 구현:** [#240](https://github.com/aihuman-7th/proj2-3/pull/240)·[#241](https://github.com/aihuman-7th/proj2-3/pull/241)·[#251](https://github.com/aihuman-7th/proj2-3/pull/251) 곽호준 작성 이관·OAuth/루트 동선·현재 서비스 화면 내용 갱신. 원본/이관 자산과 PR/브라우저 테스트를 검증했더라도 **개인 단독 구현이나 10/8 이전 기능으로 소급하지 않음**.
- **목업/운영 구분:** 11페이지 PDF는 소개 사이트의 **디자인 입력물**이지 자체 사용자 효과나 A/B 실험 결과가 아니다. 기획용 3단 코칭 UI를 실제 운영 기능으로 광고하지 않고, demo image는 구버전일 수 있다. 로그인 OAuth 실제 성공·실마이크/STT/운영 UI 배포 상태/Pages 현재 응답은 별도 검증 필요.
- **이력서 활용 판단:** UX 직무 단독 '디자인 전면 리뉴얼'보다 **(1) UI 코드 리뷰로 실제 사용성·대비 버그를 추적해 개선 요구, (2) 11페이지 기획안을 의존성 없는 정적 소개 사이트로 구현, (3) 과장 없는 제품·프로젝트 홍보/증거 구분** 경험으로 표현하는 것이 정확하다.

원본: [테마 리뷰 PR #211](https://github.com/aihuman-7th/proj2-3/pull/211) · [UI 테마/코칭 화면 Spec](https://github.com/aihuman-7th/proj2-3/blob/main/docs/specs/ui-theme-coaching-refresh.md) · [UI 개선 Worklog](https://github.com/aihuman-7th/proj2-3/blob/main/docs/worklogs/2026-10-07_ui-theme-coaching-refresh-design.md) · [개인 소개 사이트 PR #214](https://github.com/aihuman-7th/proj2-3/pull/214) · [사이트 작업 기록](https://github.com/aihuman-7th/proj2-3/blob/main/docs/worklogs/2026-10-08_malhaebom-intro-site.md) · [개인 쇼케이스](https://github.com/nanocode00/malhaebom-showcase) · [10/10 소개 이관](https://github.com/aihuman-7th/proj2-3/blob/main/docs/worklogs/2026-10-10_showcase-intro-integration.md).

---

## 4. 확정된 1차 사실

### MB-04 — 평가 정답 검토

- 작업군 검증 완료. **5단계 서사, 개인 기여, PR #126 및 재채점의 세부 근거는 §3-4 참조.**

### MB-06 — 공고 원문 분석 E2E

- 작업군 검증 완료. **5단계 서사, PR #149 개인 커밋/리뷰, 로컬 web fallback과 후속 팀원 운영 증거 구분은 §3-6 참조.**

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

1. **MB-01~11 개별 근거 조사·5단계 경험 복원 완료 (11/12).** 모델 평가·계약 리뷰·사람 ground truth·인증 E2E·STT 실험은 각 PR 작성자·동료 리뷰·Worklog·실험 조건을 대조했다. AI 코딩 도구·팀원 기여를 개인 단독 구현으로 혼동하지 않는다.
2. **다음 조사: MB-12.** 실제 마이크/STT·서비스 E2E, UI·UX/경쟁사·사용자 개선, PR·CI 운영, 마감·홍보/산출물 등의 정확한 작업군은 위 원장 항목의 명칭·근거를 우선해 추적한다.
3. **12개 작업군 전체 완료 후 누락·중복·귀속 역검증.** 사용자의 UI 제안·평가/검증·PR 운영과 팀원 프론트 구현·운영 배포를 혼동하지 않는다.
4. **사례 선별.** 모델 평가/선정, 신뢰 계약·검증, 사람 ground truth, STT 실험, 인증 E2E 중 직무별 상위 3~5개 선택.
5. **전면 근거 검증 뒤** `experiences/malhaebom.md` 채용 요약을 확정하고 `experiences/README.md`, `career-timeline.md`를 업데이트. 마스터 `resume.md`는 나머지 프로젝트 점검 뒤 변경.

### 보수적으로 유지할 미확인 사항

- UI 전체, Agent 전체, RAG/LangGraph(사용하지 않음), 모든 모델 파인튜닝을 개인 단독 구현한 것으로 서술 금지.
- 계약 100%와 코칭 의미 품질 100%, 실제 사용자 만족도, 운영 전체 E2E 성공을 동의어로 사용 금지.
- C2를 무조건 기존 모든 음성에 우수하거나 production STT로 승격했다고 쓰지 않음.
- 새 프롬프트 v5의 형식 30/30을 품질 개선 성공으로 표현 금지.
- 프로젝트 공식 마감(10/8) 후 PR 병합/소스 동기화를 원래 프로젝트 기간의 새 기능 개발로 중복 계상하지 않음.
