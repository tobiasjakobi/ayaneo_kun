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

* currently unclear if the device houses a `BMI160` or a `BMI260`
* according to ACPI table information the IMU should be a `BMI160` (supported current Linux stable)
* probing the IMU fails, indicating that the ACPI information might be bogus
* out-of-tree kernel driver for the `BMI260` available: [Link](https://github.com/hhd-dev/bmi260)

CPU core management

* core scheduling on latest Linux stable might no be optimal
* P-state preferred core support is slated for Linux `6.9`: [Link](https://lore.kernel.org/linux-acpi/CAJZ5v0hRk3tME7yeC+1r0RM4-oPPrnSu2=JCsOshBbJp_Nq2Hg@mail.gmail.com/)
* Core performance boost (CPB) support is currently in review: [Link](https://lore.kernel.org/linux-pm/cover.1706255676.git.perry.yuan@amd.com/)
