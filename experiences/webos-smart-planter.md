# webOS Smart Planter — Canonical 경험 기록

> **상태 (2026-10-27):** GitHub 9개 브랜치, 고유 커밋 313개 중 본인 author 일반 커밋 26개 전수 diff 1차 검토를 반영한 **사실 기준 문서 v1**. 기능 단위 기여와 검증 한계를 확정 가능한 수준에서 정리했으나, 실제 시연 영상의 동작 구간·일부 당시 판단은 **추가 확인 필요**다. 세부 SHA·변경 전후·불확실성은 [개발작업 원장](webos-smart-planter-commit-ledger.md)을 따른다. 이 문서의 문장 전체가 바로 이력서에 실릴 수 있다는 의미는 아니다.

## 1. 프로젝트 개요

- **주제:** webOS OSE 기반 스마트 홈 가드닝 솔루션 개발. 온습도·조도·수위 등 환경 센서값을 수집하고, NeoPixel 조명·급수 펌프를 제어하며, 식물 상태 확인/원격제어 화면을 연결하는 5인 팀 프로젝트.
- **팀 및 대외 협력:** 학부생 5명, LG전자 협력 산학 프로젝트. 당시 [수행계획서](https://drive.google.com/file/d/18LWxavmFAWhKTcsNzp8NcVUUII7MqT5t7-c5WA97lR4/view)는 **2024-03-01~06-30**을 공식 수행기간으로 기재. 사용자 작성 커밋상 실제 하드웨어/서비스 집중 구현은 **6월**, Camera·운영 후속 작업은 **9~10월**에 존재. 추후 작성된 개인 졸업포트폴리오에는 3~11월 참여 활동으로 기록되어 있으나, 공식 수행기간과 **구분**해 이해한다.
- **팀 공개 자료:** [webos-gardening 저장소](https://github.com/dudgns128/webos-gardening), [프로젝트 소개 및 아키텍처](https://github.com/dudgns128/webos-gardening#readme), README에 연결된 [데모 영상](https://youtu.be/0eDjUXKEMuM).
- **대외 결과:** 개인 졸업포트폴리오에는 **2024 제22회 임베디드 소프트웨어 경진대회 webOS 부문 입선**으로 기재됨(**B: 본인 사후 기록**). 상장 원본/대회 공식 명단을 이 조사에서 직접 검증한 것은 아니며 수상의 개인 원인·순위를 과장하지 않는다.
- **초기 목표와 구현 구분:** [초기 제안 발표](https://drive.google.com/file/d/15_vLdybpZ2_TmFOs9tNfTs9j0ke8cksP/view)에는 얼굴 인식, Open API, 게이미피케이션, 알림 등 **제안 기능**이 포함된다. 최종 기능·개인 구현이 입증됐다는 자료가 아니므로 Git과 하드웨어 기록에서 확인된 범위만 본문에 기술한다.

## 2. 시스템 구조와 개인 역할의 경계

**팀 전체 구조(README 근거):**

`센서(Arduino) → I2C → Raspberry Pi 4 / webOS OSE JS Service → DB8·WebSocket·React 화면·외부 Spring Boot 서버 → I2C → NeoPixel/펌프`

- **HW:** Raspberry Pi 4 + webOS OSE 2.24.0, Arduino **Uno R3 개발 배선**(당시 하드웨어 정리 문서), **Nano 최종 구성**(README와 이전 경험 기록). SDA/SCL I2C, DHT11(pin 2), 24구 NeoPixel(pin 3), 펌프(pin 4), CDS 조도(A0), 수위(A1), 토양 수분용 A2 입력이 기록돼 있다. 펌프에는 트랜지스터·다이오드 회로 사용. **Uno→Nano 직접 전환 작업의 개인 소유권/구체적 검증 시점은 커밋만으로 미확정**.
- **webOS:** Peripheral Manager의 Luna I2C API, JS Service, DB8, 자동 환경 관리 로직.
- **프런트·서버:** React 화면 / WebSocket 연동. 외부 Spring Boot(AWS EC2), MySQL, InfluxDB 등은 **팀 전체 구성**이며 본인 단독 서버/DB 원저작으로 기재하지 않는다. README는 `main`에 **dummy data**가 남아 있고, 실제 하드웨어 통합 코드는 `Webos/devHW`를 보라고 구별한다.

| 기여 종류 | 직접 확인한 개인 기여 | 최초/최종 소유권에 대한 주의 |
|---|---|---|
| **임베디드 펌웨어** | Arduino I2C slave·10바이트 센서 패킷·NeoPixel/펌프 명령 처리, webOS CLI I2C 시험 스케치/앱 | Adafruit DHT 라이브러리 **원작은 타인**. 원시 바이트 노출 함수 `readRawBytes()`는 직접 추가했다는 사용자 회고(C)와 코드 존재(A)를 함께 기록. 10월 센서 처리 위치·초기화 수정은 **팀원 author** |
| **webOS 서비스 통합** | dummy 센싱을 I2C 실데이터 코드로 변경, Luna API·async 호출·제어 함수 연결 및 응답 형식 수정 | DB8·만족도·자동제어 기존 로직 **전체의 최초 구현은 타인/공동 작업**. 전체 최종 서비스 코드도 팀원 후속 수정 포함 |
| **원격제어/웹페이지 연결** | JS Service에 `ws` 의존성·수신 JSON 처리, React 화면의 `WebSocketUtil`·메시지 송신·이벤트 기반 상태 갱신, 자동제어 토글 버그 수정 | 기존 React 화면·외부 서버 구현/프로토콜 원안은 본인 단독 저작으로 주장하지 않음 |
| **후속 기술 탐색·유지보수** | Camera API 예제 기반 테스트앱·`luna-send` 메모, `.gitignore`, 타인 변경 revert, gh-pages 정적 빌드 반입 | 9~10월 작업을 3~6월 산학협력 정식 기간 안의 핵심 구현과 구분. 예제·외부 패키지·번들/폰트 원작 제외 |

## 3. 통합 경험 GROUP-01 — 센서·액추에이터 I2C 인터페이스 구축

**작업 ID:** `HW-01` · 핵심 근거 [a94a8fc (2024-06-02)](https://github.com/dudgns128/webos-gardening/commit/a94a8fc01c9f), [I2C 통신 로그](https://github.com/dudgns128/webos-gardening/blob/Hardware/i2clog.txt).

- **상황:** Raspberry Pi의 webOS 서비스와 Arduino를 연결해 센서값을 읽고 실제 제어 명령을 내려보내야 하는 하드웨어 통합 단계였다.
- **문제:** 센서 데이터 필드·바이트 순서와 명령 형식의 공통 인터페이스가 필요했다. 당시 테스트에선 **토양습도 센서가 없어 A2 핀을 GND에 연결**한 기록이 존재하므로 모든 센서의 실측값 검증으로 말할 수 없었다.
- **판단:** Arduino를 **I2C slave address 1**로 구성하고 mode `0`(밝기), `1`(펌프), `2`(센서 갱신)로 구분했다. 데이터는 DHT11 습도·온도 원시 바이트 **4바이트 + 아날로그 CDS·수위·토양 수분 각 2바이트 = 총 10바이트**로 만들었다. 왜 다른 인터페이스보다 I2C를 택했는지는 명시적 회고/문서가 없어 미확정.
- **조치:** `Wire.onReceive`/`onRequest` 콜백, `analogRead` 값의 상·하위 바이트 분해, 밝기 0~100→PWM용 색상 밝기 0~255 변환 및 NeoPixel 24개 설정, 펌프 GPIO ON/OFF 코드를 추가했다. **Adafruit DHT 원본 라이브러리에 `readRawBytes()` 함수를 직접 추가**해 데이터 변환 전 원시 바이트 4개를 I2C 패킷으로 전달할 수 있게 했다고 사용자 직접 확인(C). 해당 라이브러리 자체를 직접 만든 것은 아님.
- **결과(A):** Raspberry Pi의 `luna-send` 로그에는 `i2c/open`·`write([0,2])`·`read(size:10)`·`close` 성공 응답과 실제 10바이트 배열이 있다. **실제 I2C 버스에서 데이터가 왕복한 것은 확인**되나, DHT11 원시 데이터 해석의 정확도·모든 센서 정상 계측·LED/펌프의 실물 동작은 이 로그만으로 증명되지 않는다.
- **후속 구현 소유권(A):** 2024-10-09 팀원 커밋 `a6aa3a5`의 센서 읽기 콜백→`loop()` 이동, `4b52723`의 `dht.begin()` 추가는 본인 성과와 분리.

**자료 주의:** `dataIO.txt`의 ADC 2바이트 결합 `×128` 메모와 실제 Arduino(상위 8비트 이동)/JS Service(`×256`) 코드는 **불일치**한다. 근거 있는 구현 표현은 실제 소스의 `256`이며, 당시 메모를 올바른 포맷으로 무조건 채택하지 않는다.

## 4. 통합 경험 GROUP-02 — JS Service의 dummy → 실센서·장치 명령 통합

**작업 ID:** `SVC-01A` · [f2e890a](https://github.com/dudgns128/webos-gardening/commit/f2e890aab82d) → [6fc7336](https://github.com/dudgns128/webos-gardening/commit/6fc7336eae66) → [ea08aa6](https://github.com/dudgns128/webos-gardening/commit/ea08aa6a93d6) (2024-06-12).

- **상황:** 기존 webOS JS Service는 `getSensingDataJSON()`에서 랜덤 센서값을 반환하고 제어 함수에는 미완성 TODO/출력 경로가 있었다. Arduino 측 I2C 프로토콜은 앞서 구현되어 있었다.
- **문제:** JS Service와 실제 하드웨어 사이의 I2C 함수, 응답 처리 및 비동기 호출이 맞지 않았다. 실제 관찰한 오류 메시지는 남아 있지 않다.
- **판단:** webOS Peripheral Manager의 `luna://com.webos.service.peripheralmanager/i2c/{open,write,read,close}`를 이용하고 센싱/제어 함수가 동일한 `address:1`과 명령/패킷 구조를 사용하도록 연결했다. **문제가 있는 동안 제어 호출을 잠시 주석 처리하여 센서 경로만 분리 점검하려 했다는 사용자 회고(C)**가 있다. 특정 장애 원인은 확정하지 않는다.
- **조치:** `readSensor()` 호출을 통한 10바이트 파싱, `getSensingDataJSON()`을 `async`로 변경, 센싱 주기 경로에 `await` 추가. `service.call` 및 `res.payload.returnValue` 방식으로 인자/응답을 정리하고 조명·급수 동작을 I2C write에 연결. 급수 코드에는 펌프 ON→약 3초 대기→OFF 호출 흐름이 있다.
- **결과(A):** **dummy→실장 센서 데이터 읽기 코드 경로와 서비스→Arduino 명령 전송 경로의 구현**이 확인된다. 다만 `readSensor()`가 콜백 완료 신호를 직접 기다리는 대신 고정 지연(50ms→1000ms→500ms)을 이용했으므로 **비동기 완료 보장이나 장기간 통신 안정성은 입증되지 않았다**. 'HW 완성' 커밋 제목만으로 최종 E2E 통과는 주장하지 않는다.
- **팀 구현 경계:** 기존 DB8 데이터 모델/식물 만족도/자동 제어 조건 전체를 최초 구현한 것으로 쓰지 않는다.

## 5. 통합 경험 GROUP-03 — webOS↔외부 서버↔React 원격 제어 메시지 연결

**작업 ID:** `SVC-01B`, `WEB-01A`, `WEB-01B` · 2024-06-06~12.

- **상황·문제:** 기존 React 화면의 `WebOSServiceBridge` 직접 호출과 외부 WebSocket 연결 시도가 혼재했고, 초기 서버 URL·수신 JSON 구조·타이머 기반 표시 방식 등이 실제 통신 계약과 일치하도록 조정이 필요했다.
- **판단·조치:** webOS JS Service에 Node `ws` 모듈 의존성과 외부 서버 주소를 설정하고 `JSON.parse(rawMessage)` 및 명령 ID 정합성을 수정했다([1275599](https://github.com/dudgns128/webos-gardening/commit/1275599ddfce), [6783a5d](https://github.com/dudgns128/webos-gardening/commit/6783a5de18a7)). React에서는 `WebSocketUtil.js`를 추가하고 로그인(`method:10`), 식물 선택(`12`), 급수(`14`), 광량(`15`), 자동제어(`16`) 메시지를 각 화면에 연결했다([0896075](https://github.com/dudgns128/webos-gardening/commit/08960758c838) 등).
- **디버깅과 선택:** 처음엔 화면이 `setInterval(...,3000)`/1.5초 `setTimeout`으로 공유 객체 값을 기다렸다. 후속 [cd97e39](https://github.com/dudgns128/webos-gardening/commit/cd97e39ace46)부터 `WebSocketUtil.onReceive…Callback`을 연결해 **수신 메시지 도착 시 해당 React 화면의 state를 변경**하는 방식으로 수정. [1377108](https://github.com/dudgns128/webos-gardening/commit/1377108fb518)에는 수신 목록 캐시·초기 로딩 조건 보완이 있다. 이것을 변경했다는 사실은 확인되나, 당시 체감한 타이밍 장애의 정확한 원인은 미확정.
- **결과(A):** 수신 콜백 기반 상태 반영·원격 명령 생성 코드가 후속 `WebPage/dev`에도 **구조적으로 유지**된다. 그러나 WebSocket 연결 성공을 보여주는 테스트 로그, 외부 서버의 전체 API 계약 통과 또는 물리 장치까지의 원격 제어 시연 구간은 아직 직접 검증하지 않았다.
- **외부 코드 경계:** [3bd71c5](https://github.com/dudgns128/webos-gardening/commit/3bd71c54270e)에서 추가된 **5,308행은 Node `ws` 설치 패키지 파일**로, 직접 구현한 서비스 코드량이 아니다. 기존 React 디자인/화면과 Spring Boot 서버 전체의 원저작도 이 작업의 개인 성과로 합산하지 않는다.

## 6. 통합 경험 GROUP-04 — 자동제어 상태 불일치 수정

**작업 ID:** `WEB-01C` 관련 [8725eed](https://github.com/dudgns128/webos-gardening/commit/8725eedb1078) → [0ec37f0](https://github.com/dudgns128/webos-gardening/commit/0ec37f08e857) → [b257d57](https://github.com/dudgns128/webos-gardening/commit/b257d57062dd) → [5946539](https://github.com/dudgns128/webos-gardening/commit/5946539cf3fa) (6/8~12).

- **상황·문제(A):** React 자동제어 토글에서 `setCurrentState(!currentState)`와 서버로 보내는 `isAutoControl: currentState`가 서로 어긋나는 중간 코드가 있었다. 추가 수신 콜백에서는 **해당 컴포넌트에 존재하지 않는 `setPlants`/`setLoading` 함수**를 호출하도록 잘못 연결한 버전도 있었다.
- **판단·조치(A):** 전송 payload를 `!currentState`로 수정하고, 자동제어 응답을 담당 모달의 `setCurrentState(isAutoControl)`에 연결하여 **상태 변경 대상 및 메시지 값을 일치시키는 방향**으로 수정했다.
- **결과(A):** 문제 코드와 수정 diff가 나란히 존재하며 후속 `WebPage/dev`에도 같은 개념의 콜백/토글 처리가 유지된다. **프로덕션 서비스에서 모든 상황의 상태 동기화가 보장됐다는 주장**까지 이 커밋들에서 확인되는 것은 아니다.
- **대표성:** 통합 과정의 구체적인 버그 수정 사례로는 유효하지만, 더 큰 GROUP-03(React 원격 제어) 안의 서브 에피소드로 묶는 것을 우선 권장.

## 7. 보조 경험 — Camera API PoC와 후속 유지보수

- **CAM-01 (9~10월):** [2cbed98](https://github.com/dudgns128/webos-gardening/commit/2cbed98d4d47)에 webOS 카메라 열기·촬영·닫기 테스트 웹앱과 패키지, [0bc6ba5](https://github.com/dudgns128/webos-gardening/commit/0bc6ba58f073)·[991d884](https://github.com/dudgns128/webos-gardening/commit/991d8843aa7b)에 camera2 / webOS media 관련 `luna-send` 호출 메모를 추가했다. **테스트앱 HTML에 LG전자 예제 저작권 표시가 있음**. 실제 사진 저장 성공·정식 서비스 채택·코드 전체 신규 저작은 미확정.
- **OPS-01 (9~10월):** [.gitignore 추가](https://github.com/dudgns128/webos-gardening/commit/ec8f0e20fb48), [타인 UI 변경 revert](https://github.com/dudgns128/webos-gardening/commit/5fa0831b9bac), [gh-pages 정적 React 빌드 산출물 반입](https://github.com/dudgns128/webos-gardening/commit/cec22e9efdba) 및 10/31 상대 `homepage` 경로 설정. **원래 UI 저작·빌드 번들 원작·배포 정상 운영**으로 확대하지 않는다.
- 카메라 관련 **2024-10-21 팀원 커밋**에 서비스·UI 연동 코드가 별도로 있으므로 모든 Camera 관련 최종 변경을 개인 성과로 표시하지 않는다.

## 8. 시간순 요약

| 시기 | 확인 가능한 단계 | 증빙 |
|---|---|---|
| 2024.03~06 | 산학협력 프로젝트 공식 수행기간. 초기 제안에 다양한 기능 제시 | 수행계획서·초기 PPT |
| 2024.06.02 | Arduino I2C slave·센서 패킷·NeoPixel/펌프 제어 및 CLI 버스 테스트 코드 추가 | `a94a8fc`, `00e4febe`, `i2clog.txt` |
| 2024.06.06~08 | React 화면 WebSocket 초안 구현·서버 URL/응답 JSON 수정, JS Service `ws` 연결 | `1263977`, `0896075`, `8725eed`, `1275599`, `ec3d2ce` |
| 2024.06.11~12 | React 콜백 기반 상태 갱신, 자동제어 상태 버그 수정, JS Service 실센서/I2C 통합 | `cd97e39`, `1377108`, `0ec37f0`, `b257d57`, `5946539`, `f2e890` 등 |
| 2024.09~10 | 카메라 API 별도 시험, 코드 revert/빌드·배포 준비, 이후 팀원 통합 변경 | `Camera`, `Hardware`, `WebPage/dev`, `Webos/devHW` 커밋 |

## 9. 검증된 결과와 명시적 한계

| 주장 | 현재 인정 가능한 수준 | 근거 및 제한 |
|---|---|---|
| 10바이트 I2C 센서 패킷 송수신 | **직접 확인(A)** | Raspberry Pi `i2clog.txt` 성공 응답 및 값 배열. 실제 센서별 교정·물리 액추에이터 시험은 별개 |
| Arduino↔webOS JS Service I2C API 통합 | **코드 구현 확인(A)** | 코드 양쪽 프로토콜/호출 경로 일치. 서비스 전체 E2E 시연 로그 별도 미확인 |
| 원시 DHT 바이트 반환 코드 개인 작성 | **함수 존재(A) + 사용자 직접 확인(C)** | 원본 Adafruit DHT 라이브러리의 확장으로만 표현 |
| NeoPixel·펌프 자동 동작 | **회로/제어 코드·팀 README 확인(A/B)** | 실제 모든 장치의 E2E 동작·안정성은 사용자가 봤던 시연 구간으로 추가 확인 필요 |
| 토양 수분 센서 실제 측정 | **코드·최종 구성의 계획/기재는 확인(A/B)** | 당시 A2를 GND에 연결한 개발 단계 확인. 실제 최종 측정 결과 미확인 |
| React 원격제어·콜백 구조 | **구현 및 후속 유지 확인(A)** | 서버/디바이스 전체 원격 시험은 별도 |
| webOS 부문 입선 | **사용자 사후 이력 기록(B)** | 원본 상장/공식 결과를 이번 조사에서 직접 확인하지 않음 |
| Camera 촬영/공개 배포 성공 | **테스트앱·명령 및 정적 파일 존재(A)** | 실제 성공 영상·로그, 공개 서비스 정상 운영 미확인 |

**시연 증거의 범위:** GitHub README는 데모 영상 URL을 제공한다. 이번 조사에서는 링크 존재까지만 확인했고 **영상의 구체적인 작동 장면을 직접 검수하지 못했다**. 따라서 읽지 못한 화면/장면을 보았다고 서술하지 않는다.

## 10. 이력서·포트폴리오로 압축할 후보

다음은 **확정 사실 수준**으로 작성한 후보이며, 사용자가 구체적 시연을 확인하면 결과 문구를 그 범위에서 보강할 수 있다.

1. **대표 우선 — HW/SW 인터페이스:** Arduino에서 주소 1의 I2C slave·10바이트 센서 패킷·NeoPixel/펌프 명령 처리 코드를 구현하고, Raspberry Pi webOS OSE의 Peripheral Manager로 10바이트 실제 응답을 확인. Adafruit DHT 라이브러리에 원시 바이트 반환 인터페이스 추가.
2. **대표 우선 — 실제 데이터 통합:** webOS JS Service의 dummy 센서값을 I2C 센서 읽기로 교체하고 Luna API 비동기 응답·`await` 처리, 제어 함수 연동을 수정. 센서/제어 호출을 분리해 문제를 좁혀본 경험 포함.
3. **보조 또는 웹/서비스 직무용 — 원격 상태 갱신:** React의 WebSocket 연동 화면을 주기적 공유 데이터 읽기에서 수신 콜백 기반으로 변경하고, 자동제어 토글의 전송값/상태 갱신 오류를 수정.
4. **부수 기록 — Camera/배포:** webOS Camera2 API 사용 탐색, 웹 앱 build/배포 설정. 핵심 HW 통합 성과와 별개이며 최초 성공 여부 미확인.

**추천:** 이 프로젝트에서 대표 경험은 GROUP-01+GROUP-02를 묶은 **하드웨어·서비스 데이터 계약의 양방향 통합**, 두 번째는 GROUP-03+GROUP-04의 **WebSocket 수신과 React 상태 갱신 문제 해결** 정도로 압축. Camera·배포는 희망 직무와 증거 수준에 따라 보조 항목으로만 포함한다. **이 문서를 작성했다고 `resume.md`가 자동 확정되지는 않는다.**

## 11. 근거 인덱스와 남은 확인

- **전체 SHA별 개인 기여·커밋별 diff·타인 구현 경계:** [`webos-smart-planter-commit-ledger.md`](webos-smart-planter-commit-ledger.md). 총 **고유 커밋 313개**, 개인 author **31개(일반 26개 + 병합 5개)**, 원장 일반 커밋 **누락 0개**.
- **실제 HW 시기·부품·A2 대체 센서:** [당시 webOS 하드웨어 정리 문서](https://drive.google.com/file/d/1zytZEiC4wiBWrzqpI_Gq6s3I1vR2YBp3/view), [팀 README Hardware Setup](https://github.com/dudgns128/webos-gardening#hardware-setup).
- **공식 기간·초기 목표:** [산학협력 수행계획서](https://drive.google.com/file/d/18LWxavmFAWhKTcsNzp8NcVUUII7MqT5t7-c5WA97lR4/view), [초기 제안 PPT](https://drive.google.com/file/d/15_vLdybpZ2_TmFOs9tNfTs9j0ke8cksP/view). *후자는 결과 보고서가 아니다.*
- **본인 이후 참여/수상 회고:** [졸업포트폴리오](https://drive.google.com/file/d/1y3A65qJXOEoiqHoqyxLqhpcQe8qR6Yvk/view) — 사후 이력 기록이므로 당시 기능 검증을 대체하지 않음.
- **남은 증거 1 — 실제 HW 시연:** 온습도·조도·수위 표시 및 LED·펌프 물리 동작이 각각 **어느 단계에서 직접 작동했는지** 사용자 기억/사진·영상으로 구분해 확인.
- **남은 증거 2 — 설계 판단:** 10바이트 패킷/I2C 선택의 제약, 고정 지연 500ms의 실제 문제와 선택 이유를 기억으로 확인. 기억 불명확하면 가설로 남긴다.
- **남은 증거 3 — 원격·Camera:** 웹페이지→외부 서버→webOS→실장 제어 흐름/Camera 실제 캡처 성공을 입증하는 로그·시연. 없으면 구현 경로만 인정.
