---
author: Morfik
categories:
- Linux
date: "2016-01-30T16:52:05Z"
date_gmt: 2016-01-30 15:52:05 +0100
published: true
status: publish
tags:
- system-plików
- hdd
- ssd
- ext4
- luks
title: Zmiana identyfikatora UUID systemu plików EXT4 i kontenera LUKS
---

[Na forum DUG'a po raz kolejny pojawił się post][1] dotyczący unikalnych identyfikatorów, które są
nadawane partycjom dysków twardych. Nie wiem jak sprawa ma się w przypadku windowsów ale linux na
podstawie tych numerów [UUID][2] i ([GUID][3]) jest w stanie identyfikować konkretne urządzenia.
Czasem się zdarza tak, że dwa dyski czy partycje mają taki sam identyfikator, co prowadzi zwykle do
problemów. Kolizja numerów identyfikacyjnych może być wynikiem pozostałości po procesie
produkcyjnym ale może także powstać za sprawą klonowania nośnika za pomocą narzędzia `dd` . Tak czy
inaczej, przydałoby się wiedzieć jak ustalić, poprawnie wygenerować czy też zmienić UUID wszędzie
tam, gdzie jest on wykorzystywany i o tym będzie ten wpis.

<!--more-->
## Ustalanie identyfikatora UUID

Identyfikator UUID w linux'ie możemy podejrzeć za pomocą poleceń `lsblk` czy też `blkid` . Z reguły
nie powinniśmy odnotować dwóch takich samych UUID. W tym przypadku, jedna z partycji została
sklonowana przy pomocy `dd` i umieszczona na dysku USB. Mając podpięte oba dyski w tym samym czasie,
`lsblk` zwraca poniższy wynik:

    # lsblk /dev/sda6 /dev/sdb5
    NAME     SIZE FSTYPE      TYPE  LABEL MOUNTPOINT  UUID
    sda6   141.9G crypto_LUKS part                    f3c10054-0583-4e10-937b-dcdc9a05a25c
    └─kabi 141.9G ext4        crypt kabi  /media/Kabi b47e6dcd-924e-40fa-a8b1-7593419f86d7
    sdb5   141.9G crypto_LUKS part                    f3c10054-0583-4e10-937b-dcdc9a05a25c
    └─sdb5 141.9G ext4        crypt kabi              b47e6dcd-924e-40fa-a8b1-7593419f86d7

Może i system działa prawidłowo ale trzeba się zastanowić co się stanie, gdy uruchomimy system i oba
te dyski będą podpięte. Jeśli montujemy zasoby w fazie boot i opieramy się o te identyfikatory, to
który z tych powyższych zasobów zostanie wybrany? Ciężko powiedzieć ale wiemy, że taki stan rzeczy
nie powinien mieć miejsca i te zdublowane identyfikatory powinniśmy zmienić.

## Zmiana UUID systemu plików EXT4

W przypadku systemu plików EXT4, ten widoczny wyżej numerek jest odczytywany bezpośrednio z
informacji zawartych w superbloku systemu plików. My również możemy je wydobyć przy pomocy narzędzi
`tune2fs` lub `dumpe2fs` , przykładowo:

    # tune2fs -l /dev/mapper/sdb5 | grep UUID
    Filesystem UUID:          b47e6dcd-924e-40fa-a8b1-7593419f86d7

Wiemy zatem, że ta informacja jest przypisana do systemu plików i wędruje wraz z nim. No to teraz
nasuwa się pytanie, jak zmienić ten widoczny wyżej ciąg? W przypadku EXT4 identyfikator zmieniamy
również przy pomocy `tune2fs` w poniższy sposób:

    # tune2fs -U random /dev/mapper/sdb5

Parametr `-U` może przyjąć jedną z trzech wartości:

  - `clear` -- czyści przypisany identyfikator.
  - `random` -- nadaje losowo wygenerowany identyfikator.
  - `time` -- nadaje identyfikator, który zostanie wygenerowany w oparciu o czas.

Identyfikator UUID zawsze możemy też podać ręcznie, przykładowo:

    # tune2fs -U "b47e6dcd-924e-40fa-a8b1-7593419f86d7" /dev/mapper/sdb5

Trzeba jednak pamiętać, że ten identyfikator nie jest losowym ciągiem HEX. Posiada on swój format
zdefiniowany w [rfc4122][4] i nigdy nie należy go dostosowywać ręcznie.

## Zmiana UUID kontenera LUKS

Zmiana unikalnego identyfikatora systemu plików to nic trudnego. Problem jednak zaczyna się, gdy w
grę wchodzi kontener LUKS, a na nim nie operuje `tune2fs` . Zatem gdzie jest przechowywany ten
identyfikator i jak go zmienić? UUID kontenerów LUKS znajduje się w nagłówku kontenera, czyli w tych
pierwszych 2 MiB zaraz na początku partycji. W skrócie jest to sześć bajtów począwszy od pozycji
168. W osobnym wpisie jest nieco więcej informacji na temat tego [jak zbudowany jest nagłówek
LUKS][5]. Ten UUID możemy naturalnie wydobyć wykorzystując narzędzie `cryptsetup` :

    # cryptsetup luksUUID /dev/sdb5
    f3c10054-0583-4e10-937b-dcdc9a05a25c

Również przy pomocy `cryptsetup` możemy ten identyfikator zmienić. Problem w tym, że nie generuje on
losowych numerów i w parametrze `--uuid` trzeba podać gotowy UUID. Jak wspomniałem wyżej, nie jest
to losowy ciąg HEX i musimy go jakoś wygenerować. W debianie, w pakiecie `uuid-runtime` , mamy
narzędzie `uuidgen` i to nim się posłużymy. By uzyskać nowy UUID, wpisujemy w terminalu to poniższe
polecenie:

    $ uuidgen --random
    90402f62-fbd0-4faf-9297-dd1f2eba7ce3

By zmienić teraz identyfikator kontenera LUKS, wydajemy poniższą komendę:

    # cryptsetup --uuid "90402f62-fbd0-4faf-9297-dd1f2eba7ce3" luksUUID /dev/sdb5

## Ryzyko utraty danych

Proces zmiany unikalnego identyfikatora UUID pod linux'em nie niesie ze sobą żadnego ryzyka utraty
danych. Ważne jest tylko to, by nie dokonywać tego zabiegu przy zamontowanych zasobach. Pamiętajmy
też, by po każdej zmianie numerów identyfikacyjnych dostosować odpowiednio pliki `/etc/fstab` i
`/etc/crypttab` .


[1]: https://forum.dug.net.pl/viewtopic.php?id=28210
[2]: https://en.wikipedia.org/wiki/Universally_unique_identifier
[3]: https://pl.wikipedia.org/wiki/Globally_Unique_Identifier
[4]: http://www.ietf.org/rfc/rfc4122.txt
[5]: /post/naglowek-kontenera-luks-trzymany-na-pendrive/
