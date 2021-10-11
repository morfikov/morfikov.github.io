---
author: Morfik
categories:
- Linux
date: "2015-11-02T23:54:42Z"
date_gmt: 2015-11-02 21:54:42 +0100
published: true
status: publish
tags:
- pliki
- foldery
- prywatność
- gtk2
- gtk3
GHissueID: 250
title: Ostatnio używane pliki (recently-used.xbel)
---

Wielu ludzi nie lubi gdy maszyny monitorują każdy ich krok. W tym przypadku chodzi o pliki, które
otwieramy czy zmieniamy podczas codziennej pracy na komputerze. Nasz system domyślnie tworzy listę i
skrupulatnie dodaje do niej nowe pozycje. Ta lista jest przechowywana w pliku `recently-used.xbel` ,
który znajduje się w katalogu `~/.local/share/` . Gdy popatrzymy na tę funkcjonalność trochę pod
inny kątem, możemy zauważyć, że w pewnych sytuacjach zagraża ona naszej prywatności. Skasowanie tego
pliku nie rozwiązuje problemu, bo jest on tworzony na nowo, a nadawanie atrybutu odporności (
`chattr +i` ) nie jest żadnym rozwiązaniem. Na szczęście jest sposób na to by ten mechanizm
dezaktywować i o tym będzie poniższy wpis.

<!--more-->
## Prywatność czy paranoja?

Listę dostępnych w systemie plików można bez problemu wyciągnąć. Zatem po co usuwać te pozycje,
które trafiają do pliku `recently-used.xbel` ? Powodów jest kilka. Przede wszystkim nie o
wszystkich plikach w systemie możemy wiedzieć i pomijam oczywiście te, do których nie mamy
odpowiednich uprawnień. Chodzi mi o te pliki, które są umieszczone w kontenerze TrueCrypt czy LUKS.
Jeśli w pliku `recently-used.xbel` znajdą się ścieżki do nieistniejącej partycji w systemie, może to
doprowadzić do ujawnienia kontenera. Niekoniecznie możemy od razu poznać jego położenie w drzewie
katalogów ale nie powinno to stanowić większego problemu, przecie kontenery mają z reguły parę GiB.

Poza ujawnieniem zaszyfrowanych kontenerów, musimy także liczyć się z metadanymi o plikach, które
otwieramy. Za pomocą listy otwieranych plików możemy pozyskać nazwę pliku, datę jego edycji, czy też
i rozmiar. Do tego dochodzi jeszcze rozszerzenie danego pliku. Zatem możemy określić czy jest to
plik tekstowy, graficzny, a może materiał video. Czasem te dane mogą nam wiele powiedzieć o tym co
znajduje się w pliku i nawet nie musimy do niego zaglądać.

## GTK2, GTK3 i plik recently-used.xbel

Środowiska graficzne zwykły wykorzystywać plik `recently-used.xbel` i umieszczać z nim te wszystkie
metadane, o których wspomniałem wyżej. Wobec czego, menadżery plików, edytory graficzne,
przeglądarki obrazków, czy playery audio/video są w stanie skorzystać z tych informacji i ułatwić
nam nieco życie. By wyjaśnić jak działa ten mechanizm, musimy posłużyć się przykładem. Otwórzmy w
swoim ulubionym graficznym edytorze tekstu jakiś plik. Następnie go zamknijmy. Ponownie odpalmy
edytor, z tym, że tym razem nie otwierajmy żadnego pliku. Następnie w menu wybierzmy File -> Open.
Jest tam pozycja `Recently Used` . Po przejściu do niej, powinniśmy być w stanie uzyskać listę
plików, które ostatnio otwieraliśmy w tym edytorze. U mnie wygląda to tak:

![](/img/2015/11/1.recently-used.xbel-geany.png#huge)

Co w przypadku, gdy posiadamy w systemie pliki, które nie powinny być umieszczane na takiej liście?
Każda aplikacja graficzna posiadająca interfejs GTK2/GTK3 jest w stanie dodawać do pliku
`recently-used.xbel` pozycje i tym samym kompromitować poufność danych. Możemy jednak sprawić by
tego nie robiły poprzez zmianę globalnej konfiguracji dla stylów GTK.

Interfejs graficzny jest nam prezentowany w pewien określony sposób. Wygląd tego interfejsu zależy
od wybranego stylu, a to może określić sobie każdy użytkownik przy pomocy, np. `lxappearance` .
Większość z tego typu narzędzi oferuje jedynie podstawową konfigurację. Mamy jednak możliwość
ręcznej edycji plików `~/.gtkrc-2.0` oraz `~/.config/gtk-3.0/settings.ini` , przez co możemy
nakazać aplikacjom GTK by zachowywały się w pewien określony sposób.

### Aplikacje GTK2

By nakazać aplikacjom GTK2, by nie zapisywały pliku `recently-used.xbel` , dopisujemy ten poniższy
kod do pliku `~/.gtkrc-2.0` :

    gtk-recent-files-max-age=0

### Aplikacje GTK3

W przypadku aplikacji GTK3 dopisujemy te dwie linijki do pliku `~/.config/gtk-3.0/settings.ini` :

    gtk-recent-files-max-age=0
    gtk-recent-files-limit=0

## Kompromis

W obu powyższych przypadkach możemy pójść na kompromis i nieco poluzować politykę prywatności.
[Wartość parametru
gtk-recent-files-max-age](https://people.gnome.org/~shaunm/girdoc/C/Gtk.gtk-recent-files-max-age.html)
oznacza liczbę dni. Zatem jeśli ustawimy tutaj `2` , to wpisy w pliku `recently-used.xbel` będą
trzymane jedynie przez dwa dni, co jest w miarę przyzwoitym okresem czasu. Z kolei jeśli chodzi o
[parametr
gtk-recent-files-limit](https://people.gnome.org/~shaunm/girdoc/C/Gtk.gtk-recent-files-limit.html) ,
to określa on ile wpisów plik `recently-used.xbel` może zawierać maksymalnie.
