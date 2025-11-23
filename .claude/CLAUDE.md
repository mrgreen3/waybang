# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## ArchBANG ISO Build Project

ArchBANG is a minimal Arch Linux-based distribution featuring the Openbox window manager, designed as a lightweight live/rescue ISO. The project is structured as a standard Arch ISO build profile using mkarchiso.

## Build System Architecture

The ISO build process is orchestrated by a single large build script (1890 lines) that:
1. Validates build requirements (pacman, mkarchiso, mksquashfs, etc.)
2. Copies and customizes the airootfs (live filesystem template)
3. Installs 174 packages via pacstrap
4. Runs customize_airootfs.sh for post-install configuration
5. Creates a SquashFS image (XZ compressed)
6. Sets up dual-boot: BIOS/Syslinux and UEFI/systemd-boot
7. Generates the final ISO 9660 filesystem

**Key Files:**
- **`build`** — Main mkarchiso-based orchestration script with 50+ functions
- **`profiledef.sh`** — Metadata: ISO name/label, boot modes (bios.syslinux, uefi.systemd-boot), compression (SquashFS XZ, Bootstrap zstd)
- **`pacman.conf`** — Build system package manager config (parallel downloads enabled)
- **`packages.x86_64`** — 174 packages organized by function (base system, Xorg, Openbox stack, utilities, drivers, bootloaders)

## Live Environment Architecture

**Desktop Stack:**
- Window Manager: Openbox (configured in `etc/skel/.config/openbox/rc.xml`)
- Panel: Polybar with Rofi launcher, workspace switcher, audio control, system tray
- Notifications: Dunst daemon
- System Monitor: Conky widget
- Terminal: Alacritty with Tokyo Night theme
- Applications: Firefox, GIMP, LibreOffice, development tools

**Boot & Session:**
1. BIOS/UEFI bootloader (Syslinux or systemd-boot)
2. Kernel loads with archiso hooks (base, udev, microcode, archiso_loop_mnt, block, filesystems)
3. systemd boots with graphical.target
4. getty@tty1 auto-logins to "ablive" user (configured in `systemd/system/getty@tty1.service.d/autologin.conf`)
5. X11 session launches Openbox
6. Openbox autostart script runs: background, battery monitor, network manager, Polybar, Dunst, Conky

**User Environment Template (`etc/skel/`):**
- Openbox + Polybar configuration with ArchBANG theme
- GTK3 settings and custom theme
- Alacritty terminal config
- Xfce4 helpers (for file manager/terminal preferences)
- AB_Scripts: abinstall (disk installer), startpanel, scrot-notify, ab-gparted, archbang_yay

## Boot Configurations

**BIOS/Syslinux (`syslinux/syslinux.cfg`):**
- Menu with custom splash (syslinux/splash.png)
- Default: archbang (Openbox live), 15-second timeout
- Kernel options: archisobasedir, archisolabel, cow_spacesize=4G (4GB copy-on-write space)
- Additional options: existing OS boot, Memtest86+, HDT, reboot, poweroff

**UEFI/systemd-boot (`efiboot/loader/`):**
- Default entry: ArchBANG Linux (Openbox) with same kernel parameters
- Timeout: 15 seconds
- Also includes: Memtest86+ entry

## Post-Install Customization

The `airootfs/root/customize_airootfs.sh` script runs during build to:
- Set en_US.UTF-8 locale
- Enable pacman parallel downloads
- Configure sudo (wheel group NOPASSWD)
- Create "ablive" live user (no password, wheel group)
- Set hostname to "archbang", timezone to Canada/Montreal
- Enable Pipewire/WireePlumber audio
- Enable pacman-init and NetworkManager services
- Set console to US keyboard layout with Lat2-Terminus16 font

## Building the ISO

```bash
./build
```

Basic build creates: `out/archbang-DDMM-x86_64.iso` (label ARCHBANG_DDMM)

The build system creates:
1. Work directory with pacstrap environment
2. airootfs files copied with special permissions (0600 for /etc/shadow, 0700 for /root/.gnupg)
3. SquashFS rootfs image (XZ compressed with BCJ x86 filter)
4. ISO 9660 filesystem with hybrid MBR for BIOS/UEFI compatibility

## Customization Points

**Add/Remove Packages:**
Edit `packages.x86_64` (174 packages organized by category), rebuild with `./build`

**Modify Desktop Environment:**
Change files in `airootfs/etc/skel/.config/`:
- Openbox keybindings: `openbox/rc.xml`
- Panel layout/colors: `polybar/config.ini`
- Application launcher: `rofi/config.rasi` and `rofi/archbang.rasi`
- System monitors: `conky/conky.conf`
- Terminal theme: `alacritty/alacritty.toml` and `alacritty/themes/`

**System Boot Configuration:**
- BIOS menu: `syslinux/syslinux.cfg`
- UEFI entries: `efiboot/loader/entries/*.conf`
- Kernel parameters: Modify `archisobasedir`, `archisosearchuuid`, `cow_spacesize` in boot configs

**Live Environment Scripts:**
- Creation: `airootfs/root/customize_airootfs.sh`
- Auto-launched: `airootfs/etc/skel/.config/openbox/autostart`
- User scripts: `airootfs/etc/skel/AB_Scripts/`

## Development Workflow

1. Modify configuration or code
2. Test build: `./build` (takes 5-15 minutes depending on system)
3. Boot ISO in VM or hardware to verify
4. Test both BIOS and UEFI boot modes
5. Commit changes with descriptive messages
6. Update `todo` file with any remaining tasks

## Architecture Summary

| Component | Purpose | Location |
|-----------|---------|----------|
| Build Script | ISO orchestration (1890 lines) | `./build` |
| Profile Config | Metadata, boot modes, compression | `profiledef.sh` |
| Package List | 174 essential packages | `packages.x86_64` |
| Live Filesystem | Boot-time environment template | `airootfs/` |
| Openbox Config | Window manager settings | `airootfs/etc/skel/.config/openbox/` |
| Polybar Config | System panel | `airootfs/etc/skel/.config/polybar/` |
| BIOS Boot | Legacy bootloader | `syslinux/` |
| UEFI Boot | EFI system partition config | `efiboot/` |
| Post-Install | Live user & system setup | `airootfs/root/customize_airootfs.sh` |

## Key Build Considerations

- **Copy-on-Write Space:** Default 4GB (cow_spacesize=4G) — increase if live environment fills RAM
- **Compression:** SquashFS XZ (good compression, moderate speed) — alternatives: EROFS or ext4+squashfs
- **Dual Boot:** Configured for both BIOS/UEFI with automatic detection
- **Permissions:** Special handling: /etc/shadow (0600), /root (0750), /root/.gnupg (0700), scripts (0755)
- **Architecture:** x86_64 only
- **ISO Size:** Typically 1.5-2.0 GB compressed

## References

- [mkarchiso Documentation](https://man.archlinux.org/man/mkarchiso.8)
- [Arch Wiki - Archiso](https://wiki.archlinux.org/title/Archiso)
- [ArchBANG Website](https://www.archbang.org)
