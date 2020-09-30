---
author: Morfik
categories:
- Linux
date: "2015-10-17T18:00:43Z"
date_gmt: 2015-10-17 16:00:43 +0200
published: true
status: publish
tags:
- gnome
- dconf
title: Oczyszczanie bazy danych dconf
---

Jak możemy przeczytać [na wiki projektu GNOME][1], `dconf` to swojego rodzaju baza danych, która
zawiera informacje o konfiguracji systemu w postaci klucz-wartość. Jak nie korzystam, co prawda, ze
środowisk graficznych ale wykorzystuję w swoim linux'ie szereg ich elementów, które mogą działać z
powodzeniem w okrojonym systemie, np. `gnome-keyring` . Część programów wykorzystuje tę bazę danych
do przechowywania swoich ustawień, które możemy zmieniać via `gsettings` , `gconf` lub też `dconf` .
Problem pojawia się po dłuższym czasie używania systemu, gdzie cześć z ustawień jest już
przestarzała, np. w wyniku zaprzestania użytkowania jakiejś aplikacji. Przydałoby się zatem raz na
jakiś czas oczyścić te bazę danych ze zbędnych wpisów i o tym będzie ten wpis.

<!--more-->
## Gsettings, gconf oraz dconf

Zgodnie z informacjami jakie znalazłem [tutaj][2], [tutaj][3] i [tutaj][4], `gconf` jest już
przestarzały i odnosi się do konfiguracji w środowiskach GNOME 2.x . Część z aplikacji jednak nadal
może wykorzystywać to API. W nowszych wersjach GNOME, korzysta się z `gsettings` . Z kolei
`gsettings` wykorzystuje `dconf` jako backend oferujący GUI ( `dconf-editor` ), które ułatwia zmianę
ustawień. Tak na marginesie, opisy poszczególnych kluczy w `dconf` są pobierane z katalogu
`/usr/share/glib-2.0/schemas/` , natomiast baza danych [jest trzymana w min. katalogu
użytkownika][5] pod `~/.config/dconf/user` .

## Plik ~/.config/dconf/user

Przyjrzyjmy się zatem bliżej plikowi `~/.config/dconf/user` . Jest to plik binarny i nie damy rady
podejrzeć go za pomocą edytora teksu. Wyżej wspomniałem, że możemy podejrzeć zawartość bazy
korzystając, np. z `dconf-editor` , odpalmy go zatem:

![](/img/2015/10/1.baza-danych-dconf-oczyszanie.png#big)

Problem w tym, że `dconf-editor` umożliwia jedynie odczyt i zapis wartości poszczególnych kluczy
dostępnych w bazie. Nie ma jednak opcji usuwania takich kluczy, czy czyszczenia bazy danych. W tym
celu będziemy musieli wydobyć wpisy z bazy danych i usunąć ręcznie te, które są zbędne. Następnie, z
tych pozostałych wpisów trzeba będzie sporządzić nową bazę danych.

Przechodzimy zatem do katalogu `~/.config/dconf/` i wydobywamy zawartość bazy danych:

    $ cd ~/.config/dconf/
    $ cp user user.back
    $ dconf dump / > database

Po otwarciu pliku `database` , powinna nam się ukazać zawartość bazy. Poniżej jest przykładowa
sekcja ustawień:

    [apps/light-locker]
    late-locking=false
    lock-after-screensaver=uint32 1
    lock-on-suspend=true
    lock-on-lid=false

Są to dokładnie te wpisy, które widzieliśmy wyżej w `gconf-editor` . Niemniej jednak, baza danych
zawiera jedynie wpisy, które różnią się od wartości domyślnych, czyli te, które zmieniliśmy. Dlatego
też nie wpadajmy w panikę, gdy w pliku tekstowym bazy danych nie będzie linijek od opcji, które są
pokazane w `gconf-editor` .

## Odtwarzanie bazy danych dconf

Jeśli w powyższym pliku widzimy sekcje odnoszące się do programów, których nie mamy już w systemie,
możemy takie bloki kodu usunąć. Dodatkowo, możemy także masowo zmienić konfigurację wszystkich
widocznych tam parametrów -- zawsze to wygodniejsze niż szukanie po kluczach w GUI.

Dokonywanie zmian na bazie danych `dconf` wymaga usunięcia starego pliku `~/.config/dconf/user` , z
którego korzysta system. Po usunięciu tego pliku, trzeba ponownie zalogować się w sesji graficznej.
Ma to na celu reset ustawień, który zaowocuje utworzeniem nowego pliku `user` niezawierającego
praktycznie żadnej treści. Dopiero po ponownym zalogowaniu możemy przywrócić zmienioną bazę danych.
Jeśli nie przeprowadzimy tych czynności, to po odtworzeniu bazy i ponownym zalogowaniu się,
wszystkie zmiany, które wprowadziliśmy, zostaną zresetowane.

By zbudować bazę wpisujemy w terminal poniższe polecenia:

    $ cd ~/.config/dconf/
    $ rm user

W tym miejscu restartujemy sesję graficzną. Po zalogowaniu się, ponownie przechodzimy do katalogu
`~/.config/dconf/` i przeprowadzamy poniższe kroki:

    $ cd ~/.config/dconf/
    $ dconf load / < ./database
    $ dconf update /

Teraz już tylko restartujemy sesję graficzną. I to w zasadzie wszystko.


[1]: https://wiki.gnome.org/action/show/Projects/dconf
[2]: https://askubuntu.com/questions/249887/gconf-dconf-gsettings-and-the-relationship-between-them
[3]: https://askubuntu.com/questions/34490/what-are-the-differences-between-gconf-and-dconf
[4]: https://wiki.archlinux.org/index.php/GNOME_package_guidelines#GSettings_schemas
[5]: https://developer.gnome.org/dconf/unstable/dconf-overview.html
