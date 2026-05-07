$ ls /dev/video*
$ sudo journalctl -u camera-service -f
$ rm -rf /mnt/usb_storage/*
$ sudo mkfs.ntfs -f -L "REC_USB" -F /piusb.bin
$ sudo vim /etc/fstab
```
/piusb.bin /mnt/usb_storage ntfs defaults,uid=1000,gid=1000,umask=022 0 0
```
$ echo DIS_LIC_001_v1.0.2 > ~/README.md
$ mkdir tmp; mv lic-apps/ venv/ scripts/ launch.sh __pycache__/ tmp/
$ mkdir etc; mv * etc/; mv etc/tmp .; mv tmp/* .; rm -rf tmp/
$ mv scripts/ .scripts/
$ mv etc/ .etc
$ cd lic-apps/
$ cd apps-camera/
$ sudo ./install_usb_otg_controller.sh
```
Do you want to reboot now? (y/n) y
Rebooting in 3 seconds...
```
	
$ systemd-analyze
$ systemd-analyze blame

$ sudo timedatectl set-ntp false
$ sudo systemctl disable systemd-timesyncd
$ sudo systemctl disable fake-hwclock.service
$ cd ./.scripts
$ sudo ./install_usb_otg_controller.sh
$ sudo ./install_gpio_monitor.sh

sudo journalctl -u camera-service.service -f
sudo journalctl -u usb-storage-controller.service -f
sudo journalctl -u gpio-monitor.service -f