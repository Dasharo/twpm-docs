# Compliance with specification

This page lists all recognized behaviors that differ from the specification. It
also documents implementation details that TPM standards left as vendor- or
implementation-specific.

## List of specifications

* [TCG PC Client Platform TPM Profile Specification for TPM 2.0, Version 1.05,
  Revision 14, 9/4/2020](https://trustedcomputinggroup.org/wp-content/uploads/PC-Client-Specific-Platform-TPM-Profile-for-TPM-2p0-v1p05p_r14_pub.pdf),
  shortened to PC Client PTP in this document

## Deviations from specifications

None discovered at this point.

## Implementation details

### PC Client PTP, 6.5.2.4

> Any write operation to the TPM_ACCESS_x register with more than one field set
> to a 1 MAY be treated as vendor specific.

This implementation acts on least significant set bit. Reasoning behind such
approach is that a write tries to request the use of locality peacefully before
seizing it, if both corresponding bits in `TPM_ACCESS_x` register are set.
