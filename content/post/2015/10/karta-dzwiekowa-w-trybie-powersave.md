---
author: Morfik
categories:
- Linux
date: "2015-10-06T10:47:38Z"
date_gmt: 2015-10-06 08:47:38 +0200
published: true
status: publish
tags:
- moduły-kernela
- dźwięk
- powersave
title: Karta dźwiękowa w trybie powersave
---

Kilka dni temu, [na forum DUG'a](https://forum.dug.net.pl/viewtopic.php?pid=291349), jeden z
użytkowników miał problem z dźwiękiem. Udało się tę niedogodność wprawdzie poprawić ale został tam
poruszony temat trybu powersave, czyli oszczędzania energii, jaki może posiadać karta dźwiękowa. Na
dobrą sprawę, nigdy mi nawet do głowy nie przyszło, by te karty mogły przełączać sobie stan i zjadać
mniej prądu, tak jak to robią, np. karty WiFi. Oczywiście, postanowiłem zgłębić to zagadnienie i
ustalić na ile przydatna jest ta funkcja i czy da radę bez problemów słuchać muzyki lub oglądać
filmy po jej aktywowaniu.

<!--more-->
## Karta dźwiękowa na module snd\_hda\_intel

Ta karta dźwiękowa, która znajduje się w moim laptopie, działa na module kernela `snd_hda_intel` i
to głównie w oparciu o niego jest pisany ten post. Jeśli ktoś posiada kartę na module
`snd_ac97_codec` , to niech rzuci okiem [na ten
link](http://www.thinkwiki.org/wiki/How_to_enable_audio_codec_power_saving).

W każdym razie, by włączyć tryb powersave w danej karcie, potrzebny jest nam odpowiednie urządzenie
(które będzie wspierać taką opcję), dodatkowo będziemy musieli odpowiednio skonfigurować sam moduł
odpowiedzialny za obsługę karty dźwiękowej.

Moduł najlepiej ładować od razu na starcie systemu, dlatego też trzeba go dopisać do pliku
`/etc/modules` :

    snd_hda_intel

Musimy teraz ustalić jakie parametry ma ten moduł i czy w ogóle wspiera tryb powersave. Do tego celu
posłuży nam narzędzie `systool` , które znajduje się w pakiecie `sysfsutils` . Wydajemy zatem
poniższe polecenie:

    # systool -v -m snd_hda_intel | grep power
        power_save          = "0"
        power_save_controller= "Y"

Parametr `power_save` definiuje ile czasu musi upłynąć aby sterownik przełączył kartę w tryb
mniejszego zużycia energii od momentu, w którym przechodzi ona w stan bezczynności (IDLE). Z kolei
zaś ` power_save_controller` odpowiada za wyłączenie również kontrolera, co przekłada się na jeszcze
większą oszczędność energii. Zatem moduł posiada opcje umożliwiające przełączenie karty dźwiękowej w
tryb powersave. Musimy zatem odpowiednio ustawić te dwa powyższe parametry.

Opcje dla modułu definiujemy, np. w pliku `/etc/modprobe.d/sound.conf` , przykładowo:

    options snd-hda-intel power_save=10 power_save_controller=Y

Po restarcie maszyny, sprawdzamy czy moduł został załadowany, a parametry ustawione na pożądane
wartości:

    # journalctl -b -u systemd-modules-load.service | grep snd_hda_intel
    Oct 05 18:20:23 morfikownia systemd-modules-load[352]: Inserted module 'snd_hda_intel'
    
    # systool -v -m snd_hda_intel | grep power
        power_save          = "10"
        power_save_controller= "Y"

Zatem urządzenie zostało z powodzeniem skonfigurowane do pracy w trybie powersave.

## Powersave i PulseAudio

PulseAudio nie wpływa w żaden sposób na to jak działa tryb powersave karty dźwiękowej. Mylący może
być stan sink'u, który można podejrzeć via `pacmd` :

    $ pacmd list-sinks | egrep -i "state|suspend"
            state: RUNNING
            suspend cause:

Ten powyższy stan może zmienić wartość z `RUNNING` (jakaś aplikacja odtwarza dźwięk) na `IDLE`
(zakończenie odtwarzania) oraz po jakichś 5s na stan `SUSPENDED` , czyli zamknięcie urządzenia
ALSA. Jak widać, w zależności od tego czy są odtwarzane dźwięki, zmienia się się stan sink'u ale [to
nie jest to samo co tryb
powersave](https://lists.freedesktop.org/archives/pulseaudio-discuss/2015-October/024518.html) karty
dźwiękowej.

W opcjach modułu ustawialiśmy jedynie czas, po którym sterownik sygnalizuje możliwość przejścia w
tryb powersave, a robi to dopiero gdy stan sink'u jest ustawiony na `SUSPENDED` . Reasumując, po 5s
od zaprzestania odtwarzania dźwięku przez aplikacje, sterownik przełączy kartę w stan powersave po
10 następnych sekundach. Przynajmniej taka jest teoria.

## Stany oszczędzania energii

Zaglądając do plików `/proc/asound/card0/codec\#*` możemy podejrzeć szereg parametrów naszej karty
dźwiękowej. Są tam min. dane dotyczące aktualnego trybu pracy urządzenia:

    $ cat /proc/asound/card0/codec\#0 | grep -i power
      Power states:  D0 D1 D2 D3 CLKSTOP EPSS
      Power: setting=D0, actual=D0

[W dokumentacji kernela](https://www.kernel.org/doc/Documentation/power/pci.txt) możemy wyczytać, że
urządzenia PCI mają zdefiniowane cztery stany połączeń: `D0`, `D1`, `D2`, `D3` . Im wyższy numer,
tym mniej prądu pobiera urządzenie w takim stanie, czyli w powyższym przypadku, stan `D0` oznacza,
że karta dźwiękowa pracuje na pełnym zasilaniu. Czytając dalej podlinkowaną dokumentację, możemy
dopatrzeć się także informacji na temat tego, że tylko `D0` oraz `D3` są wymagane przy
implementacji, a pozostałe dwa opcjonalne. Różnica między tymi czterema stanami polega nie tylko na
ilości pobieranego prądu ale także tkwi w czasach opóźnienie między przełączeniem się urządzenia ze
stanu powersave do stanu pełnego zasilania. Krótko mówiąc, im wyższy numer, tym więcej czasu to
zajmuje.

## Problemy z trybem powersave

Nie każda karta dźwiękowa jest w stanie przełączyć się w stan powersave, nawet jeśli wyraźnie
włączymy taką opcję w module kernela. Objawia się to brakiem przełączania między stanami D0-D3 i
karta cały czas pracuje w stanie `D0` . Jako, że tryb powersave może powodować więcej problemów niż
przynieść pożytku (wystarczy popatrzeć na bezprzewodowe karty sieciowe WiFi), to zwykle takie
wynalazki są domyślnie wyłączone w konfiguracji kernela ( `CONFIG_SND_HDA_POWER_SAVE_DEFAULT=0` ).

Jeśli mamy jakieś problemy z dźwiękiem i podejrzewamy przy tym, że może być winny tryb powersave, to
warto wiedzieć, że mamy możliwość konfigurowania parametrów modułu w locie, przykładowo, by wyłączyć
tryb oszczędzania energii, wpisujemy w terminal poniższe polecenie:

    # echo "0" > /sys/module/snd_hda_intel/parameters/power_save
