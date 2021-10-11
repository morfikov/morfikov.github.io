---
author: Morfik
categories:
- Linux
date: "2016-06-06T10:47:58Z"
date_gmt: 2016-06-06 08:47:58 +0200
published: true
status: publish
tags:
- terminal
- tmux
GHissueID: 356
title: FZF (fuzzy finder) dla tmux'a
---

Jakiś czas temu pisałem o [implementacji tmux'a na
debianie](/post/implementacja-multipleksera-tmux/). Dla tych, którzy nie za bardzo
wiedzą [czym tmux jest](https://tmux.github.io/), to wyjaśniam, że jest to narzędzie, które min.
potrafi dzielić okno terminala na kilka mniejszych okienek. W ten sposób możemy korzystać ze swojego
ulubionego terminala i cieszyć się funkcjonalnością znaną choćby z terminatora. Ta umiejętność
dzielenia okien w tmux idealnie współgra z [FZF (fuzzy finder)](https://github.com/junegunn/fzf).
Jest to narzędzie, które pomaga min. przeszukiwać historię poleceń shell'a. BASH czy ZSH są, jakby
nie patrzeć, dość ograniczone pod tym względem. Może i ZSH nieco lepiej radzi sobie z odnajdywaniem
poleceń w historii od BASH ale i tak jego umiejętności pozostają daleko w tyle za FZF. Dlatego też w
tym artykule spróbujemy sobie zainstalować ten cały "fuzzy finder".

<!--more-->
## Instalacja FZF za pomocą git

FZF jest trzymany w repozytorium na github'ie i najprostszą metodą zainstalowania tego narzędzia
jest sklonowanie jego repo do katalogu domowego. Potrzebny nam będzie zatem pakiet `git` , który
jest standardowo dostępny w debianie i nie powinno być problemów z jego pozyskaniem. Następnie
przechodzimy do katalogu domowego i klonujemy repozytorium FZF:

    $ git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf

I instalujemy to narzędzie:

    $ ~/.fzf/install

Podczas tego procesu zostaniemy poproszeni o udzielenie odpowiedni na kilka pytań:

![](/img/2016/06/1.fzf-tmux-instalacja.png#huge)

Cała niezbędna konfiguracja w przypadku shell'a BASH i ZSH została automatycznie uwzględniona w ich
plikach konfiguracyjnych i nie musimy przeprowadzać żadnych dodatkowych czynności. Musimy jedynie
załadować plik `~/.bashrc` lub `~/.zshrc` ponownie przy pomocy jednego z poniższych poleceń:

    # source ~/.bashrc
    # source ~/.zshrc

## Przeszukiwanie historii poleceń z FZF w tmux

Pora zobaczyć jak działa FZF i dlaczego idealnie nadaje się do zastosowania go w przypadku tmux'a. W
zależności od wykorzystywanego shell'a mamy oddelegowany jakiś skrót klawiszowy do aktywacji FZF. W
tym przypadku jest wykorzystywany ZSH i skrót Ctrl-R . Odpalmy zatem terminal z tmux'em i
przyciśnijmy ten skrót:

![](/img/2016/06/2.fzf-tmux-przeszukiwanie-historia-shell-zsh.png#huge)

Jak widzimy, okno terminala zostało podzielone na dwie części. W dolnym oknie mamy zainicjowany FZF,
gdzie mamy kilka ostatnich poleceń shell'a ZSH. Liczby po lewej stronie to pozycje w historii. Obok
nich mamy polecenia, które zostały wcześniej wpisane do terminala. Tą listę możemy przeszukać. Z
tym, że mamy kilka dodatkowych znaków, które są nam w stanie pomóc dopasować wzór wyszukiwania. Są
to te poniższe:

  - `^abc` -- dopasowuje pozycje w historii zaczynające się od `abc` .
  - `abc$` -- dopasowuje pozycje w historii kończące się na `abc` .
  - `'abc` -- dopasowuje pozycje w historii zawierające dokładne dopasowanie `abc` .
  - `!abc` -- dopasowuje pozycje w historii, które nie pasują do `abc` .
  - `!'abc` -- dopasowuje pozycje w historii które nie zawierają `abc` .

W taki sposób jesteśmy w stanie bardzo szybko odszukać pozycje w historii zawierające, np. polecenie
`cd` oraz katalog `debian` :

![](/img/2016/06/3.fzf-tmux-przeszukiwanie-historia-shell-zsh.png#huge)

Zaznaczając pozycję i przyciskając Enter , zostanie ona wybrana i wklejona do okienka powyżej:

![](/img/2016/06/4.fzf-tmux-wybor-pozycja-historia.png#huge)

Jak widzimy, okienko z FZF zniknęło automatycznie po wciśnięciu klawisza Enter . Dlatego właśnie ten
mechanizm jest tak wielce użyteczny w przypadku tmux'a.

## Zmiana katalogów

Jeśli ta powyższa mechanika nas lekko irytuje w przypadku przeglądania struktury danego katalogu, to
zawsze możemy skorzystać ze skrótu Alt-C . Odpowiada on za przeglądanie i przechodzenie do
podrzędnych katalogów względem tego, w którym się aktualnie znajdujemy. By zobrazować jak działa
ten mechanizm, przejdźmy do przykładowego katalogu i przyciśnijmy ten w/w skrót klawiszowy:

![](/img/2016/06/5.fzf-tmux-cd-katalog.png#huge)

Okno terminala zostało podzielone tak jak poprzednio ale zwrócone pozycje już są nieco inne. Na
listingu są uwzględnione tylko katalogi podrzędne. W dalszym ciągu działa przeszukiwanie tych
pozycji ale po wybraniu którejś z nich, zostaniemy automatycznie przeniesieni do tego katalogu:

![](/img/2016/06/6.fzf-tmux-cd-przejscie-katalog.png#huge)

## Auto uzupełnianie dla FZF

FZF dysponuje także auto uzupełnianiem poleceń. Załóżmy, że chcemy zmienić obecny katalog. Chcemy
także przejść do jakiegoś podkatalogu. Sytuacja jest mniej więcej podobna do tej opisanej powyżej, z
tą różnicą, że w tym przypadku zastosujemy auto uzupełnianie polecenia przy pomocy `**` oraz
klawisza Tab . Ten mechanizm działa w następujący sposób. Po przejściu do konkretnego katalogu
wpisujemy jakąś frazę i za nią dwie gwiazdki. Dla przykładu, znajdujemy się w katalogu domowym i
chcemy przejść pod `~/.config/autostart/` . Jak tylko przyciśniemy klawisz Tab po `**` , to otworzy
nam się lista pasujących podkatalogów:

![](/img/2016/06/7.fzf-tmux-auto-uzupelnianie.png#huge)

W dalszym ciągu tę listę możemy przeszukać jeśli wynik nie spełnia naszych oczekiwań. Trzeba jednak
liczyć się z faktem, że im więcej plików/folderów znajduje się w jakimś katalogu, tym wolniej
uzyskamy pełną listę z wynikami.

## Ubijanie procesów z FZF

Poza przeszukiwaniem historii poleceń oraz przechodzeniem do określonych katalogów, FZF potrafi
także ułatwić nam zabijanie procesów. Zwykle użytkownicy różnych dystrybucji linux'a używają do
tego celu polecenia `killall -9` i podają nazwę procesu. Jeśli nie znamy dokładnej nazwy procesu, to
możemy mieć problem z ubiciem konkretnej aplikacji. Oczywiście można także korzystać ze zwykłego
`kill` i podawać PID procesu. PID z kolei nie jest zbytnio akceptowaną formą nazywania procesów z
ludzkiego punktu widzenia i przeciętny człowiek musiałby się posiłkować dodatkowymi aplikacjami,
które pomogą ten numer identyfikacyjny ustalić. Względnie można by skorzystać z `pidof` . FZF łączy
ze sobą wszystkie te funkcjonalności i umożliwia nam przeszukiwanie procesów w czasie rzeczywistym i
wybieranie do ubicia tych procesów, których nazwy dobrze nam są znane.

Dla przykładu otworzyłem kilka okien edytora `geany` i przydałoby się ubić wszystkie te procesy. W
tym celu w teminalu wpisujemy słówko `kill` i przyciskamy klawisz Tab . Oczywiście, możemy podać kod
sygnału, który chcemy przesłać do procesu. Okno terminala powinno zostać podzielone i w dolnej
części powinniśmy ujrzeć procesy działające w systemie:

![](/img/2016/06/8.fzf-tmux-kill-procesy.png#huge)

Listę tych procesów możemy standardowo przeszukać. W tym przypadku szukamy `geany` :

![](/img/2016/06/9.fzf-tmux-kill-procesy.png#huge)

Jako, że mamy kilka procesów, to zaznaczamy/odznaczamy je przy pomocy Tab i Shift-Tab . Po
zaznaczeniu przyciskamy Enter . W poleceniu `kill` powinny zostać uwzględnione PID'y tych procesów.
Po wydaniu polecenia `kill` , wszystkie te procesy powinny zostać ubite.

![](/img/2016/06/10.fzf-tmux-kill-procesy.png#huge)

## Zmienne sterujące FZF

Zachowanie oraz wygląd FZF można dostosować przy pomocy odpowiednich zmiennych środowiskowych.
Obecnie są obsługiwane te poniższe zmienne:

  - `$FZF_TMUX` -- precyzuje czy korzystamy z tmux'a, domyślnie `1` .
  - `$FZF_TMUX_HEIGHT` -- określa proporcje podziału okna w tmux, domyślnie `40%` będzie
    przeznaczone dla FZF.
  - `$FZF_COMPLETION_TRIGGER` -- definiuje trigger aktywujący auto uzupełnianie, domyślnie `**` .
  - `$FZF_COMPLETION_OPTS` -- określa opcje, które są dodawane przy wywoływaniu FZF.

Wszystkie te powyższe zmienne możemy sobie dostosować umieszczając je w pliku konfiguracyjnym
używanego przez nas shell'a. Poniżej jest przykład zmiennej `$FZF_COMPLETION_OPTS` ustawiającej
kolory dla FZF:

    export FZF_DEFAULT_OPTS="--color dark,hl:33,hl+:37,fg+:235,bg+:136,fg+:254"
