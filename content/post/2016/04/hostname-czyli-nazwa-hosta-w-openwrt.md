---
author: Morfik
categories:
- OpenWRT
date: "2016-04-27T23:49:32Z"
date_gmt: 2016-04-27 21:49:32 +0200
published: true
status: publish
tags:
- chaos-calmer
- router
- hostname
GHissueID: 406
title: Hostname, czyli nazwa hosta w OpenWRT
---

Po wgraniu świeżego firmware OpenWRT, mamy domyślnie skonfigurowany system, który umożliwia nam
nawiązanie połączenia sieciowego ze światem. Niemniej jednak, w przypadku posiadania w domu kilku
stacji roboczych, zapamiętanie ich adresów IP może być niemałym problemem. Niemniej jednak, każdemu
hostowi w sieci możemy przypisać nazwę, tzw. hostname. W ten sposób możemy powiązać nazwy
poszczególnych komputerów z przypisanymi in adresami IP. To rozwiązanie ma tę zaletę, że nie musimy
pamiętać już adresów IP. Możemy za to posługiwać się nazwami, które są o wiele łatwiejsze do
zapamiętania dla człowieka. Działa to mniej więcej na tej samej zasadzie co domeny internetowe. W
tym wpisie postaramy się skonfigurować domenę, w której pracuje router oraz nazwy hostów w sieci.

<!--more-->
## Domyślny hostname i domain name

Każdy router po flash'owaniu, ma ustawiony adres IP na 192.168.1.1 . Nazwa hosta również została
przypisana i jest to `OpenWrt` . Możemy się o tym przekonać nawiązując połączenie z routerem przez
protokół telnet lub SSH. W takim przypadku, nazwa hosta jest wyświetlana koło prompt'a:

![domyslny-hostname-openwrt](/img/2016/04/1.domyslny-hostname-openwrt.png#big)

Wyżej logowaliśmy się na router z wykorzystaniem jego adresu IP. Nic jednak nie stoi na
przeszkodzie, by zacząć się posługiwać jego hostname. Poniżej przykład z wykorzystaniem polecenia
`ping` :

![ping-po-hostname-openwrt](/img/2016/04/2.ping-po-hostname-openwrt.png#big)

Wielkość znaków nie ma znaczenia. Można używać małych, a i tak odpowie nam prawidłowa maszyna. Z
tym, że w odpowiedzi widzimy zapis `OpenWrt.lan (192.168.1.1)` . Ustaliliśmy już, że `OpenWrt`
odpowiada za nazwę hosta. Za co zatem odpowiada `.lan` ? Jest to standardowa nazwa domeny, w której
pracuje router. Jeśli przeszkadzają nam te domyślne wartości i mamy lepsze odpowiedniki dla nich, to
nic nie stoi na przeszkodzie, by dostosować sobie ten nazwy.

## Zmiana hostname routera

Hostname dla routera pod kontrolą OpenWRT jest ustawiany standardowo za pomocą skryptu init
`/etc/init.d/system` . Przyjmuje on konfigurację z pliku `/etc/config/system` , a tam z kolei mamy
min. ten poniższy wpis:

    config system
          option hostname 'OpenWrt'
    ...

Nazwa hosta ustawiona tutaj jest przesyłana przy pomocy `echo` do pliku
`/proc/sys/kernel/hostname` .

## Zmiana domeny routera

Zasada rozwiązywania nazw domenowych w lokalnej sieci domowej jest mniej więcej taka sama co w
przypadku wysyłania żądania do zdalnego serwera www. Weźmy dla przykładu ten powyższy `ping` .
Chcieliśmy odpytać hosta o nazwie `openwrt` , a że komputery operują na adresach IP, to ta nazwa
musiała zostać pierw rozwiązana przez serwer DNS. W OpenWRT za serwer DNS robi [oprogramowanie
dnsmasq](https://wiki.openwrt.org/doc/howto/dhcp.dnsmasq), którego konfiguracja jest umieszczona w
pliku `/etc/config/dhcp` . Interesujące nas wpisy są poniżej:

    config dnsmasq
    ...
          option local '/lan/'
          option domain 'lan'
          option expandhosts '1'
    ...

Opcja `local` sprawia, że wszystkie zapytania w tej domenie będą rozwiązywane lokalnie, tj. w
oparciu o plik `/etc/hosts` lub lease wydawane za sprawą protokołu DHCP. Opcja `domain` ustawia
domenę dla routera. W efekcie wszystkie hosty, które otrzymują konfiguracje od serwera DHCP, będą
skonfigurowane na tę domenę. Dodatkowo, nazwa określona tutaj jest wykorzystywana w opcji
`expandhosts` , która z kolei odpowiada za dodanie części domenowej do nazwy hostów, które jej nie
posiadają.

W taki oto sposób, ping'ując router tylko po nazwie, do `dnsmasq` trafia żądanie bez części
domenowej. Takie żądania są rozwiązywane lokalnie. Akurat w przypadku samego routera, nazwa hosta
nie jest określona ani w pliku `/etc/hosts` ani nie została uzyskana od serwera DHCP routera, który
nasłuchuje zapytań z sieci. Niemniej jednak, router jest świadom swojej nazwy oraz domeny, w której
pracuje. Dlatego też potrafi rozwiązać przesłaną nazwę hosta i zwrócić odpowiedź w żądaniu `ping` .

Możemy zweryfikować ten powyższy schemat działań przez przesłanie zapytania `ping` do innego hosta w
sieci. Poniżej przykład:

![ping-hostname-domain-name-openwrt](/img/2016/04/3.ping-hostname-domain-name-openwrt.png#big)

Bez `dnsmasq` trzeba by konfigurować hosty przez dodawanie wpisów w pliku `/etc/hosts` . Obecnie
jesteśmy w stanie wysyłać żądania na trzy sposoby. Pierwszy przez podanie samego hostname
( `openwrt` ). Drugi przez pełną nazwę domenową ( `openwrt.lan` ). Trzeci przez adres IP
( `192.168.1.1` ).

## Klient udhcpc nie przesyła hostname do serwera DHCP

Klient DHCP obecny w OpenWRT nie przesyła standardowo nazwy domeny w zapytaniach o konfigurację do
serwera DHCP naszego ISP. Czasem może to nie być pożądane zachowanie. Jeśli chcemy, aby nasz router
przesyłał swoją nazwę hosta do serwera DHCP, to możemy nakazać `udhcpc` , aby tak zrobił. W tym celu
edytujemy plik `/etc/config/network` i w sekcji z interfejsem WAN dodajemy tę poniższą linijkę:

    config interface 'wan'
    ...
            option hostname 'the-mountain'
    ...
