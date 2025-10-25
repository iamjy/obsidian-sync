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
$ tar xf apps-camera.tar.gz; rm apps-camera.tar.gz
$ cd apps-camera/
$ ./install_usb_otg_controller.sh
	Do you want to reboot now? (y/n) y
	Rebooting in 3 seconds...
$ systemd-analyze
$ sudo journalctl -u camera-service -f






sync; cd .scripts/; sudo ./install_usb_otg_controller.sh 

sync
sudo reboot
cat launch.sh









sudo timedatectl set-ntp false
sudo systemctl disable systemd-timesyncd
sudo systemctl disable fake-hwclock.service

cd ./.scripts
sudo ./install_usb_otg_controller.sh
sudo ./install_gpio_monitor.sh

sudo reboot

sudo journalctl -u camera-service.service -f
sudo journalctl -u usb-storage-controller.service -f
sudo journalctl -u gpio-monitor.service -f

시료 (시간 동기화 적용)
2(겉면) - 완료 (fx3 reset, 스냅샷 안정화)
4(겉면) - 완료 (fx3 reset, 스냅샷 안정화)
1(겉면) - 완료 (fx3 reset, 스냅샷 안정화)
6(겉면) - 완료 (fx3 reset, 스냅샷 안정화)
10(겉면) - 완료 (fx3 reset, 스냅샷 안정화)
7(겉면) - 완료 (fx3 reset, 스냅샷 안정화)
13(겉면) -  (fx3 reset, 스냅샷 안정화)
14(겉면) -  (fx3 reset, 스냅샷 안정화)







**생산품질팀 테스트 시 필수 준수사항**
생산품질팀에서는 테스트 진행 시 다음 사항을 반드시 확인하시기 바랍니다.

**1. 시간 동기화**
OTG 케이블 연결 해제 후 3초 경과 시점에 시간이 반영됩니다.

**2. 녹화 기능 테스트**
녹화 버튼은 화면의 빨간색 표시등이 켜지거나 꺼질 때까지 계속 눌러주셔야 합니다.

**주의사항:**
1. 녹화 버튼은 OSD 화면의 초 단위 시간을 확인하며, 버튼을 누른 시점을 기준으로 **최소 4초 이상** 유지되어야 합니다. (정확히 3.5초 또는 3초에 맞추기 어려운 시스템 특성상 여유를 두고 진행)
2. **녹화 시작**: 버튼을 누른 상태에서 3.5초 경과 시 빨간색 표시등이 점등되며 녹화가 시작됩니다.
3. **녹화 중지**: 버튼을 누른 상태에서 3초 경과 후 녹화 중지 시퀀스가 시작되며, 2~3초 후 녹화가 완전히 중지되고 빨간색 표시등이 소등됩니다. 
4. **녹화 시작 직후 중지 불가**: 녹화 시작과 동시에 중지 버튼을 누를 경우 중지가 처리되지 않습니다. 녹화 시작 후 버튼에서 손을 뗀 시점을 기준으로 **2~3초 이후**에 중지 버튼을 눌러주시기 바랍니다.
5. **최소 녹화 시간 제한**: 현재 녹화 중지 로직이 최적화 단계에 있어, **6초 미만의 영상은 녹화 파일이 생성되지 않습니다.** 간헐적으로 5초 녹화 파일이 생성될 수 있으나, 해당 파일은 재생되지 않으니 참고하시기 바랍니다.