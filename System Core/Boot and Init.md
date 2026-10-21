# Boot and Init

## Boot Chain

* OpenBMC
* CoreBoot signed BIOS w/ no ME
* UEFI Secure Boot w/ HW root of Trust
* Automatic Disk Unlock or Passphrase
* Plymouth
    * GRUB Proper Resolution
    * Flicker-free Boot
    * Hide Kernel Messages on Sleep/Wake
    * Suppress All Messages

## Quiet Boot with Plymouth

##### /etc/kernel/cmdline

    quiet loglevel=2 vt.global_cursor_default=0 splash
    sdbootutil update-all-entries

##### /etc/systemd/system.conf.d/10-show-status.conf

    # Show service status messages only when a step fails or boot takes more than 25 seconds
    [Manager]
    ShowStatus=auto

Note: with `auto`, a hang shows up as "A start job is running for ..." once boot has run for 25 seconds; press Esc on the Plymouth splash to see it.  `quiet` on the kernel command line would otherwise select `error`, which shows failures but not a step that hangs, and `no` hides both

## Shutdown Timeout

    mkdir -p /etc/systemd/system.conf.d /etc/systemd/user.conf.d /etc/systemd/system/user@.service.d

##### /etc/systemd/system.conf.d/10-shutdown.conf

    # Change default stop time for processes from 90 seconds to 15 seconds
    [Manager]
    DefaultTimeoutStopSec=15s

##### /etc/systemd/user.conf.d/10-shutdown.conf

    # Change default stop time for user session processes from 90 seconds to 15 seconds
    [Manager]
    DefaultTimeoutStopSec=15s

##### /etc/systemd/system/user@.service.d/10-shutdown.conf

    # Change stop time for the user session from 120 seconds to 15 seconds
    [Service]
    TimeoutStopSec=15s

## Auto-Update with Notify

    echo 'REBOOT_METHOD=notify' | sudo tee /etc/transactional-update.conf
    sudo systemctl enable --now transactional-update.timer
    systemctl --user enable --now transactional-update-notifier.service
    sudo systemctl enable health-checker.service
    sudo systemctl enable create-dirs-from-rpmdb.service

	sudo install -d -m 0755 /etc/rebootmgr/rebootmgr.conf.d
	printf '[rebootmgr]\nstrategy=off\n' | sudo tee /etc/rebootmgr/rebootmgr.conf.d/50-strategy.conf
	sudo systemctl restart rebootmgr
    sudo systemctl disable --now rebootmgr.service

health-checker rolls back if the first boot after an update fails: TCBL's patched version (home:technicomp:benchtop) makes the newest snapshot that passed the checks the default for the next boot, without rebooting or starting an emergency shell.  create-dirs-from-rpmdb creates the /var directories of packages installed by transactional-update

### x86-64-v3 Libraries

After an update, install the x86-64-v3 optimized builds of the system libraries (glibc HWCAPS) on CPUs that support them.  tcbl-x86-64-v3.service runs this while the update waits for a restart:

    transactional-update --continue --non-interactive pkg in --force --recommends patterns-glibc-hwcaps-x86_64_v3
[Link](https://build.opensuse.org/package/show/openSUSE:Factory/x86_64_v3-branding-Aeon)

## DBUS Broker

	systemctl enable dbus-broker.service
    sudo systemctl --global enable dbus-broker.service
Note: I believe this is now OpenSUSE default

## systemd-bsod

systemd-bsod (introduced in systemd 255) displays a full-screen error message on the framebuffer when the system fails to boot. It captures the emergency-level log messages and renders them in a readable format, similar to Windows' blue screen. This replaces the old behavior of dropping to a tiny-font emergency shell that most users cannot read.

To enable:

    sudo systemctl enable systemd-bsod.service

Note: Requires the framebuffer to be available at early boot (i.e., working Plymouth/KMS). On OpenSUSE MicroOS/Aeon, this may already be enabled.

## Graphical Logins Only

No text logins on the virtual consoles.  The first console belongs to the login screen

    sudo systemctl disable getty@tty1.service

##### /etc/systemd/logind.conf.d/10-no-text-login.conf

    [Login]
    NAutoVTs=0
    ReserveVT=0

For a rescue shell when the desktop doesn't start: hold Space while the computer starts to show the boot menu, press e, and add `systemd.unit=rescue.target SYSTEMD_SULOGIN_FORCE=1` to the kernel command line.  root has no password, and SYSTEMD_SULOGIN_FORCE=1 lets the rescue shell start without one.  The TPM only unlocks the disk for the unchanged command line, so the disk asks for its recovery key

## Console renderer idea (kmscon)

Consider replacing the kernel's fbcon with [kmscon](https://github.com/kmscon/kmscon), a userspace KMS/DRM console that renders a scalable font (the TTF monospace stack rather than fbcon's bitmap font), with Unicode, scrollback and a mouse.  The maintained revival is at version 10, packaged in openSUSE:Factory with the freetype renderer and libseat, so it needs no new packaging.  [Fedora write-up](https://fedoramagazine.org/modernizing-the-console-with-kmscon/)

This records an idea, not a decision.  kmscon renders the virtual terminals, and Graphical Logins Only turns those off (NAutoVTs=0, getty disabled, tty1 is the login screen).  kmscon is a late systemd service, so it does not render the early-boot kernel messages or the rescue sulogin shell, which stay on fbcon.  Using it would mean reopening text consoles as kmscon VTs, which reverses the graphical-only choice for those terminals.

Open questions before any switch: whether to stay graphical-only (then kmscon has little to render) or deliberately bring back Ctrl+Alt+F terminals as kmscon consoles; the SELinux gap the openSUSE maintainer flagged (kmscon's issue display is patched out pending policy, so kmscon under the targeted policy needs checking for denials); and the DRM-master handoff between a kmscon VT and GDM on real hardware.  If the real want is only a legible rescue console on a HiDPI panel, a larger console font may do instead.

## Linux Resume Quirks

* Reset Bluetooth
* Reset WiFi
* Reset Trackpad
* Reset Touchscreen

## Flicker-free Boot

A flicker-free boot requires the entire chain from firmware to desktop to maintain the same display mode without any mode-setting transitions:

1. Firmware (UEFI GOP) sets the native resolution
2. GRUB/systemd-boot must not change the video mode — use `GRUB_GFXMODE=auto` or systemd-boot with `console-mode auto`
3. Kernel must use early KMS (Kernel Mode Setting) — the GPU driver must be built into the initramfs, not loaded as a module later
4. Plymouth must use the DRM (direct rendering) backend, not the fbdev fallback

##### /etc/kernel/cmdline (already set in Quiet Boot section)

    quiet loglevel=2 vt.global_cursor_default=0 splash

`vt.global_cursor_default=0` hides the console cursor, which otherwise blinks on screen until Plymouth starts

##### /etc/dracut.conf.d/10-early-kms.conf

    # Force GPU driver into initramfs for early KMS
    # For Intel:
    force_drivers+=" i915 "
    # For AMD:
    force_drivers+=" amdgpu "
    # For NVIDIA (open kernel module):
    force_drivers+=" nvidia nvidia_modeset nvidia_uvm nvidia_drm "

Rebuild initramfs after changes:

    sudo dracut --force

## Hide Kernel Messages on Sleep/Wake

Kernel messages during suspend/resume break the visual experience. To suppress them:

##### /etc/kernel/cmdline

    quiet loglevel=0

`loglevel=0` suppresses normal console printing, including KERN_EMERG at level 0: a message is normally printed only when its numeric priority is lower than the console threshold. See the [kernel printk documentation](https://cdn.kernel.org/doc/html/latest/core-api/printk-basics.html). Use `loglevel=2` during development and `loglevel=0` for the final distro image. Alternatively, keep `loglevel=2` and rely on Plymouth to mask the console output visually during suspend/resume transitions.
