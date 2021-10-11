---
author: Morfik
categories:
- Android
- Linux
date:    2016-10-08 20:34:36 +0200
lastmod: 2016-10-08 20:34:36 +0200
published: true
status: publish
tags:
- smartfon
- adb
- fastboot
- debian
GHissueID: 440
title: 'Android: Jak zainstalować ADB i fastboot pod linux'
---

Bawiąc się [smartfonem Neffos C5][1] od TP-LINK chciałem sprawdzić czy da radę zainstalowanemu na
nim Androidowi 5.1 (Lollipop) zrobić root'a. Chodzi o uzyskanie dostępu administracyjnego do systemu
plików na flash'u telefonu. Proces się powiódł, z tym, że by do niego przystąpić, potrzebne są
narzędzia takie jak `adb` i `fastboot` , bo przy ich pomocy możemy sterować telefonem, np. z poziomu
jakiegoś linux'a. Niemniej jednak, system komputera jak i smartfona trzeba pierw przygotować
odpowiednio, by taka komunikacja była możliwa i ten proces zostanie właśnie opisany poniżej.

<!--more-->
## Androidowe narzędzia pod linux

Na linux'ie nie musimy instalować całego zestawu narzędzi developerskich dla Androida (Software
Development Kit). Bardziej zaawansowani programiści potrzebują pełnego pakietu, bo to oprogramowanie
jest raczej im niezbędne do pracy z systemem Android. Nas interesują głównie dwa narzędzia: [Android
Debug Bridge (adb)][2] oraz [fastboot][3].

Fastboot jest protokółem diagnostycznym, który jest wykorzystywany zwykle do wprowadzania zmian w
firmware telefonu (ROM), np. aktualizacje. Podczas przełączania Androida w tryb recovery, mamy na
liście pozycję fastboot mode, zatem Neffos C5 obsługuje ten protokół.

Informacje na temat aktywowania tego protokołu są w dalszej części artykułu. Niemniej jednak, nie
wszystkie smartfony mają aktywny tryb fastboot i trzeba mieć to na uwadze przy ewentualnej próbie
skomunikowania komputera z telefonem.

Za obsługę `adb` w Debianie odpowiada pakiet `android-tools-adb` . Z kolei jeśli chodzi zaś o
fastboot, to musimy doinstalować w systemie pakiet `android-tools-fastboot` .

## Android Debug Bridge (ADB)

W zależności od wersji Androida, którą mamy zainstalowaną w telefonie, musimy się upewnić, że
posiadamy odpowiednią wersję narzędzia `adb` . Dla Androida 5.1 (Lollipop) musimy dysponować co
najmniej wersją 1.0.32. Jeśli nie wiemy, którą wersję `adb` mamy zainstalowaną w systemie, to zawsze
możemy to sprawdzić wpisując w terminalu poniższe polecenie:

    $ adb version
    Android Debug Bridge version 1.0.32

Na smartfonie musimy włączyć z kolei `Debugowanie USB` . Możemy to uczynić w chodząc w ustawienia,
gdzie na samym dole mamy pozycję `Informacje o telefonie` . Tam z kolei (również na dole) mamy
informację dotyczącą numeru kompilacji:

![](/img/2016/10/1.adb-informacje-telefon-smartfon-linux.png#medium)

Musimy stuknąć w ten numerek siedem razy. Dopiero wtedy w ustawieniach telefonu pojawi się pozycja
`Opcje programistyczne` , w których to będziemy mogli zaznaczyć szukany tryb debugowania USB:

![](/img/2016/10/2.adb-telefon-smartfon-debug-usb-linux.png#huge)

Teraz możemy podłączyć telefon do komputera i sprawdzić czy zostanie on rozpoznany przez naszego
linux'a. Mój Neffos C5 z początku nie został rozpoznany:

    # adb devices

    * daemon not running. starting it now on port 5037 *
    * daemon started successfully *
    List of devices attached

Okazuje się, że [różne rzeczy mogą mieć wpływ na to, czy telefon zostanie wykryty][4]. Przede
wszystkim, upewnijmy się, że ekran telefonu nie jest zablokowany. Być może przełączenie trybu USB w
telefonie pomoże. Chodzi o przełączenie między protokołem MTP/PTP. Warto też odłączyć i ponownie
podłączyć telefon do portu USB komputera, jak i ubić demona `adb` przez wydanie w terminalu
poniższego polecenia :

    # adb kill-server

Jeśli chodzi zaś o mojego Neffos'a C5, to żadne z tych powyższych czynności nie pomogły. System był
w stanie go wykryć dopiero po dodaniu do pliku `~/.android/adb_usb.ini` numeru widocznego w
`idVendor=2357` (domyślnie ów plik nie istnieje). Ten numer można odczytać z logu systemowego po
podłączeniu telefonu do portu USB, lub też z wyjścia polecenia `lsusb` . Mając numer, dodajemy go
do ww. pliku jako:

    0x2357

Od tej chwili, za każdym razem jak tylko podłączymy telefon w trybie debugowania USB do komputera za
pomocą przewodu, te dwa urządzenia będą parowane automatycznie. Oczywiście za pierwszym razem na
telefonie musimy dodać informację o kluczu publicznym, którym identyfikuje się komputer. Klucz jest
zaś w katalogu `~/.android/` :

![](/img/2016/10/3.adb-telefon-smartfon-parowanie-linux.png#medium)

Po zweryfikowaniu klucza publicznego komputera, możemy ponownie wpisać w terminalu polecenie `adb
devices` i tym razem już powinniśmy ujrzeć na liście nasz smartfon:

    # adb devices

    List of devices attached
    TSL7DA69OBSO49PJ        device

## Fastboot

Jeśli telefon ma odblokowany tryb fastboot, a tak jest w przypadku tego Neffos'a C5 od TP-LINK, to
możemy odpalić go zwyczajnie w tym trybie. W tym celu musimy pierw wyłączyć telefon. Następnie
przyciskamy i trzymamy przycisk Volume Up oraz przyciskamy równocześnie przycisk Power. Trzymamy oba
przyciski wciśnięte do momentu aż telefon się włączy. Na ekranie powinniśmy ujrzeć fastboot mode,
który musimy wybrać zaznaczając go przyciskiem Volume Up i aktywując przyciskiem Volume Down.

W tej chwili tryb fastboot działa, a my możemy podłączyć telefon do komputera. Sprawdźmy jeszcze czy
nasz linux jest w stanie zobaczyć tego smartfona. Odpalamy terminal i wpisujemy w nim poniższe
polecenie:

    # fastboot devices
    TSL7DA69OBSO49PJ        fastboot


[1]: http://www.neffos.pl/product/details/C5
[2]: https://developer.android.com/studio/command-line/adb.html
[3]: https://wiki.cyanogenmod.org/w/Doc:_fastboot_intro
[4]: https://wiki.cyanogenmod.org/w/Doc:_adb_intro
