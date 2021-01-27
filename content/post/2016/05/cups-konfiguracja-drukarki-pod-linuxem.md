---
author: Morfik
categories:
- Linux
date: "2016-05-11T21:45:38Z"
date_gmt: 2016-05-11 19:45:38 +0200
published: true
status: publish
tags:
- drukarka
- cups
- debian
title: CUPS, czyli konfiguracja drukarki pod linux'em
---

Ja jestem tym szczęśliwcem, który posiada takie fajne urządzenie zwane drukarką. Bardzo umila ono
życie pod warunkiem, że działa jak należy, a z tym różnie bywa, zwłaszcza pod linux'em. W debianie
do obsługi drukarek używa się [CUPS (Common Unix Printing System)][1] i można powiedzieć, że radzi
on sobie z tym zadaniem całkiem nieźle, przynajmniej w przypadku mojej drukarki. Drukarka, o której
mowa, to dość stary model, konkretnie jest to EPSON Stylus Color 760. Potrzebne są jej odpowiednie
sterowniki, które są dostępne w repozytorium debiana. Jedyne czego potrzeba do szczęścia, to
skonfigurowanie demona `cupsd` , który będzie zarządzał tą drukarką. W tym artykule postaramy się
skonfigurować to powyższe urządzenie.

<!--more-->
## Oprogramowanie do obsługi CUPS

Sterowniki do drukarki EPSON Stylus Color 760 znajdują się w pakiecie `printer-driver-gutenprint` .
Dodatkowo, musimy zainstalować pakiet `cups` oraz kilka dodatkowych rzeczy, by móc bez przeszkód
konfigurować drukarkę. Wszystkie pakiety są uwzględnione w poniższym poleceniu:

    # aptitude install cups ghostscript colord printer-driver-gutenprint libpaper-utils

W tym przypadku nie będziemy mieli do czynienia z lokalną drukarką, która jest podpięta do
standardowego komputera PC. Zamiast tego będzie to drukarka sieciowa udostępniana przez router za
sprawą demona `p910nd` . Niemniej jednak, proces postępowania w przypadku konfiguracji obu drukarek
jest dokładnie taki sam, tylko trzeba odpowiednio dobrać lokalizację urządzenia. Drukarką będziemy
zarządzać przez przeglądarkę internetową, za pomocą panelu webowego. Po zainstalowaniu tych
powyższych rzeczy, demon `cupsd` powinien zostać aktywowany, a my mieć możliwość dostania się do
panela administracyjnego wpisują w polu adresu przeglądarki `:631` . Można tutaj
przeprowadzić dosłownie wszystkie prace administracyjne, począwszy od dodania drukarki, skończywszy
na szczegółowej jej konfiguracji.

## Konfiguracja drukarki via CUPS

Pliki konfiguracyjne dla demona `cupsd` znajdują się w katalogu `/etc/cups/` . Jeśli chcemy
dostosować usługę drukowania do własnych potrzeb rezygnując przy tym ze standardowej konfiguracji
demona, powinniśmy zainteresować się plikami `/etc/cups/cupsd.conf` oraz
`/etc/cups/cups-files.conf` . W przypadku tej drukarki, nie musimy nic ruszać, bo po dodaniu jej
działa ona OOTB. Prawdopodobnie większość drukarek z CUPS działa w ten sposób. Także przechodzimy
na podany wyżej adres. Naszym oczom powinien ukazać się poniższy panel:

![](/img/2016/05/1.cups-drukarka-linux-debian-serwer-wydruku.png#huge)

Klikamy na zakładkę `Administration` i dodajemy nową drukarkę klikając w `Add Printer` :

![](/img/2016/05/2.cups-drukarka-linux-debian-serwer-wydruku.png#huge)

Jeśli konfigurujemy drukarkę sieciową, to z dostępnych opcji wybieramy `AppSocket/HP JetDirect` . W
przypadku lokalnych drukarek, powinny one być widoczne automatycznie:

![](/img/2016/05/3.cups-drukarka-linux-debian-serwer-wydruku.png#huge)

W polu `Connection` określamy adres drukarki sieciowej. W tym przypadku
`socket://192.168.1.1:9100` , jako, że na routerze nasłuchuje na porcie 9100 demon `p910nd` :

![](/img/2016/05/4.cups-drukarka-linux-debian-serwer-wydruku.png#huge)

Teraz opisujemy drukarkę. Podajemy jej nazwę, krótki opis oraz lokalizację:

![](/img/2016/05/5.cups-drukarka-linux-debian-serwer-wydruku.png#big)

Następnie wybieramy producenta drukarki:

![](/img/2016/05/6.cups-drukarka-linux-debian-serwer-wydruku.png#huge)

Oraz jej model:

![](/img/2016/05/7.cups-drukarka-linux-debian-serwer-wydruku.png#huge)

Dostosowujemy także domyślne ustawienia wydruku (można je zmienić w dowolnym czasie):

![](/img/2016/05/8.cups-drukarka-linux-debian-serwer-wydruku.png#huge)

Po chwili drukarka powinna zostać dodana:

![](/img/2016/05/9.cups-drukarka-linux-debian-serwer-wydruku.png#huge)

Na samym dole mamy informacje, że póki co, drukarka jest w stanie bezczynnym, bo nie przetwarza
żadnych prac. Przetestujmy ją zatem, by sprawdzić czy nam coś wydrukuje. Dla przykładu odpalmy
pierwszy z brzega edytor tekstu i z jego menu wybierzmy `Print` . Powinno nam się pojawić to
poniższe okienko:

![](/img/2016/05/10.cups-drukarka-linux-debian-serwer-wydruku.png#big)

Widzimy, że drukarka została rozpoznana. Po ikonce z lewej strony możemy stwierdzić, że jest to
drukarka sieciowa. Mamy również informacje na temat jej nazwy oraz lokalizacji. Gdy klikniemy na
drukarkę, rozpocznie się próba połączenia z demonem, który ją obsługuje na routerze. Na powyższej
fotce widzimy też, że trwa pobieranie informacji o drukarce. Jeśli połączenie zostanie ustanowione,
informacje zostaną pobrane i będziemy mogli wydrukować sobie stronę testową:

![](/img/2016/05/11.cups-drukarka-linux-debian-serwer-wydruku.png#huge)

Powyżej widzimy, że plik został zakolejkowany i jest aktualnie przetwarzany przez drukarkę. Po
chwili zadanie zostanie ukończone i drukarka wypluje zadrukowaną kartkę:

![](/img/2016/05/12.cups-drukarka-linux-debian-serwer-wydruku.png#huge)


[1]: https://en.wikipedia.org/wiki/CUPS
