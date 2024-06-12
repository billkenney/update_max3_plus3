these are the steps i had to take to install the image from @CChen616. i ran into quite a few issues, so i ended up switching back to my image

qidi has unoficially released a completely updated image for the max3 with all of the newest software. but the company itself is not providing warranty for updated systems. after a bit of testing, it appears to intermittently fix the issues with the image preview. it seems to work better when you print ftom the screen (like it did with the old software). it does nothing when i click the network tab on the screen--it doesnt even go into network settings. so if you don't have ethernet, you should probably get an adapter before continuing. you can set up wifi wothout the screen (sudo nmtui) but you need to be able to ssh into your printer

here's the link to the image from @CChen616: https://github.com/CChen616/QIDI_Max3_Bookworm

in order to get the image from @CChen617 working, you have to run the following commands (this fixes the issues towards the bottom of his readme):<br>
`sudo rm /etc/environment ; sudo touch /etc/environment ; sudo sed -i '/^\[http\]/d;/^\[https\]/d;/proxy/d' /root/.gitconfig ; sed -i '/^\[https\]/d;/proxy/d' /home/mks/.gitconfig ; sudo mv /etc/apt/sources.list.bak /etc/apt/sources.list ; sudo sed -i '/Defaults env_keep+="http_proxy https_proxy no_proxy"/d' /etc/sudoers ; sudo rm /etc/wpa_supplicant/wpa_supplicant-wlan0.conf ; sudo touch /etc/wpa_supplicant/wpa_supplicant-wlan0.conf ; sudo service makerbase-wlan0 restart`

fix your locale/timezone: `sudo dpkg-reconfigure locales`. find your time zone here: https://en.m.wikipedia.org/wiki/List_of_tz_database_time_zones (it should be in the format America/Chicago), then run `sudo timedatectl set-timezone [your_timezone] ; sudo timedatectl set-ntp 1` replacing [your_timezone] with your actual timezone

if youre using a 32gb emmc, run `sudo systemctl enable armbian-resize-filesystem ; sudo reboot`

for all printers, you need to flash the firmware on the mcus. you can complete steps 3-5 of this guide or follow qidi's guide: https://github.com/QIDITECH/QIDI_PLUS3/issues/27#issuecomment-2073932891. i believe klipper.uf2 and X_4.bin are in the /root/klipper_modified folder of qidi's image, and they should be the same as the ones on this repo

qidi's updated printer.cfg is quite a bit different than the one that ships with the firmware, and i'm not sure what changes would need to be made to get it working with the plus3/smart3. so plus3/smart3 owners probably have to complete steps 7-9 to get it working, or go through the changes on your own

if you have the max3 with the inductive probe, skip step 7 and complete steps 8-9

if you have the max3 with the bltouch, skip step 7 and complete steps 8-9, however, you also need to modify some references to 'probe' in the printer.cfg file and the gcode_macro.cfg. run these commands after updating if you use the image from @CChen616: <br>
`sed -i 's/pin: \^MKS_THR:gpio21/sensor_pin:\^\MKS_THR:gpio21\ncontrol_pin:MKS_THR:gpio11\nstow_on_each_sample: False/' ~/printer_data/config/printer.cfg ; sed -i 's/printer\.configfile\.settings\.probe\.x_offset/printer\.configfile\.settings\.bltouch\.x_offset/g;s/printer\.configfile\.settings\.probe\.y_offset/printer\.configfile\.settings\.bltouch\.y_offset/g' ~/printer_data/config/gcode_macro.cfg`

you may or may not have to run the below command. i had to do this initially, but when i installed the btt sfs i have to reverse it. if you get errors about probecommandhelper then reverse it: sed -i `s/endstop_pin:probe:z_virtual_endstop/endstop_pin:bltouch:z_virtual_endstop/;s/^\[probe\]/\[bltouch\]/` ~/printer_data/config/printer.cfg
