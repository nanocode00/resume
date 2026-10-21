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
| MB-01 | 문제 정의·경쟁 비교·프로젝트 범위 고정 | PROJECT_OVERVIEW, RESEARCH, D2 체크포인트 | T/B | 문제 정의서·경쟁 조사 직접 작성/판단 범위와 사용 확인 여부 |
| MB-02 | Agent의 Prompt·Skill·MCP·서버 정책 분리 | ARCHITECTURE, AGENT_REPORT, 코드 | T/B | 초기 baseline, 모듈별 개인 diff 및 팀 역할 |
| MB-03 | OpenRouter 모델 다양성 탐색·Final Four | EVAL_REPORT §9~13, 9/21 실험 | T/B | runner 구현/실행자, 본인 의사결정, 원본 재현 설정 |
| MB-04 | 합성 평가셋 30건의 정답 사람 검토·정정 | reviewer-assignment, 9/30 offline regrade | **A** | 검토표·수정 반영 커밋 연결만 추가 |
| MB-05 | Prompt/Skill A/B 및 회귀 대응 | 10/2 30건, 10/6 Skill smoke | T/B | 직접 만든 harness/실험 실행과 팀 검토·프롬프트 작성 분리 |
| MB-06 | 로그인 사용자 공고 원문 분석 E2E | 10/1 14:00 김재훈 Worklog, 토큰 0600 검증 | **A** | 해당 PR 번호·diff, 반복 재현 여부 확인 |
| MB-07 | FastMCP stdio/HTTP, /api/agent, DB 저장 | AGENT_REPORT, CURRENT_STATE, 서버 코드, PR | T/B | stdio 도구 기여/통합 검증과 구현 코드 책임 구분 |
| MB-08 | Whisper Medium 50k → B0 → C2 학습 | stt-finetuning-lineage, 50k/C2 실험 | T/B | 본인 수행 train configs/로그/checkpoint SHA, 운영 모델 선택 |
| MB-09 | 실제 마이크/STT·NLU·Supabase 통합 E2E | checkpoint, EVAL_REPORT, 일부 worklog | T/B | 인증 사용자 부분 E2E와 운영 전체 E2E 분리 |
| MB-10 | Langfuse Prompt/trace·운영 관측 | AGENT_REPORT, 10/8~10/9 운영 증거 | T/B | 직접 운영 trace 확인/배포와 팀 공통 관측 분리 |
| MB-11 | UI/UX 개선과 소개 페이지·배포 | 10/10 README, 운영 비교 PR, 별도 showcase 저장소 | T/B | 개인 제안·목업·별도 showcase 설계와 팀 프론트 구현 구분 |
| MB-12 | D1~D12 체크포인트·PR·CI 협업/종료 | 10/1 개인 Worklog, PR/CI 기록 | A/B | 직접 처리한 리뷰·브랜치·테스트·병합 대표 3~5건 선정 |

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
