sudo systemctl disable bluetooth.service
sudo systemctl disable hciuart.service
sudo systemctl disable wpa_supplicant.service
sudo systemctl disable alsa-state.service
sudo systemctl disable man-db.timer
sudo systemctl disable fstrim.service
sudo systemctl disable keyboard-setup.service
sudo systemctl disable ModemManager.service
sudo systemctl disable NetworkManager.service
sudo systemctl disable systemd-networkd-wait-online.service
sudo systemctl disable ssh.service
sudo systemctl disable dhcpcd.service # x
sudo systemctl disable networking.service # x
sudo systemctl --user disable pulseaudio.socket # x
sudo systemctl --user disable pulseaudio.service # x

sudo systemctl status bluetooth.service
sudo systemctl status hciuart.service
sudo systemctl status wpa_supplicant.service
sudo systemctl status alsa-state.service
sudo systemctl status man-db.timer
sudo systemctl status fstrim.service
sudo systemctl status keyboard-setup.service
sudo systemctl status ModemManager.service
sudo systemctl status NetworkManager.service
sudo systemctl status systemd-networkd-wait-online.service
sudo systemctl status ssh.service
sudo systemctl status dhcpcd.service # x
sudo systemctl status networking.service # x
sudo systemctl --user status pulseaudio.socket # x
sudo systemctl --user status pulseaudio.service # x






sudo systemctl disable bluetooth.service; sleep 1; sudo systemctl disable hciuart.service; sleep 1; sudo systemctl disable wpa_supplicant.service; sleep 1; sudo systemctl disable dhcpcd.service; sleep 1; sudo systemctl disable networking.service; sleep 1; sudo systemctl disable alsa-state.service; sleep 1; sudo systemctl --user disable pulseaudio.socket; sleep 1; sudo systemctl --user disable pulseaudio.service; sleep 1; sudo systemctl disable man-db.timer; sleep 1; sudo systemctl disable fstrim.service; sleep 1; sudo systemctl disable keyboard-setup.service; sleep 1; sudo systemctl disable ModemManager.service; sleep 1; sudo systemctl disable NetworkManager.service; sleep 1; sudo systemctl disable systemd-networkd-wait-online.service; sleep 1; sudo systemctl disable ssh.service; sleep 1;