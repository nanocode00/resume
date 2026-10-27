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
| **WEB-01 React·WebSocket 인터페이스** | `126397712f02`, `08960758c838`, `8725eedb1078`, `ec3d2ce378b0`, `3bd71c54270e`, `cd97e39ace46`, `1377108fb518`, `0ec37f08e857`, `6c4283ff31f7`, `b257d57062dd`, `5946539cf3fa`, `a543715ba059` | **미검토**. 커밋 제목 기준 임시 분류이며 UI·웹소켓·배포 수정 단위로 분해 예정 |
| **SVC-01 webOS JS Service 통합** | `1275599ddfce`, `6783a5de18a7`, `d799833ca911`, `f2e890aab82d`, `6fc7336eae66`, `ea08aa6a93d6` | **부분 검토**. WebSocket 수신/메시지 파싱, 센서 실데이터 교체, I2C API, 자동 제어 연결, `await` 및 LS2 응답 형식 후속 수정. 세부 기여·결과 확정 전 |
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
3. 동일 커밋: `arduino/DHT.cpp`, `DHT.h` 포함. 헤더에 **Adafruit Industries 원작 및 MIT 라이선스**가 명시되어 있으므로 라이브러리 전체를 직접 작성했다고 주장하지 않음. 현재 코드의 `readRawBytes(uint8_t[])`는 DHT 원시 바이트 4개를 노출하는 프로젝트용 인터페이스지만 **정확히 누가 어느 원본과 비교해 수정했는지는 별도 확인**.
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
- DHT 라이브러리의 `readRawBytes` 노출은 직접 수정한 부분인가? Adafruit 원본 코드를 통째로 개인 작성이라 주장하지 않음.
- 10바이트 패킷과 명령 모드 0/1/2의 **선택 이유**(센서별 길이 고정/구현 단순성 등)는 당시 어떤 제약 때문인가? 추정 금지.

**묶음 상태:** 상황·코드 변경·버스 테스트 일부 복원. **최초 판단 근거·기기 실동작 범위는 미확정**으로 남기고, 다음은 webOS JS Service의 I2C 통합 커밋 전후를 확인.

## 4. 다음 작업 순서

1. **SVC-01의 6월 12일 변경군:** `f2e890aab82d` → `6fc7336eae66` → `ea08aa6a93d6`를 우선 비교. `getSensingDataJSON()`의 dummy → 실데이터, `bridge.call` → `service.call`, 비동기 `await`, 자동제어 조건, 최종 버그 여부를 코드 단위로 검토.
2. **WEB-01 WebSocket·화면 제어:** React/webOS 서비스 양쪽 변경을 별도 문제로 재분류.
3. Camera/기타 브랜치 및 개발기간 이후 정리 커밋까지 매핑. **전체 개인 non-merge 26개 모두 diff 기준으로 역검증**한 뒤 canonical 경험 문서 확정.
4. 결과/개인 기여가 불명확한 부분만 사용자 기억을 물음. `resume.md` 및 기존 경험 문서는 근거 조사 종료 전 수정하지 않음.
