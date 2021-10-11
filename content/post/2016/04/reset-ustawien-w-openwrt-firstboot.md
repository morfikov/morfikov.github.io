---
author: Morfik
categories:
- OpenWRT
date: "2016-04-23T17:30:52Z"
date_gmt: 2016-04-23 15:30:52 +0200
published: true
status: publish
tags:
- chaos-calmer
- router
GHissueID: 403
title: Reset ustawień w OpenWRT (firstboot)
---

Routery zwykle nie posiadają monitorów, klawiatur czy myszy. Gdy z takim urządzeniem zaczynają się
dziać problemy, mamy bardzo niewielkie pole manewru. Nie dość, że nie mamy jak zarządzać takim
routerem, to jeszcze rozebranie go zwykle nic nam nie da. Nie są to przecie dekstop'y, z których
można wymontować dysk czy podpiąć do nich pendrive live i odratować znajdujący się na nich system
operacyjny. Czasem wystarczy drobny błąd konfiguracyjny, by router nie chciał się nam odpalić lub
nie będziemy w stanie się z nim połączyć. Z powodu tak ograniczonego dostępu do systemu routerów w
przypadku awarii, ich [firmware zwykle jest wyposażony w mechanizm resetowania
konfiguracji](http://www.tp-link.com/en/faq-140.html) do ustawień fabrycznych. Podobnie sprawa ma
się w przypadku firmware OpenWRT. Cały mechanizm resetowania ustawień nosi nazwę `firstboot` lub
`factory defaults` albo też `factory reset` . W tym wpisie postaramy się zresetować ustawienia
routera na kilka sposobów.

<!--more-->
## Aktywowanie firstboot z poziomu działającego systemu

Gdy sporo eksperymentujemy z oprogramowaniem, może się zdarzyć sytuacja, że chcielibyśmy cofnąć
wszystkie wprowadzone zmiany i mieć świeży system. Niby nic naszemu routerowi nie dolega ale z racji
pozostawionych śmieci przydałoby się przeczyścić mu system bez ponownego flash'owania. Możemy, co
prawda, próbować odinstalować wszystkie zainstalowane pakiety i usunąć wszystkie wgrane pliki ale to
jest czasochłonne i nie mamy pewności, czy aby na pewno pozbyliśmy się wszystkich śmieci z systemu.
Jedyny pewny sposób to aktywowanie procedury `firstboot` .

By zresetować ustawienia poprawnie działającego OpenWRT, logujemy się na router via SSH i wydajemy
to poniższe polecenie:

    # jffs2reset -y && reboot &
    /dev/mtdblock3 is mounted as /overlay, only erasing files

Po chwili router powinien się zresetować. Po tym jak system się uruchomi, możemy się zalogować
korzystając z protokołu telnet.

## Aktywowanie firstboot przez tryb failsafe

W przypadku, gdy straciliśmy praktycznie całą kontrolę nad routerem i nie możemy się zalogować po
SSH/telnet, możemy rozważyć wejście w [tryb
failsafe](/post/tryb-ratunkowy-failsafe-w-openwrt/) i z tego trybu awaryjnego
spróbować odratować system. Jeśli nie jesteśmy w stanie doprowadzić do porządku plików
konfiguracyjnych, to zawsze możemy zaaplikować procedurę `firstboot` będąc w trybie failsafe. Po
zalogowaniu się w trym trybie przez telnet ( `telnet 192.168.1.1` ), wpisujemy te poniższe
polecenia:

    # firstboot
    # reboot -f

Jeśli te polecenia nie zresetują ustawień do fabrycznych, to możemy spróbować wyczyścić partycję ze
zmianami w poniższy sposób:

    # mtd -r erase rootfs_data
    Unlocking rootfs_data ...
    Erasing rootfs_data ...

    Rebooting ...

Po chwili router powinien się zresetować i załadować z domyślnymi ustawieniami.

## Aktywowanie firstboot za pomocą przycisku na obudowie

Ostatnią opcją jaką mamy, by zresetować ustawienia routera do fabrycznych, to wciśnięcie przycisku
na jego obudowie. Z reguły routery mają kilka przycisków. Niekoniecznie wszystkie z nich muszą być
obsługiwane przez OpenWRT ale w zdecydowanej większości routerów co najmniej jeden z nich działa i
ma przypisanych szereg akcji.

W katalogu `/etc/rc.button/` mamy kilka plików. Nazwy ich wskazują na konkretne przyciski, np.
`reset` odpowiada za akcje przypisane do przycisku, który resetuje router. W tym przypadku, treść
tego pliku wygląda następująco:

    #!/bin/sh

    [ "${ACTION}" = "released" ] || exit 0

    . /lib/functions.sh

    logger "$BUTTON pressed for $SEEN seconds"

    if [ "$SEEN" -lt 1 ]
    then
            echo "REBOOT" > /dev/console
            sync
            reboot
    elif [ "$SEEN" -gt 5 ]
    then
            echo "FACTORY RESET" > /dev/console
            jffs2reset -y && reboot &
    fi

    return 0

Mamy zatem przypisane dwie akcje do tego przycisku. Jedna z nich resetuje router, druga zaś
przywraca mu ustawienia fabryczne. To, która z tych dwóch akcji zostanie zainicjowana zależy od
czasu trzymania przycisku. Wyżej mamy dwa warunki: `if [ "$SEEN" -lt 1 ]` oraz `elif [ "$SEEN" -gt 5
]` . Zatem jeśli przycisk będziemy trzymać przez mniej niż jedną sekundę, to po jego puszczeniu
router zostanie zresetowany. W przypadku, gdy będziemy trzymać ten przycisk przez 5 sekund i po tym
czasie go puścimy, to zainicjujemy procedurę `firstboot` , która przywróci ustawienia fabryczne
routera. Zarówno akcje przypisane do przycisków jak i czas trzymania można sobie dostosować wedle
uznania.

## Problemy związane z firstboot

Jeśli po resecie ustawień do fabrycznych nie mamy połączenia z siecią, a nasz router zwyczajnie nie
odpowiada na żądania ping czy próby połączenia za pomocą protokołu telnet, oznacza to prawdopodobnie
złą adresację. Jeśli nasz router przed dokonaniem `firstboot` miał inną adresację LAN niż
standardowo, tj. 192.168.1.0/24, to musimy zresetować połączenie sieciowe na komputerze, z którego
chcemy się podłączyć do routera po resecie jego ustawień.
