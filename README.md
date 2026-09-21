# 8086-mars-rover-embedded
An Intel 8086 assembly language embedded system simulation designed to control a Mars Rover's path execution using an 8255 Programmable Peripheral Interface (PPI), matrix keypad, 7-segment display, and an NPN-driven audio sounder[cite: 1].

---
## ⚡ Address Decoding & Port Mappings

Address decoding is performed using a **74LS138 decoder**[cite: 1]. Based on base address calculation ($35\text{H} \text{ AND } \text{F8H} = 30\text{H}$), the 8255 PPI ports operate in **Mode 0** with the Control Word `88H`[cite: 1]:

| Port / Register | I/O Address | Configuration | Connected Peripheral |
| :--- | :--- | :--- | :--- |
| **Port A** | `30H` | Output | Common Anode 7-Segment Display[cite: 1] |
| **Port B** | `32H` | Output | NPN Transistor (2N2222) driving Speaker[cite: 1] |
| **Port C** | `34H` | Lower: Output (PC0–PC3)<br>Upper: Input (PC4–PC6) | 3x4 Matrix Keypad Rows & Columns[cite: 1] |
| **Control Reg** | `36H` | Write Only | Control Word Configuration (`88H`)[cite: 1] |

---

## 🎹 Matrix Keypad Scanning & Mapping

The active-low keypad scanning drives one row LOW at a time and reads column states[cite: 1]. 

| Key Pressed | Row Driven (Output) | Column Detected (Input) | Navigation Command |
| :---: | :---: | :---: | :---: |
| **2** | Row 1 (`PC0`)[cite: 1] | Col 2 (`PC5`)[cite: 1] | **Up / Forward**[cite: 1] |
| **4** | Row 2 (`PC1`)[cite: 1] | Col 1 (`PC4`)[cite: 1] | **Left**[cite: 1] |
| **5** | Row 2 (`PC1`)[cite: 1] | Col 2 (`PC5`)[cite: 1] | **EXECUTE MISSION**[cite: 1] |
| **6** | Row 2 (`PC1`)[cite: 1] | Col 3 (`PC6`)[cite: 1] | **Right**[cite: 1] |
| **8** | Row 3 (`PC2`)[cite: 1] | Col 2 (`PC5`)[cite: 1] | **Down / Backward**[cite: 1] |

### Debouncing Solution
To resolve mechanical contact bounce, a custom **Software Delay Method** was implemented[cite: 1]:
1. Calls a `SETTLE` subroutine (NOP delay loop) after driving rows LOW[cite: 1].
2. Invokes a `WAIT_KEY_RELEASE` polling routine to confirm key release before processing[cite: 1].
3. Applies a `DELAY_200MS` hold to prevent false double-triggering[cite: 1].

---

## 📺 7-Segment Display Mapping

The display utilizes Common Anode logic (`0 = ON`, `1 = OFF`) driven via Port A (`PA0` to `PA7` mapped to segments `a` through `dp`)[cite: 1].

| Character | Output Value | Display Output Meaning | `dp` | `g` | `f` | `e` | `d` | `c` | `b` | `a` |
| :---: | :---: | :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **F** | `8EH` | Forward (`Up`)[cite: 1] | 1 | 0 | 0 | 0 | 1 | 1 | 1 | 0 |
| **b** | `83H` | Backward (`Down`)[cite: 1] | 1 | 0 | 0 | 0 | 0 | 0 | 1 | 1 |
| **L** | `C7H` | Left[cite: 1] | 1 | 1 | 0 | 0 | 0 | 1 | 1 | 1 |
| **r** | `AFH` | Right[cite: 1] | 1 | 0 | 1 | 0 | 1 | 1 | 1 | 1 |
| **E** | `86H` | Critical Failure / Error[cite: 1] | 1 | 0 | 0 | 0 | 0 | 1 | 1 | 0 |

---

## 🔄 Execution Logic Flow

1. **Initialization**: Configures 8255 control word `88H`, initializes display test (`TEST_7SEG`), silences speaker, and maps  terrain grid into RAM[cite: 1].
2. **Input Staging**: Reads keypad commands (`2`, `4`, `6`, `8`) and briefly flashes the letter on the 7-segment display while appending moves to a sequence buffer[cite: 1].
3. **Pre-Execution Check (Phase 1)**: Upon pressing `5`, the processor dry-runs the path in RAM memory to check for boundary overflows or obstacles[cite: 1].
4. **Execution / Alarm (Phase 2)**:
   * **Safe Path**: Outputs moves sequentially to the 7-segment display with ~1 second software delay per step[cite: 1].
   * **Crash**: Displays `'E'` and triggers an alarm melody ("Twinkle Twinkle") on the sounder[cite: 1].
5. **Reset**: Memory buffer resets to length 0 and loops back to input phase[cite: 1].

---

## 📂 File Structure

* `src/Microprocessors_hardwareproject_code.asm`: 8086 Assembly source code[cite: 2].
* `simulation/microprocessors_hardware_project.pdsprj`: Proteus Isis schematic and design workspace[cite: 2].
* `docs/project hardware.pdf`: Technical documentation and system schematics[cite: 1].

---

## 💻 Requirements & Running the Simulation

1. **Proteus VSM Design Suite**: Open `simulation/microprocessors_hardware_project.pdsprj`[cite: 2].
2. **Assembler**: Assemble `Microprocessors_hardwareproject_code.asm` using MASM/TASM to generate the `.BIN` or `.HEX` binary file[cite: 2].
3. Attach the compiled binary to the 8086 processor component inside Proteus and start the simulation[cite: 1].
