---
author: Morfik
categories:
- OpenWRT
date: "2016-10-22T21:22:08Z"
date_gmt: 2016-10-22 19:22:08 +0200
published: true
status: publish
tags:
- chaos-calmer
- router
- tp-link
GHissueID: 450
title: Jak przy pomocy trybu recovery odzyskać router TP-LINK
---

Przy okazji zabawy z [konsolą szeregową przy ratowaniu jednego z moich routerów
TP-LINK](/post/konsola-szeregowa-adapter-usb-uart-uszkodzony-router-tp-link/)
([TL-WR1043ND](http://www.tp-link.com.pl/products/details/TL-WR1043ND.html) V2), parokrotnie
przewinęła mi się informacja na temat trybu recovery, który ma być dostępny w części routerów. W
czym nam taki tryb może pomóc i czy nasz router go obsługuje? Jeśli tak, to jak za jego pomocą
naprawić urządzenie, które nie chce wystartować, np. po przerwanym procesie wgrywania firmware
TP-LINK czy też OpenWRT/LEDE? W trym artykule postaramy się odpowiedzieć na te pytania.

<!--more-->
## Czym jest tryb recovery w routerach WiFi

Tryb recovery ma nam nieco ułatwić odzyskiwanie uszkodzonego routera bez potrzeby zaciągania do tego
celu konsoli szeregowej. Może i nasz router posiada wyprowadzenia portu szeregowego na swoim PCB ale
by się do niego podłączyć musimy zwykle przebrnąć przez proces lutowania pinów. Poza tym, musimy
także postarać się o zakup odpowiedniego adaptera USB-UART.

W przypadku trybu recovery nie musimy się bawić w lutowanie czy zakup dodatkowych narzędzi, które
pomogą nam odzyskać router. Cały proces odbywa się przez sieć, mniej więcej na takiej samej zasadzie
co przy wgrywaniu nowej wersji firmware. Oczywiście są drobne różnice ale grunt, że praktycznie ten
proces można przeprowadzić z wykorzystaniem przewodu ethernet (RJ-45), a to już ułatwia nam znacznie
robotę.

Trzeba jednak sobie zdawać sprawę, że ten recovery nie jest lekiem na każde zło. Jest to tryb
bootloader'a, a by móc z niego skorzystać, bootloader musi być sprawny. Jeśli z jakiegoś powodu
uszkodziliśmy bootloader, tryb recovery nam nie zadziała. Z kolei zaś, by ocenić czy bootloader
funkcjonuje poprawnie, możemy przyjrzeć się diodom na obudowie routera. Jeśli te migają, to jest
szansa, że bootloader jest sprawny i możemy spróbować naprawić router wgrywając firmware przez tryb
recovery.

## Jak sprawdzić czy router posiada tryb recovery

Trzeba także sobie zdawać sprawę, że nie wszystkie routery ten tryb recovery posiadają. To czy nasz
router WiFi ma taki tryb zależy od producenta sprzętu, który wypuścił (bądź też i nie) aktualizację
oficjalnego firmware zawierającego stosowną poprawkę dla bootloader'a. Dlatego też po ewentualnym
zakupie routera, dobrze jest wgrać najnowszy firmware ze strony TP-LINK. Jeśli zawiera on nowszą
wersję bootloader'a, to prawdopodobnie będziemy mieć dostęp do trybu recovery.

Jeśli nie mamy pewności czy nasz router posiada tryb recovery, to nic nie stoi na przeszkodzie, by
zbadać tę kwestię. Wyłączamy zatem urządzenie, po czym wciskamy i przytrzymujemy przycisk Reset.
Następnie włączamy router przyciskiem Power. Jeśli zaświeci nam się dioda WPS, to bootloader posiada
tryb recovery.

Ja korzystając z dobrodziejstw wyprowadzonego portu dla konsoli szeregowej mogę podejrzeć co się
dzieje na routerze po uruchomieniu go z wciśniętym przyciskiem reset. Poniżej znajduje się log
bootloader'a z mojego TL-WR1043ND V2:

![](/img/2016/10/1.tryb-recovery-router-tp-link-openwrt-lede.png#huge)

W stosunku do normalnego procesu boot zmienił się `is_auto_upload_firmware` z `0` na `1` , co
sugeruje automatyczny upload obrazu firmware przy starcie routera z wciśniętym przyciskiem Reset.
Mamy też informację na temat adresu IP serwera TFTP, z którego ten obraz firmware ma zostać pobrany.
W tym przypadku jest to `192.168.0.66` . Jest też nazwa pliku, którego bootloader będzie szukał na
serwerze TFTP, tj. `wr1043v2_tp_recovery.bin` . Tak będziemy musieli nazwać plik, którym zamierzamy
wgrać na router z poziomu bootloader'a. Na tę nazwę składa się model routera ( `wr1043` ), jego
wersja sprzętowa ( `v2` ) oraz fraza `_tp_recovery.bin` .

## Jak odzyskać router przez tryb recovery

Wiemy zatem, że nasz przykładowy router TL-WR1043ND V2 posiada bootloader, który jest w stanie
przełączyć się w tryb recovery. Na potrzeby tego doświadczenia popsułem po raz kolejny swój
router, by sprawdzić jak wygląda odzyskiwanie tego urządzenia przez tryb recovery. Oczywiście
bootloader jest w dalszym ciągu sprawny. Diody migają, choć router się zapętla i nie chce
wystartować.

### Statyczna adresacja komputera

Jako, że w procesie ratowania routera nie będzie nam działał serwer DHCP, to musimy ustawić
statyczną adresację na komputerze, który zamierzamy podpiąć przewodem ethernet do routera. Na
linux'ie możemy to zrobić w poniższy sposób:

    # ifdown eth0
    # ip link set dev eth0 up
    # ip addr add 192.168.0.66/24 brd + dev eth0

### Serwer TFTP

By naprawić router, potrzebny nam będzie serwer TFTP. To taki bardzo prosty serwer FTP pozbawiony
całej masy rzeczy. W różnych systemach operacyjnych jest dostępne inne oprogramowanie, które możemy
wykorzystać do postawienia takiego serwera. Ja na linux'ie będę korzystał z `atftpd` . Ten pakiet
jest dostępny standardowo w dystrybucji Debian, zatem nie powinno być problemów z jego instalacją.
Natomiast konfiguracja serwera TFTP sprowadza się do edycji pliku `/etc/default/atftpd` , który
musimy przepisać do poniższej postaci:

    USE_INETD=false
    OPTIONS="--tftpd-timeout 300 --retry-timeout 5 --maxthread 100 --verbose=5 --bind-address=192.168.0.66 /srv/tftp-openwrt"

Teraz odpalamy serwer TFTP i sprawdzamy czy demon nasłuchuje:

    # /etc/init.d/atftpd start

    # netstat -tupan | grep tft
    udp        0      0 192.168.0.66:69         0.0.0.0:*                           56791/atftpd

### Wgrywanie firmware na router

Wyłączamy router. Pobieramy obraz firmware, czy to ze strony TP-LINK czy OpenWRT/LEDE. Plik nazywamy
zgodnie z instrukcją, którą wypisał nam bootloader, tj. `wr1043v2_tp_recovery.bin` i wrzucamy go do
katalogu `/srv/tftp-openwrt/` :

    # cp /home/morfik/Desktop/TL-WR1043ND/openwrt-15.05-ar71xx-generic-tl-wr1043nd-v2-squashfs-factory.bin /srv/tftp-openwrt/wr1043v2_tp_recovery.bin

W przypadku obrazów firmware pobranych ze strony TP-LINK trzeba trochę uważać. Jako, że zawierają
one część bootloader'a, to trzeba ten kawałek obrazu pierw usunąć. Jeśli nie chcemy się w to bawić,
to zawsze możemy pobrać już [gotowe obrazy recovery, np.
stąd](https://tplinkforum.pl/c/porady-serwisowe/pliki-recovery-dla-routerow-tp-link). Te obrazy są
takie same jak te bez boot dostępne choćby
[tutaj](https://tplinkforum.pl/t/firmware-bez-bootloadera-dla-routerow-tp-link/9661). Jedyna różnica
między nimi to zmieniona nazwa na taką, której bootloader żąda. Trochę dziwne, że TP-LINK nie ma na
swojej stronie tego typu obrazów.

Mając już odpowiedni plik recovery, wciskamy i trzymamy przycisk reset na obudowie routera, po czym
włączamy zasilanie przyciskiem Power. Proces flash'owania potrwa chwilę. Jest on w pełni
automatyczny i nie musimy wykonywać praktycznie żadnych czynności. Wystarczy trochę poczekać. Router
powinien się samoczynnie uruchomić ponownie, tym razem już z działającym systemem.

Cały ten powyższy proces podejrzałem sobie na konsoli szeregowej:

![](/img/2016/10/2.tryb-recovery-router-tp-link-openwrt-lede.png#huge)

Jak widać, tryb recovery automatyzuje cały proces naprawy routera przez konsolę szeregową. Niżej zaś
w logu mamy jeszcze:

![](/img/2016/10/3.tryb-recovery-router-tp-link-openwrt-lede.png#huge)

Czyli proces flash'owania przebiegł bez problemów i router startuje. Zatem jeśli bootloader w naszym
routerze posiada taki tryb recovery, to możemy zapomnieć o bawieniu się konsolą szeregową,
przynajmniej jeśli chodzi o odzysk routerów po nieudanym wgraniu firmware.
