# webOS Smart Planter — 커밋 기반 개발작업 원장

> 조사 시작: 2026-10-27 · 원본: [dudgns128/webos-gardening](https://github.com/dudgns128/webos-gardening) · 상태: **9개 브랜치 커밋 graph 역검증 완료 / 본인 일반 커밋 26개 diff 1차 검토 완료 / 결과·의사결정 검증은 일부 미완료**. 이 파일은 이력서 확정본이 아니다. `resume.md`는 미변경이며, `experiences/webos-smart-planter.md`는 본 조사 결과를 바탕으로 **canonical v1**으로 갱신했다. 이 원장은 그 문서의 세부 근거 레이어다.

## 1. 조사 범위와 분류 원칙

- 확인한 브랜치 **9개**: `Camera`, `Hardware`, `Server/dev`, `WebPage/dev`, `Webos/dev`, `Webos/devHW`, `gh-pages`, `main`, `test`. 각 브랜치 커밋 목록을 끝까지 페이지 단위 조회하고 **브랜치 간 중복 SHA 제거**.
- GitHub 계정 `nanocode00`을 author로 갖는 **고유 커밋 31개**, 그중 **일반 커밋 26개(부모 0개인 gh-pages 시작 커밋 포함)** 및 **병합 커밋 5개**. 브랜치별 동일 커밋을 중복 계산하지 않음.
- **작업 ID는 커밋 ID가 아니라 문제 해결 단위**. 아래 모든 26개 일반 커밋은 잠정 그룹에 매핑했고, 향후 각 diff를 읽으며 그룹을 분리·통합할 예정. 현재 임시 그룹명을 최종 경험 또는 개인 저작 결론으로 삼지 않음.
- 기술적 근거 우선순위: **diff → 이후 최종 코드와 테스트·로그 → 당시 문서 → 사용자 직접 기억**. 커밋 메시지만으로 완성/테스트 성공 판정 금지.
- 사용한 주요 자료: [하드웨어 브랜치](https://github.com/dudgns128/webos-gardening/tree/Hardware), [HW 통합 브랜치](https://github.com/dudgns128/webos-gardening/tree/Webos/devHW), 저장소 README, 2024년 당시 Google Drive `webOS 하드웨어 정리.docx`, 산학협력 수행계획서(2024-03-01~06-30, LG전자 협력, 학부 5인), **초기 제안 발표 슬라이드**. 해당 슬라이드는 목표·계획을 적은 자료로 최종 구현/시연 결과를 증명하지 않는다. 팀 README의 영상 링크는 확인했으나 영상을 직접 검수하지 못했다.

## 2. 본인 일반 커밋 26개 — 누락 방지 인벤토리

| 잠정 작업군 | 해당 author 커밋(앞 12자리 SHA) | 단계 / 조사 범위 |
|---|---|---|
| **HW-01 Arduino/I2C 센서·제어** | `a94a8fc01c9f`, `00e4febe4ae0` | **첫 묶음 1차 검토 완료**. 펌웨어 및 webOS I2C 독립 테스트 앱. 결과 확인의 한계는 아래 |
| **WEB-01A React 화면의 WebSocket 통합 초안** | `126397712f02`, `08960758c838`, `8725eedb1078`, `ec3d2ce378b0` | **4커밋 diff·후속 코드 조사 완료**. 브리지 임시 비활성화 → WebSocketUtil 적용 → 제어명령 보정 → URL·응답 스키마 수정. 당시 완성/무결점 주장 금지 |
| **WEB-01B React 수신 콜백·데이터 갱신** | `3bd71c54270e`, `cd97e39ace46`, `1377108fb518`, `0ec37f08e857` | **4커밋 diff 검토**. 6/11 `ws` 외부 패키지 `node_modules` 반입(자체 개발 아님), 6/12 폴링→수신 콜백, 식물 목록 캐시 처리 및 자동제어 응답 처리 초안. 오류 가능성·후속 수정 함께 기록 |
| **WEB-01C 모달 동기화 수정·표시·배포 후속** | `6c4283ff31f7`, `b257d57062dd`, `5946539cf3fa`, `a543715ba059` | **4커밋 diff 검토**. 레이아웃/이미지 크기, 자동제어 토글 전송값·콜백 버그 수정, 10/31 웹 앱 배포 상대 경로 설정. 전체 화면 품질 검증과 분리 |
| **SVC-01A JS Service I2C 실센서·구동 통합** | `f2e890aab82d`, `6fc7336eae66`, `ea08aa6a93d6` | **3커밋 diff 확인**. dummy → I2C 실데이터, 서비스 API 응답·비동기 수정, 제어 호출 잠시 분리·재개. 실제 장애 원인/E2E 미확정 |
| **SVC-01B JS Service WebSocket 연결·파싱** | `1275599ddfce`, `6783a5de18a7` | **2커밋 diff 확인**. `ws` 의존성·서버 주소 설정, 수신 JSON 파싱 및 메서드 값 정합성 수정. 서버 전체는 팀 작업 |
| **CLEAN-01 JS Service DB 파일 정리** | `d799833ca911` | **1커밋 diff 확인**. `plantInfo.js` 단일 파일 삭제. 다른 DB 모듈의 최초 작성이라고 주장하지 않음 |
| **CAM-01 webOS Camera API 후속 실험** | `2cbed98d4d47`, `0bc6ba58f073`, `991d8843aa7b` | **3커밋 diff 확인**. 9월 예제 성격의 카메라 테스트 웹앱, 10월 camera2/미디어 API 명령 실험 메모. 실제 촬영·서비스 채택 미확정 |
| **OPS-01 운영·정리·rollback/빌드** | `ec8f0e20fb48`, `5fa0831b9bac`, `cec22e9efdba` | **3커밋 diff 확인**. gitignore 추가, 타인 UI 변경 커밋 전체 revert, gh-pages 독립 root 커밋에 생성된 React 정적 빌드 아티팩트. 실제 서비스 배포 성공 미검증 |

**개인 author 병합 커밋 5개(개발작업으로 중복 산정하지 않음):** `4d357d94eeb4`, `0319b2d366ad`, `aa319bd64295`, `7be82da2905a`, `d51bffe6ae27`.

**전체 graph 역검증(2026-10-27):** 총 9개 브랜치의 모든 페이지 재조회(Camara 8, Hardware 7, Server/dev 43, WebPage/dev 117, Webos/dev 142, Webos/devHW 159, gh-pages 1, main 204, test 131커밋; 브랜치별 집계는 SHA 중복 있음). **저장소 고유 커밋 313개**, GitHub `nanocode00` author **31개(부모 수 0 또는 1의 일반 커밋 26개, 다중 부모 merge 5개)**로 확인. 앞선 1차 조사의 295개는 미완료 페이징에 따른 과소집계였으며 **313개로 정정**. 현재 원장의 일반 커밋 목록 26개에 대한 **누락 SHA 0개**. `Server/dev`에서는 조회된 커밋의 `nanocode00` author 기록 없음; 이를 사용자의 모든 서버 관련 활동 부재로 확대하지 않음. 본인 외 author의 커밋도 저자 귀속 없이 팀 기여·후속 변경으로 구분. 2024-09~10 후속 커밋은 산학협력 수행계획서의 3~6월 기간과 별개로 표시.

## 3. HW-01 — Arduino/I2C 센서·제어 펌웨어 + 독립 통신 테스트

**검토 대상:** 본인 [`a94a8fc01c9f` (2024-06-02)](https://github.com/dudgns128/webos-gardening/commit/a94a8fc01c9f), [`00e4febe4ae0` (같은 날)](https://github.com/dudgns128/webos-gardening/commit/00e4febe4ae0). 비교 baseline은 `a94a8fc01c9f` 부모 `eee596f70e70`(해당 Arduino 파일 없음). **2024-10-09 타인 후속 변경은 별도 소유권으로 구분**.

### 상황

2024년 5인 산학협력 스마트 홈 가드닝 팀의 **webOS OSE(Raspberry Pi 4) ↔ Arduino 센서/액추에이터** 통합이 필요했다. 당시 하드웨어 메모에는 Raspberry Pi 4의 webOS OSE 2.24.0, Arduino Uno R3의 SDA/SCL 연결, 24-LED NeoPixel(핀 3), 펌프(핀 4), DHT11(핀 2), CDS(A0), 수위(A1), 토양습도(A2)가 적혀 있다. 최종 저장소 README에는 Arduino Nano로 설명돼 있으나 **Uno 개발 단계와 Nano 구성을 실제로 언제 전환했는지 미확정**.

### 문제

- 센서 데이터를 webOS 쪽에서 구조적으로 읽고, 서비스의 조명/급수 요청을 Arduino로 내려보낼 통신 방식·데이터 포맷이 필요함(프로젝트 목표와 새 파일로 확인).
- **초기 단계의 구체적 장애 현상/전송 방식 비교·선정 근거는 아직 미확인.** '기존 I2C가 실패해서 새 프로토콜로 고쳤다' 같은 서술은 자료로 뒷받침되지 않음.
- 2024년 하드웨어 문서에는 **토양습도센서가 없어 A2를 GND에 연결해 사용**했다고 명시. 초기 단계 4종 센서 모두를 실측한 것으로 쓸 수 없음.

### 판단

- `Wire.begin(1)`로 Arduino를 I2C slave address **1**에 두고 `Wire.onReceive`/`Wire.onRequest`로 **명령 수신과 값 요청을 분리**. 명령 모드 `0`은 NeoPixel 밝기, `1`은 펌프, `2`는 센서 측정.
- 센서 데이터는 **10바이트 고정형**: DHT11 원시 습도/온도 각 2바이트(총 4) + CDS 2 + 수위 2 + 토양습도 2. 아날로그 값 상·하위 바이트는 Arduino 소스에서 `>>8` / `&0xFF`로 분할.
- 단, **왜 다른 인터페이스보다 I2C를 선택했는지**, 모드 구분/10바이트 포맷의 제약·의사결정은 당시 명시적 문서가 없어 *판단 근거 미확정*. `dataIO.txt`에는 재결합에 `*128`이 적혀 있으나 소스 쪽 분할은 8비트이므로 **문서와 구현 간 불일치**를 확인할 필요가 있음.

### 조치 — 본인 커밋의 실제 코드

1. **`a94a8fc01c9f`**: `arduino/i2cSlaveFinal/i2cSlaveFinal.ino`·센서 및 I2C 테스트 스케치 추가. 밝기 0~100을 0~255로 변환해 NeoPixel 24개에 적용하는 `controlNeopixel`, 펌프 핀을 HIGH/LOW로 제어하는 `controlPump`, `readSensors`·`processAnalogData` 및 10바이트 `onRequest` 전송 구현.
2. 동일 커밋: `i2cTestMaster`/`i2cTestSlave`로 10바이트 왕복 테스트, `i2cSensorTestSlave`·`timeTest` 별도 스케치, `i2cTest/index.html` webOS I2C 독립 테스트 UI, `dataIO.txt` 포맷·`i2clog.txt` 명령 결과 자료 추가.
3. 동일 커밋: `arduino/DHT.cpp`, `DHT.h` 포함. 헤더에 **Adafruit Industries 원작 및 MIT 라이선스**가 명시되어 있으므로 라이브러리 전체를 직접 작성했다고 주장하지 않음. **2026-10-27 사용자 직접 확인(C):** DHT 센서에서 변환된 값이 아니라 **원시 바이트값 자체가 필요했기 때문에 Adafruit DHT 라이브러리에 `readRawBytes(uint8_t[])`를 직접 추가**했다. 함수는 DHT 센서 읽기 성공 시 내부 `data[0..3]`을 외부 버퍼로 복사하고 실패 시 0을 기록한다(A: 해당 함수 본문 확인). **기존 Adafruit 라이브러리 전체의 저작자는 아니며, 원시 바이트 노출을 위한 확장 부분이 개인 기여**다. 원본 라이브러리와의 줄 단위 전체 diff는 아직 비교하지 않았다.
4. **`00e4febe4ae0`**: Arduino 펌웨어를 새로 수정한 커밋이 **아니며**, 주로 `i2cTest/index.html`의 지연 호출·callback 방식과 데이터 표시 문자열을 수정. 테스트 IPK와 참고 링크도 갱신. 두 커밋을 모두 '펌웨어 구현'이라고 합치지 않음.

### 결과와 이후 유지 여부

- **I2C 통신 실측(A):** [`i2clog.txt`](https://github.com/dudgns128/webos-gardening/blob/Hardware/i2clog.txt)에서 Raspberry Pi의 `luna-send`로 `i2c/open` → 명령 `[0,2]` `write` → `size:10` `read` → `close` 응답 `returnValue:true`와 10바이트 배열 확인. 여러 번 읽은 결과 일부 바이트가 달라져 실제 버스 왕복 및 응답이 있었음. **전 센서 정확도/동작 안정성/펌프·LED 실물 작동까지 이 로그만으로 보장되지는 않음.**
- **후속 타인 수정 구분(A):** 2024-10-09 [`a6aa3a5657e5`](https://github.com/dudgns128/webos-gardening/commit/a6aa3a5657e5) (author `HONG-2019110129`)가 `Wire.onReceive`에서 즉시 `readSensors()` 하는 형태를 **`reading=true` 플래그 설정 → `loop()`에서 센서 읽기**로 변경. [`4b52723da0f8`](https://github.com/dudgns128/webos-gardening/commit/4b52723da0f8) (동일 타인 author)가 `dht.begin()` 호출·debug 정의를 추가. **사용자 최초 구현에 이 후속 안정화 작업을 귀속하지 않는다.**
- **증빙 경계:** 초기 하드웨어 메모의 A2/GND 대체 설정을 인정하므로 '4종 센서 전체 실측', '실시간 통신 품질 보장', '펌프/NeoPixel HW E2E 완전 검증'은 이 묶음에서 아직 확정하지 않음. 위 팀원 수정의 *실제 버그 원인*은 diff만으로 단정하지 않는다.

### 근거

- [2024-06-02 개인 첫 I2C 커밋](https://github.com/dudgns128/webos-gardening/commit/a94a8fc01c9f), [테스트 앱 수정](https://github.com/dudgns128/webos-gardening/commit/00e4febe4ae0)
- [Arduino 펌웨어 — Hardware 브랜치](https://github.com/dudgns128/webos-gardening/blob/Hardware/arduino/i2cSlaveFinal/i2cSlaveFinal.ino), [센서 패킷 메모](https://github.com/dudgns128/webos-gardening/blob/Hardware/dataIO.txt), [I2C 터미널 로그](https://github.com/dudgns128/webos-gardening/blob/Hardware/i2clog.txt)
- 당시 [webOS 하드웨어 정리.docx](https://docs.google.com/document/d/1zytZEiC4wiBWrzqpI_Gq6s3I1vR2YBp3/edit) — Uno R3, 핀 연결, A2/GND 대체 센서 상태; [산학협력 수행계획서](https://docs.google.com/document/d/18LWxavmFAWhKTcsNzp8NcVUUII7MqT5t7-c5WA97lR4/edit)
- 이후 팀원 수정: [센서 callback 경계](https://github.com/dudgns128/webos-gardening/commit/a6aa3a5657e5), [DHT 시작 초기화](https://github.com/dudgns128/webos-gardening/commit/4b52723da0f8)

### 남은 확인(좁힌 질문만 추후 확인)

- 당시 실제 배선/시연에서 **펌프·NeoPixel 물리 동작까지 어떤 범위를 검증**했는가? `i2clog.txt`는 버스 센서 응답만 뒷받침.
- **해결됨 (2026-10-27 C):** `readRawBytes()`는 원시 바이트값을 I2C 패킷으로 전달하기 위해 직접 추가한 함수. 라이브러리 전체 저작으로 확대하지 않음.
- 10바이트 패킷과 명령 모드 0/1/2의 **선택 이유**(센서별 길이 고정/구현 단순성 등)는 당시 어떤 제약 때문인가? 추정 금지.

**묶음 상태:** 상황·코드 변경·버스 테스트 일부 복원. **최초 판단 근거·기기 실동작 범위는 미확정**으로 남기고, 다음은 webOS JS Service의 I2C 통합 커밋 전후를 확인.

## 4. SVC-01A — webOS JS Service의 dummy 센서 → I2C 실데이터 통합

**커밋 묶음(모두 `nanocode00` author, 2024-06-12):** [`f2e890aab82d`](https://github.com/dudgns128/webos-gardening/commit/f2e890aab82d) → [`6fc7336eae66`](https://github.com/dudgns128/webos-gardening/commit/6fc7336eae66) → [`ea08aa6a93d6`](https://github.com/dudgns128/webos-gardening/commit/ea08aa6a93d6). 각 커밋의 실제 파일별 patch 및 부모 커밋을 확인했다. **기존 JS Service·DB·만족도 계산 로직의 최초 저작은 별도 팀원 선행 작업**이며 이 묶음은 그 위에 Hardware API를 붙인 기여를 중심으로 한다.

### 상황

기존 `Webos/dev` JS Service는 `getSensingDataJSON()`이 랜덤값을 반환하고 `controlLight`/`controlWater`는 출력 메시지 또는 TODO로 남아 있었다. 별도로 **HW-01에서 Arduino I2C 센서 패킷과 장치 명령 처리**가 구현돼 있었다. 이 둘을 연결하여 webOS의 주기적 센싱·만족도/자동제어 로직이 실제 하드웨어를 호출하게 하는 통합 단계.

### 문제

- 기존 센서값이 **dummy**이고, JS Service의 제어 함수에서 LED·펌프의 실제 명령 호출이 없었다(A: `f2e890` 이전 diff).
- **서로 다른 webOS API 호출 방식이 혼재**했다. 첫 통합 시 `openI2C`와 NeoPixel은 `service.call`을 사용했지만 펌프는 일부 `bridge.call`을 사용했고, LS2 응답을 문자열 `JSON.parse(msg)` 처리했다. JS Service용 `service.call`과 응답 객체 구조로 맞추는 후속 diff가 존재한다. **실제 관찰한 오류 메시지/재현 조건은 미확인**.
- `getSensingDataJSON()`을 `async`로 변경했는데, 기존 5초 센싱 경로에서 초기에 `await`가 없어서 결과가 Promise일 수 있었다(A: `ea08aa` 수정 diff).

### 판단

- webOS Peripheral Manager `luna://com.webos.service.peripheralmanager/i2c/{open,write,read,close}`를 JS Service에서 호출하고, Arduino 쪽 HW-01 프로토콜의 slave address **1**, mode **2**, **10바이트** 데이터 구조를 동일하게 파싱했다.
- `getSensingDataJSON()`에서 `readSensor()`를 `await`하고 제어 함수를 기존 만족도/급수 흐름에 연결. **이 선택의 대안·실제 디버깅 기준은 코드에 명시되지 않아 확인 필요**.
- 센서 조회를 두 단계(`write([0,2])` 후 `read(size:10)`)로 구성하고 **23ms setTimeout 뒤 read**, 함수는 추가 지연 후 반환하도록 설계. 지연은 초기 50ms → 중간 1000ms → 6/12 최종 **500ms**로 변경됐다. **하드웨어 완료/성공 callback을 await하는 구조는 아니어서 통신 완료 시점 보장으로 표현하지 않는다**.

### 조치(변경 추적)

| 커밋 | 실제 코드 변경 | 한계/비고 |
|---|---|---|
| `f2e890` 최초 통합 | `start()`에서 `openI2C()` 호출. `getSensingDataJSON()`을 `async`로 바꾸고 랜덤값을 `readSensor()` 데이터로 교체. `controlNeopixel`, `controlPump`, `readSensor`, `delay` 추가. 급수 시 `controlPump(1) → delay(3000) → controlPump(0)` 구성. deploy 스크립트 web app 권한에 `peripheralmanager.i2c.operation` 추가 | `service.call`과 `bridge.call` 혼재, 문자열 파라미터/응답 파싱 사용. 주기적 호출의 `await` 누락 |
| `6fc733` 중간 수정 | 광량/펌프 호출을 일시 주석 처리, `closeI2C`도 주석 처리, `readSensor` 고정 대기 **50 → 1000ms**. React 일부 레이아웃 및 `appinfo.json` 관련 정리 | 최초 구현을 그대로 완성판으로 간주하면 중간 주석 처리/되돌림을 놓친다. 사용자 회고(C, 2026-10-27): 어떤 동작이 잘되지 않아 센서 기능부터 분리해서 확인하려던 것으로 기억함. 실패한 구체적 기능과 오류 원인은 미확정 |
| `ea08aa` 후속 안정화 | 주기적 `const data = await getSensingDataJSON()`으로 수정. I2C 호출 파라미터를 JS 객체 형태로, 응답은 `res.payload.returnValue` 및 `arg = res.payload`로 수정. 펌프의 `bridge.call`을 `service.call`로 통일. NeoPixel/펌프 실행과 `isAutoControl` 조건 활성화. 센서 고정 대기 **1000 → 500ms**, 광량 모달과 MainPage UI도 일부 수정 | 커밋 제목은 `Fix: HW 완성`이지만 **실물 E2E 시험 기록이나 장기 재현성은 이 메시지 하나로 증명 불가** |

### 결과 및 이후 변경 여부

- **A: 코드 단계**에서 Arduino 10바이트 형식과 webOS `readSensor` 파싱이 연결되고, 서비스 센싱/제어 함수가 실제 I2C write/read 경로를 호출하도록 수정된 사실은 확인됨. 최초 dummy 데이터 경로에 비해 실데이터/구동 경로가 만들어졌다는 것이 직접 성과.
- **시험 범위**: HW-01의 `i2clog.txt`는 Raspberry Pi에서 **CLI로 I2C 센서 패킷을 읽은 기록**이다. 동일 날짜 JS Service에서 전체 자동센싱·DB 갱신·실제 펌프와 NeoPixel 동작까지 한번에 통과한 로그는 이번 묶음에서 **아직 확인하지 못함**.
- **비동기 제약**: `readSensor()`는 I2C 읽기 callback에서 지역변수를 채운 뒤 **고정된 500ms 지연 후 반환**한다. callback 실행 지연 또는 오류 시 필드가 `undefined`일 수 있고, 실제 I2C 응답 완료와 성공을 직접 await/reject하지 않는다. 따라서 '완전히 안정적인 동기화'나 '동작 100% 검증'은 주장하지 않는다.
- **분기·후속 타인 기여**: GitHub README는 `main`의 dummy data 사용을 별도 명시하고 실제 HW branch `Webos/devHW` 확인을 안내. `Webos/devHW`의 최종 JS 서비스에는 **2024-10-11~30 후속 타인 author 변경**(예: `605550e01289` 지연 조정, `18e6e1677669` 물탱크 레벨, `0d8d6baf1985` 자동 조명/간격 조정)이 더 있다. 최종 파일의 모든 처리 규칙을 6/12 본인 커밋으로 역귀속하지 않는다.
- **결과 범위**: 실물 데모가 있었는지는 팀 README에서 데모 링크가 존재하는 점까지 확인. **2026-10-27 사용자 직접 회고(C): 최종 시연에서 센서값의 화면 표시·NeoPixel 조명 제어·물 펌프 작동을 모두 직접 확인**. 단, 영상 파일 자체의 독립 검수나 자동/원격 제어 경로 전체의 시험 조건은 아직 확인되지 않았다.

### 남은 좁힌 확인

- **기억 보완 완료(C, 2026-10-27):** 동작하지 않는 부분이 있어 센서 경로만 분리하여 점검하려 했던 것으로 회고. 정확한 장애 원인은 아직 확인되지 않음.
- 당시 `service.call`/응답 `payload` 정리와 500ms 대기는 **어떤 관찰된 증상을 해결하는 과정**이었는지? 원인과 의사결정을 기억으로 보완해야 `문제 → 판단`을 확정 가능.
- **사용자 회고로 해소(C, 2026-10-27):** 최종 시연에서 센서값이 화면에 표시되고 NeoPixel 조명과 물 펌프가 실제 작동하는 것을 **모두 직접 확인**했다고 진술. 추가 영상·계측 로그 없이 장기 안정성이나 모든 센서별 정확도·원격 전체 경로까지 확대하지 않음.

**묶음 상태:** JS Service 통합 구현과 수정 순서는 A 확인. **API 오류의 실제 증상, 고정 대기 선정 이유 및 종단 실물 결과는 미확정**. 이력서용 문장은 아직 작성하지 않는다.

## 5. SVC-01B / CLEAN-01 — WebSocket 연결·메시지 수신 및 파일 정리 — 3커밋 조사

**상황:** 기존 webOS JS Service와 외부 서버의 원격 제어 메시지 처리가 필요했던 단계. 기존 서비스 전체 코드와 서버 자체의 최초 구현자는 이 작업의 저작자로 간주하지 않는다.

**문제:** 2024-06-08 이전에는 WebSocket 의존성 호출이 비활성화되고 접속 주소가 예시 값이었다. 2024-06-12 수신 처리에서는 메시지를 파싱하지 않고 속성을 참조했고, 자동제어 명령의 숫자/문자열 분기값이 서로 달랐다. 이는 diff에서 확인되는 불일치이며 실제 오류 메시지까지 확인된 것은 아니다.

**조치·결과(A: Git diff):**
- [1275599ddfce](https://github.com/dudgns128/webos-gardening/commit/1275599ddfce) (6/8): JS Service에 `ws` 의존성을 추가하고 WebSocket import와 외부 서버 주소를 실제 설정값으로 변경. **연결 성공 로그는 미확인**.
- [6783a5de18a7](https://github.com/dudgns128/webos-gardening/commit/6783a5de18a7) (6/12): 수신 원문을 `JSON.parse(rawMessage)`로 파싱하고 자동제어 명령 분기를 `16`/`'16'`으로 통일. 기존 분기와 실제 제어 함수의 최초 구현 자체를 이 커밋에 귀속하지 않는다.
- [d799833ca911](https://github.com/dudgns128/webos-gardening/commit/d799833ca911) (6/12): `JsService/db8/plantInfo.js` 91줄 파일 삭제. 단순 파일 정리이며 전체 DB 리팩터링·회귀 없는 성공을 단정하지 않는다.

**유지 여부:** 2024-10월 `Webos/devHW` 최종 파일에는 WebSocket import, 메시지 파싱, 명령 분기 패턴이 남아 있다. 다만 후속 팀원 수정도 포함되어 **원격 제어 E2E 성공, 서버 코드 저작, 장애 해결 원인**까지 이 묶음에서 확정하지 않는다.

**잠정 구분:** 첫 두 커밋은 WebSocket 수신 통합 작업, 세 번째는 독립된 정리 커밋으로 분리. 이력서에는 커밋 수를 성과로 나열하지 않는다.

## 6. WEB-01A — React 화면의 WebSocket 원격 제어 연결 초안 (2024-06-06~08)

**범위:** `WebPage/dev`에서 사용자 `nanocode00`가 author인 비병합 커밋 4개. 이 4개가 연결된 순서로 Git 부모 SHA·패치를 대조했다. **기존 React 컴포넌트 전부를 직접 최초 작성했다고 주장하지 않는다.** 검토 링크: [`126397712f02`](https://github.com/dudgns128/webos-gardening/commit/126397712f02), [`08960758c838`](https://github.com/dudgns128/webos-gardening/commit/08960758c838), [`8725eedb1078`](https://github.com/dudgns128/webos-gardening/commit/8725eedb1078), [`ec3d2ce378b0`](https://github.com/dudgns128/webos-gardening/commit/ec3d2ce378b0).

### 상황 → 문제 → 판단 → 조치 → 결과 (임시 복원)

- **상황(A):** 팀의 React 앱은 기존 webOS `WebOSServiceBridge`를 통한 JS Service 직접 호출 및 화면별 API 코드가 섞여 있었다. 외부 서버에서 원격 식물 조회·급수·광량·자동제어를 처리하는 WebSocket 연동 방식으로 접속 경로를 추가하던 시기.
- **문제(A):** React 화면별 브리지 호출/서버 연동이 아직 통일되지 않았다. 6/8 첫 WebSocket 구현에는 실제 엔드포인트와 응답 JSON 형태의 불일치, 화면 선택·센싱 데이터 갱신 시 잘못된 state 갱신 코드가 보인다. **왜 브리지 호출을 중단했는지와 각 장애의 실제 재현 로그는 아직 없음**.
- **판단(A 또는 D):** 여러 컴포넌트가 하나의 `WebSocketUtil`을 통해 식물 데이터와 명령을 주고받도록 설계했다(A: 코드). 반면 이를 webOS 브리지 방식보다 선택한 조직·환경적 이유는 문서로 확인되지 않아 **추정(D)으로도 확정하지 않음**.
- **조치(A):** 아래 4개 커밋에 따라 브리지 호출 제거/비활성화 → WebSocket 연결 및 메시지 코드 작성 → 제어 메시지 세부 값 수정 → 연결 URL 및 수신 객체 접근 변경.
- **결과(A):** React 여러 화면에서 같은 WebSocket 관리 객체를 참조하고 서버의 명령 메시지를 처리하려는 코드가 실제로 추가됐다. **연결 성공·실물 펌프 및 조명 원격 구동·모든 React 화면 정상 작동은 이 4커밋과 메시지만으로 검증되지 않았다**. 후속 커밋에 추가 수정이 이어짐.

### 커밋별 실제 변경

| 커밋 | 검증된 diff | 귀속·주의 |
|---|---|---|
| **6/6 `1263977`** | Calendar·PlantAutocontrol·PlantCondition·ControlLight에서 `WebOSServiceBridge` 생성 및 Luna 서비스 호출을 **주석 처리**. `UserPlantRegister.js` 파일 삭제(334라인)와 라우트에서 `PlantSelection` 사용, `useHistory` → `useNavigate` 시도 | 결과는 **기존 기능을 잠시 비활성화·구성 변경한 것**이지 정상 대체 완료가 아님. 이 시점 `navigate.push()`가 남아 있어 Hook API 사용 일관성이 없음. 기존 등록 화면의 334라인을 본인이 구현했다고 주장하지 않음 |
| **6/8 `0896075`** | `WebSocketUtil.js` 신설: WebSocket 생성, `onopen`에서 식별 메시지 `method:30`/`JSnum:false` 전송, `onmessage`의 `method:11` 식물 목록·`method:13` 센싱/페이지 데이터 처리. UserLogin(`10`), PlantSelection(`12`), ControlWater(`14`), ControlLight(`15`), MainPage·PlantConditionModal에 공통 객체 참조와 데이터/제어 메시지 추가 | **기초 WebSocket 연동 구현의 주요 개인 커밋**. 그러나 초기 주소는 `wss://…/webpage`, 수신 경로는 `arg.data.plants`/`arg.data`; 이후 바뀜. 초기 PlantSelection의 `const [plants, setPlants]` 상태 변수에 재대입하려는 코드도 남아 있으며 완성 동작 아님 |
| **6/8 `8725eed`** | PlantAutocontrolModal에 `method:16` 자동제어 토글 메시지 추가. ControlLight의 문자열 템플릿값을 숫자/입력값 전달로 조정. 식물 선택·로그인의 `userPlant` 식별자를 `null` → `0`으로 변경 | 서버 프로토콜 대응을 위한 값 보정은 확인. `setCurrentState(!currentState)` 뒤 이전 `currentState`를 전송하는 코드가 있어 당시 UI 상태와 보낸 값의 정합성은 의문; **정상 토글 성공으로 확대하지 않음** |
| **6/8 `ec3d2ce`** | WebSocket URL을 `wss://…/webpage` → `ws://…/ws`로 변경. 수신 JSON 객체를 `arg.data.plants` → `arg.plants`, `arg.data` → `arg`로 수정. PlantSelection에서 `navigate.push()` → `navigate()`, 식물 ID `plant.plantId` → `plant.id`로 수정하고, 데이터 로딩을 1.5초 `setTimeout` 기반으로 시도. MainPage에 `useLocation` import | **URL과 실제 기대 응답 스키마를 맞춘 코드 수정**은 확인. 1.5초 지연은 수신 완료를 보장하지 않으며 `useEffect(...,[plants])`에 후속 문제가 남을 수 있다. 제목의 '서버 연결 및 오류 수정'만으로 E2E 성공 주장 금지 |

### 후속 유지·교체 확인

- **2024-06-12 본인 `cd97e39ace46`:** 단순 WebSocketUtil 전역 데이터 폴링을 화면별 `onReceivePlantsCallback`, `onReceivePlantPageDataCallback`, `onReceivePlantConditionDataCallback`으로 전달하는 방식으로 변경. PlantSelection도 1.5초 지연 폴링 대신 식물목록 수신 콜백 + React state 갱신으로 수정. **6/8의 지연 기반 초안은 그대로 최종 적용된 방식이 아니다**. 이 후속 커밋은 별도 WEB-01B 묶음으로 추가 검토 예정.
- **2024-06-12 본인 `1377108fb518` / `0ec37f08e857`:** WebSocket callback 존재 검사, 수신목록 반영, 자동제어 메시지 콜백 등 수정이 이어짐. 구현 결과·충돌 여부는 후속 묶음 전체 diff 확인 필요.
- **최종 `WebPage/dev` 코드:** WebSocketUtil과 식물 선택·자동제어 화면은 남아 있지만, 수신 콜백 기반 데이터 전달 및 더 많은 구조 변경이 반영됨. **6/8 상태를 최종 UI 품질이나 성공 판본으로 귀속하지 않는다.**
- **역할 경계:** WebSocketUtil·공통 메시지 구성 및 해당 React 컴포넌트의 연결 변경은 개인 커밋으로 확인. 기존 전체 React UI·외부 WebSocket 서버·API 원안은 팀원/공동 구현 가능성이 있으므로 본인 단독 저작이라고 쓰지 않음.

### 남은 확인

- 6/6에 기존 `WebOSServiceBridge`를 비활성화한 구체적인 실행 환경/문제는 당일 로그 없이 추정 불가.
- 실제 6/8~12 React 화면에서 **로그인→식물 선택→실시간 센서 표시→조명/펌프 제어** 중 어느 범위까지 시연 성공했는지 원본 녹화·당시 회고로 확인 필요.
- 원격 제어 명령의 `method:10/12/14/15/16` 규약은 Git에 코드로 남지만 서버 측 계약의 최초 설계·모든 API 정상 응답 여부는 추적 필요.

**묶음 상태:** 네 커밋 **diff 확인 완료**. 통합 방향과 주요 본인 코드 수정은 직접 근거(A), 실제 동작·선정 이유는 미확정. **성급한 이력서 서술 없이 후속 WEB-01B를 이어 조사**한다.

## 7. WEB-01B — React 수신 콜백 전환과 데이터 흐름 수정 (2024-06-11~12)

**조사 커밋:** [`3bd71c54270e`](https://github.com/dudgns128/webos-gardening/commit/3bd71c54270e) (6/11), [`cd97e39ace46`](https://github.com/dudgns128/webos-gardening/commit/cd97e39ace46), [`1377108fb518`](https://github.com/dudgns128/webos-gardening/commit/1377108fb518), [`0ec37f08e857`](https://github.com/dudgns128/webos-gardening/commit/0ec37f08e857) (6/12). 본인 author의 **일반 커밋 4개**이며, 전후 파일·개별 patch·`WebPage/dev` 후속 판본 확인.

### 상황

6/8 WEB-01A에서 `WebSocketUtil`이라는 공통 객체가 추가돼 React 화면의 식물 목록·센싱 수치를 읽거나 원격제어 명령을 전송할 수 있도록 통합 중이었다. 초기 MainPage/PlantConditionModal은 `setInterval(..., 3000)`으로 공유 데이터 필드를 폴링했고, PlantSelection은 `setTimeout(..., 1500)`으로 목록이 채워지길 기다렸다.

### 문제와 판단

- **A(코드):** 일정 시간 후 공유 객체의 데이터가 도착한다고 가정해 읽는 접근은 실제 수신 이벤트와 동기화되지 않는다. 메시지가 늦거나 `WebSocketUtil.plants`가 초기화되지 않았을 때 정상 UI 갱신을 보장하지 못하는 구조였다. 이것이 *당시 실제로 관찰된 버그였는지*는 로그 부족으로 미확정.
- **A(변경된 방향):** 각 화면이 `WebSocketUtil.onReceive…Callback` 핸들러를 등록하고, WebSocket `onmessage`가 `method`별로 해당 콜백을 호출하도록 연결. 즉 **고정 지연·주기적 조회 초안에서 이벤트 수신 시 React state를 갱신하는 방식으로 전환**. 왜 이 방식을 선택했는지에 대한 별도 회고는 아직 없음.

### 구체적 조치

| 커밋 | 실제 diff로 확인한 변경 | 검증 주의 |
|---|---|---|
| `3bd71c5` | `JsService/node_modules/.package-lock.json`와 `node_modules/ws/**` **20개 외부 패키지 파일**, 총 **5,308행 추가**. 6/11 JS Service에서 `ws` 모듈을 시험적으로 포함 | 추가 행 대부분이 **제3자 패키지 자체**. 개인 작성한 5,308라인 기능이 아니다. 동작 성공/배포/설치 규칙 검증의 근거도 아님 |
| `cd97e39` | `WebSocketUtil.onmessage`에서 식물 목록 `onReceivePlantsCallback` 및 센싱 `onReceivePlantPageDataCallback`, `onReceivePlantConditionDataCallback` 호출 추가. MainPage·PlantConditionModal은 3초 폴링 대신 콜백으로 React 상태 갱신, unmount 시 핸들러 해제. PlantSelection은 1.5초 타이머 제거, 수신 시 `setPlants`·로딩 종료. ControlLightModal(`method:15`)과 ControlWaterModal(`method:14`)도 기존 브리지 코드 대신 WebSocket 제어 메시지 전송 | **기능 연결의 개인 diff는 확인**했지만 서버 송수신 E2E 성공이나 모든 모달 정상 동작 증거는 별도 필요. 기존 모달 UI 전체를 직접 신규 개발한 것으로 주장하지 않음 |
| `1377108` | 식물 목록 캐시 `WebSocketUtil.plants` 설정(현재는 수신 콜백이 있을 때만 저장), PlantSelection 초기 `loading` 여부를 캐시 존재에 따라 계산하고 기존 캐시의 목록을 `setPlants`로 복원. 콜백 존재 여부 `!= undefined` → `!== undefined`로 수정 | **메시지가 화면 등록 전에 도착해 콜백이 없다면 여전히 캐시에 저장되지 않을 수 있다**는 조건부 제약이 있음. 실제 발생 여부 미확인 |
| `0ec37f0` | `method:3` 자동제어 응답을 `onReceiveAutoControlCallback`에 전달하는 경로 추가. PlantAutocontrolModal에서 이전 상태를 사용하려는 코드와 콜백을 연결. MainPage의 배경·경험치 진행 막대 표시 추가 | **자동제어 모달 콜백에서 정의되지 않은 `setPlants`/`setLoading` 호출을 도입**한 중간판. 오류 없음·정상 동작이라고 해석하면 안 되며 후속 `5946539`에서 `setCurrentState`로 수정 |

### 결과 / 후속 유지 여부

- **A:** 화면별 수신 이벤트 핸들러와 자동제어 데이터 갱신 경로가 Git diff에 구현돼 있으며, 2024-10월 `WebPage/dev` 최종 코드에도 PlantSelection/MainPage/PlantConditionModal/PlantAutocontrolModal의 이벤트 기반 패턴은 남아 있다.
- **A:** 최초 브리지 직접 호출·타이머에 기반한 대기 경로는 대부분 대체됐다. 다만 **직접 WebSocket onmessage → callback을 등록하는 방식이라 완전한 수신 보장이나 모든 화면 상태의 정합성이 입증된 것은 아님**.
- **A:** `0ec37f0` 시점 자동제어 수신 함수는 타 화면의 `setPlants`/`setLoading`을 잘못 사용했고, 후속 `5946539`에서 바로잡음. 이 버그는 **일시적으로 도입됐다가 고친 사례**이며 처음부터 정상 구현이라고 쓰지 않는다.
- **C:** 이 4커밋에 관한 장애 재현 현상과 원인·테스트 결과는 사용자 기억으로 추가 확인 가능하나, 회고 없이 완료/성공을 단정하지 않는다.

**묶음 상태:** 초안→이벤트 기반 수정 흐름, 외부 패키지와 개인 기여 분리, 실제 남았던 결함 및 후속 수정 검증은 **A 확인**. 실기기·서버 E2E의 성공 범위는 미확정.

## 8. WEB-01C — UI 표시·자동제어 토글·배포 설정 후속 (2024-06-12~10-31)

| 커밋 | 실제 diff와 결과 경계 |
|---|---|
| [`6c4283ff31f7`](https://github.com/dudgns128/webos-gardening/commit/6c4283ff31f7) (6/12) | MainPage의 여백 제거 및 식물 이미지 높이·레이아웃 수정 5줄. 실제 화면 개선 정도는 별도 스크린샷/실행 검증 전 |
| [`b257d57062dd`](https://github.com/dudgns128/webos-gardening/commit/b257d57062dd) (6/12) | 자동제어 토글에서 `setCurrentState(!currentState)` 뒤 **이전 `currentState`를 보낸 코드**를, payload가 `!currentState`를 전달하도록 수정. 사용자 제어 의도와 전송값이 어긋날 수 있는 버그 수정(A) |
| [`5946539cf3fa`](https://github.com/dudgns128/webos-gardening/commit/5946539cf3fa) (6/12) | 자동제어 응답 콜백에서 **존재하지 않는 `setPlants`/`setLoading` 호출을 제거**하고 `setCurrentState(isAutoControl)`로 수정. 잘못된 상태 갱신 함수의 적용 범위를 고침(A). 같은 커밋에서 effect 의존성에 `currentState` 추가. 그 선택의 효과/오류 여부는 확정하지 않음 |
| [`a543715ba059`](https://github.com/dudgns128/webos-gardening/commit/a543715ba059) (10/31) | `gardening_react/package.json`에 `"homepage": "."` 1줄 추가. **프로젝트 본 개발기간 밖에서 이루어진 상대경로 배포 설정**. 실제 배포 정상 완료 여부는 별도 확인 필요 |

**후속 비교:** `WebPage/dev` 최종 PlantAutocontrolModal은 수신된 자동제어 상태를 `setCurrentState`에 적용하고, 토글 전송 `newState = !currentState`를 사용한다. 같은 코드를 독립적으로 더 수정한 팀원 후속 변경도 있으므로 최종 파일 전체의 소유권을 위 2건에만 귀속하지 않는다.

**그룹 가치:** 자동제어 토글/수신 상태의 **두 가지 구체적 결함 수정**은 S/W 문제 해결 증거 후보. 이미지 크기·배포 설정은 보조 유지보수이며 이력서 대표 에피소드로 따로 과장하지 않는다.

## 9. CAM-01 — 2024년 9~10월 카메라 API 별도 실험

**대상(A: 본인 commit author):** [`2cbed98d4d47` (9/18)](https://github.com/dudgns128/webos-gardening/commit/2cbed98d4d47), [`0bc6ba58f073` (10/13)](https://github.com/dudgns128/webos-gardening/commit/0bc6ba58f073), [`991d8843aa7b` (10/13)](https://github.com/dudgns128/webos-gardening/commit/991d8843aa7b). 3~6월 수행계획서의 산학협력 기간 이후 **후속 기술 탐색**이다.

- **상황(A):** `Camera` 브랜치에 webOS Camera2 Peripheral API 연동 가능성을 시험하려는 별도 파일이 있다. 이것이 최종 스마트 화분의 정식 필수 기능인지, 독립 실험인지 당시 의도는 확인되지 않았다.
- **문제·판단:** 당시의 구체적인 카메라 장애, 다른 API와의 비교, 촬영 테스트 실패/선택 이유는 커밋 메시지나 남은 화면으로 판정할 수 없다. **카메라 기능의 미완성 또는 실패로 단정하지 않는다**.
- **조치(A):** 9/18 `cameraTest/appinfo.json`, `index.html`, 테스트앱 `.ipk`, 아이콘 등을 추가. webOS 권한 `camera.operation`, `WebOSServiceBridge`에서 `camera2/open → camera2/capture → camera2/close`를 호출하고 성공 여부·handle/path를 표시하는 화면 코드. **HTML 상단에 LG Electronics 예제 저작권 및 Apache-2.0 표시가 존재**하므로 앱 코드를 전부 본인 창작이라고 주장하지 않는다.
- **조치(A):** 10/13 `cameraApi.txt`에 `luna-send` 기반 `getCameraList`, `getInfo`, `open`, `getSolutions`, `startCapture`, `capture`, `setFormat`, `startPreview`, `webos.media/load`, `takeCameraSnapshot`, `unload`, `stopPreview`, `close` 호출 순서와 인자 예시를 기록·보완. 개인 기기 경로·주소·숫자 핸들 등 실행환경별 값은 이 문서에 복제하지 않음.
- **결과(A):** 카메라 웹 테스트 앱/패키지와 API 호출 절차 메모를 **작성·저장소에 추가했다는 사실**을 확인. 실제 카메라에서 사진이 성공적으로 저장됐는지, 어떤 장비인지, 그 결과가 최종 프로젝트에 투입됐는지 검증 가능한 실행 로그는 이 세 커밋에 없다.
- **후속 팀 코드 경계(A):** 2024-10-21 `932af3a7c6f3`(GitHub `nanocode00` author로 확인되지 않는 타인 커밋)의 MainPage/JS Service에는 카메라 화면 연결을 위한 추가 수정이 있다. **그 기능의 원저자나 최종 성공을 `Camera` 브랜치 3커밋 작성자 개인 성과로 합산하지 않는다.**

**잠정 소유권:** 카메라 API 연동 실험 및 명령 정리/테스트앱 반입. 원본 예제 코드 저작·실제 완제품 사용은 확인 필요. 포트폴리오에서 메인 HW/SW 통합 사례와 별도 **보조 경험**으로 취급.

## 10. OPS-01 — 저장소 정리·Revert·정적 배포 (2024년 9~10월)

| 개인 커밋(A) | 실제 diff / 검증 범위 |
|---|---|
| [`ec8f0e20fb48`](https://github.com/dudgns128/webos-gardening/commit/ec8f0e20fb48), 9/18 | `gardening_react/.gitignore` 23줄 추가. `node_modules`, 빌드 산출물, 로컬 환경 파일 등을 제외하려는 설정. 표준 ignore 템플릿에 가까워 **직접 구현한 기능 성과가 아닌 환경 정리**. 이 커밋이 기존 tracked 파일을 모두 Git에서 제거했다는 근거는 아님 |
| [`5fa0831b9bac`](https://github.com/dudgns128/webos-gardening/commit/5fa0831b9bac), 9/24 | `Revert "reative UI"` — 실제로는 [타인 author `f3e1976fb994`](https://github.com/dudgns128/webos-gardening/commit/f3e1976fb994cd7aad722aa0815cffbe0fe518c6)의 9파일/331줄 변경을 되돌림. UI·JS Service·서버 주소 등의 직전 변경 복구. **돌려놓은 코드의 원 저작을 본인에게 귀속하지 않음.** revert 수행 이유·이후 채택 정책 및 실제 정상 작동 결과는 미확정 |
| [`cec22e9efdba`](https://github.com/dudgns128/webos-gardening/commit/cec22e9efdba), 10/31 | `gh-pages` 브랜치에서 **부모가 없는 root commit**으로 정적 React 빌드 파일 `index.html`, asset manifest, `static/js/css`, 폰트/이미지, 개발환경 설정 등 21파일 반입. **생성물·외부 라이브러리·폰트/자산을 직접 코딩한 기능 줄 수로 집계하지 않음.** commit 존재는 배포용 파일을 올린 증거이나 URL 공개·실제 배포 성공은 확인 전 |

**결론:** 세 작업은 본 개발의 기능구현과 분리된 유지보수/변경 회수/배포 준비다. 특히 revert는 *타인의 변경을 원상복구한 운영 조치*이고, gh-pages root는 *개인 작성한 프런트엔드 150행*이 아니다. 이력서의 대표 개발 성과보다는 프로젝트 부수 활동으로 두는 편이 정확하다.

## 11. 개인 작업 누락 역검증 및 소유권 구분

**검증 결과(A):** 2026-10-27에 원본 저장소에서 `Camera`, `Hardware`, `Server/dev`, `WebPage/dev`, `Webos/dev`, `Webos/devHW`, `gh-pages`, `main`, `test`의 9개 브랜치를 각 페이지 끝까지 다시 조회하여 313개 **고유 SHA** 확보. 그중 `nanocode00` author의 **31개 고유 SHA** 확인. 아래 작업군 대표 SHA 총합은 개인 일반 커밋 **26개와 정확히 일치**(누락 **0**, 중복 **0**, 타인 author 혼입 **0**). 본인 병합 커밋 5개는 별도 목록으로 추적·중복 구현 성과 제외. 초기 `295개` 집계는 일부 브랜치 페이지를 끝까지 확인하지 못한 값으로 정정.

| 작업 ID / 증빙 단위 | 포함 일반 커밋 | 주된 개인 기여 범위 | 우선순위·검증 상태 |
|---|---:|---|---|
| HW-01 Arduino slave·통신 테스트 | 2 | I2C 10바이트 패킷·장치 핀 제어·webOS 독립 버스 검증, Adafruit 원시 바이트 함수 개인 확장(C) | **높음** · CLI 실제 통신 확인, 센서 전체·구동 E2E 미확인 |
| SVC-01A JS Service HW 제어 통합 | 3 | dummy→실센서, JS 서비스 I2C API, async/응답 패턴 수정, 제어함수 연결 | **높음** · 코드 확인, E2E 미확인 |
| SVC-01B WebSocket JS Service 수신 | 2 | `ws` 사용 설정 및 JSON 메시지 파싱·명령 ID 수정 | 중간 · 최종 패턴 잔존, 서버 측 전체 저작 아님 |
| CLEAN-01 서비스 파일 삭제 | 1 | 불필요하다고 판단한 파일 제거 | 낮음 · 원인/기능 회귀 결과 미확인 |
| WEB-01A WebSocket 초기 React 연결 | 4 | 기존 화면 공통 통신·명령 처리 초안 및 API URL/필드 수정 | 중간 · 초기 결함/후속 변경 확인 |
| WEB-01B 화면별 수신 상태 갱신 | 4 | 이벤트 콜백 전환 및 식물목록 캐시·제어 모달; `ws` 외부 패키지 반입 1커밋 포함 | **높음** · 화면 코드 확인, 런타임 E2E 미확인 |
| WEB-01C 토글·레이아웃·배포 후속 | 4 | 토글 전송값/콜백 수정·화면 수정·배포 경로 1줄 | 중간 · 두 오류 수정은 근거 있음, 배포 성공 미확인 |
| CAM-01 Camera 후속 PoC | 3 | 예제 기반 Camera2 테스트앱 추가·API 명령 탐색 기록 | 낮음 · 후속 실험, 실제 촬영/정식 사용 미확인 |
| OPS-01 유지보수·원상복구·빌드 | 3 | ignore 설정, 타인 UI 변경 revert, `gh-pages` 빌드 결과 반입 | 낮음 · 기능의 원저작과 배포 성공으로 확대 금지 |
| **합계** | **26** | **9개 잠정 조사 묶음**(실제 이력서 경험 9개를 의미하지 않음) | **0개 미매핑** |

**외부·팀·개인 경계:**
- **외부 라이브러리/예제:** Adafruit DHT(원작)와 사용자 직접 추가 `readRawBytes()`(개인 확장)를 분리. `ws` 패키지 전체 5,308 추가행과 LG전자 Camera 예제 코드, 정적 번들/폰트는 개인 신규 소스 코드로 산정하지 않음.
- **팀 선행:** React 기본 UI, JS Service DB·만족도 처리, 외부 WebSocket 서버 등은 최초 author와 통합 담당의 기여를 구별. 사용자 개인 commit의 일부가 이 팀 구현을 수정했다고 해서 전체 기능 저작자가 되는 것은 아님.
- **팀 후속:** 2024-10-09 Arduino 센서 측정 위치 변경·초기화, 2024-10월 JS Service 자동조명/카메라·React 화면 개선은 해당 타인 commit과 분리. revert된 다른 사람의 UI 코드도 개인 구현으로 표시하지 않음.
- **검증 경계:** `i2clog.txt`로 Raspberry Pi 10바이트 수신 성공(A) 확인. **최종 시연의 센서값 화면 표시·NeoPixel·펌프 물리 작동은 사용자 직접 회고(C)**로 별도 확인. 센서별 계측 정확도·반복 안정성, 원격 HTTP/WebSocket 전체 구간, 카메라 실물 촬영, 정적 배포 정상 운영은 이번 Git diff와 회고로 보장되지 않음.

**작업 원장 상태:** 개인 일반 커밋 **26개 1차 diff/귀속 검토와 역매핑 완료**. 9개 조사 묶음은 [canonical v1](webos-smart-planter.md)에서 **핵심 GROUP-01~04 및 보조 Camera/운영 사례**로 통합했다. 현 단계에 남는 일은 일부 **실물 시연·오류 원인·선택 기준의 증거 보강**이다.

## 12. 후속 검증 단계 (canonical v1 작성 이후)

1. **최종 HW 시연 회고 확보(C):** 사용자 본인이 센서값 표시, NeoPixel 조명, 물 펌프 작동을 모두 직접 확인했다고 진술. 당시 영상의 구체적 장면, 센서별 정확도·안정성 및 원격제어 전체 구간은 향후 독립 증거가 있으면 추가 검증한다.
2. **판단 근거 확인:** I2C·고정 10바이트 데이터 형식 채택 이유, JS Service에서 500ms 고정 지연을 사용한 제약/당시 장애를 가능한 범위에서 사용자 회고로 보강한다.
3. **기타 후속 실험:** Camera API 테스트에서 실제 캡처에 성공했는지, GitHub Pages 배포가 정상 동작했는지를 로그/기억으로 확인한다. 해당 증거가 없으면 `미확정`을 유지한다.
4. **포트폴리오와 이력서:** canonical [GROUP-01+02](webos-smart-planter.md)를 우선 대표 사례, GROUP-03+04를 두 번째 사례 후보로 선정했다. `resume.md` 반영과 최종 PR 병합은 이력서 전체 프로젝트들의 우선순위 검토 후 별도 판단한다.
