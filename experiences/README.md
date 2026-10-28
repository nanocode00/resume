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
| [Beamworks](beamworks.md) | 요구사항, Jira, ERD, DB, DTO | 실무 경험, 협업, 설계 |
| [Paperware](paperware.md) | micro:bit, PCB, 메이커, 사용성 | 창의성, 개선, HW 제작 |
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
