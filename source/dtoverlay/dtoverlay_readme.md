# usdk support dtoverlay #

## Modify

- **support dtoverlay should modify**

```
cp 0001-add-overlay-config.patch freelight-u-sdk/
cd freelight-u-sdk/
git am 0001-add-overlay-config.patch

cp 0001-add-configfs-driver.patch freelight-u-sdk/linux
cd linux
git am 0001-add-configfs-driver.patch
```

- **test dtoverlay should modify**

```
cp 0001-dtoverlay-test.patch freelight-u-sdk/linux
cd linux
git am 0001-dtoverlay-test.patch
```

> Note: 
>
> [0001-add-overlay-config.patch](https://github.com/jianlonghuang/docs/blob/master/source/dtoverlay/0001-add-overlay-config.patch)
>
> [0001-add-configfs-driver.patch](https://github.com/jianlonghuang/docs/blob/master/source/dtoverlay/0001-add-configfs-driver.patch)
>
> [0001-dtoverlay-test.patch](https://github.com/jianlonghuang/docs/blob/master/source/dtoverlay/0001-dtoverlay-test.patch)



## Upload dtoverlay

1. **write a dtoverlay file**

   > for example:
   >
   > [dtoverlay.dts](https://github.com/jianlonghuang/docs/blob/master/source/dtoverlay/dtoverlay.dts)

2. **compile dts**

   ```
   cpp -I /home/jianlong.huang/usdk/test/freelight-u-sdk/linux/include/ -I .  -E -P -x assembler-with-cpp dtoverlay.dts -o dtoverlay.dts.pre
   
   ../freelight-u-sdk/work/linux/scripts/dtc/dtc -I dts -O dtb -o dtoverlay.dtbo dtoverlay.dts.pre
   ```

   > Note:
   >
   > 1. Path should modify according to actual path
   >
   >    /home/jianlong.huang/usdk/test/freelight-u-sdk/linux/include/: linux header file path
   >
   >    ../freelight-u-sdk/work/linux/scripts/dtc/dtc: dtc tool path
   >
   > 2. output file: dtoverlay.dtbo
   >
   > 3. There are many warnings during compiling dts, as follows, just ignore them

   ```
   jianlong.huang@soft03:~/usdk/test/dtoverlay$ ../freelight-u-sdk/work/linux/scripts/dtc/dtc -I dts -O dtb -o dtoverlay.dtbo dtoverlay.dts.pre
   dtoverlay.dts.pre:71.5-15: Warning (reg_format): /fragment@3/__overlay__/spi@0:reg: property has invalid length (4 bytes) (#address-cells == 2, #size-cells == 1)
   dtoverlay.dtbo: Warning (pci_device_reg): Failed prerequisite 'reg_format'
   dtoverlay.dtbo: Warning (pci_device_bus_num): Failed prerequisite 'reg_format'
   dtoverlay.dtbo: Warning (simple_bus_reg): Failed prerequisite 'reg_format'
   dtoverlay.dtbo: Warning (i2c_bus_reg): Failed prerequisite 'reg_format'
   dtoverlay.dtbo: Warning (spi_bus_reg): Failed prerequisite 'reg_format'
   dtoverlay.dts.pre:68.20-72.6: Warning (avoid_default_addr_size): /fragment@3/__overlay__/spi@0: Relying on default #address-cells value
   dtoverlay.dts.pre:68.20-72.6: Warning (avoid_default_addr_size): /fragment@3/__overlay__/spi@0: Relying on default #size-cells value
   dtoverlay.dtbo: Warning (avoid_unnecessary_addr_size): Failed prerequisite 'avoid_default_addr_size'
   dtoverlay.dtbo: Warning (unique_unit_address): Failed prerequisite 'avoid_default_addr_size'
   ```

3. **copy dtoverlay.dtbo to the board**

4. **upload dtoverlay**

   execute the following command to upload dtoverlay

   ```
   mount -t configfs none /sys/kernel/config
   mkdir -p /sys/kernel/config/device-tree/overlays/dtoverlay
   cd <the dtoverlay.dtbo path>
   cat dtoverlay.dtbo > /sys/kernel/config/device-tree/overlays/dtoverlay/dtbo
   ```

   for example:

   ```
   # cat dtoverlay.dtbo > /sys/kernel/config/device-tree/overlays/dtoverlay/dtbo
   OF: overlay: WARNING: memory leak will occur if overlay removed, property: /soc/i2c@118c0000/clock-frequency
   OF: overlay: WARNING: memory leak will occur if overlay removed, property: /soc/i2c@118c0000/i2c-sda-hold-time-ns
   OF: overlay: WARNING: memory leak will occur if overlay removed, property: /soc/i2c@118c0000/i2c-sda-falling-time-ns
   OF: overlay: WARNING: memory leak will occur if overlay removed, property: /soc/i2c@118c0000/i2c-scl-falling-time-ns
   OF: overlay: WARNING: memory leak will occur if overlay removed, property: /soc/i2c@118c0000/pinctrl-names
   OF: overlay: WARNING: memory leak will occur if overlay removed, property: /soc/i2c@118c0000/pinctrl-0
   OF: overlay: WARNING: memory leak will occur if overlay removed, property: /soc/i2c@118c0000/status
   OF: overlay: WARNING: memory leak will occur if overlay removed, property: /soc/spi@12410000/pinctrl-names
   OF: overlay: WARNING: memory leak will occur if overlay removed, property: /soc/spi@12410000/pinctrl-0
   OF: overlay: WARNING: memory leak will occur if overlay removed, property: /soc/spi@12410000/status
   OF: overlay: WARNING: memory leak will occur if overlay removed, property: /soc/pwm@12490000/pinctrl-names
   OF: overlay: WARNING: memory leak will occur if overlay removed, property: /soc/pwm@12490000/pinctrl-0
   OF: overlay: WARNING: memory leak will occur if overlay removed, property: /soc/pwm@12490000/status
   i2c_designware 118c0000.i2c: coherent device 0 dev->dma_coherent 0
   dw_spi_mmio 12410000.spi: coherent device 0 dev->dma_coherent 0
   dw_spi_mmio 12410000.spi: DMA init failed
   pwm-sifive-ptc 12490000.pwm: coherent device 0 dev->dma_coherent 0
   ```



## Test

- **PWM**

  for example:

  ```
  # cd /sys/class/pwm/pwmchip0/
  # echo 0 > export 
  sifive_pwm_ptc_get_state in,no:0....
  data_hrc:0x7d0 0xfa0 
  period:40000
  duty_cycle:20000
  polarity:0
  enabled:1
  # cd pwm0/
  # cat period 
  1000000
  # cat duty_cycle 
  20000
  # echo 500000 > duty_cycle 
  # echo 1000000 > period 
  # echo 10000000 > period 
  ```

  > Note:
  >
  > 1. use oscilloscope to check period and duty_cycle
  > 2. support 8 pwm

- **SPI2**

  copy spidev_test_510 to the board, short connect miso and mosi, excute the follow commands

  ```
  # chmod 777 spidev_test_510
  # ls /dev/spi*
  /dev/spidev0.0
  # ./spidev_test_510 -D /dev/spidev0.0 -v -p string_to_send
  mode = 0
  spi mode: 0x0
  bits per word: 8
  max speed: 100000 Hz (100 KHz)
  TX | 73 74 72 69 6E 67 5F 74 6F 5F 73 65 6E 64 __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __  | string_to_send
  RX | 73 74 72 69 6E 67 5F 74 6F 5F 73 65 6E 64 __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __  | string_to_send
  ```

- **I2C1**

  connect hal sensors to 40pins i2c interface, there are i2c0 and i2c1 before upload dtoverlay

  after upload dtoverlay, add i2c2, detect i2c2 bus as follow:

  ```
  # i2cdetect -y -r 2
       0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
  
  00:                         -- -- -- -- -- -- -- -- 
  10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
  20: -- -- -- -- -- -- -- -- -- 29 -- -- -- -- -- -- 
  30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
  40: -- -- -- -- -- -- -- -- 48 -- -- -- -- -- -- -- 
  50: -- -- -- -- -- -- -- -- -- -- -- -- 5c -- -- -- 
  60: -- -- -- -- -- -- -- -- 68 -- -- -- -- -- -- -- 
  70: 70 -- -- -- -- -- -- --         
  ```
