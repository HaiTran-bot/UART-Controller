# UART Controller on Arty Z7-20

![Board](https://img.shields.io/badge/Board-Digilent_Arty_Z7--20-red)
![Interface](https://img.shields.io/badge/Interface-UART_%7C_AXI--Stream_Like-blue)
![Reliability](https://img.shields.io/badge/Reliability-16x_Oversampling_%2B_Voting-green)

## 📌 Overview

This project implements a robust, full-duplex **Universal Asynchronous Receiver-Transmitter (UART)** controller tailored for the **Digilent Arty Z7-20** FPGA. 

Unlike basic academic implementations, this core is designed for industrial reliability. It features **16x Oversampling** with **Majority Voting logic** for noise immunity, **Double Buffering** to prevent data loss, and a **Valid/Ready Handshake interface** compatible with standard FIFO or CPU interconnects.

---

## 🚀 Key Features

* **Configurable Parameters:** Supports arbitrary Clock Frequencies (Default: 125 MHz) and Baud Rates (Default: 115200).
* **Robust Receiver (RX):**
    * **16x Oversampling:** Samples each bit 16 times per baud period to synchronize perfectly.
    * **Majority Voting:** Logic samples the data line 3 times around the center point (ticks 7, 8, 9) and votes to determine the bit value. This filters out glitches and noise spikes.
    * **Start Bit Verification:** Re-checks the start bit at the middle of the cycle to prevent false starts.
* **Flow Control & Buffering:**
    * **Double Buffering:** RX module holds the current byte and a buffered byte, giving the CPU more time to read data before an Overrun occurs.
    * **Handshake Interface:** Uses `valid`, `ready`, and `ack` signals (similar to AXI-Stream) to ensure no data is dropped during transmission or reception.
* **Error Detection:**
    * **Frame Error:** Detects if the Stop Bit is missing (line not pulled high).
    * **Overrun Error:** Detects if the host failed to read data before a new byte arrived.

---

## 🏗️ Architecture Design

### 1. UART Transmitter (`uart_tx.v`)
A Finite State Machine (FSM) driven module that serializes 8-bit parallel data.
* **States:** `IDLE` -> `START` -> `DATA` (Shift LSB first) -> `STOP`.
* **Flow Control:** Asserts `tx_ready` when idle. Accepts data only when `tx_valid` is high.

### 2. UART Receiver (`uart_rx.v`)
A sophisticated receiver designed for noisy environments.
* **Mechanism:** 1. Detects falling edge of Start Bit.
    2. Starts a `tick_counter` (Baud * 16).
    3. At the center of each bit (ticks 7, 8, 9), it accumulates the signal value.
    4. **Voting:** `bit_value = (sum >= 2) ? 1 : 0`.
* **Output:** Provides `rx_data` along with `rx_ready`. Clears data only when `rx_ack` is received from the master.

---

## ⏱️ Simulation Results

The design includes a self-checking testbench (`tb_uart.v`) covering single-byte transfers, random patterns, and back-to-back stress tests.

### 1. Waveform Analysis
The waveform below shows the transmission of `0x55` and `0xA3`. Note the **Handshake signals** (`tx_valid`/`tx_ready`) managing the flow.

### 2. Testbench Log
The simulation performs automated checking of sent vs. received data.


---
