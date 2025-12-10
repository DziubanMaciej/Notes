parentTopic:: [[Linux]]

# Xorg dotfiles

```mermaid
graph TD

boot[Computer launch]
tty[Terminal login]

subgraph Graphical environemnt
    xinit[xinit]
    X[X server]
    DM[Display Manager]
    startx[Startx]
    WM[Window manager]

    xinitrc(.xinitrc)
    xprofile(.xprofile)
    session("Session (.desktop file)")
end

subgraph Textual environment
    profile_t(.profile)
    bashrc_t(.bashrc)
end

boot --> tty
boot -->|Run by systemd| DM
xinit --> xinitrc
tty --> profile_t
tty -->|Run by user| startx
xinit --> X
DM --> xprofile --> session
DM --> X
profile_t --> bashrc_t
bashrc_t -->|Run by script| startx
startx --> xinit
xinitrc --> WM
session --> WM
```

### Commands
xinit - executable binary which starts the X server and a window manager (rarely used directly)
startx - a frontend shell script to call xinit

### Dotfiles
- .xinitrc - configures X environment, runs background startup programs, runs main program (window manager)
- .xsession - similar to .xinitrc, used by display managers. Sometimes it may be necessary to create .xsession package just so the display manager calls our .xinitrc (see `xinit-xsession` package)
- .xprofile - similar to .xsession, used by different display managers
- .profile - setup not related to Xorg or any particular shell
- .bashrc - bash-specific setup (non login shells)
- .bash_profile - bash-specific setup (login shells)

### Other files
- Xsession files - .desktop files located in `/usr/share/xsessions/`.
- Xssesion scripts - in app specific directories, e.g. `/etc/lightdm/Xsession`.
