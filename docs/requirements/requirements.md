# Hardware requirements of TwPM module

[tcg_client_v13r27]:<https://trustedcomputinggroup.org/wp-content/uploads/TCG_PCClientTPMInterfaceSpecification_TIS__1-3_27_03212013.pdf>
[tcg_client_PTP_v1p5r14]:<https://trustedcomputinggroup.org/wp-content/uploads/PC-Client-Specific-Platform-TPM-Profile-for-TPM-2p0-v1p05p_r14_pub.pdf>

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

### SPI Interface

#### Frequency

Default clock frequency for PC Client platforms is defined to be 24MHz.
Clock frequency signal, minimum, is 10MHz.
Every platform, with higher frequencies is recommended.

Source: [tcg_client_v13r27] Chapter 6.4.1, line 1331

### Voltage levels

1. TPM MUST support a supply and I/O voltage of 1.8V or 3.3V
1. TPM MAY support supply and I/O voltages of both 1.8 and 3.3V.
1. TPM MAY support other supply and I/O voltages.

Source: [tcg_client_v13r27] lines 1373-1375

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
