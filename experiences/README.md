# Experience Database

이 폴더는 이력서와 자기소개서에 사용할 **경험 원본 데이터베이스**입니다.

각 파일은 단순 프로젝트 소개보다 다음 정보를 보존하는 데 목적이 있습니다.

1. 무엇을 만들었는가
2. 왜 그 문제가 중요했는가
3. 본인이 맡은 역할은 무엇인가
4. 어떤 문제가 발생했는가
5. 어떻게 원인을 좁히고 선택했는가
6. 결과와 수치가 무엇인가
7. 어떤 역량을 보여주는 경험인가
8. 어떤 자소서 문항에 활용할 수 있는가

## 경험 목록

| 경험 | 핵심 키워드 | 활용하기 좋은 문항 |
|---|---|---|
| [Mallo](mallo.md) | Voice-first Kiosk, OrderCommand/Cart SSOT, Whisper/Qwen/Melo, 4-프로세스 추론, Senior-first UX·Real E2E | AI 시스템/백엔드·ML 모델 검증·리딩·HCI |
| [말해봄 Mk.2](malhaebom.md) | FastMCP/Agent 검증, OpenRouter 모델 평가, Whisper QLoRA, JWT 공고 E2E, PR·CI 품질 게이트 | AI Agent·백엔드, 모델 평가, 시스템 신뢰성, 기술 의사결정 |
| [HummingBlocks](hummingblocks.md) | Android, CV, 제품 출시, 사용자 경험, 문제 해결 | 대표 프로젝트, 어려움 극복, 직무 역량, 협업 |
| [CNN Accelerator](cnn-accelerator.md) | PyTorch, Verilog, 검증, OpenROAD | 기술 역량, 디버깅, HW/SW 융합 |
| [Verilog CPU](verilog-cpu.md) | CPU, CP0, 예외 처리, 파형 디버깅 | 문제 해결, 기본기, 디버깅 |
| [webOS Smart Planter](webos-smart-planter.md) | Raspberry Pi, Arduino, React, 시스템 통합 | 협업, 임베디드, 시스템 설계 |
| [Beamworks](beamworks.md) | 병원 CRM DealFlow: 제품 전체 Figma 목업 분석 → ERD → SQL → DTO 작성, Jira 작업 관리(기억 기반) | B2B 도메인 분석, DB·DTO 설계, 실무 협업 |
| [고교 ‘찰칵!’ 도난 방지 시스템](highschool-theft-prevention-camera.md) | 교실 도난 문제 → 초음파 문 열림 감지·Raspberry Pi 사진 촬영 모형, Bluetooth 연결 검토 | 문제 발굴, 센서·카메라 결합, 팀 리딩, 시제품 제작 |
| [Raspberry Pi CCTV (2019)](raspberry-pi-cctv-2019.md) | Python 카메라 영상 저장 → 네트워크 연결 → 녹화 영상 웹 조회, 영상인식 목표 미완료 | 범위 조정, Linux/Web 연동, 실패·대응 경험 |
| [Paperware](paperware.md) | 2019 WCRC 기존 Paperware 출품·직접 MakeCode 구현, 2021경 NeoPixel 커스텀 PCB | 메이커 제작, 임베디드 SW, PCB 개선 |
| [Crypto LOB Prediction](crypto-lob-prediction.md) | CNN, Mamba, 시계열, 백테스트 | AI, 데이터, 실패/검증 경험 |
| [HCI Stock Service](hci-stock-service.md) | 5-table SQLite/SQLAlchemy, CSV 적재·조회, 18명 사용자 비교 평가(팀 성과) | 데이터 모델링, Backend, 사용자 중심 검증 |
| [Computer Vision](computer-vision.md) | 2025 개인 과제 1·2: Threshold/Canny 27조건, Manhattan/Chamfer, Hough, RANSAC, Clustering | CV 구현 기본기, 실험·실패 분석, 영상처리 직무 |
| [Career Timeline](career-timeline.md) | 학습/활동 연대기 | 성장 과정, 경험 발굴, 면접 준비 |

## 작성 원칙

- 기억이 불확실한 수치나 기술은 사실처럼 확정하지 않습니다.
- 역할을 팀 전체 성과와 개인 기여로 구분합니다.
- 결과만 적지 않고 판단 기준과 문제 해결 과정을 함께 남깁니다.
- 같은 경험도 지원 직무에 따라 다른 부분을 강조할 수 있도록 세부 내용을 충분히 보존합니다.
- 말로는 [12개 개인/팀 기여·수치/모집단 교차검증 원장](mallo-rework-checkpoint.md)과 [AI 시스템/백엔드 핵심 사례 5선](mallo.md#채용용-핵심-사례-5선--ai-시스템백엔드-기준)을 구분합니다. 60대 초기 탐색 설문 13명과 미실행 최종 사용자 평가는 별개이며, Real E2E p95 3.338초는 요구 1.5초 미달이라는 한계를 포함합니다.
- 말해봄은 [12개 작업군·대화 누락 감사·귀속/중복 교차검증 원장](malhaebom-rework-checkpoint.md)과 [AI Agent/Backend 우선 사례 5개](malhaebom.md#채용용-핵심-사례-5선--ai-agentbackend-기준)를 구분해 활용합니다. 검증 완료는 실제 운영 전체 E2E·마감 제출·독립 사람 품질 평가를 뜻하지 않습니다.

- HCI 주식정보 서비스는 [개인 DB/쿼리 Git 커밋·팀 설문 A/B 검증 원장](hci-stock-service-evidence-ledger.md)을 근거로 작성했습니다. 18명 팀 사용자 평가와 개인 구현을 분리하며, 목표가 과거 적중률·Gemini 전체 재현·현재 서비스 운영은 검증 완료로 간주하지 않습니다. 원본 CSV 12/6과 발표 13/5의 일부 설문 집계 차이도 원장에 명시했습니다.

- 컴퓨터비전은 [개인 과제 1·2 제출 자료와 과제 2 배포 Notebook 비교 원장](computer-vision-evidence-ledger.md)을 기준으로 정리했습니다. OpenCV/scikit-learn을 사용한 부분과 배포된 Notebook 빈 부분에 직접 구현한 알고리즘을 분리하고, 수업 입력의 시각적 결과를 범용 정확도 성과로 확대하지 않습니다.

- Beamworks(빔웍스)는 [기억 근거·불확실성 원장](beamworks-memory-ledger.md)을 별도로 두고, **DealFlow가 병원 고객관리(CRM) 플랫폼이며 병원·병원 내 담당자·데모·초음파 기기·VOC 정보를 관리한다는 2026-10-29 직접 회고**를 구분해 기록했습니다. **제품 전체 Figma 목업의 데이터 구조 분석과 ERD→SQL→DTO 직접 작성, Jira 작업 관리**는 2026-10-29 본인 직접 회고로 확인된 개인 설계 범위입니다. 단, 제품 전체 설계와 런타임 기능 전체 구현을 동일시하지 않으며 기존 기록과 회고에 의존합니다. 원본 Figma/Jira/DB/코드/검수 자료는 없어 현재 문서는 **기억 기반 초안**이며 수치·배포·의료영상 AI 개발 성과를 임의로 추가하지 않습니다.

- 2019 [Raspberry Pi CCTV](raspberry-pi-cctv-2019.md)는 [제안→중간→최종 발표와 Python·HTML 코드 근거 원장](raspberry-pi-cctv-2019-evidence-ledger.md)으로 귀속을 분리했습니다. **학교 쉬는 시간 절도 문제를 계기로 만든 고교 ‘찰칵!’ 카메라·센서 시스템은 별도 프로젝트**임을 2026-10-29 사용자 회고와 당시 [팀 발표·기술 원장](highschool-theft-prevention-evidence-ledger.md)으로 확인했습니다. [고교 경험 문서](highschool-theft-prevention-camera.md)에 **팀장·프로그래머·엔지니어**, **초음파 센서로 문 열림 감지 → 사진 촬영 모형**, **Bluetooth 검토**, **미설치·교내 수상명 미확정**을 분리했습니다. **영상인식 목표는 미구현**이며, 개인 웹서버 업무 완료와 팀의 PIR/챗봇/다운로드 기능을 혼동하지 않습니다. 보존 코드·시연 영상의 존재만으로 전체 개인 개발 및 실제 외부 서비스 운영을 주장하지 않습니다.

- 2019 WCRC micro:bit 대회 출품은 **진행 중이던 Paperware 프로젝트의 연장**이라는 2026-10-29 본인 회고를 [Paperware](paperware.md)와 [당시 MakeCode 기술 원장](linkcube-2019.md)에 반영했습니다. 금상 수상 기록은 유지하되 객관적 경쟁률을 모르는 상태에서 치열한 경쟁을 통한 기술 우수성 입증으로 해석하지 않습니다. 후속 2021년경 PCB 개선은 별도의 개발 단계로 구분합니다.
