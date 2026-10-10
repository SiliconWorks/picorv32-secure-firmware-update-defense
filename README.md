# 🛡️ PicoCrypt — Hardware-Accelerated Secure Firmware Update and Boot System

**Using PicoRV32, AES-128 Hardware, ESP32-S3 Wi-Fi Gateway, and ZedBoard FPGA for Defense-Oriented Applications**


---

## 📌 Overview

**PicoCrypt** is a web-based secure firmware deployment dashboard designed for an FPGA-based firmware update and boot system. The proposed architecture integrates a browser dashboard, an ESP32-S3 Wi-Fi gateway, and a ZedBoard FPGA running a PicoRV32-based SoC with AES-128 cryptographic hardware and secure boot control logic.

The dashboard is designed to allow users to select a raw firmware binary (`.bin`), calculate its SHA-256 digest, encrypt the firmware in the browser using AES-128 in the configured CBC or CTR mode, and upload the encrypted package over a local Wi-Fi network to the ESP32-S3.

The ESP32-S3 gateway is intended to:

- Connect to the same local Wi-Fi network as the host laptop using a WiFiManager configuration portal.
- Receive the encrypted firmware package and metadata through an HTTP upload endpoint.
- Store the uploaded package in its LittleFS filesystem.
- Transfer the stored package to the ZedBoard over UART using XMODEM-CRC.

On the FPGA, the PicoRV32 processor controls the SoC peripherals through memory-mapped I/O (MMIO). The system is intended to receive and stage the firmware, perform the configured AES operation, validate the firmware using an implemented integrity and authenticity mechanism, and permit execution only when the required checks succeed.

The project follows a hardware–software co-design approach involving Verilog RTL, embedded C, ESP32-S3 firmware, browser-side JavaScript, peripheral integration, and RTL simulation.

**Project focus:** Research and prototyping of hardware-accelerated cryptographic processing and secure firmware deployment for defense-oriented embedded systems.

---

## 🎯 Project Objectives

- Integrate the PicoRV32 RISC-V processor into a custom FPGA-based SoC.
- Develop a web-based firmware deployment dashboard called PicoCrypt.
- Select and prepare raw firmware binary files.
- Calculate SHA-256 digests for firmware integrity checking.
- Encrypt firmware in the browser using the configured AES-128 mode.
- Connect the ESP32-S3 gateway to the host laptop's local Wi-Fi network.
- Upload encrypted firmware through HTTP.
- Store uploaded firmware packages in ESP32-S3 LittleFS.
- Transfer firmware from the ESP32-S3 to the ZedBoard using UART and XMODEM-CRC.
- Receive and buffer serial data using UART and FIFO logic.
- Integrate AES-128 hardware acceleration into the FPGA SoC.
- Connect the processor and peripherals through MMIO.
- Coordinate firmware staging, verification, and boot decisions.
- Verify individual modules before demonstrating the complete update workflow.

---

## 🏗️ Complete System Architecture

```text
                    HOST LAPTOP / PC
              ┌──────────────────────────┐
              │     PicoCrypt Dashboard  │
              │                          │
              │  Select firmware (.bin)  │
              │  Calculate SHA-256       │
              │  AES-128 encryption      │
              │  HTTP firmware upload    │
              └────────────┬─────────────┘
                           │
                    Local Wi-Fi Network
                           │ HTTP
                           ▼
              ┌──────────────────────────┐
              │      ESP32-S3 Gateway    │
              │                          │
              │  WiFiManager              │
              │  HTTP Upload Server       │
              │  LittleFS File Storage    │
              │  XMODEM-CRC UART Sender   │
              └────────────┬─────────────┘
                           │
                     UART TX / RX / GND
                     XMODEM-CRC Transfer
                           │
                           ▼
┌────────────────────────────────────────────────────────────┐
│                     ZEDBOARD FPGA                           │
│                                                            │
│  ┌───────────────────┐                                     │
│  │ UART RX / TX      │                                     │
│  └─────────┬─────────┘                                     │
│            ▼                                               │
│  ┌───────────────────┐                                     │
│  │ RX FIFO / Buffer  │                                     │
│  └─────────┬─────────┘                                     │
│            ▼                                               │
│  ┌───────────────────┐                                     │
│  │ PicoRV32 CPU      │                                     │
│  │ RISC-V Processor  │                                     │
│  └─────────┬─────────┘                                     │
│            ▼                                               │
│  ┌───────────────────┐                                     │
│  │ MMIO Interconnect │                                     │
│  │ Address Decoder   │                                     │
│  └──────┬─────┬──────┴─────────┐                           │
│         │     │                │                           │
│         ▼     ▼                ▼                           │
│  ┌──────────┐ ┌────────────┐ ┌──────────────────┐          │
│  │ AES-128  │ │ Firmware   │ │ Secure Boot      │          │
│  │ Hardware │ │ Staging /  │ │ Controller FSM   │          │
│  │          │ │ Program RAM│ │                  │          │
│  └──────────┘ └────────────┘ └──────────────────┘          │
│                                                            │
│            Clock, Reset, Status and Control                │
└────────────────────────────────────────────────────────────┘
```

*This diagram represents the intended architecture. Actual signal paths, register addresses, memory behavior, and security checks must be confirmed against the current RTL and embedded software.*

---

## 🔄 End-to-End Firmware Deployment Workflow

### Stage 1: Firmware Preparation

1. The user opens the PicoCrypt web dashboard.
2. The user selects a firmware binary file (`.bin`).
3. The dashboard calculates the firmware's SHA-256 digest.
4. The dashboard encrypts the firmware using the configured AES-128 mode.
5. The dashboard prepares the payload and required metadata.

### Stage 2: Wi-Fi Upload to ESP32-S3

6. The ESP32-S3 connects to the same local Wi-Fi network as the laptop using WiFiManager.
7. The dashboard discovers or connects to the ESP32-S3 using the mechanism implemented by the application.
8. The encrypted firmware package is uploaded to the ESP32-S3 over HTTP.
9. The ESP32-S3 stores the uploaded package and metadata in LittleFS.

### Stage 3: UART Transfer to ZedBoard

10. The ESP32-S3 reads the stored firmware package.
11. The gateway sends the package over UART using XMODEM-CRC.
12. The FPGA UART receiver receives the serial data.
13. The RX FIFO buffers incoming data until it is consumed by the processor or associated logic.
14. PicoRV32 and the update-control logic manage the reception and staging process.

### Stage 4: Cryptographic Processing and Verification

15. The processor configures the AES-128 hardware through MMIO.
16. The AES accelerator performs the configured cryptographic operation.
17. The system verifies firmware integrity and authenticity using the implemented security mechanism.
18. The secure boot controller evaluates the validation result.
19. Invalid firmware must be rejected.
20. Firmware execution should be permitted only after all required validation checks succeed.

The complete sequence is the intended workflow. It should be described as fully operational only after end-to-end testing confirms each stage.

---

## 🧩 Main Hardware and Software Components

### 1. PicoCrypt Web Dashboard

The browser dashboard provides the interface for preparing and deploying firmware.

**Intended responsibilities:**

- Firmware `.bin` selection.
- SHA-256 digest calculation.
- Browser-side AES-128 encryption.
- ESP32-S3 discovery or connection.
- HTTP upload of the encrypted package and metadata.
- Upload progress and result reporting, if implemented.

**Important security note:** A SHA-256 digest is useful for detecting changes when compared against a trusted expected digest. If an attacker can replace both the firmware and its digest, the digest alone does not establish firmware authenticity.

### 2. ESP32-S3 Wi-Fi Gateway

The ESP32-S3 acts as the bridge between PicoCrypt and the FPGA.

| Component | Responsibility |
|---|---|
| WiFiManager | Configure the gateway's Wi-Fi connection |
| HTTP upload server | Receive encrypted firmware and metadata |
| LittleFS | Store the received package in ESP32-S3 flash |
| UART sender | Transmit the stored package to the ZedBoard |
| XMODEM-CRC | Provide block-based serial transfer with CRC error detection |
| Status handling | Report transfer results if implemented |

LittleFS on the ESP32-S3 is gateway staging storage. It is separate from the firmware memory inside the FPGA.

### 3. UART Communication Interface

UART provides serial communication between the ESP32-S3 and ZedBoard.

Relevant FPGA modules may include:

- `uart_rx.v` — UART receive logic.
- `uart_tx.v` — UART transmit logic.
- `baud_gen.v` — Baud-rate timing generation.
- `rx_fifo.v` — Receive FIFO.
- SoC-level UART integration logic.

The RX FIFO temporarily stores received bytes until the processor or associated logic consumes them.

**Typical UART wiring:**

- ESP32-S3 TX → ZedBoard UART RX
- ESP32-S3 RX ← ZedBoard UART TX
- ESP32-S3 GND ↔ ZedBoard GND

Confirm the actual pin mapping, baud rate, UART interface, and voltage compatibility before connecting the hardware.

### 4. PicoRV32 RISC-V Processor

PicoRV32 is the processor core used to execute embedded firmware and coordinate peripheral operations.

Responsibilities include:

- Executing embedded C instructions.
- Accessing peripherals through MMIO.
- Controlling UART and AES operations.
- Coordinating firmware-update and boot-control operations.

This project integrates the existing PicoRV32 core rather than designing a processor from scratch.

Reference: [YosysHQ PicoRV32](https://github.com/YosysHQ/picorv32)

### 5. Memory-Mapped I/O (MMIO)

MMIO assigns address ranges to peripheral registers. The CPU accesses those registers through normal load and store instructions.

The MMIO interconnect and address decoder route CPU requests to the appropriate memory or peripheral.

The hardware RTL and C drivers must agree on:

- Peripheral base addresses.
- Register offsets.
- Access widths and byte enables.
- Start, busy, done, error, and status bits.
- Bus-ready and wait-state behavior.

A mismatch between software register definitions and RTL decoding can cause peripheral accesses to fail or the processor to wait indefinitely.

### 6. AES-128 Hardware Accelerator

AES-128 is a symmetric-key block cipher that operates on 128-bit data blocks using a 128-bit key.

The hardware accelerator is intended to offload configured cryptographic operations from the CPU.

A typical interface may include:

- Key registers.
- Input-data registers.
- Start/control register.
- Busy/done status register.
- Output-data registers.

The exact interface depends on the actual RTL implementation.

**AES encryption alone does not authenticate firmware.**

- AES-CBC and AES-CTR provide confidentiality when used correctly, but do not authenticate the firmware by themselves.
- XMODEM-CRC detects certain transmission errors but does not authenticate the sender.
- SHA-256 alone does not establish who produced a firmware image.
- Firmware authenticity requires a correctly verified MAC, such as AES-CMAC or HMAC with a protected key, or a digital signature checked against a trusted public key.

Only claim a particular authentication mechanism is implemented when the relevant RTL/software and tests confirm it.

### 7. Firmware Memory

The FPGA firmware memory may use RAM or BRAM to store program instructions, working data, or a staged firmware image.

The exact behavior depends on the memory map and implemented control logic.

- **ESP32-S3 LittleFS:** stores the uploaded package at the gateway.
- **FPGA BRAM/RAM:** may store program code, working data, or staged firmware.
- **External nonvolatile flash:** is needed for persistent FPGA-side firmware storage across power cycles if the design requires it and includes the necessary interface.

A simulation `.hex` file used to initialize memory is not automatically persistent storage on the physical FPGA.

### 8. Secure Boot Controller

The secure boot controller coordinates boot-related state transitions and update decisions.

Its intended responsibilities include:

- Managing boot and update states.
- Coordinating firmware validation.
- Reporting status.
- Controlling whether execution may proceed.
- Handling rejected updates according to the implemented design.

Secure boot is effective only when execution is actually gated by trustworthy verification results.

---

## 💻 Embedded Software and Drivers

Embedded C software is intended to execute on PicoRV32 and communicate with hardware through MMIO.

| Software component | Responsibility |
|---|---|
| UART driver | Transmit and receive serial data using the implemented registers |
| AES driver | Configure and control the AES accelerator |
| GPIO driver | Access GPIO registers if implemented |
| Memory driver | Access the actual firmware memory interface |
| Update manager | Coordinate firmware-update stages and error handling |
| Bootloader | Perform boot-time checks and select accepted firmware |

The software driver register definitions must match the actual hardware register map.

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

This is only an illustrative example. The address `0x10000000` must not be assumed to be the correct address for a particular peripheral without checking the integrated RTL.

---

## 🧪 Simulation and Verification

The project uses RTL simulation to debug hardware behavior and processor/peripheral integration.

A full-SoC testbench and Tcl simulation flow have been prepared for ModelSim/QuestaSim. Passing status must be based on the current simulation logs and test results.

### Suggested Verification Order

1. **Compilation and elaboration:** Confirm all modules compile and dependencies are included.
2. **Clock and reset:** Verify that the processor and peripherals leave reset correctly.
3. **CPU/MMIO:** Confirm that the CPU can access memory and peripheral registers.
4. **UART/FIFO:** Test serial reception, transmission, FIFO status, and byte ordering.
5. **AES-128:** Run a known-answer test and compare the result with the expected ciphertext.
6. **Boot controller:** Verify state transitions and rejection behavior.
7. **ESP32-S3 gateway:** Test Wi-Fi, HTTP upload, LittleFS read/write, and UART transfer separately.
8. **End-to-end deployment:** Demonstrate the full dashboard-to-FPGA update flow.
9. **Negative testing:** Test corrupted data, invalid authentication, interrupted transfer, and reset during an update.

### AES-128 Known-Answer Test

For a compatible AES-128 encryption implementation, use the following standard test vector:

- **Key:** `000102030405060708090a0b0c0d0e0f`
- **Plaintext:** `00112233445566778899aabbccddeeff`
- **Expected ciphertext:** `69c4e0d86a7b0430d8cdb78070b4c55a`

Confirm byte ordering and register packing in the actual accelerator before comparing the result.

### Simulation Environment

| Technology | Purpose |
|---|---|
| Verilog HDL | Hardware design |
| PicoRV32 | RISC-V processor core |
| ModelSim / QuestaSim | RTL simulation |
| Tcl | Simulation automation |
| C | Embedded firmware and drivers |
| FPGA development tools | Synthesis, implementation, and hardware validation |

### Running the Simulation

If the repository contains `run_secure_soc_full.tcl`, run it from the appropriate simulator working directory:

```tcl
do run_secure_soc_full.tcl
```

Before running, confirm:

- All RTL dependencies are included.
- The firmware image exists at the configured path.
- The testbench references the correct top-level module.
- File paths are valid on the current machine.
- The compilation order matches module dependencies.

Repository-relative paths are recommended where practical.

---

## 📁 Suggested Project Structure

The structure below is illustrative. Use the actual file names and folders in the repository.

```text
picorv32-secure-firmware-update-defense/
├── dashboard/                  # PicoCrypt web dashboard, if stored here
├── esp32_gateway/               # ESP32-S3 firmware, if stored here
│   ├── src/
│   └── platformio.ini           # Or the actual build configuration
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
│   ├── update_manager.c          # If implemented
│   ├── bootloader.c              # If implemented
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

The suggested layout does not imply that every listed file has already been created.

---

## 🔐 Security Considerations

- **Confidentiality:** Use AES-128 with an appropriate mode and correct key handling.
- **Integrity:** Detect changes to firmware data.
- **Authenticity:** Verify that firmware comes from an authorized source.
- **Anti-rollback:** Prevent unauthorized downgrades if version protection is implemented.
- **Secure boot:** Gate execution on successful validation.
- **Recovery:** Define safe behavior for rejected or interrupted updates.
- **Key protection:** Protect cryptographic keys against unauthorized access.
- **Transport security:** Do not assume that local Wi-Fi or HTTP is protected against every attacker.

These are design requirements, not a claim that all protections have already been implemented. Deployment in operational defense systems would require a defined threat model, appropriate authentication, protected key storage, fault testing, and independent security evaluation.

---

## 🚧 Development Status

The project is under active development. The presence of source files or modules does not by itself prove correct operation.

### Development Artifacts Reported

- [x] PicoRV32 selected for processor integration.
- [x] UART receiver and transmitter RTL modules developed.
- [x] Baud-rate generation and RX FIFO modules developed.
- [x] Firmware memory and secure boot controller RTL developed.
- [x] SoC top-level integration and MMIO interface developed.
- [x] Full-SoC testbench and Tcl simulation flow prepared.
- [x] Embedded driver source/header development initiated.

### Verification and Integration Tasks

- [ ] Confirm all RTL dependencies compile and elaborate.
- [ ] Verify UART transmission and reception.
- [ ] Validate AES-128 using a known-answer test.
- [ ] Verify CPU-to-peripheral MMIO transactions.
- [ ] Complete and validate the update manager and bootloader.
- [ ] Verify firmware integrity and authenticity checks.
- [ ] Verify ESP32-S3 Wi-Fi, HTTP, LittleFS, and UART functions.
- [ ] Demonstrate the complete PicoCrypt-to-FPGA update flow.
- [ ] Test corrupted and unauthorized firmware rejection.
- [ ] Verify reset behavior and recovery after update failure.
- [ ] Complete FPGA implementation and hardware demonstration.

Mark each task complete only after its corresponding test passes.

---

## 🔮 Future Enhancements

- HMAC or digital-signature-based firmware authentication.
- A/B firmware slots and safe rollback.
- External SPI flash integration.
- Anti-rollback protection using trusted version metadata.
- Watchdog-based recovery.
- Fault injection for corrupted or interrupted updates.
- FPGA resource-utilization and timing reports.
- Hardware-versus-software cryptographic performance benchmarking.
- Stronger gateway authentication and protected key provisioning.

---

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
- JavaScript
- ModelSim / QuestaSim
- Tcl simulation automation
- FPGA development tools

---

## 👥 Project Contributions

### Hardware / FPGA Team

- PicoRV32 SoC integration.
- UART and FIFO RTL development.
- MMIO and peripheral integration.
- AES hardware and secure boot-control integration.
- Testbench development and simulation debugging.

### Embedded / Gateway Team

- ESP32-S3 Wi-Fi configuration using WiFiManager.
- HTTP firmware upload endpoint.
- LittleFS package storage.
- UART/XMODEM-CRC transfer to the ZedBoard.
- Transfer status and error handling.

### Dashboard / Application Team

- PicoCrypt firmware selection interface.
- SHA-256 digest calculation.
- Browser-side AES-128 encryption.
- ESP32-S3 discovery or connection.
- HTTP upload interface and status reporting.

The final contribution breakdown should reflect the work actually completed by each team member.

---

## 📄 License

The PicoRV32 core and third-party components retain their respective licensing terms. Choose a project-level license only after checking compatibility with all dependencies.

## 🙏 Acknowledgements

- [PicoRV32 by YosysHQ](https://github.com/YosysHQ/picorv32)
- The open-source FPGA, embedded-systems, and hardware-verification communities.

---

**Project Focus:** Research and prototyping of a secure firmware deployment pipeline combining PicoCrypt, an ESP32-S3 Wi-Fi gateway, UART transfer, PicoRV32 control, AES-128 hardware, firmware memory, and boot-validation logic for defense-oriented embedded-system research.
