# 말해봄 Mk.2 — 근거 기반 AI 면접 코칭 에이전트

> **경험 복원 1차 초안 (2026-10-21).** 확정된 개인 기여와 팀 전체 구현·실험 결과를 분리했다. 코드·PR 작성 주체를 추가 대조하기 전에는 이 문서의 미확정 항목을 제출용 이력서에 그대로 사용하지 않는다.

## 채용용 경험 요약 — 검증된 범위

신입 개발자의 면접 답변을 녹음·전사한 뒤, 채용공고 문맥 및 사용자가 **실제로 말한 문장 근거**를 연결해 구조화된 피드백과 재답변 비교를 제공하는 3인 팀 AI Agent 프로젝트. React/TypeScript, FastAPI, Supabase, FastMCP, OpenRouter, Langfuse를 사용하는 서비스 및 평가 체계를 구축했다.

- **기간:** 2026-09-18 ~ 2026-10-08 (프로젝트 본 일정). 10월 10~12일 저장소·배포 동기화 및 종료 정리는 별도 후속 작업.
- **팀:** 곽호준·김재훈·서정현, 3인 ([팀 README](https://github.com/aihuman-7th/proj2-3/blob/main/README.md)).
- **개인 기여, 직접 기록 확인:** 합성 평가셋 30건 사람 기준 검토·ground truth 교정; 인증된 채용공고 원문 분석 E2E 재현 및 보안 처리; **Prompt 템플릿 변수 검증 취약점과 MCP/Agent E2E 테스트의 거짓 양성·음성 검출 리뷰, 신뢰 경계 문서화**; 체크포인트/평가 근거 정합화와 PR 협업.
- **개인 기여, 추가 귀속 검증 중:** 다수 모델 후보 평가 실행과 기본 후보 선정, Prompt/Skill 평가 설계·실행, Whisper Medium QLoRA 실험, API/Agent 통합 검증. 프로젝트 산출물이 존재하지만 단계별 커밋·학습 실행 주체를 더 확인해야 한다.
- **프로젝트 기술:** React·TypeScript·Vite, FastAPI·Pydantic, Docker Compose, Supabase, FastMCP, OpenRouter, Langfuse, Whisper/CTranslate2, Cloud Run·Vercel.
- **범위:** 말해봄 Mk.2는 RAG·벡터 DB·LangGraph를 사용하지 않았다. 추후 의료 RAG 프로젝트와 혼동하지 않는다.

### 문제와 설계 기준

취업준비생이 답변에서 본인의 역할·원인·선택·결과 검증을 충분히 설명했는지 판단하기 어렵다는 문제를 대상으로 했다. LLM의 자연어 코칭과 실제 상태 변경을 분리하고, 확인된 답변 revision과 인용·문장 ID·기준 ID를 연결하며, 스키마·Pydantic·서버 validator로 출력의 근거·누락·중복을 검사하는 방식을 취했다. 보류 가능한 상태(`met / partial / not_observed / undetermined`)를 구분해 답변에 없는 행동을 실제로 하지 않았다고 단정하지 않도록 했다.

구현의 **팀 전체 구조**는 `녹음/WAV → STT·VAD → 답변 확인 → 공고·MCP 문맥 → LLM 코칭 → 서버 검증·저장 → 재답변 비교`다. 서버의 권한·동의·저장 계약과 모델의 코칭 텍스트 생성은 분리되어 있으며, 재답변 비교는 호환 가능한 두 결과를 서버에서 계산한다. 이 구조 전체를 개인 단독 구현으로 기술하지 않는다.

근거: [프로젝트 정의](https://github.com/aihuman-7th/proj2-3/blob/main/docs/PROJECT_OVERVIEW.md) · [에이전트 역할·구현 보고서](https://github.com/aihuman-7th/proj2-3/blob/main/docs/AGENT_REPORT.md) · [현재 구현](https://github.com/aihuman-7th/proj2-3/blob/main/docs/CURRENT_STATE.md).

## 핵심 경험 후보

### 1. 평가셋 정답의 사람 검토 및 오답 원인 분리 — 개인 기여 확인

**문제:** LLM 출력의 자동 점수가 높더라도 정답표 자체에 모호한 판정이 섞여 있으면 모델 개선 여부를 잘못 판단할 수 있다.

**판단·조치:** 2026-09-30 Dataset v2의 합성 30건(정상 12·경계 10·실패 유도 8)의 원문·기대 C/A/V 판정과 근거를 직접 검토했다. 29건은 확정하고, 여러 프로젝트의 결과가 섞여 귀속 대상을 특정할 수 없는 `coach-boundary-17`의 결과 검증(V)을 `partial → undetermined`로 수정했다. 모델을 재호출하지 않고 저장된 90개 실행 출력을 새 정답표로 offline 재채점했다.

**결과:** 계약 준수 **90/90은 동일**, 전체 label 일치율 **87.8% → 86.9%**, V 일치율 **85.6% → 82.2%**로 정정했다. 모델이 갑자기 나빠진 것이 아니라 **사람 정답 기준을 바로잡은 효과**임을 분리했다. 단일 검토자라 inter-rater agreement는 측정하지 않았다.

근거: [판정자·근거 30건](https://github.com/aihuman-7th/proj2-3/blob/main/evals/reviewer-assignment.md) · [offline 재채점](https://github.com/aihuman-7th/proj2-3/blob/main/docs/experiments/2026-09-30_d4-boundary17-offline-regrade.md).

### 2. 로그인 사용자 공고 원문 분석 E2E의 실제 재현 — 개인 작업 기록 확인

**문제:** 공고 조회·요구사항 추출·원문 인용 검증 코드가 개별적으로 존재해도 실제 로그인 사용자 권한 아래에서 저장까지 이어지는지 불확실했다. 외부 MCP 인증 요청은 503, 서비스 재기동 뒤 기존 JWT는 401을 반환했다.

**판단·조치:** 브라우저에서 얻은 실제 로그인 세션 JWT로 프런트와 같은 API 순서를 검증하고, 비대화형 E2E runner가 권한 `0600`인 Git ignored 토큰 파일만 허용하도록 보완했다(권한 우선순위·0644 거부 등 단위검사 3건). 당시 PlayMCP 503은 기능 전체의 실패와 구분해 보존된 `web` provider 경로로 전환했고, 새로운 JWT로 테스트했다.

**결과:** `NCS 선택 → 외부 공고 추천 3개 → 원문 조회 → requirement 5개 추출 → quote validation 통과 → 인증 사용자 목록에 저장 → 요구사항 2개 선택 → 연습 세션 생성` 경로 성공을 redacted 기록으로 남겼다. **브라우저 클릭 UI E2E, 운영 배포에서의 동일 검증, PlayMCP 복구를 완료한 것은 아니다.**

근거: [2026-10-01 개인 Worklog](https://github.com/aihuman-7th/proj2-3/blob/main/docs/worklogs/2026-10-01_1400_job-source-authenticated-e2e.md).

### 검증된 추가 사례. AI Agent 계약 및 검증 하네스 리뷰 — 개인 기여 확인

**문제:** Prompt·Skill·MCP·서버 책임을 분리해도, 검증 하네스가 실제 API 계약을 잘못 가정하거나 템플릿의 미지원 변수를 통과시키면 **정상 응답을 실패 처리하거나 저장되지 않은 요청을 성공으로 오인**할 수 있다.

**판단·조치:** [PR #119](https://github.com/aihuman-7th/proj2-3/pull/119)의 코드 리뷰에서 `{{SKILL_TEXT}}`·`{{skill_text }}` 같은 잘못된 템플릿 슬롯이 정규식 검사를 우회해 system prompt에 남는 문제를 지적하고 미지원 변수 거절 및 회귀 검사를 요청했다. [PR #116](https://github.com/aihuman-7th/proj2-3/pull/116)에서는 하네스가 실제 코칭 응답에 없는 필드·헤더를 전제해 정상 응답을 실패로 처리하는 문제와, 반대로 **HTTP 200만 확인하고 실제 DB 저장 성공을 주장하는 문제**를 연속 수정 요청으로 지적했다. [PR #123](https://github.com/aihuman-7th/proj2-3/pull/123)에 사용자 입력·외부 공고·MCP 반환값·LLM 출력의 신뢰 경계 및 서버 검증 원칙을 문서로 정리했다.

**결과:** 템플릿의 불허 변수/재치환을 차단하는 코드·회귀 테스트가 팀 PR에 반영됐고, D6 검증 하네스의 역할은 실제 확인 가능한 **Agent HTTP 계약 확인**으로 바로잡았다. DB persistence 확인은 별도 수동 SQL 증거로 구분해 테스트 결과를 과장하지 않도록 했다. **주요 구현 커밋의 작성자는 팀원(분리 PR #119 곽호준, D6 검증 PR #116 서정현)**이며, 나의 확인된 기여는 **구체적 결함 발견·검토 기준 제시·변경 요청·신뢰 경계 정리**다. PR #119의 백엔드 526개 테스트는 팀의 코드 변경 검증 결과이며 개인 단독 구현 실적이 아니다.

근거: [PR #119 코드 리뷰](https://github.com/aihuman-7th/proj2-3/pull/119#discussion_r4139564132) · [PR #116 수정 요청과 최종 승인](https://github.com/aihuman-7th/proj2-3/pull/116) · [신뢰 경계 문서](https://github.com/aihuman-7th/proj2-3/blob/main/docs/TRUST_BOUNDARY.md).

### 3. LLM 후보 비교에서 형식 안정성·지연·비용 분리 — 프로젝트 실험 확인, 개인 실행 범위 재확인 필요

**문제:** 계약을 충족하는 생성 모델이라도 코칭 지연, 비용, 오류/실패 유도 입력에서의 형식 유지 능력이 다르다.

**실험:** 합성 Dataset v2 30건, 고정 Prompt/Skill/Schema와 모델별 기본 생성 설정에서 Final Four를 재실행했다. 2026-09-21 결과:

| 후보 | 계약 준수 | 완료 지연 p50 / p95 | 알려진 30건 비용 |
| --- | ---: | ---: | ---: |
| Gemini 3.5 Flash Lite | 30/30 | 3.41 / 4.20초 | $0.107065 |
| Gemini 3.8 Flash | 30/30 | 12.60 / 21.28초 | $0.349329 |
| GPT-5.6 Luna | 29/30 | 9.36 / 13.77초 | $0.039149 |
| GPT-5.6 Terra | 10/30 | 1.18 / 8.29초 | **최소** $0.105707 |

Terra의 20건 실패는 request/schema/integration 단계 실패로, 의미 품질 불합격으로 단정하지 않는다. 3건의 로컬 Docker/Supabase/MCP 경로 추가 측정에서 Flash Lite는 **3/3**, Luna는 최초 **1/3**, 제한된 JSON envelope 보정 뒤 **2/3** 통과했다. 응답 지연·계약 안정성을 기준으로 Flash Lite를 코칭 기본 후보로 선정했다. **30건 생성 평가와 3건 서비스 경로 검증은 다른 모집단·조건**이다.

근거: [Final Four](https://github.com/aihuman-7th/proj2-3/blob/main/docs/experiments/2026-09-21_openrouter-final-four.md) · [평가 보고서 13장](https://github.com/aihuman-7th/proj2-3/blob/main/EVAL_REPORT.md).

### 4. Prompt/Skill 변경의 회귀를 측정하고 승격 보류 — 프로젝트 실험 확인, 귀속 재확인 필요

2026-10-02 동일 Gemini Flash Lite·Skill·합성 30건에서 Prompt v4/v5를 비교했다. 양쪽 계약은 **30/30**이었지만 종합 자동 통과는 **22/30 → 20/30**, 기대 intent/C/A/V 일치는 **116/120 → 107/120**으로 후퇴했다. 알려진 provider 비용은 **$0.19638110 → $0.17027067**, p95는 **4.621 → 4.622초**였다. 비용 절감에도 코칭 의미 판정 회귀가 발생해 품질 개선이나 출시 게이트 통과로 판단하지 않았다.

10월 6일의 단일 Skill 규칙 smoke는 계약 양쪽 1/1이었으나 target hit **2/3 → 1/3**으로 악화돼 전체 확대 실행을 중단했다. 같은 날 별도 배포 이미지 기반 30건×3회 반복 평가에서는 계약 **90/90**, 종합 자동 통과 **57/90**이었다. 이 세 실험의 버전·모집단·조건을 합쳐 하나의 개선율로 표시하지 않는다.

근거: [EVAL_REPORT 4·6·7장](https://github.com/aihuman-7th/proj2-3/blob/main/EVAL_REPORT.md) · [AGENT_REPORT](https://github.com/aihuman-7th/proj2-3/blob/main/docs/AGENT_REPORT.md).

### 5. Whisper Medium 추가 학습·도메인 적응 — 프로젝트 실험 확인, 개인 수행 범위 재확인 필요

기존 STT adapter 인계 누락 뒤 50k QLoRA 재학습 → placeholder 제거 B0 → AI Hub 71592 면접 답변 C1/C2 도메인 적응을 진행했다. B0 기준 C2는 replay 20%를 넣어 기존 도메인 망각을 완화하도록 설계했다. 기술 질문 8건의 발화형 정규화 WER은 **20.11% → 11.73%**였으나, 기존 119 validation WER은 **15.33% → 16.81%**로 악화돼 trade-off가 남았다. C2의 별도 면접 final test는 CER **2.5876%**, WER **9.7740%**였다.

위 수치는 프로젝트 실험 결과이며 각각 다른 데이터·평가 조건을 섞어 개선율을 계산하지 않는다. C2가 배포 기본 STT 모델로 교체됐다고 주장하지 않는다. 실험의 개인 주도 구간·학습 명령·checkpoint 귀속은 추가 확인한다.

근거: [STT 파인튜닝 계보](https://github.com/aihuman-7th/proj2-3/blob/main/docs/stt-finetuning-lineage.md).

## 기타 경험과 후속 검증

- 실제 서비스는 운영 배포·소개 페이지, OAuth, 코칭 API, 답변 revision, MCP 조회, Langfuse trace, 이력서 PDF 지원 기능까지 확장됐다. **팀 전체 제품 범위**와 해당 시점의 운영 확인을 먼저 적고 개인 코드 기여를 나중에 연결한다.
- PR·CI 정책, 체크포인트 동기화, 재현성 문서화 및 출시 증거 관리 경험은 개인 Worklog/PR별로 추가 분류한다. 코드 작성자와 AI 코딩 에이전트, PR 승인·병합 주체를 혼동하지 않는다.
- 정확도와 안정성 주장 시 **계약 통과 / 규칙 기반 자동 판정 / 사람 평가 / 인증 사용자 부분 E2E / 운영 전체 E2E**를 분리한다. 최신 모델·프롬프트를 단순히 최종 승격·품질 개선으로 서술하지 않는다.
- 제출용 이력서에 쓸 **개인 주도 핵심 사례 3~5개**는 추가 교차검증 뒤 확정한다.

추가 확인 및 작업 원장: [malhaebom-rework-checkpoint.md](malhaebom-rework-checkpoint.md).

## 근거 기준

- [프로젝트 README](https://github.com/aihuman-7th/proj2-3/blob/main/README.md)
- [EVAL_REPORT.md](https://github.com/aihuman-7th/proj2-3/blob/main/EVAL_REPORT.md) (실험 세대별 조건)
- [AGENT_REPORT.md](https://github.com/aihuman-7th/proj2-3/blob/main/docs/AGENT_REPORT.md) (제품 구조·운영 대조)
- [CURRENT_STATE.md](https://github.com/aihuman-7th/proj2-3/blob/main/docs/CURRENT_STATE.md) (날짜별 구현/배포/검증 범위)
- [evals/reviewer-assignment.md](https://github.com/aihuman-7th/proj2-3/blob/main/evals/reviewer-assignment.md) (개인 검토)
- [인증 사용자 공고 E2E Worklog](https://github.com/aihuman-7th/proj2-3/blob/main/docs/worklogs/2026-10-01_1400_job-source-authenticated-e2e.md) (개인 수행)
