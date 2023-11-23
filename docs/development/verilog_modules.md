# Verilog modules

Below is description of FPGA modules used by TwPM for EOS-S3. The design may
change as the project progresses, state described here is valid for revision
**TBD**
of top module.

## Top level

Top level module code is located [in this repository](https://github.com/Dasharo/TwPM_toplevel).
It glues together other modules, which are referenced by that repository as git
submodules. It also describes how the signals are connected to external IO pins.

Current FPGA utilization:

```text
**TBD**
```

## LPC module

Source code: [Dasharo/verilog-lpc-module](https://github.com/Dasharo/verilog-lpc-module)

This module is responsible for managing LPC communication. It responds only to
TPM cycles, other cycle types are ignored. SERIRQ (both continuous and quiet
mode), cycle aborts and LPC resets are implemented.

![LPC peripheral module](/images/lpc_periph.svg)

Ports for LPC interface (refer to [LPC specification](https://www.intel.com/content/dam/www/program/design/us/en/documents/low-pin-count-interface-specification.pdf)
for details):

- `input clk_i`: LPC clock.
- `input nrst_i`: LPC reset (active low).
- `input lframe_i`: LPC frame input (active low).
- `inout lad_bus`: LPC data bus, slow pull-up on host side.
- `inout serirq`: LPC SERIRQ signal, slow pull-up on host side. More information
  about SERIRQ can be found [here](https://web.archive.org/web/20150502224201/http://hackipedia.org/Platform/x86/PCI/Serialized%20IRQ%20Support%20for%20PCI%20Systems.doc.pdf).

Ports for signals to/from data provider:

- `output lpc_addr_o`: 16-bit address of TPM register.
- `input lpc_data_i`: data received from data provider (TPM registers module) to
  be sent to host.
- `output lpc_data_o`: data received from host to be sent to TPM registers
  module.
- `output lpc_data_wr`: signal to data provider that `lpc_addr_o` and
  `lpc_data_o` have valid data and write is requested.
- `input lpc_wr_done`: signal from data provider that `lpc_data_o` has been
  read. This signal should be driven until LPC module stops driving
  `lpc_data_wr`, after which this signal should be stopped being driven no later
  than 1 `clk_i` cycle (30ns). LPC module changes `lpc_data_wr` on falling
  `clk_i` edge so it is suggested that data provider module samples that signal
  on rising edge.
- `output lpc_data_req`: signal to data provider that data is requested (rising
  edge of this signal) or has been read (falling edge) from `lpc_data_i`.
  `lpc_data_req` is changed on falling edge of `clk_i`.
- `input lpc_data_rd`: signal from data provider that `lpc_data_i` has valid
  data for reading. This signal should be driven in response to `lpc_data_req`.
  `lpc_data_rd` is sampled by LPC module on falling `clk_i` edge. This signal
  should be driven until LPC module stops driving `lpc_data_req`, after which
  this signal should be stopped being driven no later than 1 `clk_i` cycle
  (30ns). LPC module changes `lpc_data_req` on falling `clk_i` edge so it is
  suggested that data provider module samples that signal on rising edge.
- `input irq_num`: IRQ (interrupt request) number, for TPM this is configured by
  `TPM_INT_VECTOR_x.sirqVec` (see [TCG PC Client Platform TPM Profile
  Specification for TPM 2.0](https://trustedcomputinggroup.org/wp-content/uploads/PC-Client-Specific-Platform-TPM-Profile-for-TPM-2p0-v1p05p_r14_pub.pdf)
  chapter 6.6.1.3). No parsing is done by LPC module, meaning that both IRQ0
  (which in some, but not all parts of TPM specification this means "IRQ
  disabled") and IRQ2 (SMI) are valid. IRQ number is sampled on `clk_i` falling
  edge on SERIRQ start frame, turn-around phase. This is done to avoid sending
  one interrupt with two different IRQ numbers in one cycle.
- `input interrupt`: whether interrupt should be signaled to host to which TwPM
  is connected, active high. It is checked at the beginning of SERIRQ frame of
  IRQ latched from `irq_num`, both in quiet and continuous mode. In addition to
  that, in quiet mode this signal initializes SERIRQ cycle. Data provider should
  drive this signal as long as reason for interrupt is valid.

## TPM registers module

Source code: [Dasharo/verilog-tpm-fifo-registers](https://github.com/Dasharo/verilog-tpm-fifo-registers)

This module implements TPM register space. It also handles locality transitions,
TPM interrupt generation and command finite state machine. Register values are
reported accordingly to the current state. Registers not defined by PC Client
specification return 0xFF on reads, and writes are dropped.

The module is located between host interface module (LPC or, in the future, SPI)
and memory buffer for TPM commands and responses. It also exposes hardware
interface that is translated by top module into software interface for TPM stack
running on NEORV32 processor.

![TPM registers module](/images/regs_module.svg)

Ports for signals to/from LPC module:

- `input clk_i`: LPC clock is used for this module to allow for synchronous
  communication with LPC module. Because of that, all registers' values are
  available in one clock cycle and no wait states have to be inserted.
- `input addr_i`: 16-bit address of register to access.
- `input data_i`: 8-bit data from LPC module.
- `output data_o`: 8-bit data to LPC module.
- `input data_wr`, `output wr_done`, `input data_req`, `output data_rd`: 4
  signals coordinating communication over `data_i` and `data_o`. Their functions\
  can be found in the [LPC module description](#lpc-module) above.
- `output irq_num`, `output interrupt`: configuration and request of interrupts
  sent to host, see [LPC module description](#lpc-module) for details. Note that
  these are not interrupts sent towards NEORV32.

Ports for signals for MCU interface:

- `output op_type`: type of operation requested from TPM stack. Currently only
  `0` (no operation) and `1` (execute TPM2 command located in TPM RAM) are used.
- `output locality`: locality at which the operation was requested.
- `output buf_len`: length of data (e.g. TPM2 command) in TPM RAM.
- `output exec`: signal that all of the above are valid and TPM stack should
  start performing the operation specified by `op_type`.
- `output abort`: signal that the host requested the current operation to abort,
  TPM stack may check this bit intermittently and either finish or stop
  executing the ongoing operation.
- `input complete`: signal that the operation has been finished and TPM RAM
  contains the response (if any).

Ports for TPM RAM interface:

- `output RAM_addr`: address in RAM on which to operate. 
- `input RAM_data_r`: data read from `RAM_addr`.
- `output RAM_data_w`: data to be written at `RAM_addr`.
- `output RAM_wr`: do a write. `RAM_data_r` is undefined as long as this signal
  is active, this is to allow for different FPGA implementations to be used.
