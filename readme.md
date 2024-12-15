################################################################################################################################################################

NOTE: i have added support for the smart3 based on a comment from qidi (https://github.com/QIDITECH/QIDI_MAX3/issues/53#issuecomment-2151230861) stating that the klipper mcu firmware files are the same--so you can use the ones from my repo, and the only difference is xindi (likely because the screen is a different size) and the printer.cfg file. you can follow the same process outlined below, the only difference is that you have to reinstall the smart3 firmware (see step 7), which will install the correct version of xindi for your printer. i am not aware of anyone updating the smart3 to the latest software yet, so this method is 100% untested and could brick your printer. probably a good idea to have a backup emmc (or backup image and emmc adapter) on hand if you can't get it to work

################################################################################################################################################################

this guide will allow you to update qidi max3 / plus3 / smart3 to debian bookworm with the edge kernel, and the latest klipper, moonraker, and fluidd (or mainsail) without losing functionality of the screen. just follow steps 1-10 (and optionally 11-13)

a qidi employee has also created an image. i installed it but ran into some pretty significant issues, and ended up reverting to my image. if you want to install his image, instructions are here: https://github.com/billkenney/update_max3_plus3/blob/main/cchen616-image.md

another community member has created custom screen firmware for the max3/plus3 (and a beta version for the smart3), which runs on completely unmodified software (so you never have to install or update qidi firmware). you can use his image, or you can install the screen firmware and use it with my image. im using it right now with my image and it works great (no more z offset issues and the thumbnail works). he also made some significant improvements to the structure of the printer.cfg file, which i have been using without issue. https://github.com/Phil1988/FreeDi

for my image, if you don't have ethernet, you should probably get an adapter before continuing (i have been told the smart3 has an ethernet port if you take the back panel off). the wifi menu on the printer screen does not work after upgrading (see https://github.com/billkenney/update_max3_plus3/issues/5). you can run `sudo nmtui` to use the network-manager service to create or manage network connections, and it will automatically connect on boot, but you need to be able to ssh into your printer. for now you need ethernet and will have to complete step 10 to get the wifi on the screen working again

also, as discussed in this issue (https://github.com/billkenney/update_max3_plus3/issues/6), qidi has apparently been using different wifi modules. if you have an aic8800 module, you may be able to follow these steos to get it working as the drivers apparently do not work on my image: https://forum.beagleboard.org/t/success-with-brostrend-usb-wifi-dongle-somewhere-to-document-the-process/37007/3

if you want to revert for some reason, you can follow these steps to revert to the stock image and flash the mcus with the old software: https://github.com/billkenney/update_max3_plus3/blob/main/revert.md

if you've done all of the steps and are getting a message that the system starts abnormally, its possible that you did not correctly flash the extruder mcu (https://github.com/billkenney/update_max3_plus3/issues/4). try running step 4 again

i added klippain-shaketune, as well as spoolman, which is using mysql (or mariadb) for the database, i had problems with the default psql database so i changed it. the setup for this release should be mostly the same as the previous release. the image itself is actually around 7.5gb (although it uses less than 6gb of space on the emmc), so i think you might need a 32GB emmc to install it

the default user is 'mks' and password (for ssh and sudo) is 'makerbase'

1. write this image to your emmc (you need a 32gb emmc, i havent figured out how to shrink the image small enough to fit on the 8gb emmc): https://github.com/billkenney/update_max3_plus3/releases/download/qidi_update_v2/qidi_update_v2.7z

using a terminal client such as putty for windows, Terminal on macos or linux, or an app like shelly or terminus on your phone, ssh into your printer: `ssh mks@printer.ip.address` replacing 'printer.ip.address' with your printers ip address. the username is 'mks' and the password is 'makerbase' any time you are prompted for a password

2. the language is set to English and the timezone is set to America/Chicago. to change this, run `sudo dpkg-reconfigure locales`. find your time zone here: https://en.m.wikipedia.org/wiki/List_of_tz_database_time_zones (it should be in the format America/Chicago), then run `sudo timedatectl set-timezone [your_timezone] ; sudo timedatectl set-ntp 1` replacing [your_timezone] with your actual timezone

3. to flash the mcu firmware, download this file, copy it to a micro sd card, plug it into the printer, and restart your printer: https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/X_4.bin

![325057677-0d15df45-8cd8-4e4c-88ec-a71b152b8cbb](https://github.com/billkenney/update_max3_plus3/assets/30010560/ce1f6465-d539-4137-80bd-90f31bab7661)

4. to flash the extruder mcu, you need to unplug all usb devices except for the extruder, then hold the BOOT bottom left button on the back of your extruder board (see the image) for like 2 minutes or until the screen loads up fully, then ssh into your printer and run `sudo mount /dev/sda1 /mnt`. on the newer extruder boards, the BOOT button is on the top right. you can check to see if it's properly mounted with `ls /mnt`, you shuld see `INDEX.HTM  INFO_UF2.TXT`. if you get an error that it's mounted read-only, the extruder has not been mounted properly. restart and try to boot into dfu mode again before continuing. if it mounts with no errors, run `sudo systemctl daemon-reload ; wget --no-check-certificate https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/klipper.uf2 ; sudo mv klipper.uf2 /mnt` then restart your printer

![325058698-1a76832d-02ad-4cd7-aa7c-f63277600226](https://github.com/billkenney/update_max3_plus3/assets/30010560/46a879b1-d77c-468d-b7ab-371fcdcf8673)

5. update klipper with kiauh to the latest version (`~/kiauh/kiauh.sh`) before flashing the rpi mcu, then run `cd ~/klipper ; make clean ; make menuconfig` and configure as shown in the below image. Then press 'q' and select the option to save, then `sudo service klipper stop ; make flash ; sudo service klipper start`

![325061507-b820ced1-ac3a-4627-b366-04fd95770e5d](https://github.com/billkenney/update_max3_plus3/assets/30010560/de954ba9-a158-42d0-b564-d3a71169f4bc)

6. ssh into your printer and run `sudo mv ~/printer_data/MKS_THR.cfg ~/printer_data/MKS_THR.cfg.bak ; ls /dev/serial/by-id/* > /tmp/path ; sed -Ei 's/^/\[mcu MKS_THR\]\nserial:/;s/([0-9][0-9])$/\1\n/' /tmp/path ; sudo mv /tmp/path ~/printer_data/MKS_THR.cfg ; rm -rf ~/printer_data/config/MKS_THR.cfg ; ln -s ~/printer_data/MKS_THR.cfg ~/printer_data/config/MKS_THR.cfg`

7. you can skip this step if you have the max3 with the bltouch

for the max3 with the inductive probe run: `wget --no-check-certificate https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/printer-max3_probe.cfg ; mv printer-max3_probe.cfg ~/klipper_config/config/printer.cfg`

for the plus3 run: `wget --no-check-certificate https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/printer-plus3.cfg ; mv printer-plus3.cfg ~/klipper_config/config/printer.cfg`

for the smart3 run: `wget https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/printer-smart3.cfg ; mv printer-smart3.cfg ~/klipper_config/config/printer.cfg ; wget --no-check-certificate https://raw.githubusercontent.com/billkenney/qidi_3series_recovery/main/mksclient-smart3.deb ; sudo dpkg -i mksclient-smart3.deb`

after installing the firmware for the smart3, you need to delete and reinstall klipper and moonraker. `cd /home/mks ; sudo service klipper stop ; sudo service moonraker stop ; sudo systemctl disable klipper.service ; sudo systemctl disable moonraker.service ; sudo rm -rf klipper klipper-env moonraker moonraker-env /etc/systemd/system/klipper.service /etc/systemd/system/moonraker.service ; sudo systemctl daemon-reload ; kiauh/kiauh.sh` (say y if prompted to update, then run the script again). 1 for the install menu, 1 for klipper, and skip the example printer.cfg. follow the same procedure to install moonraker, skip the sample config

reinstall klippain_shaketune: `~/klippain_shaketune/install.sh`

run these commands to make sure the services are still working properly: `sudo systemctl disable dhcpcd.service ; sudo systemctl enable dhcpcd@wlan0.service ; sudo systemctl disable wpa_supplicant.service ; sudo systemctl enable makerbase-wlan0.service`. for some reason my lan interface was renamed from eth0 to end1, so i couldn't see the ip address on the screen. run this to rename it to eth0: `macd=$( ip link show | grep 'ether' | head -n 1 | sed -E 's/^.*ether (.*) brd.*$/\1/' ) ; sudo printf "[Match]\nMACAddress=$macd\n[Link]\nName=eth0\n" > /tmp/macd ; sudo mv /tmp/macd /etc/systemd/network/10-eth0.link`

you may have to reinstall timelapse. `/home/mks/moonraker-timelapse/scripts/install.sh`

if you get an error about gcode_shell_command, reinstall it with kiauh, the option is under extensions i believe: `~/kiauh/kiauh.sh`

8. to install the screen firmware for the max3 or the plus3 (which is not necessary if you installed the latest screen firmware on your printer prior to updating), run `wget --no-check-certificate https://raw.githubusercontent.com/billkenney/qidi_3series_recovery/main/800_480.tft ; sudo mv 800_480.tft /root/800_480.tft`, restart your printer, you should see a white screen with a progress indicator similar to this image (it could take a few minutes to start, so be patient)

to install the screen firmware for the smart3 (which is not necessary if you installed the latest screen firmware on your printer prior to updating), run `wget --no-check-certificate https://raw.githubusercontent.com/billkenney/qidi_3series_recovery/main/800_480-smart3.tft ; sudo mv 800_480-smart3.tft /root/800_480.tft`, you should see a white screen with a progress indicator similar to this image (it could take a few minutes to start, so be patient)

![IMG_2028](https://github.com/billkenney/update_max3_plus3/assets/30010560/f5cf29b5-9c42-475f-9e84-a78b302265bf)

9. if you have a webcam, run `vidpath=$( ls -la /dev/v4l/by-id/ | grep index0 | grep -o 'video[0-9]' ) ; sed -i "s/device: \/dev\/video[0-9]/device: \/dev\/$vidpath/;s/resolution: [0-9]*x[0-9]*/resolution: 1280x960/" ~/klipper_config/config/crowsnest.conf ; sudo service crowsnest restart`. restart your printer and you should have the latest software and a working screen

10. run this to get wifi on the screen working again (and a couple other minor fixes). i'll integrate it into an updated image when i have the time: `sudo mv /etc/resolv.conf /etc/resolv.conf.bak ; sudo ln -s /run/systemd/resolve/stub-resolv.conf /etc/resolv.conf ; sudo printf 'ctrl_interface=/var/run/wpa_supplicant\nctrl_interface_group=0\nupdate_config=1\n\nnetwork={\n  ssid="network"\n  psk="password"\n  mesh_fwding=1\n}\n' > /tmp/wpa ; sudo mv /tmp/wpa /etc/wpa_supplicant/wpa_supplicant-wlan0.conf ; sudo chown root:root /root/.bash_history /root/.zsh_history /etc/wpa_supplicant/wpa_supplicant-wlan0.conf ; sudo sed -Ei 's/^After.*$/Before=dhcpcd@wlan0.service\nAfter=dbus.service/;s/^ExecStart=.*$/ExecStart=\/sbin\/wpa_supplicant -c\/etc\/wpa_supplicant\/wpa_supplicant-wlan0.conf -Dnl80211,wext -iwlan0 -u/;s/^Alias=.*$/WantedBy=network.target/' /lib/systemd/system/makerbase-wlan0.service ; sudo sed -Ei 's/^(Description.*$)/\1\nAfter=mariadb.service/' /etc/systemd/system/Spoolman.service ; sudo systemctl daemon-reload ; sudo apt install dhcpcd -y ; sudo systemctl disable dhcpcd.service ; sudo systemctl enable dhcpcd@wlan0.service ; sudo systemctl disable wpa_supplicant.service ; sudo systemctl enable makerbase-wlan0.service ; sudo reboot`

11. if you have the max3 and want to update the firmware to 4.3.15, follow these steps (you can ignore all of the errors). first `wget --no-check-certificate https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/libboost_filesystem.so.1.67.0 ; sudo mv libboost_filesystem.so.1.67.0 /usr/lib/aarch64-linux-gnu/ ; wget --no-check-certificate https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/mksclient-max3-4.3.15.deb ; sudo dpkg -i mksclient-max3-4.3.15.deb ; sudo rm mksclient-max3-4.3.15.deb ; ln -s /home/mks/printer_data /home/mks/klipper_config`

delete and reinstall klipper and moonraker with kiauh: `cd /home/mks ; sudo service klipper stop ; sudo service moonraker stop ; sudo systemctl disable klipper.service ; sudo systemctl disable moonraker.service ; sudo rm -rf klipper klipper-env moonraker moonraker-env /etc/systemd/system/klipper.service /etc/systemd/system/moonraker.service ; sudo systemctl daemon-reload ; kiauh/kiauh.sh` (say y if prompted to update, then run the script again). 1 for the install menu, 1 for klipper, and skip the example printer.cfg. follow the same procedure to install moonraker, skip the sample config

reinstall klippain_shaketune: `~/klippain_shaketune/install.sh`

run these commands to make sure the services are still working properly: `sudo systemctl disable dhcpcd.service ; sudo systemctl enable dhcpcd@wlan0.service ; sudo systemctl disable wpa_supplicant.service ; sudo systemctl enable makerbase-wlan0.service`. for some reason my lan interface was renamed from eth0 to end1, so i couldn't see the ip address on the screen. run this to rename it to eth0: `macd=$( ip link show | grep 'ether' | head -n 1 | sed -E 's/^.*ether (.*) brd.*$/\1/' ) ; sudo printf "[Match]\nMACAddress=$macd\n[Link]\nName=eth0\n" > /tmp/macd ; sudo mv /tmp/macd /etc/systemd/network/10-eth0.link`

install the screen firmware update: `wget --no-check-certificate https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/800_480_max3_4.3.15.tft ; sudo mv 800_480_max3_4.3.15.tft /root/800_480.tft` then turn your printer off and on again. the screen should go white for like 30 min with a progress indicator

if your webcam stops working after the update run step 9 again

if your date/time isnt working properly after the update, run `sudo apt install systemd-timesyncd ; sudo timedatectl set-ntp 1`

you may have to reinstall timelapse. `/home/mks/moonraker-timelapse/scripts/install.sh`

if you get an error about gcode_shell_command, reinstall it with kiauh, the option is under extensions i believe: `~/kiauh/kiauh.sh`

reboot your printer