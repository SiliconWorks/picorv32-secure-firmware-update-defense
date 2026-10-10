🛡️ icoCrypt — Hardware-Accelerated Secure Firmware Update and Boot System
Using PicoRV32, AES-128 hardware, an ESP32-S3 Wi-Fi gateway, and a ZedBoard FPGA
![Language](https://img.shields.io/badge/Language-Verilog%20%7C%20C%20%7C%20JavaScript-blue)
![Processor](https://img.shields.io/badge/Processor-PicoRV32-orange)
![Architecture](https://img.shields.io/badge/Architecture-RISC--V-purple)
![Cryptography](https://img.shields.io/badge/Cryptography-AES--128-red)
![Gateway](https://img.shields.io/badge/Gateway-ESP32--S3-green)
![Platform](https://img.shields.io/badge/Platform-FPGA%20%7C%20Embedded-lightgrey)
![Status](https://img.shields.io/badge/Status-Under%20Development-yellow)
> **Project status:** This repository describes a research and prototyping project. The end-to-end update flow is still being integrated and verified. A listed feature should not be considered operational until its corresponding test passes.
---
📌 Overview
icoCrypt is a web-based firmware deployment dashboard for a prototype secure firmware-update system. The intended system combines a browser dashboard, an ESP32-S3 Wi-Fi gateway, and a ZedBoard FPGA running a PicoRV32-based SoC with AES-128 cryptographic hardware and boot-control logic.
The dashboard is intended to let a user select a raw firmware binary (`.bin`), calculate its SHA-256 digest, encrypt the payload in the browser using AES-128 (CBC or CTR, depending on the configured implementation), and upload the encrypted package over the local Wi-Fi network to the ESP32-S3.
The ESP32-S3 is intended to:
Join the same local Wi-Fi network as the host laptop using a WiFiManager configuration portal.
Receive the encrypted firmware package and metadata through an HTTP upload endpoint.
Store the received file in its LittleFS filesystem.
Transfer the stored package to the ZedBoard over UART using XMODEM-CRC.
On the ZedBoard, the PicoRV32 SoC is intended to receive and stage the image, control the AES-128 hardware through memory-mapped I/O (MMIO), run the implemented verification procedure, and allow firmware execution only if the required validation checks succeed.
This is a hardware–software co-design project involving FPGA RTL, embedded C, ESP32 firmware, browser-side cryptography, peripheral integration, and simulation.
🎯 Project Objectives
Integrate the PicoRV32 RISC-V processor into a custom FPGA-based SoC.
Provide a browser-based firmware deployment interface.
Transfer the encrypted firmware package from the laptop to an ESP32-S3 over local Wi-Fi and HTTP.
Store the received package in ESP32-S3 LittleFS.
Transfer the package from the ESP32-S3 to the ZedBoard over UART using XMODEM-CRC.
Receive and buffer serial data using UART and FIFO logic.
Connect the CPU to peripherals through an MMIO interconnect.
Integrate AES-128 cryptographic hardware.
Coordinate firmware staging, verification, and boot decisions.
Verify each subsystem independently before claiming end-to-end operation.
🏗️ Complete System Architecture
```text
┌─────────────────────────────────────┐
│         HOST LAPTOP / PC            │
│            icoCrypt UI              │
│                                     │
│  Select .bin firmware               │
│  Calculate SHA-256 digest            │
│  Encrypt payload using AES-128       │
│  Upload encrypted package via HTTP   │
└──────────────────┬──────────────────┘
                   │ Local Wi-Fi
                   │ HTTP upload
                   ▼
┌─────────────────────────────────────┐
│          ESP32-S3 GATEWAY            │
│                                     │
│  WiFiManager configuration portal    │
│  HTTP upload server                  │
│  LittleFS package storage            │
│  XMODEM-CRC UART sender              │
└──────────────────┬──────────────────┘
                   │ UART TX/RX + GND
                   │ XMODEM-CRC
                   ▼
┌────────────────────────────────────────────────────────────┐
│                     ZEDBOARD FPGA                           │
│                                                            │
│  ┌────────────────────┐      ┌─────────────────────────┐  │
│  │ UART RX / TX        │─────▶│ RX FIFO / buffering     │  │
│  └────────────────────┘      └────────────┬────────────┘  │
│                                           │               │
│                                   ┌───────▼────────┐      │
│                                   │ PicoRV32 CPU   │      │
│                                   │ RISC-V core    │      │
│                                   └───────┬────────┘      │
│                                           │               │
│                                   ┌───────▼────────┐      │
│                                   │ MMIO interconnect│     │
│                                   │ / address decode│     │
│                                   └───┬────┬────┬───┘     │
│                                       │    │    │         │
│                         ┌─────────────┘    │    └──────┐  │
│                         ▼                  ▼           ▼  │
│                  ┌────────────┐   ┌─────────────┐ ┌──────┐│
│                  │ AES-128    │   │ Firmware    │ │Secure││
│                  │ accelerator│   │ staging /   │ │boot  ││
│                  │            │   │ program RAM │ │ FSM  ││
│                  └────────────┘   └─────────────┘ └──────┘│
│                                                            │
│              Clock, reset, status and control               │
└────────────────────────────────────────────────────────────┘
```
This is the intended architecture. Confirm the actual signal paths, address map, memory behavior, and implemented verification logic against the current source code.
🔄 End-to-End Firmware Deployment Workflow
Select firmware: The user selects a firmware binary (`.bin`) in icoCrypt.
Prepare the package: The dashboard calculates the SHA-256 digest and encrypts the payload using the configured AES-128 mode.
Discover/connect to gateway: The ESP32-S3 joins the laptop's local Wi-Fi network. Device discovery must be supported by the actual dashboard and ESP32 implementation.
HTTP upload: The dashboard uploads the encrypted payload and required metadata to the ESP32-S3.
Store in LittleFS: The ESP32-S3 writes the received package to its flash filesystem.
Start UART transfer: The ESP32-S3 reads the stored file and sends it to the ZedBoard using XMODEM-CRC.
Receive and stage: The FPGA UART receives the serial data; the SoC stages the image in the configured memory region.
Decrypt: The PicoRV32-controlled AES hardware performs the configured cryptographic operation.
Verify: The system checks the image using the implemented integrity and authenticity mechanism.
Accept or reject: The boot controller makes a decision based on the actual verification result and update policy.
Boot accepted firmware: Execution is permitted only after the required checks pass.
The full flow is a development objective until it has been demonstrated end to end on the actual software and hardware.
🧩 Main System Components
1. icoCrypt Web Dashboard
The browser dashboard is the user's entry point for firmware deployment.
Intended responsibilities:
Select a firmware `.bin` file.
Calculate and display the SHA-256 digest.
Encrypt the firmware in the browser using the configured AES-128 mode (CBC or CTR).
Discover or connect to the ESP32-S3 gateway on the local network.
Upload the encrypted payload and metadata through HTTP.
Display transfer progress and result if implemented.
Important: A SHA-256 digest detects changes only when compared with a trusted expected digest. If an attacker can replace both the firmware and its digest, the digest alone does not authenticate the firmware.
2. ESP32-S3 Wi-Fi Gateway
The ESP32-S3 bridges the Wi-Fi dashboard and the FPGA's serial interface.
Function	Intended responsibility
WiFiManager	Configure the ESP32-S3 to join the local Wi-Fi network
HTTP upload server	Receive the encrypted firmware package and metadata
LittleFS	Store the received package in ESP32-S3 flash filesystem
UART sender	Read the stored package and transmit it to the FPGA
XMODEM-CRC	Provide block-based transfer and CRC error detection
Status handling	Report upload/transfer outcome if implemented
LittleFS storage on the ESP32-S3 is gateway staging storage. It is not the same as the FPGA's firmware memory.
3. UART and RX FIFO
UART provides serial communication between the ESP32-S3 and the ZedBoard. The exact UART port, pins, baud rate, and handshake behavior must match both implementations.
Relevant FPGA modules may include:
`uart_rx.v` — UART receive logic.
`uart_tx.v` — UART transmit logic.
`baud_gen.v` — baud-rate timing generation.
`rx_fifo.v` — receive FIFO.
SoC-level UART integration logic.
The RX FIFO temporarily stores received bytes until the processor or associated logic consumes them.
Typical wiring:
ESP32-S3 TX → ZedBoard UART RX
ESP32-S3 RX ← ZedBoard UART TX (needed for bidirectional protocol acknowledgements)
ESP32-S3 GND ↔ ZedBoard GND
Check the actual board pin mapping and voltage compatibility before connecting hardware. Do not connect pins based only on a generic diagram.
4. PicoRV32 RISC-V Processor
PicoRV32 executes the embedded firmware and coordinates peripheral operations. The project integrates the existing open-source core rather than designing a CPU from scratch.
Responsibilities may include:
Reading UART/FIFO data.
Controlling peripherals through MMIO.
Starting AES operations and checking status.
Coordinating update and boot-control logic.
Reference: YosysHQ PicoRV32
5. MMIO Interconnect
Memory-mapped I/O (MMIO) assigns address ranges to peripheral registers. The CPU accesses these registers using normal load/store instructions.
The RTL and C drivers must agree on:
Base addresses and register offsets.
Access widths and byte enables.
Start, busy, done, error, and status bit definitions.
CPU bus ready/wait behavior.
A wrong address or mismatched register definition can cause the CPU to wait indefinitely or access the wrong peripheral.
6. AES-128 Hardware Accelerator
AES-128 is a symmetric-key block cipher that uses a 128-bit key and 128-bit data block. The RTL accelerator is intended to perform the configured cryptographic operation under CPU control.
A typical CPU interface contains key/data registers, a start control, status flags, and output registers, depending on the actual implementation.
Security distinction:
AES-CBC/CTR provide confidentiality when used correctly, but do not authenticate a firmware image on their own.
XMODEM-CRC detects transmission errors; it does not prove who sent the image.
SHA-256 alone is not proof of origin.
Firmware authenticity requires a verified MAC (for example, AES-CMAC or HMAC with a protected key) or a digital signature checked against a trusted public key.
Only describe AES-CMAC, signatures, or other authentication features as implemented if the corresponding RTL/software and tests actually support them.
7. Firmware Memory
The FPGA design may use RAM/BRAM for program instructions, working data, or a staging image. The actual memory map determines how the incoming image is stored and whether it can be executed.
ESP32-S3 LittleFS: stores the uploaded package at the gateway.
FPGA BRAM/RAM: may hold processor firmware, working data, or a staging image.
External nonvolatile flash: is required if the design needs persistent firmware storage across FPGA power cycles and has an appropriate interface.
A simulation `.hex` file is an initialization input; it is not automatically persistent storage on the FPGA.
8. Secure Boot Controller
The secure boot controller coordinates update/boot state transitions. The intended state sequence may include idle, receive, verify, accept, and reject, depending on the implemented FSM.
Expected responsibilities:
Track update and boot state.
Coordinate validation.
Report status.
Prevent execution when required validation fails.
Secure boot is only meaningful if the execution decision is actually gated by trustworthy verification results.
💻 Embedded Software and Drivers
The embedded C software is intended to run on PicoRV32 and access hardware via MMIO.
Software block	Responsibility
UART driver	Transmit/receive serial data using the implemented UART registers
AES driver	Configure and control the AES accelerator
GPIO driver	Access GPIO only if its registers are implemented
Memory driver	Access the actual firmware memory interface
Update manager	Coordinate update stages, errors, and status
Bootloader	Perform boot-time checks and choose an accepted image
Driver interfaces must match the actual RTL register map. File existence alone does not demonstrate correct operation.
Illustrative MMIO Example
```c
#include <stdint.h>

#define PERIPHERAL_REG (*(volatile uint32_t *)0x10000000u)

void peripheral_write(uint32_t value)
{
    PERIPHERAL_REG = value;
}

uint32_t peripheral_read(void)
{
    return PERIPHERAL_REG;
}
```
This is an example of MMIO access only. `0x10000000` is not a universal address; confirm the address from the integrated RTL before using it in firmware.
🧪 Simulation and Verification
The project uses RTL simulation to debug the hardware and CPU/peripheral integration. A full-SoC testbench and Tcl flow have been prepared for ModelSim/QuestaSim; actual pass/fail status must be based on the current simulation logs.
Suggested Verification Order
Compile and elaborate: Confirm all source files and module dependencies compile.
Clock/reset: Confirm the CPU and peripherals leave reset correctly.
CPU/MMIO: Confirm the CPU can access memory and peripheral registers.
UART/FIFO: Verify serial receive, transmit, FIFO flags, and byte ordering.
AES: Run a known-answer test and compare the result with the expected ciphertext.
Boot controller: Check state transitions and rejection behavior.
ESP32-S3 gateway: Verify Wi-Fi setup, HTTP upload, LittleFS write/read, and UART transfer separately.
End-to-end update: Demonstrate the entire dashboard-to-FPGA path.
Negative tests: Test corrupted data, incorrect authentication, incomplete transfer, and reset during update.
AES-128 Known-Answer Test
Use this standard AES-128 encryption vector when checking a compatible AES implementation:
Key: `000102030405060708090a0b0c0d0e0f`
Plaintext: `00112233445566778899aabbccddeeff`
Expected ciphertext: `69c4e0d86a7b0430d8cdb78070b4c55a`
Confirm byte order and register packing in the actual accelerator before comparing results.
Simulation Environment
Technology	Purpose
Verilog HDL	RTL hardware design
PicoRV32	RISC-V processor core
ModelSim / QuestaSim	RTL simulation
Tcl	Simulation automation
C	Embedded firmware and drivers
FPGA development tools	Synthesis, implementation, and hardware validation
Running the Simulation
The repository may contain a Tcl script named `run_secure_soc_full.tcl`. From the simulator's working directory, run:
```tcl
do run_secure_soc_full.tcl
```
Before running, confirm:
All RTL dependencies are included in the compile order.
The firmware image exists at the configured path.
The testbench references the correct top-level module.
Paths are valid on the current machine.
The selected simulator supports the constructs used by the design.
Prefer repository-relative paths where practical so the simulation can run on another computer.
📁 Suggested Project Structure
The structure below is illustrative. Keep filenames and folders aligned with the actual repository.
```text
picorv32-secure-firmware-update-defense/
├── dashboard/                  # icoCrypt web dashboard, if stored here
├── esp32_gateway/              # ESP32-S3 firmware, if stored here
│   ├── src/
│   └── platformio.ini          # or the project's actual build configuration
├── rtl/
│   ├── uart_rx.v
│   ├── uart_tx.v
│   ├── baud_gen.v
│   ├── rx_fifo.v
│   ├── firmware_mem.v
│   ├── secure_boot_ctrl.v
│   └── secure_boot_mmio.v
├── soc/
│   └── secure_soc_top.v
├── firmware/
│   ├── drivers/
│   ├── update_manager.c        # if implemented
│   ├── bootloader.c            # if implemented
│   └── main.c
├── firmware_aes/
│   └── system_firmware.hex
├── sim/
│   ├── tb_secure_soc.v
│   └── run_secure_soc_full.tcl
├── docs/
│   ├── architecture.md
│   └── verification.md
└── README.md
```
Do not treat the suggested tree as evidence that every listed file or feature already exists.
🔐 Security Considerations
Confidentiality: Use AES-128 with a correctly configured mode and protected key handling.
Integrity: Detect changes in the firmware data.
Authenticity: Verify that an update was produced by an authorized party.
Anti-rollback: Prevent unauthorized downgrade if version protection is implemented.
Secure boot: Gate execution on successful verification.
Recovery: Define safe behavior after a rejected or interrupted update.
Key protection: Protect cryptographic keys against unauthorized access.
Transport: Use an appropriate network configuration; local Wi-Fi/HTTP alone should not be assumed secure against all attackers.
These are design requirements, not a claim that every protection is already implemented. Defense-grade deployment would require a defined threat model, suitable authentication, protected key storage, fault testing, and independent security evaluation.
🚧 Development Status
The project is under active development. The following list distinguishes development artifacts from verified functionality.
Development Artifacts Reported
[x] PicoRV32 selected for processor integration.
[x] UART receiver/transmitter RTL modules developed.
[x] Baud-rate generation and receive FIFO modules developed.
[x] Firmware memory and secure boot controller RTL developed.
[x] SoC top-level integration and MMIO interface developed.
[x] Full-SoC testbench and Tcl simulation flow prepared.
[x] Embedded driver source/header development initiated.
[ ] ESP32-S3 Wi-Fi, HTTP, LittleFS, and XMODEM-CRC path verified end to end.
[ ] Complete dashboard-to-FPGA update flow demonstrated.
Remaining Verification Tasks
[ ] Confirm all RTL dependencies compile and elaborate.
[ ] Verify UART transmission and reception.
[ ] Validate AES-128 against a known-answer test.
[ ] Verify CPU-to-peripheral MMIO transactions.
[ ] Complete and validate update manager and bootloader.
[ ] Verify firmware integrity and authenticity checks.
[ ] Demonstrate the complete host-to-FPGA update flow.
[ ] Test corrupted and unauthorized firmware rejection.
[ ] Verify reset behavior and recovery after update failure.
[ ] Complete FPGA implementation and hardware demonstration.
Mark a task complete only after its corresponding test passes. The presence of RTL, C source, a dashboard, or a testbench is not by itself proof that the system works end to end.
🔮 Future Enhancements
HMAC or digital-signature-based firmware authentication.
A/B firmware slots and safe rollback.
External SPI flash integration.
Anti-rollback protection with trusted version metadata.
Watchdog-based recovery.
Fault injection for corrupted or interrupted updates.
FPGA resource-utilization and timing reports.
Hardware-versus-software cryptographic performance benchmarking.
Stronger gateway authentication and protected key provisioning.
🛠️ Technologies Used
Verilog HDL
PicoRV32 RISC-V processor
AES-128 hardware acceleration
ESP32-S3
WiFiManager
HTTP upload
LittleFS
UART and XMODEM-CRC
Memory-mapped I/O (MMIO)
Embedded C
ModelSim / QuestaSim
Tcl simulation automation
FPGA development tools
Browser-side JavaScript cryptography
👥 Project Contributions
Hardware / FPGA
PicoRV32 SoC integration.
UART and FIFO RTL.
MMIO and peripheral integration.
AES hardware and secure boot-control integration.
Testbench development and simulation debugging.
Embedded / Gateway
ESP32-S3 Wi-Fi setup using WiFiManager.
HTTP firmware upload endpoint.
LittleFS package storage.
UART/XMODEM-CRC transfer to the ZedBoard.
Status and error handling.
Dashboard / Application
Firmware selection and metadata handling.
SHA-256 digest calculation.
Browser-side AES-128 encryption.
ESP32-S3 discovery/connection and HTTP upload UI.
The final contribution breakdown should reflect the actual work completed by each team member.
📄 Licensing and Acknowledgements
The PicoRV32 core and third-party components retain their respective licensing terms. Select a project-level license only after checking compatibility with all dependencies.
PicoRV32 by YosysHQ
The open-source FPGA, embedded-systems, and hardware-verification communities.
---
Project focus: Research and prototyping of an FPGA-based firmware-update path that combines browser-side package preparation, an ESP32-S3 Wi-Fi gateway, UART transfer, PicoRV32 control, AES-128 hardware, and boot-validation logic for defense-oriented embedded-system research.
