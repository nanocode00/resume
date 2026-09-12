# Resume Archive

김재훈의 이력서, 프로젝트 경험, 자기소개서 작성 기준과 지원 기록을 관리하는 저장소입니다.

## 구조

```text
.
├── README.md
├── resume.md                         # 통합 이력서 원본
├── experiences/                      # 경험별 상세 기록
│   ├── README.md
│   ├── career-timeline.md
│   ├── mallo.md
│   ├── hummingblocks.md
│   ├── cnn-accelerator.md
│   ├── verilog-cpu.md
│   ├── webos-smart-planter.md
│   ├── beamworks.md
│   ├── paperware.md
│   ├── crypto-lob-prediction.md
│   ├── hci-stock-service.md
│   └── computer-vision.md
├── guides/
│   └── cover-letter-writing.md       # 자소서 작성 원칙
└── cover-letters/
    └── README.md                      # 기존 자소서 아카이브 인덱스
```

## 관리 원칙

- `resume.md`는 지원 직무에 맞춰 내용을 덜어내기 전의 **마스터 이력서**로 관리합니다.
- 프로젝트의 세부 사실, 문제 상황, 선택 이유, 수치와 성과는 `experiences/`에 보존합니다.
- `experiences/career-timeline.md`에는 이력서에서 빠진 학습과 제작 경험도 연대기 형태로 남깁니다.
- 자소서를 쓸 때는 `experiences/`에서 필요한 경험을 골라 사용하고, 작성 규칙은 `guides/cover-letter-writing.md`를 따릅니다.
- 실제 제출했던 자소서는 회사별로 `cover-letters/`에 보관합니다.
- 공개 저장소이므로 전화번호, 개인 이메일, 주소 등 불필요한 개인정보는 커밋하지 않습니다.

## 현재 대표 경험

- Mallo: 고령 사용자 대상 Voice-first Kiosk, 팀장, 음성 AI 파이프라인 및 TTS 파인튜닝
- HummingBlocks: 시각장애인을 위한 코딩교육 Android 앱, Google Play 출시 및 블록 키트 판매
- CNN Accelerator: PyTorch 양자화 → Verilog 검증 → OpenROAD/ASAP7 합성 및 정확도 검증
- Verilog CPU: 단일 사이클 CPU와 CP0 예외 처리 구현
- webOS Smart Planter: Raspberry Pi, Arduino, React, AWS를 연결한 스마트 화분
- Beamworks Internship: 요구사항 분석, Jira 업무 구조화, ERD/DB/SQL/DTO 설계
- Paperware: micro:bit 교육용 메이커 키트와 커스텀 PCB 모듈 설계
- Crypto LOB Prediction: 거래소 호가 데이터 기반 CNN+Mamba 시계열 분류 실험
- HCI Stock Service: 증권사 리포트 기반 초보 투자자용 정보 탐색 서비스 개선

> 이 저장소는 완성된 제출용 문서만 두는 곳이 아니라, 지원 직무별 이력서와 자소서를 빠르게 만들 수 있도록 경험 원본을 축적하는 작업 저장소입니다.
