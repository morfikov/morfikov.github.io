---
author: Morfik
categories:
- Linux
date: "2016-08-07T13:06:03Z"
date_gmt: 2016-08-07 11:06:03 +0200
published: true
status: publish
tags:
- debian
- apt
- anonimowość
- tor
- repozytorium
title: 'Debian: Anonimowe pobieranie aktualizacji (apt-transport-tor)'
---

Dystrybucja linux'a Debian oferuje możliwość pobierania pakietów `.deb` za pomocą sieci TOR. W ten
sposób jesteśmy w stanie ukryć nieco informacji na temat zainstalowanego w naszym systemie
oprogramowania. Jakby nie patrzeć, aplikacje mają pełno dziur i nie wszystkie z tych programików są
łatane natychmiast po opublikowaniu podatności. Z chwilą dokonywania aktualizacji systemu,
potencjalny atakujący może dowiedzieć się zatem z jakich programów korzystamy, wliczając w to ich
wersje. Znając te dane, można ocenić czy system posiada jakieś błędy. By zaimplementować w
menadżerze pakietów `apt`/`aptitude` możliwość korzystania z [sieci TOR][1], musimy posiadać w
systemie skonfigurowanego klienta TOR oraz zainstalować pakiet `apt-transport-tor` . W tym artykule
postaramy się skonfigurować ten cały mechanizm TOR'owych aktualizacji.

<!--more-->
## Jak ustalić z jakiego oprogramowania korzysta system

Zakładając, że jesteśmy podłączeni do sieci, w której ktoś może węszyć, przydałoby się ocenić jakie
informacje zdradzamy takiej osobie w procesie aktualizacji naszego linux'a. Najprościej jest to
uczynić instalując sniffer sieciowy i pobierając przykładowy pakiet z repozytorium Debiana. Poniżej
jest przykładowy zrzut tego co złapał `wireshark` :

![](/img/2016/08/1.wireshark-debian-informacje-o-pakiecie.png#huge)

Wyżej widzimy, że zdradzamy nie tylko nazwę pakietu, który zamierzamy pobrać, ale również jego
wersję, numer rewizji, architekturę systemu, rodzaj dystrybucji linux'a oraz też natywny język
administratora systemu. Także trochę tych informacji wędruje po kablach i nie zawsze chcielibyśmy,
aby inni użytkownicy sieci mogli je dojrzeć i przeanalizować.

## Konfiguracja TOR'a

By móc skorzystać z wspieranego przez `apt`/`aptitude` transportu TOR, musimy pierw zainstalować
sobie i skonfigurować klienta sieci TOR. W Debianie jest on dostępny w pakiecie `tor` . Z instalacją
pakietu nie powinno być problemów, zatem przejdźmy od razu do konfiguracji, która znajduje się w
pliku `/etc/tor/torrc` . Na dobrą sprawę, to klient działa OOTB ale dobrze jest w tym pliku ustawić
ten poniższy parametr:

    SocksPort 127.0.0.1:9050

Upewnijmy się jeszcze, że usługa `tor` działa w tle i możemy przejść do instalacji pakietu
`apt-transport-tor` .

## Pakiet apt-transport-tor

Menadżer pakietów `apt`/`aptitude` posiada kilka transportów, które może wykorzystać podczas
pobierania pakietów z repozytorium Debiana. Jednym z nich jest [apt-transport-tor][2] implementujący
obsługę sieci TOR. Po pomyślnym skonfigurowaniu klienta TOR, wystarczy ten pakiet zainstalować i
`apt`/`aptitude` jest już gotowy do pracy. Potrzebne nam są tylko źródła repozytoriów, które musimy
dodać do pliku `/etc/apt/sources.list` .

Źródła repozytoriów możemy dodać na dwa sposoby. Ten poniższy sposób automatycznie dostosuje
lokalizację mirroru repozytorium w zależności od exit node TOR'a:

    deb tor+http://http.debian.net/debian sid main

Alternatywnie możemy określić URL w taki sposób, by wskazywał na [zasób Debiana w sieć TOR][3]:

    deb tor+http://vwakviie2ienjx6t.onion/debian sid main

Teraz już wystarczy wydać w terminalu polecenie `apt-get update` , a listy repozytoriów powinny
polecieć przez sieć TOR:

![](/img/2016/08/2.deian-apt-transport-tor-ruch-przez-siec-tor.png#huge)

Wyżej określiliśmy wpisy z `tor+http://` . Zatem ruch wewnątrz sieci TOR do mirroru Debiana nie
będzie szyfrowany. Wszystkie mechanizmy walidacji pakietów w `apt`/`aptitude` obowiązują tak jak w
przypadku korzystania ze zwykłego mirroru i nie musimy się obawiać, że ktoś nam zmieni pakiety.
Jeśli spróbuje, to menadżer pakietów wyrzuci stosowną informację.


[1]: https://www.torproject.org/
[2]: https://github.com/diocles/apt-transport-tor
[3]: https://onion.debian.org/
