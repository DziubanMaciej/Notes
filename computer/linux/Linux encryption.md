parentTopic:: [[Linux]]

# dm-crypt
Part of the linux kernel. It is responsible for encrypting partitions/drives File `/proc/crypto` contains a list of supported ciphers and modes.

# cryptsetup
User-space utility for interacting with `dm-crypt`.


# Encryption schemes
#### TrueCrypt/Veracrypt
TrueCrypt was an open-source project for on-the-fly encrypting. It has been discontinued and superseded by Veracrypt, which extended its encryption format header. The format is supported by the Linux kernel (`dm-crypt`).

Argument to `cryptsetup` is `--type tcrypt --veracrypt`. Technically `--veracrypt` should no longer be needed.

#### Plain
Not recommended for normal people. Very error prone.

#### LUKS
Linux Unified Key Setup. The most popular one.

#### Bitlk
Window Bitlocker.

#### Loop AES:
Old, not recommended.

# Operations
#### Creating encrypted partition
Not supported by `dm-crypt`/`cryptsetup`. Install `veracrypt` and use the GUI.

#### Opening partition
```bash
device_name="SecretStuff"
encrypted_file_path="/home/jonasz/MyEncryptedStuff"
decrypted_device_path="/dev/mapper/$device_name"
decrypted_mount_path="/home/jonasz/MyDecryptedStuff"

mkdir "$decrypted_mount_path" # Might need to setup permissions
cryptsetup --type tcrypt --veracrypt open "$encrypted_file_path" "$device_name"
mount "$decrypted_device_path" "$decrypted_mount_path" -o dmask=000 -o fmask=000
```
#### Closing partition
```bash
encrypted_file_path="/home/jonasz/SecretStuff"
decrypted_device_path="/dev/mapper/SecretStuff"
decrypted_mount_path="/mnt/MyMount"

umount "$decrypted_mount_path"
rmdir "$decrypted_mount_path"
cryptsetup close "$device_name"
```
