# MacBook8-1

Linux enablement on MacBook8,1 (aka MacBook Retina, 12-inch, Early 2015)

## Documentation & Reports

- [Technical Report: Broadcom BCM4350 (`brcmfmac`) D3cold Suspend/Resume Analysis & eBPF Empirical Evidence](brcmfmac-suspend-d3cold-analysis-report.md)
  - Empirical eBPF validation of `PCI_D3cold` (`state 4`), ACPI `ARPT` GPE wakeup sources, cold-resume teardown elimination, and upstream kernel patch architecture.
- [Technical Report: Apple NVMe Controller (`106b:2001`) D3cold Suspend/Resume Failures & PCI Quirk Resolution](nvme-suspend-d3cold-analysis-report.md)
  - Comprehensive architectural analysis of S3 `deep` vs `D3cold` power states, NVMe controller reset timeouts on resume, empirical proof via `d3cold_allowed = 0`, and proposed PCI fixup quirk in `drivers/pci/quirks.c`.
- [Technical Report: Apple SMC (`applesmc`) HWMON API Modernization & Concurrency Refactoring](applesmc-hwmon-conversion-report.md)
  - Full architectural report on converting `applesmc` to `hwmon_device_register_with_info()`, fixing DCL concurrency races, and HWMON ABI compliance.
- [Technical Evidence Report: SPI Keyboard & Trackpad Operation Mode](spi-keyboard-touchpad-technical-evidence-report.md)
  - Empirical cross-OS (macOS `ioreg` & Windows `RWEverything` MMIO) and 820-00244-A logic board schematic evidence confirming Out-Of-Band (OOB) GPIO interrupt + PIO/FIFO operation mode (bypassing PCH LPSS-DMA).
- [Sanitized macOS IOKit Registry Dump (AppleIntelLpssSpi.log)](AppleIntelLpssSpi.log)
  - Full `ioreg -p IOService -l` dump showing native macOS driver stack (`AppleIntelLpssSpiController`, `AppleHSSPIController`, `AppleHSSPIHIDDriver`, `AppleIntelLpssDmacChannel = 0`).
