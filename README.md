# meta-myportfolio — Embedded Linux Portfolio, project 2/3

Custom Yocto layer for the `qemuarm64` machine, built on Poky (scarthgap, 5.0).
Demonstrates extending a Yocto build with a custom recipe, rather than just
building a stock reference image.

## Contents

- `recipes-kernel/hello/` — recipe for `hello.ko`, the first kernel module from
  project 1 ([p1-i2c](https://github.com/AlbertPhaseLab/p1-i2c)), rebuilt via
  Yocto's `module.bbclass` instead of a manual cross-compile. The module is set
  to auto-load at boot (`KERNEL_MODULE_AUTOLOAD`), unlike the manual `insmod`
  workflow used in project 1.
