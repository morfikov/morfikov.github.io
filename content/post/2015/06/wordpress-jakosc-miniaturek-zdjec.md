---
author: Morfik
categories:
- WordPress
date: "2015-06-19T21:23:31Z"
date_gmt: 2015-06-19 19:23:31 +0200
published: true
status: publish
tags:
- blog
title: 'WordPress: Jakość miniaturek zdjęć'
---

Przy umieszczaniu zdjęć czy też różnego rodzaju grafik na blogu, możemy zauważyć, że owe fotki nie
są umieszczane w postach w oryginalnych rozmiarach. Zamiast nich są [używane
miniaturki](https://codex.wordpress.org/Post_Thumbnails), które mają konfigurowalny rozmiar.
Naturalnie możemy nakazać WordPressowi by umieszczał obrazki w pełnym ich wymiarze, choć jednak ze
względów estetycznych lepiej zachować określone ich rozmiary i jeśli jakaś grafika wychodzi po za
nie, to powinna zostać przycięta. Taka miniaturka powinna linkować do obrazka pełnowymiarowego,
gdzie można zobaczyć więcej szczegółów. Taki jest przynajmniej schemat tego działania w WordPressie.
Problem jest jednak z jakością samych miniaturek.

<!--more-->
## Wymiary miniaturek

Zacznijmy może od określenia wymiarów miniaturek. Naturalnie możemy to zrobić przez panel
administracyjny (Settings =\> Media):

![]({{< baseurl >}}/img/2015/06/1.wordpress-jakosc-miniaturek-ustawienia.png)

Jeśli teraz wgramy jakiś obrazek i spróbujemy umieścić go na blogu, powinniśmy mieć do wyboru kilka
dodatkowych rozmiarów:

![]({{< baseurl >}}/img/2015/06/2.wordpress-jakosc-miniaturek-rozmiar.png)

Każdy z tych rozmiarów, za wyjątkiem `Full Size` prawdopodobnie będzie w dość strasznej jakości.
Naturalnie, że skoro obrazki są mniejsze, to i ich jakość nie jest taka sama jak tego
pełnowymiarowego osobnika ale w przypadku WordPressa, jakość miniaturek może czasami pozostawić
wiele do życzenia.

## Jakość miniaturek

Generalnie rzecz biorąc mamy dwa formaty obrazków, które WordPress bardzo lubi. Pierwszym z nich
jest format `.png` , drugim zaś `.jpg` . W przypadku `.png` raczej nie powinniśmy narzekać na jakość
przeskalowanego zdjęcia, natomiast jeśli chodzi o `.jpg` , to tutaj już jest ona wręcz nie do
zaakceptowania.

Prawdopodobnie bardzo wielu blogerom przeszkadzała jakość miniaturek i WordPress postanowił
zaimplementować zmienną jakość fotek, tak by każdy sobie ją mógł dostosować wedle uznania. Musimy
tylko dopisać poniższy kod do pliku `functions.php` w używanym motywie strony:

    function thumbnail_quality( $quality ) {
        return 100;
    }
    add_filter( 'jpeg_quality', 'thumbnail_quality' );
    add_filter( 'wp_editor_set_quality', 'thumbnail_quality' );

Wartość `100` oznacza stopień kompresji, a właściwie jakość. W tym przypadku 100% jakości
oryginalnego zdjęcia. Jeśli chcemy by te miniaturki wyglądały jakoś po ludzku, to ustawmy większą
wartość, choć ja uważam, że nie ma co ustawić mniej niż 100.

Powyższe filtry dotyczą jedynie formatu `.jpg` i nie mają żadnego wpływu na miniaturki generowane w
formacie `.png` . Także jeśli z jakiegoś powodu miniaturki `.png` mają słabą jakość, to trzeba
poszukać winnego w oprogramowaniu, które je generuje.

## Regeneracja miniaturek po zmianie jakości

Ostatnia rzecz, która nas czeka po zmianie stopnia kompresji miniaturek, to wygenerowanie na nowo
wszystkich obrazków. W przeciwnym wypadku, wszystkie miniaturki, które zostały do tej pory
wygenerowane, będą mieć taką jakość jaką aktualnie posiadają.

By wygenerować miniaturki wszystkich obrazków, które trafiły do katalogu `wp-content/uploads/` ,
musimy posłużyć się albo jakąś wtyczką, albo zaciągnąć do tego celu narzędzie
[wp-cli]({{< baseurl >}}/post/wordpress-instalacja-przy-pomocy-wp-cli/) . Ja korzystałem z tego
drugiego rozwiązania, bo przy odpowiedniej konfiguracji, wygenerowanie miniaturek sprowadza się do
wydania tylko tego poniższego polecenia:

    $ wp media regenerate

    Do you realy want to regenerate all images? [y/n] y
    Found 98 images to regenerate.
    Regenerated thumbnails for "obrazek_1" (ID 369).
    Regenerated thumbnails for "obrazek_2" (ID 368).
    Regenerated thumbnails for "obrazek_3" (ID 367).
    Regenerated thumbnails for "obrazek_4" (ID 359).
    ...
    Success: Finished regenerating all images.
