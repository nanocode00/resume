# 허밍블럭스 개발작업 인벤토리

> 최종 교차검증: 2026-09-19  
> 기준 저장소: `nanocode00/nemo_codeblock`  
> 기준 브랜치: 저장소의 전체 8개 브랜치  
> 목적: 전체 8개 브랜치의 commit graph를 기준으로 실제 개발작업을 복원하고, 세부 작업과 이력서용 통합 작업군을 함께 관리

## 번호 체계

- 개인 개발작업은 예외 접미사 없이 `HB-01~HB-45` 연속 번호로 관리한다.
- 번호는 현재 인벤토리의 논리적 작업 순서를 유지하는 식별자이며, 번호 자체가 엄밀한 시간순을 뜻하지는 않는다.
- 팀 선행·공동 구현은 `PRE-*`, `TEAM-*`, 상위 통합 단위는 `GROUP-*`를 유지한다.

## 분류 기준

- **A — 직접 근거 강함:** `nanocode00` 작성 커밋과 구체적인 코드 변경이 확인됨
- **B — 묶음 확인 필요:** 개인 작성 커밋은 있으나 여러 기능·리소스가 섞여 있어 실제 문제와 기여 범위를 더 확인해야 함
- **C — 팀 기여 확인 필요:** 최초 구현 커밋이 다른 팀원에게 있거나 역할 분담 확인이 필요함
- **D — 설계 기여 확인:** 코드·테스트 작성자는 다른 팀원이지만 재훈님의 요구사항·규칙 설계 기여가 확인됨
- **실험:** 구현 흔적은 있으나 제품 브랜치에 최종 반영되지 않음

커밋 작성자가 같다는 사실만으로 모든 변경을 혼자 설계·구현했다고 단정하지 않는다. 각 작업을 상세화할 때 코드 diff와 사용자 기억을 함께 확인한다.

## 0. 브랜치별 과거 기록 재검토

| 브랜치 | 확인된 역할 | 인벤토리 반영 원칙 |
|---|---|---|
| `main` | 2022년 Python 프로토타입부터 2023년 초기 Android 코드로 이어지는 가장 오래된 계보 | 초기 개발 흐름 확인에 사용 |
| `pyqt` | 2023.03 DNN 결과 처리 수정이 남은 분기 | PyQt라는 브랜치명만 보고 개인 작업으로 단정하지 않음 |
| `music_2023_03_24` | `pygame`으로 5개 악기 음원을 마디별 동시 재생하고 pause·조건 입력을 처리한 데스크톱 실험 | Android 음악 실행 구조의 선행 팀 작업으로 분리 |
| `app_develop_madi_divide_send` | Python이 전체 결과를 한 번에 반환하지 않고 마디 수와 마디별 데이터를 나눠 전달하는 인터페이스 실험 | Android–Python 통합 직전의 팀 작업으로 분리 |
| `app_develop_extra_track` | Funk 음원·메타데이터를 조정한 별도 분기 | `app_develop`과 중복되지 않는 커밋만 반영 |
| `app_develop_bpm` | BPM 이후 기능을 개발한 장기 분기이며, 상당수가 다른 SHA로 `app_develop`에 다시 반영됨 | 중복 작업을 새 작업으로 세지 않고 문제의 연속성 확인에 사용 |
| `app_develop_tflite` | Java/TFLite 전환과 전처리 보정 실험 | 제품 미반영 실험으로 유지 |
| `app_develop` | 2023.03 이후 Android 제품의 주 개발 계보 | 제품 반영 여부의 기준 |

주의할 점:

- `main`의 2024.02 커밋 `7e09868`은 메시지가 QR scan 추가이지만 실제 변경은 `.idea` 메타데이터뿐이다. 개발작업 근거로 사용하지 않는다.
- 재훈님의 첫 GitHub Android 커밋은 `cede7ee`(2023-03-20)이지만, 완성된 초기 앱과 테스트 모듈을 한 번에 추가했다. 따라서 실제 작성 시점은 커밋일보다 앞설 수 있다.
- 2026-09-12 사용자 확인을 통해 `cede7ee`에 일괄 반영된 초기 Android 코드는 모두 재훈님이 직접 구현한 것으로 확정했다.
- 2022.10~2023.03의 Python 프로토타입 커밋은 다른 팀원 명의다. 프로젝트의 기술적 기원에는 포함하되, 사용자 확인 없이 개인 기여로 쓰지 않는다.

### 0.1 전체 commit graph 교차검증

- 2026-09-19 GitHub REST commit collection으로 저장소의 **전체 8개 브랜치**를 다시 열거했다.
- 브랜치 간 중복 SHA를 제거하면 **229개 고유 커밋**, 그중 `nanocode00` 작성 커밋은 **162개**였다.
- 제품 기준 브랜치 `app_develop`은 총 **199개 커밋**이며, 기존 checkpoint/inventory에 SHA가 직접 등장하지 않던 `nanocode00` non-merge 커밋 **54개**를 별도로 역검토했다.
- 또한 최종 `app_develop`에 포함되지 않은 다른 브랜치의 `nanocode00` 고유 커밋 **21개**를 확인해 TFLite 실험, BPM 분기, 대용량 파일/음악·캐릭터 병렬 작업이 기존 항목에 이미 반영됐는지 대조했다.
- 이 교차검증에서 새 독립 작업으로 추가할 가치가 확인된 것은 **2023.10 Play Asset Delivery 기반 `musics`/`weights` asset pack 분리(HB-33)**였다. 최종 재검토 결과, 그 외 개인 독립 작업 누락은 발견되지 않았다.
- 별도 ID로 늘리지 않고 기존 작업에 흡수해야 할 중요한 누락 단계도 확인했다. 대표적으로 `3889c17`의 2024.02 앱 전반 percentage Guideline 전환은 HB-12→HB-34→HB-42로 이어지는 반응형 UI 작업군의 중간 단계이며, `61ea93b`은 HB-16 MusicPlayer/scheduler 재설계의 핵심 근거다.
- 나머지 미참조 커밋은 대부분 동일 기능의 후속 bugfix, asset 교체, UI polish, revert/재적용, 또는 다른 브랜치에서 동일 작업을 다른 SHA로 진행한 기록이었다. 따라서 commit message 하나만 보고 새 개발작업으로 중복 계상하지 않는다.
- 이 검증은 저장소에 남은 commit graph와 diff를 기준으로 한다. 커밋되지 않은 로컬 실험이나 저장소 밖 작업까지 존재하지 않았음을 증명하는 것은 아니다.
- 최종 재검토에서 `2e30063`의 그래피툰 QR/block JSON 공유 기능은 `EunbinSeo` 구현으로 확인해 `TEAM-07`로 분리했다. 그래피툰 공유 QR과 HB-27의 접근성 앱 바코드 촬영 흐름은 서로 다른 기능이다.

### 0.2 Android 이전·초기 Python 팀 작업

| ID | 시기 | 선행 개발작업 | 브랜치·대표 근거 | 작성자 | 개인 기여 확인 포인트 |
|---|---|---|---|---|---|
| PRE-01 | 2022.10 | 객체 좌표를 열·행 순서로 정렬하고 Play·Loop·Class 블록 묶음을 분리하는 Python/Tkinter 에뮬레이터 | 공통 초기 계보 `623ff8b`, `2c005e8` | `EunbinSeo` | 기획·테스트 또는 알고리즘 논의 참여 여부 |
| PRE-02 | 2022.10 | `pygame`으로 악기 레벨·증감·재생·리셋·함수 블록을 실행하는 데스크톱 음악 프로토타입 | 공통 초기 계보 `623ff8b` | `EunbinSeo` | 음악 규칙이나 블록 명세 설계 참여 여부 |
| PRE-03 | 2022.11 | PyQt 기반 사진 선택·장르 선택·블록 실행 화면 구현 | `main` 계보 `db8d617` | `Kim-Dong-Ju` | UI 요구사항 또는 사용 흐름 설계 참여 여부 |
| PRE-04 | 2022.11 | 키보드 입력으로 좌·우 기울기와 else 조건을 모사하고 반복문 선택 UI 추가 | `main` 계보 `0911c7c`, `c23cf04` | `Kim-Dong-Ju` | 센서 입력으로 전환할 때 이어받은 설계 범위 |
| PRE-05 | 2023.03 | Play·Loop·Star, 악기 레벨과 연산 블록을 실행 정보로 변환하는 Python 인터프리터 확장 | `main` 계보 `83c5110`, `764d9f9` | `Kim-Dong-Ju` | 이후 Android 연동·재설계에서 직접 바꾼 범위 |
| PRE-06 | 2023.03 | OpenCV DNN 추론, NMS, x/y 좌표 정렬을 기존 블록 실행기에 결합 | `main`, `pyqt` 계보 `7c3b366`, `e48144a`, `42e4b0a` | `EunbinSeo` | Android용 입출력·후처리 이식 범위 |
| PRE-07 | 2023.03 | 5개 악기 음원을 마디별로 동시 재생하고 좌·우·else 조건에 따라 실행하는 데스크톱 음악 실험 | `music_2023_03_24`의 `361e103`, `fb1f727`, `d1443a3` | `Kim-Dong-Ju` | Android MediaPlayer 구조 설계 시 참고·재사용한 범위 |
| PRE-08 | 2023.04 | Python 실행 결과를 마디 수와 마디별 악기 데이터로 나눠 Android에 전달하는 인터페이스 실험 | `app_develop_madi_divide_send`의 `fa13aea`~`db368ae` | `Kim-Dong-Ju` | Java에서 마디별 결과를 소비하도록 구현한 범위 |

역할 확인 메모(2026-09-13):

- Play·Loop·Star, 좌·우·else 조건, 악기 레벨과 증감·재생·리셋 등 **블록 규칙은 팀 회의에서 재훈님이 주로 의견을 내며 설계했다.**
- 위 규칙의 초기 Python 코드 작성과 테스트는 다른 팀원이 담당했다.
- 따라서 개인 경험에는 `블록 실행 규칙을 설계했다`고 쓸 수 있지만, 초기 Python 인터프리터를 직접 구현·테스트했다고 표현하지 않는다.
- 이후 Android에서 해당 결과를 소비하고 화면·센서·음악 실행으로 연결한 부분은 별도의 개인 구현 작업으로 구분한다.

### 0.3 초기 Android 일괄 반영에서 발견한 개인 작업 후보

아래 코드는 모두 재훈님 작성 커밋 `cede7ee`에서 2023-03-20에 처음 저장소에 들어왔다. 2026-09-12 사용자 확인을 통해 모두 직접 구현으로 확정했다. 실제 세부 작성 순서는 아직 확인이 필요하다.

| ID | 추정 시기 | 개발작업 후보 | 코드 근거 | 구분 | 상세화 우선순위 |
|---|---|---|---|---|---|
| HB-01 | 2023.03 이전 | Loading → Main → 장르 선택 → 촬영 → 재생으로 이어지는 초기 Android 화면 구조와 Activity 전환 설계 | `LoadingActivity`, `MainActivity`, `SelectActivity`, `ScanActivity`, `PlayActivity` | A·사용자 확인 | 매우 높음 |
| HB-02 | 2023.03 이전 | 카메라 권한 요청, `SurfaceView` preview, 촬영 callback을 연결한 카메라 프로토타입 구현 | `camtest1`, `CameraSurfaceView`, `ScanActivity` | A·사용자 확인 | 높음 |
| HB-03 | 2023.03 이전 | 촬영 bitmap을 90도 회전해 cache JPEG로 저장하고 재생 화면에 파일 경로로 전달 | `ScanActivity` → `PlayActivity` | A·사용자 확인 | 높음 |
| HB-04 | 2023.03 이전 | 중력센서 값을 각도로 변환해 좌·우·중립 상태를 판단하고 지휘자·화살표 UI에 반영 | `acctest`, `PlayActivity` | A·사용자 확인 | 매우 높음 |
| HB-05 | 2023.03 이전 | RecyclerView와 ViewBinding으로 음악 장르 선택 상태를 관리하고 메인 화면에 결과 반영 | `SelectActivity`, `MainActivity`, `Common` | A·사용자 확인 | 중간 |
| HB-06 | 2023.03 이전 | 카메라·중력센서 기능을 별도 테스트 모듈에서 검증한 뒤 본 앱에 통합 | `camtest1`, `acctest`, 본 앱 모듈 | A·사용자 확인 | 높음 |

## 1. 개인 개발작업 후보

| ID | 시기 | 개발작업 후보 | 대표 근거 | 구분 | 상세화 우선순위 |
|---|---|---|---|---|---|
| HB-07 | 2023.03 | Android에서 Python 코드를 실행하기 위한 Chaquopy 연동 및 빌드 구성 | `17aa4a0`, `5adb514` | A | 높음 |
| HB-08 | 2023.04~06 | 카메라 촬영 방향 보정 및 촬영 화면 흐름 개편 | `8cc6dc9`, `7ba3775` | B | 중간 |
| HB-09 | 2023.04 | 여러 악기 음원을 동시에 실행하기 위한 MediaPlayer 재생 구조 실험 | `8cc6dc9`, `406589a`, `5cd4f9f` | A | 높음 |
| HB-10 | 2023.04 | 음악 실행 중 일시정지·재개·정지 상태 처리 | `9937f39`, `b170496` | A | 높음 |
| HB-11 | 2023.04~07 | CV 인식 결과를 `Runner`의 SUCCESS/ERROR/WARNING 계약과 블록 실행 상태로 변환해 Camera→Play 재생 흐름에 연결 | `ccfc11f`, `8c21546`, `7ba3775`, `7d823b7` | A(통합·후속 구현) / 초기 Python port는 팀원 | 매우 높음 |
| HB-12 | 2023.04 | 화면비가 다른 기기에서 재생 화면이 깨지는 문제를 9:16 비율과 percentage Guideline으로 개선 | `4fb4957` | A | 매우 높음 |
| HB-13 | 2023.05 | 단일 음원 구성을 여러 음악 장르로 확장하고 장르별 음원을 매핑 | `5efb0ac` | A | 중간 |
| HB-14 | 2023.05~06 | 팀원이 이식한 Python 블록 실행 코드를 실제 Android 흐름에 맞춰 리팩터링하고 Java `Runner` 결과 계약·오류/경고 enum을 정리 | `c7a7d3c`, `f517396`, `8c21546` | A(후속 리팩터링) / 초기 interpreter는 팀원 | 높음 |
| HB-15 | 2023.06 | 음악 장르에 맞춰 캐릭터·무대·악기 UI를 변경하고 새로운 촬영 화면을 통합 | `7ba3775` | B | 중간 |
| HB-16 | 2023.07~09 | 음악 재생을 별도 `MusicPlayer`로 캡슐화하고 `ScheduledThreadPoolExecutor`의 Prepare/Start phase와 pause/resume delay 복원 구조로 개편 | `7d823b7`, `61ea93b` | A | 매우 높음 |
| HB-17 | 2023.07 | 재생 진행도와 인식된 블록 입력을 화면에 시각화 | `7d823b7` | A | 높음 |
| HB-18 | 2023.09~10 | 3단계 BPM 음원 선택과 `beats × 60000/BPM` 기반 마디 길이 계산을 재생 scheduler에 통합 | `2cc6c58`, `5a8fcad`, `6950154`, `1bce3cf` | A | 매우 높음 |
| HB-19 | 2023.10 | BPM 메타데이터가 없는 음악도 재생할 수 있도록 호환 처리 | `44037d8` | A | 높음 |
| HB-20 | 2023.10~2025.01 | TalkBack 설명·탐색 순서·처리중 focus 제어를 시작으로 화면 전반의 Button semantics와 접근성 상태 안내를 지속 보강 | `44037d8`, `7a06422`, `1bce3cf`, `190a583`, `e52f941`, `197bb19` | A | 매우 높음 |
| HB-21 | 2023.10~2024.04 | 인식 블록을 사람이 읽는 label로 보여주고 이후 PLAY/LOOP/STAR·조건 scope와 괄호를 갖는 코드형 구조 및 TalkBack용 설명까지 확장 | `0ac520c`, `7a06422`, `3889c17`, `c2cca66`, `190a583` | A | 높음 |
| HB-22 | 2023.10 | 진동 피드백·설정 화면·BPM 설정 등 사용자 제어 기능 추가 | `28468a6` | B | 중간 |
| HB-23 | 2023.11 | Python bridge를 제거하고 Java/TensorFlow Lite 추론 경로와 후처리를 구현 | `7cdc9e1`, `fc771fd` | 실험/A | 매우 높음 |
| HB-24 | 2023.11 | TFLite 전처리의 이미지 비율·padding을 보정하고 정확도 저하로 제품 경로를 원복 | `fc771fd`, `1bb3f7c` | 실험/A | 매우 높음 |
| HB-25 | 2024.04 | 시각장애 사용자용 앱을 별도 Android 모듈로 분리 | `88d70d6`, `e0356b8` | A | 높음 |
| HB-26 | 2024.05 | 여러 악기 음원을 합성해 MP3로 추출하고 저장 음악 흐름에 연결 | `ce30682`, `273e11a` | A | 매우 높음 |
| HB-27 | 2024.05~07 | QR/바코드 촬영 흐름의 권한·화면 전환 오류 수정과 선택 화면 UI 개선 | `8353f20`, `48af586` | B | 중간 |
| HB-28 | 2024.07 | 재생 pause/resume 시 악기·지휘자 Lottie 애니메이션 상태를 음악 재생 상태와 동기화 | `c63b2bb` | A | 중간 |
| HB-29 | 2024.07 | 첫 음악 시작 전 2.5초 준비 구간을 3·2·1 countdown으로 시각화하고 pause/resume 시 남은 countdown을 복원 | `3ef066b`, `8ff5698` 일부 | A | 중간 |
| HB-30 | 2024.07 | 다음 마디부터 기존 악기를 교체 음원으로 바꾸고 UI·캐릭터·저장 JSON·SavedMusic·MP3까지 상태를 일관되게 유지 | `4d37e3b`, `c22b525`, `063713c`, `1969c30` 일부, `8ff5698` 일부 | A | 매우 높음 |
| HB-31 | 2024.07 | 음악별 3개 boolean 진행도를 전역 `Progress`의 순차 단계 정수로 재설계하고 lock/unlock·퀘스트 안내 UI와 즉시 영속화를 연결 | `e837550`, `28bee75` | A | 매우 높음 |
| HB-32 | 2024.08 | 약 257MB YOLO weight와 bundletool을 Git LFS로 관리하고 AAB→universal APK 추출을 batch script로 자동화 | `e452f2d` (`bef51f4`는 인접 release maintenance) | A | 높음 |
| HB-33 | 2023.10 | 음악·YOLO 모델 자산을 install-time Play Asset Delivery `:musics`/`:weights` pack으로 분리하고 weight/cfg를 내부 파일로 materialize해 Python/OpenCV에 실제 경로 전달 | `a7003ee` | A | 매우 높음 |
| HB-34 | 2024.08 | 2024.02 앱 전반의 percentage layout(`3889c17`)을 기반으로 디자이너 Figma 좌표·비율을 다시 계산해 Main/Select/Play/Loading·다이얼로그를 360×740 Guideline·auto-size UI로 전면 구현 | `3889c17`(선행 단계), `e5c69ad`, `65ee77b` | A(구현) / 디자인 원안은 디자이너 | 매우 높음 |
| HB-35 | 2024.09~12 | 첫 사용자용 튜토리얼과 시작 안내 팝업 구현·개선 | `9d9d330`, `35d4c6c`, `3f47bb7`, `a086476` | A | 높음 |
| HB-36 | 2024.08~09 | 음악별 JSON block 조건을 실제 인식 결과와 비교해 퀘스트를 순차 완료하고 별 표시·교체 기능 해금·unlock animation까지 연결 | `0881ce8`, `35d4c6c`, `164edf8`, `dc2ad19`, `37446ca` | A(구현) / 기획은 팀 공동 | 매우 높음 |
| HB-37 | 2024.05~10 | 오류/경고별 시각 피드백을 보강한 뒤 접근성 앱에서 QR 1~4 bitmask로 카메라 이동 방향을 추론해 화살표·TalkBack 안내·3개 이상 자동 촬영·촬영 복구까지 폐루프로 통합 | `c3871c9`(선행 피드백), `2c0cf19` | A(구현) / 기획은 팀 공동 | 매우 높음 |
| HB-38 | 2024.10~11 | 저장 음악을 `MusicFile` 단일 JSON 모델로 통합하고 전체/장르별 목록·재생 상태·제목수정/삭제·QR 연동·MP3 다운로드/공유까지 재구성 | `4db36f8`, `1f11d93` | A(구현) / 기획은 팀 공동 | 매우 높음 |
| HB-39 | 2024.10 | 악기별 volume·진동·교체버튼 표시 설정을 `Setting` 모델로 정리하고 설정/진행도 reset·영속화 UI를 통합 | `4db36f8` | A(구현) / 기획은 팀 공동 | 높음 |
| HB-40 | 2024.05~11 | 공통 asset 경로와 metadata 계약을 이용해 Cyberpunk·Classic 등 신규 음악을 Java 수정 없이 추가하고 음원/이미지/Lottie 콘텐츠를 통합·유지보수 | `f924c6d`, `35d7e18`, `eb8a47f`, `64a4afb` 외 | A(통합) / 원본 자산은 팀·외부 제작 | 높음 |
| HB-41 | 2024.12 | 2.0.0 전환에서 pre-v25 설치를 감지해 장르별 구형 저장 음악·block·MP3/cache를 `MusicFile` 구조로 1회 migration하고 version I/O를 `VersionUtil`로 분리 | `36f8253`, `6d66423`, `fbd31c9` | A(구현·디버깅) | 매우 높음 |
| HB-42 | 2024.12 | 재생 화면 버튼·label의 개별 좌표 Guideline을 행·열 단위 공통 percentage Guideline으로 재구성해 화면 비율 변화에 따라 조작 UI 전체가 함께 확대·축소되도록 보정 | `bfc794e`, `3c5b4e2` | A(구현) / 디자인 원안은 디자이너 | 높음 |
| HB-43 | 2024.11~2025.01 | 팀원 CameraX 녹화 prototype을 저장 `MusicFile` 선택·음악 동기화·`tempN.mp4` 분할 녹화·오류/이탈 cleanup이 가능한 제품 흐름으로 재구성 | `83d51f6`(팀원 prototype), `d85faeb`, `719732b`(후속 상태 보완) | A(제품화 구현) / 최초 prototype은 팀원 | 매우 높음 |
| HB-44 | 2024.12~2025.01 | `tempN.mp4` 분할 영상을 FFmpeg concat demuxer로 무재인코딩 결합하고 저장 음악 MP3를 AAC로 mux해 MediaStore/DCIM에 최종 뮤직비디오를 저장 | `d85faeb`, `719732b` | A(구현·후속 동기화 보완) | 매우 높음 |
| HB-45 | 2025.01 | 저장 음악 재생을 생성 MP3 직접 `MediaPlayer` 재생으로 단순화하고 관리 화면의 영상 제작 진입·전용 음악 선택 흐름·접근성 semantics를 최종 통합 | `719732b`; `a2f2037`,`7a1cb0f`는 후속 UI/tutorial fix | A(구현·통합) | 매우 높음 |

## 2. 교차검증 후 통합 작업군

아래 작업군을 **이력서/포트폴리오에서 사용할 상위 단위**로 본다. 위 HB 번호는 커밋·상세 근거 추적용으로 유지하며, 같은 문제를 여러 차례 개선한 항목은 여기서 하나의 연속 경험으로 묶는다.

| 통합 ID | 통합 작업군 | 포함 세부 작업·추가 근거 | 통합 관점 |
|---|---|---|---|
| GROUP-CORE-01 | Android–Python CV/블록 실행 제품화 | HB-02~HB-04, HB-07, HB-08, HB-11, HB-14; `8c21546` | 카메라 입력·방향 보정·Chaquopy·Runner 계약·Python 실행 결과를 하나의 Android 실행 흐름으로 연결 |
| GROUP-AUDIO-01 | 실시간 다중 악기 재생 엔진 | HB-09, HB-10, HB-16, HB-17, HB-29; `61ea93b` | MediaPlayer gap 해결에서 시작해 scheduler phase, pause/resume delay, condition input, countdown까지 하나의 시간축 제어 문제로 통합 |
| GROUP-UI-01 | 화면 비율 기반 반응형 Android UI | HB-12 → `3889c17` → HB-34 → HB-42 | Play 화면에서 시작한 percentage Guideline 방식을 앱 전반으로 확장하고, Figma 시안을 상대 좌표로 재구현한 뒤 공통 행·열 Guideline으로 보정 |
| GROUP-ACCESS-01 | TalkBack·촉각·시각장애 사용자 흐름 | HB-20, HB-21, HB-22, HB-25, HB-37; `c2cca66`, `190a583`, `c3871c9`, `e52f941` | 탐색 순서/음성 설명 → 블록 읽기 → 진동 → 별도 접근성 앱 → QR 위치 기반 촬영 안내로 접근성 수준을 단계적으로 확장 |
| GROUP-MEDIA-01 | 저장 음악·MP3·영상 제작 파이프라인 | HB-26, HB-38, HB-43, HB-44, HB-45; `dbcdde3`, `3889c17` | 재생 상태 저장→MP3 합성→MusicFile 통합→CameraX 분할 녹화→FFmpeg mux→저장 음악 라이브러리/영상 UX로 확장 |
| GROUP-PROGRESS-01 | 교체 악기·퀘스트·진행도 시스템 | HB-30, HB-31, HB-36 | 다음 마디 교체 상태를 저장 가능한 모델로 만들고, 실제 인식 block 조건과 progress·별·unlock animation까지 연결 |
| GROUP-CONTENT-01 | 데이터 기반 음악 콘텐츠 확장 | HB-13, HB-15, HB-40 | 음악별 metadata/path convention으로 장르·음원·캐릭터·Lottie를 코드 수정 최소화 구조에 통합 |
| GROUP-DELIVERY-01 | 대용량 자산·Android 배포 파이프라인 | HB-33, HB-32 | Play Asset Delivery asset pack과 내부 model materialization → Git LFS → AAB universal APK 추출 자동화의 연속 배포/자산 관리 경험 |
| GROUP-RELEASE-01 | 저장 데이터 호환·버전 전환 | HB-41 | v25 이전 설치의 구형 저장 음악을 새 MusicFile 구조로 1회 migration하고 version marker/util을 정리 |
| GROUP-INFERENCE-EXP | Java/TFLite 추론 전환 실험 | HB-23, HB-24 | Python bridge 제거·Java 후처리·전처리 보정·실기기 속도 비교 후 정확도 저하로 rollback한 하나의 실험/판단 사례 |
| GROUP-BPM-01 | BPM 기능과 콘텐츠 호환 처리 | HB-18, HB-19 | BPM 3단계 재생 기능과 BPM 자산이 없는 음악의 fallback을 하나의 기능군으로 통합; 최종 제거 이유는 미확인 |
| GROUP-ONBOARD-01 | 실제 기능 재사용형 튜토리얼 | HB-35 | 실제 Camera/Play/센서/저장 기능을 재사용한 단계형 onboarding과 최초 실행 상태·hold-to-skip을 한 경험으로 유지 |

통합 원칙:
- 세부 HB를 삭제하지 않는다. 동일 문제의 시간적 진화를 보존하는 **근거 레이어**로 사용한다.
- 이력서에서는 한 문제를 여러 bullet로 중복 설명하지 않고 위 GROUP 단위에서 대표 문제·판단·결과를 선별한다.
- `undone`, `revert`, asset-only 교체는 해당 작업군의 시행착오/maintenance 근거로만 사용하고 독립 성과로 세지 않는다.

## 3. 팀 작업 또는 개인 기여 범위 확인이 필요한 항목

아래 항목은 제품에서 중요하지만 최초 또는 핵심 커밋 작성자가 다른 팀원이므로, 개인 경험으로 쓰려면 역할을 먼저 확인한다.

| ID | 작업 | 확인된 대표 작성자·근거 | 사용자에게 확인할 내용 |
|---|---|---|---|
| TEAM-01 | YOLO/OpenCV DNN 추론 모듈과 bounding box 정렬의 초기 구현 | `EunbinSeo`, `7c3b366` | 모델 실행·정렬 중 김재훈이 직접 담당하거나 이후 재설계한 범위 |
| TEAM-02 | 검출 confidence·NMS 등 인식 파라미터 조정 | `EunbinSeo`, `16dbf7e`, `0bfda1f`, `6f80bac` | 현장 피드백·테스트·코드 수정에서 담당한 범위 |
| TEAM-03 | 바코드를 이용한 자동 촬영 기능 | `EunbinSeo`, `0bed5d0`, `5ba4b00` | 기능 설계·통합·유지보수에서 담당한 범위 |
| TEAM-04 | 퀘스트 데이터와 화면의 초기 구현 | `Tinto-Verano`, `34cc098`, `270eb93`, `7523133` | 이후 조건·저장·UI 로직 중 직접 맡은 범위 |
| TEAM-05 | 영상 녹화 화면 초기 프로토타입 | `Tinto-Verano`, `83d51f6` | 프로토타입을 이어받아 `d85faeb`에서 새로 구현하거나 보완한 범위 |
| TEAM-06 | 제품 디자인·캐릭터·음원·모델 학습 | 여러 팀원·외부 리소스 | 개발 통합과 원천 제작을 구분할 필요 |
| TEAM-07 | 그래피툰 QR 연동·block JSON 공유 기능 | `EunbinSeo`, `2e30063`; `QrScanActivity`, Retrofit/OkHttp, `ApiService`, `RetrofitClient` | 개인 HB로 계상하지 않음. 접근성 앱의 바코드 촬영 흐름(HB-27)과는 별개 기능 |

## 4. 교차검증 이후 정리 순서

1. 위 `GROUP-*` 단위별로 겹치는 세부 HB 문장을 하나의 문제 해결 흐름으로 압축한다.
2. `experiences/hummingblocks.md`는 기능 나열이 아니라 4~6개의 강한 통합 경험을 중심으로 다시 작성한다.
3. `resume.md`에는 그중 지원 직무와 직접 연결되는 3~4개의 결과·판단만 남긴다.
4. 정량 수치는 원본 조건이 확인된 것만 사용한다. TFLite 정확도 `약 90%→57%`는 데이터셋·샘플 수가 미확인이라 조건 확인 전 확정 성과 수치로 쓰지 않는다.
5. 구현됐지만 최종 미채택된 TFLite/BPM 등은 제품 기능과 섞지 않고 `실험→검증→판단` 사례로 표현한다.
## 5. 각 작업을 확정할 때 사용할 형식

1. **현상/문제:** 실제로 어떤 상황에서 무엇이 동작하지 않았는가?
2. **원인/제약:** 코드·기기·사용자·일정상 제약은 무엇이었는가?
3. **선택과 판단:** 어떤 대안 중 왜 이 방식을 선택했는가?
4. **구현:** 클래스·라이브러리·데이터 구조·알고리즘 수준에서 무엇을 바꿨는가?
5. **검증:** 어떤 기기·입력·데이터·사용자 환경에서 확인했는가?
6. **결과:** 정량·정성적으로 무엇이 개선됐는가?
7. **근거:** 관련 커밋·코드·당시 문서·사용자 기억은 무엇인가?

## 6. 현재 상태

- 전체 8개 브랜치 commit graph 교차검증 완료: **229 unique commits / `nanocode00` 162 commits**.
- 제품 기준 `app_develop` 199 commits 중 기존 문서에 직접 매핑되지 않은 user non-merge 54개를 역검토했다.
- 최종 브랜치 밖의 user unique commit 21개도 별도 확인했다.
- 새 독립 개발작업으로 **HB-33 Play Asset Delivery 자산 분리/내부 materialization**을 추가했다.
- `3889c17`의 앱 전반 percentage Guideline 전환은 별도 성과로 중복 세지 않고 GROUP-UI-01의 중간 단계로 통합했다.
- `8c21546`, `61ea93b`, `1bce3cf`, `190a583`, `c3871c9`, `e52f941` 등 미참조 핵심 커밋은 기존 HB의 근거·진화 단계에 흡수했다.
- 상세 추적 단위는 예외 없이 `HB-01~HB-45` 연속 번호로 유지한다.
- 이력서용 상위 단위는 `GROUP-*` 작업군을 기준으로 한다.
- 다음 단계는 `experiences/hummingblocks.md`에서 **중복 작업을 통합한 4~6개 핵심 경험 구조**를 만드는 것이다.
- 기존 Draft PR은 계속 draft 상태로 유지하며, `main`에는 반영하지 않는다.
