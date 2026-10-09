# picorv32-secure-firmware-update-defense

# 🛡️ Hardware-Accelerated Secure Firmware Update and Boot System Using PicoRV32 and AES-128 on FPGA for Defense Systems

**An FPGA-based secure embedded-system architecture for firmware update processing, cryptographic acceleration, and controlled boot operations in defense-oriented applications.**

![Language](https://img.shields.io/badge/Language-Verilog%20%7C%20C-blue)
![Processor](https://img.shields.io/badge/Processor-PicoRV32-orange)
![Architecture](https://img.shields.io/badge/Architecture-RISC--V-purple)
![Cryptography](https://img.shields.io/badge/Cryptography-AES--128-red)
![Platform](https://img.shields.io/badge/Platform-FPGA-green)
![Status](https://img.shields.io/badge/Status-Under%20Development-yellow)

## 📌 Overview

Modern defense-oriented embedded systems require reliable mechanisms for updating firmware while protecting devices against unauthorized software, malicious modifications, and unsafe boot operations.

This project explores the design and development of a **hardware-accelerated secure firmware update and boot system** using the PicoRV32 RISC-V processor and AES-128 cryptographic hardware on an FPGA.

The system integrates a processor, UART communication interface, receive FIFO, firmware memory, cryptographic processing hardware, and secure boot control logic. Embedded C software communicates with hardware peripherals through a memory-mapped I/O (MMIO) interface.

The project follows a hardware–software co-design approach, combining Verilog RTL development, embedded firmware, peripheral integration, and RTL simulation.

The intended application is research and prototyping for secure embedded systems in defense-oriented environments.

## 🎯 Project Objectives

- Design a processor-based FPGA system using the PicoRV32 RISC-V core.
- Develop UART communication and receive-buffering logic for firmware transfer.
- Integrate AES-128 hardware acceleration for cryptographic processing.
- Connect hardware peripherals through a memory-mapped interface.
- Develop embedded C drivers for peripheral access.
- Implement firmware-update management and secure boot control.
- Verify hardware and software integration through simulation.
- Establish a foundation for firmware integrity verification, authenticity checks, and recovery mechanisms.

## 🏗️ System Architecture

The proposed architecture consists of a host PC and an FPGA-based embedded system.

```text
                 HOST PC
          Firmware Update Package
                    |
                    v
             USB-to-UART
                    |
                    v
+------------------------------------------------+
|                     FPGA                       |
|                                                |
|  +------------------------------------------+  |
|  |              PicoRV32 CPU                |  |
|  |           RISC-V Processor               |  |
|  +--------------------+---------------------+  |
|                       |                        |
|                MMIO Interface                  |
|                       |                        |
|       +---------------+---------------+        |
|       |               |               |        |
|       v               v               v        |
|  +----------+   +-----------+   +-----------+  |
|  | UART and |   | AES-128   |   | Firmware  |  |
|  | RX FIFO  |   | Accelerator|  | Memory    |  |
|  +----------+   +-----------+   +-----------+  |
|       |               |               |        |
|       +---------------+---------------+        |
|                       |                        |
|             +--------------------+             |
|             | Secure Boot and    |             |
|             | Update Controller  |             |
|             +--------------------+             |
|                                                |
+------------------------------------------------+
```

*The diagram shows the intended architecture. Actual connectivity and verification status depend on the current RTL integration.*

## 🔄 Firmware Update Workflow

The intended end-to-end operation consists of the following stages.

1. **Firmware preparation:** Build the new firmware image on the host PC.
2. **Package preparation:** Prepare the update data and the required cryptographic metadata.
3. **Serial transmission:** Send the update package to the FPGA through UART.
4. **Data reception:** Receive and buffer incoming bytes through the UART receiver and RX FIFO.
5. **Processor control:** PicoRV32 reads received data and controls peripherals using MMIO registers.
6. **Cryptographic processing:** Use AES-128 hardware for the configured encryption or decryption operation.
7. **Firmware verification:** Validate the image using the implemented integrity and authenticity mechanism.
8. **Update decision:** Accept or reject the image based on the validation results and applicable update policies.
9. **Firmware storage:** Store an accepted image in the designated memory region.
10. **Boot control:** Permit execution only after the implemented boot-validation checks succeed.
11. **Application execution:** Transfer control to the accepted firmware image.

The complete workflow is a development objective. Each stage must be implemented and tested before the entire system can be considered functional.

## 🧩 Hardware Components

### 1. PicoRV32 RISC-V Processor

PicoRV32 is the processor core used to execute the embedded C firmware and control the SoC peripherals.

The project integrates the existing open-source processor into a custom system rather than designing a CPU from scratch.

Responsibilities include:

- Executing embedded firmware.
- Controlling peripherals through MMIO.
- Managing firmware-update operations.
- Coordinating cryptographic processing and boot-control operations.

Reference: https://github.com/YosysHQ/picorv32

### 2. UART Communication Interface

UART (Universal Asynchronous Receiver/Transmitter) provides serial communication between the host PC and FPGA.

The UART subsystem includes receive and transmit logic, baud-rate generation, and receive buffering.

Relevant modules include:

- `uart_rx.v` — UART receive logic.
- `uart_tx.v` — UART transmit logic.
- `baud_gen.v` — Baud-rate timing generation.
- `rx_fifo.v` — Receive FIFO.
- SoC-level UART integration logic.

### 3. Receive FIFO

The RX FIFO temporarily stores incoming bytes until the processor retrieves them.

It provides buffering between the UART receiver and processor-controlled data consumption.

The design includes data storage, read and write control, occupancy tracking, and empty/full handling.

### 4. AES-128 Hardware Acceleration

AES-128 (Advanced Encryption Standard with a 128-bit key) is the cryptographic hardware component of the project.

The accelerator is intended to offload cryptographic operations from software to dedicated RTL hardware.

The processor-facing interface must support the required data, key, control, and status operations.

AES encryption alone does not authenticate firmware. A secure update system must also verify that firmware originates from an authorized source and has not been modified.

### 5. Firmware Memory

The firmware memory subsystem stores firmware data used during update and boot operations.

Its interface must support the processor and update-control logic. Persistent storage, inactive firmware slots, and recovery behavior depend on the final implementation.

### 6. Secure Boot Controller

The secure boot controller coordinates boot-related state transitions and update-control decisions.

The intended functionality includes:

- Managing boot and update states.
- Reporting controller status.
- Coordinating firmware validation.
- Controlling whether execution may proceed.

Secure boot is only effective when the boot decision depends on trustworthy verification results and the system protects its trust anchors.

### 7. Memory-Mapped I/O Integration

Memory-mapped I/O allows the processor to access peripheral registers through defined memory addresses.

The SoC integration connects the PicoRV32 processor with the required memory and peripheral interfaces.

The software drivers and RTL must use matching register addresses, offsets, access widths, and control/status bit definitions.

## 💻 Embedded Software

The embedded software is written in C and is intended to execute on the PicoRV32 processor.

### Driver Responsibilities

| Component | Responsibility |
|---|---|
| UART driver | Initialize UART and transmit or receive data |
| GPIO driver | Access implemented GPIO registers |
| AES driver | Configure and control the AES hardware |
| Memory driver | Access the implemented firmware memory interface |
| Update manager | Coordinate firmware-update states |
| Bootloader | Perform boot-time checks and select firmware for execution |

Driver interfaces must match the actual hardware register map.

### MMIO Example

The following illustrative C code demonstrates a memory-mapped register access:

```c
#include <stdint.h>

#define PERIPHERAL_REG \
    (*(volatile uint32_t *)0x10000000u)

void peripheral_write(uint32_t value)
{
    PERIPHERAL_REG = value;
}

uint32_t peripheral_read(void)
{
    return PERIPHERAL_REG;
}
```

This is an example only. The address shown is not a universal address for the project's peripherals. The actual address must be confirmed from the integrated RTL.

## 🧪 Simulation and Verification

The project uses RTL simulation to evaluate hardware behavior and system integration.

A full-SoC testbench and Tcl simulation flow have been prepared for system-level verification using ModelSim/QuestaSim.

### Verification Targets

- Clock and reset operation.
- UART transmission and reception.
- RX FIFO buffering and data access.
- PicoRV32 MMIO transactions.
- AES-128 known-answer test vectors.
- Processor and peripheral interaction.
- Secure boot controller state transitions.
- Firmware update and application-execution behavior.

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

The repository includes a Tcl script named `run_secure_soc_full.tcl`.

From the appropriate simulator environment, run:

```tcl
do run_secure_soc_full.tcl
```

Before running the script, verify that:

- All RTL dependencies are included.
- The firmware image exists at the configured path.
- The testbench references the correct top-level module.
- File paths are valid on the current machine.
- The simulator compilation order matches the module dependencies.

Use repository-relative paths where practical to make the simulation reproducible across different computers.

## 📁 Project Structure

The following is a suggested documentation layout. Keep the actual filenames and paths in the repository if they differ.

```text
picorv32-secure-firmware-update-defense/
│
├── rtl/
│   ├── uart_rx.v
│   ├── uart_tx.v
│   ├── baud_gen.v
│   ├── rx_fifo.v
│   ├── firmware_mem.v
│   ├── secure_boot_ctrl.v
│   └── secure_boot_mmio.v
│
├── soc/
│   └── secure_soc_top.v
│
├── firmware/
│   ├── drivers/
│   │   ├── uart_driver.c
│   │   ├── uart_driver.h
│   │   ├── aes_driver.c
│   │   ├── aes_driver.h
│   │   ├── gpio_driver.c
│   │   └── gpio_driver.h
│   ├── update_manager.c
│   ├── bootloader.c
│   └── main.c
│
├── firmware_aes/
│   └── system_firmware.hex
│
├── sim/
│   ├── tb_secure_soc.v
│   └── run_secure_soc_full.tcl
│
├── docs/
│   ├── architecture.md
│   └── verification.md
│
└── README.md
```

*The structure is illustrative; it does not imply that every listed file has already been created.*

## 🔐 Security Considerations

Security is a core design objective of this project.

- **Confidentiality:** Use AES-128 with an appropriate cryptographic mode and correct key handling where confidentiality is required.
- **Integrity:** Detect modifications to firmware data.
- **Authenticity:** Verify that an update is authorized by a trusted source.
- **Anti-rollback:** Prevent unauthorized downgrades if version protection is implemented.
- **Secure boot:** Require successful verification before executing the selected firmware.
- **Recovery:** Define safe behavior for rejected updates or interrupted writes.
- **Key protection:** Protect cryptographic keys against unauthorized access.

These are design requirements, not a certification or claim that all protections are implemented. Defense-grade deployment would require a defined threat model, appropriate cryptographic authentication, protected key storage, fault testing, and independent security evaluation.

## 🚧 Development Status

The project is under active development.

### Existing Development Artifacts

- [x] PicoRV32 selected for processor integration.
- [x] UART receiver and transmitter RTL modules developed.
- [x] Baud-rate generation and receive FIFO modules developed.
- [x] Firmware memory and secure boot controller RTL developed.
- [x] SoC top-level integration and MMIO interface developed.
- [x] Full-SoC testbench and Tcl simulation flow prepared.
- [x] Embedded driver source/header development initiated.

### Remaining Verification and Integration Tasks

- [ ] Confirm that all RTL dependencies compile and elaborate successfully.
- [ ] Verify UART transmission and reception.
- [ ] Validate AES-128 against known-answer test vectors.
- [ ] Verify processor-to-peripheral MMIO transactions.
- [ ] Complete and validate the update manager and bootloader.
- [ ] Verify firmware integrity and authenticity checks.
- [ ] Demonstrate the complete host-to-FPGA update flow.
- [ ] Test corrupted and unauthorized firmware rejection.
- [ ] Verify reset behavior and recovery after update failure.
- [ ] Complete FPGA implementation and hardware demonstration.

The existence of RTL or software files does not itself prove successful operation. Checklist items should be marked complete only after the corresponding tests pass.

## 🔮 Future Enhancements

Potential future improvements include:

- SHA-256-based firmware hashing.
- HMAC or digital-signature-based firmware authentication.
- A/B firmware slots and safe rollback.
- External SPI flash integration.
- Anti-rollback protection with trusted version metadata.
- Watchdog-based recovery.
- Fault injection for corrupted or interrupted updates.
- FPGA resource-utilization and timing reports.
- Hardware-versus-software cryptographic performance benchmarking.

## 🛠️ Technologies Used

- Verilog HDL
- PicoRV32 RISC-V processor
- AES-128 hardware acceleration
- UART serial communication
- Memory-mapped I/O
- Embedded C
- ModelSim / QuestaSim
- Tcl simulation automation
- FPGA development tools

## 👥 Project Contributions

**Hardware development**
- Verilog RTL design and peripheral integration.
- PicoRV32 SoC integration.
- UART and receive FIFO development.
- Cryptographic and secure boot hardware integration.
- Testbench development and simulation debugging.

**Embedded software development**
- C driver development.
- Memory-mapped peripheral access.
- Firmware-update management.
- Bootloader development and system integration.

The final contribution breakdown should reflect the actual work completed by each team member.

## 📄 License

The PicoRV32 core and third-party components retain their respective licensing terms. A project-level license should be selected after checking compatibility with all included dependencies.

## 🙏 Acknowledgements

- [PicoRV32](https://github.com/YosysHQ/picorv32) — Open-source RISC-V processor core.
- The open-source FPGA, embedded-systems, and hardware-verification communities.

---

**Project focus:** Research and prototyping of hardware-accelerated cryptographic processing and secure firmware-update mechanisms for FPGA-based embedded systems intended for defense-oriented applications.
