---
author: Morfik
categories:
- Linux
date: "2015-06-15T15:20:52Z"
date_gmt: 2015-06-15 13:20:52 +0200
published: true
status: publish
tags:
- cd
- dvd
- iso
- pliki
title: Suma kontrolna nagranego obrazu .iso
---

Suma kontrolna daje możliwość sprawdzenia czy dane zawarte w pliku nie zostały zmienione podczas
transferu z jednego medium informacyjnego na inne. Jeśli weźmiemy przykład pakietów sieciowych, to
pliki przesyłane między dwoma punktami są dzielone na mniejsze kawałki. W każdym z nich są zawarte
sumy kontrole danych, które zawierają. Komputer odbierający taki pakiet generuje własną sumę
kontrolą i porównuje ją z tą otrzymaną w pakiecie. W przypadku gdy suma kontrolna się nie zgadza,
mamy do czynienia z błędami przesyłu, tj. pakiet został uszkodzony gdzieś po drodze. W tej sytuacji
maszyna odbierająca dane prosi o ponowne przesłanie uszkodzonego segmentu. Ta sytuacja może się
zdarzyć ale błędy są automatycznie naprawiane. Problem jest taki, że przy pobieraniu plików z
internetu nie zawsze korzystamy z bezpiecznych połączeń, poza tym, zawsze ktoś może uzyskać dostęp
do serwera i podmienić pliki. Czy możemy zatem mieć pewność, że dany plik trafił do nas w formie
takiej jak powinien?

<!--more-->
## Serwery lustrzane i sieć P2P

Poza mainstreamowym źródłem pobierania plików, mamy także całą gamę innych miejsc, z których możemy
pozyskać obraz płytki instalacyjnej czy systemu live. Mogą to być serwery lustrzane hostujące
określone pliki ale czy kiedykolwiek weryfikowaliśmy taki mirror? Czy wiemy kto za nim stoi i kim
tak naprawdę jest ta osoba? Innym miejscem może być sieć P2P ale czy pliki w niej są bezpieczne?
Wystarczy przeszukać na jakimś trackerze bazę za słowem `debian` i zostanie nam zwrócony szereg
wyników. Który obraz pobrać zapytacie?

Te wszystkie powyższe pytania tak naprawdę nie mają większego znaczenia. Załóżmy, że poszukujemy
obrazu instalacyjnego debiana. Co po pobraniu pliku należałoby zrobić, by mieć pewność, że jest on
dokładnie taki jak oczekujemy? Trzeba sprawdzić jak wygląda jego suma kontrolna i nie ma znaczenia
tutaj miejsce, z którego plik został pobrany. Jeśli suma kontrolna będzie zgodna z tą dostarczoną
przez, w tym wypadku, deweloperów debiana, możemy spać spokojnie.

## Wiarygodna suma kontrolna

Będąc w posiadaniu obrazu `.iso` , przechodzimy na [serwer
debiana](https://www.debian.org/CD/http-ftp/) i szukamy płytki, którą rzekomo mamy na dysku. Zwykle
obok obrazów mamy podane pliki o nazwach: `MD5SUMS` , `SHA1SUMS` , `SHA256SUMS` , `SHA512SUMS` . Co
oznaczają nazwy plików i jaka jest ich zawartość? Nazwy plików zawierają algorytm za pomocą którego
zostały wygenerowane sumy kontrolne obrazów.

Tak wygląda przykładowy plik `SHA1SUMS` :

    018ac2307ca33f021ee33ac9e26f05c7a47eb0e2  debian-8.1.0-amd64-DVD-1.iso
    fd96c9f95d5a5c610981d157bc163442f82a158f  debian-8.1.0-amd64-DVD-10.iso
    04cdd65fdfe45bb9102e1dce81a160f8cbf8e059  debian-8.1.0-amd64-DVD-11.iso
    a4bc0f3f6bd6e3459d98197bb74016d6c406028c  debian-8.1.0-amd64-DVD-12.iso
    f526bf8b5e132836747d738529b4168eca2103dc  debian-8.1.0-amd64-DVD-13.iso
    48f1e0dc85eba978b5b834104747e3d6fef59bc0  debian-8.1.0-amd64-DVD-2.iso
    5c64e52bf8722072117dce090db49a32af957cad  debian-8.1.0-amd64-DVD-3.iso
    667c1b7baa4b1b45086dcf3bd023a6315ce71018  debian-8.1.0-amd64-DVD-4.iso
    137857851607845fd35966d8872d552adc6c5af1  debian-8.1.0-amd64-DVD-5.iso
    62695c91ca0fb897c5583efc6019ddb89b646739  debian-8.1.0-amd64-DVD-6.iso
    1e0a89f25a1d4ccddf9f31862c7e1ec21d0f67ac  debian-8.1.0-amd64-DVD-7.iso
    8044bc98277107f6823241c8d3654b800224043e  debian-8.1.0-amd64-DVD-8.iso
    c293700d004f8745f84b38bba6f552d9f7633fb7  debian-8.1.0-amd64-DVD-9.iso
    6260e63c93af2cc112f1d9a7143f98b872f52ad5  debian-update-8.1.0-amd64-DVD-1.iso

Lewa kolumna zawiera sumy kontrolne wygenerowane za pomocą narzędzia `sha1sum` , prawa zaś
odpowiadające im pliki.

Mamy także inny plik o bardzo podobnej nazwie `SHA1SUMS.sign` . Jest to sygnatura stojąca na straży
bezpieczeństwa pliku sum kontrolnych. Musimy zweryfikować sygnaturę i sprawdzić czy integralność
pliku z sumami nie została zmieniona. Pobieramy zatem oba pliki na dysk i weryfikujemy sumę:

    $ gpg --verify SHA1SUMS.sign
    gpg: assuming signed data in `SHA1SUMS'
    gpg: Signature made Mon 08 Jun 2015 12:32:24 AM CEST
    gpg:                using RSA key 0xDA87E80D6294BE9B
    gpg: Good signature from "Debian CD signing key " [unknown]
    gpg: WARNING: This key is not certified with a trusted signature!
    gpg:          There is no indication that the signature belongs to the owner.
    Primary key fingerprint: DF9B 9C49 EAA9 2984 3258  9D76 DA87 E80D 6294 BE9B

Informacja o `Good signature` , świadczy, że dane w pliku nie zostały zmienione.

Ok, to mamy zatem punkt oparcia -- sumę kontrolną. Jak teraz sprawdzić czy pobrany przez nas obraz
`.iso` posiada dokładnie taką samą sumę kontrolną, która widnieje na serwerze? Najprościej jest
odpalić `sha1sum`, podając w argumencie ścieżkę do obrazu. W tym przypadku zostanie wyrzucony ciąg
znaków, który następnie trzeba porównać z tym na stronie. Trochę to niezbyt praktyczne. Ja zwykle
porównywałem dwa pierwsze i dwa ostanie znaki. Jeśli się zgadzały, zakładałem, że suma kontrolna się
zgadza. Innym rozwiązaniem jest wrzucenie pliku z sumami ( `SHA1SUMS` ) do folderu z plikiem
`.iso` . Teraz już wystarczy wydać poniższe polecenie:

    $ sha1sum -c SHA1SUMS
    debian-live-8.1.0-amd64-cinnamon-desktop.iso: OK

W przypadku gdyby suma się nie zgadzała, zostanie wyrzucony stosowny komunikat.

    $ sha1sum -c SHA1SUMS
    debian-live-8.1.0-amd64-cinnamon-desktop.iso: FAILED
    sha1sum: WARNING: 1 computed checksum did NOT match

Gdy w pliku `SHA1SUMS` jest zdefiniowanych więcej niż jedna suma kontrolna, dostaniemy sporo błędów.
Dlatego też ze względów estetycznych najlepiej posiadać w pliku wpis od określonego obrazu lub
obrazów. Istnieje również [skrypt](https://people.debian.org/~danchev/debian-iso/check_debian_iso),
którym można się posłużyć.

Dla tych, którzy nie przepadają za konsolą są i graficzne narzędzia. Mi osobiście bardzo przypadł do
gustu [wxHexEditor](http://www.wxhexeditor.org/). Co w nim takiego ciekawego? Potrafi liczyć zarazem
kilkadziesiąt sum:

![](/img/2015/06/1.suma-kontrolna-wxhexeditor-lista.png#big)

Wystarczy wskazać mu plik, po czym wybrać interesujące nas algorytmy, a ten program nam to wszystko
ładnie wyliczy:

![](/img/2015/06/2.wynikowa-suma-kontrolna.png#huge)

## Suma kontrolna wypalonego obrazu ISO

Większość z nas pobiera obrazy płyt by je wypalić na cd/dvd tudzież pendrive. Jak by nie patrzeć, w
procesie przenoszenia takiego obrazu z dysku również mogą pojawić się jakieś błędy. W pierwszej
chwili możemy dość do wniosku, że sprawdzenie sumy kontrolnej zawartości płytki jest niemożliwe.
Przecie są tam setki plików, a nie jeden plik jak w przypadku obrazu `.iso` . Fakt, lecz to tylko z
naszej, ludzkiej perspektywy tak wygląda. Dla maszyny jest to zwyczajnie ciąg znaków i jej wszystko
jedno czy ma do czynienia z danymi w obrazie na dysku czy danymi na płycie cd/dvd. Jak zatem
sprawdzić czy wypalony obraz nie zawiera błędów?

Trzeba zapamiętać dwie rzeczy. Pierwsza, by poddawać sprawdzeniu dane raw, a druga to fakt, że obraz
ma określony rozmiar, w bajtach. A płytki cd/dvd mieszczą z reguły więcej danych niż obrazy `.iso` ,
co jest logiczne, gdyż w przeciwnym wypadku, taki obraz by się na płytę po prostu nie zmieścił.

Najprostszym sposobem na sprawdzenie integralności danych na płycie jest nakarmienie programu
`sha1sum` danymi z urządzenia `/dev/sr0` , z tym, że w takim przypadku, zostałaby wygenerowana suma
kontrola całego nośnika, czyli wypalonego obrazu `.iso` z danymi oraz pustego miejsca (za końcem
obrazu). W wyniku czego uzyskalibyśmy inną sumę kontrolną. Musimy zatem wskazać miejsce, w którym ma
się skończyć liczenie sumy kontrolnej nośnika. Jak to zrobić? Sprawdźmy przy pomocy `ls -al` jaki
rozmiar w bajtach ma pobrany obraz ` :iso` :

    s -al debian-live-8.1.0-amd64-cinnamon-desktop.iso
    -rw-r--r-- 1 morfik morfik 1159495680 2015-06-15 14:49:18 debian-live-8.1.0-amd64-cinnamon-desktop.iso

Mamy zatem rozmiar: `1159495680` . Co dalej? Musimy nakarmić `sha1sum` danymi z `/dev/sr0` o takiej
długości. Posłuży nam do tego stary dobry `dd` :

    $ dd if=/dev/sr0 bs=1 count=1159495680 | sha1sum

Trzeba przy tym zauważyć by zwiększyć odpowiednio parametr `bs` , gdyż przy wartości `1` ,
sprawdzanie sumy kontrolnej potrwa bardzo długo. Przy obrazach `.iso` można zastosować wielokrotność
`bs=2048` . Przy czym trzeba pamiętać by liczbę bajtów podzielić przez wartość parametru `bs` . W
tym przypadku będzie to 1159495680/2048=566160 . Odpalamy `dd` z parametrami `bs=2048` i
`count=141312` . Po chwili suma kontrolna zostanie wygenerowana. Porównujemy ją z sumą kontrolną
obrazu, jeżeli się zgadzają, to wszystko jest w porządku. W przypadku gdyby sumy się nie zgadzały,
została naruszona integralność danych na nośniku i trzeba płytkę wypalać jeszcze raz.
