# Superscalar Processor with Dual Pipelines

A **superscalar processor** is an advanced microprocessor capable of executing more than one instruction during a single clock cycle. This capability is achieved by incorporating **dual pipelines**, which allow simultaneous execution of multiple instructions, significantly improving processor throughput. This project implements a superscalar processor using **dual pipelining** to support **instruction-level parallelism** and efficient hazard management.

## Approach to Superscalar Pipeline Design

The superscalar processor consists of two parallel pipelines that handle different stages of instruction execution: **Fetch, Decode, Execute, Memory Access, and Writeback**. The use of dual pipelining introduces challenges such as resource contention and **hazard management**, which are addressed through various techniques.

The pipeline stages are defined as follows:

- **Fetch Stage**: This stage is responsible for fetching the next instruction(s) from memory. The **instruction fetch unit** can fetch two instructions simultaneously, which are then dispatched to the respective pipelines.

- **Decode Stage**: During decoding, instructions are parsed to determine their operation and required operands. A dual instruction decoder processes two instructions concurrently, ensuring that both pipelines receive the necessary information.

- **Execute Stage**: The execute stage performs arithmetic and logical operations. An **ALU (Arithmetic Logic Unit)** is incorporated into each pipeline, allowing parallel execution of independent operations.

- **Memory Access Stage**: This stage handles data memory operations such as load and store. Both pipelines share a unified memory unit, with arbitration logic to manage simultaneous access requests.

- **Writeback Stage**: In this final stage, computation results are written back to the register file. Dual write ports allow both pipelines to perform writeback operations concurrently.


<img width="748" alt="image" src="https://github.com/user-attachments/assets/58e7a89c-6620-4e4d-b0ea-14ad207c8b47">

<img width="612" alt="image" src="https://github.com/user-attachments/assets/f8892b08-c314-4d5e-912a-72d1be562846">

## ISA Design

The superscalar processor is designed as a **Harvard machine** with dedicated instruction memory. The **Instruction Set Architecture (ISA)** supports basic arithmetic, logical, and branching operations. The ISA specifications are as follows:

- **16-bit Instruction Word** (IR size = 16 bits)
- **Number of Instructions**: 16
- **Number of Registers**: 8 (with the eighth register being the Flag Register `r7`)
- **Memory Size**: 2^11 blocks
- **Program Counter (PC) Size**: 16 bits
- **Opcode Size**: 4 bits
- **Immediate Bit Size**: 1 bit
- **Register Size**: 3 bits

### Instruction Format

The instruction format is divided as follows:

<img width="703" alt="image" src="https://github.com/user-attachments/assets/a8b4344b-c5f9-4b7e-9c00-077c640cb979">

The custom processor supports 16 instructions:

1. **NOP** - No operation (Opcode: 0000)
2. **ADD rd, rs1, rs2** - Add (Opcode: 0001)
3. **SUB rd, rs1, rs2** - Subtract (Opcode: 0010)
4. **MUL rd, rs1, rs2** - Multiply (Opcode: 0011)
5. **LOAD rd, [rs1 + imm]** - Load (Opcode: 0100)
6. **STORE rs1, [rs2 + imm]** - Store (Opcode: 0101)
7. **CMP rs1, rs2** - Compare (Opcode: 0110)
8. **MOV rd, rs1** - Move (Opcode: 0111)
9. **OR rd, rs1, rs2** - Logical OR (Opcode: 1000)
10. **AND rd, rs1, rs2** - Logical AND (Opcode: 1001)
11. **NOT rd, rs1** - Logical NOT (Opcode: 1010)
12. **LSL rd, rs1, rs2** - Logical Shift Left (Opcode: 1011)
13. **JUMP target** - Unconditional Jump (Opcode: 1100)
14. **LSR rd, rs1, rs2** - Logical Shift Right (Opcode: 1101)
15. **BEQ target** - Branch if Equal (Opcode: 1110)
16. **BGT target** - Branch if Greater (Opcode: 1111)

*Note: Division and modulo operations are generally significantly slower than addition and multiplication. Their inclusion increases the maximum delay of the Execute stage, potentially making the pipeline unbalanced. Consequently, many simple pipelined processors avoid implementing these instructions.*

## CPI Formula and Instruction-Level Parallelism

The **Cycles Per Instruction (CPI)** is a key metric for evaluating processor performance. In an ideal scenario without hazards or stalls, the dual-pipeline processor aims to achieve a CPI of approximately **0.5**, since it can potentially execute two instructions per cycle. However, achieving this ideal CPI in practice is challenging due to factors such as pipeline hazards, resource limitations, and data dependencies between instructions.

The **effective CPI** of the processor is calculated using the formula:

**CPI = (Total Cycles) / (Total Instructions Executed)**

In the dual-pipeline design, factors such as pipeline stalls, branch mispredictions, and data hazards increase the total cycle count, resulting in a higher CPI. The implementation aims to minimize these stalls and maintain a CPI as close to 0.5 as possible through effective hazard management techniques.

## Hazard Management: Stalling and Data Forwarding

**Hazard management** in a pipelined processor refers to handling situations where instructions cannot execute simultaneously due to resource conflicts or data dependencies. Three primary types of hazards are addressed:

1. **Data Hazards**: These occur when instructions depend on the results of previous instructions that are still in the pipeline. Data hazards are addressed using **data forwarding** and **stalling** mechanisms.

   - **Data Forwarding**: This technique directly transfers results from one pipeline stage to another without waiting for the normal writeback stage. A **relayer unit** implements forwarding paths, ensuring that data is available for dependent instructions as soon as possible. The relayer unit monitors the pipeline stages and forwards data from the Execute or Memory stages to the Decode stage when necessary, thereby reducing stalls.

   - **Stalling**: If forwarding is not feasible, **pipeline stalls** are introduced to delay instruction execution until the required data becomes available. Dedicated control logic generates control signals to freeze the relevant pipeline stages while allowing other stages to continue, thereby minimizing the performance impact.

2. **Structural Hazards**: These occur when two instructions require the same hardware resource simultaneously. In the dual-pipeline design, **structural hazards** are minimized through resource duplication (e.g., dual ALUs) and arbitration logic for shared resources such as memory access units. The control unit manages resource contention to avoid unnecessary stalls.

3. **Control Hazards**: Control hazards arise due to branch instructions that change the flow of execution. A **branch prediction mechanism** is used to mitigate control hazards. The branch predictor attempts to predict the outcome of branch instructions, allowing the pipeline to continue fetching subsequent instructions without waiting for the branch to be resolved. In case of a misprediction, the pipeline is flushed, and the correct instructions are fetched, introducing a performance penalty.

## Hazard Management Implementation (Referencing GitHub Repository)

The **hazard detection** logic is integrated with the pipeline architecture. A combination of **buffers** and a **relayer unit** manages data dependencies and control signal propagation across the two pipelines. Detailed information about the implementation is available in the GitHub repository [here](https://github.com/namang17/Superscalar_Processor_with_Dual_Pipeline/tree/main/src).

- **Buffers Before Pipelining**: Buffers are used before the Decode and Execute stages to synchronize instructions between the dual pipelines. These buffers temporarily store instruction and data values, managing instruction flow, reducing potential conflicts, and enabling smoother transitions between pipeline stages.

- **Relayer Unit**: The **relayer unit** detects data dependencies between the two pipelines and initiates data forwarding when applicable. This unit ensures that dependent instructions do not result in incorrect execution. It also coordinates with the hazard detection logic to determine when stalling is required, providing an integrated approach to managing data hazards.

- **Control Unit**: The **control unit** generates control signals for instruction fetch, decode, execution, memory access, and writeback stages. It also manages pipeline flushing in case of branch mispredictions, ensuring that both pipelines do not execute incorrect instructions.

## Managing Dual Pipelines with Limited Resources

Managing **dual pipelines** with limited resources requires careful handling of shared components, such as the **register file** and **memory units**. The following mechanisms are incorporated into the design:

- **Resource Arbitration**: Both pipelines access shared resources through **resource arbitration** techniques. The arbitration logic monitors access requests and grants access to prevent resource conflicts.

- **Control Unit Coordination**: A dedicated **control unit** coordinates the simultaneous execution of instructions across the two pipelines, ensuring that resource contention is minimized while maximizing parallelism. It also handles situations where both pipelines need to access the same memory location, ensuring that memory operations are performed correctly.

- **Register File Design**: The **register file** supports dual read and write ports, allowing both pipelines to access registers concurrently without causing data corruption. Additional control logic handles simultaneous read and write requests, ensuring data consistency across both pipelines.

- **Branch Prediction and Flushing**: A branch prediction unit predicts the outcome of branch instructions to keep the pipeline full. When a misprediction occurs, the control unit flushes incorrect instructions from the pipeline, minimizing the performance penalty.

## Instruction Execution Principles

- **Negative Clock Edge Execution**: Instructions are executed during the negative edge of the clock cycle to ensure synchronization across the dual pipelines.

- **Dependency Checking Before Execution**: Before two instructions enter the dual pipelines, dependency checks are performed for **WAW (Write After Write)**, **RAW (Read After Write)**, and **WAR (Write After Read)** hazards. If any of these hazards exist, the instructions are executed serially in a single pipeline while inserting a **bubble (NOP)** in the other pipeline.

- **Instruction Fetch and Hazard Resolution**: Two instructions are fetched simultaneously from shared memory using two Program Counters (PCs), enabling instructions to enter both pipelines concurrently. The instructions are then stored in two **Instruction Registers (IR1 and IR2)** before proceeding through the pipelines. **RAW hazards** are resolved using data forwarding wherever possible, while stalling is used otherwise to minimize stall cycles.

## Conclusion

The superscalar processor with dual pipelines is designed to enhance processing throughput by executing two instructions simultaneously. Techniques such as data forwarding, stalling, branch prediction, and resource arbitration address the complexities introduced by instruction-level parallelism and limited resource availability.

The GitHub repository contains the Verilog code, testbenches, and documentation outlining the pipeline architecture and hazard management logic.

The **dual-pipelining** approach, combined with hazard management techniques, demonstrates the potential of superscalar architectures to improve throughput through parallel instruction execution while maintaining correctness and efficiency.

<img width="603" alt="image" src="https://github.com/user-attachments/assets/8e0aa196-6381-4654-a46a-828382646f3b">

The ALU circuit diagram illustrates the different functional units (Adder, Multiplier, Shift Unit, Logical Unit, and Mov Unit) and how the control signals (`isAdd`, `isSub`, `isMul`, `isCmp`, etc.) determine the operations performed on operands `A` and `B`. The outputs from these units are directed to `aluResult`, which is subsequently used in the pipeline stages.




