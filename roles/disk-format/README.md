# disk-format

Role to partition the installation disk(s).

* Assumes use of _GRUB_ bootloader.
* Assumes use of _sgdisk_ disk-formatting program.

## Partitioning Scheme

### BIOS Motherboard

1. Create a boot partition.

    * Set "BIOS Boot Partition" code (_gdisk_ code `EF02`), per
      [Arch Linux GRUB Page](https://wiki.archlinux.org/index.php/GRUB#GUID_Partition_Table_(GPT)_specific_instructions),
      and [_sgdisk_ Man Page](https://www.freebsd.org/cgi/man.cgi?query=sgdisk).
    * Set partition size to 2 MB, and place in first part of disk, per
      [Arch Linux ZFS Install](https://wiki.archlinux.org/index.php/Install_Arch_Linux_on_ZFS#Partition_the_destination_drive).
      Other resources recommend 1 MB or 2MB sizes.
    * __Do not mount the partition, or place in fstab.__ Per
      [Arch Linux GRUB Page](https://wiki.archlinux.org/index.php/GRUB#GUID_Partition_Table_(GPT)_specific_instructions)
      and [Arch Linux Partitioning Page](https://wiki.archlinux.org/index.php/Partitioning#Example_layouts),
      the partition has nothing to do with the `/boot` directory. _GRUB_ will
      provision the partition accordingly.

2. Create a zfs partition.

    * Set "Solaris Root" code (_gdisk_ code `BF00`), per
      [Arch Linux ZFS Install](https://wiki.archlinux.org/index.php/Install_Arch_Linux_on_ZFS#Partition_the_destination_drive).
    * Use the remainder of the disk for this partition.

### UEFI Motherboard

1. Create a boot partition.

    * Set "BIOS Boot Partition" code (_gdisk_ code `EF00`), per
      [Arch Linux EFI System Partition Page](https://wiki.archlinux.org/index.php/EFI_system_partition#GPT_partitioned_disks)
      and [_sgdisk_ Man Page](https://www.freebsd.org/cgi/man.cgi?query=sgdisk).
    * Set partition size to 550 MB, and place in first part of disk, per
      [_sgdisk_ Man Page](https://www.freebsd.org/cgi/man.cgi?query=sgdisk).
      Other resources recommend 100 MB or 260MB sizes.
    * Mount the partition in `/efi` directory, per
      [Arch Linux Partitioning Page](https://wiki.archlinux.org/index.php/Partitioning#Example_layouts).
      _GRUB_ will provision the partition accordingly.

2. Create a zfs partition.

    * Set "Solaris Root" code (_gdisk_ code `BF00`), per
      [Arch Linux ZFS Install](https://wiki.archlinux.org/index.php/Install_Arch_Linux_on_ZFS#Partition_the_destination_drive).
    * Use the remainder of the disk for this partition.

