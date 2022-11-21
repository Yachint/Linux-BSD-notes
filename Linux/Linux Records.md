# Linux Doc

This is a general purpose helper document which aims to cover the streamlined distros which are either based on Debian or Ubuntu like Linux Mint, Fedora, Elementary OS, MX Linux etc.
Therefore most of the documentation here will be aimed at using Systemd as init system. For guides on other init systems checkout the Arch Records.

## <ins>Changing DNS</ins>

---
<aside> 💡 These steps may depend on the fact that the system is using Systemd-resolved.service. It can be that the steps mentioned after this are still valid but if not working, try again with installing the service

</aside>
---

The main file which provides the nameservers for the System's DNS name resolution is ***resolv.conf*** located at:

```
/etc/resolv.conf
```

Now, if we try to modify the nameserver's directly by changing in this file, then those changes will be overwritten by NetworkManager. 

*This happens because this file is sym-linked to another file located at "/run/systemd/resolve/stub-resolv.conf" which is then modified by Network manager on boot or while system is running due to which the contents of this file keeps getting overwritten*

---
💡 Another option is to use a command like ***chattr*** to set attribute of the file as immutable so that the Network manager cannot change the contents after we change the DNS.

```bash
rm -r /etc/resolv.conf
nano /etc/resolv.conf
chattr +i /etc/resolv.conf
```

But this is not advisable as it might cause issues in the future with Network Manager.

---

### Soultion?

The best solution is to change the symlink of the resolv.conf file from ==**"/run/systemd/resolve/stub-resolv.conf"**== to -> 
==**"/run/systemd/resolve/resolv.conf"**== after we change the contents of the file at "/run/systemd/resolve/resolv.conf" to our linking.

```bash
sudo vi /run/systemd/resolve/resolv.conf
sudo rm -f /etc/resolv.conf
sudo ln -s /run/systemd/resolve/resolv.conf /etc/resolv.conf
```

Example Config:
```bash
# This is /run/systemd/resolve/resolv.conf managed by man:systemd-resolved(8).
# Do not edit.
#
# This file might be symlinked as /etc/resolv.conf. If you're looking at
# /etc/resolv.conf and seeing this text, you have followed the symlink.
#
# This is a dynamic resolv.conf file for connecting local clients directly to
# all known uplink DNS servers. This file lists all configured search domains.
#
# Third party programs should typically not access this file directly, but only
# through the symlink at /etc/resolv.conf. To manage man:resolv.conf(5) in a
# different way, replace this symlink by a static file or a different symlink.
#
# See man:systemd-resolved.service(8) for details about the supported modes of
# operation for /etc/resolv.conf.

nameserver 1.0.0.1
nameserver 1.1.1.1
nameserver fe80::8226:89ff:fe1c:15bc%3
# Too many DNS servers configured, the following entries may be ignored.
nameserver fd80::8226:89ff:fe1c:15bc
search .
```

### For Systemd-resolved

![](attachments/Pasted%20image%2020221121125347.png)