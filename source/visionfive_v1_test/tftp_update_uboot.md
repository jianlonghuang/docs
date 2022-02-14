# tftp_update_uboot

ALL the steps take 1 minute

## Start Tftpd32 server

PC IP: 192.168.120.33

1. Open **[Tftpd32.exe](https://github.com/jianlonghuang/docs/blob/master/source/visionfive_v1_test/tools/tftpd32.exe)**

2. Choose the file **[fw_payload_visionfive_e2_1230.bin.out](https://github.com/jianlonghuang/docs/blob/master/source/visionfive_v1_test/tools/fw_payload_visionfive_e2_1230.bin.out)** path 

   

## Update fw_payload_visionfive_e2_1230.bin.out

1. Turn on the power, Press 'v' if you see the infomation as follows:

   `Autoboot in 2 seconds`

2. erase environment parameter:

   `eraseenv`

2. Set enviroment parameter:

   `setenv ipaddr 192.168.120.200;setenv serverip 192.168.120.33`

3. Flash detect

   `sf probe`

4. Tftpboot fw_payload_visionfive_e2_1230.bin.out to ddr

   `tftpboot 0x90000000 192.168.120.33:fw_payload_visionfive_e2_1230.bin.out`

   the infomation for example as follows:

   > VisionFive #tftpboot 0x90000000 192.168.120.33:fw_payload_visionfive_e2_1230.bin.out
   > Speed: 1000, full duplex
   > Using dwmac.10020000 device
   > TFTP from server 192.168.120.33; our IP address is 192.168.120.200
   > Filename 'fw_payload_visionfive_e2_1230.bin.out'.
   > Load address: 0x90000000
   > Loading: ##################################################  2.9 MiB
   >          4.1 MiB/s
   > done
   > Bytes transferred = 3025228 (2e294c hex)

5. Update flash

   `sf update 0x90000000 0x40000 0x2E2C00`

   > 0x2E2C00: the file fw_pay_load_visionfive_e2_1230.bin.out size; 
   >
   > ​					you can get from step 4 (2e294c hex), 1k align

6. reboot the board

6. check opensbi and uboot version

   > bootloader version:211102-0b86f96
   > ddr 0x00000000, 1M test
   > ddr 0x00100000, 2M test
   > DDR clk 2133M,Version: 211102-d086aee
   > 0 crc flash: 5595e732, crc ddr: 5595e732
   > crc check PASSED
   >
   > bootloader.
   >
   > **OpenSBI v1.0**
   >
   > ____                    _____ ____ _____
   >   / __ \                  / ____|  _ \_   _|
   >  | |  | |_ __   ___ _ __ | (___ | |_) || |
   >  | |  | | '_ \ / _ \ '_ \ \___ \|  _ < | |
   >  | |__| | |_) |  __/ | | |____) | |_) || |_
   >   \____/| .__/ \___|_| |_|_____/|____/_____|
   >         | |
   >         |_|
   >
   > Platform Name             : StarFive VisionFive V1
   > Platform Features         : medeleg
   > Platform HART Count       : 2
   > Platform IPI Device       : aclint-mswi
   > Platform Timer Device     : aclint-mtimer @ 6250000Hz
   > Platform Console Device   : uart8250
   > Platform HSM Device       : ---
   > Platform Reboot Device    : ---
   > Platform Shutdown Device  : ---
   > Firmware Base             : 0x80000000
   > Firmware Size             : 296 KB
   > Runtime SBI Version       : 0.3
   >
   > Domain0 Name              : root
   > Domain0 Boot HART         : 1
   > Domain0 HARTs             : 0*,1*
   > Domain0 Region00          : 0x0000000002000000-0x000000000200ffff (I)
   > Domain0 Region01          : 0x0000000080000000-0x000000008007ffff ()
   > Domain0 Region02          : 0x0000000000000000-0xffffffffffffffff (R,W,X)
   > Domain0 Next Address      : 0x0000000080200000
   > Domain0 Next Arg1         : 0x0000000082200000
   > Domain0 Next Mode         : S-mode
   > Domain0 SysReset          : yes
   >
   > Boot HART ID              : 1
   > Boot HART Domain          : root
   > Boot HART ISA             : rv64imafdcsux
   > Boot HART Features        : scounteren,mcounteren
   > Boot HART PMP Count       : 16
   > Boot HART PMP Granularity : 4096
   > Boot HART PMP Address Bits: 36
   > Boot HART MHPM Count      : 2
   > Boot HART MIDELEG         : 0x0000000000000222
   > Boot HART MEDELEG         : 0x000000000000b109
   >
   > **U-Boot 2022.01-rc4-VisionFive-g0c08d335c5 (Dec 30 2021 - 08:30:15 +0800)StarFive**
   >
   > CPU:   rv64imafdc
   > Model: StarFive VisionFive V1
   > DRAM:  8 GiB
   > MMC:   mmc@10000000: 0, mmc@10010000: 1
   > Loading Environment from SPIFlash... cadence_spi spi@11860000: Can't get reset: -524
   > SF: Detected gd25lq128 with page size 256 Bytes, erase size 4 KiB, total 16 MiB
   > OK
   > StarFive EEPROM format v1
   >
   > --------EEPROM INFO--------
   > Vendor : StarFive Technology Co., Ltd.
   > Product full SN: VF7100A1-2160-D008E000-01001001;
   > data version: 0x1
   > PCB revision: 0x1
   > BOM revision: 
   > Ethernet MAC address: 82:be:6e:02:48:15
   > --------EEPROM INFO--------
   >
   > In:    serial@12440000
   > Out:   serial@12440000
   > Err:   serial@12440000
   > Net:   dwmac.10020000
   > MMC CD is 0x1, force to True.
   > MMC CD is 0x1, force to True.
   > switch to partitions #0, OK
   > mmc0 is current device
   > MMC CD is 0x1, force to True.
   > MMC CD is 0x1, force to True.
   > Failed to load 'uEnv.txt'
   > Failed to load '/boot/uEnv.txt'
   > Autoboot in 2 seconds
   
   





