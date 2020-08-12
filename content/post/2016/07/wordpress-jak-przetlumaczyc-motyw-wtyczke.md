---
author: Morfik
categories:
- WordPress
date: "2016-07-27T20:00:04Z"
date_gmt: 2016-07-27 18:00:04 +0200
published: true
status: publish
tags:
- gettext
- blog
- php
title: 'WordPress: Jak przetłumaczyć motyw/wtyczkę'
---

WordPress został przetłumaczony na dość sporo języków, w tym też i na język polski. Niemniej jednak,
pliki bazowe to nie to samo co pliki różnych dodatków. Dlatego też czasem po zmianie języka na
polski, nie wszystkie elementy naszego bloga są przetłumaczone. Nie ma przy tym znaczenia czy
[ustawialiśmy język podczas instalacji
WordPress'a]({{< baseurl >}}/post/wordpress-domyslny-jezyk-instalacji/), czy też później z poziomu
panela administracyjnego. Taki stan rzeczy nie wygląda zbyt estetycznie i przydałoby się coś z tym
zrobić. Jeśli zajrzymy do katalogu wtyczek czy motywów, to zwykle znajdziemy tam pliki `.mo` oraz
`.po` , które są używane przy tłumaczeniu tekstu z wykorzystaniem
[gettext](https://www.gnu.org/software/gettext/). Jako, że motyw, który jest wykorzystywany na tym
blogu nie jest przetłumaczony, to postanowiłem go przetłumaczyć i przy okazji opisać ten niezbyt
skomplikowany proces.

<!--more-->
## Pliki .mo i .po

By zrozumieć różnicę między tymi plikami `.mo` i `.po` , trzeba zagłębić się nieco w proces
translacji tekstu w WordPress'ie. Najprościej jest to wytłumaczyć na przykładzie standardowych
plików językowych, które [można pobrać bezpośrednio ze strony
WordPress'a](https://downloads.wordpress.org/translation/core/4.4/pl_PL.zip). W paczce mamy kilka
par plików MO/PO. One zawsze występują w parach. Skróty oznaczają: Machine Object (MO) oraz Portable
Object (PO). Różnica między nimi jest taka, że ten drugi jest w formie zrozumiałej dla człowieka i
ten plik będziemy poddawać edycji. Następnie plik `.po` będzie kompilowany do postaci binarnej,
czyli otrzymamy plik `.mo` , który zostanie wykorzystywany przez WordPress w [procesie tłumaczenia
tekstu](https://codex.wordpress.org/I18n_for_WordPress_Developers).

Pliki `.po` można tworzyć i edytować ręcznie. Niemniej jednak, trzeba mieć instrukcję, w oparciu o
którą będziemy wiedzieć jaki tekst w ogóle podlega tłumaczeniu. Nie możemy sobie po prostu od tak
napisać, że dana fraza ma w języku polskim być zapisana w jakiś konkretny sposób. Ta instrukcja
zawarta jest zwykle w pliku `xx_XX.pot` , który jest dostarczany w paczce z motywem czy wtyczką.
Jest to zwykły plik tekstowy, który zawiera wpisy podobne do tego poniżej:

    #: ../functions/init.php:67
    msgid "Footer"
    msgstr ""

Wystarczy ten plik skopiować i nazwać odpowiednio. Na warunki polskie jest to `pl_PL.po` . Wszędzie
tam, gdzie mamy `msgstr` , musimy wpisać polski odpowiednik wiadomości, która figuruje w `msgid` . Z
kolei komentarz w pierwszej linijce informuje nas, gdzie względem pliku tłumaczenia znajduje plik,
który zawiera powyższą frazę. Zatem w tym przypadku jest to plik `theme/functions/init.php` , a
liczba `67` orientacyjnie wskazuje nam linijkę, w której ta fraza figuruje. Komentarz pełni rolę
jedynie informacyjną i jego obecność nie wpływa na przebieg tłumaczenia.

## Narzędzie poedit

Nie musimy ręcznie tworzyć i edytować pliku `.po` , czy też kompilować pliku binarnego `.mo` przy
pomocy narzędzi dostarczanych w pakiecie `gettext` . Mamy do dyspozycji lepszą alternatywę w postaci
pakietu [poledit](https://poedit.net/) . W nim znajduje się graficzny edytor plików, który pomoże
nam się uporać z tłumaczeniem. Odpalamy zatem terminal i wpisujemy w nim `poledit` . W okienku,
które nam wyskoczy, przechodzimy do menu, z którego to wybieramy File \> From POT/PO File.
Wskazujemy lokalizację pliku `.pot` lub `.po` . Po chwili plik powinien zostać załadowany, a my
poproszeni o wybór
języka:

![]({{< baseurl >}}/img/2016/07/1.poedit-tlumaczenie-theme-plugin-wordpress-gettext.png)

Teraz wystarczy już uzupełnić odpowiednio prawą kolumnę w oparciu o tekst, który widnieje w
lewej:

![]({{< baseurl >}}/img/2016/07/2.poedit-tlumaczenie-theme-plugin-wordpress-gettext.png)

Gdy skończymy, zapisujemy. Oba pliki zostaną utworzone, a WordPress automatycznie dobierze sobie
nazwę pliku binarnego `.mo` w oparciu o ustawienia językowe określone w panelu administracyjnym.

## Jak przetłumaczyć niepełny projekt

Wtyczki i motywy mają do siebie to, że nie zawsze są one w pełni dostosowane pod kątem tłumaczenia.
Chodzi generalnie o kod PHP. Jeśli ten kod dostarczony nam nie zawiera odpowiednich wywołań, to bez
jego poprawy nie damy rady przetłumaczyć danego projektu. Na szczęście edycja kodu PHP w kwestii
dostosowania na potrzeby tłumaczenia nie jest rzeczą trudną. Musimy tylko w pliku motywu czy wtyczki
znaleźć tę część kodu, która odpowiada za wypisanie konkretnej frazy na stronie bloga. Dla przykładu
weźmy sobie ten poniższy kod:

    if ( is_day() ) {
        echo "<strong>";
        printf( 'Daily Archives: ' . get_the_date() );
        echo "</strong>";
    }

Wypisze on statyczny tekst `Daily Archives:` i dołączy do niego to co zostanie zwrócone przez
`get_the_date()` . Ten tekst będzie po angielsku bez względu na to jaki język ustawimy w
WordPress'ie. Jeśli chcemy, by podlegał on tłumaczeniu, to [musimy go
owinąć](https://codex.wordpress.org/I18n_for_WordPress_Developers#Strings_for_Translation) w `__(
)` . Powinno to wyglądać mniej więcej tak:

    if ( is_day() ) {
        echo "<strong>";
        printf( __( 'Daily Archives: %s', 'theme' ), get_the_date() );
        echo "</strong>";
    }

Fraza `theme` jest dowolna. Jest to tzw. Text Domain, która w obrębie kodu danego motywu czy wtyczki
powinna zostać taka sama. Zatem jeśli musimy przetłumaczyć jakieś dodatkowe frazy, to upewnijmy się,
że ta domena się zgadza z tą już wykorzystaną w kodzie.

Teraz możemy włączyć tę frazę do pliku `.pot` lub `.po` , z tym, że nie wiem czy da radę to zrobić
przy pomocy `poedit` . U mnie zwyczajnie po przeskanowaniu katalogów nic nie znajduje. Pozostaje mi
zatem ręczne dodanie określonej frazy:

    msgid "Daily Archives: %s"
    msgstr "Archiwum z dnia: %s"

W przypadku, gdy tłumaczona fraza zawiera na końcu spację, ją również musi uwzględnić.
