# Install Ubuntu MATE arm64 Raspberry Pi Image to Parallels on Apple Silicon

This guide will walk you through the process of installing the Ubuntu MATE ARM64 (aarch64) Raspberry Pi image to Parallels on Apple Silicon. 
<img width="1680" alt="Screenshot 2024-10-02 at 01 14 04" src="https://github.com/user-attachments/assets/99329a41-e718-4577-9de1-c569c1a524a9">

## Steps

### 1. Obtain Live ISO of Fedora Workstation for ARM64
1. Open [Fedora Workstation](https://getfedora.org/) on your browser.
2. Scroll down to find the ARM64 editions, then click the **Download Now**.
3. Download the Live ISO for ARM® aarch64 systems.

### 2. Create a Virtual Machine for Fedora
1. Start Parallels and launch the **Create New** wizard.
2. Select **Install Windows, Linux, or macOS from an image file**.
3. Choose the Fedora Workstation ISO image file downloaded in the previous step.
4. Change the VM name as desired. Optionally, customize the settings (e.g., set guest OS type to **Other Linux**).

### 3. Boot Fedora Workstation Live
1. Start the VM and boot into the live environment.
2. Click **Not Now** when prompted by the Fedora installer.
   
### 4. Obtain the Ubuntu MATE ARM64 Image
1. In the Fedora VM, click the **Activities** button at the top-left corner of the screen, then launch **Firefox**.
2. Open the address bar and navigate to [Ubuntu MATE's download page](https://ubuntu-mate.org/download/arm64/).
3. Choose ARM64 for the architecture and click **Download**.
4. Right-click the **Image** button and choose **Copy Link** to obtain the download URL.

### 5. Download the Image and Write It to the VM’s Hard Disk
1. Open **Terminal** from the Activities menu.
2. Run the following command to download and write the image directly to the virtual hard disk:
    ```bash
    wget -O - <PASTE_LINK_HERE> | xzcat | sudo dd of=/dev/sda bs=4M
    ```
3. Once complete, open **gnome-disk-utility**, and extend the `dev/sda2` partition (Linux filesystem/root).

### 6. Set Up the Bootable Partition with Parted
1. Format the dev/sda1 partition to FAT32
     ```bash
    sudo mkfs.vfat -F32 /dev/sda1
    ```
2. Open **parted** and make the FAT32 partition bootable with the following commands:
    ```bash
    sudo parted /dev/sda
    print
    set 1 boot on
    set 1 esp on
    quit
    ```

### 7. Mount the Partitions
1. Mount the root filesystem:
    ```bash
    sudo mount /dev/sda2 /mnt
    ```
2. Create the `/boot/efi` directory on the root partition:
    ```bash
    sudo mkdir -p /mnt/boot/efi
    ```
3. Mount the FAT32 partition to the `/boot/efi` folder:
    ```bash
    sudo mount /dev/sda1 /mnt/boot/efi
    ```

### 8. Bind the Mounted Filesystems
1. Bind the necessary directories for the chroot environment:
    ```bash
    sudo mount --bind /dev /mnt/dev
    sudo mount --bind /dev/pts /mnt/dev/pts
    sudo mount --bind /proc /mnt/proc
    sudo mount --rbind /sys /mnt/sys
    sudo mount --bind /run /mnt/run
    ```

### 9. Chroot into the Mounted Partition
1. Enter the chroot environment:
    ```bash
    sudo chroot /mnt
    ```

### 10. Install Grub and Linux Kernel
1. Update package lists:
    ```bash
    apt update
    ```
2. Install GRUB and Linux Image packages:
    ```bash
    apt install grub-common grub-efi linux-image-generic linux-headers-generic
    ```
3. Update the initramfs:
    ```bash
    update-initramfs -u -k all
    ```
    
### 11. Update fstab for EFI Partition
1. Get the UUID of `/dev/sda1`:
    ```bash
    blkid 
    ```
2. Copy the UUID of `/dev/sda1`.
3. Open `/etc/fstab`:
    ```bash
    nano /etc/fstab
    ```
4. Replace the line for `/dev/sda1` with the following:
    ```bash
    UUID=XXXX-XXXX /boot/efi vfat umask=0077 0 1
    ```
    Replace `XXXX-XXXX` with the UUID you copied earlier.

### 12. Install and Configure GRUB
1. Install GRUB to the EFI directory:
    ```bash
    grub-install --efi-directory=/boot/efi
    ```
2. Update the initramfs again:
    ```bash
    update-initramfs -u -k all
    ```
3. Open `/etc/default/grub`:
    ```bash
    nano /etc/default/grub
    ```
4. Set the GRUB menu to show with a timeout:
    ```bash
    GRUB_TIMEOUT_STYLE=menu
    GRUB_TIMEOUT=n
    ```
5. Update GRUB configuration:
    ```bash
    update-grub
    ```
    
### 13. Exit Chroot and Reboot
1. Exit the chroot environment:
    ```bash
    exit
    ```
2. Run a filesystem check on the `/dev/sda1` partition:
    ```bash
    fsck /dev/sda1
    ```
3. Reboot the system:
    ```bash
    reboot
    ```

### 14. Boot into the Non-Raspberry Pi Kernel
1. After reboot, choose the **non-Raspberry Pi kernel** from the GRUB menu.
2. You have successfully installed Ubuntu MATE ARM64 on Parallels.
3. Update the system and install Parallels Tools
4. Optional, you can now upgrade the system to 24.04

## Done!
Your Ubuntu MATE arm64 installation should now be running on Parallels with Apple Silicon.
