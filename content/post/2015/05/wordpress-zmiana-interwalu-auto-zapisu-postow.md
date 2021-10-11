---
author: Morfik
categories:
- Blog
date: "2015-05-26T19:35:08Z"
date_gmt: 2015-05-26 17:35:08 +0200
published: true
status: publish
tags:
- wordpress
GHissueID: 226
title: 'WordPress: Zmiana interwału auto zapisu postów'
---

WordPress dysponuje bardzo rozbudowanym edytorem, przy pomocy którego można tworzyć dość
zaawansowane posty. Ten edytor potrafi nawet automatycznie zapisywać stan artykułu przesyłając w ten
sposób zmiany do bazy danych, co zapobiega utracie treści. Oczywiście, że potrzebna będzie dodatkowa
przestrzeń w samej bazie by kolejną kopię postu przechować ale wydaje się to niebyt wygórowaną ceną
za minimalizowanie ryzyka utraty części albo i całego wpisu.

<!--more-->
## Zmiana interwału auto zapisu

WordPress domyślnie dokonuje automatycznego zapisu co 60s. Czy to dużo czy mało, to już kwestia
dyskusyjna. Jeśli uważamy, że 1 min odstępu pomiędzy zapisem dokumentu, to zbyt często, będziemy
chcieli raczej podbić ten interwał i vice versa. Na szczęście mamy do tego celu oddelegowaną
specjalną opcję, którą trzeba dopisać do pliku `wp-config.php` :

    define( 'AUTOSAVE_INTERVAL', 300 );

Liczba `300` jest wyrażona w sekundach, czyli krótko mówiąc 5 min.

Jeśli nie ufamy WordPresowi, a raczej jego edytorowi, to zawsze możemy sprawdzić datę ostatniego
zapisu. Znajduje się ona w stopce formularza tekstowego:

![](/img/2015/05/0.status-zapisu.png#big)

W zależności od tego czy edytujemy szkic wiadomości, czy jest to opublikowany post, zachowanie auto
zapisu ulega zmianie.

Szkice artykułów są automatycznie przepisywane i żadna dodatkowa treść nie pojawia się w bazie
danych. Natomiast sprawa ma się nieco inaczej w przypadku opublikowanego już kontentu -- ten nigdy
nie zostanie automatycznie nadpisany za pośrednictwem auto zapisu, co jest zrozumiałe. Zamiast tego,
tworzony jest nowy wpis w bazie danych zawierający dodatkową kopię artykułu wraz z wprowadzonymi
zmianami i to właśnie ta kopia jest przepisywana podczas kolejnego auto zapisu.

## Problemy przy auto zapisie

Ten cały mechanizm zapisu danych działa w oparciu o daty postu oraz tego co zostało automatycznie
zapisane w bazie. To są dwa osobne wpisy dlatego też można ten czas porównać ze sobą. Gdy się klika
na przycisk **Publikuj**, zmieniany jest czas postu. Natomiast w przypadku gdy auto zapis zadziała,
to aktualizowana jest data i czas kopi postu. Tak czy inaczej, możemy czasem natrafić na poniższy
komunikat:

![](/img/2015/05/1.autosave-blad.png#huge)

Jest on wynikiem automatycznego zapisu opublikowanego już postu ale tylko w przypadku gdy nie
zaktualizowaliśmy samego artykułu przy pomocy przycisku **Publikuj/Aktualizuj**.

## Auto zapis podczas trybu offline

Jeśli z jakiegoś powodu stracimy połączenie z siecią lub też i na własne życzenie przejdziemy w tryb
offline, to dalej możemy pisać artykuł i wszelkie zmiany zostaną zachowane w lokalnym cache
przeglądarki (o ile ten jest włączony i jest tam dostateczna ilość miejsca) i będą tam rezydować do
momentu ponownego podłączenia się do internetu. O całym zdarzeniu zostaniemy poinformowani poniższym
monitem:

![](/img/2015/05/2.autosave-browser.png#huge)

Mamy tam informację iż wersja, która siedzi w cache naszej przeglądarki jest inna od tej ostatnio
zapisanej w bazie danych WordPressa i mamy do wyboru dwie opcje: zaaplikować backup z przeglądarki
lub też zignorować komunikat i przejść do edycji postu wyciągniętego bezpośrednio z bazy.

## Jak odróżnić rewizje auto zapisu?

Ludzie z WordPressa przewidzieli tego tupu mankamenty i postanowili odpowiednio oznaczyć rewizje
automatycznych zapisów. Jeśli spojrzymy na te dwie fotki poniżej:

![](/img/2015/05/3.roznica-rewizji.png#huge)

oraz:

![](/img/2015/05/4.roznica-rewizji.png#huge)

To jasno możemy stwierdzić, który zapis został dokonany przez nas, a który automatycznie przez
WordPressa.
