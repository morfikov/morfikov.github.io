---
author: Morfik
categories:
- Blog
date: "2015-05-24T11:28:16Z"
date_gmt: 2015-05-24 09:28:16 +0200
published: true
status: publish
tags:
- wordpress
GHissueID: 237
title: 'WordPress: Administrator bloga'
---

Domyślna konfiguracja użytkowników w WordPress'ie nie należy do bezpiecznych. Samą nazwę
administratora witryny możemy naturalnie zmienić podczas instalacji tego CMS'a i prawdopodobnie ten
zabieg wystarczy większości blogerom. Jednak z punktu widzenia bezpieczeństwa instalacji naszego
bloga, nie powinniśmy się ograniczać jedynie do zmiany nazwy konta administracyjnego. Administrator
ma za zadanie zarządzać blogiem, natomiast konta autorów wpisów podlegają już pod osobną grupę i o
nie również powinniśmy się zatroszczyć.

<!--more-->
## Administrator WordPress'a i zmiana jego ID

Przeważnie zawsze po zainstalowaniu skryptu WordPressa, konto administratora ma numer ID `1` i
przydałoby się coś z tym zrobić. Bez edycji bazy danych raczej się nie obejdzie ale znowu nie będzie
to aż takie trudne, zwłaszcza jeśli posiada się jakieś graficzne narzędzie typu **mysql-workbench**
ułatwiające pracę z bazami danych.

Poniżej znajdują się przykładowe tabele bazy danych w dopiero co zainstalowanym blogu:

![wordpress-tabele](/img/2015/05/1.wordpress-tabele.png#small)

Przechodzimy zatem do tabeli `wp_users` i listujemy jej zawartość, a ta w moim przypadku wygląda
mniej więcej tak jak na fotce poniżej:

![wordpress-tabela-wp-users](/img/2015/05/2.wordpress-tabela-wp-users.png#huge)

Zmieniamy numerek w pierwszej kolumnie (najlepiej losowy), dla przykładu `9999` :

![Wordpress-zmiana-id-admina](/img/2015/05/3.Wordpress-zmiana-id-admina.png#small)
