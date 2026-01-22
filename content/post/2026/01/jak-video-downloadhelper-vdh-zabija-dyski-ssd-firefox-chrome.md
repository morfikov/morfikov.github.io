---
author: Morfik
categories:
- Linux
date:    2026-01-22 14:55:00 +0100
lastmod: 2026-01-22 14:55:00 +0100
params:
  params:
  published: true
status: publish
tags:
- debian
- firefox
- chrome
- chromium
- vdh
- ssd
GHissueID: 606
title: Jak Video DownloadHelper (VDH) zabija dyski SSD (Firefox/Chrome)
---

Jakiś już czas temu naskrobałem kawałek artykułu na temat [mechanizmu Trim/discard w kontekście
LUKS/LVM na dysku SSD pod Debian linux][1], przy okazji poruszając tam kwestie nieco daleko
idącej [optymalizacji systemu pod kątem przedłużenia żywotności tego typu dyskom opartym o
technologię flash][2]. Opisane w tym powyższym artykule zabiegi sprawiły, że zapis danych na tym
dysku systemowym waha się w okolicy 3 GiB na dzień (średnia za rok 2024 i 2025). Gdyby ten stan
rzeczy utrzymać, to ten dysk SSD posłużyłby jeszcze przez około 160 lat i taka żywotność nośnika
SSD mnie jak najbardziej zadowala... Przez te dwa ostatnie lata tylko kilka aplikacji chciało
zapisywać ogromne ilości danych na dysku systemowym bez wyraźnej przyczyny, np [QuiteRSS][3], a tak
poza tym, to ten dysk SSD nie wykazywał większych odchyłów od normy w kwestii zapisu danych.
Niestety ten stan rzeczy uległ zmianie kilka dni temu. Szukając przyczyny udało się zawęzić krąg
podejrzanych do przeglądarki Firefox (ewentualnie też Google Chrome i Chromium). Okazało się, że
jeden z jej dodatków, a konkretnie [Video DownloadHelper][4] (VDH), z jakiegoś powodu zapisuje
ogromne ilości danych w katalogu profilu tej przeglądarki (tj. w `~/.mozilla/firefox/` ).
Pobierając dużo materiałów audio/video, np. z YouTube czy CDA, bardzo szybko nam taki dysk SSD
padnie i to bez znaczenia czy docelowy katalog zapisu pobranych danych z internetu mamy ustawiony,
np. na osobnym dysku HDD. Czy istnieje zatem jakiś sposób, który by powstrzymał dodatek Video
DownloadHelper od zniszczenia nam dysku SSD?

<!--more-->

## Video DownloadHelper do wersji 9.5.0.3

Z Video DownloadHelper korzystam od dawna i w zasadzie nie było z tym dodatkiem żadnych problemów.
No może za wyjątkiem wsparcia pewnych stron, do których to obsługi trzeba było
posiadać [Companion Application][5] (CoApp). Tak czy inaczej, poprawnie skonfigurowany VDH i CoApp
potrafiły obrabiać bardzo dużo serwisów i pobierać z nich materiały audio/video. Tak pobrane pliki
zapisywane były bezpośrednio w miejscu docelowym, tj. tam gdzie wskazywała
opcja `Download directory` w ustawieniach dodatku -- nie było przy tym żadnych plików tymczasowych
czy innych pośrednich katalogów. I tak to sobie działało bez zarzutu do wersji `9.5.0.3`
włącznie ([ostatnia wersja v9.x][6]).

## Nowy Video DownloadHelper (v10.x)

VDH wypuścił wersje v10.x dnia 2025-12-09. Przez ten ostatni miesiąc za wiele materiałów nie
pobierałem przy pomocy tego dodatku, więc w zasadzie nie było widać, że jakieś większe zmiany
zostały poczynione, które mogą się odbić na żywotności dysku SSD.

Parę dni temu jednak, zaszła potrzeba by z Video DownloadHelper skorzystać i się okazało, że
mechanika jego działania uległa zmianie. Niby pliki były pobierane ale coś było nie tak... Próbując
dojść WTF, zauważyłem, że przy [repozytorium CoApp][8] widnieje złowrogi
komunikat `This repository was archived by the owner on Dec 15, 2025. It is now read-only.`, co
zwykle oznacza zakończenie projektu. Niby CoApp nie działa już ale VDH w v10.x pobiera pliki bez
problemu. Pomyślałem, że może CoApp już nie jest do niczego potrzebny i cała jego funkcjonalność
została zaimplementowana bezpośrednio w VDH v10.x.

Pobierając kolejny plik (już do testów), dalej coś nie grało. Niby po kliknięciu "pobierz plik",
pobieranie się rozpoczynało ale zaglądając do katalogu docelowego pliku brak... Nie było nawet
pliku z końcówką `.part` (czy jakąkolwiek inną), który wskazywałby, że proces pobierania pliku w
Firefox jeszcze się nie zakończył. Gdzie zatem się ten plik pobiera?

Z początku myślałem, że może gdzieś do katalogu `/tmp/` (podmontowany w RAM) ale nie rozrasta się
on praktycznie wcale. Podobnie katalog `~/.cache/mozilla/firefox/` , który również trzymał swój
rozmiar (u mnie cały `~/.cache/` podmontowany jest w RAM). Po wydaniu polecenia `sync`, na
monitorze systemu `conky` zauważyłem sporą aktywność na dysku systemowym... Więc podejrzenie padło
na katalog `~/.mozilla/firefox/` . Tutaj `ncdu` pokazał, rozrastający się
folder `~/.mozilla/firefox/PROFILE_NUMBERS/storage/default/moz-extension+++VDH_HASH/fs/` . Te
zmienne PROFILE_NUMBERS i VDH_HASH są chyba unikatowe dla każdej instalacji, reszta ścieżki jest
już stała.

## VDH v10.x i pliki tmp

Wiedząc już gdzie tkwi problem, zacząłem szukaj informacji o możliwym błędzie, no bo przecie
ścieżka docelowa wskazuje na inny dysk, a Video DownloadHelper zapisuje dane w zupełnie innym
miejscu i dopiero gdy plik zostanie całkowicie pobrany, to jest przenoszony do lokalizacji
docelowej. Natrafiłem na [kilka pośrednich postów][9], które nie do końca traktowały o tym moim
problemie ale najwyraźniej inni użytkownicy też mieli jakiś większy czy mniejszy problem po zmianie
wersji z v9.x na v10.x.

Postanowiłem [zapytać o tę przypadłość na forum VDH][10] opisując również tymczasowe obejście tego
problemu. Z informacji zwrotnej, którą otrzymałem wynika, że moje zapytanie zostało potraktowane
jako SPAM, no bo najwyraźniej to nie bug a ficzer... Zostało mi podlinkowanych parę innych
podobnych tematów, które w żaden sposób nie odnoszą się do kwestii poruszanej przeze mnie, a mój
wpis został oznaczony jako ich duplikat -- no nie mam pytań. :] Mam za to takie nieodparte wrażenie,
że to A.I. pisało odpowiedź na moje zapytanie... Tak czy inaczej, dowiedziałem się tylko tyle,
że `nie` , tj. VDH v10.x `będzie zabijało` dyski SSD i co nam pan zrobisz?

## Próba wyłączenia disk cache w Firefox

Można by pomyśleć, że skoro przeglądarka Firefox ma opcje od cache trzymanego na dysku, to różnego
rodzaju dodatki, w tym też i  Video DownloadHelper będą te ustawienia respektować. No nic bardziej
mylnego.

Mając w `about:config` ustawione `browser.cache.disk.enable` na `false` jesteśmy w stanie
całkowicie wyłączyć cache na dysku i wymusić na Firefox korzystanie tylko z cache w pamięci RAM, co
możemy zaobserwować przechodząc w pasku adresu pod `about:cache` :

    Information about the Network Cache Storage Service

    memory
    Number of entries: 	260
    Maximum storage size: 	2048000 KiB
    Storage in use: 	8232 KiB
    Storage disk location: 	none, only stored in memory
    List Cache Entries

    disk
    Number of entries: 	260
    Maximum storage size: 	2048000 KiB
    Storage in use: 	8232 KiB
    Storage disk location: 	none, only stored in memory
    List Cache Entries

Widzimy tutaj, że `Storage disk location: 	none, only stored in memory` , a mimo to VDH pobiera
tymczasowo plik do katalogu profilu przeglądarki i to tylko po to, by po ukończeniu pobierania
przenieść go w miejsce docelowe...

## Jak zmusić VDH v10.x by zapisywał pliki tmp w katalogu /tmp/ , a nie w profilu Firefox

Najwyraźniej nie ma woli ze strony twórców Video DownloadHelper'a, by się zając problemem, który
dotyczy całej masy ludzi i przy okazji niszczy nośniki pamięci masowej w postaci dysków SSD. Na
szczęście istnieje proste rozwiązanie całej sytuacji, choć nie do końca jeszcze wiem jak trwałe ono
będzie -- o ile numerek profilu w katalogu `~/.mozilla/firefox/` jest stały, to nie wiem jak z
numerkami dodatków tj. `moz-extension+++...` , bo w zasadzie nigdy się tymi katalogami nie
interesowałem. Tak czy inaczej, możemy zmusić VDH, by zapisywał dane w katalogu `/tmp/` (lub
dowolnym innym), który w Debianie standardowo jest montowany w pamięci RAM. W ten sposób będziemy w
stanie dramatycznie odciążyć dysk SSD, choć kosztem pamięci RAM i maszyny z niewielką ilością
pamięci operacyjnej mogą nie być w stanie sobie z takim rozwiązaniem poradzić. Wszystko zależy od
wielkości plików, które będziemy pobierać -- jeśli film będzie miał 4 GiB, to o tyle wzrośnie nam
wykorzystanie pamięci RAM podczas pobierania plików.

Zaczynamy od zamknięcia przeglądarki Firefox i usunięcia folderu `/fs/` z katalogu dodatku VDH, tj.

    # rm -R /home/morfik/.mozilla/firefox/PROFILE_NUMBERS/storage/default/moz-extension+++VDH_HASH/fs/

Następnie tworzymy sobie plik `/etc/tmpfiles.d/vdh.conf` , który wypełniamy poniższą treścią:

    d /tmp/video-download-helper/    0700 morfik morfik 24h -
    L+ /home/morfik/.mozilla/firefox/PROFILE_NUMBERS/storage/default/moz-extension+++VDH_HASH/fs/    - morfik morfik - /tmp/video-download-helper/

Pamiętajmy o dostosowaniu zmiennych `PROFILE_NUMBERS` oraz `VDH_HASH` .

Teraz w terminalu wydajemy poniższe polecenie:

    # systemd-tmpfiles --create --exclude-prefix=/dev

Powinniśmy mieć już katalog `/tmp/video-download-helper/` na swoim miejscu oraz link do tego
katalogu w strukturze folderów dodatku VDH:

    $ ls -al /home/morfik/.mozilla/firefox/PROFILE_NUMBERS/storage/default/moz-extension+++VDH_HASH/

    total 96
    drwxr-xr-x    3 morfik morfik  4096 2026-01-21 19:40:23 ./
    drwxr-xr-x 1253 morfik morfik 81920 2026-01-21 19:44:54 ../
    lrwxrwxrwx    1 morfik morfik    27 2026-01-21 19:29:05 fs -> /tmp/video-download-helper/
    drwxr-xr-x    2 morfik morfik  4096 2026-01-21 19:21:45 ls/
    -rw-r--r--    1 morfik morfik   142 2026-01-21 20:09:55 .metadata-v2

Widzimy, że link wskazuje na `fs -> /tmp/video-download-helper/` i wszystko co powędruje do
katalogu `/fs/` poleci tak naprawdę do katalogu `/tmp/` , czyli do pamięci RAM.

Zarówno link jak i katalog w `/tmp/` będą tworzone wraz ze startem systemu i nie ma co sobie nimi
już głowy zawracać w późniejszym czasie -- ten mechanizm jest w pełni automatyczny. Dodatkowo
katalog `/tmp/video-download-helper/` będzie cyklicznie czyszczony z plików, do których procesy nie
odwoływały się przez ostatnie 24 godziny (czas można sobie naturalnie dostosować), np. jakieś
pozostałości po przerwanym pobieraniu, etc.

## Downgrade VDH do v9.x

Jeśli to powyższe rozwiązanie nam nie pasuje z jakiegoś powodu i nie chcemy z niego korzystać, to
możemy cofnąć wersję VDH do v9.x. Wystarczy [przejść pod ten link][6] i zainstalować zarówno VDH
jak i CoApp (jeśli go nie mamy już zainstalowanego). Wszystko powinno praktycznie natychmiast
wrócić do normy. Trzeba jednak zdawać sobie sprawę, że ta starsza wersja prawdopodobnie nie jest
już rozwijana i prędzej czy później przestanie wykrywać materiały audio/video w różnych serwisach.

## Podsumowanie

Co by nie mówić o Video DownloadHelper, to jednak jest to dość użyteczne narzędzie, które
automatyzuje w dużej mierze proces pobierania plików audio/video z różnych serwisów internetowych
pokroju YouTube czy CDA. Niemniej jednak, podejście twórców tego dodatku do Firefox/Chrome
pozostawia sporo do życzenia. Jakby nie patrzeć, tworzenie nowej wersji dodatku, która niszczy
nośniki SSD i to tylko dlatego, że nie potrafi zapisać pliku w dedykowanym katalogu TMP (czy też
bezpośrednio w docelowej lokalizacji) zasługuje wręcz na naganę. Widzimy też, że zgłoszenie dev'om
tego faktu jest traktowane jako SPAM. No nic dodać, nic ująć. Dobrze chociaż, że "linux jest linux"
i można przy pomocy systemd i jego mechanizmu obrabiania plików tymczasowych zautomatyzować
tworzenie odpowiedniej struktury katalogów i linków, które są w stanie wymusić na VDH umieszczanie
plików w katalogach, które są do tego bardziej odpowiednie niż profil przeglądarki...


[1]: https://morfikov.github.io/post/trim-discard-przy-luks-lvm-na-dysku-ssd-pod-debian-linux/
[2]: /post/trim-discard-przy-luks-lvm-na-dysku-ssd-pod-debian-linux/#odci%C4%85%C5%BCenie-dysku-ssd-przez-wykorzystanie-ramdysk%C3%B3w
[3]: https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=1058022
[4]: https://github.com/aclap-dev/video-downloadhelper
[5]: https://github.com/aclap-dev/video-downloadhelper/wiki/CoApp-Installation
[6]: https://github.com/aclap-dev/video-downloadhelper/wiki/Older-versions
[7]: https://github.com/aclap-dev/video-downloadhelper/discussions/2817
[8]: https://github.com/aclap-dev/vdhcoapp/
[9]: https://github.com/aclap-dev/video-downloadhelper/discussions/2866#discussioncomment-15260008
[10]: https://github.com/aclap-dev/video-downloadhelper/discussions/3040
