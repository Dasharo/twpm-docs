# Hardware requirements of TwPM module

[tcg_client_v13r27]:<https://trustedcomputinggroup.org/wp-content/uploads/TCG_PCClientTPMInterfaceSpecification_TIS__1-3_27_03212013.pdf>
[tcg_client_PTP_v1p5r14]:<https://trustedcomputinggroup.org/wp-content/uploads/PC-Client-Specific-Platform-TPM-Profile-for-TPM-2p0-v1p05p_r14_pub.pdf>
[intel_300_series_pch_ds_1of2]:<https://www.intel.com/content/www/us/en/products/docs/chipsets/300-series-chipset-on-package-pch-datasheet-vol-1.html>
[intel_600_series_pch_ds_1of2]:<https://edc.intel.com/content/www/us/en/design/ipla/software-development-platforms/client/platforms/alder-lake-desktop/intel-600-series-chipset-family-platform-controller-hub-pch-datasheet-volume/004/spi0-support-for-tpm/>
## Requirements

### Power supply

#### Power supply voltage

* TPM MUST support a supply voltage of 1.8V or 3.3V.
* TPM MAY support supply and I/O voltages of both 1.8V and 3.3V.
* TPM MAY support other supply voltages.

Source: [tcg_client_v13r27] lines 1373-1375

#### Power supply max current

Maximum power supply taken by TPM MUST be 250mA.

Source: [tcg_client_PTP_v1p5r14], Table 60, VDD row

### Memory requirements

1. The TPM SHALL be capable of supporting a minimum of 100KB written
to TPM non-volatile memory over the life of the TPM.
2. The TPM shall support allocation of at least 68 indices, with a total
minimum data size of 3834 Bytes(decimal).

The TPM specification does not define the required lifetime or endurance
of the NV memory.
Source: [tcg_client_PTP_v1p5r14], Chapter 4.5.1

Additional memory requirements, for program and data:
* at least 2 block, 16KiB each for NV data (plus application)
* at least 1 block, 16KiB RAM (plus RAM used by application)

### SPI Interface

#### Frequency

TPM must be capable of operating at frequencies 10-24MHz. However, the maximum
frequency requirement may be raised in the future, so the target platform
should have the SPI interface with capability to handle higher frequencies.

Source: [tcg_client_v13r27] Chapter 6.4.1, line 1331

#### Frequency supported by PCH

This requirement is crucial from the point of view of the environment,
which the TwPM device will work.

The PCH’s SPI0 flash controller supports a discrete TPM on the platform
via its dedicated SPI0_CS2# signal. The platform must have no more than 1 TPM.

SPI0 controller supports accesses to SPI0 TPM at approximately 17 MHz,
33 MHz and 48 MHz depending on the PCH soft strap. 20 MHz is the reset default,
a valid PCH soft strap setting overrides the requirement for the 20 MHz. 
SPI0 TPM device must support a clock of 20 MHz, and thus should handle
15-20 MHz. It may but is not required to support a frequency
greater than 20 MHz.

Source: [intel_300_series_pch_ds_1of2], [intel_600_series_pch_ds_1of2]

#### Power sequence

* TPM_Init (LRESET#/SPI_RST#) signal connected to platform CPU Reset signal
* TPM main power pins (3V) must be connected such that the TPM is powered
during ACPI states S0-S2 and may be powered in S3-S5

Source: [tcg_client_v13r27], Chapter 6.7.2

### LPC Interface

#### LPC Interrupts

* Service of SIRQ protocol for interrupts
* Implementation: emulate the set of individual hardware signals using time
division multiplexing between frames
Source: [tcg_client_PTP_v1p5r14], chapter 6.6.1

### Run-time requirements

#### Command Duration

Command duration MUST be less than 750ms.

Source: [tcg_client_v13r27], chapter 5.6.5

#### Timeout

Four kind of timeouts:

* TIMEOUT_A - default value 750ms
* TIMEOUT_B - default value 2s
* TIMEOUT_C - default value 750ms
* TIMEOUT_D - default value 750ms

Source: [tcg_client_v13r27], chapter 5.6.6

#### Reset timing

* Within 500 microseconds of the completion of TPM_Init:
 all fields within all TPM_ACCESS_x registers must be valid logical level
 as indicated by the tpmRegValidSts field;

* Within 30 milliseconds of the completion of TPM_Init:
  all fields within the access register and all other registers MUST return
  with the state of their fields valid.

* The TPM MUST be ready to receive a command in 30ms

Source: [tcg_client_v13r27], chapter 6.6

## TCG PC Client Platform Physical Presence Interface Specification

* link: [TCG PC Client Platform Physical Presence Interface Specification](https://trustedcomputinggroup.org/wp-content/uploads/Physical-Presence-Interface_1-30_0-52.pdf)
* Version 1.30 Revision 0.0.52
* ACPI Functions

## Bibliography

* [Trusted Computing Group, PC Client specific Platform, TPM Profile](https://trustedcomputinggroup.org/wp-content/uploads/PC-Client-Specific-Platform-TPM-Profile-for-TPM-2p0-v1p05p_r14_pub.pdf)

* [Trusted Computing Group, collection of documents, abbreviations, acronyms](https://trustedcomputinggroup.org/work-groups/pc-client/)
