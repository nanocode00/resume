# Computer Vision Coursework & Practice — 2025 가을 실습 복원

> **2026-10-28 검증본 v1.** ITEC0424-001 Computer Vision 개인 과제 1·2의 **직접 제출한 보고서·코드 ZIP**과 **과제 2 배포 템플릿 ZIP**을 비교했다. 구현·실행 결과·사용한 라이브러리·제공된 스캐폴드를 구분한다. 상세 근거는 [증거 원장](computer-vision-evidence-ledger.md).

## 한 줄 요약

2025년 가을 컴퓨터비전 개인 과제에서 Thresholding, Denoising, Canny 파라미터를 조정하며 결과·한계를 분석하고, 제공된 Jupyter Notebook의 빈 구현부에 **Manhattan Distance Transform / Chamfer Matching, K-Means·Mean Shift 활용, Hough 누적 투표, 단일·이중 RANSAC 직선 피팅**을 완성했다. 연구·제품 개발이 아니라 **학습용 입력에 대한 알고리즘 구현과 비교 실험**이다.

## 원본과 개인 기여의 경계

- **과제 1:** 개인 보고서 **Computer Vision Assignment 1.pdf**와 **Computer Vision Assignment 1.zip**. 네 개 Python 파일 및 출력 PNG 확인. 사용한 OpenCV·PyTorch 함수 자체를 직접 재구현했다고 주장하지 않는다.
- **과제 2:** 제공 원본 **CV-2025-fall-Assignment02.zip**과 학번이 붙은 **CV-2025-fall-Assignment02-2019111450.zip**에서 네 Notebook을 비교했다. 학생이 직접 추가한 코드는 기존 utility·예제 데이터·시각화 코드와 구분해 확인했다.
- **코드·실험의 검증 범위:** 보존된 소스, 과제 보고서, Notebook 실행 결과·출력 이미지를 근거로 한다. 자동채점 점수·독립 재현 테스트·처리 시간 향상·일반화 정확도는 확인되지 않았다.

## 경험 1 — 임계값 기반 분할의 경계와 실패 분석

**상황:** 숫자와 원형 패턴이 섞인 흑백 영상을 특정 숫자만 남도록 분할하는 실습.

**문제:** 단일 threshold로 숫자 3만 구분하려 하면 **중앙 원형 영역 일부가 함께 남았다**.

**판단:** 이진·반전 threshold **55, 129, 200**을 먼저 비교하고, 두 임계값을 동시에 사용하는 **window threshold** 범위를 더 촘촘하게 탐색했다.

**조치:** OpenCV threshold와 bitwise_and를 사용해 구간 이진화를 구현하고, **T=125~134** 및 **하한 128/129·상한 130/131/132** 조합의 출력 이미지를 저장했다.

**결과:** 보고서에서는 **129~130 조합에서 불필요한 원형 고리가 가장 얇았으나 남아 있다**고 판단했다. **thresholding만으로 숫자 3 이외의 모든 영역을 완전히 제거할 수 없다는 한계**를 정직하게 서술했다.

## 경험 2 — 노이즈와 경계 보존의 파라미터 실험

**상황:** Gaussian noise 이미지의 denoising과 돌고래 영상의 Canny 에지 검출을 비교했다.

**문제:** smoothing을 강하게 할수록 노이즈는 줄지만 중요한 경계도 약해지며, Canny 임계값도 에지 잡음과 연결성에 영향을 주었다.

**판단:** 필터 크기, Gaussian sigma, Canny의 lower/higher threshold를 독립적인 실험 변수로 두고 시각적 결과를 비교했다.

**조치:** 평균 필터 **3×3·5×5·7×7·9×9**, Gaussian 9×9의 **σ=0.5·1·2·4**로 8개 결과를 생성했다. Canny는 **K=3/7/11, L=50/100/150, H=100/150/200 총 27개 조합**을 순회하고 **σ=0.3×((K−1)/2−1)+0.8**에 따른 GaussianBlur를 선행했다.

**결과:** 보고서에서 **Gaussian σ=1**을 노이즈 감소·경계 보존의 주관적 절충안으로 선택했다. 돌고래 영상의 Canny에서는 **K=11, L=100, H=200**을 선택해 불필요한 에지 감소와 주요 윤곽 연결성 유지라는 이유를 적었다. **PSNR·정확도·정밀도/재현율 계측 근거는 없다**.

## 경험 3 — 필터의 분리 가능성 확인과 구현 경계

**상황:** 3×3 필터를 2D 연산과 두 번의 1D 연산으로 적용하는 과제.

**문제:** 분리 가능한 커널인지 수학적으로 확인하고, 두 구현 출력을 검토해야 했다.

**판단:** 3×3 커널 **[[1,0,-1],[2,0,-2],[1,0,-1]]**을 **세로 [1,2,1]ᵀ × 가로 [1,0,-1]**로 분해했다.

**조치:** PyTorch의 conv2d로 직접 2D 경로와 가로→세로 분리 경로를 각각 구현했다. 커널 외적 동일성 assertion을 실행하고 출력 PNG를 저장했다.

**결과:** **커널의 수학적 분리 가능성**과 **양 경로의 출력 생성**은 확인된다. 그러나 저장 PNG는 50×50 중 **45픽셀이 다르며** 출력 동등성 assertion은 없다. 과제 명세는 padding=0이지만 소스는 직접 2D 경로에 padding=1을 사용했다. 따라서 **결과가 완전히 동일하다거나 속도 개선을 측정했다고 주장하지 않는다**.

## 경험 4 — Manhattan Distance Transform / Chamfer Matching

**상황:** 과제 2의 이진 scene에서 template edge 패턴을 검출하는 제공 Notebook의 미완성 구현을 채웠다.

**문제:** template 위치별 차이를 평가할 수 있는 distance transform map이 필요했다.

**판단:** feature 위치를 거리 0으로 두고 **forward/backward 두 번 순회**하며 Manhattan 최소거리를 전파하고, template를 sliding하여 거리 합이 최소인 곳을 찾는 구조로 구성했다.

**조치:** 배포된 빈 함수 **distance_transform_manhattan / chamfer_matching**을 구현했다. 전후방 이웃의 거리+1 갱신과 template 영역 누적 거리 합, argmin 위치 탐색을 작성했다.

**결과:** 제출 Notebook에는 **4×4 Manhattan 거리맵 출력**과 matching 시각화 figure가 저장됐다. 실제 발견 위치의 개별 수치, 외부 이미지 성공률 및 회전·스케일 변화에서의 성능은 미검증이다.

## 경험 5 — K-Means 및 Mean Shift 색상 군집

**상황:** 꽃 이미지의 대표 색상들을 묶어 원본과 군집 결과 및 팔레트를 비교했다.

**문제:** K-Means에서는 군집 수 K 선택이 필요하고 Mean Shift와 결과를 비교해야 했다.

**판단:** 제공된 Lab 색공간 전처리 helper를 사용하면서 **elbow inertia + silhouette score**로 K 후보를 평가했다.

**조치:** Notebook의 비어 있는 함수 본문에 scikit-learn **KMeans, silhouette_score, MeanShift, estimate_bandwidth**를 호출하는 구현을 추가하고 시각화 셀을 실행했다. **군집 알고리즘 내부 자체를 바닥부터 구현한 것은 아니다.**

**결과:** 저장 실행 로그에서 **Optimal number of clusters: 4**를 확인했고 K-Means/Mean Shift 결과 figure가 남았다. 양 방법의 우열에 대한 정량 지표와 다른 영상에서의 성능은 확인되지 않는다.

## 경험 6 — Hough 누적 투표와 RANSAC 직선 모델

**상황:** 영상 에지에서 4개 직선을 검출하고, 이상치가 있는 점군에서 1개·2개 직선을 fitting하는 실습.

**문제:** Hough에서는 (ρ,θ) 누적 투표와 검출 임계값이 필요하고, RANSAC에서는 적은 점으로 가정한 후보를 반복 평가해야 했다.

**판단:** Hough는 각 edge pixel에 대해 **θ=0~179°**의 ρ를 계산해 투표했다. RANSAC은 두 점으로 단일 직선, 네 점으로 두 직선을 생성하고 수직거리 내 inlier 수를 비교하도록 했다.

**조치:** 배포 Hough Notebook의 미구현 투표 루프·직선 시각화 코드를 작성하고 **투표 threshold=110**을 지정했다. RANSAC의 **single-line / two-line** 함수를 1,000회 반복, inlier 거리 임계값 1.5로 구현해 그래프를 남겼다.

**결과:** 저장된 Hough 실행 결과는 **최대 투표값 221, 검출 직선 4개**다. RANSAC은 한 줄·두 줄에 대한 출력 그래프가 존재한다. 이미지·CSV 정답에 대한 정량 오차, 여러 seed의 안정성 및 일반적인 장면에 대한 검출 성능은 확인하지 않았다.

## 학습 내용과 독립 구현의 구분

강의에는 영상 형성·필터·gradient/edge·binary morphology·segmentation·feature matching·alignment·warping/stitching·recognition 등이 포함되지만, **강의 PDF가 있다고 모두 직접 구현했다고 쓸 수는 없다**. 본 문서는 실제 과제 제출 파일이 있는 알고리즘에 한정한다.

**이력서용 짧은 문장:** “컴퓨터비전 개인 과제에서 분할·denoising·Canny 파라미터를 비교해 출력의 노이즈·경계 유지 한계를 분석했습니다. 배포된 실습 Notebook의 미구현 부분에 Manhattan Distance Transform/Chamfer Matching, Hough 변환, 단일·이중 RANSAC line fitting을 작성하고 K-Means/Mean Shift 군집화 결과를 비교했습니다. Hough 실습 입력에서 4개의 직선을 검출했으며, 범용 검출 성능이 아닌 수업 입력의 재현 결과로 구분합니다.”

**활용 포인트:** 전통 CV 기본기, 파라미터 실험과 결과 해석, 비교 가능한 중간 산출물 생성. 제품 성능·정량적 기술 혁신·개인 논문 수준의 연구로 확대하지 않는다.
