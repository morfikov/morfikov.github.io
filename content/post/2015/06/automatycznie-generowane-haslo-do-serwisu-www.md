---
author: Morfik
categories:
- Linux
date: "2015-06-10T20:51:06Z"
date_gmt: 2015-06-10 18:51:06 +0200
published: true
status: publish
tags:
- bezpieczeństwo
- hasła
title: Automatycznie generowane hasło do serwisu WWW
---

Każdy z nas dla wygody, albo raczej przez lenistwo, stara się nie używać zbyt skomplikowanych haseł.
Zwykle też korzystamy z tego samego hasła do większości kont online w internecie. Jeśli zdarzyło nam
się tworzyć proste, krótkie i do tego łatwe do przewidzenia hasła, np. w oparciu o datę urodzenia
lub inne tego typu informacji, to przydałoby się nieco popracować nad zabezpieczeniami tych poufnych
danych, tak by nie były proste do odgadnięcia czy też i złamania.

<!--more-->
## SuperGenPass

Natrafiłem na projekt o nazwie [SuperGenPass](https://chriszarate.github.io/supergenpass/) , który
potrafi w dość znacznym stopniu ułatwić zarządzanie hasłami do kont internetowych. Jego źródła są
otwarte i hostowane na [githubie](https://github.com/chriszarate/supergenpass/) , zaś licencja to
[GNU General Public License version 2](http://www.gnu.org/licenses/gpl-2.0.html) .

SuperGenPass działa w obrębie przeglądarki i wszystkie hasła są generowane lokalnie. Jest on
dostarczany w postaci [bookmarkletu](https://en.wikipedia.org/wiki/Bookmarklet) (połączenie bookmark
oraz applet) , czyli dodatkowej zakładki zawierającej polecenia JavaScript, które rozbudowują
funkcjonalność samej przeglądarki. By utworzyć taki bookmarklet, po prostu postępujemy zgodnie z
instrukcjami na stronie projektu, tj. przeciągamy na pasek zakładek ten poniższy przycisk:

![](/img/2015/06/1.supergenpass-bookmarklet.png#medium)

Jeśli teraz klikniemy w nowo utworzoną zakładkę, to powinno nam się wyświetlić to poniższe okienko:

![](/img/2015/06/2.supergenpass-okno-glowne.png#small)

Mamy tam Master Password (hasło główne) oraz morfitronix.lh (domenę). W oparciu o te dwa parametry
zostanie wygenerowany hash (md5 lub sha1). Hasło jest tworzone na zasadzie karmienia funkcji
haszującej ciągiem znaków w postaci `masterpassword:domain.com` , po czym wynik jest przycinany do
zdefiniowanej ilości znaków, w tym przypadku 24. Jeśli domena lub hasło główne będą się różnić choć
o jeden znak, wtedy hash będzie diametralnie inny, co zapewnia różnorodność haseł.

Mamy tam jeszcze drugie okienko (Secret Password), o którym warto powiedzieć dwa słowa. To hasło
jest używane jako zabezpieczenie generujące [identikonę](https://en.wikipedia.org/wiki/Identicon) ,
która zawsze powinna wskazywać ten sam obrazek ilekroć pojawia się okno SuperGenPass. W przypadku
gdyby była ona inna, oznaczać to może, że ktoś próbuje przechwycić główne hasło i pod żadnym pozorem
nie powinniśmy w takim przypadku go wpisywać.

SuperGenPass daje także lekką ochronę przed phishingiem. Załóżmy, że ktoś w podesłanym nam emailu
załączył link o adresie `www.amaz0n.com` . Różni się on od oryginalnego odpowiednika zaledwie
jednym znakiem, zatem hash będzie inny i atak mający na celu wykradzenie hasła się nie powiedzie.

Jeśli chodzi zaś o to jakie hasła są generowane przy pomocy tego narzędzia, to dla maksymalnej
kompatybilności ze wszystkimi serwisami online, stosowane są tylko duże litery (A-Z), małe litery
(a-z) oraz cyfry (0-9). Dodatkowo, pierwszy znak takiego hasła zaczyna się zawsze z małej litery,
zawsze obecna jest co najmniej jedna wielka litera oraz jedna cyfra. Hasło nie może być krótsze niż
4 znaki i nie dłuższe niż 24, domyślnie ma 10 znaków.

## Generujemy losowe hasło

Spróbujmy zatem przetestować ten mechanizm na przykładowym blogu WordPressa. Wchodzimy na stronę
logowania:

![](/img/2015/06/3.supergenpass-haslo-generowanie.png#big)

Po czym uzupełniamy hasło główne i klikamy `generate` :

![](/img/2015/06/4supergenpass-haslo-wpisywanie.png#big)

Hasło zostało automatycznie wpisane w odpowiednie miejsce. Problem z tym rozwiązaniem jest taki, że
dalej trzeba ręcznie wpisać nazwę konta. Przynajmniej zmiana hasła działa bez problemu:

![](/img/2015/06/5.supergenpass-haslo-zmiana.png#huge)

Jak widać, można mieć jedno, a wykorzystywać wiele skomplikowanych haseł bez większego wysiłku w
każdym serwisie, w którym posiadamy jakieś konto.
