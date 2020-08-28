# CTAG Face Alsa Driver

The CTAG Face 2|4 is an audio card for Beaglebone devices, which is equipped with 2 input and 4 output channels. One of the goals for this GSoC project was to port the CTAG Face driver for usage on the BBAI and the BBB to linux kernel 4.19-rt.

WARNING: To use the CTAG Face with the BBAI, stackable headers need to be used. This is due to the height of the passive cooling unit on the CPU.

## Adjustments

Here the adjustments, which needed to be made are shown. During the project, I realized, that the linux kernel 4.19-rt is still not running stable on BBAI (As of August, 22 2020). Because of that, I also tried to port the driver to kernel version 5.4-rt, this however failed, because it took too long to adjust for the API changes in ALSA-ASoC API from 4.19 to 5.4.
For both versions, the codec driver of the AD1938 had to be adjusted. This is due to the changes in how overlays are handled on BeagleBoards since kernel version 4.4. Back in that time overlays were loaded using the bone capemanager. A script was handled, how the face was loaded back then. There the reset pin (P8.34) had to be set to HIGH and then to LOW. This is needed to properly start the codec. Because the capemanager is now deprecated the reset functionality is now implemented in the ```sound/soc/codecs/ad193x.c``` file. The probe function of the codec is now responsible to reset the codec at startup.

## Information on Overlays
During the course of GSoC, another GSoC student at BeagleBoard.org Deepak Khatri worked on a compatibility layer for overlays. Therefore he ported the overlays I created for BBB and BBAI which can be found [here](https://github.com/NiklasWan/bb.org-overlays/tree/dev_gsoc_face) to the new overlay repository. The current status is the compatibility update 3 on his repository, which can be found [here](https://github.com/lorforlinux/BeagleBoard-DeviceTrees/tree/compatibility_Update3). The documentation on how overlays are handled now can be found [here](https://deepaklorkhatri.me/GSoC2020_BeagleBoard.org/).
Links to overlays:

- The implementation of the overlay BBAI (old way) can be found [here](https://github.com/NiklasWan/bb.org-overlays/blob/dev_gsoc_face/src/arm/BBAI_BB-BONE-FACE-8CH-00A0.dts)
- The implementation of the overlay BBB (old way) can be found [here](https://github.com/NiklasWan/bb.org-overlays/blob/dev_gsoc_face/src/arm/BB-CTAG-SW-8CH-00A0.dts)
- The implementation for both overlays (new way) can be found [here](https://github.com/NiklasWan/BeagleBoard-DeviceTrees/blob/dev_face_fix/src/arm/overlays/BB-CTAG-SW-8CH-00A0.dts)

## Drivers for 4.19-rt
The implementation of the driver code can be found [here](https://github.com/NiklasWan/linux/tree/dev_gsoc_face_4.19-rt)

## Drivers for 5.4-rt
The implementation for the driver code can be found [here](https://github.com/NiklasWan/linux/tree/dev_gsoc_face_5.4-rt)

Known issues:
- driver loads and ```aplay -l``` and ```arecord -l``` list the CTAG Face card, however it can not be controlled via ```amixer``` or ```alsamixer``. This is due to the spi-gpio bitbang driver being not loaded

## Compilation and installation of the kernel module

The CTAG Face Alsa Driver runs on 4.19-rt, for both BBB and BBAI. To get the Face running on your device, please follow all steps.

IMPORTANT NOTE: The uboot overlay process changed during the course of the project, both the BBAI and BBB version have been tested and are working. For installation on BBB the whole setup has to be done on the internal eMMC chip instead of an micro SD card.

1.) Clone kernel from [here](https://github.com/NiklasWan/linux.git)
    
    git clone https://github.com/NiklasWan/linux.git && cd linux && git chekout dev_gsoc_face_4.19-rt
2.) Load BB device configuration:
    
    make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- bb.org_defconfig
3.) Open menuconfig and Go to Device Drivers - Sound Card Support - Advanced Linux Sound Architecture - ALSA for SoC audio support:
    Enable module build for SoC Audio Support for CTAG face-2-4 Audio Card (AD1938)

4.) Build kernel:
    
    make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- bindeb-pkg -j8
5.) Download current Image for your device at: [Latest images](https://beagleboard.org/latest-images)

6.) Install image using Balena Etcher

7.) Copy built linux image, linux header and linux libc to sd card:
    
    sudo cp linux-* /media/dev/rootfs/home/debian
8.) Start BB from Sd Card and Install kernel:
    
    sudo dpkg -i linux-headers*
    
    sudo dpkg -i linux-libc*
    
    sudo dpkg -i linux-image*
9.) Restart:

    sudo reboot

10.) Install bootloader update:

    sudo ./opt/scripts/tools/developers/update_bootloaders.sh

11.) Update system:

    sudo apt update && sudo apt upgrade
12.) Clone overlay repo:
    
    git clone https://github.com/NiklasWan/BeagleBoard-DeviceTrees.git && cd BeagleBoard-DeviceTrees && git checkout compatibility_Update3

13.) Install overlays:
    
    sudo make install
13.) Edit Uboot uEnv.txt:

    For BBAI:
      add the following lines to /boot/uEnv.txt:
        enable_uboot_overlays=1
        uboot_overlay_addr0=BB-CTAG-SW-8CH-00A0.dtbo
    For BBB:
      uncomment:
        
        disable_uboot_overlay_video=1
        disable_uboot_overlay_audio=1
      add: 
        
        uboot_overlay_addr0=BB-CTAG-SW-8CH-00A0.dtbo

14.) Clone https://github.com/ctag-fh-kiel/ctag-face-2-4.git:
  
    git clone https://github.com/ctag-fh-kiel/ctag-face-2-4.git & cd ctag-face-2-4
15.) Install alsa config:
  
    cd alsa-configs & sudo cp asound.conf.8ch ~/.asoundrc
16.) Reboot your board and use the audio card :-)