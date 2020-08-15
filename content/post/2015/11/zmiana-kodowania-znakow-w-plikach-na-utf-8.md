---
author: Morfik
categories:
- Linux
date: "2015-11-13T16:18:56Z"
date_gmt: 2015-11-13 15:18:56 +0100
published: true
status: publish
tags:
- pliki
- foldery
- utf-8
title: Zmiana kodowania znaków w plikach na UTF-8
---

Pliki tekstowe w linux'ie są zakodowane w [formacie
UTF-8](https://www.cl.cam.ac.uk/~mgk25/unicode.html). Ta cyfra `8` może być jednak myląca, bo ilość
bajtów potrzebnych do zakodowania pojedynczego znaku może się różnić i wynosi od 1 do 4. Tak czy
inaczej, środowiska linux'owe już dawno zaimplementowały obsługę tego systemu kodowania i uczyniły
go sobie domyślnym. Są jednak takie systemy operacyjne, które nie wykorzystują domyślnie UTF-8 do
kodowania tekstu. Wobec czego, gdy spróbujemy otworzyć w edytorze taki plik, to w pewnych miejscach
będziemy mieli krzaczki, zwykle tam gdzie są polskie znaki. To zjawisko jest bardzo
charakterystyczne dla napisów w filmach. Niemniej jednak, zarówno edytory tekstu jak i player'y
video są w stanie dokonać automatycznego doboru systemu kodowania i zwrócić nam czytelny plik. Nie
zawsze jednak tak robią. Zamiast bawić się w tego typu automatyczne wynalazki, dużo lepszym
rozwiązaniem jest zmiana kodowania plików, tak by przekonwertować je do formatu UTF-8 i w tym
wpisie postaramy się to zrobić.

<!--more-->
## Zmiana kodowania plików przy pomocy iconv

W debianie mamy kilka narzędzi, przy pomocy których zmiana kodowania pliku tekstowego trwa dosłownie
chwilę. Jednym z popularniejszych jest [iconv](https://pl.wikipedia.org/wiki/Iconv). By za jego
pomocą przekonwertować plik z jednego typu kodowania na inny, to w terminalu wpisujemy polecenie na
wzór tych dwóch poniższych:

    $ iconv -f iso8859-2 -t utf8 "plik.txt" -o plik-utf-8.txt
    $ iconv -f windows-1250 -t utf8 "plik.txt" -o plik-utf-8.txt

Generalnie rzecz biorąc, to cała trudność związana ze zmianą kodowania sprowadza się do określenia
formatu wejściowego i wyjściowego pliku tekstowego. W przypadku windowsów, najczęstszymi kodowaniami
z jakimi będziemy się spotykać, to `iso8859-2` lub `windows-1250` . Dlatego te dwie powyższe linijki
zawsze jest mieć dobrze pod ręką. A na wypadek gdybyśmy nie znali kodowania danego pliku, to możemy
skorzystać z `file` lub `enca` .

## Zmiana kodowania plików przy pomocy enca

Zmiana kodowania plików może także zostać przeprowadzona za pomocą narzędzia `enca` . W porównaniu
do poprzednika, musimy sobie je w systemie doinstalować. Mając już ten pakiet zainstalowany,
przechodzimy do ustalenia formatu jakim został zakodowany przykładowy plik tekstowy. Trzeba jednak
mieć na względzie, że `enca` operuje na zmiennych środowiskowych ( `LC_CTYPE` i innych), które
odpowiadają za ustawienie języka w systemie. Może to prowadzić do nieporozumień i przyjęcia błędnych
założeń co do wejściowego kodowania pliku. Dlatego też dobrze jest korzystać z opcji `-L` ręcznie
precyzując język, przykładowo:

    $ enca -L polish -m plik.txt
    ISO-8859-2

Mamy zatem do czynienia z plikiem zakodowanym w `ISO-8859-2` , czyli w formacie jaki jest
standardowo obsługiwany w windowsach. By teraz ten plik przekonwertować do formatu `UTF-8` ,
wpisujemy w terminalu to poniższe polecenie:

    $ enca -L polish -x utf-8 plik.txt
    $ enca -L polish -m plik.txt
    UTF-8

W ten sposób można zmieniać nie tylko kodowanie plików tekstowych ale także plików `.html` czy
skryptów `.php` . Choć na dobrą sprawę, to one wszystkie są tekstowe.
