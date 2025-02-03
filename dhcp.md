# Config for vagrant dchp server

If you already have a working dhcp server or want to set the certbox box with a fixed IP, you can ignore this.

In my lab I have a Hyper-V switch named NATSwitch which is Internal only because Hyper-V unreliably delivers DHCP to guests.

To configure the network on a switch, you need to use Powershell.  Creating in the GUI will give you a random dynamic range to start with.

``` Powershell
    New-VMSwitch -Name "NATSwitch" -switchtype Internal
    Get-HNSNetwork | ? Name -Like "NATSwitch"  # Shows current networks that were autocreated
    New-NetIPAddress -InterfaceAlias 'vEthernet (NATSwitch)' -IPaddress 172.25.32.1 -prefixlength 24   #Sets DGW using .1 for the subnet found above
    New-NetNat -Name NATNetwork -InternalIPInterfaceAddressPrefix 172.25.32.0/24
```

On the box "dhcp" Must update IP config

```bash
sudo nano /etc/netplan/01-netcfg.yaml
  
  version: 2
  ethernets:
    eth0:
      dhcp4: no
      addresses:
        - 172.25.32.11/24
      routes:
        - to: 0.0.0.0/0
          via: 172.25.32.1
          on-link: true
      nameservers:
        addresses:
          - 192.168.100.1
          - 192.168.100.2```

Then install DHCP:

sudo apt update
sudo apt install isc-dhcp-server
sudo nano /etc/dhcp/dhcpd.conf

subnet 172.25.32.0 netmask 255.255.255.0 {
  range 172.25.32.50 172.25.32.254;
  option routers 172.25.32.1;
  option domain-name-servers 192.168.100.1, 192.168.100.2
  option domain-name "dom.loc"
}

sudo nano /etc/default/isc-dhcp-server
INTERFACESv4="eth0"

sudo systemctl restart isc-dhcp-server
sudo systemctl enable isc-dhcp-server
sudo systemctl status isc-dhcp-server


sudo dhcpd -t -cf /etc/dhcp/dhcpd.conf  to check the file for syntax errors
```
