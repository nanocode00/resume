# 김재훈 — Resume

> Android/CV 제품 출시와 운영, Verilog/OpenROAD HW 검증, STT/NLU/TTS 모델 실험과 배포까지 경험한 SW/HW 융합 개발자

## Profile

- 경북대학교 컴퓨터학부
- 전자공학 부전공
- 관심 분야: Software, Computer Vision, Embedded/IoT, AI, HW/SW Co-design
- GitHub: https://github.com/nanocode00

## Summary

- **HummingBlocks 제품화**: 공동 창업자·Mobile App Developer로서 실물 코딩 블록의 `Camera → YOLO/OpenCV → 배치 해석 → 음악 실행` Android 파이프라인을 구현하고 Google Play 출시와 2.x 유지보수까지 수행했습니다. Java/TFLite 경로를 별도 브랜치에서 구현해 처리 시간을 **9.1s → 5.4s(-40.7%)**로 줄였지만 정확도가 **90% → 57%(-33%p)**로 하락해 제품 신뢰성을 기준으로 기존 runtime을 유지했습니다.
- **CNN Accelerator HW/SW 검증**: 3인 팀에서 기존 공개 CNN RTL을 PyTorch Quantization 결과와 정합하고 synthesis-friendly RTL로 수정. ModelSim에서 **MNIST 1,000장 기준 RTL accuracy 96%**를 확인하고 OpenROAD ASAP7 synthesis와 physical design final stage까지 진행했습니다. Netlist 정상 파형 확보와 timing closure는 완료하지 못했습니다.
- **Mallo Voice AI 통합**: 3인 팀 Team Lead / Product & Integration으로 PRD/TRD, Order Engine, Senior-first UI, TTS 파인튜닝과 STT/NLU 추가 학습·검증, runtime/deployment를 연결했습니다. 동일 20문장 GPU 비교에서 **Melo Base 198.5ms, Melo Friendly 204.1ms, Qwen Base 6.91s, Qwen Friendly 28.20s**를 측정해 MeloTTS Friendly를 선택했고, 최종 production TTS는 **mean 0.38s / p95 0.53s / RTF 0.0894**를 기록했습니다.
- **수상**: 2019 과학기술정보통신부장관상·산업통상자원부장관상, 2021 부총리 겸 교육부장관상, 2022 전국 장애·비장애 대학생 창업경진대회 대상, 2024 임베디드 SW 경진대회 webOS 부문 입선.

> HummingBlocks의 9.1s/5.4s 및 90%/57%는 당시 측정값을 이후 경험기술서에 기록한 수치입니다. Git에는 TFLite 전환 구현과 롤백이 남아 있으나 원본 benchmark spreadsheet는 현재 저장소/Drive 검색에서 확인하지 못했습니다.

## Experience

### Beamworks — 현장실습

**2026.01 · 4주 · 사내 업무관리 플랫폼 요구사항/데이터 설계**

- Figma 화면에서 입력·조회·저장 값을 분리해 요구사항을 분석하고 Jira Epic/Story/Task로 개발 단위를 구조화
- ERD와 DB 테이블, SQL, DTO를 작성하며 화면 요구사항을 데이터 구조와 구현 단위로 변환
- 의료 AI 기업에서 수행한 인턴십이지만 담당 업무는 의료영상이 아닌 사내 업무관리 플랫폼

상세: [experiences/beamworks.md](experiences/beamworks.md)

## Main Projects

### HummingBlocks — AI 음악코딩 Android 제품

**2022.03 ~ 2023.10 주요 제품 개발, 2025.01까지 유지보수 · 공동 창업자 / Mobile App Developer**  
**Android / Java / Python / YOLO / OpenCV / Chaquopy / TensorFlow Lite / CameraX / FFmpeg**

- 실물 코딩 블록을 촬영하면 객체의 종류와 배치를 인식해 음악으로 실행하는 Android 제품에서 Camera/CV 연동, 프로그램 해석, 음악 실행을 구현하고 **2023.10 Google Play 출시** 후 2.x까지 유지보수했습니다.
- detection box를 좌표 기준으로 정렬·그룹화해 PLAY/LOOP/STAR, 조건, 악기 단계 연산으로 변환하고, **오류 23종·경고 2종**을 실행 전에 검사해 수정 가능한 안내로 연결했습니다.
- 별도 브랜치에서 Python bridge를 제거한 Java/TFLite inference, NNAPI/GPU/CPU fallback, NMS·좌표 후처리를 구현해 Galaxy S9+ 기준 **9.1s → 5.4s(-40.7%)**로 단축했습니다. 다만 정확도가 **90% → 57%(-33%p)**로 하락해 오인식 비용을 기준으로 기존 Chaquopy/OpenCV DNN 경로를 유지했습니다.
- 출시 후 ScheduledThreadPool 기반 음악 재생·pause/resume와 BPM, FFmpeg MP3·영상 export, TalkBack·촬영 방향 안내, 진행도·버전 migration을 추가했습니다.

제품 후속 누적 성과: 2026.05 회사 사업계획서 기준 **앱 다운로드 1,500+, 키트 1,830개, 자사 판매 약 80개 학교·제품 도달 약 200개 학교**. 개인 개발 기간의 직접 판매 실적과는 구분합니다.  
수상: 2022 전국 장애·비장애 대학생 창업경진대회 대상, KNU 창업경진대회 대상, 소셜벤처 경연대회 TS청년벤처상·대구광역시장상.

Google Play: https://play.google.com/store/apps/details?id=com.nemo.hummingblocks  
상세: [experiences/hummingblocks.md](experiences/hummingblocks.md)

### CNN Accelerator — PyTorch Quantization에서 OpenROAD까지

**2025.06 · 3인 팀 · Quantization/HW reference, synthesis-oriented RTL 수정, ModelSim/OpenROAD 검증**  
**PyTorch / Verilog / ModelSim / OpenROAD / ASAP7**

- **상황**: 기존 공개 2-layer MNIST CNN RTL을 기준으로 SW 모델의 정수 표현부터 ASIC flow까지 연결하는 논리회로설계 기말 프로젝트를 진행했습니다.
- **문제**: PyTorch quantized tensor와 RTL의 fixed-point 표현을 맞춰야 했고, simulation용 RTL 일부는 그대로는 synthesis하기 어려웠습니다.
- **판단**: 최종 class만 비교하지 않고 layer 출력과 channel별 MAC 결과를 SW reference로 저장해 불일치 시작 지점을 좁히고, fixed parameter wiring은 synthesis-friendly 정적 연결로 바꾸기로 했습니다.
- **조치**: `int_repr()`와 scale을 이용해 weight/bias와 중간 출력을 HW용 정수/hex 데이터로 변환하고, Conv/FC의 procedural unpacking을 `generate` + continuous `assign` 구조로 리팩터링했습니다. RTL 저장소를 Git submodule로 OpenROAD Flow Scripts의 ASAP7 design source에 연결했습니다.
- **결과**: ModelSim에서 **MNIST 1,000장 RTL accuracy 96%**를 확인했고 synthesized Verilog와 physical design final output까지 생성했습니다. 다만 gate-level simulation에서는 올바른 파형을 확보하지 못해 Netlist accuracy를 측정하지 못했고, final timing도 **VIOLATED** 상태로 timing closure는 달성하지 못했습니다.

> 최종 PPA/TOPS/W 값과 post-synthesis accuracy는 확보하지 못했으므로 성과 수치로 사용하지 않습니다.

상세: [experiences/cnn-accelerator.md](experiences/cnn-accelerator.md)

### Mallo — Senior-Friendly Voice Ordering Kiosk

**2026.06 ~ 2026.08 · 3인 팀 · Team Lead / Product & Integration**  
**Whisper / Qwen / MeloTTS / QLoRA / FastAPI / Vercel / Cloudflare**

- **상황**: 화면 탐색과 옵션 선택에 익숙하지 않은 사용자가 음성과 화면을 함께 사용해 주문을 끝까지 완료할 수 있는 voice-first kiosk를 개발했습니다.
- **문제**: STT/NLU/TTS를 각각 개선해도 model latency, dependency 충돌, 주문 상태의 안정성, 복잡한 UI가 동시에 제품 경험을 제한했습니다. TTS 후보도 품질과 생성 시간이 크게 달랐습니다.
- **판단**: AI 출력과 주문 상태 변경을 분리해 deterministic Order Engine을 두고, 모델은 독립 resident sidecar로 운영하며 정확도뿐 아니라 실제 latency와 사용 흐름을 함께 기준으로 선택했습니다. TTS는 고령 사용자 친화적 억양을 확보하면서 interactive latency를 만족하는 모델을 우선했습니다.
- **조치**: PRD/TRD와 공통 command contract, Order Engine, Senior-first UI를 구성하고 AI Hub 친절 발화로 MeloTTS를 fine-tuning했습니다. 동일 20문장 GPU benchmark에서 **Melo Base 0.1985s / Melo Friendly 0.2041s / Qwen Base 6.9075s / Qwen Friendly 28.1953s**를 비교해 MeloTTS Friendly를 선택했습니다. 프로젝트 후반에는 Whisper Medium QLoRA 후보 sweep과 Qwen3-1.7B QLoRA 3-epoch 학습도 직접 수행했으며, Gateway/STT/NLU/TTS를 `8000/8001/8002/8003` sidecar 구조로 통합했습니다.
- **결과**: Fine-tuned MeloTTS는 baseline보다 평균 생성 시간이 약 **5.6ms(+2.8%)** 늘었지만 친절체 억양을 얻으면서 0.2초 수준의 interactive latency를 유지했습니다. 이후 `G_2800` production checkpoint 기준 benchmark는 **mean 0.38s / p95 0.53s / RTF 0.0894**였습니다. Vercel frontend + Cloudflare Tunnel + local FastAPI runtime으로 배포했고, real STT+NLU와 MockTTS 기반 50-turn isolated E2E는 **p95 312.2ms**로 STT를 주 bottleneck으로 확인했습니다. 2026년 8월 기능 동결과 repository cleanup까지 마치고 프로젝트를 종료했습니다.

상세: [experiences/mallo.md](experiences/mallo.md)

## Additional Projects

### Verilog Single-cycle CPU

**2025.05 · 개인 RTL 구현/검증 · Verilog / ModelSim**  
기존 Logisim MIPS 구조를 분석해 32-bit Single-cycle CPU를 Verilog로 재구현하고 Decoder, ALU, Register File, Memory, CP0, Syscall을 통합했습니다. Syscall은 `$v0/$a0` 기반 Hex/Halt 흐름으로, CP0는 **외부 `ExpSrc` exception → EPC 저장 → 0x800 → ERET 복귀**로 분리해 구현하고 모듈별 testbench/waveform으로 검증했습니다.  
상세: [experiences/verilog-cpu.md](experiences/verilog-cpu.md)

### webOS Smart Planter

**2024.03 ~ 2024.07 · 5인 팀 · Arduino/I2C firmware 및 webOS HW integration**  
Arduino I2C slave firmware와 10-byte 센서 protocol을 구현하고 webOS Peripheral Manager Luna API로 실제 센서값, NeoPixel, pump 제어를 JS Service에 연결했습니다. 개발 중반 Uno R3에서 검증한 뒤 최종형을 Nano로 이식했으며, **2024 임베디드 SW 경진대회 webOS 부문 입선**을 수상했습니다.  
상세: [experiences/webos-smart-planter.md](experiences/webos-smart-planter.md)

### Crypto Microstructure Prediction — MAMBA Trading

**2025.10 ~ 2025.12 · 모델 개발 4인 · MAMBA 담당 · Python / Mamba / Time-series**  
Binance BTCUSDT tick 데이터를 50-trade sequence로 구성해 4-layer Mamba로 5초 뒤 방향을 예측했습니다. 무수수료 backtest는 **+23.26%**였지만 Spot 0.1%/side 수수료를 적용하면 약 **-99.18%**로 붕괴해, high-turnover 전략에서는 분류 성능보다 transaction cost와 trade edge가 중요하다는 점을 확인했습니다.  
상세: [experiences/crypto-lob-prediction.md](experiences/crypto-lob-prediction.md)

### Paperware

**HummingBlocks 이전 네모감성 메이커 프로젝트 · micro:bit / PCB Design**  
카드보드와 구리박 테이프 기반 교육 키트에서 납땜이 필요한 NeoPixel 모듈을 문제로 보고, 넓은 접점으로 바로 연결할 수 있는 커스텀 PCB를 설계해 시제품과 카드보드 무드등에 적용했습니다.  
상세: [experiences/paperware.md](experiences/paperware.md)

### HCI Stock Information Service

**대학 4학년 HCI 프로젝트 · Web / SQLite / Database Design**  
증권사 리포트의 목표가·투자의견·과거 정확도를 초보 투자자가 탐색할 수 있도록 정보 구조와 화면 흐름을 설계하고 CSV 데이터를 SQLite 3정규형 구조로 구성했습니다.  
상세: [experiences/hci-stock-service.md](experiences/hci-stock-service.md)

## Computer Vision Coursework

- Gaussian filter, Canny, separable filter, Manhattan distance transform, Chamfer Matching, Hough Transform, RANSAC 등을 직접 구현·실습
- 촬영 조건과 parameter 변화에 따라 결과가 달라지는 문제를 원본·중간 처리 결과·최종 결과로 분리해 비교

상세: [experiences/computer-vision.md](experiences/computer-vision.md)

## Awards

- **2024.11.29** 제22회 임베디드 소프트웨어 경진대회 webOS 부문 입선
- **2022.02.10** 제1회 전국 장애·비장애 대학생 창업경진대회 대상
- **2021.11.13** 제6회 글로벌 이노베이터 페스타 메이커톤 KT 트랙 대상 — 부총리 겸 교육부장관
- **2019.12.13** 국제로봇콘테스트 WCRC micro:bit 창작 대학일반 부문 금상 — 산업통상자원부장관
- **2019.11.14** 대한민국 마이스터대전 WCRC 2차본선 micro:bit 창작 대학일반 부문 1위(금상) — 과학기술정보통신부장관

## Training

- **2023.07.14 ~ 2023.08.03** 제품 기능설계 MVP 실전캠프, 60시간 — 대구창조경제혁신센터

---

이 문서는 경험을 보존하는 마스터 이력서이면서, 상단과 주력 프로젝트는 채용 담당자가 핵심 근거를 빠르게 확인할 수 있도록 구성합니다. 세부 구현과 source provenance는 `experiences/` 문서에서 관리하고, 실제 지원 시에는 공고에 맞춰 1~2페이지로 압축합니다.
