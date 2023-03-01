# Hardware requirements of TwPM module

## Acronyms

* Legend:
* PTP – Platform TPM Profile
* CRB – Command Response Buffer interface
* DDWG – Device Driver’s Writers Guide
* Certification PP – Certification Protection Profile
* TIS – TPM Interface Specification

## Requirements

### Power supply

#### Power supply voltage

1. TPM MUST support a supply voltage of 1.8V or 3.3V.
2. TPM MAY support supply and I/O voltages of both 1.8V and 3.3V.
3. TPM MAY support other supply voltages.

Source: [tcg_client_v13r27] l.1373-1375

#### Power supply max current

Maximum power supply taken by TPM MUST be 250mA.

### SPI Interface

#### Frequency

Default clock frequency for PC Client platforms is defined to be 24MHz.
Clock frequency signal, minimum, is 10MHz.

Source: [tcg_client_v13r27] Chapter 6.4.1, line 1331

#### Voltage levels

1. TPM MUST support a supply and I/O voltage of 1.8V or 3.3V
2. TPM MAY support supply and I/O voltages of both 1.8 and 3.3V.
3. TPM MAY support other supply and I/O voltages. 

Source: [tcg_client_v13r27] l.1373-1375

#### Power supply


#### Power sequence

  * TPM_Init (LRESET#/SPI_RST#) signal connected to platform CPU Reset signal
  * TPM main power pins (3V) must be connected such that the TPM is powered
  during ACPI states S0-S2 and may be powered in S3-S5
  * Optional implementation VBAT and/or 3VSB pins

Source: [tcg_client_v13r27], Chapter 6.7.2

#### Command Duration

Command duration MUST be less than 750ms.

Source: [tck_client_v13r27], chapter 5.6.5

#### Timeout

Four kind of timeouts:
 * TIMEOUT_A - default value 750ms
 * TIMEOUT_B - default value 2s
 * TIMEOUT_C - default value 750ms
 * TIMEOUT_D - default value 750ms

Source: [tck_client_v13r27], chapter 5.6.6

#### Reset timing
 
* Within 500 microseconds of the completion of TPM_Init:
 all fields within all TPM_ACCESS_x registers must be valid logical level
 as indicated by the tpmRegValidSts field;

* Within 30 milliseconds of the completion of TPM_Init:
 all fields within the access register and all other registers MUST return
with the state of their fields valid.

* The TPM MUST be ready to receive a command in 30ms

Source: [tcg_client_v13r27], chapter 6.6 


[TCG PC Client, v1.3, rev 27][tcg_client_v13r27]: https://trustedcomputinggroup.org/wp-content/uploads/TCG_PCClientTPMInterfaceSpecification_TIS__1-3_27_03212013.pdf
[TCG TPM I2C Interface Specification][tcg_i2cspec_v200rev1]https://trustedcomputinggroup.org/wp-content/uploads/TCG-TPM-I2C-Interface-Specification-v1.00.pdf


# TCG PC Client Platform Physical Presence Interface Specification
* link: https://trustedcomputinggroup.org/wp-content/uploads/Physical-Presence-Interface_1-30_0-52.pdf
* Version 1.30 Revision 0.0.52
* ACPI Functions
# Bibliography
* Trusted Computing Group, PC Client specific Platform, TPM Profile
https://trustedcomputinggroup.org/wp-content/uploads/PC-Client-Specific-Platform-TPM-Profile-for-TPM-2p0-v1p04_r0p37_pub-1.pdf

* Trusted Computing Group, collection of documents, abbreviations, acronyms
https://trustedcomputinggroup.org/work-groups/pc-client/


