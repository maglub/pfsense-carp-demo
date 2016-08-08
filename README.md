
# Introduction

* em0 -> WAN -> (fw01: 192.168.149.2, fw02: 192.168.149.3) -> vboxnet10
* em1 -> LAN (fw01: 192.168.150.2, fw02: 192.168.150.3) -> vboxnet11
* em2 -> SYNC -> vboxnet12

Note, networks for em0 and em1 are configured to allow promiscous mode.

* Installation of 2 pfSense instances => 40 sec from start to installed/reboot
* Installation of 2 pfSense instances => 1m minutes from start to booted firewall
* Configuration of base setup (NICs and IP, ssh) => 3min06sek from start
* Config of HA config => 5min05sek
* CARP IP address configured 8min20sek

# Setup

* Download the ISO from pfsense.org

```
wget https://frafiles.pfsense.org/mirror/downloads/pfSense-CE-2.3.2-RELEASE-amd64.iso.gz
gunzip pfSense-CE-2.3.2-RELEASE-amd64.iso.gz
```

* Set up the two firewall VM's.

```
bin/createVM
```

* Install pfSense on both VMs, defaults are ok

* Configure interfaces (menu item 1) (type 1 n em0<enter> em1<enter> em2<enter> <enter> y)
  * No VLAN
  * em0
  * em1
  * em2

* Configure IP address on LAN interface on both VMs (menu item 2, interface 2)
  * fw01 => 192.168.150.2 (24 bit netmask)
  * fw02 => 192.168.150.3 (24 bit netmask)
  * No DHCP server
  * Do not revert to http as the web configurator protocol

* Enable ssh (menu item 14)

* Use your web-browser to log in (admin/pfsense) to the web gui
  * https://192.168.150.2
  * https://192.168.150.3
  * Click the logo to avoid the configuration

* Rename Interface OPT1 to SYNC (Interfaces->OPT1) and set IP on both nodes
  * enable the interfaces
  * Set static IP address on the SYNC interface on both nodes (IPv4 configuration type: Static IPv4) NOTE: netmask 24 bits
    * fw01 => 192.168.151.2/24
    * fw02 => 192.168.151.3/24

* Add rule to allow TCP traffic from the sync network to "any" (Firewall->Rules->SYNC, Add above)
  * Source: SYNC net
  * Save
  * Apply

* Configure State Synchronization Settings (pfsync) (System->High Avail Sync) on BOTH nodes
  * Interface: SYNC
  * IP address of the other node
  * Save

* Configure Configuration Synchronization Settings (XMLRPC Sync) (only on master node)
  * IP Address: 192.168.151.3
  * Toggle all
  * Save

* Add virtual IP on the LAN interface (192.168.150.4) on the master node (Firewall->Virtual IPs)
  * Add
  * Click CARP radio button
  * Interface LAN
  * Address: 192.168.150.4 Netmask 24 bit
  * Virtual IP Password: anything you like
  
* Check that the virtual IP also exist on the slave node in the web GUI (Firewall->Virtual IPs)
* Check that the status is correct on both nodes (Status->CARP (failover))

Now you can ping 192.168.150.4 from your laptop. Do this, and shut down fw01, and see that the ping continues to work after a couple of missed packages.


# Goodies

* Delete the firewall VMs

```
for vmName in fw01 fw02
do
VBoxManage controlvm "$vmName"  poweroff 
VBoxManage unregistervm "$vmName" --delete
done
```

# References

* https://doc.pfsense.org/index.php/Configuring_pfSense_Hardware_Redundancy_(CARP)
