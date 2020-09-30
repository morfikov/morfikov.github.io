---
author: Morfik
categories:
- Android
date: "2016-09-24T00:53:01Z"
date_gmt: 2016-09-23 22:53:01 +0200
published: true
status: publish
tags:
- smartfon
- kontakty
title: Jak przenieść kontakty ze starego telefonu do smartfona
---

Przy zmianie telefonu, zwłaszcza z tych nieco starszych na te nowsze smartfony, może pojawić się
problem z przeniesieniem listy kontaktów. Jeśli nasz stary telefon ma slot na kartę SD, to kontakty
zapisane w pamięci tego telefonu można zrzucić na kartę SD i na nowym telefonie z niej zaimportować.
Co jednak w przypadku, gdy nasz stary telefon nie dysponuje takim slotem, a my na liście kontaktów
mamy zapisanych ze 100-200 pozycji albo nawet i więcej? Nie będziemy raczej przepisywać każdego
kontaktu ręcznie, bo trochę się nam z tym zejdzie. Możemy za to nieco inaczej podejść do procesu
przenoszenia kontaktów. Najprościej to wykonać backup listy kontaktów starego telefonu do pliku i
zapisać go na komputerze. Oczywiście zwykle jest do tego celu potrzebny przewód USB, bluetooth czy
IrDA. Na linux'ach będzie nam także niezbędne [narzędzie Wammu](), które umożliwi nam zrobienie
takiego backupu.

<!--more-->
## Jak połączyć się do telefonu przez Wammu

Niezbędne nam oprogramowanie znajduje się w Debianie w pakiecie `wammu` . Instalujemy ten pakiet i
uruchamiamy aplikację. W tym przypadku stary telefon jest podłączony do komputera za pomocą przewodu
USB. Po podpięciu przewodu, w logu systemowym powinniśmy zobaczyć komunikaty podobne do tych
poniżej:

    ...
    cdc_acm 2-1.3:1.1: ttyACM0: USB ACM device
    cdc_acm 2-1.3:1.3: ttyACM1: USB ACM device
    ...

Wyżej mamy dwa [interfejsy
ACM](https://rfc1149.net/blog/2013/03/05/what-is-the-difference-between-devttyusbx-and-devttyacmx/).
To przy ich pomocy `wammu` będzie w stanie nawiązać komunikację z telefonem. [Konfiguracja wammu
jest opisana w osobnym artykule](/post/wysylanie-odbieranie-sms-w-wammu/). My tutaj
jedynie połączymy się z telefonem wykorzystując interfejs `/dev/ttyACM0` :

![](/img/2016/09/1.wammu-podlaczenie-stary-telefon-linux.png#big)

## Jak wykonać backup listy kontaktów w Wammu

Po nawiązaniu połączenia z telefonem, pobieramy listę kontaktów. Do wyboru mamy trzy opcje: kontakty
SIM, kontakty z pamięci oraz oba powyższe jednocześnie. Kontakty na karcie SIM są bardzo okrojone i
one nas nie będą interesować. Dlatego też skupimy się jedynie na kontaktach z pamięci telefonu:

![](/img/2016/09/2.wammu-podlaczenie-stary-telefon-linux-backup.png#huge)

Po tym jak kontakty zostaną pobrane, na liście możemy zobaczyć szereg pozycji, które mają w polu
`Name` różne zakończenia, np. `/M` (main) czy `/W` (work) , itp. W telefonie, zwykle mamy jeden
kontakt ale możemy mu przypisać kilka numerów. Niemniej jednak, na liście kontaktów każdy numer ma
osobną pozycję i tak też one są nam prezentowane.

Problem z tym zapisem jest jeden. Przy importowaniu takich kontaktów na listę w nowym telefonie,
będziemy mieć kilka pozycji odnoszących się do pojedynczego kontaktu. Różnice będą jedynie w
zakończeniu nazwy, tj. jeden kontakt będzie miał `/M` a drugi `/W` czy jeszcze inne oznaczenie. Te
kontakty będzie można w bardzo prosty sposób scalić ale o tym później. Mając listę kontaktów,
zapisujemy ją do pliku na dysku komputera:

![](/img/2016/09/3.wammu-podlaczenie-stary-telefon-linux-backup.png#huge)

## Konwertowanie listy kontaktów do formatu vCard

Teraz już możemy odłączyć telefon od naszego PC, bo nie będzie on nam już do niczego potrzebny.
Backup listy kontaktów mamy, z tym, że nie zaimportujemy tego pliku w smartfonie. Musimy
przekonwertować [format listy do vCard](https://pl.wikipedia.org/wiki/VCard). Możemy to zrobić przy
pomocy `gammu` . Instalujemy ten pakiet i w terminalu wydajemy to poniższe polecenie:

    $ gammu convertbackup phone.backup kontakty-stary-fon.vcf

Uzyskany w taki sposób plik `.vcf` możemy teraz przesłać na smartfon do dowolnego katalogu.
Następnie na smartfoine przechodzimy na `Kontakty`. Rozwijamy menu i powinniśmy ujrzeć pozycję
`Importuj/Eksportuj` . Wskazujemy miejsce, gdzie wgraliśmy plik `.vcf` oraz miejsce, gdzie
zamierzamy zaimportować te
kontakty:

![](/img/2016/09/4.importowanie-listy-kontaktow-smartfon.png#huge)

Zostaniemy poproszeni o wskazanie pliku `.vcf` (powinien zostać automatycznie znaleziony) :

![](/img/2016/09/5.importowanie-listy-kontaktow-smartfon.png#huge)

Importowanie kontaktów chwilę potrwa, a gdy ten proces dobiegnie końca, na liście będziemy mogli
zobaczyć takie oto zdublowane pozycje:

![](/img/2016/09/6.importowanie-listy-kontaktow-smartfon.png#medium)

## Scalanie kontaktów

Jako, że na tej liście mamy szereg pozycji odnoszących się do pojedynczego kontaktu, to przydałoby
się je wszystkie scalić. W tym celu zaznaczamy pierwszą pozycję z listy i przechodzimy do edycji
kontaktu. Nadajemy mu pożądaną nazwę i z menu wybieramy `Połącz` , po czym wskazujemy drugi kontakt.
Nawet mamy listę sugerowanych pozycji, które w tym przypadku znajdują jak najbardziej zastosowanie.
Wybieramy jedną z nich i proces ponawiamy dla każdego numeru, który chcemy przypisać do danego
kontaktu.

![](/img/2016/09/7.laczenie-scalanie-kontaktow-smartfon.png#huge)

W tym przypadku powinniśmy mieć trzy telefony na jednym kontakcie. Wszystkie rzeczy związane z
każdym poszczególnym numerem możemy sobie dostosować wedle uznania. Później przy wyborze tego
kontaktu będziemy mieli dostępne trzy jego numery.

![](/img/2016/09/8.laczenie-scalanie-kontaktow-smartfon.png#medium)

W tym przypadku mamy do czynienia z pseudo numerem usług jednej z sieci GSM. Niemniej jednak, w
przypadku realnych kontaktów numery komórkowe i domowe możemy upchnąć na jednej pozycji i nie
potrzebujemy do tego celu kilku osobnych. Generalnie, samo scalenie kontaktów jest bardzo szybkie, a
jeśli chcemy sobie nieco taki kontakt dopracować, to zajmie nam to oczywiście odpowiednio więcej
czasu.
