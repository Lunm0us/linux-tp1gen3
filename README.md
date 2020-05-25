# Linux support patches for Lenovo X1 Tablet 3rd generation

 - [ACPI patches (volume/powerbutton and standby)](#acpi-patches)
 - [HID driver patches (keyboard functional keys and LEDs)](#linux-hid-patches)
  - [Installation on Arch Linux](#arch-linux-installation)
  - [Installation on Ubuntu (and possibly other distributions)](#ubuntu-installation-dkms)
 - [Additional Information](#additional-information)

## ACPI Patches

**Caution**: These steps will modify your computers firmware. This may break your device and/or attached hardware.

For S3 sleep state and the power/volume buttons of the device to work the ACPI DSDT tables must be modified. The patches contained in this repository
incorporate the information from the [Delta-Xi Blog][dxi]

Prerequisites:
 - Intel ACPI Source Language compiler which is usually provided by the acpica or acpica-tools package
 - make
 - one of the following BIOS versions:
   - N1ZET76W (1.32)
   - N1ZET79W (1.35)
   
   other versions may work but may also require changes to the patch. The current BIOS version can be checked via
   ```
   sudo dmidecode  --string "bios-version"
   ```
 - A Lenovo X1 Tablet 3rd Generation. To check if you have one of these devices you may run
   ```
   sudo dmidecode  --string "system-product-name"
   ```
   The output should beginn, according to the lenovo website with "20KJ" or "20KK". The device the patch was tested on returned "20KJ001NGE".

To apply the ACPI patches change to the acpi subdirectory and run make with the appropriate patch for your BIOS version.
Choose between `patch132` for version "1.32" and `patch135` for version "1.35".

```{.sh}
cd acpi
sudo make dsdt.dat
make patch135 compile
sudo make install
```

The patch should apply cleanly. If not you may patch the dsdt.dsl file by hand.
Finally add the acpi_override file as another initrd to your bootloader configuration.

```
initrd=/boot/acpi_override mem_sleep_default=deep
```

## Linux HID Patches

**Caution**: These steps will modify your kernel. Doing so might prevent your system from booting.

The attachable keyboard uses non standard keycodes for the functional keys and an additional USB endpoint for control of the LEDs. The provided sourcecode is a patched version of the upstream [hid-lenovo module][hid-lenovo]. For reference the patches (for the first generation device?) by [Dennis Wassenberg][hid-lenovo-patches] were used. Furhtermore this
repository contains a patched version of the upstream [hid-multitouch module][hid-multitouch] which fixes the pointstick for kernel versions > 5.4.11 (for details see [here][poinstick-issue]).

Prerequisites:
 - make
 - GCC C compiler
 - Linux headers/source for the currently installed kernel

### Arch Linux installation
When running Arch Linux you may build the DKMS package and install it via pacman:

```{.sh}
cd hid
makepkg .
pacman -U hid-lenovo-tp1gen3-dkms-0.2.0-1-x86_64.pkg.tar.xz
```

If you install the compiled module keep in mind, that you have to recompile the module every time your kernel is updated.

```{.sh}
cd hid
make
sudo make install
```

As the multitouch module is not an extension but a replacement of the upstream module, the latter must be blacklisted. While the Arch Linux package should add the necessary lines
automatically it might be necessary to regenerate the initramfs as the original module also must be replaced there. For details see [here][aw-blacklisting].


### Ubuntu installation (DKMS)

1. download and extract `linux-tp1gen3-master` 
```
cd linux-tp1gen3-master
```

2. edit `./hid/src/dkms.conf`
   Replace the placeholders **@_PKGBASE@** and **@VERSION@** with the actual values being hid-lenovo-tp1gen3 and the current version (e.g. 0.2.0). The version can be determined from the PKBUILD file.

3. install the DKMS package
```{.sh}
sudo apt-get install build-essential dkms 
```

4. copy files
```{.sh}
sudo mkdir -p /usr/src/hid-lenovo-tp1gen3-<version>
sudo cp -a ./hid/src/* /usr/src/hid-lenovo-tp1gen3-<version> 
```

5. build and install
```{.sh}
sudo dkms add -m hid-lenovo-tp1gen3 -v <version>
sudo dkms build -m hid-lenovo-tp1gen3 -v <version>
sudo dkms install -m hid-lenovo-tp1gen3 -v <version>
```
Check if the module was successfully added to dkms
```{.sh}
~$ dkms status
hid-lenovo-tp1gen3, 0.2.0, 5.6.14, x86_64: installed
```

6. blacklist the old module
   Add `blacklist hid-multitouch` to the file `/etc/modprobe.d/blacklist.conf`
   Then reboot.

## Contributions
Thanks goes to
 * [bitbacchus](https://github.com/bitbacchus) for writing the Ubuntu Installation section
 * [parenthetical](https://github.com/parenthetical) for providing the BIOS patch for version 1.35


## Additional Information

 * [intel hid module description (aka the volume buttons) ](https://lkml.org/lkml/2018/6/28/636)
 * [Kernel sleep state information](https://www.kernel.org/doc/html/v4.15/admin-guide/pm/sleep-states.html)
 * [Arch Linux Wiki on using the generated apci_override and DSDT in general](https://wiki.archlinux.org/index.php/DSDT#Using_a_CPIO_archive)

[dxi]: https://delta-xi.net/blog/#056 "Delta-Xi Blog"
[hid-lenovo]: https://github.com/torvalds/linux/blob/9f7582d15f82e86b2041ab22327b7d769e061c1f/drivers/hid/hid-lenovo.c "Linux hid-lenovo module sourcecode"
[hid-multitouch]: https://github.com/torvalds/linux/blob/9f7582d15f82e86b2041ab22327b7d769e061c1f/drivers/hid/hid-multitouch.c "Linux hid-multitouch module sourcecode"
[hid-lenovo-patches]: https://www.spinics.net/linux/fedora/linux-sound/msg00626.html "hid-lenovo: Add support for X1 Tablet special keys and LED control"
[aw-blacklisting]: https://wiki.archlinux.org/index.php/Kernel_module#Blacklisting
[poinstick-issue]: https://github.com/Lunm0us/linux-tp1gen3/issues/2

