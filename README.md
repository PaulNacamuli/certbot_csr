# Use Certbot to create a Cert from a CSR

If you have received a CSR from a vendor and been directed to use a public CA, you can use Certbot to complete the request against Let's Encrypt.  This example is using Route53 DNS verification instead of a website.  You could use other DNS verification if you follow the Certbot documentation.   The trick here is that you provide the CSR file, and you get back .pem that the vendor imports into the system that contains the private key.  

This repo includes a vagrant file that works with Hyper-V to add a ubuntu box to add certbot to.  You can use any working certbot install.

* Read [dhcp](#config-for-vagrant-dchp-server) if you need to configure Hyper-V network
* Read [certbot](#config-for-vagrant-certbot-server) if you need a machine for Certbot

## Running Certbot

This is the actual command set to use.

The --dns-route53 switch requires an AWS ID and key set with adequate permissions.  See the certbot docs for details.

This sets the access ID and key

``` bash
export AWS_ACCESS_KEY_ID="your_access_key_id"
export AWS_SECRET_ACCESS_KEY="your_secret_access_key"
```

This code relies on the mapped /pc folder as defined in the vagrant file for **certbot**.  You place the .csr file from the vendor there, and run this code.

``` bash
sudo -E certbot certonly \
  --dns-route53 \
  --csr /pc/vendor.csr \
  --cert-path /pc/vendorcert.pem \
  --chain-path /pc/vendorchain.pem \
  --fullchain-path /pc/vendorfullchain.pem
```

The required DNS is created, the CSR is read and the cert is issued.

&nbsp;
&nbsp;
&nbsp;
&nbsp;
&nbsp;
&nbsp;

## Config for vagrant dchp server

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

## Config for vagrant certbot server

To configure the certbot server using the vagrant file, you'll need a root synced folder for the mounted 'pc' folder.  In the file I used serviceuser and servicepw as the username and password.  You'll want to specify these.  When the server comes up, need to install certbot.

```bash
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo snap install certbot-dns-route53    
  ** This had an error and I ran sudo snap set certbot trust-plugin-with-root=ok
```

I repeatedly had issues with time service being out of sync so this wouldn't work with route-53.
The following command showed that the time was out of sync

```bash
sudo systemctl status hv-kvp-daemon.service 
```

I used ntpdate to force the time to sync so I could issue my cert.  I'm sure a better solution exists but I just needed to convert this CSR quickly.

```bash
sudo apt update
sudo apt install ntpdate
sudo ntpdate pool.ntp.org
```
