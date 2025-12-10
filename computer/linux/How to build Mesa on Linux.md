parentTopic:: [[Linux]]

# Building Mesa
Install required libs
```bash
sudo apt-get install libgl1-mesa-dev
sudo apt-get -o Dpkg::Options::="--force-overwrite" install --fix-broken
sudo apt-get install libgl1-mesa-dev
sudo apt-get install freeglut3-dev
sudo apt-get install libglew-dev
```

Install Meson (build tool) - apt-get installs too old version, have to download it manually
```bash
cd
git clone https://github.com/mesonbuild/meson
sudo chmod +x meson/meson.py
```

Build Mesa. Remember to properly setup `-Dprefix`, so you don't overwrite your own system installation.
```bash
git clone https://gitlab.freedesktop.org/mesa/mesa     # (WARNING: symlinks in repo, don't clone on Windows!)
cd mesa
~/meson/meson.py build/
cd build
mkdir installation
~/meson/meson.py configure -Ddri-drivers=i915,i965 -Dgallium-drivers=i915,iris -Ddebug=true -Dprefix=`realpath installation` -Doptimization=0
~/meson/meson.py --reconfigure
time ninja install
```

Some misc notes
- Libdrm version might be too low, in that case build it and install yourself - http://www.linuxfromscratch.org/blfs/view/svn/x/libdrm.html
    ```bash
    sudo apt-get install libpciaccess-dev
    ```
- glxinfo
    - useful for checking driver capabilities, like extensions, or hardware constants
    - mesa-utils package
    - requires display
