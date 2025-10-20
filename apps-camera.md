$ ls /dev/video*
$ sudo journalctl -u camera-service -f
$ rm -rf /mnt/usb_storage/*
$ sudo mkfs.ntfs -f -L "REC_USB" -F /piusb.bin
$ sudo vim /etc/fstab
	/piusb.bin /mnt/usb_storage ntfs defaults,uid=1000,gid=1000,umask=022 0 0
$ echo DIS_LIC_001_v1.0.2 > ~/README.md
$ mkdir tmp; mv lic-apps/ venv/ scripts/ launch.sh __pycache__/ tmp/
$ mkdir etc; mv * etc/; mv etc/tmp .; mv tmp/* .; rm -rf tmp/
$ mv scripts/ .scripts/
$ mv etc/ .etc
$ cd lic-apps/
$ tar xf apps-camera-20251020.tar.gz; rm apps-camera-20251020.tar.gz

$ sudo reboot
$ systemd-analyze
$ sudo journalctl -u camera-service -f
