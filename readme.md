this guide will allow you to update qidi max3 or plus3 to debian bookworm with the edge kernel, and the latest klipper, moonraker, and fluidd (or mainsail) without losing functionality of the screen. the easiest way is to write the qidi_max3_updated.img file to your emmc, then replace the printer.cfg file for your printer. if you have the max3 with the bltouch, you don't have to do anything else unless you want to install qidi's patch files, which, as far as i can tell, only allow you to see the thumbnails on the screen (which never really worked for me anyways)

1. write this image to your emmc: https://github.com/redrathnure/armbian-mkspi/releases/download/mkspi%2F0.3.4-24.2.0-trunk/Armbian-unofficial_24.2.0-trunk_Mkspi_bookworm_edge_6.7.5.img.xz
2. if you have the inductive probe on the max3, run `rm ~/klipper_config/config/printer.cfg ; wget https://raw.githubusercontent.com/billkenney/max3_plus3_recovery/main/printer-max3_probe.cfg ; mv printer-max3_probe.cfg /home/mks/klipper_config/config/printer.cfg`
3. if you have the plus3, run `rm ~/klipper_config/config/printer.cfg ; wget https://raw.githubusercontent.com/billkenney/max3_plus3_recovery/main/printer-plus3.cfg ; mv printer-plus3.cfg /home/mks/klipper_config/config/printer.cfg`
4. if you want to use qidi's patches, overwrite all of the files specified in issue 27 (https://github.com/QIDITECH/QIDI_PLUS3/issues/27#issuecomment-2073932891) with those in the klipper zip file (https://github.com/QIDITECH/QIDI_PLUS3/files/15087211/files_to_replace.zip), except for ~/klipper/klippy/extras/virtual_sdcard.py. overwrite the files specified here (https://github.com/QIDITECH/moonraker/issues/1#issuecomment-1985564638) with those in the moonraker zip file (https://github.com/QIDITECH/moonraker/files/14537786/moonraker_patch_2024_3_8.zip), except for ~/moonraker/moonraker/components/machine.py. overwrite virtual_sdcard.py and machine.py with the ones on this repository (the qidi files don't work, I had to comment or delete a few lines) `wget https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/virtual_sdcard.py ; mv virtual_sdcard.py ~/klipper/klippy/extras/virtual_sdcard.py ; wget https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/machine.py ; mv machine.py ~/moonraker/moonraker/components/machine.py` (press y when prompted to overwrite files).
5. if you are not using qidi's patch files, and have the max3 with the inductive probe or the plus3, run these commands: `sed -i 's/printer\.configfile\.settings\.bltouch/printer\.configfile\.settings\.probe/g' ~/home/mks/klipper_config/config/printer.cfg`
6. if you are using qidi's patch files, and have the max3 with the bltouch, run these commands: `sed -i 's/printer\.configfile\.settings\.bltouch\.x_offset/printer\.probe\["x_offset"\]/g;s/printer\.configfile\.settings\.bltouch\.y_offset/printer\.probe\["y_offset"\]/g' /home/mks/klipper_config/config/printer.cfg`
7. if you are using qidi's patch files, and have the max3 with the inductive probe or the plus3, run these commands: `sed -i 's/printer\.configfile\.settings\.bltouch\.x_offset/printer\.probe\["x_offset"\]/g;s/printer\.configfile\.settings\.bltouch\.y_offset/printer\.probe\["y_offset"\]/g' /home/mks/klipper_config/config/printer.cfg`
8. restart klipper (`sudo service klipper restart`) and you should be good to go
