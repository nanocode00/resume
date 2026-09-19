# 허밍블럭스 이력서 재정리 작업 체크포인트

> 최종 갱신: 2026-09-19  
> 상태: 초기 Android 작업 HB-E01~E05 및 HB-01~HB-36, HB-31S 확인 완료  
> 다음 확인 대상: HB-37 저장 음악·영상 제작 UX의 최종 통합 보완

## 0. 작업 단위 복원 원칙

프로젝트 전체를 한 번에 재작성하지 않고 실제 개발 경험을 아래 형식의 **작업 단위**로 하나씩 복원한다.

`현상/문제 → 원인·제약 → 대안과 선택 근거 → 구체 구현 → 검증·결과`

진행 원칙:

1. GitHub 코드·커밋에서 작업 흔적을 먼저 확인한다.
2. 코드가 증명하는 사실과 사용자 기억이 필요한 사실을 분리한다.
3. 사용자 확인이 끝난 작업만 확정한다.
4. 실제 개발 시점 순서대로 누적한다.
5. 충분히 쌓인 뒤 `experiences/hummingblocks.md`와 `resume.md`를 다시 구성한다.
6. 기존 전체 재작성 Draft PR은 참고 초안으로만 유지하고 검증 완료 전에는 merge하지 않는다.

## 1. 역할·기여 경계

- 2022.10~2023.03 Python/Tkinter·PyQt·pygame·OpenCV DNN 프로토타입의 초기 코드는 다른 팀원이 작성했다.
- Play·Loop·Star, 좌·우·else 조건, 악기 레벨·증감·재생·리셋 등 블록 규칙은 팀 회의에서 재훈님이 주로 의견을 내며 설계했고, 초기 Python 인터프리터 구현·테스트는 다른 팀원이 담당했다.
- 최종 제품은 처음부터 Android로 계획됐고 Python 프로그램은 Android 개발 전 데모/프로토타입이었다.
- 2023-03-20 `cede7ee`에 처음 들어온 초기 Android 앱과 `camtest1`, `acctest` 코드는 사용자 확인상 모두 재훈님 직접 구현이다.
- 초기 Python 코드를 Android에 이식한 작업은 팀원이 먼저 담당했고, 이후 실제 앱에서 발견된 문제를 재훈님이 수정·리팩토링하며 실행 가능 여부와 오류/경고 반환 구조를 보강했다.
- 음원은 전부 외주 제작. 재훈님은 Android 리소스 구조화, 장르/BPM/악기/레벨 매핑, 재생·스케줄링 통합을 담당했다.
- 캐릭터·배경·무대 원본 이미지는 디자이너 제작. 연주 상태 표시용 흑백 캐릭터가 없어 재훈님이 변환 도구를 찾아 직접 생성해 앱에 적용했다.

## 2. 확정 작업 단위

### HB-E01 — 초기 Android 앱 구조와 사용자 흐름 구현

- 팀에서 합의한 `Loading → Main → 장르 선택 → 촬영 → 재생 → 재촬영/홈` 흐름을 Android Activity 구조로 구현했다.
- `Intent`, Activity Result, 권한 처리, 화면 간 상태·결과 전달을 연결했다.
- 표현 시 사용자 흐름을 단독 기획했다고 쓰지 않고, 팀 합의 흐름을 제품 코드로 구현했다고 쓴다.

### HB-E02 — 카메라 preview·촬영 기능 분리 검증 및 제품 통합

- 카메라 기능을 `camtest1` 독립 모듈에서 먼저 구현·시험했다.
- `SurfaceView + SurfaceHolder.Callback`으로 camera open/preview/capture/release와 권한 처리를 검증한 뒤 제품 앱에 통합했다.

### HB-E03 — 촬영 이미지 파일화 및 Python/OpenCV 인식 경로 연결

- 초기에는 휴대폰을 왼쪽으로 90도 돌린 촬영 자세를 기준으로 bitmap에 고정 `90°` 보정을 적용했다.
- Python/OpenCV가 `cv2.imread(path)`로 파일 경로를 요구해 촬영 이미지를 cache JPEG로 저장하고 절대 경로를 Python에 전달했다.
- 성공 시 Python 모듈에 유지된 실행 상태를 `PlayActivity`가 조회해 음악 실행 데이터로 사용했다.

### HB-E04 — 중력센서 기반 left/right 조건 입력 구현

- 팀 기획의 휴대폰 좌/우 기울기 조건 입력을 Android `TYPE_GRAVITY` 센서로 구현했다.
- 별도 `acctest`에서 기울기 각도 계산을 검증한 뒤 `LEFT / RIGHT / MIDDLE` 조건 입력과 지휘자·화살표 UI에 연결했다.

### HB-E05 — 향후 멀티 장르를 고려한 장르 선택 구조 구현

- 초기 실제 음원은 한 세트였지만 이후 멀티 장르 확장을 고려해 `RecyclerView` 기반 선택 UI와 `current_genre` 상태를 먼저 구현했다.
- 이 시점에는 실제 장르별 음원 전환까지 되지 않았으므로 `멀티 장르 지원 완료`로 표현하지 않는다.

### HB-01 — 기존 Python 자산 재사용을 위한 Chaquopy Android–Python 연동

- 기존 Python CV·블록 처리 자산을 Java로 전부 재작성하거나 외부 서버에 의존하지 않고 Android 내부에서 사용하기 위해 대안을 조사했다.
- 서버 통신, NDK/cross compile, Android 내부 Python 실행을 비교한 뒤 Chaquopy를 선택했다.
- `5adb514`에서 Python runtime을 시작하고 Java → Python 함수 호출 → 반환값 표시까지 최소 PoC를 검증했다.

### HB-02 — 기기 방향에 따른 촬영 이미지 회전 보정

- 고정 90° 보정 때문에 휴대폰 방향이 바뀌면 CV 입력 방향이 틀어져 인식이 실패했다.
- 2023-04-06 `8cc6dc9`에서 중력센서로 기기 방향을 판단해 0/90/180° 회전값을 동적으로 적용했다.
- 이후 CameraX + `OrientationEventListener` 구조로 확장하며 0/90/180/270° 4방향을 처리했다.

### HB-03 — 다중 악기 동시 재생과 마디 전환 끊김 개선

- 한 마디에서 여러 악기를 동시에 재생하고 마디 사이를 연속해서 이어야 했다.
- 실제로 마디 전환 때 약 0.5초 끊기는 현상을 확인했고, `MediaPlayer` 생성 지연을 원인으로 판단했다.
- 다음 마디용 MediaPlayer를 미리 준비하고 두 묶음을 번갈아 사용하는 방식으로 전환해 생성과 재생 시점을 분리했다.

### HB-04 — 교차 예약된 오디오 재생의 pause/resume/stop 상태 관리

- 두 MediaPlayer 묶음을 번갈아 예약하는 구조에서는 현재 재생 위치뿐 아니라 다음 예약 상태도 함께 관리해야 했다.
- `Timer/TimerTask` 예약을 pause 시 취소하고 재생 위치·경과 상태를 저장한 뒤 resume 시 예약 지연을 다시 계산해 복원했다.
- 정확한 과거 버그 증상은 기억이 희미하므로 특정 현상을 단정하지 않는다.

### HB-05 — Python 실행기의 초기 Android E2E 통합

- 2023년 4월 촬영 이미지 → Python CV/블록 실행 → `PlayActivity` → 음악 재생까지 실제 앱 흐름이 처음 연결됐다.
- 초기 Python 실행 코드의 Android 이식은 다른 팀원이 담당했다.
- 재훈님은 Android 측 촬영·호출·결과 처리·재생 화면 연결을 포함한 제품 통합 흐름을 다뤘다.
- 이 단계의 실제 사용에서 Python 실행기 버그와 예외 처리 부족이 드러났고, 이후 HB-08 리팩토링으로 이어졌다.

### HB-06 — PlayActivity 반응형 레이아웃 전환

- 2023-04-25 `4fb4957`에서 `PlayActivity`를 독립적으로 반응형 레이아웃으로 개편했다.
- 9:16 콘텐츠 영역을 유지하면서 고정 dp 크기·margin 중심 배치를 percentage Guideline 사이의 `0dp` constraint 구조로 전환했다.
- 이후 태블릿/Z Flip에서 다른 화면 문제가 발견되어 Main/Home 등에도 별도 대응했으며, 이 작업은 HB-06과 분리한다.

### HB-07 — 외주 음원을 멀티 장르 재생 구조에 통합

- 음원 제작은 외주이며 재훈님은 Android 통합을 담당했다.
- 2023-05-02 `5efb0ac`에서 기존 `악기 × 레벨` 음원 구조를 `장르 × 악기 × 레벨` 구조로 확장했다.
- Jazz/Pop 등 현재 장르에 따라 실제 음원 세트를 선택하고 장르별 마디 길이를 재생·pause/resume 스케줄링에 반영했다.

### HB-08 — Python 블록 실행기 리팩토링 및 예외 처리 체계화

- HB-05 초기 통합 후 실사용에서 드러난 버그와 예외 처리 부족을 별도 안정화 작업으로 다뤘다.
- 2023-05-02 `c7a7d3c`에서 `block_runner.py`, 오류/경고 목록을 만들며 구조 재정리를 시작했다.
- 2023-06-08에는 `Runner.java + block_runner.py` 구조로 앱에 통합하고 `SUCCESS / ERROR / WARNING` 결과 계약을 분리했다.
- 이후 검증 규칙을 확장해 최종적으로 23종 오류와 2종 경고를 구분해 사용자에게 안내하는 구조로 발전했다.

### HB-09 — 음악별 캐릭터·무대 테마와 연주 상태 시각화

- 원본 캐릭터·배경·무대 이미지는 디자이너가 제작했다.
- 음악별 배경·무대·5개 악기 캐릭터와 배치 비율을 묶어 `PlayActivity`에서 선택 음악에 따라 동적으로 적용했다.
- 현재 연주 중인 악기는 컬러, 미사용 악기는 흑백으로 표시해 블록 실행 결과를 시각화했다.
- 흑백 캐릭터 리소스가 별도로 없어 재훈님이 변환 도구를 찾아 직접 생성해 추가했다.

### HB-10 — 재생 스케줄러 현대화 및 준비/재생 단계 분리

- 오래된 `Timer/TimerTask` 기반 구조를 더 현대적이고 제어가 명확한 `ScheduledThreadPoolExecutor`로 교체했다.
- `PrepareTrack1 → StartTrack1 → PrepareTrack2 → StartTrack2`로 준비와 실제 재생 시점을 분리했다.
- pause/resume도 `phase`와 `ScheduledFuture.getDelay()`를 이용해 예약 상태를 재구성하도록 바꿨다.
- 주된 동기는 특정 버그보다 오래된 Timer 기반 구조의 교체와 스케줄링 구조 정리였다.

### HB-11 — 마디 진행률 및 실제 조건 입력 시각화

- 사용자가 다음 마디/입력 시점을 예상할 수 있도록 스케줄러의 남은 시간을 이용해 현재 마디 진행률을 표시했다.
- 마디 준비 시점에 확정한 left/right/else 값을 화면 아이콘으로 보여주고, 동일 값을 Python `play_block(input)`에도 전달했다.
- 목적은 사용자가 자신의 기울기 입력이 실제 조건 평가에 제대로 반영됐는지 확인하도록 하는 것이었다.

### HB-12 — 3단계 BPM 전환 및 마디별 템포 제어

- 2023-09-21~27에 느림/보통/빠름 3개 버튼으로 사용자가 연주 도중 BPM을 선택하고 다음 마디부터 템포를 바꾸는 기능을 구현했다.
- playback speed를 단순 변경하지 않고 BPM별로 외주 제작된 별도 음원 세트를 매핑했다.
- `60000 / BPM × beat 수`로 마디 길이를 계산하고 BPM 변경 시 음원 선택과 ScheduledThreadPoolExecutor 주기를 함께 재구성했다.
- 기능은 구현·검증됐지만 이후 최종 제품 방향에서는 제거됐다. 제거 이유는 현재 기억과 코드만으로 확인되지 않으므로 추정하지 않는다.

### HB-13 — BPM 미지원 음악 호환 처리

- 모든 기존 음악이 느림/보통/빠름 3단계 BPM 음원을 가진 것은 아니었기 때문에 BPM 기능 추가 후 기존 음악이 재생 불가능해지는 문제를 막아야 했다.
- 2023-10-04 `44037d8`에서 음악 메타데이터에 `bpm_available`을 추가했다.
- BPM 미지원 음악은 기본 BPM만 사용하고 BPM 선택 버튼을 숨기도록 처리해 동일 재생 엔진에서 BPM 지원/미지원 콘텐츠를 함께 재생할 수 있게 했다.
- 이후 최종 `app_develop`에서는 사용자 BPM 선택 기능 자체가 제거되고 음악별 단일 BPM 구조로 정리됐다.

### HB-14 — TalkBack 탐색 순서 및 동적 상태 안내 보강

- 실사용 과정에서 TalkBack의 읽기·탐색 순서가 실제 사용 흐름과 맞지 않는 문제가 드러나 접근성 순서를 보강하기로 했다. 이 문제의 출발점은 사용자 기억으로 확인했으며, 특정 외부 사용자 테스트에서 나온 피드백이라고 단정하지 않는다.
- 2023-10-04 `44037d8`에서 버튼·이미지의 `contentDescription`을 현재 음악명과 동작이 드러나는 문장으로 구체화하고, 조작 불가능하거나 장식적인 요소는 `importantForAccessibility`로 탐색 대상에서 제외했다.
- 촬영 후 이미지 처리 중에는 카메라 버튼과 방향 안내를 일시적으로 TalkBack 탐색에서 제외하고 `AccessibilityEvent.TYPE_ANNOUNCEMENT`로 `로딩중입니다`를 직접 알린 뒤, 처리가 끝나면 다시 접근성 대상으로 복구했다.
- 재생 화면에서도 BPM 버튼이 조작 불가능한 동안 접근성 대상에서 제외하고 다시 조작 가능해질 때 복구해, TalkBack 사용자가 현재 실제로 누를 수 있는 요소만 탐색하도록 했다.
- 저장 음악·선택·재생·결과 보기 화면에는 `accessibilityTraversalBefore/After`를 적용해 화면 배치 순서와 TalkBack 탐색 순서가 어긋나는 문제를 보정했다.
- 처음에는 Java의 `setAccessibilityTraversalBefore/After()`로 순서를 지정했지만, 2023-10-06 `7a06422`에서 대부분을 XML의 `android:accessibilityTraversalBefore/After`로 옮겨 화면별 탐색 순서를 레이아웃 정의에서 명시적으로 관리하도록 정리했다.
- 결과 보기 화면의 블록 설명도 `드럼0` 같은 축약형에서 `드럼 0단계 실행`, `왼쪽`에서 `왼쪽 기울이면`, `반복`에서 `계속 반복 실행`처럼 음성만 들어도 의미를 파악하기 쉬운 표현으로 보강했다.

### HB-15 — 인식된 블록 결과 확인 화면 추가·개편

- 2023-10-05 `0ac520c`에서 `PlayActivity`에 `인식 결과 확인` 버튼과 별도 `ResultViewActivity`를 추가했다.
- Python `block_runner.py`에 `get_result()`를 추가해 내부 `block_list`를 Java로 반환하고, Android에서 각 블록 코드를 `block_label`과 색상 배열에 매핑해 인식된 블록 순서를 사람이 읽을 수 있는 형태로 표시했다.
- 목적은 사용자가 CV가 인식한 블록 결과를 확인하는 동시에, 촬영한 물리 블록이 앱 내부에서 어떤 블록 코드·실행 의미로 변환됐는지 확인할 수 있게 하는 것이었다. 따라서 단순 디버그 화면이 아니라 `인식 결과 확인 + 변환 과정 가시화` 기능으로 정리한다.
- 초기 구현은 `GridLayout`에 `TextView`를 동적으로 추가했으며 `PLAY / LOOP / STAR` 시작 지점 앞에 빈 항목을 넣어 블록 묶음을 시각적으로 구분했다.
- 2023-10-06 `7a06422`에서는 결과 목록을 `RecyclerView + LinearLayoutManager`로 개편하고, 블록 묶음 구분용 `-1` 항목을 리스트에 삽입하는 방식으로 데이터와 렌더링 구조를 분리했다. 고정 GridLayout보다 가변 길이 결과를 목록으로 관리하기 쉬운 구조가 됐지만, 실제 개편 동기는 코드만으로 단정하지 않는다.
- TalkBack 탐색에서도 결과 목록을 먼저 읽고 뒤로가기 버튼으로 이동하도록 traversal을 연결했으며, HB-14에서 블록 라벨 자체도 음성으로 의미를 이해하기 쉬운 문장형으로 보강했다.
- 사용자 확인상 인식 정확도를 눈으로 검증하고, 물리 블록이 앱에서 어떤 의미로 해석됐는지 보여주기 위해 추가한 기능이다.

### HB-16 — 박자 진동 피드백과 사용자 설정 저장

- 2023-10-11 `28468a6`에서 음악 재생 중 촉각으로 진행을 알리는 진동 피드백과 이를 제어하는 설정 항목을 추가했다.
- `MusicPlayer`는 각 마디의 길이를 8등분한 주기로 `Vibrate` 작업을 예약했다. 마디 첫 진동은 강도 255, 이후 진동은 51로 구분해 마디 시작과 이후 진행을 촉각적으로 다르게 전달했다.
- pause 시 다음 진동까지 남은 지연시간을 보존하고 resume 시 동일한 주기로 다시 예약했으며, stop이나 새 마디 시작 시 진동 카운터를 초기화해 기존 음악 스케줄러와 동기화했다.
- 설정 화면에는 기존 5개 악기별 음량 조절에 더해 `진동 사용 여부`와 `BPM 버튼 표시 여부` 스위치를 추가했다. BPM 버튼을 숨기면 재생 화면에서 중간 BPM을 기본값으로 사용하도록 처리했다.
- 설정값은 `Setting`의 정적 상태로 관리하고 앱 내부 `setting.json`에 악기별 음량, 진동 사용 여부, BPM 버튼 표시 여부를 JSON으로 저장·복원했다.
- 이후 최종 `app_develop`에서도 진동 설정은 `enable_vibration` 이름으로 유지됐다. BPM 버튼 표시 설정은 제품 기능 변화에 따라 다른 설정으로 교체됐으므로, 2023년 기능을 최종 제품까지 그대로 유지했다고 표현하지 않는다.
- 팀 내에서 음악 진행을 시각 정보 외에도 촉각으로 느낄 수 있으면 좋겠다는 요구가 나와 추가한 기능이다. 이를 기존 음악 스케줄러와 동기화해 구현했다.

### HB-17 — Chaquopy/OpenCV 추론 경로의 Java/TFLite 전환 실험

- `7cdc9e1`에서 앱 모듈의 Chaquopy 플러그인과 Python의 NumPy/OpenCV 의존성을 제거하고 TensorFlow Lite 2.9.0 의존성을 추가해, CV 추론을 Android Java 내부에서 직접 수행하는 실험을 진행했다.
- 기존 Python `cv2.dnn` 추론 대신 Java `Classifier`를 추가하고 TFLite `Interpreter`를 이용해 416×416 입력 tensor 생성, 추론, confidence filtering, NMS를 구현했다.
- NNAPI → GPU delegate → CPU thread 순으로 TFLite 실행 backend를 초기화하도록 fallback 구조를 만들었다.
- CameraX의 `ImageProxy`를 Bitmap으로 변환해 416×416 입력으로 만들고, Java에서 얻은 detection 결과를 `Runner`에 전달하도록 변경했다.
- 블록의 x/y 위치 정렬과 그룹화도 Python 결과에 의존하지 않고 Java `Runner`에서 detection bounding box의 평균 크기와 threshold를 이용해 처리하도록 옮겼다.
- 즉 단순히 모델 파일 형식만 바꾼 것이 아니라, Python bridge에 있던 CV 추론·후처리·블록 정렬 경로를 Android 네이티브 코드 쪽으로 옮기는 실험이었다.

### HB-18 — TFLite 입력 전처리 보정 후 정확도 저하로 원복

- 후속 `fc771fd`에서는 CameraX 입력을 416×416으로 만드는 과정에서 가로·세로 비율이 왜곡되던 전처리를 수정했다.
- 기존의 서로 다른 x/y scale 적용 대신 `ratio = 416 / width` 하나로 등비율 scale을 적용하고, 남는 좌우 영역을 검은 padding으로 채우도록 변경해 원본 비율을 유지했다.
- 이 변경은 Python/OpenCV 경로가 사용하던 padding 기반 전처리와 더 유사한 입력을 만들기 위한 보정으로 볼 수 있다.
- 그러나 이후 `1bb3f7c`에서 Java TFLite `Classifier`와 Bitmap 전처리 경로를 제거하고 Chaquopy, NumPy, OpenCV 의존성을 다시 추가했다. 촬영 이미지를 파일로 저장한 뒤 Python `Classifier.classify()`를 호출하는 구조로 복귀했다.
- 따라서 TFLite 경로는 제품 최종 채택이 아니라 성능 개선을 위한 실험으로 분리한다.
- 속도 비교는 여러 대의 Samsung Galaxy S9+ 기기를 준비해 각 기기에서 기존 Chaquopy/OpenCV 버전과 TFLite 버전을 각각 1회씩 실행해 측정했다. 당시 기록은 약 `9.1초 → 5.4초`로 TFLite 쪽이 빨랐다. 반복 측정 평균이 아니라 기기별 1회 비교였으므로 이력서에서는 정밀 benchmark보다 `실기기 비교에서 추론 지연 감소` 정도로 표현한다.
- `약 90% → 57%` 정확도 저하 수치도 당시 기록으로 남아 있지만, 정확도 평가에 사용한 데이터셋·샘플 수·집계 방식은 현재 확인되지 않아 속도 benchmark 조건과 혼동하지 않고 별도 미확인 항목으로 유지한다.
- 미채택 판단은 속도 이점보다 정확도 하락이 제품 핵심 기능인 블록 인식에 더 큰 리스크였다는 기존 기록과 일치하지만, 정확한 의사결정 문서가 발견되기 전까지는 이 이상의 원인을 확대 해석하지 않는다.

### HB-19 — 시각장애 사용자용 독립 Android 앱 모듈 분리

- 2024-04의 `88d70d6`에서 기존 앱 프로젝트에 `hummingblocks_for_blind_ones` 모듈을 새로 추가하고 별도 namespace/applicationId(`com.nemo.hummingblocks_for_blind_ones`)를 부여했다.
- 새 모듈은 기존 제품의 `MusicInfo`, `MusicPlayer`, `Setting`, Python `Classifier` 등 핵심 로직을 복사·이식해 출발했으며, 단순 테마 변형이 아니라 별도 APK로 빌드 가능한 독립 앱 구조였다.
- `e0356b8`에서 분리 작업을 마무리하면서 launcher를 일반 메인 화면이 아닌 `CameraActivity`로 두고, 앱 흐름을 `촬영 → 인식 → 재생` 중심으로 단순화했다. 별도의 MainActivity는 이 시점에 제거됐다.
- 촬영 화면 진입 시 TalkBack `TYPE_ANNOUNCEMENT`로 “4개 QR코드가 화면에 들어오도록 찍어주세요” 같은 촬영 가이드를 직접 읽어주고, 처리 중에는 촬영 버튼을 접근성 탐색 대상에서 제외했다.
- 재생 화면 역시 `contentClicker` 등 현재 조작 가능한 요소만 `importantForAccessibility`로 노출하도록 상태에 따라 접근성 대상을 바꿨으며, 기존 BPM 조작 UI는 주석 처리해 흐름을 줄였다.
- 반면 기존 앱의 음악 재생 스케줄러, 중력센서 기반 방향 입력, 진동 피드백, Python 분류기와 `Runner`의 블록 검증·실행 로직은 상당 부분 재사용했다.
- 기존 앱은 기능과 화면이 이미 많이 늘어나 구조가 복잡해진 상태였고, 팀 내에서 TalkBack 지원만 덧붙이는 방식으로는 시각장애 사용자가 원활하게 사용하기 어렵다고 판단했다.
- 이에 기존 앱의 모든 UX를 접근성에 맞게 억지로 보정하기보다, 핵심 기능만 남긴 단순한 `촬영 → 인식 → 재생` 흐름의 별도 앱을 만드는 방향을 선택했다.
- 따라서 이 작업은 기존 앱 전체를 새로 구현한 것이 아니라, 공통 실행 코어를 재사용하면서 접근성 요구에 맞춘 단순 UX를 독립적으로 발전시키기 위해 별도 Android 앱 모듈로 분기한 작업으로 정리한다.

### HB-20 — 재생 상태 기반 MP3 합성·저장 및 공유

- 2024-05의 `ce30682`에서 사용자가 만든 음악을 JSON뿐 아니라 실제 MP3 파일로 저장·공유할 수 있도록 `mobile-ffmpeg-full 4.4`를 도입했다.
- 재생 중에는 각 마디가 시작될 때 현재 5개 악기의 레벨과 BPM level을 `[drum, brass, guitar, piano, bass, bpm]` 형태의 `JSONArray`로 `music_data`에 누적했다.
- 저장 시 이 JSON을 그대로 파일로 보존하면서, 같은 데이터로 실제 사용할 악기 MP3 경로를 다시 계산했다. 즉 재생 화면의 소리를 녹음한 것이 아니라 `재생 상태 데이터 → 원본 악기 음원 조합` 방식으로 결과 오디오를 재구성했다.
- 각 마디에서는 레벨이 1 이상인 악기 음원만 선택하고, 아무 악기도 없는 마디는 별도의 `empty.mp3`를 사용해 마디 길이를 유지했다.
- FFmpeg `amix`로 한 마디 안의 여러 악기 음원을 동시에 합성한 뒤, 마디별 merge 결과를 `concat`으로 순서대로 연결해 하나의 MP3로 만들었다.
- 초기 구현 후 `273e11a`에서 `amix` 시 입력 수만큼 `volume`을 보정해 여러 트랙을 합칠 때 음량이 과도하게 작아지는 문제를 수정했다.
- 저장 파일명은 사용자가 지정할 수 있게 하고 금지 문자를 검사했으며, 동일 이름이 존재하면 숫자를 붙여 JSON/MP3 파일이 덮어써지지 않도록 했다.
- 저장 음악 화면에서는 JSON 공유와 MP3 공유를 분리했다. MP3 파일이 없으면 저장된 JSON을 다시 읽어 `AudioUtil.saveMp3FromJSON()`으로 생성한 뒤 `FileProvider`를 통해 `audio/mpeg` 형식으로 공유했다.
- 따라서 이 작업은 실시간 오디오 캡처가 아니라, 이미 구조화해 둔 음악 실행 데이터를 재사용해 동일한 결과를 오프라인에서 합성·내보내는 기능으로 정리한다.

### HB-21 — 바코드 자동 촬영 흐름의 lifecycle·권한 처리 안정화 및 선택 UI 보완

- 바코드 자동 촬영 기능의 최초 구현은 팀원(EunbinSeo)의 `0bed5d0`, `5ba4b00`에서 시작됐다. ML Kit Barcode Scanning과 CameraX `ImageAnalysis`를 이용해 화면에서 여러 바코드를 감지하면 자동으로 촬영하는 구조였다.
- 재훈님의 `8353f20`에서는 `BarcodeActivity` 내부의 카메라 권한 확인·요청 코드를 제거했다. 이 시점에는 `SelectActivity`가 launcher이자 진입 화면으로 권한을 먼저 확인한 뒤 `BarcodeActivity`로 이동하므로, 촬영 화면에서 같은 권한 흐름을 다시 처리하지 않도록 책임을 한 곳으로 모았다.
- `48af586`에서는 `BarcodeActivity`의 `setupCamera()` 호출을 `onCreate()`에서 `onStart()`로 옮겼다. 기존 코드는 `onStop()`에서 `ProcessCameraProvider.unbindAll()`을 호출하면서도 카메라 초기화는 최초 생성 시 한 번만 했기 때문에, 재생 화면 등 다른 Activity에 갔다가 돌아오는 경우 카메라 use case가 다시 bind되지 않을 수 있는 lifecycle 문제가 있었다.
- 이를 `onStart() → setupCamera()` 구조로 바꿔 화면에 다시 진입할 때 CameraX preview, ImageCapture, ImageAnalysis와 ML Kit scanner를 재구성하도록 했다. `onStop()`에서는 기존처럼 use case를 해제해 화면을 벗어난 동안 카메라 리소스를 정리했다.
- 같은 커밋에서 음악/설정 초기화와 YOLO weight·cfg 파일 복사 작업도 `BarcodeActivity`가 아니라 launcher인 `SelectActivity`에서 수행하도록 옮겨, 촬영 화면이 카메라·인식 역할에 더 집중하도록 정리했다.
- `SelectActivity`의 음악 버튼은 `GridLayout`의 row/column weight를 제대로 사용하도록 동적으로 만든 버튼의 width/height를 0으로 설정했고, padding을 25dp에서 20dp로 조정해 가변 음악 개수에서도 셀을 균등하게 채우도록 보완했다.
- `onResume()`에서는 현재 Locale에 맞춰 동적 음악 버튼의 label을 다시 적용하도록 해 화면 복귀 시 표시 문자열을 갱신했다.
- 따라서 개인 기여는 `바코드 자동 촬영 기능 자체 개발`이 아니라, 팀원이 만든 자동 촬영 기능을 시각장애 사용자용 앱의 실제 화면 흐름에 통합하면서 권한 책임 중복과 CameraX lifecycle 문제를 정리하고 선택 UI를 안정화한 작업으로 표현한다.
- 실제로 재생 화면 등 다른 Activity에 갔다가 촬영 화면으로 돌아오면 카메라 preview와 자동 촬영이 다시 동작하지 않는 문제가 있었고, 이를 CameraX lifecycle 처리 문제로 확인해 수정했다.

### HB-22 — 재생 pause/resume과 Lottie 애니메이션 상태 동기화

- 2024-07-05 `c63b2bb`에서 시각장애 사용자용 앱의 `PlayActivity`에 재생 일시정지·재개와 Lottie 애니메이션 상태를 함께 제어하는 처리를 추가했다.
- 기존에는 음악 스케줄러가 pause되어도 현재 연주 중인 악기 캐릭터 Lottie가 계속 움직이거나, 재개 시 음악과 애니메이션 상태가 어긋날 수 있었다.
- `onPauseMusic()`에서 현재 `inst_level`이 0보다 큰 악기의 `LottieAnimationView.pauseAnimation()`을 호출하고, 조건 입력 대기 상태가 아닐 때의 지휘자 애니메이션도 함께 pause하도록 했다.
- `onResumeMusic()`에서는 같은 조건으로 악기 애니메이션과 지휘자 애니메이션을 `resumeAnimation()`해 음악 재생 상태와 시각 피드백을 다시 맞췄다.
- 따라서 이 커밋은 오류 안내 UI 작업이 아니라 `음악 재생 상태 ↔ Lottie 시각 피드백 상태`의 일관성을 보강한 유지보수 작업으로 정리한다.
- 기존 inventory가 함께 묶었던 `2c0cf19`는 2024-10-12의 대규모 시각장애 사용자용 촬영·방향 안내 UX 개편으로 확인됐으므로 HB-22 근거에서 제외하고 HB-30에서 별도로 다룬다.

### HB-23 — 첫 재생 준비 시간을 3·2·1 countdown으로 시각화

- 2024-07-26 `3ef066b`에서 재생 버튼을 누른 직후 첫 음악이 실제로 시작되기 전의 준비 구간을 `3 → 2 → 1` countdown으로 표시했다.
- 직전 `8ff5698`에서 `initial_input_length`가 1000ms에서 2000ms로 늘었고, `MusicPlayer`는 첫 트랙을 `initial_delay + prepare_length(500ms)` 뒤에 시작한다. 따라서 첫 실제 재생까지의 대기 시간은 총 약 2500ms였다.
- countdown은 833ms 주기로 실행되어 3, 2, 1을 표시한다. `833ms × 3 ≈ 2499ms`이므로 첫 `StartTrack` 시점과 사실상 동일한 길이로 맞춰져 있다.
- 즉 countdown은 별도 타이머를 임의로 붙인 것이 아니라 기존 오디오 준비·스케줄링 지연 시간을 사용자가 인지할 수 있는 시각적 준비 신호로 바꾼 것이다.
- 재생 시작 시 countdown 배경과 텍스트를 표시하고, 첫 트랙이 실제 시작되는 `onPostStartTrack()` 시점에는 이를 숨기고 countdown task를 취소했다.
- 준비 구간 중 pause가 발생해도 countdown이 처음부터 다시 시작되지 않도록 남은 `ScheduledFuture` delay를 `count_down_delay`에 저장하고 resume 시 그 지연값으로 다시 schedule했다.
- 이 기능은 일반 `app` 모듈의 `PlayActivity`에 구현됐으며, 이 커밋에는 별도의 TalkBack announcement가 추가되지 않았다. 따라서 접근성용 음성 countdown으로 확대 해석하지 않는다.
- 기존 inventory에서 HB-23에 함께 묶였던 `8ff5698`의 원형 progress overlay는 실제로 악기 교체 버튼의 다음 적용 시점/남은 마디 진행을 보여주는 UI이므로 HB-24의 악기 교체 기능 쪽 근거로 이동한다. 다만 같은 커밋의 `initial_input_length 1000→2000ms` 변경은 HB-23 countdown 준비 시간과 직접 연결된다.

### HB-24 — 다음 마디 단위 악기 교체 기능과 저장·상태 일관성 보완

- 2024-07의 `4d37e3b`에서 일부 음악에 대해 기존 5개 악기 중 지정된 악기를 별도의 교체 음원으로 바꾸는 기능을 추가했다. `MusicInfo`에 `replace_available`과 3개 교체 대상 인덱스 `replace_data`를 넣고, 교체 가능 음악만 추가 음원 `replace0~2`와 전용 버튼 리소스를 로드하도록 했다.
- 교체 버튼은 3개의 `CheckBox`로 관리했다. 사용자가 현재 마디 중 버튼을 바꾸면 `onPrepareTrack()`에서 체크 상태를 읽어 `next_replace`로 넘기고, `MusicPlayer.prepareTrack()`이 다음 마디용 MediaPlayer를 준비할 때 실제 사용할 음원 인덱스를 바꿨다. 따라서 재생 중 즉시 소리를 갈아끼우는 것이 아니라 기존 prepare/start 이중 버퍼 구조에 맞춰 **다음 마디부터 교체 상태를 적용**하는 방식이었다.
- `replace_data[i]`는 `replace0~2`가 원래 5개 악기 중 어느 악기를 대신하는지를 나타냈다. `next_replace[i]`가 켜지면 원래 악기 인덱스를 `i + 5`의 교체 음원 인덱스로 치환하되, 음량·레벨 값은 원래 악기의 level을 그대로 사용했다.
- 초기 `4d37e3b`에는 이 매핑에서 오류가 있었다. 교체 후 음원 인덱스가 5~7이 될 수 있는데 `next_inst_level[inst]`처럼 교체된 인덱스로 5개짜리 레벨 배열을 조회할 수 있는 구조였다. `c22b525`에서 level 조회는 항상 `next_inst_level[inst_order[i]]`처럼 원래 악기 위치 0~4를 사용하고, `getInstrumentMusicSource()`에 전달하는 음원 인덱스만 교체된 `inst`를 쓰도록 분리했다.
- 같은 `c22b525`에서는 `replace_available == false`인 음악에서 `replace` 메타데이터를 무조건 읽지 않도록 조건화하고, 교체 가능 음악에서만 기본 5개 악기 외 `replace0~2` 리소스까지 총 8종의 음원을 로컬 파일로 준비하도록 정리했다.
- 동일한 원본 악기를 두 개 이상의 교체 버튼이 가리킬 수 있는 경우 동시에 여러 교체가 활성화되지 않도록, 한 버튼을 켜면 같은 `replace_data`를 가진 다른 버튼의 체크를 해제했다.
- `063713c`에서는 저장 데이터 구조를 본격적으로 수정했다. 초기 구현은 `current_replace.put(replace)`처럼 boolean 배열 전체를 각 항목에 잘못 넣고, `level/replace`를 담은 `current_section`을 만들어 놓고도 실제 `music_data`에는 `current_level`만 넣는 오류가 있었다.
- 이를 마디별 `{ "level": [5개 level], "replace": [3개 boolean] }` 객체를 저장하도록 고쳤다. `SavedMusicActivity`도 같은 구조로 level과 replace 상태를 읽어 `setNextInstLevel()`과 `setNextReplace()`에 함께 전달해, 저장한 음악을 다시 재생할 때 당시 교체 상태까지 재현하도록 했다.
- MP3 내보내기 역시 `AudioUtil`이 저장 JSON의 `replace` 값을 읽어 각 마디에서 원래 악기 파일 대신 `replace0~2` 음원을 선택하도록 확장했다. 따라서 화면에서 들은 교체 결과와 저장 음악 재생·MP3 합성 결과가 같은 데이터 계약을 사용하게 됐다.
- 같은 커밋에서는 교체 상태 변화만 감지하기 위해 `prev_replace`와 XOR(`replace[i] ^ prev_replace[i]`)을 사용했다. 여러 교체 버튼이 같은 원본 악기를 대상으로 할 때 한 교체를 끄더라도 다른 교체가 계속 활성 상태라면 원본 캐릭터/애니메이션으로 잘못 되돌리지 않도록 추가 조건을 뒀다.
- `1969c30`의 일부 수정에서는 reset 시 교체 대상 캐릭터를 복원할 때 잘못된 drawable 인덱스를 사용하던 부분을 `drawables[replace_data[i] + 5]`로 고쳤다. 같은 커밋의 Star 사용 여부 시각화는 별도의 기능 변화가 섞인 부분이므로 악기 교체 핵심 기여와 구분한다.
- `8ff5698`에서는 교체 버튼을 누른 직후 바로 다시 조작하지 못하도록 버튼을 잠시 비활성화하고, 버튼 위에 원형 progress overlay를 표시했다. 이 progress는 `MusicPlayer.getLeftTime()`에서 계산한 현재 마디 진행률의 역값(`100 - progress`)을 사용해 **교체가 실제 다음 마디에 적용될 때까지 남은 시간**을 보여줬다. 마디 준비/전환 뒤에는 overlay를 숨기고 버튼을 다시 활성화했다.
- 결과적으로 교체 기능은 단순 UI 토글이 아니라 `현재 UI 선택 → next_replace → 다음 마디 MediaPlayer 준비 → 캐릭터/Lottie 상태 → 저장 JSON → SavedMusic 재생 → FFmpeg MP3 합성`까지 동일한 교체 상태를 전달하도록 확장된 기능이었다.
- 악기 교체를 포함한 제품 기능 기획은 대체로 팀 내부 논의에서 나온 것으로 확인했다. 개인 기여는 코드로 확인되는 Android 구현·통합·디버깅 범위로 표현한다.

### HB-25 — 퀘스트 진행도 저장을 boolean 배열에서 순차 단계 모델로 재설계

- `e837550`에서는 먼저 음악별 퀘스트 안내 문구를 코드에 하드코딩하지 않고 `music/<music>/dialog_text.json`에서 로드하도록 바꿨다. `MusicInfo.getProgressString(Locale)`이 현재 locale의 `introduction`·`description`을 읽고, 지원하지 않는 locale이면 `en-US`로 fallback하도록 했다.
- 이전 진행도 저장은 `ProgressSetting`이 음악별로 길이 3의 boolean 배열(`[false,false,false]`)을 `progress.json`에 저장하는 구조였다. `PlayActivity`가 현재 음악의 배열을 직접 읽어 각 교체 버튼의 잠금/해금 여부를 판단했고, Activity 종료 시 배열 전체를 다시 저장했다.
- 이 구조는 각 퀘스트가 서로 독립된 boolean이라 `현재 몇 번째 단계까지 순서대로 진행했는가`를 직접 표현하지 못했고, 저장 형식도 3개 단계에 강하게 고정되어 있었다. 초기화 코드 또한 하나의 정적 `boolean[] progress`와 `JSONArray`를 음악별 저장에 재사용하는 구조여서 상태 책임이 분산돼 있었다.
- `28bee75`에서 `ProgressSetting`을 제거하고 별도 `Progress` 클래스로 교체했다. 앱 전체 진행 상태는 `tutorial_done`과 음악별 정수 `quest_progress`로 관리한다.
- `progress.json`은 `{"tutorial_done": false, "quest_progress": {"pop": 1, "jazz": 0, ...}}` 형태가 되었고, 각 음악의 정수값은 `0~3` 범위에서 현재까지 몇 개의 퀘스트/교체 기능이 해금됐는지를 의미한다.
- `LoadingActivity`에서 `Progress.Load()`를 호출하고 파일이 없거나 JSON 일부를 읽지 못하면 기본값(모든 음악 progress 0)을 유지한 채 `Progress.Save()`로 다시 저장하도록 초기화 흐름을 통합했다. 음악 목록 전체를 기준으로 key를 순회하므로 특정 replace 지원 장르만 따로 모아 초기화하던 이전 구조도 제거됐다.
- `Progress.Load()`는 음악별 key 읽기를 개별 try/catch로 감싸 일부 음악 key가 빠진 경우 해당 항목은 0을 유지하고 load 실패 상태를 반환하도록 했다. 이는 이후 음악이 추가되거나 저장 파일이 부분적으로 오래된 상태에서도 기본값으로 복구할 수 있는 기반이 됐다.
- 재생 화면에서는 `i < Progress.quest_progress[current_music]`이면 해당 교체 버튼을 보이고, 아직 도달하지 않은 단계는 lock 버튼으로 표시해 **순차 해금 상태**를 정수 하나로 렌더링한다.
- 잠금 버튼을 누르면 기존 `DialogProgress` 대신 `DialogQuest`를 열고, 음악별 JSON에서 읽은 퀘스트 제목·설명을 보여준다. 확인 동작에서 `index == current quest_progress`인 경우에만 값을 `index + 1`로 증가시키고 즉시 `Progress.Save()`한 뒤 해당 lock을 실제 교체 버튼으로 바꾼다. 따라서 이미 지난 단계나 뒤 단계가 진행도를 건너뛰어 변경하지 못하도록 했다.
- 이 시점의 코드는 퀘스트 안내 확인 자체가 해금 트리거이며, `quest.json`의 블록 조합(`match_block`)을 실제 현재 인식 결과와 비교해 성공 여부를 자동 판정하는 로직은 아직 연결되지 않았다. 실제 완료 조건·별 표시·해금 애니메이션은 HB-29에서 별도로 다룬다.
- 같은 커밋에서 결과/삭제/퀘스트 다이얼로그 레이아웃도 각각 전용 배경·버튼·Guideline 기반 구조로 재작성했다. HB-25에서는 이 중 진행도 해금 흐름과 직접 연결된 `DialogQuest` 재설계를 핵심으로 보고, 다른 다이얼로그의 시각 개편은 UI 전반 개편 작업과 겹치므로 과대해석하지 않는다.
- 따라서 이 작업은 단순 JSON 저장 수정이 아니라 `화면별 임시 boolean 상태 → 앱 전역 Progress 모델 → 음악별 순차 단계 정수 → 즉시 영속화 → lock/unlock UI 렌더링`으로 진행 상태의 책임과 데이터 계약을 정리한 작업으로 본다.

### HB-26 — 대용량 모델 파일의 Git LFS 관리와 AAB→universal APK 패키징 자동화

- 2024-08-02 `e452f2d`에서 기존에 저장소에서 통째로 ignore하던 `weights` 자산을 버전 관리 대상으로 전환했다. 작은 `nemo.cfg`는 일반 Git에 넣고, YOLO 가중치 `nemo_best.weights`는 Git LFS로 추적하도록 `.gitattributes`와 `manage_large_file.bat`을 추가했다.
- LFS pointer 기준 `nemo_best.weights`의 실제 크기는 `257,286,840 bytes`(약 257 MB)였다. 일반 Git blob으로 직접 관리하지 않고 LFS로 분리해 대형 모델 파일을 저장소 이력에 포함시키면서도 Git 객체가 불필요하게 비대해지는 것을 피했다.
- 같은 방식으로 APK 변환에 필요한 `bundletool-all-1.17.1.jar`도 Git LFS로 관리했다. 이 JAR의 LFS 대상 크기는 `32,456,876 bytes`(약 32.5 MB)였다.
- 프로젝트의 일반 앱과 시각장애 사용자용 앱 모두 Gradle에서 `assetPacks = [":musics", ":weights"]`를 사용하고, `musics`·`weights`는 `com.android.asset-pack`의 `install-time` delivery로 구성돼 있었다. 따라서 배포/테스트 산출물은 단순 APK보다 AAB + asset pack 구조와 연결돼 있었다.
- `extract_apk.bat`을 추가해 모듈명을 인자로 받아 release AAB를 `bundletool build-apks --mode=universal`로 `.apks` 파일로 변환하고, ZIP으로 풀어 나온 `universal.apk`를 `app.apk` 또는 `hummingblocks_for_blind_ones.apk`처럼 모듈명 기반 파일명으로 바꾸도록 자동화했다.
- 스크립트는 변환 후 중간 `.zip`과 `toc.pb`를 삭제해 최종 APK만 남겼고, 같은 한 개의 batch script를 `app`과 `hummingblocks_for_blind_ones` 두 모듈에 재사용하도록 만들었다.
- 즉 Android Studio/명령행에서 AAB를 만든 뒤 `bundletool 실행 → .apks 생성 → 압축 해제 → universal.apk 찾기 → 이름 변경 → 중간 파일 삭제`를 수동으로 반복하던 배포 준비 절차를 하나의 명령으로 줄인 작업으로 정리한다.
- 이 커밋에서 음악·이미지 asset pack 자체를 새로 설계한 것은 아니다. `musics`와 `weights` asset-pack 구조는 이미 존재했고, HB-26의 핵심 개인 작업은 **대용량 모델/빌드 도구의 버전 관리 방식 정리와 universal APK 추출 자동화**다.
- 바로 뒤 `bef51f4`는 release maintenance 성격의 인접 작업이다. 일반 앱의 `compileSdk/targetSdk`를 33→34, version을 `1.2.1(code 20)`→`1.3.0(code 22)`로 올리고 AppCompat, Material, CameraX, Play Asset Delivery와 테스트 라이브러리 버전을 함께 갱신했다. 이는 HB-26 자동화의 핵심 구현과는 분리하되 같은 배포 정비 구간의 작업으로 기록한다.
- 당시 batch script에는 로컬 release signing 설정을 직접 참조하는 부분이 있으나, 재사용 가능한 경험으로 정리할 때는 signing 비밀값이나 개인 PC 경로가 아니라 `release AAB를 bundletool로 서명된 universal APK로 변환하는 자동화`만 기술한다.

### HB-27 — 주요 화면을 360×740 기준 percentage layout으로 전면 UI 개편

- 2024-08-22 `e5c69ad`는 본격 개편 전의 UI sync 성격으로, Camera/QR 화면의 font 통일, 저장·삭제·제목 입력 다이얼로그 drawable/layout 교체, level indicator 리소스 변경 등을 반영했다.
- 2024-08-28 `65ee77b`에서는 `Loading`, `Main`, `Select`, `Play`, 코드 보기 화면과 여러 다이얼로그까지 대규모로 개편했다. 새 배경·버튼·아이콘·컨테이너 리소스를 추가하고 기존 화면 요소 명칭과 사용자 흐름도 함께 정리했다.
- 핵심 layout 방식은 화면 내부에 `layout_width=0dp`, `layout_height=0dp`, `layout_constraintDimensionRatio="360:740"`인 기준 콘텐츠 영역을 두고, 요소 배치를 `Guideline`의 percentage 값으로 정의하는 방식이었다. Main/Select/Play에서 동일한 360×740 좌표계를 반복 사용해 디자인 좌표를 Android constraint 비율로 옮겼다.
- 예를 들어 Main은 기존 `18:37` 기준 영역을 동일 비율의 명시적 `360:740`으로 정리하고 preview·헤드폰 안내·시작 버튼 등을 새 좌표에 맞춰 재배치했다. Select도 기존 다수의 오래된/주석 처리된 UI를 정리하고 음악 preview carousel, 이전/다음, 촬영 시작, 저장 음악, 설정 버튼을 새 percentage guideline 구조로 다시 배치했다.
- Play 화면도 `360:740` 기준 영역을 유지하면서 조건 표시, 재생/일시정지, 재시작, 코드 보기, 저장·설정 등 주요 조작 UI와 drawable을 새 디자인에 맞게 교체했다. 기존 `ResultViewActivity` 계열은 `ViewCodeActivity`/`activity_view_code`로 이름과 화면 구조가 정리됐다.
- 텍스트는 여러 화면에서 `android:autoSizeTextType="uniform"`과 constraint 영역을 결합해 고정 sp만으로 배치하지 않도록 했다. 일반 TextView와 달리 기본 auto-size가 직접 적용되기 어려운 `EditText`에는 `EditTextAutoSizeUtil`을 추가해 동일 크기의 invisible TextView로 계산한 textSize를 EditText에 반영했다.
- 즉 이 작업은 개별 버튼 margin을 조금씩 고치는 수준보다, 디자인 기준 좌표(360×740)를 percentage constraint로 변환해 여러 화면에 일관되게 적용하고 텍스트 크기도 컨테이너에 맞춰 조정하는 **UI 시스템 전면 교체**에 가깝다.
- 사용자 확인상 이 레이아웃 전략은 태블릿/Z Flip 같은 **특정 기기별 분기**를 두는 방식이 아니었다. 화면의 실제 크기에 맞춰 별도 `layout-sw...` 리소스를 만드는 대신, 기준 화면에서 계산한 위치·크기를 percentage `Guideline`으로 표현하고 각 View를 `0dp` constraint에 묶어 **화면 비율에 따라 컴포넌트 전체가 함께 늘고 줄도록** 구현한 방식이었다.
- 따라서 HB-27은 `특정 기기 대응`이 아니라 Figma 기준 좌표를 비율 좌표로 변환해 전체 화면을 상대 배치한 responsive layout 구현으로 정리한다.
- 새 UI의 원본 디자인은 디자이너가 Figma에 제작한 시안이었다. 재훈님은 해당 시안을 보고 각 요소의 위치·크기·비율을 계산해 Android의 `ConstraintLayout`, percentage `Guideline`, `0dp` constraint, auto-size text 구조로 옮겨 구현했다. 따라서 개인 기여는 `UI 시안 디자인`이 아니라 **Figma 시안을 Android 반응형 레이아웃으로 변환·구현한 작업**으로 표현한다.

### HB-30 — QR 위치 상태를 이용한 시각장애 사용자용 자동 촬영·방향 안내 UX 재설계

- 2024-10-12 `2c0cf19`는 별도 접근성 앱 `hummingblocks_for_blind_ones`의 Select→Camera→Play 흐름을 크게 다시 정리한 커밋이다. 제품·UX 방향은 팀 내부 논의 결과로 보고, 아래 Android 구현·통합을 개인 기여로 본다.
- 기존에는 `SelectActivity`가 `BarcodeActivity`를 열고, ML Kit가 한 프레임에서 barcode를 4개 이상 발견하면 자동 촬영하는 비교적 단순한 구조였다. 이 커밋에서 `BarcodeActivity`를 제거하고 `CameraActivity` 하나에 CameraX ImageAnalysis, ML Kit barcode scanning, 촬영, Python 분류, 결과 복구 흐름을 통합했다.
- 새 CameraActivity에는 수동 촬영 버튼이 없고 barcode analyzer에서만 `capture()`가 호출된다. 즉 사용자는 카메라를 블록 쪽으로 맞추기만 하면 되고, 촬영 시점은 앱이 자동으로 결정하도록 단순화했다.
- 단순히 barcode 개수만 세지 않고, QR의 display value가 `1~4`일 때 각각 bit로 누적해 `contain_barcode` bitmask를 만들었다. `7, 11, 13, 14, 15`이면 촬영하는데, 이는 기대 QR 1~4 중 **아무 3개 또는 4개가 모두 보이는 상태**에 해당한다. 기존 `barcodes.size() >= 4`보다 필요한 특정 QR의 가시성을 직접 확인하면서도 1개가 프레임 밖에 있어도 촬영 가능하게 완화한 구조다.
- 아직 촬영 조건이 안 되면 bitmask 패턴에 따라 이동 방향을 정했다. `1/4/5`는 오른쪽, `2/8/10`은 왼쪽, `12`는 위, `3`은 아래 이동 안내로 매핑했다. 코드상 핵심은 검출된 QR ID 조합을 **사용자가 카메라를 어느 방향으로 움직여야 하는지**로 변환한 것이다.
- 방향 상태는 한 소스에서 시각·음성 피드백으로 동시에 표현했다. 동일한 분기에서 arrow rotation을 `0/90/180/270°`로 바꾸고, 화면 문구를 `왼쪽/오른쪽/위/아래로 움직여 주세요`로 갱신하며, `AccessibilityEvent.TYPE_ANNOUNCEMENT`로 같은 의미의 TalkBack 안내를 전송했다.
- arrow는 `AnimatedVectorDrawable`로 움직임을 보여주고 안내 텍스트는 600ms 후 fade-out animation을 시작했다. `isShowingArrow`를 약 1.2초 동안 유지해 ImageAnalysis가 매 프레임 같은 방향을 검출하더라도 화살표 애니메이션과 TalkBack 안내가 과도하게 연속 재생되지 않도록 throttling했다.
- 방향을 특정할 수 없는 QR 조합에서는 별도 이동 안내를 강제로 만들지 않고 기본 문구 `모든 QR코드가 화면에 들어오도록 해주세요`로 복귀시켜 잘못된 방향 지시를 피했다. CameraActivity 시작 시에도 같은 기본 안내를 TalkBack announcement로 전달했다.
- 자동 촬영이 시작되면 `isProcessing` guard로 중복 촬영을 막고, 상단 loading layer를 표시한 뒤 `잠시만 기다려주세요`를 TalkBack으로 알렸다. Preview의 SurfaceProvider를 잠시 분리하고 `temp.jpg`에 저장한 뒤 기존 Chaquopy/Python classifier와 Runner로 결과를 판정했다.
- 인식 성공 시 PlayActivity로 진입한다. Error와 Warning은 별도 중복 코드를 없애 `show_dialog_result(image, message)` 하나로 통합하고, `제대로 연결하였나요?` 제목·오류/경고별 그림·구체 설명·확인 버튼을 갖는 결과 다이얼로그로 표시했다.
- 이전 BarcodeActivity에서는 Warning 확인 후 PlayActivity로 진행하는 코드가 있었지만, 새 구조에서는 Error/Warning 모두 다이얼로그가 닫히면 Preview SurfaceProvider를 복구하고 loading layer를 숨긴 뒤 `isProcessing=false`로 돌린다. 따라서 문제가 있는 인식 결과는 같은 촬영 화면에서 다시 맞춰 촬영하는 일관된 복구 흐름이 됐다.
- Camera layout도 실제 QR 위치를 맞추기 위한 4개의 QR corner guide, 중앙 crosshair, 방향 arrow, percentage Guideline 기반 안내 문구로 다시 구성했다. 시각 정보만 추가한 것이 아니라 TalkBack announcement와 같은 상태를 공유하도록 구현한 점이 핵심이다.
- 앱의 진입점은 계속 `SelectActivity`로 유지하고, 음악/장르를 고르면 새 CameraActivity로 이동하도록 변경했다. Select 화면도 360×740 Guideline 레이아웃, 큰 genre button grid, 명시적인 앱 종료 버튼으로 정리해 접근성 앱의 단순한 진입 흐름을 유지했다.
- PlayActivity는 일반 앱의 최신 재생 UI/로직 일부를 접근성 앱에 다시 동기화했다. 음악 라벨, Lottie 캐릭터, 진행 원, 조건 상태, 좌우 방향 표시, pause/resume animation 동기화, 재촬영·메인 복귀·다시 시작 흐름을 유지하면서 저장·퀘스트·복잡한 교체 UI 같은 일반 앱 기능은 중심 흐름에서 제외했다.
- 같은 커밋에는 일반 앱과 접근성 앱 `Classifier.py`의 OpenCV DNN 호출 방식 변경(`dnn_DetectionModel`, confidence 0.85→0.9 등)과 SDK/CameraX/ML Kit dependency 업데이트도 함께 들어 있다. 이는 자동 촬영·방향 안내 UX와 직접 동일한 작업은 아니므로 HB-30의 핵심 기여와 구분해 mixed-scope maintenance로 기록한다.
- 결과적으로 HB-30은 `QR ID 실시간 분석 → 촬영 가능 여부 판단 → 부족한 방향 추론 → 시각 화살표/문구 + TalkBack 동시 안내 → 자동 촬영 → CV 판정 → 오류·경고 시 촬영 상태 복구`를 하나의 폐루프로 만든 접근성 촬영 UX 구현으로 정리한다.
- 후속 `d32fc8e`(2024-10-15)는 저장 음악 작업이 아니라 접근성 앱의 Select/Play 디자인 보정과 `ViewCodeActivity` 추가 작업이다. HB-31 근거에서 제외하고 HB-30 계열 후속 UX 보완으로 분류한다.

### HB-31 — 저장 음악을 독립 `MusicFile` 모델로 통합하고 목록·재생·관리·MP3 내보내기를 재구성

- 2024-10-10 `4db36f8`에서 저장 음악 화면을 RecyclerView 기반 카드/목록 UI로 크게 개편하고, 장기적으로 저장 데이터를 한 객체에서 관리하기 위한 `MusicFile` 클래스를 처음 추가했다. 다만 이 시점의 실제 PlayActivity/SavedMusicActivity 주요 흐름은 아직 기존 `JSONArray`와 별도 `_block.json` 파일을 사용하고 있어 데이터 모델 전환이 완전히 끝난 상태는 아니었다.
- 기존 저장 구조는 장르별 디렉터리 아래 `<name>.json`에 마디별 `{level, replace}` 배열을 저장하고 `<name>_block.json`에 인식 block 데이터를 따로 저장하는 방식이었다. SavedMusicActivity도 특정 장르 디렉터리의 JSON들을 직접 열어 `int[][] level`, `boolean[][] replace` 배열로 다시 파싱했다.
- `4db36f8`의 초기 `MusicFile`은 제목·장르·생성일·마디별 level/replace를 필드로 캡슐화하고 `load/save`를 제공했다. 이 초기 버전에는 replace load 시 `replaces_json` 대신 `levels_json`을 읽는 실수가 있었고, 이후 `1f11d93`에서 저장 모델을 실제 적용하면서 함께 수정됐다.
- 2024-11-18 `1f11d93`에서 저장 포맷을 본격 전환했다. 저장 위치를 `files/music/<genre>/<name>.json + <name>_block.json`에서 `files/music/<name>.json` 한 파일로 통합하고, JSON 안에 `music_title`, `creation_date`, `genre`, `genre_label`, `levels`, `replaces`, `blocks`를 함께 저장하도록 했다.
- `creation_date`는 문자열 대신 epoch millisecond `long`으로 저장하고 필요할 때 `Date`로 복원했으며, 장르는 숫자 code와 label을 같이 보관했다. 저장 파일만 읽어도 어떤 MusicInfo와 연결해야 하는지, 제목/생성일/마디 상태/원본 block이 무엇인지 알 수 있는 self-contained 데이터가 됐다.
- PlayActivity도 더 이상 재생 중 만든 raw JSONArray와 별도 block 파일을 직접 쓰지 않고, 저장 시 `MusicFile`을 생성해 `addSections(levels, replaces, section_count)`, `setBlocks(runner.getRefinedBlockList())`, `setCreationDate()`를 채운 뒤 단일 JSON으로 저장하도록 바뀌었다. 동일 제목이 있으면 `(1)`, `(2)`처럼 suffix를 붙였다.
- SavedMusicActivity는 앱의 `files/music` 아래 모든 JSON을 `MusicFile` 객체로 로드하고, 각 파일의 `genre_label`을 `MusicInfo.getMusicCodeByName()`으로 다시 music code에 연결했다. 이를 전체 목록과 장르별 `music_files_genre`로 동시에 분류해 하나의 화면에서 전체 저장 음악 또는 특정 장르만 탭으로 필터링할 수 있게 했다.
- 서로 다른 장르의 저장 음악을 같은 목록에서 재생하기 위해 `MusicPlayer.setMusicInfo(MusicInfo)`를 추가했다. 사용자가 다른 장르의 곡을 선택하면 해당 곡의 MusicInfo로 measure length와 replace mapping을 교체한 뒤 같은 MusicPlayer 인스턴스로 재생한다.
- 저장 음악 재생은 MusicFile의 `levels`와 `replaces`를 순서대로 `setNextInstLevel()`·`setNextReplace()`에 공급한다. 각 마디가 시작될 때 `accumulated_time`과 index를 갱신하고, 20ms 주기의 UpdateProgress가 현재 카드의 progress bar와 `mm:ss` 시간을 갱신한다.
- 다른 저장 음악을 누르면 기존 재생을 stop하고 이전 카드의 detail을 닫은 뒤 새 카드만 펼친다. start/pause/resume/stop callback에서 재생/일시정지 아이콘, TalkBack용 contentDescription, progress와 현재 시간을 함께 갱신해 MusicPlayer 상태와 RecyclerView UI가 같은 상태를 보도록 했다.
- 각 카드에는 제목, 생성일, 장르, 전체 길이와 재생 진행 상태를 표시하고 삭제·제목 수정·외부 연동·MP3 내보내기 기능을 연결했다. 삭제 시 재생 중인 항목이면 먼저 stop하고 목록/장르별 목록에서도 함께 제거한다.
- 제목 수정은 파일명 금지 문자를 검사하고 중복 제목에 `(n)` suffix를 붙인 뒤 MusicFile의 title을 바꿔 새 JSON으로 저장한다. 제목에 종속되는 MP3도 새 제목 기준으로 다시 합성하도록 연결했다.
- `MusicFile.createJSON()`은 외부 그래피툰/QR 연동에 필요한 `genre`, `levels`, `blocks`만 뽑아낸 JSON을 만들도록 정리했다. 이전처럼 SavedMusicActivity에서 원본 파일과 `_block.json`을 다시 직접 열어 조합하지 않도록 데이터 책임을 MusicFile로 이동했다.
- MP3는 `AudioUtil.saveMp3FromMusicFile()`가 MusicFile의 마디별 level/replace 상태를 읽어 HB-20과 같은 FFmpeg `amix → concat` 파이프라인으로 재구성한다. 저장 시 생성할 수도 있고, 다운로드/공유 시 MP3가 없으면 on-demand로 다시 생성한다.
- MP3 다운로드는 Storage Access Framework의 `ACTION_CREATE_DOCUMENT`로 사용자가 저장 위치를 고르게 하고 생성된 MP3를 반환 URI에 copy한다. 공유는 `FileProvider` URI와 `audio/mpeg` MIME을 사용한다. 따라서 앱 내부 저장 파일과 사용자에게 내보내는 파일 경로를 분리했다.
- 이 작업은 `재생 중 임시 JSONArray + 별도 block 파일 + 장르별 직접 파일 탐색`에서 **도메인 객체 MusicFile → 단일 self-contained JSON → 장르 독립 목록/재생 → 관리·외부공유·MP3 파생물** 구조로 저장 음악 기능을 재설계한 작업으로 정리한다.

### HB-31S — 설정 상태 모델과 설정/진행도 초기화 UI 정리 (`4db36f8`의 독립 작업)

- 같은 `4db36f8`에는 SavedMusic과 별개로 SettingActivity/Setting의 재설계가 함께 들어 있다. 한 커밋에 섞였지만 개발 작업 단위는 별도로 본다.
- setting JSON key를 `volume_label → volume_level`, `vibration_control → enable_vibration`, `replace_button_view → show_replace_buttons`처럼 실제 의미가 드러나는 이름으로 정리하고 `setting_file_name`을 Setting이 직접 소유하도록 했다.
- SettingActivity는 5개 악기 SeekBar를 배열로 묶어 `Setting.volume_level`과 동기화하고, 진동 사용 여부와 교체 버튼 표시 여부를 switch에 연결했다. `syncWithSetting()`으로 처음 진입하거나 초기화한 뒤 UI를 한 번에 상태 객체와 맞추도록 했다.
- `Setting.reset()`을 추가해 5개 volume을 기본값 5로 되돌리고 vibration/replace 표시 기본값도 한 곳에서 정의했다. 설정 초기화 버튼은 개별 View를 직접 하나씩 변경하지 않고 `Setting.reset() → syncWithSetting()`을 호출한다.
- 별도의 `진행도 초기화` 버튼에서는 `Progress.reset()` 후 즉시 progress.json을 저장하도록 연결해 튜토리얼/퀘스트 진행 상태를 설정 화면에서 리셋할 수 있게 했다.
- Activity가 pause될 때 Setting 전체를 `setting.json`에 저장하고, SeekBar 변경은 배열의 대응 index에 바로 반영한다. 따라서 설정 UI와 영속 상태를 `Setting` 정적 모델 하나를 통해 동기화하는 구조로 정리됐다.
- UI 디자인/기능 기획은 팀 내부 논의와 디자인 산출물을 바탕으로 한 것으로 보고, 개인 기여는 상태 모델 정리·Android UI 연결·초기화/영속화 구현으로 표현한다.

### HB-32 — 규격화된 asset·metadata 계약으로 신규 음악 콘텐츠를 코드 수정 없이 확장

- HB-32의 핵심은 음악/캐릭터 원본을 직접 제작했다는 것이 아니라, 팀·외부 제작자가 만든 음원·이미지·Lottie를 앱이 기대하는 공통 asset 규격에 맞춰 통합하고 메타데이터를 등록해 **새 콘텐츠를 Java 코드 변경 없이 추가할 수 있게 운영한 작업**이다.
- 2024-05-26 `f924c6d`의 Cyberpunk 추가에서 일반 앱 Java 코드는 변경되지 않았다. `music/cyberpunk/` 아래 MP3, active/inactive 캐릭터 이미지, conductor 방향 이미지, background/stage/preview, 6개 Lottie와 `values.json`을 추가하고 전역 `music_info.json`, `genres.json`, `label.json`에 id·순번·한/영 label을 등록하는 것만으로 새 음악이 앱에 들어갔다.
- 당시 `MusicInfo`는 `music_info.json`의 `id/enabled/bpm_available`를 순회해 `music_list`를 만들고, 각 id를 기준으로 `music/<id>/values.json`, `image/...`, `lottie/...`, `preview.mp3`, 악기별 MP3 경로를 동적으로 조합했다. 따라서 `switch(genre)`나 장르별 Java 상수를 추가하지 않아도 디렉터리/파일명 계약만 맞으면 같은 UI·재생 로직을 재사용했다.
- Cyberpunk 추가 시 전역 metadata 기준 음악 ID는 pop, jazz, funk, dance, bossa_nova, rock, cyberpunk, airplane, little_star, birthday, arirang, carol의 12개였다. 이 커밋에서는 `genres.json`에 cyberpunk=12를, `label.json`에 `CYBERPUNK/사이버펑크`를 추가하고 앱 버전을 `1.2.0(code19) → 1.2.1(code20)`으로 올렸다.
- 2024-08-02 `35d7e18`은 Bossa Nova 음원 세트를 `mp3/<instrument>/<instrument><level>.mp3` 규격으로 갱신하고 `empty.mp3`가 필요하다는 점을 커밋 메시지에 명시한 콘텐츠 maintenance다. 소리가 없는 마디도 동일한 measure scheduler/FFmpeg 저장 파이프라인에서 길이를 유지하려면 empty asset이 필요했으므로 콘텐츠 패키지에도 빈 트랙이 계약의 일부였다.
- 이후 MusicInfo 구조는 BPM 3종을 제거하고 `values.json`의 단일 `bpm`, 선택적으로 `replace` mapping을 읽는 형태로 발전했다. `replace_available=true`인 음악은 기본 5개 악기 외 `replace0~2`의 MP3·이미지·Lottie·quest data를 추가로 갖고, false인 음악은 기본 자산만으로 동작하도록 같은 MusicInfo가 조건 분기했다.
- 2024-11-23 `eb8a47f`의 Classic 추가는 이 데이터 기반 구조가 실제로 유지된 사례다. 이 커밋도 Java 코드를 전혀 수정하지 않고 `music_info.json`, `genres.json`, `label.json`, `classic/values.json`과 규격화된 asset만 추가했다.
- Classic은 `replace_available=false`, `beats=24`, `bpm=150`으로 등록됐고, 19개 image asset, 6개 Lottie, 5개 악기×4단계의 20개 연주 MP3 + `empty.mp3` + `preview.mp3` 등 총 22개 MP3를 동일 디렉터리 규격으로 넣었다. metadata에서는 classic을 13번째 genre code로 추가하고 한/영 label `클래식/CLASSIC`을 등록했다.
- 바로 다음 날 `64a4afb`에서는 Classic의 brass 1~3단계와 guitar 1~3단계 MP3 6개를 교체했다. 코드 변경 없이 음원 자산만 갈아끼워 콘텐츠 품질을 수정할 수 있었음을 보여주는 maintenance 사례로 기록한다. 교체 사유나 음질 평가 방식은 코드/커밋만으로 확정하지 않는다.
- 2024-10 `4db36f8`에서는 여러 replace 지원 음악의 MP3 파일이 공통 `music/<id>/mp3/{drum,brass,guitar,piano,bass,replace0,replace1,replace2}` 구조로 대량 정리돼 있었다. 같은 시기 `file_structure.txt`에도 과거 `bpm0/1/2` 디렉터리를 단일 `mp3` 구조로 통합하는 migration 메모가 남아 있다. 이는 신규 콘텐츠 자체보다 **콘텐츠 디렉터리 계약을 단순화한 인프라 정리**로 본다.
- 이미지·캐릭터·Lottie·음원의 원본 제작은 디자이너/외부 제작자 등 팀 내외 역할이 섞여 있었고, 사용자 확인상 음원 원본은 외부 제작, 캐릭터/배경/stage 원본은 디자이너가 담당했다. 재훈님은 Android asset 구조 편입, metadata/path 연결, 필요한 inactive/grayscale variant 보완, 실제 화면·재생과의 통합을 담당한 것으로 표현한다.
- 따라서 HB-32는 `새 콘텐츠마다 코드 추가`가 아니라 **metadata 등록 + 공통 디렉터리 규격에 asset 배치 → 기존 MusicInfo/Select/Play/MusicPlayer가 자동 재사용**되는 구조를 실제 여러 장르 추가·교체 과정에서 운영한 경험으로 정리한다.

### HB-33 — 2.0.0 전환 시 기존 설치 데이터를 감지해 저장 음악·캐시를 새 구조로 1회 migration

- 2024-12-01 `36f8253`에서 앱 버전을 `1.3.0(versionCode 22) → 2.0.0(versionCode 25)`로 올리면서 `version.json`과 `VersionUtil`을 도입했다. 이 구현은 저장된 version code별 migration 체인을 순차 실행하는 범용 프레임워크라기보다, **v25 이전 설치를 한 번 식별해 새 저장 구조로 옮긴 뒤 version marker를 남기는 호환 레이어**에 가깝다.
- 시작 시 `version.json`이 있으면 저장 version을 읽어 로그만 남기고 별도 migration을 실행하지 않는다. `version.json`이 없고 `setting.json`이 있으면 기존 설치(`pre_v25`), 둘 다 없으면 신규 설치로 판정했다. 즉 실제 migration 분기는 저장 version 숫자가 아니라 `version.json`/`setting.json`의 존재 여부로 이루어졌다.
- 기존 설치에서는 먼저 `manage_music_file_v25()`로 과거 저장 음악을 새 HB-31 `MusicFile` 구조로 옮기고, 이전 weight cache를 삭제해 뒤의 Loading 단계에서 최신 asset의 `nemo_best.weights`와 `nemo.cfg`를 다시 복사하도록 했다. 완료 후 현재 version code를 `version.json`에 기록했다.
- 음악 migration은 모든 `MusicInfo`의 기존 장르별 내부 디렉터리를 순회하면서 과거 `bpm0/1/2` 캐시 폴더를 제거하고, 저장 JSON과 `_block.json`을 읽어 앱 공통 `files/music/<name>.json` 위치의 `MusicFile`로 다시 저장했다. 기존 장르 폴더에 남아 있던 파생 MP3도 삭제하고 새 MusicFile을 기준으로 MP3를 다시 합성했다.
- converter가 읽는 과거 음악 본문은 각 마디를 5개 악기 level 배열로 보고 `int[5]`로 옮겼으며, 새 포맷에 추가된 3개 replace 상태는 기본 `false`로 초기화했다. 생성일은 기존 JSON 파일의 `lastModified()`를 이용해 보존했다.
- 별도 `_block.json`이 있으면 block code를 하나의 `ArrayList<Integer>`로 평탄화해 MusicFile의 `blocks`에 저장한 뒤 기존 block 파일을 삭제했다. 이로써 `장르별 저장 JSON + 별도 block JSON + 별도 MP3`를 `전역 music 디렉터리의 self-contained MusicFile JSON + 재생성 가능한 MP3` 구조로 전환했다.
- 이 migration은 모든 역사적 개발 스키마를 자동 변환하는 로직은 아니다. 코드상 과거 음악 JSON의 각 section을 배열로 읽고 replace를 false로 초기화하므로, 당시 실제 배포 사용자 데이터의 구형 형식을 v25 포맷으로 올리는 일회성 변환에 초점이 맞춰져 있다. 저장 version 숫자별 `if (v < n)` 체인은 존재하지 않는다.
- 설정 데이터는 기존 key를 하나씩 새 key로 매핑하지 않고 pre-v25 경로에서 현재 `Setting` 기본 상태를 `setting.json`에 다시 저장한 뒤 로드한다. `progress.json`도 전용 migration 함수 없이 기존 `Progress.load()`를 시도하고 실패하면 현재 구조로 다시 저장하는 기존 fallback을 사용한다. 따라서 HB-33의 가장 강한 migration 근거는 저장 음악/asset cache 쪽이다.
- 최초 `36f8253` 구현에는 실제 migration 과정에서 문제가 있었다. `bpm0~2` 삭제 loop가 `j++`가 아니라 `i++`로 작성돼 있었고, 새 MusicFile title에 `.json` 확장자가 포함됐으며, `_block.json`을 1차원 배열로 가정했고, MP3 output path에도 `.mp3` 확장자가 빠져 있었다.
- 약 1시간 뒤 `6d66423`에서 이 문제들을 연속 수정했다. 삭제 loop를 정상화하고, title에는 확장자를 제거한 `fileName`을 넣었으며, block JSON의 중첩 배열을 이중 loop로 평탄화하고, AudioUtil 출력 경로에 `.mp3`를 붙였다. 또한 기존 설치 migration이 끝난 뒤 `version.json`을 실제로 저장하도록 흐름을 보완했다.
- 이 수정 이후 migration 순서는 `MusicInfo 초기화 → pre-v25 여부 판정 → 저장 음악 변환 → old weight cache 삭제 → version marker 저장 → Setting/Progress load fallback → 최신 weight asset 재복사`가 된다.
- 2024-12-04 `fbd31c9`의 커밋 메시지는 `version modulize`이지만 **새 Gradle module/package를 만든 것은 아니다.** LoadingActivity에 직접 있던 version JSON 읽기/쓰기 코드를 `VersionUtil.read_version_file()`과 `save_version()`으로 추출해 역할을 분리한 리팩터링이다.
- `VersionUtil.read_version_file()`은 파싱 실패 시 `-1`을 반환하고, `save_version()`은 PackageManager에서 현재 versionCode를 읽어 `{ "version": ... }` 형태로 저장한다. 다만 LoadingActivity는 이미 version file이 존재하는 경우 이 값으로 추가 migration 여부를 판단하지 않고 로그만 남긴다.
- 따라서 HB-33은 **대규모 저장 구조 변경이 기존 설치 데이터를 바로 깨뜨리지 않도록 pre-v25 사용자 데이터를 새 MusicFile 구조로 실제 변환하고, 변환 중 발견된 데이터 형태/파일명/출력 문제를 수정한 호환 작업**으로 정리한다. `앱 버전별 migration 모듈`이나 `별도 migration module`이라고 과장하지 않는다.

### HB-34 — 재생 화면 버튼을 개별 좌표에서 공통 percentage Guideline 그룹으로 재정렬

- 2024-12-02 `bfc794e`, 2024-12-04 `3c5b4e2`는 일반 앱 `activity_play.xml`과 튜토리얼의 동일 UI를 복제한 `overlay_tutorial_description.xml`에서 재생 화면 조작 버튼의 크기·배치를 다시 정리한 작업이다.
- 사용자 확인상 이 계열의 반응형 대응은 특정 화면 크기나 기기 모델별 resource를 분기하는 방식이 아니라, 기준 화면 좌표를 percentage Guideline으로 바꾸고 각 컴포넌트를 해당 Guideline 사이의 `0dp` constraint로 묶어 화면 비율 변화에 따라 함께 확대·축소되도록 하는 방식이었다.
- 기존에는 `btn_view_code_left/right`, `btn_retake_photo_left/right`, `btn_back_to_main_left/right`, `btn_setting_left/right`처럼 버튼마다 별도 Guideline이 흩어져 있었다. 후속 수정에서는 이를 `buttons_left_1/right_1`, `buttons_left_2/right_2`, `buttons_left_3/right_3`, `buttons_top/bottom_*`처럼 **행·열 단위 공통 기준선**으로 재구성했다.
- View Code와 다시 촬영 버튼은 첫 번째 버튼 행의 공통 좌우 Guideline을 사용하고, Main 복귀와 설정 버튼은 두 번째 행의 같은 좌우 기준을 재사용하도록 바뀌었다. Restart/Save 역시 세 번째 공통 좌우 Guideline에 묶였다.
- 버튼 label도 개별 좌우 경계선 대신 `buttons_text_left/right`, `btn_play_left/right`, `buttons_text_top/bottom_*`를 공유하도록 바뀌어 버튼과 텍스트가 같은 비율 좌표계 안에서 함께 움직이도록 했다.
- `3c5b4e2`에서는 Main 복귀/설정 버튼을 기존 약 40×40 기준에서 50×50 기준으로 키우고, Y 범위를 `0.055~0.123`, 좌우 범위를 각각 약 `0.119~0.258`, `0.742~0.881`로 맞췄다. 텍스트 영역도 좌우 약 `0.037~0.963` 안에서 양쪽 버튼 그룹에 맞춰 재배치했다.
- 같은 변경을 tutorial overlay에도 거의 동일하게 반영했다. 튜토리얼이 실제 Play UI 위치를 가정해 설명을 덮어씌우는 구조였기 때문에, 본 화면과 overlay의 Guideline을 함께 수정해야 안내 위치가 어긋나지 않았다.
- 코드상 `activity_play.xml`과 tutorial overlay 모두 360×740 기준 ratio와 다수의 percentage Guideline을 유지하며, 별도의 tablet/Z Flip 전용 layout resource나 runtime device 분기 없이 같은 상대 배치 체계를 사용한다.
- 따라서 HB-34는 `특정 기기 대응`보다는 **재생 화면의 버튼·텍스트를 공통 상대 좌표 체계로 묶어, 화면 비율 변화에도 전체 조작 UI의 크기와 간격 관계가 함께 유지되도록 보정한 작업**으로 정리한다.

### HB-35 — 팀원 녹화 prototype을 저장 음악 기반 분할 CameraX 녹화 기능으로 제품화

- 최초 영상 녹화 prototype은 2024-11-29 팀원 `Tinto-Verano`의 `83d51f6`이다. 당시 `RecordActivity`는 CameraX `VideoCapture<Recorder>`로 `temporary.mp4` 하나를 녹화하고, 테스트용 `R.raw.summer`를 MediaPlayer로 동시에 재생한 뒤 별도 audio/video 합성 함수를 호출하는 실험 단계였다. 따라서 영상 녹화 아이디어와 최초 prototype 자체를 재훈님 개인 구현으로 주장하지 않는다.
- 2024-12-30 재훈님의 `d85faeb`(`Update 2.2.0 - recording video`)에서 RecordActivity를 실제 제품 저장 음악 흐름에 맞게 크게 다시 작성했다. 앱 버전은 `2.1.0(code27) → 2.2.0(code28)`으로 올리고 CameraX `camera-video` dependency를 명시적으로 추가했다.
- SelectActivity의 `영상 촬영` 진입은 바로 RecordActivity를 여는 대신 `SelectMusicActivity`로 연결됐다. 사용자는 기존 HB-31 `MusicFile` 저장 목록에서 곡을 선택·미리듣기하고, 선택 완료 시 `MUSIC_TITLE` extra로 RecordActivity에 전달한다. 필요한 MP3가 없으면 `FFmpegUtil.saveMp3FromMusicFile()`로 먼저 생성한다.
- RecordActivity는 전달받은 title로 `files/music/<title>.json`을 `MusicFile`로 로드하고, `files/music/<title>.mp3`를 MediaPlayer로 연다. 즉 고정 테스트 음원이 아니라 **사용자가 블록으로 만든 저장 음악을 영상 촬영의 타임라인/배경음 기준으로 사용**하도록 바뀌었다.
- MusicFile의 section count만큼 진행률 divider를 동적으로 생성하고, MediaPlayer 진행 위치를 20ms 주기로 읽어 progress bar를 갱신했다. 재생 곡의 마디 구조와 영상 촬영 진행을 같은 화면에서 확인하도록 한 UI다.
- CameraX는 `Preview + VideoCapture<Recorder>`를 lifecycle에 bind하고 `Quality.HIGHEST`를 선택했다. 전/후면 전환 시 `cameraFacing`을 바꿔 `unbindAll() → bindToLifecycle()`로 다시 구성했다.
- 녹화 상태는 `Recording recording`, `video_count`, `currentVideoFile`로 관리했다. `recording == null`이면 시작, 아니면 정지라는 명확한 state로 record button을 토글하고, 녹화 중에는 전/후면 전환 버튼을 숨겨 카메라 재바인딩과 active recording이 충돌하지 않도록 했다.
- 첫 녹화를 시작할 때 output base name을 timestamp로 정하고 finish 버튼을 표시한다. 각 녹화 구간은 `temp1.mp4`, `temp2.mp4`, ...처럼 앱 내부 `files/videos`에 저장되므로 사용자가 중간에 stop한 뒤 다시 record를 눌러 **여러 segment로 이어서 촬영**할 수 있다.
- `VideoRecordEvent.Start`에서 저장 음악 MediaPlayer를 시작하고 progress scheduler를 돌리며 record icon을 stop 상태로 바꾼다. `stopRecording()`에서는 Recording을 close하고 음악을 pause하므로, 다음 녹화를 시작하면 같은 MediaPlayer의 현재 위치에서 이어 재생된다. 영상 segment의 분할 지점과 저장 음악의 pause/resume 지점을 맞추는 구조다.
- MediaPlayer가 곡 끝까지 재생되면 `OnCompletionListener`가 `finishRecording()`을 호출해 자동 종료한다. 사용자가 별도 finish 버튼을 눌러 중간에 최종화를 시작할 수도 있다.
- 녹화 finalize event에 오류가 있으면 현재 녹화를 정리하고 오류 Toast를 띄우며 생성된 `temp1...tempN.mp4`를 삭제한다. 정상 finalize에서는 segment 번호를 유지해 이후 합성 단계에서 순서대로 사용할 수 있게 한다.
- 마이크 음성은 녹음하지 않는다. 기존 prototype에 남아 있던 `RECORD_AUDIO`/외부 storage permission을 manifest에서 제거했고 `prepareRecording(...).withAudioEnabled()`도 사용하지 않는다. 영상은 무음 segment로 녹화하고 저장 음악 MP3는 이후 HB-36에서 별도로 합성한다.
- lifecycle에서도 `onPause()` 시 active Recording이 있으면 stop하고, `onDestroy()`에서 MediaPlayer를 release한다. 이후 `719732b`에서는 뒤로가기를 눌렀을 때 이미 녹화한 segment가 있으면 즉시 화면을 닫지 않고 `녹화 중단` 확인 다이얼로그를 띄우고, Activity 종료 시 남은 temp segment를 삭제하도록 정리해 abandoned recording 파일을 남기지 않게 보완했다.
- `719732b`에서는 finish 시 MediaPlayer를 `stop→prepare`하지 않고 `pause→seekTo(0)`으로 정리하고, 합성 중 top loading layer를 표시하는 등 최종화 중 UI state도 보완했다. 이 중 **임시 segment/녹화 세션 상태 관리**는 HB-35 후속 보완으로, 실제 concat/audio merge는 HB-36으로 분리한다.
- 결과적으로 HB-35는 `저장 음악 선택 → MusicFile/MP3 로드 → CameraX VideoCapture → 음악과 동기화된 start/pause → tempN 분할 녹화 → 오류·이탈 시 segment 정리`를 만든 작업이다. 최초 prototype 작성은 팀원, 재훈님 기여는 이 prototype을 실제 앱 데이터·UI·상태 흐름과 연결해 제품 기능으로 재구성한 부분으로 표현한다.

### HB-36 — 분할 CameraX 영상과 저장 음악 MP3를 FFmpeg로 합성해 갤러리 영상으로 생성

- HB-35에서 만들어진 `temp1.mp4`, `temp2.mp4`, ... segment와 저장 `MusicFile`에서 생성한 MP3를 최종 영상으로 만드는 후처리 파이프라인이다. 최초 단일 녹화 prototype의 audio/video 합성 실험은 팀원 코드에 있었지만, `d85faeb`에서 재훈님이 분할 녹화·저장 음악 구조에 맞춰 `FFmpegUtil`과 `RecordActivity.finishRecording()`을 다시 구성했다.
- `mergeTempVideoFiles()`는 `video_count`만큼의 `tempN.mp4` 절대 경로를 FFmpeg concat demuxer용 `file_list.txt`에 순서대로 기록하고, 앱 내부 `videos` 디렉터리로 옮긴 뒤 `concat.mp4`를 생성한다.
- segment 연결 명령은 `-f concat -safe 0 -i file_list.txt -c copy concat.mp4`이다. 영상 stream을 재인코딩하지 않고 그대로 이어 붙이므로, CameraX에서 같은 녹화 설정으로 만들어진 segment를 빠르게 하나의 연속 영상으로 조합하는 방식이다.
- HB-35에서 record stop 시 저장 음악 MediaPlayer도 pause되고, 다시 녹화할 때 같은 위치에서 resume한다. 따라서 사용자가 촬영을 멈춘 동안의 시간은 MP3 재생 진행에도, 최종 concat video에도 포함되지 않는다. 여러 segment를 공백 없이 연결하면 실제 음악을 들으며 촬영한 구간의 시간축이 다시 이어진다.
- concat 완료 후 `merge_video_and_audio()`가 `concat.mp4`와 `files/music/<title>.mp3`를 입력으로 받아 최종 MP4를 만든다. FFmpeg 옵션은 `-c:v copy -c:a aac -shortest`로, 영상은 다시 인코딩하지 않고 복사하고 MP3 audio만 AAC로 넣는다.
- `-shortest` 때문에 사용자가 저장 음악 전체를 촬영하지 않고 중간에 `완료`를 눌렀다면 최종 출력은 더 짧은 video stream 길이에 맞춰 종료된다. 반대로 음악이 끝까지 재생되면 HB-35의 completion callback이 finishRecording을 호출하므로 전체 곡 길이 기준 영상이 만들어진다.
- RecordActivity는 합성 결과를 앱 내부 `files/videos/<timestamp>.mp4`에 먼저 만든 뒤 Android `MediaStore.Video`에 `DISPLAY_NAME=<timestamp>.mp4`, MIME `video/mp4`, `RELATIVE_PATH=DCIM/HummingBlocks`로 entry를 생성한다. ContentResolver output stream으로 내부 결과 파일을 복사해 사용자가 일반 갤러리에서 확인할 수 있게 했다.
- 갤러리 복사가 끝나면 `temp1...tempN.mp4`, `file_list.txt`, `concat.mp4`, 앱 내부 최종 `<timestamp>.mp4`를 모두 삭제하고 `video_count=0`으로 초기화한다. 즉 앱 내부 videos 폴더는 작업용 임시 공간으로 쓰고 사용자가 보관하는 결과물은 MediaStore 쪽만 남긴다.
- `d85faeb` 당시 MusicFile→MP3 생성 함수 `saveMp3FromMusicFile()`은 내부 `merge_and_concat()`을 비동기 `FFmpeg.executeAsync()`로 실행했다. 그런데 SelectMusicActivity 등 호출부는 MP3 생성을 요청한 직후 그 파일을 MediaPlayer/RecordActivity에서 사용한다.
- 2025-01-07 `719732b`에서는 이 MP3 합성도 동기 `FFmpeg.execute()`로 변경하고 async wrapper를 제거했다. 코드상 효과는 **MP3 합성이 완료된 뒤 호출부가 다음 단계로 진행하도록 실행 순서를 보장**하는 것이다. 직접적인 버그 리포트는 확인되지 않으므로 원인을 단정하지 않고, 영상 제작처럼 출력 파일을 즉시 소비하는 흐름에 맞춘 순서 안정화로 기록한다.
- `719732b`에서 finish 시 합성 작업 동안 `topLayer`를 표시하고 완료 후 숨기도록 UI state를 추가했으며, 내부 결과를 MediaStore로 복사할 때도 공통 `Utils.copyFile()`을 사용하도록 정리했다.
- FFmpeg concat/audio merge 자체는 동기 호출이라 `finishRecording()`의 `mergeTempVideoFiles() → merge_video_and_audio() → MediaStore copy → cleanup`이 순서대로 실행된다. 다만 CameraX recording finalize 직후 파일 flush를 기다리는 명시적 callback chain 대신 500ms `Handler.postDelayed()` 뒤 합성을 시작하는 구현이므로, 이를 더 강한 완료 보장 구조로 과장하지 않는다.
- 결과적으로 HB-36은 `분할 무음 영상 → concat demuxer로 무재인코딩 결합 → 저장 음악 MP3를 AAC audio로 mux → -shortest 길이 정렬 → MediaStore/DCIM 저장 → 임시 파일 정리`의 로컬 뮤직비디오 생성 파이프라인을 구현한 작업으로 정리한다.

## 3. 별도로 다시 확인할 후속 작업

- **BPM 기능 제거 사유:** 구현 완료와 이후 제거 사실은 확인됐지만 제품 판단 이유는 미확인으로 유지한다.
- **TFLite 실험:** 속도는 여러 Galaxy S9+에서 두 버전을 기기별 1회씩 실행해 비교한 조건까지 확인됐다. `약 90% → 57%` 정확도 수치의 평가 데이터셋·샘플 수·집계 방식은 여전히 미확인이다.

## 4. 다음 진행 위치

다음 작업은 **HB-37 — 저장 음악·영상 제작 UX의 최종 통합 보완**이다.

`719732b`, `a2f2037`, `7a1cb0f`를 중심으로 다음을 확인한다.

- SavedMusicActivity와 SelectMusicActivity의 역할이 최종적으로 어떻게 나뉘었는지
- 저장 음악 재생·선택·삭제·제목 변경 UI가 어떤 상태 모델로 정리됐는지
- 저장 음악에서 바로 영상 제작으로 진입하는 경로가 추가됐는지
- MP3/영상 popup, 삭제·제목 변경 dialog와 accessibility 보완 범위
- `a2f2037`, `7a1cb0f`가 실제 기능 변경인지 UI 위치/튜토리얼 후속 fix인지 분리
- HB-31/35/36과 중복되지 않는 독립 작업 단위를 최종 확정

HB-36 확정:
- `tempN.mp4` 목록을 concat demuxer input list로 만들어 `-c copy`로 하나의 `concat.mp4` 생성
- 저장 음악 MP3를 `-c:v copy -c:a aac -shortest`로 영상에 mux
- 녹화 pause와 음악 pause가 함께 움직여 segment concat 후 연속 시간축 복원
- 중간 완료 시 `-shortest`로 영상 길이에 맞춰 audio 종료
- 최종 MP4를 MediaStore `DCIM/HummingBlocks`에 저장
- MediaStore 저장 후 temp/list/concat/internal output을 정리
- `719732b`에서 MusicFile→MP3 FFmpeg 실행을 async→sync로 바꿔 출력 파일 소비 순서를 안정화
- CameraX finalize와 FFmpeg 사이에는 500ms delay를 사용했으므로 callback 기반 완료 보장으로 과장하지 않음

남은 미확인:
- HB-17~18의 `약 90% → 57%` 정확도 평가 데이터셋·샘플 수·집계 방식
- 속도 비교의 원본 측정표/로그가 남아 있는지 여부
- `2c0cf19`에 함께 포함된 Classifier.py 리팩터링의 직접적인 문제/성능 개선 목적

---

`main`은 수정하지 않았다. 체크포인트와 기존 전체 재작성은 Draft PR 브랜치에서만 관리하며, 작업 단위 검증이 완료되기 전에는 merge하지 않는다.
