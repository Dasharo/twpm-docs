# Hardware selection

## Intro

The goal of this document is to select a target hardware (development board),
which will be used for the development and prototyping of the TwPM. We have
pre-selected some of the boards based on our best knowledge, and some
suggestions from the community (e.g. during and after our
[presentation at FOSDEM 2023](https://fosdem.org/2023/schedule/event/twpm_osf_tpm/)).

We aim to evaluate whether the pre-selected boards can meet the [identified
hardware requirements](hardware-requirements.md).

## Infineon CY8C6xxxx Series

### Basic information

* SoC
    - SoC integrates 2 CPUs as part of its structure:
        * ARM Cortex M4 (150MHz)
        * ARM Cortex M0+ (100MHz)
* Memory:
    - Flash: 384-1856 KB
    - RAM: 176-944KB
* Interfaces I2C, SPI, UART, USB
* Voltage range: 1.71 to 5.5V

### SPI slave frequency

* 25 MHz

### Cryptographic accelerators

* DES/TDES, AES, SHA, CRC, TRNG, RSA/ECC

### Tools and documentation

* [Modus Toolbox](https://www.infineon.com/cms/en/design-support/tools/sdk/modustoolbox-software/)
* [SoC datasheet](https://www.infineon.com/cms/en/product/microcontroller/32-bit-psoc-arm-cortex-microcontroller/psoc-6-32-bit-arm-cortex-m4-mcu/)

## SparkFun QuickLogic Thing Plus - EOS S3

* SoC
    - eFPGA-enabled ARM Cortex©-M4F MCU
* Memory
    - Flash: 2 MB NOR
    - RAM: up to 512 KB
* Interfaces: FPGA-based solution
* Voltage range: 1.7 to 3.6V

### SPI slave frequency

* Up to 20 MHz in slave mode

### Cryptographic accelerators

* Can be implemented in FPGA

### Tools and documentation

* Supports open-source FPGA synthesis software - SymbiFlow (now known as
  F4PGA).
    - Partial upstream support from F4PGA
    - [lack of BlockRAM support](https://github.com/YosysHQ/apicula/pull/45/files)
      can be a serious obstacle for complex FPGA designs, but should not be a
      problem for a simple LPC controller
* Downstream support for [Zephyr](https://github.com/QuickLogic-Corp/zephyr/tree/eos-s3-support)
  and FreeRTOS (as part of the [QORC SDK](https://github.com/QuickLogic-Corp/qorc-sdk))
    - Based on an old version of Zephyr
    - [Mainline Zephyr](https://github.com/zephyrproject-rtos/zephyr/tree/main/boards/arm/quick_feather)
      supports the most basic peripherals, and may be generally not usable without
      porting additional drivers
* [Board webpage](https://www.quicklogic.com/products/eos-s3/sparkfun-thing-plus/)
* [SoC datasheet](https://www.quicklogic.com/wp-content/uploads/2020/06/QL-EOS-S3-Ultra-Low-Power-multicore-MCU-Datasheet.pdf)

## STM32L476RG Nucleo board

* CPU
    - ARM Cortex-M4 MCU with FPU (80MHz) with low power
* Memory
    - Flash: 1 MB
    - RAM: 128 KB
* Interfaces I2C, SPI, UART, USB
* Voltage range: 1.71 to 5.5V

### SPI slave frequency

> In reality, the maximum SPI slave frequency achieved on this board so far was
> 24 MHz and it worked reliably when writing only, not while reading. More
> details can be found in [this
> blogpost](https://blog.3mdeb.com/2023/2023-07-28-optimizing-spi-on-stm32/).

* SPI clock frequency:
    - Slave mode receiver, 1.7 < VDD < 3.6V: 40MHz
    - Full duplex/Slave mode transmit, 2.7 < VDD < 3.6V: 26MHz
    - Full duplex/Slave mode transmit, 1.71 < VDD < 3.6V: 16MHz

### Cryptographic accelerators

* True RNG
* Some (but not the one installed on the Nucleo board by default) STM32 chips
  from this family offer also AES-128, AES-256 hardware encryption

### Tools and documentation

* Good open-source RTOS Support (e.g. [Zephyr](https://docs.zephyrproject.org/3.2.0/boards/arm/nucleo_l476rg/doc/index.html))
* [STM32CubeIDE](https://www.st.com/en/development-tools/stm32cubeide.html)
* [STM32L476RG Nucleo Web Page](https://www.st.com/en/evaluation-tools/nucleo-l476rg.html)
* [STM32L476RG Nucleo Datasheet](https://www.st.com/resource/en/datasheet/stm32l476rg.pdf)

## STM32G474RE Nucleo board

* CPU
    - ARM Cortex-M4 MCU (170MHz)
* Memory:
    - Flash: 512 KB
    - RAM: 128 KB (32 KB is used for parity checks, so only 96 KB is available
      for program usage)
* Interfaces I2C, SPI, UART, USB
* Voltage range: 1.71 to 5.5V

### SPI slave frequency

* Max SPI clock frequency:
    - Slave, receiver mode,  1.71V < Vdd < 3.6V: 50MHz
    - Slave mode transmitter, with full duplex, 2.7V < Vdd < 3.6V: 41MHz
    - Slave mode Transmitter, with full duplex, 1.7V < Vdd < 3.6V: 27MHz

### Cryptographic accelerators

* True RNG
* Some (but not the one installed on the Nucleo board by default) STM32 chips
  from this family offer also AES-128, AES-256 hardware encryption
* Mathematical hardware accelerators
    - CORDIC for trigonometric functions acceleration
    - FMAC: filter mathematical accelerator
    - these are not that useful for the TwPM use-case

### Tools and documentation

* Good open-source RTOS Support (e.g. [Zephyr](https://docs.zephyrproject.org/2.7.4/boards/arm/nucleo_g474re/doc/index.html)
* [STM32CubeIDE](https://www.st.com/en/development-tools/stm32cubeide.html)
* [STM32G474 Nucleo Web Page](https://www.st.com/en/evaluation-tools/nucleo-g474re.html)
* [STM32L476RG Nucleo Datasheet](https://www.st.com/resource/en/datasheet/stm32g474re.pdf)

## Scoring methodology

* Following methodology is applied to assign grades in the below table:
    - if the desired value is met, then `+1`,
    - if the desired value is not met, then `0`,
    - if the desired value is not present and would be difficult to workaround (or
      there is no data about the given property), then `-1`,
    - if this is a critical requirement, then multiply the result by `2`
      (critical requirements are marked **in bold** in the table below).

## Summary

* The above report outlines 4 development boards, that were evaluated more
  closely, after the initial research.
* The supply voltage was not a challenging requirement and is similar across
  devices.
* The actual data on current consumption is not clear at the moment. It might
  also highly depend on the workflow, and enabled peripherals. To indicate that
  risk, `-1` grade was added, where applicable. It will need actual
  measurements in the future.
* All of the boards should meet the RAM and CPU requirements (although that was
  was set to an arbitrary value right now, and is subject to change in the future).
* The most flexible choice would be the board with the `EOS S3` SoC, due to the
  significant RAM and flash size, and integrated FPGA, which allows us to
  implement LPC interface, and potentially also offload other features (SPI,
  crypto), as needed
* If we would want to implement a TwPM module with SPI interface only, we could
  try to go with either `STM32L476RG`, or `STM32G474RE`. The first one has a bit
  more resources (flash, and RAM), but the second one features much higher CPU
  and SPI speeds, which makes it an interesting target for the future.

| Property            | Desired value       | CY8C6xxxx              | EOS S3                  | STM32L476RG      | STM32G474RE        |
|:--------------------|:--------------------|:-----------------------|:------------------------|:-----------------|:-------------------|
| Supply voltage      | 1.8-3.3 V (+1)      | 1.7-5.5 V (+1)         | 1.62-3.63 V (+1)        | 1.71-3.6 V (+1)  | 1.71-3.6 (+1)      |
| Current             | < 250 mA            | (no data) (-1)         | (no data) (-1)          | (no data) (-1)   | (no data) (-1)     |
| Flash               | >= 1 MB             | 384-1856 KB (+1)       | 2 MB (+1)               | 1 MB (+1)        | 512 KB (0)         |
| **RAM**             | >= 128 KB           | 176-944KB (+2)         | 512 KB (+2)             | 128 KB (+2)      | 128 KB [^8] (0)    |
| Clock Frequency     | >= 80 MHz           | 150 MHz (+1)           | 80 MHz (+1)             | 80 (+1)          | 170 (+1)           |
| **LPC interface**   | yes                 | no [^5] (0)            | no [^6] (0)             | no (-2)          | no (-2)            |
| **SPI Frequency**   | >= 20 MHz           | 25 MHz (+2)            | 20 (+2)                 | 26 (+1)          | 41 (+2)            |
| **OSS support**     | Fully OSS toolchain | limited (-2) [^1]      | yes, limited (+1) [^3]  | yes (+2)         | yes (+2)           |
| Crypto Acceleration | RNG/hash/enc        | AES/SHA/RNG (+1) [^2]  | can use FPGA (+1) [^4]  | RNG (+1) [^7]    | RNG (+1) [^7]      |
| Score               |                     | 5                      | 8                       | 6                | 4                  |

[^1]: It looks like the usage of proprietary tools is required, especially for the
      programmable logic part
[^2]: Score is increased from `+1` o `+2` due to the support of additional
      hash/encryption algorithms
<!-- when "-" is in path, the link in mkdocs is generated with a single
character, not three -->
<!-- markdownlint-disable MD051 -->
[^3]: Support in the upstream project might miss some crucial drivers, as described
      in the [EOS S3 section](#sparkfun-quicklogic-thing-plus-eos-s3)
<!-- markdownlint-enable MD051 -->
[^4]: The SoC does not provide crypto accelerators, but we could potentially
      use FPGA to offload some operations
[^5]: Potentially could be implemented in programmable logic (not a "standard"
      FPGA)
[^6]: Can be implemented in FPGA via LPC module, for example via Verilog
[^7]: Some STM32 chips from this family offer also AES-128, AES-256 hardware encryption
[^8]: 32 KB is used for parity checks, so only 96 KB is available for program usage
