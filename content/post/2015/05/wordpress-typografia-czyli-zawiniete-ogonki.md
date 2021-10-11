---
author: Morfik
categories:
- WordPress
date: "2015-05-28T14:59:54Z"
date_gmt: 2015-05-28 12:59:54 +0200
published: true
status: publish
tags:
- blog
GHissueID: 223
title: 'WordPress: Typografia, czyli zawinięte ogonki'
---

Typografia ma na celu wprowadzenie pewnego rodzaju urozmaiceń do pisanego tekstu, takich jak np.
bardziej zawinięte ogonki w cudzysłowach, albo zmianę ich pozycji (lewy dolny róg, zamiast górnego),
czy też mniejszy odstęp między trzema kropkami lub ewentualnie łączenie kilku myślników w jeden
dłuższy. W WordPressie ten mechanizm automatycznie będzie przepisywał nam te powyższe znaki (i
wiele innych), co w przypadku systemów linuxowych nie zawsze jest rzeczą pożądaną, a to z tego
względu, że np. skrypty shellowe, czy polecenia systemowe, korzystają jedynie z prostych apostrofów
`'` oraz prostych cudzysłowów `"` i wszelkie inne odpowiedniki, to inny numerek w tablicy kodowej, a
więc i inny znak. Wobec czego, tak skopiowany tekst (czy skrypt) będzie zawierał błędy składni.

<!--more-->
## Jak wyłączyć typografię

Jeśli mamy zamiar otworzyć bloga o tematyce informatycznej, to nie zastanawiajmy się nawet czy
powinniśmy wyłączyć ten ficzer typograficzny, tylko zróbmy to od razu po instalacji WordPressa. W
przeciwnym wypadku, każdy wpis, który powędruje do bazy danych, będzie zawierał przepisane znaki i
trzeba będzie je później poprawić.

[By wyłączyć całkowicie moduł
typografi](https://codex.wordpress.org/Plugin_API/Filter_Reference/run_wptexturize), musimy
wyedytować jeden plik w motywie Wordpressa, tj.`wp-content/themes/nazwa_theme/functions.php` i
dopisać tam poniższy kod:

    add_filter( 'run_wptexturize', '__return_false' );

Możemy także wyłączyć typografię w pewnych określonych miejscach na blogu, np. w tytule postu, czy
też w jego treści albo nawet i komentarzach.

    // Disable wptexturize
    remove_filter('the_title', 'wptexturize');
    remove_filter('comment_text', 'wptexturize');
    remove_filter('the_excerpt', 'wptexturize');
    remove_filter('the_content', 'wptexturize');

Spowoduje to usunięcie przepisywania znaków w formularzach na blogu i od tej pory nie ważne ile
`---` wstawimy sobie, to zawsze będzie ich tyle ile wpisaliśmy. Dokładny opis zasady działania
funkcji `wptexturize` można znaleźć
[tutaj](https://codex.wordpress.org/Function_Reference/wptexturize).
