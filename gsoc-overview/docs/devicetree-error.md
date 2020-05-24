# Problem Description:

I try to port CTAG Face drivers from kernel v 4.4 to 4.19 but got a problem while loading the device tree overlays. When loading the .dtbo with U-boot I get the FDT_ERR_NOTFOUND error. The complete error log can be found in this document.

Steps taken to compile and start kernel 4.19-rt on BBB:

- Modified kernel source can be found here: https://github.com/NiklasWan/linux/tree/dev_ctag_4.19-rt

# Steps on development machine
- Flash current image to microsd card (AM3358 Debian 10.3 2020-04-06 4GB SD IoT)

- Generate BeagleBone Kernelconfiguration:
```bash
    make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- bb.org_defconifg
```
- Set CTAG SoundCard Compilation Flag in Menuconfig:
```bash
    make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- menuconfig
```
- Build Kernel:
```bash
    make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j8
```
- Build uImage and DTBs:
```bash
    make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- uImage dtbs LOADADDR=0x80008000 -j8
```
- Compile Modules:
```bash
    make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- modules -j8
```
- Install Modules to tmp folder:
```bash
    make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- INSTALL_MOD_PATH=modules_tmp modules_install
```
- Copy kernel to sd card:
```bash
    sudo cp arch/arm/boot/zImage /media/dev/rootfs/boot/vmlinuz-4.19.94+
```
- Copy modules to sd card:
```bash
    sudo cp -r modules_tmp/lib /media/dev/rootfs/
```
- Copy generated Device Tree Blobs to sd card:
```bash
    sudo cp -r arch/arm/boot/dts/*.dtb /media/dev/rootfs/boot/dtbs/4.19.94+/
```

- Copy kernel config to sd card:
```bash
    sudo cp .config /media/dev/rootfs/boot/config-4.19.94+
```

- Open /media/dev/rootfs/boot/uEnv.txt in your favourite editor and change name_r=4.19.94-ti-r42 to uname_r=4.19.94+

# Next steps need to be done directly on BBB:
- Clone Overlay Repository:
```bash
    git clone https://github.com/beagleboard/bb.org-overlays.git
```
- Install DTC and DTBOs to /lib/firmware:
```bash
    cd ./bb.org_overlays
    ./dtc-overlay.sh
    ./install.sh
```
- Reboot
- Change /boot/uEnv.txt to the following:
```
    #Docs: http://elinux.org/Beagleboard:U-boot_partitioning_layout_2.0

    uname_r=4.19.94+
    #uuid=
    dtb=/boot/dtbs/4.19.94+/am335x-boneblack-ctag-face.dtb

    ###U-Boot Overlays###
    ###Documentation: http://elinux.org/Beagleboard:BeagleBoneBlack_Debian#U-Boot_Overlays
    ###Master Enable
    enable_uboot_overlays=1
    ###
    ###Overide capes with eeprom
    #uboot_overlay_addr0=/lib/firmware/<file0>.dtbo
    #uboot_overlay_addr1=/lib/firmware/<file1>.dtbo
    #uboot_overlay_addr2=/lib/firmware/<file2>.dtbo
    #uboot_overlay_addr3=/lib/firmware/<file3>.dtbo
    ###
    ###Additional custom capes
    #uboot_overlay_addr4=/lib/firmware/<file4>.dtbo
    #uboot_overlay_addr5=/lib/firmware/<file5>.dtbo
    #uboot_overlay_addr6=/lib/firmware/<file6>.dtbo
    #uboot_overlay_addr7=/lib/firmware/<file7>.dtbo
    ###
    ###Custom Cape
    dtb_overlay=/lib/firmware/BB-CTAG-SW-8CH-00A0.dtbo
    ###
    ###Disable auto loading of virtual capes (emmc/video/wireless/adc)
    #disable_uboot_overlay_emmc=1
    #disable_uboot_overlay_video=1
    #disable_uboot_overlay_audio=1
    #disable_uboot_overlay_wireless=1
    #disable_uboot_overlay_adc=1
    ###
    ###PRUSS OPTIONS
    ###pru_rproc (4.14.x-ti kernel)
    #uboot_overlay_pru=/lib/firmware/AM335X-PRU-RPROC-4-14-TI-00A0.dtbo
    ###pru_rproc (4.19.x-ti kernel)
    uboot_overlay_pru=/lib/firmware/AM335X-PRU-RPROC-4-19-TI-00A0.dtbo
    ###pru_uio (4.14.x-ti, 4.19.x-ti & mainline/bone kernel)
    #uboot_overlay_pru=/lib/firmware/AM335X-PRU-UIO-00A0.dtbo
    ###
    ###Cape Universal Enable
    enable_uboot_cape_universal=1
    ###
    ###Debug: disable uboot autoload of Cape
    #disable_uboot_overlay_addr0=1
    #disable_uboot_overlay_addr1=1
    #disable_uboot_overlay_addr2=1
    #disable_uboot_overlay_addr3=1
    ###
    ###U-Boot fdt tweaks... (60000 = 384KB)
    #uboot_fdt_buffer=0x60000
    ###U-Boot Overlays###

    cmdline=coherent_pool=1M net.ifnames=0 lpj=1990656 rng_core.default_quality=100 quiet

    #In the event of edid real failures, uncomment this next line:
    #cmdline=coherent_pool=1M net.ifnames=0 lpj=1990656 rng_core.default_quality=100 quiet video=HDMI-A-1:1024x768@60e

    ##enable Generic eMMC Flasher:
    ##make sure, these tools are installed: dosfstools rsync
    #cmdline=init=/opt/scripts/tools/eMMC/init-eMMC-flasher-v3.sh
```
- Reboot

# U-Boot Error Log:
```
Board: BeagleBone Black                                                         
<ethaddr> not set. Validating first E-fuse MAC                                  
BeagleBone Black:                                                               
BeagleBone: cape eeprom: i2c_probe: 0x54:                                       
BeagleBone: cape eeprom: i2c_probe: 0x55:                                       
BeagleBone: cape eeprom: i2c_probe: 0x56:                                       
BeagleBone: cape eeprom: i2c_probe: 0x57:                                       
Net:   eth0: MII MODE                                                           
cpsw, usb_ether                                                                 
Press SPACE to abort autoboot in 0 seconds                                      
board_name=[A335BNLT] ...                                                       
board_rev=[00C0] ...                                                            
switch to partitions #0, OK                                                     
mmc0 is current device                                                          
SD/MMC found on device 0
switch to partitions #0, OK                                                     
mmc0 is current device                                                          
Scanning mmc 0:1...                                                             
gpio: pin 56 (gpio 56) value is 0                                               
gpio: pin 55 (gpio 55) value is 0                                               
gpio: pin 54 (gpio 54) value is 0                                               
gpio: pin 53 (gpio 53) value is 1                                               
switch to partitions #0, OK                                                     
mmc0 is current device                                                          
gpio: pin 54 (gpio 54) value is 1                                               
Checking for: /uEnv.txt ...                                                     
Checking for: /boot.scr ...                                                     
Checking for: /boot/boot.scr ...                                                
Checking for: /boot/uEnv.txt ...                                                
gpio: pin 55 (gpio 55) value is 1                                               
2116 bytes read in 51 ms (40 KiB/s)                                             
Loaded environment from /boot/uEnv.txt                                          
debug: [dtb=/boot/dtbs/4.19.94+/am335x-boneblack-ctag-face.dtb] ...             
Using: dtb=/boot/dtbs/4.19.94+/am335x-boneblack-ctag-face.dtb ...               
Checking if uname_r is set in /boot/uEnv.txt...                                 
gpio: pin 56 (gpio 56) value is 1                                               
Running uname_boot ...                                                          
loading /boot/vmlinuz-4.19.94+ ...
10141736 bytes read in 938 ms (10.3 MiB/s)                                      
debug: [enable_uboot_overlays=1] ...                                            
debug: [enable_uboot_cape_universal=1] ...                                      
debug: [uboot_base_dtb_univ=am335x-boneblack-uboot-univ.dtb] ...                
uboot_overlays: [uboot_base_dtb=am335x-boneblack-uboot-univ.dtb] ...            
uboot_overlays: Switching too: dtb=am335x-boneblack-uboot-univ.dtb ...          
loading /boot/dtbs/4.19.94+/am335x-boneblack-uboot-univ.dtb ...                 
162266 bytes read in 117 ms (1.3 MiB/s)                                         
uboot_overlays: [fdt_buffer=0x60000] ...                                        
uboot_overlays: loading /lib/firmware/BB-ADC-00A0.dtbo ...                      
867 bytes read in 281 ms (2.9 KiB/s)                                            
uboot_overlays: loading /lib/firmware/BB-BONE-eMMC1-01-00A0.dtbo ...            
1584 bytes read in 572 ms (2 KiB/s)                                             
uboot_overlays: loading /lib/firmware/BB-HDMI-TDA998x-00A0.dtbo ...             
4915 bytes read in 440 ms (10.7 KiB/s)                                          
uboot_overlays: loading /lib/firmware/AM335X-PRU-RPROC-4-19-TI-00A0.dtbo ...    
3801 bytes read in 784 ms (3.9 KiB/s)                                           
uboot_overlays: [dtb_overlay=/lib/firmware/BB-CTAG-SW-8CH-00A0.dtbo] ...        
uboot_overlays: loading /lib/firmware/BB-CTAG-SW-8CH-00A0.dtbo ...              
3393 bytes read in 856 ms (2.9 KiB/s)                                           
failed on fdt_overlay_apply(): FDT_ERR_NOTFOUND                                 
loading /boot/initrd.img-4.19.94+ ...
5302052 bytes read in 522 ms (9.7 MiB/s)                                        
debug: [console=ttyO0,115200n8 bone_capemgr.uboot_capemgr_enabled=1 root=/dev/m.
debug: [bootz 0x82000000 0x88080000:50e724 88000000] ...                        
ERROR: Did not find a cmdline Flattened Device Tree                             
Could not find a valid device tree                                              
** Invalid partition 2 **                                                       
** Invalid partition 3 **                                                       
** Invalid partition 4 **                                                       
** Invalid partition 5 **                                                       
** Invalid partition 6 **                                                       
** Invalid partition 7 **
```

# Fdtdump Outputs:

## Output for BB-CTAG-SW-8CH-00A0.dtbo

```
    /dts-v1/;
    // magic:		0xd00dfeed
    // totalsize:		0xd41 (3393)
    // off_dt_struct:	0x38
    // off_dt_strings:	0xb18
    // off_mem_rsvmap:	0x28
    // version:		17
    // last_comp_version:	16
    // boot_cpuid_phys:	0x0
    // size_dt_strings:	0x229
    // size_dt_struct:	0xae0

    / {
        compatible = "ti,beaglebone", "ti,beaglebone-black", "ti,beaglebone-green";
        part-number = "BB-CTAG-SW-8CH";
        version = "00A0", "A0";
        exclusive-use = "P9.30", "P9.28", "P9.31", "P9.29", "P9.12", "P9.27", "P9.25", "P8.32", "P8.33", "P8.14", "P8.17", "mcasp0", "spi_gpio";
        fragment@0 {
            target-path = "/";
            __overlay__ {
                chosen {
                    overlays {
                        BB-CTAG-SW-8CH-00A0 = "Sat May 23 15:11:32 2020";
                    };
                };
            };
        };
        fragment@1 {
            target = <0xffffffff>;
            __overlay__ {
                P9_30_pinmux {
                    status = "disabled";
                };
                P9_28_pinmux {
                    status = "disabled";
                };
                P9_31_pinmux {
                    status = "disabled";
                };
                P9_29_pinmux {
                    status = "disabled";
                };
                P9_12_pinmux {
                    status = "disabled";
                };
                P9_27_pinmux {
                    status = "disabled";
                };
                P9_25_pinmux {
                    status = "disabled";
                };
                P8_32_pinmux {
                    status = "disabled";
                };
                P8_33_pinmux {
                    status = "disabled";
                };
                P8_14_pinmux {
                    status = "disabled";
                };
                P8_17_pinmux {
                    status = "disabled";
                };
            };
        };
        fragment@2 {
            target = <0xffffffff>;
            __overlay__ {
                pinmix_mcasp0_pins {
                    pinctrl-single,pins = <0x000001ac 0x00000194 0x000001a4 0x00000198 0x0000006f 0x70696e6d 0x5f737069 0x00000020 0x00000028 0x0000002c 0x0000006f 0x00000002 0x40330000 0xffffffff>;
                    phandle = <0x00000002>;
                };
                pinmux_audiocard_spi_pins {
                    pinctrl-single,pins = <0x000000dc 0x000000d4 0x00000003 0x00000002 0x66726167 0x00000004 0x5f5f6f76 0x00000008>;
                    phandle = <0x00000001>;
                };
            };
        };
        fragment@3 {
            target = <0xffffffff>;
            __overlay__ {
                pinctrl-names = "default";
                pinctrl-0 = <0x00000001>;
                status = "okay";
                #address-cells = <0x00000001>;
                #size-cells = <0x00000000>;
                ad193x@0 {
                    #address-cells = <0x00000001>;
                    #size-cells = <0x00000000>;
                    compatible = "analog,ad1938";
                    reg = <0x00000000>;
                    spi-max-frequency = <0x000186a0>;
                    phandle = <0x00000003>;
                };
            };
        };
        fragment@4 {
            target = <0xffffffff>;
            __overlay__ {
                pinctrl-names = "default";
                pinctrl-0 = <0x00000002>;
                status = "okay";
                op-mode = <0x00000000>;
                tdm-slots = <0x00000008>;
                num-serializer = <0x00000010>;
                serial-dir = <0x00000002 0x00000000 0x00000000 0x00000000 0x00000003 0x00000003 0x00000002 0x6d656e74 0x0000004d 0x65726c61 0x64000000 0x63746167 0x00000003 0x20666163 0x00000003 0x00000003>;
                tx-num-evt = <0x00000001>;
                rx-num-evt = <0x00000001>;
            };
        };
        fragment@5 {
            target = <0xffffffff>;
            __overlay__ {
                sound {
                    compatible = "ctag,face-2-4";
                    model = "CTAG face-2-4 8CH";
                    audio-codec = <0x00000003>;
                    mcasp-controller = <0xffffffff>;
                    audiocard-tdm-slots = <0x00000008>;
                    codec-clock-rate = <0x01770000>;
                    cpu-clock-rate = <0x01770000>;
                    audio-codec-bit-delay-dac = <0x00000001>;
                    audio-codec-bit-delay-adc = <0x00000001>;
                    mcasp-controller-bit-delay-tx = <0x00000001>;
                    mcasp-controller-bit-delay-rx = <0x00000001>;
                    bb-device = <0x00000000>;
                    audio-routing = "Line Out", "DAC1OUT", "Line Out", "DAC2OUT", "Line Out", "DAC3OUT", "Line Out", "DAC4OUT", "ADC1IN", "Line In", "ADC2IN", "Line In";
                };
            };
        };
        __symbols__ {
            mcasp0_pins = "/fragment@2/__overlay__/pinmix_mcasp0_pins";
            audiocard_spi_pins = "/fragment@2/__overlay__/pinmux_audiocard_spi_pins";
            ad193x = "/fragment@3/__overlay__/ad193x@0";
        };
        __fixups__ {
            ocp = "/fragment@1:target:0", "/fragment@5:target:0";
            am33xx_pinmux = "/fragment@2:target:0";
            spi_gpio = "/fragment@3:target:0";
            mcasp0 = "/fragment@4:target:0", "/fragment@5/__overlay__/sound:mcasp-controller:0";
        };
        __local_fixups__ {
            fragment@3 {
                __overlay__ {
                    pinctrl-0 = <0x00000000>;
                };
            };
            fragment@4 {
                __overlay__ {
                    pinctrl-0 = <0x00000000>;
                };
            };
            fragment@5 {
                __overlay__ {
                    sound {
                        audio-codec = <0x00000000>;
                    };
                };
            };
        };
    };
```

## Output for am335x-boneblack-ctag-face.dtb:
```
    /dts-v1/;
    // magic:		0xd00dfeed
    // totalsize:		0xe5ac (58796)
    // off_dt_struct:	0x38
    // off_dt_strings:	0xd364
    // off_mem_rsvmap:	0x28
    // version:		17
    // last_comp_version:	16
    // boot_cpuid_phys:	0x0
    // size_dt_strings:	0x1248
    // size_dt_struct:	0xd32c

    / {
        compatible = "ti,am335x-bone-black", "ti,am335x-bone", "ti,am33xx";
        interrupt-parent = <0x00000001>;
        #address-cells = <0x00000001>;
        #size-cells = <0x00000001>;
        model = "TI AM335x BeagleBone Black AudioCard";
        chosen {
            stdout-path = "/ocp/serial@44e09000";
            base_dtb = "am335x-boneblack-ctag-face.dts";
            base_dtb_timestamp = "Sun May 24 10:47:52 2020";
        };
        aliases {
            i2c0 = "/ocp/i2c@44e0b000";
            i2c1 = "/ocp/i2c@4802a000";
            i2c2 = "/ocp/i2c@4819c000";
            serial0 = "/ocp/serial@44e09000";
            serial1 = "/ocp/serial@48022000";
            serial2 = "/ocp/serial@48024000";
            serial3 = "/ocp/serial@481a6000";
            serial4 = "/ocp/serial@481a8000";
            serial5 = "/ocp/serial@481aa000";
            d-can0 = "/ocp/can@481cc000";
            d-can1 = "/ocp/can@481d0000";
            usb0 = "/ocp/usb@47400000/usb@47401000";
            usb1 = "/ocp/usb@47400000/usb@47401800";
            phy0 = "/ocp/usb@47400000/usb-phy@47401300";
            phy1 = "/ocp/usb@47400000/usb-phy@47401b00";
            ethernet0 = "/ocp/ethernet@4a100000/slave@4a100200";
            ethernet1 = "/ocp/ethernet@4a100000/slave@4a100300";
            spi0 = "/ocp/spi@48030000";
            spi1 = "/ocp/spi@481a0000";
            phandle = <0x00000060>;
        };
        cpus {
            #address-cells = <0x00000001>;
            #size-cells = <0x00000000>;
            cpu@0 {
                compatible = "arm,cortex-a8";
                enable-method = "ti,am3352";
                device_type = "cpu";
                reg = <0x00000000>;
                operating-points-v2 = <0x00000002>;
                clocks = <0x00000003>;
                clock-names = "cpu";
                clock-latency = <0x000493e0>;
                cpu-idle-states = <0x00000004>;
                cpu0-supply = <0x00000005>;
            };
            idle-states {
                mpu_gate {
                    compatible = "arm,idle-state";
                    entry-latency-us = <0x00000028>;
                    exit-latency-us = <0x0000005a>;
                    min-residency-us = <0x0000012c>;
                    ti,idle-wkup-m3;
                    phandle = <0x00000004>;
                };
            };
        };
        opp-table {
            compatible = "operating-points-v2-ti-cpu";
            syscon = <0x00000006>;
            phandle = <0x00000002>;
            opp50-300000000 {
                opp-hz = <0x00000000 0x000001ab>;
                opp-microvolt = <0x000e7ef0 0x00000008 0x00000003>;
                opp-supported-hw = <0x00000006 0x000001ca>;
                opp-suspend;
            };
            opp100-275000000 {
                opp-hz = <0x00000000 0x000001ab>;
                opp-microvolt = <0x0010c8e0 0x00000008 0x00000003>;
                opp-supported-hw = <0x00000001 0x000001ca>;
                opp-suspend;
            };
            opp100-300000000 {
                opp-hz = <0x00000000 0x000001ab>;
                opp-microvolt = <0x0010c8e0 0x00000008 0x00000003>;
                opp-supported-hw = <0x00000006 0x000001ca>;
                opp-suspend;
            };
            opp100-500000000 {
                opp-hz = <0x00000000 0x000001ab>;
                opp-microvolt = <0x0010c8e0 0x00000008 0x00000002>;
                opp-supported-hw = <0x00000001 0x6f707031>;
            };
            opp100-600000000 {
                opp-hz = <0x00000000 0x000001ab>;
                opp-microvolt = <0x0010c8e0 0x00000008 0x00000002>;
                opp-supported-hw = <0x00000006 0x6f707031>;
            };
            opp120-600000000 {
                opp-hz = <0x00000000 0x000001ab>;
                opp-microvolt = <0x00124f80 0x00000008 0x00000002>;
                opp-supported-hw = <0x00000001 0x6f707031>;
            };
            opp120-720000000 {
                opp-hz = <0x00000000 0x000001ab>;
                opp-microvolt = <0x00124f80 0x00000008 0x00000002>;
                opp-supported-hw = <0x00000006 0x6f707074>;
            };
            oppturbo-720000000 {
                opp-hz = <0x00000000 0x000001ab>;
                opp-microvolt = <0x001339e0 0x00000008 0x00000002>;
                opp-supported-hw = <0x00000001 0x6f707074>;
            };
            oppturbo-800000000 {
                opp-hz = <0x00000000 0x000001ab>;
                opp-microvolt = <0x001339e0 0x00000008 0x00000002>;
                opp-supported-hw = <0x00000006 0x6f70706e>;
            };
            oppnitro-1000000000 {
                opp-hz = <0x00000000 0x000001ab>;
                opp-microvolt = <0x001437c8 0x00000008 0x00000002>;
                opp-supported-hw = <0x00000004 0x6f70706e>;
            };
            oppnitro@1000000000 {
                opp-supported-hw = <0x00000006 0x00000001>;
            };
        };
        pmu@4b000000 {
            compatible = "arm,cortex-a8-pmu";
            interrupts = <0x00000003>;
            reg = <0x4b000000 0x000001e1>;
            ti,hwmods = "debugss";
        };
        soc {
            compatible = "ti,omap-infra";
            mpu {
                compatible = "ti,omap3-mpu";
                ti,hwmods = "mpu";
                pm-sram = <0x00000007 0x00000001>;
            };
        };
        ocp {
            compatible = "simple-bus";
            #address-cells = <0x00000001>;
            #size-cells = <0x00000001>;
            ranges;
            ti,hwmods = "l3_main";
            phandle = <0x00000061>;
            l4_wkup@44c00000 {
                compatible = "ti,am3-l4-wkup", "simple-bus";
                #address-cells = <0x00000001>;
                #size-cells = <0x00000001>;
                ranges = <0x00000000 0x00000004 0x776b7570>;
                phandle = <0x00000062>;
                wkup_m3@100000 {
                    compatible = "ti,am3352-wkup-m3";
                    reg = <0x00100000 0x00000003 0x00646d65 0x000001e1>;
                    reg-names = "umem", "dmem";
                    ti,hwmods = "wkup_m3";
                    ti,pm-firmware = "am335x-pm-firmware.elf";
                    phandle = <0x00000024>;
                };
                prcm@200000 {
                    compatible = "ti,am3-prcm", "simple-bus";
                    reg = <0x00200000 0x0000001c>;
                    #address-cells = <0x00000001>;
                    #size-cells = <0x00000001>;
                    ranges = <0x00000000 0x00000004 0x636c6f63>;
                    phandle = <0x00000063>;
                    clocks {
                        #address-cells = <0x00000001>;
                        #size-cells = <0x00000000>;
                        phandle = <0x00000064>;
                        clk_32768_ck {
                            #clock-cells = <0x00000000>;
                            compatible = "fixed-clock";
                            clock-frequency = <0x00008000>;
                            phandle = <0x00000018>;
                        };
                        clk_rc32k_ck {
                            #clock-cells = <0x00000000>;
                            compatible = "fixed-clock";
                            clock-frequency = <0x00007d00>;
                            phandle = <0x00000017>;
                        };
                        virt_19200000_ck {
                            #clock-cells = <0x00000000>;
                            compatible = "fixed-clock";
                            clock-frequency = <0x0124f800>;
                            phandle = <0x0000001f>;
                        };
                        virt_24000000_ck {
                            #clock-cells = <0x00000000>;
                            compatible = "fixed-clock";
                            clock-frequency = <0x016e3600>;
                            phandle = <0x00000020>;
                        };
                        virt_25000000_ck {
                            #clock-cells = <0x00000000>;
                            compatible = "fixed-clock";
                            clock-frequency = <0x017d7840>;
                            phandle = <0x00000021>;
                        };
                        virt_26000000_ck {
                            #clock-cells = <0x00000000>;
                            compatible = "fixed-clock";
                            clock-frequency = <0x018cba80>;
                            phandle = <0x00000022>;
                        };
                        tclkin_ck {
                            #clock-cells = <0x00000000>;
                            compatible = "fixed-clock";
                            clock-frequency = <0x00b71b00>;
                            phandle = <0x00000016>;
                        };
                        dpll_core_ck@490 {
                            #clock-cells = <0x00000000>;
                            compatible = "ti,am3-dpll-core-clock";
                            clocks = <0x00000009 0x00000106>;
                            reg = <0x00000490 0x00000004 0x00000001>;
                            phandle = <0x0000000a>;
                        };
                        dpll_core_x2_ck {
                            #clock-cells = <0x00000000>;
                            compatible = "ti,am3-dpll-x2-clock";
                            clocks = <0x0000000a>;
                            phandle = <0x0000000b>;
                        };
                        dpll_core_m4_ck@480 {
                            #clock-cells = <0x00000000>;
                            compatible = "ti,divider-clock";
                            clocks = <0x0000000b>;
                            ti,max-div = <0x0000001f>;
                            reg = <0x00000480>;
                            ti,index-starts-at-one;
                            phandle = <0x00000012>;
                        };
                        dpll_core_m5_ck@484 {
                            #clock-cells = <0x00000000>;
                            compatible = "ti,divider-clock";
                            clocks = <0x0000000b>;
                            ti,max-div = <0x0000001f>;
                            reg = <0x00000484>;
                            ti,index-starts-at-one;
                            phandle = <0x0000001a>;
                        };
                        dpll_core_m6_ck@4d8 {
                            #clock-cells = <0x00000000>;
                            compatible = "ti,divider-clock";
                            clocks = <0x0000000b>;
                            ti,max-div = <0x0000001f>;
                            reg = <0x000004d8>;
                            ti,index-starts-at-one;
                            phandle = <0x00000065>;
                        };
                        dpll_mpu_ck@488 {
                            #clock-cells = <0x00000000>;
                            compatible = "ti,am3-dpll-clock";
                            clocks = <0x00000009 0x00000106>;
                            reg = <0x00000488 0x00000004 0x00000001>;
                            phandle = <0x00000003>;
                        };
                        dpll_mpu_m2_ck@4a8 {
                            #clock-cells = <0x00000000>;
                            compatible = "ti,divider-clock";
                            clocks = <0x00000003>;
                            ti,max-div = <0x0000001f>;
                            reg = <0x000004a8>;
                            ti,index-starts-at-one;
                            phandle = <0x00000066>;
                        };
                        dpll_ddr_ck@494 {
                            #clock-cells = <0x00000000>;
                            compatible = "ti,am3-dpll-no-gate-clock";
                            clocks = <0x00000009 0x00000106>;
                            reg = <0x00000494 0x00000004 0x00000001>;
                            phandle = <0x0000000c>;
                        };
                        dpll_ddr_m2_ck@4a0 {
                            #clock-cells = <0x00000000>;
                            compatible = "ti,divider-clock";
                            clocks = <0x0000000c>;
                            ti,max-div = <0x0000001f>;
                            reg = <0x000004a0>;
                            ti,index-starts-at-one;
                            phandle = <0x0000000d>;
                        };
                        dpll_ddr_m2_div2_ck {
                            #clock-cells = <0x00000000>;
                            compatible = "fixed-factor-clock";
                            clocks = <0x0000000d>;
                            clock-mult = <0x00000001>;
                            clock-div = <0x00000002>;
                            phandle = <0x00000067>;
                        };
                        dpll_disp_ck@498 {
                            #clock-cells = <0x00000000>;
                            compatible = "ti,am3-dpll-no-gate-clock";
                            clocks = <0x00000009 0x00000106>;
                            reg = <0x00000498 0x00000004 0x00000001>;
                            phandle = <0x0000000e>;
                        };
                        dpll_disp_m2_ck@4a4 {
                            #clock-cells = <0x00000000>;
                            compatible = "ti,divider-clock";
                            clocks = <0x0000000e>;
                            ti,max-div = <0x0000001f>;
                            reg = <0x000004a4>;
                            ti,index-starts-at-one;
                            ti,set-rate-parent;
                            phandle = <0x00000014>;
                        };
                        dpll_per_ck@48c {
                            #clock-cells = <0x00000000>;
                            compatible = "ti,am3-dpll-no-gate-j-type-clock";
                            clocks = <0x00000009 0x00000106>;
                            reg = <0x0000048c 0x00000004 0x00000001>;
                            phandle = <0x0000000f>;
                        };
                        dpll_per_m2_ck@4ac {
                            #clock-cells = <0x00000000>;
                            compatible = "ti,divider-clock";
                            clocks = <0x0000000f>;
                            ti,max-div = <0x0000001f>;
                            reg = <0x000004ac>;
                            ti,index-starts-at-one;
                            phandle = <0x00000010>;
                        };
                        dpll_per_m2_div4_wkupdm_ck {
                            #clock-cells = <0x00000000>;
                            compatible = "fixed-factor-clock";
                            clocks = <0x00000010>;
                            clock-mult = <0x00000001>;
                            clock-div = <0x00000004>;
                            phandle = <0x00000068>;
                        };
                        dpll_per_m2_div4_ck {
                            #clock-cells = <0x00000000>;
                            compatible = "fixed-factor-clock";
                            clocks = <0x00000010>;
                            clock-mult = <0x00000001>;
                            clock-div = <0x00000004>;
                            phandle = <0x00000069>;
                        };
                        clk_24mhz {
                            #clock-cells = <0x00000000>;
                            compatible = "fixed-factor-clock";
                            clocks = <0x00000010>;
                            clock-mult = <0x00000001>;
                            clock-div = <0x00000008>;
                            phandle = <0x00000011>;
                        };
                        clkdiv32k_ck {
                            #clock-cells = <0x00000000>;
                            compatible = "fixed-factor-clock";
                            clocks = <0x00000011>;
                            clock-mult = <0x00000001>;
                            clock-div = <0x000002dc>;
                            phandle = <0x0000006a>;
                        };
                        l3_gclk {
                            #clock-cells = <0x00000000>;
                            compatible = "fixed-factor-clock";
                            clocks = <0x00000012>;
                            clock-mult = <0x00000001>;
                            clock-div = <0x00000001>;
                            phandle = <0x00000013>;
                        };
                        pruss_ocp_gclk@530 {
                            #clock-cells = <0x00000000>;
                            compatible = "ti,mux-clock";
                            clocks = <0x00000013 0x00000106>;
                            reg = <0x00000530>;
                            phandle = <0x0000006b>;
                        };
                        mmu_fck@914 {
                            #clock-cells = <0x00000000>;
                            compatible = "ti,gate-clock";
                            clocks = <0x00000012>;
                            ti,bit-shift = <0x00000001>;
                            reg = <0x00000914>;
                            phandle = <0x0000006c>;
                        };
                        timer1_fck@528 {
                            #clock-cells = <0x00000000>;
                            compatible = "ti,mux-clock";
                            clocks = <0x00000009 0x00000016 0x00000004 0x00000004 0x00000001 0x30380000 0x00000000>;
                            reg = <0x00000528>;
                            phandle = <0x00000034>;
                        };
                        timer2_fck@508 {
                            #clock-cells = <0x00000000>;
                            compatible = "ti,mux-clock";
                            clocks = <0x00000016 0x00000000 0x00000508 0x00000035 0x72335f66>;
                            reg = <0x00000508>;
                            phandle = <0x00000035>;
                        };
                        timer3_fck@50c {
                            #clock-cells = <0x00000000>;
                            compatible = "ti,mux-clock";
                            clocks = <0x00000016 0x00000000 0x0000050c 0x0000006d 0x72345f66>;
                            reg = <0x0000050c>;
                            phandle = <0x0000006d>;
                        };
                        timer4_fck@510 {
                            #clock-cells = <0x00000000>;
                            compatible = "ti,mux-clock";
                            clocks = <0x00000016 0x00000000 0x00000510 0x0000006e 0x72355f66>;
                            reg = <0x00000510>;
                            phandle = <0x0000006e>;
                        };
                        timer5_fck@518 {
                            #clock-cells = <0x00000000>;
                            compatible = "ti,mux-clock";
                            clocks = <0x00000016 0x00000000 0x00000518 0x0000006f 0x72365f66>;
                            reg = <0x00000518>;
                            phandle = <0x0000006f>;
                        };
                        timer6_fck@51c {
                            #clock-cells = <0x00000000>;
                            compatible = "ti,mux-clock";
                            clocks = <0x00000016 0x00000000 0x0000051c 0x00000070 0x72375f66>;
                            reg = <0x0000051c>;
                            phandle = <0x00000070>;
                        };
                        timer7_fck@504 {
                            #clock-cells = <0x00000000>;
                            compatible = "ti,mux-clock";
                            clocks = <0x00000016 0x00000000 0x00000504 0x00000071 0x74675f66>;
                            reg = <0x00000504>;
                            phandle = <0x00000071>;
                        };
                        usbotg_fck@47c {
                            #clock-cells = <0x00000000>;
                            compatible = "ti,gate-clock";
                            clocks = <0x0000000f>;
                            ti,bit-shift = <0x00000008>;
                            reg = <0x0000047c>;
                            phandle = <0x00000072>;
                        };
                        dpll_core_m4_div2_ck {
                            #clock-cells = <0x00000000>;
                            compatible = "fixed-factor-clock";
                            clocks = <0x00000012>;
                            clock-mult = <0x00000001>;
                            clock-div = <0x00000002>;
                            phandle = <0x00000019>;
                        };
                        ieee5000_fck@e4 {
                            #clock-cells = <0x00000000>;
                            compatible = "ti,gate-clock";
                            clocks = <0x00000019>;
                            ti,bit-shift = <0x00000001>;
                            reg = <0x000000e4>;
                            phandle = <0x00000073>;
                        };
                        wdt1_fck@538 {
                            #clock-cells = <0x00000000>;
                            compatible = "ti,mux-clock";
                            clocks = <0x00000017 0x00000003 0x00000003 0x00000002>;
                            reg = <0x00000538>;
                            phandle = <0x00000074>;
                        };
                        l4_rtc_gclk {
                            #clock-cells = <0x00000000>;
                            compatible = "fixed-factor-clock";
                            clocks = <0x00000012>;
                            clock-mult = <0x00000001>;
                            clock-div = <0x00000002>;
                            phandle = <0x00000075>;
                        };
                        l4hs_gclk {
                            #clock-cells = <0x00000000>;
                            compatible = "fixed-factor-clock";
                            clocks = <0x00000012>;
                            clock-mult = <0x00000001>;
                            clock-div = <0x00000001>;
                            phandle = <0x00000076>;
                        };
                        l3s_gclk {
                            #clock-cells = <0x00000000>;
                            compatible = "fixed-factor-clock";
                            clocks = <0x00000019>;
                            clock-mult = <0x00000001>;
                            clock-div = <0x00000001>;
                            phandle = <0x00000077>;
                        };
                        l4fw_gclk {
                            #clock-cells = <0x00000000>;
                            compatible = "fixed-factor-clock";
                            clocks = <0x00000019>;
                            clock-mult = <0x00000001>;
                            clock-div = <0x00000001>;
                            phandle = <0x00000078>;
                        };
                        l4ls_gclk {
                            #clock-cells = <0x00000000>;
                            compatible = "fixed-factor-clock";
                            clocks = <0x00000019>;
                            clock-mult = <0x00000001>;
                            clock-div = <0x00000001>;
                            phandle = <0x00000023>;
                        };
                        sysclk_div_ck {
                            #clock-cells = <0x00000000>;
                            compatible = "fixed-factor-clock";
                            clocks = <0x00000012>;
                            clock-mult = <0x00000001>;
                            clock-div = <0x00000001>;
                            phandle = <0x00000079>;
                        };
                        cpsw_125mhz_gclk {
                            #clock-cells = <0x00000000>;
                            compatible = "fixed-factor-clock";
                            clocks = <0x0000001a>;
                            clock-mult = <0x00000001>;
                            clock-div = <0x00000002>;
                            phandle = <0x0000003e>;
                        };
                        cpsw_cpts_rft_clk@520 {
                            #clock-cells = <0x00000000>;
                            compatible = "ti,mux-clock";
                            clocks = <0x0000001a 0x00000106>;
                            reg = <0x00000520>;
                            phandle = <0x0000003f>;
                        };
                        gpio0_dbclk_mux_ck@53c {
                            #clock-cells = <0x00000000>;
                            compatible = "ti,mux-clock";
                            clocks = <0x00000017 0x00000000 0x0000053c 0x0000007a 0x67636c6b>;
                            reg = <0x0000053c>;
                            phandle = <0x0000007a>;
                        };
                        lcd_gclk@534 {
                            #clock-cells = <0x00000000>;
                            compatible = "ti,mux-clock";
                            clocks = <0x00000014 0x00000004 0x00000000>;
                            reg = <0x00000534>;
                            ti,set-rate-parent;
                            phandle = <0x0000001c>;
                        };
                        mmc_clk {
                            #clock-cells = <0x00000000>;
                            compatible = "fixed-factor-clock";
                            clocks = <0x00000010>;
                            clock-mult = <0x00000001>;
                            clock-div = <0x00000002>;
                            phandle = <0x0000007b>;
                        };
                        gfx_fclk_clksel_ck@52c {
                            #clock-cells = <0x00000000>;
                            compatible = "ti,mux-clock";
                            clocks = <0x00000012 0x0000027a>;
                            ti,bit-shift = <0x00000001>;
                            reg = <0x0000052c>;
                            phandle = <0x0000001b>;
                        };
                        gfx_fck_div_ck@52c {
                            #clock-cells = <0x00000000>;
                            compatible = "ti,divider-clock";
                            clocks = <0x0000001b>;
                            reg = <0x0000052c>;
                            ti,max-div = <0x00000002>;
                            phandle = <0x00000048>;
                        };
                        sysclkout_pre_ck@700 {
                            #clock-cells = <0x00000000>;
                            compatible = "ti,mux-clock";
                            clocks = <0x00000018 0x0000001c 0x00000700 0x0000001d 0x7574325f>;
                            reg = <0x00000700>;
                            phandle = <0x0000001d>;
                        };
                        clkout2_div_ck@700 {
                            #clock-cells = <0x00000000>;
                            compatible = "ti,divider-clock";
                            clocks = <0x0000001d>;
                            ti,bit-shift = <0x00000003>;
                            ti,max-div = <0x00000008>;
                            reg = <0x00000700>;
                            phandle = <0x0000001e>;
                        };
                        clkout2_ck@700 {
                            #clock-cells = <0x00000000>;
                            compatible = "ti,gate-clock";
                            clocks = <0x0000001e>;
                            ti,bit-shift = <0x00000007>;
                            reg = <0x00000700>;
                            phandle = <0x0000007c>;
                        };
                    };
                    clockdomains {
                        phandle = <0x0000007d>;
                    };
                    l4_per_cm@0 {
                        compatible = "ti,omap4-cm";
                        reg = <0x00000000 0x0000001c>;
                        #address-cells = <0x00000001>;
                        #size-cells = <0x00000001>;
                        ranges = <0x00000000 0x00000004 0x636c6b40>;
                        phandle = <0x0000007e>;
                        clk@14 {
                            compatible = "ti,clkctrl";
                            reg = <0x00000014 0x00000213>;
                            #clock-cells = <0x00000002>;
                            phandle = <0x00000015>;
                        };
                    };
                    l4_wkup_cm@400 {
                        compatible = "ti,omap4-cm";
                        reg = <0x00000400 0x0000001c>;
                        #address-cells = <0x00000001>;
                        #size-cells = <0x00000001>;
                        ranges = <0x00000000 0x00000004 0x636c6b40>;
                        phandle = <0x0000007f>;
                        clk@4 {
                            compatible = "ti,clkctrl";
                            reg = <0x00000004 0x00000213>;
                            #clock-cells = <0x00000002>;
                            phandle = <0x00000080>;
                        };
                    };
                    mpu_cm@600 {
                        compatible = "ti,omap4-cm";
                        reg = <0x00000600 0x0000001c>;
                        #address-cells = <0x00000001>;
                        #size-cells = <0x00000001>;
                        ranges = <0x00000000 0x00000004 0x636c6b40>;
                        phandle = <0x00000081>;
                        clk@4 {
                            compatible = "ti,clkctrl";
                            reg = <0x00000004 0x00000213>;
                            #clock-cells = <0x00000002>;
                            phandle = <0x00000082>;
                        };
                    };
                    l4_rtc_cm@800 {
                        compatible = "ti,omap4-cm";
                        reg = <0x00000800 0x0000001c>;
                        #address-cells = <0x00000001>;
                        #size-cells = <0x00000001>;
                        ranges = <0x00000000 0x00000004 0x636c6b40>;
                        phandle = <0x00000083>;
                        clk@0 {
                            compatible = "ti,clkctrl";
                            reg = <0x00000000 0x00000213>;
                            #clock-cells = <0x00000002>;
                            phandle = <0x00000084>;
                        };
                    };
                    gfx_l3_cm@900 {
                        compatible = "ti,omap4-cm";
                        reg = <0x00000900 0x0000001c>;
                        #address-cells = <0x00000001>;
                        #size-cells = <0x00000001>;
                        ranges = <0x00000000 0x00000004 0x636c6b40>;
                        phandle = <0x00000085>;
                        clk@4 {
                            compatible = "ti,clkctrl";
                            reg = <0x00000004 0x00000213>;
                            #clock-cells = <0x00000002>;
                            phandle = <0x00000086>;
                        };
                    };
                    l4_cefuse_cm@a00 {
                        compatible = "ti,omap4-cm";
                        reg = <0x00000a00 0x0000001c>;
                        #address-cells = <0x00000001>;
                        #size-cells = <0x00000001>;
                        ranges = <0x00000000 0x00000004 0x636c6b40>;
                        phandle = <0x00000087>;
                        clk@20 {
                            compatible = "ti,clkctrl";
                            reg = <0x00000020 0x00000213>;
                            #clock-cells = <0x00000002>;
                            phandle = <0x00000088>;
                        };
                    };
                };
                scm@210000 {
                    compatible = "ti,am3-scm", "simple-bus";
                    reg = <0x00210000 0x0000001c>;
                    #address-cells = <0x00000001>;
                    #size-cells = <0x00000001>;
                    #pinctrl-cells = <0x00000001>;
                    ranges = <0x00000000 0x00000004 0x70696e6d>;
                    phandle = <0x00000089>;
                    pinmux@800 {
                        compatible = "pinctrl-single";
                        reg = <0x00000800 0x0000001c>;
                        #address-cells = <0x00000001>;
                        #size-cells = <0x00000000>;
                        #pinctrl-cells = <0x00000001>;
                        pinctrl-single,register-width = <0x00000020>;
                        pinctrl-single,function-mask = <0x0000007f>;
                        phandle = <0x0000008a>;
                        user_leds_s0 {
                            pinctrl-single,pins = <0x00000054 0x0000005c 0x00000003 0x00000002 0x3263305f 0x00000010 0x0000018c 0x000000e4>;
                            phandle = <0x0000005d>;
                        };
                        pinmux_i2c0_pins {
                            pinctrl-single,pins = <0x00000188 0x00000003 0x00000002 0x61727430>;
                            phandle = <0x0000002c>;
                        };
                        pinmux_uart0_pins {
                            pinctrl-single,pins = <0x00000170 0x00000003 0x00000002 0x61756c74>;
                            phandle = <0x0000002b>;
                        };
                        cpsw_default {
                            pinctrl-single,pins = <0x00000110 0x00000118 0x00000120 0x00000128 0x00000130 0x00000138 0x00000140 0x000000e4 0x63707377 0x00000068 0x00000114 0x0000011c 0x00000124 0x0000012c 0x00000134 0x0000013c 0x00000003 0x00000002 0x6d64696f 0x00000003 0x00000030 0x00000004 0x00000001 0x5f736c65 0x000002d1 0x00000027>;
                            phandle = <0x00000040>;
                        };
                        cpsw_sleep {
                            pinctrl-single,pins = <0x00000110 0x00000118 0x00000120 0x00000128 0x00000130 0x00000138 0x00000140 0x000000e4 0x64617669 0x61756c74 0x000002d1 0x00000010 0x00000042 0x6e63695f 0x00000003 0x00000027 0x00000004 0x00000001 0x70696e73 0x000002d1 0x00000030 0x00000030 0x00000030 0x00000004 0x00000001 0x70696e73>;
                            phandle = <0x00000041>;
                        };
                        davinci_mdio_default {
                            pinctrl-single,pins = <0x00000148 0x00000003 0x00000002 0x6d64696f>;
                            phandle = <0x00000042>;
                        };
                        davinci_mdio_sleep {
                            pinctrl-single,pins = <0x00000148 0x00000003 0x00000002 0x6d63315f>;
                            phandle = <0x00000043>;
                        };
                        pinmux_mmc1_pins {
                            pinctrl-single,pins = <0x00000160 0x000000f8 0x000000f0 0x00000100 0x000000e4 0x70696e6d 0x00000000 0x00000080 0x00000000 0x00000008 0x00000010 0x00000018 0x00000003 0x00000002>;
                            phandle = <0x0000002e>;
                        };
                        pinmux_emmc_pins {
                            pinctrl-single,pins = <0x00000080 0x00000000 0x00000008 0x00000010 0x00000018 0x00000003 0x00000002 0x395f3139 0x00000000 0x0000017c 0x000000e4 0x70696e6d 0x6f5f7069 0x000002d1 0x00000004 0x00000001 0x5f677069 0x00000003 0x00000037 0x0000004b>;
                            phandle = <0x00000031>;
                        };
                        pinmux_P9_19_default_pin {
                            pinctrl-single,pins = <0x0000017c 0x000000e4>;
                            phandle = <0x00000049>;
                        };
                        pinmux_P9_19_gpio_pin {
                            pinctrl-single,pins = <0x0000017c 0x000000e4>;
                            phandle = <0x0000004a>;
                        };
                        pinmux_P9_19_gpio_pu_pin {
                            pinctrl-single,pins = <0x0000017c 0x000000e4>;
                            phandle = <0x0000004b>;
                        };
                        pinmux_P9_19_gpio_pd_pin {
                            pinctrl-single,pins = <0x0000017c 0x000000e4>;
                            phandle = <0x0000004c>;
                        };
                        pinmux_P9_19_gpio_input_pin {
                            pinctrl-single,pins = <0x0000017c 0x000000e4>;
                            phandle = <0x0000004d>;
                        };
                        pinmux_P9_19_timer_pin {
                            pinctrl-single,pins = <0x0000017c 0x000000e4>;
                            phandle = <0x00000052>;
                        };
                        pinmux_P9_19_can_pin {
                            pinctrl-single,pins = <0x0000017c 0x000000e4>;
                            phandle = <0x0000004f>;
                        };
                        pinmux_P9_19_i2c_pin {
                            pinctrl-single,pins = <0x0000017c 0x000000e4>;
                            phandle = <0x00000050>;
                        };
                        pinmux_P9_19_spi_cs_pin {
                            pinctrl-single,pins = <0x0000017c 0x000000e4>;
                            phandle = <0x0000004e>;
                        };
                        pinmux_P9_19_pru_uart_pin {
                            pinctrl-single,pins = <0x0000017c 0x000000e4>;
                            phandle = <0x00000051>;
                        };
                        pinmux_P9_20_default_pin {
                            pinctrl-single,pins = <0x00000178 0x000000e4>;
                            phandle = <0x00000053>;
                        };
                        pinmux_P9_20_gpio_pin {
                            pinctrl-single,pins = <0x00000178 0x000000e4>;
                            phandle = <0x00000054>;
                        };
                        pinmux_P9_20_gpio_pu_pin {
                            pinctrl-single,pins = <0x00000178 0x000000e4>;
                            phandle = <0x00000055>;
                        };
                        pinmux_P9_20_gpio_pd_pin {
                            pinctrl-single,pins = <0x00000178 0x000000e4>;
                            phandle = <0x00000056>;
                        };
                        pinmux_P9_20_gpio_input_pin {
                            pinctrl-single,pins = <0x00000178 0x000000e4>;
                            phandle = <0x00000057>;
                        };
                        pinmux_P9_20_timer_pin {
                            pinctrl-single,pins = <0x00000178 0x000000e4>;
                            phandle = <0x0000005c>;
                        };
                        pinmux_P9_20_can_pin {
                            pinctrl-single,pins = <0x00000178 0x000000e4>;
                            phandle = <0x00000059>;
                        };
                        pinmux_P9_20_i2c_pin {
                            pinctrl-single,pins = <0x00000178 0x000000e4>;
                            phandle = <0x0000005a>;
                        };
                        pinmux_P9_20_spi_cs_pin {
                            pinctrl-single,pins = <0x00000178 0x000000e4>;
                            phandle = <0x00000058>;
                        };
                        pinmux_P9_20_pru_uart_pin {
                            pinctrl-single,pins = <0x00000178 0x000000e4>;
                            phandle = <0x0000005b>;
                        };
                        mcasp0_pins {
                            pinctrl-single,pins = <0x000001ac 0x00000194 0x000001a4 0x00000198 0x00000003 0x00000002 0x696e735f 0x00000040 0x0000019c 0x00000190 0x00000078 0x0000006c 0x000000e4 0x00000001 0x00000003 0x6f6e0073>;
                            phandle = <0x00000046>;
                        };
                        mcasp0_pins_sleep {
                            pinctrl-single,pins = <0x000001ac 0x00000194 0x000001a4 0x00000198 0x00000003 0x00000002 0x636f6e66 0x00000000 0x652d6275 0x00000106 0x00000004 0x00000004 0x0000000c 0x00000800 0x00000006 0x00000003>;
                            phandle = <0x00000047>;
                        };
                    };
                    scm_conf@0 {
                        compatible = "syscon", "simple-bus";
                        reg = <0x00000000 0x0000001c>;
                        #address-cells = <0x00000001>;
                        #size-cells = <0x00000001>;
                        ranges = <0x00000000 0x00000004 0x636c6f63>;
                        phandle = <0x00000006>;
                        clocks {
                            #address-cells = <0x00000001>;
                            #size-cells = <0x00000000>;
                            phandle = <0x0000008b>;
                            sys_clkin_ck@40 {
                                #clock-cells = <0x00000000>;
                                compatible = "ti,mux-clock";
                                clocks = <0x0000001f 0x00000003 0x00000003 0x00000003>;
                                ti,bit-shift = <0x00000016>;
                                reg = <0x00000040>;
                                phandle = <0x00000009>;
                            };
                            adc_tsc_fck {
                                #clock-cells = <0x00000000>;
                                compatible = "fixed-factor-clock";
                                clocks = <0x00000009>;
                                clock-mult = <0x00000001>;
                                clock-div = <0x00000001>;
                                phandle = <0x0000008c>;
                            };
                            dcan0_fck {
                                #clock-cells = <0x00000000>;
                                compatible = "fixed-factor-clock";
                                clocks = <0x00000009>;
                                clock-mult = <0x00000001>;
                                clock-div = <0x00000001>;
                                phandle = <0x00000032>;
                            };
                            dcan1_fck {
                                #clock-cells = <0x00000000>;
                                compatible = "fixed-factor-clock";
                                clocks = <0x00000009>;
                                clock-mult = <0x00000001>;
                                clock-div = <0x00000001>;
                                phandle = <0x00000033>;
                            };
                            mcasp0_fck {
                                #clock-cells = <0x00000000>;
                                compatible = "fixed-factor-clock";
                                clocks = <0x00000009>;
                                clock-mult = <0x00000001>;
                                clock-div = <0x00000001>;
                                phandle = <0x0000008d>;
                            };
                            mcasp1_fck {
                                #clock-cells = <0x00000000>;
                                compatible = "fixed-factor-clock";
                                clocks = <0x00000009>;
                                clock-mult = <0x00000001>;
                                clock-div = <0x00000001>;
                                phandle = <0x0000008e>;
                            };
                            smartreflex0_fck {
                                #clock-cells = <0x00000000>;
                                compatible = "fixed-factor-clock";
                                clocks = <0x00000009>;
                                clock-mult = <0x00000001>;
                                clock-div = <0x00000001>;
                                phandle = <0x0000008f>;
                            };
                            smartreflex1_fck {
                                #clock-cells = <0x00000000>;
                                compatible = "fixed-factor-clock";
                                clocks = <0x00000009>;
                                clock-mult = <0x00000001>;
                                clock-div = <0x00000001>;
                                phandle = <0x00000090>;
                            };
                            sha0_fck {
                                #clock-cells = <0x00000000>;
                                compatible = "fixed-factor-clock";
                                clocks = <0x00000009>;
                                clock-mult = <0x00000001>;
                                clock-div = <0x00000001>;
                                phandle = <0x00000091>;
                            };
                            aes0_fck {
                                #clock-cells = <0x00000000>;
                                compatible = "fixed-factor-clock";
                                clocks = <0x00000009>;
                                clock-mult = <0x00000001>;
                                clock-div = <0x00000001>;
                                phandle = <0x00000092>;
                            };
                            rng_fck {
                                #clock-cells = <0x00000000>;
                                compatible = "fixed-factor-clock";
                                clocks = <0x00000009>;
                                clock-mult = <0x00000001>;
                                clock-div = <0x00000001>;
                                phandle = <0x00000093>;
                            };
                            ehrpwm0_tbclk@44e10664 {
                                #clock-cells = <0x00000000>;
                                compatible = "ti,gate-clock";
                                clocks = <0x00000023>;
                                ti,bit-shift = <0x00000000>;
                                reg = <0x00000664>;
                                phandle = <0x0000003b>;
                            };
                            ehrpwm1_tbclk@44e10664 {
                                #clock-cells = <0x00000000>;
                                compatible = "ti,gate-clock";
                                clocks = <0x00000023>;
                                ti,bit-shift = <0x00000001>;
                                reg = <0x00000664>;
                                phandle = <0x0000003c>;
                            };
                            ehrpwm2_tbclk@44e10664 {
                                #clock-cells = <0x00000000>;
                                compatible = "ti,gate-clock";
                                clocks = <0x00000023>;
                                ti,bit-shift = <0x00000002>;
                                reg = <0x00000664>;
                                phandle = <0x0000003d>;
                            };
                        };
                    };
                    wkup_m3_ipc@1324 {
                        compatible = "ti,am3352-wkup-m3-ipc";
                        reg = <0x00001324 0x000001d6>;
                        interrupts = <0x0000004e>;
                        ti,rproc = <0x00000024>;
                        mboxes = <0x00000025 0x000002f5>;
                        ti,scale-data-fw = "am335x-bone-scale-data.bin";
                        phandle = <0x00000094>;
                    };
                    dma-router@f90 {
                        compatible = "ti,am335x-edma-crossbar";
                        reg = <0x00000f90 0x00000306>;
                        #dma-cells = <0x00000003>;
                        dma-requests = <0x00000020>;
                        dma-masters = <0x00000027>;
                        phandle = <0x0000002d>;
                    };
                    clockdomains {
                        phandle = <0x00000095>;
                    };
                };
            };
            interrupt-controller@48200000 {
                compatible = "ti,am33xx-intc";
                interrupt-controller;
                #interrupt-cells = <0x00000001>;
                reg = <0x48200000 0x000000e4>;
                phandle = <0x00000001>;
            };
            edma@49000000 {
                compatible = "ti,edma3-tpcc";
                ti,hwmods = "tpcc";
                reg = <0x49000000 0x000001fa>;
                reg-names = "edma3_cc";
                interrupts = <0x0000000c 0x00000027 0x696e7400>;
                interrupt-names = "edma3_ccint", "edma3_mperr", "edma3_ccerrint";
                dma-requests = <0x00000040>;
                #dma-cells = <0x00000002>;
                ti,tptcs = <0x00000028 0x0000002a 0x00000369 0x00000004 0x00000001 0x30000000>;
                ti,edma-memcpy-channels = <0x00000014 0x000000e4>;
                phandle = <0x00000027>;
            };
            tptc@49800000 {
                compatible = "ti,edma3-tptc";
                ti,hwmods = "tptc0";
                reg = <0x49800000 0x000001d6>;
                interrupts = <0x00000070>;
                interrupt-names = "edma3_tcerrint";
                phandle = <0x00000028>;
            };
            tptc@49900000 {
                compatible = "ti,edma3-tptc";
                ti,hwmods = "tptc1";
                reg = <0x49900000 0x000001d6>;
                interrupts = <0x00000071>;
                interrupt-names = "edma3_tcerrint";
                phandle = <0x00000029>;
            };
            tptc@49a00000 {
                compatible = "ti,edma3-tptc";
                ti,hwmods = "tptc2";
                reg = <0x49a00000 0x000001d6>;
                interrupts = <0x00000072>;
                interrupt-names = "edma3_tcerrint";
                phandle = <0x0000002a>;
            };
            gpio@44e07000 {
                compatible = "ti,omap4-gpio";
                ti,hwmods = "gpio1";
                gpio-controller;
                #gpio-cells = <0x00000002>;
                interrupt-controller;
                #interrupt-cells = <0x00000002>;
                reg = <0x44e07000 0x000001d6>;
                interrupts = <0x00000060>;
                gpio-line-names = "MDIO_DATA", "MDIO_CLK", "SPI0_SCLK", "SPI0_D0", "SPI0_D1", "SPI0_CS0", "SPI0_CS1", "ECAP0_IN_PWM0_OUT", "LCD_DATA12", "LCD_DATA13", "LCD_DATA14", "LCD_DATA15", "UART1_CTSN", "UART1_RTSN", "UART1_RXD", "UART1_TXD", "GMII1_TXD3", "GMII1_TXD2", "USB0_DRVVBUS", "XDMA_EVENT_INTR0", "XDMA_EVENT_INTR1", "GMII1_TXD1", "GPMC_AD8", "GPMC_AD9", "NC", "NC", "GPMC_AD10", "GPMC_AD11", "GMII1_TXD0", "RMII1_REFCLK", "GPMC_WAIT0", "GPMC_WPN";
                phandle = <0x0000002f>;
            };
            gpio@4804c000 {
                compatible = "ti,omap4-gpio";
                ti,hwmods = "gpio2";
                gpio-controller;
                #gpio-cells = <0x00000002>;
                interrupt-controller;
                #interrupt-cells = <0x00000002>;
                reg = <0x4804c000 0x000001d6>;
                interrupts = <0x00000062>;
                gpio-line-names = "GPMC_AD0", "GPMC_AD1", "GPMC_AD2", "GPMC_AD3", "GPMC_AD4", "GPMC_AD5", "GPMC_AD6", "GPMC_AD7", "UART0_CTSN", "UART0_RTSN", "UART0_RXD", "UART0_TXD", "GPMC_AD12", "GPMC_AD13", "GPMC_AD14", "GPMC_AD15", "GPMC_A0", "GPMC_A1", "GPMC_A2", "GPMC_A3", "GPMC_A4", "GPMC_A5", "GPMC_A6", "GPMC_A7", "GPMC_A8", "GPMC_A9", "GPMC_A10", "GPMC_A11", "GPMC_BE1N", "GPMC_CSN0", "GPMC_CSN1", "GPMC_CSN2";
                phandle = <0x0000005e>;
            };
            gpio@481ac000 {
                compatible = "ti,omap4-gpio";
                ti,hwmods = "gpio3";
                gpio-controller;
                #gpio-cells = <0x00000002>;
                interrupt-controller;
                #interrupt-cells = <0x00000002>;
                reg = <0x481ac000 0x000001d6>;
                interrupts = <0x00000020>;
                gpio-line-names = "GPMC_CSN3", "GPMC_CLK", "GPMC_ADVN_ALE", "GPMC_OEN_REN", "GPMC_WEN", "GPMC_BE0N_CLE", "LCD_DATA0", "LCD_DATA1", "LCD_DATA2", "LCD_DATA3", "LCD_DATA4", "LCD_DATA5", "LCD_DATA6", "LCD_DATA7", "LCD_DATA8", "LCD_DATA9", "LCD_DATA10", "LCD_DATA11", "GMII1_RXD3", "GMII1_RXD2", "GMII1_RXD1", "GMII1_RXD0", "LCD_VSYNC", "LCD_HSYNC", "LCD_PCLK", "LCD_AC_BIAS_EN", "MMC0_DAT3", "MMC0_DAT2", "MMC0_DAT1", "MMC0_DAT0", "MMC0_CLK", "MMC0_CMD";
                phandle = <0x00000096>;
            };
            gpio@481ae000 {
                compatible = "ti,omap4-gpio";
                ti,hwmods = "gpio4";
                gpio-controller;
                #gpio-cells = <0x00000002>;
                interrupt-controller;
                #interrupt-cells = <0x00000002>;
                reg = <0x481ae000 0x000001d6>;
                interrupts = <0x0000003e>;
                gpio-line-names = "GMII1_COL", "GMII1_CRS", "GMII1_RXER", "GMII1_TXEN", "GMII1_RXDV", "I2C0_SDA", "I2C0_SCL", "EMU0", "EMU1", "GMII1_TXCLK", "GMII1_RXCLK", "NC", "NC", "USB1_DRVVBUS", "MCASP0_ACLKX", "MCASP0_FSX", "MCASP0_AXR0", "MCASP0_AHCLKR", "MCASP0_ACLKR", "MCASP0_FSR", "MCASP0_AXR1", "MCASP0_AHCLKX", "NC", "NC", "NC", "NC", "NC", "NC", "NC", "NC", "NC", "NC";
                phandle = <0x00000097>;
            };
            serial@44e09000 {
                compatible = "ti,am3352-uart", "ti,omap3-uart";
                ti,hwmods = "uart1";
                clock-frequency = <0x02dc6c00>;
                reg = <0x44e09000 0x000001d6>;
                interrupts = <0x00000048>;
                status = "okay";
                dmas = <0x00000027 0x0000001b 0x000003b9 0x00000008 0x00000003 0x00000003>;
                dma-names = "tx", "rx";
                pinctrl-names = "default";
                pinctrl-0 = <0x0000002b>;
                phandle = <0x00000098>;
            };
            serial@48022000 {
                compatible = "ti,am3352-uart", "ti,omap3-uart";
                ti,hwmods = "uart2";
                clock-frequency = <0x02dc6c00>;
                reg = <0x48022000 0x000001d6>;
                interrupts = <0x00000049>;
                status = "disabled";
                dmas = <0x00000027 0x0000001d 0x000003b9 0x00000004 0x00000001 0x30303000>;
                dma-names = "tx", "rx";
                phandle = <0x00000099>;
            };
            serial@48024000 {
                compatible = "ti,am3352-uart", "ti,omap3-uart";
                ti,hwmods = "uart3";
                clock-frequency = <0x02dc6c00>;
                reg = <0x48024000 0x000001d6>;
                interrupts = <0x0000004a>;
                status = "disabled";
                dmas = <0x00000027 0x0000001f 0x000003b9 0x00000004 0x00000001 0x30303000>;
                dma-names = "tx", "rx";
                phandle = <0x0000009a>;
            };
            serial@481a6000 {
                compatible = "ti,am3352-uart", "ti,omap3-uart";
                ti,hwmods = "uart4";
                clock-frequency = <0x02dc6c00>;
                reg = <0x481a6000 0x000001d6>;
                interrupts = <0x0000002c>;
                status = "disabled";
                phandle = <0x0000009b>;
            };
            serial@481a8000 {
                compatible = "ti,am3352-uart", "ti,omap3-uart";
                ti,hwmods = "uart5";
                clock-frequency = <0x02dc6c00>;
                reg = <0x481a8000 0x000001d6>;
                interrupts = <0x0000002d>;
                status = "disabled";
                phandle = <0x0000009c>;
            };
            serial@481aa000 {
                compatible = "ti,am3352-uart", "ti,omap3-uart";
                ti,hwmods = "uart6";
                clock-frequency = <0x02dc6c00>;
                reg = <0x481aa000 0x000001d6>;
                interrupts = <0x0000002e>;
                status = "disabled";
                phandle = <0x0000009d>;
            };
            i2c@44e0b000 {
                compatible = "ti,omap4-i2c";
                #address-cells = <0x00000001>;
                #size-cells = <0x00000000>;
                ti,hwmods = "i2c1";
                reg = <0x44e0b000 0x000001d6>;
                interrupts = <0x00000046>;
                status = "okay";
                pinctrl-names = "default";
                pinctrl-0 = <0x0000002c>;
                clock-frequency = <0x00061a80>;
                phandle = <0x0000009e>;
                tps@24 {
                    reg = <0x00000024>;
                    compatible = "ti,tps65217";
                    interrupt-controller;
                    #interrupt-cells = <0x00000001>;
                    interrupts = <0x00000007>;
                    interrupt-parent = <0x00000001>;
                    ti,pmic-shutdown-controller;
                    phandle = <0x00000039>;
                    charger {
                        compatible = "ti,tps65217-charger";
                        interrupts = <0x00000000 0x00000350>;
                        interrupt-names = "USB", "AC";
                        status = "okay";
                    };
                    pwrbutton {
                        compatible = "ti,tps65217-pwrbutton";
                        interrupts = <0x00000002>;
                        status = "okay";
                    };
                    regulators {
                        #address-cells = <0x00000001>;
                        #size-cells = <0x00000000>;
                        regulator@0 {
                            reg = <0x00000000>;
                            regulator-compatible = "dcdc1";
                            regulator-name = "vdds_dpr";
                            regulator-always-on;
                            phandle = <0x0000009f>;
                        };
                        regulator@1 {
                            reg = <0x00000001>;
                            regulator-compatible = "dcdc2";
                            regulator-name = "vdd_mpu";
                            regulator-min-microvolt = <0x000e1d48>;
                            regulator-max-microvolt = <0x00149f4c>;
                            regulator-boot-on;
                            regulator-always-on;
                            phandle = <0x00000005>;
                        };
                        regulator@2 {
                            reg = <0x00000002>;
                            regulator-compatible = "dcdc3";
                            regulator-name = "vdd_core";
                            regulator-min-microvolt = <0x000e1d48>;
                            regulator-max-microvolt = <0x00118c30>;
                            regulator-boot-on;
                            regulator-always-on;
                            phandle = <0x000000a0>;
                        };
                        regulator@3 {
                            reg = <0x00000003>;
                            regulator-compatible = "ldo1";
                            regulator-name = "vio,vrtc,vdds";
                            regulator-always-on;
                            phandle = <0x000000a1>;
                        };
                        regulator@4 {
                            reg = <0x00000004>;
                            regulator-compatible = "ldo2";
                            regulator-name = "vdd_3v3aux";
                            regulator-always-on;
                            phandle = <0x000000a2>;
                        };
                        regulator@5 {
                            reg = <0x00000005>;
                            regulator-compatible = "ldo3";
                            regulator-name = "vdd_1v8";
                            regulator-always-on;
                            regulator-min-microvolt = <0x001b7740>;
                            regulator-max-microvolt = <0x001b7740>;
                            phandle = <0x000000a3>;
                        };
                        regulator@6 {
                            reg = <0x00000006>;
                            regulator-compatible = "ldo4";
                            regulator-name = "vdd_3v3a";
                            regulator-always-on;
                            phandle = <0x000000a4>;
                        };
                    };
                };
                baseboard_eeprom@50 {
                    compatible = "atmel,24c256";
                    reg = <0x00000050>;
                    #address-cells = <0x00000001>;
                    #size-cells = <0x00000001>;
                    phandle = <0x000000a5>;
                    baseboard_data@0 {
                        reg = <0x00000000 0x000000e4>;
                        phandle = <0x000000a6>;
                    };
                };
            };
            i2c@4802a000 {
                compatible = "ti,omap4-i2c";
                #address-cells = <0x00000001>;
                #size-cells = <0x00000000>;
                ti,hwmods = "i2c2";
                reg = <0x4802a000 0x000001d6>;
                interrupts = <0x00000047>;
                status = "disabled";
                phandle = <0x000000a7>;
            };
            i2c@4819c000 {
                compatible = "ti,omap4-i2c";
                #address-cells = <0x00000001>;
                #size-cells = <0x00000000>;
                ti,hwmods = "i2c3";
                reg = <0x4819c000 0x000001d6>;
                interrupts = <0x0000001e>;
                status = "okay";
                pinctrl-names = "default";
                pinctrl-0;
                clock-frequency = <0x000186a0>;
                phandle = <0x000000a8>;
                cape_eeprom0@54 {
                    compatible = "atmel,24c256";
                    reg = <0x00000054>;
                    #address-cells = <0x00000001>;
                    #size-cells = <0x00000001>;
                    phandle = <0x000000a9>;
                    cape_data@0 {
                        reg = <0x00000000 0x000000e4>;
                        phandle = <0x000000aa>;
                    };
                };
                cape_eeprom1@55 {
                    compatible = "atmel,24c256";
                    reg = <0x00000055>;
                    #address-cells = <0x00000001>;
                    #size-cells = <0x00000001>;
                    phandle = <0x000000ab>;
                    cape_data@0 {
                        reg = <0x00000000 0x000000e4>;
                        phandle = <0x000000ac>;
                    };
                };
                cape_eeprom2@56 {
                    compatible = "atmel,24c256";
                    reg = <0x00000056>;
                    #address-cells = <0x00000001>;
                    #size-cells = <0x00000001>;
                    phandle = <0x000000ad>;
                    cape_data@0 {
                        reg = <0x00000000 0x000000e4>;
                        phandle = <0x000000ae>;
                    };
                };
                cape_eeprom3@57 {
                    compatible = "atmel,24c256";
                    reg = <0x00000057>;
                    #address-cells = <0x00000001>;
                    #size-cells = <0x00000001>;
                    phandle = <0x000000af>;
                    cape_data@0 {
                        reg = <0x00000000 0x000000e4>;
                        phandle = <0x000000b0>;
                    };
                };
            };
            mmc@48060000 {
                compatible = "ti,omap4-hsmmc";
                ti,hwmods = "mmc1";
                ti,dual-volt;
                ti,needs-special-reset;
                ti,needs-special-hs-handling;
                dmas = <0x0000002d 0x0000002d 0x00000003 0x78000000 0x00000040 0x48060000 0x000003ad 0x00000004>;
                dma-names = "tx", "rx";
                interrupts = <0x00000040>;
                reg = <0x48060000 0x000003ad>;
                status = "okay";
                bus-width = <0x00000004>;
                pinctrl-names = "default";
                pinctrl-0 = <0x0000002e>;
                cd-gpios = <0x0000002f 0x00000004 0x00000004>;
                vmmc-supply = <0x00000030>;
                phandle = <0x000000b1>;
            };
            mmc@481d8000 {
                compatible = "ti,omap4-hsmmc";
                ti,hwmods = "mmc2";
                ti,needs-special-reset;
                dmas = <0x00000027 0x00000003 0x000003b9 0x00000004 0x00000008 0x00000003>;
                dma-names = "tx", "rx";
                interrupts = <0x0000001c>;
                reg = <0x481d8000 0x000003ad>;
                status = "okay";
                vmmc-supply = <0x00000030>;
                pinctrl-names = "default";
                pinctrl-0 = <0x00000031>;
                bus-width = <0x00000008>;
                phandle = <0x000000b2>;
            };
            mmc@47810000 {
                compatible = "ti,omap4-hsmmc";
                ti,hwmods = "mmc3";
                ti,needs-special-reset;
                interrupts = <0x0000001d>;
                reg = <0x47810000 0x000003ad>;
                status = "disabled";
                phandle = <0x000000b3>;
            };
            spinlock@480ca000 {
                compatible = "ti,omap4-hwspinlock";
                reg = <0x480ca000 0x000001e1>;
                ti,hwmods = "spinlock";
                #hwlock-cells = <0x00000001>;
                phandle = <0x000000b4>;
            };
            wdt@44e35000 {
                compatible = "ti,omap3-wdt";
                ti,hwmods = "wd_timer2";
                reg = <0x44e35000 0x000001d6>;
                interrupts = <0x0000005b>;
                phandle = <0x000000b5>;
            };
            can@481cc000 {
                compatible = "ti,am3352-d_can";
                ti,hwmods = "d_can0";
                reg = <0x481cc000 0x0000011e>;
                clocks = <0x00000032>;
                clock-names = "fck";
                syscon-raminit = <0x00000006 0x00000004 0x00000009>;
                interrupts = <0x00000034>;
                status = "disabled";
                phandle = <0x000000b6>;
            };
            can@481d0000 {
                compatible = "ti,am3352-d_can";
                ti,hwmods = "d_can1";
                reg = <0x481d0000 0x0000011e>;
                clocks = <0x00000033>;
                clock-names = "fck";
                syscon-raminit = <0x00000006 0x00000004 0x00000009>;
                interrupts = <0x00000037>;
                status = "disabled";
                phandle = <0x000000b7>;
            };
            mailbox@480c8000 {
                compatible = "ti,omap4-mailbox";
                reg = <0x480c8000 0x000001d6>;
                interrupts = <0x0000004d>;
                ti,hwmods = "mailbox";
                #mbox-cells = <0x00000001>;
                ti,mbox-num-users = <0x00000004>;
                ti,mbox-num-fifos = <0x00000008>;
                phandle = <0x00000025>;
                wkup_m3 {
                    ti,mbox-send-noirq;
                    ti,mbox-tx = <0x00000000 0x0000000c 0x00000003>;
                    ti,mbox-rx = <0x00000000 0x00000004 0x00000002>;
                    phandle = <0x00000026>;
                };
            };
            timer@44e31000 {
                compatible = "ti,am335x-timer-1ms";
                reg = <0x44e31000 0x000001d6>;
                interrupts = <0x00000043>;
                ti,hwmods = "timer1";
                ti,timer-alwon;
                clocks = <0x00000034>;
                clock-names = "fck";
                phandle = <0x000000b8>;
            };
            timer@48040000 {
                compatible = "ti,am335x-timer";
                reg = <0x48040000 0x000001d6>;
                interrupts = <0x00000044>;
                ti,hwmods = "timer2";
                clocks = <0x00000035>;
                clock-names = "fck";
                phandle = <0x000000b9>;
            };
            timer@48042000 {
                compatible = "ti,am335x-timer";
                reg = <0x48042000 0x000001d6>;
                interrupts = <0x00000045>;
                ti,hwmods = "timer3";
                phandle = <0x000000ba>;
            };
            timer@48044000 {
                compatible = "ti,am335x-timer";
                reg = <0x48044000 0x000001d6>;
                interrupts = <0x0000005c>;
                ti,hwmods = "timer4";
                ti,timer-pwm;
                phandle = <0x000000bb>;
            };
            timer@48046000 {
                compatible = "ti,am335x-timer";
                reg = <0x48046000 0x000001d6>;
                interrupts = <0x0000005d>;
                ti,hwmods = "timer5";
                ti,timer-pwm;
                phandle = <0x000000bc>;
            };
            timer@48048000 {
                compatible = "ti,am335x-timer";
                reg = <0x48048000 0x000001d6>;
                interrupts = <0x0000005e>;
                ti,hwmods = "timer6";
                ti,timer-pwm;
                phandle = <0x000000bd>;
            };
            timer@4804a000 {
                compatible = "ti,am335x-timer";
                reg = <0x4804a000 0x000001d6>;
                interrupts = <0x0000005f>;
                ti,hwmods = "timer7";
                ti,timer-pwm;
                phandle = <0x000000be>;
            };
            rtc@44e3e000 {
                compatible = "ti,am3352-rtc", "ti,da830-rtc";
                reg = <0x44e3e000 0x000001d6>;
                interrupts = <0x0000004b 0x000001e1>;
                ti,hwmods = "rtc";
                clocks = <0x00000018 0x00000003 0x636c6b00 0x00000000>;
                clock-names = "ext-clk", "int-clk";
                system-power-controller;
                phandle = <0x000000bf>;
            };
            spi@48030000 {
                compatible = "ti,omap4-mcspi";
                #address-cells = <0x00000001>;
                #size-cells = <0x00000000>;
                reg = <0x48030000 0x000001d6>;
                interrupts = <0x00000041>;
                ti,spi-num-cs = <0x00000002>;
                ti,hwmods = "spi0";
                dmas = <0x00000027 0x00000011 0x00000000 0x00000003 0x72783000 0x00000009 0x00000000 0x000000c0 0x34383161 0x0000000f 0x2d6d6373 0x0000001c>;
                dma-names = "tx0", "rx0", "tx1", "rx1";
                status = "disabled";
                phandle = <0x000000c0>;
            };
            spi@481a0000 {
                compatible = "ti,omap4-mcspi";
                #address-cells = <0x00000001>;
                #size-cells = <0x00000000>;
                reg = <0x481a0000 0x000001d6>;
                interrupts = <0x0000007d>;
                ti,spi-num-cs = <0x00000002>;
                ti,hwmods = "spi1";
                dmas = <0x00000027 0x0000002b 0x00000000 0x00000003 0x72783000 0x00000009 0x00000000 0x000000c1 0x34373430 0x0000000e 0x782d7573 0x00000106>;
                dma-names = "tx0", "rx0", "tx1", "rx1";
                status = "disabled";
                phandle = <0x000000c1>;
            };
            usb@47400000 {
                compatible = "ti,am33xx-usb";
                reg = <0x47400000 0x000001f3>;
                ranges;
                #address-cells = <0x00000001>;
                #size-cells = <0x00000001>;
                ti,hwmods = "usb_otg_hs";
                status = "okay";
                phandle = <0x000000c2>;
                control@44e10620 {
                    compatible = "ti,am335x-usb-ctrl-module";
                    reg = <0x44e10620 0x00000003 0x6374726c 0x00000005>;
                    reg-names = "phy_ctrl", "wakeup";
                    status = "okay";
                    phandle = <0x00000036>;
                };
                usb-phy@47401300 {
                    compatible = "ti,am335x-usb-phy";
                    reg = <0x47401300 0x000001fa>;
                    reg-names = "phy";
                    status = "okay";
                    ti,ctrl_mod = <0x00000036>;
                    #phy-cells = <0x00000000>;
                    phandle = <0x00000037>;
                };
                usb@47401000 {
                    compatible = "ti,musb-am33xx";
                    status = "okay";
                    reg = <0x47401400 0x00000003 0x6f6e7472 0x000001d6>;
                    reg-names = "mc", "control";
                    interrupts = <0x00000012>;
                    interrupt-names = "mc", "vbus";
                    dr_mode = "peripheral";
                    mentor,multipoint = <0x00000001>;
                    mentor,num-eps = <0x00000010>;
                    mentor,ram-bits = <0x0000000c>;
                    mentor,power = <0x000001f4>;
                    phys = <0x00000037>;
                    dmas = <0x00000038 0x00000001 0x00000000 0x00000038 0x00000005 0x00000000 0x00000038 0x00000009 0x00000000 0x00000038 0x0000000d 0x00000000 0x00000038 0x00000002 0x00000001 0x00000038 0x00000006 0x00000001 0x00000038 0x0000000a 0x00000001 0x00000038 0x0000000e 0x000003b9 0x72783400 0x72783800 0x31007278 0x00727831 0x33007478 0x37007478 0x78313100 0x31340074 0x000005eb 0x00000000 0x000000c3 0x70687940 0x00000003 0x6d333335 0x00000003 0x00000100 0x70687900 0x6f6b6179 0x00000589 0x00000595 0x000000e4 0x75736240 0x00000003 0x7573622d 0x00000005 0x00000003 0x00000400 0x0000000b 0x6f6c0000 0x00000013 0x6d630000 0x686f7374 0x000005a8 0x000005ba 0x000005c9 0x000005d9 0x000005e6 0x000003b4 0x00000038 0x00000011 0x00000000 0x00000038 0x00000015 0x00000000 0x00000038 0x00000019 0x00000000 0x00000038 0x0000001d 0x00000001 0x00000038 0x00000012 0x00000001 0x00000038 0x00000016 0x00000001 0x00000038 0x0000001a 0x00000001 0x00000038 0x00000084 0x72783300 0x72783700 0x00727831 0x72783134 0x32007478>;
                    dma-names = "rx1", "rx2", "rx3", "rx4", "rx5", "rx6", "rx7", "rx8", "rx9", "rx10", "rx11", "rx12", "rx13", "rx14", "rx15", "tx1", "tx2", "tx3", "tx4", "tx5", "tx6", "tx7", "tx8", "tx9", "tx10", "tx11", "tx12", "tx13", "tx14", "tx15";
                    interrupts-extended = <0x00000001 0x00000003 0x00000002 0x34373430>;
                    phandle = <0x000000c3>;
                };
                usb-phy@47401b00 {
                    compatible = "ti,am335x-usb-phy";
                    reg = <0x47401b00 0x000001fa>;
                    reg-names = "phy";
                    status = "okay";
                    ti,ctrl_mod = <0x00000036>;
                    #phy-cells = <0x00000000>;
                    phandle = <0x0000003a>;
                };
                usb@47401800 {
                    compatible = "ti,musb-am33xx";
                    status = "okay";
                    reg = <0x47401c00 0x00000003 0x6f6e7472 0x000001d6>;
                    reg-names = "mc", "control";
                    interrupts = <0x00000013>;
                    interrupt-names = "mc";
                    dr_mode = "host";
                    mentor,multipoint = <0x00000001>;
                    mentor,num-eps = <0x00000010>;
                    mentor,ram-bits = <0x0000000c>;
                    mentor,power = <0x000001f4>;
                    phys = <0x0000003a>;
                    dmas = <0x00000038 0x00000010 0x00000000 0x00000038 0x00000014 0x00000000 0x00000038 0x00000018 0x00000000 0x00000038 0x0000001c 0x00000000 0x00000038 0x00000011 0x00000001 0x00000038 0x00000015 0x00000001 0x00000038 0x00000019 0x00000001 0x00000038 0x0000001d 0x000003b9 0x72783400 0x72783800 0x31007278 0x00727831 0x33007478 0x37007478 0x78313100 0x31340074 0x000000e4 0x646d612d 0x37343032 0x00000000 0x70693431 0x00000106 0x00001000 0x00004000 0x676c7565 0x73636865 0x67720000 0x00000011 0x676c7565 0x00000306 0x000005ff 0x0000060d 0x000003ad 0x00000004 0x00000002 0x38333030 0x00000000 0x6d737300 0x48300000 0x000001e1 0x00000004 0x00000004 0x00000009 0x00000000 0x48300100 0x48300180 0x00000080 0x000000c5 0x30303130 0x00000000 0x61700074 0x70000000 0x00000003 0x48300100 0x0000011e 0x00000125 0x000001d6 0x00000350 0x00000009 0x00000000 0x000000c6 0x40307834 0x0000000f 0x782d6571 0x00000106 0x00000004 0x00000004 0x00000004 0x00000004 0x00000009 0x00000000 0x000000c7 0x34383330 0x00000022>;
                    dma-names = "rx1", "rx2", "rx3", "rx4", "rx5", "rx6", "rx7", "rx8", "rx9", "rx10", "rx11", "rx12", "rx13", "rx14", "rx15", "tx1", "tx2", "tx3", "tx4", "tx5", "tx6", "tx7", "tx8", "tx9", "tx10", "tx11", "tx12", "tx13", "tx14", "tx15";
                    phandle = <0x000000c4>;
                };
                dma-controller@47402000 {
                    compatible = "ti,am3359-cppi41";
                    reg = <0x47400000 0x47403000 0x00000003 0x00636f6e 0x64756c65 0x00000003 0x00000003 0x00000000>;
                    reg-names = "glue", "controller", "scheduler", "queuemgr";
                    interrupts = <0x00000011>;
                    interrupt-names = "glue";
                    #dma-cells = <0x00000002>;
                    #dma-channels = <0x0000001e>;
                    #dma-requests = <0x00000100>;
                    status = "okay";
                    phandle = <0x00000038>;
                };
            };
            epwmss@48300000 {
                compatible = "ti,am33xx-pwmss";
                reg = <0x48300000 0x000001e1>;
                ti,hwmods = "epwmss0";
                #address-cells = <0x00000001>;
                #size-cells = <0x00000001>;
                status = "disabled";
                ranges = <0x48300100 0x48300180 0x00000080 0x000000c5 0x30303130 0x00000000 0x61700074 0x70000000 0x00000003>;
                phandle = <0x000000c5>;
                ecap@48300100 {
                    compatible = "ti,am3352-ecap", "ti,am33xx-ecap";
                    #pwm-cells = <0x00000003>;
                    reg = <0x48300100 0x0000011e>;
                    clocks = <0x00000023>;
                    clock-names = "fck";
                    interrupts = <0x0000001f>;
                    interrupt-names = "ecap0";
                    status = "disabled";
                    phandle = <0x000000c6>;
                };
                eqep@0x48300180 {
                    compatible = "ti,am33xx-eqep";
                    reg = <0x48300180 0x0000011e>;
                    clocks = <0x00000023>;
                    clock-names = "fck";
                    interrupt-parent = <0x00000001>;
                    interrupts = <0x0000004f>;
                    status = "disabled";
                    phandle = <0x000000c7>;
                };
                pwm@48300200 {
                    compatible = "ti,am3352-ehrpwm", "ti,am33xx-ehrpwm";
                    #pwm-cells = <0x00000003>;
                    reg = <0x48300200 0x0000011e>;
                    clocks = <0x0000003b 0x00000125>;
                    clock-names = "tbclk", "fck";
                    status = "disabled";
                    phandle = <0x000000c8>;
                };
            };
            epwmss@48302000 {
                compatible = "ti,am33xx-pwmss";
                reg = <0x48302000 0x000001e1>;
                ti,hwmods = "epwmss1";
                #address-cells = <0x00000001>;
                #size-cells = <0x00000001>;
                status = "disabled";
                ranges = <0x48302100 0x48302180 0x00000080 0x000000c9 0x30323130 0x00000000 0x61700074 0x70000000 0x00000003>;
                phandle = <0x000000c9>;
                ecap@48302100 {
                    compatible = "ti,am3352-ecap", "ti,am33xx-ecap";
                    #pwm-cells = <0x00000003>;
                    reg = <0x48302100 0x0000011e>;
                    clocks = <0x00000023>;
                    clock-names = "fck";
                    interrupts = <0x0000002f>;
                    interrupt-names = "ecap1";
                    status = "disabled";
                    phandle = <0x000000ca>;
                };
                eqep@0x48302180 {
                    compatible = "ti,am33xx-eqep";
                    reg = <0x48302180 0x0000011e>;
                    clocks = <0x00000023>;
                    clock-names = "fck";
                    interrupt-parent = <0x00000001>;
                    interrupts = <0x00000058>;
                    status = "disabled";
                    phandle = <0x000000cb>;
                };
                pwm@48302200 {
                    compatible = "ti,am3352-ehrpwm", "ti,am33xx-ehrpwm";
                    #pwm-cells = <0x00000003>;
                    reg = <0x48302200 0x0000011e>;
                    clocks = <0x0000003c 0x00000125>;
                    clock-names = "tbclk", "fck";
                    status = "disabled";
                    phandle = <0x000000cc>;
                };
            };
            epwmss@48304000 {
                compatible = "ti,am33xx-pwmss";
                reg = <0x48304000 0x000001e1>;
                ti,hwmods = "epwmss2";
                #address-cells = <0x00000001>;
                #size-cells = <0x00000001>;
                status = "disabled";
                ranges = <0x48304100 0x48304180 0x00000080 0x000000cd 0x30343130 0x00000000 0x61700074 0x70000000 0x00000003>;
                phandle = <0x000000cd>;
                ecap@48304100 {
                    compatible = "ti,am3352-ecap", "ti,am33xx-ecap";
                    #pwm-cells = <0x00000003>;
                    reg = <0x48304100 0x0000011e>;
                    clocks = <0x00000023>;
                    clock-names = "fck";
                    interrupts = <0x0000003d>;
                    interrupt-names = "ecap2";
                    status = "disabled";
                    phandle = <0x000000ce>;
                };
                eqep@0x48304180 {
                    compatible = "ti,am33xx-eqep";
                    reg = <0x48304180 0x0000011e>;
                    clocks = <0x00000023>;
                    clock-names = "fck";
                    interrupt-parent = <0x00000001>;
                    interrupts = <0x00000059>;
                    status = "disabled";
                    phandle = <0x000000cf>;
                };
                pwm@48304200 {
                    compatible = "ti,am3352-ehrpwm", "ti,am33xx-ehrpwm";
                    #pwm-cells = <0x00000003>;
                    reg = <0x48304200 0x0000011e>;
                    clocks = <0x0000003d 0x00000125>;
                    clock-names = "tbclk", "fck";
                    status = "disabled";
                    phandle = <0x000000d0>;
                };
            };
            ethernet@4a100000 {
                compatible = "ti,am335x-cpsw", "ti,cpsw";
                ti,hwmods = "cpgmac0";
                clocks = <0x0000003e 0x00000125>;
                clock-names = "fck", "cpts";
                cpdma_channels = <0x00000008>;
                ale_entries = <0x00000400>;
                bd_ram_size = <0x00002000>;
                mac_control = <0x00000020>;
                slaves = <0x00000001>;
                active_slave = <0x00000000>;
                cpts_clock_mult = <0x80000000>;
                cpts_clock_shift = <0x0000001d>;
                reg = <0x4a100000 0x00000003 0x00000003 0x00000003>;
                #address-cells = <0x00000001>;
                #size-cells = <0x00000001>;
                interrupts = <0x00000028 0x00000003 0x00000004 0x00000005>;
                ranges;
                syscon = <0x00000006>;
                status = "okay";
                pinctrl-names = "default", "sleep";
                pinctrl-0 = <0x00000040>;
                pinctrl-1 = <0x00000041>;
                phandle = <0x000000d1>;
                mdio@4a101000 {
                    compatible = "ti,cpsw-mdio", "ti,davinci_mdio";
                    #address-cells = <0x00000001>;
                    #size-cells = <0x00000000>;
                    ti,hwmods = "davinci_mdio";
                    bus_freq = <0x000f4240>;
                    reg = <0x4a101000 0x000003ad>;
                    status = "okay";
                    pinctrl-names = "default", "sleep";
                    pinctrl-0 = <0x00000042>;
                    pinctrl-1 = <0x00000043>;
                    phandle = <0x00000044>;
                };
                slave@4a100200 {
                    mac-address = [00 00 00 00 00 00];
                    phy_id = <0x00000044 0x000006b4>;
                    phy-mode = "mii";
                    phandle = <0x000000d2>;
                };
                slave@4a100300 {
                    mac-address = [00 00 00 00 00 00];
                    phandle = <0x000000d3>;
                };
                cpsw-phy-sel@44e10650 {
                    compatible = "ti,am3352-cpsw-phy-sel";
                    reg = <0x44e10650 0x000001fa>;
                    reg-names = "gmii-sel";
                    phandle = <0x000000d4>;
                };
            };
            ocmcram@40300000 {
                compatible = "mmio-sram";
                reg = <0x40300000 0x000001f3>;
                ranges = <0x00000000 0x00000004 0x00000004>;
                #address-cells = <0x00000001>;
                #size-cells = <0x00000001>;
                phandle = <0x000000d5>;
                pm-sram-code@0 {
                    compatible = "ti,sram";
                    reg = <0x00000000 0x000006bd>;
                    protect-exec;
                    phandle = <0x00000007>;
                };
                pm-sram-data@1000 {
                    compatible = "ti,sram";
                    reg = <0x00001000 0x000006ca>;
                    pool;
                    phandle = <0x00000008>;
                };
            };
            pruss-soc-bus@4a326004 {
                compatible = "ti,am3356-pruss-soc-bus";
                reg = <0x4a326004 0x000001e1>;
                ti,hwmods = "pruss";
                #address-cells = <0x00000001>;
                #size-cells = <0x00000001>;
                ranges;
                status = "disabled";
                phandle = <0x000000d6>;
                pruss@4a300000 {
                    compatible = "ti,am3356-pruss";
                    reg = <0x4a300000 0x000001d6>;
                    interrupts = <0x00000014 0x00000018 0x00000003 0x3200686f 0x73743500 0x686f7374 0x00000004 0x00000004>;
                    interrupt-names = "host2", "host3", "host4", "host5", "host6", "host7", "host8", "host9";
                    #address-cells = <0x00000001>;
                    #size-cells = <0x00000001>;
                    ranges;
                    status = "disabled";
                    phandle = <0x000000d7>;
                    memories@4a300000 {
                        reg = <0x4a300000 0x4a310000 0x000001fa 0x73687264 0x00000004 0x00000001>;
                        reg-names = "dram0", "dram1", "shrdram2";
                        phandle = <0x000000d8>;
                    };
                    cfg@4a326000 {
                        compatible = "syscon";
                        reg = <0x4a326000 0x000000e4>;
                        phandle = <0x000000d9>;
                    };
                    iep@4a32e000 {
                        compatible = "syscon";
                        reg = <0x4a32e000 0x000000e4>;
                        phandle = <0x000000da>;
                    };
                    mii-rt@4a332000 {
                        compatible = "syscon";
                        reg = <0x4a332000 0x000000e4>;
                        phandle = <0x000000db>;
                    };
                    interrupt-controller@4a320000 {
                        compatible = "ti,am3356-pruss-intc";
                        reg = <0x4a320000 0x0000032a>;
                        interrupt-controller;
                        #interrupt-cells = <0x00000001>;
                        phandle = <0x00000045>;
                    };
                    pru@4a334000 {
                        compatible = "ti,am3356-pru";
                        reg = <0x4a334000 0x4a322400 0x000001fa 0x00646562 0x000006cf 0x66770000>;
                        reg-names = "iram", "control", "debug";
                        firmware-name = "am335x-pru0-fw";
                        interrupt-parent = <0x00000045>;
                        interrupts = <0x00000010 0x00000350>;
                        interrupt-names = "vring", "kick";
                        phandle = <0x000000dc>;
                    };
                    pru@4a338000 {
                        compatible = "ti,am3356-pru";
                        reg = <0x4a338000 0x4a324400 0x000001fa 0x00646562 0x000006cf 0x66770000>;
                        reg-names = "iram", "control", "debug";
                        firmware-name = "am335x-pru1-fw";
                        interrupt-parent = <0x00000045>;
                        interrupts = <0x00000012 0x00000350>;
                        interrupt-names = "vring", "kick";
                        phandle = <0x000000dd>;
                    };
                    mdio@4a332400 {
                        compatible = "ti,davinci_mdio";
                        reg = <0x4a332400 0x0000011e>;
                        clocks = <0x00000012>;
                        clock-names = "fck";
                        bus_freq = <0x000f4240>;
                        #address-cells = <0x00000001>;
                        #size-cells = <0x00000000>;
                        status = "disabled";
                        phandle = <0x000000de>;
                    };
                };
            };
            elm@48080000 {
                compatible = "ti,am3352-elm";
                reg = <0x48080000 0x000001d6>;
                interrupts = <0x00000004>;
                ti,hwmods = "elm";
                status = "disabled";
                phandle = <0x000000df>;
            };
            lcdc@4830e000 {
                compatible = "ti,am33xx-tilcdc";
                reg = <0x4830e000 0x000001d6>;
                interrupts = <0x00000024>;
                ti,hwmods = "lcdc";
                status = "disabled";
                phandle = <0x000000e0>;
            };
            tscadc@44e0d000 {
                compatible = "ti,am3359-tscadc";
                reg = <0x44e0d000 0x000001d6>;
                interrupts = <0x00000010>;
                ti,hwmods = "adc_tsc";
                status = "disabled";
                dmas = <0x00000027 0x00000039 0x000003b9 0x00000003 0x00000001 0x00000000>;
                dma-names = "fifo0", "fifo1";
                phandle = <0x000000e1>;
                tsc {
                    compatible = "ti,am3359-tsc";
                };
                adc {
                    #io-channel-cells = <0x00000001>;
                    compatible = "ti,am3359-adc";
                    phandle = <0x000000e2>;
                };
            };
            emif@4c000000 {
                compatible = "ti,emif-am3352";
                reg = <0x4c000000 0x000001e1>;
                ti,hwmods = "emif";
                interrupts = <0x00000065>;
                sram = <0x00000007 0x000006ef>;
                ti,no-idle;
                phandle = <0x000000e3>;
            };
            gpmc@50000000 {
                compatible = "ti,am3352-gpmc";
                ti,hwmods = "gpmc";
                ti,no-idle-on-init;
                reg = <0x50000000 0x000001d6>;
                interrupts = <0x00000064>;
                dmas = <0x00000027 0x00000005 0x00000003>;
                dma-names = "rxtx";
                gpmc,num-cs = <0x00000007>;
                gpmc,num-waitpins = <0x00000002>;
                #address-cells = <0x00000002>;
                #size-cells = <0x00000001>;
                interrupt-controller;
                #interrupt-cells = <0x00000002>;
                gpio-controller;
                #gpio-cells = <0x00000002>;
                status = "disabled";
                phandle = <0x000000e4>;
            };
            sham@53100000 {
                compatible = "ti,omap4-sham";
                ti,hwmods = "sham";
                reg = <0x53100000 0x000001d6>;
                interrupts = <0x0000006d>;
                dmas = <0x00000027 0x00000003 0x00000005>;
                dma-names = "rx";
                status = "okay";
                phandle = <0x000000e5>;
            };
            aes@53500000 {
                compatible = "ti,omap4-aes";
                ti,hwmods = "aes";
                reg = <0x53500000 0x000001d6>;
                interrupts = <0x00000067>;
                dmas = <0x00000027 0x00000005 0x000003b9 0x00000005 0x00000003 0x00000002>;
                dma-names = "tx", "rx";
                status = "okay";
                phandle = <0x000000e6>;
            };
            mcasp@48038000 {
                compatible = "ti,am33xx-mcasp-audio";
                ti,hwmods = "mcasp0";
                reg = <0x48038000 0x00000003 0x64617400 0x00000050>;
                reg-names = "mpu", "dat";
                interrupts = <0x00000050 0x00000350>;
                interrupt-names = "tx", "rx";
                status = "okay";
                dmas = <0x00000027 0x00000009 0x000003b9 0x0000000e 0x736c6565 0x000003d1>;
                dma-names = "tx", "rx";
                pinctrl-names = "default", "sleep";
                pinctrl-0 = <0x00000046>;
                pinctrl-1 = <0x00000047>;
                op-mode = <0x00000000>;
                tdm-slots = <0x00000002>;
                serial-dir = <0x00000002 0x00000003 0x00000003 0x00000003>;
                tx-num-evt = <0x00000001>;
                rx-num-evt = <0x00000001>;
                phandle = <0x000000e7>;
            };
            mcasp@4803c000 {
                compatible = "ti,am33xx-mcasp-audio";
                ti,hwmods = "mcasp1";
                reg = <0x4803c000 0x00000003 0x64617400 0x00000052>;
                reg-names = "mpu", "dat";
                interrupts = <0x00000052 0x00000350>;
                interrupt-names = "tx", "rx";
                status = "disabled";
                dmas = <0x00000027 0x0000000b 0x000003b9 0x00000004 0x00000001 0x00000000>;
                dma-names = "tx", "rx";
                phandle = <0x000000e8>;
            };
            rng@48310000 {
                compatible = "ti,omap4-rng";
                ti,hwmods = "rng";
                reg = <0x48310000 0x000001d6>;
                interrupts = <0x0000006f>;
                phandle = <0x000000e9>;
            };
            sgx@56000000 {
                compatible = "ti,am3352-sgx530", "img,sgx530";
                ti,hwmods = "gfx";
                reg = <0x56000000 0x000001d6>;
                interrupts = <0x00000025>;
                clocks = <0x00000048>;
                clock-names = "fclk";
                status = "disabled";
                phandle = <0x000000ea>;
            };
            P9_19_pinmux {
                compatible = "bone-pinmux-helper";
                status = "okay";
                pinctrl-names = "default", "gpio", "gpio_pu", "gpio_pd", "gpio_input", "spi_cs", "can", "i2c", "pru_uart", "timer";
                pinctrl-0 = <0x00000049>;
                pinctrl-1 = <0x0000004a>;
                pinctrl-2 = <0x0000004b>;
                pinctrl-3 = <0x0000004c>;
                pinctrl-4 = <0x0000004d>;
                pinctrl-5 = <0x0000004e>;
                pinctrl-6 = <0x0000004f>;
                pinctrl-7 = <0x00000050>;
                pinctrl-8 = <0x00000051>;
                pinctrl-9 = <0x00000052>;
            };
            P9_20_pinmux {
                compatible = "bone-pinmux-helper";
                status = "okay";
                pinctrl-names = "default", "gpio", "gpio_pu", "gpio_pd", "gpio_input", "spi_cs", "can", "i2c", "pru_uart", "timer";
                pinctrl-0 = <0x00000053>;
                pinctrl-1 = <0x00000054>;
                pinctrl-2 = <0x00000055>;
                pinctrl-3 = <0x00000056>;
                pinctrl-4 = <0x00000057>;
                pinctrl-5 = <0x00000058>;
                pinctrl-6 = <0x00000059>;
                pinctrl-7 = <0x0000005a>;
                pinctrl-8 = <0x0000005b>;
                pinctrl-9 = <0x0000005c>;
            };
            cape-universal {
                compatible = "gpio-of-helper";
                status = "okay";
                pinctrl-names = "default";
                pinctrl-0;
                P9_19 {
                    gpio-name = "P9_19";
                    gpio = <0x0000002f 0x00000000 0x000007c3>;
                    input;
                    dir-changeable;
                };
                P9_20 {
                    gpio-name = "P9_20";
                    gpio = <0x0000002f 0x00000000 0x000007c3>;
                    input;
                    dir-changeable;
                };
            };
        };
        memory@80000000 {
            device_type = "memory";
            reg = <0x80000000 0x6c656473>;
        };
        leds {
            pinctrl-names = "default";
            pinctrl-0 = <0x0000005d>;
            compatible = "gpio-leds";
            led2 {
                label = "beaglebone:green:usr0";
                gpios = <0x0000005e 0x0000000a 0x74000000>;
                linux,default-trigger = "heartbeat";
                default-state = "off";
            };
            led3 {
                label = "beaglebone:green:usr1";
                gpios = <0x0000005e 0x00000005 0x00000003>;
                linux,default-trigger = "mmc0";
                default-state = "off";
            };
            led4 {
                label = "beaglebone:green:usr2";
                gpios = <0x0000005e 0x00000005 0x00000003>;
                linux,default-trigger = "cpu0";
                default-state = "off";
            };
            led5 {
                label = "beaglebone:green:usr3";
                gpios = <0x0000005e 0x00000005 0x00000003>;
                linux,default-trigger = "mmc1";
                default-state = "off";
            };
        };
        fixedregulator0 {
            compatible = "regulator-fixed";
            regulator-name = "vmmcsd_fixed";
            regulator-min-microvolt = <0x00325aa0>;
            regulator-max-microvolt = <0x00325aa0>;
            phandle = <0x00000030>;
        };
        spi_gpio {
            compatible = "spi-gpio";
            #address-cells = <0x00000001>;
            ranges;
            gpio-sck = <0x0000002f 0x0000000c 0x00000000>;
            gpio-mosi = <0x0000002f 0x0000000c 0x00000000>;
            gpio-miso = <0x0000002f 0x00000018 0x00000000>;
            cs-gpios = <0x0000002f 0x0000000a 0x00000822 0x000003ad 0x00000003 0x00000002>;
            num-chipselects = <0x00000002>;
            status = "disabled";
            phandle = <0x000000eb>;
        };
        clk_mcasp0_fixed {
            #clock-cells = <0x00000000>;
            compatible = "fixed-clock";
            clock-frequency = <0x01770000>;
            phandle = <0x0000005f>;
        };
        clk_mcasp0 {
            #clock-cells = <0x00000000>;
            compatible = "gpio-gate-clock";
            clocks = <0x0000005f>;
            enable-gpios = <0x0000005e 0x00000004 0x00000001>;
            phandle = <0x000000ec>;
        };
        __symbols__ {
            aliases = "/aliases";
            mpu_gate = "/cpus/idle-states/mpu_gate";
            cpu0_opp_table = "/opp-table";
            ocp = "/ocp";
            l4_wkup = "/ocp/l4_wkup@44c00000";
            wkup_m3 = "/ocp/l4_wkup@44c00000/wkup_m3@100000";
            prcm = "/ocp/l4_wkup@44c00000/prcm@200000";
            prcm_clocks = "/ocp/l4_wkup@44c00000/prcm@200000/clocks";
            clk_32768_ck = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/clk_32768_ck";
            clk_rc32k_ck = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/clk_rc32k_ck";
            virt_19200000_ck = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/virt_19200000_ck";
            virt_24000000_ck = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/virt_24000000_ck";
            virt_25000000_ck = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/virt_25000000_ck";
            virt_26000000_ck = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/virt_26000000_ck";
            tclkin_ck = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/tclkin_ck";
            dpll_core_ck = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/dpll_core_ck@490";
            dpll_core_x2_ck = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/dpll_core_x2_ck";
            dpll_core_m4_ck = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/dpll_core_m4_ck@480";
            dpll_core_m5_ck = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/dpll_core_m5_ck@484";
            dpll_core_m6_ck = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/dpll_core_m6_ck@4d8";
            dpll_mpu_ck = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/dpll_mpu_ck@488";
            dpll_mpu_m2_ck = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/dpll_mpu_m2_ck@4a8";
            dpll_ddr_ck = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/dpll_ddr_ck@494";
            dpll_ddr_m2_ck = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/dpll_ddr_m2_ck@4a0";
            dpll_ddr_m2_div2_ck = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/dpll_ddr_m2_div2_ck";
            dpll_disp_ck = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/dpll_disp_ck@498";
            dpll_disp_m2_ck = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/dpll_disp_m2_ck@4a4";
            dpll_per_ck = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/dpll_per_ck@48c";
            dpll_per_m2_ck = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/dpll_per_m2_ck@4ac";
            dpll_per_m2_div4_wkupdm_ck = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/dpll_per_m2_div4_wkupdm_ck";
            dpll_per_m2_div4_ck = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/dpll_per_m2_div4_ck";
            clk_24mhz = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/clk_24mhz";
            clkdiv32k_ck = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/clkdiv32k_ck";
            l3_gclk = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/l3_gclk";
            pruss_ocp_gclk = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/pruss_ocp_gclk@530";
            mmu_fck = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/mmu_fck@914";
            timer1_fck = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/timer1_fck@528";
            timer2_fck = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/timer2_fck@508";
            timer3_fck = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/timer3_fck@50c";
            timer4_fck = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/timer4_fck@510";
            timer5_fck = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/timer5_fck@518";
            timer6_fck = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/timer6_fck@51c";
            timer7_fck = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/timer7_fck@504";
            usbotg_fck = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/usbotg_fck@47c";
            dpll_core_m4_div2_ck = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/dpll_core_m4_div2_ck";
            ieee5000_fck = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/ieee5000_fck@e4";
            wdt1_fck = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/wdt1_fck@538";
            l4_rtc_gclk = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/l4_rtc_gclk";
            l4hs_gclk = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/l4hs_gclk";
            l3s_gclk = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/l3s_gclk";
            l4fw_gclk = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/l4fw_gclk";
            l4ls_gclk = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/l4ls_gclk";
            sysclk_div_ck = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/sysclk_div_ck";
            cpsw_125mhz_gclk = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/cpsw_125mhz_gclk";
            cpsw_cpts_rft_clk = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/cpsw_cpts_rft_clk@520";
            gpio0_dbclk_mux_ck = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/gpio0_dbclk_mux_ck@53c";
            lcd_gclk = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/lcd_gclk@534";
            mmc_clk = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/mmc_clk";
            gfx_fclk_clksel_ck = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/gfx_fclk_clksel_ck@52c";
            gfx_fck_div_ck = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/gfx_fck_div_ck@52c";
            sysclkout_pre_ck = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/sysclkout_pre_ck@700";
            clkout2_div_ck = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/clkout2_div_ck@700";
            clkout2_ck = "/ocp/l4_wkup@44c00000/prcm@200000/clocks/clkout2_ck@700";
            prcm_clockdomains = "/ocp/l4_wkup@44c00000/prcm@200000/clockdomains";
            l4_per_cm = "/ocp/l4_wkup@44c00000/prcm@200000/l4_per_cm@0";
            l4_per_clkctrl = "/ocp/l4_wkup@44c00000/prcm@200000/l4_per_cm@0/clk@14";
            l4_wkup_cm = "/ocp/l4_wkup@44c00000/prcm@200000/l4_wkup_cm@400";
            l4_wkup_clkctrl = "/ocp/l4_wkup@44c00000/prcm@200000/l4_wkup_cm@400/clk@4";
            mpu_cm = "/ocp/l4_wkup@44c00000/prcm@200000/mpu_cm@600";
            mpu_clkctrl = "/ocp/l4_wkup@44c00000/prcm@200000/mpu_cm@600/clk@4";
            l4_rtc_cm = "/ocp/l4_wkup@44c00000/prcm@200000/l4_rtc_cm@800";
            l4_rtc_clkctrl = "/ocp/l4_wkup@44c00000/prcm@200000/l4_rtc_cm@800/clk@0";
            gfx_l3_cm = "/ocp/l4_wkup@44c00000/prcm@200000/gfx_l3_cm@900";
            gfx_l3_clkctrl = "/ocp/l4_wkup@44c00000/prcm@200000/gfx_l3_cm@900/clk@4";
            l4_cefuse_cm = "/ocp/l4_wkup@44c00000/prcm@200000/l4_cefuse_cm@a00";
            l4_cefuse_clkctrl = "/ocp/l4_wkup@44c00000/prcm@200000/l4_cefuse_cm@a00/clk@20";
            scm = "/ocp/l4_wkup@44c00000/scm@210000";
            am33xx_pinmux = "/ocp/l4_wkup@44c00000/scm@210000/pinmux@800";
            user_leds_s0 = "/ocp/l4_wkup@44c00000/scm@210000/pinmux@800/user_leds_s0";
            i2c0_pins = "/ocp/l4_wkup@44c00000/scm@210000/pinmux@800/pinmux_i2c0_pins";
            uart0_pins = "/ocp/l4_wkup@44c00000/scm@210000/pinmux@800/pinmux_uart0_pins";
            cpsw_default = "/ocp/l4_wkup@44c00000/scm@210000/pinmux@800/cpsw_default";
            cpsw_sleep = "/ocp/l4_wkup@44c00000/scm@210000/pinmux@800/cpsw_sleep";
            davinci_mdio_default = "/ocp/l4_wkup@44c00000/scm@210000/pinmux@800/davinci_mdio_default";
            davinci_mdio_sleep = "/ocp/l4_wkup@44c00000/scm@210000/pinmux@800/davinci_mdio_sleep";
            mmc1_pins = "/ocp/l4_wkup@44c00000/scm@210000/pinmux@800/pinmux_mmc1_pins";
            emmc_pins = "/ocp/l4_wkup@44c00000/scm@210000/pinmux@800/pinmux_emmc_pins";
            P9_19_default_pin = "/ocp/l4_wkup@44c00000/scm@210000/pinmux@800/pinmux_P9_19_default_pin";
            P9_19_gpio_pin = "/ocp/l4_wkup@44c00000/scm@210000/pinmux@800/pinmux_P9_19_gpio_pin";
            P9_19_gpio_pu_pin = "/ocp/l4_wkup@44c00000/scm@210000/pinmux@800/pinmux_P9_19_gpio_pu_pin";
            P9_19_gpio_pd_pin = "/ocp/l4_wkup@44c00000/scm@210000/pinmux@800/pinmux_P9_19_gpio_pd_pin";
            P9_19_gpio_input_pin = "/ocp/l4_wkup@44c00000/scm@210000/pinmux@800/pinmux_P9_19_gpio_input_pin";
            P9_19_timer_pin = "/ocp/l4_wkup@44c00000/scm@210000/pinmux@800/pinmux_P9_19_timer_pin";
            P9_19_can_pin = "/ocp/l4_wkup@44c00000/scm@210000/pinmux@800/pinmux_P9_19_can_pin";
            P9_19_i2c_pin = "/ocp/l4_wkup@44c00000/scm@210000/pinmux@800/pinmux_P9_19_i2c_pin";
            P9_19_spi_cs_pin = "/ocp/l4_wkup@44c00000/scm@210000/pinmux@800/pinmux_P9_19_spi_cs_pin";
            P9_19_pru_uart_pin = "/ocp/l4_wkup@44c00000/scm@210000/pinmux@800/pinmux_P9_19_pru_uart_pin";
            P9_20_default_pin = "/ocp/l4_wkup@44c00000/scm@210000/pinmux@800/pinmux_P9_20_default_pin";
            P9_20_gpio_pin = "/ocp/l4_wkup@44c00000/scm@210000/pinmux@800/pinmux_P9_20_gpio_pin";
            P9_20_gpio_pu_pin = "/ocp/l4_wkup@44c00000/scm@210000/pinmux@800/pinmux_P9_20_gpio_pu_pin";
            P9_20_gpio_pd_pin = "/ocp/l4_wkup@44c00000/scm@210000/pinmux@800/pinmux_P9_20_gpio_pd_pin";
            P9_20_gpio_input_pin = "/ocp/l4_wkup@44c00000/scm@210000/pinmux@800/pinmux_P9_20_gpio_input_pin";
            P9_20_timer_pin = "/ocp/l4_wkup@44c00000/scm@210000/pinmux@800/pinmux_P9_20_timer_pin";
            P9_20_can_pin = "/ocp/l4_wkup@44c00000/scm@210000/pinmux@800/pinmux_P9_20_can_pin";
            P9_20_i2c_pin = "/ocp/l4_wkup@44c00000/scm@210000/pinmux@800/pinmux_P9_20_i2c_pin";
            P9_20_spi_cs_pin = "/ocp/l4_wkup@44c00000/scm@210000/pinmux@800/pinmux_P9_20_spi_cs_pin";
            P9_20_pru_uart_pin = "/ocp/l4_wkup@44c00000/scm@210000/pinmux@800/pinmux_P9_20_pru_uart_pin";
            mcasp0_pins = "/ocp/l4_wkup@44c00000/scm@210000/pinmux@800/mcasp0_pins";
            mcasp0_pins_sleep = "/ocp/l4_wkup@44c00000/scm@210000/pinmux@800/mcasp0_pins_sleep";
            scm_conf = "/ocp/l4_wkup@44c00000/scm@210000/scm_conf@0";
            scm_clocks = "/ocp/l4_wkup@44c00000/scm@210000/scm_conf@0/clocks";
            sys_clkin_ck = "/ocp/l4_wkup@44c00000/scm@210000/scm_conf@0/clocks/sys_clkin_ck@40";
            adc_tsc_fck = "/ocp/l4_wkup@44c00000/scm@210000/scm_conf@0/clocks/adc_tsc_fck";
            dcan0_fck = "/ocp/l4_wkup@44c00000/scm@210000/scm_conf@0/clocks/dcan0_fck";
            dcan1_fck = "/ocp/l4_wkup@44c00000/scm@210000/scm_conf@0/clocks/dcan1_fck";
            mcasp0_fck = "/ocp/l4_wkup@44c00000/scm@210000/scm_conf@0/clocks/mcasp0_fck";
            mcasp1_fck = "/ocp/l4_wkup@44c00000/scm@210000/scm_conf@0/clocks/mcasp1_fck";
            smartreflex0_fck = "/ocp/l4_wkup@44c00000/scm@210000/scm_conf@0/clocks/smartreflex0_fck";
            smartreflex1_fck = "/ocp/l4_wkup@44c00000/scm@210000/scm_conf@0/clocks/smartreflex1_fck";
            sha0_fck = "/ocp/l4_wkup@44c00000/scm@210000/scm_conf@0/clocks/sha0_fck";
            aes0_fck = "/ocp/l4_wkup@44c00000/scm@210000/scm_conf@0/clocks/aes0_fck";
            rng_fck = "/ocp/l4_wkup@44c00000/scm@210000/scm_conf@0/clocks/rng_fck";
            ehrpwm0_tbclk = "/ocp/l4_wkup@44c00000/scm@210000/scm_conf@0/clocks/ehrpwm0_tbclk@44e10664";
            ehrpwm1_tbclk = "/ocp/l4_wkup@44c00000/scm@210000/scm_conf@0/clocks/ehrpwm1_tbclk@44e10664";
            ehrpwm2_tbclk = "/ocp/l4_wkup@44c00000/scm@210000/scm_conf@0/clocks/ehrpwm2_tbclk@44e10664";
            wkup_m3_ipc = "/ocp/l4_wkup@44c00000/scm@210000/wkup_m3_ipc@1324";
            edma_xbar = "/ocp/l4_wkup@44c00000/scm@210000/dma-router@f90";
            scm_clockdomains = "/ocp/l4_wkup@44c00000/scm@210000/clockdomains";
            intc = "/ocp/interrupt-controller@48200000";
            edma = "/ocp/edma@49000000";
            edma_tptc0 = "/ocp/tptc@49800000";
            edma_tptc1 = "/ocp/tptc@49900000";
            edma_tptc2 = "/ocp/tptc@49a00000";
            gpio0 = "/ocp/gpio@44e07000";
            gpio1 = "/ocp/gpio@4804c000";
            gpio2 = "/ocp/gpio@481ac000";
            gpio3 = "/ocp/gpio@481ae000";
            uart0 = "/ocp/serial@44e09000";
            uart1 = "/ocp/serial@48022000";
            uart2 = "/ocp/serial@48024000";
            uart3 = "/ocp/serial@481a6000";
            uart4 = "/ocp/serial@481a8000";
            uart5 = "/ocp/serial@481aa000";
            i2c0 = "/ocp/i2c@44e0b000";
            tps = "/ocp/i2c@44e0b000/tps@24";
            dcdc1_reg = "/ocp/i2c@44e0b000/tps@24/regulators/regulator@0";
            dcdc2_reg = "/ocp/i2c@44e0b000/tps@24/regulators/regulator@1";
            dcdc3_reg = "/ocp/i2c@44e0b000/tps@24/regulators/regulator@2";
            ldo1_reg = "/ocp/i2c@44e0b000/tps@24/regulators/regulator@3";
            ldo2_reg = "/ocp/i2c@44e0b000/tps@24/regulators/regulator@4";
            ldo3_reg = "/ocp/i2c@44e0b000/tps@24/regulators/regulator@5";
            ldo4_reg = "/ocp/i2c@44e0b000/tps@24/regulators/regulator@6";
            baseboard_eeprom = "/ocp/i2c@44e0b000/baseboard_eeprom@50";
            baseboard_data = "/ocp/i2c@44e0b000/baseboard_eeprom@50/baseboard_data@0";
            i2c1 = "/ocp/i2c@4802a000";
            i2c2 = "/ocp/i2c@4819c000";
            cape_eeprom0 = "/ocp/i2c@4819c000/cape_eeprom0@54";
            cape0_data = "/ocp/i2c@4819c000/cape_eeprom0@54/cape_data@0";
            cape_eeprom1 = "/ocp/i2c@4819c000/cape_eeprom1@55";
            cape1_data = "/ocp/i2c@4819c000/cape_eeprom1@55/cape_data@0";
            cape_eeprom2 = "/ocp/i2c@4819c000/cape_eeprom2@56";
            cape2_data = "/ocp/i2c@4819c000/cape_eeprom2@56/cape_data@0";
            cape_eeprom3 = "/ocp/i2c@4819c000/cape_eeprom3@57";
            cape3_data = "/ocp/i2c@4819c000/cape_eeprom3@57/cape_data@0";
            mmc1 = "/ocp/mmc@48060000";
            mmc2 = "/ocp/mmc@481d8000";
            mmc3 = "/ocp/mmc@47810000";
            hwspinlock = "/ocp/spinlock@480ca000";
            wdt2 = "/ocp/wdt@44e35000";
            dcan0 = "/ocp/can@481cc000";
            dcan1 = "/ocp/can@481d0000";
            mailbox = "/ocp/mailbox@480c8000";
            mbox_wkupm3 = "/ocp/mailbox@480c8000/wkup_m3";
            timer1 = "/ocp/timer@44e31000";
            timer2 = "/ocp/timer@48040000";
            timer3 = "/ocp/timer@48042000";
            timer4 = "/ocp/timer@48044000";
            timer5 = "/ocp/timer@48046000";
            timer6 = "/ocp/timer@48048000";
            timer7 = "/ocp/timer@4804a000";
            rtc = "/ocp/rtc@44e3e000";
            spi0 = "/ocp/spi@48030000";
            spi1 = "/ocp/spi@481a0000";
            usb = "/ocp/usb@47400000";
            usb_ctrl_mod = "/ocp/usb@47400000/control@44e10620";
            usb0_phy = "/ocp/usb@47400000/usb-phy@47401300";
            usb0 = "/ocp/usb@47400000/usb@47401000";
            usb1_phy = "/ocp/usb@47400000/usb-phy@47401b00";
            usb1 = "/ocp/usb@47400000/usb@47401800";
            cppi41dma = "/ocp/usb@47400000/dma-controller@47402000";
            epwmss0 = "/ocp/epwmss@48300000";
            ecap0 = "/ocp/epwmss@48300000/ecap@48300100";
            eqep0 = "/ocp/epwmss@48300000/eqep@0x48300180";
            ehrpwm0 = "/ocp/epwmss@48300000/pwm@48300200";
            epwmss1 = "/ocp/epwmss@48302000";
            ecap1 = "/ocp/epwmss@48302000/ecap@48302100";
            eqep1 = "/ocp/epwmss@48302000/eqep@0x48302180";
            ehrpwm1 = "/ocp/epwmss@48302000/pwm@48302200";
            epwmss2 = "/ocp/epwmss@48304000";
            ecap2 = "/ocp/epwmss@48304000/ecap@48304100";
            eqep2 = "/ocp/epwmss@48304000/eqep@0x48304180";
            ehrpwm2 = "/ocp/epwmss@48304000/pwm@48304200";
            mac = "/ocp/ethernet@4a100000";
            davinci_mdio = "/ocp/ethernet@4a100000/mdio@4a101000";
            cpsw_emac0 = "/ocp/ethernet@4a100000/slave@4a100200";
            cpsw_emac1 = "/ocp/ethernet@4a100000/slave@4a100300";
            phy_sel = "/ocp/ethernet@4a100000/cpsw-phy-sel@44e10650";
            ocmcram = "/ocp/ocmcram@40300000";
            pm_sram_code = "/ocp/ocmcram@40300000/pm-sram-code@0";
            pm_sram_data = "/ocp/ocmcram@40300000/pm-sram-data@1000";
            pruss_soc_bus = "/ocp/pruss-soc-bus@4a326004";
            pruss = "/ocp/pruss-soc-bus@4a326004/pruss@4a300000";
            pruss_mem = "/ocp/pruss-soc-bus@4a326004/pruss@4a300000/memories@4a300000";
            pruss_cfg = "/ocp/pruss-soc-bus@4a326004/pruss@4a300000/cfg@4a326000";
            pruss_iep = "/ocp/pruss-soc-bus@4a326004/pruss@4a300000/iep@4a32e000";
            pruss_mii_rt = "/ocp/pruss-soc-bus@4a326004/pruss@4a300000/mii-rt@4a332000";
            pruss_intc = "/ocp/pruss-soc-bus@4a326004/pruss@4a300000/interrupt-controller@4a320000";
            pru0 = "/ocp/pruss-soc-bus@4a326004/pruss@4a300000/pru@4a334000";
            pru1 = "/ocp/pruss-soc-bus@4a326004/pruss@4a300000/pru@4a338000";
            pruss_mdio = "/ocp/pruss-soc-bus@4a326004/pruss@4a300000/mdio@4a332400";
            elm = "/ocp/elm@48080000";
            lcdc = "/ocp/lcdc@4830e000";
            tscadc = "/ocp/tscadc@44e0d000";
            am335x_adc = "/ocp/tscadc@44e0d000/adc";
            emif = "/ocp/emif@4c000000";
            gpmc = "/ocp/gpmc@50000000";
            sham = "/ocp/sham@53100000";
            aes = "/ocp/aes@53500000";
            mcasp0 = "/ocp/mcasp@48038000";
            mcasp1 = "/ocp/mcasp@4803c000";
            rng = "/ocp/rng@48310000";
            sgx = "/ocp/sgx@56000000";
            vmmcsd_fixed = "/fixedregulator0";
            spi_gpio = "/spi_gpio";
            clk_mcasp0_fixed = "/clk_mcasp0_fixed";
            clk_mcasp0 = "/clk_mcasp0";
        };
    };
```