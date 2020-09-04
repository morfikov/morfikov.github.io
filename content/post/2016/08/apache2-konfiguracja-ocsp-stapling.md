---
author: Morfik
categories:
- Linux
date: "2016-08-06T19:16:09Z"
date_gmt: 2016-08-06 17:16:09 +0200
published: true
status: publish
tags:
- bezpieczeństwo
- apache2
- ssl
- tls
title: 'Apache2: Konfiguracja OCSP Stapling'
---

Serwery udostępniające nam różnego rodzaju strony www na protokole SSL/TLS posiadają certyfikaty,
które są ważne przez pewien okres czasu. Z reguły jest to rok albo, jak w przypadku
[letsencrypt]({{< baseurl >}}/post/certyfikat-letsencrypt-dla-bloga-certbot/), są to 3 miesiące.
Taki certyfikat może zostać unieważniony z różnych przyczyn ale informacja o tym fakcie musi trafić
do wszystkich klientów odwiedzających taki serwis www. Do tego celu mogą posłużyć dwa mechanizmy.
Pierwszym z nich są listy [Certificate Revocation
Lists](https://pl.wikipedia.org/wiki/Lista_uniewa%C5%BCnionych_certyfikat%C3%B3w) (CRL). Drugim zaś
jest [Online Certificate Status
Protocol](https://pl.wikipedia.org/wiki/Online_Certificate_Status_Protocol) (OCSP). W tym wpisie
postaramy się zaimplementować to drugie rozwiązanie na serwerze Apache2.

<!--more-->
## Czy jest Online Certificate Status Protocol (OCSP)

OCSP, to protokół pozwalający sprawdzić statusu certyfikatu x509. Odbywa się to na takiej zasadzie,
że przeglądarka internetowa wysyła zapytanie o certyfikat do serwera OCSP (OCSP responder). Każdy
CA (Certificate Authority) ma taki serwer i dzięki niemu może przesłać klientom cyfrowo podpisane
komunikaty zawierające status certyfikatu. Klient przesyłając zapytanie o certyfikat wstrzymuje
automatycznie akceptację takiego certyfikatu do momentu aż responder udzieli odpowiedzi.

W odpowiedzi na żądanie klienta mogą zostać zwrócone trzy komunikaty zwrotne: `good` , `revoked`
oraz `unknown` . W przypadku, gdy serwer OCSP zwróci odpowiedź `good` , to klient wie, że certyfikat
jest ważny. Gdy serwer zwróci status `revoked` , oznacza to, że certyfikat został cofnięty
(unieważniony) tymczasowo/permanentnie z jakiegoś powodu. Natomiast jeśli chodzi zaś o status
`unknown` , to jest on zwracany w sytuacjach, gdzie nic nie wiadomo o certyfikacie, o który
dopytywał się klient.

Przeglądarka musi jednak wiedzieć, gdzie wysłać zapytanie o ważność certyfikatu. Ta informacja jest
zawarta bezpośrednio w samym certyfikacie CA w jednym z jego rozszerzeń:

![]({{< baseurl >}}/img/2016/08/1.ocsp-adres-responder-przegladarka.png#big)

Więcej informacji na temat samego protokołu OCSP można znaleźć w [dokumencie
RFC2560](https://tools.ietf.org/html/rfc2560).

## Wady i zalety protokołu OCSP

Problem w przypadku protokołu OCSP wydaje się być widoczny gołym okiem. Wystarczy zadać sobie
pytanie: co się stanie w przypadku niedostępności respondera, czyli serwera odpowiadającego na
zapytania o status certyfikatu? Oczywistym jest, że nie damy rady zweryfikować certyfikatu. Ale ten
problem można rozwiązać implementując jako backup listy CRL, choć przeglądarki do tej kwestii
podchodzą różnie. Za to na korzyść może przemawiać fakt, że odpowiedź z respondera ma jedynie kilka
KiB w porównaniu do kilku MiB w przypadku list CRL.

Wadą protokołu OCSP jest także wydłużenie czasu, jaki potrzebny jest do załadowania się strony www.
Trzeba przecież wysłać dodatkowe zapytanie do serwera o status certyfikatu i czekać na odpowiedź.

Do tego wszystkiego dochodzi jeszcze także sprawa konfiguracji cache. W przypadku, gdy czas wpisu w
cache będzie dłuższy, to po cofnięciu certyfikatu, klientowi może zostać zwrócony status `good` , a
to już niedobrze.

Ostatnim chyba problemem związanym z protokołem OCSP może być kwestia prywatności, gdzie właściciel
CA naszego certyfikatu może poznać odwiedzane przez klientów strony www. Chodzi o to, że do
respondera jest wysyłany numer seryjny certyfikatu danej strony.

## OCSP Stapling

Problem zarówno z opóźnieniami związanymi z wysłaniem zapytania o status certyfikatu jak i
przesyłaniem jego numeru seryjnego do respondera można zaadresować implementując [OCSP
Stapling](https://raymii.org/s/tutorials/OCSP_Stapling_on_Apache2.html) na serwerze www. Ten
mechanizm działa na takiej zasadzie, że to serwer www odpytuje responder o status swojego
certyfikatu. Gdy responder udzieli mu odpowiedzi, ta jest buforowana przez jakiś okres czasu. Gdy
klient podłącza się do serwera ten rekord OCSP jest do niego przesyłany w momencie zestawiania
szyfrowanego tunelu SSL/TLS. W efekcie cały proces weryfikacji certyfikatu przez klienta jest
krótszy i nie jest przy tym zagrożona jego prywatność. Trzeba przy tym wyraźnie zaznaczyć, że
serwer www prześle taką zbuforowaną odpowiedź do klienta, tylko i wyłącznie w sytuacji, gdy ten o
nią wyraźnie poprosi. Może to zrobić przez ustawienie w pakiecie "Client Hello" rozszerzenia
`status_request` . Wygląda to mniej więcej tak jak na tej fotce poniżej:

![]({{< baseurl >}}/img/2016/08/2.ocsp-status-request-wireshark-klient.png#huge)

Odpowiedź od respondera może siedzieć w cache serwera www do 48 godzin. Co pewien okres czasu,
serwer www będzie odpytywał responder o status certyfikatu i pobierał od niego odpowiedź.

## Implementacja OCSP Stapling w Apache2

Do zaimplementowania mechanizmu OCSP Stapling potrzebny nam jest serwer Apache2 w wersji minimum
2.3.3 oraz OpenSSL w wersji 0.9.8h i wyższej. Stabilny debian spełnia te wymagania, zatem możemy
przejść do konfiguracji samego serwera www. Ta z kolei sprowadza się do dodania poniższych dyrektyw
w pliku `/etc/apache2/apache2.conf` :

    SSLUseStapling on
    SSLStaplingCache "shmcb:logs/stapling-cache(150000)"
    SSLStaplingResponseMaxAge 900

Dyrektywa `SSLUseStapling` włącza OCSP Stapling na serwerze i klient łączący się z serwerem będzie
zwolniony z bezpośredniego odpytywania respondera o status certyfikatu. Dalej mamy parametr
`SSLStaplingCache` , który konfiguruje cache dla odpowiedzi z respondera. W tym przypadku jest
wykorzystywany [bufor cykliczny](https://pl.wikipedia.org/wiki/Bufor_cykliczny) o rozmiarze około
150KiB wewnątrz segmentu pamięci współdzielonej (wymaga modułu `mod_socache_shmcb`). Następnie mamy
dyrektywę `SSLStaplingResponseMaxAge` , która ustawia czas świeżości cache na 15 minut (900s). Czyli
co 15 minut serwer będzie odpytywał responder o status certyfikatu i pakował otrzymaną odpowiedź do
cache.

## Test OCSP Stapling'u

OCSP Stapling możemy przetestować zaprzęgając do pracy sniffer `wireshark` i podglądając pakiet
"Server Hello". Musi on mieć uwzględnione rozszerzenie `status_request` . Natomiast samą odpowiedź
możemy sprawdzić wpisując w terminalu to poniższe polecenie:

    $ openssl s_client -connect morfitronik.pl:443 -tls1 -tlsextdebug -status

W logu powinna znajdować się odpowiedź OCSP:

![]({{< baseurl >}}/img/2016/08/3.ocsp-stapling-test-server-responder.png#huge)

Status certyfikatu w odpowiedzi wskazuje na `good` , zatem certyfikat jest ważny.
