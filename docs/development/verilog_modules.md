<!--
SPDX-FileCopyrightText: 2024 3mdeb <contact@3mdeb.com>

SPDX-License-Identifier: CC-BY-SA-4.0
-->

# Verilog modules

Below is description of FPGA modules used by TwPM for OrangeCrab. The design may
change as the project progresses, state described here is valid for revision
marked with tag `v0.2.0` of the top module.

This document includes diagrams generated with [Symbolator](https://kevinpt.github.io/symbolator/).
On these diagrams, input ports are on the left, and outputs and bi-directional
signals are on the right. Module parameters are at the top, on gray background.
In general, clocks are marked with triangle inside, and active-low signals with
circle outside of rectangle symbolizing the module. However, Symbolator makes
a guess about signal function based only on its name, which in some cases gives
wrong results. Such cases are mentioned in the signal descriptions under the
diagrams.

Current FPGA utilization for LPC:

```text
Info: Device utilisation:
Info: 	          TRELLIS_IO:    65/  197    32%
Info: 	                DCCA:     5/   56     8%
Info: 	              DP16KD:     5/   56     8%
Info: 	          MULT18X18D:     1/   28     3%
Info: 	              ALU54B:     0/   14     0%
Info: 	             EHXPLLL:     1/    2    50%
Info: 	             EXTREFB:     0/    1     0%
Info: 	                DCUA:     0/    1     0%
Info: 	           PCSCLKDIV:     0/    2     0%
Info: 	             IOLOGIC:    44/  128    34%
Info: 	            SIOLOGIC:     0/   69     0%
Info: 	                 GSR:     0/    1     0%
Info: 	               JTAGG:     0/    1     0%
Info: 	                OSCG:     0/    1     0%
Info: 	               SEDGA:     0/    1     0%
Info: 	                 DTR:     0/    1     0%
Info: 	             USRMCLK:     1/    1   100%
Info: 	             CLKDIVF:     1/    4    25%
Info: 	           ECLKSYNCB:     1/   10    10%
Info: 	             DLLDELD:     0/    8     0%
Info: 	              DDRDLL:     1/    4    25%
Info: 	             DQSBUFM:     2/    8    25%
Info: 	     TRELLIS_ECLKBUF:     3/    8    37%
Info: 	        ECLKBRIDGECS:     1/    2    50%
Info: 	                DCSC:     0/    2     0%
Info: 	          TRELLIS_FF:  5049/24288    20%
Info: 	        TRELLIS_COMB: 12639/24288    52%
Info: 	        TRELLIS_RAMW:   121/ 3036     3%
```

Current FPGA utilization for SPI:

```text
Info: Device utilisation:
Info: 	          TRELLIS_IO:    62/  197    31%
Info: 	                DCCA:     6/   56    10%
Info: 	              DP16KD:     5/   56     8%
Info: 	          MULT18X18D:     0/   28     0%
Info: 	              ALU54B:     0/   14     0%
Info: 	             EHXPLLL:     1/    2    50%
Info: 	             EXTREFB:     0/    1     0%
Info: 	                DCUA:     0/    1     0%
Info: 	           PCSCLKDIV:     0/    2     0%
Info: 	             IOLOGIC:    44/  128    34%
Info: 	            SIOLOGIC:     0/   69     0%
Info: 	                 GSR:     0/    1     0%
Info: 	               JTAGG:     0/    1     0%
Info: 	                OSCG:     0/    1     0%
Info: 	               SEDGA:     0/    1     0%
Info: 	                 DTR:     0/    1     0%
Info: 	             USRMCLK:     1/    1   100%
Info: 	             CLKDIVF:     1/    4    25%
Info: 	           ECLKSYNCB:     1/   10    10%
Info: 	             DLLDELD:     0/    8     0%
Info: 	              DDRDLL:     1/    4    25%
Info: 	             DQSBUFM:     2/    8    25%
Info: 	     TRELLIS_ECLKBUF:     3/    8    37%
Info: 	        ECLKBRIDGECS:     1/    2    50%
Info: 	                DCSC:     0/    2     0%
Info: 	          TRELLIS_FF:  5025/24288    20%
Info: 	        TRELLIS_COMB: 12175/24288    50%
Info: 	        TRELLIS_RAMW:   121/ 3036     3%
```

## Top level

Top level module code is located [in this repository](https://github.com/Dasharo/TwPM_toplevel).
It glues together other modules, which are referenced by that repository as git
submodules. It also describes how the signals are connected to external IO pins.

![TwPM top level module](/images/twpm_top_orangecrab.svg)

Top level module parameters:

- `TPM_RAM_*`, `TPM_REGS_*`, `TPM_REG_*`: those specify addresses and sizes used
  in [Communication between SoC and FPGA](/development/soc_fpga_communication/).
- `LITEDRAM_*` and `RAM_*`: LiteDRAM controller and main (DDR3) RAM base address
  and size.
- `DEFAULT_READ_VALUE`: value read from `TPM_REGS` region outside of any defined
  register.
- `COMPLETE_PULSE_WIDTH`: width (in system clock cycles) of `complete` signal.
  See [TPM registers module description](#tpm-registers-module) for details on
  that signal.

External ports:

- `clk_i`: input (crystal) clock running at 48 MHz. Note that it is only used
  for driving PLL, which in turn generates 50 MHz clock for most of the system.
- `rstn_i`: asynchronous reset signal, active low. It is converted to
  synchronous reset that is used for most of the components. (Note: it is not
  marked as negated on the diagram due to how Symbolator detects such signals,
  i.e. its name doesn't end with either `_n` or `_b`).
- `uart_rxd_i`, `uart_txd_o`: UART running at 115200n8.
- LPC interface: those are to be connected to the mainboard, see
  [Connecting TwPM to mainboard](/tutorials/mainboard-connection/). Note that
  only one of LPC or SPI interface is present at any given time, depending on
  build configuration.
- SPI interface: those are to be connected to the mainboard, see
  [Connecting TwPM to mainboard](/tutorials/mainboard-connection/). Note that
  only one of LPC or SPI interface is present at any given time, depending on
  build configuration.
- SPI flash signals: connected to onboard SPI flash. Note that there is no clock
  signal on the diagram, a hardware macro must be used instead of defining it as
  a port.
- DDR3 interface: signals to and from onboard DRAM, connected directly to
  LiteDRAM module. Negative part of differential signals (CK, DQS) are
  implemented with hardware macros, so they are not listed as I/O ports.
- `led_r`, `led_g`, `led_b`: outputs driving RGB LED. All of them are active low
  (this is how LEDs usually work), even though only one of them is marked as
  such (again, due to how Symbolator parses names).

## NEORV32

[The NEORV32 processor](https://stnolting.github.io/neorv32/) is used to run
software TPM stack on. It is highly configurable, and many of its modules are
disabled in TwPM, so not all of the [Processor top entity signals](https://stnolting.github.io/neorv32/#_processor_top_entity_signals)
are used.

![NEORV32 module](/images/neorv32_verilog_wrapper.svg)

- `clk_i`, `rstn_i`: main clock and reset signals. Note that these are not the
  top level clock and reset, instead these are outputs from LiteDRAM described
  below. This clock runs at 50 MHz. Reset is active-low, but Symbolator didn't
  represent it correctly on diagram.
- `uart0_rxd_i`, `uart0_txd_o`: main UART, connected directly to I/O pads.
- JTAG signals: unused for now.
- WISHBONE bus interface: configured as WISHBONE Classic. Used for accessing
  modules external to the processor, like TPM module, DRAM and LiteDRAM
  controller.
- SPI interface: connected to top level I/O either directly or through hardware
  macro (`spi_clk_o`).
- GPIO: only 3 outputs are connected to LEDs to provide some kind of output even
  if UART is not connected. Inputs are hardwired to 0 because there is no way of
  implementing just the outputs.

## LiteDRAM

While the module was generated with [LiteDRAM](https://github.com/enjoy-digital/litedram),
the entire source is included in [TwPM_toplevel repository](https://github.com/Dasharo/TwPM_toplevel/tree/main/fpga/orangecrab_litedram),
along with configuration file used to create it. This is done to make the code
reproducible, as well as to add some customizations, listed in comment at the
top of Verilog file.

![LiteDRAM module](/images/litedram_core.svg)

List of ports:

- `clk`, `rst`, `user_clk` and `user_rst`: this module implements PLL and reset
  synchronizer, these are respective inputs and outputs for them. On input we
  have 48 MHz clock and asynchronous reset, and on output 50 MHz clock with
  synchronous reset signal. Note that both reset signals are **active-high**,
  contrary to the rest of the project. Both are negated in top level module.
- DDR3 RAM signals: routed directly to top level module's ports.
- `init_done`, `init_error`, `pll_locked`: status signals, not really used in
  current implementation. First two are controlled by software doing the
  initialization, and the `user_rst` is active until PLL is locked, which in
  turn doesn't release NEORV32 core. `pll_locked` is connected to blue LED as a
  sign of life.
- RAM WISHBONE interface: WISHBONE Classic interface for accessing the main
  memory. Mostly standard, except for `user_port_wishbone_adr` - it specifies
  (4 bytes) _word_ address, not _byte_ address. Its width is also limited to the
  size of RAM.
- Controller WISHBONE interface: also a WISHBONE Classic interface, used to set
  up and initialize RAM before it is usable. Refer to the LiteDRAM and
  [liblitedram](https://github.com/enjoy-digital/litex/tree/master/litex/soc/software/liblitedram)
  (which for some reason is in main LiteX repository) for details on how to
  interact with it. `wb_ctrl_adr` is also using word addresses as above, but its
  width is not limited otherwise. Only lower bits are compared in the module,
  which means that top level has to arbitrate other signals (e.g. CYC or STB) to
  avoid aliasing.

## LPC module

Source code: [Dasharo/verilog-lpc-module](https://github.com/Dasharo/verilog-lpc-module)

Testbench results:

```text
VCD info: dumpfile lpc_periph_tb.vcd opened for output.
Performing TPM write w/o delay
Performing TPM write with delay
Performing TPM read with delay
Performing TPM read w/o delay
Testing reset behaviour - TPM write w/o delay
Testing reset behaviour - TPM read w/o delay
Testing reset behaviour - TPM write with delay
Testing reset behaviour - TPM read with delay
Testing non-TPM transactions
Testing extended LFRAME# timings - write
Testing extended LFRAME# timings - read
Testing abort mechanism - write
Testing abort mechanism - read
Testing interrupts - Continuous mode:
  no interrupt reported when not requested?
  proper IRQ reported?
  IRQ number latched at start frame?
  IRQ keeps being sent while active?
  IRQ stops being sent when no longer active?
  recovery and turn-around phases executed when int is deactivated?
  IRQs reported with idle clock cycles before stop frame?
  IRQs reported with idle clock cycles after stop frame?
  IRQs reported with longer start pulse width?
Testing interrupts - switching between modes:
  peripheral doesn't initialize SERIRQ cycle in Quiet mode when not needed?
  peripheral initializes SERIRQ cycle when IRQ needed in Quiet mode?
  reset switches peripheral to Continuous mode?
Testing interrupts - Quiet mode:
  proper IRQ reported?
  IRQ number latched at start frame?
  IRQ keeps being sent while active?
  IRQ stops being sent when no longer active?
  recovery and turn-around phases executed when int is deactivated?
  IRQs reported with idle clock cycles before stop frame?
  peripheral keeps working after spurious interrupt?
  IRQs reported with longer start pulse width?
Testing interrupts - IRQ stops being reported on reset
lpc_periph_tb.v:1344: $stop called at 518601000 (1ps)
```

This module is responsible for managing LPC communication. It responds only to
TPM cycles, other cycle types are ignored. SERIRQ (both continuous and quiet
mode), cycle aborts and LPC resets are implemented.

![LPC peripheral module](/images/lpc_periph.svg)

Ports for LPC interface (refer to [LPC specification](https://www.intel.com/content/dam/www/program/design/us/en/documents/low-pin-count-interface-specification.pdf)
for details):

- `clk_i`: LPC clock.
- `nrst_i`: LPC reset (active low).
- `lframe_i`: LPC frame input (active low).
- `lad_bus`: LPC data bus, slow pull-up on host side.
- `serirq`: LPC SERIRQ signal, slow pull-up on host side. More information about
  SERIRQ can be found [here](https://web.archive.org/web/20150502224201/http://hackipedia.org/Platform/x86/PCI/Serialized%20IRQ%20Support%20for%20PCI%20Systems.doc.pdf).

Ports for signals to/from data provider:

- `lpc_addr_o`: 16-bit address of TPM register.
- `lpc_data_i`: data received from data provider (TPM registers module) to be
  sent to host.
- `lpc_data_o`: data received from host to be sent to TPM registers module.
- `lpc_data_wr`: signal to data provider that `lpc_addr_o` and
  `lpc_data_o` have valid data and write is requested.
- `lpc_wr_done`: signal from data provider that `lpc_data_o` has been read. This
  signal should be driven until LPC module stops driving `lpc_data_wr`, after
  which this signal should be stopped being driven no later than 1 `clk_i` cycle
  (30ns). LPC module changes `lpc_data_wr` on falling `clk_i` edge so it is
  suggested that data provider module samples that signal on rising edge.
- `lpc_data_req`: signal to data provider that data is requested (rising edge of
  this signal) or has been read (falling edge) from `lpc_data_i`. `lpc_data_req`
  is changed on falling edge of `clk_i`.
- `lpc_data_rd`: signal from data provider that `lpc_data_i` has valid data for
  reading. This signal should be driven in response to `lpc_data_req`.
  `lpc_data_rd` is sampled by LPC module on falling `clk_i` edge. This signal
  should be driven until LPC module stops driving `lpc_data_req`, after which
  this signal should be stopped being driven no later than 1 `clk_i` cycle
  (30ns). LPC module changes `lpc_data_req` on falling `clk_i` edge so it is
  suggested that data provider module samples that signal on rising edge.
- `irq_num`: IRQ (interrupt request) number, for TPM this is configured by
  `TPM_INT_VECTOR_x.sirqVec` (see [TCG PC Client Platform TPM Profile
  Specification for TPM 2.0](https://trustedcomputinggroup.org/wp-content/uploads/PC-Client-Specific-Platform-TPM-Profile-for-TPM-2p0-v1p05p_r14_pub.pdf)
  chapter 6.6.1.3). No parsing is done by LPC module, meaning that both IRQ0
  (which in some, but not all parts of TPM specification this means "IRQ
  disabled") and IRQ2 (SMI) are valid. IRQ number is sampled on `clk_i` falling
  edge on SERIRQ start frame, turn-around phase. This is done to avoid sending
  one interrupt with two different IRQ numbers in one cycle.
- `interrupt`: whether interrupt should be signaled to host to which TwPM is
  connected, active high. It is checked at the beginning of SERIRQ frame of IRQ
  latched from `irq_num`, both in quiet and continuous mode. In addition to
  that, in quiet mode this signal initializes SERIRQ cycle. Data provider should
  drive this signal as long as reason for interrupt is valid.

## SPI module

Source code: [Dasharo/verilog-spi-module](https://github.com/Dasharo/verilog-spi-module)

Testbench results:

```text
VCD info: dumpfile spi_periph_tb.vcd opened for output.
Performing TPM write w/o delay
Performing TPM write with delay
Performing TPM read with delay
Performing TPM read w/o delay
Testing transfers with scattered clock between bytes
Testing over-sized transfers
Testing non-TPM addresses
Testing crossing registers boundary
spi_periph_tb.v:349: $stop called at 86540000 (1ps)
```

This module is responsible for managing SPI communication with PC. It only
supports SPI protocol as described in TPM specification.

![SPI peripheral module](/images/spi_periph.svg)

Ports for SPI interface:

- `clk_i`: SPI clock.
- `cs_n`: Chip select (active low).
- `mosi`: SPI Main Out Sub In.
- `miso`: SPI Main In Sub Out, slow pull-up on host side.

Ports for signals to/from data provider:

- `addr_o`: 16-bit address of TPM register.
- `data_i`, `data_o`: data received from or sent to TPM registers module.
- `data_wr`: signal to data provider that `addr_o` and `data_o` have valid data
  and write is requested.
- `wr_done`: signal from data provider that `data_o` has been read. This signal
  isn't used by SPI module because it would most likely arrive when the clock is
  no longer running. Contrary to the LPC, SPI clock runs only during the
  transmission.
- `data_req`: signal to data provider that data is requested.
- `data_rd`: signal from data provider that `data_i` has valid data for reading.
  This signal should be driven in response to `data_req`.

Note that there are no signals responsible for interrupts. SPI uses PIRQ, which
doesn't require any additional logic, so `interrupt` signal from TPM registers
module is used to drive it directly in the top level module.

## TPM registers module

Source code: [Dasharo/verilog-tpm-fifo-registers](https://github.com/Dasharo/verilog-tpm-fifo-registers)

Testbench results:

```text
VCD info: dumpfile regs_tb.vcd opened for output.
Testing simple register reads without delay
Testing simple register reads with delay
Checking register values against expected.txt
Checking if RO registers are writable
Testing mechanisms for changing locality
Testing mechanisms for seizing locality
Testing TPM_INT_VECTOR write without delay - proper locality
Testing TPM_INT_VECTOR write with delay - proper locality
Testing TPM_INT_VECTOR write without delay - wrong locality
Testing TPM_INT_VECTOR write with delay - wrong locality
Testing TPM_INT_VECTOR write without delay - no locality
Testing TPM_INT_VECTOR write with delay - no locality
Testing command/response exchange and TPM state machine - basic
Testing command/response exchange and TPM state machine - advanced
regs_tb.v:1075: $stop called at 2023220000 (1ps)
```

This module implements TPM register space. It also handles locality transitions,
TPM interrupt generation and command finite state machine. Register values are
reported accordingly to the current state. Registers not defined by PC Client
specification return 0xFF on reads, and writes are dropped.

The module is located between host interface module (LPC or SPI) and memory
buffer for TPM commands and responses. It also exposes hardware interface that
is translated by top module into software interface for TPM stack running on
NEORV32 processor.

![TPM registers module](/images/regs_module.svg)

Ports for signals to/from LPC or SPI module:

- `clk_i`: LPC/SPI clock is used for this module to allow for synchronous
  communication with LPC/SPI module. Because of that, all registers' values are
  available in one clock cycle and no wait states (LPC) or exactly one wait
  state (SPI) has to be inserted. For LPC,the clock is free-running, but for SPI
  it is enabled only during the communication.
- `reset`: reset signal, required to reset registers to their initial values,
  active low.
- `addr_i`: 16-bit address of register to access.
- `data_i`: 8-bit data from LPC/SPI module.
- `data_o`: 8-bit data to LPC/SPI module.
- `data_wr`, `wr_done`, `data_req`, `data_rd`: 4 signals coordinating
  communication over `data_i` and `data_o`. Their functions can be found in the
  [LPC module description](#lpc-module) or [SPI module description](#spi-module)
  above.
- `irq_num`, `interrupt`: configuration and request of interrupts sent to host,
  see [LPC module description](#lpc-module) for details. In case of SPI,
  `interrupt` is negated and routed directly to I/O pin in top level and
  `irq_num` is not used. Note that these are not interrupts sent towards
  NEORV32.

Ports for signals for MCU interface:

- `op_type`: type of operation requested from TPM stack. Currently only `0` (no
  operation) and `1` (execute TPM2 command located in TPM RAM) are used.
- `locality`: locality at which the operation was requested.
- `buf_len`: length of data (e.g. TPM2 command) in TPM RAM.
- `exec`: signal that all of the above are valid and TPM stack should start
  performing the operation specified by `op_type`.
- `abort`: signal that the host requested the current operation to abort, TPM
  stack may check this bit intermittently and either finish or stop executing
  the ongoing operation.
- `complete`: signal that the operation has been finished and TPM RAM contains
  the response (if any).

Ports for TPM RAM interface:

- `RAM_addr`: address in RAM on which to operate, in bytes.
- `RAM_data_r`: data read from `RAM_addr`.
- `RAM_data_w`: data to be written at `RAM_addr`.
- `RAM_wr`: do a write. `RAM_data_r` is undefined as long as this signal is
  active, this is to allow for different FPGA implementations to be used.

## r512x32

This module is also [part of TwPM_toplevel repository](https://github.com/Dasharo/TwPM_toplevel/blob/main/fpga/ram/r512x32_512x32.v).
It implements FPGA RAM (in this case BRAM is used) for TPM command and response
buffer. As the name suggests, it has depth of 512 words, 32 bits each. The width
was chosen to keep connection to WISHBONE simple. TPM registers module uses
8-bit accesses, with simple byte decoder implemented in top level module. While
it would be possible to use dual-port RAM hardware macros, this implementation
is more portable. Synthesis tools are usually smart enough to use those macros
regardless.

Implemented memory is simple read-first, one-port RAM with byte-enable signal
for writes.

![TPM RAM module](/images/r512x32.svg)

Ports:

- `A`: address, counted in 32-bit words.
- `WD`, `RD`: input and output data, respectively.
- `Clk`: input clock, arbitrated by top level between LPC/SPI and system clocks.
- `WEN`: write enable for each byte of `WD`.
