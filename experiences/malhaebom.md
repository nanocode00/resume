# 말해봄 Mk.2 — 근거 기반 AI 면접 코칭 에이전트

> **경험 복원 1차 초안 (2026-10-21).** 확정된 개인 기여와 팀 전체 구현·실험 결과를 분리했다. 코드·PR 작성 주체를 추가 대조하기 전에는 이 문서의 미확정 항목을 제출용 이력서에 그대로 사용하지 않는다.

## 채용용 경험 요약 — 검증된 범위

신입 개발자의 면접 답변을 녹음·전사한 뒤, 채용공고 문맥 및 사용자가 **실제로 말한 문장 근거**를 연결해 구조화된 피드백과 재답변 비교를 제공하는 3인 팀 AI Agent 프로젝트. React/TypeScript, FastAPI, Supabase, FastMCP, OpenRouter, Langfuse를 사용하는 서비스 및 평가 체계를 구축했다.

- **기간:** 2026-09-18 ~ 2026-10-08 (프로젝트 본 일정). 10월 10~12일 저장소·배포 동기화 및 종료 정리는 별도 후속 작업.
- **팀:** 곽호준·김재훈·서정현, 3인 ([팀 README](https://github.com/aihuman-7th/proj2-3/blob/main/README.md)).
- **개인 기여, 직접 기록 확인:** **AI 코딩 도구를 활용한 OpenRouter 모델 40종 사전검증·22종 660건 본평가, 4종 120건 재측정과 코칭 서비스 후보 통합 검증·기본 모델 선정**; 합성 평가셋 30건 사람 기준 검토·ground truth 교정; 인증된 채용공고 원문 분석 E2E 재현 및 보안 처리; **Prompt 변수 검증 결함 및 MCP/Agent E2E 테스트의 거짓 양성·음성 검출 리뷰, 신뢰 경계 문서화**.
- **개인 기여, 추가 귀속 검증 중:** Prompt/Skill A/B 평가 설계·실행, Whisper Medium QLoRA 실험, 기타 API/Agent 통합 검증. 프로젝트 산출물은 존재하지만 단계별 커밋·학습 주체를 계속 대조한다. **초기 기본 모델 평가 하네스와 일부 별도 모델 평가 확장은 팀원 기여**로 구분한다.
- **프로젝트 기술:** React·TypeScript·Vite, FastAPI·Pydantic, Docker Compose, Supabase, FastMCP, OpenRouter, Langfuse, Whisper/CTranslate2, Cloud Run·Vercel.
- **범위:** 말해봄 Mk.2는 RAG·벡터 DB·LangGraph를 사용하지 않았다. 추후 의료 RAG 프로젝트와 혼동하지 않는다.

### 문제와 설계 기준

취업준비생이 답변에서 본인의 역할·원인·선택·결과 검증을 충분히 설명했는지 판단하기 어렵다는 문제를 대상으로 했다. LLM의 자연어 코칭과 실제 상태 변경을 분리하고, 확인된 답변 revision과 인용·문장 ID·기준 ID를 연결하며, 스키마·Pydantic·서버 validator로 출력의 근거·누락·중복을 검사하는 방식을 취했다. 보류 가능한 상태(`met / partial / not_observed / undetermined`)를 구분해 답변에 없는 행동을 실제로 하지 않았다고 단정하지 않도록 했다.

구현의 **팀 전체 구조**는 `녹음/WAV → STT·VAD → 답변 확인 → 공고·MCP 문맥 → LLM 코칭 → 서버 검증·저장 → 재답변 비교`다. 서버의 권한·동의·저장 계약과 모델의 코칭 텍스트 생성은 분리되어 있으며, 재답변 비교는 호환 가능한 두 결과를 서버에서 계산한다. 이 구조 전체를 개인 단독 구현으로 기술하지 않는다.

근거: [프로젝트 정의](https://github.com/aihuman-7th/proj2-3/blob/main/docs/PROJECT_OVERVIEW.md) · [에이전트 역할·구현 보고서](https://github.com/aihuman-7th/proj2-3/blob/main/docs/AGENT_REPORT.md) · [현재 구현](https://github.com/aihuman-7th/proj2-3/blob/main/docs/CURRENT_STATE.md).

## 핵심 경험 후보

### 1. Ground Truth 30건 직접 검토와 재채점 기준 교정 — 개인 기여 확인

**상황:** 말해봄에는 정상 12건·경계 10건·실패 유도 8건의 합성 코칭 평가셋이 있었지만, 기대 판정에 사례별 실제 사람 검토자·근거가 확정되지 않아 자동 평가 수치를 그대로 신뢰하기 어려웠다.

**문제:** `coach-boundary-17`은 여러 프로젝트의 캐시·로그·인덱스 조치와 성능 개선 주장이 혼재해 있었다. 기존 결과 확인(V)의 정답 `partial`은 **어느 조치의 성능이 개선됐는지 특정할 수 없다는 오류**를 제대로 반영하지 못했다.

**판단:** 결과 자체가 없는 `not_observed`와 결과의 **귀속 대상이 불명확한 `undetermined`**를 구분해야 한다고 판단했다. Ground Truth 교정 후에는 모델을 새로 실행하지 않고 **동일한 저장 출력만 재채점**해야 정답 수정 효과와 모델 변화가 섞이지 않는다.

**조치:** 2026-09-30 평가셋 **30건의 원문·기대 C/A/V·판정 근거를 직접 검토**하고 29건은 유지, `coach-boundary-17`의 V를 **`partial → undetermined`**로 수정했다. 문항별 판정자·근거를 기록하고 V 판단 원인을 7종 보조 분류로 구분해, 원래 모델 출력 계약을 변경하지 않고 사람 검토에서 오류 원인을 분석할 수 있게 했다. 이후 9/22 Gemini Flash-Lite·Prompt v4.1의 **30건×3회 저장 출력 90개**를 provider 재호출 없이 오프라인 재채점했다.

**결과:** 자동 label 일치율이 **316/360(87.8%) → 313/360(86.9%)**, V 일치율이 **77/90(85.6%) → 74/90(82.2%)**로 바로잡혔다. 출력 계약 **90/90**, 근거 일치율 **96%**는 변하지 않았다. **성능을 떨어뜨린 실험이 아니라 잘못된 기대 정답으로 인한 과대평가를 교정한 결과**다. 한 명의 사람 검토자만 참여했으므로 평가자 간 일치도는 측정하지 않았다.

**개인 기여 근거:** [PR #126 — 병합 완료](https://github.com/aihuman-7th/proj2-3/pull/126)의 두 커밋은 `nanocode00` 작성자이며, 동료가 수정한 기대 행렬·재채점 증거와 8개 D4 테스트를 검토한 뒤 승인했다. 선행 [PR #125](https://github.com/aihuman-7th/proj2-3/pull/125)는 병합되지 않았고, [PR #123](https://github.com/aihuman-7th/proj2-3/pull/123)의 중복 D4 커밋은 되돌려 **최종 반영 경로를 #126으로 정리**했다. 모델 출력 90개를 사람이 전수 검토한 것은 아니며 **30개 사례의 Ground Truth를 사람 검토**하고 저장 출력 90개를 자동 재채점한 것이다.

근거: [30건 사람 검토표](https://github.com/aihuman-7th/proj2-3/blob/main/evals/reviewer-assignment.md) · [9/30 작업 기록](https://github.com/aihuman-7th/proj2-3/blob/main/docs/worklogs/2026-09-30_d4-human-review.md) · [offline 재채점](https://github.com/aihuman-7th/proj2-3/blob/main/docs/experiments/2026-09-30_d4-boundary17-offline-regrade.md).

### 2. 로그인 사용자 공고 원문 분석 E2E 검증 — 개인 수행 확인

**상황:** 말해봄에는 공고 원문 조회·요구사항 추출·서버 인용 검증·로그인 사용자 공고 저장 기능이 있었으나, 강사 피드백의 **공고 선택부터 연습 세션 생성까지** 실제 인증된 사용자 문맥에서 연속 통과한 근거가 없었다.

**문제:** 비대화형 검증 도구에 로그인 JWT를 안전하게 전달할 방법이 부족했고, 실행 중 초기 NCS 요청과 PlayMCP 인증에서 각각 **HTTP 503**, API 재생성 후 기존 JWT에서 **401**이 발생했다. 개별 provider 인증 장애를 전체 공고 분석 기능 결함으로 오인할 위험도 있었다.

**판단:** **PlayMCP 자체 인증과 공고 원문 분석의 기능적 E2E를 별도 검증**하기로 했다. 브라우저에서 발급된 실제 로그인 JWT를 사용하되 프런트와 동일한 API 흐름을 자동 실행하고, PlayMCP 503 시 서버가 이미 제공하는 `web` 검색 provider 복귀 경로를 통해 원문 인용·저장·세션 생성 기능을 확인한다. 토큰과 외부 공고 원문은 저장소 기록에 남기지 않는다.

**조치:** [PR #149](https://github.com/aihuman-7th/proj2-3/pull/149)에서 인증 E2E runner에 **환경변수 우선·Git ignored JWT 토큰 파일 fallback·POSIX 0600 권한 검사**를 도입하고, 정상 파일·환경변수 우선·과도한 권한(0644) 거부 **3개 단위 테스트**를 추가했다. NCS 장애 시 재시도하고, PlayMCP 인증 503 시 `JOB_SEARCH_PROVIDER=web`으로 전환했으며, 컨테이너/API 재생성 후에는 로그인 브라우저의 최신 JWT를 사용했다. 결과에는 인증 상태·건수·검증 통과 여부만 기록했다.

**결과:** 실제 **로컬 Supabase 로그인 사용자 JWT**로 **NCS `skill-server` 선택 → 공고 추천 3건 → 사람인 공고 원문 재조회 → 요구사항 5건 추출·원문 quote 검증 통과 → 개인 `private_job` 저장·목록 조회 → 요구사항 2개 선택 → `preset-04` 연습 세션 생성**을 **로컬 API E2E 1회** 성공했다. 해당 PR은 **JaeHun Kim/nanocode00 작성, 서정현 승인·CI 통과 후 병합**됐다. 이 경로는 PlayMCP **직접 성공이 아닌 web fallback 성공**이며 운영·브라우저 클릭 전체 검증도 아니다.

**후속 작업 및 한계:** [PR #167](https://github.com/aihuman-7th/proj2-3/pull/167)의 운영 HTTP 로그 증거 재정리는 **곽호준 기여**이고 새 연속 E2E 실행이 아니며, [PR #186](https://github.com/aihuman-7th/proj2-3/pull/186)의 Windows 토큰 파일 fallback 금지는 **서정현 구현·내 리뷰**다. 이후 실제 운영 브라우저에서 공고 분석과 연습 기능 성공을 확인한 수동 보고는 있으나 **같은 요청의 PlayMCP 호출 로그와 연계된 증거는 별도**다. 내 검증 결과를 제품 전체·모든 provider·운영 사용자 E2E 성공률로 확대하지 않는다.

근거: [PR #149](https://github.com/aihuman-7th/proj2-3/pull/149) · [로컬 E2E 증거](https://github.com/aihuman-7th/proj2-3/blob/main/docs/evidence/job-source-analysis-local-e2e-2026-10-01.md) · [개인 Worklog](https://github.com/aihuman-7th/proj2-3/blob/main/docs/worklogs/2026-10-01_1400_job-source-authenticated-e2e.md).

### 검증된 추가 사례. AI Agent 계약 및 검증 하네스 리뷰 — 개인 기여 확인

**상황:** 말해봄은 Prompt의 역할·출력 계약, Skill의 면접 판정 규칙, MCP의 제한된 문맥 조회, 서버의 권한·출력 검증을 분리해 운영하는 구조로 전환 중이었다. 주요 분리·하네스 코드는 팀원들이 구현했다.

**문제:** Prompt·Skill·MCP·서버 책임을 분리해도, 검증 하네스가 실제 API 계약을 잘못 가정하거나 템플릿의 미지원 변수를 통과시키면 **정상 응답을 실패 처리하거나 저장되지 않은 요청을 성공으로 오인**할 수 있다.

**판단·조치:** [PR #119](https://github.com/aihuman-7th/proj2-3/pull/119)의 코드 리뷰에서 `{{SKILL_TEXT}}`·`{{skill_text }}` 같은 잘못된 템플릿 슬롯이 정규식 검사를 우회해 system prompt에 남는 문제를 지적하고 미지원 변수 거절 및 회귀 검사를 요청했다. [PR #116](https://github.com/aihuman-7th/proj2-3/pull/116)에서는 하네스가 실제 코칭 응답에 없는 필드·헤더를 전제해 정상 응답을 실패로 처리하는 문제와, 반대로 **HTTP 200만 확인하고 실제 DB 저장 성공을 주장하는 문제**를 연속 수정 요청으로 지적했다. [PR #123](https://github.com/aihuman-7th/proj2-3/pull/123)에 사용자 입력·외부 공고·MCP 반환값·LLM 출력의 신뢰 경계 및 서버 검증 원칙을 문서로 정리했다.

**결과:** 템플릿의 불허 변수/재치환을 차단하는 코드·회귀 테스트가 팀 PR에 반영됐고, D6 검증 하네스의 역할은 실제 확인 가능한 **Agent HTTP 계약 확인**으로 바로잡았다. DB persistence 확인은 별도 수동 SQL 증거로 구분해 테스트 결과를 과장하지 않도록 했다. **주요 구현 커밋의 작성자는 팀원(분리 PR #119 곽호준, D6 검증 PR #116 서정현)**이며, 나의 확인된 기여는 **구체적 결함 발견·검토 기준 제시·변경 요청·신뢰 경계 정리**다. PR #119의 백엔드 526개 테스트는 팀의 코드 변경 검증 결과이며 개인 단독 구현 실적이 아니다.

근거: [PR #119 코드 리뷰](https://github.com/aihuman-7th/proj2-3/pull/119#discussion_r4139564132) · [PR #116 수정 요청과 최종 승인](https://github.com/aihuman-7th/proj2-3/pull/116) · [신뢰 경계 문서](https://github.com/aihuman-7th/proj2-3/blob/main/docs/TRUST_BOUNDARY.md).

### 3. OpenRouter 후보 대규모 탐색부터 서비스 기본 모델 선정까지 — 개인 기여 확인

**상황:** 기존 팀 평가 하네스와 제한된 모델 비교 기록은 있었지만, 모델 계열 다양성부터 실제 코칭 서비스 경로까지 이어지는 기본 모델 선정 근거가 필요했다.

**문제:** 계약을 충족하는 생성 모델이라도 코칭 지연, 비용, 오류/실패 유도 입력에서의 형식 유지 능력이 다르다.

**판단:** 모델의 코칭 가능성을 하나의 계약 통과율이나 최저 비용으로 결정하지 않고, **접근 가능성·구조화 출력 계약·p50/p95·비용·실제 서비스 통합**을 단계적으로 검증하기로 했다. 모델 출력의 형식 안정성과 면접 코칭 의미 품질은 서로 다른 판단 대상이다.

**조치:** 기존 팀 평가 하네스와 Dataset v2 합성 30건을 재사용해 OpenRouter 제공 모델을 discovery → 3건 preflight → 통과 모델만 30건 평가로 단계화했다. **40개 모델을 preflight하고 22개 통과 모델을 총 660건 평가**, 이후 별도의 고정 Prompt/Skill/Schema·모델 기본 생성 설정으로 Final Four 4×30=120건을 재실행했다. API 호출 재개/결과 저장·비용 누락 보존·dry-run 기본값·재현 경로를 구축했다. 2026-09-21 Final Four 결과:

| 후보 | 계약 준수 | 완료 지연 p50 / p95 | 알려진 30건 비용 |
| --- | ---: | ---: | ---: |
| Gemini 3.5 Flash Lite | 30/30 | 3.41 / 4.20초 | $0.107065 |
| Gemini 3.8 Flash | 30/30 | 12.60 / 21.28초 | $0.349329 |
| GPT-5.6 Luna | 29/30 | 9.36 / 13.77초 | $0.039149 |
| GPT-5.6 Terra | 10/30 | 1.18 / 8.29초 | **최소** $0.105707 |

**결과:** 서비스 통합 검증 및 후보 선정 과정에서는 Terra의 20건 실패는 request/schema/integration 단계 실패로 의미 품질 불합격으로 단정하지 않았다. FastAPI 독립 하네스 3건씩에서는 Flash Lite·Luna 모두 계약 3/3이었지만, **Docker+로컬 Supabase·stdio MCP·DB 통합 경로**에서는 Flash Lite **3/3(p50 3.44초)**, Luna 최초 **1/3** 및 JSON code fence 보정 후 **2/3**을 기록했다. 최초 `temperature=0`과 OpenRouter 지원 설정의 충돌로 발생한 HTTP 404를 모델 기본 설정에 맞춰 해결하고, 보수적인 JSON envelope 복구만 허용했다. **Flash Lite를 기본 코칭 후보로 결정해 `.env.example`·README에 반영**했다. 팀원 PR 리뷰에서 Luna 두 실행의 3/3·2/3 혼용과 환경 기본값 불일치를 지적받아 재현 기록·설정을 보완했다. **계약 30/30은 코칭 의미 품질 100%를 뜻하지 않고**, 3건 서비스 결과는 전체 사용자 품질 보증이 아니다.

**기여 귀속:** 다양성 평가 core·CLI [PR #36](https://github.com/aihuman-7th/proj2-3/pull/36)·[#37](https://github.com/aihuman-7th/proj2-3/pull/37), 재현/선정 문서 [#40](https://github.com/aihuman-7th/proj2-3/pull/40)·[#41](https://github.com/aihuman-7th/proj2-3/pull/41), 후보 통합 [#47](https://github.com/aihuman-7th/proj2-3/pull/47)은 모두 `nanocode00` 커밋이다. **Codex·Antigravity를 통해 구현/실행한 부분**과 내 실험 운영·선정 판단을 구분하며, 초기 하네스 [#19](https://github.com/aihuman-7th/proj2-3/pull/19)는 서정현의 팀원 구현으로 인정한다.

근거: [다양성 실험](https://github.com/aihuman-7th/proj2-3/blob/main/docs/experiments/2026-09-21_openrouter-model-diversity.md) · [Final Four](https://github.com/aihuman-7th/proj2-3/blob/main/docs/experiments/2026-09-21_openrouter-final-four.md) · [서비스 통합](https://github.com/aihuman-7th/proj2-3/blob/main/docs/experiments/2026-09-21_service-candidate-integration.md) · [평가 총괄](https://github.com/aihuman-7th/proj2-3/blob/main/EVAL_REPORT.md).

### 4. Prompt/Skill A/B 평가의 회귀 검출과 확대 실험 중단 — 개인 기여·팀 역할 구분 완료

**상황:** 말해봄은 같은 코칭 모델에서도 Prompt·Skill 수정으로 인한 판정 변화, 서버 출력 계약 안정성, 비용·지연을 각각 검증해야 했다. 기존 Prompt v4→v5 30건 페어 실험과 단일 Skill 규칙 smoke, 별도 D11 인젝션 3건은 목적과 실행 조건이 달랐다.

**문제:** JSON 계약이 유효하더라도 개인 행동·결과 검증 판정이 회귀할 수 있었다. 특히 단일 Skill 규칙 A/B의 첫 구현은 특정 `problem_context`만 개선 대상으로 보거나 C/A/V 전체 합산 점수만 비교해 **다른 기준의 개선·회귀를 놓칠 위험**이 있었다.

**판단:** 모델을 개선했다고 주장하려면 **계약·C/A/V 개별 정답·전체 자동 통과·비용·지연을 함께 보며**, 한 target 기준이 좋아져도 다른 기준의 퇴행이 있으면 보류해야 한다고 판단했다. 목표 사례 smoke에서 회귀가 나오면 전체 평가를 확대하지 않고, 단일 검토자 인젝션 실험과 불균등한 재시도의 성공을 방어 효과로 일반화하지 않는다.

**조치:** 동료 구현의 영문 Prompt v4/v5 30건 비교 [PR #155](https://github.com/aihuman-7th/proj2-3/pull/155)를 코드·실험 조건·보고서와 CI 관점에서 검토했다(원시 출력의 독립 재계산은 미수행). 개인 주도 [PR #185](https://github.com/aihuman-7th/proj2-3/pull/185)에서는 `coach-boundary-19`를 대상으로 **Skill 한 문장만 교체하는 A/B runner**와 고정 조건 검사를 준비하고, 동료 수정 요청을 반영해 **전체 target C/A/V 집계 및 기준별 무회귀 gate**를 추가했다. [PR #189](https://github.com/aihuman-7th/proj2-3/pull/189)에 실제 1건 smoke 결과를 기록했고, [PR #215](https://github.com/aihuman-7th/proj2-3/pull/215)에 신규 인젝션 3건의 Prompt v4/v5 비교와 추가 1회씩의 재시도 결과를 분리 보존했다.

**결과:** 
- **10/2 Prompt v4/v5 (팀원 실험, 개인 리뷰):** 계약 양쪽 **30/30**이지만 종합 자동 통과 **22/30→20/30**, C/A/V·intent 판정 **116/120→107/120**. 호출 비용은 약 **13.3% 감소**했으나 품질 개선 승격은 보류했다. 실제 구현/호출은 **곽호준** 작업이므로 개인 단독 실행 성과가 아니다.
- **10/6 개인 Skill A/B:** baseline/candidate 계약 **각 1/1**이어도 목표 C/A/V 일치는 **2/3→1/3**, label accuracy **75%→50%**. 자동 게이트 `DISCARD_CANDIDATE`가 작동해 **전체 30건×2회 확대 호출은 중단**했다. 최종 사람 검토를 거친 확정 폐기와는 구분한다.
- **10/8 신규 인젝션 잠정 A/B:** 실패 건 추가 호출을 포함한 **사례별 마지막 시도 기준** v4/v5 계약 **3/3 대 3/3**, 기대 판정 전체 일치 **3/3 대 2/3**. 단, v5 **첫 호출 계약은 1/3**이었고 조건별 재시도 횟수·타임아웃과 독립 사람 검토가 달라 방어 개선·출시 통과로 주장하지 않는다.

**개인 기여 귀속:** [PR #185](https://github.com/aihuman-7th/proj2-3/pull/185)의 단일 Skill A/B 준비 및 판정 게이트 개선 커밋, [#189](https://github.com/aihuman-7th/proj2-3/pull/189)의 회귀 실측 기록, [#215](https://github.com/aihuman-7th/proj2-3/pull/215)의 추가 인젝션 실험 증거는 `nanocode00` 작성으로 확인됐다. **실험의 보수적 판정과 실패 시 확대 중단**이 핵심 경험이다. 10/6 배포 이미지 별도 30×3 반복 결과(계약 90/90, 자동 통과 57/90)는 이 A/B 개선율과 합치지 않는다.

근거: [10/2 비교](https://github.com/aihuman-7th/proj2-3/blob/main/docs/experiments/2026-10-02_latest-english-v5-comparison.md) · [10/6 단일 규칙 기록](https://github.com/aihuman-7th/proj2-3/blob/main/docs/worklogs/2026-10-06_single-skill-rule-offtopic-cav.md) · [10/8 잠정 인젝션 A/B](https://github.com/aihuman-7th/proj2-3/blob/main/docs/experiments/2026-10-08_d11-injection-prompt-ab-provisional.md).

### 5. MCP·Agent·DB 통합 검증과 저장 성공 거짓 양성 제거 — 개인 기여 확인

**상황:** 말해봄은 `/api/agent`에서 로그인 사용자 문맥을 FastMCP로 조회하고, LLM 출력의 스키마·인용·ID 계약을 검증한 뒤 `coaching_runs`에 저장하는 팀 공통 Agent 경로를 구축했다.

**문제:** **단위 테스트의 성공만으로 실제 인증 사용자 흐름·실제 제공자 호출·새 DB 저장까지 성공했다고 볼 수 없었다.** MCP gateway 실패, 유효한 `needs_more_info` 상태의 오판정, 모델 출력 검증 후 로컬 Supabase migration 누락에 의한 저장 실패가 발견됐다. 더 위험하게는 DB 저장 검증기가 이번 요청의 insert가 없어도 **과거의 같은 답변 행을 읽고 성공으로 판정할 수 있었다.**

**판단:** fixture 기반 MCP 권한·도구 호출, 로컬 인증 사용자 Agent E2E, 실제 provider/Langfuse trace, **이번 요청으로 생성된 DB 행 검증**을 서로 다른 증거로 다뤘다. `needs_more_info`는 설명 근거 부족을 정확히 알리는 정상 결과일 수 있으므로 단일 `ok` 강제 기준을 고쳐야 한다. 저장 검증은 HTTP 200이나 기존 저장 행 존재가 아니라 **POST 전·후 행 ID 변화, revision/snapshot 일치, API terminal status와 DB status 일치**로 확인해야 한다고 판단했다.

**조치:** [PR #112](https://github.com/aihuman-7th/proj2-3/pull/112)의 개인 커밋으로 `/api/agent → 실제 stdio FastMCP 도구 3개` 및 HTTP `/mcp`의 scope 밖 ID 차단·gateway 오류 처리 통합 테스트를 추가했다. [#114](https://github.com/aihuman-7th/proj2-3/pull/114)에는 로그인 사용자 WAV/STT→Agent→MCP→provider→검증·저장·Langfuse 흐름과 통제된 통신 오류·재시도 trace를 정리했다. [#115](https://github.com/aihuman-7th/proj2-3/pull/115)에서는 실제 로컬 로그인 UI에서 정상·경계·인젝션 3종을 검증하고, **`needs_more_info`를 허용하는 case별 상태 판정·회귀 테스트**를 수정했다. [#118](https://github.com/aihuman-7th/proj2-3/pull/118)에서는 인증 테스트용 analysis job을 명시 선택해 합성 revision을 추가하는 자동 E2E runner를 작성했다. **팀원 작성 [#122](https://github.com/aihuman-7th/proj2-3/pull/122)의 리뷰에서는 기존 행을 현재 요청의 저장 성공으로 오인하는 결함과 DB 상태 불일치를 지적해 변경을 요청**했고, 요청 전 마지막 ID와 요청 후 새 ID를 비교하는 수정·회귀 테스트 3건을 재검토해 승인했다.

**결과:** 로컬 통합 검증에서 **HTTP MCP 11개·Coaching runtime 24개·Agent API 12개 테스트**가 통과했다. 로그인 사용자의 정상·경계·인젝션 테스트 모두 상황에 적절한 `needs_more_info`를 반환했고, **1차 답변 저장·코칭 → 재답변 저장·코칭 → 비교 화면**까지 실제 로컬 흐름을 확인했다. 실제 Langfuse에서는 network 오류 후 **재시도 1회와 최종 실패**, 별도 실제 provider 성공 이후 **DB migration 누락 오류**를 구분했고, 로컬 migration 반영 후 성공 trace를 기록했다. **과거 DB 행을 이용해 저장 누락을 정상으로 판정하는 verifier 결함도 후속 코드에서 차단했다.**

**기여 경계:** Agent/MCP의 핵심 도구·runtime·DB 스키마 전체를 혼자 구현한 것은 아니다. 주요 코어는 **곽호준 등 팀원 소유**이며, D6 저장 verifier 본체와 후속 MCP 오류 분리 구현은 **서정현 소유**다. 내 강한 기여는 **MCP 통합·사용자 흐름 검증, 검증 상태 정책 보정, false-positive 결함 리뷰와 재검증**이다. 실사용 전체 서비스 성공률이나 배포된 동일 요청의 MCP→DB end-to-end 연계를 측정한 결과는 아니다.

근거: [PR #112](https://github.com/aihuman-7th/proj2-3/pull/112) · [PR #114](https://github.com/aihuman-7th/proj2-3/pull/114) · [PR #115](https://github.com/aihuman-7th/proj2-3/pull/115) · [PR #118](https://github.com/aihuman-7th/proj2-3/pull/118) · [DB 검증 리뷰 #122](https://github.com/aihuman-7th/proj2-3/pull/122) · [D6 실제 사용자 기록](https://github.com/aihuman-7th/proj2-3/blob/main/docs/worklogs/2026-09-29_1632_d6-agent-live-e2e.md).

### 6. Whisper Medium QLoRA 재학습·B0 정제·C2 면접 도메인 적응 — 개인 주도 실험 확인

**상황:** 1차 프로젝트의 경상도 음성 약 6,000개 LoRA 적응 모델은 CTranslate2 추론 모델만 남고 후속 학습용 adapter가 인계되지 않았다. 말해봄 STT의 신규 학습·평가를 진행하려면 원본 `openai/whisper-medium`에서 별도 재학습 계보를 만들어야 했다.

**문제:** 기존 quick-eval에는 신규 train과 **181개 중복 segment**가 있어 검증 데이터 누수 위험이 있었고, 50k 학습 label 일부는 `#이름#`·회사명 placeholder가 음성과 맞지 않았다. 또 새 AI Hub 71592 면접 답변은 긴 음성을 분할해야 했으며, 해당 데이터만 추가 학습하면 기존 AI Hub 119 발화의 인식력이 떨어지는 **망각 위험**이 있었다.

**판단:** legacy CT2를 복원된 adapter로 오인하지 않고 **독립 Whisper Medium QLoRA 재학습**, **source-disjoint 고정 holdout**, placeholder 제거 후 **별도 Test B**, 면접 validation과 기존 도메인 retention을 나누어 검증하는 설계를 선택했다. 기존 도메인 망각을 줄일 목적으로 71592 단독 C1과 **71592 80% + 기존 clean 데이터 replay 20%**의 C2를 동일 조건에서 비교하고, 최종 test를 열기 전 모델을 고정하도록 했다. 실험 후보 선정은 운영 STT 배포 결정과 분리했다.

**조치:** 개인 작성 [PR #23~34](https://github.com/aihuman-7th/proj2-3/pull/34)에서 데이터 검증·고정 holdout·clean manifest·Whisper QLoRA 학습기·CT2 evaluator 및 장시간 GPU 실행 도구를 구성하고, **50,000 train / 5,000 validation, NF4 4-bit, LoRA q/v rank16, effective batch32, 2 epoch / 3,126 optimizer step**의 50k 재학습을 수행했다. 데이터 정제로 **48,000 train / 4,802 validation B0**를 구성했다. 이어 [PR #51~56](https://github.com/aihuman-7th/proj2-3/pull/56)에서 B0 타임스탬프 정렬을 활용해 공식 전사문을 유지한 AI Hub 71592 장문 면접 답변을 분할, **24,556개 면접 segment + 6,139개 기존 도메인 replay**로 C1/C2 비교를 수행했다. 리뷰를 통해 재귀 삭제 범위를 제한하고 **C2 선택 adapter SHA·모델 fingerprint·validation·retention 결과를 저장한 frozen selection artifact**가 봉인 테스트보다 먼저 검증되게 보강했다. [PR #69~71](https://github.com/aihuman-7th/proj2-3/pull/71)에서 **1인 녹음 30건 raw/발화형 정규화 지표와 3모델×3 beam 평가**를 수행하고 평가 데이터·모델 fingerprint를 고정했다.

**결과:** 50k QLoRA의 고정 holdout 500건 CER은 **9.1435→8.2708%**로 줄었으나 WER은 **16.1931→16.2937%**로 조금 악화됐다. 독립 Test B 500건에서 50k→B0 정제 후 CER **5.6038→5.3738%**, WER **13.5489→13.1213%**로 개선됐다. 71592 validation에서 C1·C2 결과는 사실상 유사했지만, 기존 119 retention WER은 **C1 17.1378%, C2 16.8062%**로 C2가 망각을 덜 일으켜 **상대적 실험 후보 C2를 선택**했다(기존 B0는 **15.3344%**, 여전히 더 좋음). 선택 이후 봉인된 71592 최종 test **1,233건에서 C2 CER 2.5876%, WER 9.7740%**를 보고했다. 단, 같은 test의 B0 값은 없어 직접 개선율을 산출할 수 없다.

**실사용 및 운영 후보 검증:** 1인 녹음 전체 30건 raw에서 B0→C2의 CER은 **16.03→15.14%** 개선됐지만 WER은 **19.59→20.91%**로 악화됐다. 숫자·영문 stress를 제외한 24건의 발화형 정규화 WER은 **11.90→10.71%**, 기술 질문 8건은 **20.11→11.73%**였다. 별도 모델×beam 9조건에서 **large-v3-turbo beam1**이 동일 녹음의 core 정규화 CER/WER **3.29/6.63%**, RTF **0.020**으로 우세해 다음 서비스 추론 실험 기준선으로 삼았다. 이 30건은 단일 화자 대본 낭독·모델 선택용 devset이므로 다화자/운영 정확도로 일반화하지 않았다.

**개인 기여와 한계:** GitHub의 학습·평가 PR 작성자는 **nanocode00**, STT 원본 Worklog는 **김재훈/ChatGPT 작업**으로 확인됐다. 재학습 계보 구성과 도메인 적응/재현성 검증은 개인 주도 경험으로 볼 수 있다. 그러나 기존 legacy 6k는 같은 adapter의 연속 학습이 아니고, C2가 기존 성능을 완전히 보존하지도 않았으며 **C2가 운영 기본 STT로 승격됐다고 주장하지 않는다.** 모델 학습과 large-v3-turbo의 beam/prompt/runtime 추론 최적화도 구분한다.

근거: [학습·선정 계보](https://github.com/aihuman-7th/proj2-3/blob/main/docs/stt-finetuning-lineage.md) · [C1/C2 실험](https://github.com/aihuman-7th/proj2-3/blob/main/docs/experiments/stt/2026-09-22_aihub71592-domain-adaptation.md) · [실사용 평가](https://github.com/aihuman-7th/proj2-3/blob/main/docs/experiments/stt/2026-09-27_real-user-recording-eval.md) · [model×beam 평가](https://github.com/aihuman-7th/proj2-3/blob/main/docs/experiments/stt/2026-09-28_real-user-model-beam-matrix.md).

### 7. 인증 사용자 WAV/STT·코칭·재답변 E2E와 STT 서비스 지연 측정 — 개인 기여 확인

**상황:** 말해봄은 음성 답변을 비동기 업로드·STT/VAD로 전사한 뒤 사용자가 원문을 확정하고, LLM Agent로 코칭을 받아 다시 답변하고 비교하는 서비스였다. 개별 구성요소 테스트만으로는 **로그인 사용자 관점의 연속 실행과 사용자 체감 대기시간**을 보장할 수 없었다.

**문제:** Docker와 로컬 Supabase 인증·Storage·migration이 서로 맞지 않으면 **LLM 출력만 성공해도 최종 DB 저장은 실패**할 수 있었다. 실제 답변의 `needs_more_info`를 HTTP 실패로 해석하거나, 재답변 후 비교 화면만 보고 음성 개선을 주장할 위험도 있었다. STT 단일 추론 RTF와 **업로드부터 analysis job terminal까지의 전체 지연**도 달랐다.

**판단:** 실제 사용자 JWT로 **WAV 업로드 → 분석 완료 → 전사문 확정 → 코칭·DB 저장 → 재답변 비교**의 기능 경로를 검증하는 작업과, **업로드·VAD·STT·저장·polling 전체 지연을 분리 측정하는 실험**을 각각 수행해야 한다고 판단했다. 1·2차에 같은 WAV를 사용했으므로 **내용 코칭 판정 차이만 확인**하고 전달력 개선 효과는 주장하지 않는다. Cloud Run 임시 benchmark의 mock/local 저장소를 Hosted Supabase 운영 E2E로 취급하지 않는다.

**조치:** 사용자 계정 [PR #47](https://github.com/aihuman-7th/proj2-3/pull/47)과 Antigravity 협업 실험에서 곽호준 작성 로컬 인증 수정 [#42](https://github.com/aihuman-7th/proj2-3/pull/42)를 반영해 Docker `Frontend:18080 / API:8000`와 로컬 Supabase에서 **7초 음성 업로드·STT/VAD → 확정 revision → 실제 MCP/LLM 코칭 → `coaching_runs` 저장** 경로를 검증했다. 개인 [PR #115](https://github.com/aihuman-7th/proj2-3/pull/115)에서는 로컬 로그인 UI의 **정상·경계·인젝션** 세 사례에 대한 전사 확정→코칭 결과를 확인하고, 부족한 개인 행동·선택 이유·공고 연결을 보완한 **2차 답변 저장 및 1차/2차 내용 비교 카드**를 점검했다. 개인 [PR #104](https://github.com/aihuman-7th/proj2-3/pull/104)에는 인증 `/analysis-jobs` 파일 업로드→202→terminal polling/RTF·단계별 VAD/STT/hint/persist timing을 계측하는 benchmark를 구현해 로컬 CPU와 Cloud Run CPU를 비교했다. [#106](https://github.com/aihuman-7th/proj2-3/pull/106)에는 CPU-only PyTorch·사전 포함된 Whisper 모델의 별도 컨테이너와 **고정 모델 revision/image label**을 추가했다.

**결과:** 로컬 인증 사용자 경로에서 공고·세션→WAV→전사 확정→Agent·DB 저장을 통과했고, 다른 실험에서 정상·경계·인젝션 **3종 코칭 및 재답변 비교**를 확인했다. 2차 답변에서 개인 행동·선택 이유 기준은 1차보다 충족으로 나타났지만, **동일 WAV를 사용해 전달력 속도 319음절/분·긴 침묵 0·반복 0**은 동일했다. 별도 로컬 Docker CPU/int8 **10회 측정**에서 전체 분석 E2E **p50 3.670초/p95 3.825초**로, STT 구간 p50 **3.370초**가 대기시간 대부분을 차지했다. Cloud Run mock/local storage benchmark에서 2/4/8 vCPU 전체 E2E p50은 각각 **21.082/10.717/7.988초**였고, CPU-only 이미지 최적화 후 크기는 **3.444→1.934GB**, build time은 **589.1→266.6초**로 줄었다.

**개인 기여·한계:** 모델 선정 실험은 MB-03, Agent 상태·MCP·DB 검증은 MB-07, 모델 파인튜닝과 1인 STT 정확도는 MB-08로 분리한다. 이번 경험의 고유 기여는 **인증 사용자의 음성 파일 기반 연속 흐름 검증과 upload→terminal 대기시간 분석**이다. 원본 기록에는 **실제 마이크로 실시간 녹음한 E2E 또는 Hosted Supabase/RLS를 포함한 배포 전체 STT E2E**가 확인되지 않는다. Cloud Run은 benchmark 전용 mock/local storage이며, 위 이미지 축소·속도 측정은 **모델 revision 고정 이전** 이미지 결과라 새 고정 이미지를 재측정했다고 쓰지 않는다. 프런트·STT 파이프라인 전부를 독자 구현했다고 주장하지 않는다.

근거: [로컬 Docker 인증 흐름](https://github.com/aihuman-7th/proj2-3/blob/main/docs/worklogs/2026-09-21_2315_local-supabase-candidate-integration-e2e.md) · [실제 사용자 3종·재답변 비교](https://github.com/aihuman-7th/proj2-3/blob/main/docs/worklogs/2026-09-29_1632_d6-agent-live-e2e.md) · [분석 E2E benchmark](https://github.com/aihuman-7th/proj2-3/blob/main/docs/experiments/stt/2026-09-28_analysis-e2e-benchmark.md).

### 8. Langfuse 실행 버전 정합성·운영 Trace 검증 — 개인 작성 및 리뷰 기여 확인

**상황:** 말해봄은 코칭 모델의 Prompt·Skill·토큰·비용·지연·재시도를 Langfuse에서 관측하고, 실제 코칭 결과를 Supabase `coaching_runs`에 저장하는 구조였다. 개발자의 로컬 Prompt/Skill 버전과 관리형 Langfuse Prompt label, DB에 남는 버전 정보가 일치한다는 보장이 부족했다.

**문제:** 기본 Skill 카탈로그가 `interview-coach-v6.1-domain-rules`로 바뀌어도 관측 상수는 오래된 `interview-coach-v6`을 반환할 수 있었고, 관리형 Prompt를 사용해도 DB는 `COACHING_PROMPT_VERSION` 환경변수를 독립적으로 읽어 **실제로 사용하지 않은 Prompt 버전을 저장**할 수 있었다. 운영 Langfuse 화면의 JSON 검증 통과가 어느 Cloud Run revision/DB 행/코칭 의미 품질과 연결되는지도 구분되지 않았다.

**판단:** 실제 서비스 실행에서 선택한 **effective Prompt·Skill version을 단일 기준으로 관측과 DB 저장에 전달**해야 한다. 이미 끝난 합성 모델 평가는 provider를 다시 호출하지 않고 Langfuse로 옮기며, 원본 latency·cost를 새 호출값으로 위장하지 않는다. 운영 증거도 **같은 Langfuse trace 안에서 Prompt generation과 validator가 연결된 것**, **사이트 요청의 HTTP header·Langfuse·Cloud Run 로그가 직접 연결된 것**을 별도 단계로 다룬다.

**조치:** 개인 [PR #101](https://github.com/aihuman-7th/proj2-3/pull/101)에서 모델 비교 저장 결과의 마지막 attempt·case·repeat·quality Score·원래 지연/비용을 Langfuse Experiment로 옮기는 도구를 작성하고 **합성 Dataset 전용·dry-run 기본·provider 재호출 없음**을 강제했다. [PR #162](https://github.com/aihuman-7th/proj2-3/pull/162)에서는 실제 코칭 서비스가 선택한 관리형 Prompt·Skill 버전이 Langfuse와 `coaching_runs`에 일치하도록 관측 상수·runtime·저장 함수 및 회귀 테스트를 수정했다. [PR #114](https://github.com/aihuman-7th/proj2-3/pull/114)로 인증 사용자 코칭의 네트워크 오류·재시도·정상 provider 호출 뒤 DB migration 오류를 trace 단계별로 기록했다. [PR #216](https://github.com/aihuman-7th/proj2-3/pull/216)에서는 10월 8일 사용자 제공 Langfuse 화면에서 **관리형 Prompt 숫자 @4 `production` 라벨과 실제 같은 trace의 generation·validator**를 대조했다. 팀원 곽호준이 작성한 [PR #221](https://github.com/aihuman-7th/proj2-3/pull/221)의 HTTP 요청–Langfuse–Cloud Run ID 연결은 **코드/테스트/실제 200·422 증거를 리뷰해 승인**했다.

**결과:** 버전 정합성 회귀 테스트를 포함한 Langfuse SDK v4.15.1 **70건 테스트 통과**가 확인됐고, Experiment export는 mock 2후보×2반복×30건 **120회 저장 결과에 대한 4개 run 계획 dry-run**을 통과했다(**원격 apply 미실행**). 로컬 인증 사용자 Langfuse 관측에서는 네트워크 오류의 2번 전송 시도·1회 재시도, 그리고 provider/validator 정상이어도 DB migration 누락 시 persistence가 실패하는 원인을 분리했다. 10월 8일 운영 Langfuse 한 코칭 trace에서 **Gemini Flash-Lite, 관리형 Prompt @4, 약 8.19초·24,266 tokens·$0.006809, `valid=true`·오류 0**을 확인했다. 이어 10월 9일 운영 사이트의 **준비 채팅 200(모델 호출 1회)과 스키마 오류 422(모델 호출 0회)**에서 HTTP·Langfuse·Cloud Run 로그가 같은 요청 ID로 연결된 증거를 리뷰했다.

**개인 기여 및 한계:** #101·#114·#162·#216은 **nanocode00 명의 작성 커밋**이며 #221의 기능 구현·운영 배포는 **곽호준 작업**, 내 역할은 리뷰다. 10/8 Prompt @4의 **release/rollback을 직접 수행하지 않았고**, 그 당시 코칭 trace의 Cloud Run revision·실제 Skill bundle SHA·`coaching_runs` DB 행은 확인하지 못했다. 10/9 운영 요청 연결은 **준비 채팅**으로, 코칭·마이크·STT 전체 E2E나 응답의 사람 품질 검증이 아니다. `production` 라벨·코드 v4/v5·Langfuse 숫자 @4는 별개이며, 계약 검증 PASS가 의미 정확도를 보장하지 않는다.

근거: [평가 결과 Langfuse export](https://github.com/aihuman-7th/proj2-3/blob/main/docs/worklogs/2026-09-28_1540_langfuse-evaluation-export.md) · [실제 버전 기록 정합성](https://github.com/aihuman-7th/proj2-3/blob/main/docs/worklogs/2026-10-02_coaching-version-provenance.md) · [10/8 관리형 Prompt 운영 관측](https://github.com/aihuman-7th/proj2-3/blob/main/docs/worklogs/2026-10-08_d10-production-readonly-evidence.md) · [10/9 요청–Trace 연결](https://github.com/aihuman-7th/proj2-3/blob/main/docs/evidence/request-trace-linkage-2026-10-09.md).

### 9. UI 테마·전사문 가독성 리뷰와 독립 쇼케이스 웹사이트 구현 — 개인 역할 구분 확인

**상황:** 말해봄은 음성 면접 연습·전사 확인·코칭·재답변·기록 기능을 갖춘 상태였지만, 처음 방문하는 이용자에게 서비스 목적을 설명하는 콘텐츠와 제품 UI의 시각적 일관성이 필요했다. 라이트/다크 테마와 시작·녹음·전사 상태 화면 개선이 검토 대상이었고, 11페이지 랜딩 개선 PDF가 별도 소개 사이트 설계의 기초가 됐다.

**문제:** UI 테마 구현 과정에서 **정의되지 않은 CSS 변수, 다크 모드에 연결되지 않은 화면별 고정색, 전사문 편집기의 어두운 배경/글자색 cascade 충돌**이 발견됐다. 소개 사이트에는 기획 목업, 당시 서비스 데모 이미지, 실제 운영 기능이 섞여 사용자에게 검증되지 않은 기능·성능·데이터 보관 정책을 확정된 내용처럼 보여줄 위험이 있었다.

**판단:** 제품 UI 리뷰에서는 화면 전체 색상 교체만 확인하지 않고 **CSS 토큰 유효성·개별 입력 상태의 대비·기존 녹음/연습/기록 기능 보존**을 검증하기로 했다. 제품 코어와 배포를 변경하지 않으면서 **의존성 없는 정적 HTML/CSS/JS로 소개 콘텐츠를 독립 제공**, 브랜드 보라색·마스코트·상태 이미지 등 기존 자산을 재사용하고 **기획용 UI/실서비스 예시 및 확인된/미확인 정보를 구분**하기로 했다.

**조치:** [UI 개선 PR #211](https://github.com/aihuman-7th/proj2-3/pull/211)에서 CSS 미정의 토큰 3개와 다크 모드의 공통 화면 대비 누락을 지적했고, 후속 리뷰에서 `.transcript-editor`의 기존 고정 글자색이 새 dark selector보다 specificity가 높아 **전사문을 읽기 어려워지는 결함**을 발견해 다시 수정 요청했다. 마지막 리뷰에서 정상 입력 **15.29:1**, 비활성 **8.46:1** 대비를 확인하고 승인했다. 이 PR의 ThemeProvider·홈 화면·상태 이미지 구현 자체는 **팀/코딩 도구 작업**, 내 직접 기여는 결함 발견·재검토·승인이다.

별도 [PR #214](https://github.com/aihuman-7th/proj2-3/pull/214)는 **nanocode00 명의 3개 커밋**으로 11쪽 랜딩 개선 기획을 **서비스 소개와 프로젝트 소개 2개 정적 페이지**로 제작했다. Light/Dark/System 테마, 반응형 메뉴, FAQ, 연습 시작 CTA, 사용 흐름/기획용 3단 코칭 예시를 구성하고, 제품 로고·마스코트·녹음/전사 캐릭터·데모 4장·배경을 최적화해 재사용했다. 정적 파일·앵커·이미지 시그니처 검사 도구를 추가하고 프로젝트 기간을 확정하되 실제 삭제 정책으로 검증하지 못한 **24시간 보관 문구는 공개 확정 정보로 사용하지 않았다**. 별도 [개인 쇼케이스 저장소](https://github.com/nanocode00/malhaebom-showcase)에도 정적 사이트 파일을 보존했다.

**결과:** UI 리뷰에서 세 종류의 실제 표시 오류를 수정 요청해 다크 모드의 전사 편집기 가독성과 공통 스타일 일관성 개선을 확인했다. 개인 정적 사이트는 2개 페이지·최적화된 브랜드/제품 자산·반응형/테마·공개 전 검증 체크리스트를 포함했고, 최종 정적 검사에서 **서비스 18개 ID·31개 링크/자산, 프로젝트 7개 ID·30개 링크/자산**, 로컬 HTTP 200과 빌드 불필요 정적 검증을 통과했다. 별도 GitHub 저장소 소스도 확인했다. 실제 GitHub Pages 현재 접근·운영 사용량은 이번 조사에서 재측정하지 않았다.

**기여 경계:** PR #211에서 작성자 기록의 **프런트 181개 테스트·빌드 통과**는 확인했지만 나는 당시 Windows npm/WSL 설치 환경 문제로 테스트를 독립 재실행하지 못했다. **10월 10일 이후** 소개 화면을 운영 React/Vite의 `/` 진입점 및 로그인/연습 경로에 통합한 [PR #240](https://github.com/aihuman-7th/proj2-3/pull/240)·[#241](https://github.com/aihuman-7th/proj2-3/pull/241), 소개 콘텐츠 후속 [#251](https://github.com/aihuman-7th/proj2-3/pull/251)은 **곽호준 작성 팀 작업**으로, 내 10월 8일 독립 사이트 제작과 구분한다. 이 후속 통합을 10월 8일 이전 개인 기능 구현이나 실제 Google 로그인 전체 검증으로 주장하지 않는다.

근거: [UI 리뷰 #211](https://github.com/aihuman-7th/proj2-3/pull/211) · [쇼케이스 개인 PR #214](https://github.com/aihuman-7th/proj2-3/pull/214) · [10/8 소개 사이트 Worklog](https://github.com/aihuman-7th/proj2-3/blob/main/docs/worklogs/2026-10-08_malhaebom-intro-site.md) · [개인 쇼케이스 저장소](https://github.com/nanocode00/malhaebom-showcase) · [10/10 팀 통합 Worklog](https://github.com/aihuman-7th/proj2-3/blob/main/docs/worklogs/2026-10-10_showcase-intro-integration.md).

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
