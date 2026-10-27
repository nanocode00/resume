# CNN Accelerator — 커밋 기반 개발작업 원장

> 조사 시작 **2026-10-27**. 기존 [canonical 경험 문서](cnn-accelerator.md) 및 [커밋 기반 복원 가이드](../guides/commit-based-experience-reconstruction.md)를 따른다. **상태: 3개 저장소 기본 브랜치의 개인 일반 커밋 19개(4+12+3), 자식 RTL merge 1개를 기능별 1차 diff 조사 완료**. 6/27 ModelSim 1,000입력 정확도 **96% 직접 로그 확인**. 추가 OpenROAD 브랜치·최종 STA 원본 자료 대조는 잔여. 기존 canonical 및 `resume.md`는 변경하지 않음.

## 1. 프로젝트 범위 / 출처

| 저장소 | 원본과 관계 | 개인 커밋(현재 확인한 기본 브랜치 기준) | 이번 조사 |
|---|---|---|---|
| [`nanocode00/CNN-Implementation-in-Verilog`](https://github.com/nanocode00/CNN-Implementation-in-Verilog) | [`boaaaang/CNN-Implementation-in-Verilog`](https://github.com/boaaaang/CNN-Implementation-in-Verilog)의 fork; 2021 baseline은 타인 작성 | **2025-06-07~08 개인 일반 4개** | **4개 diff 검토**: RTL submodule, PyTorch quantization, fixing, fix done |
| [`nanocode00/cnn_verilog`](https://github.com/nanocode00/cnn_verilog) | RTL 및 ModelSim용 별도 child repo | 기본 브랜치에서 **13개(일반 12 + merge 1)** | **기본 브랜치 12개 일반 커밋 diff·merge parent 조사 완료**. 6/27 최종 ModelSim 로그 확인, Netlist 비교 실패 이력 구분 |
| [`nanocode00/OpenROAD-flow-scripts`](https://github.com/nanocode00/OpenROAD-flow-scripts) | [OpenROAD Flow Scripts](https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts)의 fork; upstream 전체는 타인 작업 | 기본 브랜치에서 **개인 일반 3개** | **3개 diff 확인**: submodule, ASAP7 config+SDC, pinned revision 갱신 |

**잠정 범위:** 세 저장소 기본 브랜치의 4 + 12 + 3 = **19개 개인 일반 커밋**, merge 1개. OpenROAD fork에는 별도 브랜치가 2개 더 있으므로 이 숫자를 **전체 브랜치 역검증 완료치로 쓰지 않는다**. 팀원·upstream 원작, 모델 데이터·생성 텍스트·합성 산출물은 직접 작성 RTL 라인수로 합산하지 않는다.

## 2. CW-01 — CNN RTL 저장소 분리와 Gitlink 고정 (2025-06-07)

**상황:** 기존 공개 CNN RTL을 PyTorch 정수 모델과 비교하고 별도로 synthesis flow에 투입할 준비.

**문제:** 기존 fork에 배치된 `cnn_verilog/` 하위 RTL·데이터 경로를 부모 프로젝트 및 OpenROAD 양쪽에서 동일 revision으로 다룰 필요가 있었음. **그 동기가 실제로 버전 불일치 장애 때문이었는지는 확인되지 않음**.

**조치(A, Git diff):**
- [`f21aeba3325e`](https://github.com/nanocode00/CNN-Implementation-in-Verilog/commit/f21aeba3325e)에서 `cnn_verilog` 폴더를 `nanocode00/cnn_verilog` submodule로 교체. `.gitmodules`에 URL·path를 추가하고 Gitlink를 자식의 `4326abb6509e`로 고정.
- [`19d9d49667e5`](https://github.com/nanocode00/OpenROAD-flow-scripts/commit/19d9d49667e5)에서 `flow/designs/src/cnn_verilog`의 submodule을 **동일 SHA `4326abb6509e`**로 설정. 동일 RTL baseline을 두 상위 저장소에서 참조하는 구조(A).
- 이 때 대량 삭제로 보이는 diff는 일반 소스 제거라기보다 하위 디렉터리를 **Gitlink 1개로 교체하면서 생긴 변경**을 포함. **5만여 줄의 알고리즘 코드를 본인이 삭제/재구현한 성과로 표현하지 않는다**.

**결과(A):** submodule 설정과 동일 Gitlink SHA를 양쪽에서 확인. 특정 snapshot을 참조했다는 사실을 넘어서 실제 배포/합성 결과를 보장하지 않음.

## 3. CW-02 — PyTorch quantization과 HW reference 생성·오류 수정 (2025-06-08)

**커밋:** [`0815e86b6251`](https://github.com/nanocode00/CNN-Implementation-in-Verilog/commit/0815e86b6251) → [`2faf6277dc61`](https://github.com/nanocode00/CNN-Implementation-in-Verilog/commit/2faf6277dc61) → [`c32db292c824`](https://github.com/nanocode00/CNN-Implementation-in-Verilog/commit/c32db292c824).

**상황:** 공개 2층 MNIST CNN baseline의 Conv1/Conv2/FC 결과를 RTL 정수·hex 입력 형식과 비교할 PyTorch 추출 스크립트가 필요.

**문제(A):** 초기 quantized model에서는 일부 Tensor에 `.detach().numpy()`를 직접 사용하고, Conv2 가중치도 `model.conv2.weight.detach().numpy()`로 접근. Quantized layer의 `weight` 접근 방식과 `int_repr()` 정수 표현이 float Conv API와 달라 **참조 데이터 추출 경로의 정합성 문제가 존재**했다. 실제 stack trace/오류 출력은 아직 확보되지 않았으므로 구체적 exception 이름은 붙이지 않는다.

**조치(A):**
- `0815e86`: `pyTorch/MNIST_CNN_QUANTIZATION.py` 새로 추가. `QuantStub`/`DeQuantStub`와 `fbgemm` 양자화 설정, calibration/convert, Conv1/Conv2/FC의 정수 가중치 `weight().int_repr()`와 scale을 사용한 bias 변환, 음수 8-bit 2's complement 및 `txt_quantized/*.txt` HEX 저장 코드 작성. Conv/Pool/FC 단계의 중간 출력과 Conv2 channel별 5×5 곱셈 누적을 추적할 reference path 추가. **기존 CNN 학습/데이터 흐름은 공개 예제와 타인 baseline을 참고하므로 완전 독창 신규 모델이라고 주장하지 않는다**.
- `2faf627`: forward 중간 출력에서 `x.is_quantized`에 따라 `dequantize().numpy()` 또는 `detach().numpy()`를 선택하도록 수정. Quantized tensor의 `view`를 `reshape`로 변경, `scale.item()` 호출을 해당 `scale` 객체 사용으로 정리. 생성된 `txt_quantized` 중간값·가중치 파일 동반 갱신.
- `c32db292`: Conv2 reference weight를 `model.conv2.weight.detach().numpy()`에서 `model.conv2.weight().int_repr().numpy()`로 수정하고, bias를 `bias().detach()`로 접근. 수동 계산에 사용할 정수 weight 표현으로 재설정. 다시 저장한 `txt_quantized` 파일은 **생성 결과물**이며 각각의 숫자를 수작업으로 코딩한 기여가 아니다.

**판단:** 전체 출력만 비교하기보다 Conv/Pool/FC 단계와 Conv2 channel별 MAC 중간 결과를 꺼내어 비교 가능한 형태로 기록한 선택이 코드로 확인(A). 그러나 이를 택하게 된 특정 실제 오류·실행 성능/정확도 감소량은 이 3커밋만으로 확정 불가.

**결과:** quantized weight extraction·intermediate reference 경로의 **수정 전후 코드(A)**가 명확하다. 해당 revision에서 PyTorch 실행이 전 환경에서 끝까지 성공했다는 CI/완전 실행 로그는 아직 별도 대조 전. 최종 RTL accuracy는 자식 RTL repo 로그를 검증한 뒤 대표 성과로 사용한다.

## 4. CW-03 — OpenROAD Flow Scripts와 ASAP7 설정 (2025-06-07~18)

**커밋:** [`19d9d49667e5`](https://github.com/nanocode00/OpenROAD-flow-scripts/commit/19d9d49667e5) → [`e430e1543d1a`](https://github.com/nanocode00/OpenROAD-flow-scripts/commit/e430e1543d1a) → [`c44590700b3c`](https://github.com/nanocode00/OpenROAD-flow-scripts/commit/c44590700b3c).

**조치(A):**
- `e430e154`에서 `flow/Makefile` 기본 `DESIGN_CONFIG`를 `./designs/asap7/cnn_verilog/config.mk`로 지정하고, CNN 전용 config에 `PLATFORM=asap7`, `DESIGN_NAME=chip`, RTL glob `.../src/cnn_verilog/verilog/*.v` 등을 설정.
- `CORE_UTILIZATION=22`, `CORE_ASPECT_RATIO=1.2`, `CORE_MARGIN=6`, `PLACE_DENSITY=0.45`, `CLOCK_PORT=clk`, config `CLOCK_PERIOD=10.0`을 기록.
- 동일 커밋 `constraint.sdc`에는 실제 `create_clock` 생성용 `clk_period=400` 설정. 따라서 **10ns/100MHz 타이밍을 달성했다고 결론내리면 안 됨**. 기본 config와 적용 SDC의 수치를 구분.
- child Gitlink를 `4326abb6` → `c909e315` → `5f34734c`로 갱신. OpenROAD에서 특정 RTL revision을 참조한 흔적 확인.

**결과(A):** config/SDC/submodule 포인터 동기화는 Git으로 입증. OpenROAD 합성 결과/STA 최종 성과는 기존 발표자료·아티팩트 증거와 후속 조사에서 구분. **설정만으로 physical design 성공·timing closure를 주장하지 않음**.

## 5. CW-04 — 자식 RTL 저장소 초안·테스트벤치·합성 산출물 (2025-06-07~26)

**자료:** [`nanocode00/cnn_verilog`](https://github.com/nanocode00/cnn_verilog) 기본 `master`, 개인 author 고유 일반 커밋 12개 + merge 1개. 본인 commit author 사실과 **원본 RTL의 최초 저작자**는 분리한다.

| SHA / 일자 | 개인 commit의 실제 변화(A) | 개인 기술 기여로 볼 수 있는 범위 |
|---|---|---|
| [`4326abb6509e` 6/7](https://github.com/nanocode00/cnn_verilog/commit/4326abb6509e) | 117파일·63,029행의 **root commit**. 원본 CNN Verilog, weights/images, Vivado xsim 캐시/로그, TB 등 반입 | baseline을 가져온 기록. `conv1` 등 소스 헤더에는 **2021 Bo Young Kang** 저작자 표시가 존재. 반입된 전체 63,029행을 본인 최초 RTL 작성으로 기록하지 않음 |
| [`c909e315352c` 6/18](https://github.com/nanocode00/cnn_verilog/commit/c909e315352c) | `synth/1_synth.v` **1,836,390행 생성물** 반입 | synthesis 생성 산출물 저장. 행수는 개인 직접 RTL 작성량 아님 |
| [`5f34734c749d` 6/18](https://github.com/nanocode00/cnn_verilog/commit/5f34734c749d) | 1,000장 이미지 순회 `top_tb_1000.v` 추가(229행), 기존 단일 이미지 TB 수정 | ModelSim 1,000장 평가를 위한 TB 구성 |
| [`6e8029e8d528` 6/25](https://github.com/nanocode00/cnn_verilog/commit/6e8029e8d528) | 대형 `1_synth.v` 제거, std-cell 라이브러리/비교 TB/ModelSim 환경 추가, 구조 조정 | RTL-vs-synth 비교 경로 도입. ASAP7 외부 라이브러리 파일·netlist는 원본 저작 제외 |
| [`1bf0660e3031` / `a07be30bdd5d` 6/26](https://github.com/nanocode00/cnn_verilog/commit/a07be30bdd5d) | 같은 부모 `6e8029e8d528`를 갖고 **동일 Git tree `ebd719d26a184bc`**를 가리키는 서로 다른 SHA의 `prepare refactoring`. `modelsim/result.log`, `modelsim_synth/chip_synth.v`(약 **1,512,573행**) 추가, 대량 소스/라이브러리 이동 | **동일한 파일 상태를 두 번 독립 개발·성과로 계산하지 않음**. 저장소 정리·벤치 결과 반입. 6/26 merge [`925f9e45202f`](https://github.com/nanocode00/cnn_verilog/commit/925f9e45202f)는 `bcd3bf`와 `1bf0660` 양쪽 부모를 합친 기록 |
| [`878257bbe8c2` 6/26](https://github.com/nanocode00/cnn_verilog/commit/878257bbe8c2) | `chip.v`, `conv1.v`, `conv2.v`, `fc.v` 등을 별도 root source로 추가 | baseline의 copied RTL을 합성/시뮬레이션 작업 대상으로 재배치. 새 알고리즘 설계 전체로 귀속 금지 |
| [`bcd3bf7605a4` 6/26](https://github.com/nanocode00/cnn_verilog/commit/bcd3bf7605a4) | `conv1.v→conv1_layer.v`, `conv2.v→conv2_layer.v`, `fc.v→fully_connected.v`, MaxPool 분리 등 top/module 계층화 | 합성용 파일 분리·계층 구조 보완. rename/delete가 많은 diff를 신규 전체 구현으로 해석하지 않음 |
| [`f3ba7976f958` 6/26](https://github.com/nanocode00/cnn_verilog/commit/f3ba7976f958) | 이전 `modelsim/top_tb*.v` 2파일 삭제 | 신규 통합 TB로 교체하는 중간 정리 |
| [`0c20f29532bc` 6/26](https://github.com/nanocode00/cnn_verilog/commit/0c20f29532bc) | `modelsim/top_tb.v`, `top_tb_1000.v` 재추가/수정, `modelsim/result.log`, 단일 결과·TCL 갱신 | 실제 ModelSim stimulus/로그 결과를 함께 확인할 검증 단위 |
| [`091fc2de3faa` 6/27](https://github.com/nanocode00/cnn_verilog/commit/091fc2de3faa) | `modelsim_synth/chip_synth.v` **수백만 행 교체**, `tb_synth.v` 수정, 커밋 메시지는 **`synth simulation (failed)`** | 합성 후 netlist 비교 시도는 확인, **동작 성공/성능/accuracy는 미확인** |
| [`e6af4816d302` 6/27](https://github.com/nanocode00/cnn_verilog/commit/e6af4816d302) | Conv1·Conv2·FC의 `reg array + integer/always`를 `wire array + generate/genvar + assign`으로 변경. ModelSim 결과 로그 갱신 | 원본 RTL에 대한 가장 뚜렷한 **합성 지향 직접 수정**. commit title이 'refactoring for synth'인 사실과 실제 diff가 일치 |

**Git 그래프 주의:** `1bf0660`과 `a07be30`는 SHA 두 개이지만 동일 부모·동일 tree로 별도 구현 성과 2건이 아니다. 한쪽 브랜치의 `bcd3bf` 수정과 합치면서 merge `925f9e`가 발생함. 개인 일반 커밋 12개 카운트 자체는 유지하되 **코드 내용의 중복을 명시**한다.

## 6. CW-05 — RTL 정확도·컴파일 실패·합성 Netlist 검증 경계

**직접 로그(A):** [`modelsim/result.log`](https://github.com/nanocode00/cnn_verilog/blob/master/modelsim/result.log)에 1st~1000th 입력별 `original value`·`decision`·`Success/Fail` 로그가 있으며 끝에 **`Final Accuracy for 1000 Input Image` → `Accuracy : 96%`**가 명시돼 있다. **이 1,000개 입력/자체 테스트 기준의 RTL 결과**이지 임의 전체 MNIST test set으로 일반화하거나 Netlist 정확도로 확대하지 않는다.

**실제 오류 로그(A):** [`modelsim/result_single.log`](https://github.com/nanocode00/cnn_verilog/blob/master/modelsim/result_single.log)에는 `conv1_layer.v`, `conv2_layer.v`, `fully_connected.v`에서 **`Register is illegal in left-hand side of continuous assignment`**로 컴파일 에러가 발생한 이력이 있다. `e6af4816`의 `reg → wire` 및 `always → generate/assign` 수정은 이 에러 종류와 **직접 부합**한다. 로그 안에는 다른 실험의 Errors 0, Errors 13도 섞여 있으므로 **전체 로그가 단일 연속 성공 run이라는 주장 금지**.

**Gate-level의 한계(A/B):** `modelsim_synth/tb_compare.v`는 RTL `chip`과 synthesized `chip_synth`에 같은 `$readmemh` 입력을 주고 두 `decision`·accuracy·match 수치를 비교하려는 코드지만, `091fc2de` 커밋은 실패로 표시돼 있고 정상 비교 waveform 증거가 없다. **RTL accuracy 96%와 synthesized Netlist 성공은 별개**. 해당 TB의 `match` 개별 비교와 누적 `match_cnt`·표시 표현도 후속 검토 필요, **기준 검증 통과로 확정하지 않음**.

**생성물 규모:** 2025-06-18 `1_synth.v` 약 184만 행, 2025-06-26 `chip_synth.v` 약 151만 행, 2025-06-27 `chip_synth.v` 대규모 재생성 — **생성된 Verilog와 외부 ASAP7 표준셀을 원 작성 RTL 성과나 개발 역량의 코드량으로 계산하지 않음**. 대형 파일 Git 보관 자체가 합성·STA·배선 성공 판정이 아님.

**잠정 성과:** 정수·양자화 reference 생성(CW-02) → 1,000 입력 RTL ModelSim **96%** 및 컴파일 오류 해결(CW-05) → synthesis-oriented weight/bias wiring(CW-04) → Netlist 비교 시도/실패. OpenROAD physical/STA의 최종 값은 발표자료와 별도 확인 후 상위 경험에 반영.

## 7. 툴 Fork과 ORFS 서브모듈의 실제 구조 — 사후 관리 분석

**2025년 6월 7일 생성된 도구 Fork(2026-10-27 조회):**
- [`nanocode00/OpenROAD`](https://github.com/nanocode00/OpenROAD) (부모 `The-OpenROAD-Project/OpenROAD`)
- [`nanocode00/yosys`](https://github.com/nanocode00/yosys) (부모 `The-OpenROAD-Project/yosys`)
- [`nanocode00/OpenSTA`](https://github.com/nanocode00/OpenSTA) (부모 `The-OpenROAD-Project/OpenSTA`)
- [`nanocode00/abc`](https://github.com/nanocode00/abc) (부모 `The-OpenROAD-Project/abc`)

위 **4개 도구 Fork의 `nanocode00` author commit은 각 0개로 조회**됨. 도구 소스를 따로 수정한 개인 개발 성과는 현재 증거 없음. 단, 이는 작성자 필터 조사이며 타인 계정 작성 커밋/동기화 방식까지 별도 조사한 것은 아니다. 원본 ORFS `.gitmodules`의 상대 경로 `../OpenROAD.git`는 **조직 fork가 아니라 사용자 fork에서 clone하면 `nanocode00/OpenROAD`를 참조하도록 상대적으로 해석**될 수 있다. 실제 [사용자 ORFS `.gitmodules`](https://github.com/nanocode00/OpenROAD-flow-scripts/blob/master/.gitmodules)에서도 이 상대 경로가 유지돼 있다. `tools/yosys`는 `../../The-OpenROAD-Project/yosys.git`의 상위 저장소를 가리킴.

**2026년 '다시 한다면'의 제안(D, 과거 개인 성과 아님):** 사용자 설계 RTL·PyTorch bit-accurate 정수 reference·테스트·ORFS 설정·작은 실행 기록을 **하나의 본인 소스 repo**에 두고, 공식 ORFS는 **버전 고정 외부 clone 또는 읽기전용 submodule 1개**로 관리. tool source 수정을 하지 않는 한 OpenROAD·Yosys·OpenSTA·ABC Fork는 만들지 않는다. Docker/container image digest, ORFS Git SHA, simulator/version, 정확한 SDC·clock 값을 manifest에 남기고 generated netlist·표준셀·ModelSim work 파일은 원칙적으로 Git에서 제외. 공식 ORFS는 자체 [Adding New Designs](https://openroad-flow-scripts.readthedocs.io/en/latest/user/AddingNewDesign.html)와 `make DESIGN_CONFIG=...` 방식을 제공한다. 외부 저장소 소스 코드의 재사용·배포 전에는 **해당 라이선스/저작자 표시 확인이 필요**하다.

## 8. 다음 묶음 / 검증 계획

1. **기본 브랜치 RTL 커밋 12개 1차 조사 완료:** 6/7~27 baseline/중간 저장소 정리, ModelSim TB·로그, synth-generated 대용량 netlist, 합성 지향 RTL 패치, Gate-level 비교 실패까지 구분. 추가로 테스트 반복성·출력 자체의 의미가 더 필요한 경우 원본 로그 상세 점검.
2. **OpenROAD fork의 추가 2개 브랜치 `secure-clock-repair-test`, `update-openroad`에 `author=nanocode00` 필터로 조회된 커밋은 각 0개.** 상위 프로젝트 upstream/타인 작업은 제외. 필요시 branch 전체 pagination으로 최종 역검증.
3. **96%/1,000장 RTL 수치는 원본 `modelsim/result.log`에서 직접 확인**. 여전히 OpenROAD physical final, slack `-2040.66`, Netlist 비교 시도/실패의 발표자료·실험 조건을 역검증하고 시점/환경을 구분.
4. 3개 저장소 submodule 참조 SHA를 트레이스해 회귀 관계 확인 후 canonical `experiences/cnn-accelerator.md` 보완. `resume.md`는 최종 사례 선정 전 수정하지 않음.
