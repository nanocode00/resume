# CNN Accelerator — PyTorch 양자화부터 OpenROAD 합성까지

## 한 줄 요약

기존 공개 CNN RTL을 baseline으로 분석한 뒤, PyTorch Quantization 결과를 HW용 정수 데이터로 변환하고 RTL을 synthesis 가능한 형태로 리팩터링했습니다. ModelSim에서 MNIST 1,000개 입력 기준 RTL 정확도를 검증하고, RTL 저장소를 Git submodule로 OpenROAD Flow Scripts에 연결해 ASAP7 synthesis와 physical design, gate-level 검증까지 시도한 3인 팀 프로젝트입니다.

## 기간 및 프로젝트 정보

- GitHub에서 직접 작업이 확인되는 기간: **2025년 6월**
- 경북대학교 전자공학부 논리회로설계 기말 프로젝트
- 3인 팀: Ahn Jae Ho, Hwa Jin Kim, Jae Hun Kim
- 최종 발표 제목: `CNN Accelerator RTL to Netlist`

최종 발표자료에서는 세 저장소의 역할을 다음과 같이 구분했습니다.

- `nanocode00/OpenROAD-flow-scripts`: 합성을 위한 설정 파일
- `nanocode00/CNN-Implementation-in-Verilog`: Quantization 학습 및 SW reference 생성
- `nanocode00/cnn_verilog`: Verilog RTL 및 simulation

## 소스와 기여 범위

이 프로젝트는 기존 공개 프로젝트 `CNN-Implementation-in-Verilog`의 2-layer MNIST CNN을 baseline으로 사용했습니다. 저장소 README와 2021년 원본 RTL은 원저작자의 작업이므로 개인 구현으로 포함하지 않습니다.

2025년 작업은 다음 세 저장소로 분리되어 있습니다.

```text
CNN-Implementation-in-Verilog
├─ pyTorch/                  # Quantization, HW reference data 생성
└─ cnn_verilog -------------┐
                            │ Git submodule
nanocode00/cnn_verilog <----┘
├─ baseline/                 # 원본 RTL 보존
├─ RTL 수정본
├─ modelsim/                 # RTL simulation
└─ modelsim_synth/           # synthesized netlist 검증 환경

OpenROAD-flow-scripts
└─ flow/designs/src/cnn_verilog
   └─ nanocode00/cnn_verilog의 특정 commit을 가리키는 submodule
```

따라서 이 경험은 CNN accelerator RTL 전체를 처음부터 설계한 프로젝트라기보다, **기존 CNN RTL을 이해하고 PyTorch 정수 모델과 맞춘 뒤 synthesis 가능한 RTL로 수정해 ASIC flow와 검증 환경까지 연결한 HW/SW co-design 경험**으로 설명하는 것이 정확합니다.

## baseline CNN 구조

RTL top module인 `chip`은 다음 순서로 동작합니다.

`Conv1 → MaxPool/ReLU → Conv2 → MaxPool/ReLU → Fully Connected → Comparator`

- 입력: 28×28 grayscale image를 8-bit stream으로 전달
- Conv1: 1 input channel → 3 output channels, 5×5 kernel
- Conv2: 3 input channels → 3 output channels, 5×5 kernel
- Fully Connected: 48 → 10
- Comparator: 0~9 classification 결과 출력

이 구조 자체는 upstream baseline이며, 아래 내용부터가 Git과 최종 산출물로 확인되는 2025년 작업입니다.

## 1. PyTorch Quantization 및 HW reference data 생성

2025년 6월 8일 `quantization`, `fixing`, `fix done` 커밋에서 PyTorch 모델의 Quantization과 HW 입력 데이터 생성 과정을 수정했습니다.

주요 내용:

- PyTorch `QuantStub` / `DeQuantStub` 적용
- `fbgemm` backend를 사용한 Quantization 준비 및 변환
- quantized weight의 `int_repr()`로 실제 integer representation 추출
- quantized scale을 반영해 weight / bias를 HW용 정수 표현으로 변환
- 음수를 8-bit 2's complement 형태로 변환
- Conv1 / Conv2 / FC weight와 bias를 hexadecimal text로 저장
- Conv1, MaxPool1, Conv2, MaxPool2, FC 중간 입출력을 reference file로 저장

수정 과정에서는 float Tensor와 quantized Tensor를 같은 방식으로 처리하면서 발생한 문제를 해결하기 위해 `dequantize()`, `int_repr()`, layer scale을 구분해 사용했습니다.

최종 발표에서도 전력 최적화 후보 중 실제 프로젝트에서 시도한 방법을 **Quantization**으로 명시했습니다.

## 2. 중간 레이어와 MAC 단위 비교

최종 classification 결과만 비교하지 않고 PyTorch에서 각 layer의 중간 출력을 저장했습니다.

특히 Conv2 계산에서는 다음 데이터를 별도로 확인하는 코드가 있습니다.

- 3개 input channel의 5×5 window
- channel별 5×5 weight
- channel별 MAC 결과
- 3개 channel MAC의 합
- bias 적용 결과
- 실제 PyTorch Conv2 output

이를 통해 SW와 RTL 결과가 다를 경우 최종 inference 결과만 보는 대신 **어느 layer, 어느 channel의 convolution 연산에서 차이가 시작되는지 단계적으로 좁혀가는 방식**으로 디버깅했습니다.

## 3. RTL 저장소 분리와 Git submodule

2025년 6월 7일 `CNN-Implementation-in-Verilog`에 `.gitmodules`를 추가하고 RTL 작업을 별도 `nanocode00/cnn_verilog` 저장소로 분리했습니다.

부모 저장소에는 RTL 파일 전체가 복사되는 것이 아니라 다음 정보가 저장됩니다.

- `.gitmodules`: submodule repository URL
- `cnn_verilog` gitlink: 사용할 child repository의 정확한 commit SHA

같은 방식으로 `OpenROAD-flow-scripts`의 `flow/designs/src/cnn_verilog`에도 RTL 저장소를 submodule로 연결했습니다.

작업 중 child RTL repository에서 testbench를 수정한 뒤 OpenROAD 부모 저장소의 submodule pointer를 새 SHA로 올린 기록도 있습니다. 따라서 RTL과 ASIC flow 환경을 독립적으로 관리하면서 특정 RTL revision을 고정해 사용할 수 있는 구조를 실제로 경험했습니다.

### 현재 저장소를 볼 때 주의할 점

OpenROAD 저장소가 마지막으로 가리키는 `cnn_verilog` revision에는 `verilog/` 디렉터리가 존재합니다. 이후 child `cnn_verilog` 저장소에서는 RTL을 repository root로 옮기고 파일 구조를 추가로 리팩터링했습니다.

즉 현재 두 repository의 최신 tree만 비교하면 경로가 맞지 않아 보이지만, 당시 OpenROAD flow는 **submodule pointer가 고정한 과거 child revision**을 사용했습니다.

## 4. ModelSim RTL 검증

기존 testbench의 `C:/cnn_verilog/...` 같은 절대경로를 repo-relative path로 수정하고 single-image / 1,000-image 검증 환경을 정리했습니다.

검증 환경:

- `modelsim/top_tb.v`: 단일 입력 검증
- `modelsim/top_tb_1000.v`: 1,000개 입력 classification 검증
- weight / bias를 `$readmemh`로 로드
- expected label과 RTL decision 비교
- 최종 accuracy 계산

현재 Git에 남은 마지막 ModelSim 로그에서 RTL은 MNIST 1,000개 입력에 대해 **96% accuracy**를 기록했습니다.

최종 발표자료에는 실행 시점에 따라 96%와 97% 결과가 함께 남아 있습니다. 이력서와 자소서에서는 Git에 재현 가능한 마지막 로그와 일치하는 **96%를 대표값으로 사용**합니다.

## 5. synthesis를 위한 RTL refactoring

원본 RTL은 simulation에서는 동작했지만 synthesis 과정에서 문제가 되는 배열 연결 표현이 있었습니다.

`refactoring for synth` 커밋에서는 Conv1, Conv2, Fully Connected의 weight / bias unpacking을 다음과 같이 변경했습니다.

- `reg` array + combinational `always @(*)` loop
- → `wire` array + `generate / genvar` + continuous `assign`

고정 parameter wiring을 procedural assignment가 아니라 elaboration 시 결정되는 정적 연결로 변경한 것입니다.

이 수정은 Conv1 weight/bias, Conv2 weight/bias, FC weight/bias에 적용했고, 이후 ModelSim에서 수정된 RTL을 다시 compile/simulation했습니다.

## 6. OpenROAD Flow Scripts + ASAP7 연결

`OpenROAD-flow-scripts` fork에 `cnn_verilog`를 design source로 등록했습니다.

구성:

- `flow/designs/src/cnn_verilog`: RTL repository submodule
- `flow/designs/asap7/cnn_verilog/config.mk`
- `flow/designs/asap7/cnn_verilog/constraint.sdc`
- top module: `chip`
- platform: `asap7`

CNN용 `config.mk`에 남은 값:

- `CORE_UTILIZATION = 22`
- `CORE_ASPECT_RATIO = 1.2`
- `CORE_MARGIN = 6`
- `PLACE_DENSITY = 0.45`
- `CLOCK_PORT = clk`
- `CLOCK_PERIOD = 10.0`

### Clock constraint 주의

`config.mk`에는 `CLOCK_PERIOD=10.0`이 있지만 별도 `constraint.sdc`에는 `clk_period=400`이 남아 있고, 최종 발표의 timing report도 400 기준으로 생성된 것으로 보입니다.

따라서 이 프로젝트를 설명할 때 **10 ns clock에서 timing을 만족했다거나 100 MHz 동작을 검증했다고 주장하지 않습니다.**

## 7. Synthesis 및 physical design 결과

최종 발표자료에는 OpenROAD에서 생성된 `1_synth.v`와 `asap7/cnn_verilog/base - 6_final - chip` 결과 화면이 남아 있습니다. 따라서 CNN RTL을 OpenROAD flow에 연결해 synthesis 및 physical design 산출물을 생성한 것까지는 확인할 수 있습니다.

다만 **timing closure는 달성하지 못했습니다.** 최종 단계 timing 화면에는 negative slack과 `VIOLATED` 상태가 남아 있습니다.

따라서 이 경험은 다음처럼 구분합니다.

- OpenROAD ASAP7 flow 구성: 완료
- synthesized Verilog 생성: 완료
- physical design final stage까지 실행: 완료
- timing report 확인: 완료
- timing closure: **미완료**

일부 중간 report에는 timing과 power 값이 존재하지만, constraint가 서로 다른 흔적이 있기 때문에 이를 최종 ASIC 성능 수치로 이력서에 사용하지 않습니다.

## 8. Gate-level RTL / Netlist 비교 환경

`cnn_verilog/modelsim_synth/`에는 합성 결과인 대용량 `chip_synth.v`, ASAP7 standard-cell simulation model, `tb_synth.v`, `tb_compare.v`가 남아 있습니다.

`tb_compare.v`는 같은 image, weight, bias를 다음 두 design에 동시에 입력합니다.

- RTL `chip`
- synthesized `chip_synth`

각각의

- `decision`
- `decision_synth`
- accuracy
- RTL / netlist match 여부

를 비교하도록 구성했습니다.

즉 gate-level 검증 환경 자체는 실제로 구축했습니다.

## 9. Netlist 정확도 검증은 최종 미완료

Git history에는 이 단계의 commit이 `synth simulation (failed)`로 남아 있으며, 최종 발표자료에도 다음과 같이 명시되어 있습니다.

> Quantization 방법으로 진행하였으나 최종적으로 올바른 파형을 얻지 못해 정확도를 확인할 수 없었다.

따라서 다음과 같은 표현은 사용하지 않습니다.

- post-synthesis accuracy 96% 달성
- RTL과 synthesized netlist의 functional equivalence 검증 완료
- synthesis 전후 정확도 유지 확인

정확한 설명은 **동일 입력으로 RTL과 synthesized netlist를 비교하는 환경을 구축했지만, gate-level simulation에서 정상 파형을 확보하지 못해 Netlist 정확도 검증은 완료하지 못했다**입니다.

## 10. TOPS/W와 PPA 평가 범위

최종 발표에는 `Netlist TOPS/W Calculation` 단계가 포함되어 있지만 유효한 최종 TOPS/W 값까지 도출되지는 않았습니다.

프로젝트에서 확인한 것은 다음 범위입니다.

- OpenROAD synthesis / physical design 실행
- timing / power report 확인
- TOPS/W 계산에 필요한 MAC 수, latency, power 관계 검토

따라서 **PPA 또는 TOPS/W 평가를 완료했다고 표현하지 않고, 관련 report를 분석하고 계산을 시도했다**고 설명합니다.

## 11. 보관 ZIP의 수치를 사용할 때 주의

Google Drive의 `openroad_project.zip`은 당시 작업 자료이지만 내부 구성이 완전히 CNN 전용은 아닙니다.

ZIP의 일부 `config.mk`와 layout image는 `aes_cipher_top` / AES 예제에 해당하며, 여기에는 `CORE_UTILIZATION=40`, `PLACE_DENSITY=0.65` 등의 값이 들어 있습니다.

반면 CNN용 OpenROAD repository와 최종 발표 화면에 남은 설정은 `CORE_UTILIZATION=22`, `PLACE_DENSITY=0.45`입니다.

따라서 **40 / 0.65 값을 CNN Accelerator의 최종 설정이나 성과로 사용하지 않습니다.**

## Git 및 산출물에서 확인되는 주요 흐름

### 2025년 6월 7일

- 기존 CNN RTL을 별도 `cnn_verilog` 저장소로 분리
- `CNN-Implementation-in-Verilog`에서 submodule로 연결
- OpenROAD Flow Scripts에도 동일 RTL 저장소를 design source submodule로 연결

### 2025년 6월 8일

- PyTorch Quantization 구현
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
- 1,000-image RTL simulation 수행: **96%**
- OpenROAD synthesis 및 physical design 수행
- `chip_synth` + ASAP7 cell library 기반 gate-level testbench 구성
- RTL / synthesized netlist 비교 시도
- 정상적인 Netlist waveform 확보 실패
- Netlist accuracy 및 timing closure는 최종 미완료

## 이력서에서 강하게 쓸 수 있는 내용

- 기존 CNN RTL을 분석해 PyTorch quantized model과 HW integer representation 연결
- quantized weight / bias 및 layer intermediate output을 reference data로 생성하고 MAC 단위까지 내려가 SW/HW mismatch 추적
- simulation용 RTL의 parameter wiring을 `generate` 기반 정적 연결로 변경해 synthesis 가능한 구조로 리팩터링
- ModelSim MNIST 1,000-image RTL simulation에서 **96% classification accuracy** 확인
- Git submodule로 RTL repository를 OpenROAD Flow Scripts에 연결하고 ASAP7 design configuration 구성
- OpenROAD synthesis 및 physical design final stage까지 수행하고 timing / power report 분석
- synthesized gate-level netlist와 RTL을 동일 입력으로 비교하는 verification environment 구축
- gate-level 검증 실패까지 포함해 검증 완료 범위와 한계를 명확히 판단

## 표현할 때 주의할 부분

다음 표현은 현재 자료보다 강하므로 피합니다.

- `CNN accelerator RTL 전체를 처음부터 설계했다`
- `CNN architecture를 직접 설계했다`
- `10 ns / 100 MHz timing closure를 달성했다`
- `post-synthesis accuracy 96%를 달성했다`
- `RTL과 synthesized netlist의 functional equivalence를 최종 검증했다`
- `TOPS/W 또는 PPA 평가를 완료했다`
- `CORE_UTILIZATION=40`, `PLACE_DENSITY=0.65`를 CNN 최종 설정이라고 제시한다

대신 다음 표현이 현재 Git과 최종 발표자료를 모두 만족합니다.

> 기존 공개 CNN RTL을 baseline으로 PyTorch Quantization 결과와 RTL의 정수 표현을 맞추고, 중간 layer와 MAC 결과를 비교하며 정합성을 검증했습니다. 이후 synthesis에 맞게 RTL을 리팩터링하고 Git submodule로 OpenROAD Flow Scripts의 ASAP7 flow에 연결했습니다. ModelSim에서 MNIST 1,000개 입력 기준 RTL 정확도 96%를 확인했고 OpenROAD synthesis와 physical design까지 수행했습니다. 또한 RTL과 합성 Netlist를 동일 입력으로 비교하는 gate-level 검증 환경을 구축했으나, 최종 Netlist simulation에서는 정상 파형을 확보하지 못해 정확도 검증과 timing closure까지 완료하지는 못했습니다.

## 보여주는 역량

- PyTorch Quantization과 fixed-point / integer representation 이해
- Python ↔ Verilog reference data 연결
- CNN convolution / MAC 구조 이해
- RTL simulation 및 중간 결과 기반 디버깅
- synthesis-oriented RTL refactoring
- Git submodule을 활용한 multi-repository dependency 관리
- OpenROAD Flow Scripts / ASAP7 integration
- synthesis / physical design / gate-level verification 흐름 이해
- 성공한 범위와 미완료 범위를 분리해 판단하는 엔지니어링 태도

## 자소서 활용 포인트

- AI 모델을 HW 표현으로 옮기는 과정에서 발생한 정합성 문제 해결
- SW와 RTL을 오가며 문제 지점을 단계적으로 좁힌 경험
- simulation에서는 되지만 synthesis에서는 문제가 되는 RTL을 수정한 경험
- 처음 접한 Git submodule 구조를 이해하고 실제 ASIC flow repository에 연결한 경험
- 합성 이후까지 검증 범위를 확장하고 실패 원인과 검증 한계를 확인한 경험
