# Testing

## Overview

This document presetns the current status and progress of testing the TwPM
module features. The tests used here are located in the
[Dasharo OSFV repository](https://github.com/Dasharo/open-source-firmware-validation/blob/main/dasharo-security/tpm2-commands.robot).

## Tests results

The latest results (as of 11/01/2024) can be found
[here](/test-results/2024_01_11_orange_crab_without_create_primary.html).

## Hardware setup

### Hardware list

* [Protectli VP4670](https://docs.dasharo.com/variants/protectli_vp46xx/overview/)
    - The TwPM is connected to LPC TPM header on this board, so we can test the
      TPM features of the TwPM
* [Orange Crab](https://github.com/orangecrab-fpga/orangecrab-hardware)
    - The TwPM is implemented on this board

### Connection to the platform

Follow the [mainboard connection tutorial](/tutorials/mainboard-connection.md).

![](/images/twpm_connection.png)

### Development connection

The Orange Crab still requires reflashing over UART during testing, that is why
the USB-UART converter must also be connected.

![](/images/twpm_connection_dev.png)

## Running tests

* Follow the
  [Getting started section](https://github.com/Dasharo/open-source-firmware-validation#getting-started)
  of test repository first

* Apply the [following
  diff](https://github.com/Dasharo/open-source-firmware-validation/issues/198#issuecomment-1893483736)
  if not using [RTE](https://shop.3mdeb.com/shop/open-source-hardware/rte/).
  Adjust OS credentials to your case. Follow that issue for progress on improving the testing experience
  with no additional hardware involved.

* Execute a selection of test:

> At the moment, CreatePrimary() is not supported, that is why tests using it
> are excluded from the suite. More information can be found in [this issue](TBD).

> Replace `$DEVICE_IP` with the IP address of your device, where TwPM is
> connected. It is assumed running Ubuntu 22.04 OS with OpenSSH server enabled
> via password authentication.

```bash
robot -L TRACE -v device_ip:$DEVICE_IP -v config:protectli-vp4670 -v snipeit:no -t "TPMCMD00[0-469]" -t "TPMCMD010" dasharo-security/tpm2-commands.robot
```
