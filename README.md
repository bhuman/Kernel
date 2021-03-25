# Kernel

This is a linux 5.4-rt kernel intended for SoftBank Robotics NAO V6 with some of the changes ported from [SoftBank's 4.4 kernel](https://github.com/aldebaran/linux-aldebaran/tree/sbr/v4.4.86-rt99-baytrail).

The modifications (compared to the standard 5.4-rt kernel) are:
- a workaround by SoftBank in `arch/x86/mm/ioremap.c` for the congatec board
- the CGOS driver 1.03.029 by congatec
  - including a patch by SoftBank to enable 32-bit compat ioctls (which is only necessary as long as 32-bit SoftBank executables are used)
- the SoftBank driver `cgos-sbr` that exports some data as sysfs files
  - modified for 5.4 due to removal of a crypto API
- the SoftBank driver `sbre-usb-i2c` for an STM32 that provides a USB-I2C bridge
  - modified for 5.4 because USB transfers cannot use stack memory
- the SoftBank driver `sbre-dfuse` that manages the firmware upload process of the USB-I2C bridge
  - modified for 5.4 because USB transfers cannot use stack memory
- add a quirk flag for a SDHCI device (by SoftBank)
- the SoftBank patch for the audio device WM8860

Furthermore, we include a configuration (`.config`) that seems to work.

## Compiling

On Linux, assuming `${installPath}` has been set to a directory where the built kernel shall be placed:

```bash
LOCALVERSION="" make ARCH=x86_64 -j$(nproc)
mkdir -p "${installPath}"
INSTALL_PATH="${installPath}" make ARCH=x86_64 install
make INSTALL_MOD_PATH="${installPath}" ARCH=x86_64 modules_install
rm "${installPath}"/lib/modules/*/source
rm "${installPath}"/lib/modules/*/build
```

## Known Issues

With this kernel, the root device is called `/dev/mmcblk1` instead of `/dev/mmcblk0` (as there are two SDIO controllers in the system). This is almost no problem because most parts of the system use UUIDs, but it breaks the `kexec` call after flashing the robot, as that passes `root=/dev/mmcblk0p3` to the kernel. This means you either need to patch the installer in the `.opn` image to do a proper reboot or set the "halt after upgrade" flag in order to shut the robot down.
