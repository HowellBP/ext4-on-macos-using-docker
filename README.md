Create and work on an ext4 filesystem image inside of macOS using Docker for Mac. Use this method if you aren't using external USB drives and/or don't want to install a fat VM like VirtualBox.

## 1. Install Docker
Follow the steps outlined on [Docker's website](https://docs.docker.com/docker-for-mac/install/).

## 2. Create a working folder
Pick a directory to start in: e.g. `mkdir -p $HOME/ext4fs`

## 3. Run an Ubuntu container
Run the container in privileged mode and mount the directory you just created:
```
docker run --privileged -it -v $HOME/ext4fs:/ext4fs ubuntu:18.04 bash
```

## 4. Create a raw disk image
Create a blank file and format it as ext4:
```
dd if=/dev/zero of=ext4fs.img bs=1G count=0 seek=6
# or set "seek=" to however many gigabytes you want
mkfs.ext4 ext4fs.img
losetup -fP --show ext4fs.img
# this will return a /dev/loop ID, e.g. /dev/loop0
mount /dev/loop0 /ext4fs
```

## 5. Verify the image is mounted
Enter the directory where the image is mounted to see if it formatted correctly:
```
root@b710398aa9e9:/# cd /ext4fs
root@b710398aa9e9:/ext4fs# ls -lah
total 20K
drwxr-xr-x 3 root root 4.0K Dec  5 23:47 .
drwxr-xr-x 5 root root  160 Dec  5 23:46 ..
drwx------ 2 root root  16K Dec  5 23:47 lost+found
```

## 6. Profit
When finished, you can unmount the directory (`umount /ext4fs`) and exit the container. Then on your host filesystem you'll have a shiny new raw ext4 image which you can then use other tools to convert to other image types, e.g. VirtualBox, VMWare, etc., using tools like [Packer](https://packer.io).

Note: this method does not mount external drives, as currently USB devices aren't supported in Docker for Mac. You may be able to mount partitions or other disk images, if you copy them into and mount another directory alongside your ext4 filesystem:
```
docker run --privileged -it -v $HOME/ext4fs:/ext4fs -v $HOME/otherdir:/otherdir ubuntu:18.04 bash
```
You can then use any tools available to Ubuntu to work on other disk images or partitions while inside the container, having full read-write access to your ext4 partition.

Note: your mounted filesystem will not be visible to the host OS, as it's mounted in a loopback inside the container's OS.
