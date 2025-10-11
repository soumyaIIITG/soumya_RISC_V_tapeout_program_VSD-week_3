
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

---------

# Static Timing Analysis (STA) of VSDBabySoC with OpenSTA

This document provides a comprehensive overview of Static Timing Analysis (STA) fundamentals and their practical application on the VSDBabySoC project using the open-source tool, OpenSTA. It covers the installation process, basic and advanced analysis techniques, and a multi-corner PVT (Process-Voltage-Temperature) analysis to ensure the design's timing robustness.

### Introduction to Static Timing Analysis

**Static Timing Analysis (STA)** is a fundamental technique used to verify the timing of a digital design without requiring dynamic simulation. The analysis is "static" because it does not depend on the specific data values being applied at the input pins. This is in contrast to timing simulation, which verifies functionality and timing by applying input stimuli and observing the circuit's behavior over time. STA can be performed at many different stages of the implementation flow, from synthesis to post-layout verification.

<img width="932" height="526" alt="image" src="https://github.com/user-attachments/assets/9aadc886-8b77-420d-abae-7f258a5c9360" />

In a CMOS digital design flow, the static timing analysis can be performed at many different stages of the implementation. Figure below shows a typical flow.

<img width="509" height="501" alt="image" src="https://github.com/user-attachments/assets/44c05578-9dad-4b16-831a-242aeef14486" />

**OpenSTA** is an open-source static timing analyzer that uses a Tcl command interpreter to read a design, specify timing constraints, and generate detailed timing reports.

<img width="1010" height="341" alt="image" src="https://github.com/user-attachments/assets/3437a7e7-543b-4296-8a9f-8aad0d9f1d61" />


### Core STA Concepts

#### Timing Paths

A timing path is the logical route a signal takes through a circuit, from a start point to an end point, passing through combinational logic.

  * **Start Point**: Typically an input port or the clock pin of a register where data is launched.
  * **End Point**: Typically an output port or the data input pin (D pin) of a register where data is captured.
  * **Combinational Logic**: The gates and interconnects between the start and end points.

The four primary types of timing paths are Input to Register (in2reg), Register to Register (reg2reg), Register to Output (reg2out), and Input to Output (in2out).

#### Setup and Hold Checks

  * **Setup Check**: This ensures that data is stable for a minimum amount of time **before** the capturing clock edge arrives. A setup violation limits the maximum operating frequency of the design.
  * **Hold Check**: This ensures that data remains stable for a minimum amount of time **after** the capturing clock edge has passed. A hold violation can cause data corruption and is independent of the clock frequency.

#### Slack Calculation

Slack is the margin between the required time and the actual arrival time of a signal.

  * **Setup Slack** = Data Required Time - Data Arrival Time 
  * **Hold Slack** = Data Arrival Time - Data Required Time

A **positive slack** indicates that timing is met, while a **negative slack** signifies a timing violation.

### OpenSTA Installation and Usage

The recommended installation method uses Docker, which packages all necessary dependencies.

1.  **Clone the Repository**:

    ```bash
    git clone https://github.com/parallaxsw/OpenSTA.git
    cd OpenSTA
    ```

2.  **Build the Docker Image**:

    ```bash
    docker build --file Dockerfile.ubuntu22.04 --tag opensta .
    ```

3.  **Run the OpenSTA Container**:
    This command starts an interactive session and mounts your local home directory to the `/data` directory inside the container.

    ```bash
    docker run -i -v $HOME:/data opensta
    ```

    <img width="1208" height="224" alt="image" src="https://github.com/user-attachments/assets/061844c8-3fe4-40af-8b93-92f4c01198b1" />


### Performing Timing Analysis

#### Basic Analysis with Inline Commands

Once inside the OpenSTA shell (indicated by the `%` prompt), you can perform a quick STA run. This example uses a slow library for setup (max delay) analysis.

```tcl
# Load the Liberty timing library
read_liberty /OpenSTA/examples/nangate45_slow.lib.gz

# Load the gate-level Verilog netlist
read_verilog /OpenSTA/examples/example1.v

# Link the design to the library cells
link_design top

# Define a 10ns clock and set input delays
create_clock -name clk -period 10 {clk1 clk2 clk3}
set_input_delay -clock clk 0 {in1 in2}

# Generate the timing report
report_checks
```
<img width="1207" height="802" alt="image" src="https://github.com/user-attachments/assets/e8917926-a78b-44b7-bc27-b230950cd8ab" />



## Analyzing Report Outcomes

This section details the synthesis of a sample Verilog file, `example1.v`, and showcases various advanced reporting commands available in OpenSTA for a thorough timing analysis.

### Verilog Netlist: example1.v

The following Verilog module is used for the analysis. It consists of three flip-flops, a buffer, and an AND gate.

```verilog
module top (in1, in2, clk1, clk2, clk3, out);
  input in1, in2, clk1, clk2, clk3;
  output out;
  wire r1q, r2q, u1z, u2z;

  DFF_X1 r1 (.D(in1), .CK(clk1), .Q(r1q));
  DFF_X1 r2 (.D(in2), .CK(clk2), .Q(r2q));
  BUF_X1 u1 (.A(r2q), .Z(u1z));
  AND2_X1 u2 (.A1(r1q), .A2(u1z), .ZN(u2z));
  DFF_X1 r3 (.D(u2z), .CK(clk3), .Q(out));
endmodule
````

### Yosys Synthesis

The netlist was synthesized using Yosys with the following commands, mapping the design to the `nangate45` standard cell library.

```shell
soumya@soumya-VirtualBox:~$ cd ~/VSDBabySoC/OpenSTA/examples/
soumya@soumya-VirtualBox:~/VSDBabySoC/OpenSTA/examples$ yosys
yosys> read_liberty -lib nangate45_slow.lib
yosys> read_verilog example1.v
yosys> synth -top top
3.25. Printing statistics.

=== top ===

   Number of wires:                 10
   Number of wire bits:             10
   Number of public wires:          10
   Number of public wire bits:      10
   Number of ports:                  6
   Number of port bits:              6
   Number of memories:               0
   Number of memory bits:            0
   Number of processes:              0
   Number of cells:                  5
     AND2_X1                       1
     BUF_X1                        1
     DFF_X1                        3

3.26. Executing CHECK pass (checking for obvious problems).
Checking module top...
Found and reported 0 problems.
```
<img width="871" height="427" alt="image" src="https://github.com/user-attachments/assets/b3f16faa-0219-4dcf-bd3f-5efc14205bfb" />


### SPEF-Based Timing Analysis

For more accurate, post-layout timing analysis, a **SPEF (Standard Parasitic Exchange Format)** file can be included. This enables a more realistic computation of delay and slack by incorporating RC parasitic data from the physical layout.

The flow in OpenSTA is as follows:

```tcl
# Start the container
docker run -i -v $HOME:/data opensta

# Run analysis commands
read_liberty /OpenSTA/examples/nangate45_slow.lib.gz
read_verilog /OpenSTA/examples/example1.v
link_design top
read_spef /OpenSTA/examples/example1.dspef
create_clock -name clk -period 10 {clk1 clk2 clk3}
set_input_delay -clock clk 0 {in1 in2}
report_checks
```
<img width="1227" height="616" alt="image" src="https://github.com/user-attachments/assets/59f886b8-c5b2-43ed-ab16-99054e8d4a1f" />



### Advanced Reporting Options in OpenSTA

OpenSTA provides several commands to generate detailed reports for deep-dive analysis.

#### Report Capacitance per Stage

This report shows the net capacitance at each stage of a timing path, helping to identify high-capacitance nodes that might be causing delays.

```shell
% report_checks -digits 4 -fields capacitance
```

<img width="1106" height="715" alt="image" src="https://github.com/user-attachments/assets/e334b9cb-b086-4f8e-af32-8621d122cf89" />


#### Report Timing with Capacitance, Slew, Input Pins, and Fanout

For an even more detailed view, this command reports capacitance, slew (transition time), input pins, and fanout for each stage of the path.

```shell
% report_checks -digits 4 -fields [list capacitance slew input_pins fanout]
```

<img width="1088" height="743" alt="image" src="https://github.com/user-attachments/assets/0dbc2558-be34-4084-abaa-b1638c9cbdd8" />


#### Report Total and Component Power

The `report_power` command performs static power analysis using Liberty power models, breaking down consumption into internal, switching, and leakage components.

```shell
% report_power
```

<img width="1015" height="392" alt="image" src="https://github.com/user-attachments/assets/3da7a013-84cc-4c28-b4e1-ba4be0abdd17" />


#### Report Pulse Width Checks

This command verifies that the clock pulses are wide enough to meet the minimum requirements for the sequential cells in the clock network.

```shell
% report_pulse_width_checks
```

<img width="873" height="221" alt="image" src="https://github.com/user-attachments/assets/d554843b-1c97-470b-a5b6-7ffabf2b7637" />


#### Report Units

Displays the default units used for physical quantities in reports and commands.

```shell
% report_units
```

<img width="987" height="191" alt="image" src="https://github.com/user-attachments/assets/eb8ca165-04e8-4308-919c-a890f8005c43" />


```
```











#### Automated Analysis with a Tcl Script

For repeatable and comprehensive analysis, commands are placed in a `.tcl` script. The script below performs both setup (max) and hold (min) checks by loading both slow and fast corner libraries.

```tcl
# Load liberty files for max and min analysis
read_liberty -max /data/VLSI/VSDBabySoC/OpenSTA/examples/nangate45_slow.lib.gz
read_liberty -min /data/VLSI/VSDBabySoC/OpenSTA/examples/nangate45_fast.lib.gz

# Read the gate-level Verilog netlist and link the design
read_verilog /data/VLSI/VSDBabySoC/OpenSTA/examples/example1.v
link_design top

# Define clocks and input delays
create_clock -name clk -period 10 {clk1 clk2 clk3}
set_input_delay -clock clk 0 {in1 in2}

# Generate a full min/max timing report
report_checks -path_delay min_max
```

This script can be executed non-interactively via Docker:

```bash
docker run -it -v $HOME:/data opensta /data/VLSI/VSDBabySoC/OpenSTA/examples/min_max_delays.tcl
```

### Case Study: VSDBabySoC Multi-Corner PVT Analysis

To ensure the VSDBabySoC design is robust, STA was performed across 13 different **PVT (Process-Voltage-Temperature)** corners using the available Sky130 timing libraries. Critical corners for analysis include:

  * **Worst Max (Setup-critical)**: Slow-slow process, high temperature, low voltage (`ss_HighTemp_LowVolt`).
  * **Worst Min (Hold-critical)**: Fast-fast process, low temperature, high voltage (`ff_LowTemp_HighVolt`).

A Tcl script was created to iterate through each of the 13 library files, run a full timing analysis, and extract key metrics such as **Worst Hold Slack (WHS)**, **Worst Setup Slack (WSS)**, **Worst Negative Slack (WNS)**, and **Total Negative Slack (TNS)** into summary files.

After fixing a syntax error in the `avsdpll.lib` file (replacing `//` comments with `/*...*/` block comments), the analysis was executed successfully.

### Post-Synthesis STA Results and Visualization

The metrics from all 13 PVT corner runs were compiled into a summary table.

This data was then plotted to visualize the timing performance across the different operating conditions.

#### Worst Hold Slack (WHS) Across Corners

#### Worst Setup Slack (WSS) Across Corners

#### Worst Negative Slack (WNS) Across Corners

#### Total Negative Slack (TNS) Across Corners

