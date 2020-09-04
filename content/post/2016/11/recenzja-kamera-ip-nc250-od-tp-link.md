---
author: Morfik
categories:
- Hardware
date: "2016-11-13T20:43:02Z"
date_gmt: 2016-11-13 19:43:02 +0100
published: true
status: publish
tags:
- recenzja
- tp-link
- kamera
- aparat
title: 'Recenzja: Kamera IP NC250 od TP-LINK'
---

Ludzkie oko ma to do siebie, że nie widzi całej masy szczegółów w postrzeganym obrazie. Nawet jeśli
część z nas jest w stanie "widzieć więcej" niż inni, to i tak nie zawsze możemy znaleźć się w danym
miejscu i czasie, by te konkretne zdarzenia być w stanie zarejestrować. Subiektywna opinia kogoś,
kto próbuje nam jedynie opisać dane wydarzenie, nie zawsze oddaje sendo sytuacji, a i tak przecież
są rzeczy, które wolelibyśmy zobaczyć sami na własne oczy. Dlatego właśnie dobrze jest mieć jakiś
rejestrator w postaci kamery, który będzie w stanie uwiecznić obraz w formie niezmienionej. Taki
materiał video możemy później sobie odtworzyć w dowolnym czasie i raczej żaden szczegół nie umknie
naszej uwadze, zwłaszcza w przypadku włamań do monitorowanych przez nas pomieszczeń. Jako, że mam na
wyposażeniu [kamerę IP NC250](http://www.tp-link.com.pl/products/details/cat-19_NC250.html) od
TP-LINK, to postanowiłem napisać o miej kilka słów w kontekście wykorzystania tego sprzętu pod
linux'em.

<!--more-->
## Zawartość opakowania kamery NC250

Zacznijmy może od pudełka i jego zawartości. Wewnątrz opakowania mamy oczywiście samą kamerę NC250.
Jest też kilka dodatkowych rzeczy.

![]({{< baseurl >}}/img/2016/11/001.nc250-kamera-tp-link-opakowanie.jpg#big)

![]({{< baseurl >}}/img/2016/11/002.nc250-kamera-tp-link-opakowanie-zawartosc.jpg#big)

![]({{< baseurl >}}/img/2016/11/003.nc250-kamera-tp-link-opakowanie-zawartosc.jpg#big)

Do zestawu jest dołączony przewód ethernet RJ-45 kategorii 5 (CAT5) o długości 1,2 m:

![]({{< baseurl >}}/img/2016/11/004.nc250-kamera-tp-link-rj-45.jpg#big)

Oraz zasilacz 9V/0,6A:

![]({{< baseurl >}}/img/2016/11/005.nc250-kamera-tp-link-zasilacz.jpg#big)

![]({{< baseurl >}}/img/2016/11/006.nc250-kamera-tp-link-zasilacz.jpg#big)

Przewód na zasilaczu ma około 1,5 m. Dla tych użytkowników, którzy uważają, że może być to ździebko
za mało, to w zestawie jest także dołączony przedłużacz, który ma około 2,5 m:

![]({{< baseurl >}}/img/2016/11/007.nc250-kamera-tp-link-przedluzacz.jpg#big)

Kamera generalnie składa się z dwóch części, tej właściwej oraz podstawki:

![]({{< baseurl >}}/img/2016/11/008.nc250-kamera-tp-link.jpg#big)

![]({{< baseurl >}}/img/2016/11/009.nc250-kamera-tp-link-podstawka.jpg#big)

Podstawkę trzeba dokręcić do kamery, z tym, że w miejscu łączenia podstawki z kamerą mamy także
regulator kulowy:

![]({{< baseurl >}}/img/2016/11/010.nc250-kamera-tp-link-regulator.jpg#big)

![]({{< baseurl >}}/img/2016/11/011.nc250-kamera-tp-link-regulator.jpg#big)

Niezbyt mi takie rozwiązanie przypadło do gustu. Praktycznie każda regulacja położenia kamery
sprawia, że podstawka się odkręca. Mocniejsze dokręcenie podstawki uniemożliwia z kolei regulację.
Także moim zdaniem niezbyt udane rozwiązanie. Z tym, że jeśli ktoś ustawi jedno położenie kamery, to
raczej nie będzie mu ten fakt przeszkadzał.

![]({{< baseurl >}}/img/2016/11/012.nc250-kamera-tp-link-regulator.jpg#big)

![]({{< baseurl >}}/img/2016/11/013.nc250-kamera-tp-link-regulator.jpg#big)

![]({{< baseurl >}}/img/2016/11/014.nc250-kamera-tp-link-z-podstawka.jpg#big)

Kamerę można postawić na biurku/szafce i od razu można z niej zacząć korzystać. Podstawka nie ma
jednak żadnych gumowych nóżek przez co może nam jeździć dość mocno po blacie. Jeśli chcielibyśmy
kamerę umiejscowić w pewnym określonym punkcie, to zawsze możemy ją przykleić dołączoną do zestawu
taśmą. W przypadku ścian i innych tego typu miejsc, lepszym rozwiązaniem będzie stałe mocowanie przy
pomocy śrubek. W podstawce są już nawiercone odpowiednie otwory, choć w zestawie brak śrubek.

![]({{< baseurl >}}/img/2016/11/015.nc250-kamera-tp-link-montaz.jpg#big)

Za to jak zwykle nie mogło zabraknąć całej sterty papierzysk. Dobrze, że jest tutaj również skrócona
instrukcja obsługi w języku polskim, choć moim zdaniem nie powinno być problemów z obsługą tej
kamery.

## Specyfikacja kamery NC250

No to wiemy mniej więcej jak kamera NC250 wygląda. Sprawdźmy teraz jej parametry techniczne. Na
tylnym panelu kamery NC250 poza gniazdem zasilania znajduje się także port RJ-45 o przepustowości
100 mbit/s.

![]({{< baseurl >}}/img/2016/11/016.nc250-kamera-tp-link-gniazda.jpg#big)

Niżej zaś ulokowany jest przycisk RESET/WPS, dzięki któremu możemy łatwo zresetować kamerę do
ustawień fabrycznych oraz możemy także nawiązać połączenie z siecią WiFi.

![]({{< baseurl >}}/img/2016/11/017.nc250-kamera-tp-link-wps.jpg#big)

Kamera NC250 jest w stanie łączyć się bezprzewodowo w standardzie 802.11n do 300 mbit/s w paśmie 2,4
GHz. Warto też odnotować, że sama kamera jest w stanie także robić za ekstender sieciowy, przez co
może poszerzyć zasięg naszej domowej sieci WiFi. Klienta sieci bezprzewodowej jak i sam ekstender
można naturalnie wyłączyć w panelu administracyjnym.

![]({{< baseurl >}}/img/2016/11/018.nc250-kamera-tp-link-wifi.png#huge)

![]({{< baseurl >}}/img/2016/11/019.nc250-kamera-tp-link-ekstender-wifi.png#huge)

To co jest niewątpliwą zaletą tej kamery, to obsługa trybu nocnego. NC250 posiada bowiem 6 diod
podczerwieni zdolnych oświetlić dość przyzwoicie obszar do 5 metrów w absolutnej ciemności:

![]({{< baseurl >}}/img/2016/11/020.nc250-kamera-tp-link-diody-podczerwien.jpg#medium)

Kamera rejestruje obraz w formacie h.264 w rozdzielczości 1280x720 (1 mpix) przy 10, 15, 20, 25 i 30
FPS. Oprogramowanie kamery pozwala także na zoom cyfrowy 2x, 3x i 4x. Pole widzenia (FOV) kamery
jest na poziomie 64 stopni.

Nie znam się za bardzo na obiektywach, dlatego też zamieszczam jedynie informacje, które można
znaleźć na stronie TP-LINK, tj. F: 2,8 f: 3,85 mm, przetwornik CMOS o wielkości 1/4 cala. Może
komuś się te cyferki przydadzą, mi one niewiele mówią. Generalnie mogę jedynie ocenić jakość
obrazu, a ta jest dość przeciętna. Z rozpoznaniem ludzkiej twarzy z kilku-kilkunastu metrów w dzień
i w nocy raczej nie powinniśmy mieć problemu. Zatem do prostego monitoringu pomieszczeń ten sprzęt
wystarczy ale raczej nic poza tym.

NC250 ma także wbudowany mikrofon i jest w stanie rejestrować dźwięk. Jakość dźwięku pozostawia
jednak wiele do życzenia, choć sam fakt możliwości zarejestrowania odgłosów można wykorzystać i
przerobić kamerę na czujnik dźwięku wysyłający notyfikacje przez email lub FTP. Szkoda, że zabrakło
głośnika, przez co jesteśmy w stanie jedynie odbierać dźwięk, bez możliwości komunikacji w drugą
stronę.

Zużycie prądu waha się w granicach 3,2 W przy braku podłączonych klientów. Gdy transfer obrazu
odbywa się po WiFi, wtedy zużycie energii wzrasta do około 3,6 W. Natomiast przy połączeniu
przewodowym i kompletnym wyłączeniu WiFi, ilość pobieranej energii przez kamerę oscyluje w granicach
2 W w stanie spoczynku i 2,2 W podczas transferu danych.

## Kamera NC250 pod linux

Po podpięciu kamery NC250 do sieci zostanie jej przydzielony adres IP. Na tym adresie IP będzie
dostępny panel administracyjny. To co się rzuca w oczy niemal natychmiast, to brak możliwość
podejrzenia obrazu z kamery pod linux'em z poziomu przeglądarki. Wygląda na to, że potrzebny jest
jakiś niestandardowy plugin, który nie ma wersji na linux'a.

![]({{< baseurl >}}/img/2016/11/021.nc250-kamera-tp-link-plugin-web.png#huge)

Niemniej jednak, część z widocznych wyżej przycisków działa, tylko co nam po nich, skoro nie mamy
podglądu? Licząc od lewej strony, to nie działa nam przycisk od robienia fotek i nagrywania obrazu.
Działa za to odbijanie obrazu w pionie i poziomie, jak i przełączanie kamery w tryb
dzienny/nocny/auto. Nie działa zoom ale jest możliwość podbicia poziomu dźwięków wyłapywanych przez
kamerę. Damy radę też manipulować ustawieniami jasności, kontrastu i nasycenia obrazu.

Może i nie damy rady uzyskać podglądu bezpośrednio z panelu administracyjnego ale możemy go uzyskać
w nieco inny sposób. Generalnie rzecz biorąc, ta kamera ma otwartych kilka portów:

    80/tcp   open  http
    554/tcp  open  rtsp
    1935/tcp open  rtmp
    2020/tcp open  xinupageserver
    8080/tcp open  http-proxy

Na porcie 80 jest panel administracyjny, a na pozostałych? Eksperymentując i [podpierając się tym
linkiem](https://github.com/reald/nc220), udało mi się uzyskać obraz z kamery w poniższy sposób:

    $ cvlc rtsp://admin:admin@192.168.1.159:554/h264_vga.sdp   # 10 FPS
    $ cvlc --network-caching=0 http://admin:YWRtaW4=@192.168.1.159:8080/stream/video/mjpeg
    $ mpv --no-cache http://admin:YWRtaW4=@192.168.1.159:8080/stream/video/h264?resolution=HD

Audio z kolei można wydobyć w poniższy sposób (16000Hz mono 1ch s16):

    $ mpv http://admin:YWRtaW4=@192.168.1.159:8080/stream/audio/mpegmp2

Dostęp do stream'ów wymaga podania loginu i hasła, tych samych używanych przy logowaniu do panelu
administracyjnego. W niektórych poleceniach, hasło trzeba zakodować w base64. Standardowo jest
ustawione hasło `admin` , co po zakodowaniu daje `YWRtaW4=` .

Zapisanie pojedynczej klatki obrazu z kamery do pliku można przeprowadzić przy pomocy poniższego
polecenia.

    $ wget --user=admin --password=YWRtaW4= http://192.168.1.159:8080/stream/snapshot.jpg

Nie wiem jak ta kamera się sprawuje w połączeniu z [ZoneMinder](https://zoneminder.com/) ale z tego
co zdążyłem wyczytać na stronie projektu, to każda kamera IP zdolna nadawać strumień MJPEG powinna
bez problemu działać z tym oprogramowaniem. Na pewno za jakiś czas przebadam sobie ten soft i dodam
tutaj stosowną adnotację. (FIXME)

## Czujnik dźwięku

Jako, że kamera NC250 ma wbudowany mikrofon, to rejestruje dźwięk. Potrafi także rejestrować poziom
tego dźwięku w danej chwili. Można zatem zaprogramować system tej kamery tak, by po przekroczeniu
pewnego ustalonego poziomu dźwięku wysłał nam powiadomienie o aktywności.

![]({{< baseurl >}}/img/2016/11/022.nc250-kamera-tp-link-dzwiek-detekcja.png#huge)

Takie powiadomienie może się odbyć drogą mailową jak i za pośrednictwem protokołu FTP. W obu
przypadkach po przekroczeniu dopuszczalnego poziomu dźwięku zostaną wysłane cztery fotografie. W
przypadku wybrania opcji powiadamiania przez email, wysyłane jest także poniższe info:

![]({{< baseurl >}}/img/2016/11/023.nc250-kamera-tp-link-mail.png#huge)

Mając podgląd na fotkach w wiadomości email, łatwo możemy ustalić co się dzieje w monitorowanym
pomieszczeniu. Jeśli nawet na fotkach nic nie zostało uchwycone, to taka notyfikacja zawsze może dać
nam sygnał do włączenia podglądu z kamery w celu sprawdzenia czy aby wszystko jest w należytym
porządku.

## Czujnik ruchu

Czujnik dźwięku nie zawsze może się sprawdzić. Jeśli złodziej będzie bardzo cicho się zachowywał, to
jest wielce prawdopodobne, że go ten mechanizm nie wychwyci i nie zarejestruje. Dlatego też lepszym
rozwiązaniem jest czujnik ruchu. W przypadku tego czujnika duże znaczenie mają zmiany monitorowanej
sceny. Niekoniecznie musi to być cały obszar, który jest obserwowany przez kamerę. Może to być
dowolny wycinek obrazu, w zależności od naszych upodobań. Problem w tym, że pod linux'em nie działa
nam plugin w przeglądarce i nie mamy możliwości skonfigurowania czujnika ruchu:

![]({{< baseurl >}}/img/2016/11/024.nc250-kamera-tp-link-detekcja-ruchu-web.png#huge)

W dalszym ciągu możemy jednak korzystać z tego czujnika ruchu ale będzie brany pod uwagę cały
monitorowany obszar. Powiadomienia są wysyłane dokładnie w taki sam sposób, co przy czujniku
dźwięku.

## Aplikacja tpCamera

Jeśli mamy smartfona z Androidem, to możemy na nim bez większego trudu doinstalować [aplikację
tpCamera](https://play.google.com/store/apps/details?id=com.tplink.skylight) od TP-LINK. Ten
programik umożliwi nam podgląd obrazu z kamery i to nie tylko będąc podłączonym do lokalnej sieci
domowej WiFi ale też zdalnie za pomocą [serwisu tplinkcloud](https://www.tplinkcloud.com/). Nie będę
tutaj opisywał samej aplikacji ale jeśli ktoś jest ciekaw co ma ona nam do zaoferowania to zapraszam
do zapoznania się z [osobnym wpisem, który jest poświęcony aplikacji
tpCamera]({{< baseurl >}}/post/aplikacja-tpcamera-do-obslugi-kamer-tp-link-z-poziomu-smartfona/).
