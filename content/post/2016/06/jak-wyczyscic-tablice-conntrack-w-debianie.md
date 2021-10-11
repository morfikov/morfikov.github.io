---
author: Morfik
categories:
- Linux
date: "2016-06-05T12:34:55Z"
date_gmt: 2016-06-05 10:34:55 +0200
published: true
status: publish
tags:
- bezpieczeństwo
- iptables
- debian
GHissueID: 359
title: Jak wyczyścić tablicę conntrack'a w debianie
---

Sporo użytkowników lunux'a, zwłaszcza dystrybucji debian, korzysta z własnych skryptów firewall'a
aplikujących reguły `iptables` . Tego typu rozwiązanie ma jednak swoje wady i zalety. Niewątpliwie
do zalet można zaliczyć brak dodatkowego oprogramowania obsługującego zaporę sieciową. Jeśli chodzi
zaś o wady, to niestety cały skrypt trzeba sobie dobrze przemyśleć przed zaaplikowaniem. Ludzie
często zapominają tutaj o śledzeniu połączeń przez kernel. To właśnie na podstawie wpisów w
`/proc/net/ip_conntrack` lub `/proc/net/nf_conntrack` system wie, które pakiety należy na zaporze
przepuścić, a które zablokować. Jeśli teraz dodajemy reguły do filtra `iptables` , to nowa polityka
zapory nie będzie odnosić się do tych nawiązanych już połączeń, które są określone w tablicy
conntrack'a. By się upewnić, że tego typu scenariusz nigdy nas nie spotka, musimy tę tablicę
opróżnić.

<!--more-->
## Czy tablica conntrack'a powinna być czyszczona

Przede wszystkim, większość zapór sieciowych w linux'ch operuje na stanach połączeń, przykładowo
NEW, ESTABLISHED, INVALID. W momencie, gdy jakiś host z sieci próbuje nawiązać połączenie z naszym
serwerem, kernel umieszcza wpis w tablicy conntrack'a. Poniżej mamy dwa
    wpisy:

    tcp      6 60 SYN_RECV src=192.168.10.100 dst=192.168.10.10 sport=59334 dport=443 src=192.168.10.10 dst=192.168.10.100 sport=443 dport=59334
    tcp      6 432000 ESTABLISHED src=192.168.10.100 dst=192.168.10.10 sport=59334 dport=443 src=192.168.10.10 dst=192.168.10.100 sport=443 dport=59334 [ASSURED]

Mamy tutaj połączenie na port 443/tcp. W protokole TCP połączenia mają kilka faz. Wyżej widzimy, że
serwer otrzymał pakiet synchronizacyjny ( `SYN_RECV` ) i po chwili połączenie zostało ustanowione (
`ESTABLISHED` ). Na tym porcie chcemy teraz założyć jakąś politykę, przykładowo, dostęp do serwera
www zostanie odmówiony w określonych godzinach. Jeśli dodalibyśmy stosowne reguły do skryptu
`iptables` i przeładowali zaporę, to tylko nowe żądania będą zrzucane. A co w przypadku osób
mających aktywne sesje, np. pobierają one pliki z tego serwera? System nie odmówi im dostępu i nie
przerwie transferu danych. Jeśli chcemy mieć pewność, że cały ruch do serwera przejdzie przez nowe
reguły firewall'a, to musimy tę tablicę conntrack'a wyczyścić.

## Czyszczenie tablicy conntrack'a

W różnych linux'ach czyszczenie tablicy może przebiegać nieco inaczej. Zależy to od możliwości
zapisu plików `/proc/net/ip_conntrack` lub `/proc/net/nf_conntrack` . W OpenWRT mamy możliwość
zapisu tych plików przy pomocy `echo f` . Niemniej jednak w debianie ta opcja odpada:

    # ls -al /proc/net/*_conntrack
    -r--r----- 1 root root 0 2016-06-05 11:46:07 /proc/net/ip_conntrack
    -r--r----- 1 root root 0 2016-06-05 11:46:07 /proc/net/nf_conntrack

Nie wiem czemu w debianie nie da rady zapisać bezpośrednio tych plików ale w tej dystrybucji [mamy
narzędzia, które umożliwiają operowanie na tablicy
conntrack'a](http://conntrack-tools.netfilter.org/manual.html). Te narzędzia zawarte są w pakiecie
`conntrack` . Zainstalujmy zatem je sobie.

Ilość aktualnie śledzonych połączeń możemy odczytać poniższym poleceniem:

    # conntrack -C
    43

Liczba `43` wskazuje na ilość wpisów w tablicy i nie jest to ilość aktualnych połączeń. By opróżnić
teraz całą tablicę śledzonych połączeń, wpisujemy w terminalu to poniższe polecenie:

    # conntrack -F
    conntrack v1.4.3 (conntrack-tools): connection tracking table has been emptied.

Jeśli ponownie spróbujemy odczytać ilość wpisów w tablicy, to powinniśmy zobaczyć tam liczbę `0` :

    # conntrack -C
    0

Oznacza to, że wszystkie sesje zostały ubite, a ewentualne pakiety z nimi związane zostaną oznaczone
jako INVALID i zrzucone na zaporze sieciowej.
