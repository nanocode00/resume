# Verilog Single-cycle CPU — 커밋 기반 개발작업 원장

> 조사 시작 2026-10-27. 원본: [nanocode00/MIPS-CPU](https://github.com/nanocode00/MIPS-CPU) (**[yuxincs/MIPS-CPU](https://github.com/yuxincs/MIPS-CPU) fork**). **상태: Fork 이후 개인 일반 커밋 5개 및 전체 기능별 diff·최종 코드·당시 보고서의 1차 검토 완료.** 특히 CP0/ERET의 정적 불일치(A)와 **예외 진입 당시 EPC 복귀 주소 저장이 되지 않아 시간 부족으로 미해결 종료했다는 사용자 회고(C, 2026-10-27)**를 함께 기록. 남은 일은 당시 시험 입력 등 증거 및 판단 이유 보강. 본 조사 결과를 [canonical `verilog-cpu.md`](verilog-cpu.md) v1에 통합했으며, **`resume.md`만 미수정**. 이 원장은 세부 커밋 근거를 보존한다.

## 1. 범위, 소스, 기여 경계

- **원본 저장소:** `nanocode00/MIPS-CPU`. 2025-05-02 `yuxincs/MIPS-CPU`에서 포크. 현재 기본 브랜치 `main` **1개**. 포크에서 개인 `nanocode00` author 커밋은 **고유 일반 커밋 5개**, 2025-05-09~05-28. 그 직전 기준 원본 SHA는 `c9961f5376af`(2022-07-17)다. 그 이전의 2016~2022 원본 저장소 커밋(회로·프로그램·자료)은 **본인의 직접 작성 이력으로 계산하지 않는다**.
- **A (Git):** `modelsim/`에 추가한 Verilog RTL, ModelSim 시뮬레이션 스크립트·테스트벤치, 추가·수정·삭제 내역.
- **A (당시 보고서):** [MIPS-CPU RTL 설계 및 검증 보고서](https://drive.google.com/file/d/1sXOy8DC9khJu00A1rDyuSzUtrK_np_2S/view), [동일 내용 DOCX](https://drive.google.com/file/d/1Qfh27NjfegX52KMPx5p6CIF12r_L1ssc/view). 보고서에 ModelSim 단위·Top 파형 분석 및 ALUop/RegDst/Control 오류 수정 기록. **보고서의 '완전 검증' 일반 표현은 각 테스트벤치와 별도로 교차 검증할 것**.
- 원본 Logisim 회로·`programs/`의 benchmark/exception service 어셈블리는 **참고·재구현 기준**. 포크 이후에 추가된 `modelsim/benchmark.hex`, `exception_service.hex`의 내용과 원본 어셈블리 관계는 후속 정리에서 확인. 컴파일 결과·자동 테스트 통과가 단지 파일 존재로 증명되는 것은 아니다.

## 2. 직접 커밋 전체 인벤토리(누락 방지)

| SHA | UTC 작성일 | 직접 Git 변경 | 분석 단계 / 문제 단위 |
|---|---|---|---|
| [`2f059fde7be0`](https://github.com/nanocode00/MIPS-CPU/commit/2f059fde7be0) | 2025-05-09 | `Opcode_Decoder.v`, `ALU_Decoder.v`, `Funct_Decoder.v`, `RegisterRead_Decoder.v`, `Control.v` 최초 추가. ModelSim 환경·hello/wave 시험 파일·Logisim Windows 실행파일 반입 | **CPU-01 Decoder/Control** 최초 구현 1차 diff 확인. 초판 환경 파일·실행파일은 코드 개발 성과와 별도 |
| [`74a5eee30fa5`](https://github.com/nanocode00/MIPS-CPU/commit/74a5eee30fa5) | 2025-05-24 | `ALU.v` 183줄, `Adder.v`, `Subtractor.v` 초판 추가 | **CPU-02 ALU/Arithmetic** 최초 구현 1차 diff 확인 |
| [`538b4339f383`](https://github.com/nanocode00/MIPS-CPU/commit/538b4339f383) | 2025-05-24 | `RegisterFile.v` 34줄 최초 추가 | **CPU-03 Register File** 최초 구현 1차 diff 확인 |
| [`4e8a72fdcaed`](https://github.com/nanocode00/MIPS-CPU/commit/4e8a72fdcaed) | 2025-05-28 | `SingleCycleCPU.v`, CP0, IM/DM, Syscall, Statistics, 테스트벤치·파형 스크립트 추가. Decoder·ALU·Register File 수정. `verilog/` 별도 복제·ModelSim 산출물도 추가 | **CPU-01~07 기능별 검토 완료**. CP0/ERET은 EPC 저장 미완성(C)과 정적 조건 불일치(A) 보존. 커밋 규모만으로 독립 검증/작성량 추정 금지 |
| [`750d76842c21`](https://github.com/nanocode00/MIPS-CPU/commit/750d76842c21) | 2025-05-28 | 앞서 별도로 생성한 `verilog/` 소스·설정·HEX·TB 전체 삭제, `modelsim/SingleCycleCPU.mpf` 수정 | **CPU-08 결과물 정리/중복 제거**. **본인 RTL 전체 삭제가 아님**. 최종 `modelsim/`에는 RTL/TB가 남음 |

**계수 주의:** 위 표의 커밋은 모두 하나의 `main` 선형 히스토리 위에 있고, 원본의 upstream 커밋과 구별. 실제 **문제 해결 작업 ID 수 ≠ 커밋 수**.

## 3. CPU-01 — Opcode/Funct/Control Decoder: 초판 → 통합 수정

**상황:** 기존 Logisim CPU 동작을 Verilog의 opcode/funct 판별과 datapath 제어 신호로 옮기려는 초기 개발 단계.

**문제 (코드·보고서):**
- 첫 버전(`2f059f`)은 명령어를 `!op[0] && ... && op[5]`처럼 개별 비트 조건으로 판별. `[5:0]` Verilog 인덱스 해석과 적어둔 `//000010`, `//100000` 6비트 코드 주석이 **일치하지 않을 수 있는 초판**이며, 5/28 여러 Decoder가 `op == 6'b...` 및 `Funct == 6'b...` 판별로 대규모 변경된 것이 직접 확인됨.
- 초기 Decoder에는 중복 `BEQ` wire 선언과 `ALUop` 벡터의 `[3:0]` 표기가 있음. 최종 `[0:3]` ALU 입력·Control과 비트 순서를 맞춰야 하는 통합 위험 존재.
- **보고서(A: 당시 작성 설명)는 `ALUop` decoding/Control 설정 문제를 파형 분석으로 수정했다고 기록**. 어떤 특정 test case에서 잘못된 값을 봤는지, 원인이 오직 인덱싱이었는지는 로그 없음. 즉 코드 변경의 사실을 **확정된 당시 장애 원인**으로 과장하지 않는다.

**조치 (Git):**
- 5/09: `Opcode_Decoder`, `Funct_Decoder`, `ALU_Decoder`, `RegisterRead_Decoder` 및 `Control` 최초 작성. `Control`에서 R-type과 그 외 opcode 경로를 ALU 선택 신호로 결합.
- 5/28: 모든 Decoder에서 명시적인 `6'b...` 값 비교를 사용하도록 변경, 중복 `BEQ` 제거, ALU control vector `[3:0]`→`[0:3]` 변경. `ALU_Decoder`의 ORI 조건을 `ALUop[0]`에 반영, `Control`을 중간 custom mux module 대신 삼항 연산으로 정리.
- `Opcode_Decoder`의 `ALUSrc`/`RegWrite` OR 식에서 중복 ADDI 제거는 **중복 조건 정리**. 성능 최적화나 실제 실패 수정이라고 별도로 단정하지 않음.

**결과:** `modelsim/` 최종 Decoder가 6비트 코드 비교 및 `[0:3]` ALU 제어 경로를 사용한다는 사실(A). `tb_Control.v`·`wave_Control.tcl`과 당시 보고서에서 ADD/JR/SYSCALL/LW/SW/BEQ/J/JAL 명령의 신호 관찰 근거(A/B). 해당 TB는 **`$display` 출력 기반**이어서 자동 pass/fail assertion 회귀 검증이 증명되지는 않음.

**소유권:** `modelsim/` RTL의 작성/수정은 본인 commit에 확인. 명령어 집합·원본 Logisim datapath 아이디어를 전부 창작했다고 주장하지 않는다.

**남은 판단 질문:** 디코더 초기의 개별 bit 판별을 equality 비교로 전환한 **결정적인 계기**가 파형에서 잘못된 opcode 인식이었는지, RTL 표현을 단순화하는 과정이었는지? 당시 기억이 없으면 **원인 미확정**.

## 4. CPU-02 — ALU·가감산·부호 비교: 초판 → 통합 수정

**상황:** 32비트 ALU에서 shift, add/sub, 논리·비교, 곱셈·나눗셈을 control code로 선택하도록 구현.

**문제 (Git에 남은 초판→후속 차이):**
- 5/24 초판 `ALU.v`에서 `Result1/Result2/CF/OF`가 `output`으로만 선언돼 있지만 `always @(*)` 절에서 값을 할당한다. 통합판(`4e8a72f`)에서는 이를 **`output reg`로 수정**. 즉 초판을 최종 컴파일/검증 완료본으로 취급하면 안 된다.
- `Multiplier`, `Divider`, `Adder`, `Subtractor`의 연결되지 않은 세 번째 입력을 통합판에서 `1'b0`으로 명시. `Shifter`, `Multiplier`, `Divider`, signed/unsigned comparator를 ALU 내부 정의에서 **별도 파일로 분리**.
- 5/24 `Adder/Subtractor`는 `fadd/fsub` 셀 배열과 carry/borrow 체인으로 구성. 5/28 각각 `{1'b0,X} + {1'b0,Y} + cin`, `{1'b0,X} - {1'b0,Y} - bin`의 33비트 확장 산술 표현으로 변경하고 overflow를 **피연산자와 결과 sign bit 관계**로 계산하도록 교체. 성능 수치/검증은 별도 없음.
- `ALU`의 control code 포트 `[3:0]`→`[0:3]`는 Decoder/Top과 통일되는 방향으로 변경. `lt_signed`와 `lt_unsigned`를 비교기 결과에서 직접 `{31'b0, lt_*}`로 확장.

**조치:** [5/24 ALU 초판](https://github.com/nanocode00/MIPS-CPU/commit/74a5eee30fa5) → [5/28 수정·통합](https://github.com/nanocode00/MIPS-CPU/commit/4e8a72fdcaed)의 정확한 파일 diff에 해당.

**결과:** 최종 `modelsim/ALU.v`, `Adder.v`, `Subtractor.v`의 선언·연산·모듈 연결이 초판보다 일관되게 정리됨(A). [`tb_ALU.v`](https://github.com/nanocode00/MIPS-CPU/blob/main/modelsim/tb_ALU.v)와 `wave_ALU.tcl` 등 ModelSim 자산을 `4e8a72f`에서 추가. 당시 보고서는 ADD/SUB/SLL/SRA 등의 파형 기능 검증을 서술하지만, **시험 케이스 전체 자동 통과 로그·비교 수치 없음**. `tb_ALU.v`는 일부 입력값을 넣고 `$display`로 결과를 기록할 뿐 `assert`를 수행하지 않는다.

**남은 판단:** 왜 ripple-style `fadd/fsub`에서 확장 산술 표현으로 교체했는지(연산 결과·carry/overflow 오류인지, 구현 간결화인지)는 보고서의 일반적 설명만으로 확정 불가.

## 5. CPU-03 — Register File: 32×32bit, 2R/1W, $zero

**상황:** Decoder·ALU와 함께 CPU의 register read/write 저장소를 구현하는 단계.

**조치:**
- [5/24 `538b433`](https://github.com/nanocode00/MIPS-CPU/commit/538b4339f383): `reg [31:0] registers [31:0]`, 두 combinational read port `reg1/reg2`, positive-edge synchronous write(WE=1 및 `write_reg_number != 0`) 구현. `$zero` 읽기는 조건식으로 0으로, 쓰기는 조건으로 차단. `$a0`/`$v0` 및 추가 `$s0`/`$s1`/`$s2`/`$ra`의 별도 관측 출력 추가.
- [5/28 `4e8a72f`](https://github.com/nanocode00/MIPS-CPU/commit/4e8a72fdcaed): `initial` 반복문으로 32개 register 0 초기화. `$s0`/`$s1`/`$s2`/`$ra`의 별도 출력 제거, 최종 별도 `$a0`/`$v0` 출력만 유지.

**결과:** 최종 [`modelsim/RegisterFile.v`](https://github.com/nanocode00/MIPS-CPU/blob/main/modelsim/RegisterFile.v)에 32×32비트, 2읽기/1쓰기 및 `$zero` 보호와 시뮬레이션 초기화 로직이 존재(A). [`tb_RegisterFile.v`](https://github.com/nanocode00/MIPS-CPU/blob/main/modelsim/tb_RegisterFile.v)는 `$a0`/ `$v0`에 쓰고 이를 read port로 조회한 뒤 `$display` 출력. **모든 레지스터/모든 지연조건을 포괄하는 자동 회귀가 아니다.**

**남은 판단:** 초기 관측 포트 `$s0`·`$ra` 등의 제거는 단순 인터페이스 간소화인지, top-level 통합 연결 수정인지 당시 결정 이유 미확정.

## 6. CPU-04 — 최상위 SingleCycleCPU 데이터 경로와 PC 갱신 (2025-05-28)

**개인 구현(A):** [`4e8a72fdcaed`](https://github.com/nanocode00/MIPS-CPU/commit/4e8a72fdcaed)에서 [`modelsim/SingleCycleCPU.v`](https://github.com/nanocode00/MIPS-CPU/blob/main/modelsim/SingleCycleCPU.v) 121줄이 최초 추가됐다. 원본 Logisim 회로를 Verilog datapath로 옮긴 본인 커밋이다.

**상황:** Decoder/ALU/RegFile 단위 모듈을 하나의 명령어 실행 흐름으로 조합하는 통합 단계.

**조치(A):**
- `PC`, `PC_plus_4 = PC+4`, `InstructionMemory(PC[10:2], PC[11], Inst)`, `Control`, `RegisterFile`, `ImmediateExtender`, `ALU`, `DataMemory`를 순차 연결.
- PC 분기: `IsJR`면 `R1`, branch 명령과 `Equal ^ BneOrBeq` 조건이 일치하면 `PC_plus_4 + (ImmExt<<2)`, Jump/JAL이면 `{PC_plus_4[31:28],Inst[25:0],2'b00}`, 일반은 `PC+4`. 예외 `HasExp`에서는 주소 `0x00000800`; `IsEret && IsCOP0`에서는 CP0의 `PCout` 선택 **코드 경로**가 존재.
- `RW = IsJAL ? 31 : (RegDst ? rd : rt)`, `Din = IsCOP0 ? Dout : (IsJAL ? PC+4 : (MemtoReg ? Data : ALU_Result))`, `WE = IsCOP0 ? ExRegWrite : RegWrite`로 목적 레지스터/Write-back 경로 구성.
- `X = IsShamt ? R2 : R1`, `Y = IsShamt ? {27'b0, Inst[10:6]} : (ALUSrc ? ImmExt : R2)`의 ALU 입력 다중화, Data Memory에 `ALU_Result[11:2]` 주소를 사용.
- PC 업데이트용 `clk_PC`는 `((IsEret && IsCOP0) || HasExp) ? ~clk : clk`으로 반전되도록 설계돼 있다. 이는 **일반 클럭과 예외/복귀 클럭 에지 변경 코드가 존재한다는 사실**이지, 타이밍이 검증됐다는 의미는 아니다.

**검증(A/B):** [`tb_SingleCycleCPU.v`](https://github.com/nanocode00/MIPS-CPU/blob/main/modelsim/tb_SingleCycleCPU.v)는 클럭을 생성하고 `ExpSrc=3'b000` 고정으로 J/R/I, TotalCycles, Hex를 `$display` 출력한다. [`wave_SingleCycleCPU_syscall.tcl`](https://github.com/nanocode00/MIPS-CPU/blob/main/modelsim/wave_SingleCycleCPU_syscall.tcl)은 Inst, R1/R2, ALUop, ALU X/Y, RegDst, WE, Hex/Halt를 함께 관찰하도록 설정. 제출 보고서가 waveform 확인을 설명하나, **이 TB는 exception 입력을 변화시키지 않으며 자동 pass/fail 검증기가 아니다**.

**결과:** RTL에 일반 PC·branch/jump·writeback·syscall/메모리 연결 경로가 존재한다(A). 다만 복잡한 CP0 exception/ERET 경로의 실제 기능·timing은 다음 CPU-05에서 별도로 평가; 전체 CPU가 모든 명령을 올바르게 수행했다고 무제한 주장하지 않음.

## 7. CPU-05 — CP0/ERET 설계와 보존된 RTL·테스트의 불일치 (중요 검증 경계)

**본인 커밋:** [`4e8a72fdcaed`](https://github.com/nanocode00/MIPS-CPU/commit/4e8a72fdcaed)에서 [`modelsim/CP0.v`](https://github.com/nanocode00/MIPS-CPU/blob/main/modelsim/CP0.v), [`tb_CP0.v`](https://github.com/nanocode00/MIPS-CPU/blob/main/modelsim/tb_CP0.v), [`wave_CP0.tcl`](https://github.com/nanocode00/MIPS-CPU/blob/main/modelsim/wave_CP0.tcl) 추가. `750d768` 이후에도 `modelsim/` 판본이 남는다.

**상황/의도(A):** 원본 Logisim CPU의 외부 exception 처리 구조를 Verilog CP0 레지스터 집합 `EPC/Status/Block/Cause`, `mfc0`/`mtc0`, exception vector `0x800`, `eret` 복귀 경로로 옮기려 했다.

**조치(A):**
- `CP0`에 EPC/Status/Block/Cause 32비트 reg, `ExpSrc[2:0]`, `Sel=Inst[12:11]`, `BlockSrc=Block[2:0]`, `ExpClick = (|(BlockSrc&ExpSrc)) && !ExpBlock`, `HasExp = clk && counter0` 추가.
- `PCout = EPC`, `Dout`는 `Sel`별 CP0 레지스터값을 선택. `SingleCycleCPU`에서 `HasExp`이면 PC `0x800`, `IsEret && IsCOP0`면 EPC 복귀가 **의도된 신호 경로**.

**정적 코드 분석에서 발견된 미해결 불일치(A: 직접 소스 비교, 동적 시뮬레이션 수행 전):**
1. **ERET 디코딩:** `CP0.v`는 `assign IsEret = Inst[5:0] == 6'b000110` (값 6)이나 `tb_CP0.v`의 ERET 명령은 `32'h42000018`이고 그 하위 6비트는 `6'b011000` (값 24). **보존된 ERET 테스트 자극은 현재 RTL에서 `IsEret`로 디코딩되지 않는다.** 같은 조건문이 5월 28일 생성 후 삭제된 `verilog/CP0.v` 대체 판본에도 남아 있다.
2. **EPC exception 자동저장 조건:** `HasExp = clk && counter0`인 반면, EPC 갱신 절은 `if (!clk) begin if (HasExp) EPC <= PCin;` 형태다. 일반 0/1 정상 논리에서 `!clk`와 `HasExp`는 동시에 참일 수 없어 **예외로 인한 EPC 자동저장 경로에 논리 모순**이 있다. CP0가 `mtc0`에 의해 EPC를 수정하는 별도 경로까지 부정하는 것은 아님.
3. **단위 TB의 예외 자극 불충분:** `tb_CP0`는 `ExpSrc=001`을 주지만, `Block`은 0으로 초기화되고 TB에서 `mtc0` 명령으로 Block을 활성화하지 않는다. 따라서 `BlockSrc & ExpSrc == 000` 상태에서 `ExpClick`가 발생하지 않는다. 이 TB만으로 실제 EPC exception store, ERET 정상 복귀를 통과했다고 말할 수 없다.
4. **최상위 TB의 예외 없음:** `tb_SingleCycleCPU.v`에서는 `ExpSrc=000`으로 고정. **top-level exception→service→ERET 종단 시험 데이터 없음**.
5. **검증 형식:** TB들은 `$display` 중심이고 출력값 assertion·golden comparison 없음. `wave_CP0.tcl`의 `run 100ns`는 파형 구성·실행 지시를 의미할 뿐 기대값 통과 판정을 의미하지 않는다.

**사용자 당시 기억(C, 2026-10-27):** 마지막에 **예외 경로 진입 시 복귀 지점(EPC)이 저장되지 않는 문제를 관찰**했고, **남은 시간이 부족해 해결하지 못한 상태로 작업을 마쳤다**. 이는 코드에서 확인되는 EPC 자동 저장 조건 모순(A)과 **부합**하지만, 당시 테스트의 구체적인 입력·파형이나 원인의 유일성은 확정하지 않는다.

**상황 → 문제 → 판단 → 조치 → 결과:** CP0에 외부 예외 진입/복귀 구조를 구현하려 했다(상황). 최종 시험에서 EPC 복귀 주소가 저장되지 않았다(C)(문제). 어떤 해결안을 검토했는지 자료 부족으로 판단은 미확정. CP0·Top PC·TB/파형 코드를 작성했다(A)(조치). **문제를 고치지 못하고 시간 부족으로 미완성 종료했다(C)**(결과). 당시 보고서의 포괄적 예외 검증 기술(B)보다 명시적인 실패 회고(C)와 남은 코드(A)의 제약을 우선한다.

**원본 Logisim 직접 역대조(A, 2026-10-27):**
- 원본 [`src/common/cp0.circ`](https://github.com/yuxincs/MIPS-CPU/blob/master/src/common/cp0.circ)의 EPC 저장소는 32비트 Logisim `Register`이며 XML에서 `trigger="low"`(LOW 레벨 감응), `label="EPC"`로 확인. EPC 주변에는 `HasExp` 터널, 32비트 Multiplexer, 쓰기 허용 게이트, `clk` 연결이 존재. Logisim **레벨 감응 소자·데이터 선택/enable 회로**를 Verilog의 `always @(*)`와 `if(!clk)`로 1:1 옮겨도 원래 동작·전파 타이밍이 그대로 보존된다고 말할 수 없다.
- 그러나 **원본 Logisim이 EPC 저장을 정상 수행했는지 자체는 별도 동적 결과로 확인하지 못함**. 원본 회로의 `HasExp` 생성과 LOW 레벨 감응 경로 또한 구체적 timing 검증이 필요하므로 'Logisim에서 정상이었던 회로를 Verilog만 잘못 이식했다'고 확정하지 않는다.
- Verilog에서 `~clk & (clk & counter0) == 0`인 **자동 저장 경로 논리 모순(A)**은 분명하다. 이 코드 제약은 당시 **EPC 저장 실패(C)**와 부합하지만, 실제 당시 파형 및 유일 원인(D)은 미확정.

**사후 개선 아이디어(D; 프로젝트 당시 구현한 코드 아님):**
1. `exception_taken`을 `clk` 신호 레벨 자체와 분리된 동기화 대상 조건으로 생성, 클럭 상승 에지에 `EPC <= exception_pc;`와 `PC <= 32'h00000800;`을 비차단 할당하여 같은 에지에서 원래 PC를 보존. 복귀 PC의 정의는 원래 ISA의 재실행/다음 명령 계약을 별도로 확인.
2. 실제 ERET 인코딩 `32'h42000018` 판별로 수정; 기존 `6'b000110`과의 불일치 제거.
3. `Block[2:0]` 예외 허용 설정 및 `ExpSrc` 자극을 갖는 모듈·Top 통합 TB 작성. EPC/PC/exception 상태를 기대값 assertion으로 판정하고, 예외 진입·루틴·복귀를 동시에 검사.
4. 이벤트 생성 신호를 클럭처럼 반전·게이트하는 `clk_PC`/비동기 `PC_Buffer` 저장 동작을 함께 재구성할 필요가 있다. **당시 검증된 해결안이 아니라 정적 소스에 대한 사후 대안**.

**남은 확인:** 예외 진입 과정에서 EPC 저장 실패를 관찰했다는 회고(C)는 확보. 보존된 `tb_CP0`가 해당 상황을 재현하지 않으므로 당시 추가 시험 입력·파형·다른 revision은 아직 미확인. **성공적으로 ERET 복귀했다고 재해석하지 않는다.**

## 8. CPU-06 — Syscall, Statistics, Memory 및 예외 프로그램

**개인 author:** [`4e8a72fdcaed`](https://github.com/nanocode00/MIPS-CPU/commit/4e8a72fdcaed)에서 해당 RTL·테스트벤치가 추가됨. 참고 Logisim 회로·기존 assembly 원본 전체를 본인 창작으로 귀속하지 않음.

- **InstructionMemory(A):** `rom0[0:511]` / `rom1[0:511]` 각 512×32 bit를 `$readmemh("benchmark.hex",rom0)` / `$readmemh("exception_service.hex",rom1)`로 읽고 `sel`로 선택. Top은 `PC[10:2]`의 9비트 index, `PC[11]`의 ROM 선택을 사용하므로 `0x800`부터 예외 ROM으로 들어가도록 **주소 구조가 작성됨**. `tb_InstructionMemory`는 각 ROM의 index 0/4를 `$display`한다.
- **DataMemory(A):** `ram[0:1023]` 1024×32 bit 초기 0, `posedge clk`에 `Store`이면 쓰기, `Load`면 조합 읽기 아니면 0 반환. `tb_DataMemory`는 주소 10·20 쓰기/읽기, `Load=0` 시험.
- **ImmediateExtender(A):** 16비트 `ImmIn`을 `ZeroExtend`면 앞에 16비트 0, 아니면 sign bit 복제로 32비트 확장. 테스트벤치는 0x1234/FFFF/8000 등의 조건을 출력.
- **SyscallDecoder(A):** syscall enable에서 `v0 == 32'd10`이면 `Halt`, `Hex`는 clock edge에 `a0`를 갱신하는 코드. Top의 syscall 명령인 경우 RegisterFile 읽기 번호를 `$v0`(2), `$a0`(4)로 전환. **Halt는 Top 출력 포트가 아니라 `clk = Halt ? 1'b1 : Clock`에 사용되는 내부 wire**임. 당시 보고서가 Halt를 Top output처럼 쓴 부분과 실제 코드 구분. `tb_SyscallDecoder`는 v0=1/10 및 disable 케이스를 `$display`.
- **Statistics(A):** `Statistics.v`는 op를 R/I/J로 분류하는 `i,r,j` 1비트 출력을 생성, Top에서 `posedge clk`마다 J/R/I 및 TotalCycles 누적. `tb_Statistics`는 각각 명령 종류와 unknown을 출력. **모든 가능한 opcode를 정량적으로 검증한 값은 없음**.
- **자료 출처:** `modelsim/benchmark.hex` 및 `exception_service.hex`는 개인 커밋으로 **시뮬레이션 메모리 이미지 파일이 추가**됐지만, 기존 `programs/benchmark.asm`·`programs/exception_service.asm` 등 원본 설계의 프로그램들을 참고하는 맥락. **개인이 새로운 프로그램 자체를 최초 작성했다고 주장하지 않는다.**

**결과:** 위 각 모듈의 인터페이스·기능 로직과 시뮬레이션 자극은 확인(A). 각 명령어가 통합된 CPU에서 end-to-end 올바른 출력을 얻었다는 일반적 보고서 주장(B)은 TB와 실행 결과를 구별하며 한정. 특히 CP0의 정적 조건 불일치는 별도 CPU-05에 명시.

## 9. CPU-07/08 — ModelSim 검증 범위 및 마무리 정리

**검증 자산(A):** [`4e8a72f`](https://github.com/nanocode00/MIPS-CPU/commit/4e8a72fdcaed)에 `modelsim/tb_ALU.v`, `tb_Control.v`, `tb_RegisterFile.v`, `tb_ImmediateExtender.v`, `tb_InstructionMemory.v`, `tb_DataMemory.v`, `tb_CP0.v`, `tb_SyscallDecoder.v`, `tb_Statistics.v`, `tb_SingleCycleCPU.v` **10개 TB**와 `wave_*.tcl` **11개**(Top syscall 특화 wave 포함)를 추가했다. `modelsim/SingleCycleCPU.mpf`의 프로젝트 설정/환경 파일과 `modelsim/work/*`, `vsim.wlf` 등 컴파일·파형 아티팩트도 Git에 올라온다.

- 당시 [제출 보고서](https://drive.google.com/file/d/1sXOy8DC9khJu00A1rDyuSzUtrK_np_2S/view)는 Control의 ALUop/RegDst/설정 누락과 파형 디버깅, 각 모듈 및 top 시뮬레이션을 기술한다. **보고서(B)와 특정 Git diff(A)는 일치하는 큰 방향**이지만 각 오류의 재현 단계·확인 값까지 완전 복원된 것은 아님.
- 확인한 TB들은 주로 **입력 지정→`#10` 지연→`$display`로 실제값 출력**하며, 상태별 `assert`나 pass/fail 비교가 보이지 않는다. wave 파일의 `vsim`, `add wave`, `run 100ns`도 실행 절차이지 pass 판정이 아님. '검증 코드를 작성하고 시뮬레이션으로 관찰'은 근거가 있으나 '자동회귀 100% 통과'는 없음.
- [`750d76842c21`](https://github.com/nanocode00/MIPS-CPU/commit/750d76842c21)은 `verilog/` 중복 디렉터리(약 25개 RTL/HEX/환경·TB 파일)를 제거했으며 `modelsim/`은 그대로 남았다. 심지어 초판 `verilog/SingleCycleCPU.v`에는 출력 선언·`isJAL` 표기·세미콜론 등 문제 가능한 초기 코드가 있으므로 **이 삭제는 완성 RTL 전체 제거가 아니라 중복·초판 자료 정리**라는 맥락. 삭제의 직접 의도/정리 기준은 commit 제목만으로 완전히 확인 불가.
- `modelSim` 프로젝트 파일과 실행 바이너리·임시 시뮬레이션 파일을 **직접 개발한 CPU 모듈 코드량/성능 결과로 합산하지 않음**.

**다음 평가:** 전체 5커밋의 `modelsim/` 소스·환경·생성물·삭제 경계를 조사했다. 원장에 미매핑 개인 SHA 없음. 대표 후보는 **CPU-01/02 Decoder↔ALU 제어 비트 정합성·산술 수정**과 **CPU-03/04 datapath 통합**. **CPU-05 CP0는 최종 EPC 저장 실패를 해결 못한 실험·미완성 경험(C)**으로 분리하며 성공 사례로 기재하지 않는다.

## 10. Canonical v1 이후 잔여 검증

1. **CP0 사용자 회고 확보(C):** 예외 경로 진입 당시 EPC 복귀 지점 저장 실패, 시간 부족으로 미완성 종료 확인. 보존 RTL의 ERET 비교값·EPC 저장 조건(A)과 연결하되 **단일 원인을 단정하지 않음**. 추가 파형·테스트 입력이 있다면 실패 조건을 좁히는 용도로 비교.
2. **판단 근거:** Decoder 개별 bit 판별→동등 비교, ripple-style `fadd/fsub`→32비트 arithmetic 및 overflow 계산 교체의 이유를 사용자 기억/보고서로 구분. 실제 관찰한 현상이 없으면 추정으로 표시.
3. **Canonical 갱신 완료:** [`verilog-cpu.md`](verilog-cpu.md) v1에서 Decoder/ALU·datapath 통합과 CP0 미해결 로직을 분리해 기록. 향후 추가 테스트 근거 확보 시 수정. `resume.md` 미수정.
4. **PR:** Draft에 원장 + canonical 2문서 변경. **예외 진입 이후 EPC 저장 실패 및 미완성 종료(C)**를 포함. 정상 exception/ERET E2E라고 쓰지 않음. 최종 검토·병합은 별도 판단.
