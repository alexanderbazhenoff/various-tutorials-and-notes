# switch back to cgroupsv1 in Ubuntu 20.10

to avoid systemd and dbus troubles inside containers:

```bash
cat /etc/default/grub
GRUB_CMDLINE_LINUX="systemd.unified_cgroup_hierarchy=0"
# reboot and check:
reboot
lxc-checkconfig
```

> In Ubuntu 21.10, systemd is being switched to the 'unified' cgroup hierarchy (cgroup v2) by default instead of 
> (cgroup v1) in previous releases 
> ([source](https://askubuntu.com/questions/1369947/after-ubuntu-21-10-upgrade-cannot-attach-cgroup-program-operation-not-permitt)).