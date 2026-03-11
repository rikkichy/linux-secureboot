# Secure Boot on Linux

> This guide covers distros using the **Limine** bootloader. If your distro is not arch, just replace commands with your package manager.

---

## 1 · Install `sbctl`

```bash
sudo pacman -S sbctl
```

## 2 · Reboot into BIOS

```bash
sudo systemctl reboot --firmware-setup
```

## 3 · Enable Setup Mode

Once inside your BIOS/UEFI firmware:

1. Navigate to the **Secure Boot** section.
2. Locate **Setup Mode** and **enable** it.
3. Save and reboot.

> [!WARNING]
> **MSI and some other motherboards** have a known quirk where Setup Mode
> cannot be activated directly.
>
> **Workaround:** First **uncheck** *"Include factory keys"* and save. After
> that, you can enable Setup Mode without issues.

## 4 · Verify

```bash
sudo sbctl status
```

Expected output:

```
✓ Secure Boot:  Disabled
✓ Setup Mode:   Enabled
```

## 5 · Generate Custom Secure Boot Keys

```bash
sudo sbctl create-keys
```

This creates several key files in `/usr/share/secureboot/keys/`. Expected output:

```
Creating secure boot keys...✓
Created Owner UUID: [UUID string]
Creating keys...✓
```

## 6 · Enroll Keys into UEFI Firmware

```bash
sudo sbctl enroll-keys -m
```

The `-m` flag enrolls Microsoft keys alongside your custom keys, maintaining Windows compatibility. Expected output:

```
Enrolling keys to EFI variables...✓
Platform Key enrolled
Key Exchange Key enrolled
Signature Database enrolled
```

## 7 · Confirm Enrollment

```bash
sudo sbctl status
```

You should now see:

- Your custom keys listed
- Platform Key (PK), Key Exchange Key (KEK), and Signature Database (db) entries
- Setup Mode is now **Disabled**

## 8 · Update Initial RAM Disk Configuration

Edit the mkinitcpio configuration:

```bash
sudo nano /etc/mkinitcpio.conf
```

Add `btrfs-overlayfs` to the **HOOKS** array, then save and exit (`CTRL+X`).

## 9 · Rebuild Boot Configuration

Regenerate the initramfs and update the Limine configuration:

```bash
sudo limine-mkinitcpio
```

## 10 · Enable Secure Boot

Reboot into UEFI firmware setup:

```bash
sudo systemctl reboot --firmware-setup
```

In BIOS/UEFI:

1. Navigate to **Secure Boot** settings.
2. **Enable** Secure Boot.
3. Verify Secure Boot Mode is set to **Custom**.
4. Save and exit.
Now you have secure boot on Linux!
