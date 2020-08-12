---
author: Morfik
categories:
- Android
date: "2017-03-03T17:58:36Z"
date_gmt: 2017-03-03 16:58:36 +0100
published: true
status: publish
tags:
- szyfrowanie
- smartfon
- marshmallow
title: Jak zmienić hasło do zaszyfrowanego smartfona z Androidem
---

Każdy nowszy smartfon z Androidem oferuje możliwość zaszyfrowania wszystkich danych użytkownika
zlokalizowanych na partycji `/data/` . Cały proces można przeprowadzić w bardzo prosty sposób i bez
większych problemów. Raz zaszyfrowanego telefonu nie da rady cofnąć do stadium przed szyfrowaniem i
w zasadzie to zabezpieczenie można zdjąć jedynie przez przywrócenie urządzenia do ustawień
fabrycznych. My tutaj jednak nie będziemy zajmować się samym szyfrowaniem smartfona i skupimy się
bardziej na hasłach zabezpieczających mających stać na straży dostępu do naszych cennych danych,
które mamy w telefonie. Większość z nas wykorzystuje krótkie hasło do odblokowania ekranu. To samo
hasło z kolei jest wykorzystywane do zaszyfrowania klucza używanego w procesie
szyfrowania/deszyfrowania danych na flash'u smartfona. W ustawieniach Androida nie ma jednak opcji
rozdzielenia tych haseł i można by pomyśleć, że wykorzystanie czterocyfrowego kodu PIN jako
zabezpieczenie mija się z celem. Na pewno w części smartfonów tak ale niekoniecznie we wszystkich
modelach. Tak się składa, że akurat leży u mnie nieużywany Neffos Y5 od TP-LINK, to postanowiłem
przyjrzeć się nieco bliżej tej kwestii haseł i sprawdzić czy jest się czego obawiać stosując krótkie
hasła w zaszyfrowanych Androidach.

<!--more-->
## Krótkie hasło zabezpieczające i szyfrowanie danych

Proces szyfrowania i deszyfrowania danych jest jako taki transparentny dla użytkownika ale by był on
możliwy, potrzebny jest nam stosowny klucz szyfrujący. Taki klucz jest przechowywany zwykle na końcu
partycji `/data/` , na przestrzeni ostatnich 16 KiB. Nie jest to jednak reguła ale w przypadku tych
TP-LINK'owych telefonów, klucz znajduje się właśnie w tym miejscu. W niektórych modelach smartfonów,
do tego celu jest wykorzystywana osobna partycja ale generalnie zasada działania mechanizmu
szyfrującego jest taka sama, tj. trzeba ten klucz załadować do pamięci telefonu.

Gdyby ten klucz szyfrujący sobie leżał w formie czytelnej na partycji `/data/` , to szyfrowanie
danych byłoby pozbawione sensu. Dlatego też ten klucz trzeba również zaszyfrować i robi się to
drugim kluczem, który pochodzi od hasła użytkownika. W taki sposób klucz szyfrujący dane na
smartfonie nie jest przechowany w formie jawnej, co utrudnia, bądź też uniemożliwia, jego
odzyskanie. No chyba, że do zaszyfrowania tego klucza zostało użyte krótkie hasło.

Krótkie hasła mają ten problem, że są bardzo podatne na ataki siłowe mające na celu odgadnięcie
hasła przez próbowanie wszystkich możliwych kombinacji. W przypadku standardowego kodu PIN, to jest
raptem 10000 możliwości, a praktycznie każdy domowy komputer jest w stanie znaleźć ten właściwy kod
w czasie paru czy parunastu minut.

Może i nasz smartfon ma zabezpieczenia, które po kilku nieudanych próbach odblokowania ekranu
spowolnią kolejne próby zdjęcia blokady lub też żądają zresetowania telefonu albo nawet
przeprowadzają bez pytania proces Factory Reset ale ten mechanizm chroni nasz telefon jedynie, gdy
ten działa. Nie ochroni nas on jednak w przypadku, gdy telefon wyłączymy i zrobimy sobie kopię
binarną całej partycji `/data/` , na której mamy zaszyfrowany klucz główny. Tutaj ogromne znaczenie
ma skomplikowane i długie hasło, które potrafi wydłużyć czas jego łamania do rzędu dziesiątek czy
setek lat.

Jak już zostało wspomniane we wstępie, Android domyślnie korzysta z tego samego hasła zarówno w
przypadku blokady ekranu jak i do odszyfrowania klucza szyfrującego dane na smartfonie. W
ustawieniach Androida nie mamy praktycznie żadnej opcji zmiany hasła szyfrującego klucz główny, no
chyba, że zmienimy również hasło blokady ekranu. Problem w tym, że raczej nikt z nas nie będzie
wpisywał długiego hasła za każdym razem, gdy chce skorzystać z telefonu.

Część smartfonów z nowszą wersją Androida (5.0+) wspiera sprzętowe szyfrowanie i tak samo jest w
przypadku Neffos'a Y5. Tutaj główną rolę odgrywa TEE ([Trusted Execution
Environment](https://source.android.com/security/trusty/)). W starszych wersjach Androida, klucz
główny jest szyfrowany kluczem wygenerowanym przez [scrypt](https://en.wikipedia.org/wiki/Scrypt)
na podstawie hasła użytkownika i przechowywanej soli. W przypadku nowszych smartfonów, takie
urządzenia mają zaszyty w SoC dodatkowy unikalny klucz szyfrujący, którego nie da rady w żaden
sposób wydobyć. To właśnie ten dodatkowy klucz sprawia, że odszyfrowanie danych w trybie off-line
jest praktycznie niemożliwe. Dlatego też hasło użytkownika nie ma tutaj zbyt dużego znaczenia.
Trzeba jednak mieć na uwadze, że nie wszystkie smartfony posiadają wsparcie dla sprzętowego
szyfrowania i w ich przypadku przydałoby się używać dwóch różnych haseł. Choć ja uważam, że również
i w każdym innym przypadku dwa różne hasła są lepsze niż jedno.

## Zmiana hasła szyfrowania przez adb

Technicznie rzecz biorąc, mając w smartfonie wsparcie dla sprzętowego szyfrowania możemy sobie
darować zmianę hasła. Niemniej jednak, dla tych osób, które chcą jeszcze bardziej poprawić
bezpieczeństwo danych przechowywanych na smartfonie, istnieje możliwość rozdzielenia haseł, tak by
z jednej strony telefon miał krótkie hasło blokady ekranu, a zarazem długie hasło wykorzystywane
przy szyfrowaniu danych. Możemy to zrobić w zasadzie na dwa sposoby: przy pomocy [aplikacji Cryptfs
Password](https://play.google.com/store/apps/details?id=org.nick.cryptfs.passwdmanager)
([źródła](https://github.com/nelenkov/cryptfs-password-manager)) lub też przy pomocy `adb` . To
co łączy te dwie metody, to niestety ukorzeniony system (root), a jak wiadomo większość użytkowników
smartfonów nie posiada root'a. W przypadku tego Neffos'a Y5, [proces root można przeprowadzić bez
większego problemu]({{< baseurl >}}/post/android-root-smartfona-neffos-y5-od-tp-link/) i
naturalnie mój model ma już ten proces za sobą.

Jeśli nie zamierzamy instalować dodatkowych aplikacji w telefonie, to naturalnie możemy skorzystać z
`adb` . Podłączamy zatem nasz telefon do portu USB komputera i logujemy się na użytkownika root:

```
 # adb shell
shell@Y5:/ $ su
root@Y5:/ #
```

Luzujemy nieco politykę bezpieczeństwa SELinux tak, by widzieć komunikaty zwracane przez `vdc` .
Zmiany są jedynie tymczasowe i po zresetowaniu smartfona wszystko wróci do normy. Same komunikaty
potrzebne nam są natomiast do zweryfikowania czy polecenia, które będziemy wykonywać, wykonały się z
powodzeniem, czy też wystąpił jakiś błąd.

    root@Y5:/ # supolicy --live 'allow vdc devpts chr_file {read write getattr ioctl}'

    supolicy v2.79 (ndk:armeabi-v7a) - Copyright (C) 2014-2016 - Chainfire

    Patching policy ...
    (Android M policy compatibility mode)
    -allow:vdc:devpts:chr_file:read=ok
    -allow:vdc:devpts:chr_file:write=ok
    -allow:vdc:devpts:chr_file:getattr=ok
    -allow:vdc:devpts:chr_file:ioctl=ok
    - Success

W tym przypadku hasło do odblokowania ekranu jest ustawione na PIN 1111. By odszyfrować telefon,
podczas jego startu również trzeba ten kod wprowadzić. Po zmianie hasła, kod do blokady ekranu
zostanie taki sam, natomiast hasło przy starcie będzie już inne. Hasło zmieniamy w poniższy sposób
(w zależności od wersji Androida, polecenia do zmiany hasła się różnią):

    root@Y5:/ #  vdc cryptfs changepw password 1111 jakies-haslo
    200 0 0

Te cyferki wyżej ( `200 0 0` ) wskazują, że hasło zostało zmienione. Gdyby tutaj było `200 0 -1` ,
wtedy wystąpił jakiś błąd przy zmianie hasła i dalej mamy stare hasło.

To hasło można zweryfikować poniższym poleceniem:

    root@Y5:/ # vdc cryptfs verifypw jakies-haslo
    200 0 -1

Problem jednak w tym, że jak widać wyżej po cyferkach, mamy błąd, a to oznacza, że przy pomocy tego
hasła nie da rady odszyfrować telefonu. Do końca nie wiem w czym tkwi problem ale samo hasło działa
jak najbardziej i można przy jego pomocy odszyfrować telefon. Być może są jakieś błędy w kodzie
Androida, który jest wgrany na ten smartfon, np. [podobne do tych opisanych
tutaj](https://github.com/nelenkov/cryptfs-password-manager/issues/14). Niemniej jednak, to o czym
warto pamiętać, to jeśli zmienimy to hasło i podczas startu smartfona nie będziemy w stanie go
odszyfrować, to czeka nas przywracanie ustawień urządzenia do domyślnych, czyli utracimy
bezpowrotnie wszelkie dane z partycji `/data/` . Dlatego też lepiej na firmowym smartfonie nie
przeprowadzać procesu zmiany hasła bez absolutnej pewności co do późniejszego odszyfrowania
zawartości telefonu.

Po zmianie hasła resetujemy telefon i podczas startu podajemy już nowe hasło. W taki oto prosty
sposób mamy dwa różne hasła (jedno do blokady ekranu, a drugie do szyfrowania partycji `/data/` ).

Jeśli kogoś interesuje zagadnienie szyfrowania danych w Androidzie, to naturalnie może sobie
poczytać o szczegółach [tutaj](https://source.android.com/security/encryption/full-disk),
[tutaj](https://source.android.com/security/trusty/),
[tutaj](https://nelenkov.blogspot.com/2014/10/revisiting-android-disk-encryption.html) oraz
[tutaj](https://nelenkov.blogspot.com/2012/08/changing-androids-disk-encryption.html).

## Czy zmiana hasła blokady ekranu wpływa na hasło klucza szyfrującego

Czytając kilka artykułów, natknąłem się na wzmiankę, że zmiana hasła do blokady ekranu ma zmieniać
również hasło od klucza szyfrującego. Nie wiem jak jest w przypadku innych telefonów czy wersji
Androida ale w przypadku Neffos'a Y5, tego typu problem nie występuje. Hasło do blokady można
zmienić bez większego problemu w ustawieniach ale hasło od klucza pozostaje takie jak zostało
ustawione przez `adb` .
