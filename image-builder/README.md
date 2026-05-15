# image-builder — SD Card Image Builder

Docker-based tooling to build complete Raspberry Pi OS SD card images with all Satellite1 components pre-installed and pre-configured.

> **Status: Work in Progress**

## Overview

The `image-builder` creates ready-to-flash SD card images that include:

- Raspberry Pi OS (Bookworm) base
- Custom kernel with FUSB302 USB-C PD support
- Satellite1 device tree overlays and ALSA config
- Satellite1 Python SDK and CLI
- Pre-configured WiFi and system settings (optional)

This enables one-step deployment: flash the image, boot the Pi, and the Satellite1 HAT is immediately operational.

## Current Status

⚠️ **This component is not yet complete.** The image builder is under active development. The following describes the intended workflow and current progress.

## Source Layout

```
image-builder/
├── Dockerfile              # Build environment (Debian + pi-gen + tools)
├── Makefile                # Build orchestration
├── config/                 # pi-gen configuration (staged customizations)
├── stage-sat1/             # Custom stage scripts for Satellite1 integration
└── build-assets/           # Built .deb packages (input to image build)
```

## Prerequisites

- Docker
- `make`
- Significant disk space (several GB for build artifacts)
- Internet connection (for downloading base packages and Raspberry Pi OS packages)

## Build Process

### Assembly of dependencies

Before building an image, collect all required `.deb` packages:

```
build-assets/
├── pkg.list                # List of .deb filenames to include
├── satellite1-rpi-sdk_*.deb
├── satellite1-rpi-setup_*.deb
└── linux-image-6.12.58-fusb302-rpi-v8_*.deb
```

These can be built separately via the top-level Makefile or manually copied.

### Building the image

```bash
# Build the Docker image
make docker-image

# Build the Raspberry Pi OS image
make pi-image
```

This will:
1. Start a privileged Docker container with pi-gen
2. Mount local config/stage directories
3. Run `./build.sh` from the pi-gen repository
4. Copy the resulting image from the container to `./deploy/`

Output: `deploy/Satellite1-SDK-*.zip` containing the raw `.img` file.

### Interactive mode

For debugging or manual image customization:

```bash
make shell
```

Opens a shell inside the container where you can run `./build.sh` manually or examine the pi-gen configuration.

## Configuration

Image creation is driven by [Raspberry Pi pi-gen](https://github.com/RPi-Distro/pi-gen) with customizations:

- `config/` — contains pi-gen config files to skip default stages and enable Satellite1-specific setup
- `stage-sat1/` — custom stage scripts that install your `.deb` packages and perform final configuration

## Customization points

| File/Directory | Purpose |
|----------------|---------|
| `config/` | pi-gen YAML config; defines which stages to run |
| `stage-sat1/*` | Shell scripts executed during build; install packages, write files, enable services |
| `build-assets/pkg.list` | List of `.deb` filenames to `dpkg -i` during the build |

## Expected output

After a successful build, the `deploy/` directory contains:

```
deploy/
└── Satellite1-SDK-<date>.zip
    └── satellite1-sdk-<date>.img  (FAT32 boot partition + ext4 root partition)
```

Flash this image to an SD card using `dd`, Raspberry Pi Imager, or similar.

## Known limitations / TODOs

- The full build pipeline is not yet validated end-to-end
- Stage scripts need to be written/refined to install packages in correct order
- Network configuration (WiFi) preseed is not yet implemented
- First-boot setup (hostname, user creation) may need customization
- Image signing and release packaging not implemented

## Dependencies

The Dockerfile installs pi-gen dependencies:

- `e2fsprogs`, `parted`, `qemu-user-static`, `debootstrap`, `zerofree`
- `zip`, `dosfstools`, `libcap2-bin`, `rsync`, `curl`, `xz-utils`
- Plus standard GNU utilities

## Development workflow

1. Build all three `.deb` packages first (kernel, setup, sdk)
2. Copy packages to `build-assets/` and update `pkg.list`
3. Run `make pi-image`
4. Test the resulting image on actual hardware
5. Iterate on `stage-sat1/` scripts as needed

## Repository

https://github.com/futureproofhomes/Satellite1-RPi/tree/main/image-builder
