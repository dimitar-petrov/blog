+++
title = "Setting up the all servers needed for PXE"
author = ["Dimitar Petrov"]
date = 2020-07-21T19:46:00+03:00
lastmod = 2020-07-22T21:12:00+03:00
tags = ["pxe", "linux", "arch"]
draft = false
+++

We are going to use dnsmasq package to supply DHCP and TFTP functionality to your PXE server.

-   Install and configure dnsmasq

<!--listend-->

```sh
sudo pacman --quiet --noconfirm -S dnsmasq
sudo cp -rp /etc/dnsmasq.conf /etc/dnsmasq.conf_$(date "+%Y%m%d")
sudo sh -c "cat > /etc/dnsmasq.conf <<EOF
port=0
interface=eth0
bind-interfaces
dhcp-range=192.168.10.50,192.168.10.60,12h
dhcp-boot=/arch/boot/syslinux/lpxelinux.0
dhcp-option-force=209,boot/syslinux/archiso.cfg
dhcp-option-force=210,/arch/
dhcp-option-force=66,192.168.10.102
enable-tftp
tftp-root=/srv/pxe
EOF"
```

```text
resolving dependencies...
looking for conflicting packages...

Packages (3) libnetfilter_conntrack-1.0.6-1  libnfnetlink-1.0.1-2  dnsmasq-2.76-4

Total Installed Size:  0.91 MiB

:: Proceed with installation? [Y/n]
(1/3) checking keys in keyring                                  [###########-----------------------]  33%(2/3) checking keys in keyring                                  [######################------------]  66%(3/3) checking keys in keyring                                  [##################################] 100%
(1/3) checking package integrity                                [#---------------------------------]   4%(2/3) checking package integrity                                [#####-----------------------------]  16%(3/3) checking package integrity                                [##################################] 100%
(1/3) loading package files                                     [#---------------------------------]   4%(2/3) loading package files                                     [#####-----------------------------]  16%(3/3) loading package files                                     [##################################] 100%
(1/3) checking for file conflicts                               [###########-----------------------]  33%(2/3) checking for file conflicts                               [######################------------]  66%(3/3) checking for file conflicts                               [##################################] 100%
(1/3) checking available disk space                             [###########-----------------------]  33%(2/3) checking available disk space                             [######################------------]  66%(3/3) checking available disk space                             [##################################] 100%
:: Processing package changes...
(1/3) installing libnfnetlink                                   [##################################] 100%
(2/3) installing libnetfilter_conntrack                         [##################################] 100%
(3/3) installing dnsmasq                                        [##################################] 100%
:: Running post-transaction hooks...
(1/2) Updating system user accounts...
(2/2) Arming ConditionNeedsUpdate...
```

Now start and enable(if preferrable) dnsmasq service

```sh
sudo systemctl start dnsmasq
sudo systemctl enable dnsmasq
sudo systemctl status dnsmasq
```

```text

● dnsmasq.service - fA lightweight DHCP and caching DNS server
  Loaded: loaded (/usr/lib/systemd/system/dnsmasq.service; enabled; vendor preset: disabled)
  Active: active (running) since Wed 2017-04-26 21:17:20 EEST; 12s ago
    Docs: man:dnsmasq(8)
Main PID: 1253 (dnsmasq)
  CGroup: /system.slice/dnsmasq.service
          └─1253 /usr/bin/dnsmasq -k --enable-dbus --user=dnsmasq --pid-file

Apr 26 21:17:20 durden.evolve.inc systemd[1]: Starting A lightweight DHCP and caching DNS server...
Apr 26 21:17:20 durden.evolve.inc dnsmasq[1250]: dnsmasq: syntax check OK.
Apr 26 21:17:20 durden.evolve.inc systemd[1]: Started A lightweight DHCP and caching DNS server.
Apr 26 21:17:20 durden.evolve.inc dnsmasq[1253]: started, version 2.76 DNS disabled
Apr 26 21:17:20 durden.evolve.inc dnsmasq[1253]: compile time options: IPv6 GNU-getopt DBus i18n ID…otify
Apr 26 21:17:20 durden.evolve.inc dnsmasq[1253]: DBus support enabled: connected to system bus
Apr 26 21:17:20 durden.evolve.inc dnsmasq-dhcp[1253]: DHCP, IP range 192.168.10.50 -- 192.168.10.60…e 12h
Apr 26 21:17:20 durden.evolve.inc dnsmasq-dhcp[1253]: DHCP, sockets bound exclusively to interface eth0
Apr 26 21:17:20 durden.evolve.inc dnsmasq-tftp[1253]: TFTP root is /srv/pxe
Hint: Some lines were ellipsized, use -l to show in full.
```

-   Set up a simple http - darkhttpd

Install the package and start the service

```sh
sudo pacman --quiet --noconfirm -S darkhttpd
sudo nohup darkhttpd /srv/pxe &
```

```text
resolving dependencies...
looking for conflicting packages...

packages (1) darkhttpd-1.12-3

total download size:   0.02 mib
total installed size:  0.08 mib

:: proceed with installation? [y/n]
:: retrieving packages...
darkhttpd-1.12-3-i686                   0.0   b  0.00b/s 00:00 [----------------------------------]   0% darkhttpd-1.12-3-i686                   0.0   b  0.00b/s 00:00 [----------------------------------]   0% darkhttpd-1.12-3-i686                   0.0   b  0.00b/s 00:00 [----------------------------------]   0% darkhttpd-1.12-3-i686                  19.1 kib   191k/s 00:00 [##################################] 100%
(1/1) checking keys in keyring                                  [##################################] 100%
(1/1) checking package integrity                                [##################################] 100%
(1/1) loading package files                                     [##################################] 100%
(1/1) checking for file conflicts                               [##################################] 100%
(1/1) checking available disk space                             [##################################] 100%
:: processing package changes...
(1/1) installing darkhttpd                                      [##################################] 100%
:: running post-transaction hooks...
(1/1) arming conditionneedsupdate...
[1] 1333
```

check if webserver is working, it is important that server is running on port 80, so make sure you start it as root

```sh
curl -s http://localhost:80/ | egrep href
```

```text

..</a>/
EFI</a>/
arch</a>/
isolinux</a>/
loader</a>/
```
