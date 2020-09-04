---
author: Morfik
categories:
- Linux
date: "2016-08-09T16:04:54Z"
date_gmt: 2016-08-09 14:04:54 +0200
published: true
status: publish
tags:
- debian
- apt
- aptitude
title: 'Debian: Bezpieczne pobieranie aktualizacji (apt-transport-https)'
---

Posiadanie aktualnego systemu za sprawą regularnych aktualizacji może znacząco przyczynić się do
poprawy bezpieczeństwa naszego linux'a. Niemniej jednak, niezabezpieczony proces aktualizacji może
zdradzić pewne informacje, które mogą się okazać przydatne dla potencjalnego atakującego. Dlatego
też menadżer pakietów `apt`/`aptitude` w Debianie wyposażony jest w dodatkowe transporty
umożliwiające komunikację z serwerem repozytorium w oparciu o różne protokoły. Standardowy
protokół, którym posługują się maszyny mające na pokładzie dystrybucję Debian, to HTTP
(ewentualnie FTP). Oba z nich ślą wszelkie informacje w postaci czystego tekstu, który nadaje się do
analizy przez człowieka. Możemy jednak skorzystać z protokołu SSL/TLS i zaszyfrować proces
pobierania aktualizacji za sprawą pakietu `apt-transport-https` .

<!--more-->
## Anonimowość i poufność informacji

Przede wszystkim, trzeba rozróżnić dwie kwestie: anonimowość i poufność informacji. Można być
anonimowym i przy tym posługiwać się informacją, którą przesyłamy przez sieć w sposób jawny. Wtedy
wiadome jest jaki komunikat dana osoba przesłała, choć nie można tego delikwenta zidentyfikować.
Podobnie jest w przypadku, gdy wiadome jest z kim mamy do czynienia, choć nie wiemy nic na temat
treści komunikatu, bo ta zwykle jest zaszyfrowana.

Transport dla menadżera pakietów `apt`/`aptitude` , który może zostać zaimplementowany za sprawą
pakietu `apt-transport-https` , dotyczy tego drugiego przypadku. Potencjalny atakujący być może
będzie w stanie ustalić, że jakieś pakiety pobieramy z repozytorium (w końcu adres IP będzie znany)
ale nie będzie on w stanie stwierdzić które z nich. Podobnie w przypadku wersji danego pakietu.
Jeśli natomiast interesuje nas także ukrycie tych pozostałych jawnych informacji, możemy pokusić
się o instalację pakietu `apt-transport-tor`, który daje nam możliwość [przeprowadzania
aktualizacji systemu za pośrednictwem sieci
TOR]({{< baseurl >}}/post/debian-anonimowe-pobieranie-aktualizacji-apt-transport-tor/). W
przypadku, gdy korzystamy ze standardowego transportu przy pobieraniu pakietów trochę informacji
zdradzamy wszystkim, którzy nasłuchują:

![]({{< baseurl >}}/img/2016/08/1.wireshark-pobieranie-pakietu-http.png#huge)

## Pakiet apt-transport-https, a repozytoria

Debian oferuje sposób implementacji bezpiecznego połączenia z serwerem aktualizacji od dość dawna
ale w przeszłości oficjalne repozytoria tej dystrybucji nie wspierały protokołu SSL/TLS. Być może
było to za sprawą dość sporego obciążenia jakie niosło zaszyfrowanie całej tej komunikacji ze
wszystkimi użytkownikami, którzy ciągle pobierali pakiety. Niemniej jednak, od jakiegoś czasu Debian
to wsparcie dla protokołu SSL/TLS za sprawą pakietu `apt-transport-https` posiada i wypadałoby się
tym pakietem zainteresować.

Oficjalne repozytoria Debiana nie są również jedynymi, które mogą nam dostarczyć pakietów do
instalacji w systemie. Praktycznie każdy z nas może sobie takie [repozytorium stworzyć za pomocą
reprepro]({{< baseurl >}}/post/tworzenie-repozytorium-przy-pomocy-reprepro/) i udostępnić w nim
oprogramowanie jakie tylko chce. Oczywiście pozostaje tylko rozważenie kwestii zaufania do autora
tego przedsięwzięcia. Niemniej jednak, takie repozytorium można skonfigurować, by serwowało pakiety
używając bezpiecznego protokołu.

Szereg repozytoriów w internecie ten transport HTTPS już wykorzystuje. Przykładem może być
[Opera](https://deb.opera.com/manual.html), czy
[MEGA](https://mega.co.nz/linux/MEGAsync/Debian_9.0/). Jeśli chcielibyśmy z nich korzystać, to
musimy doinstalować pakiet `apt-transport-https` .

## Jak konfigurować repozytoria w pliku sources.list

Mając zainstalowany pakiet `apt-transport-https` , możemy przejść do konfiguracji repozytoriów w
pliku `/etc/apt/sources.list` . Jako, że wyżej wspomniałem o wsparciu tego transportu przez Operę i
MEGA, to na ich przykładzie opiszę ten prosty proces. Edytujemy zatem wspomniany plik i dodajemy w
nim poniższe wpisy:

    deb https://mega.co.nz/linux/MEGAsync/Debian_9.0/ ./
    deb https://deb.opera.com/opera-stable/ stable non-free

Generalnie rzecz biorąc, różnica polega na tym, że adres repozytorium rozpoczyna się od `https://` ,
a nie jak w standardzie od `http://` . Teraz możemy zaktualizować listy repozytoriów przy pomocy
`apt-get update` i spróbować zainstalować/zaktualizować pakiet `opera-beta` . Zanim rozpocznie się
faktyczne pobieranie pakietu, zostanie zestawiony tunel TLS, który posłuży do zaszyfrowania
wszystkich danych. Potencjalny atakujący nie będzie miał w ten sposób już możliwość podejrzeć jakie
pliki są pobierane:

![]({{< baseurl >}}/img/2016/08/2.wireshark-pobieranie-pakietu-https.png#huge)

## Kwestia security.debian.org oraz deb.debian.org/debian-security

W wydaniu stabilnym i testowym dystrybucji Debian są dostępne repozytoria zawierające poprawki
bezpieczeństwa. Te repozytoria są dostępne pod jednym z dwóch adresów: `security.debian.org` oraz
`deb.debian.org/debian-security` . W zasadzie nie ma znaczenia, z którego adresu będziemy korzystać,
bo i tak te same aktualizacje zostaną zainstalowane w naszym systemie ale tylko ten drugi adres
wspiera transport HTTPS. Poniżej znajdują się przykładowe wpisy dla stabilnego wydania Debiana:

    deb https://deb.debian.org/debian-security stable/updates main
    deb-src https://deb.debian.org/debian-security stable/updates main
