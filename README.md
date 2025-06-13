# secureboot-grub-arch-artix
How to enable Secure Boot with GRUB in Arch and Arch-Based distros.

I took this from [this reddit post](https://www.reddit.com/r/archlinux/comments/10pq74e/my_easy_method_for_setting_up_secure_boot_with/) credits to the original writer


This post is both for my own reference and for anyone with GRUB who is struggling to set up Secure Boot. Here is what I found to be the easiest method:

### Disclaimer: This method does not work with "Secured-core" PCs

Re-install GRUB to utilize Microsoft's CA certificates (as opposed to shim) -- replace 'esp' with your EFI system partition:
```
sudo grub-install --target=x86_64-efi --efi-directory=esp --bootloader-id=GRUB --modules="tpm" --disable-shim-lock
```
Regenerate your grub configuration:
```
sudo grub-mkconfig -o /boot/grub/grub.cfg
```
Install the sbctl tool:
```
sudo pacman -S sbctl
```
As a pre-requisite, in your UEFI settings, set your secure boot mode to setup mode.

Upon re-booting, verify that you are in setup mode:
```
sbctl status
```
Create your custom secure boot keys:
```
sudo sbctl create-keys
```
Enroll your custom keys (note -m is required to include Microsoft's CA certificates)
### DO NOT MODIFY THIS IT COULD BREAK YOUR WHOLE PC AT THE HARDWARE LEVEL (Some firmware is signed and verified with Microsoft's keys when secure boot is enabled. Not validating devices could brick them. DO NOT use your keys without enrolling Microsoft's unless you know your shit)

```
sudo sbctl enroll-keys -m
```
Verify that your keys have successfully been enrolled:
```
sbctl status
``` 
Check which files need to be signed for secure boot to work:
```
sudo sbctl verify
```
Sign all unsigned files (below is what I needed to sign, adjust according to your needs):
```
sudo sbctl sign -s /efi/EFI/GRUB/grubx64.efi
```
You can use this command to sign everything (beware you may still need to to the next step)
```
sbctl verify | sed 's/✗ /sbctl sign -s /e'

```

You may also need to sign the kernels like this, (they may not show up in sbctl verify):
```
sudo sbctl sign -s /boot/vmlinuz-linux
```
You may get an error because of an issue with certain files being immutable. To make those files mutable, run the following command for each file then re-sign afterwards:
```
sudo chattr -i /sys/firmware/efi/efivars/<filename>
```
Verify that everything has been signed:
```
sudo sbctl verify
```
Finally, in your UEFI settings, enable secure boot, and reboot.

Verify that secure boot is enabled:
```
sbctl status
```
Note that sbctl comes with a pacman hook for automatic signing, so you don't need to worry when you update your system.
