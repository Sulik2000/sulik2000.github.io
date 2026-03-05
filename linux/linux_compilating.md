# How I compiled Linux with NFS root file system for STM32MP157D-DK1 board

## TF-A, OP-TEE and U-Boot
I couldn't compile these three sources cause in official GitHub repositories there is no Device Tree support for this board, there is only for STM32MP157A-DK1, and using this U-Boot doesn't print anything to console. And compiling TF-A without OP-TEE(`AARCH32_SP=sp_min`) isn't recommended by STM documentation

In the beginning I used STM forks of TF-A, OP-TEE and STM32, all was good, but U-Boot didn't output anything in console. The reason was in incorrect format of SD-card, correct format of SD-card in `lsblk -o NAME,PARTLABEL,SIZE` is:

```text
NAME   PARTLABEL   SIZE 
sda               14.8G 
├─sda1 fsbl1       256K # Trusted Firmware-A (Binary)
├─sda2 fsbl2       256K # TF-A (Backup)
├─sda3 metadata    256K # TF-A (State)
├─sda4 fip           4M # FIP(Contains U-Boot and OP-TEE)
└─sda5 u-boot-env  256K # Persistent storage for U-Boot variables
```
You can do this partitions by command:
```text
sudo sgdisk -o -a 1 /dev/sda -n 1:34:545     -c 1:fsbl1 \-n 2:546:1057   -c 2:fsbl2 \-n 3:1058:1569  -c 3:metadata \-n 4:1570:9761  -c 4:fip -n 5:9762:10273 -c 5:u-boot-env
```

- `fsbl1` and `fsbl2` will contain raw bits of TF-A binary after compilation. You can do that(`sda` is example name of SD-card) by `dd if=... of=/dev/sda1 conv=fdatasync status=progress bs=1M`. 
- `metadata` partition is needed by TF-A
- `fip` will contain `fip.bin` whit is result of compilation of TF-A. Use same command as with fsbl1 and fsbl2
- `u-boot-env` partition isn't necessary, but is useful for saving environment variables by `saveenv` in U-Boot. You don't need to write anything by yourself, this is raw data which is used by U-Boot

After this all was good, U-Boot was working, but when I came to Linux, I faced with another problem

## Problems with launching Linux

When you compile this steps, you'll face with another problem. Linux will not launch printing similar lines:

```text
[   13.304308] platform 5a006000.usbphyc: deferred probe pending: (reason unknown)
[   13.310371] platform 5800d000.usb: deferred probe pending: platform: wait for supplier /soc/usbphyc@5a006000/usb-phy@0
[   13.320994] amba 58005000.mmc: deferred probe pending: amba: wait for supplier /soc/bus@5c007000/i2c@5c002000/stpmic@33/regulators/buck4
[   13.333312] platform 50001000.pwr: deferred probe pending: platform: wait for supplier /soc/bus@5c007000/i2c@5c002000/stpmic@33/regulators/ldo4
[   13.346237] platform 49000000.usb-otg: deferred probe pending: platform: wait for supplier /soc/usbphyc@5a006000/usb-phy@1
```

Problem was as figured out in OP-TEE, cause OP-TEE wasn't using SCMI protocol, which is required by Linux. You can try to configure Linux to not using SCMI, and I think this is better solution. But then I used another way.

I compiled U-Boot using STM fork(if you use official repo, there isn't correct device tree, and you'll get silent U-Boot) and new official versions of OP-TEE and TF-A(because there is `stm32mp157a-dk1-scmi.dts` in OP-TEE which provides support for SCMI protocol).

Then you'll successfully get Linux panic cause your SD-card lacks a rootfs partition.

## Problems with SD-card

I after launching Linux did set up NFS rootfs by BusyBox and then Linux successfully launched. But when I tried to launch with rootfs on SD-card, Linux gave similar errors, and I don't know how to figure out this. I'm sure that problem is in Device Tree for OP-TEE or Linux, but I don't know what exact is.