---
author: Morfik
categories:
- Linux
date: "2015-09-28T16:13:16Z"
date_gmt: 2015-09-28 14:13:16 +0200
published: true
status: publish
tags:
- pulseaudio
- dźwięk
- moduły-pulseaudio
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

Przeglądając youtube, natrafiłem na [materiał](https://www.youtube.com/watch?v=NXE-kSrhk_w)
opisujący ciekawy moduł, który można załadować w PulseAudio. Rozchodzi się o `module-ladspa-sink` ,
który ma za zadanie przerobić strumień audio przyciszając go tam gdzie jest zbyt głośny i
pogłaśniając tam gdzie jest zbyt cichy. Fachowo się to nazywa [Dynamic range
compression](https://en.wikipedia.org/wiki/Dynamic_range_compression).

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

Dyrektywa `load-module` ładuje określony moduł. Dalej są precyzowane [opcje dla samego
modułu](https://www.freedesktop.org/wiki/Software/PulseAudio/Documentation/User/Modules/):

  - `sink_name` definiuje nazwę, która zwracana jest, np. w `pacmd` .
  - `plugin` ładuje pożądany filter.
  - `label` określa etykietę dla pluginu, który jest po niej identyfikowany.
  - `control` zawiera ustawienia sterujące zachowaniem pluginu.

Problematyczne może być rozszyfrowanie wartości, które zostały podane w opcji `control` . Generalnie
rzecz biorąc, zależą one od samego pluginu i mogą się dość znacznie różnić. W tym przypadku został
użyty plugin [sc4m\_1916](http://plugin.org.uk/ladspa-swh/docs/ladspa-swh.html#tth_sEc2.92) . By
dowiedzieć się coś nieco więcej o tym pluginie, musimy doinstalować pakiet `ladspa-sdk` , po czym
wydać poniższe polecenie:

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

### Porty w sc4m\_1916

Doszukałem się na necie w miarę przystępnego wyjaśnienia [jak działa kompresor
dźwięku](http://www.studiomastering.net/mastering05.html). Poniżej jest krótki opis portów
wykorzystywanych przez plugin `sc4m_1916` .

  - `Threshold level` (dB) -- poziom sygnału, od którego kompresor zaczyna działać. Z reguły nie ma
    większych wartości od 0 (max) i jest niewiele mniejszych od -30 (min). W tym przypadku próg
    został ustawiony na -30, czyli przez kompresor będą przepuszczane praktycznie wszystkie dźwięki.
    Nie za dobre dla mp3 ale idealne dla filmów.
  - `Ratio` -- stosunek, w jakim kompresor będzie redukował dynamikę sygnału gdy ta przekroczy
    Threshold level. Jeśli jest tutaj ustawiona duża wartość (jak w tym przypadku, 20), to będzie
    stanowić to limit głośności określony przez Threshold level, którego sygnał nie będzie mógł
    przekroczyć. Niższe wartości będą ucinać nieco dB wyciszając tym samym dźwięk. Np. dla ratio
    4:1, każdy dźwięk powyżej Threshold level zostanie zredukowany o 75%. W przypadku 2:1, o 50%,
    itd. Zatem jeśli by pojawił się dźwięk o 8dB ponad Threshold level, to przy ratio 4:1 zostanie
    przycięty do wartości 2dB ponad limit, w przypadku 2:1, do 4dB ponad limit. Dźwięk poniżej
    Threshold level nie jest ruszany.
  - `Makeup gain` (dB) -- określa o ile dB podbić ciche dźwięki, lub też te wyciszone za sprawą
    Ratio, tak by były głośniejsze.
  - `Attack/Release time` (ms) -- służą do ustawienia czasu zadziałania kompresora oraz czasu
    powrotu całego układu do stanu neutralnego. Im mniejszy Attack time, tym miej szybkich skoków w
    sygnale przedostaje się przez kompresor. Z kolei Release time określa, jak długo kompresor ma
    działać na dźwięk. Zbyt krótki Release time może spowodować zniekształcenie dźwięku.
  - `Knee radius` -- odpowiada za to jak kompresor podchodzi do obcinania dB. Im niższa wartość, tym
    kompresor będzie działał bardziej gwałtownie przy zbliżaniu się do poziomu dźwięku określonego w
    Threshold level, przez co będziemy w stanie odczuć kompresję dźwięku. Z kolei, im większa
    wartość, tym kompresor wcześniej zacznie obcinać dB (nawet te poniżej Threshold level) ale za
    to przejście będzie o wiele łagodniejsze i mniej odczuwalne dla naszych uszu.

Warto jednak dodać, że te domyślne ustawienia spisują się całkiem dobrze i raczej powinny zadowolić
każdego.

## Test kompresora

Po zresetowaniu PulseAudio, w `pavcontrol` powinniśmy ujrzeć dodatkowy strumień, do którego możemy
przesyłać dźwięki z określonych aplikacji, co wygląda mniej więcej tak:

![]({{< baseurl >}}/img/2015/09/1.normalizacja-glosnosci-w-pulseaudio.png)

Mimo, że amarok został złapany na tej force podczas odtwarzania dźwięku ze sporym natężeniem, to ten
niebieski pasek, który widnieje także pod strumieniem modułu, wskazuje, że dźwięk jest odtwarzany
sporo ciszej. Oczywiście, strumienie są niezależne i można nimi dowolnie operować i nie koniecznie
muszą być one przepuszczone przez kompresor dźwięku
