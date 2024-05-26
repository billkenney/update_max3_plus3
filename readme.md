this guide will allow you to update qidi max3 or plus3 to debian bookworm with the edge kernel, and the latest klipper, moonraker, and fluidd (or mainsail) without losing functionality of the screen. just follow steps 1-10 (and optionally 11-12). if you're unable to add wifi connections on the screen, you can run `sudo nmtui` to use the network-manager service to create or manage network connections. the default user is 'mks' and password (for ssh and sudo) is 'makerbase'

if you want to try the much more complicated manual install method, you can follow the manual steps: https://github.com/billkenney/update_max3_plus3/blob/main/manual.md

if you want to revert for some reason, you can follow these steps to revert to the stock image and flash the mcus with the old software: https://github.com/billkenney/update_max3_plus3/blob/main/revert.md

qidi has released some patch files, which, as far as i can tell, only allow you to see the thumbnails on the screen (which never really worked for me anyways). other people have said the patch files cause problems, so i would recommend skipping steps 11-12

1. write this image to your emmc: https://github.com/billkenney/update_max3_plus3/releases/download/qidi_update/qidi_update.img.7z

2. if youre using a 32gb emmc, run `sudo systemctl enable armbian-resize-filesystem ; sudo reboot`. the language is set to English and the timezone is set to America/Chicago. to change this, run `sudo dpkg-reconfigure locales`

3. to flash the mcu firmware, download this file, copy it to a micro sd card, plug it into the printer, and restart your printer: https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/X_4.bin

4. to flash the extruder mcu, you need to unplug all usb devices except for the extruder, then hold the bottom left button on the back of your extruder board (see the image) for like 2 minutes or until the screen loads up fully, then ssh into your printer and run `sudo mount /dev/sda1 /mnt ; sudo systemctl daemon-reload ; wget https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/klipper.uf2 ; mv klipper.uf2 /mnt` then restart your printer. if you don't unplug all usb devices, i think the extruder is available at /dev/sdb1, you can check with `sudo fdisk -l`, in which case you could probably run `sudo mount /dev/sdb1 /mnt ; sudo systemctl daemon-reload ; wget https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/klipper.uf2 ; mv klipper.uf2 /mnt` to flash the mcu. if you get an error that it's mounted read-only, the extruder has not been mounted properly. restart and try to boot into dfu mode again

![325058698-1a76832d-02ad-4cd7-aa7c-f63277600226](https://github.com/billkenney/update_max3_plus3/assets/30010560/46a879b1-d77c-468d-b7ab-371fcdcf8673)

5. to flash the rpi mcu, ssh into your printer and run `cd ~/klipper ; make clean ; make menuconfig` and configure as shown in the below image. Then press 'q' and select the option to save, then `sudo service klipper stop ; make flash ; sudo service klipper start`

![325061507-b820ced1-ac3a-4627-b366-04fd95770e5d](https://github.com/billkenney/update_max3_plus3/assets/30010560/de954ba9-a158-42d0-b564-d3a71169f4bc)

6. ssh into your printer and run `path=$(ls /dev/serial/by-id/*) ; printf "[mcu MKS_THR]\nserial:$path\n" > ~/klipper_config/MKS_THR.cfg ; rm ~/klipper_config/config/MKS_THR.cfg ; ln -s ~/klipper_config/MKS_THR.cfg ~/klipper_config/config/MKS_THR.cfg`

7. if you are skipping the installation of qidi's patch files (which i recommend), for the max3 with the inductive probe run: `wget https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/printer-max3_probe.cfg ; mv printer-max3_probe.cfg ~/klipper_config/config/printer.cfg`. for the plus3 run: `wget https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/printer-plus3.cfg ; mv printer-plus3.cfg ~/klipper_config/config/printer.cfg`. you can skip this step if you have the bltouch

8. to install the screen firmware (which is not necessary if you've already installed the latest firmware on your printer), run `sudo mv /root/800_480.tft.bak /root/800_480.tft`, restart your printer, you should see a white screen with a progress indicator similar to this image:

![IMG_2028](https://github.com/billkenney/update_max3_plus3/assets/30010560/f5cf29b5-9c42-475f-9e84-a78b302265bf)

9. if you have a webcam, run `vidpath=$( ls -la /dev/v4l/by-id/ | grep index0 | grep -o 'video[0-9]' ) ; sed -i "s/device: \/dev\/video[0-9]/device: \/dev\/$vidpath/;s/resolution: [0-9]*x[0-9]*/resolution: 1280x960/" ~/klipper_config/config/crowsnest.conf ; sudo service crowsnest restart`. if you do not have a webcam, run `cd ~ ; sudo service crowsnest stop ; sudo rm -rf crowsnest ; rm ~/klipper_config/config/crowsnest.conf ; sudo systemctl disable crowsnest.service ; sudo rm /etc/systemd/system/crowsnest.service ; sudo systemctl daemon-reload`. you may also have to delete the camera in fluidd. restart your printer and you should have the latest software and a working screen. 

10. if you want to run the resonance testing macro, you need to install these: `sudo apt install python3-numpy python3-matplotlib libatlas-base-dev libopenblas-dev ; ~/klippy-env/bin/pip install -v numpy`

########################################
the installation of these patch files is not necessary, and i recommended you skip these steps as they could cause problems. although they could get thumbnails working on the screen again
########################################

11. if you are installing qidi's patch files, for the max3 with the bltouch run: `wget https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/printer-max3_bltouch_patch.cfg ; mv printer-max3_bltouch_patch.cfg ~/klipper_config/config/printer.cfg`. for the max3 with the inductive probe run: `wget https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/printer-max3_probe_patch.cfg ; mv printer-max3_probe_patch.cfg ~/klipper_config/config/printer.cfg`. for the plus3 run: `wget https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/printer-plus3_patch.cfg ; mv printer-plus3_patch.cfg ~/klipper_config/config/printer.cfg`

12. to install qidi's patch files, overwrite all of the files specified in issue 27 (https://github.com/QIDITECH/QIDI_PLUS3/issues/27#issuecomment-2073932891) with those in the klipper zip file (https://github.com/QIDITECH/QIDI_PLUS3/files/15087211/files_to_replace.zip), except for ~/klipper/klippy/extras/virtual_sdcard.py. overwrite the files specified here (https://github.com/QIDITECH/moonraker/issues/1#issuecomment-1985564638) with those in the moonraker zip file (https://github.com/QIDITECH/moonraker/files/14537786/moonraker_patch_2024_3_8.zip), except for ~/moonraker/moonraker/components/machine.py. overwrite virtual_sdcard.py and machine.py with the ones on this repository (the qidi files don't work, I had to comment or delete a few lines) `wget https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/virtual_sdcard.py ; mv virtual_sdcard.py ~/klipper/klippy/extras/virtual_sdcard.py ; wget https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/machine.py ; mv machine.py ~/moonraker/moonraker/components/machine.py` (press y if prompted to overwrite files).