# Drivers and Firmware

The lists below describe the intended hardware and format coverage. Package commands are working references and still need to resolve against the image’s selected repositories.

## Goal
Complete driver packaging and firmware built-in:

* Printers, Scanners, GPU, WiFi, Function Keys
* Proprietary drivers built-in
* Proprietary firmware built-in
* Optimus/Dynamic GPU support

## AMDGPU

##### /etc/modprobe.d/amdgpu.conf

	# Force using of the amdgpu driver for Southern Islands (GCN 1.0+) and Sea
	# Islands (GCN 2.x) generations.
	options amdgpu si_support=1 cik_support=1
	options radeon si_support=0 cik_support=0

Note: amdgpu is already the default for these GPUs since kernel 6.19.  The 6.18 LTS kernel still defaults to radeon
[Link](https://github.com/CachyOS/CachyOS-Settings/blob/master/usr/lib/modprobe.d/amdgpu.conf)

## Video Codecs

Ensure hardware-accelerated decoding (VA-API / VDPAU) and all common software codecs are pre-installed.

### Hardware Acceleration

    # Intel (recent):
    sudo zypper install intel-media-driver     # iHD driver (Broadwell+)
    # Intel (legacy):
    sudo zypper install libva-intel-driver     # i965 driver (Sandy Bridge–Coffee Lake)
    # AMD (built into Mesa):
    # No additional packages needed; Mesa's radeonsi/RADV handle VA-API natively
    # NVIDIA:
    # NVDEC/NVENC provided by the proprietary driver

### Software Codecs (from Packman repo)

    sudo zypper install ffmpeg gstreamer-plugins-bad gstreamer-plugins-ugly gstreamer-plugins-libav
    sudo zypper install libavcodec-full        # All ffmpeg codecs including non-free (h264, h265, AAC)
    sudo zypper install x264 x265 libvpx libdav1d

Key codec coverage:

* **H.264/AVC** — Universal web video, Blu-ray (x264 encoder, openh264 decoder)
* **H.265/HEVC** — 4K streaming, modern cameras (x265 encoder)
* **VP9** — YouTube, WebRTC (libvpx)
* **AV1** — Open video codec; YouTube, Netflix (dav1d decoder, SVT-AV1 or rav1e encoder)
* **AAC** — Standard audio codec (fdkaac for high quality, or ffmpeg's built-in)
* **FLAC, Vorbis, Opus** — Open audio codecs (included in base gstreamer)
* **AC3/DTS** — Surround sound for movies (a52dec, libdca)

### Verification

    vainfo                    # Check VA-API support
    vdpauinfo                 # Check VDPAU support (NVIDIA)
    gst-inspect-1.0 | grep -i "264\|265\|av1\|vp9"   # Check GStreamer codec availability

## Image Formats

    sudo zypper install libjpeg-turbo libpng libtiff libwebp libheif libavif libraw libJXL

Key format coverage:

* **JPEG/JPEG XL** — Photo formats (libjpeg-turbo, libJXL)
* **PNG** — Lossless screenshots, graphics (libpng)
* **WebP** — Web images (libwebp)
* **HEIF/HEIC** — Apple photos format (libheif — requires HEVC decoder)
* **AVIF** — AV1-based image format (libavif)
* **TIFF** — Professional/scientific imaging (libtiff)
* **RAW** — Camera raw formats (libraw — covers CR2, NEF, ARW, DNG, etc.)
* **SVG** — Vector graphics (librsvg, built into GTK)
* **EXR** — HDR imaging (OpenEXR — needed by Blender, GIMP)

Verify Nautilus/Gwenview can generate thumbnails for all formats after installation.

## Hardware-Specific Tools

* [OpenRGB](https://openrgb.org/) — RGB lighting control
* [OpenRazer](https://openrazer.github.io/) — Razer peripheral support
* [linux-surface](https://github.com/linux-surface) — Microsoft Surface support
* [asus-linux](https://asus-linux.org/) — ASUS laptop support
* [MrChromebox](https://docs.mrchromebox.tech/) — Chromebook firmware
* [Asahi Linux](https://asahilinux.org/) — Apple Silicon support
* [t2linux](https://t2linux.org/) — Apple T2 Mac support

## Core Components

* OpenRGB-udev-rules
* rasdaemon

## OpenRGB SMBus Access

OpenRGB needs the /dev/i2c-* device nodes for RAM and motherboard lighting.  Load i2c-dev at boot, and give the device nodes to administrators (wheel) instead of the openrgb group, which has no members unless someone is added to it

##### /etc/modules-load.d/i2c-dev.conf

	i2c-dev

##### /etc/udev/rules.d/90-i2c-wheel.rules

	ACTION!="remove", SUBSYSTEM=="i2c-dev", RUN+="/usr/bin/setfacl -m g:wheel:rw $devnode"

Note: the rule has to sort after OpenSUSE's 60-openrgb.rules, whose `RUN=` assignment discards the RUN entries of earlier rules

## NTSYNC

Load the NTSYNC driver at boot, used with compatible wine/proton builds.  The module does not load on its own:

##### /etc/modules-load.d/ntsync.conf

	ntsync

Note: the driver creates /dev/ntsync readable and writable by everyone, so no udev rule is needed.  Flatpak apps need device access to use it; Steam's Flatpak has it (--device=all)

See [NTSYNC Kernel Docs](https://docs.kernel.org/userspace-api/ntsync.html)
See [CachyOS Settings](https://github.com/CachyOS/CachyOS-Settings/blob/master/usr/lib/modules-load.d/ntsync.conf)

## USB Device Emulation (raw-gadget)

raw-gadget lets userspace emulate USB devices — one you are developing, or a custom-class device — for host-side testing and software proxying.  The `raw_gadget` module is already in the image, along with the configfs and legacy gadget modules (`g_hid`, `g_ether`, `g_mass_storage`, `gadgetfs`, `libcomposite`).  Emulating a device without a USB Device Controller needs `dummy_hcd`, a software controller the stock kernel does not build; the custom kernel adds it (see [Build Configuration](../Build%20Configuration.md#custom-kernel)).

Give `/dev/raw-gadget` to administrators (wheel), as with the i2c nodes.  The modules do not load on their own.

##### /etc/modules-load.d/raw-gadget.conf

	raw_gadget
	dummy_hcd

##### /etc/udev/rules.d/90-raw-gadget-wheel.rules

	ACTION!="remove", KERNEL=="raw-gadget", RUN+="/usr/bin/setfacl -m g:wheel:rw $devnode"

Note: dummy_hcd does not exist until the custom kernel builds it, so its modules-load line does nothing until then.  Any user of /dev/raw-gadget can present an arbitrary USB device to this machine, so the node is restricted to administrators rather than left open

## Firmware Flashing (flashrom)

flashrom is in the base set, for programming BIOS/UEFI/coreboot and SPI flash.  External USB programmers (CH341A, FT2232/FT232H, Dediprog, serprog boards) work through libusb and need only device permissions.  The internal programmer (the board's own flash) needs kernel changes that belong to the custom kernel — lockdown off, and the spi-intel driver kept off the flash controller (see [Build Configuration](../Build%20Configuration.md#custom-kernel)).

Give the supported USB programmers to administrators (wheel).  FT2232-based programmers are bound by ftdi_sio as a serial port; detaching them lets flashrom's libftdi take them.  serprog boards appear as /dev/ttyUSB* or /dev/ttyACM* and use the dialout group.

##### /etc/udev/rules.d/90-flashrom-wheel.rules

	# Illustrative ids: CH341A (WCH 1a86:5512), FT2232H (0403:6010), Dediprog
	# SF100 (0483:dada). Verify against the programmers actually used.
	SUBSYSTEM=="usb", ATTR{idVendor}=="1a86", ATTR{idProduct}=="5512", RUN+="/usr/bin/setfacl -m g:wheel:rw $devnode"
	SUBSYSTEM=="usb", ATTR{idVendor}=="0403", ATTR{idProduct}=="6010", RUN+="/usr/bin/setfacl -m g:wheel:rw $devnode"
	SUBSYSTEM=="usb", ATTR{idVendor}=="0483", ATTR{idProduct}=="dada", RUN+="/usr/bin/setfacl -m g:wheel:rw $devnode"

##### /etc/modprobe.d/spi-intel.conf (custom kernel)

	# Keep the kernel from binding the PCH SPI flash controller, so flashrom can
	# reach it. Gives up the kernel's MTD access and write-protection enforcement
	# for the onboard flash.
	blacklist spi_intel_pci

Note: the internal programmer stays blocked by kernel lockdown until the custom kernel drops the Secure Boot lockdown trigger (Build Configuration), so the blacklist only matters from that kernel on.  `/dev/mem` and `/dev/port` are already built in (CONFIG_DEVMEM, CONFIG_DEVPORT)

## Monitoring Services

### Log Monitoring
Add users to systemd-journal group on creation

### Core Dumps

##### /etc/tmpfiles.d/30-coredump.conf

	# Override coredump cleanup age from 2w (systemd default) to 3d
	e /var/lib/systemd/coredump - - - 3d
[Link](https://github.com/CachyOS/CachyOS-Settings/blob/master/usr/lib/tmpfiles.d/coredump.conf)

### Kernel Crash Logs (pstore)

##### /etc/modprobe.d/pstore.conf

	# Keep the kernel log of a crash across the reboot
	options efi_pstore pstore_disable=0

OpenSUSE's kernel builds the UEFI backend of pstore switched off.  At an oops or panic the kernel writes the end of its log into UEFI variables, and at the next boot systemd-pstore moves it to /var/lib/systemd/pstore.  To test on a spare machine: `echo c | sudo tee /proc/sysrq-trigger`, then check /var/lib/systemd/pstore after the reboot

### Hardware Monitoring

    sudo systemctl enable --now smartd
    sudo systemctl enable --now rasdaemon

### Performance Monitoring Tools

* htop
* iotop-c
* nvtop
* iftop
* nethogs
* ss
* powertop
* atop
