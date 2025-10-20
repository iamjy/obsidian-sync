sudo vim /boot/firmware/config.txt
# Disable Unused Blocks
dtoverlay=disable-bt
dtoverlay=disable-wifi
dtoverlay=disable-ethernet
dtparam=audio=off
start_x=0
#gpu_mem=16
disable_splash=1
boot_delay=0