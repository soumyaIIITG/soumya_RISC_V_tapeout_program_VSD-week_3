
-----

# GLS OF BABYSOC

## POST-SYNTHESIS SIMULATION

### Purpose of GLS:

Gate-Level Simulation is used to verify the functionality of a design after the synthesis process. Unlike behavioral or RTL (Register Transfer Level) simulations, which are performed at a higher level of abstraction, GLS works on the netlist generated post-synthesis. This netlist includes the actual gates and connections used to implement the design.

### Key Aspects of GLS for BabySoC:

1.  **Verification with Timing Information:**

      * GLS is performed using Standard Delay Format (SDF) files to ensure timing correctness.
      * This checks if the SoC behaves as expected under real-world timing constraints.

2.  **Design Validation Post-Synthesis:**

      * Confirms that the design's logical behavior remains correct after mapping it to the gate-level representation.
      * Ensures that the design is free from issues like metastability or glitches.

3.  **Simulation Tools:**

      * Tools like Icarus Verilog or a similar simulator can be used for compiling and running the gate-level netlist.
      * Waveforms are typically analyzed using GTKWave.

4.  **Importance for BabySoC:**

      * BabySoC consists of multiple modules like the RISC-V processor, PLL, and DAC. GLS ensures that these modules interact correctly and meet the timing requirements in the synthesized design.

## Here is the step-by-step execution plan for running the commands manually:

### **Step 1: Load the Top-Level Design and Supporting Modules**

```bash
yosys
```
<img width="748" height="436" alt="image" src="https://github.com/user-attachments/assets/7fc76cd4-659a-47c1-ade1-3af06c068052" />


Inside the Yosys shell, run:

```yosys
read_verilog /home/ananya123/VSDBabySoCC/VSDBabySoC/src/module/vsdbabysoc.v
read_verilog -I /home/ananya123/VSDBabySoCC/VSDBabySoC/src/include /home/ananya123/VSDBabySoCC/src/module/rvmyth.v
read_verilog -I /home/ananya123/VSDBabySoCC/VSDBabySoC/src/include /home/ananya123/VSDBabySoCC/src/module/clk_gate.v
```

<img width="1212" height="433" alt="image" src="https://github.com/user-attachments/assets/945c03dd-80b2-4a34-b60c-19b05b292743" />


-----

### **Step 2: Load the Liberty Files for Synthesis**

Inside the same Yosys shell, run:

```yosys
read_liberty -lib /home/ananya123/VSDBabySoCC/VSDBabySoC/src/lib/avsdpll.lib
read_liberty -lib /home/ananya123/VSDBabySoCC/VSDBabySoC/src/lib/avsddac.lib
read_liberty -lib /home/ananya123/VSDBabySoCC/VSDBabySoC/src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

<img width="1070" height="272" alt="image" src="https://github.com/user-attachments/assets/96756041-6af0-4095-bc9c-395f02debef9" />


-----

### **Step 3: Run Synthesis Targeting `vsdbabysoc`**

```yosys
synth -top vsdbabysoc
```
<img width="1156" height="442" alt="image" src="https://github.com/user-attachments/assets/4c3fee40-5a8e-44a5-9347-2af30d47325e" />
<img width="1203" height="466" alt="image" src="https://github.com/user-attachments/assets/359fc4f6-6dc9-41bd-a194-17fb4b3d84cc" />
<img width="1167" height="625" alt="image" src="https://github.com/user-attachments/assets/11dcfacc-423d-4398-9274-9dce13c4815d" />
<img width="1062" height="422" alt="image" src="https://github.com/user-attachments/assets/4d3e57a5-66d7-404e-91d8-ad536bbd30b9" />


-----

### **Step 4: Map D Flip-Flops to Standard Cells**

```yosys
dfflibmap -liberty /home/ananya123/VSDBabySoCC/VSDBabySoC/src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```
<img width="1200" height="573" alt="image" src="https://github.com/user-attachments/assets/e71fa16f-3052-4e2d-ad9b-8fe405dded5c" />


-----

### **Step 5: Perform Optimization and Technology Mapping**

```yosys
opt
abc -liberty /home/ananya123/VSDBabySoCC/VSDBabySoC/src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib -script +strash;scorr;ifraig;retime;{D};strash;dch,-f;map,-M,1,{D}
```
<img width="1192" height="683" alt="image" src="https://github.com/user-attachments/assets/1daab3ce-aa1f-41df-8a76-1fc4e3ea5505" />
<img width="1187" height="632" alt="image" src="https://github.com/user-attachments/assets/61cb05db-d4cc-485a-a5be-ba2bea72c983" />


-----

### **Step 6: Perform Final Clean-Up and Renaming**

```yosys
flatten
setundef -zero
clean -purge
rename -enumerate
```
<img width="898" height="406" alt="image" src="https://github.com/user-attachments/assets/aeb7886b-c4db-4735-a2c4-9852d144e814" />


-----

### **Step 7: Check Statistics**

```yosys
stat
```
<img width="1193" height="736" alt="image" src="https://github.com/user-attachments/assets/a2defad1-9a3f-47fa-97f1-083f151e9f6d" />
<img width="998" height="577" alt="image" src="https://github.com/user-attachments/assets/19f0eae3-c819-4928-8a4e-5419ba52d205" />


-----

### **Step 8: Write the Synthesized Netlist**

```yosys
write_verilog -noattr /home/ananya123/VSDBabySoCC/VSDBabySoC/output/post_synth_sim/vsdbabysoc.synth.v
```
<img width="937" height="108" alt="image" src="https://github.com/user-attachments/assets/128540c7-9c8c-440c-bba9-9a0d46962828" />


-----

## POST\_SYNTHESIS SIMULATION AND WAVEFORMS

-----

### **Step 1: Compile the Testbench**

Run the following `iverilog` command to compile the testbench:

```bash
iverilog -o /home/ananya123/VSDBabySoCC/VSDBabySoC/output/post_synth_sim/post_synth_sim.out -DPOST_SYNTH_SIM -DFUNCTIONAL -DUNIT_DELAY=#1 -I /home/ananya123/VSDBabySoCC/VSDBabySoC/src/include -I /home/ananya123/VSDBabySoCC/VSDBabySoC/src/module /home/ananya123/VSDBabySoCC/VSDBabySoC/src/module/testbench.v
```

-----

### **Step 2: Navigate to the Post-Synthesis Simulation Output Directory**

```bash
cd output/post_synth_sim/
```

-----

### **Step 3: Run the Simulation**

```bash
./post_synth_sim.out
```

-----

### **Step 4: View the Waveforms in GTKWave**

```bash
gtkwave post_synth_sim.vcd
```

<img width="1017" height="527" alt="image" src="https://github.com/user-attachments/assets/6a264ee1-5928-4477-a9c0-56626b5500eb" />
<img width="1025" height="655" alt="image" src="https://github.com/user-attachments/assets/25f6ed79-e8f5-4ec7-8c75-51def94e6bc7" />



-----


# 🧠 Static Timing Analysis (STA) — Complete Overview

This document summarizes key concepts in **Static Timing Analysis (STA)** — the process of verifying timing in digital circuits to ensure correct operation at the desired clock frequency.

---

## ⚙️ 1. Timing Path Overview

A **timing path** represents signal propagation between sequential elements:
Launch Flip-Flop → Combinational Logic → Capture Flip-Flop

Each path is analyzed for **setup** and **hold** requirements based on the clock signal.

### 🔸 Components of a Timing Path
- **Startpoint:** Launch flip-flop or input port
- **Endpoint:** Capture flip-flop or output port
- **Combinational Logic:** Gates and interconnects between start and end points

---

## ⏱️ 2. Arrival Time (AAT), Required Time (RAT), and Slack

| Term | Meaning | Equation |
|---|---|---|
| **Arrival Time (AAT)** | When data actually arrives at endpoint | Sum of all preceding delays |
| **Required Time (RAT)** | Latest time data *must* arrive to be valid | Depends on clock period & setup time |
| **Slack** | Margin between required and actual arrival | `Slack = RAT - AAT` |

### ✅ Interpretation
- **Positive Slack** → Timing met
- **Zero Slack** → Just meets timing
- **Negative Slack** → Timing violation

---

## 🧭 3. Clock Definitions

The clock serves as the timing reference for all sequential elements.

| Parameter | Description |
|---|---|
| **Clock Period (Tclk)** | Time between active clock edges |
| **Clock Skew** | Difference in arrival time of clock at launch & capture flops |
| **Clock Jitter** | Random variation of clock edge position |
| **Clock Latency** | Delay from clock source to flip-flop pin |

### 🧩 Clock Path Interaction
For two flops (FF1 → FF2):
- **Launch Edge:** Clock at FF1 triggers data output (clk→Q delay)
- **Capture Edge:** Clock at FF2 captures incoming data
- **Skew & Jitter** directly affect setup and hold timing

---

## 🧮 4. Setup and Hold Checks

### 🔹 Setup Check — “Data should not be late”
Ensures that data arrives **before** the next active clock edge.
Tclk ≥ (Tclk→Q + Tlogic + Tsetup) - Tskew

**Violation:** Data arrives too late → **setup violation**
**Impact:** Limits maximum frequency (fmax)

---

### 🔹 Hold Check — “Data should not be early”
Ensures that data remains **stable after** the capture clock edge.
Tclk→Q + Tlogic ≥ Thold - Tskew

**Violation:** Data changes too early → **hold violation**
**Impact:** Causes functional instability; independent of clock period.

---

### ⚖️ Setup vs Hold Summary

| Aspect | Setup Check | Hold Check |
|---|---|---|
| Reference Edge | Next clock edge | Same clock edge |
| Condition | Data not late | Data not early |
| Fix | Add pipeline / slower clock | Insert delay buffers |
| Impact | Frequency limit | Data corruption risk |

---

## 🔗 5. Timing Graph and Node Conversion

STA converts logic gates into a **timing graph**:
- Each **pin or gate** → Node
- Each **connection** → Edge (with delay)

Delays are propagated through this graph to compute:
- **Actual Arrival Time (AAT)** → Forward propagation
- **Required Arrival Time (RAT)** → Backward propagation
- **Slack** → `RAT - AAT`

---

## 🧩 6. Graph-Based vs Path-Based Analysis

| Method | Description | Pros | Cons |
|---|---|---|---|
| **GBA (Graph-Based Analysis)** | Uses worst-case delay at each node | Fast | Pessimistic |
| **PBA (Path-Based Analysis)** | Reconstructs real paths end-to-end | Accurate | Slower |

🧠 **PBA removes pessimism** by recognizing shared logic between paths that GBA overestimates.

---

## ⚙️ 7. Slew, Load, and Clock Checks

| Concept | Description | Effect |
|---|---|---|
| **Slew** | Transition time (10–90%) of a signal | Larger slew = slower transitions |
| **Load** | Total capacitance driven by an output | Higher load = more delay |
| **Clock Checks** | Ensure period, skew, and jitter are within constraints | Violations reduce timing margin |

---

## 🔍 8. Latch Timing and Data Checks

- **Latches** are **level-sensitive**; they allow **time borrowing**.
- Timing analysis includes **open** and **close** windows.
- **Data checks** ensure sequencing between asynchronous signals (e.g., enable vs data).

---

## 🧠 9. Transistor-Level Timing Parameters

| Parameter | Definition |
|---|---|
| **Setup Time (Tsetup)** | Minimum time data must be stable *before* clock edge |
| **Hold Time (Thold)** | Minimum time data must stay stable *after* clock edge |
| **Clk→Q Delay** | Time from clock edge to valid output transition |

These are extracted using SPICE simulations during **library characterization** and stored in `.lib` files.

---

## 📊 10. Eye Diagram and Jitter

- **Eye Diagram** overlays multiple bits to visualize timing margin.
- **Eye opening** = region where data is valid.
- **Jitter** = clock edge uncertainty → narrows eye opening → reduces setup/hold margin.

---

## 🌫️ 11. Sources of Variation

| Source | Effect |
|---|---|
| **Etching Variation** | Alters metal width → changes resistance |
| **Oxide Thickness Variation** | Changes Vth → affects drive current |
| **Temperature & Voltage** | Modify delay and transition times |

### Relation:
Higher Resistance → Lower Current → Higher Delay

---

## 🧮 12. OCV (On-Chip Variation) and Pessimism Removal

**OCV** models manufacturing variations across the chip.

### 🔹 OCV in Setup Analysis
- Increase delay on **launch path**
- Decrease delay on **capture path**
→ More conservative check

### 🔹 OCV in Hold Analysis
- Opposite derating (launch path decreased)

### 🔹 Pessimism Removal
When both launch and capture flops share common clock paths, common delay is cancelled to avoid double-counting variation.

---

## 📘 13. Example: Setup and Hold Slack Calculation

| Parameter | Value (ns) |
|---|---|
| Clock Period | 5.0 |
| Clk→Q Delay | 0.2 |
| Logic Delay | 4.0 |
| Setup Time | 0.3 |
| Hold Time | 0.1 |
| Clock Skew | 0.1 |

### Setup Slack
Slack = (5.0 - 0.3 - 0.1) - (0.2 + 4.0)
= 4.6 - 4.2 = **+0.4 ns** ✅

### Hold Slack
Slack = (0.2 + 4.0) - (0.1 + 0.1)
= 4.2 - 0.2 = **+4.0 ns** ✅

---

## 🧩 14. Summary Table

| Concept | Description |
|---|---|
| **Setup Check** | Data must arrive before capture edge |
| **Hold Check** | Data must remain stable after capture edge |
| **Slack** | Timing margin (RAT - AAT) |
| **Clock Definitions** | Period, latency, skew, jitter |
| **Path-Based Analysis (PBA)** | Realistic, path-accurate analysis |
| **GBA** | Node-based, pessimistic analysis |
| **Slew/Load** | Affect signal speed and delay |
| **Latch Timing** | Includes transparency and borrowing |
| **OCV** | Accounts for process variation |
| **Pessimism Removal** | Cancels common path variation |

---

## 🧩 15. Key Takeaways

- **STA ensures correctness without simulation** by checking all timing paths.
- **Setup violations** limit clock frequency; **Hold violations** cause logic corruption.
- **Positive slack = safe**, **Negative slack = timing failure**.
- **PBA** gives accurate, realistic analysis compared to GBA.
- **Clock skew, jitter, and OCV** are major contributors to timing uncertainty.

---

## 🧰 Tools Commonly Used

| Tool | Purpose |
|---|---|
| **OpenSTA** | Open-source STA tool |
| **Synopsys PrimeTime** | Industry standard STA engine |
| **Cadence Tempus** | Advanced multi-corner STA |
| **Liberty (.lib)** | Timing library for cells |
| **SDF** | Standard Delay Format for back-annotation |

---


---
