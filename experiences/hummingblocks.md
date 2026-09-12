# HummingBlocks

## 채용용 경험 요약

시각장애 학습자를 출발점으로 일반·특수교육 환경에서 함께 사용할 수 있도록, 실물 코딩 블록의 AI 인식 결과를 실행 가능한 프로그램과 음악으로 변환하는 Android 제품을 개발했습니다. 공동 창업자이자 Mobile App Developer로서 Camera/CV 연동, 블록 배치 해석, 음악 실행, 접근성, 저장·내보내기, 출시 후 데이터 호환성까지 제품 파이프라인을 맡았습니다.

## Project at a glance

| 항목 | 내용 |
|---|---|
| 기간 | 2022.03 ~ 2023.10 주요 제품 개발, 2025.01까지 유지보수 커밋 확인 |
| 팀/역할 | 네모감성 공동 창업자 · Mobile App Developer |
| 제품 | 실물 블록을 촬영해 순차·반복·조건·함수·연산 규칙을 음악으로 실행하는 Android 교육제품 |
| 담당 | Android 앱, Camera/CV integration, detection 후처리, 프로그램 해석, 음악 실행, 접근성, 저장·migration |
| 기술 | Android, Java, Python, YOLO, OpenCV DNN, Chaquopy, TensorFlow Lite, CameraX, FFmpeg |
| 출시 | Google Play, 2023.10.31 |

Google Play: https://play.google.com/store/apps/details?id=com.nemo.hummingblocks  
Code: https://github.com/nanocode00/nemo_codeblock

## 제품과 사용자

허밍블럭스는 화면 안에서만 조작하는 코딩교육 대신, 형태와 점자로 구분할 수 있는 실물 블록을 연결하고 카메라로 촬영해 음악을 만드는 제품입니다. 초기 개발 문서와 2023년 현장 기록에는 시각장애 학습자가 명시되어 있으며, 이후 일반 초·중등과 특수교육 환경을 함께 포괄하는 제품으로 확장됐습니다.

사용자는 시작, 반복, 조건, 함수, 연산, 악기·단계 블록을 배치합니다. 앱은 사진에서 블록을 검출한 뒤 배치 순서를 프로그램으로 해석하고, 드럼·브라스·기타·피아노·베이스 트랙의 실행 계획으로 변환합니다. 따라서 검출 정확도뿐 아니라 **블록 간 관계를 올바르게 해석하고 잘못된 프로그램을 실행 전에 설명하는 것**이 제품 품질의 핵심이었습니다.

## 내가 맡은 범위

### 직접 담당한 영역

- Android 앱의 화면 흐름과 Camera/CV inference 연동
- 검출된 class와 bounding box를 순서·행·열 구조로 바꾸는 후처리
- PLAY/LOOP/STAR, 조건 분기, 악기 단계 연산을 실행 상태로 변환하는 로직
- 여러 악기 트랙의 스케줄링, BPM, pause/resume, 진행 표시
- TalkBack, 접근성 탐색 순서, 촬영 방향 안내, 진동·설정
- 음악 저장·공유, MP3 생성, CameraX 영상 녹화, 영상·음원 결합
- 진행도 JSON과 버전 migration을 통한 기존 사용자 데이터 호환
- YOLO 학습 데이터 라벨링과 현장 인식 문제를 AI 담당자에게 전달

### 팀 기여로 구분하는 영역

- YOLO 모델 학습 전체와 모델 품질 개선
- 실물 블록의 산업 디자인·금형·제조
- UI 디자인 원본과 음악 콘텐츠 제작
- 회사의 영업·교육 운영 및 개발 이후 누적 판매 성과

## 제품 파이프라인

1. `CameraActivity`가 블록 배치를 촬영하고 입력 경로를 준비합니다.
2. 제품 브랜치에서는 Chaquopy를 통해 Python `Classifier.py`를 호출합니다.
3. OpenCV DNN이 Darknet weight/cfg로 416×416 입력을 추론하고 confidence filtering과 NMS를 수행합니다.
4. bounding box를 x/y 좌표 기준으로 정렬·그룹화해 블록의 공간 구조를 만듭니다.
5. Java `Runner`가 class ID 묶음을 PLAY/LOOP/STAR와 조건·연산 규칙으로 해석하고 유효성을 검사합니다.
6. `MusicInfo`와 `MusicPlayer`가 실행 계획을 악기 트랙, BPM, 진행 상태로 변환해 재생합니다.
7. 완성된 결과는 JSON·MP3로 저장하거나 CameraX 영상과 결합할 수 있습니다.

## Engineering Case 1 — 검출 결과를 실행 가능한 프로그램으로 변환

### 문제

YOLO 출력은 각 블록의 class, confidence, bounding box일 뿐입니다. 한 사진 안의 여러 블록을 어떤 순서로 읽고, 어느 제어 블록에 속하는지, 어떤 악기를 몇 단계로 실행할지 결정하는 별도 도메인 로직이 필요했습니다.

### 구현

- 검출 box의 평균 너비·높이를 기준으로 x/y 방향의 가까운 블록을 그룹화했습니다.
- x축 순서와 그룹 내부 y축 순서를 조합해 블록의 읽기 순서를 구성했습니다.
- 시작 시 한 번 실행하는 PLAY, 반복 실행하는 LOOP, 재사용 가능한 STAR 구역을 분리했습니다.
- 좌·우 기울기 조건과 ELSE, 악기·STAR 단계의 증가·감소·유지·초기화를 상태 변화로 해석했습니다.
- 실행 전에 중복 제어 블록, 초기화 전 연산, STAR 정의 오류, IF 없는 ELSE 등을 검사했습니다.

현재 제품 코드에는 **오류 코드 23종과 경고 메시지 2종**이 UI 리소스와 함께 정의되어 있습니다. 기존 문서의 `22가지 예외`는 집계 기준이 확인되지 않아 사용하지 않습니다.

### 의미

모델을 Android에 붙이는 데서 끝나지 않고, 비정형 vision 출력을 사용자가 이해할 수 있는 코딩 규칙과 결정적 실행 상태로 변환했습니다. 인식 실패와 문법 오류도 단순 크래시가 아니라 사용자가 블록 배치를 수정할 수 있는 안내로 연결했습니다.

## Engineering Case 2 — 모바일 추론 최적화와 미채택 결정

### 상황과 가설

제품 mainline은 `Android Java → Chaquopy → Python/OpenCV DNN → Darknet weights/cfg` 경로였습니다. 당시 Galaxy S9+ 측정 기록에서 촬영 후 평균 처리 시간이 9.1초였고, Python bridge를 제거한 Java/TensorFlow Lite 경로가 지연을 줄일 수 있다고 판단했습니다.

### 실험 구현

mainline을 바로 교체하지 않고 `app_develop_tflite` 브랜치에서 다음을 실제로 구현했습니다.

- Chaquopy/Python runtime 제거 및 TensorFlow Lite 2.9 연동
- 416×416 bitmap을 RGB float buffer로 변환
- `Interpreter.runForMultipleInputsOutputs()` 기반 추론
- Java confidence filtering, class별 NMS, IoU 계산
- bounding box 정렬·그룹화와 프로그램 해석을 Java로 이식
- NNAPI → GPU delegate → 4-thread CPU fallback
- 촬영 bitmap의 비율 유지, 중앙 정렬, 좌우 padding 전처리 보정

`7cdc9e1`과 `fc771fd`에 전환과 전처리 보정이 남아 있으며, 같은 날 제품 브랜치의 `1bb3f7c`에서 TFLite 코드를 제거하고 Chaquopy/Python 경로를 복원했습니다. 실험 브랜치는 mainline에 병합되지 않은 상태로 남아 있습니다.

### 결과와 판단

당시 기록값은 다음과 같습니다.

| 항목 | 기존 경로 | TFLite 경로 | 변화 |
|---|---:|---:|---:|
| 평균 처리 시간 | 9.1s | 5.4s | -40.7% |
| 인식 정확도 | 90% | 57% | -33%p |

속도는 개선됐지만 잘못 읽은 블록이 다른 프로그램과 음악으로 이어지는 제품에서는 정확도 저하 비용이 더 컸습니다. 따라서 빠른 구현을 채택하는 대신, 사용자 결과의 신뢰성을 우선해 기존 runtime을 유지했습니다.

> 위 성능·정확도 수치는 당시 실험 결과를 이후 경험기술서에 남긴 값입니다. Git으로 구현과 롤백은 검증되지만 원본 benchmark spreadsheet, 데이터셋, 산출 방식은 아직 확인되지 않았으므로 상세 면접에서는 `당시 기록값`이라고 구분합니다.

## Engineering Case 3 — 음악 실행과 콘텐츠 생성

### 실시간 음악 실행

2023년 7월 `7d823b7`에서 `ScheduledThreadPoolExecutor`와 `ScheduledFuture`를 이용한 재생 흐름을 구성했습니다. 두 MediaPlayer 그룹을 교대로 prepare/start해 여러 악기 트랙을 이어 재생하고, pause 시 예약 작업을 취소한 뒤 현재 phase와 남은 delay를 저장해 resume 시점에 재예약했습니다. 이후 BPM에 따라 section 길이와 전체 진행률을 계산하도록 확장했습니다.

### 저장과 내보내기

- 2024년 5월: 한 section의 악기 트랙을 FFmpeg `amix`로 병합하고 section 결과를 `concat`해 MP3 생성
- 2024년 12월: CameraX VideoCapture로 구간 영상을 녹화하고 임시 영상을 concat한 뒤 완성 음악을 mux
- 2025년 1월: 저장 음악 재생 상태, 영상 제작 진입, 중단·정리 흐름 개선

이로써 블록 인식 결과가 일회성 재생에 그치지 않고 사용자가 보관·공유하는 음악과 영상 콘텐츠로 이어졌습니다.

## Engineering Case 4 — 출시 이후 접근성과 데이터 호환성

### 접근성

- TalkBack announcement와 접근성 탐색 순서 설정
- 로딩·촬영 상태에서 불필요한 요소의 focus 제어
- 블록이 화면 밖에 있으면 좌·우·상·하 카메라 이동 방향을 화면과 음성으로 안내
- 진동 설정과 BPM 버튼의 접근 가능 상태 관리
- 시각장애인용 별도 앱 모듈의 촬영·선택·재생 흐름 개선

2023년 과거 사례 폴더에는 시각장애인 1인 테스트와 서울맹학교 사용 기록이, 2024년 APK 폴더에는 스크린리더 최적화·자동촬영·시각장애인용 업데이트 결과물이 남아 있습니다.

### 데이터와 버전 호환

- `progress.json`에 튜토리얼 완료 여부와 음악별 퀘스트 진행도를 저장
- `version.json`으로 설치된 데이터 버전을 확인
- 구버전 음악·block JSON을 신규 구조로 변환
- 교체가 필요한 weight/music asset을 갱신하고 MP3를 재생성

출시 후 기능 구조가 바뀌어도 기존 사용자의 저장 음악과 진행 상태를 잃지 않도록 migration 경로를 구현했습니다.

## 개발 타임라인

| 시기 | 변경 | 근거 |
|---|---|---|
| 2023.07 | 음악 재생 스케줄링, pause/resume, 진행 표시 | `7d823b7` |
| 2023.09~10 | BPM, TalkBack, 접근성 순서, 진동·설정 | `5a8fcad`, `44037d8`, `7a06422`, `28468a6` |
| 2023.10.31 | Google Play 출시 | 회사 사업계획서의 출시 기록 |
| 2023.11 | Java/TFLite 전환·전처리 보정·롤백 | `7cdc9e1`, `fc771fd`, `1bb3f7c` |
| 2024.05 | MP3 생성·공유 | `ce30682` |
| 2024.07 | 진행도 저장 구조와 퀘스트 흐름 개편 | `28bee75` |
| 2024.10 | 시각장애인용 촬영·재생 흐름 업데이트 | `2c0cf19` |
| 2024.12 | 데이터·asset 버전 migration | `36f8253`, `6d66423`, `fbd31c9` |
| 2024.12~2025.01 | CameraX 녹화, 영상 concat·음악 mux, 저장 음악 개선 | `d85faeb`, `719732b` |

## 결과와 성과

### 개발 기간의 직접 결과

- Android 앱의 Camera → CV → 프로그램 해석 → 음악 실행 파이프라인 구현
- 2023.10.31 Google Play 출시 후 2.x 기능과 기존 사용자 데이터 호환성 유지
- 2022 제1회 전국 장애·비장애 대학생 창업경진대회 대상: 상장에 김재훈을 포함한 4인 팀 명시
- 2022 KNU 창업경진대회 대상: 상장에 김재훈을 포함한 3인 팀 명시
- 2022 소셜벤처 경연대회 TS청년벤처상·대구광역시장상: 네모감성 팀 수상

### 개발 이후 제품의 후속 누적 성과

2026년 5월 회사 사업계획서 기준 수치로, 개인의 직접 판매 실적과 구분합니다.

- 누적 앱 다운로드 1,500+
- 누적 키트 판매 1,830개
- 자사 판매 기관 약 80개 학교
- 제품 도달 기관 약 200개 학교
- 2025 CES Innovation Awards Honoree 등 후속 제품 성과

## 근거 수준과 주의할 표현

| 주장 | 근거 | 사용 원칙 |
|---|---|---|
| Android/CV·음악·접근성·migration 구현 | 코드와 `nanocode00` 커밋 | 직접 기여로 표현 가능 |
| Co-founder · Mobile App Developer | 본인 졸업 포트폴리오·자기소개 자료 | 역할명으로 사용 가능 |
| 9.1s → 5.4s, 90% → 57% | 이후 경험기술서의 당시 기록 | 원본 benchmark 부재를 인지하고 사용 |
| 오류 23종·경고 2종 | `Runner.java`, 한국어 UI 리소스 | `22가지 예외` 대신 현재 코드 기준 사용 |
| YOLO 모델 개선 | 팀의 AI 담당 영역 | 라벨링·현장 피드백·앱 연동만 직접 기여로 표현 |
| 1,830개·약 200개 학교 | 2026.05 회사 사업계획서 | 제품의 후속 누적 성과로만 표현 |

Android runtime을 `PyTorch native`라고 설명하지 않습니다. 제품 mainline은 Chaquopy + Python/OpenCV DNN + Darknet weight/cfg 구조이며, TensorFlow Lite는 별도 브랜치에서 구현 후 미채택한 실험입니다.

## 이력서용 핵심 bullet 후보

- 공동 창업자·Mobile App Developer로서 실물 코딩 블록의 `Camera → YOLO/OpenCV → 배치 해석 → 음악 실행` Android 파이프라인을 구현하고 Google Play 출시와 2.x 유지보수까지 수행
- 검출 box를 순서·반복·조건·함수·연산으로 변환하고 오류 23종·경고 2종을 실행 전에 안내하는 도메인 후처리 구현
- Java/TFLite inference·NMS·좌표 후처리를 별도 브랜치에 이식해 Galaxy S9+ 기준 9.1s → 5.4s로 단축했으나 정확도 90% → 57% 저하를 확인해 제품 신뢰성을 기준으로 기존 runtime 유지
- 출시 후 음악 스케줄링과 BPM, FFmpeg MP3·영상 export, TalkBack·촬영 안내, 진행도·버전 migration까지 확장

## 예상 면접 질문

- bounding box를 어떤 기준으로 행·열과 실행 순서로 묶었는가?
- PLAY/LOOP/STAR와 조건·연산을 어떤 상태 구조로 해석했는가?
- 오류 23종과 경고 2종은 어디에서 검사하고 어떻게 사용자에게 보여주는가?
- TFLite 전환에서 정확도가 떨어진 원인을 당시 어떻게 추적했는가?
- pause/resume 시 예약된 음악과 남은 시간을 어떻게 복구했는가?
- 데이터 migration 중 구버전 JSON과 생성된 MP3를 어떻게 다뤘는가?
- 시각장애 사용자 테스트가 Camera와 TalkBack UX에 어떤 변경으로 이어졌는가?
