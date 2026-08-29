# OpenWrt Morse HaLow Firmware for Raspberry Pi 5

[![OpenWrt 23.05](https://img.shields.io/badge/OpenWrt-23.05-blue)](https://github.com/MorseMicro/openwrt)
[![Kernel 6.6](https://img.shields.io/badge/Linux%20kernel-6.6-lightgrey)](https://github.com/openwrt/openwrt)
[![Raspberry Pi 5](https://img.shields.io/badge/Raspberry%20Pi-5-c51a4a)](https://www.raspberrypi.com/products/raspberry-pi-5/)
[![Morse Micro HaLow](https://img.shields.io/badge/Morse%20Micro-HaLow-6f42c1)](https://www.morsemicro.com/)
[![Build walkthrough](https://img.shields.io/badge/YouTube-build%20walkthrough-red)](https://youtu.be/2y5DjqLjSXw)

![Haven Pi 5 — Raspberry Pi 5 with Morse Micro HaLow radio, UPS HAT, and battery stack](docs/haven-pi5.png)

OpenWrt-based Morse Micro HaLow firmware targeting the Raspberry Pi 5. Brings sub-GHz long-range 802.11ah (HaLow) Wi-Fi to the Pi 5 via Morse Micro radios.

This repo is a community backport. The [Morse Micro OpenWrt SDK](https://github.com/MorseMicro/openwrt) ships on OpenWrt 23.05 (kernel 5.15) and supports Pi 4. The bcm2712/RP1 hardware definitions from upstream OpenWrt 24.10 (kernel 6.6) have been vendored back so the same Morse SDK can build for Pi 5.

> **Just want to flash?** Download the pre-built Raspberry Pi 5 image from the [Releases](../../releases) section — no build required.

> **Field notes & test results:** The [Haven Guide](https://buildwithparallel.com/products/haven) has real-world range tests, hardware setup notes, and deployment tips from building with this firmware. Not required to build or flash — just useful if you want to go deeper.

> **Note:** This repo exists as a bridge. When Morse Micro releases their OpenWrt 24.x SDK — which is expected to include native Raspberry Pi 5 support — this repo will likely be superseded by the official upstream fork. Until that happens, this backport is the way to get HaLow running on Pi 5.

**Software Specifications**
- Morse Micro OpenWrt SDK, 23.05.5 base
- Linux kernel 6.6 (vendored from openwrt-24.10)
- mac80211 backports 6.12.61
- Morse Micro driver 1.16.4

## About

This firmware is part of [Build with Parallel](https://buildwithparallel.com/), a project focused on practical off-grid communications, resilient mesh networking, and field-ready Raspberry Pi builds.

## Project Links

| Link | What it is |
|---|---|
| [Releases](../../releases) | Pre-built Raspberry Pi 5 firmware images |
| [Build walkthrough video](https://youtu.be/2y5DjqLjSXw) | End-to-end source build, customization, and SD-card flashing |
| [Haven Guide](https://buildwithparallel.com/products/haven) | Field notes, hardware setup, range tests, and deployment notes |
| [Haven mesh setup scripts](https://github.com/buildwithparallel/haven-manet-ip-mesh-radio) | Gate/point node setup scripts for the wider Haven mesh topology |
| [Build with Parallel](https://buildwithparallel.com) | Project site and broader Haven updates |
| [Data Slayer on YouTube](https://www.youtube.com/@DataSlayerMedia) | Build videos, demos, and field updates |
| [Data Slayer on X](https://x.com/data_slayer) | Short-form updates and project notes |

## Supported Hardware

### HaLow radios

| Device                          | Chip   | Interface         | Status                     |
|---------------------------------|--------|-------------------|----------------------------|
| Seeed Studio HaLow HAT          | MM6108 | SPI               | Does not bind yet          |
| Gateworks MM8108                | MM8108 | USB (HAT-to-Pi)   | Working                    |

The original goal is the Seeed Studio HaLow HAT on Pi 5 over SPI — that path is still open. The DesignWare SPI / RP1 DMA / Morse SPI overlay are in place but the MM6108 chip isn't binding yet on Pi 5. As a working alternative, the MM8108 currently runs on Pi 5 via a USB cable from the HaLow board to the Pi.

### Regular Wi-Fi

| Device                          | Chip          | Interface | Status         |
|---------------------------------|---------------|-----------|----------------|
| Pi 5 onboard Wi-Fi              | BCM43455      | SDIO      | Included       |
| Panda Wireless USB dongle       | Ralink RT5370 | USB       | Included       |

Drivers and firmware for both are in the build (`kmod-brcmfmac` + `cypress-firmware-43455-sdio` for onboard, `kmod-rt2800-usb` for the dongle). Configure them at runtime via LuCI or `/etc/config/wireless`.

## Included Add-Ons

Beyond the base Morse HaLow stack, this build ships a curated set of extras ready to use out of the box:

- **Mesh networking** — `batman-adv` + `batctl` Layer-2 mesh with LuCI integration, plus `mesh11sd` for 802.11s
- **VPN** — WireGuard (kernel module + LuCI proto) and Tailscale client/daemon (compatible with Headscale for off-grid use)
- **ADS-B aircraft tracking** — RTL-SDR library, `dump1090` decoder, and Python 3 with `cryptography` for `adsbcot` (Cursor-on-Target feed)
- **LoRa / Reticulum sidecar** — Python 3 with `pyserial` and `netifaces`; USB-ACM and USB-serial drivers for:
  - Heltec V1 (nRF52) / V4 (ESP32-S3)
  - RAK4631 (nRF52840)
  - Seeed Xiao ESP32-S3
  - Walter (ESP32-S3)
  - Muzi Works Base Duo
  - Null Hop Mesh Toad (CH341)
- **Network debugging** — `tcpdump`, `iperf3`, `mtr`, `nmap`, full `iw`, `lsusb`
- **Web UI** — LuCI with Morse Argon theme, first-boot setup wizard, WireGuard and batman-adv config UIs
- **General utilities** — Bash, `curl` (TLS + CA bundle), `nano`, `htop`, `screen`, mDNS (`umdns`)

## Building

Tested on Ubuntu 22.04 / 24.04.

For a visual walkthrough of the full process, including building the OpenWrt Morse Micro firmware from source, applying customization, and flashing the resulting image to an SD card, see the [build and flash video](https://youtu.be/2y5DjqLjSXw).

### 1. Install build dependencies

```
sudo apt update
sudo apt install build-essential clang flex g++ gawk gcc-multilib g++-multilib git gettext \
  libncurses5-dev libssl-dev python3-setuptools rsync unzip zlib1g-dev swig file wget \
  libnl-3-dev libnl-genl-3-dev pkg-config
```

### 2. Clone

```
git clone -b rpi5-mm-23.05 https://github.com/buildwithparallel/openwrt-morse-rpi5.git
sudo chown -R $USER:$USER openwrt-morse-rpi5
cd openwrt-morse-rpi5
```

The `chown` ensures every file in the cloned tree is owned by your user. OpenWrt's build system refuses to run as root and trips over root-owned files, so this avoids permission errors later.

### 3. Update and install feeds

```
./scripts/feeds update -a
./scripts/feeds install -a
```

This pulls the Morse HaLow, OpenWrt, and LuCI package feeds (defined in `feeds.conf.default`) and registers their packages with the build system.

### 4. Drop in the Pi 5 config

```
cp boards/ekh-bcm2712/target_diffconfig .config
make defconfig
```

`target_diffconfig` selects the rpi-5 device, the Morse HaLow driver/firmware/CLI, the S1G hostapd/wpa_supplicant stack, mesh, and LuCI apps. `make defconfig` expands it into a full `.config`, filling in defaults.

### 5. Download sources

```
make -j$(nproc) download
```

### 6. Build

```
make -j$(nproc) V=sc 2>&1 | tee log.txt
```

`V=sc` enables verbose compile output; `tee log.txt` captures it for troubleshooting.

When the build finishes, the image is at:

```
bin/targets/bcm27xx/bcm2712/openwrt-bcm27xx-bcm2712-rpi-5-squashfs-factory.img.gz
```

Flash to an SD card with [Raspberry Pi Imager](https://www.raspberrypi.com/software/) (choose "Use custom" and select the `.img.gz`) and boot the Pi 5.

## Setting the Region

On first boot, the Morse Micro LuCI setup wizard prompts you to select your country/region. This sets the regulatory domain for both HaLow (S1G) and standard Wi-Fi, controlling available channels, TX power limits, and bandwidth options. S1G channel plans vary significantly by region (US, AU, and EU each use completely different frequency allocations), so getting this right matters.

To change the region after initial setup:

**LuCI web UI:**

1. Go to **Network → Wireless**
2. For **each radio** (HaLow, onboard Wi-Fi, USB dongle, etc.), click **Edit**
3. Confirm the **Country Code** is set correctly (e.g. **United States**)
4. Click **Save** on that radio
5. Back on the Wireless overview page, click **Save & Apply**

That last step matters — editing each radio and saving individually is not enough; you must **Save & Apply** on the main Wireless page for the regulatory domain to take effect.

**CLI:**
```sh
uci set wireless.radio0.country='US'
uci commit wireless
wifi reload
```

## Upstream sources

- Morse Micro OpenWrt SDK: https://github.com/MorseMicro/openwrt (23.05.5 base, branch `mm/v23.05.5`)
- Morse Micro package feed: https://github.com/MorseMicro/morse-feed
- Morse Micro driver: https://github.com/MorseMicro/morse_driver
- Upstream OpenWrt 24.10 (Pi 5 hardware donor): https://github.com/openwrt/openwrt
