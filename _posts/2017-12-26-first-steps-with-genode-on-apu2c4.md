---
title: First steps with Genode on apu2c4
date: 2017-12-26 23:20
tags: cobalt genode
---

Back in November I ordered an [apu2c4 board](https://www.pcengines.ch/apu2c4.htm)
plus a Compex WLE600VX WiFi module, antennas, power supply, and a nice blue
enclosure from <https://www.apu-board.de/>. After a few days the package
arrived and I was eager to see how Genode may run on the device I dubbed
*Cobalt* for its color.

![photo cobalt device](/assets/img/2017-12-26-cobalt.png)

I settled on Genode's PCI bus discovery scenario for the first boot, so I
prepared a bootable ISO for my USB thumb drive like follows in a clone of
[Genode](https://github.com/genodelabs/genode).

```
make -C build/x86_64 run/pci KERNEL=nova
<CTRL-C>
sudo dd if=build/x86_64/var/run/pci.iso of=/dev/sdb bs=1M conv=fsync
picocom -rlb 115200 /dev/ttyUSB0
```

After powering up Cobalt I was presented with a detailed log in my terminal
window uncovering the guts of the device. So let's do a quick walk through.

```
Terminal ready
PCEngines apu2
coreboot build 20170228
4080 MB ECC DRAM

SeaBIOS (version rel-1.10.0.1)

Press F10 key now for boot menu

Booting from Hard Disk...
GRUB loading.
Welcome to GRUB!

Bender: Hello World.
NOVA Microhypervisor v7-4911bb0 (x86_64): Dec 27 2017 20:26:00 [gcc 6.3.0] [MBI2]

[ 0] TSC:998133 kHz BUS:99813 kHz (measured)
[ 0] CORE:0:0:0 16:30:1:0 [0] AMD GX-412TC SOC
[ 3] CORE:0:3:0 16:30:1:0 [0] AMD GX-412TC SOC
[ 2] CORE:0:2:0 16:30:1:0 [0] AMD GX-412TC SOC
[ 1] CORE:0:1:0 16:30:1:0 [0] AMD GX-412TC SOC
Hypervisor features SVM
Hypervisor reports 4x1 CPUs
CPU ID (genode->kernel:package:core:thread) remapping
 remap (0->0:0:0:0) boot cpu
 remap (1->1:0:1:0) 
 remap (2->2:0:2:0) 
 remap (3->3:0:3:0) 
Hypervisor info page contains 11 memory descriptors:
core     image  [0000000000100000,000000000044b000)
binaries region [0000000000278000,000000000044b000) free for reuse
detected physical memory: 0x0000000000000000 - size: 0x000000000009f800
use      physical memory: 0x0000000000000000 - size: 0x000000000009f000
detected physical memory: 0x0000000000100000 - size: 0x00000000dfead000
use      physical memory: 0x0000000000100000 - size: 0x00000000dfead000
detected physical memory: 0x0000000100000000 - size: 0x000000001f000000
use      physical memory: 0x0000000100000000 - size: 0x000000001f000000
```

Cobalt powers up, detects the USB thumb drive, and executes the installed GRUB
bootloader. The [NOVA microhypervisor](https://github.com/alex-ab/NOVA)
initializes the quadcore AMD GX-412TC (including SVM virtualization support)
and detects 4 GiB RAM with about 500M located above 4 GiB physically. The
following output comes from Genode base components and the `test-pci`
component.

```
ROM modules:
 ROM: [000000007ffd1000,000000007ffee4a8) acpi_drv
 ROM: [000000007fffe000,000000007fffe7b5) config
 ROM: [0000000000009000,000000000000a000) core_log
 ROM: [0000000000007000,0000000000008000) hypervisor_info_page
 ROM: [000000007fe4b000,000000007fe9b780) init
 ROM: [000000007fef4000,000000007ffd05e8) ld.lib.so
 ROM: [000000007fe9c000,000000007fef32d8) platform_drv
 ROM: [0000000000005000,0000000000006000) platform_info
 ROM: [000000007fe2c000,000000007fe4a660) report_rom
 ROM: [000000007ffef000,000000007fffded8) test-pci

Genode 17.11-95-gf44edd4
4039 MiB RAM and 63253 caps assigned to init
[init -> test-pci] --- Platform test started ---
[init -> acpi_drv] Found MADT
[init -> acpi_drv] MADT IRQ 0 -> GSI 2 flags: 0
[init -> acpi_drv] MADT IRQ 9 -> GSI 9 flags: 15
[init -> test-pci] 0:0.0 class=0x600 vendor=0x1022 (unknown) device=0x1566
[init -> test-pci] 0:2.0 class=0x600 vendor=0x1022 (unknown) device=0x156b
[init -> test-pci] 0:2.2 class=0x604 vendor=0x1022 (unknown) device=0x1439
[init -> test-pci] 0:2.3 class=0x604 vendor=0x1022 (unknown) device=0x1439
[init -> test-pci] 0:2.4 class=0x604 vendor=0x1022 (unknown) device=0x1439
[init -> test-pci] 0:8.0 class=0x1080 vendor=0x1022 (unknown) device=0x1537
[init -> test-pci]   Resource 0 (MEM): base=0xfeb00000 size=0x20000 prefetchable
[init -> test-pci]   Resource 2 (MEM): base=0xfe900000 size=0x100000 
[init -> test-pci]   Resource 3 (MEM): base=0xfeb24000 size=0x1000 
[init -> test-pci]   Resource 4 (MEM): base=0xfea00000 size=0x100000 
[init -> test-pci]   Resource 5 (MEM): base=0xfeb20000 size=0x2000 
[init -> test-pci] 0:10.0 class=0xc03 vendor=0x1022 (unknown) device=0x7814
[init -> test-pci]   Resource 0 (MEM): base=0xfeb22000 size=0x2000 
[init -> test-pci] 0:11.0 class=0x101 vendor=0x1022 (unknown) device=0x7800
[init -> test-pci]   Resource 0 (I/O): base=0x4010 size=0x8 
[init -> test-pci]   Resource 1 (I/O): base=0x4020 size=0x8 
[init -> test-pci]   Resource 2 (I/O): base=0x4018 size=0x8 
[init -> test-pci]   Resource 3 (I/O): base=0x4020 size=0x8 
[init -> test-pci]   Resource 4 (I/O): base=0x4000 size=0x10 
[init -> test-pci]   Resource 5 (MEM): base=0xfeb25000 size=0x400 
[init -> test-pci] 0:13.0 class=0xc03 vendor=0x1022 (unknown) device=0x7808
[init -> test-pci]   Resource 0 (MEM): base=0xfeb25400 size=0x100 
[init -> test-pci] 0:14.0 class=0xc05 vendor=0x1022 (unknown) device=0x780b
[init -> test-pci] 0:14.3 class=0x601 vendor=0x1022 (unknown) device=0x780e
[init -> test-pci] 0:14.7 class=0x805 vendor=0x1022 (unknown) device=0x7813
[init -> test-pci]   Resource 0 (MEM): base=0xfeb25500 size=0x100 
[init -> test-pci] 0:18.0 class=0x600 vendor=0x1022 (unknown) device=0x1580
[init -> test-pci] 0:18.1 class=0x600 vendor=0x1022 (unknown) device=0x1581
[init -> test-pci] 0:18.2 class=0x600 vendor=0x1022 (unknown) device=0x1582
[init -> test-pci] 0:18.3 class=0x600 vendor=0x1022 (unknown) device=0x1583
[init -> test-pci] 0:18.4 class=0x600 vendor=0x1022 (unknown) device=0x1584
[init -> test-pci] 0:18.5 class=0x600 vendor=0x1022 (unknown) device=0x1585
[init -> test-pci] 1:0.0 class=0x200 vendor=0x8086 (Intel) device=0x157b
[init -> test-pci]   Resource 0 (MEM): base=0xfe600000 size=0x20000 
[init -> test-pci]   Resource 2 (I/O): base=0x1000 size=0x20 
[init -> test-pci]   Resource 3 (MEM): base=0xfe620000 size=0x4000 
[init -> test-pci] 2:0.0 class=0x200 vendor=0x8086 (Intel) device=0x157b
[init -> test-pci]   Resource 0 (MEM): base=0xfe700000 size=0x20000 
[init -> test-pci]   Resource 2 (I/O): base=0x2000 size=0x20 
[init -> test-pci]   Resource 3 (MEM): base=0xfe720000 size=0x4000 
[init -> test-pci] 3:0.0 class=0x200 vendor=0x8086 (Intel) device=0x157b
[init -> test-pci]   Resource 0 (MEM): base=0xfe800000 size=0x20000 
[init -> test-pci]   Resource 2 (I/O): base=0x3000 size=0x20 
[init -> test-pci]   Resource 3 (MEM): base=0xfe820000 size=0x4000 
[init -> test-pci] --- Platform test finished ---
```

There are the typical PCI system components including various kinds of bridges,
SATA controller, USB XHCI and EHCI controllers, but also an SD flash controller
and three Intel NICs. What's missing is the PCI class 0300 device for a
graphics controller as this SoC is targeting head-less applications. As I did
not mount the WLE600VX yet it's also missing from the list of PCI devices.

Then, I tested a slightly adjusted ACPI/CA scenario, which impressively
executed the ACPI reset function. This enables the option to control the device
reboot by software in future scenarios and, in the best case, may hint less
obstacles with more advanced ACPI functions.

I created a [topic branch](https://github.com/chelmuth/genode/commits/cobalt)
for Cobalt-specific Genode adaptions, where I'm going to put the Intel NICs in
operation first with my next post.
