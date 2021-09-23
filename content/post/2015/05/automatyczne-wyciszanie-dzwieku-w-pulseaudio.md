---
author: Morfik
categories:
- Linux
date: "2015-05-28T11:43:22Z"
date_gmt: 2015-05-28 09:43:22 +0200
published: true
status: publish
tags:
- pulseaudio
- dźwięk
- moduły-pulseaudio
title: Automatyczne wyciszanie dźwięku w PulseAudio
---

Wszystkie główne dystrybucje, a może raczej ich środowiska graficzne, wykorzystują do odtwarzania
dźwięku [serwer PulseAudio][1]. Niektóre wręcz są tak z nim zżyte, że nie idzie ich oddzielić od
siebie. Ja generalnie uważam, że ten kawałek oprogramowania jest jak najbardziej przydatny
człowiekowi i potrafi realizować kilka kwestii, które bez niego, albo by nie były możliwe do
osiągnięcia, albo trzeba by się natrudzić przy ich implementacji, np. przesyłanie dźwięku przez
sieć. U siebie na debianie nigdy nie miałem większych problemów z PulseAudio, z kolei zaś te, które
się przytrafiały na drodze jego użytkowania, szło w miarę prosty sposób wyeliminować. Jest jednak
jeden problem, z którym prawdopodobnie spotkaliśmy się wszyscy, przynajmniej jeśli wykorzystujemy
mikrofon w stopniu większym niż przeciętny użytkownik komputera. Chodzi o to, że po odpaleniu
pewnych aplikacji (lub też i w trakcie ich działania), takich jak np. Skype, Mumble, czy
TeamSpeak3, dźwięk we wszystkich pozostałych programach potrafi zwyczajnie zdechnąć.

<!--more-->
## Moduły zarządzające dźwiękiem

"Problem" leży oczywiście po stronie PulseAudio, a konkretnie jego modułów. W tej chwili interesują
nas dwa z nich -- module-role-cork oraz [module-role-ducking][2]. Ten pierwszy nie ma, co prawda,
dokumentacji ale wiemy, że jego zadaniem jest zatykać (mutować)  strumienie pewnych aplikacji, które
odtwarzają dźwięk w danej chwili, tak by inny program nie był przez nie zakłócany w żaden sposób.
Raczej zdarzało nam się wyłączyć choć raz muzykę, gdy dzwonił do nas telefon, prawda? I właśnie to
zachowanie próbuje zreprodukować za pośrednictwem tego modułu PulseAudio. Kłopot w tym, że nie
zawsze chcemy tłumić wszelki dźwięk wydobywający się z głośników i zamiast mutować go, chcielibyśmy
go nieco przyciszyć i za tego typu zachowanie odpowiada ten drugi moduł.

### Blokada modułu odpowiedzialnego za wyciszenie dźwięku

Domyślnie w PulseAudio jest aktywny jedynie moduł `module-role-cork`, co sprawia, że cały dźwięk, za
wyjątkiem tego jednego strumienia, np. Skype, jest wyciszany. Naturalnie możemy odmutować go ręcznie
i wtedy oba strumienie powinny być odtwarzane bez problemu, przynajmniej do momentu wystąpienia
jakiegoś zdarzenia, które znów przykręci śrubę wszystkim innym aplikacjom.

Jeśli drażni nas tego typu zachowanie i wolelibyśmy sami zatroszczyć się o zarządzanie poziomem
dźwięku w aplikacjach, to mamy kilka wyjść z tej sytuacji. Jedno z nich zakłada całkowite
usunięcie PulseAudio, co w wielu przypadkach może nie być możliwe ale lepszym wyjściem będzie
nakazanie PulseAudio by zwyczajnie nie ładował modułu `module-role-cork` podczas swojego
uruchamiania.

Konfiguracja modułów dla PulseAudio trzymana jest w pliku `/etc/pulse/default.pa` . Przechodzimy
zatem do jego edycji i odszukujemy pozycję z `module-role-cork` , po czym komentujemy tę linijkę:

    ### Cork music/video streams when a phone stream is active
    #load-module module-role-cork

Jeśli chcemy jedynie sprawdzić czy faktycznie problem tkwi w tym module, możemy wyładować go przy
pomocy poniższego polecenia:

    $ pactl load-module module-role-cork

Tak czy inaczej, problem automatycznego wyciszania dźwięku powinniśmy mieć raczej z głowy.

### Aktywacja modułu ściszającego dźwięk

Generalnie rzecz biorąc, jestem za rozgraniczeniem aplikacji na te, które w obecnej chwili powinny
być na pierwszym planie od tych wszystkich pozostałych ale znowu nie lubię absolutnej ciszy i
przydałoby się zaprogramować PulseAudio, tak by jedynie ściszył szereg aplikacji bez ich
kompletnego mutowania. By tego typu zachowanie wypracować, dodajemy poniższy kod do pliku
`/etc/pulse/default.pa` :

    load-module module-role-ducking trigger_roles=phone ducking_roles=music,video volume=60%

Możemy również ten moduł, wraz z jego opcjami, załadować w celu testowania regulacji:

    $ pactl load-module module-role-ducking trigger_roles=phone ducking_roles=music,video volume=60%

#### Parę słów o module-role-ducking

Przydało by się kilka słów wyjaśnienia odnośnie samego modułu `module-role-ducking` i użytych w nim
opcji. Przede wszystkim, PulseAudio potrafi rozróżniać strumienie dźwięku na podstawie ich
[właściwości][3], wobec czego każdy program odtwarzający dźwięk może być traktowany indywidualnie,
choć ze względów praktycznych lepiej jest je sobie pogrupować. Kompletna lista właściwości, jakie
możemy ustawić, znajduje się [tutaj][4].

Pierwszy parametr jaki widzimy wyżej, tj. `trigger_roles` , wskazuje ona na właściwość aplikacji,
której odpalenie objawiać się będzie ściszeniem dźwięku innych programów, które aktualnie coś
nadają. Z tym, że niekoniecznie z automatu muszą to być wszystkie aplikacje dźwiękowe. Przy pomocy
opcji `ducking_roles` określamy, które aplikacje mają być ściszane. Zatem w powyższym przypadku,
jeśli strumień, który ma ustawioną opcję `PA_PROP_MEDIA_ROLE` na `phone` (czyli np. Skype),
zostanie zainicjowany, spowoduje on automatyczne przyciszenie strumieni, które mają tę samą
właściwość ustawioną na `music` lub `video` . Parametr `volume` określa co zrobić z dźwiękiem i w
tym przypadku zostanie on przyciszony do poziomu `60%` Z tym, że wartość tutaj możemy podawać w
`%` , `dB` , albo jako zwykłą liczbę z zakresu `0-65536`.

Tylko pozostaje jeden problem -- sporo programów w debianie (czy w ogóle linux'ie) nie ma
ustawionych odpowiednich właściwości, co będzie się objawiać w taki sposób, że cześć z nich
zwyczajnie zignoruje politykę, którą wyżej sobie ustawiliśmy, a ludzie winą obarczą jak zwykle
PulseAudio... By uniknąć tego typu sytuacji, możemy ręcznie nadać wszystkim aplikacjom odtwarzającym
dźwięk (w tym też i playery video) odpowiednie właściwości poprzez wyeksportowanie zmiennych
systemowych. I tak dla przykładu, jako że potrzebujemy zmienić/nadać `PA_PROP_MEDIA_ROLE` ,
przepiszmy właściwości SMPlayer'owi:

    PULSE_PROP='media.role=video' smplayer film.mp4

W przypadku jakichkolwiek błędów dobrze jest odpalić pulseaudio w trybie verbose via `pulseaduio -D
-v` i prześledzić komunikaty pojawiające się przy odtwarzaniu strumieni. Przykładowo:

    pulseaudio[61212]:     media.role = "video"
    pulseaudio[61212]:     media.name = "audio stream"
    pulseaudio[61212]:     application.name = "Amarok"
    pulseaudio[61212]:     native-protocol.peer = "UNIX socket client"
    pulseaudio[61212]:     native-protocol.version = "30"
    pulseaudio[61212]:     application.id = "org.kde.phonon.amarok"
    pulseaudio[61212]:     application.version = "2.8.0"
    pulseaudio[61212]:     application.icon_name = "amarok"
    pulseaudio[61212]:     application.language = "C"
    pulseaudio[61212]:     application.process.id = "61277"
    pulseaudio[61212]:     application.process.user = "morfik"
    pulseaudio[61212]:     application.process.host = "morfikownia"
    pulseaudio[61212]:     application.process.binary = "amarok"
    pulseaudio[61212]:     window.x11.display = ":0"
    pulseaudio[61212]:     application.process.machine_id = "159815709bbc46c29ef786cfc497afd4"
    pulseaudio[61212]:     application.process.session_id = "1"
    pulseaudio[61212]:     module-stream-restore.id = "sink-input-by-media-role:video"

I jak widzimy, amarok ma przypisaną rolę `video` .


[1]: https://www.freedesktop.org/wiki/Software/PulseAudio/
[2]: https://www.freedesktop.org/wiki/Software/PulseAudio/Documentation/User/Modules/#module-role-ducking
[3]: https://www.freedesktop.org/wiki/Software/PulseAudio/Documentation/Developer/Clients/ApplicationProperties/
[4]: http://0pointer.de/lennart/projects/pulseaudio/doxygen/proplist_8h.html
