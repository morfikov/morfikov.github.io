---
author: Morfik
categories:
- Hardware
date: "2016-08-26T20:48:33Z"
date_gmt: 2016-08-26 18:48:33 +0200
published: true
status: publish
tags:
- wifi
- recenzja
- tp-link
- antena
GHissueID: 547
title: 'Recenzja: Antena TL-ANT2408C (8dBi, 2,4GHz) od TP-LINK'
---

Jako użytkownik alternatywnych systemów operacyjnych wiem, że nie łatwo o sprzęt, który po
podłączeniu do komputera działa OOTB. Niemniej jednak, na rynku jest sporo urządzeń, które są w
stanie działać pod linux'em nawet dość przyzwoicie, z tym, że trzeba pierw się naprawdę wysilić, by
je znaleźć. Tak było w przypadku [adaptera WiFi
TL-WN722N](http://www.tp-link.com.pl/products/details/TL-WN722N.html) od TP-LINK, który jest już ze
mną kilka lat. Nie miałem z nim problemów na swoim Debianie i praktycznie nie mam mu nic do
zarzucenia. No może za wyjątkiem bardzo przeciętnego zasięgu, choć ta karta dysponuje zewnętrzną
anteną 4 dBi. Postanowiłem zatem rozejrzeć się za nieco większymi antenami w celu wyeliminowania
problemów z zasięgiem. W ofercie TP-LINK'a była [antena
TL-ANT2408C](http://www.tp-link.com.pl/products/details/cat-5691_TL-ANT2408C.html) (8 dBi, 2,4 GHz),
to pomyślałem, że przetestują ją i sprawdzę czy problem słabego zasięgu zostanie w końcu
wyeliminowany.

<!--more-->
## Zawartość opakowania

Pudełko jest dość duże, bo i sama antena do najmniejszych nie należy. Jej długość to około 30 cm.
Poniżej fotki zawartości
opakowania.

![](/img/2016/08/1.antena-TL-ANT2408C-opakowanie.jpg#medium)

![](/img/2016/08/2.antena-TL-ANT2408C-zawartosc-pudelka.jpg#huge)

Poza samą anteną, w zestawie znajduje się też podstawka, do której można antenę przykręcić:

![](/img/2016/08/3.antena-TL-ANT2408C-podstawka.jpg#huge)

![](/img/2016/08/4.antena-TL-ANT2408C-podstawka-magnes.jpg#medium)

Ciekawym rozwiązaniem jest zastosowanie w podstawce magnesu, za pomocą którego jesteśmy w stanie
przyczepić antenę do metalowych przedmiotów, np. do obudowy naszego dekstopa. Sama podstawka nie
styka się bezpośrednio z powierzchnią metalową, tylko opiera się na czterech gumowych nóżkach. W
efekcie nie musimy się bać o jakiekolwiek zarysowania. Magnes jest naprawdę solidny i trzeba trochę
siły włożyć, by tę podstawkę od obudowy oderwać.

Przewód, który jest widoczny niżej to RG174 o długości około 130 cm i jest zakończony złączem
RP-SMA:

![](/img/2016/08/5.antena-TL-ANT2408C-podstawka-przewod-rp-sma.jpg#medium)

Złącze RP-SMA jest stosowane w większości routerów WiFi i adapterów/kart bezprzewodowych. Zatem bez
problemu można tę antenę stosować z tymi urządzeniami.

## Antena TL-ANT2408C i adapter WiFi TL-WN722N

Zatem wiemy mniej więcej jak wygląda antena TL-ANT2408C. Poniżej jest zaś fotka mojego adaptera WiFi
TL-WN722N z podczepioną nową anteną:

![](/img/2016/08/6.antena-TL-ANT2408C-adapter-wifi-TL-WN722N.jpg#huge)

Ten zestaw wygląda naprawdę przyzwoicie. Nic tylko podłączyć teraz adapter do portu USB komputera i
cieszyć się lepszym zasięgiem. Jest tylko jeden problem. Podobnie jak większość użytkowników, nie do
końca orientowałem się w temacie anten. Jeszcze parę dni temu myślałem sobie, że taka widoczna
wizualna różnica tych dwóch anten na pewno przełoży się na poprawę jakości sygnału w znacznym
stopniu. Tak się jednak nie stało.

W różnych sklepach internetowych, czy też w tych cenowych porównywarkach, można dostrzec całą masę
komentarzy na temat znacznej poprawy jakości sygnału/zasięgu za pomocą anteny TL-ANT2408C. Niemniej
jednak, gdy ja poddałem testom tę antenę, to okazało się, że praktycznie nie ma żadnej różnicy.
Przypadek? No niekoniecznie.

## Regulacje prawne i moc transmisyjna

Próbując rozwiązać tę zagadkę, natrafiłem na szereg informacji kierujących mnie w stronę mocy
transmisyjnej adaptera WiFi TL-WN722N i zysku energetycznego anteny TL-ANT2408C. Postanowiłem zatem
sprawdzić jaką mocą transmisyjną dysponuje moja karta WiFi. Na stronie TP-LINK'a widnieje [EIRP
(Effective Isotropical Radiated Power)](https://pl.wikipedia.org/wiki/EIRP) \<20 dBm, a to się
przekłada z kolei na około 100 mW. Za dużo to raczej nie jest i więcej nie będzie.

Okazuje się bowiem, że regulacje prawne w Polsce nakładają limity na producentów sprzętu WiFi dla
przeciętnego Kowalskiego w aspekcie mocy transmisyjnej. Maksymalna moc nie może zatem przekraczać
tych 100 mW. Z kolei na antenie widniej zapis "Admitted Power: 1 W". Zatem, by w pełni wykorzystać
potencjał tej anteny, potrzebna byłaby karta, która jest w stanie nadawać z mocą 1 W, czyli 30 dBm.

Na stronie TP-LINK'a widnieje zaś informacja, że "Należy pamiętać, iż zwiększenie siły sygnału
zależy w znaczącej mierze od używanego routera lub karty sieciowej. Antena nie może transmitować
sygnału o mocy większej niż moc danego urządzenia". Zatem mój adapter TL-WN722N jest za cienki dla
anteny TL-ANT2408C. Zresztą nie tylko mój, bo w Polsce legalnie nie powinno się dać kupić urządzenia
zdolnego wypromieniować moc 1 W.

Próbowałem w ramach testów zwiększyć moc transmisyjną tego adaptera WiFi ale nie udało mi się tego
zrobić. Zapytałem zatem TP-LINK'a min. o to czy da radę tę moc jakoś podbić. W odpowiedzi uzyskałem
zaś: "W polskiej dystrybucji nie występują karty sieciowe z możliwością zmiany mocy transmisyjnej".
Zatem nic z tego nie będzie. No to na co mi taka antena, skoro nie ma pod nią odpowiednich urządzeń,
przynajmniej nie w Polsce?

## Zysk energetyczny anteny

Antena TL-ANT2408C jest niby dookólna, czyli ma nadawać we wszystkie strony mniej więcej z tą samą
mocą. Niemniej jednak, na opakowaniu jest napisane (gain 8 dBi), czyli zysk energetyczny anteny ma
wynosić 8 dBi. Różnica w zysku między tymi dwiema antenami (dużą i mała z zestawu adaptera WiFi)
wynosi zatem 4 dBi. Sprawdźmy jak wyglądają statystyki sygnału z odległości około 4 metrów przez
niezbyt grubą ścianę. Poniższa fotka obrazuje jakość i siłę sygnału, gdy do adaptera WiFi nie ma
podłączonej żadnej anteny:

![](/img/2016/08/7.antena-TL-ANT2408C-adapter-wifi-TL-WN722N-test-sygnal.png#big)

Niżej zaś mamy fotkę, która zestawia jakość i siłę sygnału po doczepieniu anteny małej (z lewej) i
dużej (z prawej):

![](/img/2016/08/8.antena-TL-ANT2408C-adapter-wifi-TL-WN722N-test-sygnal.png#huge)

Jak widzimy, nie ma zbytnio żadnej różnicy, a biorąc pod uwagę restrykcje prawne, takiej różnicy być
nie może. Chodzi generalnie o to, że producent sprzętu by móc go legalnie sprzedawać w Polsce, musi
tak dobrać parametry (moc transmisyjna urządzeń, zysk energetyczny anteny, rodzaj i długość
przewodu, złącza), by się zmieścić poniżej 20 dBm.

W przypadku adaptera TL-WN722N mamy dołączoną antenę 4 dBi, nie mamy żadnego przewodu i mamy tylko
jedno złącze. Natomiast w przypadku anteny TL-ANT2408C mamy przewód RG174 o długości około 130 cm i
dwa złącza (jedno do adaptera, drugie do podstawki). [Pod tym linkiem znajduje się ciekawy
artykuł](http://www.dipol.com.pl/poradnik_instalatora_wlan_bib86.htm) traktujący min. o tłumieniu
sygnału za sprawą różnych czynników. Z kolei [tutaj](http://www.tp-link.com.pl/faq-3.html) jest
trochę informacji na temat parametrów samych anten.

Postanowiłem zatem poszukać ile wynosi tłumienie sygnału za sprawą tego przewodu RG174 o długości
około 130 cm w paśmie 2,4 GHz, bo częstotliwość również ma znaczenie. Zajrzałem do [pierwszego
lepszego sklepu](https://sklep.avt.pl/przewod-koncentryczny-rg174-50om-kab0031.html) zwróconego mi
przez gógla. Tam mamy poniższą informację:

    tłumienie [dB/100m]: 30MHz - 17,6; 50MHz - 21,1; 100MHz - 28,2; 146MHz - 41,6; 440MHz - 80; 1000MHz - 96; 2400MHz - 240

Jako, że jest to karta WiFi pracująca w paśmie 2,4 GHz, to interesuje nas ta ostatnia wartość.
Przewód ma 130 cm, zatem sygnał wytłumi się o około 3,12 dB. Dorzucając do tego jeszcze jedno
ekstra złącze, musimy doliczyć do strat kolejne 0,5 dB. W sumie zysk energetyczny anteny TL-ANT2408C
jest zbliżony do tej małej antenki dostarczanej w zestawie z adapterem TL-WN722N. Dlatego nie widać
praktycznie żadnej różnicy w sile i jakości sygnału.

Jeśli jednak spojrzymy sobie na te anteny, to bez wahania możemy powiedzieć, że różnią się one
znacząco od siebie. By również odczuć różnicę w jakości i sile sygnału, trzeba by zrezygnować z
przewodu/podstawki i wpiąć antenę TL-ANT2408C bezpośrednio pod adapter. W takim przypadku
zaobserwujemy poprawę mocy sygnału o około 2-3 dB, czyli sygnał będzie dwukrotnie mocniejszy.

## Czy zakup anteny TL-ANT2408C jest pozbawiony sensu

Jeśli nie widać żadnego realnego zysku, to po co mielibyśmy rozważać zakup anteny TL-ANT2408C?
Niektóre adaptery WiFi są przecie w połowie ceny tej anteny. Niewątpliwą zaletą anteny TL-ANT2408C
jest fakt, że nie jest ona doczepiona bezpośrednio do kart bezprzewodowych. W efekcie możemy
manipulować położeniem samej anteny na obszarze o promieniu prawie 1,5 metra.

Jakość sygnału w pewnych miejscach naszego domu może się diametralnie różnić. Metalowe przedmioty są
w stanie uniemożliwić dotarcie sygnału do kart bezprzewodowych, a przecie nasz komputer ma zwykle
stalową obudowę. Poza tym, cała masa sprzętu AGD/RTV (w tym też i komputery) może nam zwyczajnie
zakłócać sygnał w tym konkretnym punkcie mieszkania. Przestawianie komputera, by poprawić nieco
jakość sygnału może być dość problematyczne. Może i ta antena TL-ANT2408C nie odbiega zbytnio
sumarycznymi parametrami od tej małej antenki dołączanej do adaptera TL-WN722N ale w wielu
sytuacjach jest w stanie ona znacząco poprawić jakość odbieranego sygnału, choć w moim przypadku nie
szło zauważyć widocznej różnicy.
