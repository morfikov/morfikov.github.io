---
author: Morfik
categories:
- Linux
date: "2019-01-26T10:10:12Z"
published: true
status: publish
tags:
- debian
- kernel
- moduły-kernela
title: Jak ręcznie zweryfikować sygnaturę modułu kernela linux
---

Bawiąc się ostatnio kernelem linux na dystrybucji Debian i opcjami mającymi poprawić jego
bezpieczeństwo, włączyłem
sobie [mechanizm podpisywania modułów]({{< baseurl >}}/post/automatyczne-podpisywanie-modulow-kernela-przez-dkms/).
W ten sposób żaden zewnętrzny moduł nie zostanie załadowany przez jądro operacyjne, no chyba, że
taki moduł będzie podpisany przez ten sam klucz co i kernel. Zdziwiłem się odrobinę, gdy moim
oczom pokazał się hash `md4` w wyjściu polecenia `modinfo` . Jak się okazało później, to niezbyt
dokładne zinterpretowanie wiadomości PKCS#7 przez `kmod` było (i nadal jest) wynikiem [błędu
obecnego w tym pakiecie od już paru lat](https://bugzilla.redhat.com/show_bug.cgi?id=1320921). W
efekcie `modinfo` nie jest w stanie zweryfikować tej sygnatury, a w moim umyśle zaistniało pytanie:
czy istnieje w ogóle możliwość manualnego sprawdzenia czy ta sygnatura jest w porządku? Kernel co
prawda ten cały zabieg przeprowadza automatycznie ale przydałoby się ręcznie zweryfikować
poprawność sygnatury modułu i przy okazji obadać sobie co tak naprawdę się dzieje podczas tego
procesu.

<!--more-->
## Jak wydobyć wiadomość PKCS#7 z modułu kernela

Na potrzeby tego doświadczenia potrzebny będzie nam moduł kernela, który został już podpisany. Nie
ma przy tym znaczenia, czy moduł został podpisany na etapie budowania samego kernela, czy też
później, np. za sprawą DKMS. Potrzebny jest nam zatem jedynie plik, którego sygnaturę chcemy
zweryfikować. W tym przypadku oczywiście mamy do czynienia z modułami kernela dostarczanymi przez
mechanizm DKMS, zatem pliki, które nas będą interesować znajdują się pod
`/lib/modules/*/updates/dkms/` , gdzie `*` oznacza numerek kernela.

Mając podpisany moduł, trzeba z niego
wydobyć [wiadomość PKCS#7](https://pl.wikipedia.org/wiki/PKCS). Do tego celu można skorzystać ze
skryptu dostarczanego wraz z kernelem (plik `scripts/extract-module-sig.pl` ):

    # cd /usr/src/linux-source-4.20/
    # scripts/extract-module-sig.pl -s /lib/modules/4.20.0-amd64-morficzny/updates/dkms/xt_ACCOUNT.ko > /tmp/modsig
    Read 50200 bytes from module file
    Found magic number at 50200
    Found PKCS#7/CMS encapsulation
    Found 760 bytes of signature [308202f406092a864886f70d010702a0]

## Jak wydobyć certyfikat x509 i klucz publiczny z kernela

Następnie musimy wydobyć certyfikat x509 i klucz publiczny z obrazu kernela. Trzeba tutaj zwrócić
uwagę na fakt, że interesuje nas jedynie niespakowany obraz, czyli plik `vmlinux` , a nie
`vmlinuz` . Plik `vmlinux` jest dostępny w katalogu ze źródłami kernela. By wydobyć certyfikat i
klucz publiczny, wpisujemy kolejno te dwa poniższe polecenia:

    # scripts/extract-sys-certs.pl vmlinux /tmp/cert.x509
    Have 34 sections
    Have 130812 symbols
    Have 1513 bytes of certs at VMA 0xffffffff8310d838
    Certificate list in section .init.data
    Certificate list at file offset 0x250d838

    # openssl x509 -pubkey -noout -inform der -in /tmp/cert.x509 -out /tmp/pubkey

Powinniśmy w tej chwili mieć trzy pliki w katalogu `/tmp/` : `modsig` , `cert.x509 ` oraz `pubkey` .

## Jak rozkodować strukturę ASN.1 i wyciągnąć sygnaturę modułu

W skład pakietu `openssl` wchodzi narzędzie `asn1parse` , dzięki któremu możemy
rozkodować [strukturę ASN.1](https://pl.wikipedia.org/wiki/Abstract_Syntax_Notation_One) i wydobyć
całą wiadomość PKCS#7 w czytelnej dla człowieka formie. Trzeba jedynie wskazać plik wydobytej
wcześniej wiadomości PKCS#7:

    # openssl asn1parse -inform der -in modsig
        0:d=0  hl=4 l= 756 cons: SEQUENCE
        4:d=1  hl=2 l=   9 prim: OBJECT            :pkcs7-signedData
       15:d=1  hl=4 l= 741 cons: cont [ 0 ]
       19:d=2  hl=4 l= 737 cons: SEQUENCE
       23:d=3  hl=2 l=   1 prim: INTEGER           :01
       26:d=3  hl=2 l=  13 cons: SET
       28:d=4  hl=2 l=  11 cons: SEQUENCE
       30:d=5  hl=2 l=   9 prim: OBJECT            :sha512
       41:d=3  hl=2 l=  11 cons: SEQUENCE
       43:d=4  hl=2 l=   9 prim: OBJECT            :pkcs7-data
       54:d=3  hl=4 l= 702 cons: SET
       58:d=4  hl=4 l= 698 cons: SEQUENCE
       62:d=5  hl=2 l=   1 prim: INTEGER           :01
       65:d=5  hl=3 l= 148 cons: SEQUENCE
       68:d=6  hl=2 l= 124 cons: SEQUENCE
       70:d=7  hl=2 l=  11 cons: SET
       72:d=8  hl=2 l=   9 cons: SEQUENCE
       74:d=9  hl=2 l=   3 prim: OBJECT            :countryName
       79:d=9  hl=2 l=   2 prim: PRINTABLESTRING   :IT
       83:d=7  hl=2 l=  18 cons: SET
       85:d=8  hl=2 l=  16 cons: SEQUENCE
       87:d=9  hl=2 l=   3 prim: OBJECT            :localityName
       92:d=9  hl=2 l=   9 prim: UTF8STRING        :LocalHost
      103:d=7  hl=2 l=  20 cons: SET
      105:d=8  hl=2 l=  18 cons: SEQUENCE
      107:d=9  hl=2 l=   3 prim: OBJECT            :organizationName
      112:d=9  hl=2 l=  11 prim: UTF8STRING        :Morfikownia
      125:d=7  hl=2 l=  36 cons: SET
      127:d=8  hl=2 l=  34 cons: SEQUENCE
      129:d=9  hl=2 l=   3 prim: OBJECT            :commonName
      134:d=9  hl=2 l=  27 prim: UTF8STRING        :Key for morfikownias kernel
      163:d=7  hl=2 l=  29 cons: SET
      165:d=8  hl=2 l=  27 cons: SEQUENCE
      167:d=9  hl=2 l=   9 prim: OBJECT            :emailAddress
      178:d=9  hl=2 l=  14 prim: IA5STRING         :morfik@nsa.com
      194:d=6  hl=2 l=  20 prim: INTEGER           :08D5CCEF11AF1F630CA6C72AB2DB71A7F041DF24
      216:d=5  hl=2 l=  11 cons: SEQUENCE
      218:d=6  hl=2 l=   9 prim: OBJECT            :sha512
      229:d=5  hl=2 l=  13 cons: SEQUENCE
      231:d=6  hl=2 l=   9 prim: OBJECT            :rsaEncryption
      242:d=6  hl=2 l=   0 prim: NULL
      244:d=5  hl=4 l= 512 prim: OCTET STRING      [HEX DUMP]:
      B3239B84269AF9B6100EC4408E137D972F464A5F8B1059313818B6E346839B67748AC6F4D1
      45F179ADD73ED2E44A101953E6D4BFD53F589E5BE3F3B62CD530D23B7055C622B7596E323E
      753E097D58F487B86E93061B14709FDA0B92E7EFED6ABF474B13DE13AC0C68ABA3724A7E25
      9C47311F5E1C1CC5EBCE9621D9DDA4422768A6464B18314E7682E3DF17DA9AF43CEBC2B5ED
      3523EF7FDAE2109ED9605853505CC86FF1F5DC00AEE90D1E541097471ED540C2B6751DBD6F
      B4F5CBFF732E13E3217AA4103D19C4E9493442690227C4A12D4E8021C60C41515E912A7B6C
      8D88F53EA225D64C35EF351A8175883FF9CEC8BD7CD5A3014895805D9BDFBCE35AEE16FD4D
      C0182A17A07C984F19753346A76D938855DE6AB6D056250697A30BBCA68E935580C6213343
      1CD40DC3094B99F7F34ADDBC0967B4B07DD2A627B85C9B0A9EB999C078CEE5BCF44A393A7F
      45B34159CD2BEBD01A60643D933416D12F72F8CBEA8FE21159941817E575B32EDC8C39F7C3
      76776BD2B39A6D9CA14E26614406185E221633CB4BEFC3126A66D2E8DAEA7EE9618727DCA1
      3660804ECBB6C6198E4FBF8863CF2BC5E48EB9FD056B51725F1A06962C693F2634E02BE2E9
      1F021845147A837B2B279BD6398F64FD41EA7CAD890F762A557A29A7734339858C985DD16C
      2CB120F38A7CBA68DFF7EA1E8A9D922A998E3BEA881A850B2D21308A4C1146

Linijka mająca `[HEX DUMP]` w wyjściu wyżej (pocięta dla czytelności) to część wiadomości PKCS#7
odpowiedzialna za samą sygnaturę (dane binarne zakodowane w HEX). Musimy ten kawałek sobie wykroić
do osobnego pliku przy pomocy `dd` . Ale zanim to zrobimy, to musimy pierw ustalić co tak naprawdę
chcemy wykroić.

W tej ostatniej linijce, zaraz na początku mamy trzy wartości: `244` , `hl=4` oraz `l= 512` .
Pierwsza liczba to offset kolejnego nagłówka, z kolei `hl` to długość nagłówka, a `l` to długość
danych. Całkowity offset dla `dd` to 244+4=248, a długość to 512 bajtów, co odpowiada rozmiarowi
użytego hash'a przy podpisywaniu modułu (w tym przypadku został użyty `sha512` ). Zatem, by wykroić
ten hash, wpisujemy poniższe polecenie:

    # dd if=/tmp/modsig of=/tmp/signed-sha512.bin bs=1 skip=$[ 244 + 4 ] count=512
    512+0 records in
    512+0 records out
    512 bytes copied, 0.00649218 s, 78.9 kB/s

Możemy porównać zapis HEX w pliku `signed-sha512.bin` z tym co wcześniej było widoczne w
`[HEX DUMP]` :

    # hexdump -C /tmp/signed-sha512.bin
    00000000  b3 23 9b 84 26 9a f9 b6  10 0e c4 40 8e 13 7d 97  |.#..&......@..}.|
    00000010  2f 46 4a 5f 8b 10 59 31  38 18 b6 e3 46 83 9b 67  |/FJ_..Y18...F..g|
    00000020  74 8a c6 f4 d1 45 f1 79  ad d7 3e d2 e4 4a 10 19  |t....E.y..>..J..|
    00000030  53 e6 d4 bf d5 3f 58 9e  5b e3 f3 b6 2c d5 30 d2  |S....?X.[...,.0.|
    00000040  3b 70 55 c6 22 b7 59 6e  32 3e 75 3e 09 7d 58 f4  |;pU.".Yn2>u>.}X.|
    00000050  87 b8 6e 93 06 1b 14 70  9f da 0b 92 e7 ef ed 6a  |..n....p.......j|
    00000060  bf 47 4b 13 de 13 ac 0c  68 ab a3 72 4a 7e 25 9c  |.GK.....h..rJ~%.|
    00000070  47 31 1f 5e 1c 1c c5 eb  ce 96 21 d9 dd a4 42 27  |G1.^......!...B'|
    00000080  68 a6 46 4b 18 31 4e 76  82 e3 df 17 da 9a f4 3c  |h.FK.1Nv.......<|
    00000090  eb c2 b5 ed 35 23 ef 7f  da e2 10 9e d9 60 58 53  |....5#.......`XS|
    000000a0  50 5c c8 6f f1 f5 dc 00  ae e9 0d 1e 54 10 97 47  |P\.o........T..G|
    000000b0  1e d5 40 c2 b6 75 1d bd  6f b4 f5 cb ff 73 2e 13  |..@..u..o....s..|
    000000c0  e3 21 7a a4 10 3d 19 c4  e9 49 34 42 69 02 27 c4  |.!z..=...I4Bi.'.|
    000000d0  a1 2d 4e 80 21 c6 0c 41  51 5e 91 2a 7b 6c 8d 88  |.-N.!..AQ^.*{l..|
    000000e0  f5 3e a2 25 d6 4c 35 ef  35 1a 81 75 88 3f f9 ce  |.>.%.L5.5..u.?..|
    000000f0  c8 bd 7c d5 a3 01 48 95  80 5d 9b df bc e3 5a ee  |..|...H..]....Z.|
    00000100  16 fd 4d c0 18 2a 17 a0  7c 98 4f 19 75 33 46 a7  |..M..*..|.O.u3F.|
    00000110  6d 93 88 55 de 6a b6 d0  56 25 06 97 a3 0b bc a6  |m..U.j..V%......|
    00000120  8e 93 55 80 c6 21 33 43  1c d4 0d c3 09 4b 99 f7  |..U..!3C.....K..|
    00000130  f3 4a dd bc 09 67 b4 b0  7d d2 a6 27 b8 5c 9b 0a  |.J...g..}..'.\..|
    00000140  9e b9 99 c0 78 ce e5 bc  f4 4a 39 3a 7f 45 b3 41  |....x....J9:.E.A|
    00000150  59 cd 2b eb d0 1a 60 64  3d 93 34 16 d1 2f 72 f8  |Y.+...`d=.4../r.|
    00000160  cb ea 8f e2 11 59 94 18  17 e5 75 b3 2e dc 8c 39  |.....Y....u....9|
    00000170  f7 c3 76 77 6b d2 b3 9a  6d 9c a1 4e 26 61 44 06  |..vwk...m..N&aD.|
    00000180  18 5e 22 16 33 cb 4b ef  c3 12 6a 66 d2 e8 da ea  |.^".3.K...jf....|
    00000190  7e e9 61 87 27 dc a1 36  60 80 4e cb b6 c6 19 8e  |~.a.'..6`.N.....|
    000001a0  4f bf 88 63 cf 2b c5 e4  8e b9 fd 05 6b 51 72 5f  |O..c.+......kQr_|
    000001b0  1a 06 96 2c 69 3f 26 34  e0 2b e2 e9 1f 02 18 45  |...,i?&4.+.....E|
    000001c0  14 7a 83 7b 2b 27 9b d6  39 8f 64 fd 41 ea 7c ad  |.z.{+'..9.d.A.|.|
    000001d0  89 0f 76 2a 55 7a 29 a7  73 43 39 85 8c 98 5d d1  |..v*Uz).sC9...].|
    000001e0  6c 2c b1 20 f3 8a 7c ba  68 df f7 ea 1e 8a 9d 92  |l,. ..|.h.......|
    000001f0  2a 99 8e 3b ea 88 1a 85  0b 2d 21 30 8a 4c 11 46  |*..;.....-!0.L.F|
    00000200

Widać, że jest to dokładnie ten sam ciąg binarny.

## Weryfikacja sygnatury modułu kernela

Sygnatura siedzi sobie już w osobnym pliku i tę sygnaturę możemy już ręcznie zweryfikować:

    # openssl rsautl -verify -pubin -inkey /tmp/pubkey < /tmp/signed-sha512.bin > /tmp/verified.bin

W `-inkey` podajemy ścieżkę do klucza publicznego wydobytego z kernela zaraz na początku artykułu.
To powyższe polecenie stworzy nam plik `verified.bin` , który także jest w formacie ASN.1 . Wygląda
on tak:

    # hexdump -C /tmp/verified.bin
    00000000  30 51 30 0d 06 09 60 86  48 01 65 03 04 02 03 05  |0Q0...`.H.e.....|
    00000010  00 04 40 8a 4f f7 fc be  62 df a3 f9 44 8c 1b 36  |..@.O...b...D..6|
    00000020  87 44 0d 66 f8 2d 39 86  8e bc dd 63 24 78 8b d2  |.D.f.-9....c$x..|
    00000030  df 2f 00 6b e7 c5 55 90  24 99 96 a7 2b e4 f3 7c  |./.k..U.$...+..||
    00000040  e2 c5 d0 3a 09 24 47 e9  36 f3 92 ee 75 68 68 17  |...:.$G.6...uhh.|
    00000050  2a e1 08                                          |*..|
    00000053

Możemy go zdekodować w poniższy sposób:

    # openssl asn1parse -inform der -in /tmp/verified.bin
        0:d=0  hl=2 l=  81 cons: SEQUENCE
        2:d=1  hl=2 l=  13 cons: SEQUENCE
        4:d=2  hl=2 l=   9 prim: OBJECT            :sha512
       15:d=2  hl=2 l=   0 prim: NULL
       17:d=1  hl=2 l=  64 prim: OCTET STRING      [HEX DUMP]:
       8A4FF7FCBE62DFA3F9448C1B3687440D66F82D39868EBCDD6324788BD2DF2F006BE7C5559
       0249996A72BE4F37CE2C5D03A092447E936F392EE756868172AE108

Mamy tutaj informację na temat użytej funkcji skrótu oraz hash pliku, pod którym została złożona
sygnatura (zanim plik został podpisany). Zatem ten ciąg binarny ( w `[HEX DUMP]` ) musi pasować do
tego co zostanie zwrócone przez polecenie `sha512sum` na pliku modułu kernela.

    # sha512sum /lib/modules/4.20.0-amd64-morficzny/updates/dkms/xt_ACCOUNT.ko
    8a4ff7fcbe62dfa3f9448c1b3687440d66f82d39868ebcdd6324788bd2df2f006be7c5559
    0249996a72be4f37ce2c5d03a092447e936f392ee756868172ae108

Oczywiście to powyższe polecenie dotyczy modułu kernela, który jeszcze nie został podpisany. Jeśli
byśmy spróbowali wyciągnąć hash z podpisanego modułu, to ten będzie inny, jako, że po podpisaniu
modułu [na jego końcu doczepiana jest sygnatura](https://www.kernel.org/doc/html/v4.17/admin-guide/module-signing.html#signed-modules-and-stripping).
Jeśli nie dysponujemy źródłowym plikiem, to zawsze możemy uciąć tę sygnaturę z modułu w poniższy
sposób:

    # strip --strip-debug /lib/modules/4.20.0-amd64-morficzny/updates/dkms/xt_ACCOUNT.ko

I dopiero wtedy porównać czy hash się zgadza. Jeśli tak, to sygnatura została pomyślnie
zweryfikowana.

Więcej informacji można
znaleźć [tutaj](https://unix.stackexchange.com/questions/493170/how-to-verify-a-kernel-module-signature)
i [tutaj](https://qistoph.blogspot.com/2012/01/manual-verify-pkcs7-signed-data-with.html).
