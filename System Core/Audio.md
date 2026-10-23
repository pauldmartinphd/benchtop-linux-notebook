# Audio Subsystem

## Pipewire

PipeWire is the default audio/video server replacing both PulseAudio and JACK. It handles Bluetooth audio codecs, screen sharing (for Wayland), and pro-audio routing.

### JACK (through PipeWire)

PipeWire is the JACK server; no standalone `jackd` is installed, and adding one would conflict with it. `pipewire-jack` (in the base pattern) and `pipewire-libjack-0_3` ship PipeWire's JACK libraries — `libjack.so.0`, `libjackserver.so.0` and `libjacknet.so.0` under `/usr/lib64/pipewire-0.3/jack/`.

It is wired transparently: `pipewire-libjack-0_3` installs `/etc/ld.so.conf.d/pipewire-jack-x86_64.conf`, which puts those libraries on the dynamic-linker path, so any application that links `libjack.so.0` uses PipeWire's implementation automatically, with no per-application setup. `pw-jack` is also installed for launching a program against PipeWire's JACK explicitly. A Flatpak pro-audio application instead uses its runtime's own PipeWire-JACK layer over the shared PipeWire socket.

Low-latency scheduling uses rtkit with the rtprio and memlock limits below (Realtime Audio Configuration), so a JACK client needing SCHED_FIFO must run as a user in that limits group. As built: Build31.1, PipeWire 1.6.9.

### Bluetooth codecs

The goal is to include the codec support needed by the supported headsets. The notes list SBC/SBC-XQ, AAC, aptX variants, LDAC, and LC3/LC3plus for evaluation. Availability depends on the PipeWire build, session-manager configuration, Bluetooth transport, and both devices; installing one package does not establish support for every codec.

The proposed openSUSE package is `pipewire-nonfree-codecs`. Confirm its contents against the image build rather than treating “non-free codecs” as a complete capability list.

    sudo zypper install pipewire-nonfree-codecs

For inspection, retain the existing command as a starting point:

    pw-cli info all | grep -A 10 bluez

Check the active device and stream properties, not just the installed packages. PipeWire documents the supported `bluez5.codecs` names in its [property reference](https://docs.pipewire.org/page_man_pipewire-props_7.html). The earlier `spa-acp-tool list-codecs` recipe has not been established as a Bluetooth codec check.

[PipeWire Guide](https://github.com/mikeroyal/PipeWire-Guide)

## Realtime Audio Configuration

https://wiki.linuxaudio.org/wiki/system_configuration#audio_group

### Group Limits
User must be member of a group with sufficient rtprio and memlock set (e.g., audio or realtime):

    sudo usermod -a -G <group_name> <user_name>

### RT Priorities
The original audio check could not acquire SCHED_FIFO priority 80. The selected group limits are recorded in [Memory Management](../Performance/Memory%20Management.md); this observation is the reason for configuring them:
See https://wiki.linuxaudio.org/wiki/system_configuration#limitsconfaudioconf

### Power Management for Audio
The original check found that the user could not access `/dev/cpu_dma_latency`. Review the device permissions needed by applications such as Ardour and Reaper; this was an access problem in the tested setup, not a general inability to control latency from userspace.
See https://wiki.linuxaudio.org/wiki/system_configuration#quality_of_service_interface

### Swappiness for Audio
Keep the selected `vm.swappiness=180` for zram-backed swap. The older check recommended 10 for avoiding disk-backed swap latency, but the September 16 correction explicitly retained 180 and required latency-critical audio memory to be locked. See [Memory Management](../Performance/Memory%20Management.md).
See https://wiki.linuxaudio.org/wiki/system_configuration#sysctlconf

## Audio Enhancement (EasyEffects)

EasyEffects is a PipeWire effects host for input and output streams. Its processing options include limiting, automatic gain, compression, equalization, bass enhancement, crossfeed, reverb, delay, and convolution using impulse responses.

Evaluate it for laptop-speaker correction, with per-model presets in `tc-benchtop-settings` and a choice of package or Flatpak delivery. JackHack96’s “Advanced Auto Gain” preset is a candidate from the references. The reported improvements are reasons to test it on supported laptops, not evidence that one preset will work well on every model.

The spatial-audio link below was saved from its title and has not been reviewed. An impulse-response convolver should not be described as a complete Dolby Atmos replacement on that basis.

Reference links:

* EasyEffects should be part of every distro (laptop speaker quality): https://www.osnews.com/story/145883/easyeffects-should-be-part-of-every-linux-distribution-and-desktop-environment-to-massively-improve-laptop-speaker-sound-quality/
* PSA: EasyEffects can drastically improve audio: https://www.reddit.com/r/linux/comments/1laetsl/psa_easyeffects_can_drastically_improve_audio/
* Dolby Atmos alternative for Linux (spatial audio; convolver/IR-based approaches): https://www.reddit.com/r/linux_gaming/comments/1w2f441/dolby_atmos_alternative_for_linux/ — link filed from title; thread content not yet reviewed (Reddit blocks automated fetch)

## Audio Cues (UX Sound Design)

Design audio cues for:

* Any delayed response/action
* Drag and drop/file copy
* File download
* Empty trash
* Action not allowed (e.g., click outside box when input required)

Reference: https://utcc.utoronto.ca/~cks/space/blog/linux/SystemSoundsShouldBeGranular
