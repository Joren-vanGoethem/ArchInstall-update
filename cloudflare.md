Requirements:
`bind`

```c
$> paru bind
extra/bind 9.18.21-1 [1.96 MiB 6.68 MiB]
    A complete, highly portable implementation of the DNS protocol
```

1. Download the Script 

Download the update-cloudflare-dns.sh script from the GitHub repository [DDNS-Cloudflare-Bash](https://github.com/fire1ce/DDNS-Cloudflare-Bash) and give it execute permissions:
```sh
  wget https://raw.githubusercontent.com/fire1ce/DDNS-Cloudflare-Bash/main/update-cloudflare-dns.sh
  sudo chmod +x update-cloudflare-dns.sh
  sudo mv update-cloudflare-dns.sh /usr/local/bin/update-cloudflare-dns
```

2. Download the Configuration File 
Download the update-cloudflare-dns.conf configuration file and move it to the /usr/local/bin/ directory:
```sh
  wget https://raw.githubusercontent.com/fire1ce/DDNS-Cloudflare-Bash/main/update-cloudflare-dns.conf
  sudo mv update-cloudflare-dns.conf /usr/local/bin/update-cloudflare-dns.conf
```

3. Modify the Configuration File Open the configuration file with a text editor and fill in your details:
```sh
  sudo nano /usr/local/bin/update-cloudflare-dns.conf
```
Replace the placeholders with your actual values

Run the Script You can run the script directly:
```sh
  update-cloudflare-dns
```

Or you can specify your custom configuration file:
```sh
  update-cloudflare-dns your_custom_config.conf
```

4. Schedule the Script To keep your DNS record updated
create a systemd service that will run your script. Create a new file at /etc/systemd/system/cloudflare-ddns.service and add the following content:
```sh
[Unit]
Description=Cloudflare DDNS service

[Service]
ExecStart=update-cloudflare-dns
```

create a systemd timer that will trigger your service at regular intervals. Create a new file at /etc/systemd/system/cloudflare-ddns.timer and add the following content:
```sh
[Unit]
Description=Run cloudflare-ddns.service every 15 minutes

[Timer]
OnBootSec=5min
OnUnitActiveSec=15min

[Install]
WantedBy=timers.target
```

Enable the timer so that it starts at boot, and start the timer immediately with the following commands
```sh
sudo systemctl enable cloudflare-ddns.timer
sudo systemctl start cloudflare-ddns.timer
```