---
author: Morfik
categories:
- Linux
date: "2015-06-23T22:50:14Z"
date_gmt: 2015-06-23 20:50:14 +0200
published: true
status: publish
tags:
- iptables
- ipset
- systemd
GHissueID: 98
title: PeerGuardian w oparciu o ipset i iptables
---

Wiele klientów torrent'a umożliwia ładowanie zewnętrznej listy z zakresami adresów IP i ta lista ma
służyć jako swego rodzaju filtr połączeń chroniący nas przed różnego rodzaju organizacjami, które
mogą zbierać i przetwarzać informacje na temat naszego IP i tego co on porabia w sieci p2p.
Oczywiście, kwestia czy korzystać z takiego typu rozwiązania jest bardzo dyskusyjna i wiele osób
jest zdania, że to tak naprawdę w niczym nie pomoże, a wręcz nawet przyczynia się do
samounicestwienia sieci p2p. Także taki filter może czasem przynieść więcej szkody niż pożytku,
zwłaszcza gdy się go używa lekkomyślnie, czyli na zasadzie, że ten co blokuje więcej adresów musi
być lepszy. Poniżej opiszę wykorzystanie rozszerzenia `ipset`, przy pomocy którego zostanie
zablokowany szereg klas adresów. Całość raczej nie skupia się na implementacji filtra, to tylko
przykład do czego `ipset` może posłużyć, a że ja nie umiem teoretycznie pisać, to muszę na
przykładach.

<!--more-->
## Reguły dla ipset'a

Standardowo pakietu `ipset` nie ma wgranego w systemie i trzeba go dociągnąć. Jako, że znajduje się
on w repozytorium debiana, to nie powinno być problemu z jego instalacją. Musimy także stworzyć
katalog, w którym to będziemy trzymać listy z adresami IP. Ja obrałem sobie folder
`/etc/peerblock/` . Dodatkowo musimy utworzyć osobny folder z konfiguracją dla iptables, np.
`/etc/filtr/` i od tego katalogu zaczniemy.

Potrzebne nam będą trzy pliki: `base.sh`, `ipset.sh` oraz `iptables_raw.sh` :

    # cd /etc/filtr/
    # touch base.sh ipset.sh iptables_raw.sh
    # chmod 700 *

### Plik base.sh

W pliku `base.sh` znajduje się podstawowy filtr dla stacji klienckich. Będzie on aplikowany zawsze
wtedy, gdy będziemy z jakiegoś powodu potrzebowali zdjąć główną zaporę. W ten sposób nie zostaniemy
bez ochrony. Plik ma poniższą postać:

    #!/bin/sh

    iptables_stop()
    {
          ipt="$(which iptables)"
          ip6t="$(which ip6tables)"

          $ipt -P INPUT DROP
          $ipt -P FORWARD DROP
          $ipt -P OUTPUT ACCEPT

          $ip6t -P INPUT DROP
          $ip6t -P FORWARD DROP
          $ip6t -P OUTPUT ACCEPT

          for iptable in $ipt $ip6t
          do
                for table in \
                      "-t raw" \
                      "-t mangle" \
                      "-t filter" \
                      "-t nat"
                do
                      $iptable $table -F
                      $iptable $table -X
                done
          done

          $ipt -t filter -A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
          $ipt -t filter -A INPUT -i lo -j ACCEPT
          $ipt -t filter -A INPUT -m conntrack --ctstate INVALID -j DROP
          $ipt -t filter -A INPUT -p udp -j REJECT --reject-with icmp-port-unreachable
          $ipt -t filter -A INPUT -p tcp -j REJECT --reject-with tcp-reset
          $ipt -t filter -A INPUT -j REJECT --reject-with icmp-proto-unreachable

          $ip6t -t filter -A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
          $ip6t -t filter -A INPUT -i lo -j ACCEPT
          $ip6t -t filter -A INPUT -m conntrack --ctstate INVALID -j DROP
          $ip6t -t filter -A INPUT -p tcp -j REJECT --reject-with tcp-reset
          $ip6t -t filter -A INPUT -p udp -j REJECT --reject-with icmp6-port-unreachable
    }

    ipset_stop()
    {
          ips=$(which ipset)
          sets=$($ips -n --list)

          for set in $sets
          do
                name=$($ips list $set 2> /dev/null | head -n 1 | cut -d ' ' -f2)
                if [ "$name" = "$set" ]; then
                      $ips destroy $set 2>&1
                fi
          done
    }

    iptables_stop
    ipset_stop

### Plik ipset.sh

Następnie tworzymy plik `ipset.sh` , który będzie odpowiedzialny za tworzenie list z adresami:

    #!/bin/sh

    ips="$(which ipset)"

    $ips create bt_level1 hash:net family inet maxelem 270000
    $ips create bt_spyware hash:net family inet maxelem 4000
    $ips create bt_webexploit hash:net family inet maxelem 2000
    $ips create whitelist hash:ip family inet maxelem 500

    sets="$($ips -n --list)"

    for set in $sets
    do
          cat /etc/peerblock/$set.gz |
          gunzip |
          cut -d: -f2 |
          grep -E "^[-0-9.]+$" |
          gawk -v my_set=$set '{print "add " my_set " " $1}' |
          $ips restore -exist;
    done

Mamy tutaj do obróbki trzy listy, które będziemy pobierać z serwisu [iblocklist.com][1].
Naturalnie, można sobie dobrać jakiekolwiek zestawy.

### Plik iptables\_raw.sh

Musimy także utworzyć konfigurację dla filtra `iptables` . Interesuje nas głównie tablica `raw` .
Tworzymy plik `iptables_raw.sh` o poniższej treści:

    #!/bin/sh

    ipt="$(which iptables) -t raw"

    $ipt -F
    $ipt -X

    $ipt -N peerblock_in
    $ipt -N peerblock_out

    $ipt -A PREROUTING -j peerblock_in
    $ipt -A OUTPUT -j peerblock_out

    $ipt -A peerblock_in -p all -m set --match-set whitelist src -j ACCEPT
    $ipt -A peerblock_in -p udp -m multiport ! --sports 80,443 -m set --match-set bt_level1 src -j DROP
    $ipt -A peerblock_in -p tcp -m multiport ! --sports 80,443 -m set --match-set bt_level1 src -j DROP
    $ipt -A peerblock_in -m set --match-set bt_spyware src -j DROP
    $ipt -A peerblock_in -m set --match-set bt_webexploit src -j DROP

    $ipt -A peerblock_out -p all -m set --match-set whitelist dst -j ACCEPT
    $ipt -A peerblock_out -p tcp -m multiport ! --dports 80,443 -m set --match-set bt_level1 dst -j DROP
    $ipt -A peerblock_out -p udp -m multiport ! --dports 80,443 -m set --match-set bt_level1 dst -j DROP
    $ipt -A peerblock_out -m set --match-set bt_spyware dst -j DROP
    $ipt -A peerblock_out -m set --match-set bt_webexploit dst -j DROP

Zanim pakiety dotrą do celu, trafiają pierw do tablicy `raw` i tam można je wstępnie odfiltrować.
Stworzyliśmy tam dwa dodatkowe łańcuchy, po jednym dla pakietów przychodzących i wychodzących.
Kluczowa sprawa to wywołanie modułu przy pomocy parametru `-m set` i dopasowanie pakietów do
odpowiedniego filtra w ipset przy pomocy `--match-set`. Ja mam 4 filtry, po 1 regule dla każdego,
w każdym łańcuchu, łącznie daje 8 wpisów. Białą listę dajemy na początek, tak by pakiety, które
zostałyby zablokowane przez przez 3 pozostałe filtry, były przepuszczane bez większego problemu i
wędrowały do tablicy `filter`. Lista `bt_level1` jest to typowa lista p2p i generalnie lepiej jest
jej nie używać na portach `80` i `443` , czyli na stronach www, bo ona nawet gógla blokuje.
Najlepiej ustawić sobie zakres portów, na których klient bittorrent'a może dokonywać połączeń. U
mnie qbittorrent ma zabronione łączenie się na porty poniżej numeru 1024 i na dobrą sprawę,
ustawiłem mu zakres 2k portów od 20-22k. I praktycznie można by sprecyzować ten zakres w regule
iptables. Pozostałe dwa filtry są już użyte bardziej restrykcyjnie ale to dlatego, że zakresów
adresów IP jest tam relatywnie niewielki i w ciągu dnia trafia mi się tam może 20-30 pakietów.
Różnica w stosunku do tego dużego filtra polega na tym, że te dwa mniejsze filtrują ruch również na
http i https.

## Praca dla cron'a

Co prawda skrypt z filtrem będzie ładowany przy starcie systemu ale on nie pobiera list. Zamiast
tego, ładuje je tylko do ipset'a. Ku temu jest kilka powodów. Pierwszym jest oczywiście spowolnienie
startu systemu. Kolejnym powodem może być brak sieci. Jak nie patrzeć, sieć zostanie dopiero
uruchomiona jak listy zostaną zaaplikowane, czyli muszą już istnieć na dysku. Najlepszym wyjściem w
tej sytuacji jest rozdzielenie kwestii pobierania i aplikowania list. Listy będą aktualizowane co
dzień, dlatego też najlepiej po prostu obarczyć tym zadaniem cron'a i adresy z plików, które on
stworzy, ładować do ipset'a. Tworzymy zatem plik `/etc/cron.daily/peerblock_list` o poniższej
treści:

    #!/bin/bash

    #echo "pozyskiwanie listy bt_level1..."
    curl --silent --show-error -L "http://list.iblocklist.com/?list=bt_level1&fileformat=p2p&archiveformat=gz" > /etc/peerblock/bt_level1.gz
    #echo "pozyskiwanie listy bt_spyware..."
    curl --silent --show-error -L "http://list.iblocklist.com/?list=bt_spyware&fileformat=p2p&archiveformat=gz" > /etc/peerblock/bt_spyware.gz
    #echo "pozyskiwanie listy bt_webexploit..."
    curl --silent --show-error -L "http://list.iblocklist.com/?list=ghlzqtqxnzctvvajwwag&fileformat=p2p&archiveformat=gz" > /etc/peerblock/bt_webexploit.gz

Ta lista będzie pobierana raz dziennie. Niemniej jednak, potrzebujemy wszystkich z powyższych plików
przed odpaleniem zapory. Dlatego też odpalmy ww. skrypt ręcznie, by pliki zostały pobrane i
umieszczone w odpowiednim miejscu.

## Usługa dla systemd

Pozostało nam jeszcze napisanie odpowiedniego pliku usługi dla systemd . Jeśli wcześniej
[zbudowaliśmy sobie firewall][2], to musimy jedynie dostosować plik
`/etc/systemd/system/firewall.service` . Jego zawartość powinna być mniej więcej taka:

    [Unit]
    Description=firewall
    Documentation=man:iptables
    DefaultDependencies=no
    Wants=network-pre.target systemd-modules-load.service
    Before=network-pre.target
    After=systemd-modules-load.service

    [Service]
    Type=oneshot
    RemainAfterExit=yes
    ExecStart=/bin/sh -c "/etc/filtr/ipset.sh"
    ExecStart=/bin/sh -c "/etc/filtr/iptables_raw.sh"
    ExecStart=/bin/sh -c "/etc/filtr/iptables_mangle.sh"
    ExecStart=/bin/sh -c "/etc/filtr/iptables_nat.sh"
    ExecStart=/bin/sh -c "/etc/filtr/iptables_filter.sh"
    ExecStart=/bin/sh -c "/etc/filtr/ip6tables_raw.sh"
    ExecStart=/bin/sh -c "/etc/filtr/ip6tables_mangle.sh"
    ExecStart=/bin/sh -c "/etc/filtr/ip6tables_nat.sh"
    ExecStart=/bin/sh -c "/etc/filtr/ip6tables_filter.sh"
    ExecStop=/bin/sh -c "/etc/filtr/base.sh"

    [Install]
    WantedBy=multi-user.target

To w zasadzie tyle pracy z naszej strony. W kilku plikach nie umieszczaliśmy żadnych reguł, a to z
tego względu, że mogą się one różnić w przypadku każdej maszyny.

Trzeba także pamiętać, że ilość wpisów na listach ulega zmianie i może się zdarzyć tak, że limit
ustawiony w zmiennej `maxelem` w pliku `ipset.sh` zostanie przekroczony. Gdy taka sytuacja wystąpi,
to wystarczy podbić ten limit. Nie należy jednak z nim przesadzać. Co prawda, przy długości `290000`
taka lista zajmuje koło 3,5 MiB RAMu. Nie jest to dużo ale jeśli byśmy tam mieli tylko jeden adres,
to ilość okupowanej pamięci nie ulegnie zmianie. To i tak sporo lepiej w porównaniu do qbittorrenta,
który za obsługę tej samej listy chciał ponad 200MiB.

## Niekompatybilny format list ipset'a

Format listy nie jest do przyjęcia przez ipset. Wpisy w liście wyglądają jak poniżej:

    # List distributed by iblocklist.com

    China Internet Information Center (CNNIC):1.2.4.0-1.2.4.255
    China Internet Information Center (CNNIC):1.2.8.0-1.2.8.255
    SMSHoax FakeAV Fraud Trojan:1.9.75.8-1.9.75.8
    SMSHoax FakeAV Fraud Trojan:1.9.189.65-1.9.189.65
    ...

A trzeba je przerobić do postaci:

    add bt_level1 2.115.176.190
    add bt_level1 62.52.2.0/26
    add bt_level1 212.174.156.0/23
    ...

Za to odpowiada plik `ipset.sh` , a konkretnie ten poniży kod:

    for set in $sets
    do
          cat /etc/peerblock/$set.gz |
          gunzip |
          cut -d: -f2 |
          grep -E "^[-0-9.]+$" |
          gawk -v my_set=$set '{print "add " my_set " " $1}' |
          $ips restore -exist;
    done

Poniżej wyjaśnienie poszczególnych etapów przetwarzania listy:

  - `cat /etc/peerblock/$set.gz` wczytuje do pamięci plik listy
  - `gunzip` wypakowuje ten plik
  - `cut -d: -f2` dzieli linijki po znaku dwukropka i części powstałe w taki sposób oznacza kolejno
    1,2, etc. W tym przypadku, jako że linijka ma tylko jeden dwukropek, zostanie podzielona na 2
    części oraz zostanie wycięty kawałek od dwukropka do końca wiersza.
  - `grep -E "^[-0-9.]+$"` nie wiem co dokładnie znaczy ale chyba wyciąga z wyniku wpisy, które mają
    cyfry, kropki i znak minusa powtórzone co najmniej raz. Czyli wynik w postaci
    `62.27.97.232-62.27.97.235` . Takie zakresy IP zostaną automatycznie przepisane przez ipset z
    uwzględnieniem maski podsieci (patrz [ipcalc][3]).
  - `gawk -v my_set=$set '{print "add " my_set " " $1}'` dodaje na początku frazę `add bt_level1`

Tak przerobioną listę można bez problemu teraz zaaplikować w ipsecie.

## Biała lista

Ja w powyższy sposób zaimplementowałem sobie 3 listy z adresami IP ale jest jeszcze jedna, którą
warto mieć. Jest to lista z adresami, którym udzielimy dostępu do naszej maszyny. Te filtry mogą i z
pewnością będą blokować sporo adresów, które są niewinne i dlatego przydałoby się mieć już gotowe
rozwiązanie na wypadek gdy ktoś z naszych znajomych trafi na taką listę.

Tworzymy zatem plik `/etc/peerblock/whitelist` i dodajemy tam adresy w formie `91.121.10.104` :

    # cat /etc/peerblock/whitelist
    91.121.10.104

Po edycji, plik kompresujemy programem `gzip` :

    # cd /etc/peerblock/
    # gzip whitelist

## Logowanie blokowanych przez ipset pakietów

Jeśli pakiety zostaną zablokowane z jakiegoś ip, a my nie wiemy z jakiego. Możemy włączyć logowanie
dodając do regułek iptables do łańcuchów `PREROUTING` oraz `OUTPUT` coś na wzór poniższej linijki:

    -A OUTPUT -p tcp -m multiport ! --dports 80,443 -m set --match-set bt_level1 dst -j LOG -m limit --limit 10/m  --log-level 4 --log-prefix "*** BT_LEVEL1 *** "

Oczywiście to tylko przykład logowania, prawdopodobnie trzeba będzie go dostosować. Jedna rzecz o
której trzeba pamiętać to by dodać go przed faktyczną regułą filtrującą, w przeciwnym razie nie
przejdą przez nią pakiety i nie zostaną zalogowane.

I to w sumie wszystko, testowałem tego PeerGuardian'a opartego o ipset i to po prostu działa. Jeśli
kogoś przeraża tego typu działanie, może pokusić się o [prawie gotowe rozwiązanie][4]. Wymaga tylko
skompilowania i instalacji. W każdym razie posiada GUI i zjada więcej RAMu niż ipset.


[1]: https://www.iblocklist.com/lists.php
[2]: /post/firewall-na-linuxowe-maszyny-klienckie/
[3]: http://manpages.ubuntu.com/manpages/xenial/en/man1/ipcalc.1.html
[4]: https://sourceforge.net/projects/peerguardian/
