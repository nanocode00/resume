# Verilog Single-cycle CPU — 커밋 기반 개발작업 원장

> 조사 시작 2026-10-27. 원본: [nanocode00/MIPS-CPU](https://github.com/nanocode00/MIPS-CPU) (**[yuxincs/MIPS-CPU](https://github.com/yuxincs/MIPS-CPU) fork**). **상태: 전체 브랜치/개인 커밋 분리 완료, Decoder→ALU→Register File의 최초 구현·후속 diff 1차 조사.** `4e8a72f`의 전체 CPU 통합·CP0·테스트와 최종 정리 `750d768`의 나머지 파일별 분석은 진행 예정. 사실 기준 문서 [`verilog-cpu.md`](verilog-cpu.md)와 `resume.md`는 아직 수정하지 않음.

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
| [`4e8a72fdcaed`](https://github.com/nanocode00/MIPS-CPU/commit/4e8a72fdcaed) | 2025-05-28 | `SingleCycleCPU.v`, CP0, IM/DM, Syscall, Statistics, 테스트벤치·파형 스크립트 추가. Decoder·ALU·Register File 수정. `verilog/` 별도 복제·ModelSim 산출물도 추가 | **CPU-01~03 후속 수정 일부 확인**, **CPU-04 Top/PC, CPU-05 CP0/ERET, CPU-06 Syscall/Statistics, CPU-07 검증** 상세 diff 추가 조사 예정. 커밋 규모만으로 독립 검증/작성량 추정 금지 |
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

## 6. 작업 미완료 범위와 다음 조사 순서

1. **CPU-04 통합 top module:** 5/28 `SingleCycleCPU.v`의 PC→ROM→Control→RegisterFile→ALU→DM/Writeback 순서와 branch/jump/JAL/JR의 구체적 신호 연결 확인. `RegDst`와 `ALU` operand ordering 관련 실수의 소스 대조.
2. **CPU-05 CP0·exception/ERET:** `CP0.v`, `tb_CP0.v`, `SingleCycleCPU.v`, `wave_CP0.tcl` 간 신호를 구분. **top-level TB `ExpSrc=000` 고정** 주장은 코드와 재확인; exception 회귀 성공 여부는 별도.
3. **CPU-06 syscall·Statistics·memory:** `SyscallDecoder.v`, `Statistics.v`, `ImmediateExtender.v`, `DataMemory.v`, `InstructionMemory.v` 소스, benchmark/exception hex 출처 대조.
4. **CPU-07 검증 / CPU-08 정리:** ModelSim TB·TCL 실제 coverage/주장 차이, `verilog/` 임시 복제 반입 후 제거(`750d768`) 정리. 의사결정 근거는 당시 보고서로 1차 보완한 뒤 사용자에게 좁혀 물음.
5. 전체 5커밋의 파일 변경과 본인 기여/원본·생성 파일 구분을 역검증하고 **canonical `experiences/verilog-cpu.md`를 갱신**. 그 전에는 기존 경험 문서·`resume.md` 그대로 유지.
