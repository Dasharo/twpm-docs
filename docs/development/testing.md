<!--
SPDX-FileCopyrightText: 2024 3mdeb <contact@3mdeb.com>

SPDX-License-Identifier: CC-BY-SA-4.0
-->

# Testing

## Overview

This document presents the current status and progress of testing the TwPM
module features. The tests used here are located in the
[Dasharo OSFV repository](https://github.com/Dasharo/open-source-firmware-validation/blob/main/dasharo-security/tpm2-commands.robot).

## Tests results

The latest results (as of 14/03/2024) can be found
[here](/test-results/2024_03_14_orange_crab_without_create_primary.html). It was
run on Protectli VP6670, TwPM was connected through SPI interface. Previous
version (running on LPC) is available
[here](/test-results/2024_01_11_orange_crab_without_create_primary.html).

## Hardware setup

### Hardware list

* [Orange Crab](https://github.com/orangecrab-fpga/orangecrab-hardware)
    - The TwPM is implemented on this board
* One of:
    - [Protectli VP4670](https://docs.dasharo.com/variants/protectli_vp46xx/overview/),
      the TwPM is connected to LPC TPM header on this board
    - [Protectli VP6670](https://eu.protectli.com/product/vp6670/),
      the TwPM is connected to SPI TPM header on this board

### Connection to the platform

Follow the [mainboard connection tutorial](/tutorials/mainboard-connection/).
In case of LPC, make sure that LCLK and LAD lines aren't directly next to each
other (e.g. separate those with GND), otherwise inter-signal noise would cause
bad reads. No such interference was observed for SPI.

![](/images/twpm_connection.png)

### Development connection

The Orange Crab still requires reflashing over UART during testing, that is why
the USB-UART converter must also be connected.

![](/images/twpm_connection_dev.png)

## Running tests

* Follow the
  [Getting started section](https://github.com/Dasharo/open-source-firmware-validation#getting-started)
  of test repository first

* Apply the
  [following diff](https://github.com/Dasharo/open-source-firmware-validation/issues/198#issuecomment-1893483736)
  if not using [RTE](https://shop.3mdeb.com/shop/open-source-hardware/rte/).
  Adjust OS credentials to your case. Follow that issue for progress on
  improving the testing experience with no additional hardware involved.

* Execute a selection of test:

> At the moment, CreatePrimary() is not supported, that is why tests using it
> are excluded from the suite. More information can be found in
> [this issue](https://github.com/Dasharo/TwPM_toplevel/issues/23).
> Replace `$DEVICE_IP` with the IP address of your device, where TwPM is
> connected. It is assumed running Ubuntu 22.04 OS with OpenSSH server enabled
> via password authentication. For this test suite, both VP4670 and VP6670 may
> use `protectli-vp4670` configuration.

```bash
robot -L TRACE -v device_ip:$DEVICE_IP -v config:protectli-vp4670 -v snipeit:no -t "TPMCMD00[0-469]*" -t "TPMCMD010*" dasharo-security/tpm2-commands.robot
```
