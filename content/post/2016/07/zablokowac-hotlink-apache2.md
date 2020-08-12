---
author: Morfik
categories:
- Linux
date: "2016-07-20T18:55:21Z"
date_gmt: 2016-07-20 16:55:21 +0200
published: true
status: publish
tags:
- apache2
title: Jak zablokować hotlink'i w Apache2
---

[Hotlink lub hotlinking](https://pl.wikipedia.org/wiki/Hotlink) to proceder wykorzystywania zasobów
(plików) jednej strony www przez inny serwis internetowy. Chodzi generalnie o to, że materiały,
które pojawiają się w obcych serwisach, fizycznie w dalszym ciągu są hostowane, np. na naszym
serwerze. W taki sposób osoba, która linkuje do naszych plików `.jpg` , `.png` , `.mp3` czy nawet
`.css` i tworzy swój serwis w oparciu o nie, nie ponosi przy tym praktycznie żadnego obciążenia ze
swojej strony. Cały ten ciężar jest spychany na nasz serwer, co zjada nam transfer i zapychając
łącze. W tym krótkim wpisie postaramy się zablokować możliwość hotlink'owania przez inne serwisy
www w Apache2.

<!--more-->
## Całkowita blokada hotlink'ów

Jeśli nie chce nam się zbytnio wdawać w szczegóły na temat tego, która strona hotlink'uje do zasobów
naszego serwera, to możemy pokusić się o całkowite zablokowanie tego typu akcji. W tym celu musimy
wyedytować plik `/etc/apache2/apache2.conf` . Można oczywiście korzystać z pliku `.htaccess` ale
jeśli mamy dostęp do plików serwera Apache2, to lepiej jest całą konfigurację przeprowadzać w
obrębie jego plików. Tak czy inaczej, musimy dodać tę poniższą zwrotkę:

    <Directory /apache2>
    ...
        <IfModule mod_rewrite.c>
            RewriteEngine On
            RewriteCond %{HTTP_REFERER} !^{{< baseurl >}}/post/ [NC]
            RewriteCond %{HTTP_REFERER} !^$
            RewriteRule .*\.(jpg|jpeg|png|gif|ico|css|js)$ http://www.domena.pl/obrazek.jpg [L]
        </IfModule>
    ...
    </Directory>

Do ochrony zasobów przed hotlink'ami wykorzystywany jest moduł `mod_rewrite` , który ma na celu przy
pomocy referer'a w nagłówku HTTP określić prawidłowe odwołania do plików znajdujących się na
serwerze. Mamy tutaj dwa warunki. Pierwszy z nich pasuje do każdej domeny za wyjątkiem naszej
własnej. Drugi warunek zaś pasuje do niepustego referer'a. Po spełnieniu obu tych warunków,
odwołania do plików mających rozszerzenia widoczne wyżej zostaną zablokowane, a zamiast nich,
klient zostanie przekierowany pod wskazany adres.

## Częściowa blokada hotlink'ów

Alternatywnym podejściem jest zablokowanie hotlink'owania tylko tym serwisom, które nadużywają
naszej gościnności. W przypadku, gdy interesuje nas tego typu rozwiązanie, to musimy nieco przerobić
ten powyższy kod. Tym razem potrzebujemy czegoś na poniższy wzór:

    <Directory /apache2>
    ...
        <IfModule mod_rewrite.c>
            RewriteEngine On
            RewriteCond %{HTTP_REFERER} ^https://(.+\.)?aaa\.com/ [NC,OR]
            RewriteCond %{HTTP_REFERER} ^https://(.+\.)?bbb\.pl/ [NC,OR]
            RewriteCond %{HTTP_REFERER} ^https://(.+\.)?ccc\.eu/ [NC]
            RewriteRule .*\.(jpg|jpeg|png|gif|ico|css|js)$ http://www.domena.pl/obrazek.jpg [L]
        </IfModule>
    ...
    </Directory>

Wszystkie trzy warunki `RewriteCond` zawierają domeny, przed którymi chcemy chronić nasz serwis. W
tym przypadku, korzystamy z `OR` , który sugeruje, że tylko jeden z powyższych warunków musi być
spełniony, a nie wszystkie razem, tak jak to miało miejsce w poprzednim przykładzie. I to w
zasadzie wszystko.
