# Hardware proposal

Current document covers requirements in proposal real hardware solutions,
which meet requirements gathered earlier.

## Microchip ATTPM20P

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

## Infineon CY8C6xxxx Series

* Link to documentation: [Infineon PSoC6 Family]<https://www.infineon.com/cms/en/product/microcontroller/32-bit-psoc-arm-cortex-microcontroller/psoc-6-32-bit-arm-cortex-m4-mcu/>
* Point of interest: PSoC 64-Secured MCU series
* 32-bit Programmable System-on-Chip, integrates in one structure
ARM Cortex M4 (150MHz) with Cortex M0+, 100MHz
* Various variants of memories:
  * flash 384-1856 KB
  * RAM 176-944KB
* Interfaces I2C, SPI, UART, USB
* Crypto Accelerator (DES/TDES, AES, SHA, CRC, TRNG, RSA/ECC)
* Software ModusToolbox: [Modus Toolbox]<https://www.infineon.com/cms/en/design-support/tools/sdk/modustoolbox-software/>

## SparkFun QuickLogic Thing Plus - EOS S3

* Link to documentation: <https://www.sparkfun.com/products/17273>
* eFPGA-enabled ARM  Cortex©-M4F MCU
* Fully supported with Zephyr RTOS, FreeRTOS
* Based fully on open source hardware
* Software environment is round 100% open source
(with support for FreeRTOS, Zephyr, nMingen, Docker, and SymbiFlow)

## STM32L476RG Nucleo board

* Link to documentation: <https://www.st.com/en/evaluation-tools/nucleo-l476rg.html>
* Link to datasheet: <https://www.st.com/resource/en/datasheet/stm32l476rg.pdf>
* Ultra low power ARM Cortex-M4 MCU with FPU
* Power supply 1.71-3.6V
* 80MHz system clock
* 3xSPI interfaces
* Cryptography engine
* SPI interface frequency, below citation from datasheet:
"Three SPI interfaces allow communication (...) up to 24 Mbits/s
slave modes".

## STM32G474 Nucleo board

* Link to documentation: <https://www.st.com/en/evaluation-tools/nucleo-g474re.html>
* ARM Cortex-M4 MCU
* Power supply 1.71-3.6V
* 170MHz system clock
* 4xSPI interfaces, 4-16 programmable bit frames
* Mathematical hardware accelerators
  * CORDIC for trigonometric functions acceleration
  * FMAC: filter mathematical accelerator

## ST33TPHF20SPI

* Link to documentation: <https://www.st.com/en/secure-mcus/st33tphf20spi.html>
* Compliant with Trusted Computing Group (TCG) Trusted Platform Module (TPM)
Library specifications 2.0, Level 0, Revision 138 and TCG PC Client Specific
TPM Platform Specifications 1.03
* SPI support for up to 33 MHz in FIFO and CRB protocol modes
* Arm® SecurCore® SC300™ 32-bit RISC core
* 1.8 V or 3.3 V supply voltage range
* 28-lead thin shrink small outline and 32-lead very thin fine pitch quad flat
pack ECOPACK packages

Excluded chips

## NXP EdgeLock(TM) SE050

* Link to documentation: [NXP_edgelock_se050](https://www.nxp.com/docs/en/white-paper/NXP_SE050_USE_CASE07_WP.pdf)
* Reason to exclude:
  * I2C Bus
