---
author: Morfik
categories:
- Linux
date: "2019-03-02T02:21:12Z"
published: true
status: publish
tags:
- debian
- ipset
- nftables
title: Brak wsparcia dla ipset w nftables
---

Użytkownicy Debiana często w roli firewall'a wykorzystują już dość leciwy `iptables` . W zasadzie,
to tej implementacji linux'owego filtra pakietów sieciowych nic nie dolega, no może poza szeregiem
wad konstrukcyjnych, które są obecnie tak ciężkie do zaadresowania, że w sumie trzeba by cały ten
`iptables` napisać od początku. Wszystko przez rozwój internetu, za sprawą którego pojawiło się
zapotrzebowanie na tworzenie całej masy reguł (w postaci adresów/portów źródłowych/docelowych),
gdzie w standardowym `iptables` trzeba tworzyć osobne wpisy. Im więcej reguł w filtrze, tym
przechodzenie pakietów przez zaporę sieciową trwa dłużej i wiąże się z mocnym obciążeniem dla
procesora (zwłaszcza, gdy tych reguł jest kilkadziesiąt tysięcy). By jakoś uporać się z tymi
problemami (nieznanymi w innych filtrach sieciowych) stworzono `ipset` . I faktycznie odciążył on
mocno procesor maszyny ale i tak nie wyeliminował on podstawowych wad `iptables` . Dlatego też
zaczęto szukać innego rozwiązania i tak pojawiła się alternatywa m.in. w postaci `nftables` . W
przyszłym stabilnym Debianie (buster) `nftables` będzie wykorzystywany jako domyślny filtr pakietów
i ci, który korzystali z `ipset` mogą się nieco zdziwić, że `nftables` nie posiada dla niego
wsparcia. Rzecz w tym, że `nftables` potrafi natywnie obsługiwać listy adresów/portów i `ipset`
nie jest mu w tym do niczego potrzebny.

<!--more-->
## Jak przeportować reguły ipset do nftables

Do tej pory miałem szereg list adresów pobieranych z różnych serwisów internetowych, np. z
[iblocklist.com](https://www.iblocklist.com/lists). Te listy przy pomocy skryptu były przetwarzane
i dodawane do `ipset` . Kiedyś
nawet [zaprojektowałem PeerGuardian'a dla torrent'a](/post/peerguardian-w-oparciu-o-ipset-iptables/)
(i nie tylko dla niego), który działał właśnie w oparciu o `iptables` i `ipset` . Po zmigrowaniu
na `nftables` , tamto rozwiązanie już nie znajduje zastosowania ale też nie jest ono do końca
bezużyteczne, bo można je nieco przerobić. Dla przykładu weźmy sobie jedną przykładową listę,
którą można pobrać w poniższy sposób:

    # curl --silent --show-error -L "http://list.iblocklist.com/?list=bt_level1&fileformat=p2p&archiveformat=gz" > /etc/peerblock/bt_level1.gz

Tę listę trzeba przerobić, by `nftables` był w stanie z niej zrobić jakiś użytek. Przyda się do
tego poniższy skrypt:

    #!/bin/sh

    sets="bt_level1"

    for set in $sets
    do
        cat /etc/peerblock/$set.gz |
        gunzip |
        cut -d: -f2 |
        grep -E "^[-0-9.]+$" |
        sed 's/^\([^-]*\)-\1$/\1/' |
        sed "s/$/,/g" |
        sed "s/^/\ \ \ \ /g" |
        sed "1s;^;define $set \= \{\n;" |
        sed '1s;^;\#\!\/usr\/sbin\/nft \-f\n\n;' |
        sed "$ a \}\n" |
        sed "$ a add set ip raw\-set $set \{ type ipv4_addr\; flags interval\; auto-merge\; elements \= \$$set \}" > /etc/nftables/sets/nft_set-$set.nft
    done

Zmienna `$sets` może zawierać kilka list, które sobie pobraliśmy wcześniej. Po przetworzeniu listy,
zostanie stworzony plik `/etc/nftables/sets/nft_set-bt_level1.nft` , który prezentuje się w
poniższy sposób:

    #!/usr/sbin/nft -f

    define bt_level1 = {
        1.2.4.0-1.2.4.255,
        1.9.106.186,
        1.16.0.0-1.19.255.255,
        ...
        222.249.144.0-222.249.144.127,
        223.255.241.132,
    }

    add set ip raw bt_level1 { type ipv4_addr; flags interval; auto-merge; elements = $bt_level1 }

Wszystko co się znajduje między `define bt_level1 = { }` to zawartość listy, mniej więcej ta sama,
która by powędrowała do `ipset` .

Przy ostatniej linijce wypadałoby się zatrzymać chwilę, bo to ona odprawia całą magię ogarniania
list adresów. W zasadzie to nowy set dodaje się przez regułę `add set ip raw bt_level1 { }` .
Wyrażenie `ip raw` odnosi się do tutaj do rodziny adresów oraz tablicy filtra. W tym przypadku
listę adresów dodajemy do tablicy `raw` , którą przemierzać będą jedynie adresy IPv4. Ruch IPv6 tę
tablicę pominie. W tej regule, `bt_level1` oznacza nazwę seta i nie musi ona pasować do tego co
było określone w `define bt_level1 = { }` . Dalej zaś mamy szereg atrybutów, które wpływają na
tworzenie tej listy:

- `type` -- określa typ listy (obowiązkowy). Możliwe do wyboru `ipv4_addr` (adresy IPv4) ,
`ipv6_addr`(adresy IPv6) , `ether_addr` (adresy MAC), `inet_proto` (protokoły sieciowe),
`inet_service` (porty sieciowe), `mark` (oznaczenia pakietów).
- `flags` -- ustawia flagi. Do wyboru `constant` (zawartość seta nie może ulec zmianie, gdy
aktywny) , `interval` (do czego to FIXME)  oraz `timeout` (elementy mogą być
dodawane z określonym czasem ważności).
- `timeout` -- określa czas dla dodanych do seta elementów, po którym zostaną z niego usunięte. W
tym przypadku nieobowiązkowy, przez co adresy w secie będą obecne cały czas.
- `gc-interval` -- interwał dla mechanizmu oczyszczania (garbage collection). Dostępny jedynie,
gdy zostanie zdefiniowany `timeout` lub flaga `timeout` .
- `elements` -- lista elementów przechowywanych przez set.
- `size` -- maksymalna liczba elementów. Po przekroczeniu tego limitu, nowe elementy nie będą
dodawane do seta.
- `policy` -- ustawia politykę wyboru dla seta. Domyślnie `performance` ale można też określić
`memory` (różnica między nimi? FIXME).
- `auto-merge` -- automatycznie łączy przylegające lub nakładające się zestawy elementów. Dostępne
tylko, gdy ustawiona flaga `interval` .

Mając plik seta z listą adresów możemy go podpiąć pod filtr. By to uczynić, potrzebna nam jest
odpowiednia struktura `nftables` . Dodajemy zatem do głównego pliku reguł te poniższe wpisy:

    #!/usr/sbin/nft -f
    ...
    create table ip raw

    create chain ip raw PREROUTING { type filter hook prerouting priority -300; policy accept; }
    create chain ip raw OUTPUT { type filter hook output priority -300; policy accept; }

Do tego tworzymy osobne łańcuchy i przekierowujemy do nich cały ruch za wyjątkiem tego na
interfejsie `lo` :

    create chain ip raw peerblock-in
    create chain ip raw peerblock-out

    add rule ip raw PREROUTING meta iifname !="lo" jump peerblock-in
    add rule ip raw OUTPUT meta oifname !="lo" jump peerblock-out

Następnie załączamy wcześniej wygenerowany plik z listą adresów:

    include "./sets/nft_set-bt_level1.nft"

Ścieżka do tego pliku jest podana względem głównego pliku z regułami zapory sieciowej.

W tym przypadku chcemy zablokować komunikację obustronną z hostami obecnymi na liście na portach
innych niż `80` i `443` . Możemy to zrobić przy użyciu standardowych reguł:

    add rule ip raw peerblock-in  ip saddr @bt_level1 tcp sport != { 80, 443 } counter drop
    add rule ip raw peerblock-in  ip saddr @bt_level1 udp sport != { 80, 443 } counter drop

    add rule ip raw peerblock-out ip daddr @bt_level1 tcp dport != { 80, 443 } counter drop
    add rule ip raw peerblock-out ip daddr @bt_level1 udp dport != { 80, 443 } counter drop

Kluczowe w tych regułach jest odwołanie się do listy adresów w `ip daddr` używając `@` i nazwy,
którą podaliśmy w `add set ip raw bt_level ...` przy generowaniu listy (w skrypcie).

Można także użyć `vmap` :

    add rule ip raw peerblock-in  ip saddr @bt_level1 tcp sport vmap { 80:accept, 443:accept } counter drop
    add rule ip raw peerblock-in  ip saddr @bt_level1 udp sport vmap { 80:accept, 443:accept } counter drop

    add rule ip raw peerblock-out ip daddr @bt_level1 tcp dport vmap { 80:accept, 443:accept } counter drop
    add rule ip raw peerblock-out ip daddr @bt_level1 udp dport vmap { 80:accept, 443:accept } counter drop

W przypadku `vmap` trzeba jednak uważać, bo te powyższe reguły nie zadziałają
prawidłowo. [Potrzebny jest patch](https://marc.info/?l=netfilter&m=154686155231713&w=2) na kernel,
który aktualnie chyba jeszcze nie został formalnie sporządzony, przez co `drop` nie jest honorowany
w regułach z `vmap` . Dlatego lepiej korzystać ze standardowych reguł póki co.

Tego typu listy adresów naturalnie można dodać więcej i można je wpasować w dowolne miejsce
struktury filtra i niekoniecznie musi to być tablica `raw` , choć ona wydaje się być
najrozsądniejszym miejscem, bo ta tablica jest przetwarzana przed mechanizmem śledzenia połączeń.
Niemniej jednak, jeśli set zostanie dodamy tylko do tablicy `raw` to nie będzie można się do niego
odwołać np. z tablicy `filter` .

## Problemy z listami adresów w nftables

Może i `nftables` ma wbudowaną obsługę list adresów czy portów ale przy dłuższych listach zaczynają
się schody. Przede wszystkim, jest bardzo poważny problem z wydajnością. Można go doświadczyć przy
zwracaniu list reguł. Nawet, gdy będziemy chcieli zwrócić jedynie listing krótkiego łańcucha, to
ten proces zajmie nam kilka czy kilkanaście sekund. Ten problem został
już [po części zaadresowany](https://marc.info/?l=netfilter&m=154681757819757&w=2) ale dalej
stosowna łata nie trafiła jeszcze do git'a.

Kolejna sprawa dotyczy wyjścia zwracanego przez `nftables` . Jeśli set znajdzie się przykładowo w
tablicy `filter` , to jej wyjście będzie bardzo nieczytelne, bo aktualnie `nftables` zwraca również
zawartość setów. Być może w niedługim czasie ten fakt się zmieni ale póki co sety najlepiej
umieszczać w osobnych tablicach.

Ostatni problem z listami adresów w `nftables` jaki mi się rzucił w oczy, to bardzo duże zużycie
pamięci RAM podczas aplikowania reguł. Im większy jest set (zawiera sporo adresów IP), tym
naturalnie więcej RAM on potrzebuje. Rzecz w tym, że mój set waży około 7M, a `nftables` by go
załadować potrzebuje ponad 500M wolnej pamięci operacyjnej. Po załadowaniu seta wszystko wraca do
normy ale ewidentnie coś jest nie tak z obsługą tych list. Ten problem
również [zgłosiłem na ML](https://marc.info/?l=netfilter&m=154716139906192&w=2), choć do końca
chyba nie wiadomo w czym rzecz. Pogadaliśmy trochę na priv z dev'ami i byli oni w stanie
zreprodukować tę przypadłość ale póki co łatek brak.

Widać zatem, że `nftables` ma ewidentne problemy z obsługą dużych list adresów i póki co nie jest w
stanie się równać z `ipset` podpiętym do `iptables` pod względem wydajności. Oczywiście, gdy już
zestaw reguł uda się załadować, to wszystko wraca do normy ale każde jego przeładowanie będzie
wymagać sporych zasobów systemowych i jeśli nimi nie władamy, to lepiej jest zostać na starym
rozwiązaniu, przynajmniej jeszcze przez jakiś czas.
