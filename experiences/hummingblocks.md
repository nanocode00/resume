# HummingBlocks

> 이 문서는 허밍블럭스 프로젝트의 개발 경험을 정리한 기준 문서다. 2026-09-19 기준 `nanocode00/nemo_codeblock`의 전체 8개 브랜치 commit graph를 교차검증해 복원한 `HB-01~HB-45` 티켓을 근거로 작성했다. 포트폴리오·이력서 문장은 이 문서에서 필요한 경험만 다시 선별한다.

## 1. 프로젝트 개요

| 항목 | 내용 |
|---|---|
| 프로젝트 | HummingBlocks(허밍블럭스) |
| 제품 | 실물 코딩 블록을 촬영해 블록 배치를 프로그램으로 해석하고 음악으로 실행하는 Android 교육 제품 |
| 사용자 | 시각장애 학습자를 출발점으로 일반·특수교육 환경까지 확장 |
| 역할 | 네모감성 공동 창업자 · Mobile App Developer |
| 개발 흐름 | Android 초기 구현 → Python/CV 제품 통합 → 음악 실행·접근성 → Google Play 출시 → 저장·퀘스트·영상·migration 확장 |
| 주요 기술 | Android, Java, Python, Chaquopy, OpenCV DNN, YOLO/Darknet, TensorFlow Lite, CameraX, ML Kit Barcode, FFmpeg, ConstraintLayout |
| 저장소 | https://github.com/nanocode00/nemo_codeblock |
| 출시 기록 | Google Play 2023-10-31 |

허밍블럭스는 화면 안에서 블록을 조작하는 앱이 아니라, 사용자가 실제 블록을 배치한 뒤 카메라로 촬영하면 앱이 배치를 읽고 음악 실행 규칙으로 변환하는 제품이다. PLAY·LOOP·STAR, 좌/우 조건, 악기 단계 증가·감소·초기화 등의 블록 규칙은 팀 회의에서 주로 의견을 내며 설계했고, 초기 Python 인터프리터 구현·테스트는 다른 팀원이 담당했다. 이후 Android 제품의 화면·카메라·센서·Python 연동·재생·접근성·저장·배포 흐름을 구현하고 유지보수했다.

## 2. 기여 범위와 경계

### 직접 구현·제품화한 범위

- 초기 Android Activity 흐름, 카메라 preview/촬영, 이미지 파일화와 방향 보정, 중력센서 입력을 구현했다.
- 기존 Python CV·블록 실행 자산을 Android 안에서 재사용하기 위해 Chaquopy를 연결하고 Camera → Python → Runner → Play의 제품 흐름을 통합했다.
- 초기 통합 후 드러난 실행기 오류를 정리해 Java `Runner`의 `SUCCESS / ERROR / WARNING` 결과 계약과 사용자 오류 안내 구조를 보강했다.
- 여러 악기의 동시 재생, 마디 전환, pause/resume, BPM, 진행도와 countdown을 Android 재생 스케줄러에 통합했다.
- TalkBack 탐색 순서와 상태 announcement, 진동, 인식 결과 확인, 시각장애 사용자용 자동촬영·방향 안내 흐름을 구현했다.
- Figma 시안을 `ConstraintLayout` percentage Guideline과 `0dp` constraint로 변환해 화면 비율에 따라 전체 UI가 함께 늘고 줄도록 구현했다.
- 교체 악기·퀘스트·진행도·설정 상태를 저장 모델과 연결하고, 음악 콘텐츠를 metadata/asset 규격으로 확장했다.
- 저장 음악을 `MusicFile` 중심 구조로 재설계하고 FFmpeg 기반 MP3 생성, CameraX 분할 녹화, 영상 concat·audio mux, 구버전 데이터 migration까지 연결했다.
- Python bridge 제거를 목표로 Java/TFLite 추론 경로를 별도 브랜치에서 구현·검증한 뒤 정확도 문제로 제품 적용을 철회했다.
- 대용량 음악/YOLO 자산의 Play Asset Delivery 분리, Git LFS 관리, AAB → universal APK 추출 자동화를 구성했다.

### 팀 또는 외부 기여로 구분하는 범위

- 2022~2023 초기 Python/Tkinter·PyQt·pygame 프로토타입과 최초 Python 인터프리터 코드는 다른 팀원이 작성했다.
- 초기 OpenCV DNN 추론, NMS, bounding box 정렬 코드도 다른 팀원이 구현했다. Android 통합과 후속 실행 계약·제품 흐름을 담당했으며, Java/TFLite 실험에서는 추론·NMS·좌표 후처리를 직접 Java로 옮겼다.
- YOLO 모델 학습 전체와 모델 품질 개선 자체를 개인 구현으로 주장하지 않는다. 앱 연동·실사용 문제 전달·데이터 라벨링 등 확인 가능한 범위만 구분한다.
- 캐릭터·배경·무대 원본 디자인은 디자이너, 음원 원본은 외부 제작이며, 개인 기여는 Android asset 규격화와 metadata/path 연결, 화면·재생 통합이다.
- 영상 녹화의 최초 CameraX prototype은 팀원이 작성했다. 저장 음악·분할 녹화·상태 관리·FFmpeg 후처리까지 제품화한 부분을 개인 구현으로 구분한다.
- Retrofit/OkHttp 기반 그래피툰 QR/block JSON 공유 네트워크 기능은 `EunbinSeo` 구현(`2e30063`)이다. 접근성 앱의 바코드 촬영 흐름과는 별개이며 개인 HB 티켓으로 계상하지 않는다.

## 3. 제품 파이프라인

제품의 mainline은 최종적으로 다음 흐름을 사용했다.

1. Android 카메라가 실물 블록 배치를 촬영하고 기기 방향에 맞춰 입력 이미지를 보정한다.
2. 촬영 이미지를 앱 cache에 저장하고 Chaquopy를 통해 Python/OpenCV DNN 경로로 전달한다.
3. Python 측 CV·블록 처리 결과를 Android가 받아 `Runner`의 실행/오류/경고 계약으로 변환한다.
4. `PlayActivity`와 `MusicPlayer`가 마디별 악기 level·조건·교체 상태를 실제 음원 재생으로 변환한다.
5. TalkBack·진동·화면 상태·인식 결과 화면이 실행 과정과 오류를 사용자에게 설명한다.
6. 완성한 음악은 `MusicFile` JSON과 MP3로 저장하고, 필요하면 CameraX로 촬영한 영상과 FFmpeg로 결합한다.
7. 버전 구조가 바뀐 뒤에도 기존 저장 음악은 migration을 거쳐 새 `MusicFile` 포맷으로 이전한다.

## 4. 개발 경험

### 4.1 Android/CV 제품화 — 프로토타입을 실제 모바일 흐름으로 연결

초기 Android 앱은 `Loading → Main → 장르 선택 → 촬영 → 재생` 흐름으로 구성했다. 카메라와 중력센서는 각각 별도 테스트 모듈에서 먼저 검증한 뒤 본 앱에 합쳤다. 촬영 bitmap은 당시 사용 자세에 맞춰 회전하고 cache JPEG로 저장해 파일 경로를 다음 단계로 넘겼으며, 이후 기기 방향이 달라지면 고정 90° 보정 때문에 CV 입력이 틀어지는 문제를 확인해 중력센서와 이후 `OrientationEventListener` 기반으로 0/90/180/270° 방향을 처리했다. (`HB-01~HB-08`)

기존 Python 자산을 모두 Java로 다시 쓰거나 외부 서버에 보내는 대신, Android 내부에서 Python을 실행하는 Chaquopy를 선택했다. Java → Python 함수 호출 PoC를 거쳐 촬영 이미지 → Python CV/블록 실행 → Android 결과 처리 → `PlayActivity` 음악 재생까지 E2E 흐름을 연결했다. 초기 Python 실행 코드의 Android 이식은 팀원이 먼저 담당했고, 실제 앱 통합 이후 드러난 버그와 예외 처리 부족을 정리하면서 `Runner.java + block_runner.py` 구조와 `SUCCESS / ERROR / WARNING` 결과 계약을 보강했다. 최종 코드 기준으로 오류 23종과 경고 2종을 구분해 사용자에게 설명하는 구조로 발전했다. (`HB-07`, `HB-11`, `HB-14`)

### 4.2 음악 실행 엔진 — 여러 악기의 시간축을 Android에서 안정적으로 제어

한 마디에서 여러 악기를 동시에 재생하면서 다음 마디로 끊김 없이 넘어가야 했고, 초기에 `MediaPlayer` 생성 지연 때문에 마디 전환 시 체감되는 gap이 생겼다. 다음 마디용 MediaPlayer를 미리 준비해 두 묶음을 교차 사용하는 구조로 바꾸고, pause/resume에서는 현재 재생 위치뿐 아니라 예약된 다음 작업까지 함께 복원하도록 했다. 이후 오래된 `Timer/TimerTask`를 `ScheduledThreadPoolExecutor`와 `ScheduledFuture`로 교체하고 `Prepare → Start` phase를 명시적으로 분리했다. (`HB-09`, `HB-10`, `HB-16`)

장르 확장에서는 `장르 × 악기 × 레벨` 음원을 매핑했고, 3단계 BPM 기능에서는 playback speed를 억지로 변경하는 대신 BPM별 제작 음원과 `60000 / BPM × beat 수`로 계산한 마디 길이를 scheduler에 함께 적용했다. BPM 기능은 실제 구현됐지만 이후 제품에서 제거됐으며 제거 이유는 현재 근거만으로 확정하지 않는다. 재생 상태와 Lottie 애니메이션을 pause/resume에 맞춰 동기화하고, 첫 음악 시작 전 2.5초 준비 구간을 3·2·1 countdown으로 표시하면서 pause 시 남은 countdown도 복원했다. (`HB-13`, `HB-18`, `HB-19`, `HB-28`, `HB-29`)

### 4.3 접근성 — TalkBack 보강에서 자동 촬영 폐루프까지

TalkBack 사용 시 화면의 시각적 배치 순서와 실제 탐색 순서가 맞지 않거나, 처리 중인 버튼까지 계속 focus되는 문제가 있었다. `contentDescription`, `importantForAccessibility`, `accessibilityTraversalBefore/After`, `TYPE_ANNOUNCEMENT`를 사용해 사용자가 실제로 조작 가능한 요소만 탐색하도록 하고, 촬영 처리 중에는 `로딩중입니다` 같은 동적 상태를 직접 음성으로 알렸다. 인식된 블록은 별도 결과 화면에서 사람이 읽는 label로 보여주고, 이후 PLAY/LOOP/STAR와 조건 scope·괄호가 보이는 코드형 구조와 TalkBack 설명으로 확장했다. 진동 피드백과 사용자 설정도 같은 접근성 흐름에 포함했다. (`HB-17`, `HB-20~HB-22`)

2024년에는 시각장애 사용자용 앱을 별도 모듈로 유지하면서 촬영 UX를 다시 설계했다. QR 1~4의 검출 상태를 bitmask로 만들고 특정 QR 3개 이상이 보이면 자동 촬영하도록 했으며, 아직 촬영 조건이 아니면 현재 보이는 QR 조합을 좌/우/상/하 이동 방향으로 변환했다. 같은 상태를 화살표·문구와 TalkBack announcement에 동시에 반영하고, 반복 안내를 throttle했다. 자동 촬영 후 CV가 오류/경고를 반환하면 결과 다이얼로그를 보여준 뒤 다시 camera preview로 복구해 `QR 분석 → 방향 안내 → 자동 촬영 → 판정 → 재촬영`이 한 화면에서 닫힌 루프로 동작하게 했다. (`HB-25`, `HB-27`, `HB-37`)

### 4.4 반응형 UI — 기기별 분기가 아니라 상대 좌표 체계로 전환

초기에는 특정 화면에서 고정 dp와 margin 중심 배치가 깨지는 문제가 있어 Play 화면부터 9:16 기준 콘텐츠 영역과 percentage Guideline을 사용했다. 이후 이 방식을 앱 전반으로 넓혔고, 2024년 대규모 UI 개편에서는 디자이너가 Figma에 만든 시안을 기준으로 각 위치와 크기를 360×740 좌표계의 비율로 다시 계산했다. `ConstraintLayout` 안에 percentage `Guideline`을 두고 View를 그 사이 `0dp` constraint로 묶어 화면 비율에 따라 컴포넌트 전체가 함께 확대·축소되도록 구현했다. (`HB-12`, `HB-34`)

텍스트는 `autoSizeTextType`을 적용하고, 기본 auto-size 적용이 어려운 EditText는 별도 유틸리티로 계산된 textSize를 반영했다. 후속 작업에서는 버튼마다 흩어진 Guideline을 행·열 단위 공통 기준선으로 묶어 버튼과 label이 동일한 상대 좌표계를 공유하게 했고, 튜토리얼 overlay도 실제 화면과 같은 Guideline 구조로 함께 수정했다. 특정 tablet/Z Flip 전용 layout을 만드는 방식이 아니라 같은 상대 배치 체계를 유지한 것이 핵심이다. (`HB-35`, `HB-42`)

### 4.5 진행도·교체·콘텐츠 — 기능 상태를 데이터 모델로 연결

악기 교체 기능은 단순 토글이 아니라 현재 선택 상태를 다음 마디의 MediaPlayer 준비 시점에 적용하도록 만들었다. `replace_data`로 기존 5개 악기 중 교체 대상을 지정하고, `next_replace`를 다음 마디 준비 단계에 전달했다. 초기 구현에서 교체 음원 인덱스와 원래 악기 level 인덱스가 섞이는 문제를 분리해 수정하고, 동일 원본 악기를 여러 교체 버튼이 가리키는 경우의 상호 배제도 처리했다. 이후 UI 선택 → 다음 마디 음원 → 캐릭터/Lottie → 저장 JSON → SavedMusic 재생 → FFmpeg MP3까지 같은 교체 상태가 이어지도록 통합했다. (`HB-30`)

퀘스트 진행도는 음악별 3개 boolean 배열에서 `tutorial_done + 음악별 정수 quest_progress` 구조로 바꿔 순차 해금 상태를 직접 표현했다. 음악별 JSON에 정의된 block 조건을 실제 인식 결과와 비교해 퀘스트를 완료하고, 별 표시·교체 기능 해금·unlock animation으로 연결했다. 설정 역시 악기별 volume·진동·교체 버튼 표시를 `Setting` 모델 하나로 묶고 reset·영속화 UI를 연결했다. (`HB-31`, `HB-36`, `HB-39`)

콘텐츠 확장에서는 음악마다 Java `switch`를 추가하지 않고 `music_info.json`, `values.json`, label/genre metadata와 공통 asset 경로 규격을 사용했다. Cyberpunk·Classic 같은 신규 음악은 Java 코드 변경 없이 metadata 등록과 규격화된 MP3·이미지·Lottie 추가만으로 기존 `MusicInfo`/Select/Play/MusicPlayer 흐름을 재사용했다. 원본 asset 제작이 아니라 이런 데이터 계약에 맞춰 Android에 통합하고 유지보수한 경험으로 구분한다. (`HB-40`)

### 4.6 저장·미디어 파이프라인 — 실행 상태를 재사용 가능한 결과물로 변환

처음에는 사용자가 만든 음악의 마디별 `[악기 level + BPM]` 상태를 JSON으로 저장했고, 이를 이용해 원본 음원을 다시 조합하는 방식으로 MP3를 생성했다. 각 마디의 여러 악기는 FFmpeg `amix`, 마디 간 연결은 `concat`을 사용했으며, 무음 마디에는 `empty.mp3`를 넣어 시간 길이를 유지했다. 즉 화면 출력을 녹음한 것이 아니라 구조화된 실행 상태에서 결과 오디오를 재생성했다. (`HB-26`)

이후 저장 데이터를 `MusicFile` 단일 도메인 모델로 재설계했다. 과거의 `장르별 JSON + 별도 _block.json + 파생 MP3` 구조를 `files/music/<title>.json` 하나에 title·genre·creation date·levels·replaces·blocks를 모두 담는 self-contained 구조로 바꾸고, 전체/장르별 목록·재생·삭제·제목 수정·MP3 다운로드/공유를 이 모델 중심으로 통합했다. 외부 그래피툰 연동에 필요한 JSON도 `MusicFile.createJSON()`에서 만들도록 데이터 책임을 옮겼지만 네트워크/QR 공유 구현 자체는 팀 기능이다. (`HB-38`)

2.0.0 전환에서는 `version.json`이 없는 기존 설치를 감지해 과거 장르별 저장 음악과 `_block.json`을 새 `MusicFile`로 한 번 변환하고, 구형 weight cache와 파생 MP3를 정리한 뒤 최신 asset과 MP3를 다시 생성했다. 범용 버전별 migration framework가 아니라 pre-v25 설치를 새 구조로 옮기는 일회성 호환 레이어였고, 실제 변환 과정에서 file name·block nesting·loop·MP3 path 문제를 연속으로 수정했다. (`HB-41`)

영상 기능에서는 팀원의 단일 CameraX 녹화 prototype을 저장 음악 중심 제품 흐름으로 다시 만들었다. `MusicFile`에서 선택한 MP3를 타임라인으로 사용하고 `temp1.mp4`, `temp2.mp4`처럼 여러 segment를 촬영하면서 음악도 같은 시점에 pause/resume했다. 최종화할 때 FFmpeg concat demuxer로 영상 stream을 무재인코딩 결합하고, 저장 음악 MP3를 AAC audio로 mux한 뒤 `MediaStore/DCIM/HummingBlocks`에 저장했다. 작업용 temp/concat 파일은 완료 후 정리했다. (`HB-43`, `HB-44`)

마지막에는 저장 음악 재생 자체도 `MusicFile`을 다시 실행해 합성하는 방식에서 이미 생성된 MP3를 Android `MediaPlayer`로 직접 재생하는 방식으로 단순화했다. 재생·공유·영상 제작이 동일한 MP3 결과물을 source of truth로 사용하게 하고, SavedMusic 화면에서 바로 현재 음악으로 영상 제작에 진입하도록 UX를 연결했다. (`HB-45`)

### 4.7 추론·배포 실험 — 성능을 높이되 제품 기준으로 되돌릴 수 있게 검증

제품 mainline은 `Android Java → Chaquopy → Python/OpenCV DNN → Darknet weight/cfg`였다. Python bridge 지연을 줄이기 위해 별도 `app_develop_tflite` 브랜치에서 Chaquopy와 Python 의존성을 제거하고 Java/TensorFlow Lite 경로를 직접 구현했다. 416×416 입력 tensor, confidence filtering, class별 NMS, IoU, bounding box 정렬·그룹화를 Java로 옮기고 NNAPI → GPU delegate → CPU thread fallback도 구성했다. 이후 aspect ratio를 유지하는 scale + padding 전처리까지 보정했다. (`HB-23`, `HB-24`)

실기기 비교 당시 기록은 Galaxy S9+ 여러 대에서 기존 경로와 TFLite 경로를 기기별 1회 실행해 약 `9.1초 → 5.4초`로 지연이 줄었다. 반면 당시 남은 정확도 기록은 약 `90% → 57%`였고, 정확도 측정 데이터셋·샘플 수·집계 방식은 현재 확인되지 않는다. 속도 이점만으로 제품 경로를 바꾸지 않고 TFLite 변경을 원복해 Chaquopy/OpenCV mainline을 유지했다. 이 수치는 정밀 benchmark가 아니라 당시 기록값으로만 사용한다.

대용량 자산도 별도 문제였다. 2023년에는 음악과 YOLO 자산을 install-time Play Asset Delivery의 `:musics`/`:weights` asset pack으로 분리하고, Python/OpenCV가 실제 파일 경로를 필요로 하는 weight/cfg는 내부 `files/weights`로 materialize해 넘겼다. 2024년에는 약 257MB YOLO weight와 bundletool을 Git LFS로 관리하고 AAB에서 universal APK를 추출하는 batch script를 추가했다. (`HB-33`, `HB-32`)

## 5. 시간순 요약

| 시기 | 주요 변화 | 티켓 |
|---|---|---|
| 2023.03 이전 | 초기 Android 화면·카메라·센서·장르 선택 구조와 테스트 모듈 | HB-01~HB-06 |
| 2023.03~06 | Chaquopy 연동, 촬영 방향 보정, Camera→Python→Play 통합, Runner 오류 계약 | HB-07~HB-14 |
| 2023.06~10 | 음악 테마, scheduler 재설계, BPM, 결과 시각화, TalkBack·진동 | HB-15~HB-22 |
| 2023.10~11 | PAD asset pack, Java/TFLite 전환 실험과 원복 | HB-23~HB-24, HB-33 |
| 2024.04~07 | 접근성 앱 분리, MP3 생성, 바코드 흐름 수정, Lottie/countdown, 교체·진행도 | HB-25~HB-31 |
| 2024.08~10 | Git LFS/배포, Figma 기반 responsive UI, 튜토리얼, 퀘스트, 접근성 자동촬영 | HB-32, HB-34~HB-37 |
| 2024.10~12 | MusicFile·Setting·콘텐츠 규격, 2.0 migration, Play UI 재정렬 | HB-38~HB-42 |
| 2024.11~2025.01 | CameraX 분할 녹화, FFmpeg 영상 합성, 저장 음악/영상 UX 최종 통합 | HB-43~HB-45 |

## 6. 결과와 검증 가능한 수치

- Android에서 Camera → Python/OpenCV → 실행 상태 → 음악 재생으로 이어지는 제품 흐름을 구현하고 Google Play 출시 이후 기능 확장과 데이터 호환 유지보수까지 이어갔다.
- 최종 코드의 실행 검증 체계는 오류 23종과 경고 2종을 구분해 사용자에게 안내한다.
- TFLite 전환 실험의 당시 실기기 기록은 처리 시간 약 9.1초 → 5.4초였다. 여러 Galaxy S9+에서 각 버전을 기기별 1회 비교한 기록이므로 반복 평균 benchmark로 표현하지 않는다.
- TFLite 정확도 약 90% → 57% 기록은 평가 데이터셋·샘플 수·집계 방식이 확인되지 않아 조건 없는 정량 성과로 사용하지 않는다.
- 제품은 2023-10-31 Google Play 출시 기록이 있고, 프로젝트 팀은 2022년 창업·소셜벤처 경진대회 수상 기록을 보유한다.
- 이후 회사 자료의 다운로드·판매·학교 보급·CES 성과는 제품의 후속 누적 성과이며 개인 개발 성과와 분리해 사용한다.

## 7. 표현 시 주의할 점

- 초기 Python 인터프리터와 최초 OpenCV DNN/NMS/정렬 코드를 직접 구현했다고 쓰지 않는다.
- Android mainline runtime을 `PyTorch native` 또는 `TFLite`라고 설명하지 않는다. 최종 제품 경로는 Chaquopy + Python/OpenCV DNN + Darknet weight/cfg이며 TFLite는 미채택 실험이다.
- Figma/UI 원안을 직접 디자인했다고 쓰지 않는다. 디자인을 Android percentage Guideline 기반 responsive layout으로 변환·구현했다고 표현한다.
- 음원·캐릭터·배경 원본 제작을 개인 작업으로 쓰지 않는다. asset 규격화, metadata 연결, Android 통합을 개인 기여로 쓴다.
- CameraX 녹화의 최초 prototype을 개인 구현으로 쓰지 않는다. 저장 음악 기반 분할 녹화와 상태 관리, FFmpeg 후처리 제품화를 개인 기여로 구분한다.
- 그래피툰 QR 네트워크 공유를 개인 구현으로 쓰지 않는다. `MusicFile`에서 외부 연동용 데이터 구조를 정리한 부분과 네트워크 기능을 구분한다.
- BPM 제거 이유와 TFLite 정확도 평가 조건은 확인되지 않았으므로 추정하지 않는다.

## 8. HB 티켓 인덱스

아래 45개 티켓은 상세 근거를 추적하기 위한 개발 단위다. 본문에서는 같은 문제를 여러 번 개선한 티켓을 하나의 개발 흐름으로 통합했다.

| 티켓 | 시기 | 작업 | 통합 영역 |
|---|---|---|---|
| HB-01 | 2023.03 이전 | Loading → Main → 장르 선택 → 촬영 → 재생으로 이어지는 초기 Android 화면 구조와 Activity 전환 설계 | Android/CV 제품화 |
| HB-02 | 2023.03 이전 | 카메라 권한 요청, `SurfaceView` preview, 촬영 callback을 연결한 카메라 프로토타입 구현 | Android/CV 제품화 |
| HB-03 | 2023.03 이전 | 촬영 bitmap을 90도 회전해 cache JPEG로 저장하고 재생 화면에 파일 경로로 전달 | Android/CV 제품화 |
| HB-04 | 2023.03 이전 | 중력센서 값을 각도로 변환해 좌·우·중립 상태를 판단하고 지휘자·화살표 UI에 반영 | Android/CV 제품화 |
| HB-05 | 2023.03 이전 | RecyclerView와 ViewBinding으로 음악 장르 선택 상태를 관리하고 메인 화면에 결과 반영 | Android/CV 제품화 |
| HB-06 | 2023.03 이전 | 카메라·중력센서 기능을 별도 테스트 모듈에서 검증한 뒤 본 앱에 통합 | Android/CV 제품화 |
| HB-07 | 2023.03 | Android에서 Python 코드를 실행하기 위한 Chaquopy 연동 및 빌드 구성 | Android/CV 제품화 |
| HB-08 | 2023.04~06 | 카메라 촬영 방향 보정 및 촬영 화면 흐름 개편 | Android/CV 제품화 |
| HB-09 | 2023.04 | 여러 악기 음원을 동시에 실행하기 위한 MediaPlayer 재생 구조 실험 | 음악 실행 엔진 |
| HB-10 | 2023.04 | 음악 실행 중 일시정지·재개·정지 상태 처리 | 음악 실행 엔진 |
| HB-11 | 2023.04~07 | CV 인식 결과를 `Runner`의 SUCCESS/ERROR/WARNING 계약과 블록 실행 상태로 변환해 Camera→Play 재생 흐름에 연결 | Android/CV 제품화 |
| HB-12 | 2023.04 | 화면비가 다른 기기에서 재생 화면이 깨지는 문제를 9:16 비율과 percentage Guideline으로 개선 | 반응형 UI·온보딩 |
| HB-13 | 2023.05 | 단일 음원 구성을 여러 음악 장르로 확장하고 장르별 음원을 매핑 | 음악 실행 엔진 |
| HB-14 | 2023.05~06 | 팀원이 이식한 Python 블록 실행 코드를 실제 Android 흐름에 맞춰 리팩터링하고 Java `Runner` 결과 계약·오류/경고 enum을 정리 | Android/CV 제품화 |
| HB-15 | 2023.06 | 음악 장르에 맞춰 캐릭터·무대·악기 UI를 변경하고 새로운 촬영 화면을 통합 | 음악 실행 엔진 |
| HB-16 | 2023.07~09 | 음악 재생을 별도 `MusicPlayer`로 캡슐화하고 `ScheduledThreadPoolExecutor`의 Prepare/Start phase와 pause/resume delay 복원 구조로 개편 | 음악 실행 엔진 |
| HB-17 | 2023.07 | 재생 진행도와 인식된 블록 입력을 화면에 시각화 | 접근성·결과 가시화 |
| HB-18 | 2023.09~10 | 3단계 BPM 음원 선택과 `beats × 60000/BPM` 기반 마디 길이 계산을 재생 scheduler에 통합 | 음악 실행 엔진 |
| HB-19 | 2023.10 | BPM 메타데이터가 없는 음악도 재생할 수 있도록 호환 처리 | 음악 실행 엔진 |
| HB-20 | 2023.10~2025.01 | TalkBack 설명·탐색 순서·처리중 focus 제어를 시작으로 화면 전반의 Button semantics와 접근성 상태 안내를 지속 보강 | 접근성·결과 가시화 |
| HB-21 | 2023.10~2024.04 | 인식 블록을 사람이 읽는 label로 보여주고 이후 PLAY/LOOP/STAR·조건 scope와 괄호를 갖는 코드형 구조 및 TalkBack용 설명까지 확장 | 접근성·결과 가시화 |
| HB-22 | 2023.10 | 진동 피드백·설정 화면·BPM 설정 등 사용자 제어 기능 추가 | 접근성·결과 가시화 |
| HB-23 | 2023.11 | Python bridge를 제거하고 Java/TensorFlow Lite 추론 경로와 후처리를 구현 | 추론·배포 실험 |
| HB-24 | 2023.11 | TFLite 전처리의 이미지 비율·padding을 보정하고 정확도 저하로 제품 경로를 원복 | 추론·배포 실험 |
| HB-25 | 2024.04 | 시각장애 사용자용 앱을 별도 Android 모듈로 분리 | 접근성·결과 가시화 |
| HB-26 | 2024.05 | 여러 악기 음원을 합성해 MP3로 추출하고 저장 음악 흐름에 연결 | 저장·미디어 파이프라인 |
| HB-27 | 2024.05~07 | QR/바코드 촬영 흐름의 권한·화면 전환 오류 수정과 선택 화면 UI 개선 | 접근성·결과 가시화 |
| HB-28 | 2024.07 | 재생 pause/resume 시 악기·지휘자 Lottie 애니메이션 상태를 음악 재생 상태와 동기화 | 음악 실행 엔진 |
| HB-29 | 2024.07 | 첫 음악 시작 전 2.5초 준비 구간을 3·2·1 countdown으로 시각화하고 pause/resume 시 남은 countdown을 복원 | 음악 실행 엔진 |
| HB-30 | 2024.07 | 다음 마디부터 기존 악기를 교체 음원으로 바꾸고 UI·캐릭터·저장 JSON·SavedMusic·MP3까지 상태를 일관되게 유지 | 진행도·교체·콘텐츠 |
| HB-31 | 2024.07 | 음악별 3개 boolean 진행도를 전역 `Progress`의 순차 단계 정수로 재설계하고 lock/unlock·퀘스트 안내 UI와 즉시 영속화를 연결 | 진행도·교체·콘텐츠 |
| HB-32 | 2024.08 | 약 257MB YOLO weight와 bundletool을 Git LFS로 관리하고 AAB→universal APK 추출을 batch script로 자동화 | 추론·배포 실험 |
| HB-33 | 2023.10 | 음악·YOLO 모델 자산을 install-time Play Asset Delivery `:musics`/`:weights` pack으로 분리하고 weight/cfg를 내부 파일로 materialize해 Python/OpenCV에 실제 경로 전달 | 추론·배포 실험 |
| HB-34 | 2024.08 | 2024.02 앱 전반의 percentage layout(`3889c17`)을 기반으로 디자이너 Figma 좌표·비율을 다시 계산해 Main/Select/Play/Loading·다이얼로그를 360×740 Guideline·auto-size UI로 전면 구현 | 반응형 UI·온보딩 |
| HB-35 | 2024.09~12 | 첫 사용자용 튜토리얼과 시작 안내 팝업 구현·개선 | 반응형 UI·온보딩 |
| HB-36 | 2024.08~09 | 음악별 JSON block 조건을 실제 인식 결과와 비교해 퀘스트를 순차 완료하고 별 표시·교체 기능 해금·unlock animation까지 연결 | 진행도·교체·콘텐츠 |
| HB-37 | 2024.05~10 | 오류/경고별 시각 피드백을 보강한 뒤 접근성 앱에서 QR 1~4 bitmask로 카메라 이동 방향을 추론해 화살표·TalkBack 안내·3개 이상 자동 촬영·촬영 복구까지 폐루프로 통합 | 접근성·결과 가시화 |
| HB-38 | 2024.10~11 | 저장 음악을 `MusicFile` 단일 JSON 모델로 통합하고 전체/장르별 목록·재생 상태·제목수정/삭제·QR 연동·MP3 다운로드/공유까지 재구성 | 저장·미디어 파이프라인 |
| HB-39 | 2024.10 | 악기별 volume·진동·교체버튼 표시 설정을 `Setting` 모델로 정리하고 설정/진행도 reset·영속화 UI를 통합 | 진행도·교체·콘텐츠 |
| HB-40 | 2024.05~11 | 공통 asset 경로와 metadata 계약을 이용해 Cyberpunk·Classic 등 신규 음악을 Java 수정 없이 추가하고 음원/이미지/Lottie 콘텐츠를 통합·유지보수 | 진행도·교체·콘텐츠 |
| HB-41 | 2024.12 | 2.0.0 전환에서 pre-v25 설치를 감지해 장르별 구형 저장 음악·block·MP3/cache를 `MusicFile` 구조로 1회 migration하고 version I/O를 `VersionUtil`로 분리 | 저장·미디어 파이프라인 |
| HB-42 | 2024.12 | 재생 화면 버튼·label의 개별 좌표 Guideline을 행·열 단위 공통 percentage Guideline으로 재구성해 화면 비율 변화에 따라 조작 UI 전체가 함께 확대·축소되도록 보정 | 반응형 UI·온보딩 |
| HB-43 | 2024.11~2025.01 | 팀원 CameraX 녹화 prototype을 저장 `MusicFile` 선택·음악 동기화·`tempN.mp4` 분할 녹화·오류/이탈 cleanup이 가능한 제품 흐름으로 재구성 | 저장·미디어 파이프라인 |
| HB-44 | 2024.12~2025.01 | `tempN.mp4` 분할 영상을 FFmpeg concat demuxer로 무재인코딩 결합하고 저장 음악 MP3를 AAC로 mux해 MediaStore/DCIM에 최종 뮤직비디오를 저장 | 저장·미디어 파이프라인 |
| HB-45 | 2025.01 | 저장 음악 재생을 생성 MP3 직접 `MediaPlayer` 재생으로 단순화하고 관리 화면의 영상 제작 진입·전용 음악 선택 흐름·접근성 semantics를 최종 통합 | 저장·미디어 파이프라인 |

## 9. 근거 문서

- `experiences/hummingblocks-development-task-inventory.md`: 전체 브랜치/커밋 교차검증과 45개 티켓 원장
- `experiences/hummingblocks-rework-checkpoint.md`: 티켓별 문제·구현·검증 상세 복원 기록
- `nanocode00/nemo_codeblock`: 실제 코드와 commit graph

이 문서를 HummingBlocks의 사실 기준 원본으로 두고, 다음 단계의 포트폴리오·이력서는 여기서 직무와 메시지에 맞는 경험만 압축해 사용한다.