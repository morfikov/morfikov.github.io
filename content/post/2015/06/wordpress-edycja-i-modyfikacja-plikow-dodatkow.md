---
author: Morfik
categories:
- WordPress
date: "2015-06-02T19:33:10Z"
date_gmt: 2015-06-02 17:33:10 +0200
published: true
status: publish
tags:
- blog
title: 'WordPress: Edycja i modyfikacja plików dodatków'
---

Na necie widziałem wiele tutoriali na temat zmiany konfiguracji stylów czy motywów WordPressa i w
większości z nich ludzie dokonywali poprawek w kodzie źródłowym bezpośrednio przez panel
administracyjny. Ni to wygodne, ani bezpieczne. Prze wszystkim, jeśli jakiś robak uzyskałby dostęp
do konta administracyjnego naszego bloga, to pierwsze co mu przyjdzie do głowy, to edycja plików
właśnie za pomocą wbudowanego w WordPress edytora (Appearance =\> Editor). Dlatego ze względów
bezpieczeństwa kluczowe jest [wyłączenie tej
opcji](https://codex.wordpress.org/Editing_wp-config.php#Disable_Plugin_and_Theme_Update_and_Installation)
i jeśli potrzebujemy zmieniać określone pliki WordPressa, to róbmy to poza samym skryptem i
najlepiej przy pomocy zwykłego notatnika co potrafi kolorować składnię, by uniknąć również
ewentualnych literówek.

<!--more-->
## Edycja pogwałceniem norm bezpieczeństwa

WordPress posiada dwie użyteczne opcje, które mogą pomóc w zachowaniu integralności jego plików.
Pierwszą z nich jest `DISALLOW_FILE_EDIT` , która wyłącza wbudowany edytor plików uniemożliwiając
tym samym wprowadzanie jakichkolwiek zmian w plikach motywów i wtyczek z poziomu panelu
administracyjnego. Krótko mówiąc, edycja tych plików nie będzie już możliwa. By włączyć tę dodatkową
warstwę ochronną, dopisujemy do pliku `wp-config.php` poniższą linijkę:

    define( 'DISALLOW_FILE_EDIT', true );

Być może niektórym utrudni ona życie ale ja uważam, że WordPress powinien wyeliminować tego
backdoora kompletnie i to najlepiej jak najszybciej.

Druga opcja z kolei to `DISALLOW_FILE_MODS` i odpowiada ona za wyłączenie możliwości
instalacji/aktualizacji wtyczek i motywów z poziomu panelu administracyjnego. Ponadto, automatycznie
implikuje wyłączenie edytora, czyli ustawia również i poprzednią opcję. Chodzi generalnie o to, by z
poziomu panelu admina nie być w stanie dograć nowych dodatków, których bezpieczeństwo pozostawia
często wiele do życzenia. Pamiętajmy jednak, że to nie wyłącza tej funkcjonalności całkowicie, bo w
dalszym ciągu będziemy w stanie aktualizować i instalować nowe komponenty, z tym, że manualnie lub
też półautomatycznie za pośrednictwem odpowiednich narzędzi, np.
[wp-cli]({{< baseurl >}}/post/wordpress-instalacja-przy-pomocy-wp-cli/). Tak czy inaczej, ja
zalecam ustawienie tej opcji i robimy to również przez plik `wp-config.php` :

    define( 'DISALLOW_FILE_MODS', true );

Spójrzmy prawdzie w oczy, WordPress jest dość rozbudowaną aplikacją i bardzo rzadko spotkamy się z
tylko jedną drogą, która zaprowadzi nas do pewnego określonego celu. Większości z tych rozwiązań nie
potrzebujemy i to zwykle tych, które stwarzają największe zagrożenie bezpieczeństwa instalacji
bloga. Im szybciej wyłączymy wszystkie te zbędne śmieci, tym lepiej.
