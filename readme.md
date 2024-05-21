this guide will allow you to update qidi max3 or plus3 to debian bookworm with the edge kernel, and the latest klipper, moonraker, and fluidd (or mainsail) without losing functionality of the screen. the easiest way is to write the qidi_update.img file to your emmc, then replace the printer.cfg file for your printer. if you have the max3 with the bltouch, you don't have to do much other than flashing the mcus unless you want to install qidi's patch files, which, as far as i can tell, only allow you to see the thumbnails on the screen (which never really worked for me anyways). default user is 'mks' and password (for ssh and root) is 'makerbase'

1. write this image to your emmc: https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/qidi_update.img.xz
2. to flash the mcu firmware, download this file, copy it to a micro sd card, plug it into the printer, and restart your printer: https://github.com/billkenney/update_max3_plus3/raw/main/X_4.bin
3. to flash the extruder mcu, hold the bottom left button on the back of your extruder board (see the image) for like 2 minutes or until the screen loads up fully, then ssh into your printer and run `sudo mount /dev/sda1 /mnt -w ; wget https://github.com/billkenney/update_max3_plus3/raw/main/klipper.uf2 ; mv klipper.uf2 /mnt` then restart your printer
![325058698-1a76832d-02ad-4cd7-aa7c-f63277600226](https://github.com/billkenney/update_max3_plus3/assets/30010560/46a879b1-d77c-468d-b7ab-371fcdcf8673)
4. to flash the rpi mcu, ssh into your printer and run `cd ~/klipper ; make clean ; make menuconfig` and configure as shown in the below image. Then press 'q' and select the option to save, then `sudo service klipper stop ; make flash ; sudo service klipper start`
![325061507-b820ced1-ac3a-4627-b366-04fd95770e5d](https://github.com/billkenney/update_max3_plus3/assets/30010560/de954ba9-a158-42d0-b564-d3a71169f4bc)
5. ssh into your printer and run `path=$(ls /dev/serial/by-id/*) ; printf "[mcu MKS_THR]\nserial:$path\n" > ~/klipper_config/MKS_THR.cfg ; ln -s ~/klipper_config/MKS_THR.cfg ~/klipper_config/config/MKS_THR.cfg`
6. if you have the inductive probe on the max3, run `rm ~/klipper_config/config/printer.cfg ; wget https://raw.githubusercontent.com/billkenney/max3_plus3_recovery/main/printer-max3_probe.cfg ; mv printer-max3_probe.cfg /home/mks/klipper_config/config/printer.cfg`
7. if you have the plus3, run `rm ~/klipper_config/config/printer.cfg ; wget https://raw.githubusercontent.com/billkenney/max3_plus3_recovery/main/printer-plus3.cfg ; mv printer-plus3.cfg /home/mks/klipper_config/config/printer.cfg`
8. if you want to use qidi's patches, overwrite all of the files specified in issue 27 (https://github.com/QIDITECH/QIDI_PLUS3/issues/27#issuecomment-2073932891) with those in the klipper zip file (https://github.com/QIDITECH/QIDI_PLUS3/files/15087211/files_to_replace.zip), except for ~/klipper/klippy/extras/virtual_sdcard.py. overwrite the files specified here (https://github.com/QIDITECH/moonraker/issues/1#issuecomment-1985564638) with those in the moonraker zip file (https://github.com/QIDITECH/moonraker/files/14537786/moonraker_patch_2024_3_8.zip), except for ~/moonraker/moonraker/components/machine.py. overwrite virtual_sdcard.py and machine.py with the ones on this repository (the qidi files don't work, I had to comment or delete a few lines) `wget https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/virtual_sdcard.py ; mv virtual_sdcard.py ~/klipper/klippy/extras/virtual_sdcard.py ; wget https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/machine.py ; mv machine.py ~/moonraker/moonraker/components/machine.py` (press y when prompted to overwrite files).
9. if you are not using qidi's patch files, and have the max3 with the inductive probe or the plus3, run these commands: `sed -i 's/printer\.configfile\.settings\.bltouch/printer\.configfile\.settings\.probe/g' ~/home/mks/klipper_config/config/printer.cfg`
10. if you are using qidi's patch files, run these commands: `sed -i 's/printer\.configfile\.settings\.bltouch\.x_offset/printer\.probe\["x_offset"\]/g;s/printer\.configfile\.settings\.bltouch\.y_offset/printer\.probe\["y_offset"\]/g' /home/mks/klipper_config/config/printer.cfg`
13. finally, to install the screen firmware, run `sudo mv /root/800_480.tft.bak /root/800_480.tft`, restart your printer, you should see a white screen with a progress indicator similar to this image: ![IMG_2028](https://github.com/billkenney/update_max3_plus3/assets/30010560/f5cf29b5-9c42-475f-9e84-a78b302265bf)
