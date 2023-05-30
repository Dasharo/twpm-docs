# Verilog modules

Below is description of FPGA modules used by TwPM for EOS-S3. The design may
change as the project progresses, state described here is valid for revision
[84511f8e0e66](https://github.com/Dasharo/TwPM_toplevel/tree/84511f8e0e6659eb1efbf77b91cb033ed20a1bb8)
of top module.

## Top level

Top level module code is located [in this repository](https://github.com/Dasharo/TwPM_toplevel).
It glues together other modules, which are referenced by that repository as git
submodules. It also describes how the signals are connected to external IO pins.

As of now, there is only one submodule and its ports are connected directly to
external pins, even those that have special functions and shouldn't be used as
generic IO. This was done to test if the design can be successfully placed and
routed, and to get rough estimates of FPGA utilization, but it doesn't produce
usable board yet.

After other modules are added and interconnected, this section will show mapping
of interface signals (LPC or SPI) to board's IO pins.

Current FPGA utilization:

```text
Device Utilization: 0.14 (target 1.00)
	Physical Tile TL-LOGIC:
	Block Utilization: 0.16 Logical Block: PB-LOGIC
	Physical Tile TL-RAM:
	Block Utilization: 0.00 Logical Block: PB-RAM
	Physical Tile TL-MULT:
	Block Utilization: 0.00 Logical Block: PB-MULT
	Physical Tile TL-BIDIR:
	Block Utilization: 0.94 Logical Block: PB-BIDIR
	Physical Tile TL-CLOCK:
	Block Utilization: 0.00 Logical Block: PB-CLOCK
	Physical Tile TL-SDIOMUX:
	Block Utilization: 0.79 Logical Block: PB-SDIOMUX
	Physical Tile TL-GMUX:
	Block Utilization: 0.20 Logical Block: PB-GMUX
	Physical Tile TL-ASSP:
	Block Utilization: 1.00 Logical Block: PB-ASSP
	Physical Tile TL-SYN_VCC:
	Block Utilization: 1.00 Logical Block: PB-SYN_VCC
	Physical Tile TL-SYN_GND:
	Block Utilization: 1.00 Logical Block: PB-SYN_GND
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
- `inout lpc_data_io`: data received from (write) or to be sent (read) to host.
  See descriptions of next 4 signals for requirements related to driving this
  signal.
- `output lpc_data_wr`: signal to data provider that `lpc_data_io` is driven by
  LPC module. Data provider should stop driving `lpc_data_io` immediately upon
  receiving this signal. To avoid hazards, data provider should sample data on
  rising edge of `clk_i` or wait at least half of `clk_i` cycle (15 ns for 33
  MHz) before sampling.
- `input lpc_wr_done`: signal from data provider that `lpc_data_io` has been
  read. This signal should be driven until LPC module stops driving
  `lpc_data_wr`, after which this signal should be stopped being driven no later
  than 1 `clk_i` cycle (30ns). LPC module changes `lpc_data_wr` on falling
  `clk_i` edge so it is suggested that data provider module samples that signal
  on rising edge.
- `output lpc_data_req`: signal to data provider that data is requested (rising
  edge of this signal) or has been read (falling edge) from `lpc_data_io`.
  `lpc_data_req` is changed on falling edge of `clk_i`.
- `input lpc_data_rd`: signal from data provider that `lpc_data_io` has valid
  data for reading. This signal should be driven in response to `lpc_data_req`.
  It should never be driven if `lpc_data_wr` is also active. `lpc_data_rd` is
  sampled by LPC module on falling `clk_i` edge. Data provider module may either
  start driving both `lpc_data_rd` and `lpc_data_io` signals on rising edge, or
  drive `lpc_data_io` for at least half a cycle (15 ns) before asserting
  `lpc_data_rd`. This signal should be driven until LPC module stops driving
  `lpc_data_req`, after which this signal should be stopped being driven no
  later than 1 `clk_i` cycle (30ns). LPC module changes `lpc_data_req` on
  falling `clk_i` edge so it is suggested that data provider module samples that
  signal on rising edge.
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

This module implements TPM register space. It also handles locality transitions
and TPM interrupt generation. Register values are reported accordingly to the
current state. Registers not defined by PC Client specification return 0xFF on
reads, and writes are dropped.

In the future this module will also contain command finite state machine. Until
then, `TPM_STS` and `TPM_DATA_FIFO` registers are not supported - writes are
ignored and reads return 0xFF, just as for any other unimplemented register.

The module is located between host interface module (LPC or, in the future, SPI)
and module responsible for communication with MCU. At this point, the latter is
not yet implemented and no connections towards it are defined.

**NOTE: this submodule is not included in the top level module at this point.**

![TPM registers module](/images/regs_module.svg)

Ports for signals to/from LPC module:

- `input clk_i`: LPC clock is used for this module to allow for synchronous
  communication with LPC module. Because of that, all registers' values are
  available in one clock cycle and no wait states have to be inserted.
- `input addr_i`: 16-bit address of register to access.
- `inout data_io`: 8-bit bi-directional data from or to LPC module.
- `input data_wr`, `output wr_done`, `input data_req`, `output data_rd`: 4
  signals coordinating communication over `data_io`. Their functions can be
  found in the [LPC module description](#lpc-module) above.
- `output irq_num`, `output interrupt`: configuration and request of interrupts
  sent to host, see [LPC module description](#lpc-module) for details. Note that
  these are not interrupts sent towards MCU.
