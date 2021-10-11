---
author: Morfik
categories:
- Blog
date: "2016-08-16T09:21:00Z"
date_gmt: 2016-08-16 07:21:00 +0200
published: true
status: publish
tags:
- wordpress
- widżet
- tagi
GHissueID: 555
title: 'WordPress: Jak zwiększyć ilość elementów w chmurze tagów'
---

Po drobnym dostosowaniu tego bloga, zmieniła się też nieco jego stopka. Problem w tym, że z początku
widżety ulokowane na samym dole strony niezbyt współgrały ze sobą wizualnie. Generalnie rzecz
biorąc, w stopce mam trzy widżety: ostatnie recenzje, archiwum i chmurę tagów. Można co prawda
zmieniać ich kolejność ale z racji że są tylko trzy widżety, to nie idzie ich dobrać tak, by były
przyzwoicie wyrównane. Winne są tutaj same tagi, które WordPress domyślnie wyświetla w liczbie 45.
By te widżety wyglądały przyzwoicie, trzeba by tych tagów dodać kilka, np. tak ze 20 sztuk ale jak
to zrobić? Możemy skorzystać z funkcji `wp_tag_cloud()` .

<!--more-->
## WordPress i jego chmura tagów

Widżet chmury tagów w WordPress'ie wykorzystuje [funkcję
wp\_tag\_cloud()](https://developer.wordpress.org/reference/functions/wp_tag_cloud/) do wyświetlenia
i formatowania poszczególnych elementów chmury na blogu. Jeśli zajrzymy w ten powyższy link, to tam
niżej na stronie mamy odwołanie do kodu WordPress'a, a konkretnie do pliku
`wp-includes/category-template.php` . Tam z kolei mamy szereg argumentów, które wpływają na
prezentowanie nam zawartości widżetu chmury. Jako, że celem tego wpisu jest jedynie zwiększenie
ilości jej elementów, to interesuje nas `'number' => 45` . Jak widzimy, domyślna wartość, to 45,
zatem taka liczba tagów zostanie nam zwrócona w stopce, czy też w bocznym panelu (sidebar), w
zależności od tego, gdzie tę chmurę umieściliśmy.

Oczywiście nie będziemy edytować pliku `wp-includes/category-template.php` , tylko dodamy jedną
funkcję do pliku `functions.php` wykorzystywanego przez nasz blog motywu. Funkcja, którą musimy
dodać jest poniżej:

    function wpse_235908_tag_cloud_args( $args ) {
        $args['number'] = 70;
        return $args;
    }
    add_filter( 'widget_tag_cloud_args', 'wpse_235908_tag_cloud_args' );

W tym przypadku ustawiliśmy sobie 70 tagów. Oczywiście możemy tutaj podać dowolną wartość, tylko
trzeba pamiętać o faktycznej liczbie elementów, którymi nasz blog dysponuje. Nic się jednak nie
stanie, gdy określona przez nas wartość będzie większa niż faktyczna ilość tagów.
