# MacBook8-1

Linux enablement on MacBook8,1 (aka MacBook Retina, 12-inch, Early 2015)

## Documentation & Reports

- [Technical Report: Apple SMC (`applesmc`) HWMON API Modernization & Concurrency Refactoring](applesmc-hwmon-conversion-report.md)
  - Full architectural report on converting `applesmc` to `hwmon_device_register_with_info()`, fixing DCL concurrency races, and HWMON ABI compliance.
- [Technical Evidence Report: SPI Keyboard & Trackpad Operation Mode](spi-keyboard-touchpad-technical-evidence-report.md)
  - Empirical cross-OS (macOS & Windows MMIO) and 820-00244-A logic board schematic evidence confirming Out-Of-Band (OOB) GPIO interrupt + PIO/FIFO operation mode (bypassing PCH LPSS-DMA).
