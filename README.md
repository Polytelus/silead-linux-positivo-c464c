![GitHub Repo stars](https://img.shields.io/github/stars/Polytelus/silead-linux-positivo-c464c)

# Silead Touchscreen Driver for Linux on Positivo Duo C464C

> Guide to install and configure the Silead `mssl1680` touchscreen driver. I had some copilot help because I was very tired and the guide looked confusing as all hell, but credits and some parts of the guide (like this summary) are human-edited.
---
 
## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Firmware Installation](#firmware-installation)
3. [Bootloader Configuration](#bootloader-configuration)
4. [ChromeOS / CrOS Flex Installation](#chromeos--cros-flex-installation)
5. [Verification & Troubleshooting](#verification--troubleshooting)

---

## Prerequisites

* You'll need ```sudo``` and the **firmware file**.

---

## Firmware Installation

1. **Clone the repo**

  ```bash 
  git clone https://github.com/Polytelus/silead-linux-positivo-c464c
  cd silead-linux-positivo-c464c
  ```

2. **Create firmware directory**

   ```bash
   sudo mkdir -p /lib/firmware/silead
   ```
3. **Copy `mssl1680.fw`**

   * From local source:

     ```bash
     sudo cp ./mssl1680.fw /lib/firmware/silead/
     ```

---

## Bootloader Configuration

Add the calibration parameter:

```text
i2c_touchscreen_props=MSSL1680:touchscreen-min-x=5:touchscreen-min-y=12:touchscreen-size-x=1915:touchscreen-size-y=1272
```

### For GRUB (Ubuntu, Fedora, Arch)

1. Edit `/etc/default/grub`:

   ```bash
   sudo nano /etc/default/grub
   ```
2. Append parameter to `GRUB_CMDLINE_LINUX_DEFAULT`:

   ```diff
   - GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
   + GRUB_CMDLINE_LINUX_DEFAULT="quiet splash i2c_touchscreen_props=MSSL1680:touchscreen-min-x=5:touchscreen-min-y=12:touchscreen-size-x=1915:touchscreen-size-y=1272"
   ```
3. Save, then update and reboot:

   ```bash
   sudo update-grub && sudo reboot # Should be grub-mkconfig -o /boot/grub/grub.cfg for Arch
   ```

### For ChromeOS EFI Boot

1. Mount EFI partition:

   ```bash
   sudo mkdir -p /mnt/efi
   sudo mount /dev/sdX12 /boot/efi  # Adjust sdX12, mmcblkXp12
   ```
2. Edit `/mnt/efi/EFI/Boot/grub.cfg`:

   ```bash
   sudo nano /mnt/efi/EFI/Boot/grub.cfg
   ```
3. Append parameter to the `linux` or `linuxefi` line (see above).
4. Reboot and test.

---

## ChromeOS / CrOS Flex Installation

1. **Enable Developer Mode**: Power off → `Esc+Refresh` + `Power` → `Ctrl+D`.
2. **Open Crosh with ```CTRL+ALT+T``` & Access the Shell**:

   ```bash
   shell
   sudo su -
   ```
3. **Mount EFI**:

   ```bash
   mkdir -p /mnt/EFI
   mount /dev/mmcblk0p12 /mnt/EFI
   ```
4. **Edit GRUB config**:

   ```bash
   cd /mnt/EFI/EFI/Boot
   nano grub.cfg
   # Append i2c_touchscreen_props...
   ```
5. **Reboot** and verify touchscreen.

---

## Verification & Troubleshooting

* **Firmware loaded?**

  ```bash
  dmesg | grep silead
  ```
* **Parameter applied?**

  ```bash
  cat /proc/cmdline
  ```

### Common Issues & Tips

* **Incorrect firmware path or checksum**: Re-download and verify.
* **Calibration offsets**: Tweak `min-x` / `min-y` values.
* **Bootloader edits not applied**: Double-check mount points and rerun `update-grub` or remount EFI.

### Alternative Firmware & Calibration

If you encounter persistent touch issues or require finer calibration, I strongly recommend exploring the [gsl-firmware repository](https://github.com/onitake/gsl-firmware) by Onitake. The project provides:

* Additional firmware variants for Silead controllers.
* Community-contributed fixes and enhancements.

> **Credits:** The guide builds on the work of [onitake/gsl-firmware](https://github.com/onitake/gsl-firmware). All kudos to him.

---
