$ sudo systemctl disable ( enable, status stop, start ) bluetooth.service
$ sudo systemctl disable ( enable, status stop, start ) hciuart.service
$ sudo systemctl disable ( enable, status stop, start ) wpa_supplicant.service
$ sudo systemctl disable ( enable, status stop, start ) alsa-state.service
$ sudo systemctl disable ( enable, status stop, start ) man-db.timer
$ sudo systemctl disable ( enable, status stop, start ) fstrim.service
$ sudo systemctl disable ( enable, status stop, start ) keyboard-setup.service
$ sudo systemctl disable ( enable, status stop, start ) ModemManager.service
$ sudo systemctl disable ( enable, status stop, start ) NetworkManager.service
$ sudo systemctl disable ( enable, status stop, start ) systemd-networkd-wait-online.service
$ sudo systemctl disable ( enable, status stop, start ) ssh.service
$ sudo systemctl disable ( enable, status stop, start ) dhcpcd.service # x
$ sudo systemctl disable ( enable, status stop, start ) networking.service # x
$ sudo systemctl --user ( enable, status stop, start ) disable pulseaudio.socket # x
$ sudo systemctl --user ( enable, status stop, start ) disable pulseaudio.service # x

$ sudo systemctl disable bluetooth.service; sleep 1; sudo systemctl disable hciuart.service; sleep 1; sudo systemctl disable wpa_supplicant.service; sleep 1; sudo systemctl disable dhcpcd.service; sleep 1; sudo systemctl disable networking.service; sleep 1; sudo systemctl disable alsa-state.service; sleep 1; sudo systemctl --user disable pulseaudio.socket; sleep 1; sudo systemctl --user disable pulseaudio.service; sleep 1; sudo systemctl disable man-db.timer; sleep 1; sudo systemctl disable fstrim.service; sleep 1; sudo systemctl disable keyboard-setup.service; sleep 1; sudo systemctl disable ModemManager.service; sleep 1; sudo systemctl disable NetworkManager.service; sleep 1; sudo systemctl disable systemd-networkd-wait-online.service; sleep 1; sudo systemctl disable ssh.service; sleep 1;