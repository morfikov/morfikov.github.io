---
author: Morfik
categories:
- Linux
date: "2019-04-12T19:40:40Z"
published: true
status: publish
tags:
- debian
- bezpieczeństwo
- sudo
title: Ograniczenie su do jednego użytkownika w Debianie
---

W dzisiejszych czasach dystrybucje linux'a wykorzystują mechanizm `sudo` do wykonywania operacji
jako administrator systemu. Zanika więc potrzeba stosowania polecenia `su` , by zalogować się na
konto root i to z jego poziomu wykonywać wszystkie niezbędne rzeczy. Jednym z argumentów
zwolenników `sudo` za takim sanem rzeczy jest możliwość nadania jedynie konkretnym użytkownikom w
systemie uprawnień do wykonywania poleceń jako administrator, podczas gdy inni użytkownicy
(niebędący w grupie `sudo` ) nie mogą w ogóle korzystać z tego mechanizmu . No faktycznie, dostęp
do `su` jest w zasadzie dla każdego użytkownika w systemie i tylko hasło konta admina dzieli ich od
uzyskania dość szerokich uprawnień. Niewiele jednak osób wie, że można skonfigurować `su` w taki
sposób, by dostęp do niego mieli tylko ci
użytkownicy, [którzy powinni](https://wiki.debian.org/WHEEL/PAM), np. ci obecni w grupie `wheel` .

<!--more-->
## Debian i grupa wheel

Próżno szukać grupy `wheel` w Debianie, bo tej grupy zwyczajnie tutaj nie ma. Dlatego też jeśli
chcemy skonfigurować `su` , tak by było ono dostępne tylko dla określonych użytkowników, to musimy
tę grupę sobie stworzyć sami. Odpalamy zatem terminal i wydajemy to poniższe polecenie jako root:

    # groupadd -g 5051 wheel

Warto w tym miejscu zaznaczyć, że nie może to być inna grupa niż `wheel` , bo mechanizm PAM tego od
nas wymaga. Zatem nie możemy wykorzystać już istniejącej grupy `sudo` , czy też stworzyć sobie
dowolnej grupy (chodzi o nazwę) i to jej wykorzystać w całym procesie ograniczenia możliwości
logowania się na root'a.

Wyżej został także użyty przełącznik `-g 5051` . Taki identyfikator numeryczny będzie miała grupa
`wheel` . Jeśli nie użyjemy konkretnego identyfikatora, to zostanie mu przypisany pierwszy wolny,
co w późniejszym czasie działania systemu (a konkretnie jego reinstalacji i odtwarzaniu backupu)
może generować problemy, bo sporo grup w Debianie nie ma stałych numerów i w zależności od
kolejności ich dodawania, te numerki mogą (i zapewne będą) się zmieniać.

Mając grupę `wheel` , trzeba do niej dodać tych użytkowników, którzy jako jedyni będą mieć dostęp
do polecenia `su` :

    # adduser morfik wheel

Sprawdźmy jeszcze czy user został poprawnie dodany:

    # id morfik
    uid=1000(morfik) gid=1000(morfik) groups=...5051(wheel)...

## Mechanizm PAM

By ograniczenie `su` zaczęło działać, musimy jeszcze skonfigurować mechanizm PAM. W tym celu
edytujemy plik `/etc/pam.d/su` . Odszukujemy w nim poniższą linijkę i usuwamy z jej początku znak
`#` :

    auth       required   pam_wheel.so

Warto też przejrzeć sobie cały ten plik `/etc/pam.d/su` , bo jest tam trochę opcji, które mogą nam
pomóc skonfigurować korzystanie z `su` .

## Test ograniczenia su

Wypadałoby jeszcze przetestować czy tylko ci użytkownicy w grupie `wheel` mają dostęp do polecenia
`su` :

![](/img/2019/04/001-debian-linux-su-sudo-wheel.png#huge)

No i jak widać użytkownik `morfik2` jest w stanie co prawda wydać polecenie `su` ale nawet po
podaniu prawidłowego hasła dostęp do konta root zostaje mu odmówiony z komunikatem o braku
uprawnień. Dodatkowo, w logu systemowym będzie rejestrowany komunikat z całego zdarzenia podobny do
tego poniżej:

    su[4485]: FAILED SU (to root) morfik2 on pts/9

Zatem wszystkie nieuprawnione próby logowania będzie można wyłapać i bardzo prosto ustalić czy coś
bez naszej zgody próbuje się logować na konto administratora systemu.
