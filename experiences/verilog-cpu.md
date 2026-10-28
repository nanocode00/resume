# Verilog Single-cycle CPU — Canonical 경험 기록

> **상태 (2026-10-27):** 기존 Logisim 기반 MIPS CPU를 참고한 Verilog RTL 재구현 경험의 **사실 기준 문서 v1**. Fork 이후 본인 author **5커밋 전체**, 당시 RTL/ModelSim TB·파형 스크립트·[제출 보고서](https://drive.google.com/file/d/1sXOy8DC9khJu00A1rDyuSzUtrK_np_2S/view)를 교차 검토했다. 특히 **최종 예외 진입 때 EPC 복귀 주소가 저장되지 않았고 시간 부족으로 수정하지 못해 종료했다는 사용자 회고(C, 2026-10-27)**를 반영했다. 보존 RTL의 EPC 저장 조건·ERET 디코딩 불일치(A)도 확인되어 정상 복귀를 '검증 완료'로 단정하지 않는다. SHA별 상세 변경은 [작업 원장](verilog-cpu-commit-ledger.md) 참조. 이 문서의 모든 내용을 즉시 이력서로 전재하지 않는다.

## 1. 프로젝트 개요와 소스 기준

- **기간:** 개인 Git commit에서 2025-05-09~2025-05-28 확인.
- **원본:** [yuxincs/MIPS-CPU](https://github.com/yuxincs/MIPS-CPU) — Logisim으로 작성된 시뮬레이션 MIPS CPU 회로 및 테스트 프로그램. **2025-05-02에 [nanocode00/MIPS-CPU](https://github.com/nanocode00/MIPS-CPU)로 Fork**.
- **개인 작업:** 원본 Logisim CPU의 구조를 참조해 `modelsim/`에 Verilog RTL로 재구현하고 Decoder·ALU·Register File·Memory·SingleCycleCPU·CP0·Syscall·Statistics를 연결. ModelSim TB 10개와 wave TCL 11개 등 시험 자산 추가.
- **기여 경계:** 원본 Logisim 아키텍처/회로 자체, 원본 benchmark·exception service 어셈블리, `logisim-win-2.7.1.exe`를 개인 신규 구현으로 주장하지 않음. 5월 28일의 대형 커밋에는 `verilog/` 임시 중복 소스 및 ModelSim 작업 파일도 함께 올라왔고 같은 날 `verilog/`는 삭제됨. 따라서 commit added lines를 직접 설계한 RTL 라인수/완성도로 보지 않는다.
- **당시 보고서:** [MIPS-CPU RTL 설계 및 검증(PDF)](https://drive.google.com/file/d/1sXOy8DC9khJu00A1rDyuSzUtrK_np_2S/view), [DOCX](https://drive.google.com/file/d/1Qfh27NjfegX52KMPx5p6CIF12r_L1ssc/view). ModelSim waveform으로 Control·ALU·CP0·Top 등을 관찰하고 ALUop/RegDst/Control 문제를 수정했다고 기록. **당시 보고서의 광범위한 '정상 검증' 표현과 실제 보존된 TB 커버리지는 구분**한다.

## 2. 기능 구조와 본인 작성 범위

최종 `modelsim/SingleCycleCPU.v`는 다음 구조를 연결한다.

`PC → InstructionMemory(ROM) → Control/Decoder → RegisterFile → ALU → DataMemory → Write-back`

PC에는 `PC+4`, BEQ/BNE 분기, J/JAL 점프, JR 레지스터 점프, 외부 예외 벡터 `0x00000800`, CP0 EPC 복귀를 **선택하려는 코드 경로**가 존재한다.

| 단위 | 직접 구현·수정 근거 | 주의 |
|---|---|---|
| Decoder/Control | `Opcode_Decoder`, `Funct_Decoder`, `RegisterRead_Decoder`, `ALU_Decoder`, `Control`의 5/9 초안, 5/28 비교식·control vector 수정 | ISA/Logisim 회로 원저작과 구분 |
| 32bit ALU | Shift, add/sub, signed/unsigned compare, 논리연산 및 multiply/divide 하위 모듈. 초판→통합판 수정 | 초기 `output reg`, port 연결 오류 등을 수정했음; 최초 커밋 자체를 완성·통과 버전으로 간주하지 않음 |
| Register File | 32×32비트 레지스터, 두 조합 읽기·양의 에지 쓰기, 0번 register 쓰기 차단, 최종 초기화 | 5/28에 관찰용 `s0/s1/s2/ra` 출력 제거 후 `a0/v0`만 유지 |
| Memory/Immediate | sign/zero extension, `benchmark.hex`·`exception_service.hex` 이중 ROM, 1024×32 Data Memory | 프로그램의 **원본 어셈블리 최초 작성은 아님** |
| Syscall/Statistics | `v0==10` Halt, 그 외 `a0` 데이터 표시 경로, J/R/I·cycle count | Halt는 **Top 내부 wire** (외부 Top port 아님) |
| CP0/exception | EPC/Status/Block/Cause 저장소·`ExpSrc` 입력·vector/return 선택 코드 | **예외 진입 과정에서 EPC 저장 실패를 겪고, 시간 부족으로 미완성 종료(C)**. 현재 RTL·TB 불일치도 확인(A) |

## 3. GROUP-01 — Decoder/Control과 ALU 연산 부호 통합

**근거:** [`2f059fde7be0` (5/9)](https://github.com/nanocode00/MIPS-CPU/commit/2f059fde7be0) → [`74a5eee30fa5` (5/24)](https://github.com/nanocode00/MIPS-CPU/commit/74a5eee30fa5) → [`4e8a72fdcaed` (5/28)](https://github.com/nanocode00/MIPS-CPU/commit/4e8a72fdcaed).

- **상황:** Logisim 회로의 제어 신호와 산술 연산 경로를 Verilog로 재구성하던 단계.
- **문제(A):** 첫 Decoder는 opcode/funct를 `!op[0]` 등 개별 bit의 boolean으로 판별하고 `ALUop`을 `[3:0]`으로 선언했다. ALU 초판에는 절차 대입을 사용하는 출력이 `reg`로 선언되지 않았고 일부 하위 모듈 입력이 명시적으로 연결되지 않았다. **최종 설계와 다른 초기 불일치가 Git diff에 남아 있다**.
- **판단/조치(A):** 코드 통합 단계에서 Decoder 판별을 명시적인 `op == 6'b...` 및 `Funct == 6'b...` 비교로 변경, `ALUop` 벡터 방향을 `[0:3]`으로 통일했다. `Control`의 신호 선택식을 간결화하고 ORI/ANDI 등 opcode의 ALU 연산 선택을 수정. ALU 출력 선언을 `output reg`로 수정하고 하위 연산기 input 연결을 명시했다.
- **결과(A/B):** 최종 Control/ALU 인터페이스의 디코딩 형식과 연결이 코드상 일치하도록 정리됨(A). 당시 보고서는 ALUop/Control 디코딩과 레지스터 목적지 설정 문제를 waveform으로 수정한 경험을 기록(B). **정확히 어느 명령에서 어떤 출력 오류가 발생했는지, 동등 비교 도입이 직접적인 오류 복구였는지**는 당시 wave/log 부족으로 미확정.
- **검증 주의:** `tb_Control`·`tb_ALU`는 여러 입력을 공급하고 `$display`로 실제값을 출력하므로 **검증 환경/관찰 코드를 작성했다**는 사실은 분명하나, 전 명령 자동 regression pass가 확인된 것은 아니다.

## 4. GROUP-02 — ALU 가감산·Register File·Top datapath 통합

**근거:** [`74a5eee`](https://github.com/nanocode00/MIPS-CPU/commit/74a5eee30fa5), [`538b433`](https://github.com/nanocode00/MIPS-CPU/commit/538b4339f383), [`4e8a72f`](https://github.com/nanocode00/MIPS-CPU/commit/4e8a72fdcaed).

- **상황:** 개별 산술 블록과 저장소가 구현된 뒤 fetch/decode/execute/memory/writeback을 단일 사이클로 연결하는 단계.
- **문제(A):** 초기 `Adder`/`Subtractor`는 `fadd/fsub` ripple-style 배열과 carry/borrow chain을 사용했다. 통합에서는 arithmetic과 OF/CF 신호를 일관되게 산출하도록 수정됐다. 초기 RegisterFile의 관측 출력은 실제 Top에 연결하는 인터페이스로 정리할 필요가 있었고, 초기화가 없었다.
- **조치(A):** `Adder/Subtractor`를 33비트 확장 산술 표현으로 변경하고 sign bit 비교로 overflow 판별. `RegisterFile`에는 32개 0 초기화, $zero 쓰기 차단, $a0/$v0 관측 출력 유지. Top에서 `RW = IsJAL ? 31 : (RegDst ? rd : rt)`, `Din`의 ALU/Memory/JAL/CP0 선택, `WE`, shift/즉시값용 ALU X/Y 경로, `PC+4`·branch/jump/JR를 연결.
- **결과(A):** [`SingleCycleCPU.v`](https://github.com/nanocode00/MIPS-CPU/blob/main/modelsim/SingleCycleCPU.v)에 통합 datapath가 존재한다. `tb_RegisterFile`, `tb_ImmediateExtender`, `tb_InstructionMemory`, `tb_DataMemory`, `tb_SingleCycleCPU`는 해당 단위를 관찰하기 위한 stimulus/출력 코드를 보유하고 있다. **각 상황의 자동 판정 결과나 모든 instruction 정상 수행 증거는 별도로 없음**.
- **선택 이유 미확정:** ripple-style adder/subtractor를 내장 `+/-` 표현으로 바꾼 원인이 특정 오류/시뮬레이터 호환성/간결화 중 무엇인지 보고서나 Git 메시지만으로는 판단 불가.

## 5. GROUP-03 — CP0 exception/ERET 구현 시도와 실제 검증 한계

**근거:** [`CP0.v`](https://github.com/nanocode00/MIPS-CPU/blob/main/modelsim/CP0.v), [`tb_CP0.v`](https://github.com/nanocode00/MIPS-CPU/blob/main/modelsim/tb_CP0.v), [`tb_SingleCycleCPU.v`](https://github.com/nanocode00/MIPS-CPU/blob/main/modelsim/tb_SingleCycleCPU.v), [작업 원장 CPU-05](verilog-cpu-commit-ledger.md#7-cpu-05--cp0eret-설계와-보존된-rtl테스트의-불일치-중요-검증-경계).

**당시 의도/작업(A):** `ExpSrc[2:0]`와 Block register를 바탕으로 예외 요청을 검출하고 EPC에 PC를 저장, PC를 `0x800` service로 이동, `mfc0/mtc0`와 `eret`을 통해 상태를 조회하고 복귀시키는 경로를 구현하려 했다.

**당시 관찰 결과(C, 2026-10-27):** 사용자는 최종 단계에서 **예외 경로 진입 시 복귀 지점이 EPC에 저장되지 않았으며, 더 수정할 시간이 없어 그 상태로 프로젝트를 끝냈다**고 회고했다. 예외 벡터 도달의 파형·스크립트는 현재 보존된 TB에서 독립 확인되지 않으므로 **사용자 관찰(C)**과 **코드/테스트 근거(A)**는 구분한다.

**보존된 코드에 실제로 있는 네 가지 검증 장애:**

1. **ERET 값 불일치(A):** `CP0.v`는 `Inst[5:0] == 6'b000110`(6)으로 `IsEret`을 판별하지만, 같은 저장소의 `tb_CP0.v`가 넣는 `eret=32'h42000018`의 하위 6비트는 `011000`(24). **이 테스트 명령은 보존된 코드에서 ERET로 검출되지 않는다.**
2. **EPC 저장 조건 충돌(A):** `HasExp = clk && counter0`이고 EPC 저장 절은 `if (!clk) if (HasExp) EPC <= PCin`. **일반적인 0/1 논리에서 두 조건은 동시에 참일 수 없다**. 예외로 EPC를 자동 저장하는 로직이 작동한다고 확정할 수 없으며, 별도 `mtc0` EPC 경로와는 구별한다.
3. **CP0 단위 TB의 활성화 부족(A):** `tb_CP0`는 `ExpSrc=001`을 자극하지만 `Block`은 0으로 초기화되고 TB에서 Block register를 변경하지 않는다. 이에 따라 `BlockSrc & ExpSrc`가 0이므로 **예외 트리거 `ExpClick`는 해당 자극만으로 올라가지 않는다**.
4. **Top 예외 시나리오 없음(A):** `tb_SingleCycleCPU`의 `ExpSrc=000` 고정. 정상적인 예외 서비스 진입부터 EPC 복귀까지의 실제 end-to-end stimulus가 이 TB에 없다.

**상황 → 문제 → 판단 → 조치 → 결과:** 예외 발생 후 복귀할 수 있는 CP0 경로가 필요했다(상황). 최종 시험에서 예외 경로 진입 뒤 **EPC 복귀 주소가 저장되지 않았다(C)**(문제). 당시 대안·우선순위 판단은 미확정. EPC/Status/Block/Cause와 벡터·ERET 제어, 테스트 및 파형 스크립트를 작성했다(A)(조치). **복귀 주소 저장 실패를 해결하지 못했고 남은 시간이 부족해 미완성 종료했다(C)**(결과). 보존된 RTL의 EPC 조건 충돌 및 ERET 인코딩 오류(A)는 당시 현상과 부합하지만, 실제 오류 원인이 오직 이 코드였다고 단정하지 않는다. 보고서의 포괄적 '정상 검증'보다 구체적 실패 경험을 우선해 기록한다.

### 2026-10-27 원본 Logisim 회로 역대조와 사후 개선안

- **A — 원본 구조:** [`yuxincs/MIPS-CPU/src/common/cp0.circ`](https://github.com/yuxincs/MIPS-CPU/blob/master/src/common/cp0.circ)의 EPC는 **32비트 `Register`에 `trigger="low"`**가 설정되어 있고, 그 앞에 데이터 선택용 Multiplexer, `HasExp` 터널, enable 관련 논리 및 clk 연결이 있다. Logisim의 LOW 레벨 감응 Register와 Verilog `reg`에 대한 `always @(*)` 조건 대입은 **동등한 RTL 구현이라고 자동으로 볼 수 없다**.
- **A — Verilog와 관찰된 실패의 연결:** 현재 Verilog는 `HasExp = clk && counter0`, 저장 조건은 `if (!clk) if (HasExp)`여서 이진 논리상 `~clk & HasExp == 0`. 이 조건은 EPC 자동 저장 실패(C)와 부합한다. 다만 **원본 Logisim 회로가 동일한 시험에서 정상 동작했는지**는 시뮬레이션 결과가 없어 확인할 수 없다. 저장 소자와 게이트 전파의 차이가 원인일 수 있다는 것은 **사후 분석(D)**이지 당시 확정 진단이 아니다.
- **사후 개선안(D, 당시 구현 아님):** 예외 수락 신호(`exception_taken`)를 `clk` 레벨과 분리해 만들고, **같은 `posedge clk`에 `EPC <= exception_pc;`, `PC <= 32'h800;`을 비차단 대입**하도록 PC와 EPC 업데이트 순서를 구성한다. `PC_Buffer`/반전 `clk_PC`도 새 동기화 구조와 함께 재검토한다. 저장 주소가 현재 PC인지 다음 PC인지는 재시작/복귀 ISA 규약에 따라 별도 결정해야 하며, **여기서는 완성된 코드 수정이나 실험 성공을 주장하지 않는다**.
- **사후 개선안(D):** `eret` 전체 인코딩 `32'h42000018`을 정확히 디코딩하고, 예외 마스크/Block을 설정한 testbench에서 **예외 진입 → EPC 기대값 저장 → `0x800` 벡터 → ERET → PC 복귀**를 기대값 assertion으로 검증한다. 기존 저장소에 이 수정·성공 결과를 남겼다는 의미가 아니다.

**별개인 syscall:** 일반 syscall은 `SyscallDecoder`의 `$v0/$a0`로 처리되며 `ExpSrc` exception과 연결된 자동 트리거로 작성되지 않았다. `tb_CP0`에 `Inst=0x0000000C`('syscall')을 넣고 `ExpSrc=001`도 함께 준 것을 근거로 **syscall 자체가 CP0 예외를 발생시킨다고 쓰지 않는다**.

## 6. GROUP-04 — Syscall/Statistics/Memory 구현과 ModelSim 관찰

- **Syscall:** $v0=10일 때 `Halt`, 활성 syscall에서는 `Hex<=a0`. Top의 `Halt`는 외부 출력이 아니라 Clock을 `Halt ? 1 : Clock`으로 게이트하는 **내부 wire**.
- **Statistics:** R/I/J 분류 후 Top에서 J/R/I/TotalCycles 누적.
- **Instruction Memory:** `benchmark.hex`/ `exception_service.hex` 이중 512×32 ROM, `PC[11]`으로 예외 ROM 선택. **'PC가 0x800으로 이동하는 RTL 경로'와 '해당 루틴의 정상 실행 성공'은 서로 다른 주장**.
- **Data Memory:** 1024×32 RAM, edge 기반 store, 조합 load.
- **ModelSim:** 10개 `tb_*.v`와 11개 `wave_*.tcl`가 `modelsim/`에 남아 있음. 주요 TB는 기대값 자동 assertion이 아니라 `$display`·파형 출력이며, 그 사실 자체가 무용한 것이 아니라 **검증 방식/범위의 정직한 표현**이다.
- **최종 정리:** [`750d768`](https://github.com/nanocode00/MIPS-CPU/commit/750d76842c21)에서 중복 `verilog/` 자료를 삭제, `modelsim/` 최종 Verilog와 TB·TCL은 유지했다.

## 7. 시간순 개발 흐름

| 기간 | 단계 | 근거 |
|---|---|---|
| 2025.05.02 | 원본 Logisim CPU 저장소 Fork | GitHub repository parent metadata |
| 2025.05.09 | Opcode/Funct/Control/ALU Decoder 초판 | `2f059fde7be0` |
| 2025.05.24 | 32비트 ALU+가감산 회로, Register File 초판 | `74a5eee30fa5`, `538b4339f383` |
| 2025.05.28 | Decoder/ALU/RegFile 수정, Top/CP0/IM/DM/Syscall/Statistics 및 ModelSim TB/TCL 대규모 통합 | `4e8a72fdcaed` |
| 2025.05.28 | 중복 `verilog/` 소스 정리; `modelsim/` 보존 | `750d76842c21` |

## 8. 검증 근거와 주장 경계

| 주장 | 허용 가능한 결론 |
|---|---|
| 기존 Logisim 구조를 참고해 Verilog RTL로 옮김 | **A: Fork/파일 diff로 확인**. 아키텍처 원저작 아님 |
| ALU/Decoder 통합 과정의 오류 수정 | **A: 수정 전후 코드** + **B: 당시 보고서의 파형 디버깅 회고**. 정확한 오류 재현 자극/조건은 일부 미확정 |
| RegisterFile/Memory/PC datapath 구현 | **A: 직접 코드·TB 추가**. 모든 명령어 성공 자동 시험 없음 |
| Syscall/Statistics 검증 | **A: 코드/시뮬레이션 자산** + B: 보고서. Halt Top 출력 주장 금지 |
| CP0 exception/EPC/ERET 종단 정상동작 | **미완성(C):** 예외 진입 시 복귀 주소 저장 실패를 겪고 시간 부족으로 해결 못함. 보존 RTL 조건 불일치·TB 미자극(A). 정상 복귀 검증 완료가 아님 |
| 전체 MIPS CPU 기능·pipeline·성능 | **단일 사이클 RTL 구조**만 확인, pipeline이나 캐시/forwarding 구현·전 명령 대응·성능 최적화 주장 금지 |

## 9. 이력서/포트폴리오 압축 후보

최종 등록은 다른 프로젝트와 비교해 선별한다. 현 자료 기준으로 강한 후보는 다음 두 묶음이다.

1. **RTL 구조 재구현:** 기존 Logisim CPU 회로를 참조해 32-bit single-cycle 구조를 Verilog RTL로 구성, Control/ALU/RegisterFile/Memory와 Top datapath 연결.
2. **제어·연산 블록 통합 디버깅:** Opcode/Funct 비교식, `ALUop` 비트 배치, ALU output 선언·모듈 연결·overflow 연산과 RegisterFile 초기화/인터페이스를 수정하고 ModelSim testbench·waveform으로 내부 제어 신호를 추적한 경험.

**CP0는 '예외 처리 성공 경험'이 아니다.** 대신 *예외 경로를 구성했으나 복귀 주소 저장 문제를 해결하지 못하고 마감한 실패/미완성 구현 경험*으로 보존한다. 추가 자료가 발견되면 경계를 재평가한다.

## 10. 근거 인덱스·남은 질문

- [원본 Logisim CPU](https://github.com/yuxincs/MIPS-CPU), [포크 및 본인 RTL](https://github.com/nanocode00/MIPS-CPU/tree/main/modelsim), [커밋별 작업 원장](verilog-cpu-commit-ledger.md).
- [당시 보고서 PDF](https://drive.google.com/file/d/1sXOy8DC9khJu00A1rDyuSzUtrK_np_2S/view); 최종 TB/RTL과 충돌할 때는 **문구보다 실제 코드와 시험 자극**을 우선.
- **추가 증빙 1 — 회고 확인(C, 2026-10-27):** 예외 경로 진입 뒤 EPC 복귀 주소 저장 실패, 시간 부족으로 미해결 종료한 경험을 직접 확인. 추가 자료가 있으면 당시 입력·파형·EPC 값을 대조하되, 정상 ERET 성공을 전제하지 않는다.
- **추가 증빙 2:** Decoder 코드의 동등 비교 변경, ripple adder/subtractor 대신 산술 연산자를 사용한 의사결정 이유. 당시 상황이 불명확하면 D 추정으로만 남김.
- **추가 증빙 3:** `tb_ALU` 등 출력값 캡처나 보고서 스크린샷에서 확인 가능한 구체적인 성공/실패 범위. 현재 TB 파일만으로 assertion 통과 기록은 없음.

`resume.md`는 본 검토에서 수정하지 않는다.
