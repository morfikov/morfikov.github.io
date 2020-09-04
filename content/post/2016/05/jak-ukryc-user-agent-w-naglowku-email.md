---
author: Morfik
categories:
- Linux
date: "2016-05-29T12:05:46Z"
date_gmt: 2016-05-29 10:05:46 +0200
published: true
status: publish
tags:
- email
- prywatność
- thunderbird
title: Jak ukryć user-agent w nagłówku email
---

Na każdym kroku można natknąć się na przecieki informacyjne, które zagrażają naszej prywatności.
Widzieliśmy to choćby i na przykładzie [prywatnego adresu IP widocznego w nagłówku w wiadomości
email]({{< baseurl >}}/post/jak-ukryc-prywatny-adres-ip-w-naglowku-email/). Przeglądając ten
podlinkowany wpis mogliśmy z grubsza zobaczyć jak wygląda taki nagłówek wiadomości. Bardziej uważni
czytelnicy zwrócili uwagę na jego zawartość. Poza polem zawierającym adres IP, można było też
zobaczyć pole odpowiadające za [user-agent](https://pl.wikipedia.org/wiki/User_agent). Ciąg zawarty
w tym polu zdradza informacje na temat wykorzystywanego klienta pocztowego oraz systemu
operacyjnego. Ten komunikat możemy jednak ukryć i w tym wpisie zostanie to pokazane na przykładzie
[Thunderbird'a](https://www.mozilla.org/pl/thunderbird/).

<!--more-->
## Przykładowy nagłówek zawierający user-agent

Jako, że Thunderbird pokazuje jedynie podstawowe pola w nagłówku email, to musimy sięgnąć do źródła
wiadomości. Zaznaczamy zatem wiadomość i z menu wybieramy `View Source` :

![]({{< baseurl >}}/img/2016/05/1.thunderbird-naglowek-wiadomosc-email.png#huge)

W nowym oknie powinno zostać pokazane źródło:

![]({{< baseurl >}}/img/2016/05/4.adres-ip-localhost-naglowek-email.png#huge)

W tym przypadku w polu `User-Agent:` mamy ciąg `Mozilla/5.0 (X11; Linux x86_64; rv:45.0)
Gecko/20100101 Thunderbird/45.1.0` . Widzimy wyraźnie, że system przesłał w nagłówku wiadomości dane
dotyczące wykorzystywanego klienta pocztowego jak i jego wersji oraz szereg informacji o systemie
operacyjnym.

## Ukrywanie user-agent

Najwyższa pora oduczyć Thunderbird'a przesyłania tego typu informacji do serwerów mailowych. W tym
celu przechodzimy kolejno do Edit > Preferences > Advanced > Zakładka General > Config Editor.
Tam musimy dodać wpis `general.useragent.override` i ustawić jego wartość na pustą:

![]({{< baseurl >}}/img/2016/05/2.thunderbird-nadpisanie-user-agent.png#huge)

Sprawdźmy teraz, czy wiadomość przesłana do serwera mailowego zawiera w nagłówku pole user-agent. W
tym celu wysyłamy przykładową wiadomość i ponownie patrzymy w jej źródło:

![]({{< baseurl >}}/img/2016/05/3.thunderbird-brak-user-agent-email.png#huge)

Widzimy wyraźnie na powyższej fotce, że nagłówek już nie zawiera pola user-agent. Oczywiście,
zamiast ukrywać to pole, możemy zwyczajnie podmienić jego wartość na jedną z tych [wymienionych w
tym linku](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/User-Agent/Firefox).
