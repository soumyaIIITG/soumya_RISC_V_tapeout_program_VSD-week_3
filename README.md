
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

-----
