# CNN Accelerator — PyTorch 양자화부터 OpenROAD 합성까지

## 채용용 경험 요약

기존 공개 CNN RTL을 baseline으로 분석한 뒤 PyTorch Quantization 결과를 HW용 정수 데이터로 변환하고, RTL을 synthesis 가능한 형태로 리팩터링해 ModelSim과 OpenROAD ASAP7 flow까지 연결한 3인 팀 프로젝트입니다. **3개 저장소의 개인 일반 커밋 19개(4+12+3)를 1차 대조한 상세 근거는 [개발작업 원장](cnn-accelerator-commit-ledger.md)에 정리**했습니다.

- 기간: **2025.06**
- 팀 규모: **3인**
- 과목: 경북대학교 전자공학부 논리회로설계 기말 프로젝트
- 담당 범위: PyTorch Quantization/HW reference 생성, SW↔RTL 중간 결과 비교, synthesis-oriented RTL 수정, ModelSim/OpenROAD 검증 환경
- 기술: PyTorch, Verilog, ModelSim, OpenROAD Flow Scripts, ASAP7, Git submodule

### 상황 → 문제 → 판단 → 조치 → 결과

**상황**  
기존 공개 2-layer MNIST CNN RTL을 기반으로 SW 모델의 양자화 결과를 RTL 입력 형식으로 옮기고, RTL simulation을 넘어 ASIC synthesis/physical design까지 이어가는 프로젝트를 진행했습니다.

**문제**  
첫째, PyTorch quantized model의 weight/bias와 RTL fixed-point 표현이 바로 일치하지 않아 SW reference와 HW 결과를 신뢰하기 어려웠습니다. 둘째, simulation에서 동작하던 parameter unpacking 방식 일부가 synthesis 과정에 적합하지 않았습니다.

**판단**  
최종 classification 결과만 비교하면 오차가 어디서 시작되는지 알 수 없으므로 Conv/Pool/FC 중간 출력과 channel별 MAC 결과까지 reference로 저장해 단계적으로 비교하기로 했습니다. 또한 고정 weight/bias wiring은 procedural `always` block이 아니라 elaboration 단계에서 결정되는 정적 연결로 바꾸기로 했습니다.

**조치**  
PyTorch에 `QuantStub/DeQuantStub`와 `fbgemm` quantization을 적용하고 quantized weight의 `int_repr()`와 scale을 이용해 weight/bias를 8-bit 2's complement/hex 형태로 변환했습니다. `fix done` 커밋에서는 quantized Conv2를 기존 float tensor처럼 `weight.detach()`로 접근하던 reference 계산을 `weight().int_repr()` 기반으로 바로잡았습니다. RTL에서는 Conv1/Conv2/FC의 `reg array + always @(*)` parameter unpacking을 `wire + generate/genvar + continuous assign`으로 변경했습니다. 이후 RTL repo를 Git submodule로 OpenROAD Flow Scripts의 ASAP7 design source에 연결했습니다.

**결과**  
ModelSim에서 **MNIST 1,000장 기준 RTL accuracy 96%**를 확인했고 OpenROAD에서 synthesized Verilog와 physical design final output까지 생성했습니다. **다만 Git에 고정된 OpenROAD RTL SHA는 최종 RTL 시뮬레이션 SHA보다 오래된 버전이므로 두 결과를 동일 리비전의 연속 검증으로 단정할 수 없습니다.** 그러나 gate-level simulation에서는 정상 파형을 확보하지 못해 Netlist accuracy를 측정하지 못했고, 최종 STA도 **slack -2040.66, VIOLATED**로 timing closure에 실패했습니다. 따라서 PPA/TOPS/W를 최종 성과값으로 주장하지 않습니다.

## 불일치 원인을 어디까지 확인했는가

### PyTorch reference ↔ RTL 단계에서 확인된 원인

Quantization 이후 layer는 일반 float `nn.Conv2d`와 접근 방식이 달라집니다. 초기 reference 계산 코드에서는 Conv2 weight를 float tensor처럼 다루는 흔적이 있었고, `c32db292... (fix done)`에서 다음처럼 바뀌었습니다.

- `model.conv2.weight.detach().numpy()`
- → `model.conv2.weight().int_repr().numpy()`

HW용 계산에서도 quantized integer representation과 scale/bias 처리를 구분하도록 수정했습니다. 따라서 **SW/HW 정합성 문제 중 하나는 RTL 자체의 연산 오류가 아니라 quantized PyTorch 값을 HW reference로 추출하는 과정의 표현 불일치**였다고 말할 수 있습니다.

### gate-level Netlist 단계에서 확인하지 못한 원인

합성 netlist와 RTL을 동일 입력으로 비교하는 `tb_compare` 환경까지 만들었지만 최종 Netlist simulation에서는 올바른 waveform을 얻지 못했습니다. 발표자료와 Git history 모두 이 단계를 실패로 기록합니다.

따라서 다음은 주장하지 않습니다.

- Netlist 비정상 파형의 root cause를 완전히 규명했다
- RTL ↔ Netlist functional equivalence를 완료했다
- post-synthesis accuracy 96%를 확인했다

## 핵심 수치

| 항목 | 결과 | 해석 |
|---|---:|---|
| RTL ModelSim accuracy | **96% / MNIST 1,000장** | [Git 원본 `result.log`](https://github.com/nanocode00/cnn_verilog/blob/master/modelsim/result.log) 및 최종 발표자료 6쪽에서 같은 값 확인 |
| 발표자료의 다른 RTL run | 97% | 실행 시점이 다른 별도 run, 대표값은 96% 사용 |
| CNN OpenROAD core utilization 설정 | 22 | CNN config 기준 |
| Place density 설정 | 0.45 | CNN config 기준 |
| Final STA slack | **-2040.66, VIOLATED** | [최종 발표자료](https://drive.google.com/file/d/10WVhYvzaPfui0tuNIbl6vbcIyddnvWma/view) 13쪽의 **OpenROAD `6_final - chip` 화면**; clock 400 기준, timing closure 실패 |
| Post-synthesis 중간 power report | **6.72e-02 W (0.0672 W)** | 최종 발표자료 **12쪽 `1_Post_synthesis.rpt`의 추정값**. post-route 최종 전력으로 전용 불가 |
| Netlist accuracy | 미측정 | 정상 waveform 확보 실패 |
| TOPS/W | 미도출 | 계산에 필요한 관계만 검토 |

## 프로젝트 저장소 구조와 기여 범위

최종 발표자료에서는 세 저장소의 역할을 다음과 같이 구분했습니다.

- `nanocode00/OpenROAD-flow-scripts`: 합성을 위한 설정 파일
- `nanocode00/CNN-Implementation-in-Verilog`: Quantization 학습 및 SW reference 생성
- `nanocode00/cnn_verilog`: Verilog RTL 및 simulation

이 프로젝트는 기존 공개 프로젝트 `CNN-Implementation-in-Verilog`의 2-layer MNIST CNN을 baseline으로 사용했습니다. 2021년 원본 RTL과 architecture는 원저작자의 작업이므로 개인 구현으로 포함하지 않습니다.

정확한 표현은 **기존 CNN RTL을 이해하고 PyTorch quantized model과 맞춘 뒤 synthesis 가능한 RTL로 수정해 ASIC flow와 검증 환경까지 연결한 HW/SW co-design 경험**입니다.

## baseline CNN 구조

`Conv1 → MaxPool/ReLU → Conv2 → MaxPool/ReLU → Fully Connected → Comparator`

- 입력: 28×28 grayscale image, 8-bit stream
- Conv1: 1 → 3 channel, 5×5 kernel
- MaxPool: 24×24×3 → 12×12×3
- Conv2: 3 → 3 channel, 5×5 kernel
- MaxPool: 8×8×3 → 4×4×3
- Flatten: 48
- Fully Connected: 48 → 10

## 1. PyTorch Quantization 및 HW reference 생성

2025년 6월 8일 `quantization`, `fixing`, `fix done` 커밋에서 직접 확인되는 내용:

- `QuantStub` / `DeQuantStub`
- `torch.backends.quantized.engine = 'fbgemm'`
- calibration 후 quantized model convert
- `int_repr()`로 integer weight 추출
- layer scale을 반영한 bias/int 계산
- 음수를 8-bit 2's complement로 변환
- Conv1 / Conv2 / FC weight와 bias를 hexadecimal text로 저장
- Conv1 / MaxPool1 / Conv2 / MaxPool2 / FC 중간 출력을 reference로 저장

초기 구현에서 float Tensor와 quantized Tensor를 같은 API로 다루면서 생긴 reference 오류를 후속 커밋에서 수정했습니다.

## 2. layer / channel / MAC 단위 디버깅

특히 Conv2에서 다음 값을 직접 계산하는 코드가 남아 있습니다.

- 3개 input channel별 5×5 input window
- channel별 5×5 weight
- channel별 MAC sum
- 3개 channel 결과 합
- bias 적용 결과
- 실제 PyTorch Conv2 output

따라서 `최종 class가 다르다`에서 멈추지 않고 **어느 layer와 어느 channel 연산부터 차이가 발생하는지** 확인할 수 있는 reference path를 만들었습니다.

## 3. RTL 저장소 분리와 Git submodule

2025년 6월 7일 `CNN-Implementation-in-Verilog`에 `.gitmodules`를 추가해 RTL을 `nanocode00/cnn_verilog`로 분리했습니다. 같은 RTL 저장소를 `OpenROAD-flow-scripts/flow/designs/src/cnn_verilog`에도 submodule로 연결했습니다.

부모 저장소는 child repository의 특정 commit SHA를 gitlink로 고정하므로 OpenROAD flow에서 사용한 RTL revision을 명시적으로 관리했습니다.

**Gitlink 역검증(2026-10-28):** 부모 두 개가 실제로 서로 다른 RTL revision을 가리킨다는 점을 확인했습니다.

| 상위 저장소와 시점 | `cnn_verilog` Gitlink | 실제 소스 범위 |
|---|---|---|
| [CNN-Implementation-in-Verilog `master`](https://github.com/nanocode00/CNN-Implementation-in-Verilog/tree/master/cnn_verilog) | [`4326abb6509e`](https://github.com/nanocode00/cnn_verilog/commit/4326abb6509e) (6/7) | 가져온 초기 RTL의 `verilog/*.v` |
| [OpenROAD `19d9d49`](https://github.com/nanocode00/OpenROAD-flow-scripts/commit/19d9d49667e5) (6/7) | `4326abb6509e` | 초기 RTL |
| [OpenROAD `e430e15`](https://github.com/nanocode00/OpenROAD-flow-scripts/commit/e430e1543d1a) (6/18) | [`c909e315352c`](https://github.com/nanocode00/cnn_verilog/commit/c909e315352c) | 해당 시점의 `verilog/*.v` |
| [OpenROAD `c445907` 및 최종 `master`](https://github.com/nanocode00/OpenROAD-flow-scripts/commit/c44590700b3c) (6/18) | [`5f34734c749d`](https://github.com/nanocode00/cnn_verilog/commit/5f34734c749d) | 해당 시점의 `verilog/*.v` |
| [RTL 최종 `master`](https://github.com/nanocode00/cnn_verilog/commit/e6af4816d302) (6/27) | `e6af4816d302` | `conv1_layer.v`, `conv2_layer.v`, `fully_connected.v` 등 root 경로에 배치된 리팩터링 코드와 최종 정확도 로그 |

OpenROAD 설정은 `$(DESIGN_HOME)/src/$(DESIGN_NICKNAME)/verilog/*.v`를 대상으로 합니다. **부모에 고정된 6월 18일 RTL에는 이 경로가 존재하지만, 최종 6월 27일 RTL `master`에는 `verilog/` 디렉터리가 없고 root 소스가 있습니다.** 즉, 최종 수정된 RTL을 기존 설정으로 다시 합성하려면 **Gitlink 갱신과 소스 glob 수정**이 필요합니다.

가장 중요한 증거 경계는 **6월 27일 RTL의 ModelSim 96%**와 **6월 18일 SHA를 지정한 OpenROAD 저장소에서 보인 physical `6_final` 결과**를 *같은 RTL revision에 대한 종단 검증*이라고 주장할 수 없다는 점입니다. 발표 당시 로컬 checkout에서 별도의 미커밋 수정/서브모듈 이동이 있었는지 기록이 없으므로 **그 GUI 실행의 실제 RTL SHA는 미확정**입니다.

## 4. ModelSim RTL 검증

- `modelsim/top_tb.v`: single image
- `modelsim/top_tb_1000.v`: 1,000 image classification
- `$readmemh`로 weight/bias loading
- expected label과 RTL `decision` 비교
- 최종 accuracy 계산

마지막 Git log 기준 **Accuracy: 96%**입니다.

## 5. synthesis-oriented RTL refactoring

`e6af4816... refactoring for synth`에서 다음 구조를 바꿨습니다.

- Before: `reg` array + `integer i` + combinational `always @(*)`
- After: `wire` array + `generate` / `genvar` + continuous `assign`

적용 범위:

- Conv1 weight/bias
- Conv2 weight/bias
- Fully Connected weight/bias

이후 수정된 root RTL 파일을 ModelSim에서 다시 compile/simulation했습니다.

## 6. OpenROAD ASAP7 flow

CNN용 config:

- `PLATFORM = asap7`
- `DESIGN_NAME = chip`
- `CORE_UTILIZATION = 22`
- `CORE_ASPECT_RATIO = 1.2`
- `CORE_MARGIN = 6`
- `PLACE_DENSITY = 0.45`
- `CLOCK_PORT = clk`
- `CLOCK_PERIOD = 10.0`

### Clock constraint 주의

별도 `constraint.sdc`에는 `clk_period=400`이 있고 최종 발표의 timing report 역시 400 기준입니다. 따라서 **10ns / 100MHz timing closure를 달성했다고 주장하지 않습니다.**

Google Drive의 `openroad_project.zip` 안에는 AES example config도 섞여 있습니다. 그 파일의 `CORE_UTILIZATION=40`, `PLACE_DENSITY=0.65`는 CNN 결과가 아닙니다.

## 7. synthesis / physical design 결과

확인 가능한 완료 범위:

- ASAP7 design configuration: 완료
- Yosys synthesized `1_synth.v`: 생성 완료
- OpenROAD physical design `6_final - chip`: 실행 완료
- timing/power report 확인: 완료(합성 직후 power와 배치배선 final timing은 **서로 다른 단계**)
- timing closure: **실패**
- valid final PPA: **미확정**

[팀 최종 발표자료](https://drive.google.com/file/d/10WVhYvzaPfui0tuNIbl6vbcIyddnvWma/view) **13쪽의 OpenROAD GUI 스크린샷**에서 `OpenROAD - asap7/cnn_verilog/base - 6_final - chip`, `data required time = 681.11`, `data arrival time = 2721.77`, **`slack = -2040.66 (VIOLATED)`**를 직접 읽어 확인했습니다. **이것은 OpenROAD GUI final stage를 열어 경로를 조회했다는 증거**이며 DRC/기능성/실리콘 동작까지 통과했다는 뜻은 아닙니다.

발표자료 **12쪽**의 `1_Post_synthesis.rpt` 발췌에는 `worst slack 74.33` 및 min-path 관련 `slack 33.66 (MET)`과 `Total Power = 6.72e-02 W`가 표시됩니다. 이는 **합성 직후**의 보고이며 13쪽의 **최종 물리 설계 STA slack -2040.66**과 동일 단계나 동일 검증조건의 수치로 취급할 수 없습니다. 따라서 이력서의 최종 주파수/전력 효율 및 유효한 PPA로 주장하지 않습니다.

## 8. Gate-level RTL / Netlist 검증 시도

`modelsim_synth/`에는 다음이 남아 있습니다.

- 약 59MB synthesized `chip_synth.v`
- ASAP7 standard-cell simulation models
- `tb_synth.v`
- `tb_compare.v`

`tb_compare.v`는 RTL `chip`과 synthesized `chip_synth`에 같은 image, weight, bias를 공급해 `decision`, `decision_synth`, 각 accuracy와 match를 비교하는 구조입니다.

검증 환경은 구축했지만 Git [`091fc2de`](https://github.com/nanocode00/cnn_verilog/commit/091fc2de3faa)의 제목 자체가 `synth simulation (failed)`이고, 최종 발표자료 **11쪽**의 ModelSim 화면은 정상적인 출력 파형을 확보하지 못했음을 보여 줍니다. 화면에는 **`Design size of 12593 statements exceeds ModelSim-Intel FPGA Starter Edition recommended capacity`** 경고가 있지만, 그것만으로 실패의 **유일한 원인을 확정할 수는 없습니다**. 따라서 netlist accuracy는 미측정이며 RTL 96%와 구분합니다.

## 9. TOPS/W와 PPA 평가 범위

최종 발표에는 `Netlist TOPS/W Calculation` 단계가 있지만 실제 최종 TOPS/W 값은 도출되지 않았습니다.

확인한 범위:

- total MAC count 필요성
- latency 필요성
- OpenROAD/OpenSTA power report 필요성
- timing/power report 확인

따라서 `PPA/TOPS/W 평가 완료`가 아니라 **synthesis/P&R report를 분석하고 효율 계산을 시도했다**고 표현합니다.

## Git 및 산출물에서 확인되는 주요 흐름

### 2025-06-07

- RTL을 `cnn_verilog` 별도 repo로 분리
- superproject와 OpenROAD Flow Scripts 양쪽에서 submodule 연결

### 2025-06-08

- `0815e86... quantization`: PyTorch quantization / HW hex reference 생성
- 후속 fixing
- `c32db292... fix done`: quantized weight 접근을 `weight().int_repr()`로 수정하고 Conv2 reference 재생성

### 2025-06-18

- synth 환경과 testbench 수정
- OpenROAD ASAP7 config/SDC 연결
- child RTL revision과 parent submodule pointer 동기화

### 2025-06-25 ~ 27

- synthesis/simulation 반복 수정
- `e6af4816...`: synthesis-friendly static parameter wiring으로 refactor
- 1,000-image RTL simulation **96%**
- OpenROAD synthesis/P&R
- gate-level comparison environment 구성
- Netlist waveform 확보 실패, timing closure 실패

## 결과

### 성공한 범위

- PyTorch quantized representation을 Verilog 입력/reference 형식으로 연결
- SW reference extraction 오류 수정
- layer/MAC 단위 mismatch debugging path 구축
- synthesis-oriented RTL refactor
- ModelSim 1,000-image RTL accuracy **96%**
- OpenROAD ASAP7 synthesis와 physical design final stage 실행
- RTL vs Netlist gate-level 비교 testbench 구축

### 완료하지 못한 범위

- 정상적인 gate-level Netlist waveform 확보
- post-synthesis accuracy 측정
- RTL/Netlist functional equivalence 최종 확인
- timing closure
- 유효한 최종 PPA/TOPS/W

## 이력서용 핵심 bullet

- 기존 CNN RTL의 PyTorch Quantization 결과를 `int_repr()`/scale 기반 HW 정수·hex reference로 변환하고 layer/channel별 MAC 비교로 SW↔RTL 정합성 검증
- quantized layer를 float tensor처럼 처리해 잘못된 reference가 생성되던 문제를 수정하고, Conv/FC parameter wiring을 `generate + assign` 구조로 변경해 synthesis-friendly RTL로 refactor
- ModelSim MNIST 1,000장 **RTL accuracy 96%** 확인 후 OpenROAD ASAP7 synthesis/P&R final stage까지 수행; gate-level waveform 및 timing closure 실패까지 검증 범위와 한계를 기록

## 근거 인덱스와 기여 경계

- [개발작업 원장](cnn-accelerator-commit-ledger.md): 3개 저장소 기본 브랜치 개인 일반 커밋 **19개**, RTL merge 1개 별도, 동일 tree 중복·생성물/원본 소스 귀속, 통합 테스트의 실패/성공 범위.
- [최종 팀 발표자료(2025.06, 총 17쪽)](https://drive.google.com/file/d/10WVhYvzaPfui0tuNIbl6vbcIyddnvWma/view): **6쪽 RTL Accuracy 96%; 11쪽 Netlist 파형 미확보·Starter Edition 경고; 12쪽 post-synthesis timing/power; 13쪽 OpenROAD `6_final` 및 STA -2040.66**. 각 화면을 직접 시각 검사했다.
- [원본 RTL ModelSim 평가 로그](https://github.com/nanocode00/cnn_verilog/blob/master/modelsim/result.log), [컴파일 오류 및 후속 실행 기록](https://github.com/nanocode00/cnn_verilog/blob/master/modelsim/result_single.log): 1,000 입력·96% 및 weight/bias unpacking의 `reg` continuous-assignment 오류.
- [당시 과제 안내자료](https://drive.google.com/file/d/1y03YTev2uFmBLc-wI68HPFORg33EpITc/view): 평가 방법·OpenROAD 실습 내용의 **수업/과제 안내**로, 팀의 최종 증빙과 구분.
- **정확한 개인 소유권:** 원본 CNN 계층 구현(2021 Bo Young Kang 표기), OpenROAD/Yosys/OpenSTA/ABC 도구, 표준셀 및 수백만 행 netlist는 개인 최초 작성이 아니다. 개인 기여는 PyTorch 양자화·reference 추출 수정, 고정 파라미터 unpacking의 RTL 합성 지향 수정, 테스트/ORFS 통합·오류 추적이다. 포크와 submodule은 **버전 관리·환경 설정**의 근거다.


## 표현할 때 주의할 부분

사용하지 않는 표현:

- CNN accelerator RTL 전체를 처음부터 설계했다
- CNN architecture를 직접 설계했다
- quantization 후 정확도 유지율을 최종 확인했다
- 10ns / 100MHz timing closure를 달성했다
- post-synthesis accuracy 96%를 달성했다
- RTL과 synthesized Netlist의 functional equivalence를 완료했다
- TOPS/W/PPA 최종값을 확보했다
- `CORE_UTILIZATION=40`, `PLACE_DENSITY=0.65`가 CNN 설정이다

현재 가장 정확한 한 문장:

> 기존 공개 CNN RTL을 PyTorch quantized model과 정합하고 synthesis-friendly 구조로 수정한 뒤 ModelSim에서 1,000-image RTL accuracy 96%를 확인하고 OpenROAD ASAP7 synthesis/P&R까지 확장했습니다. Gate-level 비교 환경도 구축했지만 Netlist 정상 파형과 timing closure는 확보하지 못했습니다.
