# kindlespoof

This setup can be used to connect a Kindle to a local [calibre](https://calibre-ebook.com/) content server and add books while offline. It may be especially useful if your Kindle's USB port is broken.

This repository is deliberately not minimal. A Kindle may request different connectivity-check paths depending on its model or firmware. Not all five response files are likely necessary, but I have not determined exactly which subset is required, so all five known paths are included with the same response.

> [!CAUTION]
> Use this only with devices and networks that you own or are authorized to administer. The example changes DNS resolution, firewall rules, and local network behavior. It is not affiliated with or endorsed by Amazon.

## Setup

1. Create a Wi-Fi network. `tester123` below is only an example password; choose a stronger password if other people are within range.

```
nmcli con modify ForKindle 802-11-wireless.mode ap 802-11-wireless.band bg ipv4.method shared
nmcli con modify ForKindle wifi-sec.key-mgmt wpa-psk
nmcli con modify ForKindle wifi-sec.psk tester123
nmcli connection modify ForKindle 802-11-wireless.channel 6
nmcli con up ForKindle
nmcli con show --active
journalctl -f -u NetworkManager
```

2. Monitor what the Kindle is connecting to:

`sudo tcpdump -i wlp4s0 -n udp port 53`

3. Redirect the Kindle connectivity-check hostnames to the local machine with `/etc/hosts`, then run an HTTP server:

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

Binding to port 80 generally requires elevated privileges. Running Python as root carries risk, so run this only from a directory containing files you intend to serve, stop it when finished, and do not expose it to an untrusted network.

The path requested appears to vary between devices or firmware versions. Mine definitely requested `kindle-wifi/wifistub-eink.html` and `kindle-wifi/wifiredirect.html`; the other files are retained for compatibility until their necessity is better understood.

4. Allow local hotspot traffic through UFW with `sudo ufw allow in on wlp4s0`.
5. Access the calibre content server from the Kindle at `10.42.0.1:8090/mobile` after changing its port from 8080.

## Cleanup

When finished:

1. Stop the HTTP server.
2. Remove the three entries added to `/etc/hosts`, then restart NetworkManager.
3. Remove the firewall exception with `sudo ufw delete allow in on wlp4s0`.
4. Bring down the hotspot with `nmcli con down ForKindle` (and delete the connection if you no longer need it).
