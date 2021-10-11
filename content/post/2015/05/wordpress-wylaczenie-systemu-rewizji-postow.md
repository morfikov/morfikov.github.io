---
author: Morfik
categories:
- WordPress
date: "2015-05-26T16:40:17Z"
date_gmt: 2015-05-26 14:40:17 +0200
published: true
status: publish
tags:
- blog
GHissueID: 234
title: 'WordPress: Wyłączenie systemu rewizji postów'
---

System rewizji postów to nic innego jak wersjowanie kolejnych zmian tworzonych artykułów za pomocą
WordPress'owego edytora. Domyślnie [ten ficzer jest
włączony](https://codex.wordpress.org/Revision_Management) i powoduje nadmierne rozrastanie się
bazy danych, a to z tego powodu, że każda drobna poprawka, to kolejna wersja artykułu, która wędruje
do bazy. Nie ma przy tym żadnego ograniczenia co do ilości przechowywanych rewizji. Tego typu
mechanizm jest w wielu przypadkach bardzo użyteczny, no bo przecie mamy podgląd wszelkich zmian
wprowadzanych do postów i dodatkowo możemy te zmiany śledzić na przestrzeni tygodni, miesięcy czy
nawet i lat.

<!--more-->
## System rewizji w praktyce

Jak mówi klasyk: "Niestety nie da się wytłumaczyć czym jest system rewizji w WordPressie, trzeba to
po prostu zobaczyć na własne oczy", także poniżej fotka, która powinna rozjaśnić wszystkie
wątpliwości:

![](/img/2015/05/1.podglad-rewizji.png#huge)

Zatem wiemy już czym jest system rewizji i jak widzimy wyżej, niczym się ten patent nie różni od
zwykłego diffa znanego choćby z systemów kontroli wersji (CVS), np. GIT. Jeśli nie chcemy wersjować
swoich postów lub robimy to poza WordPressem, to ten dodatek jest nam zwyczajnie zbędny, a kolejne
kopie artykułów zaśmiecają nam jedynie bazę danych, która to z kolei może przybrać dość spore
rozmiary, z czego 90% mogą zajmować same rewizje. Na szczęście wyjść z tej sytuacji jest kilka.

## Ograniczenie liczby rewizji

Najprostszy sposób jaki przychodzi mi do głowy by rozwiązać problem zbyt dużej liczby rewizji, to
pisanie artykułu poza edytorem WordPressa i wklejanie tam jedynie gotowych tekstów, które zawsze
będą mieć tylko jedną rewizję -- tę aktualną. W praktyce bardzo ciężkie do zrealizowania, chyba,
że kompletnie nie aktualizujemy napisanych postów i nasz blog przypomina bardziej śmietnik.

Innym sposobem na odchudzenie bazy danych jest wskazanie WordPessowi by ten przechowywał tylko pewną
ograniczoną ilość rewizji. Możemy się zabrać do tego na dwa sposoby: filter `wp_revisions_to_keep`
lub odpowiedni parametr w pliku `wp-config.php` . Jeśli ktoś jest zainteresowany implementacją
filtra, pod [tym linkiem są dokładne
instrukcje](https://codex.wordpress.org/Plugin_API/Filter_Reference/wp_revisions_to_keep) jak to
zrobić. My tutaj wybierzemy prostszy sposób, który pod względem efektu niczym się zbytnio nie różni
od metody z filtrem.

Edytujemy zatem plik `wp-config.php` i wklejamy tam poniższy kod:

    define( 'WP_POST_REVISIONS', 4 );

Cyfra `4` oznacza ilość rewizji, które zostaną zapisane w bazie. Do tego trzeba również doliczyć
jedną pozycję na automatyczny zapis, który standardowo dokonuje się co 60s. Wszystkie starsze
rewizje będą automatycznie usuwane z bazy danych.

Liczbę przechowywanych aktualnie rewizji w bazie danych możemy śledzić w każdym poście po przejściu
do jego edycji:

![](/img/2015/05/2.liczba-rewizji.png#small)

## Wyłączenie systemu rewizji

Wyłączenie systemu rewizji nie wpływa na automatyczny zapis w określonym interwale czasowym. W ten
sposób zawsze będziemy mieć jedną dodatkową kopię artykułu w bazie. Ponadto, Posty posiadające swoje
rewizje, zachowają je po wyłączeniu systemu rewizji i by pozbyć się ich z bazy danych, trzeba to
będzie zrobić ręcznie lub też przy pomocy jakiegoś pluginu WordPress'a.

Jeśli nie zadowala nas ograniczenie co do przechowywanej liczby rewizji w bazie danych, zawsze
możemy ten mechanizm wyłączyć całkowicie. Robimy to dokładnie w ten sam sposób i nawet przy pomocy
tej samej opcji. Jedyna różnica to wstawienie `0` lub `false` w miejsce `4` , przykładowo:

    define( 'WP_POST_REVISIONS', false );

Moim zdaniem sam system rewizji to bardzo przyzwoite narzędzie, zwłaszcza gdy w grę wchodzi
przywłaszczanie sobie czyjej pracy i podpisywanie się pod nią. Wtedy bez problemu możemy wyciągnąć
log rewizji i udowodnić, że to my napisaliśmy jakiś konkretny artykuł, bo mamy przecie wszystkie
jego zmiany i do tego otagowane datami. Dlatego też nie zalecam jego wyłączania.
