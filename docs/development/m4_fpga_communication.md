# Communication between M4 and FPGA

This document describes how FPGA and M4 communicate with each other in TwPM
project for EOS-S3. State presented here is valid for revision **TBD** of top
module.

## Memory map

| Address    | Size | Name     | Access |
|------------|------|----------|--------|
| 0x40020000 | 4B   | STATUS   | RO     |
| 0x40020004 | 4B   | OP_TYPE  | RO     |
| 0x40020008 | 4B   | LOCALITY | RO     |
| 0x4002000C | 4B   | BUF_SIZE | RO     |
| 0x40020040 | 4B   | COMPLETE | WO     |
| 0x40020800 | 2KiB | FPGA_RAM | RW     |

Reads from invalid addresses return `0xBADFABAC` (BAD FABric ACcess), writes are
dropped. Most values are valid only if `exec` bit in `STATUS` register is set.

## Registers

Reserved bits are read as 0. They may change in the future.

### STATUS

![Layout of STATUS register](/images/reg-status.png)

This register is expected to be informational only, it doesn't provide any
information that isn't conveyed through other means (interrupts).

* `exec` indicates that MCU is expected to act upon request sent from FPGA,
  specified in `OP_TYPE` register. [Interrupt](#interrupts) is generated when
  this signal gets set by FPGA.
* `abort` is set when PC aborts command currently being executed by TPM.
  [Interrupt](#interrupts) is generated when this signal gets set by FPGA.
* `complete` is a signal sent towards FPGA by writing to `COMPLETE` register.
  The signal is active for a few cycles after write to that register, after that
  is should be automatically set back to 0.

### OP_TYPE

![Layout of OP_TYPE register](/images/reg-op-type.png)

* `op_type` is a type of operation expected from MCU. 0 is used as a default
  value to which this register returns after `complete` signal is acknowledged
  by FPGA. 0xC is reserved as it may be part of 0xBADFABAC magic value. This
  register is only valid if `exec` is set.

| op_type | Operation                                  |
|--------:|--------------------------------------------|
| 0x0     | No operation                               |
| 0x1     | Execute TPM2 command located in `FPGA_RAM` |
| 0xC     | Illegal operation type                     |
| other   | Reserved for future use                    |

### LOCALITY

![Layout of LOCALITY register](/images/reg-locality.png)

* `locality` specifies the locality at which the current operation was
  initiated. Value 0xF is used when no locality was active at that point. This
  field is only valid if `exec` is set.

### BUF_SIZE

![Layout of BUF_SIZE register](/images/reg-buf-size.png)

* `buf_size` specifies size (in bytes) of data available in FPGA RAM, which is
  usually a TPM2 command. This field is only valid if `exec` is set. TPM stack
  is not expected to write size of response to this field, it is deduced from
  response's payload.

### COMPLETE

Any write to this register triggers `complete` signal to be sent to FPGA. Reads
from this register aren't implemented, meaning that 0xBADFABAC is returned.

## FPGA RAM

This region behaves like RAM if `exec` is set. Its content depends on `op_type`,
which in most cases will be TPM2 command. Response is written by MCU to this
buffer, starting from offset 0, overwriting the command.

When `exec` is not set, data read from this region is invalid and writes are
dropped.

## Interrupts

TwPM uses two interrupt signals from FPGA towards M4. First one is `exec` which
signals that there is an operation to be performed by TPM stack, and another one
tells that the currently running operation is to be aborted.

Both interrupts arrive to M4 as IRQ 4, and exact reason for interrupt can be
read from 4-byte `FB_INTR_RAW` NVIC register located at address 0x40004884. Bits
0 and 1 represent `exec` and `abort`, respectively. Interrupts are cleared by
writing the same bits to register `FB_INTR` located at 0x40004880.
