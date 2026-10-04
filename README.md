# meta-i2c-sensor-demo — Embedded Linux Portfolio, project 2/3

Custom Yocto layer for the `qemuarm64` machine, built on Poky (scarthgap, 5.0).
Demonstrates extending a Yocto build with custom recipes, rather than just
building a stock reference image — including porting kernel modules from
project 1 ([p1-i2c](https://github.com/AlbertPhaseLab/p1-i2c)).

## Contents

- `recipes-kernel/hello/` — recipe for `hello.ko`, the first kernel module from
  project 1, rebuilt via Yocto's `module.bbclass`. Auto-loads at boot
  (`KERNEL_MODULE_AUTOLOAD`), verified in QEMU (`lsmod` / `dmesg`).
- `recipes-kernel/demo-sensor/` — recipe for `demo_sensor.ko`, the hwmon I2C
  driver from project 1 (manual instantiation and Device Tree matching, hwmon
  exposure, `dev_err_probe` error handling). Requires `i2c-stub` loaded with a
  simulated chip before probing, same manual flow as project 1's week 2.
  Verified: `temp1_input` reads 291250, identical to the hand-built driver.
- `recipes-kernel/linux/linux-yocto/i2c-stub.cfg` — kernel config fragment
  enabling `CONFIG_I2C_STUB` and `CONFIG_I2C_CHARDEV`, applied via a
  `.bbappend` instead of editing `.config` by hand (the Yocto-reproducible
  equivalent of `scripts/config` from project 1).

## Usage

With a poky checkout (branch scarthgap) and a qemuarm64 build directory set up:

    bitbake-layers add-layer /path/to/meta-i2c-sensor-demo

Add to local.conf:

    IMAGE_INSTALL:append = " hello demo-sensor i2c-tools kernel-module-i2c-stub"

Then:

    bitbake linux-yocto -c kernel_configcheck -f
    bitbake core-image-minimal
    runqemu qemuarm64 nographic slirp

Inside the image:

    insmod /lib/modules/$(uname -r)/kernel/drivers/i2c/i2c-stub.ko chip_addr=0x48
    i2cset -y 0 0x48 0x00 0x1234 w
    insmod /lib/modules/$(uname -r)/updates/demo_sensor.ko
    echo demo_sensor 0x48 > /sys/bus/i2c/devices/i2c-0/new_device
    cat /sys/class/hwmon/hwmon0/temp1_input   # 291250

Kernel's own modules (e.g. i2c-stub) are packaged individually by Yocto as
kernel-module-<name> — building a module isn't enough to have it included in
the image, it must also be added to IMAGE_INSTALL.

## Status

- **Week 1, day 2** — custom layer created, `hello` kernel module ported from
  project 1 and auto-loading at boot, verified in QEMU.
- **Week 1, day 3** — `demo_sensor` (I2C hwmon driver) ported, with an
  `i2c-stub` kernel config fragment. Verified end-to-end in QEMU, matching
  project 1's results exactly.
