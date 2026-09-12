# 김재훈 — Resume

> 모바일 앱과 컴퓨터비전 기반 제품에서 시작해 임베디드/IoT, HW/EDA, AI/데이터까지 연결해 온 메이커형 SW/HW 융합 개발자

## Profile

- 경북대학교 컴퓨터학부
- 전자공학 부전공
- 관심 분야: Software, Computer Vision, Embedded/IoT, AI, HW/SW Co-design
- GitHub: https://github.com/nanocode00

## Core Strengths

- **제품까지 완성하는 개발 경험**: HummingBlocks를 프로토타입에서 Google Play 출시와 실제 판매까지 연결
- **AI 기능을 서비스로 연결하는 경험**: Mallo에서 STT/NLU/TTS 파이프라인과 접근성 중심 UI를 함께 검증
- **원인을 좁혀가는 디버깅**: 앱, AI 추론, Verilog CPU, RTL/Netlist 검증 과정에서 실행 흐름과 중간 결과를 비교하며 문제 해결
- **HW/SW 연결 경험**: Raspberry Pi, Arduino, micro:bit, Verilog, OpenROAD 등 소프트웨어와 하드웨어 경계를 넘나드는 프로젝트 수행
- **사용 목적 중심의 기술 선택**: 최신 기술 자체보다 정확도, 지연, 사용성, 구현 비용을 비교해 제품 목적에 맞는 방식을 선택

## Experience

### Beamworks — 현장실습

사내 업무관리 플랫폼의 요구사항 분석과 데이터 설계에 참여했습니다.

- Figma 화면을 바탕으로 입력값, 조회값, 저장값을 구분해 기능 요구사항 분석
- Jira의 Epic, Story, Task 단위로 업무 구조화
- ERD와 DB 테이블 설계
- SQL과 DTO 작성
- 화면 요구사항을 데이터 구조와 실제 개발 단위로 변환하는 과정을 경험

> 의료 AI 기업에서 수행한 현장실습이지만 담당 업무는 의료영상이 아닌 사내 업무관리 플랫폼입니다.

## Projects

### Mallo — Voice-first Kiosk

**Voice AI / Whisper / Local LLM / MeloTTS / Qwen3-TTS / FastAPI**

고령 사용자를 주요 대상으로 음성으로 주문을 보조하는 키오스크 프로젝트입니다. 팀장으로 참여해 TTS 파인튜닝, AI 모델 비교, 경쟁사 분석, UI 개선 방향 수립을 담당했습니다.

- Mic → VAD → STT → NLU → Validator → Order Engine → Adaptive TTS → TTS 파이프라인 구성
- AI Hub 친절체 데이터를 활용한 MeloTTS, Qwen3-TTS 파인튜닝
- 모델별 생성 시간과 표현력을 비교하고 서비스 적용 관점에서 검토
- Voice Overlay, 다시 듣기, 천천히 듣기, 직원 호출 등 접근성 UI 개선
- STT/NLU/TTS sidecar 분리와 health check 기반 런타임 구성
- 직접 녹음한 발화와 TTS checkpoint 비교를 통해 모델 결과 검증

상세: [experiences/mallo.md](experiences/mallo.md)

### HummingBlocks — 시각장애인 코딩교육 앱

**Android / Java / Python / PyTorch / YOLO / Chaquopy / TFLite**

실물 코딩 블록을 촬영하면 AI가 블록의 종류와 배치를 인식하고 음악으로 실행하는 접근성 기반 코딩교육 제품입니다.

- 약 1년 7개월 동안 Android 앱 학습, 프로토타입, 기능 개선, 출시와 운영까지 수행
- YOLO 검출 결과의 좌표를 분석해 블록 순서, 반복, 방향, 템포 등 음악 실행 규칙으로 변환
- 정상 실행이 어려운 예외 상황 22가지를 정리하고 사용자 안내 메시지 구현
- 모바일 추론 지연 개선을 위해 TFLite + Java 방식을 시험했으나 정확도 저하를 확인
- 제품 특성상 속도보다 정확한 블록 인식이 중요하다고 판단해 PyTorch + Chaquopy 방식을 유지
- 실제 사용 환경의 조명 문제와 기기별 화면 비율 문제를 발견하고 데이터 및 UI 개선에 반영
- Google Play 출시, 무료 앱과 유료 블록 키트 형태로 운영 및 판매
- 전국 장애·비장애 대학생 창업경진대회 대상

상세: [experiences/hummingblocks.md](experiences/hummingblocks.md)

### CNN Accelerator — AI 모델의 HW 이식 및 검증

**PyTorch / Verilog / ModelSim / OpenROAD / ASAP7**

PyTorch 양자화부터 Verilog 구현과 검증, 합성 이후 정확도와 PPA 평가까지 하나의 파이프라인으로 수행했습니다.

- 5×5 커널 기반 2단 CNN의 weight, bias, 레이어별 출력을 정수화해 HW 입력 형식으로 변환
- SW와 HW의 레이어 출력 및 MAC 단위 연산을 비교해 오차 원인 추적
- RTL 합성 후 `tb_compare`에서 RTL과 Netlist 결과 비교
- MNIST 1,000장 단위로 기능 및 정확도 유지 여부 검증

상세: [experiences/cnn-accelerator.md](experiences/cnn-accelerator.md)

### Single-cycle CPU — CP0 예외 처리를 포함한 Verilog CPU

**Verilog / Digital Logic / Computer Architecture**

- MIPS 형태의 단일 사이클 CPU 구현
- Control, ALU, Register File, Memory, Immediate Extender, CP0 구성
- syscall 발생 시 EPC 저장 → 예외 루틴 분기 → ERET 복귀 흐름 구현
- ALU 입력, Decoder 제어값, RegDst, ZeroExtend, EPC 저장 시점과 PC 흐름을 파형 기반으로 디버깅

상세: [experiences/verilog-cpu.md](experiences/verilog-cpu.md)

### webOS Smart Planter — LG 산학 연계 종합설계

**Raspberry Pi / webOS OSE / Arduino / React / AWS**

- 센서와 액추에이터를 연결한 Arduino 기반 하드웨어 제어
- Raspberry Pi와 장치 사이의 데이터 연결
- React UI 일부 구현
- 센서 이름, 단위, 전송 주기와 제어 기준 등 팀 간 데이터 규격 조정
- 2024 임베디드 소프트웨어 경진대회 webOS 부문 입선

상세: [experiences/webos-smart-planter.md](experiences/webos-smart-planter.md)

### Paperware — micro:bit 교육용 메이커 키트

**micro:bit / PCB Design / Prototyping**

- 카드보드와 구리박 테이프로 회로를 만드는 교육용 메이커 키트 개발
- 기존 네오픽셀 모듈의 납땜 필요성을 사용성 문제로 정의
- 카드보드에 바로 연결할 수 있도록 접점을 넓힌 커스텀 PCB 모듈 설계 및 시제품 제작

상세: [experiences/paperware.md](experiences/paperware.md)

### Crypto LOB Prediction

**Python / Deep Learning / CNN / Mamba / Time-series**

- 거래소 호가와 체결 데이터를 수집하고 전처리
- CNN과 Mamba를 결합한 시계열 분류 모델 실험
- 5초와 15초 뒤 가격 방향 예측
- 분류 정확도뿐 아니라 거래 수수료를 반영한 백테스트로 실제 활용 가능성 검토

상세: [experiences/crypto-lob-prediction.md](experiences/crypto-lob-prediction.md)

### HCI Stock Information Service

**Web / SQLite / Database Design / HCI**

- 한국 증권사 리포트를 기반으로 목표가, 투자의견, 정확도를 정리하는 서비스 기획
- 초보 투자자의 정보 탐색 흐름을 고려해 화면과 기능 설계
- CSV 데이터를 SQLite에 저장하고 3정규형 구조로 설계

상세: [experiences/hci-stock-service.md](experiences/hci-stock-service.md)

## Computer Vision

- 디지털 이미지와 노이즈, convolution/filtering, gradient와 edge detection
- binary image analysis, segmentation, clustering, feature matching
- image warping/stitching, object recognition
- Gaussian filter, Canny, separable filter, Manhattan distance transform, Chamfer Matching, Hough Transform, RANSAC 직접 실습
- 같은 알고리즘도 촬영 조건과 파라미터에 따라 결과가 달라짐을 확인하고 원본, 중간 처리 결과, 최종 결과를 함께 비교하는 습관을 형성

상세: [experiences/computer-vision.md](experiences/computer-vision.md)

## Additional Making Experience

- Arduino 센서와 액추에이터 제어
- 브레드보드 Arduino 제작 및 PCB 설계 자습
- Raspberry Pi Linux 환경과 CCTV 프로젝트
- Arduino 드론 제작 및 코딩
- micro:bit 기반 드론 제작 시도
- Arduino 기반 스마트 쓰레기통 프로젝트

전체 흐름: [experiences/career-timeline.md](experiences/career-timeline.md)

## Awards

- **2024.11.29** 제22회 임베디드 소프트웨어 경진대회 webOS 부문 입선
- **2022.02.10** 제1회 전국 장애·비장애 대학생 창업경진대회 대상
- **2021.11.13** 제6회 글로벌 이노베이터 페스타 메이커톤 KT 트랙 대상 — 부총리 겸 교육부장관
- **2019.12.13** 국제로봇콘테스트 WCRC micro:bit 창작 대학일반 부문 금상 — 산업통상자원부장관
- **2019.11.14** 대한민국 마이스터대전 WCRC 2차본선 micro:bit 창작 대학일반 부문 1위(금상) — 과학기술정보통신부장관

## Training

- **2023.07.14 ~ 2023.08.03** 제품 기능설계 MVP 실전캠프, 60시간 — 대구창조경제혁신센터

---

이 문서는 모든 경험을 보존하는 마스터 이력서입니다. 실제 지원 시에는 직무와 공고에 맞춰 관련 경험만 남기고 1~2페이지 수준으로 압축합니다.
