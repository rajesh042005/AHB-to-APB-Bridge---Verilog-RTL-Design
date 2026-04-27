<h1 align="center"> AHB to APB Bridge - Verilog RTL Design </h1>

<p align="center">
<img src="https://img.shields.io/badge/Protocol-AMBA%20AHB%20to%20APB-blue?style=for-the-badge"/>
<img src="https://img.shields.io/badge/Design-RTL-orange?style=for-the-badge"/>
<img src="https://img.shields.io/badge/Language-Verilog-green?style=for-the-badge"/>
<img src="https://img.shields.io/badge/Stage-Bridge%20Design-purple?style=for-the-badge"/>
</p>

<p align="center">
<img src="https://img.shields.io/badge/Status-Completed-success?style=flat-square"/>
<img src="https://img.shields.io/badge/Type-Protocol%20Converter-blue?style=flat-square"/>
<img src="https://img.shields.io/badge/Role-AHB↔APB%20Interface-informational?style=flat-square"/>
</p>

---

<p align="center">
Implementation of an <b>AHB to APB Bridge</b> that converts pipelined AHB transactions into sequential APB transfers using FSM-based control.
</p>

---

# Bridge Architecture

```mermaid
flowchart LR
    AHB[AHB BUS] --> BRIDGE[AHB-APB Bridge]
    BRIDGE --> APB[APB BUS]
    APB --> UART
    APB --> SPI
    APB --> I2C
    APB --> USB
```

- AHB provides high-speed pipelined transactions  
- Bridge converts protocol and timing  
- APB distributes to low-speed peripherals  
- Ensures only one APB transfer at a time  

---

# Transfer Flow

```mermaid
flowchart LR
    AHB_ADDR[AHB Address Phase] --> LATCH[Latch Signals]
    LATCH --> SETUP[APB Setup Phase]
    SETUP --> ENABLE[APB Enable Phase]
    ENABLE --> COMPLETE[Transfer Complete]
```

- Address and control captured from AHB  
- Signals stored in internal registers (pipeline break)  
- APB SETUP phase asserts PSEL  
- ENABLE phase asserts PENABLE and completes transfer  

---

# FSM Design

```mermaid
flowchart LR
    IDLE --> SETUP
    SETUP --> ENABLE
    ENABLE -->|PREADY=1| IDLE
    ENABLE -->|PREADY=0| ENABLE
```

- IDLE waits for valid AHB transfer  
- SETUP drives APB control signals  
- ENABLE performs actual data transfer  
- PREADY controls completion and wait states  

---

# Data Path

```mermaid
flowchart LR
    HWDATA --> LATCH
    LATCH --> PWDATA
    PRDATA --> HRDATA
```

- Write data flows from AHB → APB  
- Read data flows from APB → AHB  
- Internal registers isolate pipeline timing  

---

# Data Flow Example

```mermaid
flowchart LR
    CPU --> AHB
    AHB --> BRIDGE
    BRIDGE --> APB
    APB --> UART
    UART --> DONE[Data Stored]
```

- CPU initiates transaction  
- AHB carries high-speed request  
- Bridge converts protocol  
- APB executes peripheral access  

---

# FSM State Behavior

## IDLE
- Wait for valid transfer (HTRANS = NONSEQ/SEQ)  
- Latch address, write, size  
- Assert HREADY low to stall AHB  


## SETUP
- Drive PADDR, PWRITE, PWDATA  
- Assert PSEL  
- Prepare APB transfer  


## ENABLE
- Assert PENABLE  
- Wait for PREADY  
- Complete transfer and return response  

---

# Transfer Types

## Write Transfer

<img width="473" height="456" alt="image" src="https://github.com/user-attachments/assets/c065eb4c-1168-4741-8f14-8d3728b638db" />

- HWRITE = 1 → PWRITE = 1  
- Data flows to peripheral  

## Read Transfer

<img width="473" height="456" alt="image" src="https://github.com/user-attachments/assets/85a7d129-2068-41a5-b53f-d6e4381317be" />

- HWRITE = 0 → PWRITE = 0  
- Data returned to AHB  

## Wait State Transfer

- PREADY = 0  
- Bridge holds HREADY = 0  
- Stalls AHB pipeline  

## Error Transfer

- PSLVERR = 1  
- HRESP = ERROR  

---

# Key Signals

## AHB Side
- HADDR, HWRITE, HTRANS  
- HWDATA → Write data  
- HRDATA ← Read data  
- HREADY → Handshake  
- HRESP → Status  

## APB Side
- PADDR, PWRITE  
- PSEL, PENABLE  
- PWDATA / PRDATA  
- PREADY, PSLVERR  

---

# Key Design Concepts

## Pipeline Break
- AHB is pipelined  
- APB is sequential  
- Registers isolate timing  

## Handshake Conversion
- HREADY ↔ PREADY  
- HRESP ↔ PSLVERR  

## Wait Handling
- APB delay → stalls AHB  
- Maintains protocol correctness  

---

<p align="center"><b>
The AHB-APB Bridge ensures correct protocol translation between high-speed and low-speed domains, enabling reliable and scalable SoC integration.
</p>
  
---
