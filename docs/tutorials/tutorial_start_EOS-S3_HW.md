# Getting started QuickLogic EOS S3 Board

## Intro

The main goal of this task was to get familiar with the board and development
environment. QuickFeather EOS S3 Board is simple solution for getting started
with FPGA development.
SparkFun QuickLogic Thing Plus EOS S3 is development board, based on EOS S3
FPGA/ARM Cortex-M4 Microcontroller.

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

To prepare development environment steps from [this instruction][build-instr-url]
should be used(parts _Docker image_, _Building_).

> Note: There was problem during compilation for Cortex-M4 device.

```shell
make -f makefiles/Makefile_Startup
make[2]: Entering directory '/home/qorc-sdk/workspace/qorc-onion-apps/qorc_fpga_breathectrl/GCC_Project'
make[2]: *** No rule to make target '/home/qorc-sdk/workspace/qorc-onion-apps/qorc_fpga_breathectrl/GCC_Project/output/startup_EOSS3B_GCC.o', needed by 'all'.  Stop.
make[2]: Leaving directory '/home/qorc-sdk/workspace/qorc-onion-apps/qorc_fpga_breathectrl/GCC_Project'
make[1]: *** [Makefile:59: Startup] Error 2
make[1]: Leaving directory '/home/qorc-sdk/workspace/qorc-onion-apps/qorc_fpga_breathectrl/GCC_Project'
make: *** [Makefile:54: m4] Error 2
```

There is workaround for this problem. During start of the Docker container
one option should be corrected:

```shell
 -v $PWD:/home/qorc-sdk/workspace
```

to

```shell
 -v $PWD:/home/qorc-sdk/qorc-sdk/workspace
```

Warning: this correction reorganizes structure of the catalog in Docker container.

More generic solution is to pay attention about nesting the directories.

## Architecture overview

Architectural overview of this SoC has been depicted in document:
[EOS S3 Datasheet][eos-s3-datasheet-url], page 19.

There is also different block diagram of the S3, with main subsystems,
printed in [QL S3 Technical Reference Manual][ql-eos-s3-tech-ref-man-url], page 23.

## Example code

There is example applications, in [Source][examples-source].
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

Entry point for this application is [main.c](https://github.com/coolbreeze413/qorc-onion-apps/blob/master/qorc_helloworldm4/src/main.c).
Assigning the signal to pad has been done in the [qorc_hardwareSetup() function](https://github.com/coolbreeze413/qorc-onion-apps/blob/a960ca3f450fafde9da07547b35efe8e1caa2574/qorc_helloworldm4/src/qorc_hardwaresetup.c#LL48C9-L48C9).
To proper assignment the array `PadConfig pincfg_table[]`, passed as first
argument should be initialized [code here](https://github.com/coolbreeze413/qorc-onion-apps/blob/a960ca3f450fafde9da07547b35efe8e1caa2574/qorc_helloworldm4/src/pincfg_table.c#L21)

Assigning pin to FPGA structure could be performed as in the following [example](https://github.com/coolbreeze413/qorc-onion-apps/blob/a960ca3f450fafde9da07547b35efe8e1caa2574/qorc_helloworldfpga/fpga/rtl/quickfeather.pcf#LL44C8-L44C8)

For controlling GPIO via FPGA the example is [qorc_fpga_gpioctrl](https://github.com/coolbreeze413/qorc-onion-apps/tree/a960ca3f450fafde9da07547b35efe8e1caa2574/qorc_fpga_gpioctrl).

### FPGA-RAM Communication

This task has been depicted in the [qf_ramblockinit example](https://github.com/QuickLogic-Corp/qorc-testapps/tree/92bf33c9dd51aed94554d26e85fd37faf756f42e/qf_ramblockinit).

Communication between RAM and FPGA/M4 is performed by AHB bus. Initialization
is showed in the [following example](https://github.com/QuickLogic-Corp/qorc-testapps/blob/92bf33c9dd51aed94554d26e85fd37faf756f42e/qf_ramblockinit/fpga/rtl/AL4S3B_FPGA_IP.v#L119)

Access to already declared RAM has been instantiated in the following [example](https://github.com/QuickLogic-Corp/qorc-testapps/blob/92bf33c9dd51aed94554d26e85fd37faf756f42e/qf_ramblockinit/fpga/rtl/AL4S3B_FPGA_RAMs.v#L165).

[build-instr-url]:https://github.com/Dasharo/twpm-docs/blob/main/docs/tutorials/building.md
[eos-s3-datasheet-url]: https://www.quicklogic.com/wp-content/uploads/2020/12/QL-EOS-S3-Ultra-Low-Power-multicore-MCU-Datasheet-2020.pdf
[ql-eos-s3-tech-ref-man-url]: https://www.quicklogic.com/wp-content/uploads/2020/06/QL-S3-Technical-Reference-Manual.pdf
[examples-source]: https://github.com/coolbreeze413/qorc-onion-apps/tree/master/
