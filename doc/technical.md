# Technical

## Controller / Node

Each Pico PI 2 can handle one USB Device. So to connect the KVM Switch to multiple computers, 1 Pico PI is used per
computer.

The main Pico is the one with the USB ports for the HID devices, it's named the Controller (`src/controller`), and any
additional pico pi is named a Node (`src/node`).

The Controller and the Nodes are communicating using SPI. On the Controller it's using the `spi0` with the default pins
And on the first nodes it's using the `spi1` with the GPIO 12,13,14,15 to simplify cable management between the two
boards.

Each node also uses 2 GPIO:

- `spi_ready_gpio` use to notify the Controller when the node is listening on SPI and the controller can communicate
  with them.
- `data_available_gpio` use to notify the Controller when the node has data available to be read.

When there are two or more nodes, the controller uses another GPIO (`spi_selector_gpio`) to select which node to
communicate with.

When the Controller boots, it restarts all the nodes (with GPIO 15 connected to `RUN` on the nodes) and waits for them
to be ready (Waiting for an `GPIO_IRQ_EDGE_RISE` event on `spi_ready_gpio`)

## USB Device – Connected to a computer

See `usb_device.c`

The USB Port used to act as a USB device from a computer point of view is the micro USB port that comes with the
PICO-PI.
This allows running some logic on the same core that handles this usb port with Tiny USB, since this USB port is managed
by the RP2350 the timings are not as critical as when using Pico-PIO-USB.

## USB Host – Connected to the HID Devices (Keyboard / Mouse / ...)

See `usb_host.c`

The USB Host part is used to handle the HID Devices plugged into the KVM Switch. Pico-PIO-USB with TinyUSB is used to
handle the USB communication with the HID Devices. And the Core 1 on the Controller is dedicated to handle the USB
communication with the HID Devices.

The USB ports have both D+ and D-, pulled down to act as a USB host.

## HID Devices

The HID devices are connected to the USB Host part of the Controller. When a HID device is mounted, the controller reads
the HID report descriptor for every HID interface and then exposes the report descriptors as its own on the computer
side (see `hid_manager.c`). Then, when an HID Device is sending a report (when a key is pressed, for example), the
report is forwarded as-is to the selected computer.

The HID report descriptor is also decoded (see `hid_report_descriptor.c`) and any keyboard report is then interpreted to
detect if a shortcut has been pressed (see `kvm_switch_config.c`).

## WebUSB

See `web_usb_handler.c`

To allow a simple configuration of the KVM Switch, WebUSB is used to communicate with the KVM Switch from a web browser.
A Vendor interface is exposed to the computer (see `#if CFG_TUD_VENDOR` blocks in `usb_device.c`).

When a config is changed, the config is then saved to the flash. It's written on the fourth sector from the end of the
flash (see `config_persistence.c`).

## Cpu Core Usage

On the controller core 0 runs the main logic, core 1 is dedicated to handle the USB communication with the HID Devices.
On the nodes, core 0 runs the main logic, and core 1 is dedicated to the SPI communication with the controller.

## System clock

The system clock is set to 144 MHz because Pico-PIO-USB needs a multiple of 12 MHz.