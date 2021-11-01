---
author: Morfik
categories:
- Android
date:    2019-12-13 19:26:14 +0100
lastmod: 2019-12-13 19:26:14 +0100
published: true
status: publish
tags:
- aplikacje
- apk
- bezpieczeństwo
GHissueID: 291
title: Jak zweryfikować plik APK aplikacji na Androida
---

Część użytkowników smartfonów z Androidem na pokładzie żyje w głębokim przekonaniu, że instalowanie
aplikacji spoza sklepu Google Play nie jest zbyt rozważnym posunięciem. Nie chodzi tutaj tylko o
szeroko rozumiane alternatywne źródła aplikacji, np. serwis apkmirror ale również o Yalp/Aurora
Store czy też repozytoria F-Droid. Zdaniem tych osób pobieranie aplikacji z zewnętrznych źródeł
może skompromitować bezpieczeństwo systemu oraz zagrozić naszej prywatności. No jakby nie patrzeć
wgrywanie czegokolwiek bez zastanowienia się co tak naprawdę instalujemy w systemie nie jest zbyt
mądre. Dlaczego zatem nie weryfikujemy aplikacji obecnych w oficjalnym sklepie Google? Co chwila
przecież można usłyszeć w mediach o syfie, który udało się co prawda z tego sklepu usunąć ale też
jakaś większa liczba użytkowników taką aplikację zdążyła już zainstalować i używała jej przez
dłuższy lub krótszy okres czasu. Rozumowanie na zasadzie, że aplikacje ze sklepu Google Play są
bezpieczne, bo są obecne w sklepie Google Play, daje nam jedynie fałszywe poczucie bezpieczeństwa,
które jest gorsze od całkowitego braku bezpieczeństwa, bo w tym drugim przypadku człowiek
przynajmniej jest świadom czyhających na niego niebezpieczeństw i włącza myślenie. Jak zatem
odróżnić aplikacje, które są w stanie nam wyrządzić krzywdę od tych, które tego nie mają na celu i
czy faktycznie pobieranie aplikacji na Androida z innego źródła niż oficjalny sklep Google Play
jest takie niebezpieczne?

<!--more-->
## Sklep Google Play, a alternatywne źródła aplikacji

Trzeba sobie zdać sprawę, że w sklepie Google Play znajduje się cała masa aplikacji, które nie
należą do marki Google. Niby oczywista oczywistość ale już sam fakt, że w grę wchodzą podmioty
zewnętrzne powinien nam dać do myślenia, wszak wraz ze wzrostem ilości aplikacji maleje zdolność
weryfikacji tego co tak naprawdę do sklepu Google Play trafia. Człowiek zwyczajnie nie jest w
stanie zweryfikować tych miliardów linijek kodu, a to otwiera drogę do implementowania w
aplikacjach różnych niepożądanych funkcji, które mogą działać na naszą szkodę.

Sklep Google Play to jedynie medium dystrybucyjne umożliwiające zebranie w jednym miejscu
rezultatów pracy całej masy różnych deweloperów aplikacji. Dzięki takiemu rozwiązaniu nie trzeba
biegać po setkach stron w celu skompletowania zestawu appek, które chcemy mieć w systemie i później
martwić się jeszcze o proces aktualizacji tak pozyskanego oprogramowania. Niemniej jednak, ta
zaleta sklepu Google Play jest też i jego wadą zarazem, bo taki centralny punkt dystrybucyjny może
stać się celem ataków. Dlatego nie powinno się ufać środkom dystrybucji i to bez znaczenia czy mamy
do czynienia z oficjalnym sklepem Google Play czy innymi alternatywnymi źródłami aplikacji, np
F-Droid czy serwis apkmirror. Jeśli cenimy sobie bezpieczeństwo i prywatność, to powinniśmy raczej
starać się ufać konkretnym deweloperom, którzy wyrobili sobie reputację działając dla dobra jakiejś
społeczności.

## Schematy podpisywania plików APK

Aplikacje dostępne na Androida (w tym też te obecne w sklepie Google Play) są podpisane kluczami
deweloperów konkretnych projektów na etapie budowania pakietu `.apk` . Dane zawarte w tym pliku są
podpisane przy wykorzystaniu [jednego z trzech schematów][2]: JAR-signed APK (dla Androida <7) i
APK Signature Scheme v2 (oraz v3 dla Androida 9 i późniejszych). Po pobraniu aplikacji ale przed
jej instalacją w systemie, Android będzie próbował zweryfikować sygnaturę zanim zezwoli na
kontynuację procesu instalacyjnego.

### Problematyczny JAR-signed APK

W przypadku JAR-signed APK mamy do czynienia z [podpisaniem zwykłego pliku JAR][3], gdzie sumy
kontrolne są generowane osobno dla każdego z plików aplikacji. W przypadku tego schematu występują
jednak problemy z bezpieczeństwem, jako że archiwum ZIP (którym jest plik `.apk` ) może zawierać
dodatkowe bajty na początku, jak i również przed i między kolejnymi wpisami ZIP. Podpis, który jest
generowany pod takim archiwum (i później weryfikowany) bierze pod uwagę jedynie same wpisy w ZIP i
ignoruje zarazem wszystkie dodatkowe bity. Taki stan rzeczy umożliwia wstrzyknięcie pliku `.dex` w
plik `.apk` bez wpływu na złożoną sygnaturę (pozostanie ona taka sama i, co gorsza, będzie
poprawna). Podczas instalacji takiego zmienionego pakietu, Android zweryfikuje sygnaturę nie
wykrywając przy tym zmian w ZIP ale Dalvik (maszyna wirtualna Androida) zinterpretuje ten plik
`.apk` jako plik `.dex` i wykona jego kod ([podatność Janus (CVE-2017-13156)][19]).

Całą taką paczkę `.apk` trzeba też wypakować w celu weryfikacji jej zawartości, a to z kolei
spowalnia cały proces instalacji i zwiększa wykorzystanie zasobów sprzętowych telefonu (pamięć RAM
i procesor) i cierpi na tym też bateria.

### APK Signature Scheme v2/v3

Z racji, że JAR-signed APK był dość nieefektywny i miał parę podatności, to by sobie z nimi
poradzić opracowano APK Signature Scheme v2 i później też v3, w których to [podpisywany jest cały
plik .apk][4] wraz z metadanymi ZIP i zmiana nawet jednego bitu w takiej paczce będzie skutkować
błędem przy weryfikacji integralności danych, które są w niej zwarte. Jeśli chodzi zaś o różnice
między v2 i v3 to w przypadku v3 zostały wprowadzone [dodatkowe informacje][20], które są
umieszczane w bloku podpisu (signing block).

![apk-integrity-protection](/img/2019/12/001-apk-integrity-protection.png#huge)

### Wsparcie dla wszystkich schematów podpisu

Wszystkie te trzy schematy podpisywania nie są kompatybilne wstecznie ze sobą. Jeśli dana aplikacja
ma być instalowana w starszych i nowszych wersjach Androida, to trzeba ją podpisać przy pomocy
każdego ze schematów. Później przy instalacji aplikacji, system podąża według poniższych
instrukcji, by taki pakiet zweryfikować:

![apk-validation-process](/img/2019/12/002-apk-validation-process.png#huge)

## Jak zweryfikować plik APK

Dysponując wiedzą, że pliki `.apk` są podpisane, to drugorzędne znaczenie ma dla nas źródło, z
którego je pobieramy. Niemniej jednak, wypadałoby zweryfikować kto się pod taką aplikacją podpisał
oraz czy paczka (albo jej pliki) została w jakiś sposób zmieniona.

Jeśli pobieramy aplikacje ze sklepu Google Play, to tam zwykle jest wyszczególniony podmiot, od
którego ta appka pochodzi. Niekiedy przy kontakcie z deweloperem jest nieco więcej informacji, np.
strona projektu, czy adres. Warto też rzucić okiem na informacje o aplikacji, bo tam mogą być
dodatkowe linki, np. do oficjalnego forum czy kodu źródłowego.

|     |    |     |
| --- | ---| --- |
| ![](/img/2019/12/003-signal-app-google-play.png#small) | ![](/img/2019/12/004-signal-app-google-play.png#small) |![](/img/2019/12/005-signal-app-google-play.png#small) |

Nie zawsze jednak wszystkie te informacje będą podane tak jak w przypadku tej powyższej aplikacji.
Ja generalnie zalecam trzymanie się z dala od tych appek, których identyfikacja podmiotu jest
problematyczna. Jeśli uda nam się zweryfikować podmiot, który stoi za aplikacją w sklepie Google
Play, i mu ufamy, to możemy taką aplikację pobrać, zainstalować i tyle z naszej strony. Android w
naszym telefonie przeprowadzi już sobie wszystkie niezbędne kroki pod kątem weryfikacji pliku `.apk`
i albo zainstaluje aplikację jeśli weryfikacja przebiegnie z powodzeniem, albo nie pozwoli na jej
instalację.

Jeśli zaś chcemy pobierać aplikacje z innych źródeł albo jedynie ciekawi nas jak taka weryfikacja
aplikacji przebiega i jak ją przeprowadzić by zweryfikować integralność danych, to poniżej jest
zamieszczona krótka instrukcja. Jako, że są w zasadzie dwa różne schematy podpisów, to trzeba zająć
się nimi z osobna, bo taki plik `.apk` można instalować na starszych i nowszych Androidach i tylko
ten fakt będzie wpływał na to, który schemat podpisu zostanie użyty podczas weryfikacji.

Za przykładową aplikację posłuży nam Signal. Plik tej appki można pobrać ze sklepu Google Play, z
[oficjalnej strony projektu][1], lub z każdego dowolnego miejsca, które nam wpadnie w łapki.

### Weryfikacja schematu JAR-signed APK

Gdybyśmy się jedynie ograniczyli do pobrania pliku `.apk` z dowolnego źródła, do którego odesłałaby
nas wyszukiwarka, to faktycznie moglibyśmy się obawiać instalacji tego pliku na urządzeniu z
Androidem. Niemniej jednak, pobranie pliku to jedna sprawa, a jego weryfikacja, to osobna bajka.

W każdej paczce `.apk` znajduje się katalog `META-INF/` , w nim zaś znajduje się kilka użytecznych
dla nas plików:

- `META-INF/MANIFEST.MF` - plik manifestu, który jest wykorzystywany do zdefiniowania rozszerzenia
i danych powiązanych z pakietem.
- `META-INF/*.SF` - plik z sygnaturami dla pliku JAR (w miejsce gwiazdki podstawia się konkretną
nazwę).
- `META-INF/*.RSA` - blokowy plik zawierający metadane certyfikatu, klucz publiczny oraz sygnaturę
powiązaną z plikiem `*.SF` o tej samej nazwie. Plik ten jest nieczytelny dla człowieka.

#### Weryfikacja certyfikatu

Na stronie projektu Signal był podany odcisk palca SHA256 certyfikatu. Wygląda on tak:

![signal-app-manual-download-certificate-hash](/img/2019/12/006-signal-app-manual-download-certificate-hash.png#huge)

Po co nam ten odcisk? Przy jego pomocy jesteśmy w stanie zweryfikować podmiot, który ten certyfikat
wystawił. Informacje o certyfikacie są dostępne w pliku `META-INF/SIGNAL_S.RSA` i możemy je uzyskać
za sprawą narzędzia `keytool` :

    $ unzip -p Signal-website-universal-release-*.apk META-INF/SIGNAL_S.RSA | keytool -printcert
    Owner: CN=Whisper Systems, OU=Research and Development, O=Whisper Systems, L=Pittsburgh, ST=PA, C=US
    Issuer: CN=Whisper Systems, OU=Research and Development, O=Whisper Systems, L=Pittsburgh, ST=PA, C=US
    Serial number: 4bfbebba
    Valid from: Tue May 25 17:24:42 CEST 2010 until: Tue May 16 17:24:42 CEST 2045
    Certificate fingerprints:
             MD5:  D9:0D:B3:64:E3:2F:A3:A7:BD:A4:C2:90:FB:65:E3:10
             SHA1: 45:98:9D:C9:AD:87:28:C2:AA:9A:82:FA:55:50:3E:34:A8:87:93:74
             SHA256: 29:F3:4E:5F:27:F2:11:B4:24:BC:5B:F9:D6:71:62:C0:EA:FB:A2:DA:35:AF:35:C1:64:16:FC:44:62:76:BA:26
    Signature algorithm name: SHA1withRSA
    Subject Public Key Algorithm: 1024-bit RSA key
    Version: 3

Mamy tutaj informacje na temat tego kto ten certyfikat wystawił, datę jego ważności oraz, to co nas
interesuje najbardziej, czyli odcisk palca SHA256, który pasuje do tego ze strony projektu Signal.
Wiemy zatem, że certyfikat należy do Whisper Systems (twórcy aplikacji Signal), no i tym samym
udało nam się zweryfikować podmiot, który się pod tym plikiem `.apk` podpisał.

Przy weryfikacji certyfikatu można także posłużyć się poniższym poleceniem:

    $ openssl pkcs7 -inform DER -in SIGNAL_S.RSA -noout -print_certs -text

#### Weryfikacja danych pliku APK

Teraz wypadałoby zweryfikować czy integralność danych w pliku `.apk` nie została naruszona w żaden
sposób. By ręcznie zweryfikować paczkę `.apk` , trzeba posłużyć się plikiem `META-INF/MANIFEST.MF` .
Poniżej znajduje się jego skrócona wersja:

    Manifest-Version: 1.0
    Built-By: Generated-by-ADT
    Created-By: Android Gradle 3.5.1

    Name: AndroidManifest.xml
    SHA-256-Digest: S9ggdRXHREOfZVUyR3gVkkj7heljgNS3N2jG6j5AgwU=

    Name: FingerprintProtocol.proto
    SHA-256-Digest: WQ9R/MGV5m9Dye4Jtl3MiLRmX2RPoEPeinb3me7wnAU=

    ...

    Name: res/xml/syncadapter.xml
    SHA-256-Digest: Bh02ycHvc/OhwewyU6h1WDtEt+v7tNgKrauGEHs8azs=

    Name: resources.arsc
    SHA-256-Digest: DJGwbk9S1VPH6Cqkqa5A/8Sb3vYVPmbCZP/7G7EYkjs=

Poza nagłówkiem (trzema pierwszymi linijkami) mamy wpisy w postaci par `Name`/`SHA-256-Digest` .
Jest ich dość sporo bo aż 3456. Plików w tej paczce zaś jest 3459, co daje różnice 3 plików. Zatem
każdy plik za wyjątkiem tych powiązanych z podpisami ( `MANIFEST.MF` , `SIGNAL_S.SF` oraz
`SIGNAL_S.RSA` ) ma wygenerowaną sumę kontrolą zakodowaną w base64 (SHA-256-Digest).

Dla przykładu weźmy sobie plik `FingerprintProtocol.proto` . By wyliczyć jego sumę kontrolną można
posłużyć się poniższym poleceniem:

    $ unzip -p Signal-website-universal-release-*.apk FingerprintProtocol.proto | \
        sha256sum | \
        xxd -r -p | \
        base64

    WQ9R/MGV5m9Dye4Jtl3MiLRmX2RPoEPeinb3me7wnAU=

Wynikowy ciąg znaków pasuje do tego w pliku manifestu, zatem jego suma kontrolna się zgadza i plik
nie został w żaden sposób zmieniony.

Podobnie Android weryfikuje każdy z plików przed instalacją aplikacji w systemie (przynajmniej
jeśli chodzi o Andki <7). Gdyby zmienić któryś z tych plików, to jego suma kontrola byłaby inna i
proces instalacji by się nie powiódł.

Oczywiście, gdyby polegać tylko na pliku `META-INF/MANIFEST.MF` , to naturalnie można by
wygenerować nową sumę kontrolną i wstawić ją w stosowne miejsce, co czyniłoby całe zabezpieczenie
bez sensu. Dlatego też z pomocą przychodzi nam plik `META-INF/SIGNAL_S.SF` , którego zawartość jest
podobna do pliku `META-INF/MANIFEST.MF` . Poniżej jest skrócona zawartość `META-INF/SIGNAL_S.SF` :

    Signature-Version: 1.0
    Created-By: 1.0 (Android)
    SHA-256-Digest-Manifest: 5GtdBgQlYLeNtJczTVbfG/rrh6YdoNxEZ2iq+BiRNA8=
    X-Android-APK-Signed: 2, 3

    Name: AndroidManifest.xml
    SHA-256-Digest: PxbxNfqXevr9OY8iXnzwOdJf4POisDv7D/KBJ7p9yaE=

    Name: FingerprintProtocol.proto
    SHA-256-Digest: fWL5eppUj0x6UVdPqHusFkpDbXpq7P0sqchfublqEnA=

    ...

    Name: res/xml/syncadapter.xml
    SHA-256-Digest: or/WIJ91gQYMoGN/XzSmsSXP4oA8Y1Z8mI7HVyO/NgY=

    Name: resources.arsc
    SHA-256-Digest: 2oWTAFfBatMDkEJCDUocCNL5r0CTK5pl+8UFmIxLYvI=

Dwie rzeczy się rzucają w oczy od razu. Pierwszą z nich jest pole `SHA-256-Digest-Manifest` w
nagłówku, którego wartość jest zakodowaną w base64 sumą kontrolną pliku `META-INF/MANIFEST.MF` :

    $ unzip -p Signal-website-universal-release-*.apk META-INF/MANIFEST.MF | \
        sha256sum | \
        xxd -r -p | \
        base64

    5GtdBgQlYLeNtJczTVbfG/rrh6YdoNxEZ2iq+BiRNA8=

Zatem plik manifestu nie został zmieniony w żaden sposób, bo jego suma kontrolna się zgadza.

Drugą rzeczą którą można zauważyć jest fakt, że wartości w `SHA-256-Digest` różnią się od tych
obecnych w `META-INF/MANIFEST.MF` . Dzieje się tak ze względu na fakt, że w przypadku
`META-INF/SIGNAL_S.SF` nie są hash'owane same pliki, a [jedynie konkretne sekcje z pliku
META-INF/MANIFEST.MF][15]. Na sekcję zaś składa się linijka z `Name:` , `SHA-256-Digest:` oraz
pusta linia (znak `CRLF`). Trochę to skomplikowane zatem najlepiej to zobrazować na przykładzie.

Sekcja pliku `FingerprintProtocol.proto` w `META-INF/MANIFEST.MF` ma poniższą postać:

![manifest-section-hash](/img/2019/12/007-manifest-section-hash.png#huge)

Jeśli taki wycinek tekstu podda się procesowi hash'owania, to otrzymamy:

    $ cat data.txt | sha256sum | xxd -r -p | base64
    fWL5eppUj0x6UVdPqHusFkpDbXpq7P0sqchfublqEnA=

I ten hash już pasuje do tego obecnego w pliku `META-INF/SIGNAL_S.SF` przy pozycji
`FingerprintProtocol.proto` .

Oczywiście plik `META-INF/SIGNAL_S.SF` również można by poddać edycji i właśnie po to jest plik
`SIGNAL_S.RSA` , który oprócz danych certyfikatu i klucza publicznego zawiera także sygnaturę
złożoną pod plikiem `META-INF/SIGNAL_S.SF` w chwili tworzenia paczki `.apk` przez dewelopera. By
taką sygnaturę złożyć potrzebny jest klucz prywatny, do którego zwykle ma dostęp dość ograniczone
grono osób, przez co bardzo ciężko jest ten podpis podrobić.

Jeśli nie wiemy czy plik `META-INF/SIGNAL_S.RSA` zawiera jakąś sygnaturę, to zawsze możemy ten fakt
sprawdzić przy pomocy poniższego polecenia:

    $ openssl asn1parse -i -inform DER -in META-INF/SIGNAL_S.RSA
        0:d=0  hl=4 l=1023 cons: SEQUENCE
        4:d=1  hl=2 l=   9 prim:  OBJECT            :pkcs7-signedData
       15:d=1  hl=4 l=1008 cons:  cont [ 0 ]
       19:d=2  hl=4 l=1004 cons:   SEQUENCE
       23:d=3  hl=2 l=   1 prim:    INTEGER           :01
       26:d=3  hl=2 l=  15 cons:    SET
       28:d=4  hl=2 l=  13 cons:     SEQUENCE
       30:d=5  hl=2 l=   9 prim:      OBJECT            :sha256
       41:d=5  hl=2 l=   0 prim:      NULL
       43:d=3  hl=2 l=  11 cons:    SEQUENCE
       45:d=4  hl=2 l=   9 prim:     OBJECT            :pkcs7-data
       56:d=3  hl=4 l= 649 cons:    cont [ 0 ]
       60:d=4  hl=4 l= 645 cons:     SEQUENCE
       64:d=5  hl=4 l= 494 cons:      SEQUENCE
       68:d=6  hl=2 l=   3 cons:       cont [ 0 ]
       70:d=7  hl=2 l=   1 prim:        INTEGER           :02
       73:d=6  hl=2 l=   4 prim:       INTEGER           :4BFBEBBA
       79:d=6  hl=2 l=  13 cons:       SEQUENCE
       81:d=7  hl=2 l=   9 prim:        OBJECT            :sha1WithRSAEncryption
       92:d=7  hl=2 l=   0 prim:        NULL
       94:d=6  hl=3 l= 134 cons:       SEQUENCE
       97:d=7  hl=2 l=  11 cons:        SET
       99:d=8  hl=2 l=   9 cons:         SEQUENCE
      101:d=9  hl=2 l=   3 prim:          OBJECT            :countryName
      106:d=9  hl=2 l=   2 prim:          PRINTABLESTRING   :US
      110:d=7  hl=2 l=  11 cons:        SET
      112:d=8  hl=2 l=   9 cons:         SEQUENCE
      114:d=9  hl=2 l=   3 prim:          OBJECT            :stateOrProvinceName
      119:d=9  hl=2 l=   2 prim:          PRINTABLESTRING   :PA
      123:d=7  hl=2 l=  19 cons:        SET
      125:d=8  hl=2 l=  17 cons:         SEQUENCE
      127:d=9  hl=2 l=   3 prim:          OBJECT            :localityName
      132:d=9  hl=2 l=  10 prim:          PRINTABLESTRING   :Pittsburgh
      144:d=7  hl=2 l=  24 cons:        SET
      146:d=8  hl=2 l=  22 cons:         SEQUENCE
      148:d=9  hl=2 l=   3 prim:          OBJECT            :organizationName
      153:d=9  hl=2 l=  15 prim:          PRINTABLESTRING   :Whisper Systems
      170:d=7  hl=2 l=  33 cons:        SET
      172:d=8  hl=2 l=  31 cons:         SEQUENCE
      174:d=9  hl=2 l=   3 prim:          OBJECT            :organizationalUnitName
      179:d=9  hl=2 l=  24 prim:          PRINTABLESTRING   :Research and Development
      205:d=7  hl=2 l=  24 cons:        SET
      207:d=8  hl=2 l=  22 cons:         SEQUENCE
      209:d=9  hl=2 l=   3 prim:          OBJECT            :commonName
      214:d=9  hl=2 l=  15 prim:          PRINTABLESTRING   :Whisper Systems
      231:d=6  hl=2 l=  30 cons:       SEQUENCE
      233:d=7  hl=2 l=  13 prim:        UTCTIME           :100525152442Z
      248:d=7  hl=2 l=  13 prim:        UTCTIME           :450516152442Z
      263:d=6  hl=3 l= 134 cons:       SEQUENCE
      266:d=7  hl=2 l=  11 cons:        SET
      268:d=8  hl=2 l=   9 cons:         SEQUENCE
      270:d=9  hl=2 l=   3 prim:          OBJECT            :countryName
      275:d=9  hl=2 l=   2 prim:          PRINTABLESTRING   :US
      279:d=7  hl=2 l=  11 cons:        SET
      281:d=8  hl=2 l=   9 cons:         SEQUENCE
      283:d=9  hl=2 l=   3 prim:          OBJECT            :stateOrProvinceName
      288:d=9  hl=2 l=   2 prim:          PRINTABLESTRING   :PA
      292:d=7  hl=2 l=  19 cons:        SET
      294:d=8  hl=2 l=  17 cons:         SEQUENCE
      296:d=9  hl=2 l=   3 prim:          OBJECT            :localityName
      301:d=9  hl=2 l=  10 prim:          PRINTABLESTRING   :Pittsburgh
      313:d=7  hl=2 l=  24 cons:        SET
      315:d=8  hl=2 l=  22 cons:         SEQUENCE
      317:d=9  hl=2 l=   3 prim:          OBJECT            :organizationName
      322:d=9  hl=2 l=  15 prim:          PRINTABLESTRING   :Whisper Systems
      339:d=7  hl=2 l=  33 cons:        SET
      341:d=8  hl=2 l=  31 cons:         SEQUENCE
      343:d=9  hl=2 l=   3 prim:          OBJECT            :organizationalUnitName
      348:d=9  hl=2 l=  24 prim:          PRINTABLESTRING   :Research and Development
      374:d=7  hl=2 l=  24 cons:        SET
      376:d=8  hl=2 l=  22 cons:         SEQUENCE
      378:d=9  hl=2 l=   3 prim:          OBJECT            :commonName
      383:d=9  hl=2 l=  15 prim:          PRINTABLESTRING   :Whisper Systems
      400:d=6  hl=3 l= 159 cons:       SEQUENCE
      403:d=7  hl=2 l=  13 cons:        SEQUENCE
      405:d=8  hl=2 l=   9 prim:         OBJECT            :rsaEncryption
      416:d=8  hl=2 l=   0 prim:         NULL
      418:d=7  hl=3 l= 141 prim:        BIT STRING
      562:d=5  hl=2 l=  13 cons:      SEQUENCE
      564:d=6  hl=2 l=   9 prim:       OBJECT            :sha1WithRSAEncryption
      575:d=6  hl=2 l=   0 prim:       NULL
      577:d=5  hl=3 l= 129 prim:      BIT STRING
      709:d=3  hl=4 l= 314 cons:    SET
      713:d=4  hl=4 l= 310 cons:     SEQUENCE
      717:d=5  hl=2 l=   1 prim:      INTEGER           :01
      720:d=5  hl=3 l= 143 cons:      SEQUENCE
      723:d=6  hl=3 l= 134 cons:       SEQUENCE
      726:d=7  hl=2 l=  11 cons:        SET
      728:d=8  hl=2 l=   9 cons:         SEQUENCE
      730:d=9  hl=2 l=   3 prim:          OBJECT            :countryName
      735:d=9  hl=2 l=   2 prim:          PRINTABLESTRING   :US
      739:d=7  hl=2 l=  11 cons:        SET
      741:d=8  hl=2 l=   9 cons:         SEQUENCE
      743:d=9  hl=2 l=   3 prim:          OBJECT            :stateOrProvinceName
      748:d=9  hl=2 l=   2 prim:          PRINTABLESTRING   :PA
      752:d=7  hl=2 l=  19 cons:        SET
      754:d=8  hl=2 l=  17 cons:         SEQUENCE
      756:d=9  hl=2 l=   3 prim:          OBJECT            :localityName
      761:d=9  hl=2 l=  10 prim:          PRINTABLESTRING   :Pittsburgh
      773:d=7  hl=2 l=  24 cons:        SET
      775:d=8  hl=2 l=  22 cons:         SEQUENCE
      777:d=9  hl=2 l=   3 prim:          OBJECT            :organizationName
      782:d=9  hl=2 l=  15 prim:          PRINTABLESTRING   :Whisper Systems
      799:d=7  hl=2 l=  33 cons:        SET
      801:d=8  hl=2 l=  31 cons:         SEQUENCE
      803:d=9  hl=2 l=   3 prim:          OBJECT            :organizationalUnitName
      808:d=9  hl=2 l=  24 prim:          PRINTABLESTRING   :Research and Development
      834:d=7  hl=2 l=  24 cons:        SET
      836:d=8  hl=2 l=  22 cons:         SEQUENCE
      838:d=9  hl=2 l=   3 prim:          OBJECT            :commonName
      843:d=9  hl=2 l=  15 prim:          PRINTABLESTRING   :Whisper Systems
      860:d=6  hl=2 l=   4 prim:       INTEGER           :4BFBEBBA
      866:d=5  hl=2 l=  13 cons:      SEQUENCE
      868:d=6  hl=2 l=   9 prim:       OBJECT            :sha256
      879:d=6  hl=2 l=   0 prim:       NULL
      881:d=5  hl=2 l=  13 cons:      SEQUENCE
      883:d=6  hl=2 l=   9 prim:       OBJECT            :rsaEncryption
      894:d=6  hl=2 l=   0 prim:       NULL
      896:d=5  hl=3 l= 128 prim:      OCTET STRING      [HEX DUMP]:
      AF5E96B4E77526640145A948CDE65B12945A0A036A52D27B630DBB6019422
      097231B6CA78EAB90DA6E3440C63C798D2E679742594E380768B63BA84D9F
      2021F50FBE63974FFE543C09A110319D144402D2C4A917604E6933C2DAB77
      2CFFB7E880A2995633A21F0E64810FF34DCA684BFDBC5D1DFEC1A75018150
      AF59404DD6B9

Ten długi ciąg na końcu to [256 bajtowa sygnatura][16] złożona pod plikiem `META-INF/SIGNAL_S.SF` .

By teraz zweryfikować tę sygnaturę, możemy posłużyć się poniższy poleceniem:

    $ openssl cms -verify -noverify -content META-INF/SIGNAL_S.SF -in META-INF/SIGNAL_S.RSA -inform der
    ...
    Verification successful

Jak widać, weryfikacja została przeprowadzona z powodzeniem, a ufając jednocześnie certyfikatowi,
można by zaufać zawartości paczki `.apk` (oczywiście, gdyby nie problemy natury bezpieczeństwa
związane z tym schematem podpisu).

#### Jarsigner

Ręczna weryfikacja paczki `.apk` jest dość upierdliwa ale cały ten proces można zautomatyzować przy
pomocy `jarsigner` . Wystarczy wpisać to poniższe polecenie w terminal:

    $ jarsigner -verbose -verify Signal-website-universal-release-*.apk

    sm     60312 Fri Nov 30 00:00:00 CET 1979 AndroidManifest.xml
    sm       455 Thu Jan 01 01:00:00 CET 1970 FingerprintProtocol.proto
    sm      3203 Thu Jan 01 01:00:00 CET 1970 LocalStorageProtocol.proto
    ...
    sm    5616312 Fri Nov 30 00:00:00 CET 1979 resources.arsc
          409456 Fri Nov 30 00:00:00 CET 1979 META-INF/SIGNAL_S.SF
            1027 Fri Nov 30 00:00:00 CET 1979 META-INF/SIGNAL_S.RSA
    s     409391 Fri Nov 30 00:00:00 CET 1979 META-INF/MANIFEST.MF

      s = signature was verified
      m = entry is listed in manifest
      k = at least one certificate was found in keystore
      i = at least one certificate was found in identity scope

    - Signed by "CN=Whisper Systems, OU=Research and Development, O=Whisper Systems, L=Pittsburgh, ST=PA, C=US"
        Digest algorithm: SHA-256
        Signature algorithm: SHA256withRSA, 1024-bit key

    jar verified.

    Warning:
    This jar contains entries whose certificate chain is invalid. Reason: PKIX path building failed:
    sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification
    path to requested target
    This jar contains entries whose signer certificate is self-signed.
    This jar contains signatures that do not include a timestamp. Without a timestamp, users may not
    be able to validate this jar after any of the signer certificates expire (as early as 2045-05-16).

    Re-run with the -verbose and -certs options for more details.

    The signer certificate will expire on 2045-05-16.

W wyjściu, które otrzymaliśmy, widoczne są wpisy poprzedzone `sm` . Każdy plik, który zostanie
zweryfikowany poprawnie (jego suma kontrolna się zgadza) będzie miał znaczek `s` . Jeśli dodatkowo
plik jest uwzględniony w `META-INF/MANIFEST.MF` , to będzie miał znaczek `m` . Wszystkie pliki w
tej paczce za wyjątkiem trzech ostatnich są uwzględnione w pliku manifestu. Sumy kontrolne plików
również się zgadzają, w tym też suma kontrolna samego pliku manifestu. Mamy również informację, że
`jar verified` , czyli archiwum ZIP zostało zweryfikowane pomyślnie. W podsumowaniu mamy również
informację, że certyfikat jest typu `self-signed` co może budzić pewne obawy ale tylko w przypadku
gdybyśmy nie władali odciskiem palca tego certyfikatu (ten który został udostępniony na stronie
Signal).

### Weryfikacja APK Signature Scheme v2/v3

Z racji, że schemat JAR-signed APK jest obecnie odradzany, to wypadałoby zainteresować się nieco
bardziej schematem APK Signature Scheme v2/v3 i zweryfikować przy jego pomocy aplikację. Nie
znalazłem nigdzie ręcznej próby weryfikacji poszczególnych informacji, tak jak to było w przypadku
JAR-signed APK, dlatego też ograniczymy się tutaj jedynie do automatu w postaci [narzędzia
apksigner][22].

#### Apksigner

Paczkę `.apk` można także zweryfikować przy pomocy `apksigner` . Niby w Debianie jest stosowny
pakiet z tym narzędziem ale nie działa on za dobrze (zwraca błąd `zsh: exec format error:
apksigner` ). Drugi problem wiąże się z posiadaniem starszej wersji `apksigner` , bo one
najwyraźniej [mają problemy z interpretacją nowszych schematów podpisów][17] i nie są w stanie
zweryfikować pliku `.apk` nawet jeśli ten jest podpisany prawidłowo. Dlatego upewnijmy się, że mamy
najnowszą wersję tego narzędzia, np. za sprawą Android SDK Build Tools, które możemy dociągnąć przy
pomocy [Android Studio][23].

Zaletą `apksigner` jest niewątpliwie fakt, że jest on w stanie zweryfikować wszystkie schematy
podpisów używane przy podpisywaniu aplikacji na Androida. W zasadzie, by zweryfikować plik `.apk`
wystarczy wpisać w terminal to poniższe polecenie.

    $ /media/Android/SDK/build-tools/29.0.2/apksigner verify --verbose --print-certs Signal-website-universal-release-*.apk
    Verifies
    Verified using v1 scheme (JAR signing): true
    Verified using v2 scheme (APK Signature Scheme v2): true
    Verified using v3 scheme (APK Signature Scheme v3): true
    Number of signers: 1
    Signer #1 certificate DN: CN=Whisper Systems, OU=Research and Development, O=Whisper Systems, L=Pittsburgh, ST=PA, C=US
    Signer #1 certificate SHA-256 digest: 29f34e5f27f211b424bc5bf9d67162c0eafba2da35af35c16416fc446276ba26
    Signer #1 certificate SHA-1 digest: 45989dc9ad8728c2aa9a82fa55503e34a8879374
    Signer #1 certificate MD5 digest: d90db364e32fa3a7bda4c290fb65e310
    Signer #1 key algorithm: RSA
    Signer #1 key size (bits): 1024
    Signer #1 public key SHA-256 digest: 75336a3cc9edb64202cd77cd4caa6396a9b5fc3c78c58660313c7098ea248a55
    Signer #1 public key SHA-1 digest: b46cbed18d6fbbe42045fdb93f5032c943d80266
    Signer #1 public key MD5 digest: 0f9c33bbd45db0218c86ac378067538d
    WARNING: META-INF/android.support.design_material.version not protected by signature. Unauthorized modifications to this JAR entry will not be detected. Delete or move the entry outside of META-INF/.
    WARNING: META-INF/androidx.activity_activity.version not protected by signature. Unauthorized modifications to this JAR entry will not be detected. Delete or move the entry outside of META-INF/.
    ...

Z powyższych informacji widać, że pakiet został zweryfikowany ( `Verifies` ), zawiera wszystkie
trzy schematy podpisu (każdy z nich z osobna również przechodzi weryfikacje poprawnie). Są też
widoczne dane certyfikatu i klucza publicznego, w tym też jest hash SHA-256 certyfikatu, który
pasuje do tego na stronie Signal. Jest też parę ostrzeżeń dotyczących plików w katalogu
`META-INF/` , które nie są chronione przez sygnatury ale te ostrzeżenia odnoszą się jedynie do
schematu podpisu JAR-signed APK.

### JAR-signed APK i starsze wersje Androida

Czy problemy ze schematem JAR-signed APK wpływają negatywnie na instalowanie aplikacji na starszych
wersjach Androida? Jakby nie patrzeć to starsze Andki są w stanie zweryfikować plik `.apk`
posługując się tylko schematem podpisu JAR-signed APK ale w dalszym ciągu aplikacje mogą być (i
zwykle są) podpisane przy użyciu pozostałych schematów, tak jak widać to było na przykładzie
aplikacji Signal.

Z racji, że w schemacie APK Signature Scheme v2/v3 jest weryfikowany cały plik `.apk` bit po bicie,
to ta pierwsza sygnatura jest weryfikowana przez tą drugą (i trzecią), która gwarantuje
integralność danych całej aplikacji. Jeśli korzystamy ze starszej wersji systemu, to wystarczy przy
pomocy narzędzia `apksigner` ręcznie zweryfikować certyfikat i sygnaturę pliku `.apk` , który
zamierzamy zainstalować w systemie. Oczywiście ten krok nie jest potrzebny w przypadku instalowania
aplikacji z Google Play, bo ten sklep jest w stanie przeprowadzić za nas proces weryfikacji już na
etapie przesyłania do niego aplikacji przez dewelopera.

### Certyfikaty typu self-signed

Jeśli kogoś przeraża fakt, że certyfikat w aplikacji Signal był typu self-signed, czyli został
podpisany przez ten sam podmiot, który wystawił certyfikat, to może odetchnąć z ulgą, bo taki jest
świat Androida. Wszystkie aplikacje dostępne w sklepie Google Play mają tego typu podpisany przez
siebie certyfikat i nie jest to niczym nagannym, czy niewłaściwym. Wszystko sprowadza się do
zabezpieczenia klucza prywatnego, do którego dostęp ma zwykle jedna osoba i na tym kluczu opiera
się cały łańcuch bezpieczeństwa. Jeśli klucz prywatny nie wpadnie w niepowołane łapki, to możemy
być pewni co do aplikacji, która nim została podpisana.

Jeśli ktoś jest ciekaw jakie aplikacje w sklepie Google Play identyfikują się jakimś określonym
certyfikatem, to można ten fakt również ustalić odwiedzając [ten serwis i podając w nim hash SHA-1
certyfikatu][18].

## Czy pobieranie aplikacji z Yalp/Aurora Store jest bezpieczne

By zainstalować appkę ze sklepu Google Play potrzebny nam jest stosowny klient, tj. aplikacja,
która pobierze na flash smartfona plik `.apk` . Z reguły Android ma już na pokładzie aplikację
sklepu Google Play i nie trzeba nic dodatkowo instalować, by móc pobierać za jej pomocą inne appki.
Trzeba jednak wyraźnie zaznaczy tutaj, że aplikacja Google Play to jedynie klient sklepu, coś na
wzór przeglądarki www. Istnieją inne klienty sklepu Google Play, np. otwartoźródłowe alternatywy
takie jak [Yalp store][5] czy [Aurora Store][6], które można wykorzystać do pobrania aplikacji z
tego oficjalnego sklepu Google.

Zarówno Yalp Store jak i Aurora Store umożliwiają instalację tych samych aplikacji, które są
dostępne w sklepie Google Play. Przez pojęcie "ta sama aplikacja" ma się oczywiście rozumieć ten
sam plik `.apk` , który jest dostępny w sklepie Google Play (ten sam podpis cyfrowy i te same sumy
kontrolne). Jeśli ufamy sklepowi Google Play i instalujemy z niego appki tylko dlatego, że w nim
są, to taki sam poziom bezpieczeństwa nam daje Yalp/Aurora Store.

Plusem korzystania z któregoś z tych dwóch alternatywnych klientów sklepu Google Play jest fakt
możliwości zrezygnowania zarówno z oficjalnego klienta sklepu Google, jak i [pozbycie się całego
Google Play Services][7], co może nam nieco poprawić prywatność przy korzystaniu ze smartfona z
Androidem na pokładzie.

W każdym z tych trzech przypadków, pobierany jest plik `.apk` ilekroć tylko chcemy instalować czy
aktualizować jakąś aplikację, którą mamy (bądź chcemy mieć) w systemie. Po pobraniu aplikacji,
Android weryfikuje zarówno jej podpis jak i sumy kontrolne i jeśli wykryje problemy, to nie dopuści
do instalacji czy też aktualizacji takiego pakietu.

Możemy zatem bez problemu instalować aplikacje za sprawą Yalp/Aurora Store i nie powinniśmy się
obawiać, że te klienty wgrają nam same z siebie jakiś syf. A nawet jeśli już komuś takie pomysły
chodzą po głowie, to na obronę Yalp/Aurora Store można rzucić argument, że ich kod jest otwarty, w
przeciwieństwie do kodu aplikacji sklepu Google Play.

## Czy pobieranie aplikacji z F-Droid jest bezpieczne

W przypadku [F-Droid][8] mamy do czynienia z repozytoriami (coś na wzór tych linux'owych), z
których można pobierać aplikacje. Generalnie to repozytoria można podzielić na te oficjalne,
którymi zarządza F-Droid, na oficjalne innych projektów (np. [microg][9]) i nieoficjalne, do
których powinno się podchodzić z rozwagą. [Lista znanych repozytoriów F-Droid][10] jest dostępna
tutaj.

Aplikacje znajdujące się w głównym repozytorium F-Droid można z kolei podzielić na [dwa typy][11].
Pierwszym są aplikacje budowane z konkretnych źródeł, a wynikowe pliki `.apk` są podpisywane przez
klucz repozytorium. W efekcie wiele aplikacji jest podpisanych jednym kluczem. Drugim typem są
aplikacje budowane i podpisywane przez swoich deweloperów. Zwykle będziemy mieć do czynienia z tym
pierwszym typem.

By aplikacja mogła zagościć w oficjalnym repozytorium F-Droid musi być ona z gatunku
[FreeSoftware][12]. W przypadku sklepu Google Play ogromna większość aplikacji ma zamknięty kod i
do tego jest obłożona reklamami. Widać zatem fundamentalne różnice w podejściu do bezpieczeństwa i
prywatności użytkowników między tymi dwoma źródłami aplikacji i niezmiernie ciężko jest do
oficjalnego repozytorium F-Droid przemycić trefne aplikacje w przeciwieństwie do sklepu Google
Play. Dodatkowo, Google usuwa niewygodne aplikacje, dla przykładu [Blokada][13], [Adaway][14] czy
sam F-Droid.

## Czy pobieranie aplikacji z apkmirror.com jest bezpieczne

Jednym z popularniejszych serwisów z plikami `.apk` , które można pobrać i manualnie zainstalować w
Androidzie jest [apkmirror][21]. Użytkownicy cenią go sobie ze względu na te trzy poniższe cechy,
których im sklep Google Play nie może zapewnić.

Pierwszym atutem apkmirror jest możliwość zainstalowania starszych wersji aplikacji, które w
sklepie Google Play przestały być dostępne z racji aktualizacji pakietu. W efekcie jeśli nowsza
wersją aplikacji zaczyna nam zjadać z niewiadomego powodu baterię (albo zachowywać się w inny
nieprzewidziany i do końca poprawny sposób), to przy pomocy apkmirror możemy z tym problemem
sobie poradzić instalując starszą wersję aplikacji. Druga sprawa to nowsze wersje aplikacji, do
których można mieć dostęp w przypadku, gdy sklep Google Play ociąga się z jakiegoś powodu z ich
publikacją. No i ostatnia rzecz, to fakt posiadania przez apkmirror aplikacji nieobecnych w
oficjalnym sklepie Google.

Te wszystkie powyższe ficzery serwisu apkmirror sprawiają, że użytkownik może być skłonny
zainstalować z niego plik `.apk` ale czy możemy być pewni co do tych plików, które tam na stronie
się znajdują? Czy instalując taki plik na pewno nie wgramy sobie syfu do systemu? Odpowiedź może
być tylko jedna -- sprawdźmy co tam znajdziemy.

Weźmy sobie ponownie [aplikację Signal][24] na celownik, z racji, że znamy odcisk palca jej
certyfikatu. Pobieramy dowolny plik `.apk` z listy, która się nam pojawi po kliknięciu w link.
Zanim jednak faktycznie pobierzemy plik, to na stronie mamy informację, że `Verified safe to
install (read more)` , czyli, że aplikacja została zweryfikowana i jest bezpieczna do
zainstalowania. Jeśli klikniemy w ten link, to zobaczymy to poniższe okienko:

![signal-app-verify-apkmirror](/img/2019/12/008-signal-app-verify-apkmirror.png#huge)

Mamy tutaj widoczne hash'e, z których jeden zestaw zawiera odciski palca certyfikatu (niby ten sam
co na stronie projektu Signal). Lepiej jednak nie wierzyć co tam na podejrzanych stronach wypisują,
więc lepiej podejść do tej informacji ostrożnie, przecie każdy mógłby przepisać ten kawałek tekstu
z dowolnego miejsca i wstawić na stronę. Drugi zestaw hash'y odnosi się do sumy kontrolnej samego
pliku, który figuruje w serwisie apkmirror.

Pobierzmy zatem już plik i sprawdźmy czy suma kontrolna pliku się zgadza:

    $  sha256sum org.thoughtcrime.securesms_4.51.6-5792_minAPI19(arm64-v8a)(nodpi)_apkmirror.com.apk
    b0c89cc55915839ab957f9ef7d9077fbde4550013acd2b713e51ad7e534ada2c  org.thoughtcrime.securesms_4.51.6-5792_minAPI19(arm64-v8a)(nodpi)_apkmirror.com.apk

Jak widać, suma kontrolna pliku pasuje do tego co zostało napisane na stronie. Zatem pobraliśmy
plik w formie niezmienionej z serwera apkmirror. Sprawdźmy teraz odcisk palca certyfikatu, który
siedzi w pliku `.apk` :

    $ /media/Android/SDK/build-tools/29.0.2/apksigner verify --verbose --print-certs org.thoughtcrime.securesms_4.51.6-5792_minAPI19(arm64-v8a)(nodpi)_apkmirror.com.apk
    Verifies
    Verified using v1 scheme (JAR signing): true
    Verified using v2 scheme (APK Signature Scheme v2): true
    Verified using v3 scheme (APK Signature Scheme v3): true
    Number of signers: 1
    Signer #1 certificate DN: CN=Whisper Systems, OU=Research and Development, O=Whisper Systems, L=Pittsburgh, ST=PA, C=US
    Signer #1 certificate SHA-256 digest: 29f34e5f27f211b424bc5bf9d67162c0eafba2da35af35c16416fc446276ba26
    Signer #1 certificate SHA-1 digest: 45989dc9ad8728c2aa9a82fa55503e34a8879374
    Signer #1 certificate MD5 digest: d90db364e32fa3a7bda4c290fb65e310
    Signer #1 key algorithm: RSA
    Signer #1 key size (bits): 1024
    Signer #1 public key SHA-256 digest: 75336a3cc9edb64202cd77cd4caa6396a9b5fc3c78c58660313c7098ea248a55
    Signer #1 public key SHA-1 digest: b46cbed18d6fbbe42045fdb93f5032c943d80266
    Signer #1 public key MD5 digest: 0f9c33bbd45db0218c86ac378067538d
    WARNING: META-INF/android.support.design_material.version not protected by signature. Unauthorized modifications to this JAR entry will not be detected. Delete or move the entry outside of META-INF/.
    WARNING: META-INF/androidx.activity_activity.version not protected by signature. Unauthorized modifications to this JAR entry will not be detected. Delete or move the entry outside of META-INF/.
    ...

Jak widać, odcisk palca certyfikatu również się zgadza. Mamy też informację, że dane w pliku `.apk`
nie zostały zmienione w żaden sposób, zatem mamy do czynienia z plikiem, który wyszedł spod młotka
tego samego dewelopera. Nic zatem nie stoi na przeszkodzie by taki plik `.apk` zainstalować w
systemie.

## Aktualizacja aplikacji z niezaufanego źródła

Z racji, że wszystkie aplikacje na Androida (nie tylko te w sklepie Google Play) muszą być
podpisane by zostać zainstalowane w systemie, to nie ma zbytnio dla nas znaczenia skąd taką
aplikację pobierzemy, o ile weryfikacja pliku `.apk` zostanie zakończona powodzeniem.

W przypadku aktualizacji pakietu nie musimy ręcznie go weryfikować. Jeśli certyfikaty będą się
różnić albo dane w pakiecie zostaną w jakiś sposób zmienione, to Android nie dopuści do
aktualizacji takiego pakietu. Mając zainstalowaną w systemie appkę Signal, mogę spróbować wgrać na
telefon plik pobrany z serwisu apkmirror i zobaczyć co się w takim przypadku stanie:

|     |    |     |
| --- | ---| --- |
| ![](/img/2019/12/009-signal-apk-install-apkmirror.png#small) | ![](/img/2019/12/010-signal-apk-install-apkmirror.png#small) |![](/img/2019/12/011-signal-apk-install-apkmirror.png#small) |

Jak widać po fotkach wyżej, paczka `.apk` bez problemu się zainstalowała w systemie dokładnie w
taki sam sposób jakbym aktualizował ją via sklep Google Play (tylko w tym przypadku trzeba
zaznaczyć w opcjach Androida możliwość instalowania aplikacji z nieznanych źródeł). Z racji, że
proces aktualizacji aplikacji przebiegł pomyślnie, to weryfikacja certyfikatu i danych w pakiecie
`.apk` również zakończyła się powodzeniem. Podobnie sprawa będzie wyglądać przy korzystaniu z Yalp
Store czy Aurora Store, bo dla Androida nie ma znaczenia skąd i w jaki sposób pobierany jest plik,
tylko liczą się sumy kontrolne i podpisy cyfrowe.

Warto tutaj wspomnieć, że F-Droid buduje aplikacje bezpośrednio z ich źródeł, przez co wynikowe
pliki `.apk` nie są tymi samymi, które wypuścił deweloper aplikacji. Ponadto, pliki `.apk` w
repozytorium F-Droid są podpisane kluczem repozytorium, a nie kluczem dewelopera, zatem odcisk
palca certyfikatu pliku `.apk` będzie się różnił. Dlatego też możemy zapomnieć o krosowym
aktualizowaniu aplikacji (F-Droid <-> Google Play).

By zaktualizować aplikację, która została podpisana innym kluczem niż ta obecna już w systemie
naszego smartfona, trzeba ją pierw odinstalować (ewentualnie wgrać [dodatek do Xposed][25]). Chodzi
tutaj generalnie o sprawy związane z prywatnością i bezpieczeństwem. Każda aplikacja w systemie ma
przypisany inny ID użytkownika i tylko ta aplikacja ma dostęp do swoich plików. Gdyby było możliwie
zaktualizowanie aplikacji, która identyfikuje się innym certyfikatem, to potencjalnie inna appka
mogłaby uzyskać dostęp do danych tej konkretnej aplikacji (w tym też przejąc jej uprawnienia, np.
do mikrofonu czy kamery), a jak wiadomo, usunięcie aplikacji z systemu czyści jej wszystkie
prywatne pliki.

## Instalacja aplikacji z niezaufanego źródła

O ile proces aktualizacji jest bardzo prosty i automatyczny pod względem weryfikacji wszystkich
tych niezbędnych z punktu widzenia bezpieczeństwa rzeczy, to w przypadku, gdy zamierzamy instalować
po raz pierwszy aplikację z innego źródła niż oficjalny sklep Google Play, to niestety trzeba
manualnie (najlepiej przy pomocy `apksigner` ) zweryfikować plik, który zamierzamy zainstalować w
naszym smartfonie z Androidem. Dopiero po pomyślnym zweryfikowaniu pliku `.apk` możemy go
zainstalować bez obaw.

### A jeśli aplikacja nie publikuje odcisku palca certyfikatu

W przypadku aplikacji Signal wszystko było podane praktycznie na tacy i nie wymagało od użytkownika
większego wysiłku intelektualnego, by taki plik `.apk` zweryfikować. Niemniej jednak, ogromna
większość aplikacji niebędących spod znaku FreeSoftware nie posiada nawet swojej strony www.
Jedyne informacje o takiej appcę są dostępne w sklepie Google Play. Przykładem mogą być aplikacje
banków, czy też innych instytucji finansowych. Na stronie takiego banku jest z reguły tylko
wzmianka o tym, że mają oni aplikację oraz jest podany też link do sklepu Google Play skąd ją można
pobrać. Jak w takim przypadku zweryfikować plik `.apk` , który chcielibyśmy zainstalować w swoim
Androidzie pozbawionym choćby tego całego Google Play Services?

Jedyną opcją w takiej sytuacji jest pozyskanie wiarygodnego pliku `.apk` ze sklepu Google Play i
poddanie go weryfikacji przy pomocy narzędzia `apksigner` . Jeśli nie ufamy Yalp/Aurora Store, to
taki plik można też wydobyć z Androida po zainstalowaniu aplikacji przy pomocy tego oficjalnego
klienta sklepu Google Play. By wydobyć plik `.apk` ze smartfona, wystarczy zainstalować sobie
jakieś menadżer plików, np. [Amaze][26], który posiada wbudowany menadżer aplikacji. W tym
menadżerze wystarczy wykonać backup aplikacji i stosowny plik `.apk` zostanie zapisany np. na kacie
SD.

![amaze-file-manager-app-backup](/img/2019/12/012-amaze-file-manager-app-backup.png#small)

Tak wykonany backup aplikacji to nic innego jak tylko skopiowanie pliku `.apk` z jednego miejsca w
drugie. Można ten fakt potwierdzić porównując hash pliku zainstalowanej aplikacji z plikiem backupu:

    $ sha256sum Signal_org.thoughtcrime.securesms*
    b0c89cc55915839ab957f9ef7d9077fbde4550013acd2b713e51ad7e534ada2c  Signal_org.thoughtcrime.securesms-backup.apk
    b0c89cc55915839ab957f9ef7d9077fbde4550013acd2b713e51ad7e534ada2c  Signal_org.thoughtcrime.securesms.apk

Jak widać sumy kontrolne obu plików się zgadzają. Zakładając, że nie dysponujemy żadnym hash'em
certyfikatu, to mając wiarygodny plik można go poddać weryfikacji w `apksigner` i w ten sposób
uzyskać dane o certyfikacie, które mogą posłużyć później do weryfikacji certyfikatu plików `.apk`
aplikacji pobieranych z innych niezaufanych źródeł.

## Podsumowanie

Jak widać na przedstawionym przykładzie aplikacji Signal, nie ma znaczenia skąd pobieramy pliki
`.apk` do momentu, gdy jesteśmy w stanie zweryfikować/potwierdzić certyfikat i integralność danych
w pakiecie. W przeszłości mechanizm weryfikacji plików `.apk` w Androidzie nie gwarantował tego co
powinien (choć nie była to wina samego mechanizmu) ale ten fakt uległ zmianie po zastąpieniu
schematu podpisu JAR-signed APK przez APK Signature Scheme v2/v3.

Nie ma co zatem popadać w paranoję i zakładać, że tylko aplikacje pobrane z oficjalnego sklepu
Google Play są wiarygodne i bezpieczne, bo gdy instalujemy aplikację w Androidzie bez jej
weryfikacji, to zaciągamy dług wobec naszej prywatności. Z każdą kolejną bezrefleksyjnie
zainstalowaną appką z tego sklepu coraz bardziej zapominamy jak ważny jest proces weryfikacji
pochodzenia oprogramowania, od którego nasze życie coraz bardziej zaczyna zależeć. Jeśli dalej
będziemy szli tą drogą, to prędzej czy później ten dług trzeba będzie spłacić i nie będziemy z tego
faktu zbytnio zadowoleni.


[1]: https://signal.org/android/apk/
[2]: https://source.android.com/security/apksigning/
[3]: https://docs.oracle.com/javase/8/docs/technotes/guides/jar/jar.html#Signed_JAR_File
[4]: https://source.android.com/security/apksigning/v2
[5]: https://github.com/yeriomin/YalpStore
[6]: https://gitlab.com/AuroraOSS/AuroraStore
[7]: /post/czy-smartfon-z-androidem-bez-google-apps-services-ma-sens/
[8]: https://f-droid.org/
[9]: https://microg.org/fdroid.html
[10]: https://forum.f-droid.org/t/known-repositories/721
[11]: https://f-droid.org/en/docs/FAQ_-_General/#whats-the-difference-between-source-and-binary-builds
[12]: https://www.gnu.org/philosophy/free-sw.html
[13]: https://blokada.org/
[14]: https://adaway.org/
[15]: https://docs.oracle.com/javase/tutorial/deployment/jar/intro.html
[16]: https://securitypad.blogspot.com/2014/12/jar-signature-block-file-format.html
[17]: https://stackoverflow.com/questions/54782328/apksigner-does-not-verify-signature
[18]: https://androidobservatory.org/cert/45989DC9AD8728C2AA9A82FA55503E34A8879374
[19]: https://www.xda-developers.com/janus-vulnerability-android-apps/
[20]: https://www.xda-developers.com/apk-signature-scheme-v3-key-rotation/
[21]: https://www.apkmirror.com/
[22]: https://developer.android.com/studio/command-line/apksigner
[23]: https://developer.android.com/studio
[24]: https://www.apkmirror.com/apk/signal-foundation/
[25]: https://www.xda-developers.com/application-signature-verification-how-it-works-how-to-disable-it-with-xposed-and-why-you-shouldnt/
[26]: https://github.com/TeamAmaze/AmazeFileManager
