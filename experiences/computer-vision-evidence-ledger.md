# Computer Vision 2025 과제 소스·결과 교차검증 원장

> **2026-10-28 검토.** 연결된 Google Drive의 2025년 가을 Computer Vision 폴더에 보관된 문제지, 보고서, 배포·제출 ZIP을 대조했다. 자료들은 수업 과제용으로 보관된 **코드/실험 결과**이다. 설문·성적·교수 평가 및 Git 커밋 이력이 아니므로 독립 검증을 과장하지 않는다. Drive 링크·학번·개인 신상·과제 바이너리를 공개 저장소에 복제하지 않는다.

## A. 자료 인벤토리

| 자료(개인 Drive 보관) | 확인 범위 | 성격 |
|---|---|---|
| **CV-2025-fall-Assignment01-QUESTION.pdf** | 이미지 threshold·denoise·Canny·filter separability와 이론 문항, 2025-10-17 제출 기한 | 과제 명세, **개인 구현 증거 아님** |
| **Computer Vision Assignment 1.pdf** | 본인 이름이 있는 개인 답안 보고서. 파라미터·이미지·결과·한계 기록 | 개인 결과 보고서 |
| **Computer Vision Assignment 1.zip** | Python 4개 및 PNG 출력·과제 제공 이미지, 총 73개 파일 | 코드와 출력 아티팩트 묶음 |
| **CV-2025-fall-Assignment02-QUESTION.pdf** | Chamfer(30), Clustering(30), Hough(30), RANSAC(30) 과제 지시, 2025-12-02 제출 기한 | 과제 명세 |
| **CV-2025-fall-Assignment02.zip** | four-topic Jupyter scaffold 제공본 | 배포 자료; 작성자 불명/교수자 제공 |
| **CV-2025-fall-Assignment02-[개인 식별번호].zip** | 네 Notebook을 직접 변경한 제출본, 약 2.73 MB | 과제 제출 아카이브; 원본과 diff 가능 |
| **ComputerVision-00~11** 강의 PDF | 각 알고리즘의 이론·관련 범위 | 강의 교재; 개인 직접 구현 근거가 아님 |

검토 기준은 제공본/제출본의 **Notebook cell source 차이**와 제출본의 **저장된 실행 출력**, 과제 1 보고서와 실제 네 Python 파일의 일치 여부이다. Mac 메타데이터, 공급된 이미지와 설명, 배포용 utility까지 개인 작성량에 포함하지 않는다. 이 폴더 밖의 제출물·평가 결과는 조사 범위에 포함하지 않았다.

## B. 과제 1 — 파일별 코드·답안 대조

### CV1-01 — 이진·윈도우 threshold
**소스:** 1_thresholding/1_thresholding.py 및 숫자 영상 PNG 출력.

- 단일 이진·반전 threshold **T=55/129/200**, 세 구간 window threshold **55~90 / 90~129 / 129~200** 구현.
- **125~134**를 1 단위로 비교하고, **T1=128/129 / T2=130/131/132** 6개 조합을 생성.
- **보고서의 직접 관찰:** 숫자 3 분리는 **129~130**에서 원형 잔여가 가장 얇지만 완전 제거 실패. 픽셀 강도 조건만으로 둘을 분리하는 데 한계가 있다고 기술.
- **검증 경계:** 이 결과는 해당 **digits.png**에 국한하며 segment 정확도·정답 마스크 수치가 없다.

### CV1-02 — 평균·Gaussian denoise
**소스:** 2_denoising/2_denoising.py 및 평균 4장·Gaussian 4장 출력.

- OpenCV 평균 blur 커널 **3·5·7·9**, Gaussian 9×9 및 σ **0.5·1·2·4**.
- **보고서 관찰:** 평균 커널 크기가 증가할수록 더 흐려지고, Gaussian σ가 증가할수록 부드러워짐. **Gaussian σ=1** 선택은 **작성자의 눈으로 판단한 경계 유지/노이즈 절충**.
- **검증 경계:** 원본 clean image는 있지만 PSNR, SSIM, MSE 등 계측 스크립트/수치는 보고서에 없다.

### CV1-03 — Canny 27조합 실험
**소스:** 3_canny_edge_detection/3_canny_edge_detection.py 및 27개 PNG.

- **K=3,7,11 × L=50,100,150 × H=100,150,200 = 27** 경우를 반복.
- Gaussian σ는 **0.3×((K−1)/2−1)+0.8**, Canny는 OpenCV 함수 호출.
- **보고서 판단:** 돌고래 영상에서는 **K=11, L=100, H=200**을 시각적으로 가장 적절하다고 평가. 잡음 에지가 줄고 중요한 경계는 연결된다고 설명.
- **검증 경계:** Canny 4단계를 **코드로 모두 원시 구현한 것은 아님**. 내부 연산은 OpenCV가 처리. 조건 중 L≥H인 조합도 반복문에 포함돼 있어 정상적인 lower/higher 설정만을 선별한 최적화 연구는 아니다.

### CV1-04 — separability
**소스:** 4_filter_separability/4_filter_seperability.py, filtered_image.png, separable_filtered_image.png.

- **F=[[1,0,-1],[2,0,-2],[1,0,-1]] = [1,2,1]ᵀ×[1,0,-1]**. 코드에 **외적 동일성 assertion**.
- PyTorch **conv2d** 한 번 vs 가로·세로 필터 **2회**; 50×50 저장 영상 각각 생성.
- **사후 파일 비교 결과:** 저장된 두 PNG의 픽셀 값은 2,500픽셀 중 **45곳에서 다름**. 특정 차이의 원인·실제 계산 당시 입력 상태는 확정할 수 없다. 원본 소스는 과제 명세의 padding=0 대신 2D 경로 **padding=1**을 지정.
- **검증 경계:** 외적 수학적 분해 성립과 **최종 출력의 완전 일치**는 별개의 명제다. 후자는 확인 실패했으므로 포트폴리오에 성능·정확성 검증 완료 문장을 쓰지 않는다. saved image 재실행은 수행하지 않았다.

## C. 과제 2 — 배포 템플릿 대비 제출본 코드 차이

### CV2-01 — Manhattan distance transform + Chamfer Matching

**제공본의 미구현 함수:** distance_transform_manhattan, chamfer_matching.

**제출본 변경:** feature=1 위치 0 설정을 이용해 forward는 위/왼쪽, backward는 아래/오른쪽의 거리+1 최소 갱신. Distance map을 template 크기로 sliding해 template 마스크와 거리의 곱을 합산하고 **argmin** 위치를 찾는다.

**저장 실행 출력:** 4×4 Manhattan 예제 맵:
- 3,2,1,0
- 2,1,0,1
- 1,0,1,2
- 0,1,2,3

Chamfer 위치 및 거리맵 시각화 figure가 저장됐다. **get_binary_image_array, triangle template 생성, overlay 등 제공된 utility는 개인 신규 구현이 아니다.**

### CV2-02 — 색상 군집화

**제공본:** Lab 색변환·128픽셀 높이 resize·팔레트/시각화 기본 함수가 이미 존재.

**제출본 추가:** scikit-learn imports, **K=1~10** K-Means inertia 및 K≥2 silhouette 값 계산/시각화, silhouette 최댓값의 K 선택, KMeans fitting, **estimate_bandwidth(quantile=0.1, n_samples=500) + MeanShift(bandwidth=..., bin_seeding=True)** 구현, Mean Shift 예시 호출.

**저장 실행 출력:** **Optimal number of clusters: 4**, elbow/silhouette 차트, K-Means 및 Mean Shift 결과 figures. **군집 알고리즘을 직접 수학적으로 처음부터 구현한 것이 아니라 제공된 함수 뼈대 안에서 scikit-learn을 활용한 것**이다. 양 방법 비교의 독립 화질 수치는 없다.

### CV2-03 — Hough 변환

**제공본:** 이미지 로드, Canny 수행, accumulator 초기화 및 그림 표시의 기본 구조.

**제출본 추가:** edge pixel마다 θ=0..179°와 ρ를 계산하여 **accumulator 투표**, index 변환, **threshold=110**, ρ/θ에서 원본 공간 직선 좌표를 복원해 OpenCV line 출력.

**저장 실행 출력:** **image shape 840×834**, accumulator max **221**, **검출 직선 4개**. 해당 실습 입력 결과임. ρ 공간 offset/배열 크기는 제한적이어서 임의 크기의 영상에서도 완전한 Hough 검출기를 보장하지 않는다. 비최대 억제나 정량 정확도 실험은 확인되지 않았다.

### CV2-04 — RANSAC 한 줄/두 줄

**제공본:** CSV 로더 및 문제별 함수 signature, 일부 그래프 스캐폴드.

**제출본 추가:** single-line은 무작위 2점을 이용해 slope/intercept 후보를, two-line은 무작위 4점으로 각각의 직선 후보를 만든다. x좌표 차이가 0인 경우를 건너뛰고, 수직거리 **abs(ax−y+b)/sqrt(a²+1)**가 **1.5** 이내인 inlier 수 최대 후보를 선택. 두 함수 모두 **1,000회 반복 기본값**을 유지하며 출력 그래프 추가.

**저장 실행 출력:** 각 single/two-line 산점도와 선택 직선 그림. 세부 수치나 ground truth 비교는 출력되지 않는다. 추가로 inlier 기반 재피팅이나 다른 seed에서의 재현성 평가도 확인되지 않았다.

## D. 이력서 작성 기준

| 이력서 표현 | 상태 | 근거 |
|---|---|---|
| 임계값·평균/가우시안 필터·Canny의 실험 파라미터를 비교했다 | **사용 가능** | 소스+PNG+보고서 |
| 숫자 3을 단일 threshold로 완전히 분리하는 데 성공했다 | **사용 금지** | 보고서가 원형 잔여 한계를 명시 |
| OpenCV 없이 Canny 내부 알고리즘을 직접 구현했다 | **사용 금지** | 코드가 cv2.Canny 호출 |
| Manhattan DP/Chamfer sliding 구현을 채웠다 | **사용 가능** | 제공 ZIP과 제출 ZIP의 변경 내용 |
| K-Means/Mean Shift 알고리즘을 처음부터 직접 개발했다 | **사용 금지** | scikit-learn 호출 |
| Hough 4개 직선 검출 결과를 얻었다 | **사용 가능, 실습 입력 한정** | Notebook 출력 |
| RANSAC 1줄·2줄 fitting 함수를 구현하고 결과를 시각화했다 | **사용 가능** | 템플릿 diff+출력 figure |
| 필터 2D/1D 출력의 완전 동일·속도 최적화를 입증했다 | **사용 금지** | 픽셀 mismatch, 시간 측정 부재 |
| 실세계 환경의 CV 정확도를 향상했다 | **사용 금지** | 수업 제공 이미지·점군만 검토 |

**권장 포트폴리오 사례:** (1) threshold 분할의 실패 조건 탐색, (2) Canny 27조합 시각적 판단, (3) Hough 투표·RANSAC 구현. 다른 기술 경험처럼 **상황 → 문제 → 판단 → 조치 → 결과**로 서술하되, 과제 요구사항이 판단을 주도한 것과 학생이 결정한 구현·파라미터를 분리한다.

**남은 보강은 선택사항:** 개인 점수·피드백, 재현 환경, 사진/결과 영상에 대한 수동 QA, 더 넓은 데이터셋 테스트는 아직 없다. 현재 구현 내용과 당시 한계를 바탕으로 경험 정리 자체는 완료 가능하다.
