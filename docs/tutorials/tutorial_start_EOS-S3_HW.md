# Getting started with QuickLogic EOS S3 Board

## Intro

The main goal of this tutorial is to get familiar with the board and development
environment by compiling, flashing and running one of the examples from
[qorc-onion-apps](https://github.com/coolbreeze413/qorc-onion-apps).
QuickFeather EOS S3 Board is simple solution for getting started
with FPGA development. SparkFun QuickLogic Thing Plus EOS S3 is development
board, based on EOS S3 FPGA/ARM Cortex-M4 Microcontroller.

## Software development

Software development for this board involves both programming ARM Cortex-M4
and FPGA module.

### Development of the FPGA module

To develop the bitstream of the FPGA Synthesis the tool named _Symbiflow_
is used.

### Development of the ARM Cortex-M4

To develop the M4 part of the SoC standard ARM toolchain
could be used.

### Basic steps

To prepare development environment Docker image from [this instruction][build-instr-url]
should be used.

Note that some modifications are required due to different directory structure
than that used by TwPM. Without that, you may end up with cryptic error message:

```text
make -f makefiles/Makefile_Startup
make[2]: Entering directory '/home/qorc-sdk/workspace/qorc-onion-apps/qorc_fpga_breathectrl/GCC_Project'
make[2]: *** No rule to make target '/home/qorc-sdk/workspace/qorc-onion-apps/qorc_fpga_breathectrl/GCC_Project/output/startup_EOSS3B_GCC.o', needed by 'all'.  Stop.
make[2]: Leaving directory '/home/qorc-sdk/workspace/qorc-onion-apps/qorc_fpga_breathectrl/GCC_Project'
make[1]: *** [Makefile:59: Startup] Error 2
make[1]: Leaving directory '/home/qorc-sdk/workspace/qorc-onion-apps/qorc_fpga_breathectrl/GCC_Project'
make: *** [Makefile:54: m4] Error 2
```

To use `twpm-sdk` for `qorc-onion-apps`, during start of the Docker container
one option should be corrected:

```shell
 -v $PWD:/home/qorc-sdk/workspace
```

to

```shell
 -v $PWD:/home/qorc-sdk/qorc-sdk/qorc-onion-apps
```

`docker` command should be run from `qorc-onion-apps` directory. From there,
change directory to `qorc-sdk/qorc-onion-apps/qorc_<any_example>` and run
`make`. It should build image without any errors. Refer to example's README
files for instructions on flashing and using them.

## Architecture overview

Architectural overview of this SoC has been depicted in document:
[EOS S3 Datasheet][eos-s3-datasheet-url], page 19.

There is also different block diagram of the S3, with main subsystems,
printed in [QL S3 Technical Reference Manual][ql-eos-s3-tech-ref-man-url], page 23.

## Example code

There are example applications, in [Source][examples-source].
Bunch of them, with pattern: `qorc_helloworld*` are the examples of basic
snippets, which we could use as a template for further applications.

Currently, as example, I use `qorc_helloworldm4` app. This is
FreeRTOS-compatible, fully working snippet.

This program uses the following modules:

* Basic ARM Cortex M4 initialization;
* Physical interfaces:
    - UART
    - I2C
* GPIO, RGB LED diode

### Pin mapping

Entry point for this application is [main.c][snippet-main-entry].
Assigning the signal to pad has been done in the [qorc_hardwareSetup() function][snippet-hardware-setup-function].
To proper assignment the array `PadConfig pincfg_table[]`, passed as first
argument should be initialized [code here][snippet-fpga-pin-config].
Assigning pin to FPGA structure could be performed as in the following [example][snippet-pad-config].
For controlling GPIO via FPGA the example is [qorc_fpga_gpioctrl][snippet-gpio-control].

### FPGA-RAM Communication

This task has been depicted in the [qf_ramblockinit example][snippet-ramblockinit].

Communication between M4 and RAM implemented in FPGA is performed by AHB bus.
Initialization is showed in the [following example][snippet-m4-ram-fpga-communication].

Access to already declared RAM has been instantiated in the following [example][snippet-access-to-ram-from-fpga].

### Examples: qorc_helloworldm4fpga, qorc_helloworldm4fpgaheader

These two examples are located here: [qorc_helloworldm4fpga][example-helloworldm4fpga]
and [qorc_helloworldm4fpgaheader][example-helloworldm4fpga_header].
These two examples are similar, in system behaviour (blinking red LED).
The difference between those two projects is the dynamically loading bitstream
to FPGA (`qorc_helloworldm4fpgaheader`).

There are 3 significant differences between those two:

1. Different output format requested from `ql_symbiflow` in Makefile:

    `-dump binary openocd jlink` is used for producing file to be flashed, while
    to generate header to be included in C `-dump header` must be used instead.

2. Created header file is included in `main.c` along with symbol definitions:

    ```c
    #include "fpga_loader.h"
    #include <stdint.h>
    #include "helloworldfpga_bit.h"
    extern uint32_t axFPGABitStream [];
    extern uint32_t axFPGAMemInit [];
    extern uint32_t axFPGAIOMuxInit [];
    ```

3. FPGA is loaded and started from `main()` just after `Hello Onion!!`:

    ```c
    S3x_Clk_Disable(S3X_FB_21_CLK);
    S3x_Clk_Disable(S3X_FB_16_CLK);

    S3x_Clk_Enable(S3X_A1_CLK);
    S3x_Clk_Enable(S3X_CFG_DMA_A1_CLK);

    //load_fpga_with_mem_init(sizeof(axFPGABitStream), axFPGABitStream, sizeof(axFPGAMemInit), axFPGAMemInit);
    load_fpga(sizeof(axFPGABitStream), axFPGABitStream);
    fpga_iomux_init(sizeof(axFPGAIOMuxInit), axFPGAIOMuxInit);
    dbg_str("FPGA Programmed\n");
    ```

### FPGA interrupts

Four outputs from the on-chip programmable logic can be used as messages. Each
interrupt message can be selected to be either edge or level detection. For edge
detection, it can be configured to be positive or negative edge; similarly, if
selected to be level detect, it can be configured to be level high or low. All
of FPGA interrupts arrive to M4 as [IRQ 4][qorc-sdk-fb-irq], and exact interrupt
can [be read from `FB_INTR_RAW` NVIC register][example-timerctrl-fbmsg-handler].

One of the examples using dedicated FPGA interrupts is [qorc_fpga_timerctrl][example-timerctrl].
It shows how to set up both sides of the communication.

1. Configure and enable IRQ handling on M4 by calling [this function][example-timerctrl-irq-enable]
   from `main()`:

    ```c
    void hal_fpga_onion_timerctrl_init()
    {
        //Clear any pending interrupt of FPGA
        NVIC_ClearPendingIRQ(FbMsg_IRQn);

        //Register FPGA ISR callbacks FPGA Slow Down and Speed up interrupts
        FB_RegisterISR(FB_INTERRUPT_0, hal_fpga_onion_timerctrl0_isr);
        FB_ConfigureInterrupt(FB_INTERRUPT_0,
                                FB_INTERRUPT_TYPE_EDGE,
                                FB_INTERRUPT_POL_LEVEL_HIGH,
                                FB_INTERRUPT_DEST_AP_DISBLE,
                                FB_INTERRUPT_DEST_M4_ENABLE);

        //Enable the FPGA interrupts
        NVIC_EnableIRQ(FbMsg_IRQn);
    }
    ```

2. [Route interrupt signal][example-timerctrl-fpga] from FPGA towards NVIC:

    ```verilog
    // Verilog model of QLAL4S3B
    qlal4s3b_cell_macro u_qlal4s3b_cell_macro
    (
        // More signals here, removed for clarity

        // FB Interrupts
        .FB_msg_out                ( { 1'b0, 1'b0, 1'b0, FPGA_INTR[0] }       ), // input   [3:0]
        .FB_Int_Clr                ( 8'h0                           ), // input   [7:0]
        .FB_Start                  (                                ), // output
        .FB_Busy                   ( 1'b0                           ), // input

        // More signals here, removed for clarity
    );
    ```

[build-instr-url]:/tutorials/building/#docker-image
[eos-s3-datasheet-url]: https://www.quicklogic.com/wp-content/uploads/2020/12/QL-EOS-S3-Ultra-Low-Power-multicore-MCU-Datasheet-2020.pdf
[ql-eos-s3-tech-ref-man-url]: https://www.quicklogic.com/wp-content/uploads/2020/06/QL-S3-Technical-Reference-Manual.pdf
[examples-source]: https://github.com/coolbreeze413/qorc-onion-apps/tree/master/
[snippet-main-entry]: https://github.com/coolbreeze413/qorc-onion-apps/blob/master/qorc_helloworldm4/src/main.c
[snippet-hardware-setup-function]: https://github.com/coolbreeze413/qorc-onion-apps/blob/a960ca3f450fafde9da07547b35efe8e1caa2574/qorc_helloworldm4/src/qorc_hardwaresetup.c#LL48C9-L48C9
[snippet-fpga-pin-config]: https://github.com/coolbreeze413/qorc-onion-apps/blob/a960ca3f450fafde9da07547b35efe8e1caa2574/qorc_helloworldm4/src/pincfg_table.c#L21
[snippet-pad-config]: https://github.com/coolbreeze413/qorc-onion-apps/blob/a960ca3f450fafde9da07547b35efe8e1caa2574/qorc_helloworldfpga/fpga/rtl/quickfeather.pcf#LL44C8-L44C8
[snippet-gpio-control]: https://github.com/coolbreeze413/qorc-onion-apps/tree/a960ca3f450fafde9da07547b35efe8e1caa2574/qorc_fpga_gpioctrl
[snippet-ramblockinit]: https://github.com/QuickLogic-Corp/qorc-testapps/tree/92bf33c9dd51aed94554d26e85fd37faf756f42e/qf_ramblockinit
[snippet-m4-ram-fpga-communication]: https://github.com/QuickLogic-Corp/qorc-testapps/blob/92bf33c9dd51aed94554d26e85fd37faf756f42e/qf_ramblockinit/fpga/rtl/AL4S3B_FPGA_IP.v#L119
[snippet-access-to-ram-from-fpga]: https://github.com/QuickLogic-Corp/qorc-testapps/blob/92bf33c9dd51aed94554d26e85fd37faf756f42e/qf_ramblockinit/fpga/rtl/AL4S3B_FPGA_RAMs.v#L165
[example-helloworldm4fpga]: https://github.com/coolbreeze413/qorc-onion-apps/tree/a960ca3f450fafde9da07547b35efe8e1caa2574/qorc_helloworldm4fpga
[example-helloworldm4fpga_header]: https://github.com/coolbreeze413/qorc-onion-apps/tree/a960ca3f450fafde9da07547b35efe8e1caa2574/qorc_helloworldm4fpgaheader
[example-timerctrl]: https://github.com/coolbreeze413/qorc-onion-apps/tree/a960ca3f450fafde9da07547b35efe8e1caa2574/qorc_fpga_timerctrl
[example-timerctrl-fbmsg-handler]: https://github.com/coolbreeze413/qorc-onion-apps/blob/a960ca3f450fafde9da07547b35efe8e1caa2574/qorc_fpga_timerctrl/src/exceptions.c#L344
[example-timerctrl-irq-enable]: https://github.com/coolbreeze413/qorc-onion-apps/blob/a960ca3f450fafde9da07547b35efe8e1caa2574/qorc_fpga_timerctrl/fpga/src/hal_fpga_onion_timerctrl.c#L18
[example-timerctrl-fpga]: https://github.com/coolbreeze413/qorc-onion-apps/blob/a960ca3f450fafde9da07547b35efe8e1caa2574/qorc_fpga_timerctrl/fpga/rtl/AL4S3B_FPGA_Top.v#L131
[qorc-sdk-fb-irq]: https://github.com/QuickLogic-Corp/qorc-sdk/blob/d61d064146c0ee927aa12b088b3bbbce60615f4d/HAL/inc/eoss3_dev.h#L68
