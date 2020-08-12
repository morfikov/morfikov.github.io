---
author: Morfik
categories:
- Linux
date: "2016-09-26T18:29:27Z"
date_gmt: 2016-09-26 16:29:27 +0200
published: true
status: publish
tags:
- nmap
- arp
title: Jak wykryć komputer w sieci przy pomocy nmap
---

Opisywane jakiś czas temu przeze mnie [ekstendery
powerline]({{< baseurl >}}/post/transmitery-sieciowe-tl-wpa4226t-kit-tp-link/), tak bardzo mi
przypadły do gustu, że korzystam z nich praktycznie bez przerwy. Po skonfigurowaniu [roamingu sieci
WiFi w linux'ie]({{< baseurl >}}/post/jak-skonfigurowac-roaming-wifi-wpa_supplicant-linux/), mój
laptop w nieodczuwalny dla mnie w sposób rekonfiguruje połączenie w obrębie sieci domowej. Sygnał
jest na prawdę bardzo dobrej jakości w każdym zakątku domu, przez co w ogóle zapomniałem o istnieniu
tych transmiterów sieciowych. Chciałem przenieść jeden z ekstenderów z kanału 1 (domyślny), na kanał
6, który jest najmniej zatłoczony. Problem w tym, że zapomniałem jakim adresem IP dysponuje AP tego
urządzenia. Przestrzeń adresowa tej sieci to 192.168.1.0/24, czyli adresów jest 254 (192.168.1.0,
192.168.0.255 zarezerwowane dla broadcast). Nie uśmiechało mi się wpisywanie każdego z tych adresów
w przeglądarce, by ten panel admina odszukać metodą na chybił trafił. Na szczęście są o wiele
szybsze metody szukania maszyn w sieci, np. zapytania o adres IP w protokole ARP. A przy pomocy
[skanera nmap](https://nmap.org/) możemy w parę sekund ustalić adres IP wszystkich aktualnie
podłączonych komputerów do sieci.

<!--more-->
## Nmap i protokół ARP

Każdy host podłączony do sieci TCP/IP musi mieć przypisany jakiś adres IP. By taki adres IP
otrzymać, karta sieciowa takiego komputera musi posiadać adres MAC, który zwykle jest unikatowy dla
danego urządzenia. Rozwiązywanie adresów IP na MAC jest przeprowadzane za pomocą protokołu ARP przed
wysłaniem każdego pakietu.

Każdy host w sieci utrzymuje swoją tablicę IP-MAC. Wpisy w niej są dodawane po rozwiązaniu adresu IP
na MAC. Są one ważne przez jakiś czas i są regularnie odświeżane. Przy takim odświeżeniu, stare i
nieaktualne powiązania adresów są usuwane, przez co zwalniana jest zajmowana przez nie pamięć
operacyjna RAM.

Komunikacja w sieci odbywa się w oparciu o adres IP. Nasza maszyna przed wysłaniem pakietu
sieciowego musi znać ten adres IP. A w przypadku tych ekstenderów powerline właśnie adres nie jest
znany. Jak zatem ustalić gdzie znajduje się panel administracyjny? Możemy to zrobić odpytując każdy
pojedynczy host w sieci (ostatni oktet z zakresu 1-254). Nie będziemy tego robić ręcznie, bo
zajęłoby nam to sporo czasu. Możemy zapuścić `nmap` i niech on za nas odpyta wszystkie te hosty w
sieci. Zainstalujmy zatem w systemie pakiet `nmap` .

Skanowanie pod kątem obecności hostów w sieci możemy przeprowadzić z grubsza na dwa sposoby.
Pierwszym z nich jest wysłanie zapytań ARP pod każdy adres IP z naszej sieci. Drugim zaś wykonanie
pełnego skanu, w skład którego domyślnie wchodzi wysłanie pakietów ICMP echo request, TCP SYN na
port 443, TCP ACK na port 80 oraz ICMP timestamp request. Ten drugi sposób zajmuje o wiele więcej
czasu. Do skanowania maszyn w lokalnej sieci zwykle wykorzystuje się jedynie zapytania w protokole
ARP. Poniżej są przykłady obu tych skanów.

### Skanowanie ARP

W przypadku żądań ARP, w terminalu wpisujemy polecenie `nmap -sn 192.168.1.0/24` . Czas skanowania
zamknął się w granicach 3 sekund:

![]({{< baseurl >}}/img/2016/09/1.wykrywanie-komputer-host-siec-arp-nmap.png)

Poniżej jest jeszcze fotka z wireshark'a, która obrazuje cały
proces:

[![2.wykrywanie-komputer-host-siec-arp-wireshark]({{< baseurl >}}/img/2016/09/2.wykrywanie-komputer-host-siec-arp-wireshark-660x253.png)]({{< baseurl >}}/img/2016/09/2.wykrywanie-komputer-host-siec-arp-wireshark.png)

Jak widać, każdy pojedynczy adres IP został przez `nmap` odpytany. Z adresów, które są aktualnie
wykorzystywane w sieci, zostaje zwrócona odpowiedź, tak jak to widzimy w jednym przypadku wyżej na
fotce.

### Pełne skanowanie

Jeśli chcielibyśmy wykonać pełen skan wszystkich hostów w sieci, to w terminalu wydajemy polecenie
`nmap -sn --send-ip 192.168.1.0/24` . I tutaj już ten proces trwał ponad 20 sekund:

![]({{< baseurl >}}/img/2016/09/3.wykrywanie-komputer-host-siec-nmap-pelny-skan.png)

Poniżej jest zaś fotka z wireshark'a, która pokazuje ruch do jednego hosta przy korzystaniu z tego
typu
skanowania:

[![4.wykrywanie-komputer-host-siec-wireshark-pelny-skan-nmap]({{< baseurl >}}/img/2016/09/4.wykrywanie-komputer-host-siec-wireshark-pelny-skan-nmap-660x253.png)]({{< baseurl >}}/img/2016/09/4.wykrywanie-komputer-host-siec-wireshark-pelny-skan-nmap.png)

## Wykryte hosty w sieci

Jak widać, aktualnie w sieci są zalogowane cztery urządzenia. Jedno, z którego zostało wysłane
zapytanie, oraz trzy znalezione za sprawą protokołu ARP i pełnego skanowania. Znamy ich adresy MAC
oraz przypisane im adresy IP. Szukany transmiter sieciowy ma adres 192.168.1.11 i to ten adres
trzeba było wpisać w przeglądarce, by dostać się do panelu administracyjnego w celu przeniesienia AP
na inny
kanał:

[![5.siec-wifi-skan-smartfon]({{< baseurl >}}/img/2016/09/5.siec-wifi-skan-smartfon-401x660.png)]({{< baseurl >}}/img/2016/09/5.siec-wifi-skan-smartfon.png)
