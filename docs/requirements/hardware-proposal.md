# Hardware proposal

Current document covers [requirements](requirements.md) in proposal real hardware
solutions, which meet requirements gathered earlier.

## Considered boards

### Infineon CY8C6xxxx Series

* [Infineon PSoC6 Family - web page](https://www.infineon.com/cms/en/product/microcontroller/32-bit-psoc-arm-cortex-microcontroller/psoc-6-32-bit-arm-cortex-m4-mcu/)
* CPU
  * SoC integrates 2 CPUs as part of it's structure:
    * ARM Cortex M4 (150MHz) 
    * ARM Cortex M0+ (100MHz)
* Memory:
  * Flash: 384-1856 KB
  * RAM: 176-944KB
* Interfaces I2C, SPI, UART, USB
* Voltage range: 1.71 to 5.5V

#### SPI Slave Frequency

(no information)

#### Cryptographic accelerators

* Crypto Accelerator (DES/TDES, AES, SHA, CRC, TRNG, RSA/ECC)

#### Software tools

* Software ModusToolbox: [Modus Toolbox](https://www.infineon.com/cms/en/design-support/tools/sdk/modustoolbox-software/)

### SparkFun QuickLogic Thing Plus - EOS S3

* [QuickLogic Thing Plus - EOS S3 Datasheet](https://www.sparkfun.com/products/17273)
* CPU
  * eFPGA-enabled ARM Cortex©-M4F MCU
* Memory
  * Flash: 16 Mb NOR
  * RAM: up to 512KiB
* Interfaces: FPGA-based solution
* Voltage range: 1.7 to 3.6V

#### SPI Slave Frequency

* FPGA-based solution

#### Cryptographic accelerators

* FPGA-based solution

#### Software tools

* Supports open-source FPGA synthesis software - SymbiFlow (now known as F4PGA).
  * Partial upstream support from F4PGA - lack of BlockRAM support can be a
    serious obstacle for complex FPGA designs. This is not a problem for a
    simple LPC controller.

* Downstream support for [Zephyr](https://github.com/QuickLogic-Corp/zephyr/tree/eos-s3-support)
  and FreeRTOS (as part of the [QORC SDK](https://github.com/QuickLogic-Corp/qorc-sdk))
  * Based on an old version of Zephyr. Mainline Zephyr has only the most basic
    support EOS S3 and it is not usable without porting additional drivers.

### STM32L476RG Nucleo board

* [STM32L476RG Nucleo Web Page](https://www.st.com/en/evaluation-tools/nucleo-l476rg.html)
* [STM32L476RG Nucleo Datasheet](https://www.st.com/resource/en/datasheet/stm32l476rg.pdf)
* CPU
  * ARM Cortex-M4 MCU with FPU(80MHz) with low power
* Memory
  * Flash: 1MiB
  * RAM: 128KiB
* Interfaces I2C, SPI, UART, USB
* Voltage range: 1.71 to 5.5V

#### SPI Slave Frequency

* SPI clock frequency:
  * Slave mode receiver, 1.7 < VDD < 3.6V: 40MHz
  * Full duplex/Slave mode transmit, 2.7 < VDD < 3.6V: 26MHz
  * Full duplex/Slave mode transmit, 1.71 < VDD < 3.6V: 16MHz

#### Cryptographic accelerators

* Cryptography engine

#### Software tools

* RTOS Support (e.g. Zephyr)
* STM32CubeIDE [STM32CubeIDE](https://www.st.com/en/development-tools/stm32cubeide.html)

### STM32G474RE Nucleo board

* [STM32G474 Nucleo Board Documentation](https://www.st.com/en/evaluation-tools/nucleo-g474re.html)
* CPU
  * ARM Cortex-M4 MCU (170MHz)
* Memory:
  * Flash: 512KiB
  * RAM: 96KiB
* Interfaces I2C, SPI, UART, USB
* Voltage range: 1.71 to 5.5V

#### SPI Slave Frequency

* Max SPI clock frequency:
  * Slave, receiver mode,  1.71V < Vdd < 3.6V: 50MHz
  * Slave mode transmitter, with full duplex, 2.7V < Vdd < 3.6V: 41MHz
  * Slave mode Transmitter, with full duplex, 1.7V < Vdd < 3.6V: 27MHz

#### Cryptographic accelerators

* Mathematical hardware accelerators
  * CORDIC for trigonometric functions acceleration
  * FMAC: filter mathematical accelerator

#### Software tools

* RTOS Support (e.g. Zephyr)
* STM32CubeIDE [STM32CubeIDE](https://www.st.com/en/development-tools/stm32cubeide.html)

## Requirements summary

* The goal for this task is to collect and summarize, in convenient form,
functionalities of the boards available in market, with interesting
functionalities (communication, math/crypto accelerators). This document
should aim to decide, which solution best fits the requirements.
* if feature is present/useful for our solution, then factor is +1
* if feature is not present/has limited functionality factor is 0
  * If feature is hard to implement, score is -1
* if the property could add extra feature (e.g. ease of implementation,
add flexibility in further extension of the project) then multiply weight by 2. 

|Property			|Reference val.	|Infineon CY8C6xxxx		|Thing Plus - EOS S3	|STM32L476RG Nucleo	|STM32G474RE Nucleo	|
|:------------------------------|:-------------:|:------------------------------|:----------------------|:----------------------|:----------------------|
|Supply voltage			|1.8-3.3[V]	|1.7-5.5V			|1.62-3.63		|1.71-3.6		|1.71-3.6		|
|Current  			|<250[mA]	|				|			|			|			|
|NV Memory 			|> 2 * 16[KiB]	|384-1856 KiB		(+1)	|16 Mb		(+2)	|1MiB		(+1)	|512KiB		(+1)	|
|RAM Memory 			|> 16[KiB]	|176-944KiB		(+1)	|512KiB		(+2)	|128KiB		(+1)	|96KiB		(+1)	|
|Clock Frequency 		|[MHz]		|2 cores, M4:150, M0:100  (+2)	|80		(+1)	|80		(+1)	|170		(+2)	|
|SPI Frequency 			|[MHz]		|(no data)		(0)	|(no data)	(0)	|26		(+1)	|41		(+2)	|
|OSSoftware support		|		|(no data)		(0)	|FPGA-based	(+1)	|yes		(+1)	|yes		(+2)	|
|Math/crypto funcions accel.	|		|				|FPGA-based	(+1)	|yes		(+1)	|yes		(+2)	|
|Final rating			|		|				|			|			|			|

## Excluded chips

### NXP EdgeLock(TM) SE050

* Link to documentation: [NXP_edgelock_se050](https://www.nxp.com/docs/en/white-paper/NXP_SE050_USE_CASE07_WP.pdf)
* Reason to exclude:
  * Lack of SPI, LPC interface

### Microchip ATTPM20P

* Link to documentation: [mcp_attpm20p](https://ww1.microchip.com/downloads/en/DeviceDoc/ATTPM20P-Trusted-Platform-Module-TPM-2.0-SPI-Interface-Summary-Data-Sheet-DS40002082A.pdf)
* Compliant to the Trusted Computing Group (TCG) Trusted Platform Module (TPM)
Version 2.0, r116 Trusted Platform Module Library
* Single-Chip Turnkey Solution
* Hardware Asymmetric Crypto Engine
* Microchip ARM® M0+Microprocessor
* Internal FLASH Storage for Keys
* Serial Peripheral Interface (SPI) Protocol up to 36 MHz
* Secure Hardware and Firmware Design and Device Layout
* FIPS-140-2 Module Compliant Including the High-Quality Random Number
Generator (RNG), HMAC, AES, SHA, ECC, and RSA Engines

* Offered in Commercial (0°C to +70°C) Temperature Range
  * Supply Voltage: 1.8V to 3.3V
* Offered in Industrial (-40°C to +85°C) Temperature Range
  * Supply Voltage: 3.3V
* Cryptographic Support for:
  * HMAC
  * AES-128
  * SHA-1
  * SHA-256
  * ECC BN_P256, ECCNIST_P256
  * RSA 1024-2048 bit keys
* 16 KB of User-Accessible Nonvolatile Memory
* X.509 EK Certificates (Optional)

* Reason to exclude:
  * This chip does not meet the requirement of open-source hardware/software.

### ST33TPHF20SPI

* Link to documentation: [ST33TPHF20SPI Documentation](https://www.st.com/en/secure-mcus/st33tphf20spi.html)
* Compliant with Trusted Computing Group (TCG) Trusted Platform Module (TPM)
Library specifications 2.0, Level 0, Revision 138 and TCG PC Client Specific
TPM Platform Specifications 1.03
* SPI support for up to 33 MHz in FIFO and CRB protocol modes
* Arm® SecurCore® SC300™ 32-bit RISC core
* 1.8 V or 3.3 V supply voltage range
* 28-lead thin shrink small outline and 32-lead very thin fine pitch quad flat
pack ECOPACK packages

* Reason to exclude:
  * This chip does note meet the requirement of open-source hardware/software
