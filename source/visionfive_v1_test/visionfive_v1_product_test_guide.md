# Fetch code

https://github.com/starfive-tech/freelight-u-sdk.git

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



*fetch code instruction: more detail refer to [README](https://github.com/starfive-tech/freelight-u-sdk/tree/JH7100_starlight_multimedia)*

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

freelight-u-sdk//Makefile

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

modify **cfg.ini** as required

**Step3**:

copy to tf card  /dev/sdX3

```
sudo cp -r test_script/ /media/jianlong/465a8d4f-31fe-40c3-951e-4a565bd3a620/ && sync
```

Note: /media/jianlong/465a8d4f-31fe-40c3-951e-4a565bd3a620/ is mounted path of /dev/sdX3

**Step4**:

run test_script

```
cd /test_script && chmod 777 * && sh product_test.sh
```



# Running on Board

*more detail refer to [README](https://github.com/starfive-tech/freelight-u-sdk/tree/JH7100_starlight_multimedia)*

Step1: set enviroment parameter:

```
setenv fdt_high 0xffffffffffffffff;setenv initrd_high 0xffffffffffffffff;setenv scriptaddr 0x88100000;setenv script_offset_f 0x1fff000;setenv script_size_f 0x1000;setenv kernel_addr_r 0x84000000;setenv kernel_comp_addr_r 0x90000000;setenv kernel_comp_size 0x10000000;setenv fdt_addr_r 0x88000000;setenv ramdisk_addr_r 0x88300000;
```

Step2: load and excute:

```
run mmcsetup; run fdtsetup; run fatenv; echo 'running boot2...'; run boot2
```



