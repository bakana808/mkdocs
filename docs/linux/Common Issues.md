```toc
```
## Boot / Startup

#### Changing GRUB kernal boot parameters

```cmd
sudo gedit /etc/default/grub
sudo upgrade-grub
```

## Terminal setup

#### Fix TTY Screen Resolution

Install `fbset`.

`fbset -G <width> <height> <x> <y> 32`

#### Temporarily Change TTY Font

`setfont /usr/share/kbd/consolefonts/drdos8x16.psfu.gz`

## Graphics setup

##### Disabling Nouveau (default NVIDIA driver)

This is necessary when installing the official NVIDIA graphic drivers.

Add the parameters `nomodeset` `nouveau.modeset=0` to the GRUB kernal boot parameters.

### NVIDIA module unable to start after updating Linux kernel

The NVIDIA module not starting can lead to being stuck in the terminal, the display manager not starting, and `startx` not finding any screens.

This may be caused by the NVIDIA module being in the modules folder for the old Linux version. Try to start the module manually:
```
$ modprobe nvidia
```
This will fail if it cannot find the module in the correct module folder.

You can try installing `nvidia-dkms` (replaces `nvidia`), which will compile the modules for you in the correct folder:
```
$ sudo pacman -S nvidia-dkms
```

## Package management
#### Deleting orphaned packages

```
sudo pacman -Rns $(pacman -Qtdq)
```

`pacman -Qt` queries all packages that are not required by any other package.
The `d` flag filters it to only packages installed as dependencies.
The `q` flag outputs simpler text (to send to `pacman -Rns`).

#### `invalid or corrupted package` or other PGP issues

Try each of these steps in order to see if they fix the error:

1. If system time is not synced, manually sync it:
```cmd
$ ntpd -qg
$ hwclock -w
```
2. Update the `archlinux-keyring` package:
```cmd
$ sudo pacman -Sy archlinux-keyring
```
3. Refresh keys:
```cmd
$ sudo pacman-key --refresh-keys
```
4. Resetting all installed keys:
```cmd
$ sudo rm -rf /etc/pacman.d/gnupg/
$ sudo pacman-key --init
$ sudo pacman-key --populate
```
5. Manually add the key:
```cmd
$ sudo pacman-key --recv-keys <keyid>
$ sudo pacman-key --finger <keyid>
$ sudo pacman-key --lsign-key <keyid>
```
## Storage partition setup

#### Finding the UUID of a partition

```
sudo blkid /dev/<part>
```

#### Mounting NTFS partitions

Type: `ntfs-3g`
Options: `defaults,exec,nls=utf8,umask=000,uid=1000,guid=1000`

This will mount the NTFS partition with RWX permissions.

May need to turn off hibernation and fast boot on Windows before doing this.

#### Auto-mounting drives

Goes into the `/etc/fstab` file.

Use the UUID instead of the `/dev/<part>` name since partition IDs may change.

## Xorg setup

#### Overriding the .xinitrc file that is used when running `startx`

```
startx ./<config>
```

The `./` before the path is important. `startx` won't recognize it otherwise.

#### Starting an Xserver under a different display ID

```
startx -- :<id>
```

With a custom config:

```
startx ./<config> -- :<id>
```

#### Finding the IDs of all connected monitors

```
xrandr --query
```

#### Playing Windows games

On Steam: Steam will attempt to play it with its built-in Proton dist.

Not on Steam: Use `lutris`.

[[Ricing]]

## Music production

#### Native DAW options

- Bitwig Studio
- Reaper

#### Mounting & installing Native Instrument .isos

Installing products through Native Access (in Wine) will fail if it downloads an
.iso and has to mount it (the mount program doesn't work in Wine). In this case,
mount the .iso manually and install it.

```cmd
sudo mount -t udf -o unhide <iso> <mount point>
```

Afterwards, execute the installer exe within to install it.

#### Bridging .dll VST plugins

Options
- linvst
- airwave
- carla (and carla-bridges-win)

`linvst` seems to produce the best results.

```cmd
linvstconvert-cli -i <vst dir>
```

Guitar Rig 5 does not work currently. Thankfully Bitwig Studio has its own amp plugin.

#### Adobe Creative Cloud