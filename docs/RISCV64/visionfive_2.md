---
title: StarFive VisionFive 2
author: Pratham Patel
tested with: TBD
tags:
  - riscv
  - starfive
  - visionfive
---

# The VisionFive 2 SBC's Wiki

The VisionFive 2 SBC is the second generation of SBC in the VisionFive SBC lineup. This lineup of SBCs contains SoCs with the CPUs that use the RISC-V open computing ISA.

This Wiki contains the necessary information a user needs to know before using Rocky Linux on the VisionFive 2 SBC.

## Technical guide

### Hardware specifications

| Item          | Description                                                                                            |
|---------------|--------------------------------------------------------------------------------------------------------|
| Processor/SoC | StarFive JH7110 (`rv64imafdcbx` with up-to 1.5GHz max freq)                                            |
| Memory        | LPDDR4: 2GB/4GB/8GB                                                                                    |
| Storage       | SD card slot                                                                                           |
|               | SPI flash for u-boot                                                                                   |
| Video Output  | HDMI 2.0                                                                                               |
|               | MIPI-DSI                                                                                               |
| Multimedia    | Camera with MIPI CSI (up-to 2160p@30fps)                                                               |
|               | Decoding: H.264 & H.265 at 2160p@60fps Encoding: H.264 & H.265 at 1080p@30fps JPEG Encoder and Decoder |
|               | 1/4-pole stereo audio                                                                                  |
| Connectivity  | 2x RJ45 Gigabit Ethernet (some models have 1x 100Mbit instead)                                         |
|               | 4x USB 3.0 (some models have 2x USB 2.0)                                                               |
|               | M.2 M-Key NVMe drive slot                                                                              |
| Power in      | USB-C port: Qualcomm Quick Charge 3.0 compatible                                                       |
|               | GPIO power in: 5V/3A                                                                                   |
|               | PoE ("pin compatible with RPi PoE HAT")                                                                |
| GPIO          | 40-pin GPIO header                                                                                     |
| Dimensions    | 100mm x 72mm                                                                                           |
| Button(s)     | Hardware reset button                                                                                  |
| Other         | Debug pin headers                                                                                      |

### Software status/information

At the time of writing this Wiki, the software support is still being upstreamed by the hardware vendor. More details about their upstreaming efforts and the merge status of each component can be viewed at [rvspace.org](https://rvspace.org/en/project/JH7110_Upstream_Plan).

#### Relevant software repositories:

Following are the necessary arch/board specific software repositories. These are here for curious tinkerers and Rocky Linux maintainers. Suffice to say, as a normal user, you are **not** expected to touch/compile any of these items :wink:

##### Fully upstreamed

- [opensbi](https://github.com/riscv-software-src/opensbi) (package name: `TBD`)

##### Downstream forks (upstream WIP)

- [u-boot](https://github.com/starfive-tech/u-boot) (package name: `TBD`)

- [linux](https://github.com/starfive-tech/linux/) (package name: `TBD`)

- [third party software tools/drivers](https://github.com/starfive-tech/soft_3rdpart/) (package name: `TBD`)

## Installation

To install Rocky Linux on the VisionFive 2 SBC, there is one pre-requisite that needs to be satisfied. That is to update the board firmware to v2.11.5.

**Editor's note: StarFive (the hardware vendor) is almost done sending in mainlining patches for u-boot and we should be able to switch to upstream u-boot soon. Our u-boot package for the VisionFive 2 should automatically update the board firmware to whatever version we support. But the following section is present nonetheless, for the users who have an outdated firmware than what we (Rocky Linux) have a minimum spec bar for.**

### Pre-requisite: Update board firmware

**NOTE: PLEASE READ THESE STEPS AND CAREFULLY FOLLOW THEM. IF DONE INCORRECTLY, YOUR BOARD MAY GET BRICKED!**

1. Download the following files:
- [sdcard.img](https://github.com/starfive-tech/VisionFive2/releases/download/VF2_v2.8.0/sdcard.img) (this image is from release v2.8.0)
- [u-boot-spl.bin.normal.out](https://github.com/starfive-tech/VisionFive2/releases/download/VF2_v2.11.5/u-boot-spl.bin.normal.out)
- [visionfive2_fw_payload.img](https://github.com/starfive-tech/VisionFive2/releases/download/VF2_v2.11.5/visionfive2_fw_payload.img)

2. `sudo dd if=sdcard.img conv=sync status=progress bs=1M of=/dev/sdX`

3. `mkdir temp-dir`

4. `sudo mount /dev/sdX4 temp-dir`

5. `sudo cp u-boot-spl.bin.normal.out visionfive2_fw_payload.img temp-dir/root/`

6. `sudo sync; sudo sync; sudo sync; sudo sync;`

7. `sudo umount temp-dir`

8. Eject the SD card from your computer, insert it in your VisionFive 2 and power it up. **The green LED should start blinking to indicate successful board power-up**.

9. Plug the network cable in the **Ethernet port that is next to the HDMI port**. (The other port has DHCP-related issues with early firmware.)

10. `ssh root@<IP_ADDRESS>` (passwd: `starfive`)

11. Run the command `cat /proc/mtd` and you should have the following output:
```
dev: size erasesize name
mtd0: 00020000 00001000 "spl"
mtd1: 00300000 00001000 "uboot"
mtd2: 00100000 00001000 "data"
```

12. **If and only if the partition information given above matches with yours**, update the `spl` and `uboot` partitions using the following commands:
```
flashcp -v u-boot-spl.bin.normal.out /dev/mtd0
flashcp -v visionfive2_fw_payload.img /dev/mtd1
```

The board firmware has now been updated!

## Current things that are WIP

A Rocky Linux image :stuck_out_tongue_closed_eyes:

## Maintainers

There are several developers who are working on the RISC-V port of Rocky Linux in general, but following are all the maintainers who are supporting the VisionFive 2 SBC. These usernames are what they use on the [Rocky Linux Mattermost chat](https://chat.rockylinux.org/).

 - [@codedude](https://git.resf.org/codedude)
 - [@skip77](https://git.resf.org/skip)
 - [@neil](https://git.resf.org/neil)
 - [@pgreco](https://git.resf.org/pgreco)
 - [@tgo](https://git.resf.org/tgo)
 - [@mustafa](https://git.resf.org/mustafa)
 - [@alangm](https://git.resf.org/alangm)
 - [@sherif](https://git.resf.org/sherif)
 - [@thefossguy](https://git.resf.org/thefossguy)
