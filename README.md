# 🛡️ icoCrypt — Hardware-Accelerated Secure Firmware Update and Boot System

**Using PicoRV32, AES-128 Hardware, ESP32-S3 Wi-Fi Gateway, and ZedBoard FPGA**

![Language](https://img.shields.io/badge/Language-Verilog%20%7C%20C%20%7C%20JavaScript-blue)
![Processor](https://img.shields.io/badge/Processor-PicoRV32-orange)
![Architecture](https://img.shields.io/badge/Architecture-RISC--V-purple)
![Cryptography](https://img.shields.io/badge/Cryptography-AES--128-red)
![Gateway](https://img.shields.io/badge/Gateway-ESP32--S3-green)
![Platform](https://img.shields.io/badge/Platform-FPGA%20%7C%20Embedded-lightgrey)
![Status](https://img.shields.io/badge/Status-Under%20Development-yellow)

> **Project status:** This project is under active development. The complete dashboard-to-FPGA update flow is still being integrated and verified. Features described as intended functionality should not be considered operational until the corresponding tests pass.

---

## 📌 Overview

**icoCrypt** is a web-based secure firmware deployment dashboard designed for an FPGA-based embedded system. The proposed architecture combines a browser dashboard, an ESP32-S3 Wi-Fi gateway, and a ZedBoard FPGA running a PicoRV32-based SoC with AES-128 cryptographic hardware and secure boot control logic.

The dashboard allows the user to select a raw firmware binary (`.bin`), calculate its SHA-256 digest, encrypt the payload in the browser using AES-128 (CBC or CTR, depending on the configured implementation), and upload the encrypted package over the local Wi-Fi network to the ESP32-S3.

The ESP32-S3 gateway is intended to:

- Connect to the same Wi-Fi network as the host laptop using a WiFiManager configuration portal.
- Receive the encrypted firmware package and metadata through an HTTP upload endpoint.
- Store the received package in its LittleFS filesystem.
- Transfer the stored package to the ZedBoard FPGA over UART using XMODEM-CRC.

The FPGA-based SoC is intended to receive the firmware, stage the image in memory, use the AES-128 hardware accelerator for configured cryptographic operations, verify the firmware using the implemented security mechanism, and permit execution only after the required validation checks succeed.

This project follows a hardware-software co-design approach involving FPGA RTL development, embedded C, ESP32 firmware, browser-side cryptography, peripheral integration, and simulation.

## 🎯 Project Objectives

- Integrate the PicoRV32 RISC-V processor into a custom FPGA-based SoC.
- Develop a browser-based firmware deployment dashboard.
- Transfer encrypted firmware from the laptop to the ESP32-S3 over local Wi-Fi.
- Receive firmware packages through an HTTP upload server.
- Store uploaded packages in ESP32-S3 LittleFS.
- Transfer firmware from the ESP32-S3 to the ZedBoard using UART and XMODEM-CRC.
- Develop UART communication and receive-buffering logic.
- Integrate AES-128 hardware acceleration.
- Connect peripherals through memory-mapped I/O (MMIO).
- Develop embedded C drivers for peripheral access.
- Implement firmware update management and secure boot control.
- Verify the system incrementally through simulation and hardware testing.

## 🏗️ Complete System Architecture

```text
                 HOST LAPTOP / PC
              ┌──────────────────────┐
              │  icoCrypt Dashboard  │
              │                      │
              │ Select firmware .bin │
              │ Calculate SHA-256    │
              │ AES-128 encryption   │
              │ HTTP upload          │
              └──────────┬───────────┘
                         │
                   Same Wi-Fi Network
                         │
                    HTTP Upload
                         │
                         ▼
              ┌──────────────────────┐
              │    ESP32-S3 Gateway  │
              │                      │
              │ WiFiManager          │
              │ HTTP Upload Server   │
              │ LittleFS Storage     │
              │ XMODEM-CRC UART TX   │
              └──────────┬───────────┘
                         │
                  UART TX / RX / GND
                   XMODEM-CRC Transfer
                         │
                         ▼
┌──────────────────────────────────────────────────────────────┐
│                       ZEDBOARD FPGA                          │
│                                                              │
│  ┌──────────────────┐        ┌───────────────────────────┐  │
│  │ UART RX / TX     │───────▶│ RX FIFO / Buffering       │  │
│  └──────────────────┘        └─────────────┬─────────────┘  │
│                                            │                │
│                                    ┌───────▼────────┐       │
│                                    │ PicoRV32 CPU   │       │
│                                    │ RISC-V Core    │       │
│                                    └───────┬────────┘       │
│                                            │                │
│                                    ┌───────▼────────┐       │
│                                    │ MMIO           │       │
│                                    │ Interconnect   │       │
│                                    └───┬────┬────┬───┘      │
│                                        │    │    │          │
│                       ┌────────────────┘    │    └──────┐   │
│                       ▼                     ▼           ▼   │
│                ┌────────────┐       ┌────────────┐ ┌────────┐
│                │ AES-128    │       │ Firmware   │ │ Secure │
│                │ Accelerator│       │ Memory     │ │ Boot   │
│                └────────────┘       └────────────┘ │ FSM    │
│                                                    └────────┘
│                                                              │
│               Clock, Reset, Status and Control               │
└──────────────────────────────────────────────────────────────┘
```

*This is the intended architecture. Actual signal connections, address maps, memory behavior, and security functions must be confirmed against the current RTL and embedded firmware.*

## 🔄 End-to-End Firmware Deployment Workflow

1. **Select firmware:** The user selects a firmware binary (`.bin`) in the icoCrypt dashboard.
2. **Prepare the package:** The dashboard calculates the SHA-256 digest and encrypts the payload using the configured AES-128 mode.
3. **Connect to Wi-Fi:** The ESP32-S3 joins the same local Wi-Fi network as the laptop using WiFiManager.
4. **Discover the gateway:** The dashboard discovers or connects to the ESP32-S3 using the mechanism implemented in the application.
5. **Upload firmware:** The encrypted payload and required metadata are uploaded to the ESP32-S3 through HTTP.
6. **Store the package:** The ESP32-S3 stores the received package in LittleFS.
7. **Transfer over UART:** The ESP32-S3 reads the stored file and transfers it to the ZedBoard using XMODEM-CRC.
8. **Receive the image:** The FPGA UART receives the serial data, and the SoC stages the image in the configured memory region.
9. **Decrypt:** The PicoRV32-controlled AES hardware performs the configured cryptographic operation.
10. **Verify:** The system validates the firmware using the implemented integrity and authenticity mechanism.
11. **Accept or reject:** The boot controller makes a decision based on the verification result and applicable update policy.
12. **Boot accepted firmware:** Execution is permitted only when the required validation checks succeed.

The complete workflow remains a development objective until it has been demonstrated successfully end to end.

## 🧩 Hardware and System Components

### 1. PicoRV32 RISC-V Processor

PicoRV32 is the processor core used to execute embedded firmware and control the SoC peripherals.

The project integrates the existing open-source processor into a custom system rather than designing a CPU from scratch.

Responsibilities include:

- Executing embedded firmware.
- Controlling peripherals through MMIO.
- Managing firmware update operations.
- Coordinating cryptographic processing and boot-control operations.

Reference: https://github.com/YosysHQ/picorv32

### 2. UART Communication Interface

UART (Universal Asynchronous Receiver/Transmitter) provides serial communication between the ESP32-S3 gateway and the FPGA.

The UART subsystem may include:

- `uart_rx.v` — UART receive logic.
- `uart_tx.v` — UART transmit logic.
- `baud_gen.v` — Baud-rate timing generation.
- `rx_fifo.v` — Receive FIFO.
- SoC-level UART integration logic.

The UART configuration, pin assignments, baud rate, and handshake behavior must match on both devices.

### 3. Receive FIFO

The RX FIFO temporarily stores incoming bytes until the processor or associated logic consumes them.

Its responsibilities include:

- Buffering incoming UART data.
- Managing read and write operations.
- Tracking FIFO occupancy.
- Reporting empty and full conditions.
- Helping bridge differences between serial data reception and processor data consumption.

### 4. AES-128 Hardware Accelerator

AES-128 (Advanced Encryption Standard with a 128-bit key) is the cryptographic hardware component of the project.

The accelerator is intended to offload configured cryptographic operations from software to dedicated RTL hardware.

A processor-facing interface may include:

- Key registers.
- Input-data registers.
- Control and start registers.
- Busy/done status registers.
- Output-data registers.

The exact register map must match the implemented RTL and embedded C driver.

**Security note:** AES encryption alone does not authenticate firmware. The system must also verify that the image originates from an authorized source and has not been modified.

### 5. Firmware Memory

The FPGA firmware-memory subsystem is intended to store firmware data used during update and boot operations.

Different storage locations serve different purposes:

- **ESP32-S3 LittleFS:** Stores the uploaded package in the gateway's flash filesystem.
- **FPGA BRAM/RAM:** May hold processor instructions, working data, or a staged firmware image.
- **External nonvolatile flash:** Can retain firmware across power cycles when the appropriate hardware interface and control logic are implemented.

A simulation `.hex` file is an initialization input; it is not automatically persistent firmware storage on the FPGA.

### 6. Secure Boot Controller

The secure boot controller coordinates boot-related state transitions and update decisions.

Intended responsibilities include:

- Managing boot and update states.
- Coordinating firmware validation.
- Reporting controller status.
- Controlling whether execution may proceed.
- Rejecting firmware when required validation fails.

Secure boot is only effective when the boot decision depends on trustworthy verification results and the system protects its trust anchors.

### 7. Memory-Mapped I/O (MMIO)

MMIO allows the processor to access peripheral registers through defined memory addresses.

The SoC interconnect connects the PicoRV32 processor with the required memory and peripheral interfaces.

The embedded drivers and RTL must agree on:

- Peripheral base addresses and register offsets.
- Access widths and byte enables.
- Start, busy, done, error, and status bits.
- CPU bus completion and wait behavior.

A mismatch between a driver's register address and the RTL address decoder can prevent the intended peripheral from responding.

## 💻 Embedded Software

The embedded software is written in C and is intended to execute on the PicoRV32 processor.

### Driver Responsibilities

| Component | Responsibility |
|---|---|
| UART driver | Initialize UART and transmit or receive data |
| GPIO driver | Access implemented GPIO registers |
| AES driver | Configure and control the AES hardware |
| Memory driver | Access the implemented firmware-memory interface |
| Update manager | Coordinate firmware-update states |
| Bootloader | Perform boot-time checks and select firmware for execution |

Driver interfaces must match the actual hardware register map.

### Illustrative MMIO Example

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

This is an illustrative example only. The address shown is not a universal address for the project's peripherals. Confirm the actual register address in the integrated RTL before using it in firmware.

## 🧪 Simulation and Verification

The project uses RTL simulation to evaluate hardware behavior and system integration.

A full-SoC testbench and Tcl simulation flow have been prepared for ModelSim/QuestaSim. Actual pass/fail status must be based on observed simulation logs and test results.

### Verification Targets

- Clock and reset operation.
- UART transmission and reception.
- RX FIFO buffering and data access.
- PicoRV32 MMIO transactions.
- AES-128 known-answer test vectors.
- Processor/peripheral interaction.
- Secure boot controller state transitions.
- Firmware update and application-execution behavior.
- ESP32-S3 HTTP upload and LittleFS file handling.
- UART/XMODEM-CRC transfer from the ESP32-S3 to the ZedBoard.
- Rejection of corrupted or unauthorized firmware.

### AES-128 Known-Answer Test

Use this standard AES-128 encryption vector when testing a compatible implementation:

- **Key:** `000102030405060708090a0b0c0d0e0f`
- **Plaintext:** `00112233445566778899aabbccddeeff`
- **Expected ciphertext:** `69c4e0d86a7b0430d8cdb78070b4c55a`

Confirm byte ordering and register packing in the actual accelerator before comparing results.

### Simulation Environment

| Technology | Purpose |
|---|---|
| Verilog HDL | Hardware design |
| PicoRV32 | RISC-V processor |
| ModelSim / QuestaSim | RTL simulation |
| Tcl | Simulation automation |
| C | Embedded firmware and drivers |
| FPGA tools | Synthesis, implementation, and hardware validation |

### Running the Simulation

The repository may contain a Tcl script named `run_secure_soc_full.tcl`.

From the appropriate simulator working directory, run:

```tcl
do run_secure_soc_full.tcl
```

Before running the script, verify that:

- All RTL dependencies are included.
- The firmware image exists at the configured path.
- The testbench references the correct top-level module.
- File paths are valid on the current machine.
- The simulator compile order matches module dependencies.

Prefer repository-relative paths where practical to make simulation reproducible across computers.

## 📁 Suggested Project Structure

The following structure is illustrative. Keep the actual filenames and paths aligned with the repository.

```text
picorv32-secure-firmware-update-defense/
├── dashboard/                  # icoCrypt web dashboard, if stored here
├── esp32_gateway/              # ESP32-S3 firmware, if stored here
│   ├── src/
│   └── platformio.ini          # or the actual build configuration
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

*This is a suggested documentation layout. It does not imply that every listed file has already been created.*

## 🔐 Security Considerations

- **Confidentiality:** Use AES-128 with an appropriate mode and correct key handling where confidentiality is required.
- **Integrity:** Detect modifications to firmware data.
- **Authenticity:** Verify that an update is authorized by a trusted source.
- **Anti-rollback:** Prevent unauthorized downgrades if version protection is implemented.
- **Secure boot:** Require successful verification before executing the selected firmware.
- **Recovery:** Define safe behavior for rejected updates or interrupted writes.
- **Key protection:** Protect cryptographic keys against unauthorized access.
- **Transport security:** Local Wi-Fi and HTTP should not be assumed secure against every attacker.

SHA-256 detects changes only when compared against a trusted digest. AES-CBC/CTR provide encryption but do not authenticate the firmware on their own. XMODEM-CRC helps detect transmission errors but does not authenticate the sender.

A secure update system should use a properly verified MAC, such as AES-CMAC or HMAC with a protected key, or a suitable digital-signature scheme. Only describe these as implemented if the corresponding RTL/software and tests support them.

These are design requirements, not a claim that all protections are implemented. Defense-grade deployment would require a defined threat model, suitable authentication, protected key storage, fault testing, and independent security evaluation.

## 🚧 Development Status

The project is under active development. This section distinguishes development artifacts from verified functionality.

### Development Artifacts Reported

- [x] PicoRV32 selected for processor integration.
- [x] UART receiver and transmitter RTL modules developed.
- [x] Baud-rate generation and receive FIFO modules developed.
- [x] Firmware memory and secure boot controller RTL developed.
- [x] SoC top-level integration and MMIO interface developed.
- [x] Full-SoC testbench and Tcl simulation flow prepared.
- [x] Embedded driver source/header development initiated.
- [ ] ESP32-S3 Wi-Fi, HTTP, LittleFS, and XMODEM-CRC path verified end to end.
- [ ] Complete dashboard-to-FPGA update flow demonstrated.

### Remaining Verification Tasks

- [ ] Confirm all RTL dependencies compile and elaborate successfully.
- [ ] Verify UART transmission and reception.
- [ ] Validate AES-128 against known-answer test vectors.
- [ ] Verify processor-to-peripheral MMIO transactions.
- [ ] Complete and validate the update manager and bootloader.
- [ ] Verify firmware integrity and authenticity checks.
- [ ] Demonstrate the complete host-to-FPGA update flow.
- [ ] Test corrupted and unauthorized firmware rejection.
- [ ] Verify reset behavior and recovery after update failure.
- [ ] Complete FPGA implementation and hardware demonstration.

Mark tasks complete only after the corresponding tests pass. The existence of RTL, C source, a dashboard, or a testbench does not by itself prove that the system works end to end.

## 🔮 Future Enhancements

Potential improvements include:

- HMAC or digital-signature-based firmware authentication.
- A/B firmware slots and safe rollback.
- External SPI flash integration.
- Anti-rollback protection with trusted version metadata.
- Watchdog-based recovery.
- Fault injection for corrupted or interrupted updates.
- FPGA resource-utilization and timing reports.
- Hardware-versus-software cryptographic performance benchmarking.
- Stronger gateway authentication and protected key provisioning.

## 🛠️ Technologies Used

- Verilog HDL
- PicoRV32 RISC-V processor
- AES-128 hardware acceleration
- ESP32-S3
- WiFiManager
- HTTP upload
- LittleFS
- UART and XMODEM-CRC
- Memory-mapped I/O (MMIO)
- Embedded C
- Browser-side JavaScript cryptography
- ModelSim / QuestaSim
- Tcl simulation automation
- FPGA development tools

## 👥 Project Contributions

### Hardware / FPGA

- PicoRV32 SoC integration.
- UART and FIFO RTL development.
- MMIO and peripheral integration.
- AES hardware and secure boot-control integration.
- Testbench development and simulation debugging.

### Embedded / Gateway

- ESP32-S3 Wi-Fi setup using WiFiManager.
- HTTP firmware upload endpoint.
- LittleFS package storage.
- UART/XMODEM-CRC transfer to the ZedBoard.
- Transfer status and error handling.

### Dashboard / Application

- Firmware selection and metadata handling.
- SHA-256 digest calculation.
- Browser-side AES-128 encryption.
- ESP32-S3 discovery/connection and HTTP upload interface.

The final contribution breakdown should reflect the actual work completed by each team member.

## 📄 License

The PicoRV32 core and third-party components retain their respective licensing terms. Select a project-level license only after checking compatibility with all included dependencies.

## 🙏 Acknowledgements

- [PicoRV32 by YosysHQ](https://github.com/YosysHQ/picorv32)
- The open-source FPGA, embedded-systems, and hardware-verification communities.

---

**Project focus:** Research and prototyping of an FPGA-based firmware-update path combining browser-side package preparation, an ESP32-S3 Wi-Fi gateway, UART transfer, PicoRV32 control, AES-128 hardware, and boot-validation logic for defense-oriented embedded-system research.
