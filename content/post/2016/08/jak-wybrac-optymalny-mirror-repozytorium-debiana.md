---
author: Morfik
categories:
- Linux
date: "2016-08-07T15:15:20Z"
date_gmt: 2016-08-07 13:15:20 +0200
published: true
status: publish
tags:
- debian
- apt/aptitude
title: Jak wybrać optymalny mirror repozytorium Debiana
---

Debian to dość stara i rozbudowana dystrybucja linux'a, którą można spotkać praktycznie w każdym
zakątku naszego globu. Dziesiątki tysięcy pakietów dostępne w oficjalnych repozytoriach tylko
czekają aż je pobierzemy i zainstalujemy w swoim systemie. Problem zaczyna się jednak w momencie,
gdy wielu użytkowników w tym samym czasie zaczyna pobierać pakiety i to z tego samego serwera. Wtedy
aktualizacja Debiana może trwać dłużej niż zazwyczaj. By zaadresować ten problem, developerzy tej
dystrybucji stawiają serwery lustrzane (mirror) w różnych częściach świata i rozładowują w ten
sposób ruch, który by powędrował do głównego serwera. Spora część krajów ma kilka własnych
mirror'ów ale ich jakość może czasami zostawić wiele do życzenia. Co w przypadku, gdy taki mirror,
z którego my korzystamy, ulegnie awarii? Trzeba będzie poddać edycji plik `/etc/apt/sources.list` i
zmienić adres repozytorium przez dostosowanie w nim części odpowiedzialnej za lokalizację, np.
`ftp.pl` czy `ftp.us` . Istnieje jednak sposób, który dostosuje lokalizację serwera lustrzanego
automatycznie, a my już nie będziemy musieli sobie głowy zawracać edycją wspomnianego wyżej pliku.

Projekt, o którym traktuje poniższy wpis, nie jest już rozwijany przez Debiana. Więcej info
[tutaj](https://wiki.debian.org/DebianGeoMirror).

<!--more-->
## Projekt Debian HTTP-Redirector

[Projekt Debian HTTP-Redirector](http://httpredir.debian.org/) jest nam w stanie zaoferować
automatyczny wybór najlepszego repozytorium w oparciu o kilka czynników. Brana jest pod uwagę
geograficzna lokalizacja sieci użytkownika oraz mirror'u, do którego taki user miałby najbliżej.
Duże znaczenie ma też klasa adresów IP użytkownika i mirror'u oraz dostępność i świeżość
(aktualność pakietów) samych serwerów lustrzanych. W oparciu o te informacje oraz też o kilka
pomniejszych rzeczy, Debian HTTP-Redirector wybierze nam najlepszy z dostępnych serwerów.

[Pod tym linkiem](http://httpredir.debian.org/demo.html) znajduje się demonstracja całego mechanizmu
wyboru mirror'u. Wystarczy wejść pod wskazany adres i po chwili powinniśmy zobaczyć kilka informacji
wraz z sugerowanym serwerem. Wygląda to mniej więcej tak jak na tej fotce
poniżej:

![]({{< baseurl >}}/img/2016/08/1.debian-http-redirector-mirror-aktualizacja-systemu.png)

Zatem w tym przypadku zalecany mirror to `http://ftp.vectranet.pl/debian/` . Gdybym teraz zmienił
położenie geograficzne, np. wyjechał za granicę, czy nawet do innego miasta gdzieś dalej, to ten
widoczny wyżej mirror uległby zmianie. Oczywiście nadal można używać innych serwerów, np. głównego,
ale skoro mamy możliwość zoptymalizowania doboru serwera lustrzanego, z którego będziemy pobierać
pakiety przy aktualizacji systemu, to czemu z niej nie skorzystać?

## Jak dostosować mirror za pomocą Debian HTTP-Redirector

Cały ten projekt Debian HTTP-Redirector nie miałby większego sensu, gdybyśmy musieli ręcznie w
dalszym ciągu uzupełniać plik `/etc/apt/sources.list` . No ale przecież jakieś wpisy tam trzeba
umieścić. Skąd ten mechanizm ma wiedzieć, że chcemy go w ogóle używać? Musimy przepisać standardowe
adresy repozytoriów, tak by uwzględniały `http://httpredir.debian.org/debian` . Reszta wpisu
repozytorium zostaje taka sama:

Gałąź stabilna:

    deb http://httpredir.debian.org/debian/ stable main non-free contrib
    deb-src http://httpredir.debian.org/debian/ stable main non-free contrib

Gałąź testowa:

    deb http://httpredir.debian.org/debian/ testing main non-free contrib
    deb-src http://httpredir.debian.org/debian/ testing main non-free contrib

Gałąź niestabilna:

    deb http://httpredir.debian.org/debian/ unstable main non-free contrib
    deb-src http://httpredir.debian.org/debian/ unstable main non-free contrib

Gałąź eksperymentalna:

    deb http://httpredir.debian.org/debian/ experimental main contrib non-free
    deb-src http://httpredir.debian.org/debian/ experimental main contrib non-free

Oczywiście, zamiast `stable` , `testing` , `unstable` czy `experimental` możemy podać zwykłe nazwy
kodowe danego wydania, np. `jessie` .

Ten powyższy mechanizm stosuje się także do backports'ów:

    deb http://httpredir.debian.org/debian jessie-backports main
    deb-src http://httpredir.debian.org/debian jessie-backports main

W przypadku repozytorium z poprawkami bezpieczeństwa ( `http://security.debian.org/` ) , pakiety
powinny być pobierane bezpośrednio z głównego serwera, a nie z mirror'ów. Dlatego też jeśli
korzystamy z wydania stabilnego lub testowego i chcemy używać Debian HTTP-Redirector'a, to zadbajmy
o to, by to powyższe repozytorium zostało wyłączone spod działania tego mechanizmu.
