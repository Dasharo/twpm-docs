<!--
SPDX-FileCopyrightText: 2024 3mdeb <contact@3mdeb.com>

SPDX-License-Identifier: CC-BY-SA-4.0
-->

# Hardware requirements for the TwPM module

## Intro

The goal of this document is to list the most important requirements, which
must be met by the TwPM module to maintain the minimum level of compatibility
with the
[TCG PC Client Platform TPM Profile Specification for TPM 2.0][tcg_client_PTP_v1p5r14],
 and the available mainboards. These requirements will guide us during the
hardware selection process, so we can select a development board, that will
allow us to implement the minimum viable PoC (and later, hopefully, the
complete solution as well).

## Power supply

### Voltage

* TPM MUST support a supply voltage of 1.8V or 3.3V.
* TPM MAY support supply and I/O voltages of both 1.8V and 3.3V.
* TPM MAY support other supply voltages.

Source: [TCG PC Client Specific TPM Interface Specification], section `6.4.2
Electrical Specification`.

### Maximum current

* Maximum power supply taken by TPM MUST be 250mA.

Source:
[TCG PC Client Platform TPM Profile Specification for TPM 2.0][tcg_client_PTP_v1p5r14],
`Table 60`.

## Memory

### NV memory requirements from TPM specification

* The TPM SHALL be capable of supporting a minimum of 100KB written
  to TPM non-volatile memory over the life of the TPM.
* The TPM shall support allocation of at least 68 indices, with a total
  minimum data size of 3834 Bytes (decimal).
* The TPM specification does not define the required lifetime or endurance of
  the NV memory.

Additional memory requirements, for program and data:

* At least 2 blocks, 16KiB each for NV data (plus application)
* At least 1 block, 16KiB RAM (plus RAM used by the application)

Source:
[TCG PC Client Platform TPM Profile Specification for TPM 2.0][tcg_client_PTP_v1p5r14],
section `4.5.1 NV Storage Requirements`.

### Firmware stack requirements

Based on the experiments with the
[ms-tpm-20-ref](https://github.com/microsoft/ms-tpm-20-ref) up to date, we
estimate the reasonable minimal requirements to be:

* 1 MB of flash,
* 128 KB of RAM.

During the early stages of the project, we prefer to focus on features, not
fighting with memory optimization.

## SPI interface

The TwPM is going to be an SPI slave device and must comply with the SPI clock
frequency offered by the SPI host on the mainboard. We can look at the required
SPI frequency from two angles:

* what TPM specification requires in this area,
* what SPI host frequencies are offered by the existing mainboards (chipsets).

### Frequency (TPM specification)

TPM must be capable of operating at frequencies 10-24 MHz. However, the maximum
frequency requirement may be raised in the future, so the target platform
should have the SPI interface with the capability to handle higher frequencies.

Source: [TCG PC Client Platform TPM Profile Specification for TPM 2.0][tcg_client_PTP_v1p5r14],
section `6.4.1 Clocking`.

### Frequency (mainboards/chipsets)

The PCH’s `SPI0` flash controller supports a discrete TPM on the platform
via its dedicated `SPI0_CS2#` signal. The platform must have no more than 1 TPM.

`SPI0` controller supports accesses to the `SPI0` TPM at approximately
`17 MHz`, `33 MHz`, and `48 MHz` depending on the PCH soft strap. `20 MHz` is the
reset default, and a valid PCH soft strap setting can override that.

Based on the above, the device must support an SPI clock of at least `20MHz`
(preferably more - `33 MHz`, and `48 MHz`).

Source:

* [Intel® 300 Series Chipset Family On-Package Platform Controller Hub (PCH) Datasheet, Volume 1 of 2](https://www.intel.com/content/www/us/en/products/docs/chipsets/300-series-chipset-on-package-pch-datasheet-vol-1.html)
* [Intel® 600 Series Chipset Family Platform Controller Hub Datasheet, Volume 1 of 2](https://edc.intel.com/content/www/us/en/design/ipla/software-development-platforms/client/platforms/alder-lake-desktop/intel-600-series-chipset-family-platform-controller-hub-pch-datasheet-volume/004/spi0-support-for-tpm/)

## LPC interface

### Interrupts

* Service of SIRQ protocol for interrupts
* Implementation: emulate the set of individual hardware signals using time
  division multiplexing between frames

Source: [TCG PC Client Platform TPM Profile Specification for TPM 2.0][tcg_client_PTP_v1p5r14],
section `6.6.1 LPC Interrupts`.

## Timings

### Command Duration

* Duration and Timeouts for some commands are specified in `Table 17 -
  Command Timing`
* Some examples of timing/duration [ms]:
    - `TPM2_Startup`: 20/750
    - `TPM2_GetRandom`: 750/2000
    - `TPM2_PCR_Extend`: 20/750

Source: [TCG PC Client Platform TPM Profile Specification for TPM 2.0][tcg_client_PTP_v1p5r14],
section `6.5.1.3 Command Duration`.

### Timeouts

Default values timeouts:

* TIMEOUT_A - default value 750ms
* TIMEOUT_B - default value 2s
* TIMEOUT_C - default value 750ms
* TIMEOUT_D - default value 750ms

Source: [TCG PC Client Platform TPM Profile Specification for TPM 2.0][tcg_client_PTP_v1p5r14],
section `6.5.1.4 Interface Timeouts`.

### Reset timings

* Within 500 microseconds of the completion of `TPM_Init`:
    - all fields within all `TPM_ACCESS_x` registers must be valid logical level
    as indicated by the `tpmRegValidSts` field;
* Within 30 milliseconds of the completion of `TPM_Init`:
    - all fields within the access register and all other registers must return
    with the state of their fields valid
    - the TPM must be ready to receive a command

Source: [TCG PC Client Platform TPM Profile Specification for TPM 2.0][tcg_client_PTP_v1p5r14],
section `7.6 Reset Timing`.

## References

* [TCG PC Client Platform TPM Profile Specification for TPM 2.0][tcg_client_PTP_v1p5r14],
    - Version: 1.05
    - Revision: 14
    - Date: September 4, 2020

## Summary

* Power supply requirements are quite important:
    - the voltage one can be fulfilled by adding another converter on the
      board, if necessary,
    - the current consumption will be very important in the final product (so
      the module can work powered from the mainboard), but in the PoC phase, we
      may work around that.
* The timings may be important later on for some performance testing, but it is
  very difficult to judge at this stage of the project, whether the given hardware
  can achieve such metrics. We will not use them as input for the initial
  hardware selection.
* Requirements on flash and RAM are mostly driven by the firmware stack
  requirements, as the requirements from the TPM specification are negligible in
  comparison to that.
* The most important requirements are SPI and LPC interfaces, as without these,
  the module would not be able to communicate with the mainboard properly, so
  even a functional PoC cannot be constructed.

[tcg_client_PTP_v1p5r14]:<https://trustedcomputinggroup.org/wp-content/uploads/PC-Client-Specific-Platform-TPM-Profile-for-TPM-2p0-v1p05p_r14_pub.pdf>
