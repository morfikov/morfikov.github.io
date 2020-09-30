---
author: Morfik
categories:
- OpenWRT
date: "2017-08-17T21:22:08Z"
published: true
status: publish
tags:
- chaos-calmer
- router
- tp-link
title: Jak ustalić IP i nazwę pliku trybu recovery w routerach TP-Link
---

Jeden z moich routerów, a konkretnie był
to [Archer C7](http://www.tp-link.com/us/download/Archer-C7.html) v2 wymagał, by powrócić jego
firmware z LEDE/OpenWRT do tego, który widnieje na oficjalnej stronie TP-Link. Niby ta czynność nie
jest zbyt skomplikowana ale jak zwykle coś poszło nie tak. Konkretnie to odłączyłem zasilanie nie w
tej listwie co trzeba i w efekcie podczas flash'owania routera nowym firmware, to urządzenie się
zwyczajnie wyłączyło. Zawału oczywiście nie dostałem, bo przecież obraz, który był wgrywany na
router nie zawierał uboot'a, czyli części z bootloader'em, więc wiedziałem, że wystarczy przez tryb
recovery wgrać obraz jeszcze raz i po sprawie. Problem w tylko w tym, że nie znałem w zasadzie ani
nazwy pliku obrazu, ani też adresu IP, który jest wymagany dla połączenia w przypadku routera
Archer C7 v2. Te dane można naturalnie znaleźć w sieci ale co w przypadku, gdy ubijemy sobie w taki
sposób nasz jedyny router, przez co pozbawimy się jednocześnie dostępu do internetu? Czy istnieje
jakiś sposób na ustalenie tych danych, inny niż przez konsolę szeregową?

<!--more-->
### Ustalenie adresu IP oraz nazwy pliku przy pomocy wireshark'a

Po uruchomieniu routera w trybie recovery (zakładając, że nasz sprzęt go posiada), takie urządzenie
zacznie wysyłać pakiety sieciowe z pewnymi komunikatami. Nasz router będzie zatem pierw próbował za
pomocą protokołu ARP znaleźć w sieci maszynę o określonym adresie IP. Jeśli takiego urządzenia nie
ma w sieci, to naturalnie nie znajdzie żądanego adresu IP ale taki komunikat da nam informację na
temat adresu, który powinniśmy ustawić na swoim komputerze.

Gdy już skonfigurujemy adresację sieciową dla komputera, na którym zostanie odpalony serwer TFTP,
router uruchomiony w trybie recovery połączy się z naszą maszyną i przy pomocy protokołu TFTP
będzie próbował z nią rozmawiać i zażąda od serwera plik z firmware, który zamierza pobrać. To
żądanie zawierać będzie nazwę pliku i w ten sposób będziemy wiedzieć jak nazwać plik firmware.
Poniżej przykład:

![](/img/2017/08/001.tp-link-recovery-ip-nazwa-pliku-wireshark.png#huge)

Jak widać wyżej na fotce, router Archer C7 v2 szukał maszyny z adresem IP `192.168.0.66` , sam zaś
miał IP `196.168.0.86` . Jako, że mamy w zasadzie tylko jedno zapytanie ARP oraz jedną odpowiedź,
to wiemy, że router znalazł z powodzeniem maszynę, której szukał. Na tym komputerze był odpalony
serwer TFTP ale nie było tam pliku firmware. Wyżej widać, że router poprosił o plik
`ArcherC7v2_tp_recovery.bin` ale, że go nie znalazł, to po trzykrotnej próbie dał sobie spokój.

Znając adres IP oraz nazwę pliku firmware, teraz bez większego problemu możemy skonfigurować sobie
parametry dla trybu recovery praktycznie dowolnego routera TP-Link, a jeśli te dane będą poprawne,
to rozpocznie się pobieranie obrazu i flash'owanie firmware:

![](/img/2017/08/002.tp-link-recovery-ip-nazwa-pliku-wireshark.png#huge)

