1. write this image to your emmc: https://drive.google.com/file/d/169c8aaMe8YdqOP-cswjxlOCQqbp_6HrL/view?usp=drivesdk
2. to flash your motherboard mcu: `make clean ; make menuconfig`. configure with the below options, then press 'q' and select the option to save. then run `make`. download ~/klipper/out/klipper.bin to uour computer, rename it X_4.bin, put it on a micro sd card, plug it into the printer, and restart the printer. 

[*] Enable extra low-level configuration options

Micro-controller Architecture (STMicroelectronics STM32)

Processor model (STM32F401)

Bootloader offset (32KiB bootloader)

Clock Reference (8 MHz crystal)

Communication interface (Serial (on USART1 PA10/PA9))

(250000) Baud rate for serial port (NEW)
make

3. to flash the extruder mcu, you need to unplug all usb devices except for the extruder, then hold the bottom left button on the back of your extruder board (see the image) for like 2 minutes or until the screen loads up fully, then ssh into your printer. run lsblk and if it shows this output you can continue. 

sda            8:0    1   128M  0 disk
└─sda1         8:1    1   128M  0 part /home/mks/gcode_files/sda1

run `make clean ; make menuconfig`,  configure with the below options, then press 'q' and select the option to save. then run `make`. then `cp /home/mks/klipper/out/klipper.uf2 /home/mks/gcode_files/sda1`, and restart the printer

[*] Enable extra low-level configuration options

Micro-controller Architecture (Raspberry Pi RP2040)

Flash chip (W25Q080 with CLKDIV 2)

Communication interface (USB)

![325058698-1a76832d-02ad-4cd7-aa7c-f63277600226](https://github.com/billkenney/update_max3_plus3/assets/30010560/46a879b1-d77c-468d-b7ab-371fcdcf8673)

4. to flash the rpi mcu, ssh into your printer and run cd ~/klipper ; make clean ; make menuconfig and configure as shown in the below image. Then press 'q' and select the option to save, then sudo service klipper stop ; make flash ; sudo service klipper start

![325061507-b820ced1-ac3a-4627-b366-04fd95770e5d](https://github.com/billkenney/update_max3_plus3/assets/30010560/de954ba9-a158-42d0-b564-d3a71169f4bc)

5. if you have the max3 with the bltouch, run `wget https://raw.githubusercontent.com/billkenney/max3_plus3_recovery/main/mksclient-max3.deb ; sudo dpkg -i mksclient-max3.deb ; wget https://raw.githubusercontent.com/billkenney/max3_plus3_recovery/main/printer-max3_bltouch.cfg ; mv printer-max3_bltouch.cfg /home/mks/klipper_config/printer.cfg`. if you have the max3 with the inductive probe, run `wget https://raw.githubusercontent.com/billkenney/max3_plus3_recovery/main/mksclient-max3.deb ; sudo dpkg -i mksclient-max3.deb ; wget https://raw.githubusercontent.com/billkenney/max3_plus3_recovery/main/printer-max3_probe.cfg ; mv printer-max3_probe.cfg /home/mks/klipper_config/printer.cfg`. if you have the plus3, run `wget https://raw.githubusercontent.com/billkenney/max3_plus3_recovery/main/mksclient-plus3.deb ; sudo dpkg -i mksclient-plus3.deb ; wget https://raw.githubusercontent.com/billkenney/max3_plus3_recovery/main/printer-plus3.cfg ; mv printer-plus3.cfg /home/mks/klipper_config/printer.cfg`. restart yoir printer and you should be good to go