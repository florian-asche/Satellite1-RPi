# Satellite1-RPi

**Raspberry Pi SDK for the Satellite1 HAT**

Satellite1-RPi is a complete software stack for running the Satellite1 HAT on a Raspberry Pi Zero 2WH. It provides a custom kernel with USB-C Power Delivery support, device tree overlays, ALSA audio configuration, and a Python SDK with CLI tools for hardware control.

**Target Platform:** Raspberry Pi Zero W2
**OS:** Raspberry Pi OS (Trixie)

> **⚠️ Early-stage development:**
> This is early-stage experimental software. No official support is provided yet. For issues and feature requests, open an issue on the GitHub repository:
> https://github.com/futureproofhomes/Satellite1-RPi/issues

## Overview

The repository is organized into four packages:


| Package                                         | Description                                                      | Status           |
| ------------------------------------------------- | ------------------------------------------------------------------ | ------------------ |
| [`satellite1-rpi`](satellite1-rpi/)             | Python SDK library and CLI tools (`sat1`)                        | Stable           |
| [`satellite1-rpi-setup`](satellite1-rpi-setup/) | Raspberry Pi configuration: overlays, ALSA config, init service  | Stable           |
| [`rpi-kernel-fusb302`](rpi-kernel-fusb302/)     | Custom kernel with USB-C Power Delivery (FUSB302) support        | Stable           |
| [`image-builder`](image-builder/)               | Automated SD card image generation (all components preinstalled) | In Progress ⚠️ |

## Quick Start

### Pre-Build Image

The fastest way to get started is to flash the pre-built [PiCompose Satellite1 Image](https://github.com/florian-asche/PiCompose). This single image includes the OS, custom kernel, overlays, SDK, and all configuration.

For the Installation steps please follow the [PiCompose documentation](https://github.com/florian-asche/PiCompose#installation).

### Manual Setup

If building from source or installing packages manually, follow these steps in order.

1. [Install the Custom Kernel](rpi-kernel-fusb302/README.md#installation)
2. [Install System Configuration](satellite1-rpi-setup/README.md#installation)
3. [Install the Python SDK](satellite1-rpi/README.md#installation)

## CLI Usage

The `sat1` command provides unified control over Satellite1 hardware subsystems.

[More documentation about that here](satellite1-rpi/README.md#cli-usage)

## Development

### Repository structure

```
Satellite1-RPi/
├── satellite1-rpi/          # Python SDK (pyproject.toml, src/, tests/)
├── satellite1-rpi-setup/    # System config (debian/, dt-overlays/, etc/)
├── rpi-kernel-fusb302/      # Kernel (config/, Dockerfile, build script)
├── image-builder/           # Image gen (Dockerfile, pi-gen config/stage)
└── README.md                # This file
```

## Troubleshooting

### DAC not responding

- Verify `satellite1-init.service` is active: `sudo systemctl status satellite1-init`
- Check that the I²C interface is enabled: `ls /dev/i2c-*`
- Ensure the FUSB302 kernel module is loaded: `lsmod | grep fusb302`

### CLI reports "connection refused"

The HAT may not be properly seated on the Raspberry Pi GPIO header. Power off, reseat the HAT, and power on again.

### Kernel module errors

Confirm you are running the custom kernel: `uname -r`. If still on the stock kernel, ensure `linux-image-6.18.29-fusb302-rpi-v8` is installed and the bootloader is configured to load it.

### Audio output silent

- Check ALSA configuration: `cat /etc/alsa/conf.d/50-satellite1.conf`
- Use `alsamixer` to ensure DAC and speaker outputs are unmuted
- Confirm `satellite1-init.service` successfully initialized the DAC

## License

See the [LICENSE](LICENSE) file. The code is provided as-is for hobbyist and evaluation use.
