---
author: Morfik
categories:
- Linux
- Hardware
date:    2021-01-27 19:10:00 +0100
lastmod: 2021-01-27 19:10:00 +0100
published: true
status: publish
tags:
- drukarka
- debian
- hewlett-packard
- laserjet
- p2055dn
- firmware
- kvm
- qemu
- wirtualizacja
- libvirt
GHissueID: 337
title: Aktualizacja firmware drukarki HP LaserJet P2055dn pod linux
---

Bawiąc się ostatnio drukarką laserową Hewlett Packard (HP) LaserJet P2055dn, zauważyłem, że ma ona
wgrany dość stary firmware. Naturalnie sama drukarka do najmłodszych nie należy, bo została
wyprodukowana w 2011 roku ale skoro na stronie producenta jest dostępna nowsza wersja
oprogramowania dla tego urządzenia, to przydałoby się je do tej drukarki wgrać. Problem jednak
pojawia się w przypadku takich osób jak ja, tj. tych, które korzystają w swoim środowisku pracy z
maszyn mających na pokładzie system operacyjny z rodziny jakieś dystrybucji linux'a, np. Debian czy
Ubuntu. Producent drukarki udostępnia stosowne narzędzia do aktualizacji firmware ale tylko i
wyłącznie dla OS z gatunku Windows. Co mają zrobić osoby, które z Windows'a nie korzystają, a
chciałyby przy tym mieć aktualny firmware w drukarkach HP? Możemy spróbować postawić maszynę
wirtualną na bazie QEMU/KVM, tam zainstalować Windows'a i udostępnić w obrębie tej maszyny
wirtualnej drukarkę, której firmware mamy zamiar aktualizować.

<!--more-->
## Narzędzia do aktualizacji firmware tylko dla Windows 2000, XP, Vista i Win7

Lista plików, które można pobrać [ze strony producenta drukarki][2], różni się w zależności od
wybranego systemu operacyjnego. Narzędzia do aktualizacji firmware znajdują się w zasadzie tylko na
zakładce z Windows 2000, XP, Vista oraz Win7 i każda z tych wersji straciła już chyba oficjalne
wsparcie MS. W tym przypadku będzie wykorzystywany Windows 7, dlatego też trzeba będzie się na tę
zakładkę przełączyć, by pobrać stosowny plik. Interesuje nas [HP LaserJet Firmware Update
Utility][3], który jest nam w stanie zaserwować najnowszy firmware z dnia 2014-12-01 (wypuszczony
dnia 2015-01-22). Niestety w panelu webowym drukarki nie ma informacji, z którego dnia jest
aktualnie zainstalowany w niej firmware (tam gdzie powinna być data jest tylko XX). Jest za to
podana wersja oprogramowania, która wskazuje na `V.37.12.IR6498` .

Niestety nie mam u siebie żadnej maszyny, która by posiadała zainstalowany jeden z tych wspieranych
przez HP systemów operacyjnych i muszę posiłkować się maszynami wirtualnymi na bazie QEMU/KVM.

## Instalacja i konfiguracja maszyny wirtualnej QEMU/KVM

Kilka miesięcy temu opisywałem w miarę dokładnie [jak skonfigurować Debiana do pracy z maszynami
wirtualnymi QEMU/KVM][1] i nie będę tutaj poruszał całego tego zagadnienia, tylko te ważniejsze
rzeczy umożliwiające w miarę natychmiastowe i w pełni automatyczne postawienie interesującej nas
maszyny wirtualnej przy minimalnej konfiguracji ze strony użytkownika. Jeśli kogoś interesuje temat
samych maszyn wirtualnych, to zapraszam pod wskazany wyżej artykuł. Niżej zakładam zaś, że całe
oprogramowanie potrzebne do uruchomienia maszyn wirtualnych mamy już zainstalowane oraz
skonfigurowane do pracy.

Po przygotowaniu naszego linux'a do pracy z maszynami wirtualnymi QEMU/KVM, musimy pozyskać
stosowny obraz instalacyjny z Windows 7. Technicznie rzecz biorąc, można taki obraz [pobrać
legalnie nawet ze strony Microsoft][4]. Wystarczy jedynie podać numer seryjny produktu. Jako, że
praktycznie każdy laptop ma na obudowie taki klucz, to bez większego problemu powinniśmy być w
stanie nośnik instalacyjny z Windows'em pozyskać. W moim przypadku najwyraźniej się tego zrobić nie
da, bo po wpisaniu klucza dostaję taki oto poniższy komunikat:

> Błąd
>
> Wprowadzony przez Ciebie klucz produktu dotyczy prawdopodobnie oprogramowania preinstalowanego
> przez producenta urządzenia. Skontaktuj się z producentem urządzenia, aby poznać opcje
> odzyskiwania oprogramowania.

Czyli, niby zapłaciłem za Windows 7 (a raczej zrobił to ten, od kogo tego laptopa kupiłem) ale
obrazu instalacyjnego nie mogę pobrać, bo najwyraźniej jeszcze mógłbym na tym komputerze tego
windows'a zainstalować.

Oczywiście, nie jest to jakiś problem nie do przejścia i postanowiłem poszukać obrazów z Windows 7,
które mógłbym wykorzystać do stworzenia maszyny wirtualnej. Stosowny obraz znalazłem w [serwisie
winclub.pl][5], tj. plik `Win7-BN-x64-7fd8fc64.zip` ważący 3,5GiB (po wypakowaniu 3,6 GiB). Ten
wypakowany obraz ISO trzeba wskazać jako nośnik instalacyjny przy tworzeniu maszyny wirtualnej, np.
w `virt-manager` :

![](/img/2021/01/001-drukarka-hp-laserjet-p2055dn-linux-kvm-qemu.png#big)

Najwyraźniej nie działa autodetekcja systemu operacyjnego obecnego na nośniku i trzeba ręcznie
określić, że ma to być Windows 7:

![](/img/2021/01/002-drukarka-hp-laserjet-p2055dn-linux-kvm-qemu.png#medium)

Przydzielamy domyślne wartości dla pamięci RAM, procesora i dysku twardego oraz określamy przydział
sieci dla systemu gościa. Następnie rozpoczynamy proces instalacyjny systemu Windows 7 wewnątrz
maszyny wirtualnej:

![](/img/2021/01/003-drukarka-hp-laserjet-p2055dn-linux-kvm-qemu-instalacja-windows.png#huge)

Czekamy naturalnie aż proces się zakończy, po czym uruchamiamy ponownie maszynę wirtualną.

### Aktualizacja firmware po USB

Teoretycznie nie ma to większego znaczenia czy firmware będziemy aktualizować po uprzednim
podpięciu drukarki pod port USB komputera, czy też gdy z drukarką będziemy komunikować się za
pomocą sieci lokalnej. W przypadku wyboru opcji pierwszej, tj. via port USB, trzeba będzie stosowne
urządzenie USB udostępnić maszynie wirtualnej:

![](/img/2021/01/004-drukarka-hp-laserjet-p2055dn-linux-kvm-qemu-usb.png#huge)

Drukarka naturalnie powinna zostać wykryta przez system Windows'a. Czasami trzeba ponownie
uruchomić maszynę wirtualną by wszystko zadziałało. Tak czy inaczej, pozycja od naszej drukarki HP
powinna figurować na liście drukarek i faksów:

![](/img/2021/01/005-drukarka-hp-laserjet-p2055dn-linux-windows-instalacja-drukarki.png#huge)

Co ciekawe, numerek tej drukarki ździebko się różni od faktycznego ale chyba to na nic nie wpływa.
Dla pewności, że wszystko działa jak należy, możemy jeszcze wydrukować stronę testową:

![](/img/2021/01/006-drukarka-hp-laserjet-p2055dn-linux-windows-instalacja-drukarki.png#huge)

Teoretycznie aktualizacja firmware via port USB powinna przebiec bez większych problemów. W moim
przypadku tak się nie stało. Nie wiem czy jest tutaj winna sama maszyna wirtualna, czy jakieś inne
czynniki odgrywają jeszcze rolę ale przy próbie tak podłączonej drukarki proces aktualizacji
firmware nie startuje.

### Aktualizacja firmware po sieci

Jako, że nie szło zaktualizować firmware drukarki HP LaserJet P2055dn przy podłączeniu jej via port
USB, to postanowiłem sprawdzić jak sprawa będzie wyglądać, gdy spróbuję skonfigurować na Windows
drukarkę sieciową. Oczywiście w tym przypadku nie trzeba dodatkowo konfigurować żądnych urządzeń na
maszynie wirtualnej. Wystarczy, że będzie tam poprawnie funkcjonować sieć. Windows powinien bez
większego trudu wykryć drukarkę sieciową i poprawnie ją zainstalować w systemie.

Gdy mamy już dodaną drukarkę do systemu, pobieramy na maszynę wirtualną plik [HP LaserJet Firmware
Update Utility][3] (jeśli jeszcze tego nie uczyniliśmy):

![](/img/2021/01/007-drukarka-hp-laserjet-p2055dn-linux-windows-oprogramowanie.png#huge)
![](/img/2021/01/008-drukarka-hp-laserjet-p2055dn-linux-windows-oprogramowanie.png#huge)

Po pobraniu pliku, uruchamiamy go. Powinno nam się pokazać poniższe okienko:

![](/img/2021/01/009-drukarka-hp-laserjet-p2055dn-linux-aktualizacja-firmware.png#medium)

Upewniamy się, że w górnej części okna figuruje nasza drukarka i klikamy `Send Firmware` . Po
chwili firmware drukarki powinien zostać zaktualizowany:

![](/img/2021/01/010-drukarka-hp-laserjet-p2055dn-linux-aktualizacja-firmware.png#huge)

Jeśli w trakcie tego procesu pasek postępu się zawiesi na dłużej, to trzeba trochę poczekać. U mnie
po około 1-2 minutach proces został wznowiony i ostatecznie zakończył się powodzeniem.

## Sprawdzenie wersji nowego firmware drukarki

Po pomyślnej aktualizacji firmware drukarki, uruchamiamy ją ponownie i wchodzimy w jej webowy panel
administracyjny. Przechodzimy na stronę podsumowania konfiguracji. Powinniśmy tam mieć m.in.
poniższą zwrotkę:

    ---------- Informacje ogólne -----------
    Stan:                   Karta I/O gotowa

    Numer modelu:                     J8017E
    Adres sprzętowy:            984BE13B567C
    Wersja oprogr firm.:          V.37.18.SD
    Konfiguracja portu:           1000T FULL
    Negocjacja automatyczna:        Włączone
    Nr wyrobu:                ********01****
    Build Date:          11/13/2014 11:18:10

Wcześniej mieliśmy wersję `V.37.12.IR6498` , a obecnie mamy `V.37.18.SD` . Co ciekawe, po wgraniu
tego firmware, mamy również określoną datę jego budowy, która nieco różni się od tej deklarowanej
przez producenta drukarki ale grunt, że coś nowszego zostało wgrane do tej drukarki.


[1]: /post/wirtualizacja-qemu-kvm-libvirt-na-debian-linux/
[2]: https://support.hp.com/us-en/drivers/selfservice/HP-LaserJet-P2055-Printer-series/3662052/model/3662058
[3]: https://support.hp.com/us-en/drivers/selfservice/swdetails/HP-LaserJet-P2055-Printer-series/3662052/model/3662058/swItemId/lj-66014-12
[4]: https://www.microsoft.com/pl-pl/software-download/windows7
[5]: https://winclub.pl/topic/19862-win-7-x64-pro-2k19/
