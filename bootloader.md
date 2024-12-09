First stage:
Download bootloader files from:
[https://downloads.openwrt.org/releases/23.05.5/targets/mediatek/filogic/openwrt-23.05.5-mediatek-filogic-mediatek_mt7981-rfb-spim-nand-bl31-uboot.fip](https://downloads.openwrt.org/releases/23.05.5/targets/mediatek/filogic/openwrt-23.05.5-mediatek-filogic-mediatek_mt7981-rfb-spim-nand-bl31-uboot.fip)
[https://downloads.openwrt.org/releases/23.05.5/targets/mediatek/filogic/openwrt-23.05.5-mediatek-filogic-mediatek_mt7981-rfb-spim-nand-preloader.bin](https://downloads.openwrt.org/releases/23.05.5/targets/mediatek/filogic/openwrt-23.05.5-mediatek-filogic-mediatek_mt7981-rfb-spim-nand-preloader.bin)

Have a FTPD server ready at PC hosting the bootloader files.
Make sure the TCPIP adress of router and PC fall into same network adress range.
(for example router on 192.168.2.254 and PC on 192.168.2.10. But can choose adress range freely)

Connect USB->UART device to RM65 router UART pins (TX,RX,GND dont connect VCC)
start terminal program on pc to watch UART output

Start up router
Press key to interrupt bootsequence

Upgrade BL2 bootloader file
(mt981-rfb-spim-nand-preloader.bin)
dont reboot RM65 router yet.

Upgrade BL31 bootloader file
(mt7981-rfb-spim-nand-bl31-uboot.fip)
when finished reboot router.

Second stage:
Reboot the router and press key during bootup to get into bootloader shell.

Download Openwrt 23.05.5 initram.itb and sysupgrade files:
[https://downloads.openwrt.org/releases/23.05.5/targets/mediatek/filogic/openwrt-23.05.5-mediatek-filogic-mediatek_mt7981-rfb-initramfs.itb](https://downloads.openwrt.org/releases/23.05.5/targets/mediatek/filogic/openwrt-23.05.5-mediatek-filogic-mediatek_mt7981-rfb-initramfs.itb)
[https://downloads.openwrt.org/releases/23.05.5/targets/mediatek/filogic/openwrt-23.05.5-mediatek-filogic-mediatek_mt7981-rfb-squashfs-sysupgrade.itb](https://downloads.openwrt.org/releases/23.05.5/targets/mediatek/filogic/openwrt-23.05.5-mediatek-filogic-mediatek_mt7981-rfb-squashfs-sysupgrade.itb)
Having TFTPD server ready which can serve the "initramfs.itb" ramdisk kernel file for initial startup.
(IP addresses choosen here is 192.168.2.1 for RM65 router and 192.168.2.10 for PC, can choose freely)

Then copy paste the following lines in bootloader shell:
ubi detach ubi;mtd erase ubi;ubi part ubi;ubi create ubootenv 126976;ubi create ubootenv2 126976;setenv serverip 192.168.2.10;setenv ipaddr 192.168.2.1;setenv loadaddr '0x46000000';setenv bootconf 'config-1#';setenv bootcmd 'ubi read $loadaddr fit;bootm $loadaddr#$bootconf';setenv bootfile 'initramfs.itb';saveenv;saveenv;tftpboot;bootm $loadaddr#$bootconf

This will re-create partition scheme and setup bootloader environmental values and saving it to nvram and then load up initramfs.itb from TFTP server and then boot from it.

Third stage:
If everything went well, then the router is now starting up from Openwrt 23-05.5 ramdisk.
This is only ramdisk, the process of really installing the Openwrt-23 firmware will be next.

When loaded you can go to Luci interface on 192.168.2.254
and then go to administration -> upgrade firmware
and choose your sysupgrade.itb firmware file (mt7981-rfb-spim-nand compatible .itb files) to upload it to the router.
Please uncheck "keep configuration settings" because for first upgrade its better to reset its settings.

Then after the RM65 router restarts, it should start up with Openwrt 23-05.5 installed.
To get lan port 5 working and leds configured, compile the firmware with the mt7981b-rm65.dts and mt7981b-rm65.dtsi.
