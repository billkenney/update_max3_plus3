this guide will allow you to update qidi max3 or plus3 to debian bookworm with the edge kernel, and the latest klipper, moonraker, and fluidd (or mainsail) without losing functionality of the screen

note that it is not necessary to use the patch files provided by qidi (steps 3 & 6), but installing the firmware updates (which is necessary to get the screen to work) overwites some files in klipper/moonraker. so if you want to run the mainline software, you need to reinstall klipper/moonraker with kiauh after step 11. you need to use kiauh because it gives you the option to install with python3

1. write this image to your emmc: https://github.com/redrathnure/armbian-mkspi/releases/download/mkspi%2F0.3.4-24.2.0-trunk/Armbian-unofficial_24.2.0-trunk_Mkspi_bookworm_edge_6.7.5.img.xz
2. after booting up the printer, login and run: `sudo apt update ; sudo apt upgrade -y ; sudo apt install make gcc build-essential git -y ; cd /home/mks ; git clone https://github.com/dw-0/kiauh.git ; /home/mks/kiauh/kiauh.sh`. Once kiauh is installed, use it to install klipper, moonraker, and fluidd (or mainsail), in that order (run `/home/mks/kiauh/kiauh.sh` again to start the script). when prompted make sure to select the option to install with python3, not python2
3. follow the instructions here, but don't overwrite the klipper files with the qidi patch files: https://github.com/QIDITECH/QIDI_PLUS3/issues/27#issuecomment-2073932891
4. when it comes time to flash the extruder, it's not going to appear in the gcode folder. you need to run: `sudo mount /dev/sda1 /mnt ; cp /home/mks/klipper/out/klipper.uf2 /mnt`, then restart the printer
5. follow the instructions here to install the latest firmware (this overwrites some of the klipper/moonraker files): https://github.com/billkenney/max3_plus3_recovery
6. if you do not want to use qidi's patches, skip this step. if you do want to use qidi's patches, overwrite all of the files specified in issue 27 with those in the klipper zip file (step 3), except for ~/klipper/klippy/extras/virtual_sdcard.py, and the files specified here with those in the moonraker zip file: https://github.com/QIDITECH/moonraker/issues/1#issuecomment-1985564638, except for ~/moonraker/moonraker/components/machine.py. overwrite virtual_sdcard.py and machine.py with the ones on this repository (the qidi files don't work, I had to comment or delete a few lines) `wget https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/virtual_sdcard.py ; mv virtual_sdcard.py ~/klipper/klippy/extras/virtual_sdcard.py ; wget https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/machine.py ; mv machine.py ~/moonraker/moonraker/components/machine.py` (press y when prompted to overwrite files)
7. download libboost_system.so.1.67.0 and libwpa_client.so from this repo and move them to the appropriate folders: `wget https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/libboost_system.so.1.67.0 ; wget https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/libwpa_client.so ; sudo mv libboost_system.so.1.67.0 /usr/lib/aarch64-linux-gnu ; sudo mv libwpa_client.so /usr/lib`
8. download all of the service files from this repo, move them to /lib/systemd/system, and enable them: `wget https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/makerbase-byid.service ; wget https://github.com/billkenney/update_max3_plus3/blob/main/makerbase-client.service ; wget https://github.com/billkenney/update_max3_plus3/blob/main/makerbase-net-mods.service ; sudo mv *.service /lib/systemd/system ; sudo systemctl daemon-reload ; sudo systemctl enable makerbase-byid.service ; sudo systemctl enable makerbase-client.service ; sudo systemctl enable makerbase-net-mods.service ; sudo systemctl start makerbase-byid.service ; sudo systemctl start makerbase-client.service ; sudo systemctl start makerbase-net-mods.service`
9. if your screen shows the firmware update has already started, i guess leave it on until the update is complete. otherwise turn your printer off, wait for a bit, and turn it back on--the screen should be white with a progress indicator
10. in your printer.cfg, you need to change max_accel_to_decel: 10000 (deprecated) to minimum_cruise_ratio: 0.5. this should do the trick: `sed -i 's/^max_accel_to_decel.*$/minimum_cruise_ratio: 0\.5/' /home/mks/klipper_config/printer.cfg`
11. to get your wifi working again, run the following commands: `sudo ln -s /usr/src/linux-headers-6.7.5-edge-rockchip64/arch/arm64 /usr/src/linux-headers-6.7.5-edge-rockchip64/arch/aarch64 ; sudo apt install make gcc build-essential git ; cd /home/mks ; git clone https://github.com/lwfinger/rtl8xxxu ; cd /home/mks/rtl8xxxu ; make clean modules ; sudo make install ; sudo make install_fw`. Restart the printer, then: `cd /home/mks/rtl8xxxu ; sudo modprobe rtl8xxxu_git`
12. only do this step if you are not using the qidi patch files and have reinstalled clean versions of klipper/moonraker. if you have the inductive probe, in the printer.cfg, you need to change every instance of printer.probe["x_offset"] to printer.configfile.settings.probe.x_offset and every instance of printer.probe["y_offset"] to printer.configfile.settings.probe.y_offset. If you have the bltouch, replace probe with bltouch. run these commands for the probe: `sed -i 's/printer\.probe\["x_offset"\]/printer\.configfile\.settings\.probe\.x_offset/g;s/printer\.probe\["y_offset"\]/printer\.configfile\.settings\.probe\.y_offset/g' /home/mks/klipper_config/printer.cfg`. and these for the bltouch: `sed -i 's/printer\.probe\["x_offset"\]/printer\.configfile\.settings\.bltouch\.x_offset/g;s/printer\.probe\["y_offset"\]/printer\.configfile\.settings\.bltouch\.y_offset/g' /home/mks/klipper_config/printer.cfg`. restart klipper (`sudo service klipper restart`) and you should be good to go
