---
author: Morfik
categories:
- OpenWRT
date: "2016-04-04T19:44:30Z"
date_gmt: 2016-04-04 17:44:30 +0200
published: true
status: publish
tags:
- chaos-calmer
- sieć
- lte
- router
- modem
- huawei
- e3372
GHissueID: 417
title: Modem LTE pod OpenWRT (Huawei E3372s-153)
---

Modem LTE jest zwykle przeznaczony dla jednej stacji roboczej. Podpina się go do portu USB i zwykle
po chwili można zestawić połączenie z siecią. W przypadku, gdy mamy kilka komputerów i na każdym z
nich chcemy mieć internet, to mamy z grubsza trzy wyjścia. Pierwszym z nich jest dokupienie
kolejnych modemów LTE, co zwykle nie wchodzi w grę. Drugą opcją jest zakup routera LTE. Różni się on
od zwykłego routera WiFi tym, że ma już wbudowany modem LTE. Jeśli jednak dysponujemy własnym
routerem WiFi, to niekoniecznie musimy się go pozbywać, zwłaszcza w przypadku, gdy już zakupiliśmy
osobno modem LTE. Jeśli na tym routerze WiFi jesteśmy w stanie zainstalować firmware OpenWRT, to
istnieje duża szansa na to, że damy radę ten router przerobić na router LTE. W tym wpisie postaramy
się ten zabieg przeprowadzić z wykorzystaniem routera [TP-LINK Archer C7
v2](http://www.tp-link.com.pl/products/details/cat-9_Archer-C7.html) oraz modemu LTE Huawei
E3372s-153 w wersji NON-HiLink.

<!--more-->
## Czy OpenWRT rozpozna modem Huawei E3372s-153

Przed przystąpieniem do przerabiania routera na router LTE, dobrze jest zaktualizować firmware do
najnowszej wersji Chaos Calmer (CC). Obrazy można pobrać
[stąd](http://eko.one.pl/?p=openwrt-chaos-calmer). W tym przypadku został użyty OpenWRT w wersji
15.05.1 (r49087). Jako, że modemy LTE się różnią między sobą dość znacznie, to inne pakiety trzeba
będzie zainstalować na routerze. Modem Huawei E3372s-153 może pracować w trybie RAS ([Remote Access
Services](https://en.wikipedia.org/wiki/Remote_Access_Service)) lub NDIS ([Network Driver Interface
Specification](https://en.wikipedia.org/wiki/Network_Driver_Interface_Specification)). Różnica
między tymi dwoma trybami polega na tym, że w trybie RAS modem działa przez interfejs szeregowy, a
połączenie jest realizowane przy pomocy demona PPP. Natomiast w przypadku trybu NDIS, modem
zachowuje się i działa jak zwykła karta sieciowa, przez co jesteśmy w stanie osiągnąć większą
prędkość transferu. W trybie RAS nie damy rady wyciągnąć więcej niż 25-30 mbit/s. Niemniej jednak,
trzeba się liczyć z faktem zwiększenia zapotrzebowania na RAM i procesor w przypadku, gdy chcemy
korzystać z trybu NDIS.

To czy modem obsługuje tryb NDIS można ustalić wydając modemowi polecenie `AT^SETPORT?` . By to
zrobić pod OpenWRT, musimy tam doinstalować pakiet `picocom` . Następnie logujemy się na router po
SSH i wydajemy w terminalu to poniższe polecenie:

    # picocom -b 115200 /dev/ttyUSB0

Prawdopodobnie będą nam przeskakiwały jakieś komunikaty i by je wyłączyć, musimy dodatkowo wydać te
dwa poniższe polecenia:

    ate1
    at^curc=0

W tej chwili jesteśmy w stanie rozmawiać z modem bez przeszkód. Odczytajmy zatem jakie porty ma
modem Huawei E3372s-153 :

    AT^SETPORT=?

    ^SETPORT:3: 3G DIAG
    ^SETPORT:10: 4G MODEM
    ^SETPORT:1: 3G MODEM
    ^SETPORT:12: 4G PCUI
    ^SETPORT:13: 4G DIAG
    ^SETPORT:5: 3G GPS
    ^SETPORT:14: 4G GPS
    ^SETPORT:A: BLUE TOOTH
    ^SETPORT:16: NCM
    ^SETPORT:A1: CDROM
    ^SETPORT:A2: SD

Sprawdźmy także aktualną ich konfigurację:

    AT^SETPORT?

    ^SETPORT:A1,A2;12,10,16,A1,A2

Wszystko co zostało zwrócone w poleceniu `AT^SETPORT?` jest włączone. Mamy tam także `16` , który
odpowiada za tryb NDIS. Jeśli byłby on wyłączony, to wtedy modem może nie chcieć się połączyć w
trybie NDIS. W przypadku, gdy faktycznie ten port nie jest włączony, to aktywujemy go w poniższy
sposób:

    AT^SETPORT="A1,A2;12,10,16,A1,A2"
    AT^RESET

Mamy zatem modem w trybie NDIS. By działał on nam pod OpenWRT musimy doinstalować kilka pakietów.
Przede wszystkim, potrzebne nam jest wsparcie dla portu USB, a to realizowane jest przez pakiety
`kmod-usb-core` , `kmod-usb2` , `kmod-usb-serial` oraz `kmod-usb-serial-option` . Potrzebujemy także
pakietu `usb-modeswitch` , który przełączy nam tryb modemu. Chodzi o to, że standardowo modemy LTE
mają w sobie zaszyte sterowniki dla systemu windows. Po podpięciu takiego modemu do portu USB w
komputerze, system wykrywa go jako CD-ROM lub pendrive, z którego jesteśmy w stanie te sterowniki
zainstalować. Po instalacji sterowników, w systemie pojawia nam się modem w miejscu tego
poprzedniego urządzenia. By zrealizować takie zachowanie na linux'ach, w tym też i OpenWRT,
potrzebny nam jest właśnie ten `usb-modeswitch` . Musimy także doinstalować pakiet `comgt-ncm` ,
który zapewni wsparcie dla modemu pracującego w trybie NDIS (NCM). Ten pakiet dociągnie kilka
zależności, min `chat` , `comgt` i `wwan` . Poniżej dla uproszenia jest polecenie instalujące te
wszystkie rzeczy:

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
    wwan

Modemu Huawei E3372s-153 nie da rady odpalić bez odpowiedniego sterownika kernela. W OpenWRT są
dostępne dwa: `kmod-huawei-hw-cdc` oraz `kmod-usb-net-huawei-cdc-ncm` . Ten pierwszy pochodzi od
producenta Huawei i nie jest już rozwijany od lat. Dlatego też powinniśmy korzystać z
`kmod-usb-net-huawei-cdc-ncm` :

    # opkg install kmod-usb-net-huawei-cdc-ncm

W przypadku routera TP-LINK Archer C7 v2, modem został podpięty bezpośrednio do portu USB z
pominięciem aktywnego HUB'a USB. Transfer jaki udało mi się póki co zarejestrować to około 25-30
mbit/s w obie strony i jest to mniej więcej tyle ile szło wyciągnąć przy podłączeniu tego modemu
bezpośrednio do komputera. Warto jednak zaznaczyć, że routery WiFi nie mają zbyt wydolnych
napięciowo portów USB i w sporej części przypadków wymagane będzie zastosowanie aktywnego HUB'a USB
z zasilaczem. W przeciwnym wypadku modem może nie być w stanie, np. przełączyć się w tryb LTE.
Archer C7 v2 zdaje się radzić sobie bez tego dodatkowego balastu.

Po zainstalowaniu potrzebnych nam pakietów, modem powinien zostać rozpoznany przez OpenWRT mniej
więcej w poniższy sposób:

    # cat /sys/kernel/debug/usb/devices
    ...
    T:  Bus=02 Lev=01 Prnt=01 Port=00 Cnt=01 Dev#=  2 Spd=480  MxCh= 0
    D:  Ver= 2.10 Cls=00(>ifc ) Sub=00 Prot=00 MxPS=64 #Cfgs=  1
    P:  Vendor=12d1 ProdID=15b6 Rev= 1.02
    S:  Manufacturer=HUAWEI_MOBILE
    S:  Product=HUAWEI_MOBILE
    C:* #Ifs= 5 Cfg#= 1 Atr=c0 MxPwr=  2mA
    I:* If#= 0 Alt= 0 #EPs= 2 Cls=ff(vend.) Sub=02 Prot=12 Driver=option
    E:  Ad=82(I) Atr=02(Bulk) MxPS= 512 Ivl=0ms
    E:  Ad=02(O) Atr=02(Bulk) MxPS= 512 Ivl=0ms
    I:* If#= 1 Alt= 0 #EPs= 3 Cls=ff(vend.) Sub=02 Prot=10 Driver=option
    E:  Ad=84(I) Atr=03(Int.) MxPS=  10 Ivl=32ms
    E:  Ad=83(I) Atr=02(Bulk) MxPS= 512 Ivl=0ms
    E:  Ad=03(O) Atr=02(Bulk) MxPS= 512 Ivl=0ms
    I:  If#= 2 Alt= 0 #EPs= 1 Cls=ff(vend.) Sub=02 Prot=16 Driver=huawei_cdc_ncm
    E:  Ad=86(I) Atr=03(Int.) MxPS=  16 Ivl=2ms
    I:* If#= 2 Alt= 1 #EPs= 3 Cls=ff(vend.) Sub=02 Prot=16 Driver=huawei_cdc_ncm
    E:  Ad=86(I) Atr=03(Int.) MxPS=  16 Ivl=2ms
    E:  Ad=85(I) Atr=02(Bulk) MxPS= 512 Ivl=0ms
    E:  Ad=04(O) Atr=02(Bulk) MxPS= 512 Ivl=0ms
    I:* If#= 3 Alt= 0 #EPs= 2 Cls=08(stor.) Sub=06 Prot=50 Driver=usb-storage
    E:  Ad=87(I) Atr=02(Bulk) MxPS= 512 Ivl=0ms
    E:  Ad=05(O) Atr=02(Bulk) MxPS= 512 Ivl=125us
    I:* If#= 4 Alt= 0 #EPs= 2 Cls=08(stor.) Sub=06 Prot=50 Driver=usb-storage
    E:  Ad=88(I) Atr=02(Bulk) MxPS= 512 Ivl=0ms
    E:  Ad=06(O) Atr=02(Bulk) MxPS= 512 Ivl=125us

## Konfiguracja modemu Huawei E3372s-153 pod OpenWRT

Potrzebujemy jeszcze konfiguracji, która umożliwi nam nawiązanie połączenia z internetem. Interesuje
nas głównie plik `/etc/config/network` . Odszukujemy w nim sekcję `config interface 'wan'` i
przepisujemy ją do poniższej postaci:

    config interface 'wan'
          option 'device'   '/dev/ttyUSB0'
          option 'proto'    'ncm'
          option 'mode'     'lte'
          option 'pincode'  ''
          option 'apn'      'internet'
          option 'username' ''
          option 'password' ''

To jakie wartości umieścimy wyżej zależy głównie od operatora, z którego zamierzamy korzystać. W tym
przypadku połączenie zapewnia Play, a jako że oferuje on [darmowy internet
LTE](/post/darmowy-internet-lte-od-rbmplay/), to trzeba tutaj wymusić tryb
połączenia LTE przy pomocy opcji `mode` . Jeśli korzystamy z innych operatorów i nie koniecznie
łączymy się po LTE, to w pliku `/etc/gcom/ncm.json` są wyszczególnione tryby pracy modemu, z
których możemy skorzystać. Mamy do dyspozycji `preferlte` , `preferumts` , `lte` , `umts` , `gsm`
oraz `auto` . W przypadku zabezpieczenia karty SIM kodem PIN, uzupełniamy odpowiednio opcję
`pincode` . Opcje `apn` , `username` `password` dostosowujemy zgodnie z instrukcjami operatora.

Zapisujemy teraz konfigurację i uruchamiamy router ponownie wpisując w konsoli polecenie `reboot` .
Po chwili router powinien się odpalić, a modem połączyć z internetem. W przypadku odłączenia modemu
i jego ponownego podłączenia, nie musimy restartować routera. System wykryje podłączony modem i
automatycznie postara się skonfigurować połączenie.
