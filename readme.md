Update qidi max3 or plus3 to debian bookworm with the edge kernel, and the latest klipper and moonraker, without losing functionality of the screen. 

1. write this image to your emmc: https://github.com/redrathnure/armbian-mkspi/releases/download/mkspi%2F0.3.4-24.2.0-trunk/Armbian-unofficial_24.2.0-trunk_Mkspi_bookworm_edge_6.7.5.img.xz
2. follow the instructions here: https://github.com/QIDITECH/QIDI_PLUS3/issues/27#issuecomment-2073932891 (and git pull doesn't work if you cloned the qidi repository, you need to clone the actual klipper repo... also don't overwrite the klipper files yet)
3. when it comes time to flash the extruder, it's not going to appear in the gcode folder. you need to sudo mount /dev/sda1 /mnt, then copy the klipper.uf2 file to /mnt, then restart
4. follow the instructions here: https://github.com/QIDITECH/moonraker/issues/1#issuecomment-1985564638 (again, git pull doesn't work if you cloned the qidi repository... also don't overwrite the moonraker files yet)
5. download all of the service files from this repo, and run systemctl enable systemctl enable makerbase-byid.service ; sudo systemctl enable makerbase-client.service ; sudo systemctl enable makerbase-net-mods.service
6. follow the instructions here to install the latest firmware (this also overwrites klipper/moonraker files): https://github.com/billkenney/max3_plus3_recovery
7. now overwrite all of the files as described in the qidi issues, except for ~/klipper/klippy/extras/virtual_sdcard.py and ~/moonraker/moonraker/components/machine.py, overwrite those with the ones on this repository (the qidi files don't work, I had to comment or delete a few lines)
8. download libboost_system.so.1.67.0 and libwpa_client.so from this repo. sudo mv libwpa_client.so /usr/lib ; sudo mv libboost_system.so.1.67.0 /usr/lib/aarch64-linux-gnu
9. sudo systemctl start makerbase-byid.service ; sudo systemctl start makerbase-client.service ; sudo systemctl start makerbase-net-mods.service ; sudo systemctl shutdown -H now
10. turn your printer off, wait for a bit, and turn it back on. the screen should be on the firmware update screen
