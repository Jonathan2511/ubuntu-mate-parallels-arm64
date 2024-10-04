# Install Ubuntu MATE arm64 Raspberry Pi Image to Parallels on Apple Silicon

This guide will walk you through the process of installing the Ubuntu MATE ARM64 (aarch64) Raspberry Pi image to Parallels on Apple Silicon. 
<img width="1680" alt="image" src="https://github.com/user-attachments/assets/1f2f692b-0871-4ba2-b629-9c427b7c7e16">

## Steps
### 1. Obtain Live ISO of Fedora Workstation for ARM64
1. Open [Fedora Workstation](https://fedoraproject.org/workstation/download) on your browser.
2. Scroll down to find the ARM64 editions, then click the **Download Now**.
3. Download the Live ISO for ARM® aarch64 systems.

### 2. Create a Virtual Machine for Fedora
1. Start Parallels and launch the **Create New** wizard.
2. Select **Install Windows, Linux, or macOS from an image file**.
   <img width="984" alt="Screenshot 2024-10-02 at 23 25 50" src="https://github.com/user-attachments/assets/879195ec-0b72-4658-8af6-c216912e1f04">
4. Choose the Fedora Workstation ISO image file downloaded in the previous step.
5. Change the VM name as desired. Optionally, customize the settings (e.g., set guest OS type to **Other Linux**).

### 3. Boot Fedora Workstation Live
1. Start the VM and boot into the live environment.
2. Click **Not Now** when prompted by the Fedora installer.
   <img width="1680" alt="image" src="https://github.com/user-attachments/assets/9f2fa19f-1ff4-42dc-93a5-adcdb4f5299d">

   
### 4. Obtain the Ubuntu MATE ARM64 Image
1. In the Fedora VM, click the **Activities** button at the top-left corner of the screen, then launch **Firefox**.
2. Open the address bar and navigate to [Ubuntu MATE's download page](https://ubuntu-mate.org/download/arm64/).
3. Right-click the **Direct Download** button and choose **Copy Link** to obtain the download URL.
   <img width="1680" alt="image" src="https://github.com/user-attachments/assets/610e2ba0-59d9-4b9d-84d8-9093a4b25fe6">


### 5. Download the Image and Write It to the VM’s Hard Disk
1. Open **Terminal** from the Activities menu.
2. Run the following command to download and write the image directly to the virtual hard disk:
    ```bash
    wget -O - <PASTE_LINK_HERE> | xzcat | sudo dd of=/dev/sda bs=4M
    ```
    <img width="1680" alt="image" src="https://github.com/user-attachments/assets/b5a77e5b-5629-48be-ab08-f41f76d7f320">
3. Once complete, open **gnome-disks**, and extend the `dev/sda2` partition (Linux filesystem/root).
   <img width="1680" alt="image" src="https://github.com/user-attachments/assets/da361124-90c0-45b8-a411-857f71f79799">

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
    <img width="1680" alt="image" src="https://github.com/user-attachments/assets/856fe733-0757-40a6-8c2e-de0aab486a10">

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
    <img width="1680" alt="image" src="https://github.com/user-attachments/assets/1700b743-730a-4c96-8bf6-4a231f8f5a66">

### 8. Bind the Mounted Filesystems
1. Bind the necessary directories for the chroot environment:
    ```bash
    sudo mount --bind /dev /mnt/dev
    sudo mount --bind /dev/pts /mnt/dev/pts
    sudo mount --bind /proc /mnt/proc
    sudo mount --rbind /sys /mnt/sys
    sudo mount --bind /run /mnt/run
    ```
   <img width="1680" alt="image" src="https://github.com/user-attachments/assets/ec8d27f1-3bd1-4091-84b1-9f70c7080391">

### 9. Chroot into the Mounted Partition
1. Enter the chroot environment:
    ```bash
    sudo chroot /mnt
    ```
   <img width="1680" alt="image" src="https://github.com/user-attachments/assets/a22989de-3dca-418f-936a-96319e52a405">

### 10. Install Grub and Linux Kernel
1. Update package lists:
    ```bash
    apt update
    ```
    <img width="1680" alt="image" src="https://github.com/user-attachments/assets/1d3ba5ed-43ae-4936-bc0f-9b1ebec01705">

2. Install GRUB and Linux Image packages:
    ```bash
    apt install grub-common grub-efi linux-image-generic linux-headers-generic
    ```
    <img width="1680" alt="image" src="https://github.com/user-attachments/assets/271aa455-54e7-4f07-8d93-ce6202ac8713">

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
   <img width="1680" alt="image" src="https://github.com/user-attachments/assets/eff6425c-ebe0-44c2-b469-6836cc08ba5e">
3. Open `/etc/fstab`:
    ```bash
    nano /etc/fstab
    ```
4. Replace the line for `/dev/sda1` with the following:
    ```bash
    UUID=XXXX-XXXX /boot/efi vfat umask=0077 0 1
    ```
    Replace `XXXX-XXXX` with the UUID you copied earlier.
    <img width="1680" alt="image" src="https://github.com/user-attachments/assets/f6238f8e-a36b-4090-a7b3-21c371ca350d">
5. Check if the `dev/sda1` is not mounted correctly:
   ```bash
   mount /dev/sda1
   ```
   <img width="1680" alt="image" src="https://github.com/user-attachments/assets/17ba7c25-50c9-4acd-b46c-468704d45ce9">

### 12. Install and Configure GRUB
1. Install GRUB to the EFI directory:
    ```bash
    grub-install --efi-directory=/boot/efi
    ```
    <img width="1680" alt="image" src="https://github.com/user-attachments/assets/4ffd9493-fa74-4e41-aaee-bbb1ddcd32f3">
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
    <img width="1680" alt="image" src="https://github.com/user-attachments/assets/66381dcb-1ef9-4db8-96e6-e1e65c984cc4">
5. Update GRUB configuration:
    ```bash
    update-grub
    ```
    <img width="1680" alt="image" src="https://github.com/user-attachments/assets/365abf28-0a30-4d59-99c7-dc6279a5a6ee">
    
### 13. Exit Chroot and Reboot
1. Exit the chroot environment:
    ```bash
    exit
    ```
    <img width="1680" alt="image" src="https://github.com/user-attachments/assets/c8a83005-2d02-4e2e-a661-102d73a5453d">
2. Run a filesystem check on the `/dev/sda1` partition and follow the steps:
    ```bash
    sudo fsck /dev/sda1
    ```
    <img width="1680" alt="image" src="https://github.com/user-attachments/assets/94c2a0e7-0355-43ff-bb13-246dcf13e4e5">
3. Reboot the system:
    ```bash
    reboot
    ```

### 14. Boot into the Non-Raspberry Pi Kernel
1. On grub bootloader menu, select **Advanced Options for Ubuntu**
   <img width="1680" alt="image" src="https://github.com/user-attachments/assets/93243ae2-21a1-4925-8e36-2cebd69a3476">
2. Choose the **non-Raspberry Pi kernel** from the GRUB menu.
   <img width="1680" alt="image" src="https://github.com/user-attachments/assets/12e976dc-88af-45c1-a675-a3a6fe7382ed">
3. Follow the installation.
   <img width="1680" alt="image" src="https://github.com/user-attachments/assets/284c8d8e-4c4e-468a-b989-03da7b454e04">
4. You have successfully installed Ubuntu MATE ARM64 on Parallels.
   <img width="1680" alt="image" src="https://github.com/user-attachments/assets/6858e0b2-fe55-429d-9660-aaffa6b8b8b9">
5. Update the system and install Parallels Tools
6. Optional, you can now upgrade the system to 24.04
   <img width="1680" alt="image" src="https://github.com/user-attachments/assets/88f2c76c-d6ff-4cce-8250-bef8acff5c6d">

## Done!
Your Ubuntu MATE arm64 installation should now be running on Parallels with Apple Silicon.
<img width="1680" alt="image" src="https://github.com/user-attachments/assets/74eba064-c3b9-45fa-96f7-ddbd404ef34f">

## Thanks and Credit
Special thanks to [Yuizumi](https://dev.to/yuizumi) for their helpful guide on [Installing Manjaro on Parallels with Apple Silicon](https://dev.to/yuizumi/installing-manjaro-on-parallels-with-apple-silicon-e6k). This resource was invaluable in setting up my environment. 

