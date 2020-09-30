---
author: Morfik
categories:
- OpenWRT
date: "2015-07-13T18:12:33Z"
date_gmt: 2015-07-13 16:12:33 +0200
published: true
status: publish
tags:
- wifi
- sieć
- router
title: Interwał pakietów Beacon w sieciach WiFi
---

Przerabiając analizę pakietów sieciowych, dotarłem w końcu do sieci bezprzewodowych, a te różnią się
nieco od tych swoich przewodowych kuzynów. Generalnie rzecz biorąc nie będę tutaj opisywał samej
analizy pakietów, które sobie przemierzają eter w pobliżu naszych urządzeń WiFi, a jedynie poruszę
kwestię pakietów Beacon, które są rozsyłane przez punkty dostępowe w pewnych odstępach czasu.

<!--more-->
## Czy jest pakiet Beacon

Głównym zadaniem pakietów Beacon jest rozgłaszanie informacji o dostępności AP w okolicy. W tych
pakietach [są zawarte informacje
min.](http://www.dd-wrt.com/wiki/index.php/Advanced_wireless_settings#Beacon_Interval) na temat
identyfikatora sieci (SSID), częstotliwości nadawania (kanału), obsługiwanych prędkości, mocy
sygnału czy też wykorzystywanych zabezpieczeń. Poniżej przykład takiego pakietu złapanego via
wireshark:

![](/img/2016/08/1.interwal-beacon-wifi.png#huge)

Wyżej został zastosowany filtr, tak by były widoczne jedynie pakiety Beacon. Jeśli przyjrzymy się,
to po czasie możemy stwierdzić, że te pakiety są rozsyłane po czasie około 1 sekundy. Z reguły w
routerach nie mamy możliwości zmiany tego interwału ale jeśli dysponujemy routerem WiFi, który ma
wgrane alternatywne oprogramowanie, np. OpenWRT, wtedy jesteśmy w stanie dostosować ten parametr bez
większego problemu.

## Wady i zalety mniejszych i większych interwałów

Nasuwa się zatem pytanie: jaki interwał powinien zostać ustawiony? Zgodnie z tym co można znaleźć na
[wiki OpenWRT](http://wiki.openwrt.org/doc/uci/wireless) , mamy do wyboru wartości od 15 do 65535.
Przy czym wybraną wartość trzeba podzielić przez 1024. Zatem by otrzymać interwał 1 sekundę, to
trzeba określić wartość 1024. Domyślną zaś w przypadku OpenWRT jest 100, czyli około 0,1 sekundy.
Zatem, im większa wartość, tym większe będą odstępy pomiędzy wysyłaniem kolejnych pakietów.

Każdy pakiet Beacon zjada trochę łącza, konkretnie eteru, tak jak to mogliśmy zaobserwować na
powyższej fotce. W moim przypadku jest to 237 bajtów. Przy domyślnym interwale 0,1 sekundy, te
pakiety generują ruch około 2,5 KiB/s. Nie jest to, co prawda, dużo ale w przypadku zmniejszania
interwału do minimalnej wartości, tj. 15, da to około 17KiB/s. To są kilobajty, a nie kilobity, w
przypadku tych drugich, to by było jakieś 135 kbitów/s, także to już jest dość sporo. Jeśli teraz te
pakiety będą zjadać te 17KiB/s, to logiczne jest, że realny transfer danych siądzie nam o tę
wartość. Zatem mamy jedną zaletę ze stosowania wyższych wartości interwału.

Inną zaletą jest oszczędzanie baterii urządzeń. mobilnych. W przypadku gdy posiadają one tego typu
funkcjonalność, mogą zostać uśpione aż do czasu rozgłoszenia kolejnego pakietu Beacon.

W przypadku niższych wartości interwału, stacje klienckie mają większą szansę na przechwycenie
pakietu, przez co będą w stanie szybciej odkryć AP oraz podłączyć się do niego. No i oczywiście im
niższa wartość interwału, tym stacje klienckie potrafią dokonać lepszego wyboru co do AP, z którym
chcą się połączyć, np. gdy w grę wchodzi roaming.

Zatem jaka wartość będzie odpowiednia w naszym przypadku? Jeśli korzystamy z sieci WiFi w domu,
gdzie mamy do dyspozycji jeden router i nie przenosimy się zbytnio ze stacjami klienckimi podczas
oglądania filmów, czy grania w gry i przy tym mamy dobry sygnał na wszystkich stanowiskach pracy, to
możemy ustawić jedną z wyższych wartości. W przypadku zaś gdy mamy do dyspozycji wiele punktów
dostępowych, to dobrze jest obniżyć ten interwał. Trzeba przy tym pamiętać, że obniżenie interwału
sprawi, że urządzenia wykorzystujące technologię oszczędzania energii będą pobierać więcej prądu, bo
będzie je trzeba wybudzać o wiele częściej.

## Optymalizacja interwału

W zależności od tego w jakich warunkach przyszło nam pracować, to możemy lekko zoptymalizować sobie
parametr `beacon_int` w konfiguracji OpenWRT, który jest odpowiedzialny za interwał rozgłaszania
pakietów Beacon. Jeśli nie odpowiada nam domyślna wartość 100ms, to przechodzimy do edycji pliku
`/etc/config/wireless` , gdzie dopisujemy poniższy parametr:

    config wifi-device 'radio1'
    ...
    option beacon_int '65535'
    ...

Zapisujemy zmiany i resetujemy połączenie wpisując w terminalu polecenie `wifi` .
