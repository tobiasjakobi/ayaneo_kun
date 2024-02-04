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

# USB devices

* Output of `lsusb -t -v`: [Link](usb/tree.txt)
* Output of `lsusb -v -d 045e:028e`: [Link](usb/045e_028e.txt)
* Output of `udevadm info -a -p /sys/bus/usb/devices/1-4.1:1.0`: [Link](usb/xbox360_controller.txt)