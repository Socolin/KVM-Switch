## [Unreleased]

## [1.0.0] - 2026-09-13

- KVM switch built from Raspberry Pi Pico 2 boards: one controller and one or more nodes (one per computer), linked over SPI.
- Support for any HID device (keyboard, mouse, gamepad, …) through PIO USB.
- Switch the active computer instantly with a keyboard shortcut.
- Shortcut action to change which computer a specific device is forwarded to.
- Web configurator over WebUSB (`tools/configurator`):
    - Edit general settings and keyboard shortcuts.
    - Inspect each device's HID descriptor, with decoded report fields.
    - View device logs, filterable by level.
- Configuration saved in flash and kept after a reboot.
