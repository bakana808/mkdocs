# Gaming
## Issues
### NTFS Drives
Steam games installed on NTFS drives (drives formatted by Windows) may not start when playing on Linux. These drives need to be mounted with the filesystem type `ntfs-3g`. 

Example entry in `/etc/fstab`:
```bash
# <file system> <dir> <type> <options> <dump> <pass>
/dev/nvme0n1p3 /run/media/octopod/cube ntfs-3g umask=0077,gid=1000,uid=1000 0 1
```
See: [[Storage Drives#Listing Drives]]

### Async Shader Compiling
DXVK (the DirectX to Vulkan API conversion driver) can cause games to stutter as the shaders need to be compiled. This can be done asynchronously using this DXVK async patch: https://github.com/Sporif/dxvk-async

Instructions:
1. Download the pre-patched DXVK binaries from [here](https://github.com/Sporif/dxvk-async/releases/latest).
2. Run the included script. This will replace the binaries in the current WINE installation, backing up the old versions.
```bash
$ setup_dxvk.sh --install
```
3. Set the environment variable `DXVK_ASYNC` to `1`.
```shell
$ export DXVK_ASYNC=1
```

Games may have graphical errors as they won't draw things if the shader for them hasn't been compiled yet.