# CNN Accelerator — PyTorch 양자화부터 OpenROAD 합성까지

## 한 줄 요약

기존 공개 CNN RTL을 baseline으로 분석한 뒤, PyTorch 양자화 결과를 HW용 정수 데이터로 변환하고 RTL을 합성 가능한 형태로 리팩터링했습니다. 이후 ModelSim에서 1,000개 입력을 검증하고, RTL 저장소를 Git submodule로 OpenROAD Flow Scripts에 연결해 ASAP7 합성 및 gate-level 검증 환경까지 구성한 프로젝트입니다.

## 기간

- GitHub에서 직접 작업이 확인되는 기간: 2025년 6월

## 소스와 기여 범위

이 프로젝트는 기존 공개 프로젝트 `CNN-Implementation-in-Verilog`의 2-layer MNIST CNN을 baseline으로 사용했습니다. 저장소 README와 2021년 커밋은 원저작자의 작업이므로 개인 구현으로 포함하지 않습니다.

제 작업은 2025년에 다음 세 저장소로 분리되어 있습니다.

```text
CNN-Implementation-in-Verilog
├─ pyTorch/                  # 양자화, HW reference data 생성
└─ cnn_verilog -------------┐
                            │ Git submodule
nanocode00/cnn_verilog <----┘
├─ baseline/                 # 원본 RTL 보존
├─ RTL 수정본
├─ modelsim/                 # RTL simulation
└─ modelsim_synth/           # synthesized netlist 검증

OpenROAD-flow-scripts
└─ flow/designs/src/cnn_verilog
   └─ nanocode00/cnn_verilog의 특정 commit을 가리키는 submodule
```

따라서 이 경험은 CNN RTL 전체를 처음부터 설계한 프로젝트라기보다, **기존 CNN RTL을 이해하고 PyTorch 정수 모델과 맞춘 뒤 synthesis 가능한 RTL로 수정해 ASIC flow와 검증 환경까지 연결한 HW/SW co-design 경험**으로 설명하는 것이 정확합니다.

## baseline CNN 구조

RTL top module인 `chip`은 다음 순서로 동작합니다.

`Conv1 → MaxPool/ReLU → Conv2 → MaxPool/ReLU → Fully Connected → Comparator`

- 입력: 28×28 grayscale image를 8-bit stream으로 전달
- Conv1: 1 input channel → 3 output channels, 5×5 kernel
- Conv2: 3 input channels → 3 output channels, 5×5 kernel
- Fully Connected: 48 → 10
- 최종 comparator가 0~9 classification 결과를 출력

이 구조 자체는 upstream baseline이며, 아래 내용부터가 Git으로 확인되는 직접 작업입니다.

## 1. PyTorch 양자화 및 HW reference data 생성

2025년 6월 8일 `quantization`, `fixing`, `fix done` 커밋에서 PyTorch 모델의 quantization과 HW 입력 데이터 생성을 수정했습니다.

주요 내용은 다음과 같습니다.

- PyTorch quantization의 `QuantStub` / `DeQuantStub` 적용
- `fbgemm` backend를 사용한 quantization 준비 및 변환
- quantized weight에서 `int_repr()`를 사용해 실제 integer representation 추출
- quantized scale을 반영해 weight / bias 값을 HW에서 사용하는 정수 범위로 변환
- 음수를 8-bit 2's complement 표현으로 변환
- Conv1 / Conv2 / FC weight와 bias를 hexadecimal text로 저장
- Conv1, MaxPool1, Conv2, MaxPool2, FC 입출력 값을 별도 reference file로 저장

양자화 코드 수정 과정에서는 float Tensor와 quantized Tensor를 동일하게 처리하면서 생기는 오류를 해결하기 위해 `dequantize()`, `int_repr()`, layer scale을 구분해 사용했습니다.

## 2. 중간 레이어와 MAC 단위 비교

최종 classification 결과만 비교하지 않고 PyTorch에서 각 layer의 중간 출력을 저장하도록 했습니다.

특히 Conv2 계산에서는 다음 데이터를 별도로 확인하는 코드가 있습니다.

- 3개 input channel의 5×5 window
- channel별 5×5 weight
- channel별 MAC 결과
- 3개 channel의 MAC 합
- bias 적용 결과
- 실제 PyTorch Conv2 output

이를 통해 SW와 RTL 결과가 다를 경우 전체 inference 결과만 보는 대신 **어느 layer, 어느 channel의 convolution 연산에서 차이가 시작되는지 좁혀가는 방식**으로 디버깅했습니다.

## 3. RTL 저장소 분리와 Git submodule

2025년 6월 7일 `CNN-Implementation-in-Verilog`에 `.gitmodules`를 추가하고 RTL 작업을 별도 `nanocode00/cnn_verilog` 저장소로 분리했습니다.

부모 저장소에는 RTL 파일 전체가 복사되어 저장되는 것이 아니라 다음 두 정보가 저장됩니다.

- `.gitmodules`: submodule repository URL
- `cnn_verilog` gitlink: 사용할 child repository의 정확한 commit SHA

같은 방식으로 `OpenROAD-flow-scripts`의 `flow/designs/src/cnn_verilog`에도 이 RTL 저장소를 submodule로 연결했습니다.

실제 작업 중 `cnn_verilog`에서 testbench를 수정한 뒤 OpenROAD 부모 저장소에서 `submodule sync` 커밋으로 pointer를 새 SHA로 올린 기록도 남아 있습니다. 즉 RTL과 ASIC flow 환경을 독립된 저장소로 관리하면서 특정 RTL revision을 재현할 수 있도록 연결했습니다.

## 4. ModelSim RTL 검증 환경 정리

기존 testbench에는 `C:/cnn_verilog/...` 같은 절대경로가 포함되어 있었습니다. 이를 repo-relative path로 바꾸고 single-image / 1,000-image testbench를 정리해 저장소 위치와 관계없이 실행할 수 있도록 수정했습니다.

현재 RTL 검증 환경에는 다음이 포함되어 있습니다.

- `modelsim/top_tb.v`: 단일 입력 검증
- `modelsim/top_tb_1000.v`: 1,000개 입력 classification 검증
- weight / bias를 `$readmemh`로 로드해 wide input bus로 연결
- 각 입력의 expected label과 RTL decision 비교
- 최종 accuracy 계산

현재 저장된 ModelSim 실행 로그에서 RTL은 1,000개 입력에 대해 **96% accuracy**를 기록했습니다.

이 96%는 현재 `cnn_verilog`의 RTL testbench와 데이터로 실제 실행된 결과이며, upstream README에 기록된 예전 92% simulation accuracy와는 별도의 결과입니다.

## 5. synthesis를 위한 RTL refactoring

원본 RTL은 simulation에서는 동작하지만 synthesis 과정에서 문제가 되는 배열 연결 구조가 있었습니다.

`refactoring for synth` 커밋에서는 Conv1, Conv2, Fully Connected의 weight / bias unpacking을 다음과 같이 변경했습니다.

- `reg` array + combinational `always @(*)` loop
- → `wire` array + `generate / genvar` + continuous `assign`

즉 runtime procedural assignment처럼 표현되어 있던 고정 parameter wiring을 elaboration 시 결정되는 정적 연결로 바꾸었습니다.

이 수정은 Conv1 weight/bias, Conv2 weight/bias, FC weight/bias에 동일하게 적용했으며, 이후 ModelSim에서 수정된 RTL을 다시 compile/simulation해 동작을 확인했습니다.

## 6. OpenROAD Flow Scripts + ASAP7 연결

`OpenROAD-flow-scripts` fork에는 `cnn_verilog`를 실제 design source로 등록했습니다.

추가한 구성은 다음과 같습니다.

- `flow/designs/src/cnn_verilog`: RTL repository submodule
- `flow/designs/asap7/cnn_verilog/config.mk`
- `flow/designs/asap7/cnn_verilog/constraint.sdc`
- top module: `chip`
- platform: `asap7`

커밋된 `config.mk`에는 다음 floorplan 값이 남아 있습니다.

- `CORE_UTILIZATION = 22`
- `CORE_ASPECT_RATIO = 1.2`
- `CORE_MARGIN = 6`
- `PLACE_DENSITY = 0.45`
- `CLOCK_PORT = clk`
- `CLOCK_PERIOD = 10.0`

다만 현재 Git 상태에서는 `config.mk`의 `CLOCK_PERIOD=10.0`과 별도 `constraint.sdc`의 `clk_period=400`이 서로 다릅니다. 따라서 실제 flow에서 최종 적용된 clock constraint를 저장소만으로 단정해서는 안 됩니다.

또한 OpenROAD의 generated `results/` 및 report는 이 fork에 커밋되어 있지 않아, 최종 utilization / timing / power 수치는 이 저장소만으로 재검증할 수 없습니다. PPA 수치를 이력서에 사용할 경우 당시 별도 결과 로그나 보고서로 교차검증해야 합니다.

## 7. synthesized netlist 생성과 gate-level 검증 시도

OpenROAD/Yosys 합성 결과를 이용한 것으로 보이는 `chip_synth.v`가 `cnn_verilog/modelsim_synth/`에 남아 있으며, 현재 파일 크기는 약 59 MB입니다. ASAP7 standard-cell simulation model도 함께 저장되어 있습니다.

`tb_compare.v`에서는 같은 image, weight, bias를 다음 두 design에 동시에 입력하도록 구성했습니다.

- RTL `chip`
- synthesized `chip_synth`

그리고 각각의

- `decision`
- `decision_synth`
- accuracy
- RTL/netlist match 여부

를 비교하도록 testbench를 작성했습니다.

다만 Git history에는 이 단계의 커밋이 명시적으로 `synth simulation (failed)`로 남아 있으며, gate-level simulation이 RTL과 동일한 1,000-image accuracy를 달성했다는 완료 로그는 확인되지 않습니다.

따라서 이 경험은 **합성 netlist 비교 환경까지 구축하고 원인을 추적했지만, post-synthesis functional equivalence를 최종 완료했다고 주장하지 않는 것**이 정확합니다.

## Git에서 확인되는 주요 흐름

### 2025년 6월 7일

- 기존 CNN RTL을 별도 `cnn_verilog` 저장소로 분리
- `CNN-Implementation-in-Verilog`에서 submodule로 연결
- OpenROAD Flow Scripts에도 동일 RTL 저장소를 design source submodule로 연결

### 2025년 6월 8일

- PyTorch quantization 구현
- quantized tensor / scale 처리 오류 수정
- weight, bias, intermediate output 재생성

### 2025년 6월 18일

- synthesized Verilog 결과 추가
- ModelSim testbench의 절대경로와 실행 구조 수정
- OpenROAD ASAP7 design config 및 SDC 추가
- child RTL commit 변경 후 OpenROAD parent repository에서 submodule pointer 동기화

### 2025년 6월 25~27일

- synthesis / simulation 환경 반복 수정
- RTL 파일 구조 리팩터링
- synthesis-friendly parameter wiring으로 수정
- 1,000-image RTL simulation 수행: 96%
- `chip_synth` + ASAP7 cell library 기반 gate-level testbench 구성
- RTL / synthesized netlist 비교 시도
- gate-level simulation 문제는 최종 미해결 상태로 기록

## 이력서에서 강하게 쓸 수 있는 내용

- 기존 CNN RTL을 분석해 PyTorch quantized model과 HW integer representation을 연결
- quantized weight / bias 및 layer intermediate output을 reference data로 생성하고 MAC 단위까지 내려가 SW/HW mismatch를 추적
- simulation용 RTL의 parameter wiring을 `generate` 기반 정적 연결로 변경해 synthesis 가능한 구조로 리팩터링
- ModelSim 1,000-image RTL simulation에서 96% classification accuracy 확인
- RTL repository를 Git submodule로 OpenROAD Flow Scripts에 연결하고 ASAP7 synthesis configuration 구성
- synthesized gate-level netlist와 RTL을 동일 입력으로 비교하는 verification environment 구축

## 표현할 때 주의할 부분

다음 표현은 현재 Git 근거보다 강하므로 피합니다.

- `CNN accelerator RTL 전체를 처음부터 설계했다`
- `원본 CNN architecture를 직접 설계했다`
- `post-synthesis accuracy 96%를 달성했다`
- `RTL과 synthesized netlist의 functional equivalence를 최종 검증했다`
- Git에 결과가 없는 특정 PPA 수치를 검증된 최종 결과처럼 제시한다

대신 다음 표현이 현재 소스와 가장 정확합니다.

> 기존 공개 CNN RTL을 baseline으로 PyTorch 양자화 결과와 RTL의 정수 표현을 맞추고, 중간 layer와 MAC 결과를 비교하며 정합성을 검증했습니다. 이후 synthesis에 맞게 RTL을 리팩터링하고 Git submodule로 OpenROAD Flow Scripts의 ASAP7 flow에 연결했습니다. ModelSim에서 RTL 1,000개 입력 기준 96% accuracy를 확인했으며, 합성 netlist와 RTL을 동일 입력으로 비교하는 gate-level 검증 환경까지 구축했습니다.

## 보여주는 역량

- PyTorch quantization과 fixed-point / integer representation 이해
- Python ↔ Verilog reference data 연결
- CNN convolution / MAC 구조 이해
- RTL simulation 및 중간 결과 기반 디버깅
- synthesis-oriented RTL refactoring
- Git submodule을 활용한 multi-repository dependency 관리
- OpenROAD Flow Scripts / ASAP7 integration
- RTL / gate-level verification 흐름 이해
- 검증 완료 범위와 미완료 범위를 구분해 판단하는 엔지니어링 태도

## 자소서 활용 포인트

- AI 모델을 HW 표현으로 옮기는 과정에서 발생한 정합성 문제 해결
- SW와 RTL을 오가며 문제 지점을 단계적으로 좁힌 경험
- simulation에서는 되지만 synthesis에서는 되지 않는 RTL을 수정한 경험
- 처음 접한 Git submodule 구조를 이해하고 실제 flow repository에 연결한 경험
- 합성 이후까지 검증 범위를 확장하면서 실패 원인까지 추적한 경험
