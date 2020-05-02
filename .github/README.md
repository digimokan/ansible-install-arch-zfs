# ansible-install-arch-zfs

Ansible script that installs zfs-based Arch Linux from archiso.

[![Release](https://img.shields.io/github/release/digimokan/ansible-install-arch-zfs.svg?label=release)](https://github.com/digimokan/ansible-install-arch-zfs/releases/latest "Latest Release Notes")
[![License](https://img.shields.io/badge/license-MIT-blue.svg?label=license)](LICENSE.txt "Project License")

## Table Of Contents

* [Motivation](#motivation)
* [Features](#features)
* [Requirements](#requirements)
* [Quick Start](#quick-start)
* [Full Usage / Options](#full-usage--options)
* [Source Code Layout](#source-code-layout)
* [Contributing](#contributing)

## Motivation

Install a minimal, bootable, zfs-based Arch Linux to a new machine from a live
_archiso_ USB stick.

Base the _ansible_ script on the official Arch Linux
[Arch Linux ZFS Install Guide](https://wiki.archlinux.org/index.php/Install_Arch_Linux_on_ZFS)
(also consult guidance in the [Arch Linux Installation Guide](https://wiki.archlinux.org/index.php/Installation_guide)
and the [Arch Linux ZFS](https://wiki.archlinux.org/index.php/ZFS) pages)

## Features

* Auto-detects and adjusts for BIOS-motherboard or UEFI-motherboard on new
  machine.
* Installs _GRUB_ bootloader.
* Installs Arch Linux to single hard drive, or two mirrored drives.
* On Completion, the new machine will boot a minimal Arch Linux OS running the
  stable zfs kernel (`zfs-linux`). The new machine is ready for configuration,
  at the [Post-Installation](https://wiki.archlinux.org/index.php/Installation_guide#Post-installation)
  step.
* Customize the installation for your use case with these vars, using one of the
  methods in [Full Usage / Options](#full-usage--options):
    * `arch_install_def_time_zone_file` (e.g. "Canada/Central")
    * `arch_install_def_locale` (e.g. "en_US.UTF-8")
    * `arch_install_def_keymap` (e.g. "us")
    * `arch_install_def_hostname` (e.g. "omegarig")
    * `user_var_zpool_name` (e.g. "zpool_alpha")

## Requirements

* A new machine with one (or two, for mirrored setup) empty (or formatable) hard
  drive(s).
* A USB stick with a bootable _archiso_ image, and these _archiso_ additions:
    1. A zfs kernel package (e.g. `archzfs-linux`), loadable with
       `modprobe zfs`.
    2. The `git` and `ansible` packages.

## Quick Start

1. Insert _archiso_ USB stick into new machine.

2. Boot up the new machine from the _archiso_ USB stick. A shell prompt for the
   root user should be active.

3. As required: use `pacman` to install `git` and `ansible` packages:

   ```shell
   # pacman -Sy
   # pacman -S git ansible
   ```

4. Clone project into a local directory:

   ```shell
   # git clone https://github.com/digimokan/ansible-install-arch-zfs.git
   ```

5. Change to the local directory:

   ```shell
   # cd ansible-install-arch-zfs
   ```

6. Run the ansible script for a single-disk install:

   ```shell
   # ansible-playbook -i hosts -e '{"user_var_install_devices":["sda"]}' playbook.yml
   ```

7. Configure the new machine, per the [Post-Installation](https://wiki.archlinux.org/index.php/Installation_guide#Post-installation)
   instructions in the _Arch Linux Installation Guide_.

## Full Usage / Options

### Method 1: Set Required User Vars From Command Line

* For Single-Disk Install Option:

   ```shell
   # ansible-playbook -i hosts -e '{"user_var_install_devices":["sda"]}' playbook.yml
   ```

* For Two-Disk Mirrored Install Option:

   ```shell
   # ansible-playbook -i hosts -e '{"user_var_install_devices":["sda","sdb"]}' playbook.yml
   ```

* For Single-Disk Install Option, Specifying The New `zpool` Name:

   ```shell
   # ansible-playbook -i hosts -e '{"user_var_install_devices":["sda"],"user_var_zpool_name":"zpool_alpha"}' playbook.yml
   ```

### Method 2: Set Required User Vars In External Vars File

1. Define the required user vars in an external file:

   ```
   # /some/file/path/my_user_vars.yml
   user_var_install_devices:
     - "sda"
     - "sdb"
   user_var_zpool_name: "zpool_alpha"
   ```

2. Run the playbook, passing the external vars file:

   ```shell
   # ansible-playbook -i hosts -e '@/some/file/path/my_user_vars.yml' playbook.yml
   ```

### Method 3: Set Required User Vars Internally, Within Source Tree

1. Define the required user vars in project `group_vars/all` file:

   ```
   # group_vars/all
   user_var_install_devices:
     - "sda"
     - "sdb"
   user_var_zpool_name: "zpool_alpha"
   ```

2. Run the playbook:

   ```shell
   # ansible-playbook -i hosts playbook.yml
   ```

## Source Code Layout

```
├─┬ ansible-install-arch-zfs/
│ │
│ ├── group_vars/         # required user-vars, and constant playbook vars
│ │
│ ├─┬ roles/
│ │ ├── arch-install/     # install Arch to disk(s), and configure
│ │ ├── archiso-config/   # performs initial config of the live USB OS
│ │ ├── dataset-creation/ # creates reasonable set of system/user datasets
│ │ ├── disk-format/      # formats install disk(s) with boot/zfs partitions
│ │ ├── play-fact-set/    # initial calc/set of play-wide vars
│ │ ├── pool-creation/    # creates zpool on the install disk(s)
│ │ ├── user-vars-chk/    # checks that required user_var_* vars are defined
│ │ └── zroot-config/     # configures zpool for use after adding datasets
│ │
│ ├── ansible.cfg         # play-wide ansible meta-config
│ ├── hosts               # ansible inventory (configured for local host)
│ └── playbook.yml        # main ansible playbook
│
```

## Contributing

* Feel free to report a bug or propose a feature by opening a new
  [Issue](https://github.com/digimokan/ansible-install-arch-zfs/issues).
* Follow the project's [Contributing](CONTRIBUTING.md) guidelines.
* Respect the project's [Code Of Conduct](CODE_OF_CONDUCT.md).

