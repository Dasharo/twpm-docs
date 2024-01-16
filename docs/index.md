<!--
SPDX-FileCopyrightText: 2024 3mdeb <contact@3mdeb.com>

SPDX-License-Identifier: CC-BY-SA-4.0
-->

# TwPM documentation

## About

Trust**worthy** Platform Module project aims to increase the trustworthiness of
the traditional
[TPM module](https://en.wikipedia.org/wiki/Trusted_Platform_Module)
(hence the T**w**PM), by providing the open-source firmware implementation for
the TPM device,
[compliant to the TCG PC Client Specification](explanation/compliance.md).

The main goal of the project is an attempt to create open-source firmware stack,
implementing the TCG PC Client Platform TPM Profile specification. Project aims
to use already available open-source software components whenever possible
(such as TPM simulators for TPM commands handling), while developing new code
when necessary (such as LPC FPGA module, or low-level TPM FIFO interface
handling).

Another challenge is to overcome hardware restrictions and allow users to use
the open-source TPM implementation on generally-accessible development boards.

## Community

TwPM project is a project of the Dasharo community. It’s an open-source project
that welcomes community contributions, suggestions, fixes, and other form
of feedback.

* [Join Dasharo Matrix Community](https://docs.dasharo.com/ways-you-can-help-us/#join-dasharo-matrix-community)
* [Join TwPM channel in Dasharo Matrix space](https://matrix.to/#/#twpm:matrix.org)
* [How to contribute](contributing/index.md)
* [Roadmap](roadmap/index.md)

## Funding

This project was partially funded through the
[NGI Assure](https://nlnet.nl/assure) Fund, a fund established by
[NLnet](https://nlnet.nl/) with financial support from the European
Commission's [Next Generation Internet](https://ngi.eu/) programme, under the
aegis of DG Communications Networks, Content and Technology under grant
agreement No 957073.

<center>
![](https://nlnet.nl/logo/banner.svg){: style="height:75px;text-align:right"}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
![](https://nlnet.nl/image/logos/NGIAssure_tag.svg){: style="height:75px"}
</center>
