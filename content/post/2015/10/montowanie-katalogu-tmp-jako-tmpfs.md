---
author: Morfik
categories:
- Linux
date: "2015-10-21T22:19:14Z"
date_gmt: 2015-10-21 20:19:14 +0200
published: true
status: publish
tags:
- bezpieczeństwo
- system-plików
GHissueID: 168
title: Montowanie katalogu /tmp/ jako tmpfs
---

Linux jest w stanie operować na wielu systemach plików, np. ext4, ntfs, fat. Większość z nich odnosi
się do dysków twardych, czy też innych urządzeń przechowujących spore ilości danych. Problem z tego
typu systemami plików jest taki, że operacje na plikach w ich obrębie, jak i same pliki, zostawiają
ślady. Dlatego też jeśli musimy tymczasowo skopiować plik zawierający tajne dane, lub też taki plik
poddać obróbce, nie powinniśmy go umieszczać bezpośrednio na dysku. No chyba, że wykorzystujemy
pełne szyfrowanie. Inną opcją (i o wiele prostszą w implementacji) jest przeznaczenie części
pamięci operacyjnej RAM pod [system plików tmpfs](https://wiki.archlinux.org/index.php/Tmpfs) i o
tym będzie ten wpis.

<!--more-->
## Zalety stosowania tmpfs

W przypadku gdy posiadamy dość spore zasoby pamięci RAM, prawdopodobnie bardzo rzadko, o ile w
ogóle, jesteśmy w stanie je wykorzystać w pełni podczas codziennej pracy na komputerze. Wszyscy
jednak wiemy, że pamięć operacyjna jest paręset razy szybsza od dysku. Zatem dlaczego nie przenieść
części operacji do pamięci? Nie tylko poprawi to wydajność przeprowadzanych tam operacji ale także
odciąży nieco dysk, na który tak zwykliśmy narzekać.

Nas jednak tutaj będzie raczej interesować kwestia bezpieczeństwa i poufności danych. W systemie
możemy posiadać całą masę newralgicznych plików, które zwykle są przechowywane w formie szyfrowanej
na dysku. Przykładem mogą być różnego rodzaju keyring'i z hasłami. Czasem jednak potrzebujemy
wydobyć określone dane i dokonać na nich pewnych operacji. Wydobycie haseł, plików kluczy, czy też
kluczy szyfrujących zwykle równoznaczne jest z ich zapisaniem gdzieś na dysku, a ten nie zawsze jest
szyfrowany. Wobec czego, te wszystkie poufne informacje zostają ujawnione i ktoś może do nich
uzyskać dostęp.

Oczywiście, można taki plik usunąć z dysku po tym jak skończymy się nim bawić ale ta czynność nie
usuwa pliku jako takiego, a jedynie jego link (i-węzeł), za pomocą którego jesteśmy w stanie
odnaleźć plik w systemie plików. Odzyskanie takiego pliku nie powinno sprawić większego problemu
rozmaitym aplikacjom czy firmom, które się w tym specjalizują. No chyba, że taki plik zostanie
nadpisany, np. przy pomocy narzędzia `shred` .

## Katalog /tmp/ na bazie tmpfs

W katalogu `/tmp/` są przechowywane pliki tymczasowe, a te z kolei są produktem różnego rodzaju
aplikacji. W tym katalogu mogą zatem znaleźć się rozmaite dane i niewykluczone, że zawierające
informacje, którymi nie chcielibyśmy się z nikim dzielić. Pliki tymczasowe, jak sama nazwa wskazuje,
są tymczasowe, dlatego też, w przypadku linux'ów, ten katalog jest czyszczony przy każdym starcie
systemu. Nic zatem nie stoi na przeszkodzie, by zamontować go w pamięci operacyjnej, gdzie dane też
będą automatycznie kasowane przy restarcie/wyłączeniu maszyny ze względu na specyfikację pamięci.

Obecnie systemd zajmuje się montowaniem katalogu `/tmp/` w pamięci operacyjnej i nie musimy
przeprowadzać poniższych czynności. W przypadku systemd, rozmiar takiego katalogu jest zawsze
wyznaczany w oparciu o ilość dostępnej w systemie pamięci i jest to 50%, czyli przy 8 GiB pamięci,
katalog `/tmp/` będzie miał rozmiar 4 GiB.

Jeśli mamy wydzieloną osobną partycję pod katalog `/tmp/` , to będziemy musieli z niej zrezygnować i
zakomentować odpowiedni wpis w pliku `/etc/fstab` . Montowanie katalogu `/tmp/` jako `tmpfs` w
pamięci RAM również przeprowadzamy za pomocą powyższego pliku, a konkretnie, to musimy dopisać do
niego tę poniższą linijkę:

    tmpfs   /tmp         tmpfs   nodev,nosuid,size=2G          0  0

Parametr `size` określa rozmiar pamięci, którą taki katalog będzie mógł wykorzystać. Nie jest to
sztywne przypisanie, tj. w przypadku gdy mamy 8 GiB pamięci i system chciałby wykorzystać 7 GiB, to
bez problemu będzie w stanie to zrobić. Po zapisaniu pliku, wymagany jest restart maszyny.
