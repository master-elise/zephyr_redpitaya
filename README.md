# ZephyrOS on the Zynq 7010 of the Red Pitaya

## Patching Zephyr for the Red Pitaya

ZephyrOS supports the Zybo, a <a href="https://digilent.com/reference/programmable-logic/zybo-z7/reference-manual">Zynq-based evaluation board</a>. According
to the pinout, its bank 0/pin 7 is connected to LED4, which also happens to
be the red PS-accessible MIO pin on the Red Pitaya. Hence, the devicetree
entry ``zephyr/boards/digilent/zybo/zybo.dts`` stating ``gpios = <&psgpio_bank0 7 GPIO_ACTIVE_HIGH>;`` remains valid.

However, the Zibo communicates over UART1 whereas the USB-microB port is
connected to UART0 of the Zynq on the Red Pitaya. Gwenhael Goavec-Merou
provides the <a href="zephyr_redpit.patch">following patch</a> to change
the communication port: apply with ``patch -p1 < zephyr_redpit.patch`` from
the ``zephyr`` directory.

## Compiling ZephyrOS on the Red Pitaya

ZephyrOS ~~annoyingly fetches all BSPs~~ only fetches the necessary CMSIS
library needed for this demonstration when installing with
```
pip install west --break-system-packages
west init
west update cmsis # DO NOT "west update" which will download 9 GB of useless BSPs
```
so at least we save a bit of space by using the distribution (Debian) provided
cross-compiler instead of the one provided by ZephyrOS. At the end of
``zephyr-env.sh`` we add
```
export ZEPHYR_TOOLCHAIN_VARIANT=cross-compile
export CROSS_COMPILE=/usr/bin/arm-none-eabi-
```
to use the compiler provided by the ``gcc-arm-none-eabi``. Whenever ZephyrOS
is to be used, source this configuration file with
``source zephyr-env.sh``.

## Running ZephyrOS on the Red Pitaya

Assuming a <a href="https://github.com/trabucayre/redpitaya">Buildroot</a> SD
card for the Red Pitaya is available which already provides U-Boot support, 
then compile the Zephyr blinking LED example with
```
west build -b zybo  zephyr/samples/basic/blinky
```
and copy the resulting ``cp build/zephyr/zephyr.bin /mnt/sdb1/zephyr_led.bin``
on the first partition of the SD card.

Launch the Red Pitaya and stop the automatic boot sequence of U-Boot, then type
```
dcache off                                                                
fatload mmc 0 0x0 zephyr_led.bin                                          
dcache flush                                                              
go 0x0                                                                    
```
to launch the program. The terminal will display
```
Importing environment from mmc ...                                              
Checking if uenvcmd is set ...                                                  
Hit any key to stop autoboot:  0                                                
Zynq> setenv autostart yes                                                      
Zynq> dcache off                                                                
Zynq> fatload mmc 0 0x0 zephyr_led.bin                                          
41120 bytes read in 25 ms (1.6 MiB/s)                                           
Zynq> dcache flush                                                              
Zynq> go 0x0                                                                    
## Starting application at 0x00000000 ...                                       
*** Booting Zephyr OS build 1b23efc6121e ***                                    
LED state: OFF                                                                  
LED state: ON                                                                   
LED state: OFF                                                                  
LED state: ON                                                                   
LED state: OFF                                                                  
LED state: ON                                                                   
LED state: OFF                                                                  
...
```
and the red LED will be blinking.
