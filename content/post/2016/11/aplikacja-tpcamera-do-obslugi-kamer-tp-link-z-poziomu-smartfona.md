---
author: Morfik
categories:
- Android
date:    2016-11-10 21:59:13 +0100
lastmod: 2016-11-10 21:59:13 +0100
published: true
status: publish
tags:
- tp-link
- smartfon
- aplikacje
- kamera
- aparat
title: Aplikacja tpCamera do obsługi kamer TP-LINK z poziomu smartfona
---

Kamery IP to bardzo użyteczne urządzenia, choć ich obsługa nie zawsze jest wygodna. Nie chodzi tutaj
o zarządzenie nimi, bo taka kamera ma przecie swój własny adres IP i możemy bez większego problemu
dostać się do jej panelu administracyjnego przez sieć, w tym też nawet i po WiFi. Niemniej jednak,
gdy jesteśmy w terenie, to bardzo rzadko mamy przy sobie komputer czy nawet laptop, na którym
moglibyśmy zainstalować odpowiedni soft w celu uzyskania podglądu obrazu z takiej kamery. W
przypadku kamer od TP-LINK sprawa wygląda nieco inaczej, bo mamy tutaj możliwość zaprzęgnięcia do
pracy smartfona. Taki telefon można bez problemu połączyć z kamerą, z tym, że potrzebna nam będzie
do tego specjalna aplikacja: tpCamera. Zobaczmy zatem do czego może ona nam się przydać.

<!--more-->
## Instalacja tpCamera z Google Play

[Aplikację tpCamera][1] można pobrać ze sklepu Google Play. Jest ona dostarczana bezpośrednio przez
TP-LINK, no i nie zawiera reklam. Szkoda tylko, że nie jest OpenSource. Tak czy inaczej, poniżej są
fotki z procesu instalacyjnego:

![](/img/2016/11/001.tpcamera-aplikacja-android-tp-link-kamera-ip-instalacja.png#huge)

Jak widać, tpCamera trochę waży bo 23 MiB. Po uruchomieniu aplikacji przywita nas poniższe okienko:

![](/img/2016/11/002.tpcamera-aplikacja-android-tp-link-kamera-ip-logowanie.png#medium)

Kamerami TP-LINK możemy zarządzać przez tpCamera na dwa sposoby: zdalnie (przez internet) i lokalnie
(przez sieć LAN). Zdalne podłączenie wymaga utworzenia konta w [serwisie tplinkcloud][2].

## Lokalne zarządzenie kamerą

Użytkownicy, którzy niezbyt przepadają za zezwalaniem zewnętrznym serwisom na dostęp do monitoringu,
mogą wybrać opcję podglądu lokalnego, z tym, że w przeciwieństwie do opcji zdalnej, smartfon musi
być podłączony do sieci, w której znajduje się kamera. Jeśli opuścimy tę sieć, to stracimy także
podgląd.

Po uruchomieniu aplikacji przy podłączonej kamerze do sieci, nasz smartfon powinien ją bez problemu
rozpoznać:

![](/img/2016/11/003.tpcamera-aplikacja-android-tp-link-kamera-ip-lokalne-zarzadzanie.png#big)

Wystarczy teraz tapnąć w miniaturkę i zostanie nam zaserwowany podgląd live z kamery:

![](/img/2016/11/004.tpcamera-aplikacja-android-tp-link-kamera-ip-lokalny-podglad.png#medium)

Nie ma tutaj zbyt wielu opcji ale są te najważniejsze. Przede wszystkim, możemy wyciszyć dźwięk z
kamery jeśli nam hałas z jakiegoś powodu przeszkadza. Gdy orientacja kamery nie została optymalnie
ustawiona podczas jej montażu, to możemy odwrócić obraz o 180 stopni. Jest też opcja przełączania
trybu dziennego i nocnego, o ile kamera jest wyposażona w diody z podczerwienią. Wyżej mamy może i
niewielki obraz ale nic nie stoi, by go zmaksymalizować:

![](/img/2016/11/005.tpcamera-aplikacja-android-tp-link-kamera-ip-lokalny-podglad-full.png#big)

W lewym dolnym rogu jest także informacja o prędkości transferu. W przypadku tego [NC250][3],
transfer waha się w granicach 5-20 KiB/s, gdy na monitorowanym obszarze nic się nie dzieje. Ta
wartość potrafi wzrosnąć do 200-300 KiB/s, gdy scena się zmienia, a im szybciej te zmiany zachodzą,
tym więcej łącza będzie utylizował obraz z kamery.

Wszystko co pojawia nam się na ekranie smartfona możemy również zapisać na flash'u tego urządzenia.
Mamy opcję nagrania obrazu video (z dźwiękiem lub bez) i robienia pojedynczych fotek.

Wszelkie pozostałe opcje konfiguracyjne, które są w stanie zmienić ustawienia kamery, mogą zostać
dostosowane jedynie przez panel administracyjny kamery.

## Zdalne zarządzenie kamerą

Gdy chodzi o zdalne podłączenie do kamery, to warto tutaj zaznaczyć, że nie musimy zmieniać ustawień
routera w celu przepuszczenia ruchu z serwisu tplinkcloud do kamery. To kamera nawiązuje sesję z tym
serwisem i podtrzymuje to połączenie przez cały czas. Można się zastanawiać ile transferu taka
kamera zjada będąc non stop podłączona do sieci. Okazuje się, że niewiele. Kamera może i chodzi cały
czas ale jeśli nie ma do niej podłączonego żadnego klienta, tj. nikt nie ma włączonego podglądu, to
po chwili dane przestają być transferowane przez sieć. Jak tylko jakiś klient wyśle żądanie o obraz
z kamery, to dane znów zaczną płynąć przez sieć.

Mając konto w serwisie tplinkcloud, możemy uzupełnić formularz logowania. Po zalogowaniu się, musimy
dodać kamerę. Klikamy zatem na znak plusa w prawym górnym rogu ekranu smartfona i wybieramy model
kamery:

![](/img/2016/11/006.tpcamera-aplikacja-android-tp-link-kamera-ip-zdalne-dodawanie.png#big)

Następnie zostaniemy zapytani czy nasza kamera jest podłączona do prądu i czy jest ona również
podłączona do sieci lokalnej. Kamerę do routera możemy podłączyć w tym miejscu przewodowo lub
bezprzewodowo:

![](/img/2016/11/007.tpcamera-aplikacja-android-tp-link-kamera-ip-wyszukiwanie.png#big)

Jeśli jednak nie konfigurowaliśmy sieci WiFi na kamerze do tej pory, to przy opcji przewodowej
zostaniemy poproszeni o podanie konfiguracji AP:

![](/img/2016/11/008.tpcamera-aplikacja-android-tp-link-kamera-ip-wifi.png#huge)

Po chwili kamera zostanie odnaleziona i dodana do listy:

![](/img/2016/11/009.tpcamera-aplikacja-android-tp-link-kamera-ip-wykrywanie.png#medium)

Nadajemy jej jeszcze jakąś przyjazną nazwę i klikamy "Korzystaj":

![](/img/2016/11/010.tpcamera-aplikacja-android-tp-link-kamera-ip-zdalny-podglad.png#big)

Status w prawym górnym rogu ekranu wskazuje, że kamera działa i sygnał może być odbierany na
smartfonie przez internet. Panel kamery jest dokładnie taki sam jak w przypadku lokalnego
podłączenia. Niemniej jednak, w tym przypadku na miniaturce w prawym dolnym rogu mamy małe kółko
zębate. Po jego przyciśnięciu naszym oczom ukaże się nieco więcej informacji na temat samej kamery:

![](/img/2016/11/011.tpcamera-aplikacja-android-tp-link-kamera-ip-opcje.png#medium)

Jak widać, możemy tutaj wyłączyć zdalnie diodę na kamerze, jak i dostosować poziom rejestrowanego
dźwięku przez kamerę.

## Komunikacja przez kamerę

Jeśli nasza kamera posiada wbudowany głośnik i mikrofon, tak jak, np. NC450, to aplikacja tpCamera
może wykorzystać głośnik i mikrofon naszego smartfona do obustronnej komunikacji. Mówiąc do
mikrofonu w telefonie, nasz głos będzie słychać na głośniku kamery i podobnie w drugą stronę, gdzie
ktoś będzie mówił do kamery, to my go usłyszymy na głośniku w smartfonie.

![](/img/2017/02/001.tpcamera-aplikacja-android-tp-link-kamera-ip-mikrofon.png#big)

## Ustawienia czujnika ruchu i dźwięku

Z reguły kamery IP są w stanie wykryć ruch (ewentualnie dźwięk) i zainicjować nagrywanie obrazu czy
tez robienie zdjęć. W aplikacji tpCamera możemy sobie dostosować opcje czujnika ruchu i dźwięku:

![](/img/2017/02/002.tpcamera-aplikacja-android-tp-link-kamera-ip-ruch-dzwiek.png#huge)

Jak widać wyżej, jesteśmy też w stanie ustawić harmonogram i określić w jakich godzinach wykrywanie
ruchu i dźwięku ma mieć miejsce, co może nas uchronić przed masą niepożądanych powiadomień.

## Powiadomienia o wykryciu ruchu i dźwięku

Skoro już mowa o powiadomieniach, to warto tutaj wspomnieć, że poza standardowym powiadamianiem o
wykryciu ruchu lub dźwięku, TP-LINK'owe kamery są w stanie wysłać powiadomienie do aplikacji
tpCamera. W takiej sytuacji praktycznie od razu będziemy w stanie zareagować na to co się dzieje w
monitorowanych obiektach. Poniżej jest przykład takiego powiadomienia:

![](/img/2017/02/003.tpcamera-aplikacja-android-tp-link-kamera-ip-notyfikacje.png#big)

Warto tutaj zaznaczyć, że ilość notyfikacji przez email jest ograniczona do 60 na miesiąc. Nie wiem
czy ten limit można zmienić ale w wiadomości dostałem taką informację: "NC450 1.0" has 58 remaining
email alerts for this month. You can change motion detection and alert settings in your tpCamera app
or log in to the camera's web interface. Także raczej nie da rady otrzymać więcej notyfikacji niż 60
miesięcznie. Natomiast notyfikacje na aplikację tpCamera są bez ograniczeń.

W przypadku, gdy kamera jest w stanie rejestrować obraz i dźwięk na wbudowany w nią nośnik, np. karę
SD, to naturalnie możemy bez większego problemu uzyskać dostęp do tak nagranych materiałów z poziomu
smartfona.

![](/img/2017/02/004.tpcamera-aplikacja-android-tp-link-kamera-ip-nagrania-video.png#big)

## Aktualizacja firmware kamery

Aplikacja tpCamera umożliwia nam aktualizację firmware, który się znajduje w kamerze. Niby tego typu
operacje powinno się przeprowadzać raczej przez połączenie przewodowe ale tutaj nie powinno być
żadnych problemów przy wgrywaniu oprogramowania po WiFi. Poniżej są fotki z przeprowadzonego
właśnie procesu aktualizacji do nowszej wersji.

Za każdym razem, gdy pojawi się nowsza wersja firmware dla naszej kamery, to aplikacja tpCamera nas
o tym powiadomi:

![](/img/2017/02/005.tpcamera-aplikacja-android-tp-link-kamera-ip-aktualizacje.png#big)

W zasadzie wystarczy kliknąć w przycisk "Aktualizuj" i proces zostanie rozpoczęty:

![](/img/2017/02/006.tpcamera-aplikacja-android-tp-link-kamera-ip-aktualizacje.png#huge)

Jak możemy wyczytać na powyższym ostrzeżeniu, tego procesu nie da rady przerwać i będzie on trwał
stosunkowo długo, bo około 5 minut. Nie powinno być przy tym żadnych problemów.

## Problemy z połączeniem via tpCamera

Jeśli z jakiegoś powodu mamy problem z połączeniem się z kamerą przez internet, sprawdźmy pierw czy
w panelu administracyjnym kamery widnieje odpowiedni status pod Cloud Settings:

![](/img/2016/11/012.tpcamera-aplikacja-android-tp-link-kamera-ip-could-settings.png#huge)

Nawet jeśli wszystkie ustawienia kamery zdają się być w porządku, to i tak z jakiegoś powodu
pojawiają się dziwne problemy z połączeniem. W sklepie Google Play jest cała masa negatywnych opinii
w stosunku do aplikacji tpCamera. Ludzie narzekają na czarny ekran i ogólnie na brak podglądu z
kamery. W moim przypadku lokalne zarządzanie kamerą NC250 jest w porządku ale są podobne problemy
przy korzystaniu z serwisu tplinkcloud.


[1]: https://play.google.com/store/apps/details?id=com.tplink.skylight
[2]: https://www.tplinkcloud.com/
[3]: http://www.tp-link.com.pl/products/details/cat-19_NC250.html
