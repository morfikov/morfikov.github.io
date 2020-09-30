---
author: Morfik
categories:
- Hardware
date: "2017-04-09T20:09:24Z"
date_gmt: 2017-04-09 18:09:24 +0200
published: true
status: publish
tags:
- tp-link
- smartfon
- neffos
title: Podzespoły w smartfonie Neffos C5 od TP-LINK
---

Jak można było wywnioskować z [recenzji Neffos C5 od
TP-LINK](/post/recenzja-smartfon-neffos-c5-od-tp-link/), ten smartfon to w zasadzie
niedrogie i dość przyzwoite urządzenie. Korzystając z okazji postanowiłem zrobić kilka dodatkowych
zdjęć tego telefonu, z tym, że tym razem fotki obrazują to, co było wewnątrz obudowy. Jeśli
posiadacie Neffos'a C5 i jesteście ciekawi co to urządzenie skrywa pod maską, to w tym wpisie
znajdziecie odpowiedzi na dręczące was pytania.

<!--more-->
## Jak rozebrać smartfon Neffos C5

Przed przystąpieniem do rozbierania smartfona wyłączmy go pierw i usuńmy z niego baterię. Z
rozebraniem Neffos'a C5 nie ma zbytnio większego problemu. Po ściągnięciu klapki z tylnej części
obudowy mamy całą masę malutkich śrubek, które trzeba odkręcić. Jedna ze śrubek ma plombę. Po
wykręceniu ostatniej śrubki trzeba uporać się jeszcze z zatrzaskami, które trzymają dwie części
obudowy razem. To zadanie również nie jest jakoś specjalnie trudne, choć trzeba lekkiej wprawy.

Ta część obudowy, którą zamierzamy ściągnąć, składa się w zasadzie z dwóch części. Trzeba zatem je
ściągnąć osobno zahaczając paznokciem czy innym cienkim przedmiotem w rogu smartfona tuż nad
ekranem. W powstały otwór trzeba następnie wsadzić cienki kawałek plastiku, np. kartę płatniczą i
przesunąć wzdłuż krawędzi. Wszystkie zatrzaski powinny nam w ten sposób łatwo zeskoczyć umożliwiając
nam tym samym ściągnięcie obudowy.

![](/img/2017/04/001.neffos-c5-tp-link-podzespoly.jpg#huge)

## Jak wyciągnąć PCB

Jak widzimy na powyższej fotce, PCB w tym telefonie jest jednoczęściowy i zlokalizowany w zasadzie
przy górnej, dolnej i lewej części obudowy. By wyciągnąć tę płytkę, trzeba pierw odkręcić kilka
śrubek oraz odłączyć trzy konektory. W dolnej części obudowy jest także ulokowany kawałek
wibratora, z którego są wyprowadzone i przylutowane na PCB dwa przewody. Ten element jest także
przyklejony do obudowy:

![](/img/2017/04/002.neffos-c5-tp-link-podzespoly.jpg#huge)

## Podzespoły zastosowane w Neffos C5

Mając wyciągnięty PCB, możemy przyjrzeć się nieco bliżej samej płytce oraz bez większego problemu
odczytać jakie modele układów zostały zastosowane w Neffos C5.

![](/img/2017/04/003.neffos-c5-tp-link-podzespoly.jpg#huge)

![](/img/2017/04/004.neffos-c5-tp-link-podzespoly.jpg#huge)

![](/img/2017/04/005.neffos-c5-tp-link-podzespoly.jpg#huge)

![](/img/2017/04/006.neffos-c5-tp-link-podzespoly.jpg#huge)

![](/img/2017/04/007.neffos-c5-tp-link-podzespoly.jpg#huge)

![](/img/2017/04/008.neffos-c5-tp-link-podzespoly.jpg#huge)

![](/img/2017/04/009.neffos-c5-tp-link-podzespoly.jpg#huge)

![](/img/2017/04/010.neffos-c5-tp-link-podzespoly.jpg#huge)

Niżej jest zapis tekstowy wszystkich tych układów widocznych na powyższych fotkach:

    +==============+ +=============+
    |   MEDIATEK   | |  77910-11   |
    |    MT6159V   | |  1843602.1  |
    |   1618-AMTH  | |  1603   MX  |
    |   BTPAIU88   | +=============+
    +==============+

    +==============+ +================+
    |   MEDIATEK   | |   MEDIATEK     |
    |    MT6328V   | |        ARM     |
    |   1613-AEAH  | |   MT6735V      |
    |   D6020240   | |   1617-WACAH   |
    +==============+ |   CCMKTAMA     |
                     +================+

    +==============+ +=============+
    |     SEC 610  | |  MEDIATEK   |
    |        B419  | |  MT6625LN   |
    |  KMQ820013M  | |  1516-AMSL  |
    |  S40P1CA3C   | |  ETPA0A88   |
    +==============+ |  ATC-WHS4   |
                     +=============+

    +==============+
    |  GooDix      |
    |  GT915L      |
    |  1625-T17F   |
    |  15BG3400    |
    |  P0W047xxxE  |
    +==============+

No to teraz już w zasadzie wiadomo z czego składa się Neffos C5 i może w niedługim czasie pojawi się
jakiś alternatywny ROM pokroju LineageOS na ten telefon. Poniżej jeszcze znajduje się dokumentacja
techniczna, którą udało mi się znaleźć w Google:

[SKY77910-11 (GSM/LTE)](http://www.skyworksinc.com/Product/3063/SKY77910-11)

[MT6328 (GSM/LTE)](http://www.datasheetbay.com/PDF_/download.php?id=1067077)

[MT6735 (WiSoC)](http://www.datasheetbay.com/PDF_/download.php?id=925384)

[MT6625LN (WiFi, FM, GPS)](http://www.datasheet4u.com/download_new.php?id=960584)

Ten ostatni plik `.pdf` dotyczy MT6625L i ten czip ma WiFI 5G. Innego układu nie mogłem znaleźć i
nie wiem czy są jeszcze jakieś różnice między tymi dwoma czipami. W każdym razie zostawiam, bo nazwa
w zasadzie bardzo podobna.
