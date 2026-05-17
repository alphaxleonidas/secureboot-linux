# cachyos-secureboot
Commands to enable secure boot on cachyOS

# Update interrupted

```
sudo -i 
rm -i -v /var/lib/pacman/db.lck
```

# Commands only

Pre-setup
GRUB Boot Manager

If you are using GRUB, run the following command to enable secure boot support on GRUB using CA Keys.
Terminal window

```
sudo grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=cachyos --modules="tpm" --disable-shim-lock
```

```

systemctl reboot --firmware-setup

```
To Enable Setup Mode in BIOS/UEFi.

```
sudo pacman -S sbctl

sudo sbctl create-keys

sudo sbctl enroll-keys --microsoft --firmware-builtin

sudo sbctl-batch-sign

sudo sbctl sign -s -o /usr/lib/systemd/boot/efi/systemd-bootx64.efi.signed /usr/lib/systemd/boot/efi/systemd-bootx64.efi

sudo sbctl status

```

# Steps:


Enable Setup Mode of Secure Boot in BIOS/UEFI

`systemctl reboot --firmware-setup`

Pre-setup
GRUB Boot Manager

If you are using GRUB, run the following command to enable secure boot support on GRUB using CA Keys.

`sudo grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=cachyos --modules="tpm" --disable-shim-lock`

Installing sbctl and Enrolling Keys

Before proceeding, make sure to check if sbctl is installed.

sbctl is a user-friendly secure boot key manager capable of setting up secure boot, offering key management capabilities, and keeping track of files that need to be signed in the boot chain.
How to install sbctl
Open a terminal and run the following command

```sudo pacman -S sbctl```

Now that sbctl is installed, you have to setup sbctl and enroll your keys to the firmware. This process is pretty straightforward, just follow the steps below.

    Check if Setup Mode is enabled:
    Terminal window

    ```
    sudo sbctl status
    ```

    Expected Output
    Terminal window

    Installed:      ✘ sbctl is not installed
    Setup Mode:     ✘ Enabled
    Secure Boot     ✘ Disabled

    Create your custom Secure Boot keys:
    Terminal window

    ```sudo sbctl create-keys```

    Example of a successful key creation
    Terminal window

    Created Owner UUID a9fbbdb7-a05f-48d5-b63a-08c5df45ee70
    Creating secure boot keys...✔
    Secure boot keys created!

    Enroll your keys with Microsoft’s and the OEM firmware’s built-in keys:
    Terminal window

    ```sudo sbctl enroll-keys --microsoft --firmware-builtin```

    Expected output
    Terminal window

    Enrolling keys to EFI variables...✔
    Enrolled keys to the EFI variables!

    ASUS Motherboards — Do not use
    --firmware-builtin

    On some ASUS motherboards using the --firmware-builtin flag causes duplicate entries in the EFI key variables (e.g. builtin-db appearing multiple times in Vendor Keys). When activating Secure Boot in the UEFI settings afterwards, the board detects this inconsistent key structure and throws a Secure Boot Violation before the boot manager can load.

    Use only --microsoft on ASUS boards:
    Terminal window

    ```sudo sbctl enroll-keys --microsoft```

    If Vendor Keys shows microsoft builtin-db builtin-db ..., the keys need to be re-enrolled. Re-enter Setup Mode in the UEFI (using delete all keys), then run sbctl enroll-keys --microsoft again without --firmware-builtin.

    Check the status of sbctl again to make sure that the keys are enrolled and setup mode is disabled:
    Terminal window

    ```sudo sbctl status```

    Expected Output
    Terminal window

    Installed:      ✔ sbctl is installed
    Owner GUID:     a9fbbdb7-a05f-48d5-b63a-08c5df45ee70
    Setup Mode:     ✔ Disabled
    Secure Boot     ✘ Disabled
    Vendor Keys:    microsoft




    Signing the Kernel Image and Boot Manager

    systemd-boot

CachyOS provides sbctl-batch-sign, a script that takes the list of files needed to be signed from sudo sbctl verify and signs them all.

Caution

On systems with a separate /boot and /boot/efi partition layout, sbctl may only scan for EFI binaries in /boot/efi. This causes kernel images that are in /boot to not be detected. sbctl-batch-sign works around this by always scanning /boot for vmlinuz-* files.
Terminal window

```sudo sbctl verify```
Verifying file database and EFI images in /boot...
✘ /boot/1c4b5246eef05ac3bc87339323cd5101/6.10.0-cn4.0.fc40.x86_64/linux is not signed
✘ /boot/EFI/BOOT/BOOTX64.EFI is not signed
✘ /boot/EFI/systemd/systemd-bootx64.efi is not signed
✘ /boot/1c4b5246eef05ac3bc87339323cd5101/0-rescue/linux is not signed
✘ /boot/1c4b5246eef05ac3bc87339323cd5101/6.10.0-cn3.0.fc40.x86_64/linux is not signed

```sudo sbctl-batch-sign```

```sudo sbctl verify```
Verifying file database and EFI images in /boot...
✔ /boot/1c4b5246eef05ac3bc87339323cd5101/6.10.0-cn4.0.fc40.x86_64/linux is signed
✔ /boot/EFI/BOOT/BOOTX64.EFI is signed
✔ /boot/EFI/systemd/systemd-bootx64.efi is signed
✔ /boot/1c4b5246eef05ac3bc87339323cd5101/0-rescue/linux is signed
✔ /boot/1c4b5246eef05ac3bc87339323cd5101/6.10.0-cn3.0.fc40.x86_64/linux is signed

Now that all the files are signed. Reboot your system and go to your UEFI Settings to enable Secure Boot.

ASUS Motherboards — enabling Secure Boot

Some ASUS motherboards behave differently from other vendors when enabling Secure Boot:

    Use Windows UEFI Mode, not Other OS: The name is misleading this setting simply controls whether Secure Boot is enforced. Setting OS Type to Other OS actively disables Secure Boot on ASUS boards, regardless of any other settings. sbctl status will continue to show Secure Boot: Disabled even after rebooting.

    There is no separate Secure Boot on/off toggle on some ASUS boards. Secure Boot is activated implicitly by selecting Windows UEFI Mode.

The correct ASUS BIOS configuration to activate Secure Boot in this case is:

Boot → Secure Boot
  OS Type          → Windows UEFI Mode
  Secure Boot Mode → Custom

Check this part for reference

Note that this is a one-time process as signing files with -s flag will save those files to sbctl’s database.

Reminder

sbctl ships with a pacman hook meaning it will automatically sign all new files upon a kernel or boot manager update.

CachyOS uses systemd-boot-update.service provided by systemd to update the boot manager on reboot. This means that the sbctl pacman hook will not sign the updated EFI binaries. As a workaround, we can sign the boot manager directly
Terminal window

```sudo sbctl sign -s -o /usr/lib/systemd/boot/efi/systemd-bootx64.efi.signed /usr/lib/systemd/boot/efi/systemd-bootx64.efi```

Secure Boot Status Check

To check that secure boot is indeed enabled. You can run one of the following commands
Terminal window

```sudo sbctl status```
Installed:      ✓ sbctl is installed
Owner GUID:     a9fbbdb7-a05f-48d5-b63a-08c5df45ee70
Setup Mode:     ✓ Disabled
Secure Boot:    ✓ Enabled
Vendor Keys:    microsoft

bootctl
System:
      Firmware: UEFI 2.80 (INSYDE Corp. 28724.16435)
 Firmware Arch: x64
   Secure Boot: enabled (user)
  TPM2 Support: yes
  Measured UKI: no
  Boot into FW: supported

