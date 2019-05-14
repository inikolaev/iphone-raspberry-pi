Scripts and instructions on how to setup iPhone tethering on Raspberry Pi over USB.

I have the following network setup at my home:

<img src="assets/Home%20Network%20Diagram.png">

## Raspberry Pi

https://www.diyhobi.com/share-iphones-internet-home-network-lan-using-raspberry-pi/
https://web.archive.org/web/20171129092824/http://www.daveconroy.com:80/how-to-tether-your-raspberry-pi-with-your-iphone-5/

```
sudo apt-get install usbmuxd
sudo apt-get install gvfs ipheth-utils
sudo apt-get install libimobiledevice-utils gvfs-backends gvfs-bin gvfs-fuse 
sudo apt-get install openssh-server dnsmasq
```

/etc/network/interfaces

```
auto eth1
allow-hotplug eth1
iface eth1 inet dhcp
```

```
sudo systemctl enable ssh
```

```
# Reset IP tables
sudo iptables -F FORWARD
sudo iptables -F POSTROUTING -t nat

# Setup IP tables
sudo iptables -t nat -A POSTROUTING -o eth1 -j MASQUERADE
sudo iptables -A FORWARD -i eth1 -o eth0 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i eth0 -o eth1 -j ACCEPT
sudo iptables -L -n -v
sudo sh -c "iptables-save > /etc/iptables.ipv4.nat"
```
 
### Links
https://ubuntuforums.org/showthread.php?t=2228772
https://www.raspberrypi.org/documentation/configuration/wireless/wireless-cli.md
https://www.raspberrypi.org/forums/viewtopic.php?t=18356
https://blog.mattbierner.com/tenome/
https://web.archive.org/web/20171129092824/http://www.daveconroy.com:80/how-to-tether-your-raspberry-pi-with-your-iphone-5/
https://www.howtogeek.com/68999/how-to-tether-your-iphone-to-your-linux-pc/


## OpenVPN

https://www.instructables.com/id/Raspberry-Pi-VPN-Gateway/
https://www.booleanworld.com/depth-guide-iptables-linux-firewall/
https://danielmiessler.com/study/iptables/
https://www.pinoylinux.org/tutorial/the-beginners-guide-to-iptables-the-linux-firewall/
https://stuffphilwrites.com/2014/09/iptables-processing-flowchart/
https://www.digitalocean.com/community/tutorials/a-deep-dive-into-iptables-and-netfilter-architecture

Install OpenVPN

```
sudo apt-get update
sudo apt-get install openvpn
```

Enable OpenVPN

```
sudo systemctl enable openvpn
```

Move *.ovpn files into /etc/openvpn

Backup IP tables

```
sudo iptables-save > iptables-before-vpn
```

Setup IP tables

```
# Allow traffic initiated from internal network
sudo iptables -A FORWARD -i eth0 -o tap0 -j ACCEPT

# Allow traffic from external network for connections initiated from internal network
sudo iptables -A FORWARD -i tap0 -o eth0 -m state --state RELATED,ESTABLISHED -j ACCEPT

# Drop existing connections

sudo iptables -A FORWARD -i eth0 -o eth1 -j DROP
sudo iptables -A FORWARD -i eth1 -o eth0 -j DROP

sudo iptables -A FORWARD -i eth0 -o tap0 -j DROP
sudo iptables -A FORWARD -i tap0 -o eth0 -j DROP

# This basically routes traffic via VPN
sudo iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -o tap0 -j MASQUERADE

sudo iptables -t nat -F POSTROUTING
sudo iptables -t nat -A POSTROUTING -o tap0 -j MASQUERADE

sudo iptables -t nat -R POSTROUTING 1 -o eth1 -j MASQUERADE
```

Run OpenVPN as daemon

```
sudo openvpn --config "openvpn-config.ovpn" --daemon
```
