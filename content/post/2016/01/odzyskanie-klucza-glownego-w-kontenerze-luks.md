---
author: Morfik
categories:
- Linux
date: "2016-01-15T16:25:39Z"
date_gmt: 2016-01-15 15:25:39 +0100
published: true
status: publish
tags:
- luks
- szyfrowanie
GHissueID: 529
title: Klucz główny kontenera LUKS i jego odzyskanie
---

Kontenery LUKS to takie wynalazki, za których pomocą jesteśmy w stanie zaszyfrować całe dyski
twarde, a właściwie to znajdujące się na nich dane. Taki kontener składa się głównie z nagłówka,
który jest umieszczany na początku partycji. By być w stanie dokonać szyfrowania i deszyfrowania
informacji w locie, system musi posiadać klucz główny (master key). Ten klucz jest przechowywany w
nagłówku i by go wydobyć, musimy wprowadzić jedno z haseł do kontenera. Później klucz wędruje do
pamięci, a hasło jest z niej usuwane. W ten sposób system ma dostęp do klucza głównego przez cały
czas począwszy od chwili otwarcia kontenera, aż do momentu jego zamknięcia. [Ten klucz jesteśmy w
stanie bez większego problemu wydobyć][1], co może być bardzo przydatne na wypadek zapomnienia
hasła, czy też uszkodzenia samego nagłówka. W tym wpisie postaramy się odzyskać klucz główny
zaszyfrowanego kontenera LUKS.

<!--more-->
## Przykładowy kontener LUKS

Do celów testowych utworzymy sobie kontener LUKS. Potrzebne nam są do tego odpowiednie narzędzia,
które w debianie znajdują się w pakiecie `cryptsetup` . Jeśli dysponujemy jakimś zaszyfrowanym
kontenerem to naturalnie ten krok możemy pominąć. Dla wszystkich tych, którzy nie mają, to poniżej
jest prosta linijka, którą można wklepać do
    terminala:

    # cryptsetup --cipher aes-xts-plain64 --key-size 512 --hash sha512 --iter-time 5000 --use-random --verify-passphrase --verbose luksFormat /dev/sda2

To polecenie spowoduje utworzenie nagłówka, dodanie do niego potrzebnych informacji, w tym klucza
głównego, oraz również zostanie utworzony keyslot z hasłem, które wyżej podaliśmy. Otwieramy tez
kontener i tworzymy na nim system plików:

    # cryptsetup luksOpen /dev/sda2 sda2

    # mkfs.ext4 /dev/mapper/sda2
    # mount /dev/mapper/sda2 /mnt

## Klucz główny

W tej chwili mamy dostęp do klucza głównego. Możemy go odszukać w pamięci za pomocą `dmsetup` ,
przykładowo:

    # dmsetup table --target crypt --showkey /dev/mapper/sda2
    0 72407040 crypt aes-xts-plain64 9c54...1313 0 8:2 4096

Przy otwartym kontenerze (niekoniecznie system plików musi być podmontowany) klucz używany do
szyfrowania i deszyfrowania danych jest widoczny powyżej. To ten długi 128 znakowy ciąg. Dla
większej czytelności został on skrócony do postaci `9c54...1313` . Jeśli zgubimy hasło do
kontenera, a ten wciąż pozostaje otwarty, to możemy ten klucz wydobyć, zapisać w pliku i
zaimportować go później w kontenerze LUKS.

By wydobyć ten klucz, musimy go przekonwertować do postaci binarnej przy pomocy tego poniższego
polecenia:

    # echo "9c54...1313" | xxd -r -p > master-key

Pamiętajmy, że jest to klucz główny, zatem nie zapisujmy tego pliku na dysku (zwłaszcza
niezaszyfrowanym), skąd go można bez problemu odzyskać. Najlepiej zrobić to w pamięci RAM.

Tak wyeksportowany klucz powinien mieć 64 bajty. Upewnijmy się, że tak jest w istocie za pomocą
`ls` . Jeśli z jakiegoś powodu ten klucz ma mniejszy rozmiar, oznacza to, że popełniliśmy gdzieś
błąd i przy próbie dodania takiego klucza do nagłówka, zostanie nam zwrócony poniższy komunikat:

    Cannot read 64 bytes from keyfile ./master-key.
    Command failed with code 22: Invalid argument

Oczywiście takiego klucza nie uda nam się dodać. Prawdopodobnie nie skopiowaliśmy całego klucza
głównego przed konwersją do postaci binarnej.

Mając binarną reprezentację klucza, możemy odmontować system plików i zamknąć kontener:

    # umount /mnt
    # cryptsetup luksClose sda2

## Uszkodzenie nagłówka LUKS

W przypadku, gdy nagłówek zaszyfrowanego kontenera został w jakiś sposób uszkodzony, będziemy
musieli utworzyć na nowo cały kontener. Nie stanowi to zagrożenia dla danych, bo przepisywany będzie
jedynie sam nagłówek LUKS (pierwsze 2 MiB na początku partycji). Musimy tylko przy tworzeniu tego
nagłówka podać ten powyższy klucz. W ten sposób to on zostanie użyty do szyfrowania i deszyfrowania
informacji, a nie jakiś losowo wygenerowany klucz w procesie tworzenia kontenera. Jeśli kontener był
tworzony w oparciu o inne niż te standardowe parametry, to musimy je także uwzględnić podczas
tworzenia nowego nagłówka. W tym przypadku polecenie wygląda następująco:

    # cryptsetup \
      --master-key-file=./master-key \
      --cipher aes-xts-plain64 \
      --key-size 512 \
      --hash sha512 \
      --iter-time 5000 \
      --use-random \
      --verify-passphrase
      --verbose \
      luksFormat /dev/sda2

Różnica w stosunku do tego poprzedniego polecenia tworzącego zaszyfrowany kontener polega na dodaniu
parametru `--master-key-file` i wskazaniu ścieżki do binarnej postaci klucza głównego. Nowy nagłówek
powinien zostać utworzony z kluczem, który wskazaliśmy. Sprawdzamy czy damy radę otworzyć ten
kontener i zamontować jego system plików bez żadnych błędów:

    # cryptsetup luksOpen /dev/sda2 sda2
    # mount /dev/mapper/sda2 /mnt

W tym przypadku kontener się otwiera po podaniu ustawionego wyżej hasła, a system plików montuje się
bez błędów. Pliki natomiast można odczytać, zatem jest wykorzystywany dokładnie ten sam klucz
główny, co przed utworzeniem nowego nagłówka.

## Utrata hasła do kontenera LUKS

W przypadku, gdy jedynie zapomnieliśmy hasła do kontenera, to nie musimy na nowo tworzyć całego
nagłówka. Możemy zwyczajnie dodać nowy keyslot z uprzednio wyeksportowanym do postaci binarnej
kluczem. Jedyne czego nam potrzeba, to ustawić nowe hasło. Robimy to w poniższy sposób:

    # cryptsetup luksAddKey --master-key-file=./master-key /dev/sda2

Po tym procesie, stare hasło możemy wyczyścić za pomocą `luksKillSlot` , przykładowo:

    # cryptsetup luksKillSlot /dev/sda2 0

Od tego momentu otwarcie kontenera będzie odbywać się w oparciu o nowe hasło. Klucz główny kontenera
zostaje bez zmian i nie powinno być problemów przy szyfrowaniu i deszyfrowaniu informacji zawartych
na dysku.


[1]: https://gitlab.com/cryptsetup/cryptsetup/wikis/FrequentlyAskedQuestions
