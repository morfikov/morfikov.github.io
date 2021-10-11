---
author: Morfik
categories:
- OpenWRT
date: "2016-04-22T16:19:41Z"
date_gmt: 2016-04-22 14:19:41 +0200
published: true
status: publish
tags:
- chaos-calmer
- sieć
- aero2
- lte
- router
- modem
- huawei
- e3372
GHissueID: 434
title: Konfiguracja połączenia Aero2 na OpenWRT
---

Darmowy internet oferowany przez Aero2 nie grzeszy zbytnio osiągami, bo mamy do dyspozycji jedynie
512 kbit/s. Niemniej jednak, taka prędkość w zupełności wystarcza do przeglądania stron
internetowych. Gorzej z oglądaniem materiałów video w serwisach takich jak YouTube. To jednak nie ma
raczej większego znaczenia, bo przecie usługa jest za free, a poza tym, możemy dokupić szereg
pakietów aktywujących pełną prędkość w technologi LTE. Problem w tym, że od Aero2 możemy otrzymać
tylko jedną kartę SIM na użytkownika. Może i mamy możliwość dostania kilku kart na jeden adres
zamieszkania ale i tak trzeba by dla każdego SIM załatwić osobny modem LTE. Zwykle takie rozwiązanie
nie wchodzi w grę, zwłaszcza, gdy z internetu korzystamy raczej sporadycznie. Mając jednak router z
alternatywnym firmware OpenWRT, możemy do jednego z jego portów USB doczepić taki modem i udostępnić
połączenie internetowe wszystkim urządzeniom w naszej sieci domowej. W tym artykule postaramy się to
zadanie zrealizować.

<!--more-->
## Aktualizacja OpenWRT i instalacja potrzebnych pakietów

Przed przystąpieniem do konfiguracji routera, dobrze jest zaktualizować firmware OpenWRT do
najnowszej dostępnej wersji. W ten sposób unikniemy wszelkich problemów z zależnościami wynikającymi
z rozbieżności wersji oprogramowania, które będziemy musieli doinstalować. [Proces aktualizacji
firmware OpenWRT został opisany w osobnym
wątku](/post/sysupgrade-czyli-aktualizacja-firmware-openwrt/).

Standardowo OpenWRT nie ma zaimplementowanej obsługi modemów LTE i musimy ręcznie doinstalować
szereg rzeczy. To jakie pakiety będziemy musieli wgrać zależy w dużej mierze od tego jaki model
modemu posiadamy. W tym przypadku mamy do czynienia z Huawei E3372s-153 w wersji NON-HiLink. Ten
modem potrafi pracować w [trybie NDIS
(NCM)](https://en.wikipedia.org/wiki/Network_Driver_Interface_Specification). Będą nam zatem
potrzebne sterowniki zawarte w pakiecie `kmod-usb-net-huawei-cdc-ncm` . Do tego będziemy potrzebować
wparcia dla portów USB i kilku dodatkowych pakietów. Poniżej jest kompletna lista rzeczy, które
musimy wgrać, przynajmniej w przypadku tego modemu:

    # opkg update
    # opkg install \
    kmod-usb-core \
    kmod-usb2 \
    kmod-usb-serial \
    kmod-usb-serial-option \
    usb-modeswitch \
    comgt-ncm \
    comgt \
    chat \
    wwan \
    kmod-usb-net-huawei-cdc-ncm

Trochę tych pakietów jest i do tego dojdą jeszcze zależności. Ogromne znaczenie może mieć zatem
ilość wolnego miejsca na flash'u routera. Po instalacji tych w/w rzeczy, ilość dostępnego miejsca
skurczyła się o około 0,5 MiB. W przypadku routera [TP-LINK TL-WDR3600
v1.5](http://www.tp-link.com.pl/products/details/TL-WDR3600.html), flash ma 8 MiB, z czego po
wgraniu firmware OpenWRT, do dyspozycji mamy około 4 MiB. Zatem w tym przypadku nie ma problemu.
Jeśli nie dysponujemy wymaganą ilością wolnego miejsca, to albo trzeba będzie dokupić nowszy router
z większym flash'em albo zrobić [extroot](http://eko.one.pl/?p=openwrt-externalroot).

## Konfiguracja połączenia Aero2

Przechodzimy teraz do konfiguracji połączenia. Standardowo interesuje nas plik
`/etc/config/network` . By tutaj skonfigurować połączenie Aero2, musimy przerobić odpowiednio
sekcję `wan` . Domyślnie ta sekcja w tym routerze wygląda tak:

    config interface 'wan'
            option ifname 'eth0.2'
            option proto 'dhcp'

Musimy ją przepisać do poniższej postaci:

    config interface 'wan'
          option 'device'   '/dev/ttyUSB0'
          option 'proto'    'ncm'
          option 'mode'     'preferlte'
          option 'pincode'  ''
          option 'apn'      'darmowy'
          option 'username' 'aero'
          option 'password' 'aero'

W pliku `/etc/gcom/ncm.json` są wyszczególnione tryby pracy modemu, które możemy wpisać w linijce z
`mode` . Mamy do dyspozycji `preferlte` , `preferumts` , `lte` , `umts` , `gsm` oraz `auto` . W
przypadku zabezpieczenia karty SIM kodem PIN, uzupełniamy odpowiednio opcję `pincode` . W przypadku
Aero2 `apn` to `darmowy` , natomiast `username` oraz `password` przyjmują wartości `aero` .
Zapisujemy plik i restartujemy modem wpisując polecenie `reboot` .

Poniżej jest krótka analiza logów. To na wypadek, gdyby pojawiły się jakieś problemy z połączeniem.
Przede wszystkim, modem możemy podłączać do portu USB (i wyciągnąć go) w locie. Nie musimy wyłączać
lub resetować routera, by to zrobić. Po podpięciu modemu, ten powinien zostać prawidłowo rozpoznany:

    kernel: usb 1-1.2: new high-speed USB device number 4 using ehci-platform
    kernel: option 1-1.2:1.0: GSM modem (1-port) converter detected
    kernel: usb 1-1.2: GSM modem (1-port) converter now attached to ttyUSB0
    kernel: option 1-1.2:1.1: GSM modem (1-port) converter detected
    kernel: usb 1-1.2: GSM modem (1-port) converter now attached to ttyUSB1
    kernel: huawei_cdc_ncm 1-1.2:1.2: MAC-Address: 00:1e:10:1f:00:00
    kernel: huawei_cdc_ncm 1-1.2:1.2: setting rx_max = 16384
    kernel: huawei_cdc_ncm 1-1.2:1.2: NDP will be placed at end of frame for this device.
    kernel: huawei_cdc_ncm 1-1.2:1.2: cdc-wdm0: USB WDM device
    kernel: huawei_cdc_ncm 1-1.2:1.2 wwan0: register 'huawei_cdc_ncm' at usb-ehci-platform-1.2, Huawei CDC NCM device, 00:1e:10:1f:00:00
    kernel: usb-storage 1-1.2:1.3: USB Mass Storage device detected
    kernel: scsi host1: usb-storage 1-1.2:1.3
    netifd: Interface 'wan' is setting up now
    kernel: scsi 1:0:0:0: Direct-Access     HUAWEI   TF CARD Storage  2.31 PQ: 0 ANSI: 2
    kernel: sd 1:0:0:0: [sda] Attached SCSI removable disk
    block: Unkown action change
    block: Unkown action change

Później OpenWRT przesyła, w zależności od konfiguracji, szereg poleceń AT do modemu:

    netifd: wan (2052): sending -> AT
    netifd: wan (2052): sending -> ATZ
    netifd: wan (2052): sending -> ATQ0
    netifd: wan (2052): sending -> ATV1
    netifd: wan (2052): sending -> ATE1
    netifd: wan (2052): sending -> ATS0=0
    netifd: wan (2052): sending -> AT^SYSCFGEX="030201",3fffffff,2,4,7fffffffffffffff,,
    netifd: wan (2052): sending -> AT^NDISDUP=1,1,"darmowy","aero","aero"

W przedostatniej linijce mamy `030201` , co sugeruje, że modem pracuje w trybie preferującym LTE z
równoczesną obsługą UMTS i GSM.

Dalej system próbuje uzyskać adresację za pomocą protokołu DHCP:

    netifd: wan (2052): Connected, starting DHCP
    kernel: huawei_cdc_ncm 1-1.2:1.2 wwan0: kevent 12 may have been dropped
    netifd: Interface 'wan_4' is enabled
    netifd: Interface 'wan_6' is enabled
    netifd: Interface 'wan' is now up
    netifd: Network device 'wwan0' link is up
    netifd: Network alias 'wwan0' link is up
    netifd: Interface 'wan_4' has link connectivity
    netifd: Interface 'wan_4' is setting up now
    netifd: Interface 'wan_6' has link connectivity
    netifd: Interface 'wan_6' is setting up now
    netifd: wan (2052): Command failed: Unknown error
    netifd: wan (2052): Command failed: Unknown error
    netifd: wan_4 (2205): udhcpc (v1.23.2) started
    netifd: wan_4 (2205): Sending discover...
    firewall: Reloading firewall due to ifup of wan (wwan0)
    netifd: wan_4 (2205): Sending discover...
    netifd: wan_4 (2205): Sending discover...
    netifd: wan_4 (2205): Sending select for 100.82.158.7...
    netifd: wan_4 (2205): Lease of 100.82.158.7 obtained, lease time 518400
    netifd: Interface 'wan_4' is now up

Adresacja została uzyskana, co otwiera nam drogę do korzystania z internetu Aero2. Dalej w logu mamy
jeszcze aplikowanie konfiguracji resolver'a DNS, która została uzyskana z lease DHCP:

    dnsmasq[1472]: reading /tmp/resolv.conf.auto
    dnsmasq[1472]: using local addresses only for domain lan
    dnsmasq[1472]: using nameserver fe80::c66e:1fff:fe95:effe%eth0.2#53
    dnsmasq[1472]: using nameserver 212.2.96.51#53
    dnsmasq[1472]: using nameserver 212.2.96.52#53

W tej chwili powinniśmy być w stanie uzyskać połączenie internetowe ze światem. Wyżej było kilka
błędów: `kernel: huawei_cdc_ncm 1-1.2:1.2 wwan0: kevent 12 may have been dropped` oraz `netifd:
wan (2052): Command failed: Unknown error` . Z tego co wyczytałem na necie, one pojawiają się
praktycznie zawsze od poprzedniego wydania OpenWRT, tj. Barrier Breaker. Mogą zostać zignorowane, bo
nie wpływają w żaden sposób na połączenie.

## Problemy związane z Aero2 pod OpenWRT

Może i połączenie jesteśmy w stanie uzyskać ale darmowa wersja Aero2 wymaga od nas wpisywania kodów
CAPTCHA. Bez tego połączenie nie będzie działać. Przechodzimy zatem na pierwszy lepszy serwis www
wpisując w przeglądarce internetowej jego adres. Powinniśmy zostać przekierowani na adres
`http://bdi.free.aero2.net.pl:8080/` . Na standardowej konfiguracji OpenWRT, nic się nam jednak nie
wyświetli. Musimy nieco zmienić konfigurację `dnsmasq` . Poniższy kod dopisujemy do pliku
`/etc/config/dhcp` :

    config dnsmasq
    ...
          option rebind_protection '1'
          option rebind_localhost '1'
          list rebind_domain 'free.aero2.net.pl'
    ...
    config domain
            option name 'bdi.free.aero2.net.pl'
            option ip '10.2.37.78'
    ...

Zapisujemy i restartujemy `dnsmasq` :

    # /etc/init.d/dnsmasq restart

Teraz już powinniśmy zostać przekierowani bez większego problemu na odpowiednią stronę:

![](/img/2016/04/1.aero2-dostep-do-internetu-www.png#big)

I możemy wpisać kod CAPTCHA:

![](/img/2016/04/2.aero2-wpisanie-captcha.png#big)

Po chwili dostęp do sieci powinien zostać przyznany:

![](/img/2016/04/3.aero2-przywrocenie-dostepu-internet.png#huge)

Nie trzeba również resetować połączenia, tak jak to [miało miejsce w
przeszłości](http://jdtech.pl/2015/04/aero2-rezygnuje-z-koniecznosci-rozlaczania-po-wpisaniu-kodu.html).
