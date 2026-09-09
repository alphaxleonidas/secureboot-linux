  # Update Interrupted
  ```
sudo -i 
rm -i -v /var/lib/pacman/db.lck
```

## Secureboot

Disable grub shim lock:
```
sudo grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=cachyos --modules="tpm" --disable-shim-lock
```
Signing the bootloader:
```
sudo pacman -S sbctl

sudo sbctl create-keys

sudo sbctl enroll-keys --microsoft --firmware-builtin

sudo sbctl-batch-sign

sudo sbctl sign -s -o /usr/lib/systemd/boot/efi/systemd-bootx64.efi.signed /usr/lib/systemd/boot/efi/systemd-bootx64.efi

sudo sbctl status
```

[CachyOS Secureboot Wiki](https://wiki.cachyos.org/configuration/secure_boot_setup/)
