---
author: Morfik
categories:
- Linux
date: "2016-01-24T22:22:55Z"
date_gmt: 2016-01-24 21:22:55 +0100
published: true
status: publish
tags:
- xserver
- monitor
title: Odpowiedni DPI (PPI) dla monitora
---

Monitory posiadają różne wymiary fizyczne i co za tym idzie mają też inne rozdzielczości. Te
informacje nie są jakoś zbytnio tajne i każdy klient przed zakupem konkretnego modelu monitora jest
w stanie się zapoznać z tymi parametrami. To co zwykle sprzedawcy starają się ukryć przed nami, to
współczynnik [PPI (pixels per inch)](https://pl.wikipedia.org/wiki/Ppi). Zwykle też można spotkać
się z terminem [DPI (dots per inch)](https://pl.wikipedia.org/wiki/Dpi). Nie są one równoznaczne,
bo w monitorach LCD jeden piksel składa się z trzech podpikseli. Sprzedawcy wykorzystują ten fakt i
chwalą się, że dany model monitora ma 300 DPI. W skrajnych przypadkach można nawet usłyszeć i 300
PPI. Osoby, które nie rozróżniają tych dwóch pojęć mogą bardzo łatwo zostać oszukane podczas zakupu.
Problem potęguje fakt, że gdy monitor jest sporych rozmiarów i patrzymy na niego z większej
odległości, to nawet nie dostrzeżemy, że nas oszukano. Na wszelki wypadek lepiej założyć, że
sprzedawca ma co innego na myśli niż my i podaną wartość podzielić przez 3, a następnie
skontrastować tak otrzymaną liczbę z PPI większości monitorów (100). Niemniej jednak, dobrze jest
przed zakupem monitora sprawdzić jaki współczynnik PPI ma dany model i nie sugerować się tym co
sprzedawca napisał w ofercie. Dlatego też w tym wpisie postaramy się ustalić faktyczną wartość PPI
dla naszego obecnego lub przyszłego monitora.

<!--more-->
## Jak sprawdzić DPI (PPI)

Xserver jest w stanie rozpoznać nasz monitor prawidłowo w większości przypadków ale może mieć
problemy z określeniem DPI (PPI). Aktualne ustawienia monitora zawsze możemy sprawdzić posługując
się narzędziem `xdpyinfo` , przykładowo:

    $ xdpyinfo | grep dots
      resolution:    96x96 dots per inch

Stosowne informacje może odczytać także z logu Xserver'a:

    $ cat /var/log/Xorg.0.log
    ...
    [ 11016.247] (==) intel(0): DPI set to (96, 96)

Zatem DPI (PPI) dla tego monitora LCD zostało ustawione na wartość 96. Czy zostało to zrobione aby
poprawnie? Sprawdźmy.

## Jak ustalić DPI (PPI) monitora

By dowiedzieć się jakie wartości PPI i DPI ma nasz monitor, musimy ustalić jego rozdzielczość oraz
jego fizyczne wymiany. Z tym, że interesuje nas wysokość i szerokość przestrzeni widzialnej. Jeżeli
nie dysponujemy dokumentacją techniczną monitora (instrukcja obsługi), to zawsze możemy skorzystać z
`xrandr` lub też bezpośrednio zajrzeć w log Xserver'a:

```
 $ xrandr | grep -w connected
LVDS1 connected primary 1366x768+0+0 (normal left inverted right x axis y axis) 344mm x 193mm
```

Jak widzimy, rozdzielczość tego monitora, to `1366x768` . Natomiast jego przestrzeń widzialna ma
`344mm x 193mm` . By uzyskać DPI (PPI), musimy podzielić rozdzielczość poziomą przez szerokość
ekranu. Podobnie postępujemy z rozdzielczością pionową i wysokością ekranu. Następnie obie wartości
mnożymy przez 25.4. W ten sposób otrzymamy DPI (PPI) w poziomie oraz w pionie:

    (1366/344)*25.4= 100.86163
    (768/193)*25.4=  101.07358

Otrzymane liczby różnią się nieznacznie. Trzeba je zaokrąglić do pełnych wartości. W obu przypadkach
zatem mamy `101` . Zatem DPI (PPI) tego monitora to 101 i jest to wynik przeciętny.

## Dostosowanie DPI (PPI) pod Xserver'em

Domyślne wartości DPI (PPI) dla tego monitora zostały wykryte niepoprawnie. Możemy je zmienić przez
określenie parametru `DisplaySize` w sekcji `Monitor` w pliku `/etc/X11/xorg.conf.d/20-monitor.conf`
, przykładowo:

    Section "Monitor"
          Identifier "LVDS1"

          DisplaySize 344 193
    EndSection

Wartości dla `DisplaySize` odczytujemy z wyjścia `xrandr` , tak jak to zostało zrobione wyżej.
Xserver wyliczy DPI (PPI) automatycznie na podstawie zdefiniowanego tutaj rozmiaru i wykrytej
rozdzielczości. Jeśli ktoś jest ciekaw innych opcji, które można umieścić w powyższym pliku, to
niech rzuci okiem na artykuł [o konfiguracji
monitora]({{< baseurl >}}/post/monitor-i-jego-konfiguracja-pod-linuxem/).

## Problemy z niestandardowym DPI (PPI)

Jak mogliśmy zaobserwować wyżej, Xserver ustawił DPI (PPI) na 96. My z kolei określiliśmy faktyczną
wartość, która wynosi 101. Jeśli zaczniemy z niej korzystać, to mogą pojawić się [problemy z
wyświetlaniem pewnych elementów](https://wiki.archlinux.org/index.php/xorg#Setting_DPI_manually).
Chodzi głównie o skalowanie tekstu opartego o czcionki bitmapowe. Dlatego też powinniśmy korzystać z
proporcjonalnego skalowania, które zakłada ustawienie DPI (PPI) na: 96, 120 (+25%), 144 (+50%), 168
(+75%), 192 (+100%). Jako, że ten monitor ma 101 DPI (PPI), to jedyną rozsądną wartością dla tego
monitora jest wartość 96. Jeśli jednak zdecydujemy się na zmianę tej wartości, to trzeba mieć na
uwadze, że szerokość lub/i wysokość czcionek może ulec zmianie, np. w conky czy urxvt.

Jeśli nie mamy możliwości przetestowania zakupowanego urządzenia, tak by sprawdzić jego parametry,
to dobrym wyjściem może okazać się skorzystanie [z tej strony](http://dpi.lv/), gdzie możemy znaleźć
informacje o tych bardziej popularnych wyświetlaczach i to w oparciu o te dane dobrać sobie czy to
nowy monitor, czy też i smartfon.
