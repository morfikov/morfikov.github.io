---
author: Morfik
categories:
- Linux
date: "2015-06-14T18:13:33Z"
date_gmt: 2015-06-14 16:13:33 +0200
published: true
status: publish
tags:
- udev
title: UDEV, czyli jak pisać reguły dla urządzeń
---

Zapuszczając się w coraz to i głębsze warstwy systemu podczas dążenia do zbadania jak on tak
naprawdę działa, zacząłem się stykać z regułami [udeva](https://wiki.archlinux.org/index.php/Udev)
(tymi umieszczanymi w katalogu /etc/udev/rules.d/ ). Jako, że nazbierało mi się już ich kilka,
zaistniała potrzeba przebadania tego co ten katalog tak naprawdę zawiera. Na dobrą sprawę nigdy się
nad tym nie zastanawiałem, jedynie kopiowałem rozwiązania z internetu i wklejałem je do
odpowiedniego pliku i jeśli ono działało, to odznaczałem problem jako rozwiązany. Zwykle takich
reguł się nie potrzebuje, temu praktycznie niewielu ludzi w ogóle się orientuje jak ogarnąć tego
całego udeva. Są przypadki kiedy przepisanie nazw urządzeń czy wykonanie określonych akcji po
podłączeniu jakiegoś sprzętu do komputera jest wielce niezbędne i nie ma innej opcji jak tylko
zrozumieć co udev tak naprawdę robi.

<!--more-->
## Udev i jego pliki reguł

Na samym początku przyjrzyjmy się nieco katalogowi /etc/udev/rules.d/ . Zawiera on pliki zaczynające
się numerycznie, od `00` do `99` . Na końcu każdego pliku musi być także końcówka `.rules` .
Przykładowy plik wygląda zatem tak: `70-persistent-net.rules` . Pliki zlokalizowane w tym katalogu
są wczytywane jeden po drugim, te z niższymi numerkami jako pierwsze. Zawartość tych plików
wyglądają mniej więcej tak jak pokazano
    poniżej:

    SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="00:e0:4c:75:03:09", ATTR{dev_id}=="0x0", ATTR{type}=="1", KERNEL=="eth*", NAME="eth0"

Reguły również są parsowane jedna po drugiej (od góry do dołu) i jeśli udev znajdzie jakąś pasującą
regułę dla urządzenia, to nie zatrzymuje się na niej, tylko aplikuje ją i przechodzi do następnej.
Dzięki temu można precyzować wiele reguł i zostaną one wszystkie zaaplikowane dla dopasowanego
urządzenia.

Reguła składa się z grubsza z dwóch części. Pierwsza z nich ma za zadanie dopasowanie urządzenia,
druga zaś ma to urządzenie odpowiednio dostosować. Jak odróżnić zatem te części od siebie?
Parametry, na podstawie których urządzenie jest dopasowywane można poznać po znaku `==` . Z kolei,
pojedynczy znak `=` przypisuje konkretne wartości dla dopasowanego urządzenia, odpowiednio je
zmieniając.

## Pozyskiwanie informacji o urządzeniu

Atrybuty, na których to podstawie urządzenie ma być dopasowane, możemy pozyskać przy pomocy
narzędzia `udevadm`. Posiada ono kilka parametrów i jeszcze więcej opcji ale jeśli chodzi o
szukanie informacji na temat urządzenia, to do tego celu służy parametr `info` :

    # udevadm info --name /dev/sdb --attribute-walk

Informacje wypisane za pośrednictwem powyższego polecenia można bezpośrednio kopiować do pliku reguł
przy oddzieleniu ich od siebie przecinkami. Dodatkowo dla większej przejrzystości i czytelności
reguły, linie mogą być łamane przy pomocy znaku `\` .

Poza suchymi parametrami, które można wykorzystać jedynie w plikach reguł udeva, istnieją jeszcze
zmienne środowiskowe, które są uzupełniane i eksportowane tuż po podłączeniu urządzenia. W przypadku
gdy przy wykryciu danego zasobu zajdzie potrzeba uruchomienia jakiegoś skryptu, te zmienne mogą
okazać się wielce użyteczne, bo będą widoczne dla tego skryptu. Jeśli interesują nas te zmienne,
możemy je podejrzeć przy pomocy poniższego polecenia:

    # udevadm info --name /dev/sdb

### Składnia reguł

Jeszcze parę słów o znakach wykorzystywanych w plikach reguł. Wiemy już po co nam `==` oraz `=` ale
spotkamy się z szeregiem innych "krzaków" i wypadało by również wiedzieć co one oznaczają. I tak np.
przeciwieństwem `==` jest `!=` . Jeśli reguła ma przyjąć kilka wartości, kolejne z nich dodajemy
przez `+=`. Z kolei jeśli po danej wartości już nie spodziewamy się by jakieś dalsze zmiany dla
urządzenia zachodziły, przypisujemy tę wartość przy pomocy `:=` .

Dodatkowo można również używać maski i tak `*` oznacza dowolny znak powtórzony 0 lub więcej razy. Z
kolei zaś `?` oznacza dowolny znak powtórzony tylko jeden raz. I mamy jeszcze możliwość
sprecyzowania znaków przy pomocy `[ ]` . Jeśli chcemy użyć zakresu od "a" do "z", precyzujemy go w
taki sposób: `[a-z]` . Można również wyłączyć określone znaki za pomocą `!`, przykładowo bez "a-c":
`[!a-c]` .

Możemy również skorzystać z krótkich nazw oferowanych przez udev przy podłączaniu urządzenia:

|Skrót         | Nazwa           | Opis                             |
|:------------:|:---------------:|:--------------------------------:|
|%r           |$root             |katalog urządzeń, domyślnie /dev  |
|%p           |$devpath          |wartość DEVPATH                   |
|%k           |$kernel           |wartość KERNEL                    |
|%n           |$number           |numer urządzenia, dla sda3 to 3   |
|%N           |$tempnode         |tymczasowa nazwa pliku urządzenia |
|%M           |$major            |większy numer urządzenia          |
|%m            |$minor           |mniejszy numer urządzenia         |
|%s{attribute} |$attr{attribute} |wartość atrybutu sysfs            |
|%E{variable}  |$attr{variable}  |wartość zmiennej środowiskowej    |
|%c            |$result          |wartość PROGRAM                   |
|%%            |znak %           |                                  |
|$$            |znak $           |                                  |

## Omówienie reguł na przykładzie

Załóżmy, że chcemy aby partycje naszego pendrive były montowane po jego podłączeniu do komputera, z
tym, że jedna z nich jest zaszyfrowania i nie da się jej od tak automatycznie zamontować, bo pierw
trzeba ją odszyfrować. Inny problem to nazwy urządzenia (te widoczne w katalogu /dev/ ), które mogą
ulec zmianie. Jasne, że istnieją inne drogi by osiągnąć to co chcemy, np. można użyć UUID ale my
tutaj skupimy się na rozwiązaniu tej kwestii przez udev, co ma też i swoje zalety, bo UUID może ulec
zmianie, a numer seryjny urządzenia raczej jest nieco trwalszy.

Tworzymy zatem plik `/etc/udev/rules.d/99-local.rules` i dopisujemy tam regułki:

    KERNEL=="sd?3", ACTION=="add", ENV{ID_SERIAL_SHORT}=="001CC0EC34A2BB318709004B", \
          SYMLINK+="pen%n", \
          RUN+="/usr/bin/udevil mount /dev/pen%n /mnt/dane"

    KERNEL=="sd?1", ACTION=="add", ENV{ID_SERIAL_SHORT}=="001CC0EC34A2BB318709004B", \
          SYMLINK+="pen%n", \
          RUN+="/usr/bin/udevil mount /dev/pen%n /mnt/debian"

    KERNEL=="sd?2", ACTION=="add", ENV{ID_SERIAL_SHORT}=="001CC0EC34A2BB318709004B", \
          SYMLINK+="pen%n", \
          RUN+="/sbin/cryptsetup luksOpen /dev/pen%n pen%n --key-file=/home/morfik/Desktop/pen2.key", \
          RUN+="/usr/bin/udevil mount /dev/mapper/pen%n /mnt/szyfr"
    KERNEL=="sd?2", ACTION=="remove", ENV{ID_SERIAL_SHORT}=="001CC0EC34A2BB318709004B", \
          RUN+="/sbin/cryptsetup luksClose pen%n"

Co zatem mówią nam powyższe reguły?

  - 1\. Jeśli urządzenie zostanie zarejestrowane z nazwą `sd?3` przez kernel (znak `?` oznacza
    dowolny znak, czyli sda3, sdb3) oraz zmienna środowiskowa numeru seryjnego urządzenia to
    `001CC0EC34A2BB318709004B` , udev ma utworzyć link do tego urządzenia w postaci `pen3` ( `%n`
    odpowiada 3 na pozycji `KERNEL` ) i zamontować urządzenie przez ten link w katalogu `/mnt/dane`
    . Pozycja z `ACTION` informuje udev, by tę regułę zaaplikował tylko przy podłączaniu urządzenia.
  - 2\. Praktycznie to samo co wyżej.
  - 3\. Tutaj mamy dodatkową linijkę z `RUN` , która korzysta z cryptsetup do odszyfrowania
    kontenera oraz udevila, który zamontuje zasób.
  - 4\. Ta reguła ma na celu zamknąć kontener, na wypadek gdyby ten nie został zamknięty ręcznie,
    np. w wyniku zbyt wczesnego wyciągnięcia pendrive, bo pierw trzeba odmontować partycję, a potem
    zamknąć kontener i dopiero wtedy można wyciągnąć urządzenie z portu USB. W przypadku gdy jednak
    odmontujemy zasób i wyciągniemy pendrive, system dalej będzie widział otwarty kontener. Niby nic
    się nie powinno dziać w takim przypadku, ale gdy ponownie wsadzimy pendrive do portu USB,
    zostanie wykryty jako sdc, a nie sdb.

Jeśli posiadamy na pendrive 3 partycje (lub przynajmniej więcej niż 1) i nie są one zaszyfrowane,
możemy uprościć reguły samego montowania do postaci:

    KERNEL=="sd??", ACTION=="add", ENV{ID_SERIAL_SHORT}=="001CC0EC34A2BB318709004B", \
          SYMLINK+="pen%n", \
          RUN+="/usr/bin/udevil mount /dev/pen%n /mnt/%E{ID_FS_LABEL_ENC}"

Powyższa regułka mówi, że przy dodawaniu urządzenia o numerze seryjnym 001CC0EC34A2BB318709004B,
zostanie utworzony link do każdej z jego partycji, w tym przypadku pen1, pen2, pen3, oraz każda
partycja zostanie zamontowana w katalogu /mnt/ w oparciu o jej etykietę `%E{ID_FS_LABEL_ENC}` .

## Testowanie reguł

W przypadku gdy jakaś reguła nam nie działa i nie możemy dociec o co tak naprawdę chodzi, to możemy
przetestować taką regułę i zobaczyć co zostanie wypisane na konsoli. Partycja druga na pendrive jest
opisana za pomocą:

    # udevadm info --name /dev/sdb2
    P: /devices/pci0000:00/0000:00:1d.7/usb4/4-5/4-5:1.0/host26/target26:0:0/26:0:0:0/block/sdb/sdb2
    ...

Reguły dla tego urządzenia testujemy przez wydanie poniższego
    polecenia:

    # udevadm test /devices/pci0000:00/0000:00:1d.7/usb4/4-5/4-5:1.0/host26/target26:0:0/26:0:0:0/block/sdb/sdb2

Należy prześledzić log pod kątem dopasowania reguły do urządzenia i sprawdzić czy polecenia, które
chcemy wykonać nie zawierają czasem jakiejś literówki.

W przypadku gdy zmieniamy reguły, trzeba je ponownie załadować by udev mógł być świadom zmian. Można
to zrobić przez:

    # udevadm control --reload

Nazw urządzeń w kernelu nie można zmienić. W manualu jest tylko informacja, że "The name of a device
node cannot be changed by udev, only additional symlinks can be created". Więc jeśli chcemy by
urządzenie było dostępne pod inną nazwą trzeba tworzyć do niego linki, tak jak w przykładach
powyżej.

Udev posiada także monitor zdarzeń i potrafi w czasie rzeczywistym wypisywać informacje na temat
podłączanych/odłączanych urządzeń na konsoli. Jeśli interesują nas tylko zdarzenia kernela (te
zaczynające się od KERNEL), możemy sprecyzować `--kernel` przy wywoływaniu monitora. Można także
mieć wgląd do zdarzeń udeva przez podanie parametru `--udev` . Jeśli chcemy mieć oba powyższe, to
wpisujemy w konsoli:

    # udevadm monitor
