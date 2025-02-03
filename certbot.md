# Config for vagrant certbot server

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
