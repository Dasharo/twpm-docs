# TwPM_toplevel

This document serves as a starting point for building and flashing TwPM.

## Cloning

Start by obtaining `TwPM_toplevel` repository. That repository contains just a
bit of glue code and some Makefiles. All of the real code lives in other
repositories included as submodules, so be sure to clone them as well:

```shell
git clone https://github.com/Dasharo/TwPM_toplevel.git
cd TwPM_toplevel
git submodule update --init --checkout
```

## Docker image

All components are expected to be build within Docker container. If you haven't
got Docker installed yet, follow [official installation instructions](https://docs.docker.com/engine/install/)
and [post-installation steps](https://docs.docker.com/engine/install/linux-postinstall/).

### Preparing to use Docker image

TwPM Docker image is created from [twpm-sdk repository](https://github.com/Dasharo/twpm-sdk).
Before it can be used to flash FPGA image, some steps must be done on host
system. Without those steps host's services would get in the way of programmer's
application running in the container. Execute the following:

```shell
id=$(docker create ghcr.io/dasharo/twpm-sdk:main)
sudo docker cp \
  $id:/home/qorc-sdk/qorc-sdk/TinyFPGA-Programmer-Application/71-QuickFeather.rules \
  /etc/udev/rules.d/
docker container rm $id
sudo udevadm control --reload-rules
sudo systemctl restart ModemManager.service
```

This is one-time operation, it isn't required for later use. First of the
instructions above will automatically download the image.

### Starting Docker container

Before container is started, EOS S3 board has to be in programming mode. This
mode is **not** the same as debug mode, so bootstrap jumpers (J2 and J3 on the
[cheatsheet](https://cdn.sparkfun.com/assets/learn_tutorials/1/7/9/1/QuickLogic_Thing_Plus_EOS_S3_v1a.pdf))
have to be kept open. In order to enter programming mode:

* plug the EOS S3 to the PC through USB cable
* press `RST` button once, blue LED will begin to blink a dozen times or so
* while the LED is still blinking, press `USR` button
    - it has debouncing logic to it, you may have to keep it pressed for few
      hundreds milliseconds
    - after that, green LED will be lit for about a second, after which it will
      begin to blink
* confirm that a new ACM device is created (usually `/dev/ttyACM0`, you can
  check `dmesg` output if in doubt)

The container can be started from `TwPM_toplevel` with:

```shell
docker run --rm -it -v $PWD:/home/qorc-sdk/workspace \
    --device=/dev/ttyACM0:/dev/ttyS_QORC ghcr.io/dasharo/twpm-sdk:main
```

Change `ttyACM0` to proper device name, in case there is more than one `ttyACM`
in your system. `ttyS_QORC` is used to have constant name for use by Makefile.
It must start with `ttyS` or one of the other standard names, otherwise pySerial
used by some of the tools can't find it.

This will enable `qorc-sdk` environment and enter shell in the container:

```text
=========================
qorc-sdk envsetup 1.5.1
=========================


executing envsetup.sh from:
/home/qorc-sdk/qorc-sdk

[1] check (minimal) qorc-sdk submodules
    ok.

[2] check arm gcc toolchain
    initializing arm toolchain.
    ok.

[3] check fpga toolchain
    initializing fpga toolchain.
    ok.

[4] check flash programmer
    initializing flash programmer.
    ok.

[5] check openocd
    ok.

[6] check jlink
    ok.


qorc-sdk build env initialized.


(base) qorc-sdk@2fb0c54fd594:~$
```

If you get `error gathering device information while adding custom device`, most
likely EOS S3 isn't in programming mode. Repeat steps listed earlier and try
again.

All of the following steps assume that we are in this state - inside the Docker
container with `qorc-sdk` environment enabled, in container's home directory.

## Building - easy

> Here will be description of how to build whole project with one command, when
it will be ready.

## Building - advanced

Components of TwPM may also be built separately. Such approach is most useful
to developers and [hackers](https://en.wikipedia.org/wiki/Hacker_culture) to
quickly test modified code without having to build everything from scratch.

<!-- when "-" is in path, the link in mkdocs is generated with a single
character, not three -->
<!-- markdownlint-disable MD051 -->
Note that if you already have built whole project earlier, `make` should be able
to detect which components have changed and rebuild only those. In that case you
can follow [the easy path](#building-easy).
<!-- markdownlint-enable MD051 -->

### MCU

> Here will be description of building and hacking TPM stack and platform glue
code.

### FPGA

To just build:

```shell
cd workspace/fpga
make
```

This will execute all steps up to and including generation of bitstream. This
process takes few minutes, most of that time is consumed by `symbiflow_route`.
It may appear to be stuck, as this process doesn't output any lines on terminal.
It also almost doesn't access disk at all so there will be no HDD LED activity,
but it takes a lot of CPU time.

To build and flash:

```shell
cd workspace/fpga
make flash
```

Note that this just saves the bitstream in the flash. It is up to MCU code to
load it and release FPGA from reset, as well as configure I/O ports for being
used by FPGA, so for some changes flashing just the FGPA part won't be enough.
