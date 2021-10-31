---
author: Morfik
categories:
- Linux
date: "2016-01-12T20:18:33Z"
date_gmt: 2016-01-12 19:18:33 +0100
published: true
status: publish
tags:
- pliki
- foldery
- mime
- debian
- openbox
GHissueID: 528
title: Domyślne aplikacje w oparciu o typy plików (MIME)
---

Pełne środowiska graficzne zwykle oferują odpowiednie narzędzia, które mogą posłużyć skonfigurowaniu
domyślnych aplikacji. Jesteśmy zatem w stanie bardzo prosto przypisać szereg [typów MIME][1] do
odpowiednich programów. Wobec takiego stanu rzeczy, przy próbie uruchomienia jakiegoś pliku, ten
zostanie odpalony przez konkretną aplikację. Niemniej jednak, jeśli chodzi o środowiska graficzne
oparte o menadżery okien, np. Openbox, to na dobrą sprawę mamy bardzo małe pole manewru.
Niezależnie czy korzystamy z GNOME, KDE, XFCE, MATE czy też Openbox'a, wszystkie z nich
wykorzystują dokładnie ten sam mechanizm wiązania aplikacji z typami plików. Możemy zatem ominąć te
wszystkie graficzne nakładki i ręcznie skonfigurować sobie typy MIME, tak by działały niezależnie
od wykorzystywanego środowiska graficznego. W tym wpisie spróbujemy przyjrzeć się nieco bliżej
konfiguracji tego całego mechanizmu.

<!--more-->
## Jak ustalić typ MIME

W linux'ie każdy plik ma swój typ. Dlatego też małe znaczenie mają rozszerzenia pliku, a o wiele
bardziej liczy się to co siedzi w jego nagłówku. Na dobrą sprawę, to chyba każdy menadżer plików
jest nam w stanie określić typ MIME. Te informacje widnieją zwykle we właściwościach tego pliku.
Poniżej okienko właściwości na przykładzie menadżera plików SpaceFM:

![spacefm-wlasciwosci-mime-typ](/img/2016/01/1.spacefm-wlasciwosci-mime-typ.png#medium)

Mamy tutaj informację, że jest to dokument PDF, a jego typ MIME to `application/pdf` . Widzimy
również jaka aplikacja jest ustawiona jako domyślna dla tego typu plików i w tym przypadku jest to
`Okular` .

Innym sposobem na ustalenie typu MIME jest skorzystanie z tekstowych narzędzi, które w debianie są
zawarte w pakiecie `xdg-utils`, przykładowo:

    $ xdg-mime query filetype plik
    application/pdf

Dzięki tym typom, szereg aplikacji może zostać przypisanych określonym rodzajom plików, co wygląda
mniej więcej tak:

![spacefm-powiazania-typ-mime](/img/2016/01/2.spacefm-powiazania-typ-mime.png#medium)

## Domyślne aplikacje dla poszczególnych typów MIME

Wiemy już zatem jak ustalić typ MIME. Spróbujmy wobec tego dowiedzieć się na jakiej zasadzie jest
dokonywane powiązanie typu `application/pdf` z aplikacją `Okular` . Jeśli odpytamy system o domyślną
aplikację dla tego typu, dostaniemy mniej więcej taką wiadomość:

    $ xdg-mime query default application/pdf
    kde4-okularApplication_pdf.desktop

Mamy tutaj informację, że za ten typ MIME jest odpowiedzialny plik
`kde4-okularApplication_pdf.desktop` . Nie jest to faktyczna aplikacja ale jedynie sam
[plik .desktop][2], który znajdować się może w katalogu `/usr/share/applications/` lub też jego
lokalnym odpowiedniku, tj. `~/.local/share/applications/` . Plik, o którym mowa wygląda mniej
więcej tak:

    [Desktop Entry]
    MimeType=application/pdf;application/x-gzpdf;application/x-bzpdf;application/x-wwf;
    Terminal=false
    Name=Okular
    Name[pl]=Okular
    GenericName=Document Viewer
    GenericName[pl]=Przeglądarka dokumentów
    Exec=okular %U %i -caption %c
    Icon=okular
    Type=Application
    InitialPreference=8
    Categories=Qt;KDE;Graphics;Viewer;
    X-KDE-Keywords=PDF, Portable Document Format
    X-KDE-Keywords[pl]=PDF, Przenośny Format Dokumentu
    NoDisplay=true

W `MimeType` są określone wszystkie typy MIME, które mają być przypisane do aplikacji, która jest
wywoływana w linijce z `Exec` . Nie wszystkie aplikacje posiadają opcję `MimeType` w swoich plikach
`.desktop` . Są również takie programy, które w ogóle nie posiadają plików `.desktop` . Powoduje to
problemy z późniejszym przypisywaniem aplikacji, bo system zwyczajnie ich nie bierze pod uwagę. Nic
jednak nie stoi na przeszkodzie, by napisać sobie taki plik i umieścić go w katalogu
`~/.local/share/applications/` .

## Określanie domyślnych aplikacji

Często się zdarzy tak, że jako domyślna aplikacja dla danego typu pliku nie zostanie określona ta,
którą my byśmy sobie życzyli. Przydałoby się zatem to zmienić. Znając typ MIME oraz nazwę pliku
`.desktop` możemy przy pomocy `xdg-mime` ustawić domyślną aplikację. Robimy to w poniższy sposób:

    $ xdg-mime default geany.desktop text/x-shellscript

Narzędzie `xdg-mime` operuje na pliku `mimeapps.list` , który zwykle jest ulokowany w katalogu
`~/.config/` lub `~/.local/share/applications/` (ta druga lokalizacja jest już przestarzała).
Struktura tego pliku jest następująca:

    [Default Applications]
    image/png=viewnior.desktop
    ...

    [Added Associations]
    text/html=geany.desktop;Firefox.desktop;
    ...

    [Removed Associations]
    application/octet-stream=smplayer.desktop;
    ...

Mamy generalnie trzy różne bloki, z których `[Default Applications]` odpowiada za domyślne
aplikacje, `[Added Associations]` określa dodatkowe powiązania (niekoniecznie określone w pliku
`.desktop` ) , a `[Removed Associations]` odpowiada za usunięcie powiązań. Typy MIME są oddzielone
od nazw aplikacji za pomocą znaku `=` . Jeśli wiele aplikacji ma być powiązanych lub usuniętych,
można je precyzować jedna za drugą oddzielając je od siebie za pomocą `;` .

Może się zdarzyć tak, że nazwa pliku `mimeapps.list` w środowiskach graficznych przybierze nieco
inną nazwę:

  - `gnome-mimeapps.list` -- dla środowiska GNOME
  - `xfce-mimeapps.list` -- dla środowiska XFCE
  - `kde-mimeapps.list` -- dla środowiska KDE

Można naturalnie potworzyć odpowiednie dowiązania, tak by wszystkie wpisy zawsze były czytane tylko
z jego pliku i by nam ta konfiguracja się nie rozjechała i nie była inna w przypadku aplikacji
specyficznych dla określonego środowiska graficznego.

## Narzędzia do konfiguracji typów MIME

Warto też wspomnieć, że istnieją menadżery plików, np. SpaceFM, które są w stanie dokonać całej
konfiguracji typów MIME za nas. Jest to o wiele wygodniejsze niż ręczna edycja konkretnych plików,
no i unikniemy w ten sposób literówek czy jakichś większych błędów. Poniżej jest zrzut konfiguracji
w SpaceFM:

![spacefm-edycja-typ-mime](/img/2016/01/3.spacefm-edycja-typ-mime.png#big)

Każdy powiązany z plikiem typ MIME jesteśmy w stanie poddać edycji klikając na jego pozycji prawym
przyciskiem myszki. Jesteśmy w stanie edytować pliki zarówno w katalogu `/usr/share/applications/`
jak i te w `~/.local/share/applications/` czy `~/.config/` . Możemy także w łatwy sposób kopiować
odpowiednie pozycje, po których to skopiowaniu zostanie do naszej dyspozycji oddany edytor tekstu w
celu odpowiedniego dostosowania danego pliku.

## Zmienne środowiskowe

Szereg narzędzi, takich jak `xdg-open` , potrzebuje do poprawnego działania odpowiednich zmiennych
środowiskowych. W przypadku pełnych środowisk graficznych, te zmienne są automatycznie
dostosowywane i nie musimy się o nic martwić. Jeśli jednak korzystamy jedynie z Openbox'a, czy też
innego menadżera okien, to może okazać się, że część programów nadal ma problemy z rozpoznaniem
domyślnych aplikacji.

Dwie takie najbardziej problematyczne zmienne, to `GNOME_DESKTOP_SESSION_ID` oraz `DE` . Pierwsza z
nich zwykle jest ustawiana w celu unifikacji wyglądu ikonek w aplikacjach GTK i QT. Zwykle też za
jej sprawą system błędnie rozpoznaje środowisko GNOME i dlatego mogą pojawić się problemy z
określeniem domyślnych aplikacji w oparciu o typy MIME. Jeśli chodzi zaś o zmienną `DE` , to w
oparciu o jej wartość, system decyduje z jakich narzędzi ma korzystać przy odpalaniu plików.

W środowisku Openbox'a, zmienna `DE` nie jest zdefiniowana. By ją ustawić ręcznie, edytujemy plik
`~/.config/openbox/environment` i dodajemy w nim tę poniższą linijkę:

    export DE="xfce"

Ta zmienna niekoniecznie musi wskazywać na środowisko `Xfce` . Mamy także do wyboru kilka innych
wartości: `gnome` , `kde` , `lxde` oraz `mate` . Niemniej jednak, w przypadku minimalistycznego
systemu, ustawienie wartości `xfce` jest chyba jedyną opcją. Warto też wiedzieć, że w zależności od
wybranej tutaj wartości, musimy doinstalować dodatkowe pakiety. W przypadku `xfce` jest to pakiet
`exo-utils` .


[1]: https://pl.wikipedia.org/wiki/Typ_MIME
[2]: https://standards.freedesktop.org/desktop-entry-spec/desktop-entry-spec-latest.html
