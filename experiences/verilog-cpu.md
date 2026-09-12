# Verilog Single-cycle CPU

## 한 줄 요약

기존 Logisim 기반 MIPS-CPU 구조를 분석해 32-bit Single-cycle CPU를 Verilog RTL로 재구현하고, Decoder, ALU, Register File, Memory, CP0, Syscall, Statistics를 top-level datapath로 통합했습니다. ModelSim에서 모듈별 testbench와 waveform을 작성해 기능을 검증하고 control signal과 datapath를 단계적으로 추적하며 통합 오류를 수정한 프로젝트입니다.

## 기간 및 자료

- GitHub에서 직접 구현 커밋이 확인되는 기간: **2025년 5월**
- fork 기준점 이후 `nanocode00`의 직접 변경: 5 commits
- 당시 제출 보고서: `MIPS-CPU RTL 설계 및 검증_2019111450 김재훈.pdf`
- 당시 보관 ModelSim 프로젝트: `modelsim.zip`

당시 보고서에서는 프로젝트를 "MIPS 명령어 집합에 기반한 단일 사이클 CPU를 RTL 수준에서 직접 설계"한 것으로 설명했습니다. 다만 Git provenance를 다시 확인하면 저장소는 `yuxincs/MIPS-CPU`의 Logisim CPU를 기반으로 하므로, 이력서와 자소서에서는 더 엄격하게 **기존 Logisim 설계를 분석해 Verilog RTL로 재구현한 경험**으로 표현합니다.

## 소스 기준과 기여 범위

원본 저장소의 Logisim 회로, benchmark assembly, exception service assembly를 동작 기준으로 활용했고, 직접 구현한 핵심 산출물은 `modelsim/` 아래 Verilog RTL과 ModelSim 검증 환경입니다.

fork 기준점 이후 다음 모듈과 검증 파일이 추가되었습니다.

- Control 및 Opcode / Funct / ALU / Register Read Decoder
- ALU, Adder/Subtractor, Shifter, Multiplier, Divider, signed/unsigned Comparator
- 32 x 32-bit Register File
- Immediate Extender
- Instruction Memory / Data Memory
- Syscall Decoder
- CP0
- Statistics
- SingleCycleCPU top module
- 주요 모듈별 testbench와 `wave_*.tcl`

## 전체 CPU 구조

`SingleCycleCPU`는 명령어 인출부터 실행, memory access, write-back까지 하나의 instruction cycle 안에서 처리하도록 구성했습니다.

주요 datapath는 다음과 같습니다.

`PC → Instruction Memory → Control/Decoder → Register File → ALU → Data Memory → Write-back`

여기에 다음 흐름을 추가로 연결했습니다.

- branch / jump / JAL / JR
- syscall 출력 및 halt
- CP0 외부 exception 처리와 ERET 복귀
- R/I/J instruction statistics와 TotalCycles 집계

## 1. Decoder와 Control

`Opcode_Decoder`, `Funct_Decoder`, `ALU_Decoder`, `RegisterRead_Decoder`, `Control`을 구현해 opcode와 funct를 datapath 제어 신호로 변환했습니다.

주요 출력은 다음과 같습니다.

- `RegWrite`, `RegDst`, `MemRead`, `MemWrite`, `MemtoReg`
- `Branch`, `BneOrBeq`, `Jump`, `IsJAL`, `IsJR`
- `ALUSrc`, `ALUop`, `IsShamt`
- `IsSyscall`, `IsCOP0`, `ZeroExtend`

당시 제출 보고서에서도 **ALUop decoding, RegDst 위치 오류, Control unit 설정 누락**을 waveform 분석으로 찾아 수정한 사례를 명시하고 있습니다.

Git diff에서도 다음 수정이 확인됩니다.

- opcode 판별을 복잡한 개별 bit 조건에서 `op == 6'b...` 비교 방식으로 정리
- ALU control vector 방향을 `[3:0]`과 `[0:3]` 사이에서 datapath와 일치하도록 수정
- ORI/ANDI 관련 control encoding 수정
- comparator 결과와 carry/overflow 경로 수정

즉 개별 모듈이 맞더라도 decoder의 bit ordering이나 control encoding이 datapath와 다르면 CPU 전체 동작이 틀어지는 문제를 실제로 디버깅했습니다.

## 2. ALU

32-bit ALU에는 다음 연산 경로를 구성했습니다.

- SLL / SRL / SRA
- ADD / SUB
- AND / OR / XOR / NOR
- signed / unsigned comparison
- multiply / divide
- Carry / Overflow / Equal 등의 상태 신호

통합 과정에서는 ALU 입력 X/Y와 `ALUop`을 waveform으로 함께 확인해 decoder 출력과 실제 연산 경로가 일치하는지 추적했습니다.

## 3. Register File과 Memory

### Register File

- 32개의 32-bit register
- 두 개 read port, 한 개 write port
- `$zero`는 항상 0 유지
- JAL write-back을 위한 `$ra` 지원
- syscall 시 `$v0`, `$a0`를 선택해 읽는 경로 구성

### Instruction Memory

두 개의 512-word ROM을 사용합니다.

- `benchmark.hex`: 일반 프로그램
- `exception_service.hex`: exception service

`PC[11]`로 ROM을 선택하므로 `0x00000800` 영역을 exception service 시작 영역으로 사용할 수 있습니다.

### Data Memory

- 1024 x 32-bit RAM
- synchronous store
- combinational load

## 4. PC 제어

현재 top-level PC 경로는 다음 경우를 처리합니다.

- 일반 실행: `PC + 4`
- BEQ / BNE: sign-extended immediate 기반 branch target
- J / JAL: instruction target field 기반 jump
- JR: register 값으로 이동
- 외부 exception: `0x00000800`으로 이동
- ERET: CP0의 EPC 값으로 복귀

이 구조를 통해 일반 instruction flow와 exception service 영역을 하나의 CPU datapath에서 전환하도록 구현했습니다.

## 5. CP0와 외부 exception

CP0에는 다음 register가 있습니다.

- EPC
- Status
- Block
- Cause

외부 exception source는 `ExpSrc[2:0]`으로 입력됩니다. Block register와 조합해 활성 exception을 판단하고, exception 발생 시 EPC에 복귀 주소를 보존하며 top-level PC가 `0x00000800`으로 이동합니다. `eret`에서는 `PCout`, 즉 EPC를 다시 PC에 반영합니다.

`tb_CP0.v`에서는 다음 흐름을 개별적으로 확인합니다.

1. `ExpSrc = 3'b001`을 주어 exception 발생
2. EPC 저장 확인
3. `mfc0` 형태의 instruction으로 EPC read 확인
4. `eret`에서 `IsEret`과 `PCout` 확인
5. exception이 없는 상태에서 EPC 유지 확인

### CP0 test의 `syscall` 표기에 대한 주의

보관된 `tb_CP0.v`에는 첫 exception stimulus를 `syscall 시뮬레이션`이라고 적고 `Inst = 0x0000000C`를 사용합니다. 동시에 `ExpSrc = 001`도 직접 주입합니다.

하지만 실제 `CP0.v`에서 exception 발생 여부를 만드는 것은 **instruction의 syscall opcode가 아니라 `ExpSrc`**입니다. 반면 top-level의 일반 `syscall`은 별도 `SyscallDecoder`가 처리합니다.

따라서 이 프로젝트에서 다음 두 흐름은 구분해서 설명합니다.

- 일반 `syscall` → `$v0/$a0` 기반 출력 또는 halt
- 외부 `ExpSrc` exception → EPC 저장 → `0x800` service → ERET 복귀

`tb_CP0`의 syscall instruction은 exception 상황을 표현하기 위해 함께 넣은 test stimulus로 보고, **syscall 자체가 CP0 exception을 자동 발생시킨다고 설명하지 않습니다.**

## 6. Syscall Decoder

R-type `syscall`이 검출되면 Register File의 일반 rs/rt 대신 `$v0`와 `$a0`를 읽습니다.

`SyscallDecoder`는 다음과 같이 동작합니다.

- `$v0 == 10` → 내부 `Halt` 활성화
- 그 외 syscall → `$a0` 값을 `Hex` output으로 반영

`Halt`는 `SingleCycleCPU`의 외부 output port는 아니며 top module 내부에서 clock을 멈추는 신호로 사용됩니다. 당시 보고서에는 Halt를 시스템 출력으로 표현했지만 실제 최종 RTL에서는 내부 wire입니다.

별도 `wave_SingleCycleCPU_syscall.tcl`에는 `Inst`, `IsSyscall`, `$v0/$a0`, `Hex`, `Halt`, ALU X/Y, `ALUop`, write target `RW`, `WE`, `RegDst` 등을 동시에 관찰하도록 구성되어 있습니다.

## 7. Statistics

`Statistics.v`는 opcode를 R/I/J type으로 분류하고 top-level에서 다음 값을 누적합니다.

- `R`
- `I`
- `J`
- `TotalCycles`

당시 보고서에도 instruction type별 count와 전체 실행 cycle 누적을 별도 검증 항목으로 기록했습니다.

## 8. ModelSim 검증

당시 보고서와 보관된 `modelsim.zip`에서 다음 testbench를 확인할 수 있습니다.

- `tb_ALU.v`
- `tb_Control.v`
- `tb_RegisterFile.v`
- `tb_ImmediateExtender.v`
- `tb_InstructionMemory.v`
- `tb_DataMemory.v`
- `tb_CP0.v`
- `tb_SyscallDecoder.v`
- `tb_Statistics.v`
- `tb_SingleCycleCPU.v`

각 testbench에 대응하는 `wave_*.tcl`도 남아 있습니다.

보고서가 명시한 주요 검증 항목은 다음과 같습니다.

- ADD, SUB, SLL, SRA 등 ALU 결과
- exception 발생 시 `HasExp`, `PCout`
- syscall 시 `Hex`, `Halt`
- register read/write 값
- J/R/I 통계와 `TotalCycles`
- top-level instruction 실행 흐름

### 통합 exception 검증 범위

보관된 `tb_SingleCycleCPU.v`의 `ExpSrc`는 `3'b000`으로 고정되어 있어, 현재 남아 있는 top-level testbench 자체에서는 외부 exception을 실제로 주입하지 않습니다.

따라서 다음과 같이 범위를 구분하는 것이 안전합니다.

- CP0 exception 동작: `tb_CP0`에서 단위 검증
- 전체 CPU 일반 instruction/syscall 흐름: `tb_SingleCycleCPU` 및 waveform으로 통합 관찰
- exception vector와 ERET 경로: RTL 구현으로 확인 가능
- **top-level에서 exception service 진입부터 복귀까지 end-to-end simulation을 완료했다고 현재 보관 자료만으로 단정하지 않음**

자동 pass/fail regression log는 별도로 남아 있지 않으므로, 이력서에서는 testbench와 waveform 기반 기능 검증으로 표현합니다.

## 핵심 디버깅 방식

최종 출력만 보는 대신 CPU의 실행 흐름을 다음 순서로 나눠 추적했습니다.

1. PC와 현재 instruction
2. opcode/funct decoder의 control signal
3. Register File read 값
4. ALU X/Y input과 ALUop
5. ALU result와 memory access
6. write-back register와 data
7. branch/jump/ERET에 따른 다음 PC

이 과정에서 당시 보고서에 기록된 **ALUop decoding, RegDst 위치 오류, Control 설정 누락**과 Git에서 확인되는 control vector/encoding 문제를 단계적으로 수정했습니다.

## 이력서용 핵심 bullet

- 기존 Logisim 기반 MIPS CPU 구조를 분석해 **32-bit Single-cycle CPU를 Verilog RTL로 재구현**하고 Decoder, ALU, Register File, Memory, Control을 top-level datapath로 통합
- EPC, Status, Block, Cause를 갖는 **CP0와 외부 `ExpSrc` exception / `mfc0` / `mtc0` / `eret` 경로를 구현**, exception 시 `0x800` service 영역으로 전환하도록 PC 제어 구성
- ModelSim에서 ALU, Control, Register File, Memory, CP0 등 **주요 모듈별 testbench와 waveform script를 작성**하고 PC, control, ALU, write-back 신호를 단계별 추적해 통합 오류 수정
- ALUop decoding, RegDst, Control 설정과 control encoding 불일치를 추적하며 **개별 RTL 모듈부터 전체 instruction flow까지 신호 단위로 검증**

## 근거와 사용 시 주의

### 직접 경험으로 강하게 사용할 수 있는 내용

- `modelsim/`의 Verilog RTL 재구현
- 모듈별 testbench / waveform script
- SingleCycleCPU datapath 통합
- CP0 register와 exception/ERET 경로 구현
- syscall과 statistics 구현
- waveform 기반 control/datapath debugging

### 직접 기여로 표현하지 않는 내용

- 원본 Logisim CPU architecture 자체의 최초 설계
- `programs/benchmark.asm`과 `programs/exception_service.asm`의 원본 작성
- pipeline CPU 구현

### 사용하지 않는 과거 메모

- `tb_compare`: 이 MIPS 프로젝트에서는 확인되지 않음
- `EPC 초기값 0 → 4 수정`: Git과 보관된 `CP0.v` 모두 최종 EPC 초기값이 0이므로 근거 없음
- `syscall → 자동 CP0 exception`: 최종 RTL에서는 syscall과 외부 exception 경로가 분리됨
- `top-level exception end-to-end 검증 완료`: 현재 보관된 top testbench에서는 `ExpSrc=000`이므로 완료 근거 부족

## 보여주는 역량

- 컴퓨터구조와 MIPS datapath 이해
- Verilog RTL 구현
- CPU control/datapath 통합
- instruction decoding
- Register File / Memory 설계
- CP0와 exception/return 흐름 이해
- ModelSim testbench 및 waveform 기반 검증
- 여러 모듈의 control signal을 연결해 통합 오류를 추적하는 디버깅

## 자소서 활용 포인트

- 기존 회로 구조를 분석해 Verilog RTL로 재구성한 경험
- 개별 모듈 검증에서 top-level 통합으로 확장한 경험
- control signal 하나의 오류를 instruction flow 전체에서 역추적한 경험
- CPU의 PC, decoder, register, ALU, memory, exception 경로를 직접 연결하며 컴퓨터구조를 실제 구현으로 확인한 경험
