this are the steps to follow if you want to manually install the updated os and software. 

note that it is not necessary to use the patch files provided by qidi (step 11), but installing the firmware updates (which is necessary to get the screen to work) overwites some files in klipper/moonraker. so if you want to run the mainline software, you need to reinstall klipper/moonraker with kiauh after step 12. you need to use kiauh because it gives you the option to install with python3

1. write this image to your emmc: https://github.com/redrathnure/armbian-mkspi/releases/download/mkspi%2F0.3.4-24.2.0-trunk/Armbian-unofficial_24.2.0-trunk_Mkspi_bookworm_edge_6.7.5.img.xz
2. Connect to the printer via SSH. User is "root" and password "1234". This is followed by the initial setup of the Armbian OS. Among other things, you have to give the user "root" a new password, select the time zone and your favorite shell. I use zsh. You will be asked to create a new user. Enter "mks" as the name and "makerbase" as the password. You can ignore prompts to enter a real name and confirm with Enter. then `exit` to terminate the ssh session, then ssh into the printer with user "mks" and password "makerbase".
3. after booting up the printer, ssh into your printer and run: `sudo apt update ; sudo apt upgrade -y ; sudo apt install make gcc build-essential git python3-pip python3-full -y ; cd /home/mks ; git clone https://github.com/dw-0/kiauh.git ; /home/mks/kiauh/kiauh.sh`. Use kiauh to install klipper, moonraker, and fluidd (or mainsail) and crowsnest if you have a webcam, in that order (run `/home/mks/kiauh/kiauh.sh` again if you need to restart the script). when prompted make sure to select the option to install with python3, not python2. then `cd ~ ; mv ~/printer_data ~/klipper_config ; ln -s ~/klipper_config ~/printer_data ; mv ~/klipper_config/logs ~/klipper_logs ; ln -s ~/klipper_logs ~/klipper_config/logs ; mv ~/klipper_config/gcodes ~/gcode_files ; ln -s ~/gcode_files ~/klipper_config/gcodes`. if you're using a webcam, run `sed -i 's/device: \/dev\/video0/device: \/dev\/video4/;s/resolution: 640x480/resolution: 1280x960/' ~/klipper_config/config/crowsnest.conf`
4. to flash the mcu firmware, download this file, copy it to a micro sd card, plug it into the printer, and restart your printer: https://github.com/billkenney/update_max3_plus3/raw/main/X_4.bin
5. to flash the extruder mcu, hold the bottom left button on the back of your extruder board (see the image) for like 2 minutes or until the screen loads up fully, then ssh into your printer and run `sudo mount /dev/sda1 /mnt -w ; wget https://github.com/billkenney/update_max3_plus3/raw/main/klipper.uf2 ; mv klipper.uf2 /mnt` then restart your printer
![325058698-1a76832d-02ad-4cd7-aa7c-f63277600226](https://github.com/billkenney/update_max3_plus3/assets/30010560/46a879b1-d77c-468d-b7ab-371fcdcf8673)
6. to flash the rpi mcu, ssh into your printer and run `cd ~/klipper ; make clean ; make menuconfig` and configure as shown in the below image. Then press 'q' and select the option to save, then `sudo service klipper stop ; make flash ; sudo service klipper start`
![325061507-b820ced1-ac3a-4627-b366-04fd95770e5d](https://github.com/billkenney/update_max3_plus3/assets/30010560/de954ba9-a158-42d0-b564-d3a71169f4bc)
7. download libboost_system.so.1.67.0 and libwpa_client.so from this repo and move them to the appropriate folders: `wget https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/libboost_system.so.1.67.0 ; wget https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/libwpa_client.so ; sudo mv libboost_system.so.1.67.0 /usr/lib/aarch64-linux-gnu ; sudo mv libwpa_client.so /usr/lib`
8. download all of the service files from this repo, move them to /lib/systemd/system, and enable them (you can ignore any errors): `wget https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/makerbase-byid.service ; 
wget https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/makerbase-client.service ; wget https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/makerbase-net-mods.service ; sudo mv *.service /lib/systemd/system ; sudo systemctl daemon-reload ; sudo systemctl enable makerbase-byid.service ; sudo systemctl enable makerbase-client.service ; sudo systemctl enable makerbase-net-mods.service ; sudo systemctl start makerbase-byid.service ; sudo systemctl start makerbase-client.service ; sudo systemctl start makerbase-net-mods.service`
9. to install the latest firmware (again, ignore any errors) for the max3, run `wget https://raw.githubusercontent.com/billkenney/max3_plus3_recovery/main/mksclient-max3.deb ; wget https://raw.githubusercontent.com/billkenney/max3_plus3_recovery/main/800_480.tft ; sudo dpkg -i mksclient-max3.deb ; sudo mv 800_480.tft /root`. if you have the bltouch, run `rm ~/klipper_config/config/printer.cfg ; wget https://raw.githubusercontent.com/billkenney/max3_plus3_recovery/main/printer-max3_bltouch.cfg ; mkdir -p /home/mks/klipper_config ; mv printer-max3_bltouch.cfg /home/mks/klipper_config/printer.cfg ; ln -s ~/klipper_config/printer.cfg ~/klipper_config/config/printer.cfg`. if you have the inductive probe, run `rm ~/klipper_config/config/printer.cfg ; wget https://raw.githubusercontent.com/billkenney/max3_plus3_recovery/main/printer-max3_probe.cfg ; mkdir -p /home/mks/klipper_config ; mv printer-max3_probe.cfg /home/mks/klipper_config ; ln -s ~/klipper_config/printer.cfg ~/klipper_config/config/printer.cfg` to install the latest firmware for the plus3, run `wget https://raw.githubusercontent.com/billkenney/max3_plus3_recovery/main/mksclient-plus3.deb ; wget https://raw.githubusercontent.com/billkenney/max3_plus3_recovery/main/800_480.tft ; sudo dpkg -i mksclient-plus3.deb ; sudo mv 800_480.tft /root ; rm ~/klipper_config/config/printer.cfg ; wget https://raw.githubusercontent.com/billkenney/max3_plus3_recovery/main/printer-plus3.cfg ; mkdir -p /home/mks/klipper_config ; mv printer-plus3.cfg /home/mks/klipper_config/printer.cfg ; ln -s ~/klipper_config/printer.cfg ~/klipper_config/config/printer.cfg`. to include all of the macros in the printer.cfg file, run `mv ~/klipper_config/Adaptive_Mesh.cfg ~/klipper_config/config/ ; sed -Ei 's/(^\[extruder\])/\[include \/home\/mks\/klipper_config\/config\/*.cfg\]\n\[include \/home\/mks\/klipper_config\/config\/*.conf\]\n\n\1/' ~/klipper_config/printer.cfg`
10. run `sudo ls -l /root` and make sure the root folder has xindi and 800_480.tft. it should look similar to the below screenshot (the 800_480.tft is renamed to 800_480.tft.bak after the screen is updated). ![Screen Shot 2024-05-20 at 4 08 23 PM](https://github.com/billkenney/update_max3_plus3/assets/30010560/f67a8169-6591-4d3c-b43e-221a1dca0a4c)
11. if you do not want to use qidi's patches, skip this step. if you do want to use qidi's patches, overwrite all of the files specified in issue 27 (https://github.com/QIDITECH/QIDI_PLUS3/issues/27#issuecomment-2073932891) with those in the klipper zip file (https://github.com/QIDITECH/QIDI_PLUS3/files/15087211/files_to_replace.zip), except for ~/klipper/klippy/extras/virtual_sdcard.py. overwrite the files specified here (https://github.com/QIDITECH/moonraker/issues/1#issuecomment-1985564638) with those in the moonraker zip file (https://github.com/QIDITECH/moonraker/files/14537786/moonraker_patch_2024_3_8.zip), except for ~/moonraker/moonraker/components/machine.py. overwrite virtual_sdcard.py and machine.py with the ones on this repository (the qidi files don't work, I had to comment or delete a few lines) `wget https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/virtual_sdcard.py ; mv virtual_sdcard.py ~/klipper/klippy/extras/virtual_sdcard.py ; wget https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/machine.py ; mv machine.py ~/moonraker/moonraker/components/machine.py` (press y when prompted to overwrite files).
12. if your screen shows the firmware update has already started, i guess leave it on until the update is complete. otherwise turn your printer off, wait for a bit, and turn it back on--the screen should be white with a progress indicator as shown below: ![IMG_2028](https://github.com/billkenney/update_max3_plus3/assets/30010560/f5cf29b5-9c42-475f-9e84-a78b302265bf)
13. in your printer.cfg, you need to change max_accel_to_decel: 10000 (deprecated) to minimum_cruise_ratio: 0.5. this should do the trick: `sed -i 's/^max_accel_to_decel.*$/minimum_cruise_ratio: 0\.5/' /home/mks/klipper_config/printer.cfg`
14. to get your wifi working again, run the following commands: `sudo ln -s /usr/src/linux-headers-6.7.5-edge-rockchip64/arch/arm64 /usr/src/linux-headers-6.7.5-edge-rockchip64/arch/aarch64 ; sudo apt install make gcc build-essential git ; cd /home/mks ; git clone https://github.com/lwfinger/rtl8xxxu ; cd /home/mks/rtl8xxxu ; make clean modules ; sudo make install ; sudo make install_fw`. Restart the printer, then: `cd /home/mks/rtl8xxxu ; sudo modprobe rtl8xxxu_git`
15. only do this step if you are not using the qidi patch files and have reinstalled clean versions of klipper/moonraker. if you have the inductive probe, in the printer.cfg, you need to change every instance of printer.probe["x_offset"] to printer.configfile.settings.probe.x_offset and every instance of printer.probe["y_offset"] to printer.configfile.settings.probe.y_offset. If you have the bltouch, replace probe with bltouch. run these commands for the probe: `sed -i 's/printer\.probe\["x_offset"\]/printer\.configfile\.settings\.probe\.x_offset/g;s/printer\.probe\["y_offset"\]/printer\.configfile\.settings\.probe\.y_offset/g' /home/mks/klipper_config/printer.cfg`. and these for the bltouch: `sed -i 's/printer\.probe\["x_offset"\]/printer\.configfile\.settings\.bltouch\.x_offset/g;s/printer\.probe\["y_offset"\]/printer\.configfile\.settings\.bltouch\.y_offset/g' /home/mks/klipper_config/printer.cfg`. restart klipper (`sudo service klipper restart`) and you should be good to go