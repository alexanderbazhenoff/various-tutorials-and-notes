# iptables persistent

tags: [[linux]], [[debian]], [[ubuntu]], [[iptables]], [[iptables-legacy]], [[iptables-persistent]], [[iptables-legacy]]

```bash
sudo apt install -y iptables-save
sudo apt install -y iptables-persistent
```

Save rules:
```bash
iptables-legacy-save > /etc/iptables/rules.v4
# or
iptables-save > /etc/iptables/rules.v4
``` 
Make sure services are enabled:
```bash
sudo systemctl status netfilter-persistent.service
```

Test it:
```bash
# Test it
sudo reboot
# when the system up check rules, e.g. NAT rules:
sudo iptables -t nat -L
```

Restore:
```bash
iptables-legacy-restore < /etc/iptables/rules.v4
```