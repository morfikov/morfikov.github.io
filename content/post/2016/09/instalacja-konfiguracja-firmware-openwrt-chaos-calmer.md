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
pigułce"](https://tplinkforum.pl/t/openwrt-w-pigulce-konfiguracja-w-oparciu-o-tl-wr1043nd-oraz-archer-c7/6960/)
, z tym, że tamten artykuł dotyczy wydania Barrier Breaker. Artykuły, do których linki znajdują się
poniżej, odwołują się do wydania Chaos Calmer i rozwiązania opisane w nich powinny na tej wersji
firmware działać bez problemu. Mogą natomiast pojawić się problemy w przypadku konfiguracji
starszych wersji OpenWRT na naszym routerze WiFi.

<!--more-->
## Spis artykułów dotyczących firmware OpenWRT Chaos Calmer

  - [Jak wgrać firmware OpenWRT Chaos Calmer na router
    TP-LINK]({{< baseurl >}}/post/jak-wgrac-firmware-openwrt-na-router-tp-link/)
  - Uzyskiwanie dostępu do routera WiFi:
      - [SSH, SSHFS oraz telnet]({{< baseurl >}}/post/dostep-routera-openwrt-telnet-ssh-sshfs/)
      - [Likwidacja hasła dostępowego na rzecz kluczy
        SSH]({{< baseurl >}}/post/klucze-szyfrujace-rsa-w-openwrt-ssh/)
      - [Tryb ratunkowy (failsafe)]({{< baseurl >}}/post/tryb-ratunkowy-failsafe-w-openwrt/)
      - [Reset ustawień (firstboot)]({{< baseurl >}}/post/reset-ustawien-w-openwrt-firstboot/)
      - [Konsola
        szeregowa]({{< baseurl >}}/post/konsola-szeregowa-adapter-usb-uart-uszkodzony-router-tp-link/)
      - [Tryb
        recovery]({{< baseurl >}}/post/jak-przy-pomocy-trybu-recovery-odzyskac-router-tp-link/)
  - [Menadżer pakietów OPKG]({{< baseurl >}}/post/opkg-czyli-menadzer-pakietow-w-openwrt/)
  - [Backup ustawień i aktualizacja firmware do nowszej wersji
    (sysupgrade)]({{< baseurl >}}/post/sysupgrade-czyli-aktualizacja-firmware-openwrt/)
  - [Jak powrócić do oryginalnego firmware
    TP-LINK'a]({{< baseurl >}}/post/jak-powrocic-z-firmware-openwrt-tp-linka/)
  - [Skrypty startowe init]({{< baseurl >}}/post/skrypty-startowe-init-w-openwrt/)
  - Konfiguracja systemu (/etc/config/system):
      - [Nazwa hosta i domena routera]({{< baseurl >}}/post/hostname-czyli-nazwa-hosta-w-openwrt/)
      - [Strefa czasowa i czas sieciowy
        (NTP)]({{< baseurl >}}/post/strefa-czasowa-timezone-w-openwrt/)
      - [Logi systemowe (logread)]({{< baseurl >}}/post/logread-czyli-system-logowania-w-openwrt/)
      - [Konfiguracja diod
        LED]({{< baseurl >}}/post/konfiguracja-diod-w-routerze-pod-openwrt-led/)
  - [Przyciski]({{< baseurl >}}/post/konfiguracja-przyciskow-w-openwrt/)
  - Konfiguracja sieci (/etc/config/network):
      - [Interfejsy sieciowe (lo, br-lan, eth0/eth1) oraz
        IPv6]({{< baseurl >}}/post/konfiguracja-interfejsow-sieciowych-w-openwrt/)
      - [Klonowanie adresów MAC]({{< baseurl >}}/post/jak-sklonowac-adres-mac-w-openwrt/)
      - [Problemy z ISP za sprawą podwójnego adresu
        MAC]({{< baseurl >}}/post/openwrt-dwa-rozne-adresy-mac-na-porcie-wan/)
      - [Podziała switch'a na kilka VLAN'ów (2x
        WAN)]({{< baseurl >}}/post/podzial-switcha-na-kilka-vlan-w-openwrt/)
      - [Konfiguracja DMZ]({{< baseurl >}}/post/konfiguracja-dmz-openwrt/)
  - [DNS, DHCP, statyczne lease DHCP oraz opcje
    DHCP]({{< baseurl >}}/post/dhcp-dns-czyli-konfiguracja-sieci-w-openwrt/) (/etc/config/dhcp)
  - Konfiguracja WiFi (/etc/config/wireless)
      - [Tryb AP, szerokość kanałów, kanały 12 i 13, moc transmisyjna, ukrycie sieci, filtr adresów
        MAC]({{< baseurl >}}/post/siec-bezprzewodowa-wifi-w-openwrt-wlan/)
      - [Konfiguracja
        anten]({{< baseurl >}}/post/openwrt-konfiguracja-anten-via-txantenna-rxantenna/)
      - [WPS]({{< baseurl >}}/post/wps-czyli-wifi-protected-setup-w-openwrt/)
      - [Tryb WISP (Wireless ISP)]({{< baseurl >}}/post/konfiguracja-wisp-openwrt-tryb-sta-ap/)
      - [Tryb WDS (most bezprzewodowy)]({{< baseurl >}}/post/most-bezprzewodowy-openwrt-tryb-wds/)
      - [Tryb MONITOR]({{< baseurl >}}/post/karta-wifi-trybie-monitor-openwrt/)
      - [Bezprzewodowa sieć
        gościnna]({{< baseurl >}}/post/bezprzewodowa-siec-goscinna-guest-wlan/)
      - [Różne klasy adresowe dla LAN i
        WLAN]({{< baseurl >}}/post/rozne-adresy-lan-wlan-openwrt-routed-ap/)
  - Nośniki wymienne (/etc/config/fstab)
      - [Dyski zewnętrzne, pendrive, SWAP (fdisk, mkfs,
        block-mount)]({{< baseurl >}}/post/dysk-pendrive-inne-nosniki-pod-openwrt/)
      - [Extroot i whole\_root
        (fullroot)]({{< baseurl >}}/post/extroot-whole_root-fullroot-pod-openwrt/)
      - [Zmiana rozmiaru katalogu
        /tmp/]({{< baseurl >}}/post/zmiana-rozmiaru-katalogu-tmp-pod-openwrt/)
  - [Filtr pakietów
    sieciowych]({{< baseurl >}}/post/filtr-pakietow-sieciowych-w-openwrt-firewall/)
    (/etc/config/firewall)
  - Modem 3G/4G/LTE
      - [Konfiguracja modemu LTE Huawei E3372s]({{< baseurl >}}/post/modem-lte-pod-openwrt/)
      - [Stałe nazwy urządzeń (udev,
        hotplug)]({{< baseurl >}}/post/stale-nazwy-urzadzen-openwrt-hotplug-udev/)
      - [Monitor połączenia LTE
        (3ginfo)]({{< baseurl >}}/post/monitor-polaczenia-3glte-w-openwrt-3ginfo/)
      - [SMS i kody USSD (gnokii, polecenia
        AT)]({{< baseurl >}}/post/obsluga-sms-kodow-ussd-w-openwrt/)
      - [Konfiguracja Aero2]({{< baseurl >}}/post/konfiguracja-polaczenia-aero2-na-openwrt/)
      - [Automatyczna blokada internetu
        LTE]({{< baseurl >}}/post/automatyczna-blokada-internetu-lte-w-openwrt/)
  - Dodatkowe usługi:
      - [Szyfrowanie zapytań DNS
        (dnscrypt-proxy)]({{< baseurl >}}/post/konfiguracja-dnscrypt-proxy-w-openwrt/)
      - [Wake on LAN (WoL) (etherwake)]({{< baseurl >}}/post/wake-lan-z-etherwake-pod-openwrt/)
      - [Drukarka sieciowa
        (p910nd)]({{< baseurl >}}/post/drukarka-sieciowa-w-openwrt-serwer-wydruku/)
      - [Sieciowy system plików NFS]({{< baseurl >}}/post/sieciowy-system-plikow-openwrt-nfs/)
      - [FTP (vsftpd)]({{< baseurl >}}/post/serwer-ftp-routerze-openwrt-vsftpd/)
      - [Statystyki routera (collectd)]({{< baseurl >}}/post/statystyki-openwrt-collectd-rrdtool/)
      - [Blokada Facebook, YouTube i innych portali społecznościowych (iptables, dnsmasq i
        ipset)]({{< baseurl >}}/post/jak-zablokowac-facebook-youtube-openwrt/)
      - [Serwer RADIUS
        (freeradius)]({{< baseurl >}}/post/router-openwrt-jako-serwer-klient-radius/)
      - [Kształtowanie ruchu
        (qos-scripts)]({{< baseurl >}}/post/ksztaltowanie-ruchu-qos-scripts-openwrt/)
      - [Quality of Service (QoS) (tc i
        iptables)]({{< baseurl >}}/post/quality-service-qos-w-openwrt/)
      - [Failover i loadbalancking
        (mwan3)]({{< baseurl >}}/post/failover-load-balancing-openwrt-mwan3/)
      - [Tunel 6in4 (IPv6)]({{< baseurl >}}/post/konfiguracja-tunelu-6in4-w-openwrt-ipv6/)
      - [Adblock i blokowanie
        reklam]({{< baseurl >}}/post/blokowanie-reklam-adblock-na-domowym-routerze-wifi/)
      - [Udostępnianie połączenia LTE/3G ze smartfona w sieci
        lokalnej]({{< baseurl >}}/post/udostepnianie-lte-3g-ze-smartfona-przez-router-openwrt-tethering/)
