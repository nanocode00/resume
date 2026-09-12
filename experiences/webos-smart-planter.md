# webOS Smart Planter

## 한 줄 요약

Raspberry Pi 4의 webOS OSE와 Arduino를 I2C로 연결하고, 센서 수집·자동 제어·JS Service·React UI·외부 서버까지 이어지는 스마트 홈 가드닝 시스템을 구현한 프로젝트.

## 프로젝트 구조

저장소 README에서 확인되는 전체 구조는 다음과 같습니다.

- Raspberry Pi 4 + webOS OSE
- Arduino Nano
- DHT11 온습도 센서
- 조도 센서
- 수위 센서
- 토양 수분 센서
- 24개 WS2812B NeoPixel Ring
- Water Pump
- webOS JS Service
- DB8
- React WebApp
- WebSocket 기반 외부 서버 연동
- Spring Boot + AWS EC2
- MySQL + InfluxDB

센서 데이터는 Arduino에서 수집하고 Raspberry Pi의 webOS JS Service가 I2C로 읽습니다. 반대로 JS Service에서 조명 밝기와 급수 명령을 Arduino로 전달합니다. UI와 원격 서버는 JS Service를 통해 상태 조회와 장치 제어를 수행하는 구조입니다.

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
- DHT11, 조도, 수위, 토양 수분 값을 총 10바이트로 구성해 전달

### 10바이트 센서 데이터 프로토콜

Arduino가 Raspberry Pi로 전달하는 센서 데이터는 다음 순서로 구성됩니다.

- byte 0~3: DHT11 온도·습도 raw data
- byte 4~5: 조도 센서 ADC 값
- byte 6~7: 물통 수위 센서 ADC 값
- byte 8~9: 토양 수분 센서 ADC 값

아날로그 센서 값은 10-bit ADC 결과를 상위/하위 바이트로 나눠 전송하고, JS Service에서 다시 결합해 백분율 범위의 값으로 변환합니다.

### NeoPixel과 Water Pump 제어

NeoPixel Ring은 24개 LED를 사용하며 Arduino pin 3에 연결했습니다.

webOS에서 0~100 범위의 밝기 값을 전달하면 Arduino에서 이를 0~255 범위로 변환해 전체 NeoPixel에 적용합니다.

Water Pump는 Arduino pin 4에 연결하고 `0/1` 명령으로 OFF/ON을 제어합니다. JS Service의 급수 함수에서는 펌프를 켠 뒤 약 3초 후 끄는 흐름으로 실제 급수 동작을 연결했습니다.

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
- water: 토양 수분
- watertank_level

또한 기존에 랜덤값을 반환하던 `getSensingDataJSON()`을 실제 `readSensor()` 결과를 사용하는 비동기 함수로 바꿨습니다.

## 자동 제어와 실제 장치 연결

기존 서비스 로직에는 식물별 적정 환경과 현재 센서 값을 비교해 만족도를 계산하고 자동 제어 여부를 판단하는 구조가 있었습니다.

하드웨어 연동 과정에서 이 논리와 실제 actuator를 연결했습니다.

- 조도가 부족하거나 높으면 NeoPixel 밝기 조절
- 사용자가 직접 광량을 변경하면 `controlNeopixel()` 호출
- 급수 시 `controlPump(1)` → delay → `controlPump(0)` 수행
- 자동 제어 여부에 따라 actuator 제어

즉 센서 데이터를 읽는 것에서 끝나지 않고, `센싱 → 판단 → 장치 제어`가 하나의 서비스 흐름으로 이어지도록 구현했습니다.

## WebSocket 원격 제어 연결

2024년 6월 8일 `Feat: ws 모듈 추가` 커밋에서 webOS JS Service에 Node.js `ws` 모듈을 추가하고 외부 서버 WebSocket endpoint와 실제 연결했습니다.

프로젝트 README 기준으로 외부 서버는 Spring Boot/AWS EC2 환경이며, WebSocket은 센서 상태와 원격 제어 명령을 주고받는 통신 경로로 사용했습니다.

이 구조를 통해 Raspberry Pi가 로컬 하드웨어만 제어하는 데서 끝나지 않고 외부 모바일/웹 서비스에서 원격으로 상태를 확인하고 급수·광량 제어를 할 수 있도록 연결했습니다.

## React UI 연결

React 기반 webOS WebApp이 JS Service와 Luna bus API로 통신합니다.

직접 커밋에서도 MainPage의 식물 이미지 표시, 광량 제어 modal 동작 등 일부 UI 수정이 확인됩니다. 따라서 이 경험에서는 UI 전체 구현을 주 역할로 주장하기보다는, **하드웨어와 서비스 연동 과정에서 필요한 React 화면 및 제어 흐름을 함께 수정했다**고 정리하는 것이 정확합니다.

## 시스템 통합 경험

이 프로젝트의 핵심은 각 파트를 개별 구현하는 것보다 다음 흐름을 실제 장치까지 연결한 점입니다.

`Sensor → Arduino → I2C → webOS JS Service → DB/UI/Server → Control Logic → I2C → Actuator`

센서 데이터의 byte layout, 값의 단위와 범위, 장치 제어 command를 일치시켜야 전체 시스템이 정상 동작했습니다.

특히 webOS 쪽 서비스 코드와 Arduino firmware를 함께 수정하면서 SW에서 전달한 명령이 실제 pump와 LED 동작으로 이어지고, 실제 센서값이 다시 서비스로 올라오는 양방향 인터페이스를 구현했습니다.

## Git에서 확인되는 주요 흐름

### 2024년 6월 2일: Arduino I2C 하드웨어 제어

- I2C slave firmware 구현
- DHT11 / 조도 / 수위 / 토양 수분 센서 읽기
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

## 근거와 사용 시 주의

Git으로 직접 확인되는 강한 근거는 **Arduino firmware, I2C protocol, webOS JS Service의 HW 연동, WebSocket module 연결**입니다.

반면 팀 전체의 Spring Boot 서버, DB 전체 설계, React UI 전체를 개인 구현으로 표현해서는 안 됩니다. 이력서나 자소서에서는 본인의 역할을 다음과 같이 쓰는 것이 가장 안전합니다.

> Arduino 센서·액추에이터 제어와 webOS JS Service의 I2C 연동을 담당하고, 실제 센서값과 자동제어 로직을 연결했습니다. 또한 외부 서버 WebSocket 통신과 연동 과정의 UI를 함께 수정하며 HW부터 서비스까지 데이터 흐름을 통합했습니다.

## 보여주는 역량

- Embedded/IoT 시스템 통합
- Arduino firmware 개발
- I2C protocol 설계 및 디버깅
- Raspberry Pi + webOS OSE
- Luna Service / Peripheral Manager API 활용
- 센서 데이터 parsing
- actuator 제어
- WebSocket 통신
- HW와 서비스 로직 사이 인터페이스 설계
- 실제 장치까지 이어지는 end-to-end 디버깅

## 자소서 활용 포인트

- HW/SW 융합 역량
- 임베디드 시스템 통합 경험
- 통신 인터페이스를 직접 설계하고 연결한 경험
- dummy data를 실제 센서/actuator로 전환한 경험
- 서로 다른 팀 파트의 interface를 맞춘 협업 경험
- SW 로직이 실제 물리 장치 동작으로 이어지는 시스템을 완성한 경험
