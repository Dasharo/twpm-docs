# Connecting TwPM to mainboard

This document describes how TwPM can be connected to the mainboard.

## WARNING

The TPM header is **not** standardized, make sure that you're using proper
pinout for your mainboard. Just because it physically looks identical (even
including key that is supposed to make connection of bad TPM type impossible)
doesn't mean it has the same signals on the same pins. The pinout is usually
described in the mainboard's manual, refer to it before attempting to connect
the module. **Wrong connection may break your TPM, mainboard, or both.** We do
not take responsibility for potential damage caused by it. You have been warned.

## Pinout for TwPM based on OrangeCrab

This is valid for TwPM built for [OrangeCrab](https://github.com/orangecrab-fpga/orangecrab-hardware)
from release v0.2.0.

Pinout from [official documentation](https://orangecrab-fpga.github.io/orangecrab-hardware/docs/pinout/):

![OrangeCrab pinout](https://orangecrab-fpga.github.io/orangecrab-hardware/docs/r0.2/OrangeCrab_r0.2_pinout.png)

Unfortunately, there are many different labels assigned to each pin. Each of the
types of identification is important to different group of people, or for
different tasks. The table below maps signal names to physical pin numbers (i.e.
as they are described in .lpf files, this is what FPGA tools need) and I/O
numbers (i.e. what is printed on the board, you will most likely use this to
find the correct pin).

| Signal name | Physical Pin | I/O number |
|-------------|:------------:|:----------:|
| LCLK        | H2           | 12         |
| LRESET#     | J2           | 13         |
| LFRAME#     | A8           | 11         |
| LAD[0]      | B8           | 10         |
| LAD[1]      | C8           | 9          |
| LAD[2]      | B9           | 6          |
| LAD[3]      | B10          | 5          |
| SERIRQ      | C9           | SCL        |

In addition to those signals, `3V3` and `GND` must also be connected to supply
power to the OrangeCrab.

If those pins need to be changed for any reason, it can be done in
[orangecrab.lpf file](https://github.com/Dasharo/TwPM_toplevel/blob/main/fpga/orangecrab.lpf).
Remember to change `SITE` of given signal to another physical pin, not only the
comment which contains I/O number. For OrangeCrab, mapping between I/O pins and
physical ports can be read from [schematics](https://github.com/orangecrab-fpga/orangecrab-hardware/blob/main/hardware/orangecrab_r0.2.1/plot/OrangeCrab.pdf).

## Mainboard pinouts

The presented list has only mainboards that are confirmed to be valid. You are
free to try it on the boards not listed below, assuming [you know what you're
doing](#warning).

### Protectli VP46xx

Protectli platforms from [VP46xx series](https://eu.protectli.com/vault-6-port/)
have header compatible with [TPM-01](https://kb.protectli.com/kb/tpm-on-the-vault/#articleTOC_1).
It is 2x10 pin header, with key on pin 4. It is dangerously similar to multiple
Gigabyte and Supermicro TPMs, but their layout is different so those are not
compatible.

![Protectli VP46xx TPM header location](https://kb.protectli.com/wp-content/uploads/sites/9/2023/06/VP4600_TPM_2-1024x732.jpg)

Pinout:

![Protectli VP46xx TPM pinout](/images/pinout-2x10-key4-vert.png){ align=left }

| Pin No. | Definition | Pin No. | Definition |
|:-------:|------------|:-------:|------------|
| 1       | LCLK       | 2       | GND        |
| 3       | LAD0       | 4       | key        |
| 5       | LAD1       | 6       | NC         |
| 7       | LAD2       | 8       | GND        |
| 9       | LAD3       | 10      | GND        |
| 11      | LFRAME#    | 12      | GND        |
| 13      | LRESET#    | 14      | NC         |
| 15      | NC         | 16      | NC         |
| 17      | GND        | 18      | VDD (3.3V) |
| 19      | SERIRQ     | 20      | NC         |

Some of the `NC` pins are actually used for other purposes. For clarity and to
avoid making mistakes they are not marked above.
