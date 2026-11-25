# 🚦 FSM Traffic Controller – Sensor-Based Smart Control

INTERN ID:CIS441

This project implements a **sensor-aware traffic light controller** using a **Finite State Machine (FSM)** in Verilog. The design dynamically reacts to real-time traffic on **4 roads (R1–R4)** and determines optimal green light allocation between **roads 1 & 3** and **roads 2 & 4** based on vehicle presence.

---

## 🧠 Project Overview

The FSM monitors a 4-bit `Traffic` input representing traffic density on four roads:
- `Traffic[0]` – Road 1  
- `Traffic[1]` – Road 2  
- `Traffic[2]` – Road 3  
- `Traffic[3]` – Road 4

The controller groups the roads as:
- **Group 1–3** (R1 and R3)
- **Group 2–4** (R2 and R4)

When traffic is detected in a group, it activates **Green** lights for that group. The other group is held at **Red**. Transitions between Green → Yellow → Red are time-controlled using an internal countdown.

---

## 🚦 Intersection Layout

A schematic representation of the 4-road intersection modeled in this project. Roads 1 & 3 and Roads 2 & 4 are grouped together for synchronized signaling.

![4-Road Intersection Diagram](https://github.com/VLSI-Shubh/Traffic-Controller-using-FSM/blob/38e5d1ad5d19e66bc9a33ec4f339ce91196e5ccc/images/Intersection.png)

---

## 🔷 FSM State Diagram

![FSM Diagram](https://github.com/VLSI-Shubh/Traffic-Controller-using-FSM/blob/32de1aec6563966cb09b189a13eb93384b69c19a/images/Traffic%20Controller%20FSM.png)

---

## 🔁 FSM Logic & Transitions

The traffic controller FSM operates in **five states**, reacting to sensor inputs and sequencing light phases using timers.

### ➤ State Overview

- `s_idle`: All lights Red. FSM waits for traffic input.
- `s_13gg`: Green for R1 & R3.
- `s_13yy`: Yellow for R1 & R3.
- `s_24gg`: Green for R2 & R4.
- `s_24yy`: Yellow for R2 & R4.

### ➤ Priority-Based Selection

- If traffic detected on R1 or R3 → FSM prioritizes R1 & R3.
- Else, if traffic on R2 or R4 → FSM switches to R2 & R4.
- **R1/R3 group has higher priority** by design when both are active.

### ➤ Timer-Based Phase Transitions

Each Green/Yellow state sets a timer:
```verilog
parameter GREEN_TIME  = 16'd55;
parameter YELLOW_TIME = 16'd10;
```
FSM uses a countdown mechanism with a `done` flag to trigger transitions.

### ➤ FSM Transition Summary

| Current State | Condition                  | Next State |
|---------------|----------------------------|------------|
| `s_idle`      | `Traffic[0] or Traffic[2]` | `s_13gg`   |
| `s_idle`      | `Traffic[1] or Traffic[3]` | `s_24gg`   |
| `s_13gg`      | `done == 1`                | `s_13yy`   |
| `s_13yy`      | `done == 1`                | `s_idle`   |
| `s_24gg`      | `done == 1`                | `s_24yy`   |
| `s_24yy`      | `done == 1`                | `s_idle`   |

> ✅ This FSM is **scalable** and **deterministic**, making it suitable for real-time traffic management in smart cities.  
> Future work includes support for **pedestrian signals**.

---

## 🎯 Output Encoding

| Signal        | Description              |
|---------------|--------------------------|
| `Red[3:0]`     | Red lights for R1–R4     |
| `Green[3:0]`   | Green lights for R1–R4   |
| `Yellow[3:0]`  | Yellow lights for R1–R4  |

Examples:
- In `s_13gg`: R1 & R3 = Green, R2 & R4 = Red  
- In `s_24yy`: R2 & R4 = Yellow, R1 & R3 = Red

---

## 🖥️ Simulation Output & Timing Behavior

Simulation was performed using `vvp` and GTKWave.

### 📷 Waveform Snapshot

![Waveform Output](https://github.com/VLSI-Shubh/Traffic-Controller-using-FSM/blob/d35985fc8b95f99b26d0dd68ed80e3b62dd28798/images/Output.png)

### 🔍 Log Observations

- **Initial (`s_idle`)**: No traffic detected; all Red.
- **Traffic on R1** → FSM enters `s_24gg` (Green for R2/R4)
- **Timer hits 0** → FSM switches to `s_24yy`, then back to idle.

### 📈 Sample Simulation Log

| Time (ns) | T (Traffic) | State           | R     | G     | Y     | Timer | Done |
|-----------|-------------|------------------|-------|-------|-------|--------|------|
| 25000     | 1000        | `s_idle → s_24gg` | 0101 | 1010 | 0000 | 55 → 0 | 0    |
| 595000    | 1000        | `s_24gg → s_24yy` | 0101 | 0000 | 1010 | 0      | 1    |
| 615000    | 1000        | `s_24yy → s_idle` | 1111 | 0000 | 0000 | —      | 0    |
| 625000    | 0100        | `s_idle → s_24gg` | 0101 | 1010 | 0000 | 55     | 0    |
| 1195000   | 1010        | `s_idle → s_13gg` | 1010 | 0101 | 0000 | 55     | 0    |

> ⚠️ FSM state IDs (for simulation):  
`0 = s_idle`, `1 = s_13gg`, `2 = s_13yy`, `3 = s_24gg`, `4 = s_24yy`

---

## 📘 FSM Design (Schematic View)

The schematic below provides a hardware-level view of the FSM. It was generated via **Vivado** and **EDA Playground** and highlights:

- State transition logic  
- Register/flip-flop placement  
- Combinational & sequential logic blocks

📄 [View Full PDF Schematic](https://github.com/VLSI-Shubh/Traffic-Controller-using-FSM/blob/561920e7239f6b68a78176154796e8364038c277/Circuits/schematic.pdf)

![FSM Schematic](https://github.com/VLSI-Shubh/Traffic-Controller-using-FSM/blob/561920e7239f6b68a78176154796e8364038c277/images/schematic.jpg)

---

## 💻 Project Files

| File                   | Description                              |
|------------------------|------------------------------------------|
| `traffic_controller.v` | Main FSM Verilog code                    |
| `traffic_controller_tb.v` | Testbench for verification           |
| `*.vcd`                | Waveform dumps for simulation            |
| `*.vvp`                | Compiled output (Icarus Verilog)         |
| `images/`              | FSM diagrams and waveform screenshots    |
| `circuits/`            | FSM schematic files (PDF/PNG)            |

---

## ✅ Conclusion

This project demonstrates a practical, sensor-aware traffic control FSM that dynamically adjusts signals to reduce idle time and congestion. It’s a solid example of **RTL design**, **FSM modeling**, and **real-time digital logic** applications.

---

## 📎 Future Enhancements

- Add pedestrian signal timing  
- Emergency vehicle priority override  
- Countdown timer display on signals  
- Extended simulation patterns (e.g., random traffic bursts)

---

## 🛠️ Tools Used

| Tool               | Purpose                                           |
|--------------------|---------------------------------------------------|
| **Icarus Verilog** | Compile/simulate Verilog code                    |
| **GTKWave**        | View simulation waveform dumps (`.vcd` files)    |
| **EDA Playground** | Online Verilog editor and schematic viewer       |
| **Vivado**         | RTL synthesis, schematic generation              |

---

## 📝 License

This project is licensed under the terms of the [MIT License](https://github.com/VLSI-Shubh/Traffic-Controller-using-FSM/blob/3406542ecb3136956d5a9926b9e3c724a3b2199b/License.txt).
