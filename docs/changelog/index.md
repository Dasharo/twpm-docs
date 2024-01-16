# Changelog

## 2024-01-16

* Added page about [running tests](../development/testing.md)
* Published
  [tests results as part of Task 6. Base tests](../test-results/2024_01_11_orange_crab_without_create_primary.html)
* Small changes to
  [description of interrupts handling](../development/soc_fpga_communication.md#interrupts)

## 2023-11-26

* [Reduce numbers of bi-directional lines to minimum](../development/verilog_modules/)
  (Yosys doesn't like them)
* Switch to OrangeCrab
    - [Reason](../explanation/hardware-selection/#update-october-2023)
    - [Updated building instructions](../tutorials/building/) - actual
      instructions moved to README in top level repository to avoid duplications
    - [Modified memory map](../development/soc_fpga_communication/)
    - [New FPGA utilization numbers](../development/verilog_modules/#top-level)
* [Instructions for connecting TwPM to mainboard - Protectli VP46xx](../tutorials/mainboard-connection/)
* [Description of new modules](../development/verilog_modules/) (NEORV32,
  LiteDRAM, TPM buffer)

## 2023-07-31

### Added

* [Blog post on optimizing SPI communitation on STM32](https://blog.3mdeb.com/2023/2023-07-28-optimizing-spi-on-stm32/)
* Information on maximum real SPI frequency on STM32 in the [hardware selection section](../explanation/hardware-selection/#stm32l476rg-nucleo-board)

## 2023-05-24

### Added

* [New Verilog Module specification in Development section](../development/verilog_modules/#tpm-registers-module),
  added graphics and links to modules' code
* Fixed broken links in changelog (here) and [Development section](../development)
* [First implementation details documented](../explanation/compliance/)
* [Update statuses on the roadmap](../roadmap/)

## 2023-04-20

### Added

* [Hardware requirements in Explanation section](../explanation/)
* [Hardware selection in Explanation section](../explanation/)
* [Building TwPM in Tutorials section](../tutorials/building/)
* [Verilog Modules specification in Development section](../development/verilog_modules/)
