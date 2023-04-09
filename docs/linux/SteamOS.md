## Accessing BIOS
Access BIOS -> hold VOLUME UP + POWER until chime is heard
Access boot menu -> hold VOLUME DOWN + POWER until chime is heardjj
# Addons / Tweaks
## EmuDeck
[EmuDeck](https://www.emudeck.com/)
Installer (.desktop file): [link](https://www.emudeck.com/EmuDeck.desktop)
## CryoUtilities
[CryoUtilities](https://github.com/CryoByte33/steam-deck-utilities)
Installer (.desktop file): [link](https://raw.githubusercontent.com/CryoByte33/steam-deck-utilities/main/InstallCryoUtilities.desktop)
# Setup for Development
```bash
# set sudo password
$ passwd

# disable readonly mode
$ sudo steamos-readonly disable

# set up pacman-key
$ sudo pacman-key --init
$ sudo pacman-key --populate archlinux

# install base-devel (provides required development tools)
$ sudo pacman -S base-devel

# reinstall packages which may have missing files
$ sudo pacman -S glibc linux-api-headers libarchive

# install yay
$ cd /tmp && git clone 'https://aur.archlinux.org/yay.git' && cd /tmp/yay && makepkg -si && cd ~ && rm -rf /tmp/yay/

# install additional packages for python
$ sudo pacman -S python-pip
```