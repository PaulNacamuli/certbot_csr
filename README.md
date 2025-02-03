# Use Certbot to create a Cert from a CSR

If you have received a CSR from a vendor and been directed to use a public CA, you can use Certbot to complete the request.  This example is using Route53 DNS verification instead of a website.  You could use other DNS verification if you follow the Certbot documentation.   The trick here is that you provide the csr file, and you get back .pem that the vendor imports into the system that contains the private key.  

This repo includes a vagrant file that works with Hyper-V to add a ubunto box to add certbot to.  You can use any working certbot install.

Read [dhcp](dhcp.md) if you need to configure Hyper-V network
Go to certbot if you need a machine for Certbot

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

## Config for vagrant certbot server {#certbot}

When the server comes up, need to install certbot.

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

I used this to force it to be up to date to keep going.

```bash
sudo apt update
sudo apt install ntpdate
sudo ntpdate pool.ntp.org
```
