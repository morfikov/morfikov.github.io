---
author: Morfik
categories:
- OpenWRT
date: "2016-04-21T16:30:44Z"
date_gmt: 2016-04-21 14:30:44 +0200
published: true
status: publish
tags:
- chaos-calmer
- sysupgrade
- router
GHissueID: 415
title: Sysupgrade, czyli aktualizacja firmware OpenWRT
---

Alternatywne oprogramowanie OpenWRT dla routerów trzeba regularnie aktualizować. Nie chodzi tutaj
tylko o znane podatności w oprogramowaniu ale też o wyeliminowanie błędów wynikłych z niepoprawnego
działania sterownika czy jakiegoś modułu samego kernela. Proces aktualizacji firmware nie jest
niczym trudnym, choć przeprowadzany niewłaściwie może zakończyć się uwaleniem routera. Poniższy wpis
ma na celu zaprezentowanie jak krok po kroku dokonać aktualizacji oprogramowania za pomocą narzędzia
`sysupgrade` . Postaramy się to zrobić w oparciu o najnowszą dostępną wersję OpenWRT, w tym
przypadku będzie to Chaos Calmer 15.05.1 (r49172).

<!--more-->
## Pobieranie i weryfikacja obrazu z firmware

By zabrać się do aktualizacji OpenWRT do nowszej wersji, musimy postarać się o odpowiedni obraz z
firmware dla naszego routera. W tym przypadku mamy router [TP-LINK TL-WR1043ND
v2](http://www.tp-link.com.pl/products/details/TL-WR1043ND.html). Obrazy możemy zassać bezpośrednio
ze [strony OpenWRT](https://downloads.openwrt.org/). Możemy także pobrać lekko zmodyfikowane wersje
z [dl.eko.one.pl](http://dl.eko.one.pl/chaos_calmer/ar71xx/).

Dla każdego ze wspieranych routerów są dwa obrazy: `sysupgrade` oraz `factory` . Obraz `sysupgrade`
jest używany do aktualizacji OpenWRT, zaś `factory` znajduje zastosowanie, gdy zmieniamy firmware
producenta routera na OpenWRT. Zasysamy zatem obraz mający w nazwie `sysupgrade` . Nie musimy tego
robić przez przeglądarkę. Możemy zwyczajnie posłużyć się narzędziem `wget` i dociągnąć obraz
bezpośrednio na routerze. Pamiętajmy tylko, by zapisać go w pamięci RAM, a nie na flash'u. Poniżej
przykład:

    # cd /tmp/
    # wget http://dl.eko.one.pl/chaos_calmer/ar71xx/openwrt-15.05-ar71xx-generic-tl-wr1043nd-v2-squashfs-sysupgrade.bin -O obraz.bin
    Connecting to dl.eko.one.pl (178.235.241.16:80)
    obraz.bin            100% |***********************|  3648k  0:00:00 ETA

    # ls -al /tmp/obraz.bin
    -rw-r--r--    1 root     root       3735556 Apr 21 13:26 /tmp/obraz.bin

Tak pobrany obraz trzeba zweryfikować pod kątem ewentualnych błędów. Tam na serwerze z obrazami jest
dostępny plik `md5sums` . Zawiera on sumy kontrolne wszystkich udostępnionych plików. Musimy z tego
pliku wyciągnąć sumę kontrolną dla tego powyższego obrazu. Robimy to w poniższy
    sposób:

    # wget -q -O - "$@" http://dl.eko.one.pl/chaos_calmer/ar71xx/md5sums | grep 1043nd-v2 | grep sysupgrade
    d9380b171f350b5f4a4deff80053a0cd  openwrt-15.05-ar71xx-generic-tl-wr1043nd-v2-squashfs-sysupgrade.bin

Sprawdzamy teraz sumę kontrolną obrazu, który pobraliśmy do pamięci routera:

    # md5sum /tmp/obraz.bin
    d9380b171f350b5f4a4deff80053a0cd  /tmp/obraz.bin

Widzimy, że obie sumy kontrolne są takie same, zatem obraz jest wolny od błędów.

## Jak wykonać backup za pomocą sysupgrade

Przed każdą aktualizacją firmware dobrze jest zgrać sobie konfigurację routera. Tak na wszelki
wypadek, gdyby jakieś nieprzewidziane zdarzenia miały miejsce w przyszłości. Narzędzie `sysupgrade`
oferuje nam możliwość zrobienia takiego backup'u konfiguracji. Ma ono kilka użytecznych opcji i w
zależności od tego co tak naprawdę chcemy osiągnąć, będziemy używać innych parametrów. W tej chwili
interesuje nas przełącznik `-b` , za pomocą którego zapiszemy całą kopię w pliku `.tar.gz` .

`sysupgrade` ma swój plik konfiguracyjny `/etc/sysupgrade.conf` , w którym to możemy dodawać pliki,
a nawet całe katalogi. W ten sposób mówimy `sysupgrade` , które pliki mają zostać uwzględnione przy
tworzeniu kopi zapasowej. Pliki/katalogi definiujemy w formacie jeden na linię. Poniżej przykład:

    ...
    /etc/sysupgrade.conf
    /etc/openvpn/
    ...

Część plików jest backup'owana automatycznie. Możemy się o tym przekonać zaglądając do katalogu
`/lib/upgrade/keep.d/` . Po uwzględnieniu wszystkich plików, kopię tworzymy w poniższy sposób:

    # sysupgrade -b kopia.tar.gz

Ten plik trzeba zgrać z routera przy pomocy `scp` na lokalny dysk:

    $ scp root@192.168.1.1:/tmp/kopia.tar.gz ./
    root@192.168.2.1's password:
    kopia.tar.gz              100% 5351     5.2KB/s   00:00

## Aktualizacja firmware z zachowaniem zmian w konfiguracji

`sysupgrade` dysponuje także przełącznikiem `-c` , który odpowiada za zachowanie zmian we wszystkich
plikach w katalogu `/etc/` podczas dokonywania aktualizacji OpenWRT. Zasadnicza różnica w stosunku
do tego poprzedniego kroku, to fakt, że nie tworzymy jako takiej kopi zapasowej. Po prostu
dokonujemy aktualizacji, a stare pliki konfiguracyjne zostają nietknięte. Ten mechanizm działa w
oparciu o sumy kontrolne. Wszystkie pliki w katalogu `/etc/`, które są wgrywane na router przy
pomocy menadżera pakietów `opkg` mają sumy kontrolne. W efekcie czego, jeśli zmienimy chociaż jeden
znak w takim pliku, to ulegnie zmianie jego suma kontrolna i taki plik nie zostanie nadpisany
podczas aktualizacji. Takie pliki możemy wyświetlić za pomocą poniższego polecenia:

    # opkg list-changed-conffiles

Trzeba jednak pamiętać, że nie zawsze da radę zachować konfigurację między aktualizacjami dwóch
bardzo różnych wersji firmware. Oprogramowanie bardzo szybko ulega zmianom, wliczając w to również i
jego konfigurację. Może się zdarzyć tak, że nowsza wersja firmware nie będzie kompatybilna z naszą
aktualną konfiguracją, co będzie powodowało problemy w działaniu routera. W każdym innym przypadku,
aktualizacja routera z zachowaniem konfiguracji odbywa się przez wpisanie w terminalu tego
poniższego polecenia:

    # sysupgrade -c /tmp/obraz.bin

## Aktualizacja firmware bez zachowania zmian w konfiguracji

Alternatywą do zachowania zmian podczas aktualizacji firmware jest wgranie czystego obrazu ze
świeżymi plikami konfiguracyjnymi. Wadą tego rozwiązania jest fakt, że trzeba będzie na nowo
przeprowadzać konfigurację routera. Oczywiście możemy podeprzeć się konfiguracją zrobioną manualnie
w pliku `kopia.tar.gz` . Tak czy inaczej, za wgranie czystego obrazu odpowiada opcja `-n` w
`sysupgrade` . Samo polecenie aktualizacji wygląda mniej więcej tak:

    # sysupgrade -n /tmp/obraz.bin

Po wydaniu tego powyższego polecenia, w terminalu pojawi nam się szereg komunikatów informujących
nas o postępie procesu aktualizacji. Na jego końcu powinniśmy zobaczyć linijkę z `Upgrade completed,
Rebooting system...` . Po chwili system powinien się podnieść z domyślnymi ustawieniami. Na taki
router logujemy się via `telnet 192.168.1.1` i ustawiamy hasło dla użytkownika root wpisując w
terminal `passwd` .

## Przywracanie konfiguracji po aktualizacji firmware

Jeśli korzystaliśmy z tworzenia backup'u konfiguracji w pliku `kopia.tar.gz` oraz dokonaliśmy
aktualizacji OpenWRT podając `sysupgrade` przełącznik `-n` , to możemy pokusić się o odtworzenie
konfiguracji na takim routerze. Najprościej to zrobić przez zainstalowanie wszystkich potrzebnych
pakietów i ponowne utworzenie konfiguracji za pomocą `sysupgrade -b` . Po wypakowaniu tych dwóch
paczek, można porównać oba katalogi, np. przy pomocy narzędzia `meld` . Wylistuje ono nam zmiany w
poszczególnych plikach i na podstawie tych zmian będziemy w stanie stwierdzić, czy utworzony przez
nas backup nadaje się do wgrania na router. W przypadku, gdy nowsza wersja firmware wprowadziła
jakieś zmiany w konfiguracji, to wszystkie te zmiany można zwyczajnie przenieść. Wygląda to mniej
więcej tak:

![sysupgrade-meld-diff-openwrt-firmware-aktualizacja](/img/2016/04/1.sysupgrade-meld-diff-openwrt-firmware-aktualizacja.png#huge)

![sysupgrade-meld-diff-openwrt-firmware-aktualizacja](/img/2016/04/2.sysupgrade-meld-diff-openwrt-firmware-aktualizacja.png#huge)

Po zakończeniu tego procesu, pakujemy pliki:

    # cd kopia/
    # chown root:root -R *
    # tar czpf config.tar.gz etc/

Tak otrzymaną paczkę wgrywamy na router i przywracamy konfigurację przy pomocy `sysupgrade -r` :

    $ scp ./config.tar.gz root@192.168.1.1:/tmp/
    $ ssh root@192.168.1.1

    # sysupgrade -r /tmp/config.tar.gz
    # reboot

## Problemy po aktualizacji firmware

Połączenie z routerem realizowane jest przez protokół `telnet` lub `ssh` w zależności od tego czy
mamy ustawione hasło dla użytkownika root. Zwykle takie hasło jest ustawione. Protokół `ssh` operuje
na kluczach szyfrujących, a każdy taki klucz ma inny odcisk packa (fingerprint). Po aktualizacji
OpenWRT, te klucze zostają zmienione, w efekcie czego przy próbie logowania na shell'a zobaczymy
taki oto poniższy komunikat.

    $ ssh root@192.168.1.1

    @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
    @    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
    @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
    IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
    Someone could be eavesdropping on you right now (man-in-the-middle attack)!
    It is also possible that a host key has just been changed.
    The fingerprint for the RSA key sent by the remote host is
    3a:60:bf:95:8e:19:39:35:85:9a:7b:19:02:57:52:4a.
    Please contact your system administrator.
    Add correct host key in /home/morfik/.ssh/known_hosts to get rid of this message.
    Offending RSA key in /home/morfik/.ssh/known_hosts:16
      remove with: ssh-keygen -f "/home/morfik/.ssh/known_hosts" -R 192.168.1.1
    RSA host key for 192.168.1.1 has changed and you have requested strict checking.
    Host key verification failed.

By móc się zalogować na router, musimy zaktualizować plik `~/.ssh/known_hosts` . Robimy to w
poniższy sposób:

    $ ssh-keygen -f "/home/morfik/.ssh/known_hosts" -R 192.168.1.1
    # Host 192.168.1.1 found: line 16 type RSA
    /home/morfik/.ssh/known_hosts updated.
    Original contents retained as /home/morfik/.ssh/known_hosts.old

Teraz ponawiamy żądanie połączenia. Zostaniemy poproszeni o zweryfikowanie nowego odciska palca
klucza:

    $ ssh root@192.168.1.1
    The authenticity of host '192.168.1.1 (192.168.1.1)' can't be established.
    RSA key fingerprint is 3a:60:bf:95:8e:19:39:35:85:9a:7b:19:02:57:52:4a.
    Are you sure you want to continue connecting (yes/no)? yes
    Warning: Permanently added '192.168.1.1' (RSA) to the list of known hosts.
    root@192.168.1.1's password:

Po podaniu hasła do konta root, zostaniemy zalogowani na router.
