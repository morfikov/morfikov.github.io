---
author: Morfik
categories:
- WordPress
date: "2015-06-06T18:50:51Z"
date_gmt: 2015-06-06 16:50:51 +0200
published: true
status: publish
tags:
- cookies
- blog
title: 'WordPress: Ciasteczka i ich czas żywotności'
---

Ciasteczka w WordPresie są bardzo ważne, bo wykorzystuje się je w procesie uwierzytelniania
logujących się i powracających użytkowników. Domyślnie czas ważności generowanych ciasteczek to `2`
dni, chyba, że człowiek zaznaczy opcję "Pamiętaj" w formularzu logowania, wtedy ten okres ważności
ulega przedłużeniu do `14` dni. Chodzi o to, że w przypadku gdyby ciasteczka nie posiadały terminu
ważności, to jeśli ktoś by wszedł w ich posiadanie, to mógłby się zalogować w serwisie bez większych
problemów, a tego przecież nie chcemy. Są jednak sytuacje, w których 2 czy nawet 14 dni, to ździebko
za mało (lub za dużo) i w tym wpisie postaramy się dostosować żywotność ciasteczek, tak by
odpowiadała naszym upodobaniom.

<!--more-->
## Otrzymywane ciasteczka

WordPress przesyła szereg ciasteczek w zależności od tego jakie akcje przeprowadzamy na blogu.
Możemy zawsze je podejrzeć w każdej przeglądarce, poniżej fotka z firefoxa:

![]({{< baseurl >}}/img/2015/06/1.wordpress-zywotnosc-ciasteczka.png)

Na samym dole tego okienka mamy informację na temat ważności ciasteczka (Expires). W tym przypadku
jest to `Sun 21 Jun 2015 04:53:55 AM CEST` , czyli 14 dni licząc od daty publikacji tego postu. Po
upływie tego terminu, trzeba będzie się zalogować ponownie. Nie jest to znowu jakiś problem,
przynajmniej nie w przypadku ustawionych 14 dni ale co jeśli chcielibyśmy tę wartość sobie
dostosować? Mamy co prawda wybór ale znowu nie za duży, bo `2` lub `14` dni. A jeśli nasza
aplikacja wymaga mniejszego interwalu niż 2 dni albo gdybyśmy chcieli z jakiegoś powodu ustawić czas
wygaśnięcia ciasteczka na pół roku czy nawet więcej?

## Zmiana czasu wygaśnięcia ciasteczka

Na szczęście możemy bez problemu dostosować sobie ten przedział czasu. Musimy tylko dopisać poniższy
kod do pliku `functions.php` w używanym motywie WordPressa:

    function keep_me_logged_in( $expirein ) {
        return 15552000; // seconds
    }
    add_filter( 'auth_cookie_expiration', 'keep_me_logged_in' );

Wartość `15552000` w tym przypadku to pół roku w sekundach (60⋅60⋅24⋅180), z tym, że WordPress
dodaje od siebie 2 lub 14 dni w zależności od tego czy zaznaczyliśmy opcję "Pamiętaj". By zmiany
weszły w życie musimy się ponownie zalogować na blogu.

Warto jeszcze wspomnieć o tym, że przestawienie daty ważności ciasteczek, może w jakiś sposób
zagrozić bezpieczeństwu kont na blogu ale to tylko w przypadku gdy nie wdrożyliśmy
[pełnego]({{< baseurl >}}/post/wordpress-szyfrowanie-ssltls/)
[szyfrowania]({{< baseurl >}}/post/wymuszenie-ssl-tls-przy-pomocy-vhostow-apache2/) witryny i nie
korzystamy z [unikalnych kluczy/soli
uwierzytelniających]({{< baseurl >}}/post/uwierzytelniajace-klucze-ssh/). Z kolei jeśli
prowadzimy blog w pojedynkę i mamy zaimplementowany zewnętrzny system komentarzy (np. disqus), to po
ustawieniu czasu wygaśnięcia ciasteczka przykładowo na rok, chyba nie potrzebowalibyśmy w dalszym
ciągu pliku `wp-login.php` , prawda? Oczywiście przy założeniu, że publikujemy wpisy tylko z jednej
lub określonych maszyn, gdzie dostęp do systemu plików jest chroniony przy pomocy pełnego
szyfrowania dysku i mamy tam wgrane odpowiednie ciasteczko.
