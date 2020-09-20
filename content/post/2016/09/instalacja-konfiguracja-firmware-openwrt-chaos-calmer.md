---
author: Morfik
categories:
- OpenWRT
date: "2016-09-23T18:27:02Z"
date_gmt: 2016-09-23 16:27:02 +0200
published: true
status: publish
tags:
- chaos-calmer
- router
- tp-link
title: Instalacja i konfiguracja firmware OpenWRT (Chaos Calmer)
---

Ten post ma na celu zebranie wszystkich wpisów dotyczących instalacji i konfiguracji alternatywnego
firmware OpenWRT, które znajdują się na tym blogu i umieszczenie ich w jednym wpisie. Chodzi
generalnie o to, by wszystkie te artykuły były dostępne na jednej stronie w formie spisu treści
odwołującego się do poszczególnych tekstów. [Na tplinkforum.pl znajduje się post "OpenWRT w
pigułce"][1], z tym, że tamten artykuł dotyczy wydania Barrier Breaker. Artykuły, do których linki
znajdują się poniżej, odwołują się do wydania Chaos Calmer i rozwiązania opisane w nich powinny na
tej wersji firmware działać bez problemu. Mogą natomiast pojawić się problemy w przypadku
konfiguracji starszych wersji OpenWRT na naszym routerze WiFi.

<!--more-->
## Spis artykułów dotyczących firmware OpenWRT Chaos Calmer

  - [Jak wgrać firmware OpenWRT Chaos Calmer na router TP-LINK][2]
  - Uzyskiwanie dostępu do routera WiFi:
      - [SSH, SSHFS oraz telnet][3]
      - [Likwidacja hasła dostępowego na rzecz kluczy SSH][4]
      - [Tryb ratunkowy (failsafe)][5]
      - [Reset ustawień (firstboot)][6]
      - [Konsola szeregowa][7]
      - [Tryb recovery][8]
  - [Menadżer pakietów OPKG][9]
  - [Backup ustawień i aktualizacja firmware do nowszej wersji (sysupgrade)][10]
  - [Jak powrócić do oryginalnego firmware TP-LINK'a][11]
  - [Skrypty startowe init][12]
  - Konfiguracja systemu (/etc/config/system):
      - [Nazwa hosta i domena routera][13]
      - [Strefa czasowa i czas sieciowy (NTP)][14]
      - [Logi systemowe (logread)][15]
      - [Konfiguracja diod LED][16]
  - [Przyciski][17]
  - Konfiguracja sieci (/etc/config/network):
      - [Interfejsy sieciowe (lo, br-lan, eth0/eth1) oraz IPv6][18]
      - [Klonowanie adresów MAC][19]
      - [Problemy z ISP za sprawą podwójnego adresu MAC][20]
      - [Podziała switch'a na kilka VLAN'ów (2x WAN)][21]
      - [Konfiguracja DMZ][22]
  - [DNS, DHCP, statyczne lease DHCP oraz opcje DHCP][23] (/etc/config/dhcp)
  - Konfiguracja WiFi (/etc/config/wireless)
      - [Tryb AP, szerokość kanałów, kanały 12 i 13, moc transmisyjna, ukrycie sieci, filtr adresów
        MAC][24]
      - [Konfiguracja anten][25]
      - [WPS][26]
      - [Tryb WISP (Wireless ISP)][27]
      - [Tryb WDS (most bezprzewodowy)][28]
      - [Tryb MONITOR][29]
      - [Bezprzewodowa sieć gościnna][30]
      - [Różne klasy adresowe dla LAN i WLAN][31]
  - Nośniki wymienne (/etc/config/fstab)
      - [Dyski zewnętrzne, pendrive, SWAP (fdisk, mkfs, block-mount)][32]
      - [Extroot i whole_root (fullroot)][33]
      - [Zmiana rozmiaru katalogu /tmp/][34]
  - [Filtr pakietów sieciowych][35] (/etc/config/firewall)
  - Modem 3G/4G/LTE
      - [Konfiguracja modemu LTE Huawei E3372s][36]
      - [Stałe nazwy urządzeń (udev, hotplug)][37]
      - [Monitor połączenia LTE (3ginfo)][38]
      - [SMS i kody USSD (gnokii, polecenia AT)][39]
      - [Konfiguracja Aero2][40]
      - [Automatyczna blokada internetu LTE][41]
  - Dodatkowe usługi:
      - [Szyfrowanie zapytań DNS (dnscrypt-proxy)][42]
      - [Wake on LAN (WoL) (etherwake)][43]
      - [Drukarka sieciowa (p910nd)][44]
      - [Sieciowy system plików NFS][45]
      - [FTP (vsftpd)][46]
      - [Statystyki routera (collectd)][47]
      - [Blokada Facebook, YouTube i innych portali społecznościowych (iptables, dnsmasq i
        ipset)][48]
      - [Serwer RADIUS (freeradius)][49]
      - [Kształtowanie ruchu (qos-scripts)][50]
      - [Quality of Service (QoS) (tc i iptables)][51]
      - [Failover i loadbalancking (mwan3)][52]
      - [Tunel 6in4 (IPv6)][53]
      - [Adblock i blokowanie reklam][54]
      - [Udostępnianie połączenia LTE/3G ze smartfona w sieci lokalnej][55]


[1]: https://tplinkforum.pl/t/openwrt-w-pigulce-konfiguracja-w-oparciu-o-tl-wr1043nd-oraz-archer-c7/6960/
[2]: {{< baseurl >}}/post/jak-wgrac-firmware-openwrt-na-router-tp-link/
[3]: {{< baseurl >}}/post/dostep-routera-openwrt-telnet-ssh-sshfs/
[4]: {{< baseurl >}}/post/klucze-szyfrujace-rsa-w-openwrt-ssh/
[5]: {{< baseurl >}}/post/tryb-ratunkowy-failsafe-w-openwrt/
[6]: {{< baseurl >}}/post/reset-ustawien-w-openwrt-firstboot/
[7]: {{< baseurl >}}/post/konsola-szeregowa-adapter-usb-uart-uszkodzony-router-tp-link/
[8]: {{< baseurl >}}/post/jak-przy-pomocy-trybu-recovery-odzyskac-router-tp-link/
[9]: {{< baseurl >}}/post/opkg-czyli-menadzer-pakietow-w-openwrt/
[10]: {{< baseurl >}}/post/sysupgrade-czyli-aktualizacja-firmware-openwrt/
[11]: {{< baseurl >}}/post/jak-powrocic-z-firmware-openwrt-tp-linka/
[12]: {{< baseurl >}}/post/skrypty-startowe-init-w-openwrt/
[13]: {{< baseurl >}}/post/hostname-czyli-nazwa-hosta-w-openwrt/
[14]: {{< baseurl >}}/post/strefa-czasowa-timezone-w-openwrt/
[15]: {{< baseurl >}}/post/logread-czyli-system-logowania-w-openwrt/
[16]: {{< baseurl >}}/post/konfiguracja-diod-w-routerze-pod-openwrt-led/
[17]: {{< baseurl >}}/post/konfiguracja-przyciskow-w-openwrt/
[18]: {{< baseurl >}}/post/konfiguracja-interfejsow-sieciowych-w-openwrt/
[19]: {{< baseurl >}}/post/jak-sklonowac-adres-mac-w-openwrt/
[20]: {{< baseurl >}}/post/openwrt-dwa-rozne-adresy-mac-na-porcie-wan/
[21]: {{< baseurl >}}/post/podzial-switcha-na-kilka-vlan-w-openwrt/
[22]: {{< baseurl >}}/post/konfiguracja-dmz-openwrt/
[23]: {{< baseurl >}}/post/dhcp-dns-czyli-konfiguracja-sieci-w-openwrt/
[24]: {{< baseurl >}}/post/siec-bezprzewodowa-wifi-w-openwrt-wlan/
[25]: {{< baseurl >}}/post/openwrt-konfiguracja-anten-via-txantenna-rxantenna/
[26]: {{< baseurl >}}/post/wps-czyli-wifi-protected-setup-w-openwrt/
[27]: {{< baseurl >}}/post/konfiguracja-wisp-openwrt-tryb-sta-ap/
[28]: {{< baseurl >}}/post/most-bezprzewodowy-openwrt-tryb-wds/
[29]: {{< baseurl >}}/post/karta-wifi-trybie-monitor-openwrt/
[30]: {{< baseurl >}}/post/bezprzewodowa-siec-goscinna-guest-wlan/
[31]: {{< baseurl >}}/post/rozne-adresy-lan-wlan-openwrt-routed-ap/
[32]: {{< baseurl >}}/post/dysk-pendrive-inne-nosniki-pod-openwrt/
[33]: {{< baseurl >}}/post/extroot-whole_root-fullroot-pod-openwrt/
[34]: {{< baseurl >}}/post/zmiana-rozmiaru-katalogu-tmp-pod-openwrt/
[35]: {{< baseurl >}}/post/filtr-pakietow-sieciowych-w-openwrt-firewall/
[36]: {{< baseurl >}}/post/modem-lte-pod-openwrt/
[37]: {{< baseurl >}}/post/stale-nazwy-urzadzen-openwrt-hotplug-udev/
[38]: {{< baseurl >}}/post/monitor-polaczenia-3glte-w-openwrt-3ginfo/
[39]: {{< baseurl >}}/post/obsluga-sms-kodow-ussd-w-openwrt/
[40]: {{< baseurl >}}/post/konfiguracja-polaczenia-aero2-na-openwrt/
[41]: {{< baseurl >}}/post/automatyczna-blokada-internetu-lte-w-openwrt/
[42]: {{< baseurl >}}/post/konfiguracja-dnscrypt-proxy-w-openwrt/
[43]: {{< baseurl >}}/post/wake-lan-z-etherwake-pod-openwrt/
[44]: {{< baseurl >}}/post/drukarka-sieciowa-w-openwrt-serwer-wydruku/
[45]: {{< baseurl >}}/post/sieciowy-system-plikow-openwrt-nfs/
[46]: {{< baseurl >}}/post/serwer-ftp-routerze-openwrt-vsftpd/
[47]: {{< baseurl >}}/post/statystyki-openwrt-collectd-rrdtool/
[48]: {{< baseurl >}}/post/jak-zablokowac-facebook-youtube-openwrt/
[49]: {{< baseurl >}}/post/router-openwrt-jako-serwer-klient-radius/
[50]: {{< baseurl >}}/post/ksztaltowanie-ruchu-qos-scripts-openwrt/
[51]: {{< baseurl >}}/post/quality-service-qos-w-openwrt/
[52]: {{< baseurl >}}/post/failover-load-balancing-openwrt-mwan3/
[53]: {{< baseurl >}}/post/konfiguracja-tunelu-6in4-w-openwrt-ipv6/
[54]: {{< baseurl >}}/post/blokowanie-reklam-adblock-na-domowym-routerze-wifi/
[55]: {{< baseurl >}}/post/udostepnianie-lte-3g-ze-smartfona-przez-router-openwrt-tethering/
