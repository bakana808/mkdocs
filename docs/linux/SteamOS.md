## Accessing BIOS
Access BIOS -> hold VOLUME UP + POWER until chime is heard
Access boot menu -> hold VOLUME DOWN + POWER until chime is heard

# Addons / Tweaks
## EmuDeck
[website](https://www.emudeck.com/) [installer](https://www.emudeck.com/EmuDeck.desktop) (.desktop file)
## CryoUtilities
[website](https://github.com/CryoByte33/steam-deck-utilities) [installer](https://raw.githubusercontent.com/CryoByte33/steam-deck-utilities/main/InstallCryoUtilities.desktop) (.desktop file)
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