---
author: Morfik
categories:
- Linux
date: "2015-11-24T12:30:34Z"
date_gmt: 2015-11-24 11:30:34 +0100
published: true
status: publish
tags:
- bezpieczeństwo
- użytkownicy/grupy
title: Integralność plików passwd, group, shadow, gshadow
---

W pliku `/etc/passwd` jest przechowywana baza danych kont użytkowników w systemie linux. Z kolei w
`/etc/group` mamy wypisane wszystkie grupy oraz powiązanych z nimi użytkowników. Generalnie rzecz
biorąc, to te dwa pliki odpowiadają za konfigurację kont. Problem jednak zaczyna się w momencie gdy
w grę wchodzą hasła, zarówno do kont jak i do grup. Gdyby były one trzymane w tych plikach, nie
byłyby one w żaden sposób szyfrowane. Dlatego też powstał inny mechanizm, który ma na celu
przeniesienie zahashowanych haseł do plików `/etc/shadow` i `/etc/gshadow` . W debianie
użytkownikami i grupami możemy zarządzać przy pomocy odpowiednich narzędzi, które automatycznie
dostosują wszystkie powyższe pliki. Nic jednak nie stoi na przeszkodzie aby edytować każdy z nich
ręcznie. Problemy mogą się pojawić w momencie, gdy te pliki będą zawierać różne wpisy, np. w pliku
`passwd` będzie określony użytkownik, który jednocześnie nie będzie istniał w pliku `shadow` ,
podobnie z grupami. W tym wpisie postaramy się sprawdzić te pliki i upewnimy się czy aby na pewno
jest z nimi wszystko w porządku.

<!--more-->
## Narzędzia pwck i grpck

Do zarządzania kontami użytkowników mamy oddelegowanych kilka narzędzi. Przy pomocy
[adduser](http://manpages.ubuntu.com/manpages/wily/en/man8/adduser.8.html) możemy konta dodawać, a
za pomocą [userdel](http://manpages.ubuntu.com/manpages/wily/en/man8/userdel.8.html) je usuwać.
Dodatkowo mamy także [usermod](http://manpages.ubuntu.com/manpages/wily/en/man8/usermod.8.html),
przy pomocy którego to możemy edytować już te istniejące w systemie konta użytkowników. Podobnie
sprawa ma się w przypadku grup, tj. mamy odpowiednio
[groupadd](http://manpages.ubuntu.com/manpages/wily/en/man8/groupadd.8.html),
[groupdel](http://manpages.ubuntu.com/manpages/wily/en/man8/groupdel.8.html) oraz
[groupmod](http://manpages.ubuntu.com/manpages/wily/en/man8/groupmod.8.html). Dodatkowo do zmiany
hasła wykorzystujemy [passwd](http://manpages.ubuntu.com/manpages/wily/en/man1/passwd.1.html), a do
określenia shell'a [chsh](http://manpages.ubuntu.com/manpages/wily/en/man1/chsh.1.html). Możemy
także zmienić sobie informacje o koncie przy pomocy
[chfn](http://manpages.ubuntu.com/manpages/wily/en/man1/chfn.1.html).

Jeśli nie korzystamy z tego całego zestawu narzędzi, to jest niemal pewne, że prędzej czy później w
konfigurację użytkowników systemowych wkradnie się jakiś błąd. Mamy na szczęście do dyspozycji
również dwa inne ciekawe narzędzia:
[pwck](http://manpages.ubuntu.com/manpages/wily/en/man8/pwck.8.html) oraz
[grpck](http://manpages.ubuntu.com/manpages/wily/en/man8/grpck.8.html) . Pierwsze z nich zajmuje się
sprawdzeniem integralności plików `/etc/passwd` i `/etc/shadow` , drugie zaś weryfikuje wpisy w
`/etc/group` i `/etc/gshadow` .

W przypadku plików `passwd` i `shadow` , sprawdzana jest ilość pól w każdym wpisie, unikalność i
prawidłowość nawy użytkownika, identyfikatory UID/GID, główna grupa, katalog domowy oraz shell. Z
kolei jeśli chodzi o pliki `group` i `gshadow` , to tutaj sprawdzane są takie informacje jak
unikalność i prawidłowość nazw grup, identyfikatory grup oraz lista użytkowników i administratorów.
Każda z tych dwóch par plików musi posiadać dokładny zestaw wpisów, który musi do siebie pasować.

### Sprawdzenie plików /etc/passwd oraz /etc/shadow

By przeprowadzić integralność plików `/etc/passwd` oraz `/etc/shadow` , odpalamy terminal i logujemy
się na użytkownika root. Następnie wpisujemy `pwck` :

    # pwck
    user 'lp': directory '/var/spool/lpd' does not exist
    ...
    pwck: no changes

W moim przypadku pojawiło się jedynie kilka wpisów, które odnoszą się do nieistniejących katalogów
domowych pewnych użytkowników. W każdym razie, nic czym należałoby się przejmować. Jeśli byśmy mieli
jakieś poważniejsze błędy, to `pwck` powinien podjąć się próby ich naprawy. W przypadku gdyby nie
było to możliwe będzie próbował taki wpis usunąć z pliku.

Poprawek możemy naturalnie dokonać ręcznie, lub przy pomocy tego powyższego narzędzia. Niemniej
jednak, warto też wiedzieć, że do dyspozycji mamy jeszcze [pwconv i
pwunconv](http://manpages.ubuntu.com/manpages/wily/en/man8/pwconv.8.html), które potrafią dokonać
konwersji między plikami `passwd` oraz `shadow` w obu kierunkach.

### Sprawdzenie plików /etc/group /etc/gshadow

Podobnie sprawa ma się w przypadku sprawdzania integralności plików `/etc/group` i `/etc/gshadow` ,
z tym, że w terminalu wydajemy to poniższe polecenie:

    # grpck
    'morfik' is a member of the 'audio' group in /etc/gshadow but not in /etc/group

No i tutaj już widzimy klasyczny przykład niekorzystania z tych wszystkich wyżej wymienionych
narzędzi do zarządzania kontami użytkowników. W efekcie czego użytkownik `morfik` znajduje się w
grupie `audio` tylko w jednym z tych plików. Tego typu stan rzeczy należy bezzwłocznie poprawić.

Podobnie jak w przypadku plików `/etc/passwd` oraz `/etc/shadow` także możemy dokonać ręcznych
poprawek oraz mamy też do dyspozycji narzędzia [grpconv i
grpunconv](http://manpages.ubuntu.com/manpages/wily/en/man8/pwconv.8.html) , które potrafią dokonać
konwersji między plikami `group` oraz `gshadow` również w obu kierunkach.
