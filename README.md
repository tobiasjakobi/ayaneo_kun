# Ayaneo Kun

## Circuit board

Some pictures of the Kun's circuit board.

* [Picture 1](https://www.math.uni-bielefeld.de/~tjakobi/ayaneo_kun/ayaneo_kun_circuit_board_0.jpeg)
* [Picture 2](https://www.math.uni-bielefeld.de/~tjakobi/ayaneo_kun/ayaneo_kun_circuit_board_1.jpeg)
* [Picture 3](https://www.math.uni-bielefeld.de/~tjakobi/ayaneo_kun/ayaneo_kun_circuit_board_2.jpeg)
* [Picture 4](https://www.math.uni-bielefeld.de/~tjakobi/ayaneo_kun/ayaneo_kun_circuit_board_3.jpeg)
* [Picture 5](https://www.math.uni-bielefeld.de/~tjakobi/ayaneo_kun/ayaneo_kun_circuit_board_4.jpeg)
* [Picture 6](https://www.math.uni-bielefeld.de/~tjakobi/ayaneo_kun/ayaneo_kun_circuit_board_5.jpeg)

## Firmware EEPROM

The vendor/manufacturer of the EEPROM is `Winbond`, the product name is `W25Q256JWPQ`. Size is `32MiB` and interface is `SPI`.

The EEPROM is located directly above the APU, see `Picture 1`.

### Recovery

The following hardware is required for recovery.

* CH341A MinProgrammer USB programmer (easily available on Amazon)
* 1.8V level shifter for programmer
* alternatively: CH341A v1.7 USB programmer (this one can operate directly with 1.8V output level)
* DFN8/QFN8/WSON8 probe with 5x6 mm size (larger sizes don't fit the chip)

If you choose the v1.7 USB programmer you don't need the level shifter. Make sure you switch the programmer into 1.8V mode though. The EEPROM is only rated for 1.8V, hence higher voltages may permanently damage it.

Programming of the EEPROM can be done e.g. using [flashrom](https://flashrom.org).

## Information dumps

* Content of `/proc/cpuinfo`: [Link](dumps/cpuinfo.txt)
* Output of the `cpuid` utility by Todd Allen: [Link](dumps/cpuid.txt)
* Output of `lspci -v`: [Link](dumps/lspci.txt)

The following dumps were created using an Arch Linux install medium with a `6.6.8-arch1-1` kernel.

* Output of `dmesg`: [Link](dumps/dmesg.txt)
* Output of `ls /sys/devices/platform/`: [Link](dumps/sys_devices_platform.txt)
* Output of `ls /sys/module/`: [Link](dumps/sys_module.txt)

For each class in `/sys/class` we did a `udevadm info -a -p ${node_name}` for all nodes of this class type. The output was named `${class_type}_${node_name}.txt`.

* Tarball of all outputs: [Link](dumps/udev.tar)

## USB devices

* Output of `lsusb -t -v`: [Link](usb/tree.txt)
* Output of `lsusb -v -d 045e:028e`: [Link](usb/045e_028e.txt)
* Output of `udevadm info -a -p /sys/bus/usb/devices/1-4.1:1.0`: [Link](usb/xbox360_controller.txt)#

## Integrated Xbox360 controller

### Firmware upgrade

Fetch the controller firmware (Ayaneo calls it **handle firmware**) from their website. The firmware blob should be named something like `AB08_V1.5_10.bin`. Make sure the device is charged.

* Powerdown the device
* Disconnect any external power supply
* Disconnect any external USB devices
* Wait for at least 10 seconds (probably to give the controller hardware time to drain capacitors)
* Press down L3 (left controller thumbstick) and keep it pressed
* Press power button
* Wait until the bootloader comes up (e.g. GRUB) and unpress L3

Once the kernel has booted, you should now see a USB mass storage device that replaces the controller device usually found on the bus. The device has no partition layout, just one 8MiB (16391 512-byte blocks) sized FAT filesystem. Mount the filesystem, copy the firmware blob onto it and issue a `sync`. After a short while the device should drop of the USB. Powerdown the device again, and leave it for 10 seconds. Now power it up again normally.

### Firmware reset behaviour

For some reason the controller firmware seems to implement some very strict watchdog mechanism. If the firmware doesn't see a USB transfer from the host within a second (exact value is unknown), it resets the USB device. The device drops of the bus, the kernel unloads the driver. Once the device pops up again on the bus, it is then again probed by the kernel, which then loads the driver. This cycle of USB disconnect followed by almost immediate reconnect is repeated ad infinitum.

In fact this behaviour is not unique to this firmware: [Link](https://lore.kernel.org/linux-input/unufo3$det$1@ciao.gmane.io/T/)

A workaround consists of immediately opening the event device of controller and to keep it open. On the kernel driver level this results in periodic USB transfers to query button and other states. Due to these transfer happening several times each second the firmware watchdog is happy and keeps the device alive.

It is unclear why the firmware implements this behaviour, in particular because it is detrimental to powersaving. Also it is not known if the problem also appears on Windows. The kernel driver on Windows might be implemented differently, e.g. it could request periodic USB transfer regardless if the input device is used or not.

A issue report is open on the [xpad issue tracker](https://github.com/paroj/xpad/issues/256).

### Workaround implementation

A draft implementation of the workaround can be found here: [Link](controller/)

It consists of a UDev rule, a systemd unit template and a C application. The rule automatically starts the systemd unit once the event device of the controller is available. The unit is just a thin wrapper around the application that keeps the event device open until the application is terminated by the unit again.

## AT keyboard

Some of the buttons are exposed through a regular AT keyboard. 

Power:
```
0xe05e 0xe0de 0xe05e 0xe0de (press+release)
```

Volume Down:
```
0xe02e (press)
0xe0ae (release)
```

Volume Up:
```
0xe030 (press)
0xe0b0 (release)
```

LC:
```
0xe01d 0xe05b 0xe03f (press)
0xe0bf 0xe0db 0xe09d (release)
```

RC:
```
0xe01d 0xe05b 0xe025 (press)
0xe0a5 0xe0db 0xe09d (release)
```

Start (upper-right corner of screen):
```
0xe01d 0xe05b 0xe040 (press)
0xe0c0 0xe0db 0xe09d (release)
```

Ayaneo (lower-right corner of screen):
```
0xe01d 0xe05b 0xe065 (press)
0xe0e5 0xe0db 0xe09d (release)
```

Hamburger (right of Ayaneo button):
```
0xe05b 0x20 (press)
0xe0db 0xa0 (release)
```

### Standard keyboard scancodes

Note that most of the buttons generate a multi-scancode sequence.

Some common scancodes are:
```
0xe05b = Left_Meta/Windows (press)
0xe0bb = Left_Meta/Windows (release)
0xe01d = Right_Ctrl (press)
0xe09d = Right_Ctrl (release)
```

## Problematic hardware

CS9711 fingerprint sensor by `Chipsailing Technology Co., Ltd.`

* USB vendor ID `2541`, and product ID `0236`
* no Linux kernel or userspace driver available
* no reverse engineering projects known

Ryzen AI

* AI accelerator integrated in the AMD 7840U APU
* Driver stack in development: [Link](https://github.com/amd/xdna-driver)
* Depends on IOMMU SVA patches: [Link](https://lore.kernel.org/linux-iommu/20231016104351.5749-1-vasant.hegde@amd.com/)

Bosch IMU

* confirmed by [Derek J. Clark](https://github.com/pastaq) that the devices houses a `BMI260`
* ACPI table information indicates a `BMI160` (done by the vendor to provide backward compatibility to old Windows driver)
* probing the IMU with the upstream `bmi160` fails
* out-of-tree kernel driver for the `BMI260` available: [Link](https://github.com/hhd-dev/bmi260)

CPU core management

* core scheduling on latest Linux stable might no be optimal
* P-state preferred core support is slated for Linux `6.9`: [Link](https://lore.kernel.org/linux-acpi/CAJZ5v0hRk3tME7yeC+1r0RM4-oPPrnSu2=JCsOshBbJp_Nq2Hg@mail.gmail.com/)
* Core performance boost (CPB) support is currently in review: [Link](https://lore.kernel.org/linux-pm/cover.1706255676.git.perry.yuan@amd.com/)

## Pending upstream patches

The following patches are currently pending.

* [drm: panel-orientation-quirks: Add quirk for Aya Neo KUN](https://patchwork.kernel.org/project/dri-devel/patch/20240310220401.895591-1-tjakobi@math.uni-bielefeld.de/)
* [drm/amd/display: Add MSF panel to DPCD 0x317 patch list](https://patchwork.freedesktop.org/patch/582247/)

THe following patches are not submitted yet: [Link](kernel_patches/)

## UEFI firmware analysis

Analysis was done using this blob: `AB08_Kun_7840U_P6C3L6M0C15_V1.9_EC_V0.0.58_7500_OnlyS4_S011.8_D01_54W_20230809.bin`

Interesting DXE drivers are:
* `Setup`
* `AmdPbsSetupDxe`

Interesting VarStores are:
* Setup (`EC87D643-EBA4-4BB5-A1E5-3F3E36B20DA9` with VarStoreId `0xf101`)
* AMD_PBS_SETUP (`A339D746-F678-49B3-9FC7-54CE0F9DF226` with VarStoreId `0x1`)
* UsbSupport (`EC87D643-EBA4-4BB5-A1E5-3F3E36B20DA9` with VarStoreId `0xb`)

### Setup

Interesting variables:
* Enable Hibernation (offset `0x32`)

### UsbSupport

Interesting variables:
* Legacy USB Support (offset `0x1`)
* XHCI Legacy Support (offset `0x2a`)
* XHCI Hand-off (offset `0x2b`)

### AMD_PBS_SETUP

Interesting variables:
* UCSI Support (offset `0x14`)
* USB4 ACPI _DEP Support (offset `0x465`)
* Sensor Fusion User Mode Driver (offset `0x3f`)
* AMD PMF Device Support (offset `0x9d`)
* S3/Modern Standby Support (offset `0x38`)
* Modern Standby Type (offset `0x41`)
