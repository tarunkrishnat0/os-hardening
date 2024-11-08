# Check if Yubikey is recognized
```sh
sudo apt install -y yubikey-manager
yubikey_number=$(ykman list --serials)
ykman --device $yubikey_number info
```

# Set YubiKey FIDO2 for FDE unlocking
```sh
enc_disk=$(blkid -t TYPE=crypto_LUKS -o device)
sudo cryptsetup isLuks $enc_disk && echo "$enc_disk is LUKS Encrypted"
sudo systemd-cryptenroll $enc_disk --fido2-device=auto --fido2-with-client-pin=false
```

# Crypttab

```sh
# Get the UUID of the encrypted partition
enc_disk_uuid=$(blkid -t TYPE=crypto_LUKS -o value -s UUID)
echo $enc_disk_uuid

# The below command shows the line which we need to edit.
sudo cat /etc/crypttab | grep $enc_disk_uuid
dm_crypt-0 UUID=164105d2-f4f2-481b-97b4-154941452774 none luks

# Add the below string at end of the corresponding disk(in this case dm_crypt-0).
sudo vim /etc/crypttab 
,discard,fido2-device=auto

# After editing the line, crypttab should look like this.
sudo cat /etc/crypttab | grep $enc_disk_uuid
dm_crypt-0 UUID=164105d2-f4f2-481b-97b4-154941452774 none luks
```

# Dracut
```sh
sudo mkdir /etc/dracut.conf.d
echo 'hostonly="yes"' | sudo tee /etc/dracut.conf.d/hostonly.conf
sudo apt install dracut
sudo dracut -f
```

## Why Dracut
Debian’s default initramfs generator, update-initramfs of the initramfs-tools is using the old cryptsetup for mounting encrypted drives. However, cryptsetup doesn’t recognize the fido2-device option.
The simplest solution is to switch to dracut, a more modern initramfs generator, which among other things relies on systemd to activate encrypted volumes. This solves the issue of the unknown fido2-device.

# References
## systemd-cryptenroll dracut fido2

https://askubuntu.com/questions/1516511/unlocking-luks-root-partition-with-fido2-yubikey-and-ideally-without-dracut
https://www.guyrutenberg.com/2022/02/17/unlock-luks-volume-with-a-yubikey/

https://gist.github.com/kirbah/e8691bdbdadf08a6b710138879614e0b
http://0pointer.net/blog/unlocking-luks2-volumes-with-tpm2-fido2-pkcs11-security-hardware-on-systemd-248.html
https://fedoramagazine.org/use-systemd-cryptenroll-with-fido-u2f-or-tpm2-to-decrypt-your-disk/

https://research.kudelskisecurity.com/2023/12/14/luks-disk-encryption-with-fido2/

## Yubikey HMAC Support
https://support.yubico.com/hc/en-us/articles/360016649319-YubiKey-5-2-Enhancements-to-FIDO-2-Support
https://github.com/Yubico/yubikey-personalization/issues/122
https://github.com/Yubico/yubikey-personalization/issues/122

https://ubuntuforums.org/showthread.php?t=2499034

