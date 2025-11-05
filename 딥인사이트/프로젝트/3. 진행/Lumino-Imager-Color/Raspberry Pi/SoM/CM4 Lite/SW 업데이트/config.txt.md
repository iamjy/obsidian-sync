$ sudo vim /boot/firmware/config.txt
```
[cm4]
# Enable host mode on the 2711 built-in XHCI USB controller.
# This line should be removed if the legacy DWC2 controller is required
# (e.g. for USB device mode) or if USB support is not required.
#otg_mode=1
dtoverlay=dwc2,dr_mode=peripheral
#dtoverlay=dwc2,dr_mode=host
# Eanble RTC
dtparam=i2c_vc=on
dtoverlay=i2c-rtc,pcf85063a,i2c_csi_dsi
#dtoverlay=i2c-rtc,pcf85063a,i2c0
# Enable USB 3.0 and VL805 Initialization
dtparam=pciex1=on
gpio=2=op,dh
# EEPROM FW UPDATE
#dtparam=spi=on
#dtoverlay=audremap
#dtoverlay=spi-gpio40-45
# Disable Unused Blocks
dtoverlay=disable-bt
dtoverlay=disable-wifi
dtoverlay=disable-ethernet
dtparam=audio=off
start_x=0
#gpu_mem=16
disable_splash=1
boot_delay=0
```