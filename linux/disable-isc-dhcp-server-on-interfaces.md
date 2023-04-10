# disable isc-dhcp-server on interfaces

tags: [[linux]], [[debian]], [[ubuntu]], [[isc-dhcp-server]]

```bash
cat /etc/default/isc-dhcp-server
INTERFACESv4="enp0 enp1"
# or
INTERFACES="enp0 enp1"
```