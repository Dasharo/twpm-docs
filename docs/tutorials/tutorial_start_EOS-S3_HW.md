Getting started QuickLogic EOS S3 Board
======================================

# Intro

The main goal of this task was to get familiar with the board and development
environment. QuickFeather EOS S3 Board is simple solution for getting started
with FPGA development. 
SparkFun QuickLogic Thing Plus EOS S3 is development board, based on EOS S3
FPGA/ARM Cortex-M4 Microcontroller.

# Software development

Software development for this board involves both programming ARM Cortex-M4 
and FPGA module. 

## Development of the FPGA module 

To develop the bitstream of the FPGA Synthesis the tool named *Symbiflow*
is used. 

## Development of the ARM Cortex-M4

To develop the M4 part of the SoC `arm-none-eabi-*` tools could be used.

## Basic steps 

To prepare development environment steps from [this instruction](https://github.com/Dasharo/twpm-docs/blob/main/docs/tutorials/building.md) should be used.
(parts *Docker image*, *Building*).

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
```
 -v $PWD:/home/qorc-sdk/workspace
```
to 
```
 -v $PWD:/home/qorc-sdk/qorc-sdk/workspace 
```

Warning: this correction reorganizes structure of the catalog in Docker image.


