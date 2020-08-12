---
author: Morfik
categories:
- DD-WRT
date: "2016-09-11T12:42:30Z"
date_gmt: 2016-09-11 10:42:30 +0200
published: true
status: publish
tags:
- ssh
- router
- dd-wrt
title: 'DD-WRT: Dostęp do routera (telnet, ssh, panel web)'
---

Do routera posiadającego na pokładzie firmware DD-WRT możemy uzyskać dostęp na kilka sposobów. Ten
najbardziej popularny, to oczywiście panel administracyjny dostępny z poziomu www, na który możemy
się dostać wpisując w pasku adresu przeglądarki `http://192.168.1.1/` . Niemniej jednak, to nie jest
jedyna droga do zarządzania routerem. Standardowo również mamy aktywną usługę telnet. Dodatkowo
możemy aktywować SSL/TLS w panelu admina oraz dorobić usługę SSH. W tym artykule omówimy sobie
wszystkie te formy dostępu do routera.

<!--more-->
## Dostęp do routera przez panel administracyjny

Firmware DD-WRT zapewnia nam dość przyzwoity graficzny panel webowy, który tuż po wgraniu nowego
oprogramowania na router jest dostępny pod adresem `http://192.168.1.1/` . W tym panelu jesteśmy w
stanie przeprowadzić konfigurację praktycznie wszystkich aspektów pracy naszego routera. Po wpisaniu
tego powyższego adresu w przeglądarce, naszym oczom powinna pokazać się poniższa
strona:

![]({{< baseurl >}}/img/2016/09/1.dd-wrt-hard-reset-panel-admina-factory-defaults.png)

Zgodnie z ostrzeżeniem zmieniamy nazwę użytkownika i hasło. Po zatwierdzeniu danych logowania,
powinniśmy zostać przeniesieni do panelu administracyjnego:

![]({{< baseurl >}}/img/2016/09/2.dd-wrt-panel-admina.png)

### Aktywacja SSL/TLS w panelu administracyjnym

Możemy także pokusić się o implementację protokołu szyfrującego SSL/TLS w panelu administracyjnym,
by zabezpieczyć naszą komunikację z serwerem www, który działa na routerze. Mam też możliwość
wymuszenia szyfrowania i w ten sposób przepiąć panel admina na `https://` . Aktywacji protokołu
SSL/TLS w panelu można dokonać przechodząc na zakładkę Administration =\> Management. Tam zaś w
sekcji `Web Access` zaznaczamy `HTTPS` :

![]({{< baseurl >}}/img/2016/09/3.dd-wrt-panel-admina-https-ssl-tls.png)

Niemniej jednak, trzeba się liczyć z faktem niezaufanego przez przeglądarki certyfikatu, który
będzie w użyciu przez router. W efekcie po wpisaniu w pasku adresu `https://192.168.2.1/` pojawi
się nam to poniższe ostrzeżenie:

![]({{< baseurl >}}/img/2016/09/4.dd-wrt-panel-admina-https-ssl-tls-blad.png)

Oczywiście to w niczym nam nie przeszkadza i możemy dodać wyjątek:

![]({{< baseurl >}}/img/2016/09/5.dd-wrt-panel-admina-https-ssl-tls-blad-wyjatek.png)

Po dodaniu wyjątku, powinniśmy zostać zalogowani do panelu administracyjnego już z wykorzystaniem
szyfrowanego połączenia:

![]({{< baseurl >}}/img/2016/09/6.dd-wrt-panel-admina-https-ssl-tls.png)

## Dostęp do routera przez telnet

Do routera możemy także zalogować się za pomocą protokołu telnet. Standardowe dane logowania to
użytkownik `root` i hasło `admin` . Użytkownik root obowiązuje nawet w przypadku, gdy zmienialiśmy
dane logowania w panelu administracyjnym. Hasło zaś jest takie jak ustawiliśmy w panelu.

Odpalmy zatem terminal i testujemy czy aby na pewno jesteśmy w stanie się zalogować na router z
wykorzystaniem protokołu telnet:

![]({{< baseurl >}}/img/2016/09/7.dd-wrt-dostep-telnet-terminal.png)

## Dostęp do routera przez SSH

Jeśli potrzebujemy zabezpieczyć komunikację z routerem, to zamiast korzystać z protokołu telnet
lepiej jest zainwestować w implementację protokołu SSH. Domyślnie jednak usługa SSH nie jest
aktywowana i musimy ją włączyć, np. z poziomu panelu webowego. W tym celu przechodzimy na zakładkę
Services =\> Services i szukamy za `Secure Shell` :

![]({{< baseurl >}}/img/2016/09/8.dd-wrt-dostep-ssh-aktywacja.png)

Samo aktywowanie usługi da nam możliwość zalogowania się na router przy wykorzystaniu terminala.
Wyżej mamy także opcję `SSH TCP Forwarding` , która szerzej znana jest jako [SSH port
forwarding]({{< baseurl >}}/post/dd-wrt-ssh-port-forwarding-panel-aministracyjny/). Umożliwia ona
bezpieczne logowanie się do panelu administracyjnego przez interfejs WAN. Tę opcję na razie
zostawiamy w spokoju. W okienku `Authorized Keys` podajemy publiczne klucze SSH ale o tym za moment.

Po aktywowaniu SSH na routerze, odpalamy terminal i wpisujemy w nim `ssh root@192.168.2.1` , adres
naturalnie trzeba dostosować sobie:

![]({{< baseurl >}}/img/2016/09/9.dd-wrt-dostep-ssh-logowanie-terminal.png)

Przy pierwszym logowaniu na router za pomocą SSH, zostaniemy poproszeni o akceptację klucza
publicznego routera. Oczywiście akceptujemy i podajemy hasło, czego efektem będzie uzyskanie dostępu
do wiersza poleceń na routerze.

### Klucze SSH

W przypadku wykorzystywania [kluczy SSH]({{< baseurl >}}/post/uwierzytelniajace-klucze-ssh/),
musimy takie klucze sobie stworzyć. Następnie musimy przesłać nasz klucz publiczny na router
korzystając z okienka `Authorized Keys` , które mieliśmy wyżej. Po prostu wklejamy tam zawartość
pliku `*.pub` . Teraz możemy odpalić terminal i wpisać w nim `ssh 192.168.2.1` :

![]({{< baseurl >}}/img/2016/09/10.dd-wrt-dostep-ssh-logowanie-klucze-ssh.png)

Jak widzimy wyżej, tym razem nie zostaliśmy poproszeni o hasło podczas logowania. Tak właśnie
działają uwierzytelniające klucze SSH.

W przypadku wykorzystywania kluczy SSH można wyłączyć logowanie przez SSH z wykorzystaniem hasła.
Niemniej jednak, trzeba pamiętać, że w przypadku, gdy coś stanie się z naszym kluczem prywatnym, to
stracimy dostęp do routera przez SSH. Dlatego też dobrze jest zostawić aktywną opcję `Password
Login` i po implementacji kluczy SSH zmienić hasło na jakieś długie i skomplikowane i zapisać je
sobie w jakimś menadżerze haseł, typu [keepass](http://keepass.info/).

### Wyłączenie usługi telnet

Po aktywowaniu usługi SSH, możemy także wyłączyć usługę telnet, która jest nieco niżej w zakładce
Services =\> Services:

![]({{< baseurl >}}/img/2016/09/11.dd-wrt-dostep-ssh-wylaczenie-telnet.png)

Mi ta usługa praktycznie w niczym nie przeszkadza, dlatego postanowiłem pozostawić ją aktywną.
