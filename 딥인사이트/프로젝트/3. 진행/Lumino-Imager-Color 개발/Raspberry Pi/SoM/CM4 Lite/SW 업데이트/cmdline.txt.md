sudo vim /boot/firmware/cmdline.txt
```
console=serial0,115200 console=tty1 root=PARTUUID=caf16442-02 rootfstype=ext4 fsck.mode=skip rootwait quiet loglevel=3 plymouth.enable=0

fsck.repair=yes rootwait quiet loglevel=3 plymouth.enable=0
or
**fsck.mode=skip rootwait quiet loglevel=3 plymouth.enable=0**
```
