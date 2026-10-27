# CNN Accelerator — 커밋 기반 개발작업 원장

> 조사 시작 **2026-10-27**. 기존 [canonical 경험 문서](cnn-accelerator.md) 및 [커밋 기반 복원 가이드](../guides/commit-based-experience-reconstruction.md)를 따른다. **상태: 3개 저장소 메타데이터 및 SW quantization/reference 4커밋, OpenROAD 연동 3커밋 1차 diff 검토 완료**. RTL 자식 저장소의 13커밋(merge 포함)과 후속 통합 검증, 수치의 원본 로그 대조는 진행 예정. 기존 canonical 및 `resume.md`는 변경하지 않음.

## 1. 프로젝트 범위 / 출처

| 저장소 | 원본과 관계 | 개인 커밋(현재 확인한 기본 브랜치 기준) | 이번 조사 |
|---|---|---|---|
| [`nanocode00/CNN-Implementation-in-Verilog`](https://github.com/nanocode00/CNN-Implementation-in-Verilog) | [`boaaaang/CNN-Implementation-in-Verilog`](https://github.com/boaaaang/CNN-Implementation-in-Verilog)의 fork; 2021 baseline은 타인 작성 | **2025-06-07~08 개인 일반 4개** | **4개 diff 검토**: RTL submodule, PyTorch quantization, fixing, fix done |
| [`nanocode00/cnn_verilog`](https://github.com/nanocode00/cnn_verilog) | RTL 및 ModelSim용 별도 child repo | 기본 브랜치에서 **13개(일반 12 + merge 1)** | **커밋 인벤토리만 1차 확인**. refactor/1000장 TB/Netlist 테스트 상세 diff는 다음 단계 |
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

## 5. 다음 묶음 / 검증 계획

1. `cnn_verilog` 개인 커밋 12개(merge 1개 별도)의 후속 분류: baseline import → ModelSim TB → synthesis refactor(`bcd3bf`·`e6af4816`) → netlist 비교 실패(`091fc2de`). 커밋별 patch·부모·테스트 결과 조사.
2. OpenROAD fork의 `secure-clock-repair-test`, `update-openroad` 분기 전체 페이지를 확인해 본인 개인 추가 커밋 누락 여부 검사. upstream 이력은 본인 구현에서 제외.
3. 96%/1,000장 RTL 정확도, synthesized `1_synth.v`, physical final, slack `-2040.66` 근거를 저장소 로그·발표자료와 다시 대조. 정확도/STA 날짜·실행조건 불명확하면 표시.
4. 3개 저장소 submodule 참조 SHA를 트레이스해 회귀 관계 확인 후 canonical `experiences/cnn-accelerator.md` 보완. `resume.md`는 최종 사례 선정 전 수정하지 않음.
