# webOS Smart Planter — 커밋 기반 개발작업 원장

> 조사 시작: 2026-10-27 · 원본: [dudgns128/webos-gardening](https://github.com/dudgns128/webos-gardening) · 상태: **전체 커밋 인벤토리 1차 완료 / HW-01 기능 묶음 1차 복원**. 이 파일은 이력서 확정본이 아니다. `resume.md`와 기존 `experiences/webos-smart-planter.md`는 현재 변경하지 않는다.

## 1. 조사 범위와 분류 원칙

- 확인한 브랜치 **9개**: `Camera`, `Hardware`, `Server/dev`, `WebPage/dev`, `Webos/dev`, `Webos/devHW`, `gh-pages`, `main`, `test`. 각 브랜치 커밋 목록을 끝까지 페이지 단위 조회하고 **브랜치 간 중복 SHA 제거**.
- GitHub 계정 `nanocode00`을 author로 갖는 **고유 커밋 31개**, 그중 **일반 커밋 26개(부모 0개인 gh-pages 시작 커밋 포함)** 및 **병합 커밋 5개**. 브랜치별 동일 커밋을 중복 계산하지 않음.
- **작업 ID는 커밋 ID가 아니라 문제 해결 단위**. 아래 모든 26개 일반 커밋은 잠정 그룹에 매핑했고, 향후 각 diff를 읽으며 그룹을 분리·통합할 예정. 현재 임시 그룹명을 최종 경험 또는 개인 저작 결론으로 삼지 않음.
- 기술적 근거 우선순위: **diff → 이후 최종 코드와 테스트·로그 → 당시 문서 → 사용자 직접 기억**. 커밋 메시지만으로 완성/테스트 성공 판정 금지.
- 사용한 주요 자료: [하드웨어 브랜치](https://github.com/dudgns128/webos-gardening/tree/Hardware), [HW 통합 브랜치](https://github.com/dudgns128/webos-gardening/tree/Webos/devHW), 저장소 README, 2024년 당시 Google Drive `webOS 하드웨어 정리.docx`, 산학협력 수행계획서(2024-03-01~06-30, LG전자 협력, 학부 5인), 최종 발표 슬라이드.

## 2. 본인 일반 커밋 26개 — 누락 방지 인벤토리

| 잠정 작업군 | 해당 author 커밋(앞 12자리 SHA) | 단계 / 조사 범위 |
|---|---|---|
| **HW-01 Arduino/I2C 센서·제어** | `a94a8fc01c9f`, `00e4febe4ae0` | **첫 묶음 1차 검토 완료**. 펌웨어 및 webOS I2C 독립 테스트 앱. 결과 확인의 한계는 아래 |
| **WEB-01A React 화면의 WebSocket 통합 초안** | `126397712f02`, `08960758c838`, `8725eedb1078`, `ec3d2ce378b0` | **4커밋 diff·후속 코드 조사 완료**. 브리지 임시 비활성화 → WebSocketUtil 적용 → 제어명령 보정 → URL·응답 스키마 수정. 당시 완성/무결점 주장 금지 |
| **WEB-01B React 실시간 데이터 동기화·후속 안정화** | `3bd71c54270e`, `cd97e39ace46`, `1377108fb518`, `0ec37f08e857`, `6c4283ff31f7`, `b257d57062dd`, `5946539cf3fa`, `a543715ba059` | **세 커밋은 WEB-01A의 후속 흔적으로 부분 확인**. 아직 8개 전체 diff 미검토. 콜백 기반 갱신·모달 제어·배포 후속 수정 여부 확인 필요 |
| **SVC-01 webOS JS Service 통합** | `1275599ddfce`, `6783a5de18a7`, `d799833ca911`, `f2e890aab82d`, `6fc7336eae66`, `ea08aa6a93d6` | **I2C 통합 3커밋 조사 완료(6/12)**. 앞선 3커밋 중 WebSocket·메시지 파싱·파일 삭제는 다음 WEB/SVC 조사 대상. 완전한 실물 E2E 결과는 미확정 |
| **CAM-01 Camera 관련 실험** | `2cbed98d4d47`, `0bc6ba58f073`, `991d8843aa7b` | **미검토**. 브랜치·제목에서 실험 후보만 식별. 실제 사용·최종 채택 여부 미확정 |
| **OPS-01 정리·운영·기타** | `ec8f0e20fb48`, `5fa0831b9bac`, `cec22e9efdba` | **미검토**. gitignore / revert / gh-pages 작업 등을 따로 확인. 초기 개발기간 이후 작업을 원 프로젝트 구현 실적에 자동 포함하지 않음 |

**개인 author 병합 커밋 5개(개발작업으로 중복 산정하지 않음):** `4d357d94eeb4`, `0319b2d366ad`, `aa319bd64295`, `7be82da2905a`, `d51bffe6ae27`.

**전체 graph 조사 메모:** `Server/dev`는 조회한 커밋 내 `nanocode00` author 기록 없음. 그 사실만으로 사용자가 서버 전체에 관여하지 않았다고 단정하지 않음. 2024-09~10 커밋은 산학협력 수행계획서 기간(3~6월) 밖에서 발생하므로 본 개발/후속 활동을 구분.

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
- **결과 범위**: 실물 데모가 있었는지는 팀 README에서 데모 링크가 존재하는 점까지 확인. **해당 데모에서 어떤 센서·actuator가 정상 작동했는지는 직접 확인 전이므로 이 원장에서는 단정하지 않는다**.

### 남은 좁힌 확인

- **기억 보완 완료(C, 2026-10-27):** 동작하지 않는 부분이 있어 센서 경로만 분리하여 점검하려 했던 것으로 회고. 정확한 장애 원인은 아직 확인되지 않음.
- 당시 `service.call`/응답 `payload` 정리와 500ms 대기는 **어떤 관찰된 증상을 해결하는 과정**이었는지? 원인과 의사결정을 기억으로 보완해야 `문제 → 판단`을 확정 가능.
- 당시 최종 시연에서 **물리 센서 표시·조명·펌프** 중 각각 무엇을 직접 확인했는지는 실물 시험/데모 근거와 분리해 추후 확인.

**묶음 상태:** JS Service 통합 구현과 수정 순서는 A 확인. **API 오류의 실제 증상, 고정 대기 선정 이유 및 종단 실물 결과는 미확정**. 이력서용 문장은 아직 작성하지 않는다.

## 4. WebSocket 연결·메시지 수신 및 파일 정리 — 3커밋 조사

**상황:** 기존 webOS JS Service와 외부 서버의 원격 제어 메시지 처리가 필요했던 단계. 기존 서비스 전체 코드와 서버 자체의 최초 구현자는 이 작업의 저작자로 간주하지 않는다.

**문제:** 2024-06-08 이전에는 WebSocket 의존성 호출이 비활성화되고 접속 주소가 예시 값이었다. 2024-06-12 수신 처리에서는 메시지를 파싱하지 않고 속성을 참조했고, 자동제어 명령의 숫자/문자열 분기값이 서로 달랐다. 이는 diff에서 확인되는 불일치이며 실제 오류 메시지까지 확인된 것은 아니다.

**조치·결과(A: Git diff):**
- [1275599ddfce](https://github.com/dudgns128/webos-gardening/commit/1275599ddfce) (6/8): JS Service에 `ws` 의존성을 추가하고 WebSocket import와 외부 서버 주소를 실제 설정값으로 변경. **연결 성공 로그는 미확인**.
- [6783a5de18a7](https://github.com/dudgns128/webos-gardening/commit/6783a5de18a7) (6/12): 수신 원문을 `JSON.parse(rawMessage)`로 파싱하고 자동제어 명령 분기를 `16`/`'16'`으로 통일. 기존 분기와 실제 제어 함수의 최초 구현 자체를 이 커밋에 귀속하지 않는다.
- [d799833ca911](https://github.com/dudgns128/webos-gardening/commit/d799833ca911) (6/12): `JsService/db8/plantInfo.js` 91줄 파일 삭제. 단순 파일 정리이며 전체 DB 리팩터링·회귀 없는 성공을 단정하지 않는다.

**유지 여부:** 2024-10월 `Webos/devHW` 최종 파일에는 WebSocket import, 메시지 파싱, 명령 분기 패턴이 남아 있다. 다만 후속 팀원 수정도 포함되어 **원격 제어 E2E 성공, 서버 코드 저작, 장애 해결 원인**까지 이 묶음에서 확정하지 않는다.

**잠정 구분:** 첫 두 커밋은 WebSocket 수신 통합 작업, 세 번째는 독립된 정리 커밋으로 분리. 이력서에는 커밋 수를 성과로 나열하지 않는다.

## 5. WEB-01A — React 화면의 WebSocket 원격 제어 연결 초안 (2024-06-06~08)

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

## 6. 다음 작업 순서

1. **WEB-01B React 실시간 데이터·콜백 전환:** `3bd71c54270e`, `cd97e39ace46`, `1377108fb518`, `0ec37f08e857`를 묶어 수신 데이터 반영 방식과 오류 대응을 확인.
2. **WEB-01B 나머지 React 수정:** `6c4283ff31f7`, `b257d57062dd`, `5946539cf3fa`, `a543715ba059`의 UI/코드 정리 및 배포 수정 여부를 검토. 최종 파일의 타인 후속 수정과 구분.
3. Camera/기타 브랜치 및 개발기간 이후 정리 커밋까지 매핑. **전체 개인 non-merge 26개 모두 diff 기준으로 역검증**한 뒤 canonical 경험 문서 확정.
4. 결과/개인 기여가 불명확한 부분만 사용자 기억을 물음. `resume.md` 및 기존 경험 문서는 근거 조사 종료 전 수정하지 않음.
