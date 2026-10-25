# Pre-Packaged Environment

## CLI Utilities

* yq: command-line YAML, JSON, XML, CSV and properties processor — https://github.com/mikefarah/yq
* pandoc: command-line document conversion format

## Application management

GNOME Software is the application manager. Bazaar is not needed. Retain Warehouse for Flatpak management and Flatseal for permissions management.

### Flathub Per User

Flathub is added to each user's own Flatpak installation at their first login, so apps from Flathub install per user without an administrator password.  Flathub is not configured system-wide (OpenSUSE's flatpak-remote-flathub is left out of the image): with Flathub in both installations, flatpak asks on every install which one to use.  The service runs once for each user, so a user who removes Flathub does not get it back

##### tcbl-flathub.service (user service)

    flatpak remote-add --user --if-not-exists flathub /usr/share/tc-benchtop-settings/flathub.flatpakrepo

GNOME Software installs Flatpak files opened from outside it (such as the .flatpakref from the Install button on flathub.org) into the user's installation:

    gsettings set org.gnome.software install-bundles-system-wide false

### Homebrew Per User

Homebrew is installed for each user, in their own home, the first time they run brew: /usr/bin/brew runs Homebrew's own installer for them, without sudo, and hands the command over to it.  Nothing is downloaded for users who never run brew.  Homebrew's prebuilt packages (bottles) work in a prefix no longer than its default, /home/linuxbrew/.linuxbrew (26 bytes), so the prefix is ~/.linuxbrew, or ~/.brew when the home path is too long for that (user names of 10 to 14 characters); a longer home path gets an error instead of an installation that builds everything from source.  Homebrew calls this support for other prefixes new and possibly not yet fully stable

##### /usr/etc/profile.d/tcbl-homebrew.sh

	# Technicomp Benchtop Linux: the user's own Homebrew (installed by /usr/bin/brew
	# on first use) on the PATH of each login session, with its MANPATH and INFOPATH
	if [ -z "${HOMEBREW_PREFIX-}" ] && [ "$(id -u)" -ne 0 ]; then
		for _tcbl_brew in "$HOME/.linuxbrew" "$HOME/.brew" /home/linuxbrew/.linuxbrew; do
			if [ -x "$_tcbl_brew/bin/brew" ]; then
				eval "$("$_tcbl_brew/bin/brew" shellenv)"
				break
			fi
		done
		unset _tcbl_brew
	fi

Note: the profile.d script runs at login, so a user who installs Homebrew during a session has its commands on the PATH from the next login; brew itself works at once, and it prints the line that adds Homebrew to the current terminal.  OpenSUSE's bash.bashrc does not read profile.d for each new terminal, and the user's dotfiles are left alone.  The compilers Homebrew needs (gcc, gcc-c++) are in the base pattern.  Tested October 25 in a test environment (Ubuntu 24.04, glibc 2.39) as an unprivileged user: the install needed no sudo, and hello was poured from its bottle into ~/.linuxbrew

#TODO — Test on an installed Benchtop system

### AppImage backend and AppImageHub

GNOME Software needs an AppImage backend with AppImageHub integration so that users can discover and install AppImages through the same interface as other applications. This is a development requirement, not a feature already implemented in the Benchtop image.

Each AppImage should be installed per user in `~/Applications/`. Installation should generate a `.desktop` entry in `$XDG_DATA_HOME/applications/` (normally `~/.local/share/applications/`) so the application appears in the desktop launcher. The entry should use the application’s name and icon and point to the installed executable through an absolute path, with desktop-entry quoting rules applied. The installer should make the AppImage executable.

Updates should keep the launcher pointed at the installed AppImage. Uninstalling should remove the managed AppImage and its generated launcher. The backend and catalog integration still need implementation and testing.

References: [AppImageHub](https://www.appimagehub.com/) and the [Desktop Entry Specification](https://specifications.freedesktop.org/desktop-entry/latest-single/).

## Additional applications

Use Flatpak and AppImage for GUI applications, with GNOME Software as the application manager, and Homebrew for additional CLI tools. Nix is no longer under consideration.

## Backup

* restic — A modern backup program for your files

## Tailscale and desktop integration

Tailscale status, exit-node selection, and connection controls should be accessible from the GNOME desktop. `tailscaled` manages the tunnel; making NetworkManager ignore that interface is separate from exposing useful desktop controls.

The earlier notes proposed this ignore rule:

### /etc/NetworkManager/conf.d/90-tailscale.conf

    [keyfile]
    unmanaged-devices=interface-name:tailscale*

The [Tailscale quick-settings extension](https://github.com/maxgallup/tailscale-gnome-qs) is a candidate for the UI. The earlier claim that `--netfilter-mode=nodivert` enables NetworkManager integration was incorrect: it changes how Tailscale installs firewall rules. It does not create a NetworkManager connection profile. See [Tailscale’s netfilter documentation](https://tailscale.com/docs/reference/netfilter-modes).

## rclone
Mount nearly any remote storage service onto your local machine; great for multi-machine setups.
