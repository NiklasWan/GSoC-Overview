# Project Overview
**Student: Niklas Wantrupp**

**Mentors:**

- henrix
- rma
- Drew Fustini
- Indumathi Duraipandian

## Project Description
The BeagleBone AI is equipped with a high amount of processing power due to the Dual Core ARM Cortex-A15 chip as a main computing unit and its accompanying co-processors. This makes the AI a perfect fit for highly demanding applications regarding CPU consumption, like media applications which have extremely strong realtime constraints. Professional audio/video studios have to guarantee for small latencies when transmitting media signals between different devices. Different media channels in a transmitted stream need to be carefully synchronized to guarantee for e.g. lip synchronicity. To use the AI in such a scenario, the AVB protocol stack should be implemented in the Beagleboard Linux Kernel, which allows for synchronization of media streams within a local area network. Furthermore the CTAG Face audio drivers should be ported from BeagleBoard-X15 and HDMI audio out capabilities should be deployed, to allow the in- and output of audio data.

## Project Goals
Optimal outcome: Merge everything into BeagleBoard.org Linux kernel 

- Port CTAG Face drivers implemented by henrix for GSoC 2016 to Linux kernel 4.19-rt for BeagleBone AI \[[Merged driver](https://github.com/beagleboard/linux/pull/240), [Merged overlays](https://github.com/beagleboard/bb.org-overlays/pull/184), [Documentation](face-driver.md)\] &#9745;
- Port AVB driver stack implemented by indu for GSoC 2017 to Linux kernel 4.19-rt for BeagleBone AI \[[Open](https://github.com/beagleboard/linux/pull/246), [Documentation](avb-alsa.md)\] &#9745;
- Refactor and modularize present AVB driver architecture &#9745;
- Implement AVDECC protocol part of AVB to allow for device enumeration and control &#9745;

## Project Outcome

During the project I successfully ported the CTAG Face drivers by [henrix](https://github.com/henrix) from kernel version 4.4 to 4.19-rt and started porting to 5.4-rt for both BBB and BBAI. The port to 5.4-rt however still needs some adjustments, look [here](face-driver.md#Drivers%20for%205.4-rt) to see a description.

The second part of the project was to port the virtual ALSA driver of [indu](https://github.com/induarun9086) from kernel 4.4 to 4.19-rt. During the course of the project I however realized, that 4.19-rt is running quite unstable on BBAI. Because of that I changed to kernel 5.4-rt for the implementation. The driver code was modularized and refactored and the AVDECC protocol part which indu allready implemented was also ported. However the implementation is still not working correctly with third party devices, like the proprietary Apple Mac implementation. All further limitations of the AVB implementation can be found [here](avb-alsa.md#Limitations).

## List of Pull Requests

| Pull Request | Status |
| --------- | ------- |
| [CTAG Face driver](https://github.com/beagleboard/linux/pull/240)   | merged |
| [CTAG Face overlays](https://github.com/beagleboard/bb.org-overlays/pull/184)   | merged |
| [CTAG Face overlays compatibility fix](https://github.com/lorforlinux/BeagleBoard-DeviceTrees/pull/1)   | merged |
| [Virtual ALSA driver](https://github.com/beagleboard/linux/pull/246)   | open |

## Further Links
- [Youtube Intro Video](https://www.youtube.com/watch?v=Pvb3sk3QNuI)
- [Youtube Demonstration Video](https://youtu.be/xrtGZIX-Pow)
- [Elinux Project Page](https://elinux.org/BeagleBoard/GSoC/MediaIpStreaming)
- [Kernel Repository](https://github.com/NiklasWan/linux)
- [avbtest fork](https://github.com/NiklasWan/avbtest)
- [gptpd fork](https://github.com/NiklasWan/gPTPd)
- [BeagleBoard Overlay fork](https://github.com/NiklasWan/bb.org-overlays)
- [Compatibility Update fork](https://github.com/NiklasWan/BeagleBoard-DeviceTrees)
