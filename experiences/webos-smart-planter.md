# webOS Smart Planter

## 한 줄 요약

Raspberry Pi 4의 webOS OSE와 Arduino를 I2C로 연결하고, 실제 센서 수집과 조명·급수 actuator 제어를 webOS JS Service에 통합한 5인 팀 스마트 홈 가드닝 프로젝트입니다. 하드웨어 firmware와 JS Service를 함께 수정해 `Sensor → Arduino → I2C → webOS → 제어 로직 → Actuator` 흐름을 실제 장치까지 연결했습니다.

## 기간 및 프로젝트 정보

- 초기 기획 자료: **2024년 3월**
- GitHub에서 직접 HW/서비스 연동 작업이 확인되는 시점: **2024년 6월**
- 하드웨어 정리 문서: **2024년 7월**
- 5인 팀: 김영훈, 김재훈, 박지환, 오승우, 홍지승
- 2024 제22회 임베디드 소프트웨어 경진대회 webOS 부문 입선

초기 제안 발표에서는 다음 기능을 목표로 제시했습니다.

- 홈 가드닝 키트에서 센서 기반 상태 모니터링과 정보 표시
- 웹 앱에서 사용자/식물 정보 등록, 환경 데이터 확인, 알림
- 수분량·일조량 제어
- Open API 기반 정보 제공
- 사용자 얼굴 인식, 게이미피케이션

다만 이 문서는 **초기 기획 자료**이므로 얼굴 인식이나 게이미피케이션을 최종 구현 기능으로 자동 간주하지 않습니다. 이 경험 문서에서는 Git과 후속 하드웨어 자료로 확인되는 기능을 중심으로 정리합니다.

## 전체 시스템 구조

공개 README에서 확인되는 전체 구조는 다음과 같습니다.

- Raspberry Pi 4 + webOS OSE
- Arduino
- DHT11 온습도 센서
- 조도 센서
- 수위 센서
- 토양 수분 센서용 입력
- 24개 WS2812B NeoPixel Ring
- Water Pump
- webOS JS Service
- DB8
- React WebApp
- WebSocket 기반 외부 서버 연동
- Spring Boot + AWS EC2
- MySQL + InfluxDB

센서 데이터는 Arduino에서 수집하고 Raspberry Pi의 webOS JS Service가 I2C로 읽습니다. 반대로 JS Service에서 조명 밝기와 급수 명령을 Arduino로 전달합니다. React WebApp과 외부 서버는 JS Service를 통해 상태 조회와 원격 제어 흐름에 연결됩니다.

공개 README에는 InfluxDB를 약 5초 주기의 환경 센싱 시계열 데이터 저장에 사용하고, MySQL은 사용자/식물 기본 정보 저장에 사용한다고 정리되어 있습니다. 이는 팀 전체 아키텍처이며 개인 구현으로 과장하지 않습니다.

## 실제 하드웨어 구성

2024년 7월 하드웨어 정리 문서에는 다음 구성이 기록되어 있습니다.

- Raspberry Pi 4: **webOS OSE 2.24.0** 설치, Arduino 전원 공급
- Arduino: Raspberry Pi와 SDA/SCL로 I2C 연결
- NeoPixel Ring 24개: Arduino pin 3, `Adafruit_NeoPixel` 사용
- Water Pump: Arduino pin 4, **다이오드 + 트랜지스터 전류 증폭 회로** 사용
- DHT11: Arduino pin 2, 프로젝트에서 수정한 DHT library 사용
- CdS 조도 센서: A0, 10K pull-up resistor
- 수위 센서: A1
- 토양 수분 입력: A2

### Arduino 보드 모델에 대한 자료 차이

공개 README의 Hardware Setup은 **Arduino Nano**를 적고 있지만, 2024년 7월 별도 하드웨어 정리 문서는 **Arduino Uno R3**를 실제 업로드 대상으로 기록합니다.

firmware의 핵심 인터페이스는 일반 Arduino I2C/ADC/GPIO이므로 프로젝트 설명에서는 특정 보드 모델보다 **Arduino ↔ Raspberry Pi I2C 연동**에 초점을 둡니다. 이력서에서 보드 모델을 굳이 써야 한다면 자료 간 차이가 있다는 점을 확인한 뒤 사용합니다.

### 토양 수분 센서 검증 범위

firmware와 공개 README에는 A2의 토양 수분 센서 입력이 포함되어 있습니다. 그러나 2024년 7월 하드웨어 정리 문서에는 당시 토양 수분 센서를 아직 보유하지 못해 **A2를 GND에 연결해 사용했다**고 기록되어 있습니다.

따라서 다음을 구분합니다.

- 토양 수분 입력을 포함한 firmware/protocol 구현: 확인 가능
- 당시 보관된 하드웨어 구성에서 실제 토양 수분 센서를 연결해 end-to-end 검증: 확인 불가

이력서나 면접에서는 DHT11, 조도, 수위 센서의 실제 연동과 NeoPixel/Pump actuator 제어를 중심으로 설명하는 것이 안전합니다.

## Git으로 확인되는 직접 기여

`nanocode00` 계정의 커밋에서 다음 작업이 직접 확인됩니다.

### Arduino 하드웨어 제어

2024년 6월 2일 `i2c hardware control` 커밋에서 Arduino I2C 제어 코드가 추가되었습니다.

최종 `i2cSlaveFinal.ino`의 주요 구조는 다음과 같습니다.

- Arduino를 I2C slave address `1`로 설정
- Raspberry Pi에서 전달하는 mode에 따라 동작 분기
- mode `0`: NeoPixel 밝기 제어
- mode `1`: Water Pump ON/OFF
- mode `2`: 센서값 갱신 후 Raspberry Pi가 읽을 수 있도록 준비
- DHT11, 조도, 수위, 토양 수분 입력을 총 10바이트 packet으로 구성

### 10바이트 센서 데이터 프로토콜

Arduino가 Raspberry Pi로 전달하는 센서 데이터는 다음 순서로 구성됩니다.

- byte 0~3: DHT11 온도·습도 raw data
- byte 4~5: 조도 센서 ADC 값
- byte 6~7: 물통 수위 센서 ADC 값
- byte 8~9: 토양 수분 입력 ADC 값

아날로그 값은 10-bit ADC 결과를 상위/하위 바이트로 나눠 전송하고, JS Service에서 다시 결합해 서비스용 값으로 변환합니다.

## NeoPixel과 Water Pump 제어

NeoPixel Ring은 24개 LED를 사용하며 Arduino pin 3에 연결했습니다.

webOS에서 0~100 범위의 밝기 값을 전달하면 Arduino에서 이를 0~255 범위로 변환해 전체 NeoPixel에 적용합니다.

Water Pump는 Arduino pin 4에 연결하고 `0/1` 명령으로 OFF/ON을 제어합니다. 하드웨어 문서에서는 다이오드와 트랜지스터를 이용한 전류 증폭 회로를 사용했다고 기록되어 있습니다.

JS Service의 급수 함수는 펌프를 켠 뒤 약 3초 후 끄는 흐름으로 실제 급수 동작을 연결했습니다.

## webOS JS Service와 I2C 연결

2024년 6월 12일 `Feat: HW 제어 로직 추가 - i2c api 활용` 커밋에서 기존 dummy sensor 값을 실제 하드웨어 데이터로 교체했습니다.

webOS Peripheral Manager의 Luna API를 사용해 다음 기능을 구현했습니다.

- `luna://com.webos.service.peripheralmanager/i2c/open`
- `i2c/write`
- `i2c/read`
- `i2c/close`

서비스 시작 시 I2C 장치를 열고, 센서 조회 시 Arduino에 mode `2`를 전달한 뒤 10바이트 데이터를 읽습니다.

읽어온 데이터는 JS Service에서 다음 정보로 변환됩니다.

- temperature
- humidity
- light
- water
- watertank_level

기존에 랜덤값을 반환하던 `getSensingDataJSON()`을 실제 `readSensor()` 결과를 사용하는 비동기 함수로 바꾸면서 dummy logic을 실제 장치 입력으로 전환했습니다.

### `main`과 실제 HW branch 구분

공개 README는 `main` branch가 HW 연결을 제외한 `webos/dev` 내용을 합친 상태이며, 이해와 재사용을 위해 일부 기능에 **dummy data**를 사용한다고 명시합니다.

실제 HW 연결이 포함된 코드는 `webos/devHW` branch에서 확인하도록 안내되어 있습니다. 따라서 Git을 다시 볼 때 `main`만 보고 실제 센서 연동이 없다고 판단하면 안 되고, 반대로 `main`의 dummy data를 실제 HW 결과로 오해해서도 안 됩니다.

## 자동 제어와 실제 장치 연결

서비스 로직에는 식물별 적정 환경과 현재 센서 값을 비교해 만족도를 계산하고 자동 제어 여부를 판단하는 구조가 있었습니다.

하드웨어 연동 과정에서 이 논리와 실제 actuator를 연결했습니다.

- 광량 제어 시 `controlNeopixel()` 호출
- 급수 시 `controlPump(1)` → delay → `controlPump(0)` 수행
- 자동 제어 여부에 따라 actuator 동작

즉 센서 데이터를 읽는 것에서 끝나지 않고, `센싱 → 판단 → 장치 제어`가 하나의 서비스 흐름으로 이어지도록 구현했습니다.

## WebSocket 원격 제어 연결

2024년 6월 8일 `Feat: ws 모듈 추가` 커밋에서 webOS JS Service에 Node.js `ws` 모듈을 추가하고 외부 서버 WebSocket endpoint와 실제 연결했습니다.

프로젝트 README 기준 외부 서버는 Spring Boot/AWS EC2 환경이며, WebSocket은 센서 상태와 원격 제어 명령을 주고받는 통신 경로로 사용했습니다.

이를 통해 Raspberry Pi가 로컬 하드웨어만 제어하는 데서 끝나지 않고 외부 웹/모바일 서비스와 제어 흐름을 연결할 수 있도록 구성했습니다.

## React UI 연결

React 기반 webOS WebApp이 JS Service와 Luna bus API로 통신합니다.

직접 커밋에서도 MainPage의 식물 이미지 표시, 광량 제어 modal 동작 등 일부 UI 수정이 확인됩니다. 따라서 UI 전체 구현을 주 역할로 주장하기보다는, **하드웨어와 서비스 연동 과정에서 필요한 React 화면 및 제어 흐름을 함께 수정했다**고 정리하는 것이 정확합니다.

## 시스템 통합 경험

이 프로젝트의 핵심은 각 파트를 개별 구현하는 것보다 다음 흐름을 실제 장치까지 연결한 점입니다.

`Sensor → Arduino → I2C → webOS JS Service → DB/UI/Server → Control Logic → I2C → Actuator`

센서 데이터의 byte layout, 값의 단위와 범위, 장치 제어 command를 일치시켜야 전체 시스템이 정상 동작했습니다.

특히 webOS 쪽 서비스 코드와 Arduino firmware를 함께 수정하면서 SW에서 전달한 명령이 실제 pump와 LED 동작으로 이어지고, 실제 센서값이 다시 서비스로 올라오는 양방향 인터페이스를 구현했습니다.

## Git에서 확인되는 주요 흐름

### 2024년 6월 2일: Arduino I2C 하드웨어 제어

- I2C slave firmware 구현
- DHT11 / 조도 / 수위 / 토양 수분 입력 읽기
- 10바이트 센서 packet 구성
- NeoPixel 제어
- Water Pump 제어

### 2024년 6월 8일: 외부 서버 WebSocket 연결

- Node.js `ws` dependency 추가
- webOS JS Service에서 외부 WebSocket server 연결

### 2024년 6월 12일: webOS와 실제 HW 통합

- dummy sensor data를 실제 I2C sensor data로 교체
- Peripheral Manager I2C API 적용
- sensing logic과 자동제어 logic 연결
- NeoPixel 및 Water Pump 실제 제어 연결
- 하드웨어 연동 오류 수정 후 `Fix: HW 완성` 커밋

## 결과

- Raspberry Pi 4 + webOS OSE 기반 실제 동작 시스템 구현
- Arduino와 I2C를 통한 센서/액추에이터 양방향 제어
- WebSocket 기반 원격 제어 구조 연결
- 2024 제22회 임베디드 소프트웨어 경진대회 webOS 부문 입선

## 이력서용 핵심 bullet

- Raspberry Pi 4의 webOS OSE와 Arduino를 I2C로 연결하고 **센서 데이터 packet과 NeoPixel/Water Pump 제어 command를 정의해 양방향 HW 인터페이스 구현**
- webOS Peripheral Manager I2C API를 적용해 **dummy sensor data를 실제 DHT11·조도·수위 센서 입력으로 교체**하고 서비스 자동제어 로직을 실제 actuator 동작까지 연결
- Node.js WebSocket을 webOS JS Service에 연결해 **로컬 HW 제어 흐름을 외부 서버의 원격 상태 조회·제어 구조와 통합**
- Arduino firmware와 webOS service를 함께 수정하며 **HW/SW interface mismatch를 end-to-end로 디버깅**

## 근거와 사용 시 주의

Git과 추가 하드웨어 자료로 직접 확인되는 강한 근거는 다음입니다.

- Arduino firmware
- 10-byte I2C protocol
- webOS JS Service의 Peripheral Manager I2C 연동
- DHT11 / 조도 / 수위 실제 하드웨어 구성
- NeoPixel / Water Pump actuator 연결
- pump transistor/diode 회로
- WebSocket module 연결

반면 다음 내용은 개인 구현으로 과장하지 않습니다.

- 팀 전체 Spring Boot 서버 구현
- DB 전체 설계
- React UI 전체 구현
- 초기 기획의 얼굴 인식 / 게이미피케이션을 최종 구현 기능으로 단정
- 토양 수분 센서의 실제 end-to-end 검증 완료

보드 모델도 README와 하드웨어 정리 자료가 서로 다르므로, 이력서에서는 `Arduino`로 표현하는 것이 가장 안전합니다.

## 보여주는 역량

- Embedded/IoT 시스템 통합
- Arduino firmware 개발
- I2C protocol 설계 및 디버깅
- Raspberry Pi + webOS OSE 2.24.0
- Luna Service / Peripheral Manager API 활용
- 센서 데이터 parsing
- actuator 제어와 transistor 기반 pump 구동
- WebSocket 통신
- HW와 서비스 로직 사이 인터페이스 설계
- 실제 장치까지 이어지는 end-to-end 디버깅

## 자소서 활용 포인트

- HW/SW 융합 역량
- 임베디드 시스템 통합 경험
- 통신 인터페이스를 직접 정의하고 연결한 경험
- dummy data를 실제 센서/actuator로 전환한 경험
- 서로 다른 팀 파트의 interface를 맞춘 협업 경험
- SW 로직이 실제 물리 장치 동작으로 이어지는 시스템을 완성한 경험
