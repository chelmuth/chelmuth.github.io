---
title: Getting Cobalt NICs to work
date: 2017-12-27 22:30
tags: cobalt genode
---

After the first peeks into the Cobalt router hardware, I was curious if Genode
is fit for operating the NICs with the drivers ported from
[iPXE](https://ipxe.org/). I used the `lwip` run scenario for this purpose like
follows.

```
make -C build/x86_64 run/lwip KERNEL=nova
<CTRL-C>
sudo dd if=build/x86_64/var/run/pci.iso of=/dev/sdb bs=1M conv=fsync
picocom -rlb 115200 /dev/ttyUSB0
```

But as I expected, the NIC driver does not know anything about the devices.

```
[init -> nic_drv] Found: 01:00.0 8086:157b (rev 03) IRQ 00
[init -> nic_drv] no driver found
[init -> nic_drv] Found: 02:00.0 8086:157b (rev 03) IRQ 00
[init -> nic_drv] no driver found
[init -> nic_drv] Found: 03:00.0 8086:157b (rev 03) IRQ 00
[init -> nic_drv] no driver found
[init -> nic_drv] Error: could not find usable NIC device
```

We do not update our ported iPXE drivers regularly so I had a look into the
upstream sources and found out that the NIC is indeed supported but the PCI
device ID is not listed. After I added it to our
[patch](https://github.com/chelmuth/genode/blob/cobalt/repos/dde_ipxe/patches/intel.patch)
of the contrib sources the scenario started successfully.

```
[init -> nic_drv] Found: 01:00.0 8086:157b (rev 03) IRQ 00
[init -> nic_drv] using driver i210-2
[init -> nic_drv] PCI BIOS has not enabled device 01:00.0! Updating PCI command 0003->0007
[init -> nic_drv] 
[init -> nic_drv] PCI device 01:00.0 latency timer is unreasonably low at 0. Setting to 32.
[init -> nic_drv] 
[init -> nic_drv] bus_addr = fe600000 len = 20000
[init -> nic_drv] snprintf not implemented
[init -> platform_drv] 1:0.0 adjust IRQ as reported by ACPI: 0 -> 28
[init -> platform_drv] 1:0.0 uses IRQ, vector 0x1c, MSI 64bit capable, maskable
[init] child "nic_drv" announces service "Nic"
[init -> nic_drv] MAC address 19:00:00:0d:19:02
[init -> test-lwip_httpsrv] Create new socket ...
[init -> test-lwip_httpsrv] Now, I will bind ...
[init -> test-lwip_httpsrv] Now, I will listen ...
[init -> test-lwip_httpsrv] Start the server loop ...
[init -> test-lwip_httpsrv] got IP address 10.0.0.11
```

Then, I could successfully connect to the mini HTTP server and got the expected
response.

```
w3m -dump http://10.0.0.11/

Welcome to our lwIP HTTP server!

This is a small test page.
```

You may find the described adaption of our iPXE port on my
[Cobalt topic branch](https://github.com/chelmuth/genode/commits/cobalt), from
where I'll merge them into the
[Genode staging branch](https://github.com/genodelabs/genode/commits/staging)
later. Also note that I added a simple test network DHCP script
[[1]](https://github.com/chelmuth/genode/blob/cobalt/tool/netboot-dhcpd.sh)
[[2]](https://github.com/chelmuth/genode/blob/cobalt/tool/netboot-dhcpd.conf)
to the branch to separate the tests my general network infrastructure.
