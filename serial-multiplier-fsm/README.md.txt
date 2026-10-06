# 8-bit Serial Multiplier with FSM and Datapath Architecture

A Verilog implementation of an 8-bit iterative unsigned multiplier using the **shift-and-add algorithm**, featuring a decoupled **Control Unit (FSM)** and **Datapath (Registers + Arithmetic Logic)** architecture. The project includes timing, hardware resource trade-off analysis, and benchmarking against a pure combinational multiplier on the Xilinx ZCU104 FPGA.

Target Board: **Xilinx ZCU104** | Tool: **Vivado 2022.1** | Language: **Verilog**

---

## 1. Architectural Overview
The design mirrors the fundamental structure of a CPU by strictly separating control logic from computation:
* **FSM Controller (3 states):** `IDLE` (load operands), `CALC` (shift-and-add execution loop), and `DONE_ST` (output assert).
* **Datapath:** 16-bit accumulator (`acc`), 8-bit multiplicand register (`a_reg`), shift register (`b_reg`), and an iteration counter.
[A, B, start]
         |
         v
 +---------------+      Control Signals (load, add, shift)      +-----------------+
 |      FSM      | -------------------------------------------> |    DATAPATH     |
 |  (Controller) | <------------------------------------------- |  (Regs + Adder) |
 +---------------+       Status Signals (b_lsb, count_done)     +-----------------+
                                                                         |
                                                                         v
                                                                  [product, done]
### State Machine Diagram
![FSM Diagram](doc/fsm_diagram.png)

---

## 2. Verification & Simulation
Verified with a self-checking testbench (`tb/tb_serial_mult.v`) running at 100 MHz. The testbench monitors completion via the `done` flag, prevents infinite simulation stalls with a 100-cycle timeout watch, and asserts results against reference calculations.

* Validated edge cases: `0 x 10`, `1 x 200`, `255 x 1`, `255 x 255`, and arbitrary corner cases.
* **Result:** 10/10 test cases passed with 0 failures (10 clock cycles per operation).

![Simulation Waveform](sim/waveform.png)
![Console Output](sim/console_pass.png)

---

## 3. Hardware Comparison: Serial vs. Combinational Multiplication

Both designs were synthesized targeting the Xilinx ZCU104 board using Vivado:

| Design Architecture | CLB LUTs | CLB FFs | DSP Blocks | Latency | Clock Cycles |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **Serial Multiplier (`serial_mult`)** | 30 | 56 | 0 | ~140 ns | 10 cycles |
| **Combinational Multiplier (`comb_mult`)** | 70 | 0 | 0 | ~10 ns (prop delay) | 1 cycle |

### Key Trade-off Takeaways:
* **Area vs. Registers:** The combinational design requires over 2× more LUTs (70 vs 30) due to full concurrent parallel adder trees, but uses zero flip-flops. The serial architecture trades latency for area savings by iteratively reusing a single adder via registers.
* **Scalability ($O(N)$ vs $O(N^2)$):** Extending from 8-bit to 16-bit scaling in serial multipliers scales linearly in register count ($O(N)$), whereas pure combinational logic scaling exhibits quadratic LUT growth ($O(N^2)$).