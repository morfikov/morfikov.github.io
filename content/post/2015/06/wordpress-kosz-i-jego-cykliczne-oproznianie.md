---
author: Morfik
categories:
- Blog
date: "2015-06-02T18:21:18Z"
date_gmt: 2015-06-02 16:21:18 +0200
published: true
status: publish
tags:
- wordpress
- kosz
GHissueID: 91
title: 'WordPress: Kosz i jego cykliczne opróżnianie'
---

Jeśli piszemy dużo na swoim blogu i mamy sporo treści, którą trzeba uaktualnić, to prawdopodobnie
często wprowadzamy poprawki polegające na usuwaniu starych wpisów i tworzeniu szkiców nowych
artykułów. Tego typu rotacja sprawia, że sporo wiadomości ląduje w koszu, z tym, że jeśli chodzi o
bazę danych, to jej jest bez różnicy czy przenieśliśmy dany post do kosza, przynajmniej do momentu
gdy faktycznie on wyleci z bazy danych. Także pod względem objętości bazy nic się nie zmienia. Poza
tym, WordPress potrafi opróżniać kosz co pewien czas i dla jednych ten interwał może być za krótki,
a dla innych za długi. Postaramy się więc go dostosować.

<!--more-->
## Automatyczne kasowanie plików w koszu

Przede wszystkim, każda strona, post czy załączniki mają opcję przeniesienia do kosza, co wygląda
mniej więcej tak:

![wordpress-kosz](/img/2015/06/1.wordpress-kosz.png#small)

Po tej czynności, wszystkie posty, które usuniemy, trafią do koszta. To jest swojego rodzaju
zabezpieczenie mające ochraniać nas przed przypadkowym skasowaniem sobie ważnego kontentu. W każdym
razie, jeśli po skasowaniu postu się rozmyślimy, zawsze możemy go przywrócić i trafi on w miejsce
gdzie był przed skasowaniem. Jeśli jednak pozostawimy te posty w koszu samym sobie, po czasie `30`
dni od skasowania, zostaną one permanentnie usunięte z bazy danych i już w żaden sposób nie uda się
nam ich przywrócić.

Czy 30 dni to wystarczająca ilość czasu by zastanowić się i przemyśleć swoje czyny? Dla jednych to
za mało dla innych za dużo. Niektórzy chcieliby wyłączenia opcji kosza zupełnie, z tym, że połowa z
nich chciała by aby posty po skasowaniu od razu były wywalane z bazy, a pozostała połowa, by były
wiecznie przechowywane w koszu aż ich ręcznie i świadomie nie skasują.

## Manualne określenie czasu

Na szczęście WordPress ma rozwiązanie tego problemu, bowiem [wprowadził możliwość zdefiniowania
czasu](https://codex.wordpress.org/Trash_status) przebywania wiadomości czy komentarza w koszu. By z
skorzystać z tej opcji, dodajemy do pliku `wp-config.php` poniższy kod:

    define( 'EMPTY_TRASH_DAYS', 60 );

Nazwa stałej jest trochę myląca bo każdemu pojedynczemu postowi będzie liczony czas z osobna, a nie
wszystkim razem, które akurat w tym czasie przebywają w koszu. Jeśli natomiast przestawimy tę stałą
na `0` , to przenoszenie do kosza zostanie wyłączone kompletnie i w edycji postu zamiast "przenieść
do kosza", będzie widoczny tekst "usuń na zawsze".
