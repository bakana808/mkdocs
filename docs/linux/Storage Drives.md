# Storage Drives
## Listing Drives
Listing all mounted or unmounted drives on a system (device path, UUID, drive name, filesystem, etc.):
```
$ sudo blkid
```
## Mounting Drives
Mounting a drive. The directory that a drive is being mounted to needs to exist before attempting to mount.
```bash
# create the directory if it doesn't exist
$ sudo mkdir <dest>
$ sudo chown <user>:<group> <dest>

# mount the drive
$ sudo mount <device> <dest>
```
## Retrieving a User or Group ID
Users and groups are associated with an ID, which is needed to set the owner of an automounted drive.

Retrieving user ID, group ID, and all groups for a user:
```bash
$ id <username>
```
Retrieving a user ID (two methods):
```bash
# using id (without username, defaults to current user)
$ id -u <username>

# using shell variable
$ echo $UID
```
Retrieving a group ID:
```bash
$ id -g <username>
```
