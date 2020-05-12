# Community Bonding Period: May, 4 - June, 1
## Week 1: May, 6 - May, 13
- Research regarding Linux Kernel Build System
- Research regarding Linux Kernel ALSA SoC Developmenz
- Started to port CTAG Face driver from [Henrix Beagleboard.org Fork](https://github.com/henrix/beagle-linux/tree/4.4-ctag-dev) to [my Fork from Beagleboard.org 4.19-RT](https://github.com/NiklasWan/linux/tree/dev_ctag_4.19-rt) for BBB
- Found out that Davinci SoC Platform Drivers are no longer in Mainline Kernel since version 4.14 (TODO: Ask Jason on wensday)
- However Davinci SoC Drivers for newer kernel versions (>4.14) can be found at [TI git](https://git.kernel.org/pub/scm/linux/kernel/git/nsekhar/linux-davinci.git/refs/tags)
- Created Dev Branch for proting CTAG Face to [kernel 5.4-rt for Beaglebone AI](https://github.com/NiklasWan/linux/tree/dev_ctag_5.4-rt)
- Problem?: TI version of Davinci drivers still in debug for kernel 5.4
### Helpfull links from Jason regarding kernel 5.4-rt:
- http://repos.rcn-ee.com/latest/buster-armhf/LATEST-ti-rt
- Headers: repos.rcn-ee.net/debian/pool/main/l/linux-upstream/linux-headers-5.4.28-ti-rt-r7_1buster_armhf.deb
Kernel Image: repos.rcn-ee.net/debian/pool/main/l/linux-upstream/linux-image-5.4.28-ti-rt-r7_1buster_armhf.deb
- https://github.com/beagleboard/linux/tree/5.4-rt
- Kernel Log: https://github.com/beagleboard/linux/commit/42e00287293c09d0863688e965b20ed48a280888