---
author: Morfik
categories:
- Linux
date: "2016-06-09T12:44:12Z"
date_gmt: 2016-06-09 10:44:12 +0200
published: true
status: publish
tags:
- pliki/foldery
- find
title: Jak odszukać pliki utworzone godzinę temu (find)
---

W linux'ie wszystko może być reprezentowane za pomocą pliku. Te pliki są tworzone, zmieniane i
usuwane praktycznie non stop podczas pracy systemu operacyjnego. Zwykle też nie zwracamy uwagi na
metadane opisujące pliki w systemie plików, bo przecie bardziej interesuje nas ich zawartość.
Niemniej jednak, w pewnych sytuacjach te metadane mogą się okazać bardzo użyteczne. Weźmy sobie
przykład partycji `/home/` . Prawie zawsze po odpaleniu aplikacji są tworzone na niej nowe pliki lub
zmieniane te już istniejące. Jako, że nie zawsze wiemy na jakich plikach operuje dany program, to
moglibyśmy przeszukać pliki w katalogu domowym użytkownika i poddać analizie ich czas modyfikacji.
Taką operacje możemy przeprowadzić przy pomocy polecenia `find` i w tym wpisie zobaczymy jak tego
dokonać.

<!--more-->
## Pliki i ich data/czas modyfikacji

Każdy system plików przechowuje w swojej strukturze informacje min. na temat czasu modyfikacji
pliku. W przypadku, gdy plik zostaje utworzony lub zmieniony w jakiś sposób, system przepisuje ten
czas i ustawia go na aktualną datę i godzinę. Jeśli chcielibyśmy się zaznajomić z informacją o tych
znacznikach czasu, to możemy zajrzeć w wyjście polecenia `stat` , przykładowo:

![]({{< baseurl >}}/img/2016/06/1.find-stat-czas-modyfikacji-pliku.png)

Te trzy pozycje na dole, tj. `Access:` , `Modify:` i `Change:` odpowiadają kolejno za ostatni czas
dostępu do pliku, czas modyfikacji pliku i czas zmiany i-węzła odpowiedzialnego za plik.
Niekoniecznie wszystkie te trzy czasy muszą być takie same jak w tym przypadku powyżej, bo to zależy
od operacji przeprowadzanych na pliku. Tak czy inaczej, nas interesuje tutaj głównie czas
modyfikacji, w oparciu o który możemy z powodzeniem powiedzieć, czy dany plik został zmodyfikowany w
ostatnim czasie.

## Jak odnaleźć modyfikowane pliki via find

W debianie mamy pakiet `findutils` , w którym jest zawarte narzędzie `find` . Przy jego pomocy
jesteśmy w stanie przeszukiwać pliki na dysku w oparciu o całą masę dopasowań. Mamy także kilka
opcji dotyczących tych wszystkich widocznych wyżej znaczników czasów. Poniżej jest krótka rozpiska
opcji, które przydadzą nam się w procesie szukania plików po czasie modyfikacji. Jako, że na fotce
widzieliśmy trzy różne czasy, to w `find` mamy również trzy parametry:

  - `-ctime` -- dopasowanie w oparciu o czas zmiany i-węzła.
  - `-mtime` -- dopasowanie w oparciu o czas modyfikacji pliku.
  - `-atime` -- dopasowanie w oparciu o czas dostępu do pliku.

Problem z tymi opcjami jest taki, że jeśli byśmy ustawili w nich wartość, np. `1` , to zostanie ona
pomnożona przez 24 godziny. Istnieje także opcja `-daystart` , którą można wykorzystać z tymi
powyższymi. W ten sposób, czas będzie liczony od początku aktualnego dnia, zamiast 24h wstecz od
aktualnego czasu. Może i to rozwiązanie jest dobre w przypadku nieco bardziej rozległych dat ale
nasuwa się pytanie jak wyszukać pliki zmodyfikowana, np. godzinę temu? Trzeba skorzystać z tego
poniższego zestawu parametrów:

  - `-cmin` -- dopasowanie w oparciu o czas zmiany i-węzła.
  - `-mmin` -- dopasowanie w oparciu o czas modyfikacji pliku.
  - `-amin` -- dopasowanie w oparciu o czas dostępu do pliku.

Wartość, w oparciu o którą chcemy przeszukać pliki, we wszystkich w/w opcjach można określić na trzy
sposoby:

  - `1` -- pliki modyfikowane dokładnie jeden dzień/minutę temu.
  - `-1` -- pliki modyfikowane maksymalnie jeden dzień/minutę wstecz w porównaniu do aktualnego
    czasu.
  - `+1` -- pliki modyfikowane minimum jeden dzień/minutę wstecz w porównaniu do aktualnego czasu.

## Wyszukiwanie modyfikowanych plików

Jak zatem będą wyglądać polecenia, które odszukają nam pliki modyfikowane w określonym przez nas
terminie? To generalnie zależy jaki czas nas interesuje. Możemy także podać przedział czasu. Jeśli
chcemy odszukać pliki w katalogu `/home/` , które były modyfikowane jedną godzinę temu, to w
terminalu wpisujemy poniższe polecenie:

    $ find /home/ -mmin -60

Jeśli chcemy, odszukać pliki modyfikowane w ostatnich 3 dniach, to możemy naturalnie skorzystać
również z `-mmin` . NIemniej jednak, przeliczanie dni na minuty to tylko kłopot. Dlatego też lepiej
korzystać z `-mtime` :

    $ find /home/ -mtime -3

Z kolei jeśli interesują nas pliki, które były modyfikowane minimum 10 dni temu, to wpisujemy
poniższe polecenie:

    $ find /home/ -mtime +10

Przedziały czasu określamy podając dwa razy `-mtime` . Dla przykładu, chcemy uzyskać listę plików,
które były modyfikowane 2-5 dni temu:

    $ find /home/ -mtime +2 -mtime -5

Oczywiście, opcje `-ctime` , `-mtime` , `-atime` , `-cmin` , `-mmin` oraz `-amin` można łączyć z
pozostałymi parametrami polecenia `find` uzyskując w ten sposób, np. tylko pliki `.pdf` , które były
modyfikowane 2-5 dni temu. Poniżej przykład:

    $ find /home/ -mtime +2 -mtime -5 -type f -iname "*.pdf"

Więcej informacji na temat tych w/w parametrów jak i całej masy innych opcji można znaleźć w [man
find.](http://manpages.ubuntu.com/manpages/xenial/en/man1/find.1.html)
