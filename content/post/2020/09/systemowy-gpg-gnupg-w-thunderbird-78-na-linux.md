---
author: Morfik
categories:
- Linux
date:    2020-09-07 20:15:00 +0200
lastmod: 2020-09-07 20:15:00 +0200
published: true
status: publish
tags:
- debian
- thunderbird
- szyfrowanie
- gpg
GHissueID: 7
title: Systemowy GPG/GnuPG w Thunderbird 78+ na linux
---

Jakiś już czas temu Mozilla ogłosiła, że Thunderbird od wersji 78 będzie posiadał natywne wsparcie
dla szyfrowania wiadomości z wykorzystaniem kluczy GPG/PGP, przez co [dodatek Enigmail][3] będzie
już zwyczajnie zbędny. Dziś z kolei czytam sobie o [zakończeniu wsparcia dla wersji 68][1] tego
klienta pocztowego, które będzie miało miejsce z końcem września 2020, czyli został już niespełna
miesiąc i ta starsza wersja Thunderbird'a nie będzie już dostawać łat bezpieczeństwa. Pewne jest
zatem, że dystrybucje linux'a w niedługim czasie pchną wersję 78 do głównych repozytoriów. W
Debianie, wersja 78 Thunderbird'a od dłuższego czasu jest dostępna w gałęzi eksperymentalnej i można
było ją już wcześniej sobie zainstalować, jeśli ktoś wyrażał taką chęć. Gdy ja ostatni raz
testowałem wersję 78, to nie była ona zbytnio do użytku ale wygląda na to, że większość
niedogodności, których mi się udało doświadczyć, została już wyeliminowana. Pozostał w zasadzie
jeden problem, tj. Thunderbird domyślnie używa własnego keyring'a kluczy GPG/PGP, w efekcie czego
systemowy GPG/GnuPG nie jest w ogóle wykorzystywany. Taki san rzeczy sprawia, że będziemy mieć dwie
różne bazy danych kluczy (jedna dla Thunderbird'a, a druga dla reszty linux'a), co może trochę
irytować. Na szczęście jest opcja wymuszenia na TB, by korzystał on z systemowego keyring'a kluczy
GPG/PGP i celem tego artykułu jest pokazanie właśnie jak tego typu zabieg przeprowadzić.

<!--more-->
## Opcja migracji z Enigmail

Z chwilą zainstalowania Thunderbird'a w wersji 78, po jego uruchomieniu przywita nas informacja, z
której to możemy wyczytać, że OpenPGP stało się częścią Thunderbird'a oraz, że jest to ostatnia
wersja Enigmail, która zostanie wydana:

![](/img/2020/09/001-thunderbird-gpg-pgp-gnupg-enigmail-config-linux-debian.png#big)

Jak widać wyżej, mamy przycisk migracji ale nie będziemy z niego korzystać. Ta opcja jest raczej
tylko dla tych osób, które chciałby zaimportować wszystkie swoje klucze do keyring'a Thunderbird'a.
My jednak chcemy robić użytek z systemowego `gpg` , zatem całą "migrację" trzeba będzie
przeprowadzić ręcznie.

## Thunderbird i zewnętrzny GPG/GnuPG

Jako, że Thunderbird domyślnie będzie wykorzystywał swoją wbudowaną implementację OpenPGP, to
musimy go nieco przymusić do korzystania z systemowych kluczy GPG/PGP. W tym celu odpalamy
Thunderbird'a i przechodzimy kolejno w menu do `Edit` > `Preferences` > `General` i na samym dole
wybieramy `Config Editor` . Odszukujemy klucz `mail.openpgp.allow_external_gnupg` i ustawiamy jego
wartość na `true` :

![](/img/2020/09/002-thunderbird-gpg-pgp-gnupg-enigmail-config-linux-debian.png#big)

## Przypisywanie kluczy GPG/PGP do kont email

Jako, że już nie korzystamy z Enigmail, to wszystkie ustawienia dotyczące szyfrowania poczty
zostaną skasowane/usunięte i cały proces konfiguracji trzeba będzie przeprowadzić od nowa. Na
początek musimy powiązać klucze prywatne z kontami email. Wchodzimy zatem w `Edit` >
`Account Settings` i wybieramy swoje konto pocztowe. Na liście opcji powinna znajdować się pozycja
`End-To-End Encryption` . Musimy tutaj dodać nowy klucz, zatem klikamy na `Add Key` i wybieramy
klucz zewnętrzny przez GPG/GnuPG ( `Use your external key through GnuPG` ):

![](/img/2020/09/003-thunderbird-gpg-pgp-gnupg-enigmail-config-linux-debian.png#big)

Następnie podajemy identyfikator klucza GPG/PGP. Przy czym warto tutaj wspomnieć, że narzędzie
`gpg` na linux może zwracać ID klucza, np. w postaci `0x4796CCA1` lub `0x32D9CB634796CCA1` , tj. 8
lub 16 znaków HEX. Thunderbird akceptuje jedynie ID w postaci 16 znaków. Trzeba także pozbyć się
`0x` z początku ID. Zatem ID klucza będzie miało postać `32D9CB634796CCA1` i to tę wartość trzeba
podać w TB:

![](/img/2020/09/004-thunderbird-gpg-pgp-gnupg-enigmail-config-linux-debian.png#big)

Po dodaniu klucza, Thunderbird powinien automatycznie dodać `0x` , przez co my będziemy w stanie z
tego klucza skorzystać:

![](/img/2020/09/005-thunderbird-gpg-pgp-gnupg-enigmail-config-linux-debian.png#big)

Jak widać wyżej, przy kluczu widnieje tag `External GnuPG Key`, zatem Thunderbird będzie o ten
klucz prosił nasz system ilekroć tylko będzie taka potrzeba. W przypadku niedostępności tego klucza,
nie da rady np. odczytać zaszyfrowanych wiadomości, które ktoś do nas przyśle.

Po określeniu ID klucza, możemy skonfigurować także zachowanie samego szyfrowania, tj. zdecydować
czy wiadomości wysyłane do kontaktów powinny być domyślnie szyfrowane oraz czy kryptograficznie
podpisywać nasze wiadomości.

![](/img/2020/09/006-thunderbird-gpg-pgp-gnupg-enigmail-config-linux-debian.png#big)

W wiadomościach, które będziemy wysłać, może także znaleźć się nasz klucz publiczny, co powinno
ułatwić rozmówcy nawiązanie i prowadzenie z nami szyfrowanej korespondencji. Oczywiście w dalszym
ciągu można ręcznie przeszukać serwer kluczy za odpowiednim kluczem publicznym ale nie zmienia to
faktu, że taka opcja wysyłania kluczy jest lekkim uproszczeniem całego procesu. Niemniej jednak,
wygląda na to, że póki co ten klucz jest przesyłany zawsze jeśli się tylko [zaznaczy opcję
szyfrowania wiadomości lub jej podpisu][2]. Krótko mówiąc, od jakichś dwóch miesięcy jest błąd,
który jeszcze nie został poprawiony i prawdopodobnie nie zostanie fix'nięty, gdy wersja 78
Thunderbird'a pojawi się w testowym/niestabilnym repozytorium Debiana. Tak czy inaczej, jeśli nie
chcemy ciągle tego klucza publicznego dołączać w wiadomościach, to trzeba go ręcznie odznaczać w
opcjach szyfrowania.

### Publiczna część klucza GPG/PGP

Wydawać by się mogło, że to w zasadzie cała konfiguracja Thunderbird'a, którą musimy przeprowadzić,
by to szyfrowanie poczty włączyć ale tak nie jest. Tak skonfigurowane szyfrowanie nam nie zadziała,
bo Thunderbird wymaga jeszcze, by manualnie dodać do jego keyring'a wszystkie klucze publiczne, z
których zamierzamy korzystać (szyfrować nimi pocztę). Czyli wszystkie klucze publiczne jakie mamy
aktualnie w bazie `gpg` trzeba dodać ręcznie do keyring'a Thunderbird'a. Nie mam pojęcia kto wpadł
na takie dziwne rozwiązanie ale skoro już połowę pracy mamy za sobą, to wypadałoby ją dokończyć.

Przy pomocy poniższego polecenia eksportujemy publiczną część naszego klucza GPG/PGP do pliku:

    $ gpg --export --armor 0x32D9CB634796CCA1 > klucz.gpg

    $ cat klucz.gpg
    -----BEGIN PGP PUBLIC KEY BLOCK-----

    mDMEXRaE+hYJKwYBBAHaRw8BAQdADVtvGNnC7y4y14i2IuxupgValXBb5YBbzeym
    UVfQEQu0L01pa2hhaWwgTW9yZmlrb3YgKE1vcmZpaykgPG1tb3JmaWtvdkBnbWFp
    bC5jb20+iJYEExYKAD4CGwMFCwkIBwMFFQoJCAsFFgIDAQACHgECF4AWIQR1ZhNY
    xftXAnkWpwEy2ctjR5bMoQUCXj79nwUJAwmsJQAKCRAy2ctjR5bMoSfMAP9ZBENe
    Qz9MCxZwA11bL9b+ADaYruFlEWVKv9TEd+bHjAEApCH8boYJ5QZBD+HYwDCxxKYM
    iQ7yfhkn8sRWdIThYAqIlgQTFgoAPhYhBHVmE1jF+1cCeRanATLZy2NHlsyhBQJd
    FolPAhsDBQkB4TOABQsJCAcDBRUKCQgLBRYCAwEAAh4BAheAAAoJEDLZy2NHlsyh
    FLQA/00Q763bkE0jJr7D4jiJ/pm/wWC0gylLTosl2AvdCE5dAP92yg4wvIs1prto
    S/CJ1Su/sIV3btV01xJ9qNyLBpdvA7QuTWlraGFpbCBNb3JmaWtvdiAoTW9yZmlr
    KSA8bW9yZmlrb3ZAZ21haWwuY29tPoiWBBMWCgA+AhsDBQsJCAcDBRUKCQgLBRYC
    AwEAAh4BAheAFiEEdWYTWMX7VwJ5FqcBMtnLY0eWzKEFAl4+/Z8FCQMJrCUACgkQ
    MtnLY0eWzKG1GgD+Nakl4+R1/9fdnB4UZAPsRyZhrwxzxvzkifO+4M9zNmsBAN+M
    gJ9RplSPA8Aq71lmtQ+CjtY4kkB5W2qbpgreBhQIiJYEExYKAD4WIQR1ZhNYxftX
    AnkWpwEy2ctjR5bMoQUCXRaE+gIbAwUJAeEzgAULCQgHAwUVCgkICwUWAgMBAAIe
    AQIXgAAKCRAy2ctjR5bMoRAGAQD23ggi5YzUdl+V1X6AEDLCYY6XGpcLRYLdLjQq
    8mylPwEA7dNloYMQFUlxoQhAORSOLNEQGd5A/CUyGZZ8GNlaNwC4OARdFoT6Egor
    BgEEAZdVAQUBAQdA1vPaWR/g6H2DzFqi6zjEBCqEv6bOg+N6lahCEuhLc24DAQgH
    iH4EGBYKACYCGwwWIQR1ZhNYxftXAnkWpwEy2ctjR5bMoQUCXj7+CgUJAwmskAAK
    CRAy2ctjR5bMoZIbAQChdKEJzIXMxUOPs3fMn+cth5CB6NCVXQSF+BPhzJuNuQEA
    5WTZzlkCfKjXjkcqDGnDd7OHc8Es0OR1srTstnmwUAI=
    =gxUP
    -----END PGP PUBLIC KEY BLOCK-----

Oczywiście jeśli chcemy za jednym zamachem wyeksportować wszystkie klucze publiczne, to możemy
posłużyć się poniższym poleceniem:

    $ gpg --export --armor > all-public-keys.asc

Tak uzyskany plik trzeba będzie zaimportować do keyring'a Thunderbird. Przechodzimy zatem w jego
menu do `Tools` > `OpenPGP Key Manager` :

![](/img/2020/09/007-thunderbird-gpg-pgp-gnupg-enigmail-config-linux-debian.png#big)

Wybieramy z menu `File` > `Import Public Key(s) From File` i wskazujemy ścieżkę do utworzonego
wyżej pliku z kluczem publicznym (lub kluczami). Thunderbird powinien rozpoznać nasz klucz:

![](/img/2020/09/008-thunderbird-gpg-pgp-gnupg-enigmail-config-linux-debian.png#big)

Po dodaniu klucza, zostanie wyświetlone krótkie podsumowanie na jego temat. Będzie tam również
niebieski link, przy pomocy którego to można wyświetlić szczegóły klucza i określić zaufanie do
niego. Musimy kliknąć w ten link:

![](/img/2020/09/009-thunderbird-gpg-pgp-gnupg-enigmail-config-linux-debian.png#big)

![](/img/2020/09/010-thunderbird-gpg-pgp-gnupg-enigmail-config-linux-debian.png#big)

Jak widać, klucz nie ma ustawionego póki co zaufania. Świadczy o tym opcja `Not yet, maybe later` .
Do momentu określenia tutaj zaufania nie będzie można z tego klucza skorzystać przy szyfrowaniu
wiadomości. Trzeba zatem określić maksymalny poziom zaufania:

![](/img/2020/09/011-thunderbird-gpg-pgp-gnupg-enigmail-config-linux-debian.png#big)

W tym przypadku mamy do czynienia z naszym kluczem publicznym i mu ufamy ale w przypadku innych
kluczy publicznych dobrze jest zweryfikować odcisk palca takiego klucza zewnętrznym kanałem, a
najlepiej osobiście.

## Test szyfrowania i podpisu wiadomości

Konfigurację kluczy mamy z głowy. Pora przejść do testu szyfrowania wiadomości. Wysyłamy zatem sami
do siebie wiadomość testową. Naturalnie szyfrujemy ją (kluczem publicznym). Jeśli nie wybraliśmy
domyślnego szyfrowania, to wystarczy kliknąć na przycisk `Security` i ustawić tam co trzeba.
Informacja o szyfrowaniu wiadomości będzie na pasku stanu w prawym dolnym rogu:

![](/img/2020/09/012-thunderbird-gpg-pgp-gnupg-enigmail-config-linux-debian.png#big)

Jeśli z jakiegoś powodu nie możemy wysłać wiadomości, to pewnie trzeba ponownie uruchomić
Thunderbird'a. Przy wysyłaniu wiadomości, Thunderbird powinien nas poprosić o hasło do klucza (w
przypadku, gdy mamy je ustawione). Jeśli tak się dzieje, to znaczy, że wykorzystywany jest
systemowy keyring kluczy GPG/PGP, a nie ten wbudowany w Thunderbird i o to nam chodziło.

Po chwili mail powinien do nas dotrzeć:

![](/img/2020/09/013-thunderbird-gpg-pgp-gnupg-enigmail-config-linux-debian.png#big)

Jak widać, tytuł jest zaszyfrowany i nie można go odczytać. Po zaznaczeniu wiadomości, ta zostanie
zdeszyfrowana automatycznie naszym kluczem prywatnym, który został zbuforowany w pamięci podczas
szyfrowania wiadomości przy jej wysyłaniu, zatem nie trzeba będzie podawać ewentualnego hasła po
raz drugi.

![](/img/2020/09/014-thunderbird-gpg-pgp-gnupg-enigmail-config-linux-debian.png#big)

Zielony ptaszek z prawej strony świadczy o poprawności wiadomości. Jeśli wejdziemy w informacje o
bezpieczeństwie, to możemy sprawdzić jaki klucz został użyty przy składaniu podpisu:

![](/img/2020/09/015-thunderbird-gpg-pgp-gnupg-enigmail-config-linux-debian.png#big)

Zatem test szyfrowania/deszyfrowania i podpisu przebiegł pomyślnie.

## Podsumowanie

Trochę szkoda, że ten nowy Thunderbird nie potrafi robić użytku w 100% z systemowego keyring'a
kluczy GPG/PGP. Nie wiem kto wpadł na taki śmieszny pomysł, by wszystkie klucze publiczne trzeba
było dodać do Thunderbird'a osobno przy korzystaniu jednocześnie z zewnętrznego rozwiązania
zapewniającego klucze prywatne. W ten sposób mamy dwie bazy danych kluczy, które bardzo szybko się
rozsynchronizują. Pozostaje mieć nadzieje, że coś z tym faktem zostanie zrobione, choć ja na to bym
nie liczył, bo gdyby tak miało być, to dev'y Mozilli już by się o ten bardzo niewygodny szczegół
zatroszczyły. Szkoda trochę, że Enigmail nie będzie już rozwijany, a widząc w jaki sposób została
zaimplementowana obsługa kluczy GPG/PGP w Thunderbird, to myślę, że to jest jednak parę kroków w
tył zamiast do przodu. No i trochę irytujący jest ten bug z automatycznym dołączaniem klucza
publicznego do każdej szyfrowanej czy też podpisywanej przez nas wiadomości. Jeśli miałbym wybierać,
to bym został na Thunderbird 68, przynajmniej póki co.

[1]: https://blog.thunderbird.net/2020/09/openpgp-in-thunderbird-78/
[2]: https://bugzilla.mozilla.org/show_bug.cgi?id=1654950
[3]: https://enigmail.net/index.php/en/
