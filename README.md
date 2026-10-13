# SPIS

Simple PI SendSpin client

A minimal Raspberry Pi OS Lite image for Raspberry Pi that runs [SendSpin](https://github.com/Sendspin/sendspin-cli) - a high-quality audio streaming protocol for Home Assistant's Music Assistant.

## Features

- **Minimal footprint** - Built on Raspberry Pi OS Lite with only essential packages installed
- **Onboard audio by default** - Uses the Raspberry Pi's 3.5mm headphone jack out of the box, no extra hardware required
- **Optional USB DAC support** - Switch to a USB audio device via `/etc/asound.conf` if you want higher-quality output
- **Auto-start service** - SendSpin starts automatically on boot in headless mode via systemd
- **Low latency** - Direct ALSA hardware access for minimal audio buffering
- **Raspberry Pi optimized** - Built specifically for Pi 2/3/4 hardware

## Supported Hardware

| Board                       | Image   |
| --------------------------- | ------- |
| Raspberry Pi 2 Model B      | `armhf` |
| Raspberry Pi 3 Model B / B+ | `arm64` |
| Raspberry Pi 4 Model B      | `arm64` |

The `armhf` (32-bit) image also boots on the Pi 3 and 4, but `arm64` is preferred there.
A Pi 2 v1.1 cannot run 64-bit code: flashing the `arm64` image makes the green LED blink 7 times (kernel not found).

## Quick Start

### Download Pre-built Image

Download the latest image from [Releases](https://github.com/Poeschl/SPIS/releases)

Look out for the `*.img.xz` files, picking `arm64` or `armhf` for your board (see [Supported Hardware](#supported-hardware)). The `*-CREDENTIALS.txt` files contain the initial root password.

### Flash to SD Card

#### Linux / MacOS

```bash
dd if=SPIS-0.1.0-arm64.img of=/dev/sdX bs=4M status=progress
sync
```

Or [Raspberry PI Imager](https://www.raspberrypi.com/software/)

#### Windows

Use [Rufus](https://rufus.ie/) or the [Raspberry PI Imager](https://www.raspberrypi.com/software/)

### First Boot

1. Insert SD card into Raspberry Pi
2. (Optional) Connect a USB DAC if you don't want to use the onboard headphone jack
3. Power on
4. The player will automatically connect to Music Assistant on your network
5. Default credentials: `root` / a randomly generated password published alongside the release as `*-CREDENTIALS.txt` (change immediately!)

## Configuration

### Network Configuration

**Ethernet:** Works automatically via DHCP

**WiFi:** SSH into the device and configure via the points below

### Enabling WiFi

1. SSH into the device (via Ethernet, or a monitor/keyboard connected directly)
2. Set the WiFi country (required before the radio can be used, see [Troubleshooting](#wi-fi-blocked-by-rfkill) below):
   ```bash
   raspi-config
   ```
   Navigate to `5 Localisation Options` → `L4 WLAN Country` and select your country.
3. Connect to a network using `nmtui` (interactive) or `nmcli`:
   ```bash
   nmtui
   # or non-interactively:
   nmcli connect "SSID" password "your-password"
   ```
4. Verify the connection:
   ```bash
   nmcli device status
   ip a
   ```

### Hostname

Changing the hostname also changes the name of the SendSpin device in Music Assistant.

```bash
hostnamectl set-hostname mynewname
```

### Audio Device

By default, the system uses the onboard 3.5mm headphone jack (ALSA card `Headphones`). To use a USB DAC or another device instead:

1. List audio devices: `aplay -l`
2. Edit `/etc/asound.conf` and change the config to your device's card number
3. Restart sendspin: `systemctl restart sendspin`

## Building from Source

### Automated Build (GitHub Actions)

Every tagged push (`x.y.z`) triggers `.github/workflows/build-image.yml`, which builds a ready-to-flash image per architecture and publishes them as a GitHub Release:

- `SPIS-<version>-<arch>.img.xz` - the flashable image (`arm64` or `armhf`)
- `SPIS-<version>-<arch>-CREDENTIALS.txt` - the random root password generated for that build (you'll be forced to change it on first login)

The `arm64` image builds natively on an arm64 runner. The `armhf` image builds on an x86 runner under QEMU user emulation, since GitHub's arm64 runners can't execute 32-bit ARM code.

To build locally, run `sudo ./build.sh` for `arm64` or `sudo RASPIOS_ARCH=armhf ./build.sh` for `armhf` (on a non-ARM host this needs `qemu-user-static` and `binfmt-support`).

### Manual Installation

See [INSTALL.md](INSTALL.md) for complete step-by-step instructions to build your own image from scratch.

This method is recommended if you want to:
- Customize the installation
- Understand how the system works
- Build for different hardware configurations
- Troubleshoot issues

## Troubleshooting

### Audio Issues (Pops/Crackles)

Check buffer settings in `/etc/asound.conf`. Increase buffer_size if needed:
```
buffer_size 16384
```

### SendSpin Not Starting

```bash
# Check service status
systemctl status sendspin

# View logs
journalctl -u sendspin -e

# Manual test
sendspin --headless
```

### USB DAC Not Detected

```bash
# List USB devices
lsusb

# Check ALSA devices
aplay -l

# Check kernel messages
dmesg | grep -i audio
```

## Credits

- [Tycho-MEC/SASS](https://github.com/Tycho-MEC/SASS) for the base of this fork
- [SendSpin](https://github.com/Sendspin/sendspin-cli) by the SendSpin team
- [Raspberry Pi OS](https://www.raspberrypi.com/software/)
- [Home Assistant](https://www.home-assistant.io/) and [Music Assistant](https://music-assistant.io/)
