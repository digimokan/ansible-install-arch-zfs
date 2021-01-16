# ansible-install-arch-zfs

Ansible script that installs zfs-based Arch Linux from archiso.

[![Release](https://img.shields.io/github/release/digimokan/ansible-install-arch-zfs.svg?label=release)](https://github.com/digimokan/ansible-install-arch-zfs/releases/latest "Latest Release Notes")
[![License](https://img.shields.io/badge/license-MIT-blue.svg?label=license)](LICENSE.txt "Project License")

## Table Of Contents

* [Motivation](#motivation)
* [Installation Script Actions](#installation-script-actions)
* [Installation Script Customization](#installation-script-customization)
* [Post-Installation](#post-installation)
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

## Installation Script Actions

* Installs to single hard drive, or two mirrored drives.
* Auto-detects motherboard type
  ([BIOS](https://wiki.archlinux.org/index.php/Arch_boot_process#BIOS) or
   [UEFI](https://wiki.archlinux.org/index.php/Arch_boot_process#UEFI)).
* Sets up [GRUB](https://wiki.archlinux.org/index.php/GRUB) bootloader.
* Installs and enables [NetworkManager](https://wiki.archlinux.org/index.php/NetworkManager).
* Configures booting of the following
  [archzfs kernels](https://github.com/archzfs/archzfs/wiki):
    * The `zfs-linux` stable kernel (via `archzfs-linux` package): default.
    * The `zfs-linux-lts` LTS kernel (via `archzfs-linux-lts` package):
      selectable in _GRUB_ boot menu.

## Installation Script Customization

* Define required/optional vars using one of the methods in
  [Full Usage / Options](#full-usage--options).
* Required vars:
    * `user_var_install_devices` (an _ansible_ list, e.g. - "sda")
* Optional vars (have default values, and may be overridden):
    * `user_var_zpool_name` (e.g. "zpool_alpha")
    * `arch_install_def_time_zone_file` (e.g. "Canada/Central")
    * `arch_install_def_locale` (e.g. "en_US.UTF-8")
    * `arch_install_def_keymap` (e.g. "us")
    * `arch_install_def_hostname` (e.g. "omegarig")
* Optional vars (for optional actions, when defined):
    * `user_var_bootstrap_git_repo` (repo to clone and place in `/root`)
    * `user_var_extra_packages` (extra packages to install to new machine)

## Post-Installation

* The machine is now bootable and will connect via wired network interface.
* _NOTE: all systemd zfs integration tasks listed in the
  [Arch Linux ZFS Install Guide](https://wiki.archlinux.org/index.php/Install_Arch_Linux_on_ZFS#After_the_first_boot)
  are completed automatically by the script._
* The machine is now ready for normal user-level configuration, at the
  [Arch Linux Post-Installation](https://wiki.archlinux.org/index.php/Installation_guide#Post-installation)
  step.

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
   $ pacman -Sy
   $ pacman -S git ansible
   ```

4. Clone project into a local directory:

   ```shell
   $ git clone https://github.com/digimokan/ansible-install-arch-zfs.git
   ```

5. Change to the local directory:

   ```shell
   $ cd ansible-install-arch-zfs
   ```

6. Run the ansible script for a single-disk install:

   ```shell
   $ ansible-playbook -i hosts -e '{"user_var_install_devices":["sda"]}' playbook.yml
   ```

7. Configure the new machine, per the [Arch Linux Post-Installation](https://wiki.archlinux.org/index.php/Installation_guide#Post-installation)
   instructions in the _Arch Linux Installation Guide_.

## Full Usage / Options

### Method 1: Set Required User Vars From Command Line

* For single-disk install option:

   ```shell
   $ ansible-playbook -i hosts -e '{"user_var_install_devices":["sda"]}' playbook.yml
   ```

* For two-disk mirrored install option:

   ```shell
   $ ansible-playbook -i hosts -e '{"user_var_install_devices":["sda","sdb"]}' playbook.yml
   ```

* For single-disk install option, customizing a var (e.g. `zpool` name):

   ```shell
   $ ansible-playbook -i hosts -e '{"user_var_install_devices":["sda"],"user_var_zpool_name":"zpool_alpha"}' playbook.yml
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
   $ ansible-playbook -i hosts -e '@/some/file/path/my_user_vars.yml' playbook.yml
   ```

### Method 3: Set Required User Vars Internally, Within Source Tree

1. Edit the required user vars in project `group_vars/all` file:

   ```
   # group_vars/all
   user_var_install_devices:
     - "sda"
     - "sdb"
   user_var_zpool_name: "zpool_alpha"
   ```

2. Run the playbook:

   ```shell
   $ ansible-playbook -i hosts playbook.yml
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
│ │ ├── grub-install/     # install GRUB bootloader to boot partition(s)
│ │ ├── play-fact-set/    # initial calc/set of play-wide vars
│ │ ├── pool-creation/    # creates zpool on the install disk(s)
│ │ ├── post-install/     # clean up and finalize the installation
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

