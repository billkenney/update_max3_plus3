################################################################################

qidi has unoficially released a completely updated image for the max3 with all of the newest software. but the company itself is not providing warranty for updated systems. after a bit of testing, it appears to fix the issues with the image preview but only when you print from the screen. it does nothing when i click the network tab. obviously, a qidi employee is also more familiar with the qidi-specific software, and i'm sure he's done more testing than i have, so i would install the image from @CChen616: https://github.com/CChen616/QIDI_Max3_Bookworm. as with my image, it also appears 

in order to get this image working, you have to run the following commands:<br>
`sudo rm /etc/environment ; sudo touch /etc/environment ; sudo sed -i '/^\[http\]/d;/^\[https\]/d;/proxy/d' /root/.gitconfig ; sed -i '/^\[https\]/d;/proxy/d' /home/mks/.gitconfig ; sudo mv /etc/apt/sources.list.bak /etc/apt/sources.list ; sudo sed -i '/Defaults env_keep+="http_proxy https_proxy no_proxy"/d' /etc/sudoers ; sudo rm /etc/wpa_supplicant/wpa_supplicant-wlan0.conf ; sudo touch /etc/wpa_supplicant/wpa_supplicant-wlan0.conf ; sudo service makerbase-wlan0 restart`

if youre using a 32gb emmc, run `sudo systemctl enable armbian-resize-filesystem ; sudo reboot`. fix your locale/timezone: `sudo dpkg-reconfigure locales`. find your time zone here: https://en.m.wikipedia.org/wiki/List_of_tz_database_time_zones (it should be in the format America/Chicago), then run `sudo timedatectl set-timezone [your_timezone] ; sudo timedatectl set-ntp 1` replacing [your_timezone] with your actual timezone

for all printers, you need to flash the firmware on the mcus. you can complete steps 3-5 of this guide or follow qidi's guide: https://github.com/QIDITECH/QIDI_PLUS3/issues/27#issuecomment-2073932891. i believe klipper.uf2 and X_4.bin are in the /root/klipper_modified folder of qidi's image, and they should be the same as the ones on this repo

qidi's updated printer.cfg is quite a bit different than the one that ships with the firmware, and i'm not sure what changes would need to be made to get it working with the plus3/smart3. so plus3/smart3 owners probably have to complete steps 7-9 to get it working, or go through the changes on your own

if you have the max3 with the inductive probe, skip step 7 and complete steps 8-9

if you have the max3 with the bltouch, skip step 7 and complete steps 8-9, however, you also need to modify some references to 'probe' in the printer.cfg file and the gcode_macro.cfg. run these commands: <br>
`sed -i 's/endstop_pin:probe:z_virtual_endstop/endstop_pin:bltouch:z_virtual_endstop/;s/^\[probe\]/\[bltouch\]/;s/pin: \^MKS_THR:gpio21/sensor_pin:\^\MKS_THR:gpio21\ncontrol_pin:MKS_THR:gpio11\nstow_on_each_sample: False/' ~/printer_data/config/printer.cfg ; sed -i 's/printer\.configfile\.settings\.probe\.x_offset/printer\.configfile\.settings\.bltouch\.x_offset/g;s/printer\.configfile\.settings\.probe\.y_offset/printer\.configfile\.settings\.bltouch\.y_offset/g' ~/printer_data/config/gcode_macro.cfg`

################################################################################

this guide will allow you to update qidi max3 / plus3 / smart3 to debian bookworm with the edge kernel, and the latest klipper, moonraker, and fluidd (or mainsail) without losing functionality of the screen. just follow steps 1-9 (and optionally 10-13)

the wifi menu on the printer screen does not work after upgrading (see https://github.com/billkenney/update_max3_plus3/issues/5). you can run `sudo nmtui` to use the network-manager service to create or manage network connections, and it will automatically connect on boot

################################################################################

NOTE: i have added support for the smart3 based on a comment from qidi (https://github.com/QIDITECH/QIDI_MAX3/issues/53#issuecomment-2151230861) stating that the klipper mcu firmware files are the same--so you can use the ones from my repo, and the only difference is xindi (likely because the screen is a different size) and the printer.cfg file. you can follow the same process outlined below, the only difference is that you have to reinstall the smart3 firmware (see step 6), which will install the correct version of xindi for your printer. i am not aware of anyone updating the smart3 to the latest software yet, so this method is 100% untested and could brick your printer. probably a good idea to have a backup emmc (or backup image and emmc adapter) on hand if you can't get it to work

################################################################################

if you want to try the much more complicated manual install method, you can follow the manual steps (i have not created a manual guide for the smart3): https://github.com/billkenney/update_max3_plus3/blob/main/manual.md

if you want to revert for some reason, you can follow these steps to revert to the stock image and flash the mcus with the old software: https://github.com/billkenney/update_max3_plus3/blob/main/revert.md

qidi has released some patch files, which, as far as i can tell, only allow you to see the thumbnails on the screen (which never really worked for me anyways). its possible it could also fix the wifi menu on the touch screen? other people have said the patch files cause problems, so i would recommend skipping steps 11-12

if you've done all of the steps and are getting a message that the system starts abnormally, its possible that you did not correctly flash the extruder mcu (https://github.com/billkenney/update_max3_plus3/issues/4). try running step 4 again

the default user is 'mks' and password (for ssh and sudo) is 'makerbase'

1. write this image to your emmc: https://github.com/billkenney/update_max3_plus3/releases/download/qidi_update/qidi_update.img.7z

using a terminal client such as putty for windows, Terminal on macos or linux, or an app like shelly or terminus on your phone, ssh into your printer: `ssh mks@printer.ip.address` replacing 'printer.ip.address' with your printers ip address. the username is 'mks' and the password is 'makerbase' any time you are prompted for a password

2. if youre using a 32gb emmc, run `sudo systemctl enable armbian-resize-filesystem ; sudo reboot`. the language is set to English and the timezone is set to America/Chicago. to change this, run `sudo dpkg-reconfigure locales`. find your time zone here: https://en.m.wikipedia.org/wiki/List_of_tz_database_time_zones (it should be in the format America/Chicago), then run `sudo timedatectl set-timezone [your_timezone] ; sudo timedatectl set-ntp 1` replacing [your_timezone] with your actual timezone

3. to flash the mcu firmware, download this file, copy it to a micro sd card, plug it into the printer, and restart your printer: https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/X_4.bin

![325057677-0d15df45-8cd8-4e4c-88ec-a71b152b8cbb](https://github.com/billkenney/update_max3_plus3/assets/30010560/ce1f6465-d539-4137-80bd-90f31bab7661)

4. to flash the extruder mcu, you need to unplug all usb devices except for the extruder, then hold the bottom left button on the back of your extruder board (see the image) for like 2 minutes or until the screen loads up fully, then ssh into your printer and run `sudo mount /dev/sda1 /mnt`. if you get an error that it's mounted read-only, the extruder has not been mounted properly. restart and try to boot into dfu mode again before continuing. if it mounts with no errors, run `sudo systemctl daemon-reload ; wget https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/klipper.uf2 ; mv klipper.uf2 /mnt` then restart your printer

![325058698-1a76832d-02ad-4cd7-aa7c-f63277600226](https://github.com/billkenney/update_max3_plus3/assets/30010560/46a879b1-d77c-468d-b7ab-371fcdcf8673)

5. to flash the rpi mcu, ssh into your printer and run `cd ~/klipper ; make clean ; make menuconfig` and configure as shown in the below image. Then press 'q' and select the option to save, then `sudo service klipper stop ; make flash ; sudo service klipper start`

![325061507-b820ced1-ac3a-4627-b366-04fd95770e5d](https://github.com/billkenney/update_max3_plus3/assets/30010560/de954ba9-a158-42d0-b564-d3a71169f4bc)

6. ssh into your printer and run `sudo mv ~/klipper_config/MKS_THR.cfg ~/klipper_config/MKS_THR.cfg.bak ; path=$(ls /dev/serial/by-id/*) ; printf "[mcu MKS_THR]\nserial:$path\n" > ~/klipper_config/MKS_THR.cfg ; rm ~/klipper_config/config/MKS_THR.cfg ; ln -s ~/klipper_config/MKS_THR.cfg ~/klipper_config/config/MKS_THR.cfg`

7. you can skip this step if you have the max3 with the bltouch

for the max3 with the inductive probe run: `wget https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/printer-max3_probe.cfg ; mv printer-max3_probe.cfg ~/klipper_config/config/printer.cfg`

for the plus3 run: `wget https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/printer-plus3.cfg ; mv printer-plus3.cfg ~/klipper_config/config/printer.cfg`

for the smart3 run: `wget https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/printer-smart3.cfg ; mv printer-smart3.cfg ~/klipper_config/config/printer.cfg ; wget --no-check-certificate https://raw.githubusercontent.com/billkenney/qidi_3series_recovery/main/mksclient-smart3.deb ; sudo dpkg -i mksclient-smart3.deb`

8. to install the screen firmware for the max3 or the plus3 (which is not necessary if you installed the latest screen firmware on your printer prior to updating), run `sudo mv /root/800_480.tft.bak /root/800_480.tft`, restart your printer, you should see a white screen with a progress indicator similar to this image (it could take a few minutes to start, so be patient)

to install the screen firmware for the smart3 (which is not necessary if you installed the latest screen firmware on your printer prior to updating), run `wget --no-check-certificate https://raw.githubusercontent.com/billkenney/qidi_3series_recovery/main/800_480-smart3.tft ; sudo mv 800_480-smart3.tft /root/800_480.tft`, you should see a white screen with a progress indicator similar to this image (it could take a few minutes to start, so be patient)

![IMG_2028](https://github.com/billkenney/update_max3_plus3/assets/30010560/f5cf29b5-9c42-475f-9e84-a78b302265bf)

9. if you have a webcam, run `vidpath=$( ls -la /dev/v4l/by-id/ | grep index0 | grep -o 'video[0-9]' ) ; sed -i "s/device: \/dev\/video[0-9]/device: \/dev\/$vidpath/;s/resolution: [0-9]*x[0-9]*/resolution: 1280x960/" ~/klipper_config/config/crowsnest.conf ; sudo service crowsnest restart`. restart your printer and you should have the latest software and a working screen

10. if you want to run the resonance testing macro, you need to install these: `sudo apt install python3-numpy python3-matplotlib libatlas-base-dev libopenblas-dev ; ~/klippy-env/bin/pip install -v numpy`

11. if you want to install moonraker-obico, run these commands then install with kiauh: 
`sudo mkdir -p /root/.config/pip ; sudo printf "[global]\nbreak-system-packages = true\n" > /root/.config/pip/pip.conf ; sudo pip3 install -U urllib3 requests ; sudo apt install janus ; sudo systemctl enable janus ; sudo systemctl start janus`

################################################################################

the installation of these patch files is not necessary, and i recommended you skip these steps as they could cause problems. although they could get thumbnails working on the screen again

################################################################################

12. if you are installing qidi's patch files, for the max3 with the bltouch run: `wget https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/printer-max3_bltouch_patch.cfg ; mv printer-max3_bltouch_patch.cfg ~/klipper_config/config/printer.cfg`

for the max3 with the inductive probe run: `wget https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/printer-max3_probe_patch.cfg ; mv printer-max3_probe_patch.cfg ~/klipper_config/config/printer.cfg`

for the plus3 run: `wget https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/printer-plus3_patch.cfg ; mv printer-plus3_patch.cfg ~/klipper_config/config/printer.cfg`

for the smart3 run: `wget https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/printer-smart3_patch.cfg ; mv printer-smart3_patch.cfg ~/klipper_config/config/printer.cfg`

13. to install qidi's patch files, overwrite all of the files specified in issue 27 (https://github.com/QIDITECH/QIDI_PLUS3/issues/27#issuecomment-2073932891) with those in the klipper zip file (https://github.com/QIDITECH/QIDI_PLUS3/files/15087211/files_to_replace.zip), except for ~/klipper/klippy/extras/virtual_sdcard.py. overwrite the files specified here (https://github.com/QIDITECH/moonraker/issues/1#issuecomment-1985564638) with those in the moonraker zip file (https://github.com/QIDITECH/moonraker/files/14537786/moonraker_patch_2024_3_8.zip), except for ~/moonraker/moonraker/components/machine.py. overwrite virtual_sdcard.py and machine.py with the ones on this repository (the qidi files don't work, I had to comment or delete a few lines) `wget https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/virtual_sdcard.py ; mv virtual_sdcard.py ~/klipper/klippy/extras/virtual_sdcard.py ; wget https://raw.githubusercontent.com/billkenney/update_max3_plus3/main/machine.py ; mv machine.py ~/moonraker/moonraker/components/machine.py`
