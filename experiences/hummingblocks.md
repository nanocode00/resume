# HummingBlocks

## 한 줄 요약

실물 코딩 블록을 촬영하면 AI가 블록의 종류와 배치를 인식하고 음악으로 실행하는 시각장애인 대상 코딩교육 제품을 개발해 Google Play 출시와 실제 판매까지 연결한 경험.

## 기간

- 프로젝트 시작 및 초기 제품화 기록: 2022년 3월경 ~ 2023년 10월경
- GitHub `app_develop`에서 직접 개발 및 유지보수가 확인되는 기간: 적어도 2023년 6월 ~ 2025년 1월

> 커밋을 작업 직후 남기지 않은 경우가 있어 Git 날짜를 정확한 작업일로 보지는 않습니다. 이후 이력서에는 프로젝트 시작 시점과 출시 이후 유지보수 기간을 구분해서 쓰는 편이 안전합니다.

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

TFLite 실험 브랜치에서도 이 후처리를 Java로 옮겨 검증했습니다. TFLite의 raw output에서 confidence가 높은 detection을 고른 뒤 class별 NMS를 수행하고, 바운딩 박스를 x축과 y축으로 정렬·그룹화해 블록 실행 순서로 만드는 로직이 남아 있습니다.

## 예외 처리 22가지

후처리 알고리즘을 설계하면서 정상 실행이 어려운 상황을 22가지로 정리했습니다.

예:

- 시작 또는 반복을 결정하는 블록이 없는 경우
- 사용하는 변수가 초기화되지 않은 경우

후처리 단계에서 예외를 검사하고 사용자가 원인을 알 수 있도록 화면에 안내 메시지를 표시했습니다.

## 모바일 추론 방식 비교: Chaquopy vs TensorFlow Lite

### 기존 방식

당시 작성한 경험기술서에는 원본 모델을 PyTorch 기반 YOLO로 기록했습니다. Android의 실제 mainline runtime은 Chaquopy로 Java와 Python을 연결하고, Python/OpenCV DNN에서 Darknet weight/cfg를 읽어 추론하는 구조였습니다.

GitHub의 최종 `app_develop`에도 `com.chaquo.python` 플러그인, Python의 `numpy`와 `opencv-python`, `Runner.classifier.callAttr("classify", ...)` 호출 경로가 그대로 남아 있습니다.

### TFLite 전환 실험

2023년 11월경 별도 `app_develop_tflite` 브랜치에서 TFLite 방식으로 실제 전환을 시도했습니다.

Git에서 확인되는 변경은 다음과 같습니다.

- `com.chaquo.python` 플러그인과 Python runtime 설정 제거
- TensorFlow Lite, TensorFlow Lite GPU, Select TF Ops 의존성 추가
- `nemo_best.tflite` 모델을 Android Java 코드에서 직접 로드
- 입력 이미지를 416×416 RGB float buffer로 변환
- `Interpreter.runForMultipleInputsOutputs()`로 추론
- Java에서 confidence filtering 및 class별 NMS 구현
- Java에서 bounding box 위치를 x/y 방향으로 정렬하고 그룹화하는 후처리 구현
- NNAPI → GPU delegate → 4-thread CPU 순으로 fallback하는 실행 구조 시험

즉 TFLite 모델만 호출한 것이 아니라, 기존 Python 경로에 있던 추론과 후처리를 Android/Java 쪽으로 상당 부분 옮기는 실험이었습니다.

### 당시 측정 결과

2026년 SK하이닉스 지원을 위해 정리해 둔 당시 측정 기록에는 다음 수치가 남아 있습니다.

- 측정 기기: Galaxy S9+
- 기존 방식 평균 처리 시간: 9.1초
- TFLite 방식 평균 처리 시간: 5.4초
- 처리 시간: 약 41% 단축
- 기존 방식 인식 정확도: 90%
- TFLite 방식 인식 정확도: 57%

Git에는 기존 Python 경로에서 `System.currentTimeMillis()`로 실행 시간을 측정하던 코드 흔적은 남아 있지만, 위 9.1초와 5.4초의 실제 benchmark output 자체는 저장되어 있지 않습니다. 따라서 위 수치는 **당시 측정 결과를 나중에 경험기술서에 기록한 값**으로 구분해 사용합니다.

### 최종 선택

허밍블럭스는 인식 결과가 바로 음악 실행으로 이어지는 교육 제품이기 때문에, 응답 속도보다 사용자가 조립한 블록을 정확하게 판별하는 것이 더 중요하다고 판단했습니다. 속도는 약 41% 개선됐지만 정확도가 크게 떨어져 TFLite 방식은 mainline에 채택하지 않았고 기존 Chaquopy 기반 구조를 유지했습니다.

실제로 TFLite 구현은 별도 실험 브랜치에 남아 있고, 이후 `app_develop`의 2.2.0 코드까지 Chaquopy/Python/OpenCV 경로가 유지되는 것이 Git으로 확인됩니다.

이 경험은 최신 기술이나 속도 하나만 기준으로 선택하지 않고 제품 목적에 따라 정확도와 지연을 함께 비교해 결정한 사례로 활용할 수 있습니다.

## Git에서 확인되는 주요 개발 흐름

### 2023년 6~7월: 카메라 및 음악 실행 흐름 구축

- 카메라 촬영과 앱 실행 흐름 개선
- 음악별 캐릭터와 사용자 입력 표시 연결
- 여러 악기 트랙의 연속 재생을 위해 `ScheduledThreadPoolExecutor` 기반 재생 스케줄러 구현
- 두 MediaPlayer 그룹을 교대로 prepare/start하고 pause/resume 시 phase와 delay에 맞춰 스케줄을 복원

### 2023년 9~10월: BPM 및 접근성 기능 확장

- section별 BPM 상태를 음악 데이터에 포함
- BPM에 따라 각 section의 실제 길이와 전체 진행률을 동적으로 계산
- TalkBack 지원, 결과 확인 화면, 진동 및 설정 기능 추가

### 2023년 11월: TFLite 전환 실험

- `app_develop_tflite` 브랜치에서 Chaquopy/Python 경로를 TFLite + Java 구조로 바꾸는 실험 수행
- Java NMS 및 위치 기반 후처리까지 구현
- 속도 개선과 정확도 저하를 비교한 뒤 mainline에는 미채택

### 2024년 5월: FFmpeg 기반 MP3 저장

- `mobile-ffmpeg`를 연동
- 한 section의 여러 악기 트랙을 `amix`로 병합
- section별 결과를 `concat`하여 하나의 MP3로 내보내는 export pipeline 구현

### 2024년 7월: 진행도 및 사용자 데이터 구조 리팩터링

- `progress.json` 중심으로 튜토리얼, 퀘스트, 악기 해금 상태를 재설계
- 음악 JSON의 level, replace, genre 정보를 구조화
- 설정 및 저장 음악 흐름을 함께 정리

### 2024년 10월: 시각장애인용 CV 처리 흐름 개선

- Python/OpenCV DNN detection 호출 구조 개선
- 검출 box의 정렬 및 그룹화 후처리 수정
- 시각장애인용 앱의 촬영/결과 흐름과 연동

### 2024년 12월: 버전 마이그레이션

- 2.0.0 업데이트에서 `version.json`과 `VersionUtil` 도입
- 기존 저장 음악 JSON과 block 데이터를 신규 `MusicFile` 구조로 변환
- 이전 MP3와 weight 관련 파일을 정리하고 필요한 MP3를 다시 생성
- migration 과정에서 발견된 파일명, 반복문, nested block 처리 오류를 후속 커밋으로 수정

### 2024년 12월 ~ 2025년 1월: 녹화와 촬영 접근성 고도화

- CameraX VideoCapture 기반 구간별 영상 녹화
- FFmpeg로 임시 영상들을 concat하고 사용자 음악을 합쳐 최종 MP4 생성
- CameraX ImageAnalysis + ML Kit Barcode Scanner로 촬영 프레임을 실시간 분석
- 블록 위치에 따라 좌/우/상/하 이동 방향을 화면 애니메이션과 TalkBack으로 안내
- 저장 음악 재생 로직과 UI, 튜토리얼을 후속 개선

## 실제 사용 환경에서 발견한 문제

학습 데이터는 주로 실내에서 촬영했지만 야외 일몰 환경에서는 태양빛 때문에 블록 색이 달라져 인식하지 못하는 문제가 발생했습니다.

문제 상황과 촬영 조건을 AI 엔지니어에게 공유했고 해당 환경을 반영한 변형 데이터를 추가해 모델을 보완했습니다.

> 정확한 데이터 증강 방식은 추후 원본 기록을 확인하기 전까지 확정해서 쓰지 않습니다.

## UI 및 사용 흐름 개선

- 메인 화면에서 다른 화면을 거쳐 음악을 선택하던 흐름을 줄여 메인 화면에서 음악 선택 화면을 바로 열도록 변경
- 기기별 화면 비율 차이로 버튼이 작게 표시되는 문제 발견
- UI 기준 비율을 고정하고 실제 화면 크기에 맞춰 전체 요소를 조정하도록 개선
- 튜토리얼, 퀘스트, 악기 해금, 저장 음악 화면을 추가하고 반복적으로 사용 흐름을 수정

## 외부 리소스 적용

- 전달된 음원 파일의 누락 여부 확인 및 추가 요청
- 비어 있는 음원 파일 직접 보완
- 음원 길이를 확인하고 실행에 필요한 BPM 값 추가 요청
- 통일되지 않은 파일명을 앱 구조에 맞게 재정리
- 디자인 이미지를 필요한 크기와 위치에 맞게 가공
- 리소스 전달이 늦어졌을 때 GIF 프레임을 PNG로 가공해 개발 일정이 멈추지 않도록 대응

## 결과

- 전국 장애·비장애 대학생 창업경진대회 대상
- Google Play 출시
- 무료 연동 앱과 유료 블록 키트 형태로 운영 및 판매
- 출시 이후에도 2.x 버전까지 앱 기능과 사용자 데이터 호환성을 지속 개선한 기록 확인

## 보여주는 역량

- Android 제품 개발 및 장기 유지보수
- Computer Vision 결과의 서비스 후처리
- 정확도와 속도 사이의 기술적 의사결정
- Java/Python 및 TFLite runtime 비교 실험
- Android 멀티미디어 및 FFmpeg 파이프라인
- 데이터 구조 리팩터링과 기존 사용자 migration
- 접근성 중심의 CameraX/ML Kit 기능 구현
- 실제 환경 문제 대응
- 예외 처리와 사용자 경험 개선
- 외부 협업 결과물을 제품에 통합하는 능력

## 자소서 활용 포인트

- 가장 오래 몰입한 프로젝트
- 가장 어려웠던 문제
- 성능 개선 시도와 기술 선택의 기준
- 사용자 관점의 개선
- 새로운 기술을 배운 경험
- 출시 이후 유지보수 및 migration 경험
- 제품 출시 경험
- 협업 중 빈틈을 메운 경험

## Git 근거 메모

대표적으로 확인한 `nanocode00/nemo_codeblock` 커밋 및 브랜치:

- `app_develop_tflite`: TFLite 전환 실험 브랜치
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

- GitHub가 직접 증명하는 것: 구현 구조, 브랜치 분리, 작성자, 대략적인 개발 시점, mainline에 최종 남은 기술 구조
- 당시 자소서/경험기술서가 증명하는 것: Galaxy S9+ 측정 조건, 9.1초 → 5.4초, 정확도 90% → 57%, 정확도를 우선해 TFLite를 채택하지 않았다는 당시 판단
- 아직 추가 원본 확인이 필요한 것: 정확도 측정 데이터셋과 산출 방식, 야외 환경 보완 시 사용한 정확한 augmentation 기법
