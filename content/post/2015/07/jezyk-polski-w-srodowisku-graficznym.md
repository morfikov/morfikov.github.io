---
author: Morfik
categories:
- Linux
date: "2015-07-15T17:29:27Z"
date_gmt: 2015-07-15 15:29:27 +0200
published: true
status: publish
tags:
- locale
- openbox
GHissueID: 158
title: Język polski w środowisku graficznym
---

Gdy mamy do dyspozycji panel administracyjny jednego ze środowisk graficznych, np. GNOME, zmiana
języka interfejsu aplikacji w systemie nie powinna przysporzyć problemów. Natomiast jeśli chodzi o
wszelkie inne środowiska oparte jedynie o menadżery okien, np. OPENBOX, lub też i te zupełnie nie
mające graficznej sesji, to przestawienie języka jest lekko utrudnione. Przede wszystkim, musimy
rozróżnić dwie kwestie. Jedną z nich jest język interfejsu i wszelkie wiadomości, które wypisuje nam
system, np. na terminalu. A drugą są [polskie
znaki](/post/klawiatura-i-jej-konfiguracja-pod-debianem/), które możemy wpisywać
przy pomocy klawiatury. Te dwie rzeczy są ze sobą niepowiązane w żaden sposób, tj. można mieć polski
układ klawiszy i jednocześnie korzystać z angielskiej wersji systemu operacyjnego linux i vice
versa. Choć automaty środowisk graficznych synchronizują te dwa elementy, co wydaje się być raczej
zrozumiałe.

<!--more-->
## Kilka słów o locale

Za język w systemie odpowiadają zmienne środowiskowe ustawień regionalnych. Możemy je podejrzeć
przez wpisanie w terminalu polecenia `locale` :

    $ locale
    LANG=en_US.UTF-8
    LANGUAGE=
    LC_CTYPE=en_US.UTF-8
    LC_NUMERIC=en_US.UTF-8
    LC_TIME=en_US.UTF-8
    LC_COLLATE=C
    LC_MONETARY=en_US.UTF-8
    LC_MESSAGES=C
    LC_PAPER=en_US.UTF-8
    LC_NAME=en_US.UTF-8
    LC_ADDRESS=en_US.UTF-8
    LC_TELEPHONE=en_US.UTF-8
    LC_MEASUREMENT=en_US.UTF-8
    LC_IDENTIFICATION=en_US.UTF-8
    LC_ALL=

Zmienna `LANG` ustawia domyślne wartości dla lokalizacji w systemie i wszystkie zmienne `LC_` będą
automatycznie przepisane na wartość podanej w zmiennej `LANG` . Jeśli któraś ze zmiennych `LC_`
zostanie ustawiona osobno, wtedy ta wartość będzie honorowana. Nie zaleca się ustawiania za to
zmiennej `LC_ALL` , bo ona zawsze nadpisuje wartości, które zostały zdefiniowane w zmiennych `LANG`
i `LC_` . Przydaje się ona jedynie w celach testowych. Z kolei zmienna `LC_COLLATE` odpowiada za
domyślne ustawienia sortowania, np. wyników zwracanych przez polecenie `ls` . Jeśli ustawimy tę
zmienną na `C`, wtedy listowane najpierw będą pliki zawierające z przodu kropkę, tj. pliki ukryte,
następne będą pliki, których nazwy zaczynają się od wielkich liter i na końcu te zaczynające się z
małych liter. Problematyczna może być zmienna `LC_TIME` , która odpowiada za ustawienie formatu daty
i czasu. Ja bardzo lubię datę w formacie [ISO-8601](https://pl.wikipedia.org/wiki/ISO_8601), tj.
YYYY-MM-DD ale domyślne ustawienia języka en_US.UTF-8 preferują datę w zapisie MM/DD/YYYY . I tu
mamy właśnie ukazaną wyższość ręcznego dostosowywania lokalizacji nad wszelkimi rodzajami automatów,
bo możemy wpłynąć na format danych, które będą nam prezentowane przez interfejsy aplikacji.

## Jak ustawić język polski

Domyślnym językiem instalacji linuxa jest angielski. Jeśli zapomnieliśmy go zmienić, to teraz jest
najwyższy czas by to zrobić. Do zmiany ustawień języka na debianie, czy w ogóle na linuxie,
wykorzystuje się pakiet `locales` , a konkretnie jedno z jego narzędzi, `locale-gen` . Generuje ono
plik `/usr/lib/locale/locale-archive` zawierający archiwum lokalizacji w oparciu o to, co znajduje
się w pliku `/etc/locale.gen` . Im więcej lokalizacji zaznaczymy, tym większe będzie archiwum.

Plik `/etc/locale.gen` możemy edytować na dwa sposoby. Pierwszym z nich jest metoda ręczna
polegająca na usunięciu znaczka komentarza `#` z początku linii przy pożądanych pozycjach. Inny
sposobem jest skorzystanie z debianowego narzędzia `dpkg-reconfigure` . Nie ma znaczenia, którą
opcję wybierzemy, bo i tak po dokonaniu edycji pliku baza locale zostanie wygenerowana w dokładnie
ten sam sposób.

Ja jednak uważam, że lepiej jest skorzystać z `dpkg-reconfigure` by uniknąć ewentualnych literówek.
Odpalamy zatem terminal i logujemy się na konto root, po czym wydajemy poniższe polecenie:

    # dpkg-reconfigure locales

Naszym oczom powinien się ukazać następujący komunikat:

![](/img/2015/06/1.linux-jezyk-polski.png#huge)

Mamy tam informację, że standardowo powinniśmy zaznaczyć pozycje, które mają kodowanie UTF-8 ale o
tym za moment. Klikamy `OK` i teraz już powinniśmy mieć dostęp do wyboru lokalizacji:

![](/img/2015/06/2.linux-jezyk-polski.png#huge)

Jak widzimy wyżej mamy dwie pozycje odpowiadające za język polski, zgodnie ze wcześniejszą
instrukcją zaznaczamy jedynie tę przy której stoi `UTF-8` . Mi nigdy ta druga pozycja nie była do
niczego potrzebna przez te wszystkie lata, które spędziłem na linuxie. Kodowanie zawsze było takie
jak trzeba i nie miałem problemów z wyświetlaniem polskich znaków.

Język polski to nie jedyna opcja, którą powinniśmy zaznaczyć w okienku wyżej. Dobrze jest także
dodać `en_US` (również UTF-8) , bo to jest standardowy język interfejsów aplikacji w linuxie i
jeśli jakaś z nich nie została przetłumaczona na język polski, to wtedy ten locale zostanie użyty.

Jako, że mamy co najmniej dwie różne lokalizacje, to trzeba wskazać domyślną dla całego systemu:

![](/img/2015/06/3.linux-jezyk-polski.png#huge)

Ten krok spowoduje utworzenie pliku `/etc/default/locale` z ustawioną zmienną `LANG` na
`pl_PL.UTF-8` .

Po konfiguracji powinno zostać wygenerowane również i archiwum:

![](/img/2015/06/4.linux-jezyk-polski.png#huge)

Język polski powinien być dostępny po ponownym uruchomieniu komputera. Powyższe kroki jednak nie
wszędzie ustawią odpowiednie kodowanie znaków, mianowicie pod TTY będą problemy z ich wyświetlaniem
ale to zagadnienie na inny wpis.
