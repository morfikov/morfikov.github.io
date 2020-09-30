---
author: Morfik
categories:
- Hardware
date: "2017-04-07T20:44:35Z"
date_gmt: 2017-04-07 18:44:35 +0200
published: true
status: publish
tags:
- tp-link
- smartfon
- neffos
title: Podzespoły w smartfonie Neffos C5 MAX od TP-LINK
---

Jakiś czas temu [recenzowałem smartfon Neffos C5 MAX od
TP-LINK](/post/recenzja-smartfon-neffos-c5-max-od-tp-link/). W tamtym wpisie
praktycznie wszystko już zostało powiedziane na temat tego telefonu ale jednej rzeczy nie mogłem w
czasie pisania tamtego artykułu zamieścić. Chodzi o fotki podzespołów tego urządzenia. Na dobrą
sprawę producent podaje jedynie model SoC i nie wiadomo praktycznie nic w przypadku pozostałych
układów, no może poza pobieżnymi informacjami na temat WiFi, bluetooth czy GPS. Postanowiłem zatem
rozkręcić swojego Neffos'a C5 MAX i przekonać się co wewnątrz skrywa jego obudowa.

<!--more-->
## Jak rozebrać smartfon Neffos C5 MAX

Rozbieranie urządzeń typu smartfony nie należy do przyjemności. Chodzi o to, że większość
producentów telefonów zwykle tak projektuje te urządzenia, by nie można było ich tak łatwo
samodzielnie otworzyć. W przypadku TP-LINK nie jest aż tak źle. W zasadzie telefony tego producenta
otwiera się stosunkowo prosto, podobnie jak i jego routery. Trzeba jednak zaopatrzyć się w jakiś
mały wkrętak i kawałek cienkiego, sztywnego plastiku.

Neffos C5 MAX ma wbudowaną baterię. Zwykle przed rozebraniem smartfona, to urządzenie powinno się
wyłączyć, a jego baterię usunąć ale nie w tym przypadku, przynajmniej jeśli chodzi o usuwanie
baterii. Tutaj musimy pierw odkręcić wszystkie śrubki, które zostały odsłonięte po zdjęciu klapki.
Jedna z tych śrubek ma plombę. Z odkręceniem śrubek raczej nie powinno być problemu ale ściągnięcie
obudowy może być już kłopotliwe.

By ściągnąć obudowę w Neffos C5 MAX, musimy zahaczyć się w jednym z rogów telefonu, tuż koło ekranu
i ten plastik podciągnąć lekko do góry. Zatrzask powinien puścić. Tych zatrzasków jest jeszcze
kilka. W powstały otwór wystarczy teraz wsunąć nasz cienki kawałek plastiku i przejechać wzdłuż
krawędzi. W ten sposób wszystkie zatrzaski z jednej strony obudowy nam zeskoczą. Dalej już nie
powinno być problemów. Poniżej jest fotka mojego Neffos'a C5 MAX po zdjęciu obudowy:

![](/img/2017/04/001.podzespoly-neffos-c5-max-tp-link.jpg#huge)

## Jak usunąć baterię w Neffos C5 MAX

Nigdy wcześniej nie rozbierałem smartfonów i w zasadzie jedynie słyszałem jakiś komunikat ze strony
Apple, że gdy użytkownicy wezmą się za wymianę takich wbudowanych baterii, to one eksplodują. No
więc mamy taki smartfon i co teraz? By usunąć tę baterię, trzeba odłączyć ten konektor, który widać
tuż nad baterią po lewej stronie.

![](/img/2017/04/002.podzespoly-neffos-c5-max-tp-link.jpg#huge)

I tak o to odłączyliśmy baterię w Neffos C5 MAX, prawda, że trudne? Na szczęście nic nie wybuchło.

## Jak wyciągnąć PCB

W Neffos C5 MAX mamy w zasadzie dwa układy. Oba z nich są przykręcone śrubkami, które trzeba pierw
odkręcić. Trzeba także odłączyć cztery konektory (jeden znajduje się pod tym PCB w górnej części
smartfona).

![](/img/2017/04/003.podzespoly-neffos-c5-max-tp-link.jpg#huge)

![](/img/2017/04/004.podzespoly-neffos-c5-max-tp-link.jpg#huge)

I tak oto powinniśmy mieć już wydobyte wszystko to, co było w obudowie smartfona. Reszty chyba nie
da rady ruszyć, bo to raczej sklejony razem metal i plastik.

![](/img/2017/04/005.podzespoly-neffos-c5-max-tp-link.jpg#huge)

Może i Neffos C5 MAX ma plastikową obudowę ale za to w środku praktycznie cały jest z aluminium (to
ta szara masa na poniższej fotce):

![](/img/2017/04/006.podzespoly-neffos-c5-max-tp-link.jpg#huge)

## Podzespoły zastosowane w Neffos C5 MAX

W zasadzie na tym PCB w dolnej części smartfona nie ma za wiele rzeczy. Natomiast na tym drugim PCB
mamy już szereg czipów:

![](/img/2017/04/008.podzespoly-neffos-c5-max-tp-link.jpg#huge)

![](/img/2017/04/009.podzespoly-neffos-c5-max-tp-link.jpg#huge)

![](/img/2017/04/010.podzespoly-neffos-c5-max-tp-link.jpg#huge)

![](/img/2017/04/011.podzespoly-neffos-c5-max-tp-link.jpg#huge)

![](/img/2017/04/012.podzespoly-neffos-c5-max-tp-link.jpg#huge)

![](/img/2017/04/013.podzespoly-neffos-c5-max-tp-link.jpg#huge)

Niżej jest zapis tekstowy wszystkich tych układów widocznych na powyższych fotkach:

    +================+ +================+
    |  SKhynix       | |   MEDIATEK     |
    |  H9TQ17ABJTMC  | |        ARM     |
    |  URKUM  611Y   | |   MT6753V      |
    |                | |   1621-WAHHHH  |
    |  2MHL 4K36QAl  | |   CTTHM252     |
    +================+ +================+

    +=============+ +=============+
    |  MEDIATEK   | |  MEDIATEK   |
    |  MT6326V    | |  MT6169V    |
    |  1620-ANTH  | |  1619-AMAH  |
    |  DCMAYPSJ   | |  0TPA1U69   |
    +=============+ +=============+

    +==============+ +=============+
    |  MEDIATEK    | |  RF5210     |
    |  MT6625LN    | |  F14QTY1    |
    |  1516-AJCAL  | +=============+
    |  BAP0L811    |
    |  ACMQN7N6    |
    +==============+

Niektóre nazwy były ta malutkie, że z trudem dało radę je odczytać. Mam nadzieję, że nic nie
pomyliłem. Grunt, że wiadomo z czego Neffos C5 MAX się składa, a to już powinno znacznie ułatwić
budowę alternatywnych ROM'ów dla tego modelu smartfona. Poniżej zaś znajdują się dokumenty opisujące
układy, które znalazły zastosowanie w Neffos C5 MAX (nie wszystko udało mi się odszukać).

[H9TQ17ABJTMC UR-KUM (16 GiB eNAND i 2 GiB
LPDDR3)](http://www.datasheet4u.com/download_new.php?id=1055141)

[MT6753 (SoC)](http://www.datasheetbay.com/PDF_/download.php?id=953806)

[Trochę info o
MT6169V](https://www.chipworks.com/TOC/MediaTek_MT6169V_RF_Transceiver_CAR-1510-202_TOC.pdf)

[MT6625LN (WiFi, FM, GPS)](http://www.datasheet4u.com/download_new.php?id=960584)

Ten ostatni plik `.pdf` dotyczy MT6625L i ten czip ma WiFI 5G. Innego układu nie mogłem znaleźć i
nie wiem czy są jeszcze jakieś różnice między tymi dwoma czipami. W każdym razie zostawiam, bo nazwa
w zasadzie bardzo podobna.
