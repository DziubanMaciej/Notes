parentTopic:: [[Linux]]

# Installing HP printer on Linux

1. Install software
    ```bash
    yay -S cups hplip
    ```
2. Enable cups service
    ```bash
    systemctl enable cups
    systemctl start cups
    ```
3. Add printer to CUPS
    - Add user to the group `sudo gpasswd -a maciej sys`
    - via hplip (recommended)
        - Run `hp-setup`
        - Select Network connection
        - Printer should be detected and it should be easy to set it up
        - If PPD file is not found automatically, look for it in `/usr/share/ppd/HP`. There are `.gz` files in this folder, it may be necessary to unpack them.
        - Let it download and install plug-ins from HP website
    - or via cups (TODO: check if it's sufficient to only do this, and omit `hp-setup`)
        - Go to `http://localhost:631/admin`
        - Select Add printer
    - verify that printer is added in `http://localhost:631/printers/`
4. Printer should be visible in applications such as Firefox or qpdfview
