this guide will allow you to update qidi max3 or plus3 to debian bookworm with the edge kernel, and the latest klipper, moonraker, and fluidd (or mainsail) without losing functionality of the screen

installing the firmware updates (which is necessary to get the screen to work) overwites some files in klipper/moonraker. so if you want to run the mainline software, you need to reinstall klipper/moonraker with kiauh. you need to use kiauh because it gives you the option to install with python3

qidi has released some patch files, which, as far as i can tell, only allow you to see the thumbnails on the screen (which never really worked for me anyways). other people have said the patch files cause problems, so i would recommend not installing them, i.e., skip step 20

1. write this image to your emmc: https://github.com/redrathnure/armbian-mkspi/releases/download/mkspi%2F0.3.4-24.2.0-trunk/Armbian-unofficial_24.2.0-trunk_Mkspi_bookworm_edge_6.7.5.img.xz

2. using a terminal client such as putty for windows, Terminal on macos or linux, or an app like shelly or terminus on your phone, ssh into your printer: `ssh mks@printer.ip.address` replacing 'printer.ip.address' with your printers ip address. User is "root" and password "1234". This is followed by the initial setup of the Armbian OS. Among other things, you have to give the user "root" a new password, select the time zone and your favorite shell. I use zsh. You will be asked to create a new user. Enter "mks" as the name and "makerbase" as the password. You can ignore prompts to enter a real name and confirm with Enter. then `exit` to terminate the ssh session, then ssh into the printer with user "mks" and password "makerbase". then `cd ~ ; wget https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/files_services.tgz ; tar -xzf files_services.tgz`

3. you can edit the sudoers file to allow you run sudo commands without a password prompt. run `sudo visudo`, then add `mks ALL=(ALL) NOPASSWD: ALL` as shown in the below screenshot, then `ctrl+x`, `y`, `enter` to save. this will speed things up significantly

![Screen Shot 2024-05-21 at 12 23 13 PM](https://github.com/billkenney/update_max3_plus3/assets/30010560/ab748b47-6701-46ed-ad33-a8aa9ad79321)

4. after rebooting, ssh into your printer and run: `sudo apt update ; sudo apt upgrade -y ; sudo apt install make gcc build-essential git python3-pip python3-full -y ; cd /home/mks ; git clone https://github.com/dw-0/kiauh.git ; /home/mks/kiauh/kiauh.sh`. Use kiauh to install klipper, moonraker, and fluidd (or mainsail) and crowsnest if you have a webcam, in that order (run `/home/mks/kiauh/kiauh.sh` again if you need to restart the script). when prompted make sure to select the option to install with python3, not python2

5. after the installations are done, run `sudo service klipper stop ; sudo service moonraker stop ; cd ~ ; mv ~/printer_data ~/klipper_config ; ln -s ~/klipper_config ~/printer_data ; mv ~/klipper_config/logs ~/klipper_logs ; ln -s ~/klipper_logs ~/klipper_config/logs ; ln -s ~/klipper_config/gcodes ~/gcode_files ; sudo service klipper start ; sudo service moonraker start`

6. to flash the mcu firmware, download this file, copy it to a micro sd card, plug it into the printer, and restart your printer: https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/X_4.bin

7. to flash the extruder mcu, you need to unplug all usb devices except for the extruder, then hold the bottom left button on the back of your extruder board (see the image) for like 2 minutes or until the screen loads up fully, then ssh into your printer and run `sudo mount /dev/sda1 /mnt ; sudo systemctl daemon-reload ; sudo mv ~/klipper.uf2 /mnt` then restart your printer. if you don't unplug all usb devices, i think the extruder is available at /dev/sdb1, you can check with sudo fdisk -l, in which case you could probably run `sudo mount /dev/sdb1 /mnt ; sudo systemctl daemon-reload ; sudo mv ~/klipper.uf2 /mnt` to flash the mcu. if you get an error that it's mounted read-only, the extruder has not been mounted properly. restart and try to boot into dfu mode again

![325058698-1a76832d-02ad-4cd7-aa7c-f63277600226](https://github.com/billkenney/update_max3_plus3/assets/30010560/46a879b1-d77c-468d-b7ab-371fcdcf8673)

8. to flash the rpi mcu, ssh into your printer and run `cd ~/klipper ; make clean ; make menuconfig` and configure as shown in the below image. Then press 'q' and select the option to save, then `sudo service klipper stop ; make flash ; sudo service klipper start`

![325061507-b820ced1-ac3a-4627-b366-04fd95770e5d](https://github.com/billkenney/update_max3_plus3/assets/30010560/de954ba9-a158-42d0-b564-d3a71169f4bc)

9. move libboost_system.so.1.67.0 and libwpa_client.so (which are used by xindi) to the appropriate folders: `sudo mv ~/libboost_system.so.1.67.0 /usr/lib/aarch64-linux-gnu ; sudo mv ~/libwpa_client.so /usr/lib`

10. move all of the service files to /lib/systemd/system, and enable them (you can ignore any errors): `sudo mv ~/*.service /lib/systemd/system ; sudo systemctl daemon-reload ; sudo systemctl enable makerbase-byid.service ; sudo systemctl enable makerbase-client.service ; sudo systemctl enable makerbase-net-mods.service ; sudo systemctl enable makerbase-automount@1.service ; sudo systemctl enable makerbase-wlan0.service ; sudo systemctl start makerbase-byid.service ; sudo systemctl start makerbase-client.service ; sudo systemctl start makerbase-net-mods.service ; sudo systemctl start makerbase-automount@1.service ; sudo systemctl start makerbase-wlan0.service`

11. run `sudo mv ~/klipper_config/MKS_THR.cfg ~/klipper_config/MKS_THR.cfg.bak ; path=$(ls /dev/serial/by-id/*) ; printf "[mcu MKS_THR]\nserial:$path\n" > ~/klipper_config/MKS_THR.cfg ; ln -s ~/klipper_config/MKS_THR.cfg ~/klipper_config/config/MKS_THR.cfg`

11. to install the latest firmware (it's not necessary to install the 800_480.tft if you've already updated your screen to the latest firmware, and again, ignore any errors), run `sudo ln -s /lib/modules/6.7.5-edge-rockchip64 /lib/modules/5.16.20-rockchip64 ; sudo ln -s /usr/lib/python3.11/configparser.py /usr/lib/python3.11/ConfigParser.py ; wget https://raw.githubusercontent.com/billkenney/qidi_3series_recovery/main/mksclient-plus3.deb ; wget https://raw.githubusercontent.com/billkenney/qidi_3series_recovery/main/800_480.tft ; sudo dpkg -i mksclient-plus3.deb ; sudo mv 800_480.tft /root ; wget https://raw.githubusercontent.com/billkenney/qidi_3series_recovery/main/printer-plus3.cfg ; mv printer-plus3.cfg /home/mks/klipper_config/config/printer.cfg ; printf "[include fluidd.cfg]\n" > ~/klipper_config/config/macros.cfg ; mv ~/klipper_config/Adaptive_Mesh.cfg ~/klipper_config/config/ ; sed -Ei 's/(^\[extruder\])/\[include macros.cfg\]\n\n\1/' ~/klipper_config/config/printer.cfg ; sed -i 's/\[virtual_sdcard\]/#\[virtual_sdcard\]/;s/path: ~\/gcode_files/#path: ~\/gcode_files/' ~/klipper_config/config/printer.cfg`. run `sudo ls -l /root and make sure the root folder has xindi and 800_480.tft`. it should look similar to the below screenshot (the 800_480.tft is renamed to 800_480.tft.bak after the screen is updated)

![Screen Shot 2024-05-21 at 12 45 51 PM](https://github.com/billkenney/update_max3_plus3/assets/30010560/45925ca0-fbb1-432f-952c-ab1e7268a6cb)

13. if your screen shows the firmware update has already started, leave it on until the update is complete. otherwise turn your printer off, wait for a bit, and turn it back on--the screen should be white with a progress indicator as shown below (it could take a few minutes to start, so be patient):

![IMG_2028](https://github.com/billkenney/update_max3_plus3/assets/30010560/f5cf29b5-9c42-475f-9e84-a78b302265bf)

14. to reinstall mainline versions of klipper/moonraker: `sudo service klipper stop ; sudo service moonraker stop ; sudo rm -rf ~/klipper ~/klippy-env ~/moonraker ~/moonraker-env ; sudo systemctl disable klipper.service ; sudo systemctl disable moonraker.service ; sudo rm /etc/systemd/system/klipper.service /etc/systemd/system/moonraker.service ; sudo systemctl daemon-reload ; ~/kiauh/kiauh.sh` then use kiauh to reinstall them, again select python3 when prompted

15. in your printer.cfg, you need to change max_accel_to_decel: 10000 (deprecated) to minimum_cruise_ratio: 0.5. this should do the trick: `sed -i 's/^max_accel_to_decel.*$/minimum_cruise_ratio: 0\.5/' ~/klipper_config/config/printer.cfg`

16. only do this step if you are skipping the install of qidi's patch files (step 20). run `sed -i 's/printer\.probe\["x_offset"\]/printer\.configfile\.settings\.probe\.x_offset/g;s/printer\.probe\["y_offset"\]/printer\.configfile\.settings\.probe\.y_offset/g' ~/klipper_config/config/printer.cfg ; sudo service klipper restart`

17. to get your wifi working again, run the following commands: `sudo ln -s /usr/src/linux-headers-6.7.5-edge-rockchip64/arch/arm64 /usr/src/linux-headers-6.7.5-edge-rockchip64/arch/aarch64 ; sudo apt install make gcc build-essential git network-manager ; cd /home/mks ; git clone https://github.com/lwfinger/rtl8xxxu ; cd /home/mks/rtl8xxxu ; make clean modules ; sudo make install ; sudo make install_fw`. Restart the printer, then: `cd /home/mks/rtl8xxxu ; sudo modprobe rtl8xxxu_git`. you can run `sudo nmtui` to use the network-manager service to create or manage network connections. if you get any errors, run `cd /home/mks/rtl8xxxu ; make clean modules ; sudo make install ; sudo make install_fw`, reboot, and run `cd /home/mks/rtl8xxxu ; sudo modprobe rtl8xxxu_git` again

18. if you have a webcam and installed crowsnest, run `vidpath=$( ls -la /dev/v4l/by-id/ | grep index0 | grep -o 'video[0-9]' ) ; sed -i "s/device: \/dev\/video[0-9]/device: \/dev\/$vidpath/;s/resolution: [0-9]*x[0-9]*/resolution: 1280x960/" ~/klipper_config/config/crowsnest.conf ; sudo service crowsnest restart`. to get your camera working in fluidd, you need to go into the settings menu and add the camera similar to the following settings

![Screen Shot 2024-05-21 at 2 05 32 PM](https://github.com/billkenney/update_max3_plus3/assets/30010560/0355ab05-e16e-4db3-ac47-b6ec409742c1)

19. if you want to run the resonance testing macro, you need to install these: `sudo apt install python3-numpy python3-matplotlib libatlas-base-dev libopenblas-dev ; ~/klippy-env/bin/pip install -v numpy`

######################################## the installation of these patch files is not necessary, and i recommended you skip these steps as they could cause problems. although they could get thumbnails working on the screen again ########################################

20. if you want to install qidi's patches, overwrite all of the files specified in issue 27 (https://github.com/QIDITECH/QIDI_PLUS3/issues/27#issuecomment-2073932891) with those in the klipper zip file (https://github.com/QIDITECH/QIDI_PLUS3/files/15087211/files_to_replace.zip), except for ~/klipper/klippy/extras/virtual_sdcard.py. overwrite the files specified here (https://github.com/QIDITECH/moonraker/issues/1#issuecomment-1985564638) with those in the moonraker zip file (https://github.com/QIDITECH/moonraker/files/14537786/moonraker_patch_2024_3_8.zip), except for ~/moonraker/moonraker/components/machine.py. overwrite virtual_sdcard.py and machine.py with the ones on this repository (the qidi files don't work, i had to comment or delete a few lines) `wget https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/virtual_sdcard.py ; mv virtual_sdcard.py ~/klipper/klippy/extras/virtual_sdcard.py ; wget https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/machine.py ; mv machine.py ~/moonraker/moonraker/components/machine.py` (press y when prompted to overwrite files).
