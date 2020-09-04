---
author: Morfik
categories:
- Linux
date: "2015-05-25T08:12:39Z"
date_gmt: 2015-05-25 07:12:39 +0200
published: true
status: publish
tags:
- bezpieczeństwo
- firefox
- ssl
- tls
title: Logjam, czyli nowa podatność w SSL/TLS
---

Jak donoszą [ostatnio](https://blog.cryptographyengineering.com/2015/05/22/attack-of-week-logjam/)
[media](https://niebezpiecznik.pl/post/logjam-nowa-podatnosc-w-protokolach-https-ssh-i-ipsec/), mamy
kolejną dziurę (***logjam***) dotyczącą szyfrowania SSL/TLS, a konkretnie rozchodzi się o
powszechnie stosowany na całym świecie protokół
[Diffiego-Hellmana](https://pl.wikipedia.org/wiki/Protok%C3%B3%C5%82_Diffiego-Hellmana) . I znów
jest podobny scenariusz, bo ten problem nie powinien mieć miejsca ale z powodu wstecznej
kompatybilności, tj. zapewnienie wsparcia dla wszystkich tych przestarzałych szyfrów tak by te
przedwieczne systemy/maszyny mogły działać, można doprowadzić do osłabienie mechanizmów, które
powinny być wykorzystywane obecnie. OK, może nie tyle osłabić, co wykorzystać te słabsze
odpowiedniki zamiast tych mocniejszych.

<!--more-->
## Czy zagraża mi logjam?

Na obecną chwilę, wszystkie linuxowe przeglądarki są podatne, tj. `Google Chrome 43.0.2357.81` ,
`Opera 30.0.1835.26 beta` oraz `Mozilla Firefox 38.0.1` . Jak nazłość [Internet
Explorer](https://technet.microsoft.com/en-us/library/security/ms15-055.aspx) został już dawno
poprawiony... Test jako taki możemy sobie przeprowadzić sami. Wystarczy, że przejdziemy na stronę
<https://weakdh.org/> (lub też i na <https://www.ssllabs.com/>) , a na jej szczycie zastaniemy
informację, która powinna jednoznacznie wskazać nam czy nasza przeglądarka jest podatna na ten atak:

![]({{< baseurl >}}/img/2015/05/1.brak-dopornosci-na-logjam.png#huge)

W prawdzie rzadko korzystam z chrome, a już praktycznie wcale nie korzystam z opery i nie powiem za
bardzo jak na własną rękę się w tych przeglądarkach zabezpieczyć przed tą formą ataku. Natomiast
jeśli chodzi o firefoxa, to tutaj mamy do dyspozycji kilka możliwości. Przede wszystkim został
oddelegowany do naszej dyspozycji [oficjalny
dodatek](https://addons.mozilla.org/en-US/firefox/addon/disable-dhe/) wypuszczony przez Mozillę i on
fixnie ten cały problem. Jeśli jednak nie uśmiecha nam się instalowanie nowych addonów w
przeglądarce, to możemy zawsze zreprodukować działanie plugina Mozilli i przeprowadzić wszystkie
kroki ręcznie. Wystarczy, że wpiszemy w pasku adresu firefoxa `about:config` i wyszukamy `ssl3` . Po
chwili powinien nam się wyświetlić listing opcji, gdzie dwie z nich mają w nazwie jedynie `dhe` (z
reguły są to dwie pierwsze opcje). By wyeliminować podatność, trzeba te dwie pozycje przestawić na
`false`

![]({{< baseurl >}}/img/2015/05/2.uodparnianie-firefoxa-na-logjam.png#huge)

Teraz możemy przejść ponownie do testowania podatności na stronie <https://weakdh.org/> i tym razem
powinniśmy już ujrzeć komunikat oświadczający nam, że jesteśmy bezpieczni:

![]({{< baseurl >}}/img/2015/05/3.odporność-na-logjam.png#huge)

Jeśli dysponujemy serwerami, np. Apache, możemy przetestować jego podatność
[tutaj](https://weakdh.org/sysadmin.html) . W tym linku znajdują się również informację na temat
ewentualnego poprawienia konfiguracji popularniejszych aplikacji serwerowych, tak by wyeliminować
możliwość wykorzystania ataku logjam.
