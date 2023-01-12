---
layout: post
title:  "Adventures In Morello: Disabling Capability Load And Store Faulting"
author: john
date:   2023-01-12
---

## Introduction

The Morello architecture controls access to capabilities in memory, with the ability to generate a capability fault upon attempts to load or store valid capabilities. However, you may want to disable this behaviour to enable loading and storing of capabilities. In this blog post I look at what is required to achieve this.

## Disabling store faulting

Store faulting can be enabled and disabled in the block and page descriptors at stages 1 and 2 of translation. Specifically, we care here about 2 registers `TCR_ELx` for stage 1 translation and `VTCR_EL2` for stage 2. `x` corresponds to the relevant EL. If stage 2 transation is disabled then you don't need to worry about the latter register. The table below lists the registers that need to be considered for capability store faulting.

| Translation stage  | Register  | Bit  | Purpose | Value |
|---|---|---|---|---|
| 1  | TCR_ELx | CDBM (bit 59)  | Enables or disables tracking stores of valid capabilities, `0b1` enables tracking, `0b0` makes the register have no effect. For our purposes we'll make it have no effect.  | `0b0`  |
| 1  | TCR_ELx  | SC (bit 60)  | `0b0`: when CDBM is 0, fault otherwise no effect. `0b1`: has no effect.  | `0b1` |
| 1  | TCR_ELx  | HPD0 (bit 41) HPD1 (bit 42)  | HPD0 affects TTBR0_EL1 and HPD1 affects TTBR1_EL1. These need to be set to 1. | `0b1`  |
| 1  | TCR_ELx  | HWU060 (bit 44) and HWU160 (bit 48)  | Affects TTBR0_EL1 and TTBR1_EL1 respectively. These need to be set to 1 depending on the translation table used. | `0b1`  |
| 2  | VTCR_EL2 | CDBM (bit 59)  | Enables or disables tracking stores of valid capabilities, `0b1` enables tracking, `0b0` makes the register have no effect. For our purposes we'll make it have no effect.  | `0b0`  |
| 2  | VTCR_EL2  | SC (bit 60)  | `0b0`: when CDBM is 0, fault otherwise no effect. `0b1`: has no effect.  | `0b1` |
| 2  | VTCR_EL2  | HWU60 (bit 26)  | Needs to be set to 1. | `0b1`  |

## Disabling load faulting

In a similar fashion to the loading of capabilities can generate a capability fault to prevent unauthorised access. The below table lists the registers and bits needed to disable faulting. Once again, if stage 2 transation is disabled then you don't need to worry about VTCR_EL2.

| Translation stage  | Register  | Bit  | Purpose | Value |
|---|---|---|---|---|
| 1  | TCR_ELx | LC (bits 61 and 62)  | `0b00`: will zero capability tags, `0b01`: will have no effect, `0b10`: if CCTLR_ELx.TGENy is 1, fault loads of valid capabilities otherwise no effect, `0b11`: if CCTLR_ELx.TGENy is 0, fault loads of valid capabilities otherwise no effect. `x` and `y` refers to the translation table ie. TTBRy_ELx. | `0b01`  |
| 2  | VTCR_EL2  | LC (bit 61)  | `0b00`: zero capability tags. `0b01`: has no effect.  | `0b01` |
| 1  | CCTLR_ELx  | TGEN0 and/or TGEN1  | Apply to TTBR0_ELx and TTBR1_ELx. `0b0`: fault when TCR_ELx.LC is `0b11`, `0b1`: fault when TCR_ELx.LC is `0b10`. Only necessary if TCR_ELx.LC has been set to one of the values | `0b0` |


## Conclusion

After setting the relevant bits, capability faults from load and store operations should no longer be an issue!

## References

[Morello Specification](https://developer.arm.com/documentation/ddi0606/latest) <br>
[TCR_EL1](https://developer.arm.com/documentation/ddi0595/2021-06/AArch64-Registers/TCR-EL1--Translation-Control-Register--EL1-) <br>
[VTCR_EL2](https://developer.arm.com/documentation/ddi0595/2021-03/AArch64-Registers/VTCR-EL2--Virtualization-Translation-Control-Register)
