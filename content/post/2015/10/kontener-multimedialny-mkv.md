---
author: Morfik
categories:
- Linux
date: "2015-10-19T19:29:10Z"
date_gmt: 2015-10-19 17:29:10 +0200
published: true
status: publish
tags:
- video
GHissueID: 193
title: Kontener multimedialny MKV
---

Wiele osób posiada pliki video i ci co uczą się angielskiego, czy innych języków, niezbyt przepadają
za słuchaniem polskiego lektora na filmach, bo zagłusza on przecie całą oryginalną ścieżkę audio.
Poza tym, jakość tłumaczenia jest na żenująco niskim poziomie. Z samego słuchu człowiek ciężko się
uczy, zwłaszcza jak zaczyna naukę nowego języka, dlatego też można sobie dociągnąć polskie napisy.
Co jednak w przypadku, gdy chcemy mieć kilka ścieżek audio czy napisów w jednym filmie? Posiadanie
wielu plików wprowadza trochę zamętu. Poniżej zostanie opisany sposób na ogarnięcie kolekcji
filmowej, tak by został nam się tylko jeden plik w przypadku każdego ulubionego przez nas filmu.

<!--more-->
## Mkvtoolnix i inne potrzebne narzędzia

Każda kolekcja filmów różni się w zależności od tego kto te filmy zbierał i przechowywał na
przestrzeni lat. Wobec czego możemy posiadać wiele starych formatów plików, np. napisy w czystym
pliku tekstowym. Poniżej jest lista narzędzi, która nam pomoże ogarnąć ten bałagan:

  - `mkvtoolnix` zawiera szereg narzędzi obrabiających kontenery mkv.
  - `mkvtoolnix-gui ` to graficzna nakładka na `mkvtoolnix` .
  - `vobsub2srt` pomoże nam przekonwertować napisy `vobsub` do formatu `srt` .
  - `tesseract-ocr` oraz `tesseract-ocr-*` to tekstowe narzędzie OCR. W miejsce gwiazdki można
    podstawić kod danego kraju, np. `pol` , `fra` czy `spa` .

## Informacje o kontenerze MKV

Mając już zainstalowane pakiety, możemy przejść do właściwej pracy, tj. podejrzenia zawartości
takiego kontenera MKV. Służy do tego narzędzie `mkvmerge` , które może zwrócić nam więcej informacji
o kontenerze (parametr `-I` ) lub też może nam pokazać jedynie podstawowe info (parametr `-i`) . Nam
wystarczą informacje uzyskane za pomocą parametru `-i` , zatem wydajemy w terminalu to poniższe
polecenie:

    $ mkvmerge -i "film.mkv"
    File 'film.mkv': container: Matroska
    Track ID 0: video (MPEG-4p10/AVC/h.264)
    Track ID 1: audio (AAC)
    Track ID 2: subtitles (VobSub)
    Track ID 3: subtitles (VobSub)
    Track ID 4: subtitles (VobSub)
    Track ID 5: subtitles (SubRip/SRT)
    Chapters: 23 entries

Jak widać wyżej, w kontenerze mamy sześć ścieżek. Jedna ścieżka video oraz audio, cztery ścieżki z
napisami w różnych językach (angielskim, francuskim oraz hiszpańskim), z tym, że napisy angielskie
są w dwóch różnych formatach: VobSub oraz SubRip/SRT. Są także rozdziały, czyli zakładki na ścieżce
filmu, do których możemy przejść przy jego odtwarzaniu, np. w VLC.

## Wyodrębnianie ścieżek

Znając zawartość kontenera MKV, możemy wyciągnąć wszystkie jego ścieżki i pozamykać je w osobnych
plikach. Służy do tego narzędzie `mkvextract` , a ścieżki zaś wydobywamy w poniższy sposób:

    $ mkvextract tracks "film.mkv" 0:video 1:audio 2:eng 3:fra 4:spa 5:eng2

Zapis `0:video` jest w formacie `numer_ścieżki:nazwa_pliku` , gdzie `numer_ścieżki` odpowiada temu
zwróconemu przez narzędzie `mkvmerge` , zaś `nazwa_pliku` to nazwa pliku pod jakim zapisać tę
ścieżkę na dysku.

Dodatkowo trzeba wyciągnąć metadane opisujące dźwięk i obraz. Bez nich będą problemy z odtwarzaniem
filmu:

    $ mkvextract timecodes_v2 "film.mkv" 0:video.txt 1:audio.txt

Rozdziały zaś wyciągamy w poniższy sposób:

    $ mkvextract chapters "film.mkv" > chapters.xml

Nie każdy film posiada wszystkie ścieżki, które widzimy wyżej. Także jakby czegoś brakowało to bez
obaw.

## Format i kodowanie napisów

W przypadku tego filmu, w kontenerze nie było polskich napisów ale można je dociągnąć, np. przy
pomocy [napi](https://github.com/dagon666/napi/). Pozostaje tylko problem z kodowaniem. Pakietu
`napi` nie ma w debianie i jeśli ktoś nie chce pobierać w/w narzędzia, może skorzystać z `iconv` ,
który pomoże z przekonwertowaniem napisów do formatu utf-8.

Polskie napisy mogą używać różnych kodowań. Są to zwykle: utf-8, windows-1250 i iso8859-2. Jeśli
mamy krzaki przy ustawieniu kodowania na utf-8 w edytorze tekstu, musimy zmienić kodowanie pliku
jedną z poniższych linijek:

    $ iconv -f iso8859-2 -t utf8 "film.txt" -o pol.srt
    $ iconv -f windows-1250 -t utf8 "film.txt" -o pol.srt

Z kontenera MKV wyciągnęliśmy jeden format napisów w SRT, pozostałe trzy mają format VobSub, którego
nie odtwarza SMPlayer. VLC nawet łapie ten format napisów ale ich czcionka czy jakość pozostawia
wiele do życzenia. Przekonwertujmy zatem wszystkie napisy do formatu SRT.

Najpierw ustalmy w jakim języku są dane napisy -- nie zawsze to może być oczywiste:

    $ vobsub2srt --langlist ./fre
    Languages:
    0: fr

I konwertujemy:

    $ vobsub2srt fre
    Wrote Subtitles to 'fre.srt'

Podobnie postępujemy dla pozostałych napisów.

Polskie napisy, które będziemy pobierać, prawdopodobnie będą miały format SRT, mimo, że będą
zapisane w pliku z rozszerzeniem `.txt` . Jeśli jednak trafią się nam napisy w formacie innym niż
SRT, możemy je łatwo przekonwertować, np. przy pomocy `gnome-subtitles` lub też wspomnianego wyżej
`napi` , który zawiera narzędzie `subotage` .

## Tworzenie kontenera MKV

Mamy zatem wyodrębnioną ścieżkę video oraz audio, rozdziały oraz 4 wersje napisów. Czas to skleić w
jedną całość. By to uczynić, posługujemy się narzędziem `mkvmerge` :

    $ mkvmerge -o ".film.mkv.fix" \
    > "--aac-is-sbr" "0:1" \
    > "--language" "0:eng" "--default-track" "0:yes" "--forced-track" "0:no" "--timecodes" "0:./audio.txt" "-a" "0" "-D" "-S" "-T" "--no-global-tags" "--no-chapters" "(" "./audio" ")" \
    > "--sub-charset" "0:UTF-8" "--language" "0:eng" "--forced-track" "0:no" "-s" "0" "-D" "-A" "-T" "--no-global-tags" "--no-chapters" "(" "./eng.srt" ")" \
    > "--sub-charset" "0:UTF-8" "--language" "0:fre" "--default-track" "0:no" "--forced-track" "0:no" "-s" "0" "-D" "-A" "-T" "--no-global-tags" "--no-chapters" "(" "./fre.srt" ")" \
    > "--sub-charset" "0:UTF-8" "--language" "0:pol" "--default-track" "0:no" "--forced-track" "0:no" "-s" "0" "-D" "-A" "-T" "--no-global-tags" "--no-chapters" "(" "./pol.srt" ")" \
    > "--sub-charset" "0:UTF-8" "--language" "0:spa" "--default-track" "0:no" "--forced-track" "0:no" "-s" "0" "-D" "-A" "-T" "--no-global-tags" "--no-chapters" "(" "./spa.srt" ")" \
    > "--language" "0:eng" "--default-track" "0:yes" "--forced-track" "0:no" "--timecodes" "0:./video.txt" "-d" "0" "-A" "-S" "-T" "--no-global-tags" "--no-chapters" "(" "./video" ")" \
    > "--track-order" "5:0,0:0,1:0,3:0,2:0,4:0" \
    > "--chapter-language" "eng" "--chapters" "./chapters.xml"

Składanie kontenera MKV dużo prościej odbywa się przez graficzną nakładkę `mkvtoolnix-gui` . W
zależności od rozmiaru ścieżek wejściowych, muxowanie może trochę zająć. Po tym procesie, można już
wrzucić film do SMPlayer'a czy VLC i sprawdzić czy działa prawidłowo.
