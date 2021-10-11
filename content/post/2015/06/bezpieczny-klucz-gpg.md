---
author: Morfik
categories:
- Linux
date: "2015-06-14T11:55:26Z"
date_gmt: 2015-06-14 09:55:26 +0200
published: true
status: publish
tags:
- bezpieczeństwo
- gpg
- szyfrowanie
GHissueID: 118
title: Bezpieczny klucz GPG
---

W poprzednim wpisie przygotowywaliśmy sobie [plik
gpg.conf](/post/konfiguracja-gpg-w-pliku-gpg-conf/). Opcje w nim ustawione są
niezbędne do wygenerowania dobrego pod względem bezpieczeństwa klucza GPG. Taki klucz GPG nie
powinien być krótszy niż 4096 bitów. Dodatkowo, nie powinno się ustawiać daty ważności dłuższej niż
2 lata, a to z tego powodu, że zawszę tę datę można zmienić i to nawet w przypadku gdy klucz straci
ważność. Chodzi generalnie o ustawienie jakiegoś mechanizmu zabezpieczającego na wypadek gdyby nasz
klucz GPG wpadł w niepowołane ręce i stracilibyśmy nad nim panowanie. Wtedy po jakimś czasie
automatycznie się on unieważni i nie będziemy musieli się martwić czy ktoś może przez przypadek go
używać. Jest również szereg innych rzecz, o które powinniśmy się zatroszczyć i tej tematyce będzie
poświęcony ten wpis, który w dużej mierze powstał w oparciu o te
[dwa](https://www.gnupg.org/gph/en/manual/book1.html)
[linki](https://riseup.net/security/message-security/openpgp/best-practices).

<!--more-->
## Generowanie klucza

Zacznijmy od stworzenia pary kluczy GPG. Cały proces jest automatyczny i sprowadza się do wydania
poniższego polecenia:

    $ gpg --gen-key

Będzie to klucz `RSA` o długości `4096` bitów i mający datę ważności ustawioną na `2` lata. Podajemy
także imię i nazwisko, email oraz krótki komentarz.

## Wiele tożsamości w jednym kluczu

W przypadku kiedy mamy wiele kont email, to zamiast tworzyć osobne klucze dla każdego z nich, możemy
stworzyć wiele tożsamości w tym samym kluczu GPG:

    $ gpg --edit-key 0x0339957BF6449112
    ...
    gpg> adduid
    Real name: Mikhail Morfikov
    Email address: morfik@nsa.com
    Comment: another testing key
    You selected this USER-ID:
        "Mikhail Morfikov (another testing key) "

    Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O

    You need a passphrase to unlock the secret key for
    user: "Mikhail Morfikov (testing key) "
    4096-bit RSA key, ID 0x0339957BF6449112, created 2014-09-21


    pub  4096R/0x0339957BF6449112  created: 2014-09-21  expires: 2016-09-20  usage: SC
                                   trust: ultimate      validity: ultimate
    sub  4096R/0x9845A1DBE59AC095  created: 2014-09-21  expires: 2016-09-20  usage: E
    [ultimate] (1)  Mikhail Morfikov (testing key)
    [ unknown] (2). Mikhail Morfikov (another testing key)

    gpg> save

## Podział podkluczy

Dobrze jest mieć także osobny klucz (klucz główny) do podpisów cyfrowych oraz inny podklucz używany
do szyfrowania -- tak jak to widać wyżej. Mianowicie chodzi o flagi usage: `SC` i `E`. Można też
mieć dwa różne podklucze -- jeden do podpisów, drugi do szyfrowania i nie używać klucza głównego w
ogóle. Jako, że mamy osobny podklucz do szyfrowania, dodajmy podklucz do podpisów:

    $ gpg --edit-key 0x0339957BF6449112
    ...
    gpg> addkey
    Key is protected.

    You need a passphrase to unlock the secret key for
    user: "Mikhail Morfikov (another testing key) "
    4096-bit RSA key, ID 0x0339957BF6449112, created 2014-09-21

    Please select what kind of key you want:
       (3) DSA (sign only)
       (4) RSA (sign only)
       (5) Elgamal (encrypt only)
       (6) RSA (encrypt only)
    Your selection? 4
    RSA keys may be between 1024 and 4096 bits long.
    What keysize do you want? (2048) 4096
    Requested keysize is 4096 bits
    Please specify how long the key should be valid.
             0 = key does not expire
            = key expires in n days
          w = key expires in n weeks
          m = key expires in n months
          y = key expires in n years
    Key is valid for? (0) 2y
    Key expires at Tue 20 Sep 2016 07:31:43 PM CEST
    Is this correct? (y/N) y
    Really create? (y/N) y
    We need to generate a lot of random bytes. It is a good idea to perform
    some other action (type on the keyboard, move the mouse, utilize the
    disks) during the prime generation; this gives the random number
    generator a better chance to gain enough entropy.

    Not enough random bytes available.  Please do some other work to give
    the OS a chance to collect more entropy! (Need 212 more bytes)
    .............+++++

    pub  4096R/0x0339957BF6449112  created: 2014-09-21  expires: 2016-09-20  usage: SC
                                   trust: ultimate      validity: ultimate
    sub  4096R/0x9845A1DBE59AC095  created: 2014-09-21  expires: 2016-09-20  usage: E
    sub  4096R/0xDBA0A9602AF715C1  created: 2014-09-21  expires: 2016-09-20  usage: S
    [ultimate] (1)  Mikhail Morfikov (another testing key)
    [ultimate] (2)*  Mikhail Morfikov (testing key)

    gpg> save

Jak widzimy wyżej, został utworzony drugi podklucz i przy jego pozycji jest widoczny usage: `S` .

## Główny UID

Mając utworzonych kilka UID w kluczu, trzeba sprecyzować jeden z nich jako główny:

    $ gpg --edit-key 0x0339957BF6449112
    ...
    gpg> uid 2

    pub  4096R/0x0339957BF6449112  created: 2014-09-21  expires: 2016-09-20  usage: SC
                                   trust: ultimate      validity: ultimate
    sub  4096R/0x9845A1DBE59AC095  created: 2014-09-21  expires: 2016-09-20  usage: E
    sub  4096R/0xDBA0A9602AF715C1  created: 2014-09-21  expires: 2016-09-20  usage: S
    [ultimate] (1). Mikhail Morfikov (another testing key)
    [ultimate] (2)* Mikhail Morfikov (testing key)

    gpg> primary

    You need a passphrase to unlock the secret key for
    user: "Mikhail Morfikov (another testing key) "
    4096-bit RSA key, ID 0x0339957BF6449112, created 2014-09-21


    pub  4096R/0x0339957BF6449112  created: 2014-09-21  expires: 2016-09-20  usage: SC
                                   trust: ultimate      validity: ultimate
    sub  4096R/0x9845A1DBE59AC095  created: 2014-09-21  expires: 2016-09-20  usage: E
    sub  4096R/0xDBA0A9602AF715C1  created: 2014-09-21  expires: 2016-09-20  usage: S
    [ultimate] (1)  Mikhail Morfikov (another testing key)
    [ultimate] (2)* Mikhail Morfikov (testing key)

    gpg> save

## Zmiana daty ważności klucza

Za każdym razem gdy zmienimy datę ważności, klucz trzeba wysłać do serwera kluczy, by poinformować
innych o zmianach jakie przeprowadziliśmy. Datę ważności można ustawić osobno dla klucza głównego
oraz dla każdego z podkluczy. My mamy póki co jeden klucz główny oraz dwa podklucze, zatem
potrzebujemy zmienić trzy daty:

    $ gpg --edit-key 0x0339957BF6449112
    ...
    gpg> expire
    Changing expiration time for the primary key.
    Please specify how long the key should be valid.
             0 = key does not expire
            = key expires in n days
          w = key expires in n weeks
          m = key expires in n months
          y = key expires in n years
    Key is valid for? (0) 1y
    Key expires at Mon 21 Sep 2015 08:43:10 PM CEST
    Is this correct? (y/N) y

    You need a passphrase to unlock the secret key for
    user: "Mikhail Morfikov (testing key) "
    4096-bit RSA key, ID 0x0339957BF6449112, created 2014-09-21


    pub  4096R/0x0339957BF6449112  created: 2014-09-21  expires: 2015-09-21  usage: SC
                                   trust: ultimate      validity: ultimate
    sub  4096R/0x9845A1DBE59AC095  created: 2014-09-21  expires: 2016-09-20  usage: E
    sub  4096R/0xDBA0A9602AF715C1  created: 2014-09-21  expires: 2016-09-20  usage: S
    [ultimate] (1). Mikhail Morfikov (testing key)
    [ultimate] (2)  Mikhail Morfikov (another testing key)

    gpg> key 1

    pub  4096R/0x0339957BF6449112  created: 2014-09-21  expires: 2015-09-21  usage: SC
                                   trust: ultimate      validity: ultimate
    sub* 4096R/0x9845A1DBE59AC095  created: 2014-09-21  expires: 2016-09-20  usage: E
    sub  4096R/0xDBA0A9602AF715C1  created: 2014-09-21  expires: 2016-09-20  usage: S
    [ultimate] (1). Mikhail Morfikov (testing key)
    [ultimate] (2)  Mikhail Morfikov (another testing key)

    gpg> expire
    Changing expiration time for a subkey.
    Please specify how long the key should be valid.
             0 = key does not expire
            = key expires in n days
          w = key expires in n weeks
          m = key expires in n months
          y = key expires in n years
    Key is valid for? (0) 1y
    Key expires at Mon 21 Sep 2015 08:43:24 PM CEST
    Is this correct? (y/N) y

    You need a passphrase to unlock the secret key for
    user: "Mikhail Morfikov (testing key) "
    4096-bit RSA key, ID 0x0339957BF6449112, created 2014-09-21


    pub  4096R/0x0339957BF6449112  created: 2014-09-21  expires: 2015-09-21  usage: SC
                                   trust: ultimate      validity: ultimate
    sub* 4096R/0x9845A1DBE59AC095  created: 2014-09-21  expires: 2015-09-21  usage: E
    sub  4096R/0xDBA0A9602AF715C1  created: 2014-09-21  expires: 2016-09-20  usage: S
    [ultimate] (1). Mikhail Morfikov (testing key)
    [ultimate] (2)  Mikhail Morfikov (another testing key)

    gpg> key 2

    pub  4096R/0x0339957BF6449112  created: 2014-09-21  expires: 2015-09-21  usage: SC
                                   trust: ultimate      validity: ultimate
    sub  4096R/0x9845A1DBE59AC095  created: 2014-09-21  expires: 2015-09-21  usage: E
    sub* 4096R/0xDBA0A9602AF715C1  created: 2014-09-21  expires: 2016-09-20  usage: S
    [ultimate] (1). Mikhail Morfikov (testing key)
    [ultimate] (2)  Mikhail Morfikov (another testing key)

    gpg> expire
    Changing expiration time for a subkey.
    Please specify how long the key should be valid.
             0 = key does not expire
            = key expires in n days
          w = key expires in n weeks
          m = key expires in n months
          y = key expires in n years
    Key is valid for? (0) 1y
    Key expires at Mon 21 Sep 2015 08:43:46 PM CEST
    Is this correct? (y/N) y

    You need a passphrase to unlock the secret key for
    user: "Mikhail Morfikov (testing key) "
    4096-bit RSA key, ID 0x0339957BF6449112, created 2014-09-21


    pub  4096R/0x0339957BF6449112  created: 2014-09-21  expires: 2015-09-21  usage: SC
                                   trust: ultimate      validity: ultimate
    sub  4096R/0x9845A1DBE59AC095  created: 2014-09-21  expires: 2015-09-21  usage: E
    sub* 4096R/0xDBA0A9602AF715C1  created: 2014-09-21  expires: 2015-09-21  usage: S
    [ultimate] (1). Mikhail Morfikov (testing key)
    [ultimate] (2)  Mikhail Morfikov (another testing key)

    gpg> save

W powyższym przykładzie zmieniliśmy datę ważności z 2 lat na 1 rok.

## Certyfikat unieważniający

Dobrze jest także wygenerować sobie certyfikat unieważniający klucz główny -- na wypadek gdybyśmy
zapomnieli hasła do klucza, zgubili go, czy zostałby on skompromitowany. Z tym, że tego certyfikatu
trzeba strzec, bo jeśli dostanie się w niepowołane ręce, ten ktoś będzie mógł bez problemu
unieważnić nasz klucz główny. Generujemy
    certyfikat:

    $ gpg --output 0x0339957BF6449112.gpg-revocation-certificate --armor --gen-revoke 0x0339957BF6449112

W katalogu, w którym wydaliśmy powyższe polecenie został utworzony plik
`0x0339957BF6449112.gpg-revocation-certificate` i to za jego pomocą można unieważnić klucz. Jeśli
kiedyś by zaszła potrzeba skorzystania z tego certyfikatu, importujemy ten plik jak zwykły klucz
gpg:

    $ gpg --import 0x0339957BF6449112.gpg-revocation-certificate
    gpg: key 0x0339957BF6449112: "Mikhail Morfikov (testing key) " revocation certificate imported
    gpg: Total number processed: 1
    gpg:    new key revocations: 1
    gpg: 3 marginal(s) needed, 1 complete(s) needed, PGP trust model
    gpg: depth: 0  valid:   1  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 1u
    gpg: next trustdb check due at 2016-09-20

I sprawdzamy, czy klucz został cofnięty:

    $ gpg --list-keys
    /home/morfik/.gnupg/pubring.gpg
    --------------------------------
    pub   4096R/0xD71FD9994AF34F0B 2012-08-10
          Key fingerprint = 878F FB44 5E6E 13A6 4716  3BDC D71F D999 4AF3 4F0B
    uid                            https://www.sks-keyservers.net
    uid                            https://sks-keyservers.net

    pub   4096R/0x0339957BF6449112 2014-09-21 [revoked: 2014-09-21]
          Key fingerprint = 6DF3 C313 A0FE 1E7F 168D  F9DC 0339 957B F644 9112
    uid                            Mikhail Morfikov (testing key)
    uid                            Mikhail Morfikov (another testing key)

Przy naszym kluczu widnieje `revoked` , czyli z tego klucza się już nie da skorzystać. Jedyne co nam
jeszcze pozostaje do zrobienia, to wysłanie zaktualizowanych informacji do serwera kluczy via `gpg
--send-keys` .

## Przesyłanie nowego klucza do serwera kluczy

Po tym jak się przestaniemy bawić kluczami, musimy je przesłać do serwera kluczy. Robimy to w
poniższy sposób:

    $ gpg --send-key 0x0339957BF6449112
    gpg: sending key 0x0339957BF6449112 to hkps server hkps.pool.sks-keyservers.net

Dla sprawdzenia, czy faktycznie klucz został wyeksportowany na publiczny serwer kluczy, przeszukajmy
serwery pod kątem naszego klucza GPG:

    $ gpg --search-keys 0x0339957BF6449112
    gpg: searching for "0x0339957BF6449112" from hkps server hkps.pool.sks-keyservers.net
    (1)     Mikhail Morfikov (testing key)
            Mikhail Morfikov (another testing key)
              4096 bit RSA key 0x0339957BF6449112, created: 2014-09-21, expires: 2015-09-21
    Keys 1-1 of 1 for "0x0339957BF6449112".  Enter number(s), N)ext, or Q)uit >

Czasem klucz może nie zostać odnaleziony za pierwszym razem. To nie jest błąd tylko tak działają te
serwery w puli. Po prostu nie zostaliśmy podłączeni do tego, który jest w posiadaniu naszego klucza.
Po paru chwilach, wszystkie serwery sobie wymienią ten klucz automatycznie i kopia naszego klucza
GPG będzie dostępna bez problemu na każdym z nich, także bez obaw.

## Usuwanie klucza głównego

Najpierw wykonajmy backup katalogu `~/.gnupg/` . Następnie eksportujemy podklucze do pliku, po czym
usuwamy klucz główny z keyringa systemowego i dodajemy podklucze bez klucza prywatnego:

    $ cp -a ~/.gnupg/ ~/.gnupg.back/

    $ gpg -K
    /home/morfik/.gnupg/secring.gpg
    --------------------------------
    sec   4096R/0x0339957BF6449112 2014-09-21 [expires: 2016-09-20]
          Key fingerprint = 6DF3 C313 A0FE 1E7F 168D  F9DC 0339 957B F644 9112
    uid                            Mikhail Morfikov (testing key)
    uid                            Mikhail Morfikov (another testing key)
    ssb   4096R/0x9845A1DBE59AC095 2014-09-21
    ssb   4096R/0xDBA0A9602AF715C1 2014-09-21

    $ gpg --export-secret-subkeys 0x9845A1DBE59AC095! 0xDBA0A9602AF715C1! > subkeys

    $ gpg --export 0x0339957BF6449112 > pubkeys

    $ gpg --delete-secret-key 0x0339957BF6449112

    $ gpg --import pubkeys subkeys
    gpg: key 0x0339957BF6449112: "Mikhail Morfikov (testing key) " not changed
    gpg: key 0x0339957BF6449112: secret key imported
    gpg: key 0x0339957BF6449112: "Mikhail Morfikov (testing key) " 1 new signature
    gpg: Total number processed: 2
    gpg:              unchanged: 1
    gpg:         new signatures: 1
    gpg:       secret keys read: 1
    gpg:   secret keys imported: 1

Teraz jeszcze zweryfikujmy czy jest tam klucz prywatny:

    $ gpg -K
    /home/morfik/.gnupg/secring.gpg
    --------------------------------
    sec#  4096R/0x0339957BF6449112 2014-09-21 [expires: 2016-09-20]
          Key fingerprint = 6DF3 C313 A0FE 1E7F 168D  F9DC 0339 957B F644 9112
    uid                            Mikhail Morfikov (testing key)
    uid                            Mikhail Morfikov (another testing key)
    ssb   4096R/0x9845A1DBE59AC095 2014-09-21
    ssb   4096R/0xDBA0A9602AF715C1 2014-09-21

Pamiętajmy by zaktualizować informacje o kluczu na serwerze kluczy, jak i również o skopiowaniu
plików, które potworzyliśmy podczas operacji usuwania klucza prywatnego gdzieś w bezpieczne,
zaszyfrowane miejsce.

Hash w `sec#` wskazuje, że może i info o kluczu prywatnym jest wyświetlane ale samego klucza jako
takiego tam nie ma.

## Test klucza GPG

Mając już własny klucz, sprawdźmy czy przejdzie on testy bezpieczeństwa. W tym celu potrzebne będą
nam narzędzia zawarte w pakiecie `hopenpgp-tools` . Testujemy klucz:

    $ hkt export-pubkeys '6DF3 C313 A0FE 1E7F 168D  F9DC 0339 957B F644 9112' | hokey lint


    Key has potential validity: good
    Key has fingerprint: 6DF3 C313 A0FE 1E7F 168D  F9DC 0339 957B F644 9112
    Checking to see if key is OpenPGPv4: V4
    Checking to see if key is RSA or DSA (>= 2048-bit): RSA 4096
    Checking user-ID- and user-attribute-related items:
      Mikhail Morfikov (another testing key) :
        Self-sig hash algorithms: [SHA512]
        Preferred hash algorithms:
          [SHA512,SHA384,SHA256,SHA224]
        Key expiration times:
          [1y11m29d81000s = Tue Sep 20 17:04:03 UTC 2016]
      Mikhail Morfikov (testing key) :
        Self-sig hash algorithms: [SHA512]
        Preferred hash algorithms:
          [SHA512,SHA384,SHA256,SHA224]
        Key expiration times:
          [1y11m29d81000s = Tue Sep 20 17:04:03 UTC 2016]

Poprawny klucz powinien przejść wszystkie etapy testu. Jeśli z jakiegoś powodu gdzieś się zapali
czerwona lampka, znaczy, że przydałoby się poprawić coś w naszym kluczu. Na szczęście nie ma takiej
potrzeby i mój klucz przeszedł test bez problemu.

## Test szyfrowania/podpisywania

Jako, że nasz klucz GPG nie posiada już części prywatnej, przydałoby się sprawdzić czy jest on w
dalszym ciągu użyteczny, czyli czy potrafi szyfrować, deszyfrować i podpisywać wiadomości. Poniżej
jest przeprowadzony test szyfrowania kawałka tekstu:

    $ touch wiadomosc
    $ echo "To jest bardzo tajny text" > wiadomosc
    $ gpg --encrypt --armor -r morfik@nsa.com wiadomosc

    You need a passphrase to unlock the secret key for
    user: "Mikhail Morfikov (testing key) "
    4096-bit RSA key, ID 0xDBA0A9602AF715C1, created 2014-09-21
             (subkey on main key ID 0x0339957BF6449112)

Wygląda na to, że klucz GPG zaszyfrował wiadomość. Można to także poznać po utworzeniu pliku
`wiadomosc.asc` , w którym to mamy blok tekstu ujęty w `-----BEGIN PGP MESSAGE-----` oraz `-----END
PGP MESSAGE-----` .

Spróbujmy teraz odszyfrować tę wiadomość:

    $ gpg --decrypt wiadomosc.asc
    ...
    To jest bardzo tajny text

Jak widzimy powyżej, wiadomość została odszyfrowana.

Spróbujmy teraz podpisać zaszyfrowaną wiadomość:

    $ gpg --encrypt --sign --armor -r morfik@nsa.com wiadomosc

Odszyfrowanie i weryfikacja podpisu:

    $ gpg --decrypt wiadomosc.asc
    ...
    To jest bardzo tajny text
    gpg: Signature made Sun 21 Sep 2014 10:14:46 PM CEST
    gpg:                using RSA key 0xDBA0A9602AF715C1
    gpg: Good signature from "Mikhail Morfikov (testing key) "
    gpg:                 aka "Mikhail Morfikov (another testing key) "
    Primary key fingerprint: 6DF3 C313 A0FE 1E7F 168D  F9DC 0339 957B F644 9112
         Subkey fingerprint: 3CDD 36C1 9AF3 4E60 60E6  6103 DBA0 A960 2AF7 15C1

Informacja o `Good signature` mówi raczej sama za siebie.

Jeszcze tak tylko w ramach formalności, utwórzmy sobie podpis dołączony do wiadomości, bez jej
szyfrowania:

    $ gpg --clearsign -r morfik@nsa.com wiadomosc

    $ cat wiadomosc.asc
    -----BEGIN PGP SIGNED MESSAGE-----
    Hash: SHA512

    To jest bardzo tajny text
    -----BEGIN PGP SIGNATURE-----

    iQIcBAEBCgAGBQJUHz+gAAoJEM0EaBB3G2UgdBAQANhe27VYRSr+rz2NQYQIgGak
    QlbBXZiV37gRtDjgiskHaCenlhLN7GuApRXMIaEgbruC25ukc/v3uBomSHf/CgU+
    5ORPE/9hJOoc2uqTeVfBO+jaNEQEFV0+LJLIa1uSM6F6cAmsdMfZKQqSpi7Jbfv/
    FEfqYbkCPthELmBkF9My8oarh8lHFrNA6/VxG9xQI6/ht1SgYaMFBxGCVT2sgLN9
    4SVDs89JwXwEf6oZzKYfE7xdR2iyOHAHQMuTdmEV76nqDkP+aFAp66AOsIKxn/9C
    bUhxM9Bn23uW2YE3yy0PVk3GiK7gKqGv2oJWfBPef98jlzjk0kHSwCkAHzn/j4q4
    GqOpjAZRgxw/63P2+8eBN9n7YPO7F5D8/snwpGOSRK5rEzMoQO+/o6rSh9HnTCMX
    5bLP6NLTG1r7l44wJ/a7nYDy3FaEnpQjyTV1K93yhzsHufvGzwccf+62JiT8tuM5
    AdAnhHbWHuIAxFRN8vjWRkc3XVPUeztmBHhWROxsUcZZZHx09ruRtmIqZbwqKg/q
    kC/dvMzAhTRHjghYZaQ28raUovQ4saAOZ+88pnXzikAoMCyoFWFqamayPT2j1c5p
    G8rGFsOWGkfNdF498+eMB13NcpSGcXB9bN7N/DU6/PaeIcWp9knp/eHqAW4pToTd
    dNkJw0YT4gR1IUdWTm7M
    =lstD
    -----END PGP SIGNATURE-----

Teraz jeszcze weryfikujemy powyższą sygnaturę:

    $ gpg --verify wiadomosc.asc
    gpg: Signature made Sun 21 Sep 2014 10:18:54 PM CEST
    gpg:                using RSA key 0xDBA0A9602AF715C1
    gpg: Good signature from "Mikhail Morfikov (testing key) "
    gpg:                 aka "Mikhail Morfikov (another testing key) "
    Primary key fingerprint: 6DF3 C313 A0FE 1E7F 168D  F9DC 0339 957B F644 9112
         Subkey fingerprint: 3CDD 36C1 9AF3 4E60 60E6  6103 DBA0 A960 2AF7 15C1
