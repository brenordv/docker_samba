# Samba server in a docker container
I created this repo so I'll have it as a reference for future installs.
Locally, I create this container in my Raspberry Pi.

> Worked on my machine. Follow at your own risk.


# Preparing disk
## Locating the disk
```shell
lsblk
```
> Let's imagine that the disk we want is `/dev/sda`.

## Formatting the disk
> Attention, ALL DATA IN THE DISK WILL BE LOST! PROCEED AT YOUR OWN RISK!
```shell
sudo mkfs.ext4 -F /dev/sda
```

## Creating the folder that will be used in share and mounting it
Here we create a folder called `/netStorage` and give it full acccess to everything. You might want to adapt this.
1. First, create a mount point for the disk by running the following command:
```shell
sudo mkdir /netStorage
```

2. Next, find the UUID of the formatted sda drive by running the following command:
```shell
sudo blkid /dev/sda
```
This will output the UUID of the sda drive, which you'll need to add to the /etc/fstab file.

3. Edit the /etc/fstab file by running the following command:
```shell
sudo nano /etc/fstab
```

4. Add the following line to the end of the file, replacing UUID with the `UUID` of the sda drive that you obtained in step 2:
```shell
UUID=<UUID> /netStorage ext4 defaults 0 2
```
This line tells the system to mount the sda drive with the specified UUID to `/netStorage` using the ext4 file system with default mount options at boot time.

5. Save and exit the /etc/fstab file by pressing Ctrl+O followed by Ctrl+X.

6. Finally, test the new configuration by running the following command to mount all entries in the /etc/fstab file:
```shell
sudo mount -a
```

If there are any errors, the system will report them. Otherwise, the sda drive should be mounted to `/netStorage` automatically at boot time.

# Creating Docker image.

```shell
docker run -d \
--name Samba \
--restart unless-stopped \
--privileged \
-p 139:139/tcp \
-p 445:445/tcp \
-e PUID=1000 \
-e PGID=1000 \
-e USERID=1000 \
-e GROUPID=1000 \
-e PERMISSIONS=true \
-e USER="guest;guest" \
-e WORKGROUP=WORKGROUP \
-v /netStorage:/share \
dperson/samba \
-g 'map to guest = never' \
-g 'restrict anonymous = 2' \
-s "netStorage;/share;yes;no;yes;guest"
```
