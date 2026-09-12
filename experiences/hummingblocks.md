# HummingBlocks

## 채용용 경험 요약

실물 코딩 블록을 촬영하면 AI가 블록의 종류와 배치를 인식하고 음악으로 실행하는 시각장애인 대상 코딩교육 제품을 개발해 Google Play 출시와 실제 판매까지 연결한 경험입니다.

- 기간: **2022.03경 ~ 2023.10 주요 제품 개발**, 출시 이후 유지보수 지속
- GitHub에서 직접 개발 및 유지보수가 확인되는 기간: 적어도 **2023.06 ~ 2025.01**
- 팀: 네모감성 창업팀. 2022년 공개 기사에서 김재훈을 포함한 핵심 구성원 **4명 이상**이 확인됨
- 담당: Android 앱 개발, Camera/CV inference integration, YOLO 결과 후처리, 음악 실행 로직, UI/접근성, 출시 후 유지보수
- 기술: Android, Java, Python, YOLO, Chaquopy, OpenCV, TensorFlow Lite, CameraX, FFmpeg

### 상황 → 문제 → 판단 → 조치 → 결과

**상황**  
실물 블록을 촬영해 YOLO 검출 결과를 음악 실행 규칙으로 변환하는 Android 앱을 제품화했습니다. 실제 mainline runtime은 Chaquopy로 Java와 Python을 연결하고 Python/OpenCV DNN에서 Darknet weight/cfg를 읽는 구조였습니다.

**문제**  
당시 Galaxy S9+에서 기존 방식의 평균 처리 시간이 **9.1초**로 길어 촬영 후 결과를 기다리는 시간이 컸습니다.

**판단**  
Python bridge를 제거하고 TensorFlow Lite 모델을 Java에서 직접 실행하면 모바일 추론 지연을 줄일 수 있다고 보고, mainline을 바로 바꾸지 않고 별도 `app_develop_tflite` branch에서 실제 전환을 검증했습니다.

**조치**  
`7cdc9e1` 커밋에서 Chaquopy/Python runtime을 제거하고 TFLite 2.9, GPU delegate, Select TF Ops를 추가했습니다. 416×416 bitmap 입력, Java `Classifier`, confidence filtering, class별 NMS, 좌표 정렬/그룹화 후처리, NNAPI → GPU → CPU fallback까지 Android 쪽으로 옮겼습니다. 이후 `fc771fd`에서는 crop/scale/border 계산을 다시 수정해 입력 전처리를 보정했습니다.

**결과**  
평균 처리 시간은 **9.1s → 5.4s, 약 40.7% 단축**됐지만 인식 정확도가 **90% → 57%, 33%p 하락**했습니다. 허밍블럭스는 잘못 인식한 블록이 그대로 잘못된 음악 실행으로 이어지므로 속도보다 정확도를 우선해야 한다고 판단해 TFLite 전환을 철회하고 기존 Chaquopy/Python/OpenCV 경로를 유지했습니다.

> 9.1s / 5.4s, 90% / 57%는 당시 실험 결과를 이후 경험기술서에 기록해 둔 값입니다. Git에는 TFLite 전환 코드와 기존 runtime timing 계측 흔적이 남아 있지만, 원본 benchmark spreadsheet는 현재 Git 전체 branch snapshot과 주요 Drive 폴더 검색에서는 아직 확인하지 못했습니다.

## 핵심 수치

| 항목 | 결과 | 사용 시 주의 |
|---|---:|---|
| 기존 방식 평균 처리 시간 | 9.1s | Galaxy S9+, 후대 경험기술서 기록값 |
| TFLite 평균 처리 시간 | 5.4s | 약 40.7% 단축 |
| 기존 방식 인식 정확도 | 90% | 원본 benchmark sheet 추가 확인 필요 |
| TFLite 인식 정확도 | 57% | 기존 대비 -33%p |
| 후처리 예외 상황 | 22가지 | 앱 로직에서 안내 처리 |
| 현재 Google Play 공개 다운로드 | 1K+ | 현재 제품 누적 실적 |
| 2026.05 사업계획서 누적 키트 판매 | 1,830개 | 개인 개발기간 직접 판매량이 아닌 제품 후속 누적 실적 |
| 2026.05 사업계획서 누적 교육기관 | 200곳 | 제품 후속 누적 실적 |

Google Play: https://play.google.com/store/apps/details?id=com.nemo.hummingblocks

## 제품 구조

- 실물 코딩 블록 키트: 유료 판매
- Android 연동 앱: Google Play 무료 제공
- 카메라 입력 → YOLO 객체 검출 → 위치/종류 후처리 → 음악 실행

## 맡은 역할

- Android 앱 개발 학습부터 프로토타입, 기능 개선, 출시와 운영까지 수행
- YOLO 학습용 데이터 라벨링
- 카메라 입력, 모델 추론, 후처리, 음악 재생 기능 연결
- 외부 디자이너와 음원 제작자의 결과물을 앱에 적용할 수 있는 형태로 정리
- 사용 흐름 및 화면 비율 문제 개선
- 출시 이후 음악 재생, 데이터 저장, 버전 마이그레이션, 녹화 및 접근성 기능 등을 지속적으로 개선

## 왜 YOLO를 사용했는가

한 장의 사진에 여러 코딩 블록이 동시에 배치되기 때문에 여러 객체를 한 번에 검출할 필요가 있었습니다. 서비스에 적용할 수 있는 수준의 속도를 확보하면서 블록 종류와 위치를 함께 얻기 위해 YOLO를 사용했습니다.

## 검출 결과를 실행 규칙으로 바꾸기

YOLO의 출력은 블록 종류와 좌표이므로 그 자체로 음악을 실행할 수 없었습니다.

위치 좌표를 기준으로 블록의 나열 순서와 전체 배치를 분석하고 다음 정보로 변환했습니다.

- 어떤 악기가 어떤 순서로 연주되는지
- 시작할 때 한 번 실행할지 반복할지
- 블록의 좌우 기울기에 따라 실행이 어떻게 달라지는지
- 템포, 시퀀스, 블록 종류 등 음악 재생에 필요한 정보

TFLite 실험 브랜치에서도 이 후처리를 Java로 옮겨 검증했습니다. TFLite raw output에서 confidence가 높은 detection을 고른 뒤 class별 NMS를 수행하고, bounding box를 x/y 방향으로 정렬·그룹화해 블록 실행 순서로 만드는 로직이 남아 있습니다.

## 예외 처리 22가지

후처리 알고리즘을 설계하면서 정상 실행이 어려운 상황을 22가지로 정리했습니다.

예:

- 시작 또는 반복을 결정하는 블록이 없는 경우
- 사용하는 변수가 초기화되지 않은 경우

후처리 단계에서 예외를 검사하고 사용자가 원인을 알 수 있도록 화면에 안내 메시지를 표시했습니다.

## 모바일 추론 방식 비교: Chaquopy vs TensorFlow Lite

### 기존 방식

당시 작성한 경험기술서에는 원본 모델을 PyTorch 기반 YOLO로 기록했습니다. Android의 실제 mainline runtime은 Chaquopy로 Java와 Python을 연결하고 Python/OpenCV DNN에서 Darknet weight/cfg를 읽어 추론하는 구조였습니다.

최종 `app_develop`에도 `com.chaquo.python` 플러그인, Python의 `numpy`와 `opencv-python`, `Runner.classifier.callAttr("classify", ...)` 호출 경로가 남아 있습니다.

### TFLite 전환 실험

2023년 11월경 별도 `app_develop_tflite` branch에서 TFLite 방식으로 실제 전환을 시도했습니다.

Git에서 확인되는 변경은 다음과 같습니다.

- `com.chaquo.python` 플러그인과 Python runtime 설정 제거
- TensorFlow Lite, TensorFlow Lite GPU, Select TF Ops 의존성 추가
- `nemo_best.tflite` 모델을 Android Java 코드에서 직접 로드
- 입력 이미지를 416×416 RGB float buffer로 변환
- `Interpreter.runForMultipleInputsOutputs()`로 추론
- Java에서 confidence filtering 및 class별 NMS 구현
- Java에서 bounding box 위치를 x/y 방향으로 정렬하고 그룹화하는 후처리 구현
- NNAPI → GPU delegate → 4-thread CPU 순으로 fallback하는 실행 구조 시험

### 최종 선택

속도는 약 40.7% 개선됐지만 정확도가 크게 떨어져 TFLite 방식은 mainline에 채택하지 않았고 기존 Chaquopy 기반 구조를 유지했습니다. 실제 TFLite 구현은 실험 branch에 남아 있고 이후 `app_develop` 2.2.0 코드까지 Chaquopy/Python/OpenCV 경로가 유지되는 것이 Git으로 확인됩니다.

## 제품을 출시한 뒤에도 계속 수정한 내용

### 2023년 6~7월: 음악 재생 스케줄링

`7d823b7`에서는 여러 악기 트랙을 연속 재생하기 위해 기존 Timer 중심 흐름을 `ScheduledThreadPoolExecutor` / `ScheduledFuture` 기반으로 바꾸고, 두 MediaPlayer 그룹을 교대로 prepare/start하도록 수정했습니다. pause/resume 시 현재 phase와 남은 delay를 복원하는 로직도 포함됩니다.

### 2023년 9~10월: BPM 및 접근성

- section별 BPM 상태를 음악 데이터에 포함
- BPM에 따라 section 길이와 전체 진행률을 동적으로 계산
- TalkBack 지원, 결과 확인 화면, 진동 및 설정 기능 추가

### 2023년 11월: TFLite 전환 실험

- `7cdc9e1`: Java/TFLite inference + postprocessing 전환
- `fc771fd`: TFLite 입력 이미지 crop/scale/border 계산 보정
- 속도 개선과 정확도 저하를 비교한 뒤 mainline에는 미채택

### 2024년 5월: FFmpeg 기반 MP3 저장

`ce30682`에서 `mobile-ffmpeg`를 연동해 한 section의 여러 악기 트랙을 `amix`로 병합하고 section별 결과를 `concat`하여 하나의 MP3로 내보내는 export pipeline을 구현했습니다.

### 2024년 7월: 사용자 진행도 구조 리팩터링

`28bee75`에서 `progress.json`을 중심으로 튜토리얼, 퀘스트, 악기 해금 상태를 재설계하고 음악 JSON의 level, replace, genre 정보를 구조화했습니다.

### 2024년 10월: 시각장애인용 CV 흐름 개선

`2c0cf19`에서 Python/OpenCV DNN detection 호출과 검출 box 정렬·그룹화 후처리를 수정하고 시각장애인용 촬영/결과 흐름과 연결했습니다.

### 2024년 12월: 버전 마이그레이션

- `36f8253`: `version.json`과 `VersionUtil` 도입, 기존 저장 데이터를 신규 구조로 변환
- `6d66423`: migration 과정의 파일명·반복문·nested block 오류 수정
- `fbd31c9`: migration/version 처리 모듈화

기존 사용자의 음악/block 데이터를 새 구조로 변환하고 필요한 MP3를 재생성했습니다.

### 2024년 12월 ~ 2025년 1월: 녹화와 촬영 접근성

`d85faeb` 이후 CameraX VideoCapture 기반 구간별 영상 녹화, FFmpeg 영상 concat/음악 mux, CameraX ImageAnalysis + ML Kit Barcode 기반 실시간 촬영 안내 등을 추가했습니다. 블록 위치에 따라 좌/우/상/하 이동 방향을 화면 애니메이션과 TalkBack으로 안내하도록 구성했습니다.

## 실제 사용 환경에서 발견한 문제

학습 데이터는 주로 실내에서 촬영했지만 야외 일몰 환경에서는 태양빛 때문에 블록 색이 달라져 인식하지 못하는 문제가 발생했습니다.

문제 상황과 촬영 조건을 AI 엔지니어에게 공유했고 해당 환경을 반영한 변형 데이터를 추가해 모델을 보완했습니다.

> 정확한 augmentation 방식은 원본 기록을 추가 확인하기 전까지 확정해서 쓰지 않습니다.

## UI 및 사용 흐름 개선

- 메인 화면에서 음악 선택 화면으로 바로 진입하도록 흐름 단축
- 기기별 화면 비율 차이로 버튼이 작게 표시되는 문제 개선
- UI 기준 비율을 고정하고 실제 화면 크기에 맞춰 요소 크기 조정
- 튜토리얼, 퀘스트, 악기 해금, 저장 음악 화면을 추가하고 반복적으로 사용 흐름 수정

## 외부 리소스 적용

- 전달된 음원 파일 누락 여부 확인 및 추가 요청
- 비어 있는 음원 파일 보완
- 음원 길이 확인 후 실행에 필요한 BPM 값 요청
- 통일되지 않은 파일명을 앱 구조에 맞게 정리
- 디자인 이미지를 필요한 크기와 위치에 맞게 가공
- 리소스 전달 지연 시 GIF frame을 PNG로 가공해 개발 일정이 멈추지 않도록 대응

## 결과

### 개인 개발 기간에서 직접 확인되는 결과

- 전국 장애·비장애 대학생 창업경진대회 대상
- Google Play 출시
- 무료 연동 앱과 유료 블록 키트 형태로 운영 및 판매
- 출시 이후 2.x 버전까지 기능, 사용자 데이터 호환성, 접근성 기능 지속 개선

### 제품의 후속 누적 결과

- 현재 Google Play: **1K+ 다운로드**
- 2026년 5월 네모감성 사업계획서: **누적 키트 판매 1,830개, 누적 교육기관 200곳**

이 후속 누적 수치는 제품이 이후에도 운영되며 쌓인 회사/제품 실적이므로, 김재훈 개인의 개발 기간 중 직접 판매량으로 표현하지 않습니다.

## 이력서용 핵심 bullet

- 시각장애인 대상 AI 코딩교육 Android 제품에서 Camera → YOLO → 위치/종류 후처리 → 음악 실행을 통합하고 Google Play 출시·운영까지 수행
- Galaxy S9+에서 Chaquopy/Python 경로를 TFLite+Java로 이식해 처리 시간을 **9.1s → 5.4s(-40.7%)**로 줄였으나 정확도가 **90% → 57%(-33%p)**로 하락해 정확도를 우선해 기존 runtime 유지
- 출시 후 음악 스케줄링, BPM, FFmpeg MP3/MP4 export, 데이터 migration, TalkBack·CameraX 촬영 접근성까지 2.x 버전 유지보수

## 면접에서 주의할 표현

- Android runtime 자체를 `PyTorch native`라고 표현하지 않음. 실제 mainline은 Chaquopy + Python/OpenCV DNN + Darknet weight/cfg
- 9.1s/5.4s, 90%/57%의 원본 spreadsheet를 찾기 전에는 당시 기록값이라고 구분
- 현재 누적 키트 1,830개를 개인이 직접 1,830개 판매했다고 표현하지 않음
- 전체 YOLO 모델 학습을 혼자 담당했다고 주장하지 않음. 직접 확인되는 범위는 labeling, 앱 inference/postprocessing integration, 현장 문제 피드백

## Git 근거 메모

대표적으로 확인한 `nanocode00/nemo_codeblock` 커밋 및 branch:

- `app_develop_tflite`: TFLite 전환 실험 branch
- `7d823b7`: `new sound play algorithm / progress and input plot`
- `5a8fcad`: `new feature: bpm control`
- `7cdc9e1`: `update 1.0.2(undone)` — TFLite/Java 전환 실험
- `fc771fd`: `update 1.0.2(undone)` — TFLite 입력 이미지 전처리 보정
- `ce30682`: `mp3 extract feature`
- `28bee75`: `remake progress save and dialogs`
- `2c0cf19`: `blind app update`
- `36f8253`: `add version migration`
- `6d66423`: `migration fix`
- `fbd31c9`: `version modulize`
- `d85faeb`: `Update 2.2.0 - recording video`
- `719732b`: `ui fixes / saved music play logic update`

### 출처 구분

- GitHub가 직접 증명하는 것: 구현 구조, branch 분리, 작성자, 대략적인 개발 시점, mainline에 최종 남은 기술 구조
- 당시 자소서/경험기술서가 증명하는 것: Galaxy S9+ 측정 조건, 9.1초 → 5.4초, 정확도 90% → 57%, 정확도를 우선해 TFLite를 채택하지 않았다는 판단
- 현재 공개 자료가 증명하는 것: Google Play 1K+ 다운로드
- 2026년 5월 사업계획서가 증명하는 것: 누적 키트 판매 1,830개, 누적 교육기관 200곳
- 아직 추가 원본 확인이 필요한 것: benchmark spreadsheet, 정확도 측정 데이터셋과 산출 방식, 야외 환경 보완 시 사용한 정확한 augmentation 기법
