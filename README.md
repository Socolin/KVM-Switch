# KVM Switch

A KVM Switch for any HID device built with multiple [Raspberry Pi Pico 2](https://www.raspberrypi.com/products/raspberry-pi-pico-2/)

## Status

This project is still in development. The code is almost done, and I need to design the PCB.

![Photo of the KVM Switch on a breadboard](doc/img/dev_breadboard.png)

So far I can use it as a KVM switch. I can select the computer to use with a keyboard shortcut. I was not able to
test with exotic keyboard / mouse yet, but those should be supported.

The detailed documentation is available at [doc/technical.md](doc/technical.md).

The remaining tasks are: 
- Test with 2 nodes (switch between 3 computers).
- Design PCB for 1 node (and maybe for 2 nodes later)
- Test with 3 hid devices.
- Test `TUD_OPT_HIGH_SPEED`

## Build

### Prerequisites

- cmake
- Clone https://github.com/raspberrypi/pico-sdk
- Clone https://github.com/raspberrypi/picotool

### Build

```sh
git clone git@github.com:Socolin/KVM-Switch.git
cd KVM-Switch
git submodule update --init --recursive
mkdir build
cd build
cmake .. -DPICO_BOARD=pico2 -DPICO_SDK_PATH={PATH_TO_PICOSDK} -DPICOTOOL_FETCH_FROM_GIT_PATH={PATH_TO_PICOTOOL}
make -j
# Then output is in src/node and src/controller
```

### Flash

To simplify flashing both, you can use the serial number of your pico with `picotool info -a` the serial will look like `46AA0E6255B66826`

```sh
picotool load --ser $PICO_PI_NODE_SERIAL -f -x src/node/kvm_node.uf2;
picotool load --ser $PICO_PI_CONTROLLER_SERIAL -f -x src/controller/kvm_controller.uf2;
```

## Hardware Architecture

### 2 Nodes With 2 HIDs

[See Schematic](doc/2_nodes_2_hid/2_nodes_2_hid.pdf)

## Software Architecture

![Diagram showing the software architecture of the project. Detailing which core execute which part and which the architecture of node and controller boards](doc/img/software_architecture.png)

## Configuration UI

The KVM Switch can be configured through a web UI. It can be accessed with a web browser supporting WebUSB.
You can access it at https://socolin.github.io/KVM-Switch/ (You need to use a browser supporting WebUSB)
The code of the config UI is in `tools/configurator`

![Web UI](doc/img/config-ui.png)

### Build

If you need to build the tool, you need [Node.js 24](https://nodejs.org/) then run those commands

```
cd tools/configurator
npm install
npm run start
```

Then access it at http://localhost:4200 (You need to use a browser supporting WebUSB)

## AI Usage

Basically, I see AI as a tool, and I have fun writing code, so:

- All the code in `src/` is written by a human.
- AI was used as a learning assistant, to help understand and learn all the USB / HID / Electronic stuff.
- AI was used to review the code.
- AI was used to generate some test cases (but the test logic is human-made) (see `AI-generated` comments).

## History

This project started to fix an issue I had with my current commercial KVM switch, that is the delay to switch between
2 computers; I wanted a shortcut to switch instantaneously. This project is also an opportunity for me to learn a some
few things with electronic and discover dev on a microcontroller. Feel free to provide any feedback, as I'm still
learning.

I first tried to use CH9329 and CH9350 to avoid all the USB parts and get this done quickly. However, during
my testing I discovered that the CH9329 has a HID descriptor that does not expose all the features I wanted (like the
mouse pan) and the CH9350 felt the same way. So I learnt a lot about HID descriptors, tinyUSB, etc… And, now this project
should support any HID device (not just keyboard and mouse) like Gamepad etc… I kept the code I used to use those chips
in `src/legacy` if anyone need this, feel free to use it
