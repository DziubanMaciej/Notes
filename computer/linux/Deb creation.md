parentTopic:: [[Linux]]

Pirv: 492F00955B85222B

pub   rsa3072 2024-02-27 [SC]
      70F58A98F4FE78B3BFD9B9B4492F00955B85222B
uid                      MaciexDziubix (aa) <aa@wp.pl>
sub   rsa3072 2024-02-27 [E]



BFAA8B33E5FB37A60AA97854A2CCCECD0FA0359A
                        A2CCCECD0FA0359A



Package creation tools
- `dpkg-deb --build` - not recommended, doesn't perform many checks.
- `dpkg-buildpackage`
- `debuild` - front-end wrapper for `dpkg-buildpackage`, `lintian`, `fakeroot` and `debsign`.

Uploading to PPA:
- PPA is a generic term for user-based package repositories, like AUR.
- Launchpad is Ubuntu's website for hosting PPAs
    - Create account on Launchpad. Setup GPG key and sign code of conduct with it.
    - One account can have multiple PPA. Create a PPA for the application.
    - Only source packages can be uploaded. Launchpad builds binary package and hosts it in PPA.
    - Launchpad builds binary packages for only one distribution version (they call it distribution series), e.g. 22.04 a.k.a jammy. It depends on the latest note in `debian/changelog`. Debian [documentation](https://www.debian.org/doc/manuals/maint-guide/dreq.en.html#changelog) says it must be "unstable", but Launchpad actually requires distro series such as "bionic", "focal" or "jammy".
- `dput ppa:<username>/<ppa_name> package_source.changes`

Other utilities:
- `dh_make` - generate initial version of files needed for packaging, like *DEBIAN/control*, etc.
- `lintian` - package verification
- `dh_fixperms` - allegedly fixes permissions and ownerships of package files. Not needed, if we package as root.
- `dpkg-gencontrolc` - for installed-size? https://stackoverflow.com/questions/18274633/how-to-properly-set-installed-size-for-deb-package
- `apt-get`:
    - `apt-get download <package>` - download .deb file of official package
    - `apt-get source <package>` - download source packages of an official package. This includes PGP signature, source code and package metadata.
    - `apt show <package>` - show metadata about official package
- `dch -i` - for managing changelog. Not sure if it's useful.

Alternative ways:
- `CPack` - [example](https://github.com/smolBlackCat/progress-tracker/blob/main/CMakeLists.txt) [docs](https://cmake.org/cmake/help/latest/cpack_gen/deb.html)
- `FPM` - a Ruby tool for making packages for various distros. [docs](https://fpm.readthedocs.io/en/v1.15.1/)
- `checkinstall` - 
- `makedeb` - similar syntax to PKGBUILD from Arch used to generate .deb packages. [docs](https://docs.makedeb.org/makedeb/pkgbuild-syntax/)

GPG keys:
- generate new key: `gpg --full-generate-key`
- Private keys are stored in `~/.gnupg`
- To specify the GPG key in various commands we use an identifier. Identifiers are not secret. An identifier can be:
    - fingerprint - 40 hexadecimal characters
    - key-id - last 8 characters of the fingerprint
    - email address
- Get private key: `gpg --export-secret-key --armor <identfier>`
- Get public key: `gpg --export --armor <identfier>`
- Do not call `gpg --export` without `--armor` option. Armor means encoding in base64 and adding headers identifying key type. Without this option the output will be raw binary data, which cannot be copied and will also mess up the terminal.
- Move key to a different machine:
    - source machine: `gpg --output key.gpg --export-secret-key --armor <identifier>`
    - copy the file
    - destination machine: `gpg --import key.gpg`
- Sign a file
    - binary `gpg --local-user <identifier> --sign <file>`
    - text `gpg --local-user <identifier> --clearsign <file>`