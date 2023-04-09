## Arch Linux Packages

### Installing Packages
```bash
# Installing a package
$ sudo pacman -S <package>

# Removing a package
$ sudo pacman -R <package>

# Listing dependencies and other information about a package (query & print info):
$ sudo pacman -Qi <package>

# Synchronize repositories:
$ sudo pacman -Sy

# Full system upgrade (sync repositories, update the keyring, then updates all packages):
$ sudo pacman -Sy archlinux-keyring & pacman -Su

# Remove orphaned packages (packages no longer required by any other package):
$ sudo pacman -Rns $(pacman -Qtdq)
```

### Installing AUR (Community) Packages

Manually:

``` bash
$ git clone https://aur.archlinux.org/<package>.git
$ cd <package>
$ mkpkg -si
```

Using a AUR helper (`yay`):
```bash
# install yay manually
$ git clone https://aur.archlinux.org/yay.git
$ cd yay
$ mkpkg -si

# or use this one-liner
$ cd /tmp && git clone 'https://aur.archlinux.org/yay.git' && cd /tmp/yay && makepkg -si && cd ~ && rm -rf /tmp/yay/

# installing a package
$ yay -S <package>
```

#### `Yay` Install/Update One-liner
```bash
cd /tmp && git clone 'https://aur.archlinux.org/yay.git' && cd /tmp/yay && makepkg -si && cd ~ && rm -rf /tmp/yay/
```

## Sudo

`sudo`

Must add user to `sudo` group and edit `etc/sudoers` to allow users in the `sudo` group.

## Text Editors

### Neovim
`pacman -S neovim python-neovim`

## Clipboard Support
`xclip` for X11
`wl-clipboard` for Wayland

## Programming Languages

### Python
`pacman -S python3`

## Bluetooth Support
`bluez bluez-utils`

```
# sudo systemctl enable bluetooth.service
# sudo systemctl start bluetooth.service
```

If using KDE, it comes with its own Bluetooth frontend.

## Desktop Environments (DE)
Desktop environments are usually an all-in-one package of window manager (WM), compositor, and/or a full program suite including a settings GUI and default apps.

### Plasma

`plasma`


X11

`xorg-xinit xorg-server`

## Display Manager (DM)
Display managers (or login managers) automatically starts a GUI at the end of the boot process.

### No DM
`pacman -S xorg-xinit xorg-server`
Without a display manager, the GUI would need to be entered through the terminal. This can be done by running `startx` . This will source a file named `~/.xinitrc`, which should contain the commands needed to start your DE or WM, as well as any default apps.

---

To open Plasma when running `startx` (provided by `xorg-xinit`), add these lines to a file named `~/.xinitrc`.

```
~/.xinitrc
```
```
export DESKTOP_SESSION=plasma
exec startplasma-x11
```

### LightDM
`pacman -S lightdm`
A simple, lightweight display manager. Usually requires a greeter, which provides a login screen.
## File Explorers
### Thunar
`pacman -S thunar`

#### Additional Packages
`tumbler`: thumbnail support
`ffmpegthumbnailer`: video thumbnail support
`thunar-volman`: removable device management
`gvfs`: trash support, mounting with udisk and remote filesystems