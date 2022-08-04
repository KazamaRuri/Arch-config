## Arch Linux Simple Installation guide

### 1. Prepare Installation

- ##### Network Connect

  ```shell
  >> iwctl
  [iwd] >> station wlan0 connect <Wi-Fi Name>
  ```

- ##### Time Configure

  ```shell
  >> timedatectl set-ntp true
  ```

- ##### Partition the disks (UEFI)

  ```shell
  >> gdisk -l /dev/*your_device* # Check your partitions
  >> gdisk /dev/*your_device*
  ```

  > Now use `n` to create a partition
  >
  > Use `w` to confirm the changes

  | Mount point | Partition                     | Partition type        | Suggested size          | gdisk's code |
  | ----------- | ----------------------------- | --------------------- | ----------------------- | ------------ |
  | `/mnt/boot` | `/dev/*efi_system_partition*` | EFI system partition  | At least 300 MiB        | `ef00`       |
  | `[SWAP]`    | `/dev/*swap_partition*`       | Linux swap            | More than 512 MiB       | `8200`       |
  | `/mnt`      | `/dev/*root_partition*`       | Linux x86-64 root (/) | Remainder of the device | `8300`       |

- ##### Format the partitions

  ```shell
  >> mkfs.fat -F 32 -n EFI /dev/*efi_system_partition*
  >> mkswap -L SWAP /dev/*swap_partition*
  >> mkfs.btrfs -L /dev/*root_partition*
  ```
  
- ##### Mount the file systems

  - Create btrfs subvolumes

      ```shell
      >> mount /dev/*root_partition* /mnt
      >> btrfs sub create /mnt/@
      >> btrfs sub create /mnt/@home
      >> umount /mnt
      ```
      
  - Mount the subvolumes
  
      ```shell
      >> mount -o subvol=@ /dev/*root_partition* /mnt
      >> mkdir -p /mnt/home
      >> mount -o subvol=@home /dev/*root_partition* /mnt/home
      ```
      
  - Mount the EFI partitons
  
      ```shell
      >> mkdir -p /mnt/boot
      >> mount /dev/*efi_system_partition* /mnt/boot
      ```
  
  - Mount the SWAP partitons
  
    ```shell
    >> swapon /dev/*swap_partition*
    ```
    
  

### 2. Installation

- ##### Select the mirrors

  ```shell
  >> vim /etc/pacman.d/mirrorlist
  ```

- ##### Install essential packages
  
  ```shell
  >> pacstrap /mnt base base-devel linux linux-firmware linux-headers btrfs-progs networkmanager
  >> pacstrap /mnt intel-ucode # or amd-ucode
  ```

### 3. Configure the System

- ##### Fstab

  ```shell
  >> genfstab -U /mnt >> /mnt/etc/fstab
  ```

- ##### Chroot

  ```shell
  >> arch-chroot /mnt
  ```

- ##### Time zone

  ```shell
  >> ln -sf /usr/share/zoneinfo/*Region*/*City* /etc/localtime
  >> hwclock --systohc
  ```

- ##### Localization

  > First edit `/etc/locale.gen` and uncomment `en_US.UTF-8 UTF-8` and other needed locales

  ```shell
  >> echo "LANG=en_US.UTF-8" >> /etc/locale.conf
  ```

- ##### Host

  ```shell
  >> echo "*your_host_name*" >> /etc/hostname
  >> ehco "127.0.0.1	localhost" >> /etc/hosts
  >> ehco "::1	localhost" >> /etc/hosts
  >> ehco "127.0.1.1	*your_host_name*.localdomain *your_host_name*" >> /etc/hosts
  ```

- ##### Initramfs

  ```shell
  >> mkinitcpio -P
  ```

- ##### Root password

  ```shell
  >> passwd
  ```

- ##### Boot loader

  - Install grub

    ```shell
    grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
    ```

  - Config systemd-boot

    ```shell
    grub-mkconfig -o /boot/grub/grub.cfg
    ```

- ##### User create

  ```shell
  >> pacman -S zsh # Use zsh instand of bash
  >> useradd -m -G wheel -s /usr/bin/zsh *username*
  >> passwd *username*
  ```

- ##### Desktop environment

  ```shell
  >> pacman -S xorg
  >> pacman -S xf86-video-intel # If there's Intel integrated Graphics driver, or xf86-video-ati for AMD
  >> pacman -S sddm plasma-meta konsole dolphin dolphin-pulgins ark # Use KDE
  ```

- ##### Config systemd

  ```shell
  >> systemctl enable sddm
  >> systemctl enable NetworkManager
  ```

- ##### (Maybe) Fuck you Nvidia 

  ```shell
  >> pacman -S nvidia nvidia-utils nvidia-settings
  ```

- ##### Final Steps

  ```shell
  >> exit
  >> umount -R /mnt
  >> systemctl poweroff
  ```

