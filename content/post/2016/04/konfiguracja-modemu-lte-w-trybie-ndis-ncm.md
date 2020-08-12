---
author: Morfik
categories:
- Linux
date: "2016-04-05T15:45:10Z"
date_gmt: 2016-04-05 13:45:10 +0200
published: true
status: publish
tags:
- debian
- sieć
- lte
- modem
- huawei
- e3372
title: Konfiguracja modemu LTE w trybie NDIS (NCM)
---

Sporo modemów LTE potrafi pracować w kilku trybach. Weźmy na przykład modem Huawei E3372s-153 w
wersji NON-HiLink. Standardowo obsługuje on tryb RAS ([Remote Access
Services](https://en.wikipedia.org/wiki/Remote_Access_Service)) jak i NDIS ([Network Driver
Interface Specification](https://en.wikipedia.org/wiki/Network_Driver_Interface_Specification)).
Domyślnie też włączony jest NDIS ale, by móc z tego trybu korzystać na debianie, musimy nieco
inaczej skonfigurować sobie połączenie sieciowe. Gdy w grę wchodzą modemy LTE, to użytkownicy zwykli
korzystać z narzędzia `wvdial` , który zaprzęga do pracy demona PPP i w ten sposób modem zaczyna
pracować w trybie RAS, a nie NDIS. W tym wpisie skonfigurujemy sobie połączenie sieciowe na debianie
w taki sposób, by wykorzystywało ono potencjał trybu NDIS.

<!--more-->
## Modemy LTE i pakiet usb-modeswitch

[Z pakietu usb-modeswitch można
zrezygnować]({{< baseurl >}}/post/modem-lte-huawei-e3372-bez-usb-modeswitch/), o ile posiadamy
odpowiedni typ modemu LTE obsługujący polecenia AT.

Jako, że w modemach LTE znajdują się sterowniki dla systemów windows, to taki modem zwykle z
początku jest wykrywany jako pendrive czy CD-ROM. Przy takim stanie rzeczy nie będziemy mieć
odpowiednich urządzeń w katalogu `/dev/` , tj. `/dev/ttyUSB0` , etc. Dlatego też musimy doinstalować
w miarę nowy pakiet `usb-modeswitch` , który zajmie się przełączeniem stanu modemu tuż po podpięciu
go do portu USB.

## Jak ustalić czy modem LTE pracuje w trybie NDIS (NCM)

W przypadku, gdy nie wiemy jakie tryby pracy obsługuje nasz modem LTE, to przydałoby się pierw to
ustalić. Najprościej jest do tego celu wykorzystać polecenia AT. Na debianie, czy też w ogóle na
linux'ach, można bezpośrednio zapisywać urządzenie `/dev/ttyUSB0` przy pomocy `echo` . Ja jednak
wolę korzystać z odpowiedniego oprogramowania, np. `cu` , `socat` czy też `minicom` . Tutaj
posłużymy się `cu` . Instalujemy zatem ten pakiet i pytamy modem o obsługiwane tryby pracy w
poniższy sposób:

    $ cu -l /dev/ttyUSB0
    Connected.

    ate1
    at^curc=0

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

Jeśli nasz modem posiada port `16` , to obsługuje tryb NDIS. Zobaczmy czy ten port jest włączony.
Wpisujemy zatem to poniższe polecenie:

    AT^SETPORT?

    ^SETPORT:A1,A2;12,10,16,A1,A2

W tym przypadku jest. Jeśli w powyższym wyniku nie będzie portu `16` ale nasz modem go obsługuje, to
musimy ten port włączyć. Robimy to w poniższy sposób:

    AT^SETPORT="A1,A2;12,10,16,A1,A2"
    AT^RESET

By wyjść z programu `cu` musimy wpisać ~ i . (tylda i kropka).

## Konfiguracja połączenia

Po podpięciu modemu LTE w trybie NDIS do portu USB, w debianie powinien pojawić się nowy interfejs
sieciowy. Zwykle jest to `wwan0` . Możemy to odczytać z wyjścia poleceń `ifconfig` lub `ip` ,
przykładowo:

    # ip addr show wwan0
    9: wwan0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN group default qlen 1000
        link/ether 00:1e:10:1f:00:00 brd ff:ff:ff:ff:ff:ff
        inet6 fe80::21e:10ff:fe1f:0/64 scope link
           valid_lft forever preferred_lft forever

Jak widzimy wyżej, ten interfejs nie posiada jeszcze żadnej adresacji i musimy go skonfigurować. Na
debianie konfiguracja interfejsów odbywa się albo przez plik `/etc/network/interfaces` , albo też
można wykorzystać do tego moduł oferowany przez systemd. W tym przypadku skorzystamy z opcji
pierwszej. Edytujemy zatem wspomniany plik i dodajemy do niego ten poniższy blok kodu:

    allow-hotplug wwan0
    iface wwan0 inet dhcp
          pre-up sleep 1
          pre-up echo -e "AT^SYSCFGEX=\"03\",3FFFFFFF,1,2,800C5,,\r" > /dev/ttyUSB0
          pre-up echo -e "AT^NDISDUP=1,1,\"internet\"\r" > /dev/ttyUSB0
          pre-down echo -e "AT^NDISDUP=1,0\r" > /dev/ttyUSB0

Linijka z `allow-hotplug` będzie podnosić interfejs przy starcie systemu oraz przy podłączaniu
modemu LTE do portu USB. Konfiguracja tego interfejsu odbywa się za pomocą protokołu DHCP. By jednak
adresacja mogła być pobrana, musimy odpowiednio skonfigurować modem. Dlatego właśnie korzystamy z
dyrektyw `pre-up` , które zawierają polecenia AT. Te komendy zostaną wykonane tuż przed
podniesieniem interfejsu. Pierwsze z poleceń AT wymusza tryb połączenie LTE, drugie łączy z APN
`internet` . W linijce z `pre-down` jest polecenie AT, które zostanie zaaplikowane przed położeniem
interfejsu. W tym przypadku rozłączenie z APN. Generalnie rzecz biorąc, komendy mogą być dowolne,
tj. w ramach obsługiwanych przez modem poleceń AT. W moim przypadku trzeba było opóźnić nieco
przesyłanie poleceń AT przy podłączaniu modemu do portu USB, dlatego tam na początku mamy linijkę z
`sleep 1` . Bez niej, modem zwyczajnie nie mógł uzyskać adresacji via DHCP.
