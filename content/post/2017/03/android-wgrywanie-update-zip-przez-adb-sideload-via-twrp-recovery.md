---
author: Morfik
categories:
- Android
date:    2017-03-09 18:31:55 +0100
lastmod: 2017-03-09 18:31:55 +0100
published: true
status: publish
tags:
- smartfon
- twrp
- adb
GHissueID: 54
title: 'Android: Wgrywanie update.zip przez ADB sideload via TWRP recovery'
---

Po uszkodzeniu jednego z moich smartfonów TP-LINK i skasowaniu na nim wszystkich danych na partycji
`/system/` trzeba było pomyśleć nad przywróceniem tego urządzenia do życia. Jednym z rozwiązać było
[binarne wgranie obrazu systemowej partycji bezpośrednio na flash przy pomocy narzędzia dd][1]. Co
jednak w przypadku, gdy nie mamy dostępu do backup'u lub tez zwyczajnie go nie zrobiliśmy? Co w
takiej sytuacji uczynić i czy jest jakaś nadzieja dla naszego telefonu? Odpowiedź jest naturalnie
twierdząca ale wymagane są dwie rzeczy: działający tryb recovery (najlepiej TWRP) ze wsparciem dla
trybu "ADB sideload" oraz paczka `update.zip` z firmware, którą można pobrać bezpośrednio ze strony
TP-LINK/Neffos. By ulżyć nieco osobom, które do mnie piszą z zapytaniem o pomoc w przypadku
skasowania danych na partycji `/system/` (czy uszkodzenia jej w jakiś sposób), postanowiłem napisać
krótkie howto na temat używania trybu ADB sideload. W tym artykule w rolach głównych weźmie udział
[Neffos Y5][2] ale bez problemu można te kroki przeprowadzić chyba na każdym innym smartfonie.

<!--more-->
## Objawy usunięcia danych z partycji /system/

Usuwając dane z partycji `/system/` pozbawiamy nasz telefon praktycznie całego oprogramowania. Taki
smartfon nie może działa bez Androida czy innego systemu operacyjnego i w zasadzie urządzenie
resetuje się co około minutę po włączeniu.

![](/img/2017/03/001.adb-sideload-update-zip-firmware-tp-link-neffos-twrp-usuniety-system.png#medium)

W takim stanie po włączeniu smartfona, na ekranie można zobaczyć jedynie loga TP-LINK i Androida.
Natomiast w logu systemowym mojego Debiana można zaobserwować poniższe komunikaty:

    kernel: usb 2-1.1: new high-speed USB device number 7 using ehci-pci
    kernel: usb 2-1.1: New USB device found, idVendor=2357, idProduct=0328
    kernel: usb 2-1.1: New USB device strings: Mfr=1, Product=2, SerialNumber=3
    kernel: usb 2-1.1: Product: Android
    kernel: usb 2-1.1: Manufacturer: Android
    kernel: usb 2-1.1: SerialNumber: 90169635
    kernel: usb 2-1.1: USB disconnect, device number 6

Gdy dane z partycji `/system/` zostały usunięte, to nie damy rady wgrać nic na smartfon przy pomocy
narzędzia `fastboot` . Dlatego właśnie wymagany jest działający tryb recovery. Jeśli ten również z
jakiegoś powodu nie działa, to niestety uwaliliśmy telefon na dobre i trzeba będzie go odesłać do
serwisu.

## Działający TWRP recovery z trybem ADB sideload

Dla osób, które nie wiedzą czym jest [ADB sideload][3], wyjaśniam, że jest jeden z trybów pracy ADB.
Standardowo przy pomocy `adb` możemy rozmawiać z systemem smartfona przez wysyłanie do niego różnych
poleceń i w zasadzie możemy operować na takim urządzeniu mniej więcej tak jakbyśmy działali na
zwyczajnym linux'ie. W przypadku trybu ADB sideload, te standardowe polecenia są nieaktywne ale
mamy za to dostęp do komendy `adb sideload` , która jest w stanie wgrać świeży firmware z paczki
`update.zip` .

TWRP recovery począwszy od wersji 2.3 wspiera tryb ADB sideload. Dlatego też trzeba się upewnić, że
mamy wgraną na smartfonie w miarę nową wersję tego oprogramowania. [Gotowe obrazy TWRP recovery dla
smartfonów Neffos][4] są dostępne w tym wątku.

W przypadku, gdy mamy starszą wersję TWRP (< 2.3), no to niestety musimy zaktualizować to
oprogramowanie wgrywając binarnie obraz przy pomocy `dd` mniej więcej w taki sam sposób jak zostało
to opisane przy okazji odzyskiwania partycji `/system/` (link we wstępie).

## ADB sideload i stock'owy recovery

Nie jestem pewien czy ADB sideload działa na stock'owym recovery. Wiem, że smartfony oferują
możliwość wgrywania pliku `update.zip` z trybu recovery ale nie wiem jak się zachowa system, gdy
jedna lub kilka partycji ( `/boot/` , `/system/` , `/recovery/` ) zostały w jakiś sposób zmienione.
Dlatego to HOWTO dotyczy jedynie TWRP recovery, na którym pomyślnie udało mi się przetestować ADB
sideload.

## Pozyskanie pliku update.zip zawierającego firmware smartfona

ADB sideload potrzebuje pliku z obrazem firmware (ROM). Stock'owy ROM można pobrać ze strony
TP-LINK/Neffos. Linki do działu download: [Neffos Y5][5], [Neffos Y5L][6], [Neffos C5][7], [Neffos
C5 MAX][8]. Pliki są w miarę duże i ważą około 1 GiB. Każdy z tych plików w nazwie zawiera
oznaczenie modelu, np. `C5_Max_TP702A` , `Y5L_TP801` . Tutaj mamy `TP702` i `TP801` oraz w
przypadku tego pierwszego mamy również literkę `A` , która odpowiada za wersję geograficzną i w tym
przypadku `A` jest dla smartfonów na rynku europejskim.

## Jak wgrać update.zip przez ADB sideload z poziomu TWRP recovery

Pobrany firmware jest w postaci spakowanego pliku `.zip` . Tego pliku nie wypakowujemy. Zostanie on
załadowany do pamięci RAM komputera, a jego zawartość w locie przesłana na smartfon bez potrzeby
wgrywania tego pliku na flash urządzenia. Ta paczka zawiera nie tylko oprogramowanie, które jest
wgrywane na partycję `/system/` ale również obraz partycji `/boot/` , za sprawą którego zostanie
wygenerowany też świeży obraz partycji `/recovery/` . Wszystkie te trzy partycje będą poddane
procesowi flash'owania i po jego ukończeniu powinniśmy powrócić do stock'owego firmware.

Pliki `update.zip` są podpisane cyfrowo i domyślnie nie damy rady ich wgrać przez TWRP recovery, bo
zostanie nam wygenerowany poniższy błąd:

    sideload-host file size 968174348 block size 65536
    Installing zip file '/sideload/package.zip'
    Verifying zip signature...
    I:read key e=3 hash=20
    I:1 key(s) loaded from /res/keys
    I:comment is 1428 bytes; signature 1410 bytes from end
    I:TWFunc::Set_Brightness: Setting brightness control to 5
    I:TWFunc::Set_Brightness: Setting brightness control to 0
    I:failed to verify against RSA key 0
    E:failed to verify whole-file signature
    I:Zip signature verification failed: 1
    Zip signature verification failed!
    I:Signaling child sideload process to exit.
    I:Waiting for child sideload process to exit.
    sideload_host finished

Problem jak widać dotyczy weryfikacji sygnatury pliku `.zip` . Musimy zatem poinstruować TWRP by nie
weryfikował sygnatury. Możemy to zrobić odhaczając opcję `ZIP signature verification` w
ustawieniach:

![](/img/2017/03/002.adb-sideload-update-zip-firmware-tp-link-neffos-twrp-opcje.png#medium)

Teraz możemy aktywować tryb ADB sideload przechodząc w Advanced => ADB Sideload:

![](/img/2017/03/003.adb-sideload-update-zip-firmware-tp-link-neffos-twrp-menu.png#big)

Czyszczenie cache jest opcjonalne ale można je zaznaczyć. Dane użytkownika i tak pozostaną
nietknięte, zatem bez obaw. Niemniej jednak, trzeba pamiętać, że w przypadku modyfikacji partycji
`/system/` , np. przez Xposed, możemy napotkać dziwne problemy, które mogą uniemożliwić start lub
też poprawne działanie systemu i wymagane będzie przeprowadzenie procesu Factory Reset.

![](/img/2017/03/004.adb-sideload-update-zip-firmware-tp-link-neffos-twrp-cache.png#medium)

Po przesunięciu trzech strzałek na prawą stronę, ADB przełączy się w tryb sideload. Teraz wracamy na
komputer i ładujemy plik `update.zip` przy pomocy poniższego polecenia ([trzeba doinstalować
narzędzie adb][9]):

    # adb sideload /neffos/Y5_H10S100D00B20161207R1344_update.zip
    serving: '/neffos/Y5_H10S100D00B20161207R1344_update.zip'  (~24%)

Proces flash'owania plikiem `update.zip` zajmie dłuższą chwilę ale ostatecznie powinien zakończyć
się powodzeniem:

![](/img/2017/03/005.adb-sideload-update-zip-firmware-tp-link-neffos-twrp-proces.png#medium)

Uruchamiamy smartfon ponownie i ignorujemy przy tym informację, że to urządzenie nie mam
zainstalowanego systemu operacyjnego (w zasadzie właśnie go zainstalowaliśmy):

![](/img/2017/03/006.adb-sideload-update-zip-firmware-tp-link-neffos-twrp-brak-os.png#medium)

Po chwili powinien nam się załadować ekran z wyborem języka systemu:

![](/img/2017/03/007.adb-sideload-update-zip-firmware-tp-link-neffos-twrp-system.png#medium)

Po skonfigurowaniu systemu warto sprawdzić czy są dostępne jakieś aktualizacje. To na wypadek,
gdybyśmy chcieli wgrać sobie jeszcze raz TWRP recovery czy przeprowadzać proces root Androida. W
przypadku wprowadzenia jakichkolwiek zmian na partycji `/boot/` , `/system/` lub `/recovery/` nie
będziemy w stanie tych aktualizacji wgrać na smartfon i warto o tym pamiętać.

![](/img/2017/03/008.adb-sideload-update-zip-firmware-tp-link-neffos-twrp-update.png#medium)


[1]: /post/android-jak-odratowac-smartfon-po-usunieciu-partycji-system/
[2]: http://www.neffos.pl/product/details/Y5
[3]: https://twrp.me/faq/ADBSideload.html
[4]: http://tplink-forum.pl/pub/neffos/
[5]: http://www.neffos.com/en/support/download/Y5
[6]: http://www.neffos.com/en/support/download/Y5L
[7]: http://www.neffos.com/en/support/download/C5
[8]: http://www.neffos.com/en/support/download/C5-Max
[9]: /post/android-jak-zainstalowac-adb-i-fastboot-pod-linux/
