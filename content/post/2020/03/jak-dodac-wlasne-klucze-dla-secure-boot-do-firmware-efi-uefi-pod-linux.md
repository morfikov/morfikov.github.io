---
author: Morfik
categories:
- Linux
date: "2020-03-16T19:15:00Z"
lastmod: 2020-07-30 20:45:00 +0100
published: true
status: publish
tags:
- debian
- efi
- uefi
- secure-boot
- shim
- refind
- lenovo
- thinkpad
- t430
GHissueID: 25
title: Jak dodać własne klucze dla Secure Boot do firmware EFI/UEFI pod linux
---

W środowiskach linux'owych Secure Boot nie jest zbyt mile widziany i raczej postrzegany jako
źródło wszelkiego zła na świecie. Sporo użytkowników linux'a dopatruje się w Secure Boot zamachu na
wolność decydowania o swoim sprzęcie, który ma być serwowany przez Microsoft. Niemniej jednak,
odsądzanie tego mechanizmu od czci i wiary jest moim zdaniem lekko nie na miejscu. Niewielu
użytkowników linux'a zdaje sobie sprawę, że od prawie dekady istnieje taki wynalazek jak [shim][1]
(no i jest też [PreLoader][2]), który umożliwia dystrybucjom linux'a (jak i ich końcowym
użytkownikom) zaimplementowanie Secure Boot we własnym zakresie. Niemniej jednak, można całkowicie
się obejść bez tych dodatków usuwając wbudowane w firmware EFI/UEFI certyfikaty i dodając na ich
miejsce własnoręcznie wygenerowane klucze kryptograficzne. Takie podejście sprawia, że kod
podpisany tylko i wyłącznie przez nas będzie mógł zostać uruchomiony przez firmware komputera w
trybie Secure Boot, co może ździebko poprawić bezpieczeństwo naszego systemu. Poniższy artykuł ma
na celu pokazać w jaki sposób zastąpić wbudowane w firmware EFI/UEFI klucze swoimi własnymi przy
wykorzystaniu dystrybucji Debiana.

<!--more-->
## Opis działania Secure Boot w linux na przykładzie shim

By zrozumieć w jaki sposób Secure Boot jest w stanie poprawić bezpieczeństwo systemu, trzeba
najpierw zaznajomić się z procedurą startu komputera w trybie EFI/UEFI. Można spotkać się z różnymi
rozwiązaniami ale chyba najbardziej uniwersalnym i popularnym zarazem jest shim. W zasadzie te
główniejsze dystrybucje linux'a, jak Ubuntu i Debian, korzystają właśnie z shim'a. Postanowiłem
zatem przyjrzeć się nieco bliżej jak wygląda [start linux'a w trybie EFI/UEFI z wykorzystaniem
shim][3], tak by go dokładnie zrozumieć i w miarę przyzwoicie opisać.

Shim to bardzo proste oprogramowanie zaprojektowane przez deweloperów różnych dystrybucji linux'a,
by działało ono jako bootloader pierwszego poziomu na systemach EFI/UEFI. Kod shim jest
otwartoźródłowy, prosty i przy tym dobrze zrozumiały. Został on też poddany audytom bezpieczeństwa
i kompleksowo przebadany. Można mu zatem zaufać, że będzie działał w taki sposób jak się od niego
oczekuje. Dlatego też Microsoft może taki kawałek oprogramowania podpisać swoim kluczem prywatnym.
Dlaczego właśnie MS, a nie inny podmiot? Wszystko przez popularność windows'a na desktopach i
laptopach. Z racji, że ten system jest w dość powszechnym wykorzystaniu na domowych komputerach, to
producenci sprzętu zaszywają domyślnie certyfikat Microsoft'u bezpośrednio w firmware EFI/UEFI
maszyny. Ten certyfikat jest w stanie zweryfikować podpis złożony prywatnym kluczem MS. W taki
sposób, Microsoft może skupić się jedynie na podpisaniu shim'a i nie musi przy tym martwić o tę
całą masę oprogramowania, z którego my, jako docelowi użytkownicy komputera, zamierzamy korzystać.
Wykorzystanie shim'a mogłoby być opcjonalne, gdyby tylko producenci sprzętu podeszli należycie do
zakodowania kluczy różnych dystrybucji linux'a w firmware EFI/UEFI, a z tym różnie bywa i zwykle
stosownych certyfikatów w firmware brakuje i dlatego są takie problemy z mechanizmem Secure Boot w
przypadku linux'a. Dlatego też właśnie powstał shim, który z racji podpisania kluczem Microsoft'u
rozwiązuje cały zaistniały problem z Secure Boot na linux.

Shim używany jest głównie do dostarczenia klucza publicznego CA danej dystrybucji linux'a, który to
z kolei jest używany do podpisywania innych programów, np. kernela czy bootloader'a takiego jak
Grub2. Pełna lista [programów podpisywanych przez Debiana][4] znajduje się tutaj. W taki sposób
ciężar i obowiązek podpisania stosownych binarek spoczywa na dystrybucji linux'a jeśli ma ona
zamiar wspierać mechanizm Secure Boot. Dodatkowo, w ramach poprawy bezpieczeństwa, [binarki shim
dostępne w repozytorium Debiana są w pełni reprodukowalne][5].

Klucz CA dystrybucji jest zaszyty w shim na stałe i nie możemy go ruszyć. Istnieje też druga baza
danych kluczy, która może być zarządzana przez użytkownika, zwana Machine Owner Key (MOK). Klucze w
tej bazie mogą być dodawane i usuwane przez użytkownika z poziomu działającego linux'a za pomocą
narzędzia `mokutil` . Niemniej jednak, każdy klucz trzeba będzie potwierdzić w konsoli na wczesnym
etapie startu systemu, co zapobiega dodaniu podstawionych kluczy przez malware przestrzeni
użytkownika, który potencjalnie byłby w stanie obejść mechanizm Secure Boot. Zarządzanie tymi
kluczami jest niezależnie od głównego klucza CA dystrybucji linux'a.

Gdy maszyna z linux się uruchamia, to zaraz na początku fazy boot, firmware EFI/UEFI ładuje do
pamięci RAM binarkę shim, która została określona w zmiennej BootEntry w tym firmware (np. za
sprawą `efibootmgr` ). Podczas tradycyjnej instalacji linux'a, stosowny wpis powinien być
automatycznie dodany i aktualizowany ilekroć tylko będziemy instalować aktualizacje dla
bootloader'a (Grub2). Gdy przychodzi moment weryfikacji przez firmware podpisu złożonego pod shim,
to ta binarka jest zatwierdzana z racji pomyślnego zweryfikowania podpisu (jest podpisana kluczem
MS). Jako, że shim posiada również zaszyty certyfikat jakiejś dystrybucji (w tym przypadku Debiana),
jak i bazę danych MOK, to kolejne binarki mogą zostać zweryfikowane tymi dodatkowymi certyfikatami.
Następnie shim ładuje obraz drugiego poziomu (second-stage image), którym może być bootloader
(zwykle Grub2) lub MokManager jeśli zarządzanie kluczami jest z jakiegoś powodu wymagane.

W przypadku normalnego rozruchu, binarka bootloader'a ( `grub*.efi` ) jest ładowana do pamięci
operacyjnej i weryfikowana względem zaufanych certyfikatów. Jako, że Grub2 jest podpisany przez
klucz dystrybucji, to powinien zostać z powodzeniem zweryfikowany i załadowany, a proces startu
powinien być kontynuowany.

W przypadku, gdy zostanie zainicjowany proces zarządzania kluczami, to binarka MokManager
( `mm*.efi` ) zostanie załadowana do pamięci RAM. Ta binarka jest zaufana przez shim za sprawą
klucza efemerycznego (tymczasowego), który istnieje jedynie w czasie procesu kompilacji shim'a.
Oznacza to, że dana binarka MokManager'a może zostać zweryfikowana jedynie przez odpowiadający jej
shim i żaden inny shim nie będzie w stanie tej binarki zweryfikować, co zapobiega całej masie
problemów natury bezpieczeństwa.

MokManager pozwala każdemu użytkownikowi komputera mającemu dostęp do konsoli systemowej na
przeprowadzenie poniższych akcji:

- dodanie nowych kluczy do bazy
- usunięcie zaufanych kluczy z bazy
- dodanie binarnych hash'y
- przełączenie weryfikacji Secure Boot na poziomie shim

Te powyższe czynności z reguły będą dostępne jedynie po wprowadzeniu uprzednio ustawionego hasła (w
działającym systemie operacyjnym), tak by zweryfikować tożsamość osoby, która wydaje polecenia.
Takie hasła są tworzone na czas pojedynczego uruchomienia shim'a lub MokManager'a i są czyszczone
jak tylko proces dobiegnie końca lub zostanie anulowany. Gdy proces zarządzania kluczami zakończy
się, komputer restartuje i kontynuuje fazę startu systemu operacyjnego.

Gdy Grub2 zostanie uruchomiony, ładuje on sobie całą potrzebną konfigurację (zwykle z partycji ESP),
która wskazuje na kolejny plik konfiguracyjny zlokalizowany tym razem na na partycji BOOT (lub też
w obrębie głównego systemu plików). Z kolei ten plik konfiguracyjny wskazuje położenie kernela i
obrazu initrd, który trzeba będzie załadować do pamięci RAM w celu kontynuowania procesu boot.

Obraz kernela, który zostanie załadowany, również musi zostać zweryfikowany, a wcześniej też i
podpisany zaufanym kluczem, który przechowywany jest w bazie danych MOK. Oficjalne kernele Debiana
są podpisane kluczem dystrybucji i bez problemu powinny zostać zweryfikowane. Jak tylko ten proces
zakończy się powodzeniem, to kontrola nad dalszym startem maszyny jest przekazywana kernelowi. Co
ciekawe obraz initrd/initramfs nie jest weryfikowany w żaden sposób.

Gdy Secure Boot jest włączony, to każdy błąd przy weryfikacji ładowanych binarek do tego momentu
będzie skutkował przerwaniem procesu startu maszyny. W przypadku, gdy błędów nie będzie, załadowany
w pamięci operacyjnej kernel wyłączy usługi startu (Boot Services) w firmware EFI/UEFI zrzucając w
ten sposób uprawnienia i efektywnie przełączy się w tryb użytkownika (user mode), gdzie dostęp do
zaufanych zmiennych firmware EFI/UEFI jest ograniczony jedynie do odczytu.

Jako, że moduły kernela mogą posiadać rozbudowane uprawnienia, to każdy taki moduł niewbudowany
bezpośrednio w kernel musi zostać podpisany osobno zaufanym kluczem, w przeciwnym razie system nie
pozwoli go załadować. Wszystkie moduły, które są dostarczane z dystrybucyjnym kernelem Debiana są
podpisane i zaufane. Pozostałe moduły, np. budowane za sprawą mechanizmu DKMS, muszą być podpisane
przez użytkownika osobno. Próba załadowania niepodpisanego modułu zawsze zakończy się błędem, który
zostanie odnotowany w logu systemowym.

## Klucze wbudowane w firmware EFI/UEFI

[Na wiki Gentoo][14] dostępny jest przyzwoicie opisany tutorial, którego części postanowiłem
przetłumaczyć w drodze konfigurowania swojego laptopa i umieścić w niniejszym artykule. Wszystkie
rzeczy, które wymagały wyjaśnienia lub dopowiedzenia, zostały wyjaśnione i dopowiedziane, a
brakujące informacje zostały dodane i wkomponowane między wersy, tak by poniższy artykuł był nieco
pełniejszy niż ten podlinkowany tutek i zawierał wszystkie niezbędne rzeczy, których człowiek może
potrzebować przy zaoraniu swojego firmware EFI/UEFI na potrzeby bezpiecznej instalacji Debian linux.

Mając rozeznanie jak mniej więcej działa Secure Boot, można zauważyć, że pewne zagrożenie może
płynąć z racji zaszycia określonych certyfikatów bezpośrednio w firmware EFI/UEFI. W efekcie każde
oprogramowanie podpisane przez klucze prywatne, np. Microsoft'u czy Lenovo, może zostać uruchomione
na naszej maszynie i potencjalnie skompromitować zabezpieczenia jakie daje mechanizm Secure Boot. W
przypadku mojego ThinkPad'a T430 wkodowane na stałe były te poniższe certyfikaty.

Zmienna `PK` :

    # efi-readvar -v PK
    Variable PK, length 983
    PK: List 0, type X509
        Signature 0, size 955, owner 3cc24e96-22c7-41d8-8863-8e39dcdcc2cf
            Subject:
                C=JP, ST=Kanagawa, L=Yokohama, O=Lenovo Ltd., CN=Lenovo Ltd. PK CA 2012
            Issuer:
                C=JP, ST=Kanagawa, L=Yokohama, O=Lenovo Ltd., CN=Lenovo Ltd. PK CA 2012

Zmienna `KEK` :

    # efi-readvar -v KEK
    Variable KEK, length 2545
    KEK: List 0, type X509
        Signature 0, size 957, owner 7facc7b6-127f-4e9c-9c5d-080f98994345
            Subject:
                C=JP, ST=Kanagawa, L=Yokohama, O=Lenovo Ltd., CN=Lenovo Ltd. KEK CA 2012
            Issuer:
                C=JP, ST=Kanagawa, L=Yokohama, O=Lenovo Ltd., CN=Lenovo Ltd. KEK CA 2012
    KEK: List 1, type X509
        Signature 0, size 1532, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Subject:
                C=US, ST=Washington, L=Redmond, O=Microsoft Corporation, CN=Microsoft Corporation KEK CA 2011
            Issuer:
                C=US, ST=Washington, L=Redmond, O=Microsoft Corporation, CN=Microsoft Corporation Third Party Marketplace Root

Zmienna `db` :

    # efi-readvar -v db
    Variable db, length 4209
    db: List 0, type SHA256
        Signature 0, size 48, owner 7facc7b6-127f-4e9c-9c5d-080f98994345
            Hash:14e62a4905e19189e70828983165939afc0a331d0b415f3332b0e818a827f436
    db: List 1, type X509
        Signature 0, size 1515, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Subject:
                C=US, ST=Washington, L=Redmond, O=Microsoft Corporation, CN=Microsoft Windows Production PCA 2011
            Issuer:
                C=US, ST=Washington, L=Redmond, O=Microsoft Corporation, CN=Microsoft Root Certificate Authority 2010
    db: List 2, type X509
        Signature 0, size 1572, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Subject:
                C=US, ST=Washington, L=Redmond, O=Microsoft Corporation, CN=Microsoft Corporation UEFI CA 2011
            Issuer:
                C=US, ST=Washington, L=Redmond, O=Microsoft Corporation, CN=Microsoft Corporation Third Party Marketplace Root
    db: List 3, type X509
        Signature 0, size 962, owner 7facc7b6-127f-4e9c-9c5d-080f98994345
            Subject:
                C=JP, ST=Kanagawa, L=Yokohama, O=Lenovo Ltd., CN=ThinkPad Product CA 2012
            Issuer:
                C=JP, ST=Kanagawa, L=Yokohama, O=Lenovo Ltd., CN=Lenovo Ltd. Root CA 2012

Zmienna `dbx` :

    # efi-readvar -v dbx
    Variable dbx, length 3800
    dbx: List 0, type SHA256
        Signature 0, size 48, owner 7facc7b6-127f-4e9c-9c5d-080f98994345
            Hash:14e62a4905e19189e70828983165939afc0a331d0b415f3332b0e818a827f436
    dbx: List 1, type SHA256
        Signature 0, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:80b4d96931bf0d02fd91a61e19d14f1da452e66db2408ca8604d411f92659f0a
        ...
        Signature 76, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:45c7c8ae750acfbb48fc37527d6412dd644daed8913ccd8a24c94d856967df8e

Zmienna `MokList` :

    # efi-readvar -v MokList
    Variable MokList has no entries

### Zmienne PK, KEK, db, dbx i MokList

Specyfikacja EFI/UEFI definiuje 4 bezpieczne nieulotne zmienne (secure, non-volatile variables),
które są wykorzystywane do kontrolowania podsystemu Secure Boot. Są to `PK` (Platform Key) , `KEK`
(Key Exchange Key) , `db` (Signature Database) i `dbx` (Forbidden Signatures Database). Określenie
tych zmiennych mianem bezpiecznych bierze się z faktu, że zmiany w nich mogą zostać przeprowadzone
jedynie przez osobę władającą prywatną częścią klucza, który został użyty do utworzenia tych
zmiennych.

W zmiennej `PK` jest przechowywany główny klucz platformy i jego zadaniem jest kontrola dostępu do
tej zmiennej jak i również do zmiennej `KEK` . W większości implementacji firmware EFI/UEFI w
zmiennej `PK` może być przechowywany tylko jeden taki klucz (zwykle [X.509 używający schematu RSA
2048-bit][29]). Gdy zmienna `PK` jest czyszczona (czynność do przeprowadzenia w konfiguracji
firmware EFI/UEFI), system przechodzi w tryb ustawień (setup mode) i jednocześnie Secure Mode
zostaje wyłączony. W trybie ustawień, zmienne `PK` , `KEK` , `db` oraz `dbx` mogą zostać
zaktualizowane bez jakiegokolwiek uwierzytelnienia całego procesu. Niemniej jednak, natychmiast
po zapisaniu zmiennej `PK` , system przechodzi w tryb użytkownika (user mode). Dlatego też zmienna
PK powinna być aktualizowana zawsze jako ostatnia. Klucz `PK` nie może być użyty do podpisania
plików wykonywalnych.

W zmiennej `KEK` przechowywane są klucze (X509 lub RSA2048) wykorzystywane do nadzorowania procesu
aktualizacji bazy danych sygnatur przechowywanej w zmiennej `db` . Klucze `KEK` mogą także zostać
użyte do podpisywania plików wykonywalnych, choć jest to rzadkie ze względu na fakt, że do tego
celu wykorzystywane są klucze, których certyfikaty figurują w zmiennej `db` .

Baza danych sygnatur przechowywana w zmiennej `db` jest wykorzystywana do weryfikacji podpisanych
plików wykonywalnych w środowisku firmware EFI/UEFI przy włączonym Secure Boot. Może ona zawierać
dowolny miks kluczy publicznych, sygnatur i hash'y. Ta zmienna funkcjonuje w zasadzie jako biała
lista plików wykonywalnych podczas początkowej fazy startu maszyny. Z kolei baza zabronionych
sygnatur przechowywana jest w zmiennej `dbx` i jest wykorzystywana do uniemożliwienia załadowania
binarek, które z jakiegoś powodu zostały określone jako niezaufane. Zmienna `dbx` robi w ten sposób
za czarną listę.

Poza tymi czterema głównymi zmiennymi można było jeszcze zauważyć zmienną `MokList` . Jest ona
bardzo podobna do zmiennej `db` i zwykle jest pusta ale przy pomocy narzędzia `KeyTool` można do
niej dodawać/usuwać dowolne klucze, zwłaszcza klucze różnych dystrybucji linux'a, tak by mogły one
współpracować z Secure Boot, prawdopodobnie bez potrzeby zaciągania do tego shim'a. Póki co jednak,
nie udało mi się nic do tej zmiennej dodać, więc dokładnie nie wiem jakie jest jej przeznaczenie.

Gdy system jest uruchomiony w trybie użytkownika oraz Secure Boot jest włączony, to [firmware
EFI/UEFI będzie w stanie uruchomić][28] jedynie tylko te pliki wykonywalne, które:

- nie są podpisane ale mają hash w `db` i równocześnie nie mają go w `dbx`
- są podpisane i ich sygnatura występuje w `db` i nie ma jej przy tym w `dbx`
- są podpisane i ich sygnatura jest weryfikowalna za sprawą klucza publicznego obecnego w `db` lub
   klucza publicznego przechowywanego w `KEK` i zarazem ani klucz ani sygnatura nie występują w
   `dbx` .

Po nabyciu nowego komputera mającego windows'a na pokładzie, te cztery zmienne będą skonfigurowane
zwykle w poniższy sposób:

- Zmienna `PK` będzie miała załadowany klucz publiczny wystawiony przez producenta sprzętu, np.
   Lenovo.
- Zmienna `KEK` będzie miała załadowany klucz publiczny wystawiony przez Microsoft.
- Zmienna `db` będzie miała załadowany zestaw kluczy publicznych wystawionych przez szereg
   producentów autoryzowanych przez Microsoft.
- Zmienna `dbx` będzie zwykle zawierać cofnięte sygnatury lub hash'e (może też być pusta).

## Usuwanie wbudowanych kluczy i dodanie własnych

W przypadku takich osób jak ja, które nie zamierzają instalować na swoim laptopie windows'a, te
wbudowane klucze są niezbyt mile widziane patrząc z punktu widzenia bezpieczeństwa. Dodatkowo, mogą
one  powodować problemy z Secure Boot w przypadku instalacji linux'a. Część implementacji EFI/UEFI
umożliwia użytkownikowi komputera usunięcie wszystkich domyślnych kluczy. W ten sposób zostaje
otwarta droga do wdrożenia Secure Boot, nad którym to my będziemy mieć pełnię władzy z racji, że
będziemy decydować jakie oprogramowanie będzie się w stanie uruchomić na naszej maszynie.
Odbierzemy jednocześnie tę możliwość decydowania podmiotom takim jak Microsoft czy Lenovo.

Teoretycznie przy zaprzęgnięciu shim'a, Secure Boot powinien działać OOTB. Niemniej jednak, w
przypadku mojego laptopa tak się nie stało. Być może jest on za stary i dlatego ma problem z
odpaleniem Debiana w trybie Secure Boot na domyślnym zestawie kluczy wgranych w firmware EFI/UEFI.
Tak czy inaczej, postanowiłem zaorać wszystkie te klucze i wgrać tam swoje własne, tak by sprawdzić
czy to coś pomoże.

### Backup stock'owych kluczy firmware EFI/UEFI

Jako, że zamierzamy wyczyścić stock'owe klucze w firmware EFI/UEFI, to wypadałoby zacząć od
zrobienia ich backup'u. Technicznie rzecz biorąc, to w konfiguracji tego firmware powinny być
stosowne opcje do odtworzenia tych kluczy (i w przypadku mojego laptopa są) ale lepiej jest mieć
własny backup, tak na wszelki wypadek. Musimy zatem zrobić zrzut zawartości zmiennych `PK` ,
`KEK` , `db` i `dbx` . Kopię zapasową robimy przy pomocy narzędzia `efi-readvar` dostępnego w
pakiecie `efitools` :

    # mkdir -p /etc/kernel_key/efikeys.orig/
    # chmod 700 /etc/kernel_key/efikeys.orig/
    # cd /etc/kernel_key/efikeys.orig/

    # efi-readvar -v PK -o old_PK.esl
    Variable PK, length 983

    # efi-readvar -v KEK -o old_KEK.esl
    Variable KEK, length 2545

    # efi-readvar -v db -o old_db.esl
    Variable db, length 4209

    # efi-readvar -v dbx -o old_dbx.esl
    Variable dbx, length 3800

Warto tutaj zaznaczyć, że backup musi mieć format listy sygnatur czytelny dla maszyn i dlatego
określony został przełącznik `-o` . Bez niego byśmy dostali tylko tekstową reprezentację, której
nie bylibyśmy w stanie przywrócić w późniejszym czasie, gdyby zaszła taka potrzeba.

### Tworzenie nowych kluczy

Mając zrobiony backup kluczy firmware EFI/UEFI, tworzymy nowe klucze. W późniejszym czasie wgramy
te nasze klucze w miejsce tych, które zostaną usunięte. Rozmiar klucza można podać naturalnie 4096
ale niektóre implementacje EFI/UEFI nie akceptują większych kluczy niż 2048 bitów. Dlatego lepiej
pozostać przy standardowym rozmiarze. Klucze generujemy zaś przy pomocy `openssl` w poniższy sposób:

    # cd /etc/kernel_key/

    # openssl req -new -x509 -newkey rsa:2048 -subj "/CN=morfikov's platform key/" -keyout PK.key -out PK.crt -days 3650 -nodes -sha256
    # openssl req -new -x509 -newkey rsa:2048 -subj "/CN=morfikov's key-exchange-key/" -keyout KEK.key -out KEK.crt -days 3650 -nodes -sha256
    # openssl req -new -x509 -newkey rsa:2048 -subj "/CN=morfikov's kernel-signing key/" -keyout db.key -out db.crt -days 3650 -nodes -sha256

    # chmod -v 400 *.key
    mode of 'KEK.key' changed from 0600 (rw-------) to 0400 (r--------)
    mode of 'PK.key' changed from 0600 (rw-------) to 0400 (r--------)
    mode of 'db.key' changed from 0600 (rw-------) to 0400 (r--------)

Firmware EFI/UEFI różnych producentów sprzętu może mieć nieco inne wymagania co do formatów plików,
którymi będziemy chcieli się posłużyć w celu zaktualizowania zawartości zmiennych. Dlatego też
przydałoby się wygenerować kilka dodatkowych plików, tak na wszelki wypadek, gdyby w naszym
przypadku to one jednak znalazły zastosowanie.

Na początek tworzymy wersję formatu `.auth` (signed signature list) zmiennej `PK` , jako że
`efi-updatevar` (jak i sporo graficznych interfejsów firmware EFI/UEFI) akceptuje tylko ten format.
Pierwsze z poniższych poleceń tworzy listę sygnatur, która wymaga unikalnego ID, choć nie jest on
dla nas tak istotny. W drugim poleceniu zaś używamy naszego prywatnego klucza `PK` (platform key),
by tę listę sygnatur podpisać (zarówno `cert-to-efi-sig-list` jak i `sign-efi-sig-list` są dostępne
w pakiecie `efitools` ):

    # cert-to-efi-sig-list -g "$(uuidgen)" PK.crt PK.esl
    # sign-efi-sig-list -k PK.key -c PK.crt PK PK.esl PK.auth
    Timestamp is 2020-3-15 15:05:52
    Authentication Payload size 893
    Signature of size 1211
    Signature at: 40

Podobnie postępujemy w przypadku klucza `KEK` i tutaj do podpisywania również trzeba wykorzystać
prywatny klucz `PK` . Przełącznik `-a` z kolei wskazuje, że dane będą dołączane do zmiennej zamiast
zastępować je:

    # cert-to-efi-sig-list -g "$(uuidgen)" KEK.crt KEK.esl
    # sign-efi-sig-list -a -k PK.key -c PK.crt KEK KEK.esl KEK.auth
    Timestamp is 0-0-0 00:00:00
    Authentication Payload size 903
    Signature of size 1211
    Signature at: 40

Podobnie postępujemy w przypadku `db` ale tutaj wykorzystujemy do podpisu prywatny klucz `KEK` .
Parametr `-a` również trzeba określić:

    # cert-to-efi-sig-list -g "$(uuidgen)" db.crt db.esl
    # sign-efi-sig-list -a -k KEK.key -c KEK.crt db db.esl db.auth
    Timestamp is 0-0-0 00:00:00
    Authentication Payload size 905
    Signature of size 1223
    Signature at: 40

Podobnie postępujemy dla `dbx` ale tutaj skorzystamy z pliku backupu, który utworzyliśmy wcześniej.
W tym przypadku do podpisu również jest wykorzystywany klucz `KEK` :

    # cp efikeys.orig/old_dbx.esl ./
    # sign-efi-sig-list -k KEK.key -c KEK.crt dbx old_dbx.esl old_dbx.auth
    Timestamp is 2020-3-15 15:34:41
    Authentication Payload size 3842
    Signature of size 1223
    Signature at: 40

Tworzymy teraz wersję DER dla każdego z trzech naszych kluczy publicznych:

    # openssl x509 -outform DER -in PK.crt -out PK.cer
    # openssl x509 -outform DER -in KEK.crt -out KEK.cer
    # openssl x509 -outform DER -in db.crt -out db.cer

#### Wsparcie dla windows

Stwórzmy jeszcze zespolony plik `.esl` dla `KEK` i `db` (pliki `.esl` można zwyczajnie połączyć). W
tym kroku chodzi o zachowanie wsparcia dla windows'a przez połączenie starych bazy danych (tych z
backup'u) z nowymi. W przypadku tak utworzonych plików również będziemy potrzebować odpowiedników
`.auth` :

    # cp efikeys.orig/old_KEK.esl ./
    # cp efikeys.orig/old_db.esl ./
    # cat old_KEK.esl KEK.esl > compound_KEK.esl
    # cat old_db.esl db.esl > compound_db.esl

    # sign-efi-sig-list -k PK.key -c PK.crt KEK compound_KEK.esl compound_KEK.auth
    Timestamp is 2020-3-15 15:40:50
    Authentication Payload size 3448
    Signature of size 1211
    Signature at: 40

    # sign-efi-sig-list -k KEK.key -c KEK.crt db compound_db.esl compound_db.auth
    Timestamp is 2020-3-15 15:40:54
    Authentication Payload size 5114
    Signature of size 1223
    Signature at: 40

#### Brak wsparcia dla windows

W przypadku, gdy nie chcemy zachowywać wsparcia dla windows, to można pozbyć się kluczy Microsoft'u
(i innych podmiotów), które znajdowały się w zmiennych `KEK` i `db` ). Usunięcie tych kluczy sprawi,
że żaden windows zainstalowany na naszej maszynie nie będzie w stanie się uruchomić z włączonym
Secure Boot. Niemniej jednak, wsparcie dla windows będzie można zaimplementować w późniejszym
czasie dodając klucze Microsoft'u do bazy.

By usunąć klucze Microsoft'u, musimy stworzyć pliki zespolone dla `KEK` i `db` (zarówno `.esl` jak
i `.auth` ) z pominięciem komponentów Microsoft'u (bez dodawania zawartości z backup'u):

    # cp KEK.esl compound_KEK.esl
    # cp db.esl compound_db.esl

    # sign-efi-sig-list -k PK.key -c PK.crt KEK compound_KEK.esl compound_KEK.auth
    Timestamp is 2020-3-15 15:53:01
    Authentication Payload size 903
    Signature of size 1211
    Signature at: 40

    # sign-efi-sig-list -k KEK.key -c KEK.crt db compound_db.esl compound_db.auth
    Timestamp is 2020-3-15 15:53:05
    Authentication Payload size 905
    Signature of size 1223
    Signature at: 40

### Tryb ustawień (setup mode)

Mając przygotowane własne pliki z kluczami, możemy przejść do wyczyszczenia starych kluczy z
poziomu interfejsu firmware EFI/UEFI naszego laptopa. Wyczyszczenie kluczy przełączy nas
automatycznie w tryb ustawień (setup mode), w którym to będziemy mogli wgrać te wyżej utworzone
klucze bez żadnej weryfikacji całego procesu.

W pewnych implementacjach EFI/UEFI wymagane jest ustawienie hasła nadzorczego (supervisor password),
by opcja do wyczyszczenia kluczy Secure Boot stała się dostępna. W przypadku mojego laptopa Lenovo
ThinkPad T430, czyszczenie wbudowanych kluczy w firmware EFI/UEFI odbywa się w poniższy sposób.

Wchodzimy w ustawienia firmware EFI/UEFI i szukamy opcji od Secure Boot:

![](/img/2020/03/001-debian-linux-efi-uefi-firmware-bios-configuration-secure-boot.jpg#huge)

![](/img/2020/03/002-debian-linux-efi-uefi-firmware-bios-configuration-secure-boot.jpg#huge)

Jak widać wyżej, mamy pozycję od resetowania kluczy ( `Reset to Setup Mode` ) oraz od przywrócenia
kluczy fabrycznych ( `Restore Factory Keys` ). Nas interesuje ta pierwsza opcja:

![](/img/2020/03/003-debian-linux-efi-uefi-firmware-bios-configuration-secure-boot-setup-mode.jpg#huge)

![](/img/2020/03/004-debian-linux-efi-uefi-firmware-bios-configuration-secure-boot-setup-mode.jpg#huge)

Po zaakceptowaniu, pozycja `Platform Mode` powinna się zmienić z `User Mode` na `Setup Mode`, a `Secure Boot Mode`
z `Standard Mode` na `Custom Mode` :

![](/img/2020/03/005-debian-linux-efi-uefi-firmware-bios-configuration-secure-boot-setup-mode.jpg#huge)

Zapisujemy ustawienia firmware EFI/UEFI i restartujemy komputer (tryb Secure Boot powinien być
automatycznie wyłączony):

![](/img/2020/03/006-debian-linux-efi-uefi-firmware-bios-configuration-secure-boot-setup-mode-save.jpg#huge)

Po zalogowaniu się w systemie sprawdzamy czy zmienne `PK` , `KEK` , `db` oraz `dbx` zostały
wyczyszczone. Odpalamy zatem terminal i wpisujemy w nim polecenie `efi-readvar` :

    # efi-readvar
    Variable PK has no entries
    Variable KEK, length 2545
    KEK: List 0, type X509
        Signature 0, size 957, owner 7facc7b6-127f-4e9c-9c5d-080f98994345
            Subject:
                C=JP, ST=Kanagawa, L=Yokohama, O=Lenovo Ltd., CN=Lenovo Ltd. KEK CA 2012
            Issuer:
                C=JP, ST=Kanagawa, L=Yokohama, O=Lenovo Ltd., CN=Lenovo Ltd. KEK CA 2012
    KEK: List 1, type X509
        Signature 0, size 1532, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Subject:
                C=US, ST=Washington, L=Redmond, O=Microsoft Corporation, CN=Microsoft Corporation KEK CA 2011
            Issuer:
                C=US, ST=Washington, L=Redmond, O=Microsoft Corporation, CN=Microsoft Corporation Third Party Marketplace Root
    Variable db, length 4209
    db: List 0, type SHA256
        Signature 0, size 48, owner 7facc7b6-127f-4e9c-9c5d-080f98994345
            Hash:14e62a4905e19189e70828983165939afc0a331d0b415f3332b0e818a827f436
    db: List 1, type X509
        Signature 0, size 1515, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Subject:
                C=US, ST=Washington, L=Redmond, O=Microsoft Corporation, CN=Microsoft Windows Production PCA 2011
            Issuer:
                C=US, ST=Washington, L=Redmond, O=Microsoft Corporation, CN=Microsoft Root Certificate Authority 2010
    db: List 2, type X509
        Signature 0, size 1572, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Subject:
                C=US, ST=Washington, L=Redmond, O=Microsoft Corporation, CN=Microsoft Corporation UEFI CA 2011
            Issuer:
                C=US, ST=Washington, L=Redmond, O=Microsoft Corporation, CN=Microsoft Corporation Third Party Marketplace Root
    db: List 3, type X509
        Signature 0, size 962, owner 7facc7b6-127f-4e9c-9c5d-080f98994345
            Subject:
                C=JP, ST=Kanagawa, L=Yokohama, O=Lenovo Ltd., CN=ThinkPad Product CA 2012
            Issuer:
                C=JP, ST=Kanagawa, L=Yokohama, O=Lenovo Ltd., CN=Lenovo Ltd. Root CA 2012
    Variable dbx, length 76
    dbx: List 0, type SHA256
        Signature 0, size 48, owner 7facc7b6-127f-4e9c-9c5d-080f98994345
            Hash:14e62a4905e19189e70828983165939afc0a331d0b415f3332b0e818a827f436
    Variable MokList has no entries

Z powyższych informacji wynika, że w zasadzie to tylko zmienna `PK` została wyczyszczona. Zmienne
`KEK` i `db` nie zostały ruszone w żaden sposób, zaś zmienna `dbx` została dość mocno odchudzona.
Tak czy inaczej, widać, że klucze Lenovo i Microsoft'u są obecne w dalszym ciągu w zmiennych `KEK`
i `db` . W niczym to nie przeszkadza, bo za moment zawartość tych zmiennych zostanie zastąpiona
plikami, które wyżej przygotowaliśmy.

#### Aktualizacja kluczy przez efi-updatevar

Najprostszym sposobem na wgranie wygenerowanych przez nas kluczy jest posłużenie się narzędziem
`efi-updatevar` dostępnym w pakiecie `efitools` . Będąc cały czas na działającym linux'ie odpalamy
terminal i wpisujemy te poniższe polecenia:

    # efi-updatevar -e -f old_dbx.esl dbx
    # efi-updatevar -e -f compound_db.esl db
    # efi-updatevar -e -f compound_KEK.esl KEK
    # efi-updatevar -f PK.auth PK

Niestety w przypadku mojego ThinkPad'a przy próbie zapisu tych zmiennych zostają zwrócone
poniższe błędy:

	# efi-updatevar -e -f old_dbx.esl dbx
	Failed to update dbx: Operation not permitted

	# efi-updatevar -e -f compound_db.esl db
	Failed to update db: Operation not permitted

	# efi-updatevar -e -f compound_KEK.esl KEK
	Failed to update KEK: Operation not permitted

Teoretycznie te trzy polecenia powinny zadziałać w większości przypadków ale wygląda na to, że mój
przypadek się do takowych nie zalicza i posłużenie się `efi-updatevar` w takiej sytuacji zwyczajnie
odpada i trzeba posiłkować się innymi sposobami.

Jeśli napotkaliśmy ten powyższy problem, to prawdopodobnie pliki w katalogu
`/sys/firmware/efi/efivars/` [mają ustawiony bit odporności i trzeba im go usunąć][32].

#### Wgrywanie kluczy z poziomu firmware EFI/UEFI

Firmware EFI/UEFI niektórych komputerów z wyższej półki umożliwiają manipulowanie kluczami z
poziomu swojego interfejsu graficznego. W przypadku firmware tego laptopa próżno jednak szukać
stosownych opcji w konfiguracji, zatem nie da rady przepisać zmiennych `PK` , `KEK` , `db` i `dbx`
tym sposobem również.

#### Wgrywanie kluczy przez KeyTool

W pakiecie `efitools` znajduje się plik `KeyTool.efi` , którego przeznaczeniem jest umożliwienie
operowania na zmiennych firmware EFI/UEFI. Ten plik trzeba będzie uruchomić bezpośrednio w
środowisku firmware, tj. zanim zostanie załadowany system operacyjny czy nawet bootloader. Jako, że
[ja korzystam z menadżera rozruchu rEFInd][16], to mogę przekopiować plik `KeyTool.efi` na partycję
ESP do katalogu `EFI/tools/` :

	# cp /usr/lib/efitools/x86_64-linux-gnu/KeyTool.efi /efi/EFI/tools/

Na partycję ESP dobrze jest też przekopiować potrzebne pliki z kluczami (nie kopiujmy tylko kluczy
prywatnych):

    # cp /etc/kernel_key/*.{auth,cer,crt,esl} /efi/EFI/secure_boot_keys/

Po zresetowaniu maszyny stosowna ikonka powinna pojawić się w menu wyboru (to ten żółty klucz) na
fotce niżej:

![](/img/2020/03/007-debian-linux-efi-uefi-firmware-bios-configuration-secure-boot-setup-mode-mok-keytool.jpg#huge)

Naturalnie wybieramy tę pozycję, odpalamy KeyTool i z menu, które zostanie nam zaprezentowane,
wybieramy `Edit Keys` :

![](/img/2020/03/008-debian-linux-efi-uefi-firmware-bios-setup-mode-keys-edit.jpg#huge)

Teraz kolejno edytujemy zmienne zastępując ich zawartość przygotowanymi wcześniej plikami.

Zmienna `dbx` :

![](/img/2020/03/009-debian-linux-efi-uefi-firmware-bios-setup-mode-keys-edit-dbx.jpg#huge)

![](/img/2020/03/010-debian-linux-efi-uefi-firmware-bios-setup-mode-keys-edit-dbx.jpg#huge)

![](/img/2020/03/011-debian-linux-efi-uefi-firmware-bios-setup-mode-keys-edit-dbx.jpg#huge)

Zmienna `db` :

![](/img/2020/03/012-debian-linux-efi-uefi-firmware-bios-setup-mode-keys-edit-db.jpg#huge)

![](/img/2020/03/013-debian-linux-efi-uefi-firmware-bios-setup-mode-keys-edit-db.jpg#huge)

![](/img/2020/03/014-debian-linux-efi-uefi-firmware-bios-setup-mode-keys-edit-db.jpg#huge)

Zmienna `KEK` :

![](/img/2020/03/015-debian-linux-efi-uefi-firmware-bios-setup-mode-keys-edit-kek.jpg#huge)

![](/img/2020/03/016-debian-linux-efi-uefi-firmware-bios-setup-mode-keys-edit-kek.jpg#huge)

![](/img/2020/03/017-debian-linux-efi-uefi-firmware-bios-setup-mode-keys-edit-kek.jpg#huge)

Zmienna `PK` :

![](/img/2020/03/018-debian-linux-efi-uefi-firmware-bios-setup-mode-keys-edit-pk.jpg#huge)

![](/img/2020/03/019-debian-linux-efi-uefi-firmware-bios-setup-mode-keys-edit-pk.jpg#huge)

![](/img/2020/03/020-debian-linux-efi-uefi-firmware-bios-setup-mode-keys-edit-pk.jpg#huge)

Po wgraniu klucza do zmiennej `PK` , wracamy do menu głównego, w którym to powinniśmy zobaczyć, że
tryb w jakim operował firmware zmienił się z `Setup Mode` na `User Mode` (w górnej części ekranu):

![](/img/2020/03/021-debian-linux-efi-uefi-firmware-bios-setup-mode-keys-edit-exit.jpg#huge)

Wychodzimy i uruchamiamy system bez włączonego póki co jeszcze Secure Boot:

![](/img/2020/03/022-debian-linux-efi-uefi-firmware-bios-boot-system.jpg#huge)

### Sprawdzenie zmiennych

Po wgraniu zawartości plików do określonych zmiennych i uruchomieniu systemu sprawdzamy, czy udało
się nam przepisać wartości zmiennych `PK` , `KEK` , `db` oraz `dbx` :

	# efi-readvar

	Variable PK, length 853
	PK: List 0, type X509
		Signature 0, size 825, owner 90a7d962-e601-4363-a326-524f9f76132b
			Subject:
				CN=morfikov's platform key
			Issuer:
				CN=morfikov's platform key
	Variable KEK, length 861

	KEK: List 0, type X509
		Signature 0, size 833, owner 95b65092-8342-4a45-9dc2-d80233aba2b3
			Subject:
				CN=morfikov's key-exchange-key
			Issuer:
				CN=morfikov's key-exchange-key
	Variable db, length 865

	db: List 0, type X509
		Signature 0, size 837, owner 1091ff9c-7b84-4df6-8163-ed1b8aa05096
			Subject:
				CN=morfikov's kernel-signing key
			Issuer:
				CN=morfikov's kernel-signing key
	Variable dbx, length 3800

	dbx: List 0, type SHA256
		Signature 0, size 48, owner 7facc7b6-127f-4e9c-9c5d-080f98994345
			Hash:14e62a4905e19189e70828983165939afc0a331d0b415f3332b0e818a827f436
	dbx: List 1, type SHA256
		Signature 0, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
			Hash:80b4d96931bf0d02fd91a61e19d14f1da452e66db2408ca8604d411f92659f0a
		...
		Signature 76, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
			Hash:45c7c8ae750acfbb48fc37527d6412dd644daed8913ccd8a24c94d856967df8e
	Variable MokList has no entries

Jak widać, wbudowane w firmware EFI/UEFI klucze zostały zastąpione tymi wygenerowanymi przez nas i
w tej chwili tylko to oprogramowanie, które podpiszemy, może być załadowane w trybie Secure Boot na
naszym laptopie. W zmiennej `dbx` są dokładnie takie same wpisy co poprzednio, bo została ona
odtworzona, jako że jest to w zasadzie jedynie czarna lista hash'y, których raczej nie powinniśmy
ignorować.

## Testowanie Secure Boot z własnymi kluczami

Do wstępnego przetestowania mechanizmu Secure Boot z własnoręcznie wygenerowanymi kluczami
potrzebny jest nam jedynie podpisany przez nas kernel oraz zarejestrowany wpis w firmware EFI/UEFI,
który można dodać za pomocą `efibootmgr` .

### Podpisywanie kernela

O ile kernel dystrybucyjny i jego moduły są podpisane kluczem Debiana, o tyle [ten nasz
własnoręcznie zbudowany][27] nie jest. Po skończonym procesie kompilacji, wynikowy plik `vmlinuz`
nie jest w żaden sposób podpisany i trzeba go podpisać ręcznie przy pomocy prywatnego klucza `db` .
Do tego celu służy narzędzie `sbsign` dostępne w pakiecie `sbsigntool` . Proces podpisywania
binarki kernela wygląda zaś następująco:

	# cd /etc/kernel_key
	# sbsign --cert db.crt --key db.key /boot/vmlinuz-5.5.9-amd64
	Signing Unsigned original image

Zarówno certyfikat jak i klucz prywatny są niezbędne w procesie podpisywania i trzeba ścieżki do
nich określić w opcjach `--cert` i `--key` . Ostatni argument to ścieżka do pliku, który zamierzamy
podpisać. Plik, który podpisujemy nie jest zmieniany w żaden sposób, tylko jest on kopiowany z
nazwą mająca sufiks `.signed` i to ta kopia jest podpisywana. By zweryfikować czy plik został
pomyślnie podpisany, wydajemy poniższe polecenia:

	# sbverify --list '/boot/vmlinuz-5.5.9-amd64.signed'
	signature 1
	image signature issuers:
	 - /CN=morfikov's kernel-signing key
	image signature certificates:
	 - subject: /CN=morfikov's kernel-signing key
	   issuer:  /CN=morfikov's kernel-signing key

Jak widać plik zawiera sygnaturę naszego klucza `db` . Sprawdźmy czy da radę ją zweryfikować:

	# sbverify --cert db.crt /boot/vmlinuz-5.5.9-amd64.signed
	Signature verification OK

Informacja `Signature verification OK` świadczy o pomyślnym zakończeniu procesu weryfikacji.

### Podpisywanie modułów kernela

Zwykle podpisanie binarki kernela nam wystarczy ale w przypadku, gdy budowaliśmy jakieś moduły
zamiast wszystko wkompilować na stałe w kernel, to trzeba zadbać również o podpisanie tych modułów.
W konfiguracji kernela (do wglądu przez `make xconfig` ), można doszukać się opcji
`CONFIG_MODULE_SIG_KEY` , która wygląda mniej więcej tak:

![](/img/2020/03/023-debian-linux-secure-boot-kernel-config-sign-modules-key.jpg#huge)

Ta opcja (jak i jej podobne) odpowiada w zasadzie za podpisanie tylko i wyłączenie modułów, które
nie zostaną wkompilowane na stałe w kernel. Tak czy inaczej w polu widocznym wyżej musimy określić
ścieżkę do pliku klucza prywatnego `db` i jego certyfikatu. Niemniej jednak, certyfikat jest w
osobny pliku `.crt` , a klucz prywatny w osobnym pliku `.key` . Trzeba te dwa pliki pierw połączyć:

    # cat db.key db.crt > kernel-key.pem

Ścieżkę do tak otrzymanego pliku trzeba podać w zmiennej `CONFIG_MODULE_SIG_KEY` w konfiguracji
kernela.

#### Podpisywanie modułów DKMS

Jeśli budujemy jakieś moduły zewnętrzne przy pomocy mechanizmu DKMS, to również musimy zatroszczyć
się o ich podpisanie. Oczywiście ręczne podpisywanie byłoby dość mozolne i na szczęście możemy ten
[proces podpisywania modułów DKSM zautomatyzować][13]. Ten podlinkowany artykuł idealnie rozprawia
się z tą kwestią, trzeba tylko wskazać odpowiednią ścieżkę do certyfikatu i klucza prywatnego `db`
w skrypcie `/etc/kernel_key/sign-kernel.sh` .

### Firmware do WiFi/BT, a Secure Boot

Część sprzętu, który będziemy chcieli używać pod linux, np. karty WiFi czy adaptery bluetooth,
wymaga dodatkowego firmware pod linux, by prawidłowo funkcjonować. Włączenie Secure Boot nie wpływa
w żadnym stopniu na fakt czy firmware, którego potrzebuje nasza karta, zostanie załadowany przez
kernel czy też nie. Z racji, że pliki firmware sprzętu znajdujące się w katalogu `/lib/firmware/`
nie są ładowane przez firmware EFI/UEFI tylko przez kernel linux'a, to nie ma potrzeby podpisywania
tych plików. Poniżej znajduje się potwierdzenie tych słów na przykładzie karty WiFi mojego laptopa
wykorzystującej moduł `iwlwifi` , który potrzebny plik firmware `iwlwifi-6000g2a-6.ucode` ładuje
sobie z obrazu initrd/initramfs:

    ...
    kernel: secureboot: Secure boot enabled
    ...
    kernel: iwlwifi 0000:03:00.0: can't disable ASPM; OS doesn't have ASPM control
    kernel: iwlwifi 0000:03:00.0: firmware: direct-loading firmware iwlwifi-6000g2a-6.ucode
    kernel: iwlwifi 0000:03:00.0: loaded firmware version 18.168.6.1 op_mode iwldvm
    kernel: iwlwifi 0000:03:00.0: CONFIG_IWLWIFI_DEBUG enabled
    kernel: iwlwifi 0000:03:00.0: CONFIG_IWLWIFI_DEBUGFS disabled
    kernel: iwlwifi 0000:03:00.0: CONFIG_IWLWIFI_DEVICE_TRACING enabled
    kernel: iwlwifi 0000:03:00.0: Detected Intel(R) Centrino(R) Advanced-N 6205 AGN, REV=0xB0
    kernel: iwlwifi 0000:03:00.0: Radio type=0x1-0x2-0x0
    kernel: iwlwifi 0000:03:00.0: Radio type=0x1-0x2-0x0

Jak widać, brak podpisanego firmware nie przeszkadza w załadowaniu go do pamięci przez kernel.

### Dodanie wpisu w firmware EFI/UEFI via efibootmgr

Podpisany kernel kopiujemy wraz z ewentualnym obrazem initrd/initramfs na partycję ESP i przy
pomocy narzędzia `efibootmgr` dostarczanego w pakiecie o tej samej nazwie dodajemy stosowny wpis
w konfiguracji firmware EFI/UEFI:

    # cp /boot/vmlinuz-5.5.9-amd64.signed /efi/EFI/refind/vmlinuz-morfik.signed
    # cp /boot/initrd.img-5.5.9-amd64 /efi/EFI/refind/initrd.img-morfik-amd64

    # efibootmgr --create --disk /dev/sda --part 3 --label "Debian EFISTUB" \
                 --loader '\EFI\refind\vmlinuz-morfik-amd64.signed' --unicode \
                 'root=/dev/mapper/wd_blue_label-root ro \
                 initrd=\EFI\refind\initrd.img-morfik-amd64' --verbose

### Włączenie Secure Boot w konfiguracji EFI/UEFI

Pozostało nam już w zasadzie włączyć w konfiguracji firmware EFI/UEFI opcję Secure Boot:

![](/img/2020/03/024-debian-linux-efi-uefi-firmware-bios-configuration-secure-boot-enable.jpg#huge)

Jak tylko przełączymy tę opcję z `Disabled` na `Enabled` , to nie będziemy w stanie zmienić
ustawień co do trybu w jakim będzie uruchamiał się system (EFI/UEFI vs. BIOS):

![](/img/2020/03/025-debian-linux-efi-uefi-firmware-bios-configuration-secure-boot-enable.jpg#huge)

Zapisujemy ustawienia firmware EFI/UEFI i uruchamiamy maszynę ponownie:

![](/img/2020/03/026-debian-linux-efi-uefi-firmware-bios-configuration-secure-boot-enable.jpg#huge)

Podczas początkowej fazy startu komputera, w lewym górnym rogu ekranu powinna być informacja o tym,
że system operacyjny uruchamiany jest w trybie Secure Boot ( `EFI stub: UEFI Secure Boot is
enabled` ):

![](/img/2020/03/027-debian-linux-secure-boot-system-boot-message.jpg#huge)

Jeśli przegapimy tę informację, to zawsze też można sprawdzić log systemowy, by upewnić się ze nasz
linux wystartował w trybie Secure Boot:

    # journalctl -b | grep -i secureboot
    kernel: secureboot: Secure boot enabled

#### Mechanizm kernel lockdown

Włączenie Secure Boot w firmware EFI/UEFI aktywuje w kernelu linux'a [mechanizm kernel lockdown][6].
Po jego włączeniu nie będziemy mogli ładować modułów, które nie są podpisane zaufanym kluczem (np.
te [budowane za sprawą DKMS][7]). Nie będzie nam działał też [kexec][8] na niepodpisanych obrazach
kernela. Podobnie przestanie działać hibernacja systemu (przełączenie maszyny w tryb
uśpienia/wstrzymania w dalszym ciągu będzie możliwe). Dostęp do fizycznej pamięci i portów I/O
zostanie zabroniony z przestrzeni użytkownika. Nie będzie można też używać parametrów modułów,
które pozwalają na ustawianie adresów pamięci i portów I/O. Dodatkowo zapis do [rejestrów procesora
MSR][9] przez `/dev/cpu/*/msr` nie będzie już możliwy. Podobnie nie będzie można używać
niestandardowych [metod i tablic][10] ACPI. Nie będzie też można używać [APEI error injection][11].

Obecność włączonego w systemie kernel lockdown można poznać po poniższych komunikatach w logu
systemowym:

    # journalctl -b | grep Lockdown
    kernel: Lockdown: swapper: use of tracefs is restricted; see https://wiki.debian.org/SecureBoot
    kernel: Lockdown: swapper/0: use of tracefs is restricted; see https://wiki.debian.org/SecureBoot
    kernel: Lockdown: swapper/0: hibernation is restricted; see https://wiki.debian.org/SecureBoot
    kernel: Lockdown: resume: hibernation is restricted; see https://wiki.debian.org/SecureBoot
    kernel: Lockdown: systemd: /dev/mem,kmem,port is restricted; see https://wiki.debian.org/SecureBoot
    kernel: Lockdown: systemd-logind: hibernation is restricted; see https://wiki.debian.org/SecureBoot

Jak widać, kernel lockdown to dość inwazyjny mechanizm i włączenie go może coś popsuć w naszym
systemie. Naturalnie Secure Boot może działać bez kernel lockdown i jeśli potrzebujemy którejś z
tych wyżej wymienionych rzeczy, to zawsze możemy skompilować kernel z wyłączoną opcją
`CONFIG_LOCK_DOWN_IN_EFI_SECURE_BOOT` :

![](/img/2020/03/028-debian-linux-secure-boot-kernel-config-efi-uefi-force-lockdown.jpg#huge)

Można też całkowicie pozbyć się modułu lockdown odznaczając opcję `CONFIG_SECURITY_LOCKDOWN_LSM` :

![](/img/2020/03/029-debian-linux-secure-boot-kernel-config-efi-uefi-force-lockdown.jpg#huge)

W moim przypadku, mechanizm lockdown upośledza hibernację, z której korzystam dość często i nie
zamierzam rezygnować, bo daje ona możliwość zapisania stanu środowiska pracy i nie trzeba po
starcie maszyny wszystkiego na nowo uruchamiać, co znacznie przyśpiesza gotowość systemu. Dlaczego
zatem tak użyteczny mechanizm wypadł z łask lockdown? Okazuje się, że weryfikacja obrazu pamięci
RAM zrobionego przy hibernowaniu maszyny jest dość trudna i póki co [nie istnieje rozwiązanie][17],
które by taką weryfikację oferowało. Ja mam to szczęście, że korzystam z w pełni zaszyfrowanego
systemu, zatem obraz pamięci operacyjnej mam zawsze zaszyfrowany, przez co jest on niepodatny na
nieautoryzowane zmiany. Dlatego też postanowiłem całkowicie wyłączyć lockdown dla swojego systemu w
konfiguracji kernela i pewne rzeczy skonfigurować, tak by upodobnić swój system do zachowania
znanego z lockdown. [Patrząc na to co lockdown próbuje robić][25] (podpierając się również [tym
linkiem][26]), odznaczyłem te poniższe opcje w kernelu:

	CONFIG_DEVMEM:
	CONFIG_DEVKMEM:
	CONFIG_DEVPORT:
	CONFIG_ACPI_APEI_EINJ:
	CONFIG_X86_MSR:
	CONFIG_KEXEC:
	CONFIG_KEXEC_FILE:
	CONFIG_ACPI_CUSTOM_METHOD:
	CONFIG_PROC_KCORE:
	CONFIG_KPROBES:
	CONFIG_EFI_TEST:
	CONFIG_MMIOTRACE:
	CONFIG_DEBUG_KERNEL:

Nie jest to może pełny lockdown ale lepsze to niż nic i póki co system zdaje się działać stabilnie.

### Podpisanie boodloader'a

W przypadku instalacji mojego Debiana zrezygnowałem z tradycyjnego bootloader'a Grub2 i używam
jedynie menadżera rozruchu rEFInd. Niemniej jednak, po włączeniu Secure Boot, firmware EFI/UEFI nie
będzie chciał załadować binarek rEFInd'a. Pod pojęciem "binarek" mam na myśli nie tylko sam plik
`EFI/refind/refind_x64.efi` ale również wszystkie jego narzędzia dostępne w katalogu `EFI/tools/`
czy też sterowniki od systemu plików, które siedzą w `EFI/refind/drivers_x64/` . Te wszystkie pliki
mające sufiks `.efi` trzeba osobno podpisać jeśli chcemy by firmware EFI/UEFI umożliwił nam
korzystanie z nich.

Podpisanie binarki rEFInd'a (plik `EFI/refind/refind_x64.efi` ):

    # sbsign --cert db.crt --key db.key '/efi/EFI/refind/refind_x64.efi'
    Signing Unsigned original image

    # sbverify --cert db.crt '/efi/EFI/refind/refind_x64.efi.signed'
    Signature verification OK

Podpisanie sterownika ext4 dla rEFInd'a (plik `EFI/refind/drivers_x64/ext4_x64.efi` ):

    # sbsign --cert db.crt --key db.key '/efi/EFI/refind/drivers_x64/ext4_x64.efi'
    Signing Unsigned original image

    # sbverify --cert db.crt '/efi/EFI/refind/drivers_x64/ext4_x64.efi.signed'
    Signature verification OK

### Shim i MokManager

Mając skonfigurowane własne klucze odpada nam potrzeba korzystania z shim. Jeśli jednak
chcielibyśmy go zainstalować, to potrzebna nam będzie wersja niepodpisana, którą sami sobie
podpiszemy. Musimy zatem przekopiować pliki `shimx64.efi` oraz `mmx64.efi` na partycję ESP.
Dodatkowo, trzeba zainstalować rEFInd'a zmieniając mu nazwę z `refind_x64.efi` na `grubx64.efi` ,
by został podebrany przez shim:

    # cp /usr/lib/shim/shimx64.efi  /efi/EFI/refind/shimx64.efi
    # cp /usr/lib/shim/mmx64.efi /efi/EFI/refind/mmx64.efi
    # cp /usr/share/refind/refind/refind_x64.efi /efi/EFI/refind/grubx64.efi

Podpisanie shim'a (pliki `EFI/refind/shimx64.efi` ):

    # sbsign --cert db.crt --key db.key '/efi/EFI/refind/shimx64.efi'
    Signing Unsigned original image

    # sbverify --cert db.crt '/efi/EFI/refind/shimx64.efi.signed'
    Signature verification OK

Podpisanie rEFInd'a (zainstalowanego w pliku `EFI/refind/grubx64.efi` ):

    # sbsign --cert db.crt  --key db.key  '/efi/EFI/refind/grubx64.efi'
    Signing Unsigned original image

    # sbverify --cert db.crt  '/efi/EFI/refind/grubx64.efi.signed'
    Signature verification OK

Jeśli zaś chodzi o MokManager, to lepiej jest go nie podpisywać (szczegóły niżej).

### Narzędzia w EFI/tools/

Jeśli zaś chodzi o narzędzia, które znajdują się w katalogu `EFI/tools/` na partycji ESP, to w ich
przypadku trzeba nieco uważać z pospisywaniem. Dla przykładu weźmy EFIshell. Ta aplikacja robi w
zasadzie za terminal, przy pomocy którego możemy między innymi:

- uruchamiać aplikacje w środowisku firmware EFI/UEFI, np. bootloader'y
- uruchamiać programy partycjonujące dysk twardy (diskpart)
- modyfikować zmienne menadżera rozruchu (bcfg)
- ładować sterowniki EFI/UEFI
- edytować pliki tekstowe (edit)
- uzyskać informacje o systemie lub samym firmware EFI/UEFI
- uzyskać informacje o mapie pamięci (memmap)
- korzystać z hexedit

Jak widać, podpisanie takiego EFIshell wiązałoby się ze sporymi problemami jeśli chodzi o
bezpieczeństwo naszego komputera, bo każda osoba mająca fizyczny dostęp do naszej maszyny mogła by
tę binarkę załadować bez problemu i Secure Boot by jej w tym nie przeszkodził.

Jako, że jest cała masa narzędzi, które można wgrać na partycję ESP, to musimy sobie zadać pytanie,
czy te narzędzia, z których zamierzamy korzystać mogą w jakiś sposób zagrozić bezpieczeństwu naszej
maszyny. Jeśli tak, to wyjściem z tej sytuacji jest niepodpisywanie tych binarek. Oczywiście w
takim przypadku po aktywowaniu Secure Boot nie będziemy w stanie z tych aplikacji korzystać.
Dlatego też trzeba będzie ten mechanizm pierw zdezaktywować. Naturalnie każdy z fizycznym dostępem
do komputera mógłby w takim przypadku wejść w ustawienia firmware EFI/UEFI i przełączyć stosowną
opcję, dlatego też wymagane będzie założenie hasła nadzorczego (supervisor password) na firmware
EFI/UEFI, które uniemożliwi wyłączenie trybu Secure Boot.

### Sufiks .signed

W procesie podpisywania plików stworzone zostały podpisane odpowiedniki mające sufiks `.signed` .
Trzeba te nazwy zmienić, tak by nie miały tej końcówki:

    # mv '/efi/EFI/refind/refind_x64.efi.signed' '/efi/EFI/refind/refind_x64.efi'
    # mv '/efi/EFI/refind/drivers_x64/ext4_x64.efi.signed' '/efi/EFI/refind/drivers_x64/ext4_x64.efi'
    # mv '/efi/EFI/refind/grubx64.efi.signed' '/efi/EFI/refind/grubx64.efi'
    # mv '/efi/EFI/refind/shimx64.efi.signed' '/efi/EFI/refind/shimx64.efi'

W przypadku bootloader'a czy boot menadżera (i jego narzędzi), ta powyższa czynność będzie
przeprowadzana w zasadzie tylko raz. Jeśli zaś chodzi o kernel'a, to przy każdej jego aktualizacji
trzeba będzie pamiętać o dostosowaniu nazwy pliku `vmlinuz` lub też uwzględnić nazwę z końcówką
`.signed` w konfiguracji rEFInd'a.

### warning: data remaining[]: gaps between PE/COFF sections?

Podczas podpisywania plików można zaobserwować ciągłe pojawianie się ostrzeżeń, których treść brzmi
mniej więcej tak: `warning: data remaining[]: gaps between PE/COFF sections?` . [Zgodnie z tym co
można wyczytać tutaj][18], to dokładnie nie jest znana przyczyna występowania tych komunikatów.
Niemniej jednak, zdają się one nie wpływać na proces podpisu/weryfikacji binarki i można je z
powodzeniem zignorować.

## Problemy z system live przy Secure Boot

Problem możemy za to mieć z różnego różnego rodzaju systemami live dostarczanymi na płytce czy
pendrive, które są wypuszczane przez konkretną dystrybucje linux'a. Te systemy live nie będą nam
działać i przy próbie uruchomienia systemu z zawierających je nośników dostaniemy błąd: `Secure
Boot validation failure loading bootx64.efi` (lub podobny):

![](/img/2020/03/030-debian-linux-secure-boot-validation-failure-live-ubuntu.jpg#huge)

Z informacji, które widać na ekranie wynika, że mamy w zasadzie te poniższe opcje rozwiązania
zaistniałego problemu:

- możemy zrezygnować z używania systemów live (raczej odpada)
- możemy wyłączyć Secure Boot w ustawieniach firmware EFI/UEFI na czas korzystania z systemu live w
   przypadku, gdy system live nie jest w żaden sposób podpisany
- możemy podpisać bootloader, kernel (i inne potrzebne elementy systemu live) naszym kluczem
   prywatnym `db`
- można przerobić nośnik live i zbudować go z wykorzystaniem `grub-mkstandalone` , wynikowy obraz
   Grub2 trzeba będzie oczywiście podpisać
- możemy skorzystać z niepodpisanego shim'a, podpisać go własnym kluczem `db` i dodać do MOK klucz
   dystrybucji oferującej system live
- możemy do zmiennej `db` dodać klucz Microsoft'u, którym jest podpisana wersja shim'a mająca w
   nazwie `.signed` , choć tutaj też trzeba będzie dodać do bazy MOK klucz dystrybucji oferującej
   system live

Po chwili namysłu raczej dwie ostatnie opcje są godne rozważenia. Jeśli nie mamy zamiaru uruchamiać
windows'a na naszym laptopie, czy też w ogóle mieć do czynienia z Microsoft, to zostaje nam w
zasadzie tylko jedna możliwość.

### Brak kluczy do weryfikacji systemów live linux'a

Po pomyślnym skonfigurowaniu Secure Boot powinniśmy mieć mniej więcej te poniższe klucze w
[keyring'u kernela][19] pod `/proc/keys` :

    # cat /proc/keys | grep -i asym
    109d1ebc I------     1 perm 1f010000     0     0 asymmetri morfikov's kernel-signing key: 8d9ac6d38bbf26e45a55454c54c448059f530633: X509.rsa 9f530633 []
    16b541ff I------     1 perm 1f030000     0     0 asymmetri benh@debian.org: 577e021cb980e0e820821ba7b54b4961b8b4fadf: X509.rsa []
    24968f3b I------     1 perm 1f030000     0     0 asymmetri sforshee: 00b28ddf47aef9cea7: X509.rsa []
    2e344477 I------     1 perm 1f030000     0     0 asymmetri morfikov's kernel-signing key: 8d9ac6d38bbf26e45a55454c54c448059f530633: X509.rsa 9f530633 []

Widać tutaj dwa klucze mające w nazwie `morfikov's kernel-signing key` i to są klucze, które
dodaliśmy w procesie konfigurowania Secure Boot opisanym powyżej. Czemu dwa? Jeden klucz wziął się
z wbudowania go w kernel w czasie jego kompilacji, bo potrzebowaliśmy go do podpisania modułów.
Drugi klucz z kolei pochodzi ze zmiennej `db` Secure Boot -- tu również dodaliśmy swój klucz. W
zasadzie użyliśmy jednego klucza w obu tych miejscach dlatego też mamy dwie podobne (różniące się
uprawnieniami) pozycje w keyring'u kernela.

    # keyctl list %:.secondary_trusted_keys
    1 key in keyring:
    891874024: ---lswrv     0     0 keyring: .builtin_trusted_keys

    # keyctl list %:.builtin_trusted_keys
    1 key in keyring:
    775177335: ---lswrv     0     0 asymmetric: morfikov's kernel-signing key: 8d9ac6d38bbf26e45a55454c54c448059f530633

Jeśli zaś chodzi o te dwa dodatkowe klucze ( `benh@debian.org` oraz `sforshee` ), to nie mają one
nic wspólnego z Secure Boot. [Zostały one dodane za sprawą wireless-regdb][20].

Każdy klucz, który jest dodawany do keyring'u kernela jest odnotowywany w logu systemowym:

    ...
    kernel: Loading compiled-in X.509 certificates
    kernel: Loaded X.509 cert 'morfikov's kernel-signing key: 8d9ac6d38bbf26e45a55454c54c448059f530633'
    kernel: integrity: Loading X.509 certificate: UEFI:db
    kernel: integrity: Loaded X.509 cert 'morfikov's kernel-signing key: 8d9ac6d38bbf26e45a55454c54c448059f530633'
    ...
    kernel: cfg80211: Loading compiled-in X.509 certificates for regulatory database
    kernel: cfg80211: Loaded X.509 cert 'benh@debian.org: 577e021cb980e0e820821ba7b54b4961b8b4fadf'
    kernel: cfg80211: Loaded X.509 cert 'sforshee: 00b28ddf47aef9cea7'
    ...

Oczywiście ten powyższy listing nie jest pełny, bo w keyring'u kernela znajduje się także zawartość
zmiennej `dbx` (czarnej listy):

    # cat /proc/keys | grep -i "blacklist\ " | wc -l
    78

Pozostałe wpisy w keyring'u dotyczą już uruchomionej sesji i zalogowanych użytkowników. Wszystko co
w tym keyring'u kernela się znajduje można także wyciągnąć z `efi-readvar` .

Jak można było zauważyć, nie mamy żadnego klucza dystrybucji, które dostarczają systemy live, np.
Debian czy Ubuntu. Jeśli chcemy mieć możliwość uruchamiania tych systemów live, to musimy pozyskać
stosowne certyfikaty i dodać je do bazy MOK.

#### Problem z keyctl

Czasami budując własny kernel można zapomnieć o włączeniu kilku opcji, których brak może powodować
problemy, np. `Can't find 'keyring:.secondary_trusted_keys'` . Poniżej znajduje się lista opcji,
które powinny się znaleźć w kernelu, by tych problemów uniknąć:

	CONFIG_INTEGRITY=y
	CONFIG_INTEGRITY_SIGNATURE=y
	CONFIG_INTEGRITY_ASYMMETRIC_KEYS=y
	CONFIG_INTEGRITY_TRUSTED_KEYRING=y
	CONFIG_INTEGRITY_PLATFORM_KEYRING=y
	CONFIG_LOAD_UEFI_KEYS=y
	CONFIG_INTEGRITY_AUDIT=y
	CONFIG_SIGNATURE=y
	CONFIG_SECONDARY_TRUSTED_KEYRING=y
	CONFIG_SYSTEM_BLACKLIST_KEYRING=y

### Dodawanie kluczy dystrybucji do bazy MOK

Na samym początku w MOK nie powinno być póki co żadnych kluczy, bo przecie jeszcze nic tam nie
dodawaliśmy. Dla pewności sprawdźmy to przy pomocy `mokutil` :

    # mokutil --list-enrolled
    MokListRT is empty

Certyfikaty niektórych dystrybucji zostały [zebrane przez rEFInd i są dostępne tutaj][15]. O ile
certyfikat Canonical tam znajdziemy, to nie ma tam z jakiegoś powodu certyfikatu Debiana, a o te
dwa nam głównie chodzi w tej chwili. Na szczęście [certyfikat Debiana można znaleźć tutaj][22]. Oba
te certyfikaty są zakodowane w DER, czyli dokładnie w takim formacie jaki nam jest potrzebny.
Pobieramy zatem te certy:

    # wget https://dsa.debian.org/secure-boot-ca -O debian-uefi-ca.cer
    # wget https://sourceforge.net/p/refind/code/ci/master/tree/keys/canonical-uefi-ca.cer?format=raw -O  canonical-uefi-ca.cer

Dodajemy teraz te certyfikaty do bazy MOK za pomocą `mokutil` (hasło ustawiamy krótkie, np. 1234):

    # mokutil --import debian-uefi-ca.cer canonical-uefi-ca.cer
    input password:
    input password again:

Te klucze czekają teraz by je dodać z poziomu MokManager'a i musimy w tym celu uruchomić komputer
ponownie. Jeśli nie wiemy czy to powyższe polecenie dodało jakiekolwiek klucze do kolejki, to
zawsze możemy zweryfikować kolejkę nowych kluczy przy pomocy `mokutil --list-new` .

Dodawanie certyfikatów w MokManager jest stosunkowo proste:

![](/img/2020/03/031-debian-linux-secure-boot-shim-configuration-mok.jpg#huge)

![](/img/2020/03/032-debian-linux-secure-boot-shim-configuration-mok.jpg#huge)

Sprawdzamy dane certyfikatu Debiana:

![](/img/2020/03/033-debian-linux-secure-boot-shim-configuration-mok.jpg#huge)

Podobnie weryfikujemy dane certyfikatu Canonical:

![](/img/2020/03/034-debian-linux-secure-boot-shim-configuration-mok.jpg#huge)

Jeśli dane się zgadzają, to przechodzimy dalej:

![](/img/2020/03/035-debian-linux-secure-boot-shim-configuration-mok.jpg#huge)

Potwierdzamy dodanie kluczy i wpisujemy hasło, które ustawiliśmy wcześniej:

![](/img/2020/03/037-debian-linux-secure-boot-shim-configuration-mok-password.jpg#huge)

Jeśli proces zakończył się bez żadnych błędów, to restartujemy maszynę:

![](/img/2020/03/038-debian-linux-secure-boot-shim-configuration-mok-reboot.jpg#huge)

Po ponownym uruchomieniu komputera, klucze powinny zostać z powodzeniem dodane, co możemy podejrzeć
w `mokutil` :

    # mokutil --list-enrolled
    [key 1]
    SHA1 Fingerprint: 53:61:0c:f8:1f:bd:7e:0c:eb:67:91:3c:9e:f3:e7:94:a9:63:3e:cb
    Certificate:
        Data:
            Version: 3 (0x2)
            Serial Number:
                ed:54:a1:d5:af:87:48:94:8d:9f:89:32:ee:9c:7c:34
            Signature Algorithm: sha256WithRSAEncryption
            Issuer: CN=Debian Secure Boot CA
            Validity
                Not Before: Aug 16 18:09:18 2016 GMT
                Not After : Aug  9 18:09:18 2046 GMT
            Subject: CN=Debian Secure Boot CA
            Subject Public Key Info:
                Public Key Algorithm: rsaEncryption
                    RSA Public-Key: (2048 bit)
                    Modulus:
                        00:9d:95:d4:8b:9b:da:10:ac:2e:ca:82:37:c1:a4:
                        cb:4a:c3:1b:42:93:c2:7a:29:d3:6e:dd:64:af:80:
                        af:ea:66:a2:1b:61:9c:83:0c:c5:6b:b9:35:25:ff:
                        c5:fb:e8:29:43:de:ce:4b:3d:c6:12:4d:b1:ef:26:
                        43:95:68:cd:04:11:fe:c2:24:9b:de:14:d8:86:51:
                        e8:38:43:bd:b1:9a:15:e5:08:6b:f8:54:50:8b:b3:
                        4b:5f:fc:14:e4:35:50:7c:0b:b1:e2:03:84:a8:36:
                        48:e4:80:e8:ea:9f:fa:bf:c5:18:7b:5e:ce:1c:be:
                        2c:80:78:49:35:15:c0:21:cf:ef:66:d5:8a:96:08:
                        2b:66:2f:48:17:b1:e7:ec:82:8f:07:e6:ca:e0:5f:
                        71:24:39:50:0a:8e:d1:72:28:50:a5:9d:21:f4:e3:
                        61:ba:09:03:66:c8:df:4e:26:36:0b:15:0f:63:1f:
                        2b:af:ab:c4:28:a2:56:64:85:8d:a6:55:41:ae:3c:
                        88:95:dd:d0:6d:d9:29:db:d8:c4:68:b5:fc:f4:57:
                        89:6b:14:db:e0:ef:ee:40:0d:62:1f:ea:58:d4:a3:
                        d8:ba:03:a6:97:2e:c5:6b:13:a4:91:77:a6:b5:ad:
                        23:a7:eb:0a:49:14:46:7c:76:e9:9e:32:b4:89:af:
                        57:79
                    Exponent: 65537 (0x10001)
            X509v3 extensions:
                Authority Information Access:
                    CA Issuers - URI:https://dsa.debian.org/secure-boot-ca

                X509v3 Authority Key Identifier:
                    keyid:6C:CE:CE:7E:4C:6C:0D:1F:61:49:F3:DD:27:DF:CC:5C:BB:41:9E:A1

                Netscape Cert Type: critical
                    SSL Client, SSL Server, S/MIME, Object Signing, SSL CA, S/MIME CA, Object Signing CA
                X509v3 Extended Key Usage:
                    Code Signing
                X509v3 Key Usage: critical
                    Digital Signature, Certificate Sign, CRL Sign
                X509v3 Basic Constraints: critical
                    CA:TRUE
                X509v3 Subject Key Identifier:
                    6C:CE:CE:7E:4C:6C:0D:1F:61:49:F3:DD:27:DF:CC:5C:BB:41:9E:A1
        Signature Algorithm: sha256WithRSAEncryption
             77:96:3e:47:c9:ce:09:cf:8b:89:ce:59:ed:26:0e:26:0b:b9:
             ad:a9:2b:bd:a1:eb:88:79:02:ff:31:de:fe:f5:6a:07:ef:61:
             13:11:70:1e:bf:9c:4e:66:6c:e1:62:12:97:01:57:65:47:dd:
             4a:c6:f7:f4:de:a8:f1:13:62:cc:83:57:ac:3c:a6:91:15:af:
             55:26:72:69:2e:14:cd:dd:4d:b3:d1:60:24:2d:32:4f:19:6c:
             11:5e:f2:a3:f2:a1:5f:62:0f:30:ae:ad:f1:48:66:64:7d:36:
             44:0d:06:34:3d:2e:af:8e:9d:c3:ad:c2:91:d8:37:e0:ee:7a:
             5f:82:3b:67:8e:00:8a:c4:a4:df:35:16:c2:72:2b:4c:51:d7:
             93:93:9e:ba:08:0d:59:97:f2:e2:29:a0:44:4d:ea:ee:f8:3e:
             02:60:ca:15:cf:4e:9a:25:91:84:3f:b7:5a:c7:ee:bc:6b:80:
             a3:d9:fd:b2:6d:7a:1e:63:14:eb:ef:f1:b0:40:25:d5:e8:0e:
             81:eb:6b:f7:cb:ff:e5:21:00:22:2c:2e:9a:35:60:12:4b:5b:
             5f:38:46:84:0c:06:9c:cf:72:93:62:18:ee:5c:98:d6:b3:7d:
             06:25:39:95:df:4e:60:76:b0:06:7b:08:b0:6e:e3:64:9f:21:
             56:ad:39:0f

    [key 2]
    SHA1 Fingerprint: 76:a0:92:06:58:00:bf:37:69:01:c3:72:cd:55:a9:0e:1f:de:d2:e0
    Certificate:
        Data:
            Version: 3 (0x2)
            Serial Number:
                b9:41:24:a0:18:2c:92:67
            Signature Algorithm: sha256WithRSAEncryption
            Issuer: C=GB, ST=Isle of Man, L=Douglas, O=Canonical Ltd., CN=Canonical Ltd. Master Certificate Authority
            Validity
                Not Before: Apr 12 11:12:51 2012 GMT
                Not After : Apr 11 11:12:51 2042 GMT
            Subject: C=GB, ST=Isle of Man, L=Douglas, O=Canonical Ltd., CN=Canonical Ltd. Master Certificate Authority
            Subject Public Key Info:
                Public Key Algorithm: rsaEncryption
                    RSA Public-Key: (2048 bit)
                    Modulus:
                        00:bf:5b:3a:16:74:ee:21:5d:ae:61:ed:9d:56:ac:
                        bd:de:de:72:f3:dd:7e:2d:4c:62:0f:ac:c0:6d:48:
                        08:11:cf:8d:8b:fb:61:1f:27:cc:11:6e:d9:55:3d:
                        39:54:eb:40:3b:b1:bb:e2:85:34:79:ca:f7:7b:bf:
                        ba:7a:c8:10:2d:19:7d:ad:59:cf:a6:d4:e9:4e:0f:
                        da:ae:52:ea:4c:9e:90:ce:c6:99:0d:4e:67:65:78:
                        5d:f9:d1:d5:38:4a:4a:7a:8f:93:9c:7f:1a:a3:85:
                        db:ce:fa:8b:f7:c2:a2:21:2d:9b:54:41:35:10:57:
                        13:8d:6c:bc:29:06:50:4a:7e:ea:99:a9:68:a7:3b:
                        c7:07:1b:32:9e:a0:19:87:0e:79:bb:68:99:2d:7e:
                        93:52:e5:f6:eb:c9:9b:f9:2b:ed:b8:68:49:bc:d9:
                        95:50:40:5b:c5:b2:71:aa:eb:5c:57:de:71:f9:40:
                        0a:dd:5b:ac:1e:84:2d:50:1a:52:d6:e1:f3:6b:6e:
                        90:64:4f:5b:b4:eb:20:e4:61:10:da:5a:f0:ea:e4:
                        42:d7:01:c4:fe:21:1f:d9:b9:c0:54:95:42:81:52:
                        72:1f:49:64:7a:c8:6c:24:f1:08:70:0b:4d:a5:a0:
                        32:d1:a0:1c:57:a8:4d:e3:af:a5:8e:05:05:3e:10:
                        43:a1
                    Exponent: 65537 (0x10001)
            X509v3 extensions:
                X509v3 Subject Key Identifier:
                    AD:91:99:0B:C2:2A:B1:F5:17:04:8C:23:B6:65:5A:26:8E:34:5A:63
                X509v3 Authority Key Identifier:
                    keyid:AD:91:99:0B:C2:2A:B1:F5:17:04:8C:23:B6:65:5A:26:8E:34:5A:63

                X509v3 Basic Constraints: critical
                    CA:TRUE
                X509v3 Key Usage:
                    Digital Signature, Certificate Sign, CRL Sign
                X509v3 CRL Distribution Points:

                    Full Name:
                      URI:http://www.canonical.com/secure-boot-master-ca.crl

        Signature Algorithm: sha256WithRSAEncryption
             3f:7d:f6:76:a5:b3:83:b4:2b:7a:d0:6d:52:1a:03:83:c4:12:
             a7:50:9c:47:92:cc:c0:94:77:82:d2:ae:57:b3:99:04:f5:32:
             3a:c6:55:1d:07:db:12:a9:56:fa:d8:d4:76:20:eb:e4:c3:51:
             db:9a:5c:9c:92:3f:18:73:da:94:6a:a1:99:38:8c:a4:88:6d:
             c1:fc:39:71:d0:74:76:16:03:3e:56:23:35:d5:55:47:5b:1a:
             1d:41:c2:d3:12:4c:dc:ff:ae:0a:92:9c:62:0a:17:01:9c:73:
             e0:5e:b1:fd:bc:d6:b5:19:11:7a:7e:cd:3e:03:7e:66:db:5b:
             a8:c9:39:48:51:ff:53:e1:9c:31:53:91:1b:3b:10:75:03:17:
             ba:e6:81:02:80:94:70:4c:46:b7:94:b0:3d:15:cd:1f:8e:02:
             e0:68:02:8f:fb:f9:47:1d:7d:a2:01:c6:07:51:c4:9a:cc:ed:
             dd:cf:a3:5d:ed:92:bb:be:d1:fd:e6:ec:1f:33:51:73:04:be:
             3c:72:b0:7d:08:f8:01:ff:98:7d:cb:9c:e0:69:39:77:25:47:
             71:88:b1:8d:27:a5:2e:a8:f7:3f:5f:80:69:97:3e:a9:f4:99:
             14:db:ce:03:0e:0b:66:c4:1c:6d:bd:b8:27:77:c1:42:94:bd:
             fc:6a:0a:bc

Co ciekawe, na tej powyższej liście były trzy klucze, w tym jeden zdublowany. Prawdopodobnie ten
dodatkowy wpis wziął się z zaszytego w shim certyfikatu dystrybucji Debiana. Wychodzi na to, że nie
trzeba go ręcznie dodawać (wystarczyłoby dodać certyfikat Canonical'a, a certyfikat Debiana by
wskoczył automatycznie).

Jeśli teraz podejrzymy keyring kernela, to zobaczymy tam dwa dodatkowe klucze, z racji, że w tym
keyring'u mogą się znajdować klucze wbudowane w shim oraz klucze, które są przechowywane w MOK:

    # cat /proc/keys | grep -i asym
    ...
    15c78670 I------     1 perm 1f010000     0     0 asymmetri Canonical Ltd. Master Certificate Authority: ad91990bc22ab1f517048c23b6655a268e345a63: X509.rsa 8e345a63 []
    ...
    3eee887b I------     1 perm 1f010000     0     0 asymmetri Debian Secure Boot CA: 6ccece7e4c6c0d1f6149f3dd27dfcc5cbb419ea1: X509.rsa bb419ea1 []

Te klucze dostarczane przez shim i MOK będą jedynie dostępne, gdy shim będzie pośredniczył w
procesie startu systemu. W moim przypadku przed implementacją Secure Boot był wykorzystywany
menadżer rozruchu rEFInd. Jeśli odpalę system bezpośrednio z menu wyboru rEFInd'a, to te powyższe
klucze nie zostaną dodane. Dlatego też jeśli zamierzamy z nich korzystać, to trzeba zainstalować
rEFInd ze wsparciem dla shim (była o tym mowa wcześniej) i zarejestrować wpis z shim w firmware
EFI/UEFI przez `efibootmgr` , przykładowo:

    # efibootmgr --create --disk /dev/sda --part 3 --label "SHIM" --loader '\EFI\refind\shimx64.efi' --verbose

    # efibootmgr -v
    BootCurrent: 0019
    Timeout: 0 seconds
    BootOrder: 0019,0000,0001,0002,0003,001A,0018,001B,0007,0008,0009,000A,000D,000B,000C,000E,000F,0010,0011,0012
    ...
    Boot0018* rEFInd Boot Manager   HD(3,GPT,c180ec1b-183f-43e0-853f-3c76950c3c52,0x745fc000,0x100000)/File(\EFI\refind\refind_x64.efi)
    Boot0019* SHIM  HD(3,GPT,c180ec1b-183f-43e0-853f-3c76950c3c52,0x745fc000,0x100000)/File(\EFI\refind\shimx64.efi)
    ...

Wpis z rEFInd może zostać ale trzeba będzie zmienić kolejność wpisów czytanych przy starcie systemu,
tak by uruchamiany był shim.

Tak czy inaczej, płytki live mające podpisany bootloader/kernel z wykorzystaniem kluczy prywatnych
od tych dodanych przez nas certyfikatów bez większego problemu będą u nas się uruchamiać. Podobnie,
gdybyśmy chcieli zainstalować sobie na stałe Debiana lub Ubuntu, to one też z powodzeniem będą się
uruchamiać w trybie Secure Boot.

Warto tutaj zwrócić uwagę, że może i dodaliśmy klucze do bazy MOK ale zmienna `MokList` w dalszym
ciągu nie ma żadnych wpisów:

    # efi-readvar -v MokList
    Variable MokList has no entries

### Test Ubuntu live z włączonym Secure Boot

Zrestartujmy zatem jeszcze raz komputer i podłączmy do portu USB pendrive z wgranym Ubuntu live, by
przetestować czy aby na pewno ten system się nam uruchomi.

W menu rEFInd powiły się dwa dodatkowe wpisy (pierwsze dwa z lewej):

![](/img/2020/03/039-debian-linux-secure-boot-shim-configuration-mok-live-ubuntu.jpg#huge)

Czemu dwa? Bo na pendrive znajdują się dwa bootloader'y: `grubx64.efi` (podpisany przez Canonical) i
`BOOTx64.EFI` (podpisany przez Microsoft) . Możemy to zweryfikować montując obraz Ubuntu live i
listując sygnatury złożone na tych dwóch plikach przy pomocy `sbverify` :

    # sbverify --list  '/mnt/EFI/BOOT/grubx64.efi'
    signature 1
    image signature issuers:
     - /C=GB/ST=Isle of Man/L=Douglas/O=Canonical Ltd./CN=Canonical Ltd. Master Certificate Authority
    image signature certificates:
     - subject: /C=GB/ST=Isle of Man/O=Canonical Ltd./OU=Secure Boot/CN=Canonical Ltd. Secure Boot Signing
       issuer:  /C=GB/ST=Isle of Man/L=Douglas/O=Canonical Ltd./CN=Canonical Ltd. Master Certificate Authority

    # sbverify --list  '/mnt/EFI/BOOT/BOOTx64.EFI'
    warning: data remaining[1171248 vs 1334816]: gaps between PE/COFF sections?
    signature 1
    image signature issuers:
     - /C=US/ST=Washington/L=Redmond/O=Microsoft Corporation/CN=Microsoft Corporation UEFI CA 2011
    image signature certificates:
     - subject: /C=US/ST=Washington/L=Redmond/O=Microsoft Corporation/CN=Microsoft Windows UEFI Driver Publisher
       issuer:  /C=US/ST=Washington/L=Redmond/O=Microsoft Corporation/CN=Microsoft Corporation UEFI CA 2011
     - subject: /C=US/ST=Washington/L=Redmond/O=Microsoft Corporation/CN=Microsoft Corporation UEFI CA 2011
       issuer:  /C=US/ST=Washington/L=Redmond/O=Microsoft Corporation/CN=Microsoft Corporation Third Party Marketplace Root

Z racji, że nie mamy kluczy Microsoft'u, to uruchomienie systemu live z tego drugiego wpisu się nie
powiedzie. Natomiast dysponując certyfikatem od Canonical bez problemu możemy odpalić Ubuntu z
pierwszego wpisu w menu rEFInd:

![](/img/2020/03/040-debian-linux-secure-boot-live-ubuntu-grub.jpg#huge)

![](/img/2020/03/041-debian-linux-secure-boot-live-ubuntu.jpg#huge)

I jeszcze dowód, że system live działa w trybie Secure Boot:

![](/img/2020/03/042-debian-linux-secure-boot-live-ubuntu-test.jpg#huge)

## Wsparcie Secure Boot dla windows

Tworząc własne klucze i podmieniając nimi te wbudowane w firmware EFI/UEFI nie uwzględniliśmy
kluczy Microsoft'u, bo w zasadzie jak widać do tej pory wszystko działa bez większego zarzutu i
dodawanie tych kluczy jest niepotrzebne. Jeśli jednak zamierzamy kiedyś w przyszłości zainstalować
sobie windows'a, to przydałoby się dodać klucze MS bezpośrednio do firmware EFI/UEFI. Można
oczywiście na etapie zastępowania kluczy wybrać opcję z uwzględnieniem kluczy Microsoft'u ale
[istnieje sposób na dodanie tych kluczy manualnie][23] bez potrzeby przełączania systemu w tryb
Setup Mode i dlatego postanowiłem go tutaj opisać.

[Microsoft dysponuje dwoma kluczami][24]. Pierwszy z nich służy do podpisywania windows'a, a drugi
do podpisywania shim'a (i innych aplikacji trzecich). Najlepiej jest dodać oba, choć oczywiście
możemy sobie wybrać ten, który nam jest potrzebny. W tym przypadku pobieramy oba certyfikaty i
poddajemy je temu samemu procesowi, który przeprowadzaliśmy w przypadku naszych własnych kluczy.

    # cd /etc/kernel_key/

Konwertujemy certyfikaty zakodowane w DER na format PEM:

    # openssl x509 -inform DER -outform PEM -in MicWinProPCA2011_2011-10-19.crt -out MicWinProPCA2011_2011-10-19.crt.pem
    # openssl x509 -inform DER -outform PEM -in MicCorUEFCA2011_2011-06-27.crt -out MicCorUEFCA2011_2011-06-27.crt.pem

Tworzymy listy sygnatur EFI/UEFI z ID Microsoft'u (77fa9abd-0359-4d32-bd60-28f4e78f784b):

    # cert-to-efi-sig-list -g 77fa9abd-0359-4d32-bd60-28f4e78f784b MicWinProPCA2011_2011-10-19.crt.pem MS_Win_db.esl
    # cert-to-efi-sig-list -g 77fa9abd-0359-4d32-bd60-28f4e78f784b MicCorUEFCA2011_2011-06-27.crt.pem MS_UEFI_db.esl

Dla uproszczenia sobie całego procesu łączymy te dwa pliki `.esl` w jeden:

    # cat MS_Win_db.esl MS_UEFI_db.esl > MS_db.esl

I na koniec podpisujemy naszym kluczem `KEK` aktualizację dla zmiennej `db` :

    # sign-efi-sig-list -a -g 77fa9abd-0359-4d32-bd60-28f4e78f784b -k KEK.key -c KEK.crt db MS_db.esl add_MS_db.auth

Restartujemy maszynę i przy pomocy KeyTool dodajemy zawartość pliku `add_MS_db.auth` do zmiennej
`db` :

![](/img/2020/03/043-debian-linux-secure-boot-edit-db-ms-microsoft.jpg#huge)

![](/img/2020/03/044-debian-linux-secure-boot-edit-db-ms-microsoft.jpg#huge)

![](/img/2020/03/045-debian-linux-secure-boot-edit-db-ms-microsoft.jpg#huge)

W keyring'u kernela powinny pojawić się dwie nowe pozycje:

    # cat /proc/keys | grep -i asym | grep Mic
    11ffee25 I------     1 perm 1f010000     0     0 asymmetri Microsoft Windows Production PCA 2011: a92902398e16c49778cd90f99e4f9ae17c55af53: X509.rsa 7c55af53 []
    35ef3b88 I------     1 perm 1f010000     0     0 asymmetri Microsoft Corporation UEFI CA 2011: 13adbf4309bd82709c8cd54f316ed522988a1bd4: X509.rsa 988a1bd4 []

Tak samo dwie nowe pozycje powinny zostać dodane do zmiennej `db` :

    # efi-readvar -v db
    Variable db, length 4008
    db: List 0, type X509
        Signature 0, size 837, owner 1091ff9c-7b84-4df6-8163-ed1b8aa05096
            Subject:
                CN=morfikov's kernel-signing key
            Issuer:
                CN=morfikov's kernel-signing key
    db: List 1, type X509
        Signature 0, size 1515, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Subject:
                C=US, ST=Washington, L=Redmond, O=Microsoft Corporation, CN=Microsoft Windows Production PCA 2011
            Issuer:
                C=US, ST=Washington, L=Redmond, O=Microsoft Corporation, CN=Microsoft Root Certificate Authority 2010
    db: List 2, type X509
        Signature 0, size 1572, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Subject:
                C=US, ST=Washington, L=Redmond, O=Microsoft Corporation, CN=Microsoft Corporation UEFI CA 2011
            Issuer:
                C=US, ST=Washington, L=Redmond, O=Microsoft Corporation, CN=Microsoft Corporation Third Party Marketplace Root

Gdy zamierzamy korzystać z kluczy MS, to przydałoby się wgrać na partycję ESP shim w wersji
podpisanej przez Microsoft (w Debianie jest to pakiet `shim-signed` ). Jeśli mamy wątpliwości czy
shim, który mamy na tej partycji jest podpisany kluczem MS, to zawsze możemy posłużyć się
poniższym poleceniem:

    # sbverify --list /efi/EFI/refind/shimx64.efi.signed
    warning: data remaining[1159744 vs 1322936]: gaps between PE/COFF sections?
    signature 1
    image signature issuers:
     - /C=US/ST=Washington/L=Redmond/O=Microsoft Corporation/CN=Microsoft Corporation UEFI CA 2011
    image signature certificates:
     - subject: /C=US/ST=Washington/L=Redmond/O=Microsoft Corporation/CN=Microsoft Windows UEFI Driver Publisher
       issuer:  /C=US/ST=Washington/L=Redmond/O=Microsoft Corporation/CN=Microsoft Corporation UEFI CA 2011
     - subject: /C=US/ST=Washington/L=Redmond/O=Microsoft Corporation/CN=Microsoft Corporation UEFI CA 2011
       issuer:  /C=US/ST=Washington/L=Redmond/O=Microsoft Corporation/CN=Microsoft Corporation Third Party Marketplace Root

By przetestować czy te klucze działają, możemy posłużyć się obrazem Ubuntu live i wybrać tryb
fallback. Jeśli się uruchomi, to znaczy, że klucze spełniają swoją rolę.

## Wyłączenie Secure Boot na poziomie shim

Jeśli Secure Boot jest z jakiegoś powodu wymagany i nasz komputer nie chce się uruchomić bez tej
opcji włączonej, to zawsze możemy spróbować włączyć Secure Boot w ustawieniach firmware EFI/UEFI i
wyłączyć go na poziomie shim. Ten zabieg jest bardzo prosty i w zasadzie sprowadza się do wydania
w terminalu jednego polecenia:

    # mokutil --disable-validation
    password length: 8~16
    input password:
    input password again:

Teraz wystarczy zrestartować system i dokończyć proces w MokManager:

![](/img/2020/03/046-debian-linux-secure-boot-shim-disable.jpg#huge)

![](/img/2020/03/047-debian-linux-secure-boot-shim-disable-password.jpg#huge)

![](/img/2020/03/048-debian-linux-secure-boot-shim-disable-reboot.jpg#huge)

Po wyłączeniu Secure Boot w taki sposób, przy starcie systemu w lewym górnym rogu będziemy mieć
teraz informację: `Booting in insecure mode` :

![](/img/2020/03/049-debian-linux-secure-boot-shim-disable-insecure-mode.jpg#huge)

W logu systemowym będzie informacja, że `secureboot: Secure boot disabled` . Różne narzędzia takie
jak np. `bootctl` czy `mokutil` będą w dalszym ciągu zwracać, że Secure Boot jest włączony:

    # bootctl status | grep -i secure
    ...
      Secure Boot: enabled
    ...

    # mokutil --sb-state
    SecureBoot enabled

Jak interpretować te powyższe sprzeczne wydawać by się było ze sobą informacje? Secure Boot jest
włączony w konfiguracji EFI/UEFI i technicznie rzecz biorąc restrykcje jakie on nakłada będą w mocy
pod warunkiem, że system nie zostanie uruchomiony za pośrednictwem shim'a. Jeżeli shim będzie
pośredniczył w procesie startu systemu, to wtedy efekt jest podobny do wyłączenia Secure Boot w
konfiguracji firmware EFI/UEFI. Niemniej jednak, jeśli nie zamierzamy korzystać z Secure Boot i
opcje w firmware umożliwiają wyłączenie tego mechanizmu, to powinniśmy Secure Boot wyłączyć w
konfiguracji firmware EFI/UEFI.

## Hasło na EFI/UEFI

Zmiany w ustawieniach firmware EFI/UEFI, których dokonywaliśmy, może przeprowadzić praktycznie
każdy, kto ma dostęp fizyczny do komputera. Dlatego też przydałoby się ustawić hasło, by czasem
ktoś z marszu nie podmienił czy wgrał nam nowych kluczy, które mogą całkowicie skompromitować
bezpieczeństwo jakie oferuje Secure Boot.

![](/img/2020/03/050-debian-linux-efi-uefi-firmware-bios-configuration-supervisor-password.jpg#huge)

## Podsumowanie

I tak oto dobrnęliśmy do końca. Jak można było zauważyć, mechanizm Secure Boot jest w pełni
konfigurowalny, wliczając w to zastąpienie wbudowanych w firmware EFI/UEFI certyfikatów Microsoft i
Lenovo naszymi własnymi. Oczywiście trochę wysiłku trzeba było włożyć w ten proces, by wypracować
jakieś przyzwoite rozwiązanie, przynajmniej z mojego punktu widzenia, ale też nie wszystkie rzeczy
trzeba było robić w taki sposób jak zostały przedstawione w tym artykule. Przeciętny użytkownik
linux'a zadowoli się rozwiązaniem jakie oferuje shim podpisany kluczem Microsoft'u i będzie on w
stanie uruchomić praktycznie wszystkie dystrybucje linux'a, nawet te, które nie wspierają Secure
Boot.

Obawy co do możliwości uruchamiania linux'a na komputerach z firmware EFI/UEFI raczej nie znikną i
moim zdaniem nie powinny. Prawie dekadę temu [Microsoft zmienił swoje podejście w kwestii
wymagań][30] jakie się stawia certyfikowanym maszynom, które mają preinstalowany windows 8.
Niemniej jednak, w przypadku komputerów z windows 10 takich obostrzeń, by użytkownik był w stanie
Secure Boot wyłączyć, już nie ma. Jest za to jedynie zalecenie, by producenci sprzętu
elektronicznego taką możliwość końcowemu użytkownikowi zostawili. [Póki co nie słychać][31], aby
producenci komputerów rozważali uniemożliwienie użytkownikom wyłączenia Secure Boot. Należy też
pamiętać o shim i PreLoader.

Także moim zdaniem linux bez problemu da sobie radę i nie ma co winić samego mechanizmu Secure Boot
za dominującą pozycję Microsoft'u i ewentualny brak wsparcia alternatywnych systemów u producentów
sprzętu. Ja oczywiście zamierzam włączyć sobie Secure Boot na moim lapku ze względu na fakt, że ten
mechanizm potrafi być bardzo użyteczny i poprawia znacząco ochronę systemu operacyjnego we wczesnym
stadium uruchamiania komputera.


[1]: https://github.com/rhboot/shim
[2]: https://blog.hansenpartnership.com/linux-foundation-secure-boot-system-released/
[3]: https://wiki.ubuntu.com/UEFI/SecureBoot
[4]: https://wiki.debian.org/SecureBoot#Supported_architectures_and_packages
[5]: https://wiki.debian.org/ReproducibleBuilds
[6]: https://kernelnewbies.org/Linux_5.4#Kernel_lockdown_mode
[7]: /post/dkms-czyli-automatycznie-budowane-moduly/
[8]: https://en.wikipedia.org/wiki/Kexec
[9]: https://en.wikipedia.org/wiki/Model-specific_register
[10]: https://www.kernel.org/doc/Documentation/acpi/method-customizing.txt
[11]: https://www.kernel.org/doc/Documentation/acpi/apei/einj.txt
[12]: /post/memtest86-dla-efi-uefi-i-refind/
[13]: /post/automatyczne-podpisywanie-modulow-kernela-przez-dkms/
[14]: https://wiki.gentoo.org/wiki/Sakaki%27s_EFI_Install_Guide/Configuring_Secure_Boot
[15]: https://sourceforge.net/p/refind/code/ci/master/tree/keys/
[16]: /jak-przygotowac-dysk-pod-instalacje-debian-linux-z-efi-uefi/
[17]: https://lwn.net/Articles/750730/
[18]: https://www.rodsbooks.com/efi-bootloaders/controlling-sb.html
[19]: http://man7.org/linux/man-pages/man7/keyrings.7.html
[20]: https://wireless.wiki.kernel.org/en/developers/regulatory/wireless-regdb
[21]: https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=925550
[22]: https://dsa.debian.org/secure-boot-ca
[23]: https://wiki.archlinux.org/index.php/Secure_Boot#Microsoft_Windows
[24]: https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/windows-secure-boot-key-creation-and-management-guidance
[25]: https://elixir.bootlin.com/linux/latest/source/include/linux/security.h#L104
[26]: https://kernelnewbies.org/Linux_5.4#Security
[27]: /post/budowanie-kernela-linux-dla-konkretnej-maszyny-z-debianem/
[28]: https://blog.hansenpartnership.com/the-meaning-of-all-the-uefi-keys/
[29]: https://en.wikipedia.org/wiki/X.509
[30]: https://mjg59.dreamwidth.org/5850.html
[31]: https://www.rodsbooks.com/efi-bootloaders/secureboot.html#whatis
[32]: /post/problem-z-aktualizacja-zmiennych-pk-kek-db-dbx-efi-updatevar/
