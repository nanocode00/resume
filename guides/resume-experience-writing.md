# 이력서 및 경험기술서 작성 가이드

이 문서는 `resume.md`와 `experiences/` 문서를 작성하거나 수정할 때 반복해서 적용할 원칙을 정리합니다.

자기소개서 문장 구성과 분량 규칙은 `cover-letter-writing.md`에서 관리하고, 이 문서는 **경험을 어떤 근거로 복원하고 이력서에 어떻게 압축할지**에 집중합니다.

## 1. 문서 역할을 분리합니다

### `resume.md`

채용 담당자가 짧은 시간 안에 강한 근거를 확인할 수 있는 문서로 작성합니다.

- 추상적인 장점보다 실제 수치와 판단을 우선
- 주력 프로젝트 2~3개를 상세히 설명
- 보조 프로젝트는 2~3줄로 압축
- 각 프로젝트에 기간, 팀 규모, 본인 담당 범위를 표시
- 세부 provenance와 디버깅 기록은 `experiences/`로 넘김

현재 주력 프로젝트 우선순위:

1. HummingBlocks
2. CNN Accelerator
3. Mallo

### `experiences/*.md`

세부 구현, 수치, 실패 기록, source provenance를 보존하는 근거 문서입니다.

맨 위에는 채용용 요약을 두고, 아래에는 commit, log, 발표자료, 보고서 등 상세 근거를 남깁니다.

## 2. 경험은 상황 → 문제 → 판단 → 조치 → 결과로 씁니다

### 상황

처음 읽는 사람도 무엇을 만든 프로젝트인지 이해할 수 있도록 짧게 설명합니다.

함께 명시할 것:

- 기간
- 팀 규모
- 본인 역할
- 사용한 기존 코드나 공개 baseline이 있다면 그 범위

### 문제

`성능이 안 좋았다`, `오류가 있었다`처럼 추상적으로 쓰지 않습니다.

가능하면 관찰값과 조건을 함께 씁니다.

예:

- Galaxy S9+에서 평균 처리시간 9.1초
- ModelSim 1,000-image RTL simulation 96% accuracy
- gate-level simulation에서 정상 waveform 확보 실패
- MeloTTS runtime과 Qwen NLU의 `transformers` 버전 충돌

문제가 여러 개라면 정확도, latency, dependency, 사용성, synthesis, timing처럼 종류를 나눕니다.

### 판단

무엇을 했는지보다 **왜 그 방법을 골랐는지**를 남깁니다.

좋은 판단은 비교 대상과 기준이 있습니다.

- 속도보다 정확도가 제품에 더 중요한가
- 큰 모델보다 interactive latency가 더 중요한가
- LLM이 상태를 직접 변경하는 것보다 deterministic logic으로 분리하는 것이 안전한가
- 최종 output보다 intermediate layer/MAC 비교가 원인 추적에 더 유리한가

`최신 기술이라 사용했다`, `성능이 좋아 보여 선택했다` 같은 표현은 피합니다.

### 조치

기술 스택을 나열하는 대신 실제 변경한 코드나 구조를 적습니다.

예:

- Chaquopy/Python inference를 TFLite + Java로 이식
- confidence filtering, NMS, 위치 후처리까지 Java로 이동
- quantized Tensor를 `int_repr()` 기반 HW reference로 변환
- `reg + always` parameter unpacking을 `generate + assign`으로 변경
- LLM과 Cart state를 분리한 deterministic Order Engine 구현
- dependency가 충돌하는 모델을 별도 venv와 resident sidecar로 분리

가능하면 실제 commit diff를 확인해 조치를 복원합니다.

### 결과

결과는 가능하면 수치, 비교 전후, 성공 여부로 끝냅니다.

예:

- 9.1s → 5.4s, 약 40.7% 단축
- 정확도 90% → 57%, -33%p
- MeloTTS 204ms vs Qwen3-TTS 6.91~28.20s
- MNIST 1,000장 RTL accuracy 96%
- final STA slack -2040.66, VIOLATED

실패한 결과도 숨기지 않습니다.

- 정상 waveform 미확보
- timing closure 실패
- 수수료 반영 후 전략 수익성 붕괴

어디까지 성공했고 어디서 실패했는지가 명확하면 좋은 경험입니다.

## 3. Summary와 Core Strength는 사실 문장으로 씁니다

`완성하는 개발자`, `원인을 좁혀가는 사람`, `목적 중심의 기술 선택`처럼 평가형 문장을 먼저 쓰지 않습니다.

가능하면 실제 경험과 수치로 장점을 증명합니다.

예:

- TFLite 전환으로 처리시간을 40.7% 줄였지만 정확도가 33%p 하락해 기존 runtime 유지
- 4개 TTS 모델의 동일 20문장 latency를 비교해 MeloTTS Friendly 선정
- RTL 1,000장 accuracy 96%를 확인하고 OpenROAD P&R final stage까지 수행했으나 timing closure 실패까지 기록

수상도 Summary에서 짧게 묶어 전체 경험의 범위를 보여줄 수 있습니다.

## 4. 주력 프로젝트와 보조 프로젝트의 밀도를 다르게 합니다

### 주력 프로젝트

2~3개만 선택해 반 페이지 안팎으로 설명합니다.

반드시 포함할 것:

- 상황
- 문제
- 판단
- 조치
- 결과
- 핵심 수치

기술 이름보다 본인의 판단이 보여야 합니다.

### 보조 프로젝트

2~3줄 정도로 압축합니다.

형식 예시:

`한 줄 요약 + 기술 스택/담당 범위 + 결과 또는 수상`

세부 구현은 `experiences/`에서 관리합니다.

## 5. 수치는 맥락과 함께 씁니다

숫자만 크게 적지 않고 가능한 한 다음 정보를 붙입니다.

- 측정 장비 또는 환경
- 데이터 또는 sample 수
- baseline
- 비교 대상
- 단위
- 측정 조건

예:

`96% accuracy`보다 `ModelSim에서 MNIST 1,000장 기준 RTL accuracy 96%`가 좋습니다.

서로 다른 조건에서 측정한 수치를 직접 개선율로 비교하지 않습니다.

예:

- `G_2000` 20문장 GPU benchmark
- `G_2800` production benchmark

두 값의 측정 조건이 다르면 각각 별도 결과로 기록합니다.

## 6. 비교 실험은 최종 선택까지 연결합니다

두 모델이나 구현을 비교했다면 다음 네 가지를 반드시 남깁니다.

1. 무엇을 비교했는가
2. 어떤 지표를 봤는가
3. 결과가 어떻게 달랐는가
4. 무엇을 선택했고 왜 선택했는가

`A와 B를 비교했다`에서 끝내지 않습니다.

예:

- TFLite는 더 빨랐지만 정확도 손실 때문에 미채택
- Qwen3-TTS는 자연스러운 후보였지만 6.9~28.2초 생성시간 때문에 interactive kiosk에서 제외
- MeloTTS Friendly는 baseline보다 약 2.8% 느려졌지만 0.2초대 응답성을 유지하며 친절체 억양을 적용해 선택

## 7. 수치가 의사결정으로 이어져야 합니다

숫자를 많이 보여주는 것이 목적이 아닙니다.

좋은 수치는 `그래서 무엇을 결정했는가`로 이어집니다.

예:

- 40.7% 속도 개선에도 정확도가 33%p 떨어져 제품 미채택
- 분류 결과가 나와도 실제 수수료를 적용하면 손실이므로 거래 전략으로 부적합
- latency breakdown 결과 STT가 주 bottleneck이라 이후 최적화 우선순위를 STT로 설정

## 8. 기여 범위와 성과 귀속을 분리합니다

반드시 구분합니다.

- 내가 직접 작성하거나 수정한 코드
- 팀원이 주도하고 내가 통합 또는 추가 검증한 영역
- 기존 공개 baseline
- 팀 전체 결과
- 회사/제품 전체 결과
- 프로젝트 종료 이후 누적된 후속 결과

예:

- 회사 사업계획서의 `누적 키트 판매 1,830개`는 제품 후속 누적 실적이지 개인 판매 실적이 아님
- STT Lead가 별도로 있으면 `STT 전체 담당`이 아니라 `후반 candidate sweep과 production 선정에 직접 참여`
- 공개 CNN RTL을 사용했다면 `CNN Accelerator 전체 RTL을 처음부터 설계`라고 쓰지 않음

강하게 보이는 표현보다 검증 가능한 기여 범위를 우선합니다.

## 9. 근거 상태를 구분합니다

각 수치와 주장에는 가능한 한 출처 상태를 파악합니다.

예:

- Git commit/diff로 직접 확인
- final log로 확인
- 발표자료로 확인
- 제출 보고서로 확인
- 당시 측정 후 경험기술서에 기록된 값
- 현재 회사/제품의 후속 누적 실적
- 원본 benchmark sheet 추가 확인 필요

원본 Excel이나 log가 유실됐더라도 당시 기록값을 무조건 폐기하지는 않습니다. 대신 원본 근거가 현재 남아 있지 않다는 상태를 문서에 기록합니다.

근거가 없는 값을 기억만으로 새로 만들어내지 않습니다.

## 10. commit 기반으로 경험을 복원합니다

README만 읽고 경험을 확정하지 않습니다.

권장 순서:

1. 프로젝트 시작 시점의 baseline과 기존 구조 확인
2. 본인 author commit 분리
3. 문제를 처음 드러낸 commit 확인
4. 중간 수정 commit 확인
5. 최종 code, log, test 결과 확인
6. 발표자료, 보고서, Drive 자료와 교차 검증
7. 상황 → 문제 → 판단 → 조치 → 결과로 복원

commit message보다 실제 diff를 우선합니다.

예:

- `fix done` → 실제 diff에서 `weight.detach()`가 `weight().int_repr()`로 바뀐 이유 확인
- `refactoring for synth` → `reg + always`가 `wire + generate + assign`으로 변경된 것을 확인
- `sidecar` → requirements 파일에서 서로 다른 `transformers` 버전 충돌 확인

## 11. 실패와 미완료를 성공처럼 포장하지 않습니다

좋은 경험기술서는 검증 한계를 함께 기록합니다.

예:

- synthesis/P&R final stage까지 진행했으나 timing closure 실패
- gate-level 비교 환경까지 구축했으나 정상 waveform을 얻지 못해 Netlist accuracy 미측정
- 무수수료 조건에서는 수익이었으나 실제 fee 적용 후 전략 붕괴

실패한 경우 다음을 남깁니다.

- 어디까지 성공했는가
- 어디서 실패했는가
- 무엇을 확인했는가
- 이후 어떤 기준을 먼저 볼 것인가

## 12. 상세 문서 상단 형식을 통일합니다

가능하면 `experiences/*.md` 상단을 다음 형식으로 유지합니다.

```text
# 프로젝트명

## 채용용 경험 요약

한 줄 요약

- 기간
- 팀 규모
- 역할
- 기술

### 상황 → 문제 → 판단 → 조치 → 결과

상황
문제
판단
조치
결과

## 핵심 수치

...
```

그 아래에는 architecture, commit history, 세부 구현, source provenance, 면접에서 주의할 주장 등을 자유롭게 보존합니다.

## 13. 최종 체크리스트

- [ ] 기간이 있는가?
- [ ] 팀 규모가 있는가?
- [ ] 본인 담당 범위가 명확한가?
- [ ] 기존 baseline과 직접 구현 범위가 구분되는가?
- [ ] 상황 → 문제 → 판단 → 조치 → 결과가 모두 있는가?
- [ ] 결과가 성공인지 실패인지 분명한가?
- [ ] 비교 실험이라면 최종 선택과 이유가 있는가?
- [ ] 수치에 필요한 측정 조건이 붙어 있는가?
- [ ] 다른 조건의 수치를 잘못 직접 비교하지 않았는가?
- [ ] 팀/회사 성과를 개인 성과처럼 쓰지 않았는가?
- [ ] 후속 누적 실적을 당시 개인 성과와 구분했는가?
- [ ] 기술적 원인은 실제 commit/diff 또는 원본 자료로 확인했는가?
- [ ] 원본이 유실된 수치는 근거 상태를 표시했는가?
- [ ] 실패한 검증을 성공처럼 쓰지 않았는가?
- [ ] `resume.md`에는 강한 근거만 남기고 세부 내용은 `experiences/`로 보냈는가?
