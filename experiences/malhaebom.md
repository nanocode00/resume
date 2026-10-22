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

### 2. 로그인 사용자 공고 원문 분석 E2E의 실제 재현 — 개인 작업 기록 확인

**문제:** 공고 조회·요구사항 추출·원문 인용 검증 코드가 개별적으로 존재해도 실제 로그인 사용자 권한 아래에서 저장까지 이어지는지 불확실했다. 외부 MCP 인증 요청은 503, 서비스 재기동 뒤 기존 JWT는 401을 반환했다.

**판단·조치:** 브라우저에서 얻은 실제 로그인 세션 JWT로 프런트와 같은 API 순서를 검증하고, 비대화형 E2E runner가 권한 `0600`인 Git ignored 토큰 파일만 허용하도록 보완했다(권한 우선순위·0644 거부 등 단위검사 3건). 당시 PlayMCP 503은 기능 전체의 실패와 구분해 보존된 `web` provider 경로로 전환했고, 새로운 JWT로 테스트했다.

**결과:** `NCS 선택 → 외부 공고 추천 3개 → 원문 조회 → requirement 5개 추출 → quote validation 통과 → 인증 사용자 목록에 저장 → 요구사항 2개 선택 → 연습 세션 생성` 경로 성공을 redacted 기록으로 남겼다. **브라우저 클릭 UI E2E, 운영 배포에서의 동일 검증, PlayMCP 복구를 완료한 것은 아니다.**

근거: [2026-10-01 개인 Worklog](https://github.com/aihuman-7th/proj2-3/blob/main/docs/worklogs/2026-10-01_1400_job-source-authenticated-e2e.md).

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
