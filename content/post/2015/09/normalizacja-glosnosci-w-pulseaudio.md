---
author: Morfik
categories:
- Linux
date:    2015-09-28 16:13:16 +0200
lastmod: 2024-02-06 14:00:00 +0100
published: true
status: publish
tags:
- pulseaudio
- dźwięk
- moduły-pulseaudio
- ladspa
GHissueID: 146
title: Normalizacja głośności w PulseAudio
---

Każdy z nas spotkał się z sytuacją, gdzie dźwięki odtwarzane na naszych maszynach zmieniają swoje
natężenie w krótkich odstępach czasu, a my przy tym odczuwamy bardzo nieprzyjemne uczucie
dyskomfortu psychicznego. Tego typu efekt jest też bardzo często spotykany przy oglądaniu filmów,
gdzie zwykle muzyka jest sporo głośniejsza od samych dialogów, nie wspominając już o reklamach,
którymi filmy są przerywane. Innym przykładem mogą być mp3, gdzie jedna z nich jest grana zbyt
cicho, natomiast kolejna zbyt głośno. PulseAudio jest w stanie poradzić sobie z tego typu
problemami.

<!--more-->
## Dynamic range compression

Przeglądając youtube, natrafiłem na [materiał][1] opisujący ciekawy moduł, który można załadować w
PulseAudio. Rozchodzi się o `module-ladspa-sink` , który ma za zadanie przerobić strumień audio
przyciszając go tam gdzie jest zbyt głośny i pogłaśniając tam gdzie jest zbyt cichy. Fachowo się to
nazywa [Dynamic range compression][2].

Moduł `module-ladspa-sink` nie jest ładowany domyślnie, dlatego też będziemy musieli nieco przerobić
konfigurację samego PulseAudio tak by normalizacja głośności została aktywowana. Dodatkowo, wymagany
jest pakiet `swh-plugins` , który prawdopodobnie będziemy musieli sobie doinstalować (jest dostępny
w repo debiana).

## Normalizacja głośności i jej aktywacja

Przechodzimy zatem do edycji pliku `/etc/pulse/default.pa` i na jego końcu dopisujemy poniższy blok
kodu:

    .ifexists module-ladspa-sink.so
    .nofail
    load-module module-ladspa-sink sink_name=compressor plugin=sc4m_1916 label=sc4m control=1,1.5,401,-30,20,5,12
    .fail
    .endif

`.ifexists` oraz `.endif` rozpoczynają instrukcję warunkową, która sprawdza czy dostępny jest
odpowiedni moduł. Dalej mamy dwie dyrektywy: `.nofail` oraz `.fail` , które określają czy
wykonywanie skryptu zostanie przerwane gdy określone niżej akcje zwrócą jakiś błąd. Powyższa
instrukcja zakłada wykonanie się skryptu nawet w przypadku gdy załadowanie modułu
`module-ladspa-sink` się nie powiedzie z jakichś przyczyn.

Dyrektywa `load-module` ładuje określony moduł. Dalej są precyzowane [opcje dla samego modułu][3]:

  - `sink_name` definiuje nazwę, która zwracana jest, np. w `pacmd` .
  - `plugin` ładuje pożądany filter.
  - `label` określa etykietę dla pluginu, który jest po niej identyfikowany.
  - `control` zawiera ustawienia sterujące zachowaniem pluginu.

Problematyczne może być rozszyfrowanie wartości, które zostały podane w opcji `control` . Generalnie
rzecz biorąc, zależą one od samego pluginu i mogą się dość znacznie różnić. W tym przypadku został
użyty plugin [sc4m_1916][4]. By dowiedzieć się coś nieco więcej o tym pluginie, musimy doinstalować
pakiet `ladspa-sdk` , po czym wydać poniższe polecenie:

    $ analyseplugin sc4m_1916
    Plugin Name: "SC4 mono"
    Plugin Label: "sc4m"
    Plugin Unique ID: 1916
    Maker: "Steve Harris "
    Copyright: "GPL"
    Must Run Real-Time: No
    Has activate() Function: No
    Has deactivate() Function: No
    Has run_adding() Function: Yes
    Environment: Normal or Hard Real-Time
    Ports:  "RMS/peak" input, control, 0 to 1, default 0
            "Attack time (ms)" input, control, 1.5 to 400, default 101.125
            "Release time (ms)" input, control, 2 to 800, default 401
            "Threshold level (dB)" input, control, -30 to 0, default 0
            "Ratio (1:n)" input, control, 1 to 20, default 1
            "Knee radius (dB)" input, control, 1 to 10, default 3.25
            "Makeup gain (dB)" input, control, 0 to 24, default 0
            "Amplitude (dB)" output, control, -40 to 12
            "Gain reduction (dB)" output, control, -24 to 0
            "Input" input, audio
            "Output" output, audio

Jak widać, nie tylko jest podana kolejność portów ale także są przedziały możliwych do określania
wartości wraz z wartościami domyślnymi (na wypadek ich nie podania w konfiguracji). Jest tam, co
prawda, 9 portów ale te dwa ostatnie (bez wartości default) są dla wyjścia (np. podejrzenie w locie
o ile dB sygnał jest redukowany), jednak PulseAudio nie potrafi ich obsłużyć.

### Failed to load plugin "sc4m_1916"

W przypadku gdy przy próbie analizy pluginu wyskoczy nam poniższy błąd:

    $ analyseplugin sc4m_1916
    Failed to load plugin "sc4m_1916": sc4m_1916: cannot open shared object file: No such file or directory

Oznacza to, że nie posiadamy w systemie ustawionej zmiennej `$LADSPA_PATH` i by ten powyższy błąd
poprawić, wystarczy w terminalu (lub w konfiguracji shell'a bash/zsh) dopisać sobie poniższą
linijkę.

    export LADSPA_PATH="/usr/lib/ladspa/"

### Porty w sc4m_1916

Doszukałem się na necie w miarę przystępnego wyjaśnienia jak działa kompresor dźwięku. Poniżej jest
krótki opis portów wykorzystywanych przez plugin `sc4m_1916` .

  - `RMS/peak` -- w przypadku wielu współczesnych kompresorów, mamy do wyboru tryb pracy detektora
      sygnału sterującego, który może działać w trybie szczytowym (Peak) lub uśrednionym (RMS, Root
      Mean Square). Przy ustawieniu na 0, kompresor reaguje na bezwzględny poziom chwilowy (true
      peak). W przypadku ustawienia na 1, uśrednianiu podlega dłuższy fragment sygnału, na podstawie
      którego działa blok tłumienia. W praktyce reakcja na poziomy chwilowe sprawdza się, gdy chcemy
      uzyskać mocną i słyszalną kompresję lub stłumić transjenty, tj. gwałtowne szczyty i skoki w
      sygnale, np. w przypadku bębnów. Tryb RMS jest znacznie łagodniejszy w działaniu, dając
      wyrównaną kompresję w przypadku wokalu, gitar, grup i całych miksów.
  - `Attack/Release time` (ms) -- służą do ustawienia czasu zadziałania kompresora oraz czasu
      powrotu całego układu do stanu neutralnego. Im mniejszy `Attack time`, tym miej szybkich
      skoków w sygnale przedostaje się przez kompresor. Z kolei `Release time` określa, jak długo
      kompresor ma działać na dźwięk. Zbyt krótki `Release time` może spowodować zniekształcenie
      dźwięku.
  - `Threshold level` (dB) -- poziom sygnału, od którego kompresor zaczyna działać. Z reguły nie ma
      większych wartości od 0 (max) i jest niewiele mniejszych od -30 (min). W tym przypadku próg
      został ustawiony na -30, czyli przez kompresor będą przepuszczane praktycznie wszystkie
      dźwięki. Nie za dobre dla mp3 ale idealne dla filmów.
  - `Ratio` -- stosunek, w jakim kompresor będzie redukował dynamikę sygnału, gdy ta przekroczy
      `Threshold level` . Jeśli jest tutaj ustawiona duża wartość (jak w tym przypadku, 20), to
      będzie stanowić to limit głośności określony przez `Threshold level` , którego sygnał nie
      będzie mógł przekroczyć. Niższe wartości będą ucinać nieco dB wyciszając tym samym dźwięk. Np.
      dla `Ratio 4:1` , każdy dźwięk powyżej `Threshold level` zostanie zredukowany o 75%. W
      przypadku 2:1, o 50%, itd. Zatem jeśli by pojawił się dźwięk o 8 dB ponad `Threshold level` ,
      to przy ratio 4:1 zostanie przycięty do wartości 2 dB ponad limit, w przypadku 2:1, do 4 dB
      ponad limit. Dźwięk poniżej `Threshold level` nie jest ruszany.
  - `Knee radius` -- parametr Knee ma wpływ na początkowy etap działania kompresji, określając, jak
      wyraziście będzie następowało tłumienie z chwilą zbliżenia się poziomu sygnału do ustawionego
      progu w Threshold level. Może to następować natychmiast (tzw. twarde kolano, od 0 dB),
      stopniowo (miękkie kolano, do 40 dB) lub w każdy sposób pomiędzy tymi dwoma pozycjami. W
      przypadku twardego kolana kompresja aplikowana jest od razu z chwilą, gdy sygnał przekroczy
      próg kompresji, i z poziomem ustawionym regulatorem Ratio. Przy miękkim kolanie kompresja
      zaczyna stopniowo działać gdy poziom sygnału zbliża się do progu Threshold level, a pełne
      Ratio aplikowane jest wtedy, gdy go w niewielkim stopniu przekroczy. W praktyce element ten
      wpływa na brzmienie kompresji, od wyraźnie słyszalnego zakolorowania przy twardym kolanie (co
      sprawdza się przy precyzyjnej obróbce bębnów), do praktycznie niesłyszalnej kompresji
      stosowanej głównie przy masteringu.
  - `Makeup gain` (dB) -- określa o ile dB podbić ciche dźwięki, lub też te wyciszone za sprawą
    Ratio, tak by były głośniejsze.
  - `Amplitude` (dB) -- Poziom sygnału wejściowego.
  - `Gain reduction` (dB) -- stopień redukcji wzmocnienia stosowany dla sygnału wejściowego.

Poniżej jest jeszcze parę rzeczy, które można napotkać w innych kompresorach, np. tym oferowanym
przez [Easy Effects][7]:

  - `Dry/Wet Level` -- kompresja równoległa, zwana czasem "nowojorską", polega na zmieszaniu
      sygnału skompresowanego z sygnałem bez kompresji. Wiele współczesnych kompresorów już ma
      odpowiedni regulator -- zwykle pod postacią Dry/Wet w okolicach regulatora poziomu
      wyjściowego. Funkcja ta pozwala zachować definicję transjentów, a jednocześnie zaaplikować
      bardziej agresywną kompresję, uwypuklając towarzyszące jej zakolorowanie dźwięku. Sprawdza
      się to szczególnie dobrze przy przetwarzaniu surowo brzmiących bitów, ale można zastosować tę
      funkcję także na całym miksie.
  - `Lookahead` -- funkcja wyprzedzenia (lookahead) polega na opóźnianiu sygnału audio, ale nie w
      torze sterowania, dzięki czemu kompresor odpowiednio wcześniej "wie", jak ma zareagować.
      Teoretycznie jest to bardzo korzystna konfiguracja, ale należy pamiętać, że odpowiednio
      wcześniej pojawi się też etap powrotu (Release). Aby to skompensować można użyć parametru
      podtrzymania (Hold, jeśli jest taki dostępny), ustawionego na taki sam czas, jak czas
      wyprzedzenia.

Warto jednak dodać, że te domyślne ustawienia spisują się całkiem dobrze i raczej powinny zadowolić
każdego.

## Test kompresora

Po zresetowaniu PulseAudio, w `pavcontrol` powinniśmy ujrzeć dodatkowy strumień, do którego możemy
przesyłać dźwięki z określonych aplikacji, co wygląda mniej więcej tak:

![normalizacja-glosnosci-w-pulseaudio](/img/2015/09/1.normalizacja-glosnosci-w-pulseaudio.png#big)

Mimo, że amarok został złapany na tej force podczas odtwarzania dźwięku ze sporym natężeniem, to ten
niebieski pasek, który widnieje także pod strumieniem modułu, wskazuje, że dźwięk jest odtwarzany
sporo ciszej. Oczywiście, strumienie są niezależne i można nimi dowolnie operować i nie koniecznie
muszą być one przepuszczone przez kompresor dźwięku


[1]: https://www.youtube.com/watch?v=NXE-kSrhk_w
[2]: https://en.wikipedia.org/wiki/Dynamic_range_compression
[3]: https://www.freedesktop.org/wiki/Software/PulseAudio/Documentation/User/Modules/
[4]: http://plugin.org.uk/ladspa-swh/docs/ladspa-swh.html#tth_sEc2.92
[5]: http://www.studiomastering.net/mastering05.html
[7]: https://github.com/wwmm/easyeffects
