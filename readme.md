Update qidi max3 or plus3 to debian bookworm with the edge kernel, and the latest fluidd (or mainsail), klipper, and moonraker, without losing functionality of the screen. 

1. write this image to your emmc: https://github.com/redrathnure/armbian-mkspi/releases/download/mkspi%2F0.3.4-24.2.0-trunk/Armbian-unofficial_24.2.0-trunk_Mkspi_bookworm_edge_6.7.5.img.xz
2. follow the instructions here: https://github.com/QIDITECH/QIDI_PLUS3/issues/27#issuecomment-2073932891 (and git pull doesn't work if you cloned the qidi repository, you need to git clone https://github.com/Klipper3d/klipper.git... also don't overwrite the klipper files yet)
3. when it comes time to flash the extruder, it's not going to appear in the gcode folder. you need to sudo mount /dev/sda1 /mnt, then copy the klipper.uf2 file to /mnt, then restart
4. follow the instructions here: https://github.com/QIDITECH/moonraker/issues/1#issuecomment-1985564638 (again, git pull doesn't work if you cloned the qidi repository, you need to git clone https://github.com/Arksine/moonraker.git... also don't overwrite the moonraker files yet)
5. follow the instructions here to install the latest firmware (this also overwrites some of the klipper/moonraker files, which is why we didn't do it earlier): https://github.com/billkenney/max3_plus3_recovery
6. now overwrite all of the files as described in the qidi issues, except for ~/klipper/klippy/extras/virtual_sdcard.py and ~/moonraker/moonraker/components/machine.py, overwrite those with the ones on this repository (the qidi files don't work, I had to comment or delete a few lines)
7. download libboost_system.so.1.67.0 and libwpa_client.so from this repo. sudo mv libwpa_client.so /usr/lib ; sudo mv libboost_system.so.1.67.0 /usr/lib/aarch64-linux-gnu
8. download all of the service files from this repo, move them to /lib/systemd/system, and run sudo systemctl enable makerbase-byid.service ; sudo systemctl enable makerbase-client.service ; sudo systemctl enable makerbase-net-mods.service ; sudo systemctl start makerbase-byid.service ; sudo systemctl start makerbase-client.service ; sudo systemctl start makerbase-net-mods.service
9. if your screen shows the firmware update has already started, i guess leave it on. otherwise turn your printer off, wait for a bit, and turn it back on, the screen should be on the firmware update screen.
10. once you update, in your printer.cfg, you need to change max_accel_to_decel: 10000 (deprecated) to minimum_cruise_ratio: 0.5.
11. to get your wifi working again. sudo ln -s /usr/src/linux-headers-6.7.5-edge-rockchip64/arch/arm64 /usr/src/linux-headers-6.7.5-edge-rockchip64/arch/aarch64 ; sudo apt install make gcc build-essential git ; git clone https://github.com/lwfinger/rtl8xxxu ; cd rtl8xxxu ; make clean modules ; sudo make install ; sudo make install_fw. Restart the printer, then cd rtl8xxxu ; sudo modprobe rtl8xxxu_git

Note that you can update klipper/moonraker/fluidd without updating the operating system, and it is also not necessary to install the patch files from Qidi. If you did update the operating system, you have to install the firmware updates as well as the services on my repo (which overwrites files in klipper and moonraker). So if you want vanilla klipper/moonraker, you need to re/install clean versions of the software after updating the firmware. It seems like as long as the makerbase-client.service and xindi is active, the screen will work. @nessex has written a script to streamline the process, which is available here: https://gist.github.com/nessex/7b574fbe6d965439b773d922ca1b9e05

Also note that if you do not use the patch files from Qidi, if you have the inductive probe, in the printer.cfg, you need to change every instance of printer.probe["x_offset"] to printer.configfile.settings.probe.x_offset and every instance of printer.probe["y_offset"] to printer.configfile.settings.probe.y_offset. If you have the bltouch, replace those values with printer.configfile.settings.bltouch.x_offset and printer.configfile.settings.bltouch.y_offset. 
