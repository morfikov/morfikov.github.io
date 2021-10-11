---
author: Morfik
categories:
- Linux
date: "2016-08-06T09:13:20Z"
date_gmt: 2016-08-06 07:13:20 +0200
published: true
status: publish
tags:
- apache2
- iptables
- ipset
- dos
- ddos
GHissueID: 548
title: 'Apache2: Moduł evasive, ipset i iptables (anty DOS/DDOS)'
---

Apache2 ma kilka ciekawych modułów, które mogą uchronić nasz serwer www przed atakami DOS i DDOS.
Jednym z nich jest moduł `evasive` . Nie jest on jednak oficjalnym modułem i brak o nim
jakiejkolwiek wzmianki w oficjalnej dokumentacji na stronie Apache2. Niemniej jednak, jest to bardzo
prosty moduł składający się dosłownie z kilku dyrektyw, które są w stanie zablokować zapytania o
zasoby serwera w przypadku, gdy zostanie przekroczony pewien ustalony przez nas limit. Dodatkowo,
ten moduł może współgrać z filtrem `iptables` oraz `ipset` , dając nam możliwość wygodnego
blokowania uporczywych klientów na poziomie pakietów sieciowych.

<!--more-->
## Pozyskanie modułu evasive

Jako, że moduł `evasive` nie jest częścią projektu Apache2, to trzeba go pozyskać z zewnętrznego
źródła. Debian dysponuje nawet odpowiednim pakietem, który wystarczy doinstalować. Chodzi o
`libapache2-mod-evasive` . Po instalacji, moduł trzeba aktywować w poniższy sposób:

    # a2enmod evasive
    # systemctl restart apache2

Dokumentacja modułu znajduje się na [blogu autora][1]. Warto także zajrzeć do pliku
`/usr/share/doc/libapache2-mod-evasive/README.gz` .

## Zasada działania mechanizmu blokującego

W chwili nadejścia zapytania do serwera www, jest ono przetwarzane przez moduł `evasive` . Na
początku jest sprawdzany adres IP klienta, który wysłał zapytanie. Ten adres jest porównywany z
tymi, które trafiły na tymczasową czarną listę. W przypadku, gdy adres nie zostanie odnaleziony na
liście, moduł utworzy klucz w tablicy hashów, którym będzie powiązanie adresu IP z żądanym adresem
URI. Następnie ta tablica jest przeszukiwana w celu ustalenia czy jakiś host odpytał dany zasób (lub
szereg obiektów) kilka razy w ciągu określonego interwału (domyślnie 1s). Gdy tak się zdarzy w
istocie, adres IP tego klienta trafi na czarną listę przez pewien okres czasu. Wszystkie następne
zapytania od tego klienta będą kwitowane kodem błędu 403 (Forbidden). W przypadku, gdy klient nie
zaprzestanie próby podłączenia, czas bana ulegnie odświeżeniu. Te komunikaty 403 zjadają zasoby
procesora i łącza. Dlatego też powinniśmy zaprzęgnąć `iptables` do pracy.

Każde zdarzenie zablokowania adresu IP, zostawia ślad w logu systemowym, który wygląda mniej więcej
tak jak ten komunikat poniżej:

    mod_evasive[31034]: Blacklisting address 66.102.9.105: possible DoS attack.

## Dyrektywy modułu evasive

Cała konfiguracja dla modułu `evasive` jest umieszczona w pliku
`/etc/apache2/mods-enabled/evasive.conf` . Przejdźmy zatem do edycji tego pliku, by zobaczyć co tam
się znajduje:

    <IfModule mod_evasive20.c>
        DOSHashTableSize    3079
        DOSPageCount        2
        DOSSiteCount        50
        DOSPageInterval     1
        DOSSiteInterval     1
        DOSBlockingPeriod   10

        #DOSEmailNotify      you@yourdomain.com
        #DOSSystemCommand    "su - someuser -c '/sbin/... %s ...'"
        #DOSLogDir           "/var/log/mod_evasive"
    </IfModule>

Im większy rozmiar tablicy hashów określimy w dyrektywie `DOSHashTableSize` , tym więcej pamięci ona
będzie utylizować ale też lepiej będzie działał cały mechanizm, bo nie trzeba usuwać rekordów z tej
tablicy przed dodaniem nowych. Każdy proces serwera Apache2 (domyślnie 5-100), ma osobną tablicę
hashów. Nie zaleca się zwiększania tego parametru, no chyba, że nasz serwer jest sporych rozmiarów i
realizuje naprawdę wiele zapytań. Jeśli wartość, którą określimy nie będzie liczbą pierwszą, to
zostanie ona automatycznie wybrana w oparciu o tę poniższą listę:

    53         97         193       389       769
    1543       3079       6151      12289     24593
    49157      98317      196613    393241    786433
    1572869    3145739    6291469   12582917  25165843
    50331653   100663319  201326611 402653189 805306457
    1610612741 3221225473 4294967291

Dyrektywy `DOSPageInterval` i `DOSPageCount` ustanawiają limit żądań w stosunku do konkretnego
zasobu na serwerze (URI). W tym przypadku, każdy adres IP może wykonać maksymalnie dwa żądania w
ciągu 1 sekundy. Z kolei zaś dyrektywy `DOSSiteInterval` oraz `DOSSiteCount` precyzują limit dla
całego serwisu. Zatem każdy adres IP może wysłać maksymalnie 50 zapytań pod różne zasoby serwera.
Po przekroczeniu tych limitów, zapytania z tego adresu IP zostaną zablokowane na okres zdefiniowany
w `DOSBlockingPeriod` , czyli 10 sekund.

Dyrektywa `DOSEmailNotify` jest w stanie nas powiadomić drogą mailową, za każdym razem, gdy jakiś
adres IP zostanie zablokowany. Z kolei `DOSLogDir` określa tymczasowy katalog wykorzystywany przez
mechanizm blokujący. Standardowo jest folder `/tmp/` , a pliki tworzone w nim mają takie uprawnienia
na jakich operuje serwer Apache2. Domyślnie są to `www-data:www-data` . Za każdym razem, gdy jakiś
adres IP wyjdzie poza ustanowione limity, to zostanie w tym katalogu utworzony plik o nazwie
`dos-1.2.3.4` , gdzie `1.2.3.4` , to zablokowany adres IP. Te pliki nie są usuwane, czego efektem
jest blokada mechanizmu obronnego w stosunku do uprzednio zablokowanego adresu IP. By znów
zablokować dany adres, ten plik musi być pierw usunięty (o tym będzie dalej).

## Połączenie modułu evasive z iptables i ipset

Pozostała nam do omówienia ostatnia z dyrektyw, które widnieją w pliku
`/etc/apache2/mods-enabled/evasive.conf` . Jest to `DOSSystemCommand` , czyli polecenie, które
zostanie uruchomione w przypadku, gdy jakiś klient przekroczy limit zapytań. Najprostszym i zarazem
najefektywniejszym rozwiązaniem jest stworzenie [listy dla ipset'a][2] i dynamiczne dodawanie do
niej adresów IP. Wymagane jest jednak dodatkowe oprogramowanie w postaci pakietu `ipset` , które
musimy zainstalować na debianie. Będzie nam też potrzebne `sudo` , bo inaczej nie dodamy adresów do
listy, gdyż serwer Apache2 nie działa domyślnie z prawami administratora systemu.

Następnie musimy stworzyć listę, na którą będą trafiać pakiety:

    # ipset create evasive hash:net family inet maxelem 5000 --timeout 60

Teraz podpinamy listę pod filtr `iptables` :

    # iptables -t raw -N evasive-in
    # iptables -t raw -N evasive-out

    # iptables -t raw -A PREROUTING -j evasive-in
    # iptables -t raw -A OUTPUT -j evasive-out

    # iptables -t raw -A evasive-in -m set --match-set evasive src -j DROP
    # iptables -t raw -A evasive-out -m set --match-set evasive dst -j DROP

Set podpięty pod firewall. Wszystko czego nam teraz potrzeba, to polecenie, które doda adresy. To
polecenie musimy również uwzględnić w `sudo` . Edytujemy zatem konfigurację `sudo` za pomocą
`visudo` i dodajemy tam poniższe wpisy:

    Defaults!/sbin/ipset !requiretty
    Host_Alias HOSTY = localhost,domena.com
    www-data    HOSTY = (root) NOPASSWD: /sbin/ipset add evasive * -exist timeout 60

Teraz to samo polecenie, wpisujemy do pliku `/etc/apache2/mods-enabled/evasive.conf` w dyrektywie
`DOSSystemCommand` , z tym, że korzystamy z `sudo` :

    DOSSystemCommand "sudo ipset add evasive %s -exist timeout 60 && sleep 1 && rm /tmp/dos-%s"

Wyżej pojawił się znaczek `%s` . Odpowiada on za adres IP, który moduł `evasive` zwróci po
przekroczeniu limitu zapytań. Po znaku `&&` mamy drugie polecenie, które spowolni wykonanie
trzeciego polecenia. Chodzi o to, że klient może wysłać kilka zapytań i one wszystkie zostaną
przetworzone przez moduł. W efekcie pojawi się bardzo ekstensywny log, którego nie chcemy. Dlatego
właśnie czekamy 1s, aż mechanizm zapory zablokuje pakiety. Wtedy blokadę można zdjąć (usunąć plik),
bo już żadne pakiety od tego klienta nie będą do nas docierać. Od tej chwili blokować będzie go
`iptables` . Wartość 60, to 60 sekund, przez taki okres czasu adres IP będzie widniał na liście
ipset'a. Jeśli klient nie dokona żadnych akcji w tym czasie, to adres z tej listy zostanie usunięty.
Jeśli jednak, będzie próbował nawiązać połączenie, to czas ulegnie odświeżeniu.

## Test ataku DOS/DDOS

Przydałoby się teraz przetestować jak ten mechanizm anty DOS/DDOS działa. Możemy to zrobić na dwa
sposoby. Pierwszy z nich, to wykorzystanie skryptu dostarczanego z pakietem
`libapache2-mod-evasive` . Znajduje się on pod
`/usr/share/doc/libapache2-mod-evasive/examples/test.pl` . Wystarczy go przekopiować na maszynę
kliencką i nieco dostosować. Generalnie chodzi o zmianę adresu:

    ...
    PeerAddr=> "1.2.3.4:80");
    ...

Teraz ten skrypt można uruchomić kilka razy i po chwili nasz adres IP powinien trafić na listę
ipset'a. Do wglądu przez `ipset list` .

Drugą opcją jest obniżenie wartości dyrektyw `DOSPageCount` i `DOSSiteCount` do 1 oraz podbicie
wartości w dyrektywach `DOSPageInterval` i `DOSSiteInterval` do 5. Wtedy dość energiczne odwiedzanie
strony via `curl` wyzwoli mechanizm blokujący.

Jeśli adres IP nie jest dodawany do listy ipset'a, to prawdopodobnie mamy pozostałości po
wcześniejszych atakach DOS/DDOS, tj. pliki `dos-*` w katalogu `/tmp/` . Musimy je usunąć usunąć.


[1]: https://www.zdziarski.com/blog/?page_id=442
[2]: http://ipset.netfilter.org/
