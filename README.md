# kindlespoof

1. create wifi network

```
nmcli con modify ForKindle 802-11-wireless.mode ap 802-11-wireless.band bg ipv4.method shared
nmcli con modify ForKindle wifi-sec.key-mgmt wpa-psk
nmcli con modify ForKindle wifi-sec.psk tester123
nmcli connection modify ForKindle 802-11-wireless.channel 6
nmcli con up ForKindle
nmcli con show --active
journalctl -f -u NetworkManager
```

2. monitor what the kindle is connecting to

`sudo tcpdump -i wlp4s0 -n udp port 53`

3. spoof amazon connections with `/etc/hosts` and an http server

```
sudoedit /etc/hosts

cat /etc/hosts
...
10.42.0.1 dogvgb9ujhybx.cloudfront.net
10.42.0.1 dns.kindle.com
10.42.0.1 spectrum.s3.amazonaws.com

sudo systemctl restart NetworkManager
```

```
❯ tree
.
├── gen_204
├── index.html
└── kindle-wifi
    ├── wifiredirect.html
    ├── wifistub-eink.html
    └── wifistub.html

2 directories, 5 files
```

```
❯ cat kindle-wifi/wifistub.html
81ce4465-7167-4dcb-835b-dcc9e44c112a
```

```
sudo python3 -m http.server 80
```

Apparently the file devices look for changes; mine definitely looked for wifistub-eink.html and wifiredirect.html.

4. unblock UFW for local hotspot traffic with `sudo ufw allow in on wlp4s0`
5. access calibre content server on `10.42.0.1:8090/mobile` after changing the preferences from 8080.
