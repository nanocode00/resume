# Mallo — Senior-Friendly Voice Ordering Kiosk

## 한 줄 요약

고령 사용자의 키오스크 주문 부담을 줄이기 위해 만든 3인 팀 음성 주문 서비스입니다. 팀장으로서 PRD/TRD와 공통 인터페이스를 정리하고 Order Engine, Senior-first UI, TTS 파인튜닝, 실제 STT/NLU/TTS 통합, 런타임·배포, 최종 E2E 검증까지 제품 전체 흐름을 연결했습니다.

## 기간 및 프로젝트 상태

- 기간: **2026.06 ~ 2026.08**
- 2026-08-28: 기능 동결(feature freeze)
- 2026-08-30: 최종 merge/정리 단계 완료
- 현재 상태: **프로젝트 종료 및 동결**
- 3인 팀 프로젝트

최종 README 기준 역할은 다음과 같습니다.

- 김재훈: **Team Lead / Product & Integration**
- 이규헌: STT Lead
- 이승훈: NLU Lead

따라서 STT·NLU 모델 학습 전체를 개인 기여로 주장하지 않습니다. 직접 기여의 중심은 **제품 기획과 인터페이스 설계, Order Engine, TTS 실험·파인튜닝, Senior-first UI, 모델 선택과 통합, runtime/deployment, 최종 검증**입니다.

## 문제 정의

Mallo는 음성 기능을 하나 더 붙인 키오스크가 아니라, 화면 탐색과 옵션 선택이 익숙하지 않은 사용자가 주문을 끝까지 완료할 수 있도록 **voice-first이되 voice-only는 아닌 주문 경험**을 만드는 것을 목표로 했습니다.

주요 방향은 다음과 같습니다.

- 음성으로 메뉴 추가·수량 변경·삭제·장바구니 확인·결제 진행
- 화면과 음성을 함께 사용해 현재 주문 상태를 계속 확인 가능
- 다시 듣기, 천천히 듣기, 직원 호출 등 접근성 기능 제공
- 잘못 인식되거나 애매한 명령은 Validator/Order Engine에서 방어
- AI 모델의 출력이 바로 UI 상태를 변경하지 않고 공통 command schema를 거치도록 구성

## 최종 시스템 구조

최종 runtime의 핵심 흐름은 다음과 같습니다.

`Mic / Tap-to-talk → STT → NLU → Validator → Order Engine → Response Policy / Adaptive TTS → TTS → Web UI`

서비스 프로세스는 분리해 상주시켰습니다.

- Gateway / FastAPI: `127.0.0.1:8000`
- STT sidecar: `127.0.0.1:8001`
- NLU sidecar: `127.0.0.1:8002`
- TTS sidecar: `127.0.0.1:8003`

통합 launcher가 각 sidecar를 시작하고 `/health` readiness 확인, 실제 inference warm-up, gateway 시작 순서를 관리합니다. 모델마다 필요한 Python dependency가 달라 runtime과 NLU virtual environment도 분리했습니다.

## 1. 기획과 공통 계약부터 시작

프로젝트 초반에는 PRD와 TRD를 먼저 작성하고 STT/NLU/TTS를 독립적으로 개발해도 마지막에 연결할 수 있도록 공통 contract를 정리했습니다.

- 음성 입력과 STT result contract
- NLU command schema
- Validator의 허용/거부 규칙
- Cart와 Order Engine state
- TTS request/response interface
- Mock adapter를 이용한 E2E skeleton

초기 단계에서 실제 모델을 모두 기다리지 않고 Mock STT/NLU/TTS로 주문 흐름을 먼저 연결해, 각 모델 담당자가 동일한 인터페이스에 맞춰 교체할 수 있도록 했습니다.

## 2. Order Engine과 deterministic ordering flow

AI가 생성한 문장을 그대로 주문 상태에 반영하지 않고, 구조화된 command를 deterministic Order Engine에 전달했습니다.

직접 작업한 주요 주문 연산에는 다음이 포함됩니다.

- 메뉴 추가
- `REMOVE_ITEM`
- `CHANGE_QTY`
- `CLEAR_CART`
- 장바구니 조회 및 review
- fulfillment/payment flow

이 구조 덕분에 NLU 모델이 바뀌어도 Cart state와 주문 규칙은 AI 모델 밖에서 일관되게 유지할 수 있었습니다.

## 3. TTS 후보 비교와 파인튜닝

TTS는 직접 담당한 핵심 AI 영역입니다.

초기에는 MeloTTS와 Qwen 계열 TTS 후보의 한국어 품질, latency, 실행 환경을 비교하고 공통 adapter를 만들었습니다. 이후 AI Hub의 친절한 발화 데이터를 중심으로 dataset 전처리와 학습 pipeline을 구성해 MeloTTS를 fine-tuning했습니다.

주요 흐름은 다음과 같습니다.

1. TTS candidate 조사 및 latency/quality benchmark
2. AI Hub 친절체 dataset 정리와 전처리
3. 학습용 metadata/audio 검증
4. MeloTTS fine-tuning 실행
5. **2,800 step의 `G_2800.pth` checkpoint 생성**
6. baseline과 fine-tuned checkpoint listening comparison
7. 말하기 속도 Level 0/1/2 showcase 구성
8. 최종 production TTS sidecar에 `G_2800.pth` 연결

최종 정량 결과 문서의 production MeloTTS benchmark는 다음과 같습니다.

- 평균 synthesis latency: **0.38 s**
- p95 latency: **0.53 s**
- RTF: **0.0894**
- TTS CI: **49 tests passed**

Fine-tuning 결과가 모든 문장에서 baseline보다 좋아졌다고 과장하지 않습니다. 실제 listening 과정에서는 자연스러움, 억양, 음색 변화와 일부 발음 안정성 trade-off를 함께 확인했고, baseline과 checkpoint를 직접 비교할 수 있는 showcase를 남겼습니다.

## 4. STT/NLU/TTS를 accuracy가 아니라 제품 latency로 비교

팀 전체 모델을 통합하면서 모델 선택 기준을 정확도 하나로 두지 않았습니다.

### STT

최종 정량 문서에서 `openai/whisper-medium` baseline은 다음 결과를 기록했습니다.

- CER: **0.0546**
- WER: **0.1626**
- median latency: **385.57 ms**
- p95 latency: **424.15 ms**

QLoRA fine-tuned candidate는 CER/WER가 소폭 개선되거나 비슷했지만 inference가 baseline보다 약 **35~115배 느린 결과**가 나왔습니다. 따라서 정확도 개선보다 kiosk 응답성을 우선해 baseline 계열을 유지하는 판단을 했습니다.

STT fine-tuning 자체는 STT Lead의 담당 영역이며, 여기서의 직접 역할은 **후보 결과를 제품 latency 관점에서 비교하고 최종 runtime에 연결하는 통합 의사결정**입니다.

### NLU

최종 NLU benchmark에는 다음 결과가 남아 있습니다.

- source benchmark: **198 / 198 exact match**
- legacy batch: **1000 / 1000**
- schema parse error: **0**
- source benchmark mean latency: **0.153 s**
- median latency: **0.035 s**

NLU 모델 학습은 NLU Lead가 중심이었지만, 프로젝트 후반에는 Qwen3-0.6B candidate의 service integration, artifact contract 검증, latency 최적화와 전체 voice pipeline 연결에도 참여했습니다.

## 5. Senior-first UI/UX로 방향 전환

중간 통합 과정에서 AI 모델만 개선해서는 사용성이 해결되지 않는다는 문제가 드러났습니다. 음성 기능과 별개로 화면 구조가 복잡해지면 고령 사용자의 주문 부담이 그대로 남기 때문에 UI/UX 개선을 병행했습니다.

직접 반영한 주요 항목은 다음과 같습니다.

- Voice Overlay
- 큰 글자와 고대비 중심의 Senior-first 화면
- 접근성 footer 2×2 구조
- 다시 듣기
- 천천히 듣기 / 음성 속도 단계 변경
- 직원 호출
- ORDER / REVIEW / PAYMENT 단계 정리
- 결제 UI 단순화
- Express mode / one-shot voice payment flow
- 마이크 입력과 음성 session state 연결

즉 프로젝트 중반 이후에는 `모델 성능 개선`과 `화면 사용성 개선`을 별도 문제로 보고 동시에 수정했습니다.

## 6. 실제 모델 runtime 통합

후반에는 Mock adapter 중심 구조를 실제 모델로 교체했습니다.

- real STT service integration
- real TTS playback
- Qwen NLU service integration
- STT/NLU/TTS resident sidecar 분리
- process별 health/readiness 확인
- warm-up 이후 gateway 시작
- session log와 debug path 구성
- 한 줄 launcher로 전체 runtime 기동

특히 STT/NLU/TTS를 한 Python process에 모두 넣지 않고 독립 sidecar로 분리해 모델별 dependency와 GPU resident state를 관리했습니다.

프로젝트 진행 중 직접 측정한 운영 기록에서는 STT와 TTS를 동시에 GPU에 올려도 실행 가능함을 확인했고, 최종 단계에서는 각 sidecar의 `/health`가 ready/warmed 상태인지 확인한 뒤 E2E 검증을 진행했습니다.

## 7. Hybrid deployment

브라우저 UI와 로컬 AI runtime을 분리하는 형태로 배포했습니다.

`Vercel Static Frontend → Cloudflare Tunnel → Local FastAPI Gateway → STT/NLU/TTS Sidecars`

- Web UI: Vercel
- AI runtime: local Windows/WSL + CUDA environment
- backend: FastAPI gateway
- public connection: Cloudflare Tunnel
- CORS/runtime config를 통해 Vercel frontend와 local inference server 연결

큰 모델 파일을 Vercel에 올리는 대신 GPU가 있는 로컬 PC에서 inference를 수행하고 web UI만 정적 hosting하는 구조를 선택했습니다.

## 8. E2E latency를 단계별로 측정

최종 정량 검증에서는 isolated 50-turn voice pipeline run을 수행했습니다.

- E2E p50: **253.9 ms**
- E2E p95: **312.2 ms**
- STT median: 약 **188.5 ms**
- NLU median: 약 **15.6 ms**
- NLU latency 비중: 약 **6.2%**

이 결과를 통해 당시 pipeline의 주 latency가 NLU가 아니라 STT 쪽이라는 점을 확인했습니다.

단, 이 50-turn 수치는 **real STT + real NLU + MockTTS 기반 isolated latency run**입니다. 따라서 `실제 TTS 음성 생성까지 포함한 전체 p95가 312.2 ms`라고 표현하지 않습니다. 실제 TTS 성능은 별도의 MeloTTS benchmark(mean 0.38 s, p95 0.53 s)로 관리합니다.

## 9. STT/TTS showcase와 최종 검증 자산

최종 결과를 코드만으로 남기지 않고 별도 showcase도 구성했습니다.

### STT showcase

직접 녹음한 한국어 발화 5개를 사용해 실제 inference 결과를 확인했습니다. 일반 주문뿐 아니라 존댓말 어미와 머뭇거림 등이 섞인 발화도 포함했습니다.

### TTS showcase

- baseline MeloTTS
- fine-tuned `G_2800`
- speed Level 0 / 1 / 2

를 같은 문장으로 비교할 수 있도록 구성했습니다.

소개 페이지에서는 STT showcase와 TTS showcase를 함께 연결해 개발 결과를 외부에서 확인할 수 있도록 정리했습니다.

## 10. 사용자 평가를 해석할 때의 주의점

최종 정량 결과 문서는 사용자 평가에서 **n=13, 만족 응답 92.3%**를 기록합니다.

다만 목표했던 고령 사용자 표본을 충분히 확보하지 못해 실제 참가자 구성은 20~40대가 섞인 제한된 표본이었습니다. 따라서 다음과 같이 구분합니다.

- senior-friendly product를 설계하고 senior-specific evaluation protocol을 준비함: 확인 가능
- 최종 사용자 평가 n=13, 만족도 92.3%: 확인 가능
- 13명 모두 고령 사용자였다고 주장: 하지 않음

README의 사용자 수 표기와 최종 정량 문서 사이에도 차이가 있으므로, 이력서에서는 더 구체적인 최종 정량 문서의 **n=13**을 기준으로 사용합니다.

## 11. 프로젝트 종료와 repository cleanup

기능 동결 이후에는 실행에 필요한 production artifact와 대용량 training/research 자산을 분리했습니다.

cleanup audit에서 당시 WSL workspace 약 **213.83 GB**, checkpoint/output 약 **176 GB**, dataset extract 약 **28 GB** 규모를 점검하고 `keep / retain / confirm / delete` 기준으로 분류했습니다.

최종 production set에는 다음 핵심 artifact를 유지했습니다.

- STT runtime model
- NLU merged model
- TTS `G_2800.pth` + `config.json`
- runtime source / tests / docs
- showcase assets

raw dataset, intermediate checkpoints, cache, 실험 산출물은 runtime repository와 분리해 프로젝트를 동결했습니다.

## Git으로 확인되는 직접 작업 흐름

주요 PR 흐름을 보면 역할이 다음처럼 확장되었습니다.

### 초기: 제품/아키텍처

- PRD / TRD 작성
- Mock E2E integration
- NLU general order coverage
- Order Engine operation 구현
- TTS candidate 조사와 benchmark
- TTS adapter / Adaptive TTS controller

### 중기: TTS와 Senior-first UX

- AI Hub TTS dataset tooling/preprocessing
- MeloTTS/Qwen 계열 비교
- TTS latency benchmark 및 결과 정리
- Senior-first UI
- mic input / audio input 통합
- one-shot voice payment

### 후반: 실제 모델과 배포

- real STT service integration
- real TTS playback
- NLU model/service integration 지원
- voice pipeline E2E
- runtime launcher
- Vercel static deployment
- Cloudflare/local runtime 연결
- STT/NLU/TTS sidecar 분리
- latency optimization
- final quantitative results
- showcase/intro portal
- repository cleanup 및 기능 동결

## 결과

- 3인 팀의 음성 주문 제품을 기획 단계부터 실제 모델 통합·배포까지 연결
- deterministic Order Engine과 AI adapter를 분리해 모델 교체 가능한 구조 구성
- MeloTTS fine-tuning `G_2800` checkpoint 생성 및 production sidecar 적용
- production TTS mean **0.38 s**, p95 **0.53 s**, RTF **0.0894** 측정
- STT/NLU/TTS resident sidecar와 통합 launcher 구성
- Vercel + Cloudflare Tunnel + local FastAPI hybrid deployment
- real STT/NLU 기반 isolated 50-turn E2E p95 **312.2 ms** 측정
- 사용자 평가 최종 근거 문서 기준 **n=13, 만족 응답 92.3%**
- 기능 동결 후 production artifact와 대용량 학습 자산을 분리해 프로젝트 종료

## 이력서용 핵심 bullet

- 3인 팀 팀장으로 **PRD/TRD와 STT·NLU·TTS 공통 contract를 정의하고 Order Engine, Senior-first UI, 실제 AI adapter를 연결해 음성 주문 E2E 제품 구현**
- AI Hub 친절 발화 데이터를 전처리해 **MeloTTS 2,800-step fine-tuning(`G_2800`)**을 수행하고 baseline/fine-tuned listening showcase와 Adaptive TTS 속도 단계를 구축
- STT·NLU·TTS 후보를 정확도와 latency 기준으로 비교하고 **Gateway 8000 + STT 8001 + NLU 8002 + TTS 8003 sidecar 구조와 health/warm-up launcher**로 production runtime 통합
- **Vercel frontend + Cloudflare Tunnel + local FastAPI** hybrid deployment를 구성하고, real STT/NLU 기반 50-turn isolated E2E에서 **p95 312.2 ms**를 측정해 bottleneck을 STT 단계로 분해

## 면접/자소서에서 강조할 문제 해결 경험

### 1. 정확도가 좋아도 느리면 제품에 쓰지 않은 선택

STT fine-tuned candidate가 일부 accuracy 지표에서 개선됐지만 inference가 35~115배 느려지는 결과를 확인했습니다. 모델 성능 하나보다 kiosk의 응답 시간을 우선해 더 빠른 baseline을 선택했습니다.

### 2. NLU만 고치려다 UI 문제를 함께 발견한 pivot

초기에는 음성 명령 인식 성능과 latency를 주로 개선했지만, 통합 과정에서 화면 자체가 복잡하면 senior 사용성이 해결되지 않는다는 문제를 확인했습니다. 모델 최적화와 동시에 Senior-first UI를 다시 설계했습니다.

### 3. 모델마다 dependency가 달라 발생한 runtime 문제

STT/TTS와 NLU의 dependency가 충돌하고 모델을 매 요청마다 로드하면 latency가 커지는 문제를 sidecar와 별도 venv로 분리했습니다. 각 모델을 resident process로 유지하고 launcher에서 readiness와 warm-up을 검사하도록 구성했습니다.

### 4. 실험 결과를 제품 지표로 다시 측정

개별 모델 benchmark만 보는 대신 voice pipeline을 실제 조합으로 측정했습니다. 50-turn latency breakdown으로 STT가 주 bottleneck임을 확인하고 이후 최적화 우선순위를 조정했습니다.

## 근거와 사용 시 주의

강하게 사용할 수 있는 내용:

- 3인 팀 Team Lead / Product & Integration 역할
- PRD/TRD, Order Engine, common contract, UI/UX, integration/deployment 직접 기여
- TTS dataset/preprocessing/fine-tuning과 `G_2800`
- production TTS latency 수치
- sidecar/launcher/Vercel/Cloudflare runtime 구조
- final E2E latency 측정과 bottleneck 분석
- 프로젝트 종료 및 repository cleanup

개인 기여로 과장하지 않을 내용:

- STT fine-tuning 전체를 직접 수행했다고 주장
- NLU training 전체를 직접 수행했다고 주장
- 50-turn p95 312.2 ms에 실제 TTS synthesis까지 포함됐다고 주장
- 사용자 평가 13명이 모두 고령자였다고 주장
- fine-tuned TTS가 모든 발화에서 baseline보다 우수했다고 주장

## 보여주는 역량

- Team Lead / 제품 통합
- AI product architecture
- Voice AI(STT/NLU/TTS) integration
- TTS fine-tuning 및 inference benchmark
- deterministic domain logic / Order Engine
- 접근성 중심 UI/UX
- FastAPI / sidecar / process orchestration
- Vercel / Cloudflare hybrid deployment
- latency profiling과 bottleneck 분석
- 대용량 ML artifact 정리와 production handoff

## 자소서 활용 포인트

- 팀장으로 여러 AI 모델 파트를 하나의 제품으로 연결한 경험
- 모델 accuracy와 실제 latency 사이에서 기술 선택을 한 경험
- 초기 방향을 고집하지 않고 사용자 문제를 보고 UI까지 pivot한 경험
- AI model과 deterministic business logic을 분리한 설계 경험
- prototype을 deployment·showcase·cleanup까지 마무리한 경험
