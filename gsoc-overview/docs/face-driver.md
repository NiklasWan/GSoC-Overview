# CTAG Face Alsa Driver

The CTAG Face Alsa Driver runs on 4.19-rt, for both BBB and BBAI. To get the Face running on your device, please follow all steps.

1.) Clone kernel from [here](https://github.com/NiklasWan/linux/tree/dev_gsoc_face_4.19-rt)
    
    git clone https://github.com/NiklasWan/linux/tree/dev_gsoc_face_4.19-rt
2.) Load BB device configuration:
    
    make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- bb.org_defconfig
3.) Open menuconfig and Go to Device Drivers - Sound Card Support - Advanced Linux Sound Architecture - ALSA for SoC audio support:
    Enable module build for SoC Audio Support for CTAG face-2-4 Audio Card (AD1938)
4.) Build kernel:
    
    make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- bindeb-pkg -j8
5.) Download current Image for your device at: https://beagleboard.org/latest-images
6.) Install image using Balena Etcher
7.) Copy built linux image, linux header and linux libc to sd card:
    
    sudo cp linux-* /media/dev/rootfs/home/debian
8.) Start BB from Sd Card and Install kernel:
    
    sudo dpkg -i linux-headers*
    
    sudo dpkg -i linux-libc*
    
    sudo dpkg -i linux-image*
9.) Restart:
    sudo restart
10.) Clone overlay repo:
    
    git clone https://github.com/NiklasWan/bb.org-overlays.git && cd bb.org-overlays
11.) Install dtc + dependencies:
    
    sudo ./dtc-overlays.sh
12.) Install overlays:
    
    git checkout dev_gsoc_face && sudo ./install.sh
13.) Edit Uboot uEnv.txt:

    For BBAI:
      add the following lines to /boot/uEnv.txt:
        enable-uboot-overlays=1
        uboot_overlay_addr0=/lib/firmware/BBAI_BB-BONE-FACE-8CH-00A0.dtbo
    For BBB:
      comment out the following line:
        
        #enable_uboot_cape_universal=1
      uncomment:
        
        disable_uboot_overlay_video=1
        disable_uboot_overlay_audio=1
      add: 
        
        uboot_overlay_addr0=/lib/firmware/BB-CTAG-SW-8CH-00A0.dtbo

14.) Clone https://github.com/ctag-fh-kiel/ctag-face-2-4.git:
  
    git clone https://github.com/ctag-fh-kiel/ctag-face-2-4.git & cd ctag-face-2-4
15.) Install alsa config:
  
    cd alsa-configs & sudo cp asound.conf.8ch ~/.asoundrc
16.) Reboot your board and use the audio card :-)