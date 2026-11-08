# NixOS Automated Installer

Complete automated installation script for NixOS with interactive configuration wizard.

## Features

- ✅ **One-Command Installation**: Single script handles everything
- ✅ **Interactive Disk Selection**: Choose from detected disks
- ✅ **Automatic Partitioning**: GPT with EFI boot and ext4 root
- ✅ **Configuration Wizard**: Guided setup for all system options
- ✅ **Hardware Detection**: Auto-detects CPU, GPU, memory, network
- ✅ **Modular Architecture**: Clean, organized NixOS configuration
- ✅ **Zero Manual Steps**: Fully automated from start to finish

## Quick Start

### Method 1: One-Line Install (Recommended)

Boot NixOS Live USB and run:

```bash
curl -L https://raw.githubusercontent.com/julas23/nix-file/main/install.sh | sudo bash
```

### Method 2: Download and Run

```bash
# Download the script
curl -O https://raw.githubusercontent.com/julas23/nix-file/main/install.sh

# Make it executable
chmod +x install.sh

# Run as root
sudo ./install.sh
```

## Installation Flow

### 1. Pre-Installation Checks
- Verifies root privileges
- Checks internet connectivity
- Displays welcome banner

### 2. Disk Selection
```
Available Disks:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
No.  Device          Size       Model
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1    /dev/sda        500G       Samsung SSD 860
2    /dev/nvme0n1    1T         Samsung 970 EVO Plus
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Select disk number for installation (1-2): 2

Selected Disk:
  Device: /dev/nvme0n1
  Size:   1T
  Model:  Samsung 970 EVO Plus

[WARNING] ALL DATA ON THIS DISK WILL BE PERMANENTLY ERASED!

Are you absolutely sure you want to continue? (yes/no): yes
```

### 3. Automatic Partitioning
- Wipes existing data
- Creates GPT partition table
- Creates EFI boot partition (512MB, FAT32)
- Creates root partition (remaining space, ext4)
- Formats and mounts partitions

### 4. Repository Setup
- Clones configuration repository
- Generates hardware-configuration.nix
- Sets up modular architecture

### 5. Interactive Configuration Wizard

The wizard guides you through:

**System Settings**
- Hostname

**Regional Settings**
- Timezone (70+ options)
- Locale (16 languages)
- Console keymap (13 layouts)
- X11 keyboard layout (12 layouts)

**Hardware Configuration**
- GPU driver (auto-detected)
- Audio (PipeWire)
- Bluetooth
- Printing

**Desktop Environment**
- Display server (Wayland/Xorg)
- Desktop environment or window manager
  - COSMIC
  - GNOME
  - Hyprland
  - i3
  - XFCE
  - Awesome

**Services**
- Docker
- Ollama
- SSH

**User Configuration**
- Username
- Full name
- UID/GID
- Groups
- Sudo access

### 6. Installation
- Installs NixOS with generated configuration
- Sets root password
- Displays completion summary

### 7. Reboot
- Shows installation summary
- Prompts to reboot into new system

## What Gets Installed

### Partition Layout

```
/dev/sdX1  512MB   FAT32   /boot  (EFI System Partition)
/dev/sdX2  Rest    ext4    /      (Root filesystem)
```

### Directory Structure

```
/mnt/etc/nixos/
├── configuration.nix           # Main entry point
├── hardware-configuration.nix  # Auto-generated hardware config
├── flake.nix                   # Flakes support
├── modules/
│   ├── config.nix             # Centralized configuration
│   ├── core/                  # Core system modules
│   ├── locale/                # Regional settings
│   ├── hardware/              # Hardware drivers
│   ├── graphics/              # Display servers
│   ├── desktop/               # Desktop environments
│   ├── services/              # System services
│   ├── storage/               # Storage configuration
│   └── users/                 # User management
├── profiles/                   # Pre-configured profiles
└── install/                    # Installation tools
```

## Requirements

### Minimum System Requirements
- **CPU**: x86_64 processor
- **RAM**: 2GB minimum, 4GB recommended
- **Disk**: 20GB minimum, 50GB+ recommended
- **Network**: Internet connection required

### NixOS Live USB
- NixOS Minimal ISO (recommended)
- NixOS Graphical ISO (also works)
- Download from: https://nixos.org/download

## Network Configuration

If you don't have internet connectivity:

### Wired Connection
Usually works automatically via DHCP.

### WiFi Connection

**Option 1: nmtui (Text UI)**
```bash
sudo systemctl start wpa_supplicant
sudo nmtui
```

**Option 2: wpa_supplicant (Manual)**
```bash
sudo wpa_supplicant -B -i wlan0 -c <(wpa_passphrase 'SSID' 'password')
sudo dhcpcd wlan0
```

## Customization

### Before Installation

Edit the script to change defaults:
- Branch name (line 231)
- Partition sizes (lines 178-183)
- Default filesystem (currently ext4)

### After Installation

All configuration is in `/etc/nixos/modules/config.nix`:

```nix
config.system.config = {
  system.hostname = "my-nixos";
  
  locale = {
    timezone = "America/Sao_Paulo";
    language = "pt_BR.UTF-8";
  };
  
  hardware.gpu = "amd";
  
  graphics = {
    server = "wayland";
    desktop = "cosmic";
  };
  
  services = {
    docker.enable = true;
  };
};
```

Apply changes:
```bash
sudo nixos-rebuild switch
```

## Troubleshooting

### Installation Fails

**Check internet connection:**
```bash
ping -c 3 8.8.8.8
```

**Check disk space:**
```bash
df -h /mnt
```

**Check logs:**
```bash
journalctl -xe
```

### Configuration Wizard Fails

**Run manually:**
```bash
cd /mnt/etc/nixos
nix-shell -p python3 --run "python3 install/configurator.py"
```

### Hardware Not Detected

**List hardware:**
```bash
lspci  # PCI devices
lsusb  # USB devices
lsblk  # Block devices
```

**Update hardware-configuration.nix:**
```bash
sudo nixos-generate-config
```

### Boot Issues

**Boot into recovery:**
- Select "NixOS - Configuration X (rescue)" in GRUB

**Check boot partition:**
```bash
ls /boot/loader/entries/
```

**Reinstall bootloader:**
```bash
sudo nixos-rebuild boot
```

## Advanced Usage

### Custom Partitioning

Skip automatic partitioning and do it manually:

1. Comment out `partition_disk` in script
2. Partition manually before running script
3. Mount to `/mnt` and `/mnt/boot`
4. Run script

### LVM Setup

For LVM support:

```bash
# Create physical volume
pvcreate /dev/sdX2

# Create volume group
vgcreate vg0 /dev/sdX2

# Create logical volumes
lvcreate -L 50G -n root vg0
lvcreate -l 100%FREE -n home vg0

# Format and mount
mkfs.ext4 /dev/vg0/root
mkfs.ext4 /dev/vg0/home
mount /dev/vg0/root /mnt
mkdir /mnt/home
mount /dev/vg0/home /mnt/home
```

### ZFS Setup

For ZFS support:

```bash
# Create ZFS pool
zpool create -o ashift=12 -O compression=lz4 -O atime=off rpool /dev/sdX2

# Create datasets
zfs create -o mountpoint=none rpool/root
zfs create -o mountpoint=legacy rpool/root/nixos
zfs create -o mountpoint=legacy rpool/home

# Mount
mount -t zfs rpool/root/nixos /mnt
mkdir /mnt/home
mount -t zfs rpool/home /mnt/home
```

### Multiple Disks

For RAID or multiple disks:

```bash
# Software RAID (mdadm)
mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sda2 /dev/sdb2

# Or ZFS mirror
zpool create -o ashift=12 rpool mirror /dev/sda2 /dev/sdb2
```

## Security Considerations

### Disk Encryption

For LUKS encryption:

```bash
# Encrypt partition
cryptsetup luksFormat /dev/sdX2
cryptsetup open /dev/sdX2 cryptroot

# Format and mount
mkfs.ext4 /dev/mapper/cryptroot
mount /dev/mapper/cryptroot /mnt
```

Then update `hardware-configuration.nix` with:
```nix
boot.initrd.luks.devices.cryptroot = {
  device = "/dev/sdX2";
  preLVM = true;
};
```

### Secure Boot

Enable in `modules/core/boot.nix`:
```nix
boot.loader.systemd-boot.enable = true;
boot.loader.efi.canTouchEfiVariables = true;
```

## Contributing

Found a bug or want to improve the installer?

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

## Support

- **GitHub Issues**: https://github.com/julas23/nix-file/issues
- **NixOS Manual**: https://nixos.org/manual/nixos/stable/
- **NixOS Discourse**: https://discourse.nixos.org/

## License

MIT License - See LICENSE file for details

## Acknowledgments

- NixOS community for the amazing distribution
- Contributors to the modular architecture
- Everyone who tested and provided feedback

---

**Happy NixOS Installation!** 🚀
