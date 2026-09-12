# Verilog Single-cycle CPU

## 한 줄 요약

기존 Logisim 기반 MIPS-CPU 설계를 참고해 32-bit 단일 사이클 CPU를 Verilog로 재구현하고, ModelSim에서 모듈 단위와 전체 CPU의 동작을 검증한 프로젝트. CP0 기반 외부 예외 처리와 `mfc0`/`mtc0`/`eret` 복귀 흐름까지 Verilog로 연결했습니다.

## 기간

- GitHub에서 직접 구현 커밋이 확인되는 기간: 2025년 5월
- fork 기준점 이후 `nanocode00`의 직접 변경: 5 commits

## 소스 기준과 기여 범위

이 저장소는 `yuxincs/MIPS-CPU`의 Logisim 기반 MIPS CPU를 기반으로 합니다. 원본 저장소에 있던 Logisim 회로, MIPS benchmark assembly, exception service assembly를 CPU 동작의 기준으로 활용했고, 제 직접 구현은 `modelsim/` 디렉터리의 Verilog RTL과 ModelSim 검증 환경입니다.

fork 기준점과 현재 main을 비교하면 다음 Verilog 모듈과 검증 파일이 새로 추가되어 있습니다.

- ALU 및 Adder/Subtractor, Shifter, Multiplier, Divider, Comparator
- Opcode / Funct / ALU / Register Read Decoder
- Control
- Register File
- Instruction Memory / Data Memory
- Immediate Extender
- Syscall Decoder
- CP0
- Statistics
- SingleCycleCPU top module
- 각 주요 모듈용 testbench와 ModelSim waveform script

따라서 이 프로젝트는 원본 MIPS 아키텍처를 처음부터 설계했다고 표현하기보다, **Logisim으로 구현된 CPU 구조를 분석해 Verilog RTL로 옮기고 연결·디버깅·검증한 경험**으로 설명하는 것이 정확합니다.

## Git에서 확인되는 구현 흐름

### 2025년 5월 9일경: 명령어 Decoder 구현

첫 직접 커밋인 `Opcode Decoder`에서 다음 제어 모듈을 구현했습니다.

- `Opcode_Decoder`
- `Funct_Decoder`
- `ALU_Decoder`
- `RegisterRead_Decoder`
- `Control`

R-type, jump, branch, immediate, load/store, COP0 명령을 opcode/funct에 따라 해석하고 ALU 동작, register write, memory read/write, branch/jump 등의 제어 신호로 변환했습니다.

### 2025년 5월 24일경: ALU 구현

`ALU` 커밋에서 32-bit 연산 경로를 구현했습니다.

- SLL / SRA / SRL
- ADD / SUB
- AND / OR / XOR / NOR
- signed / unsigned comparison
- multiply / divide 결과 경로
- Carry / Overflow 처리

초기 구현 이후 top-level CPU를 연결하는 과정에서 ALU control vector 방향과 decoder 출력값을 다시 맞추고, adder/subtractor의 carry와 overflow 계산도 정리했습니다.

### 2025년 5월 24일경: Register File 구현

32개의 32-bit register를 가진 Register File을 구현했습니다.

- 두 개 register 동시 read
- 한 개 register write
- `$zero`는 항상 0으로 유지
- syscall 및 검증에 필요한 `$v0`, `$a0`, `$s0~$s2`, `$ra` 상태 확인용 출력 제공

### 2025년 5월 28일경: SingleCycleCPU 통합

`SingleCycleCPU` 커밋에서 개별 모듈을 하나의 CPU datapath로 통합했습니다.

주요 연결은 다음과 같습니다.

- PC 및 `PC + 4`
- Instruction Memory
- Control / Decoder
- Register File
- Immediate Extender
- ALU
- Data Memory
- Syscall Decoder
- CP0
- instruction statistics

PC 선택 경로에는 sequential execution뿐 아니라 branch, jump, JAL, JR, exception vector, ERET 복귀 경로가 포함되어 있습니다.

## 명령어 실행과 PC 제어

현재 Verilog top module에서 PC는 다음 경우를 구분해 갱신합니다.

- 일반 명령: `PC + 4`
- BEQ/BNE: sign-extended immediate를 shift해 branch target 계산
- J/JAL: instruction target field를 이용한 jump
- JR: register 값으로 PC 이동
- 외부 exception: `0x00000800`으로 이동
- ERET: CP0의 EPC 값을 PC로 복원

Instruction Memory는 두 개의 512-word ROM을 사용합니다. 일반 benchmark는 첫 번째 ROM에, exception service는 두 번째 ROM에 로드되며 `PC[11]`로 선택합니다. 이 구조로 `0x00000800`을 exception service 시작 주소로 사용했습니다.

## CP0와 예외 처리

CP0에는 다음 register를 구현했습니다.

- EPC
- Status
- Block
- Cause

외부 exception source는 `ExpSrc[2:0]`으로 입력되고, Block register와 조합하여 활성 exception을 판단합니다. exception이 발생하면 CPU는 현재 PC를 보존한 뒤 `0x00000800`의 exception service로 이동합니다.

exception service에서는 `mfc0`으로 EPC와 Cause/Block 정보를 읽고 필요한 register를 memory stack에 저장합니다. 처리 종료 시 register와 EPC를 복구하고 `mtc0`과 `eret`을 사용해 원래 실행 흐름으로 돌아갑니다.

### syscall과 CP0 exception은 구분

기존 경험 정리에는 `syscall → EPC 저장 → 0x800 → ERET` 흐름으로 적혀 있었지만 현재 Verilog 소스와 일치하지 않습니다.

이 구현에서:

- `syscall`은 `SyscallDecoder`가 `$v0`와 `$a0`를 읽어 화면 출력 또는 종료를 처리합니다.
- CP0 exception은 별도의 `ExpSrc[2:0]` 외부 exception input으로 발생합니다.
- exception 발생 시에만 EPC 저장과 `0x800` 진입, ERET 복귀 흐름이 동작합니다.

따라서 앞으로 이 프로젝트를 설명할 때 두 흐름을 분리합니다.

## Decoder와 ALU 통합 과정에서 확인되는 수정

초기 decoder를 전체 CPU에 연결하는 과정에서 제어 신호를 여러 차례 수정했습니다.

Git diff에서 직접 확인되는 예는 다음과 같습니다.

- opcode를 개별 bit 논리식으로 판별하던 방식을 `op == 6'b...` 형태로 정리
- 중복/잘못 선언된 opcode condition 정리
- ALU control vector를 `[3:0]`에서 `[0:3]` 방향으로 맞춤
- `ALUop[0]`에서 ANDI가 아닌 ORI가 사용되도록 수정
- comparator 결과를 32-bit 값으로 연결
- adder/subtractor의 carry/overflow 계산 구조 변경

이 과정은 개별 모듈이 맞더라도 decoder의 bit ordering과 datapath의 control encoding이 다르면 전체 CPU가 잘못 동작한다는 점을 확인한 디버깅 경험입니다.

## 메모리 구조

### Instruction Memory

- 32-bit instruction
- 두 개의 512-word ROM
- `benchmark.hex`와 `exception_service.hex`를 `$readmemh`로 로드
- 총 10-bit word-address 영역을 두 ROM으로 분리

### Data Memory

- 1024 x 32-bit RAM
- synchronous store
- combinational load
- 초기 RAM을 0으로 초기화

## syscall 처리

R-type `syscall` 명령이 검출되면 일반 instruction의 rs/rt 대신 `$v0`와 `$a0`를 Register File에서 읽습니다.

`SyscallDecoder`는:

- `$v0 == 10`이면 CPU halt
- 그 외 syscall에서는 `$a0` 값을 Hex output에 반영

하도록 구성되어 있습니다. benchmark assembly는 `$v0 = 34`를 사용해 중간 계산값을 출력하며 CPU 동작을 확인합니다.

## ModelSim 검증 환경

fork 기준점 이후 다음 testbench가 직접 추가되어 있습니다.

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

각 모듈의 주요 입출력을 관찰하기 위한 `wave_*.tcl` 스크립트도 함께 구성했습니다. 특히 CPU용 waveform에는 instruction, ALU input/output, register write target, RegDst, syscall 관련 `$v0/$a0` 등을 추가해 실행 흐름을 신호 단위로 추적할 수 있도록 했습니다.

현재 저장소에는 시뮬레이션 프로젝트 및 waveform artifact가 남아 있지만, 자동화된 pass/fail regression log는 별도로 보존되어 있지 않습니다. 따라서 이력서에서는 **ModelSim testbench와 waveform을 작성해 모듈 및 통합 동작을 검증했다**고 표현하고, 특정 자동 테스트 통과 횟수 등은 주장하지 않습니다.

## 핵심 디버깅 방식

CPU는 최종 출력 하나만 보고 원인을 찾기 어렵기 때문에 다음 순서로 문제를 좁혔습니다.

1. 현재 PC와 instruction 확인
2. opcode/funct decoder의 control signal 확인
3. Register File read 값 확인
4. ALU X/Y input과 ALUop 확인
5. ALU result와 memory access 확인
6. write-back register와 data 확인
7. branch/jump/exception에 따른 다음 PC 확인

이렇게 datapath를 단계별로 따라가면서 decoder와 ALU encoding, register destination, immediate extension, PC selection 문제를 분리해서 수정했습니다.

## 이력서용 핵심 bullet

- 기존 Logisim 기반 MIPS CPU 구조를 분석해 **32-bit Single-cycle CPU를 Verilog RTL로 재구현**하고 ALU, Decoder, Register File, Memory, Control을 top-level datapath로 통합
- `mfc0`/`mtc0`/`eret`을 처리하는 **CP0와 EPC·Status·Block·Cause register를 구현**하고, 외부 exception 발생 시 `0x800` service routine 진입 후 EPC로 복귀하는 흐름 구성
- ModelSim에서 ALU, Control, Register File, Memory, CP0 등 **주요 모듈별 testbench와 waveform script를 작성**하고 PC·control·ALU·write-back 신호를 단계별 추적해 통합 오류 디버깅
- decoder bit ordering과 ALU control encoding 불일치를 수정하며 **개별 RTL 모듈부터 전체 instruction execution까지 신호 단위로 검증**

## 근거와 사용 시 주의

Git으로 강하게 확인되는 것은 `modelsim/`의 Verilog 재구현과 testbench입니다.

반면 다음 항목은 직접 기여로 표현하지 않습니다.

- 원본 Logisim CPU 아키텍처 자체의 최초 설계
- `programs/benchmark.asm`과 `programs/exception_service.asm`의 원본 작성
- pipeline CPU 구현

또 과거 메모에 있던 `tb_compare`는 이 MIPS 저장소에서 확인되지 않았습니다. 해당 이름의 비교 테스트는 이 프로젝트 근거로 사용하지 않습니다.

`EPC 초기값 0 → 4 수정` 역시 현재 Git 이력에서 확인되지 않고 최종 `CP0.v`의 EPC 초기값은 0이므로, 추가 원본 자료가 발견되기 전까지 이 프로젝트의 사실로 사용하지 않습니다.

## 보여주는 역량

- 컴퓨터구조와 MIPS datapath 이해
- Verilog RTL 구현
- CPU control/datapath 통합
- instruction decoding
- Register File 및 Memory 설계
- CP0와 exception/return 흐름 이해
- ModelSim testbench 및 waveform 기반 검증
- 복잡한 시스템을 신호 단위로 나눠 원인을 추적하는 디버깅

## 자소서 활용 포인트

- 기존 시스템을 분석해 다른 표현 방식으로 재구현한 경험
- 저수준 HW/SW 동작 원리를 이해한 경험
- 여러 모듈이 연결된 시스템 통합 경험
- 제어 신호 하나의 오류를 실행 흐름 전체에서 역추적한 경험
- 컴퓨터구조와 RTL 설계 역량을 실제 구현으로 연결한 경험
