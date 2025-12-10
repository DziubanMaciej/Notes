parentTopic:: [[Linux]]

# Linux select kernel

Grub usually selects first kernel it finds by default. We can change this behavior by configuring it.

1. Install new kernel
2. Enumerate installed kernels
    ```bash
    grep -Eo "\s*(menuentry|submenu) '[^']+'"  /boot/grub/grub.cfg
    grep -Eo "menuentry '[^']+'"  /boot/grub/grub.cfg | tail +2 | awk '{print NR-1 ". " $0}'
    ```
3. Write kernel selection to the configuration file
    ```bash
    sudo nano /etc/default/grub
    GRUB_DEFAULT="1>6"    (1 means "Advanced boot options", 6 is the index of selected kernel)
    ```
4. Run `sudo update-grub`
5. Run `sudo reboot`
6. Verify with `uname -a`
