# Roadmap

[TwPM project](https://nlnet.nl/project/TwPM/index.html) is funded by the
[NLnet Foundation](https://nlnet.nl) via the
[NGI ASSURE](https://www.assure.ngi.eu/).

[3mdeb](https://3mdeb.com/) has proposed to implement the following tasks under
the grant agreement No 957073.

## 1. Public site with documentation

All of the documentation produced during this project should be publicly
available to users and developers.

Milestones:

- create repositories for the project
- prepare server and domain name
- describe project's purpose and phases
- create placeholders for description of deviations from TPM specification,
software stack and documentation changelog that will be updated after each of
the remaining phases

State: [Done (this website)](../index.md)

## 2. Gather hardware requirements and choose target board

TPMs are connected to the mainboard of PC. They have maximum allowed power
consumption and boot time. Based on those requirements and additional factors
like cost, availability and ease of use with open source tools we have to choose
a board for reference implementation.

Milestones:

- documentation of hardware requirements
- analysis and comparison of available boards
- establishment of reference board
- review and update existing documentation, add entry to changelog

State: [Done](../changelog/#2023-04-20)

## 3. Implement LPC protocol in FPGA

While the newest computers use SPI for connecting TPM devices to the mainboard,
slightly older, but still widely used hardware uses Low Pin Count (LPC)
interface. Implementing this protocol at software level through bit-banging
would require very high speed micro-controllers, which would make the cost and
power consumption unreasonably high. To the best of our knowledge, there are no
MCUs that have LPC controller included as a hardware part of SoC, due to the
fact that LPC is specific to PC only. For these reasons hardware implementation
is required.

Milestones:

- design and implementation of LPC peripheral in Verilog
- simulation of synthesized code
- documentation describing connection between designed TPM and mainboard
- review and update existing documentation, add entry to changelog

State: [Done](../changelog/#2023-04-20)

## 4. Implement basic TPM registers in FPGA

Most of the hardware TPM registers must return result immediately or almost
immediately. MCUs and their communication with FPGA are not fast enough to
acknowledge, parse, prepare response and send it to host in time. Hardware
implementation is required for registers that require fast response.

Milestones:

- implementation of TPM register space
- implementation of finite state machine for changing localities
- review and update existing documentation, add entry to changelog

State: [Done](../changelog/#2023-05-24)

## 5. Implement TPM command parsing and communication between FPGA and MCU

Minimal parsing of commands and responses (limited to just their sizes) must be
done on FPGA side in order to properly set status bits that host can use to
check whether TPM expects more bytes of command or has more bytes of response.
Full command parsing and execution takes place on MCU, so FPGA has to implement
and expose buffer with command sent by host, along with any required metadata
like type of message in the buffer or currently active locality.

Milestones:

- implementation of FIFO on FPGA
- implementation of command machine state
- design, implementation and documentation of protocol of communication between
FPGA and MCU
- application code for reading data from FIFO, passing parsed commands to the
TPM stack and writing responses back to FIFO
- review and update existing documentation, add entry to changelog

State: In progress

## 6. Base tests

For testing of the implementation done so far, a subset of tpm2-tools commands
will be used. Only commands that do not depend on Protected Storage, RNG and
Primary Keys Certificates will be tested at this point.

Milestone:

- test suite: PCR operations (read, extend, reset values, locality protection)
- test suite: object creation (primary, ordinary, derived objects of various
types)
- test suite: cryptographic support functions (hash, HMAC, encryption, signing)
- documentation of results
- review and update existing documentation, add entry to changelog

State: In progress

## 7. Implement SPI TPM protocol

SPI implementation on MCU may not be feasible because some of the registers must
return proper values immediately. This task consists of repeating all the tasks
that were done on FPGA side for LPC, but this time SPI is used as physical
interface.

Milestones:

- design and implementation of LPC peripheral in Verilog
- simulation of synthesized code
- documentation describing connection between designed TPM and mainboard
- implementation of TPM register space
- review and update existing documentation, add entry to changelog

State: Backlog

## 8. Explore the usage of using simpler hardware platform

Additionally, we want to explore whether we can meet the TPM specification
requirements using simpler (and cheaper) microcontroller platform (with no FPGA
involved). That may not be feasible due to the hardware limitations, but it
is a great potential opportunity of increasing the adoption of the solution,
so that is why believe it makes sense to try to purse that and publish the
results.

Milestones:

- explore usage of the solution using simpler hardware platform (with no FPGA)
- publish test results to the public documentation website
- review and update existing documentation, add entry to changelog

State: In progress

## 9. Flash driver for TPM stack

Nonvolatile storage is an integral part of TPM. It allows for saving user- or
vendor-defined data inside TPM, potentially with protection based on state of
TPM (PCR values, authorization sessions). With NVRAM implemented, additional
tests can be performed.

Milestones:

- flash driver for chosen board (if not available yet or has limited support)
- integration of the TPM stack with flash driver
- test suite: NV memory (persistent objects and data, vendor certificates)
- test suite: attestation and authorization (quote, authorization sessions)
- test suite: use in real-world scenario (Fobnail)
- review and update existing documentation, add entry to changelog

State: Backlog

## 10. Unique identification and randomness source

Each TPM has to be uniquely identifiable. This uniqueness is used e.g. to create
primary seeds which are used to derive primary keys for various hierarchies.
Random number generator is also included in this task - unique registers (with
e.g. serial numbers) and RNG engines are usually specific to the given hardware.
FPGA can also be used if any of those isn't available or doesn't have enough
entropy on MCU.

Milestones:

- find and obtain enough bits of unique data identifying the platform
- find and obtain enough bits of entropy for seeding PRNG
- test suite: Windows HLK
- review and update existing documentation, add entry to changelog

State: Backlog

## 11. Manufacturing process

Each TPM must be individually manufactured. This consists of committing vendor
certificate for TPM's primary Endorsement Key (EK) to its nonvolatile memory. As
each EK is unique, so is its certificate, and it must be sign by key which chain
of trust is rooted in publicly available vendor's root certificate.

Milestones:

- create manufacturing process
- post the process on the documentation site
- create script for automation of manufacturing process
- review and update existing documentation, add entry to changelog

State: Backlog

## 12. Customizable configuration

Platforms other than the reference one may support different functionalities, or
may have limited performance. To make transition to different hardware easier,
some options should be made configurable. This may include available hash
algorithms in TPM stack, physical interface used (LPC or SPI), presence of
hardware RNG engine, amount of nonvolatile memory.

Milestones:

- create easy to use build system integrating whole stack
- prepare configuration file for whole project
- review and update existing documentation, add entry to changelog

State: Backlog
