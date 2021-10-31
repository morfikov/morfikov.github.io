---
author: Morfik
categories:
- OpenWRT
date: "2016-04-21T19:41:05Z"
date_gmt: 2016-04-21 17:41:05 +0200
published: true
status: publish
tags:
- chaos-calmer
- sysupgrade
- router
- tp-link
GHissueID: 414
title: Jak powrócić z firmware OpenWRT do TP-LINK'a
---

Zmiana oryginalnego firmware na alternatywne na bazie OpenWRT nie jest procesem tylko i wyłączenie w
jedną stronę. W przypadku, gdy OpenWRT z jakichś powodów nie spełnia naszych oczekiwań, to zawsze
możemy powrócić do oprogramowania oferowanego przez producenta routera. W tym wpisie prześledzimy
sobie proces powrotu do oryginalnego firmware na przykładzie routera [TP-LINK TL-WR1043ND
v2](http://www.tp-link.com.pl/products/details/TL-WR1043ND.html).

<!--more-->
## Pozyskiwanie obrazu "bez_boot" (stripped)

W przypadku, gdy chcemy powrócić do firmware producenta naszego routera, to musimy zaopatrzyć się w
odpowiedni plik. Nie możemy jednak tak po prostu wejść na stronę TP-LINK'a i ściągnąć najnowszą
wersją oprogramowania do routera TL-WR1043ND v2. Taki obraz ma zwykle na początku dołączoną sekcję
bootloader'a. W przypadku, gdy aktualizujemy oryginalny soft na routerze, ta sekcja nie ma większego
znaczenia, bo to oprogramowanie potrafi sobie z nią poradzić. W naszym przypadku, jeśli wgralibyśmy
taki obraz "z boot", to kod odpowiedzialny za sekcję bootloader'a powędruje do pamięci w miejsce,
gdzie powinien znaleźć się kod samego firmware. W efekcie po wgraniu takiego pliku na router, ten
zwyczajnie nie będzie nam już się chciał uruchomić.

Mamy zatem do wyboru dwie opcje. Pierwsza z nich zakłada pobranie gotowego obrazu "bez_boot". Z
tym, że nie musi to być najnowsza wersja obrazu. Nawet jeśli w łapki trafi nam się bardzo stary
obraz, to nic nie stoi na przeszkodzie, aby go sobie zaktualizować pobierając tym razem już
najnowszy plik z oficjalnej strony TP-LINK'a. Drugi sposób polega na pobraniu oryginalnego firmware
i obcięciu z niego kodu bootloader'a. Tak uzyskany plik można zaaplikować routerowi.

Obrazy do szeregu modeli niezwierające bootloader'a można znaleźć w kilku miejscach. Są one dostępne
min. na [tplink-forum.pl](ftp://tplink-forum.pl/orgin_bez_boot/),
[dl.eko.one.pl](http://dl.eko.one.pl/orig/) oraz na
[friedzombie.com](http://www.friedzombie.com/tplink-stripped-firmware/). Tutaj posłużymy się
obrazami uzyskanymi z pierwszego linku. Logujemy się zatem na router i pobieramy przy pomocy `wget`
plik `wr1043v2_en_3_15_31_up(130925).bin` :

    $ ssh root@192.168.1.1

    # cd /tmp/

    # wget "ftp://tplink-forum.pl/orgin_bez_boot/wr1043v2_en_3_15_31_up(130925).bin" -O obraz.bin
    Connecting to tplink-forum.pl (188.128.181.111:21)
    obraz.bin            100% |********************|  7936k  0:00:00 ETA

## Przywracanie oryginalnego firmware

Możemy teraz przejść do następnego kroku, czyli do flash'owania routera. Proces powrotu do
oryginalnego firmware jest praktycznie automatyczny i sprowadza się do zaprzęgnięcia narzędzia
`sysupgrade` . Polecenie, które musimy wydać w terminalu jest następujące:

    # sysupgrade /tmp/obraz.bin

Po chwili powinniśmy na ekranie zobaczyć komunikat `Upgrade completed, Rebooting system...` , a
router powinien się automatycznie zrestartować. Posiadacze linux'ów przy odpowiedniej konfiguracji
`iptables` mogą zaobserwować w logu ten poniższy
    komunikat:

    kernel: [ 4089.784250] * IPTABLES: UDP * IN=bond0 OUT= MAC=ff:ff:ff:ff:ff:ff:e8:94:f6:68:79:f0:08:00
    SRC=192.168.0.1 DST=255.255.255.255 LEN=36 TOS=0x00 PREC=0x00 TTL=64 ID=0 DF PROTO=UDP SPT=42760 DPT=7437 LEN=16

Wyżej widzimy adres IP `192.168.0.1` , z którego został wysłany ten zalogowany na zaporze pakiet.
Zatem router żyje i działa, a jego interfejs webowy jest dostępny pod tym powyższym adresem.
Niemniej jednak, jest to inna sieć. Dla OpenWRT to było 192.168.1.0/24 . W przypadku oryginalnego
firmware mamy 192.168.0.0/24 . By móc przejść do panelu administracyjnego z poziomu przeglądarki
internetowej, musimy zresetować połączenie sieciowe na komputerze. Później już tylko wpisujemy w
pasku adresu `http://192.168.0.1/` i naszym oczom powinien ukazać się formularz logowania. Domyślny
użytkownik to `admin` , hasło również `admin` . Logujemy się i konfigurujemy router.

![tp-link-router-openwrt-firmware](/img/2016/04/1.tp-link-router-openwrt-firmware.png#huge)

## Aktualizacja oryginalnego firmware

Kluczową sprawą po powrocie do firmware producenta jest aktualizacja oprogramowania routera.
Przechodzimy zatem na [stronę TP-LINK'a](http://www.tp-link.com.pl/download-center.html) w celu
pobrania firmware przeznaczonego dla naszego routera. Na początek wybieramy odpowiednią wersję
routera:

![tp-link-router-openwrt-firmware](/img/2016/04/2.tp-link-router-openwrt-firmware.png#huge)

Następnie przewijamy stronę i na dole powinien znajdować się link do firmware:

![tp-link-router-openwrt-firmware](/img/2016/04/3.tp-link-router-openwrt-firmware.png#huge)

Pobieramy plik i wypakowujemy zawartość. Na dysku powinniśmy mieć plik
`wr1043v2_en_3_19_32_up_boot(150910).bin` . Wracamy teraz do panelu administracyjnego routera i
przechodzimy kolejno do System Tools -> Firmware Upgrade. Wskazujemy plik i klikamy Upgrade:

![tp-link-router-openwrt-firmware](/img/2016/04/4.tp-link-router-openwrt-firmware.png#huge)

Po chwili firmware powinien zostać aktualizowany do najnowszej wersji dostępnej na stronie
TP-LINK'a:

![tp-link-router-openwrt-firmware](/img/2016/04/5.tp-link-router-openwrt-firmware.png#huge)
