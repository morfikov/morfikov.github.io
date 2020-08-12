---
author: Morfik
categories:
- OpenWRT
date: "2016-04-28T00:58:33Z"
date_gmt: 2016-04-27 22:58:33 +0200
published: true
status: publish
tags:
- chaos-calmer
- router
- ntp
title: Strefa czasowa (timezone) w OpenWRT
---

Tanie routery WiFi przeznaczone do użytku domowego nie zawierają w sobie [zegara czasu
rzeczywistego](https://pl.wikipedia.org/wiki/Zegar_czasu_rzeczywistego) (RTC, Real Time Clock). Taki
zegar jest implementowany w standardowych komputerach PC czy laptopach ale większość routerów go nie
posiada. Niesie to za sobą pewne komplikacje. Skąd niby router ma wiedzieć jaki mamy aktualnie czas,
skoro nie ma żadnego punktu odniesienia? Komputer bez precyzyjnie ustawionego czasu może mieć
problemy z certyfikatami SSL/TLS. Niewłaściwy czas może także utrudnić analizę pewnych zdarzeń typu
nieautoryzowane próby dostępu do sieci. Ważne jest zatem, by czas na routerze wyposażonym w firmware
OpenWRT był zawsze aktualny i w tym wpisie postaramy się zadbać o to, by strefa czasowa była
odpowiednia, oraz by router uwzględniał czas letni. Przyjrzymy się także mechanizmom aktualizacji
czasu przez protokół NTP.

<!--more-->
## Strefa czasowa i zmiany czasu

Konfigurację czasu na OpenWRT zaczynamy od określenia odpowiedniej strefy czasowej, w której
znajduje się router. W Polsce, strefa czasowa jest zawsze taka sama, tj. `Europe/Warsaw` . Niemniej
jednak, do tego dochodzi jeszcze przejście na czas letni i zimowy. Te event'y z kolei nigdy nie są
stałe i zależą od konkretnego roku. Tak czy inaczej strefa czasowa wraz z konfiguracją czasu jest do
określenia w pliku `/etc/config/system` . Poniżej znajdują się dwie linijki, które nas w tej chwili
interesują:

    config system
    ...
          option timezone 'CET-1CEST,M3.5.0/2,M10.5.0/3'
          option zonename 'Europe/Warsaw'
    ...

Opcja `zonename` określa strefę czasową. W przypadku, gdybyśmy przebywali w innej lokalizacji niż
Polska, to wtedy trzeba będzie odpowiednio zmienić wartość tej opcji. [Wszystkie strefy czasowe są
wyszczególnione tutaj](https://wiki.openwrt.org/doc/uci/system#time_zones). Problematyczne może
okazać się zrozumienie opcji `timezone` . Dlatego też poniżej jest szczegółowe wyjaśnienie całego
zapisu, który został użyty w tej opcji:

  - `CET` -- to standardowy czas środkowoeuropejski (Central European Time) , w chwili gdy nie
    obowiązuje czas letni.
  - `-1` -- to przesunięcie czasu w godzinach. Negatywna wartość oznacza 1 godzinę na wschód.
  - `CEST` -- to czas letni (Central European Summer Time).
  - `,` -- jako, że nie ma sprecyzowanej żadnej wartości między CEST oraz przecinkiem, używana jest
    standardowa wartość 1 godziny do przodu przy przechodzeniu na czas letni.
  - `M3.5.0` -- określa kiedy czas letni się rozpoczyna. W tym przypadku `M3` to miesiąc 3, czyli
    marzec. Z kolei `.5` to ostatni tydzień miesiąca, a `.0` to niedziela.
  - `/2,` -- godzina czasu lokalnego, o której następuje zmiana czasu na letni.
  - `M10.5.0` -- precyzuje kiedy czas letni się kończy. W tym przypadku `M10` to miesiąc 10, czyli
    październik. Z kolei `.5` to ostatni tydzień miesiąca, a `.0` to niedziela.
  - `/3,` -- godzina czasu lokalnego, o której następuje cofnięcie czasu.

Te powyższe ustawienia wygenerują plik `/tmp/TZ` . Istnieje jednak do niego odwołanie w postaci
linku w `/etc/TZ` .

## Czas sieciowy (NTP)

Strefa czasowa została ustawiona. To jednak nie sprawi w jakiś automagiczny sposób, że nasz router
nagle będzie potrafił ustawić sobie aktualny czas. Podczas startu systemu, zegar routera będzie
wskazywać rok 1970, czyli [początek epoki linux'a](https://pl.wikipedia.org/wiki/Czas_uniksowy)
(Unix Epoch). Przynajmniej tak to wyglądało do wypuszczenia OpenWRT Barrier Breaker, gdzie
wprowadzono skrypt `/etc/init.d/sysfixtime` . Skanuje on katalog `/etc/` w poszukiwaniu czasów
modyfikacji plików, które się w nim znajdują. Ten, który ma najnowszą datę jest brany pod uwagę i to
na jego podstawie jest ustawiany nowy czas systemowy. Oczywiście, to tylko na wypadek braku
łączności z siecią.

Rozbieżności w czasie mogą przysporzyć szereg problemów. Dlatego też czas na routerze trzeba ciągle
synchronizować z serwerami czasu dostępnymi w internecie. Możemy sprecyzować kilka takich serwerów,
maksymalnie cztery. Robimy to w pliku `/etc/config/system` . Poniżej przykład:

    config timeserver 'ntp'
          list server '0.pl.pool.ntp.org'
          list server '1.pl.pool.ntp.org'
          list server '2.pl.pool.ntp.org'
          list server '3.pl.pool.ntp.org'
          option enabled '1'
          option enable_server '0'
          option interface 'br-lan'

Jeśli opcja `enabled` jest ustawiona na `1` , wtedy router będzie synchronizował czas z tymi
serwerami określonymi w `list server` . Jeśli do tego zostanie ustawiona opcja `enable_server` ,
maszyny lokalne będą mogły przesyłać zapytania do routera (na port 123/UDP) i synchronizować czas w
oparciu o dane z routera. W ten sposób można odciążyć nieco globalne serwery czasu. Trzeba jednak
pamiętać, że jeśli nasz router nie posiada zegara RTC, to w pewnych przypadkach ustawienie tej opcji
można odczuć w dość niemiły sposób. Dlatego zasada jest jedna, jeśli router dysponuje zegarem RTC,
to aktywujemy `enable_server` i określamy interfejs w opcji `interface` . W przeciwnym wypadku
wyłączamy tę opcję zupełnie.
