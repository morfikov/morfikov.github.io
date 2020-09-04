---
author: Morfik
categories:
- Linux
date: "2016-08-22T08:38:58Z"
date_gmt: 2016-08-22 06:38:58 +0200
published: true
status: publish
tags:
- firefox
- screenshot
title: Jak zrobić screenshot całej strony www w Firefox
---

Każdy z nas potrafi raczej zrobić prostego "skrina" tego co wyświetla się w danej chwili na ekranie
naszego komputera. Nie jest to jakaś zaawansowana wiedza i wystarczy przycisnąć przycisk PrintScreen
na klawiaturze i zrzut ekranu powinien zostać przechwycony przez system i zwykle gdzieś zapisany.
Niemniej jednak, strony www w przeglądarce internetowej bardzo rzadko są nam pokazywane w całej
swojej okazałości. Zwykle mamy po prawej stronie pasek przewijania (scrollbar), za pomocą którego
możemy przewinąć stronę w górę lub w dół. Pojawia się zatem pytanie: jak w takiej sytuacji zrobić
screenshot całej strony www? Można, co prawda, przewinąć stronę kilka razy, zrobić zrzut każdego
kawałka i scalić obraz w jakimś programie graficznym ale raczej za dużo z tym zachodu. Można także
zaprzęgnąć jakiś plugin do przeglądarki, np. Firefox ma na wyposażeniu [Awesome
Screenshot](https://addons.mozilla.org/en-us/firefox/addon/screenshot-capture-annotate/). Istnieje
jednak prostsza alternatywa i do tego natywnie zaimplementowana w Firefox'ie. Mowa o [wierszu
poleceń Firefox'a](https://developer.mozilla.org/en/docs/Tools/GCLI). W tym krótkim wpisie
zobaczymy jak przy pomocy tego narzędzia w bardzo prosty sposób zrobić fotkę całej witryny www.

<!--more-->
## Jak wywołać wiersz poleceń Firefox'a

Na dobrą sprawę, wiersz poleceń Firefox'a, to narzędzie kierowane do developerów. Niemniej jednak,
posiada ono kilka użytecznych funkcji, które mogą się przydać przeciętnemu użytkownikowi tej
przeglądarki. By skorzystać z tego wiersza poleceń, odpalamy przeglądarkę i przyciskamy na
klawiaturze Shift-F2 . Przy dolnej krawędzi okna przeglądarki powinien nam się pojawić pasek, w
którym możemy zacząć wpisywać jakiś tekst. Wygląda on mniej więcej tak:

![]({{< baseurl >}}/img/2016/08/1.firefox-screenshot-zrzut-ekranu.png#big)

To właśnie tutaj będziemy wpisywać różne polecenia.

## Jak zrobić screenshot przy pomocy wiersza poleceń Firefox'a

Zrobienie screenshot'a przy pomocy tego wiersza poleceń sprowadza się do wpisania w tym powyższym
okienku polecenia `screenshot` podając przy tym dwa argumenty. Pierwszym z nich jest nazwa pliku pod
jaką ma zostać zapisany zrzut ekranu. Drugim zaś jest `--fullpage` odpowiadający za zrobienie fotki
całej witryny, a nie jedynie obszaru, który aktualnie widzimy w oknie przeglądarki:

![]({{< baseurl >}}/img/2016/08/2.firefox-screenshot-zrzut-ekranu.png#big)

Nie musimy oczywiście ręcznie wpisywać całego polecenia. Działa tutaj auto uzupełnianie klawiszem
Tab i do tego mamy dość przyzwoicie opisaną każdą opcję, którą możemy wybrać sobie za pomocą
strzałek Up/Down . Gdy skończymy formułować polecenie, zatwierdzamy je przez wciśnięcie Enter .

Zrzut ekranu zostanie zapisany w katalogu, który mamy określony w ustawieniach Firefox'a dla
pobieranych plików. Pamiętajmy tylko, by nazwa pliku miała rozszerzenie `.png` .

W taki oto prosty sposób możemy pozbyć się zbędnych wtyczek, które jakby nie patrzeć zjadają nam
cenną pamięć RAM, i odciążyć nieco samą przeglądarkę.
