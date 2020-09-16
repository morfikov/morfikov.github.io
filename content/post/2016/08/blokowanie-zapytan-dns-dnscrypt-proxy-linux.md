---
author: Morfik
categories:
- Linux
date:    2016-08-25 20:04:33 +0200
lastmod: 2016-08-25 20:04:33 +0200
published: true
status: publish
tags:
- dns
- resolver
- ipv6
- debian
title: Blokowanie zapytań DNS z dnscrypt-proxy na linux'ie
---

[Narzędzie dnscrypt-proxy][1] począwszy od [wersji 1.7.0][2] ma domyślnie włączoną obsługę
wtyczek. W standardzie nie ma ich dużo, bo jedynie trzy ale mogą one się okazać dla pewnych osób
bardzo użyteczne. Dzięki tym pluginom możemy, np. zablokować rozwiązywanie nazw w protokole IPv6 na
wypadek, gdyby ten protokół nie był wspierany w naszej sieci domowej czy też u naszego ISP. Możemy
także zdefiniować sobie adresy/domeny, które powinny zostać zablokowane i w efekcie użytkownicy nie
będą w stanie odwiedzić tych miejsc w internecie. Jest także wtyczka, która może nam pomóc zalogować
zapytania DNS. Jak widać, całkiem przyzwoite są te dodatki. W tym wpisie przyjrzymy się nieco bliżej
konfiguracji poszczególnych wtyczek dla `dnscrypt-proxy` .

<!--more-->
## Blokowanie rozwiązywania nazw w protokole IPv6

Niby już od 20 lat protokół IPv6 jest wśród nas, a w dalszym ciągu cała masa providerów
internetowych nie ma zaimplementowanej obsługi tego protokołu. W efekcie aplikacje, które chcą
komunikować się z siecią, mogą próbować wysyłać zapytania o rekordy AAAA (IPv6) do serwerów DNS.
Jako, że te zapytania są zupełnie zbędne przy takiej konfiguracji sieci, to tylko spowalniają
połączenie. Poniżej jest przykładowa sytuacja:

![]({{< baseurl >}}/img/2016/08/1.wireshark-domena-rekord-aaaa-dns.png#huge)

Jak widać, do serwera DNS zostały wysłane dwa zapytania o adres IP serwisu YT. Serwer DNS zwrócił
dwie odpowiedzi zawierające dwa adresy, po jednym dla protokołu IPv4 i IPv6.

W przypadku, gdy mamy [lokalnie skonfigurowaną usługę dnscrypt-proxy][3], wszystkie te zapytania
mogą zostać przechwycone. `dnscrypt-proxy` natychmiast udzieli na nie pustej odpowiedzi, a
aplikacja nie będzie musiała czekać na odpowiedź z serwera DNS.

Plugin odpowiedzialny za blokowanie rekordów AAAA to `ldns-aaaa-blocking` . By go włączyć, musimy
poddać edycji plik `.service` i przerobić nieco dyrektywę `ExecStart` . Poniżej przykład:

    [Service]
    ...
    ExecStart=/usr/sbin/dnscrypt-proxy \
        --local-address=127.0.2.1:5353 \
        --resolver-name=cisco \
        --daemonize \
        --loglevel=4 \
        --plugin=libdcplugin_example_ldns_aaaa_blocking.so

Teraz możemy przeładować konfigurację systemd i zrestartować `dnscrypt-proxy` :

    # systemctl daemon-reload
    # systemctl restart dnscrypt-proxy

Plugin powinien zostać załadowany. Możemy także przetestować, czy zapytania DNS o rekordy AAAA są
obsługiwane
lokalnie:

![]({{< baseurl >}}/img/2016/08/2.wireshark-domena-rekord-aaaa-dns-dnscrypt-proxy.png#huge)

Jak widzimy wyżej, ponownie aplikacja wysłała dwa zapytania o IP serwisu YT ale rekord AAAA nie
zawiera adresu IPv6. Jeśli dodatkowo popatrzymy na znacznik czasu, to dostrzeżemy, że czas między
zapytaniem i odpowiedzią wynosi około 0,001 s. W przypadku, gdy aplikacja odpytywała serwer DNS, ten
czas wynosił około 0,13 s, także różnica jest dość znaczna.

## Logowanie zapytań DNS

`dnscrypt-proxy` umożliwia także logowanie zapytań DNS, które przez niego są przepuszczane. Nie jest
to jakaś wymyślna funkcjonalność, bo z taką samą możemy spotkać się, np w `dnsmasq` . Niemniej
jednak, nie wszyscy chcą implementować sobie cache DNS w swoich linux'ach i wolą słać zapytania o
domeny bezpośrednio do serwera DNS.

Jeśli chcemy tylko zalogować te zapytania, to możemy zrezygnować z `dnsmasq` i włączyć w
`dnscrypt-proxy` plugin `logging` . W tym przypadku, również musimy poddać edycji plik `.service` i
przerobić w nim dyrektywę `ExecStart` . Poniżej przykład:

    [Service]
    ...
    ExecStart=/usr/sbin/dnscrypt-proxy \
        --local-address=127.0.2.1:5353 \
        --resolver-name=cisco \
        --daemonize \
        --loglevel=4 \
        --plugin=libdcplugin_example_logging.so,/var/log/domains.log

Tutaj parametr `--plugin` przyjmuje dwa argumenty. Pierwszym z nich jest nazwa samej wtyczki. Drugim
zaś jest plik do zapisu wszystkich logowanych zapytań DNS.

Tworzymy jeszcze plik `/var/log/domains.log` i nadajemy mu odpowiedniego właściciela:

    # touch /var/log/domains.log
    # chown _dnscrypt-proxy /var/log/domains.log

Przeładowujemy demona systemd i restartujemy `dnscrypt-proxy` :

    # systemctl daemon-reload
    # systemctl restart dnscrypt-proxy

W `/var/log/domains.log` powinny zacząć gromadzić się wpisy podobne do tego poniżej:

    youtube.com     [A]
    youtube.com     [AAAA]
    46.209.58.216.in-addr.arpa      [0x0C]

## Blokowanie domen i adresów IP

Ostatnia z trzech wtyczek umożliwia nam określenie domen lub/i adresów IP, które mają być blokowane
przy rozwiązywaniu nazw DNS. W ten sposób jesteśmy w stanie zablokować, np. serwery reklamowe, czy
też serwisy społecznościowe. Jedyne czego nam trzeba to listy stron zakazanych. Zanim jednak taką
listę sobie zbudujemy, musimy dostosować dyrektywę `ExecStart` w pliku usługi systemd i włączyć
plugin `ldns-blocking` . Poniżej przykład:

    [Service]
    ...
    ExecStart=/usr/sbin/dnscrypt-proxy \
        --local-address=127.0.2.1:5353 \
        --resolver-name=cisco \
        --daemonize \
        --loglevel=4 \
        --plugin=libdcplugin_example_ldns-blocking.so,--domains=/etc/peerblock/block-domain.txt,--ips=/etc/peerblock/block-ip.txt

W tym przypadku parametr `--plugin` może przyjąć dwa lub trzy argumenty. Nazwa wtyczki jest
pierwszym z nich. Natomiast w zależności od tego czy chcemy blokować domeny czy adresy IP, to
korzystamy odpowiednio z argumentu `--domains=` lub `--ips=` . Jeśli zaś chcemy blokować zarówno
adresy IP i domeny, to definiujemy oba te argumenty równocześnie.

Trzeba pamiętać, że w tym przypadku, wszystkie trzy argumenty muszą być oddzielone przecinkami od
siebie bez stosowania spacji. Dodatkowo, pliki nie mogą być puste lub zawierać jedynie samych
komentarzy (linijek zaczynających się od znaku `#` ).

Przeładowujemy demona systemd i restartujemy `dnscrypt-proxy` :

    # systemctl daemon-reload
    # systemctl restart dnscrypt-proxy

Rekordy, które chcemy zablokować dodajemy w plikach określonych przez `--domains=` i `--ips=` w
formie jeden na linię, przykładowo:

    *.youtube.com
    *.facebook.com
    ads.*

lub:

    1.2.3.4
    2.3.4.5

Domeny mogą zawierać `*` w celu dopasowaniu szeregu adresów.

Spróbujmy teraz odwiedzić serwis YT w przeglądarce. Powinien nas przywitać ten poniższy
błąd:

![]({{< baseurl >}}/img/2016/08/3.dnscrypt-proxy-blokowanie-domen-facebook-youtube.png#huge)

Wszystkie inne domeny powinny działać bez zarzutu.


[1]: https://dnscrypt.org/
[2]: https://github.com/jedisct1/dnscrypt-proxy/releases
[3]: {{< baseurl >}}/post/dnscrypt-proxy-czyli-szyfrowanie-zapytan-dns/
