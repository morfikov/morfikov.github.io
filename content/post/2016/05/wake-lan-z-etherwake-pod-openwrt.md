---
author: Morfik
categories:
- OpenWRT
date: "2016-05-11T21:39:37Z"
date_gmt: 2016-05-11 19:39:37 +0200
published: true
status: publish
tags:
- chaos-calmer
- sieć
- router
title: Wake On LAN z etherwake pod OpenWRT
---

Router wyposażony w firmware OpenWRT potrafi wybudzać maszyny w sieci lokalnej. [Wake On LAN
(WOL)](https://pl.wikipedia.org/wiki/Wake_on_LAN) nie działa przez internet, a jedynie, jak sama
nazwa sugeruje, w sieci LAN. W tym mechanizmie wykorzystywany jest
[broadcast](https://pl.wikipedia.org/wiki/Broadcast), a routery nie forward'ują pakietów
rozgłoszeniowych. Oczywiście, nic nie stoi na przeszkodzie, by zalogować się na router via SSH od
strony WAN i wybudzić jakąś maszynę z poziomu routera. Komputery, które chcemy budzić muszą mieć
odpowiednią płytę główną. Prawdopodobnie wszystkie nowsze płyty już taką właściwość posiadają.
Dodatkowo, trzeba w BIOS'ie ustawić odpowiednie opcje. Ważną rzeczą jest, by nie wyłączać PC
przyciskiem w obudowie (lub na zasilaczu), bo wtedy nie będzie możliwe wybudzenie maszyny, nawet po
dostarczeniu jej zasilania. Wyłączenia maszyny musimy dokonać z poziomu systemu operacyjnego, tylko
wtedy WOL zadziała. W tym wpisie pokażemy jak przy pomocy `etherwake` wybudzić określonego hosta w
sieci LAN.

<!--more-->
## Instalacja i konfiguracja etherwake

Dysponując odpowiednim hardware, oraz mając ustawione odpowiednie opcje w BIOS'ie, możemy przejść do
instalacji i konfiguracji potrzebnego nam oprogramowania. Logujemy się zatem na router i instalujemy
ten poniższy pakiet:

    # opkg update
    # opkg install etherwake

Konfiguracja dla tego demona znajduje się w pliku `/etc/config/etherwake` . Edytujemy ten plik i
przepisujemy go do poniższej postaci:

    config 'etherwake' 'setup'
        option 'pathes' '/usr/bin/etherwake /usr/bin/ether-wake'
        option 'sudo' 'off'
        option 'interface' 'br-lan'
        option 'broadcast' 'on'

    config 'target'
        option 'name' 'the-hound'
        option 'mac' '00:0f:ea:38:64:4c'
    #   option 'password' 'AABBCCDDEEFF'
        option 'wakeonboot' 'off'

Mamy tutaj dwie sekcje. W pierwszej z nich określamy zachowanie samego demona, który będzie
nasłuchiwał na określonym interfejsie ( `interface` ). W tym przypadku interfejs lokalny to
`br-lan` . To przez ten interfejs będą wysyłane pakiety budzące maszyny. Dalej mamy opcję
`broadcast` , która sprawia, że magiczne pakiety zdolne wybudzić maszyny w sieci są przesyłane na
adres rozgłoszeniowy 255.255.255.255 . Sekcji `config 'target'` możemy zdefiniować wiele i w każdej
z nich, w opcji `mac` podać adres MAC maszyny, którą chcemy wybudzić. Z kolei zaś opcja `name`
ułatwia wybudzanie przy pomocy skryptu `/etc/init.d/etherwake` . W przypadku, gdy ustawimy także
opcję `wakeonboot` , to przy starcie routera, wszystkie maszyny mające włączoną tą opcję zostaną
wybudzone.

## Budzenie maszyn

Maszyny możemy budzić na dwa sposoby. Pierwszym z nich jest wyżej określona opcja automatycznego
budzenia za każdym razem, gdy system routera startuje. Drugim zaś jest wywołanie skryptu
`/etc/init.d/etherwake` i podanie mu nazwy maszyny, którą chcemy wybudzić. Załóżmy, że chcemy
wybudzić tę maszynę co została określona w sekcji `config 'target'` . Odpalamy terminal, logujemy
się na router i z poziomu konsoli wydajemy to poniższe polecenie:

    # /etc/init.d/etherwake start the-hound
    # /etc/init.d/etherwake: Waking up the-hound via /usr/bin/etherwake -i br-lan -b 00:0f:ea:38:64:4c

Jeśli maszyna została wyłączona za pomocą systemu operacyjnego, a nie przez odcięcie jej zasilania,
to powinna zostać wybudzona. Jeśli się tak nie dzieje, to być może [karta sieciowa ma wyłączoną
obsługę WOL]({{< baseurl >}}/post/wylaczenie-wol-w-karcie-sieciowej/).
