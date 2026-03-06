# Power microcontroller fix

As I said here, I faced with [this](./linux_compilating.md) problem in the end, and I can show you solution.

## Linux, TF-A, OP-TEE sources
Use STM forks of TF-A, OP-TEE and Linux, because they have Device Tree for STM32MP157D-DK1.

## Fix
When you boot Linux Kernel, you can see this log message:
```text
cpu cpu0: error -ENODEV: _opp_set_regulators: no regulator (cpu) found
```

This means that STPMIC1 controller(this controller is responsible for power supply to MMC controller). You need to check if this configuration statements are correctly assigned:
```text
# Ensure the I2C bus for the PMIC is enabled
CONFIG_I2C_STM32F7=y

# Ensure the PMIC and its regulators are built-in
CONFIG_MFD_STPMIC1=y
CONFIG_REGULATOR_STPMIC1=y
CONFIG_REGULATOR_FIXED_VOLTAGE=y

# Ensure the SD card driver is built-in
CONFIG_MMC_STM32_SDMMC=y
```

Recompile Linux sources, and you'll find out that partitions on your SD-card are available for Linux.