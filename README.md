# 김재훈 | Software Engineer Portfolio

**사용자에게 닿는 소프트웨어를 만들고, 시스템의 동작과 한계를 검증하는 개발자입니다.**

경북대학교 컴퓨터학부에서 소프트웨어와 임베디드 시스템을 공부하며 Android 제품 출시, 음성 AI 서비스 통합, Verilog RTL 구현 및 검증, AI Agent 평가 프로젝트에 참여했습니다. 기능 구현뿐 아니라 **실제 사용 흐름, 오류 원인, 검증 결과**를 중요하게 생각합니다.

**관심 분야:** Software Engineering · AI Systems · Embedded / HW–SW Integration · Computer Vision  
**바로가기:** [상세 이력서](resume.md) · [프로젝트별 경험·근거](experiences/README.md) · [GitHub 프로필](https://github.com/nanocode00)

## 대표 프로젝트

먼저 프로젝트의 **목적 → 본인 역할 → 실제 결과**를 확인할 수 있도록 요약했습니다. 코드·포트폴리오 저장소의 팀 구현과 개인 기여는 구분하여 작성했습니다. Mallo와 말해봄은 현재 공개 설명 문서를 먼저 제공하며, 공개 가능한 소스는 검수 후 추가합니다.

| 프로젝트 | 무엇을 만들었나 | 개인 역할 및 확인된 결과 | 코드 · 상세 |
|---|---|---|---|
| **HummingBlocks**<br>Android · CV · Accessibility | 실물 코딩 블록을 카메라로 인식해 음악을 만드는 시각장애인 대상 코딩교육 앱 | **공동 창업자 / Android 개발.** 카메라→Python/OpenCV→실행→재생 통합, TalkBack·자동촬영 UX, 음악 저장·FFmpeg 처리. **2023년 Google Play 출시**, 이후 유지보수 | [코드](https://github.com/nanocode00/nemo_codeblock) · [경험](experiences/hummingblocks.md) · [Google Play](https://play.google.com/store/apps/details?id=com.nemo.hummingblocks) |
| **Verilog Single-cycle CPU**<br>Verilog · ModelSim | 기존 Logisim MIPS CPU를 참고해 32비트 단일 사이클 CPU를 RTL로 재구현 | **개인 RTL 구현·테스트벤치 작성.** ALU·RegFile·Memory·제어부 통합. CP0 예외/복귀 경로를 시도했으나 **EPC 저장 문제가 해결되지 않은 상태로 종료** | [코드: modelsim/](https://github.com/nanocode00/MIPS-CPU/tree/main/modelsim) · [경험](experiences/verilog-cpu.md) |
| **Mallo (말로)**<br>Whisper · Qwen · MeloTTS | 고령 사용자 친화적인 음성 주문 키오스크 | **3인 팀장 / 제품·통합·모델 실험.** 주문 상태 엔진과 모델 서비스 통합, TTS 파인튜닝·비교. TTS production benchmark **p95 0.53초**; 최종 Real E2E **p95 3.338초**로 1.5초 요구사항에는 미달 | [공개 포트폴리오](https://github.com/nanocode00/mallo-portfolio) · [경험](experiences/mallo.md) · [소개](https://nanocode00.github.io/mallo-intro/) |
| **말해봄 Mk.2**<br>FastMCP · LLM · Whisper · CI | 음성 답변에서 근거를 확인하고 개선을 지원하는 AI 면접 코칭 서비스 | **3인 팀 / 모델 평가·STT 실험·에이전트 검증·CI.** LLM 22종×30건 합성 평가, Whisper QLoRA 도메인 적응, 저장·검증 경로 테스트 및 품질 게이트 개선. 팀 전체 Agent 구현과 개인 기여를 구분 | [공개 포트폴리오](https://github.com/nanocode00/malhaebom-portfolio) · [경험](experiences/malhaebom.md) · [서비스 소개](https://malhaebom.xyz/) |

> **기여·검증 범위:** Fork 저장소에 있는 원작 코드나 팀 전체 구현을 개인 단독 결과로 주장하지 않습니다. 숫자는 해당 문서의 실험 조건과 한계를 함께 확인해 주세요. 상세한 근거는 각 **경험** 링크에 정리했습니다.

## 추가 프로젝트

| 분야 | 프로젝트 | 핵심 기여·결과 | 코드 · 상세 |
|---|---|---|---|
| HW/SW Co-design | **CNN Accelerator** | PyTorch 양자화 정합, Verilog 합성 친화적 RTL 수정, ModelSim **MNIST 1,000장 RTL 정확도 96%**. OpenROAD 진행했으나 최종 timing closure는 미달 | [RTL](https://github.com/nanocode00/cnn_verilog) · [경험](experiences/cnn-accelerator.md) |
| Embedded / IoT | **webOS Smart Planter** | Arduino I2C 센서·펌프·LED 제어 펌웨어와 webOS 연동. 2024 임베디드 SW 경진대회 webOS 부문 입선 | [팀 코드](https://github.com/dudgns128/webos-gardening) · [경험](experiences/webos-smart-planter.md) |
| Data / Backend | **HCI Stock Service** | 초보 투자자를 위한 증권사 리포트 탐색 서비스에서 SQLite/SQLAlchemy DB 및 검색 쿼리 구현 | [팀 코드](https://github.com/nanocode00/hci) · [경험](experiences/hci-stock-service.md) |
| ML / Time Series | **Crypto Mamba** | 거래 데이터 전처리·Mamba 예측·백테스트. 수수료 적용 시 손실로 전환되는 한계를 검증 | [경험](experiences/crypto-lob-prediction.md) |
| Maker / PCB | **Paperware** | micro:bit 교육 키트, NeoPixel PCB 설계·제작 및 실제 무드등 적용 | [경험](experiences/paperware.md) |
| Computer Vision | **영상처리 과제** | Canny·Hough·RANSAC 등 영상처리 기법 구현 및 조건별 비교 | [경험](experiences/computer-vision.md) |

## 실무 경험 및 수상

- **Beamworks 현장실습 (2026.01):** 병원 CRM DealFlow의 Figma 목업을 분석해 ERD·SQL·DTO 설계, Jira 작업 구조화. [상세](experiences/beamworks.md)
- **2024:** 제22회 임베디드 SW 경진대회 webOS 부문 입선 — Smart Planter
- **2022:** 전국 장애·비장애 대학생 창업경진대회 대상 — HummingBlocks
- **2019:** WCRC micro:bit 창작 관련 금상 2회 — Paperware 계열 활동

## 자세히 보기

- **[resume.md](resume.md)** — 역할, 성과, 한계를 포함한 마스터 이력서
- **[experiences/README.md](experiences/README.md)** — 프로젝트별 경험 문서와 근거·검증 자료
- **[career-timeline.md](experiences/career-timeline.md)** — 학습·프로젝트·활동 연대기

이 저장소는 **포트폴리오 탐색 입구와 이력서 원본을 함께 제공**합니다. 실험 로그와 상세한 사실 확인 자료는 `experiences/`에 보존하되, 첫 방문자는 이 페이지의 프로젝트 링크만으로 관심 분야를 따라갈 수 있도록 구성했습니다.
