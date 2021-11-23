# Burn TF Card

If you don't want to fetch code and compile, you can do this step,  skip **Fetch code** and **Build TF Card Booting Image** chapter

downloag the lastest releases image [script_burn_tf_visionfive.tar.gz](https://github.com/jianlonghuang/script_burn_tf)

```
tar -xvf script_burn_tf_visionfive.tar.gz
cd script_burn_tf
sudo ./make_format-nvdla-rootfs.sh /dev/sdx
```

Note: 

1. Please insert the TF card and run command `df -h` to check the device name `/dev/sdXX`
2. run environment of ubunut `sudo apt-get install gdisk parted gparted`



# Fetch code

[https://github.com/starfive-tech/freelight-u-sdk.git](https://github.com/starfive-tech/freelight-u-sdk.git)

> branch:
>
> ​	JH7100_starlight_multimedia
>
> linux branch:
>
> ​	visionfive_5.13_devel
>
> HiFive_U-Boot branch:
>
> ​	JH7100_upstream
>
> buildroot branch:
>
> ​	starlight_multimedia
>
> opensbi branch:
>
> ​	master



*fetch code instruction: more details refer to [README](https://github.com/starfive-tech/freelight-u-sdk/tree/JH7100_starlight_multimedia)*

```
$ git clone https://github.com/starfive-tech/freelight-u-sdk.git
$ git checkout --track origin/JH7100_starlight_multimedia
$ git submodule update --init --recursive
$ cd buildroot && git checkout starlight_multimedia && cd ..
$ cd HiFive_U-Boot && git checkout JH7100_upstream && cd ..
$ cd linux && git checkout visionfive_5.13_devel && cd ..
$ cd opensbi && git checkout master && cd ..
```



# Build TF Card Booting Image

please modify the following file

freelight-u-sdk/Makefile

```
change
uboot_config := starfive_jh7100_starlight_smode_defconfig
to
uboot_config := starfive_jh7100_visionfive_smode_defconfig
```

conf/beaglev_defconfig_513

```
change
CONFIG_CMDLINE="earlyprintk console=tty1 console=ttyS0,115200 debug rootwait stmmaceth=chain_mode:1"
to
CONFIG_CMDLINE="earlyprintk console=tty1 console=ttyS0,115200 debug rootwait stmmaceth=chain_mode:1 root=/dev/mmcblk0p3"


change
CONFIG_FB_STARFIVE_VIDEO=y
to
#CONFIG_FB_STARFIVE_VIDEO=y

add
CONFIG_FRAMEBUFFER_CONSOLE=y
```

conf/u74_nvdla-uboot-fit-image.its

```
change
data = /incbin/("../work/HiFive_U-Boot/arch/riscv/dts/jh7100-beaglev-starlight-a1.dtb");
to
data = /incbin/("../work/linux/arch/riscv/boot/dts/starfive/jh7100-beaglev-starlight-a1.dtb");
```

HiFive_U-Boot/configs/starfive_jh7100_visionfive_smode_defconfig

```
change
CONFIG_USE_BOOTCOMMAND is not set
to
CONFIG_USE_BOOTCOMMAND=y
```

HiFive_U-Boot/include/configs/starfive-jh7100.h

```
change
 #define CONFIG_EXTRA_ENV_SETTINGS \
        STARLIGHT_FEDORA_BOOTENV \
        "loadaddr=0xa0000000\0" \
        "loadbootenv=fatload mmc ${mmcdev} ${loadaddr} ${bootenv}\0" \
to
 #define CONFIG_EXTRA_ENV_SETTINGS \
        STARLIGHT_FEDORA_BOOTENV \
        "mmcsetup=mmc part\0" \
        "fdtsetup=fdt addr ${fdtcontroladdr}\0" \
        "fatenv=setenv fileaddr a0000000; fatload mmc 0:1 ${fileaddr} u74_uEnv.txt;" \
        "env import -t ${fileaddr} ${filesize}\0" \
        "loadaddr=0xa0000000\0" \
        "loadbootenv=fatload mmc ${mmcdev} ${loadaddr} ${bootenv}\0" \
```



Please insert the TF card and run command `df -h` to check the device name `/dev/sdXX`, then run command `umount /dev/sdXX`", then run the following instructions to build TF card image:

```
make buildroot_rootfs -jx
make -jx
make DISK=/dev/sdX format-nvdla-rootfs && sync
```



Note: please also do not forget to update the `fw_payload.bin.out` which is built by this step.



# Copy test_script

**Step1**: fetch test_script

```
git clone https://github.com/jianlonghuang/test_script.git
```

**Step2**: 

modify **cfg.ini** as required,  more detail refer to [test_script readme](https://github.com/jianlonghuang/test_script)

**Step3**:

copy to tf card  /dev/sdX3

```
sudo cp -r test_script/ /media/jianlong/465a8d4f-31fe-40c3-951e-4a565bd3a620/ && sync
```

Note: /media/jianlong/465a8d4f-31fe-40c3-951e-4a565bd3a620/ is mounted path of /dev/sdX3



# Running on Board

*more details refer to [README](https://github.com/starfive-tech/freelight-u-sdk/tree/JH7100_starlight_multimedia)*

downloag the lastest releases uboot file [fw_payload.bin.out](https://github.com/jianlonghuang/script_burn_tf)

After the Visionfive is properly connected to the serial port cable, network cable and power cord turn on the power from the wall power socket to power on the Visionfive and you will see the startup information as follows:

```
bootloader version: 210209-4547a8d
ddr 0x00000000, 1M test
ddr 0x00100000, 2M test
DDR clk 2133M,Version: 210302-5aea32f
2
```

Press any key as soon as it starts up to enter the **upgrade menu**. In this menu, you can update uboot

```
bootloader version: 210209-4547a8d
ddr 0x00000000, 1M test
ddr 0x00100000, 2M test
DDR clk 2133M,Version: 210302-5aea32f
0
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
xxxxxxxxxxxFLASH PROGRAMMINGxxxxxxxxxxxxxxx
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

0:update boot
1:quit
select the function:
```

Type **"0"** to update the uboot file [fw_payload.bin.out](https://github.com/jianlonghuang/script_burn_tf) via Xmodem mode, and then Type **"1"** to exit Flash Programming.

After that, you will see the information `VisionFive #` when press the "f" button



**Step1**: set enviroment parameter:

```
setenv fdt_high 0xffffffffffffffff;setenv initrd_high 0xffffffffffffffff;setenv scriptaddr 0x88100000;setenv script_offset_f 0x1fff000;setenv script_size_f 0x1000;setenv kernel_addr_r 0x84000000;setenv kernel_comp_addr_r 0x90000000;setenv kernel_comp_size 0x10000000;setenv fdt_addr_r 0x88000000;setenv ramdisk_addr_r 0x88300000;
```

**Step2**: load and excute:

```
run mmcsetup; run fdtsetup; run fatenv; echo 'running boot2...'; run boot2
```

**Step3**:

When you see the `buildroot login:` message, then congratulations, the launch was successful

```
buildroot login:root
Password: starfive
```

# Run Product_test on Board

**Step1**:

1. Before excute test_script, you must pulg HDMI displayer/USB disk/ethernet cable.

2. Make sure ETH0 connect to PC/VM through a network cable, the PC/VM IP and eth0 IP are on the same network segment, PC/VM open the iperf3 service, such as **iperf3 -s**
3. Make sure the different network card of PC/VM connect to router, PC/VM open the iperf3 service, such as **iperf3 -s**

**Step2**:

run test_script after start kernel

```
cd /test_script && chmod 777 * && sh product_test.sh
```

**Step3**:

when testing HDMI AND PWMADC, you must input the result **y/n**, base on tester to watch displayer and to listen, output as follows

```
******************HDMI PWMADC testing...
Playing WAVE 'audio8k16S.wav' : Signed 16 bit Little Endian, Rate 8000 Hz, Stereo
Warning: rate is not accurate (requested = 8000Hz, got = 16000Hz)
         please, try the plug plugin 
please enter HDMI TEST OK(y/n?): y
please enter PWMADC TEST OK(y/n?): y
HDMI   PASS
PWMADC PASS
```

when testing BLUETOOTH, you must active the bluetooth device, so it can be scaned on, such as

*[NEW] Device B4:EE:25:EB:B8:E0 RAPOO BT3.0 Mouse*

BLUETOOTH test output as follows

```
Agent registered
[CHG] Controller 70:4A:0E:95:58:A3 Pairable: yes
[bluetooth]# scan on
Discovery started
[CHG] Controller 70:4A:0E:95:58:A3 Discovering: yes
[NEW] Device 55:90:EE:76:5A:1D 55-90-EE-76-5A-1D
[NEW] Device 74:85:06:30:21:99 74-85-06-30-21-99
[NEW] Device B4:EE:25:EC:D8:E0 RAPOO BT4.0 Mouse
[NEW] Device 70:1C:E7:10:3D:A2 DESKTOP-SF1I2R2
[NEW] Device B4:EE:25:EB:B8:E0 RAPOO BT3.0 Mouse
[CHG] Device B4:EE:25:EB:B8:E0 UUIDs: 00001200-0000-1000-8000-00805f9b34fb
[CHG] Device B4:EE:25:EB:B8:E0 UUIDs: 00000100-0000-1000-8000-00805f9b34fb
[CHG] Device B4:EE:25:EB:B8:E0 UUIDs: 00000001-0000-1000-8000-00805f9b34fb
[CHG] Device B4:EE:25:EB:B8:E0 UUIDs: 00000011-0000-1000-8000-00805f9b34fb
[CHG] Device B4:EE:25:EB:B8:E0 UUIDs: 00001124-0000-1000-8000-00805f9b34fb
[CHG] Device B4:EE:25:EC:D8:E0 RSSI: -50
scan_over: 1
Attempting to pair with B4:EE:25:EB:B8:E0
[CHG] Device B4:EE:25:EB:B8:E0 Connected: yes
[CHG] Device B4:EE:25:EB:B8:E0 Modalias: usb:vAE24p8618d0001
[CHG] Device B4:EE:25:EB:B8:E0 UUIDs: 00001124-0000-1000-8000-00805f9b34fb
[CHG] Device B4:EE:25:EB:B8:E0 UUIDs: 00001200-0000-1000-8000-00805f9b34fb
[CHG] Device B4:EE:25:EB:B8:E0 ServicesResolved: yes
[CHG] Device B4:EE:25:EB:B8:E0 Paired: yes
Pairing successful
[CHG] Device B4:EE:25:EB:B8:E0 Trusted: yes
Changing B4:EE:25:EB:B8:E0 trust succeeded
[  430.724685] hid-generic 0005:AE24:8618.0001: unknown main item tag 0x0
[  430.741609] input: RAPOO BT3.0 Mouse as /devices/platform/soc/11870000.serial/tty/ttyS1/hci0/hci0:12/0005:AE24:8618.0001/input/input0
[  430.762868] hid-generic 0005:AE24:8618.0001: input: BLUETOOTH HID v0.01 Mouse [RAPOO BT3.0 Mouse] on 70:4a:0e:95:58:a3
Attempting to connect to B4:EE:25:EB:B8:E0
Connection successful
result: 
result: Connection successful
suc_flag: 1
BLUETOOTH PASS
```

**Step4**:

the result of test_script as follows

```
************************************************
*********************Result*********************
************************************************
USB1:             FAIL  speed: 8.47 Mbit/s
USB2:             FAIL  speed: 9.73 Mbit/s
USB3:             FAIL  speed: 10.94 Mbit/s
SD:             PASS  speed: 9.98 Mbit/s
ETH0 PING:      PASS
ETH0 TX:        PASS  tx speed:   416 Mbits/sec  
ETH0 RX:        PASS  rx speed:   416 Mbits/sec  
HDMI:           PASS
PWMADC:         PASS
BLUETOOTH:      PASS
WLAN0 PING:     PASS
WLAN0 TX:         FAIL  tx speed:  1.92 Mbits/sec  
WLAN0 RX:         FAIL  rx speed:  1.82 Mbits/sec  
```



# Enter MAC and SN

**Step1**: excute the command to enter Mac and SN

```
cd /test_script && sh enter_mac_sn.sh
```

**Step2**: enter SN, must capital letter(A~Z) or numbers or "-"

such as:

```
VF7100A1-21W50-D8E0-0000001
```

**Step3**: enter MAC, must capital letter(A~F) or numbers or "-"

such as

```
11-22-33-44-55-66
```

the output as follows

```
# sh enter_mac_sn.sh 
************************************************
****************enter mac and sn****************
************************************************
please enter sn(XXXXXXXX-XXXXX-XXXX-XXXXXXX): VF7100A1-21W50-D8E0-0000001ABC-43354
sn_len:36
please enter ETH0 mac addr(XX-XX-XX-XX-XX-XX): 11-22-33-44-55-66
SN:VF7100A1-21W50-D8E0-0000001ABC-43354;
ETH0:11-22-33-44-55-66;
```

