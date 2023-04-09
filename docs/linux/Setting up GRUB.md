# Setting up a GRUB partition

1. Create an EFI partition at 512 MB

2. Set the type of the partition to "EFI System"

3. Create a FAT32 filesystem on it (assuming `/dev/sda1` is your EFI partition)
    
    ```bash
    mkfs.fat -F32 /dev/sda1
    ```
    
4. Ensure that the required packages are installed

    ```bash
    pacman -S grub efibootmgr
    # if GRUB should detect other operating systems
    pacman -S os-prober
    ```
    
5. Create a folder to mount the partition and mount it

    ```bash
    mkdir /efi
    mount /dev/sda1 /efi
    ```
    
6. Install GRUB
    
    ```bash
    grub-install --target=x86_64-efi --bootloader-id=GRUB --efi-directory=/efi
    ```
    
7. Configure GRUB

    ```bash
    grub-mkconfig -o /boot/grub/grub.cfg
    ```