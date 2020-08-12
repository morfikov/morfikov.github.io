---
author: Morfik
categories:
- WordPress
date: "2015-05-23T11:55:52Z"
date_gmt: 2015-05-23 09:55:52 +0200
published: true
status: publish
tags:
- blog
title: 'WordPress: "Briefly unavailable for scheduled maintenance"'
---

Prawdopodobnie przyjdzie nam kiedyś zobaczyć komunikat `Briefly unavailable for scheduled
maintenance. Check back in a minute.` podczas przeglądania zawartości naszej strony www czy też
bloga opartego na skrypcie WordPressa. Ja początkowo myślałem, że aktualizacja pluginów (lub też i
samego WordPressa) zajmuje dłużej niż zwykle ale okazało się jednak, że sam update zakończył się
powodzeniem i to z grubsza od razu. Niemniej jednak, ten komunikat widniał na stronie głównej mojego
testowego bloga i nie szło wbić nawet do panelu admina by coś w tej sytuacji zaradzić.

<!--more-->
## Plik .maintenance

Poszukałem trochę na necie i w [FAQ][1] WordPressa znalazłem informacje dotyczące pewnego pliku
`.maintenance` . Okazuje się, że WordPress tworzy ów plik w głównym katalogu strony podczas procesu
aktualizacji, przy czym, w linku podają, że automatycznych, a nie przy ręcznej próbie, tak jak to
miało miejsce w moim przypadku.

Tak czy inaczej zajrzałem do tego pliku `.maintenance` i ma on z grubsza poniższą strukturę:

    <?php $upgrading = 1432303413; ?>

W zmiennej `$upgrading` jest zapisany czas w formacie unixowym i w moim przypadku było to
`1432303413` , co przekłada się na datę `2015-05-22 16:03:33` . Do samego przeliczania czasu
unixowego na "ludzki" można wykorzystać narzędzie `date` :

    $ date --rfc-3339=seconds -d '@1432303413'
    2015-05-22 16:03:33+02:00

oraz w drugą stronę:

    $ date -d "2015-05-22 16:03:33+02:00" +%s
    1432303413

Niezbyt to wygodne ale nie trzeba sobie tym głowy zawracać, bo przecie istnieją pluginy zajmujące
się trybem maintenance i ja nie będę tutaj opisywał ani wtyczek ani samego trybu. Tak czy inaczej,
może się zdarzyć, że plik `.maintenance` nie zostanie usunięty po aktualizacji WordPressa czy też
jego wtyczek/motywów i by pozbyć się tytułowego komunikatu ze strony głównej i odblokować blog,
trzeba zwyczajnie usunąć wspomniany plik ręcznie.

[1]: https://wordpress.org/support/article/faq-troubleshooting/#how-to-clear-the-briefly-unavailable-for-scheduled-maintenance-message-after-doing-automatic-upgrade
