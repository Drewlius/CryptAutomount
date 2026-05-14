					  CryptSetup Walkthrough: Luks-AutoMounting 
					 	-Good Luck, Don't Get Locked Out-
---
>### For a drive/partition that is already Encrypted, do this first
```
sudo cryptsetup luksOpen /dev/sdx1 or /dev/nvme2n1p2
```
>>provide password when prompted
>### To identify storage drives and partitions
```
sudo blkid
```
>### Find Your Wanted Drive
```
/dev/nvme0n1p3: UUID="387c4dcd-1111-1111-1111-cb64437bb111" TYPE="crypto_LUKS" PARTLABEL="root" PARTUUID="ac064bb b-1111-1111-1111-df7150282111"
/dev/nvme0n1p1: UUID="1111-1111" BLOCK_SIZE="512" TYPE="vfat" PARTUUID="10042a55-1111-1111-1111-2d773b756111"
/dev/nvme0n1p4: UUID="1811-DFB8" BLOCK_SIZE="512" TYPE="vfat" PARTUUID="497ee031-1111-1111-1111-2380a4160111"
/dev/nvme0n1p2: UUID="b3062223-1111-1111-1111-405421a03111" TYPE="crypto_LUKS" PARTLABEL="root" PARTUUID="6e58c238-1111-1111-1111-d876a81db111"
/dev/mapper/luks-b3062223-1111-1111-1111-405421a03111: UUID="8e3c0a98-1111-1111-1111-b0d36e63a111
```
>> Example Output
>### Generate A Key-File
```
sudo dd if=/dev/urandom of=/root/<luks-keyfile> bs=512 count=8
or
su root head -c 32 /dev/urandom > /root/<luks-keyfile>(.bin may be necessary you can try without first)
```
```
8+0 records in
8+0 records out
4096 bytes (4.1 KB, 4.0 KiB) copied, 4.7317e-05 s, 86.6 MB/s
```
> > You should receive something like that as an output (if sudo dd is used)
>###  Reduce Permissions of key-file to root only
```
sudo chmod 400 /root/luks-keyfile
```
>### Verify Root Access
```
ls -la /root/luks-keyfile
```
```
r-------- 4.1k root  7 May 02:55 󰡯 /root/luks-keyfile
```
> >Example Output
>### Add Your Key to the partition
```
sudo cryptsetup luksAddKey /dev/nvme2n1p2 /root/luks-keyfile
```
> > Enter Password When Prompted
>### Verify Key Is Loaded Into the Dive
```
sudo cryptsetup luksOpen /dev/nvme2n1p2 test-volume --key-file /root/luks-keyfile
```
>### Next open /etc/fstab
```
sudo nano /etc/fstab
```
>### Verify your dev/mapper UUID with 
```
cryptsetup luksUUID /dev/sdX1 and blkid /dev/mapper/
or 
if you manually set name I.E. if you ran ran command: sudo cryptsetup luksOpen /dev/sdx1 --keyfile /root/luks-keyfile *custom-name* use that /dev/mapper/*custom name*
```
>### Format an entry into /etc/fstab
```
/dev/mapper/luks-4b088738-1111-1111-1111-e422dbda2111   /mnt/WayFire btrfs   noauto,nofail                          0 2
```
>> noauto,nofail is used to prevent your device from failing to boot if something was configured wrong. noauto can be removed after you have verified everything is working properly, as a precaution I would let nofail remain. ***PS: crypttab and fstab both need to have identical options I.E. nofail,noauto
>### Next Open your /etc/crypttab and again format your entry to resemble this
```
<Device Name>                                   UUID=<Device's UUID>     /root/luks-keyfile luks,noauto,nofail
```
---
>### Additional Resources can be located [Here](https://oneuptime.com/blog/post/2026-03-02-how-to-set-up-luks-key-files-for-automated-decryption-on-ubuntu/view) or using your shell/console/terminal
```
man cryptsetup
```
---
## For Reference Here are my entries in /etc/fstab
```
/dev/mapper/luks-4b088738-1111-1111-1111-e422dbda2111   /mnt/WayFire btrfs   nofail,noauto                          0 2
```
## And for /etc/crypttab
```
luks-4b088738-1111-1111-1111-e422dbda2111 UUID=4b088738-1111-1111-1111-e422dbda2111     /root/luks-keyfile luks,noauto,nofail
```