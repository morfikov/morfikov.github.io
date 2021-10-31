---
author: Morfik
categories:
- Linux
date: "2016-05-28T18:29:40Z"
date_gmt: 2016-05-28 16:29:40 +0200
published: true
status: publish
tags:
- email
- prywatność
- thunderbird
GHissueID: 497
title: Jak ukryć prywatny adres IP w nagłówku email
---

Korzystając z różnego rodzaju klientów email udostępniamy serwerom mailowym nieco więcej informacji
niż gdybyśmy to robili z panelu webowego serwera pocztowego. Chodzi generalnie o adres IP hosta,
który przesłał wiadomość na serwer. Tradycyjnie w nagłówku email'a znajdzie się nasz adres
zewnętrzny, ten przypisany przez ISP. Niemniej jednak, w części przypadków będzie załączony również
adres IP z przestrzeni prywatnej, np. 10.0.0.0/8 lub 192.168.0.0/16 . O ile nie damy rady zataić
publicznego IP naszego ISP, to jesteśmy w stanie ukryć ten prywatny adres i tym samym nieco poprawić
naszą prywatność. W tym wpisie zostanie pokazane, to jak ten adres prywatny ukryć w nagłówku
wiadomości email w kliencie [Thunderbird](https://www.mozilla.org/pl/thunderbird/).

<!--more-->
## Jak podejrzeć nagłówek wiadomości email w Thunderbird

Standardowe ustawienia Thunderbird'a pokazują jedynie podstawowe informacje zawarte w nagłówku
wiadomości. Są to min. adresat, nadawca i temat wiadomości. Niemniej jednak, nie są to wszystkie
pola, które w tym nagłówku są zawarte. Jest ich o wiele więcej ale nie będziemy się tutaj nad nimi
zbytnio rozwodzić. Nas interesuje w tej chwili jak podejrzeć cały nagłówek wiadomości. Odpalamy
zatem Thunderbird'a i zaznaczamy wiadomość, którą wysłaliśmy. Następnie z menu wybieramy `View
Source` :

![thunderbird-podlgad-naglowek-email](/img/2016/05/1.thunderbird-podlgad-naglowek-email.png#huge)

Powinna nam się pokazać wiadomość w formie tekstowej. W niej, na samej górze mamy nagłówek:

![prywatny-adres-ip-naglowek-email](/img/2016/05/2.prywatny-adres-ip-naglowek-email.png#huge)

## Prywatny adres IP w nagłówku

Widzimy wyraźnie na powyższej fotce, że w polu `Received: from` mamy uwzględnione dwa adresy IP.
Jeden z nich to adres z przestrzeni prywatnej. To właśnie ten adres IP zamierzamy ukryć. By to
zrobić, musimy wejść w opcje Thunderbird'a (Edit > Preferences > Advanced > Config Editor). Tam
odszukujemy wpis `mail.smtpserver.default.hello_argument` i zmieniamy jego wartość na `localhost` :

![about-config-thunderbird-naglowek-email-adres-ip](/img/2016/05/3.about-config-thunderbird-naglowek-email-adres-ip.png#huge)

Jeśli nie mamy tego powyższego wpisu, to musimy go sobie stworzyć. W tym celu klikamy w powyższym
oknie prawym przyciskiem myszy i wybieramy kolejno New \> String i wpisujemy
`mail.smtpserver.default.hello_argument` .

W tej chwili możemy przetestować czy nasz adres prywatny został ukryty i zastąpiony nazwą
`localhost` . W tym przypadku dodanie powyższego parametru odnosi zamierzony skutek:

![adres-ip-localhost-naglowek-email](/img/2016/05/4.adres-ip-localhost-naglowek-email.png#huge)
