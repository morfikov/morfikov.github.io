---
author: Morfik
categories:
- Linux
date: "2015-11-06T01:08:18Z"
date_gmt: 2015-11-06 00:08:18 +0100
published: true
status: publish
tags:
- bash
title: Plik .bash_history, czyli historia poleceń bash'a
---

Operowanie na linux'ie wiąże się w dużej mierze z wpisywaniem poleceń do terminala. Każdy kto
spędził trochę czasu w tym systemie, wie, że do komfortowej pracy potrzebny jest przyzwoicie
skonfigurowany shell. Domyślnym shell'em w debianie, jak i wielu innych linux'ach, jest bash. Każdy
z nas na początku wpisywał wszystkie polecenia ręcznie i nawet nie wiedział, że istnieje coś takiego
jak uzupełnianie pewnych fraz, czy też nazw, przy pomocy klawisza Tab . Z czasem nasz stopień
poznania jakiejś dystrybucji linux'a osiąga pewien dość zaawansowany poziom i wpisywanie za każdym
razem tych samych poleceń jedynie spowalnia naszą pracę. Dlatego właśnie bash, podobnie jak i inne
shell'e, mają swoje pliki konfiguracyjne, w których to możemy [dostosować naprawdę sporo rzeczy][1].
W tym wpisie skupimy się na historii poleceń, która trafia do pliku `.bash_history` w katalogu
domowym każdego użytkownika w systemie.

<!--more-->
## Konfiguracja bash'a w plik ~/.bashrc

Bash'a możemy konfigurować globalnie w pliku `/etc/bash.bashrc` lub też lokalnie w pliku
`~/.bashrc` . Jako, że te wszystkie poniższe parametry są raczej specyficzne dla konkretnego
użytkownika w systemie, to nie zalecałbym ich umieszczania w globalnym pliku konfiguracyjnym.
Dlatego też będziemy operować jedynie na pliku `~/.bashrc` i to tam będziemy umieszczać wszystkie
poniższe parametry. Po edycji tego pliku konfiguracyjnego, trzeba przeładować shell by zmiany
zaczęły obowiązywać. Możemy to zrobić poniższym poleceniem:

    $ source ~/.bashrc

### Ilość wpisów w pliku historii (HISTFILESIZE)

Zacznijmy zatem od parametru, który określa ilość wpisów w pliku `.bash_history` . Odpowiada za to
zmienna `$HISTFILESIZE` . Jeśli chcielibyśmy by w tym pliku mogło znaleźć się 20.000 ostatnich
poleceń, które wpisaliśmy w terminalu, to dopisujemy tę poniższą linijkę:

    HISTFILESIZE=20000

### Ilość wpisów ładowanych do pamięci (HISTSIZE)

Trzeba jednak pamiętać, że polecenia mogą być dość długie i plik `.bash_history` może swoje ważyć.
Dlatego też bash oferuje wczytywanie do pamięci tylko części tego pliku. Ile to będzie wpisów,
możemy określić przy pomocy zmiennej `$HISTSIZE` . Jeśli chcemy by bash ładował jedynie 1000
ostatnich poleceń, to dopisujemy ten poniższy parametr:

    HISTSIZE=1000

### Dodawanie wpisów do historii (histappend)

Bash domyślnie podmienia plik `.bash_history` po zakończeniu sesji i dopiero po wylogowaniu się,
wszystkie polecenia, które wpisaliśmy w tej sesji, trafią do tego pliku. Powoduje to problemy w
momencie gdy jesteśmy zalogowani w paru terminalach jednocześnie. W takim przypadku, gdy wykonujemy
polecenia w tych terminalach na zmianę, to po ich zamknięciu, historia poleceń będzie niekompletna i
zawierać jedynie polecenia z terminala, który został zamknięty jako ostatni. Możemy jednak temu
zaradzić poprzez ustawienie opcji `histappend` , która sprawi, że bash będzie dodawał polecenia do
pliku `.bash_history` zamiast go ciągle nadpisywać.

    shopt -s histappend

### Rzeczywiste aktualizowanie pliku historii (PROMPT\_COMMAND)

Jeśli nie odpowiada nam domyślna polityka bash'a odnośnie dodawania poleceń do pliku `.bash_history`
dopiero na końcu sesji, to możemy również i ten aspekt sobie dostosować. Problem z domyślnym
zachowaniem bash'a w tej kwestii jest taki, że bardzo często polecenia nie są zapisywane. Możemy się
z tym spotkać przy restartowaniu czy zamykaniu systemu mając otwarty jakiś terminal. Innym problemem
może być fakt, że operując na jednym terminalu nie widzimy poleceń, które wprowadziliśmy w drugim
terminalu. By nakazać bash'owi zapisywanie poleceń za każdym razem gdy tylko jakieś wpisujemy do
terminala, musimy odpowiednio ustawić zmienną `$PROMPT_COMMAND` :

    PROMPT_COMMAND="history -a; history -c; history -r; $PROMPT_COMMAND"

Po co wywołujemy tyle razy `history` ? Parametr `-a` natychmiast dodaje wpisane polecenie do pliku
historii. Z kolei parametr `-c` czyści bufor historii dla aktualnej sesji. Na koniec mamy jeszcze
parametr `-r` , który wczytuje plik `.bash_history` do pamięci i ładuje przy tym wszystkie ostatnio
wpisywane polecenia. Więcej informacji jest
[tutaj][2].

### Ignorowanie duplikatów (ignoredups)

Wpisując szereg poleceń pod rząd, może się zdarzyć tak, że kilka z nich będzie takich samych.
Wszystkie one zostaną dopisane do pliku `.bash_history` , co raczej nie zawsze jest pożądanym
zachowaniem. Za ignorowanie duplikatów odpowiada parametr `ignoredups` . Po jego dodaniu, polecenie,
które wpisujemy w terminalu, nie będzie dodawane do pliku historii tylko w przypadku gdy są w nim
już takie same wpisy. Jeśli interesuje nas taki mechanizm, to dopiszmy do konfiguracji bash'a tę
poniższą linijkę:

    HISTCONTROL=ignoredups

Zmienna `$HISTCONTROL` może zawierać wiele opcji. Jeśli mamy już zdefiniowane jakieś, to musimy je
oddzielić za pomocą `:` , przykładowo: `HISTCONTROL=ignoredups:ignorespace` .

### Czyszczenie pliku .bash\_history z duplikatów (erasedups)

Dodanie parametru `erasedups` sprawi, że wszystkie wpisy w pliku `.bash_history` , które pasują do
aktualnie wprowadzanego polecenia, zostaną usunięte przed dodaniem tego nowego wpisu. Jeśli
interesuje nas takie zachowanie, do to konfiguracji bash'a dopisujemy tę linijkę:

    HISTCONTROL=erasedups

### Ignorowanie poleceń poprzedzonych spacją (ignorespace)

Jeśli tylko sporadycznie chcemy aby dane polecenie nie trafiło do historii poleceń, to możemy
wprowadzić taką politykę, że te polecenia, które zostały poprzedzone spacją, będą wyjęte spod
mechanizmu historii bash'a. Wydaje się to być o wiele lepszym rozwiązaniem niż precyzowanie na
sztywno poleceń w `HISTIGNORE` . Dopiszmy zatem ten poniższy wpis do konfiguracji bash'a:

    HISTCONTROL=ignorespace

### Ignorowanie określonych poleceń (HISTIGNORE)

Są takie polecenia, których nie chcemy zapisywać w pliku `.bash_history` . Weźmy na przykład
`exit` . Czy naprawdę powinno ono trafić do pliku historii? Być może mamy szereg innych poleceń,
których nie chcemy umieszczać w tym pliku. Dla tych i wszelkich innych poleceń stworzono zmienną
`$HISTIGNORE` , w której możemy określić, jakie polecenia mają być wyjęte spod mechanizmu historii
bash'a przykładowo.

    HISTIGNORE="cat*:cd*:exit"

W taki sposób, wszystkie wpisy, które zaczynają się od `cat` , `cd` nie trafią do pliku historii. To
samo tyczy się polecenia `exit` .

### Data i godzina wpisów w pliku .bash\_history (HISTTIMEFORMAT)

Domyślnie wszystkie wpisy, które trafiają do pliku `.bash_history` nie są oznaczone czasowo. W ten
sposób nie jesteśmy w stanie ustalić kiedy jakieś polecenie zostało wklepane do terminala. Jeśli
chcielibyśmy mieć taką możliwość, to musimy odpowiednio ustawić sobie zmienną `$HISTTIMEFORMAT` .
Dodajmy zatem do konfiguracji bash'a ten poniższy wpis:

    HISTTIMEFORMAT="%F %T "

Format daty można wyciągnąć z help'a `date` . W tym przypadku `%F` odpowiada za datę w formie
`2015-11-05` , zaś `%T` za czas w postaci `23:09:22` .


[1]: https://www.gnu.org/software/bash/manual/bash.html#Shell-Variables
[2]: https://unix.stackexchange.com/questions/18212/bash-history-ignoredups-and-erasedups-setting-conflict-with-common-history
