---
author: Morfik
categories:
- Linux
date: "2015-06-14T23:19:51Z"
date_gmt: 2015-06-14 21:19:51 +0200
published: true
status: publish
tags:
- bezpieczeństwo
- gpg
title: Konfiguracja GPG w pliku gpg.conf
---

Narzędzie `gpg` posiada swój własny plik konfiguracyjnych, który zwykle jest zlokalizowany w
`~/.gnupg/gpg.conf` . Można w nim sprecyzować większość z opcji, które zwykle są podawane w
terminalu przy wywoływaniu polecenia `gpg` . Po zdefiniowaniu odpowiednich wpisów w pliku
konfiguracyjnym, nie będziemy musieli już wyraźnie podawać tych parametrów ilekroć będziemy chcieli
skorzystać z `gpg` . Przy okazji szukania info o kluczach GPG, natknąłem się na dość ciekawy
[artykuł na temat GnuPG](https://riseup.net/security/message-security/openpgp/best-practices). Jest
tam sporo informacji, które są wielce użyteczne w procesie konfiguracji tego narzędzia poprawiając
tym samym dość znacznie bezpieczeństwo komunikacji.

<!--more-->
## Serwery kluczy GPG

Standardowo pakiet `gnupg` jest zainstalowany w debianie i wszystkie potrzebne nam narzędzia są już
dostarczone i gotowe do użycia. Jest szereg rzeczy, które musimy wziąć pod uwagę przy konfigurowaniu
GPG, pierwszą z nich jest serwer kluczy z jakiego mamy zamiar korzystać.

Jeśli nie odświeżamy swoich publicznych kluczy regularnie, możemy przegapić pewne zdarzenia, np.
unieważnienie któregoś z nich. Zwykle użytkownicy przesyłają informacje o zmianach w swoich kluczach
do serwera kluczy i jeśli używamy jakiegoś keyservera, który działa prawidłowo, możemy skonfigurować
swoją maszynę by ta pobierała aktualizacje kluczy regularnie.

Wiele klientów OpenPGP posiada zwykle jeden skonfigurowany keyserver. To nie jest idealne
rozwiązanie, ponieważ jeśli ten serwer padnie albo, co gorsza, zdaje się działać prawidłowo, a tak
naprawdę są z nim jakieś problemy, możemy nie otrzymać krytycznych aktualizacji kluczy. Dlatego też
zalecane jest używanie [puli serwerów sks](https://sks-keyservers.net/overview-of-pools.php).
Maszyny w puli są regularnie sprawdzane czy aby działają prawidłowo i jeśli któryś ma jakieś
problemy, to zostanie usunięty z puli.

Co może człowieka zdziwić, to fakt, że zapytania o klucze GPG do sporej ilości serwerów nie wędrują
zaszyfrowanym kanałem używającym protokołu `hkps` . By z niego korzystać, trzeba doinstalować pakiet
`gnupg-curl` , po czym pobrać certyfikat serwera i
[zweryfikować](https://sks-keyservers.net/verify_tls.php) odcisk palca (fingerprint) certyfikatu.

### Weryfikacja tożsamości puli serwerów

Jak możemy wyczytać na stronie sks, certyfikat X.509 podpisany przez COMODO powinien mieć odcisk
palca SHA1 w postaci `0B:2F:DF:CE:34:17:93:AD:74:3A:4B:79:6D:0A:B9:9C:20:8D:19:A7` . Dodatkowo jest
też wzmianka na temat faktu, że ten odcisk palca nie pasuje do klucza RSA dla tego certyfikatu z
powodu różnic w informacjach jakie są brane pod uwagę przy tworzeniu tego odcisku. Generalnie rzecz
biorąc, ID klucza (KeyID) certyfikatu OpenPGP to `0xd71fd9994af34f0b` i można ten klucz odnaleźć w
puli sks. Odcisk palca tego klucza to `878F FB44 5E6E 13A6 4716 3BDC D71F D999 4AF3 4F0B` ,
dodatkowo ten klucz jest podpisany przez inny klucz o ID `0x0B7F8B60E3EDFAE3` . Sprawdźmy zatem czy
wszystko się zgadza.

By zweryfikować certyfikat dla strony www, klikamy myszą na kłódkę przy adresie:

![](/img/2015/06/1.gpg_.conf-weryfikacja-certyfikatu.png#medium)

Następnie w link `Certificate information` :

![](/img/2015/06/2.gpg_.conf-weryfikacja-certyfikatu-2.png#big)

Na samym dole widnieje odcisk SHA1 i jak widzimy, jest on zgodny z tym podanym na stronie, czyli
tożsamość serwera www została potwierdzona.

Musimy także przeprowadzić weryfikację puli serwerów. Wszystkie serwery w niej są podpisane przez
[CA sks-keyservers.net](https://sks-keyservers.net/sks-keyservers.netCA.pem). Deklarowany odcisk
palca tego certyfikatu to `79:1B:27:A3:8E:66:7F:80:27:81:4D:4E:68:E7:C4:78:A4:5D:5A:17` , a X509v3
Subject Key Identifier to `E4 C3 2A 09 14 67 D8 4D 52 12 4E 93 3C 13 E8 A0 8D DA B6 F3` . Pobieramy
zatem certyfikat i weryfikujemy go:

    $ mkdir ~/gpg_ca
    $ cd ~/gpg_ca
    $ wget https://sks-keyservers.net/sks-keyservers.netCA.pem

    $ openssl x509 -fingerprint -in sks-keyservers.netCA.pem
    SHA1 Fingerprint=79:1B:27:A3:8E:66:7F:80:27:81:4D:4E:68:E7:C4:78:A4:5D:5A:17
    ...

    $ openssl x509 -in sks-keyservers.netCA.pem -text -noout
    Certificate:
    ...
            X509v3 extensions:
                X509v3 Subject Key Identifier:
                    E4:C3:2A:09:14:67:D8:4D:52:12:4E:93:3C:13:E8:A0:8D:DA:B6:F3
                X509v3 Authority Key Identifier:
                    keyid:E4:C3:2A:09:14:67:D8:4D:52:12:4E:93:3C:13:E8:A0:8D:DA:B6:F3
    ...

Odcisk oraz ten X509v3 Subject Key Identifier zgadzają się, co oznacza, że wszystko jest w należytym
porządku.

Została nam jeszcze jedna rzecz do zrobienia, mianowicie weryfikacja klucza RSA:

    $ gpg --search-keys 0xd71fd9994af34f0b

    $ gpg --fingerprint 0xd71fd9994af34f0b
    pub   4096R/0xD71FD9994AF34F0B 2012-08-10
          Key fingerprint = 878F FB44 5E6E 13A6 4716  3BDC D71F D999 4AF3 4F0B
    uid                 [ unknown] https://www.sks-keyservers.net
    uid                 [ unknown] https://sks-keyservers.net

    $ gpg --list-sigs 0xd71fd9994af34f0b
    pub   4096R/0xD71FD9994AF34F0B 2012-08-10
          Key fingerprint = 878F FB44 5E6E 13A6 4716  3BDC D71F D999 4AF3 4F0B
    uid                 [ unknown] https://www.sks-keyservers.net
    sig          0x0B7F8B60E3EDFAE3 2013-10-27  Kristian Fiskerstrand
    sig 3        0xD71FD9994AF34F0B 2013-10-27  https://www.sks-keyservers.net
    uid                 [ unknown] https://sks-keyservers.net
    sig          0x0B7F8B60E3EDFAE3 2012-08-10  Kristian Fiskerstrand
    sig          0x16E0CF8D6B0B9508 2012-08-10  [User ID not found]
    sig          0x6D26C19CB75D0607 2013-07-10  [User ID not found]
    sig 3        0xD71FD9994AF34F0B 2012-08-10  https://www.sks-keyservers.net

Odcisk palca oraz ID klucza zgadzają się.

## Budujemy plik gpg.conf

Odpowiednia struktura katalogów w folderze domowym użytkownika jest tworzona przy pierwszym
wywołaniu programu `gpg` . W naszym przypadku jeszcze nie ma tam żadnych plików czy folderów i
musimy je sobie stworzyć:

    $ mkdir ~/.gnupg
    $ touch ~/.gnupg/gpg.conf

Teraz edytujemy plik `~/.gnupg/gpg.conf` i dodajemy do niego poniższe wpisy.

### Wymuszenie szyfrowania

Na początku wspomniałem, że z reguły odpytanie serwerów kluczy odbywa się przy pomocy
niezabezpieczonego protokołu. By wymusić szyfrowanie, dodajemy do pliku `gpg.conf` te dwa poniższe
wpisy.

    keyserver hkps://hkps.pool.sks-keyservers.net
    keyserver-options ca-cert-file=/home/morfik/gpg_ca/sks-keyservers.netCA.pem

Od tej chwili cała komunikacja z serwerem kluczy będzie szyfrowana, co utrudni nieco mapowanie
naszych kontaktów, tak jak to może mieć miejsce podczas odświeżania kluczy przy pomocy `gpg
--refresh-keys` , w przypadku gdy zapytania lecą tylko po hkp.

### Identyfikacja klucza

W zasadzie klucz można zidentyfikować na trzy sposoby -- po krótkim ID, po długim ID, oraz po
odcisku palca. Ten ID, który został wyrzucony przez program `gpg` , to długi ID. Krótkie ID kluczy
są długości 32 bitów (np. `E3EDFAE3` ) i ten ciąg może zostać użyty przez inny klucz. Te dłuższe ID
kluczy mają 64 bity ale one też mogą kolidować ze sobą. Jedyne wyjście by być pewnym co do ID
klucza, to używanie pełnego odcisku palca i nigdy nie powinno się polegać na krótkim czy nawet
długim ID klucza. Dlatego też, w pliku `gpg.conf` powinniśmy uwzględnić poniższe dwie opcje, by to
narzędzie pokazywało nam ten dłuższy ID oraz za każdym razem wyświetlało odcisk palca danego klucza:

    keyid-format 0xlong
    with-fingerprint

Jeśli jeszcze chodzi o klucze pobierane z publicznych key serwerów, to trzeba się mieć na baczności,
bo te klucze mogą należeć do każdego i bez weryfikacji osoby, która stoi za danym kluczem, nie da
się tak naprawdę powiedzieć czy mamy do czynienia z tym kimś. Równie dobrze można dodać pierwszy
lepszy klucz i mieć poczucie, że jest się bezpiecznym. Po zweryfikowaniu odciska palca klucza, sam
klucz można pobrać z serwera kluczy posługując się poniższym poleceniem:

    $ gpg --recv-key '878F FB44 5E6E 13A6 4716  3BDC D71F D999 4AF3 4F0B'

Apostrofy są wymagane by to polecenie zadziałało.

### Wymuszenie określonego serwera

Niektóre osoby tworzące klucze GPG mogą określić serwer, z którego ich klucz ma być pobrany.
Powinniśmy zignorować taki serwer, a klucz pobrać ze wcześniej ustawionej puli serwerów. Wobec
czego zalecane jest by dodać poniższą opcję do pliku `gpg.conf`:

    keyserver-options no-honor-keyserver-url

### Ustawienie odpowiedniego hasha

Klucz RSA powinien używać hasha sha512, dlatego też w pliku `gpg.conf` powinny znaleźć się również i
poniższe wpisy:

    cert-digest-algo SHA512
    default-preference-list SHA512 SHA384 SHA256 SHA224 AES256 AES192 AES CAST5 ZLIB BZIP2 ZIP Uncompressed

### Plik gpg.conf

Aktualny plik konfiguracyjny dla GPG znajduje się [na moim
gicie](https://github.com/morfikov/files/tree/master/configs/home/morfik/.gnupg).
