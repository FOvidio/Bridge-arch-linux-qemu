# Bridge-arch-linux-qemu
Qemu KVM - Bridge arch linux


Switch to directory /etc/systemd/network.
First, create a virtual bridge interface with a netdev unit file.
We tell systemd to create a device named br0 that functions as an ethernet bridge. 
I’ll create a file called 99-br0.netdev with the following content:

```
[NetDev]
Name=br0
Kind=bridge
```

Next we the bridge needs a physical interface assigned. 
That’s enp3s0 which we figured out with ip link above. 
This file needs to be loaded before 99-br0.netdev. So the first number fo the file name needs to be lower.
I’ll call the file 98-enp3s0.network with this content:

```
[Match]
Name=enp3s0

[Network]
Bridge=br0
```

If all your interfaces should be part of the bridge you can also specify Name=en* e.g.
And finally the bridge needs a network configuration. 
So I’ll create a file called 99-br0.network with this content:

```
[Match]
Name=br0

[Network]
Address=192.168.1.115/24
Gateway=192.168.1.1
DNS=8.8.8.8
DNS=1.1.1.1
Search=google.com
```

If your host should get its IP configuration via DHCP only specify DHCP=ipv4 in the [Network] section.
In my case it’s a static IP with 192.168.1.115/24 and the default gateway 192.168.1.1. So the network my physical hosts and the VMs are located in is 192.168.1.0/24. 
DNS server is Google’s 8.8.8.8. Additionally it has Quad9 DNS server configured as fallback. You can also add Domains= here as search domains.

Next we should have a few entries in /etc/hosts e.g.:

```
127.0.0.1       localhost
127.0.1.1       localhost
::1             localhost
192.168.1.115    ovidio
```

You want to change the hostname ovidio of course to your hostname and adjust the IP address accordingly
Next we need to make sure that we have the DNS resolver running once the host is up and running. 
The same is true for networking and SSH of course:

```
systemctl enable systemd-resolved.service
systemctl enable systemd-networkd.service
systemctl enable sshd.service
```
