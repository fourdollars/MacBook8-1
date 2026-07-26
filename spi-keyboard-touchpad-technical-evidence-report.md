# Technical Evidence Report: SPI Keyboard & Trackpad Operation Mode on MacBook 12"

- **Target Hardware**: Apple MacBook 12" (Model Identifier: `MacBook8,1`)
- **Logic Board Part Number**: 820-00244-A (X260 MLB)
- **Platform/Chipset**: Intel Broadwell-LP Platform Controller Hub (PCH)
- **Target Subsystem**: Intel LPSS SPI1 Controller (PCI Device ID: `8086:9CE6`)
- **Connected Peripherals**: Apple SPI Keyboard & SPI Trackpad (`AppleHSSPIHIDDriver`)

---

## 1. Executive Summary

This report documents empirical hardware and driver-level evidence gathered across multiple operating system environments (macOS and Windows) as well as the official hardware schematic for the 820-00244 logic board.

### Core Architectural Findings
Apple designed a system architecture that, via ACPI firmware and driver-level configurations, completely disables PCH LPSS-DMA channel allocations and utilizes a dedicated hardware-level Out-Of-Band (OOB) GPIO interrupt line (`TPAD_SPI_INT_L`) to wake the CPU upon user touch or keypress, fetching small input payloads via low-latency Programmed I/O (PIO) FIFO transfers across both macOS and Windows.

---

## 2. macOS IOKit Subsystem Observations (`ioreg`)

An analysis of the IOKit registry dump (`ioreg -p IOService -l`) on macOS yields the following class instance allocations and device tree states:

### 2.1 IOKit Class Instance Allocation Count
- **`AppleIntelLpssSpiController = 1`**: The primary LPSS SPI bus controller driver is active.
- **`AppleHSSPIController = 1`**: The high-speed SPI bus interface controller is loaded.
- **`AppleHSSPIHIDDriver = 5`**: Five HID driver instances for the integrated SPI keyboard and multi-touch trackpad are loaded and functional.
- **`AppleIntelLpssDmac = 1`**: The LPSS DMA hardware controller driver is loaded and instantiated.
- **`AppleIntelLpssDmacChannel = 0`**: Zero DMA channel instances are allocated or active within the LPSS DMA driver subsystem.

### 2.2 System Topology & PCI Resource Mapping
The top-level power and PCI domain tree (`IOPMrootDomain` -> `PCITopLevel`) declares both SPI1 and SDMA (the LPSS DMA Engine) at the PCH hardware level:

```text
"PCITopLevel" = ("MCHC","IGPU","HDAU","XHC1","SDMA","I2C1","SPI1","URT0","HDEF","RP01","RP03","RP04","RP05","LPCB","SBUS")
```

> **Observation**: The PCH hardware topography declares the physical availability of SDMA alongside SPI1. However, during active system operation with `AppleHSSPIHIDDriver` fully active, `AppleIntelLpssDmacChannel` remains strictly at `0`.

---

## 3. Windows MMIO Register-Level Observations (RWEverything)

Real-time Memory-Mapped I/O (MMIO) inspection was performed on Windows using RWEverything under the official Intel Serial IO SPI Host Controller driver (`iaLPSS_SPI.sys`).

### 3.1 Resource Allocation
- **Device Identification**: Vendor ID `0x8086`, Device ID `0x9CE6`, Subsystem ID `0x9CE68086`.
- **Memory Base Address**: The controller operates with MMIO mapped exclusively to BAR1 (e.g., Physical Base Address `0xC181A000`, 4KB region). BAR0 and BAR2 remain unallocated (`0x00000000`).

### 3.2 Register Behavior During Active Trackpad Interaction

#### FIFO Data Ports (Offsets `0x00`–`0x10`)
- When finger movement or tap input occurs on the trackpad, values dynamically update at offset `0x00` (e.g., `0x29`), offset `0x04` (e.g., `0x03`), and offset `0x10` (e.g., `0x03`).
- These locations correspond to the LPSS SPI Transmit/Receive FIFO data registers.

#### Idle Bus State
- When no input is applied to the keyboard or trackpad, reads across the FIFO data register range return `0xFF`, indicating an empty FIFO queue / idle bus floating state.

#### LPSS DMA Control Block (Offsets `0x800`–`0x8FF`)
- The dedicated LPSS DMA configuration and control registers (including `DMA_CTL`, channel descriptors, and transfer counters starting at offset `0x800`) remain static at `0x00` throughout all touch, gesture, and keypress activities.
- No DMA status or transfer control bit changes are observed in this region.

---

## 4. Hardware Schematic Verification (Logic Board 820-00244-A)

Circuit inspection of the Apple 820-00244-A schematic (X260 MLB) confirms the physical pinout and routing between the PCH and the trackpad/keyboard interface connector.

### 4.1 Connector Pinout (J4801 — Trackpad & Keyboard Flex Interface, Page 48)
- **Pin 2**: `TPAD_SPI_MISO` (Master In Slave Out)
- **Pin 4**: `TPAD_SPI_MOSI` (Master Out Slave In)
- **Pin 6**: `TPAD_SPI_CLK` (Serial Clock)
- **Pin 8**: `TPAD_SPI_CS_L` (Chip Select, Active Low)
- **Pin 10**: `TPAD_SPI_INT_L` (Hardware Interrupt Line, Active Low)

### 4.2 Interrupt Signaling & Pull-Up Configuration (Page 17 & Page 48)
- **Signal Routing**: `TPAD_SPI_INT_L` is routed directly from J4801 Pin 10 to a dedicated PCH GPIO / General Purpose Event (GPE) input pin.
- **Passive Circuitry**: The line is tied to `PP3V3_S0` through pull-up resistor `R1779` (100 kΩ).

> **Observation**: The presence of `TPAD_SPI_INT_L` provides a physical hardware path for the trackpad controller to assert a low-active GPIO interrupt directly to the PCH upon detecting user input.

---

## 5. Summary of Fact-Based Findings & Architecture Summary

| Domain / Layer | Target Element | Observed Fact |
| :--- | :--- | :--- |
| **macOS (`ioreg`)** | `AppleIntelLpssDmacChannel` | Value is `0` while 5 instances of `AppleHSSPIHIDDriver` are active. |
| **Windows (MMIO)** | `BAR1 + 0x800` (DMA Control) | Values remain statically `0x00` during active trackpad/keyboard input. |
| **Windows (MMIO)** | `BAR1 + 0x00` (SPI FIFO) | Values dynamically cycle during input and return `0xFF` when idle. |
| **Schematic (820-00244)** | `J4801 Pin 10` | Dedicated `TPAD_SPI_INT_L` physical interrupt line with pull-up `R1779`. |

### Final Architectural Summary Statement
> *"Apple designed a system architecture that, via ACPI firmware and driver-level configurations, completely disables PCH LPSS-DMA channel allocations and utilizes a dedicated Out-Of-Band (OOB) GPIO interrupt (`TPAD_SPI_INT_L`) to wake the CPU for low-latency PIO/FIFO transfers, completely bypassing the—otherwise supported—PCH's LPSS-DMA engine."*
