---
author: Morfik
categories:
- DD-WRT
date: "2016-09-12T17:29:22Z"
date_gmt: 2016-09-12 15:29:22 +0200
published: true
status: publish
tags:
- ssh
- router
- dd-wrt
title: 'DD-WRT: SSH port forwarding i panel aministracyjny'
---

Firmware DD-WRT oferuje kilka sposobów na uzyskanie dostępu do naszego domowego routera. Poza
graficznym panelem webowym, gdzie wszystko możemy sobie wyklikać, mamy jeszcze tekstowy telnet i
SSH. DD-WRT jest nam w stanie także zaoferować SSH port forwarding. Ten mechanizm z kolei bardzo
przydaje się w momencie, gdy chcemy uzyskać dostęp do routera przez panel administracyjny ale nie
uśmiecha nam się wystawianie go na widok publiczny po niezabezpieczonym protokole HTTP. Z kolei
serwer www posiadający zaimplementowany protokół SSL/TLS jest dość zasobożerny i jego zastosowanie
średnio nadaje się w przypadku małych routerów. Za pomocą przekierowania portów SSH możemy uzyskać
dostęp do lokalnej instancji panelu webowego omijając obydwa te powyższe problemy. Panel admina
pozostaje schowany w sieci lokalnej, a my logujemy się do niego wykorzystując połączenie SSH.

<!--more-->
## Jak włączyć SSH port forwarding w DD-WRT

Przede wszystkim, trzeba sobie zdać sprawę, że SSH port forwarding zestawia tunel między dwiema
maszynami, które zamierzają się komunikować. Wymagana zatem jest konfiguracja zarówno klienta jak i
serwera, którym w tym przypadku jest router z DD-WRT.

W przypadku routera, sprawa wygląda dość prosto. Logujemy się do panelu administracyjnego na adres
`http://192.168.1.1/` i przechodzimy na zakładkę Services => Services. Tam mamy zaś pozycję `Secure
Shell` . Po jej aktywacji będziemy mieć dostęp do opcji `SSH TCP Forwarding` . Tę opcję również
zaznaczamy:

![](/img/2016/09/1.ssh-port-forwarding-router-dd-wrt-panel-admina.png#huge)

Następnie przechodzimy na zakładkę Administration => Management i szukamy za `Remote Access` ,
gdzie zaznaczamy `SSH Management` . Port możemy sobie dostosować wedle uznania, 22 jest domyślny:

![](/img/2016/09/2.ssh-port-forwarding-router-dd-wrt-panel-admina.png#huge)

Zapisujemy i aplikujemy ustawienia.

## Jak skonfigurować SSH port forwarding na Debianie

Konfigurację routera mamy z głowy. Teraz pozostało nam jeszcze skonfigurowanie klienta, który ma
zamiar korzystać z tego całego SSH port forwarding'u. Jako, że ja wykorzystuję system operacyjny
linux, a konkretnie dystrybucję Debian, to na jej przykładzie opiszę proces konfiguracji tego
mechanizmu. Na dobrą sprawę, to musimy odpalić terminal i w nim wpisać to poniższe polecenie:

    $ ssh -L 12345:localhost:80 root@192.168.1.235 -p 22

Wydanie tego polecenia otworzy tunel SSH do routera. Jeden koniec tunelu będzie na interfejsie pętli
zwrotnej naszego komputera na porcie `12345` . Natomiast drugi koniec na pętli zwrotnej routera, z
tym, że na porcie `80` .

Zostawiamy terminal otwarty i odpalamy przeglądarkę. W pasku adresu wpisujemy `localhost:12345` ,
jako, że to jest nasza lokalna końcówka tunelu. Wszystkie pakiety, które zostaną przesłane na ten
adres i ten port, zostaną wrzucone w tunel SSH i polecą nim na port 22 routera. Tam zostaną odebrane
na interfejsie WAN i przekazane na interfejs pętli zwrotnej na port 80, gdzie nasłuchuje serwer www,
stąd nazwa SSH port forwarding.

Zamknięcie terminalu, czy wylogowanie się z routera, spowoduje zamknięcie tunelu i odcięcie nas od
panelu administracyjnego.

## Uproszczony SSH port forwarding

Wpisywanie za każdym razem tego długiego polecenia nie jest zbyt wygodne. Zamiast zestawiać tunel
SSH za jego pomocą, możemy skonfigurować sobie takie połączenie w pliku `~/.ssh/config` . W tym celu
musimy dodać do tego pliku poniższy blok kodu:

    Host 192.168.1.235
        User root
        Port 22
        LocalForward 127.0.0.1:12345 127.0.0.1:80

Teraz zestawianie tunelu SSH upraszcza się do standardowego polecenia SSH:

    $ ssh 192.168.1.235

Jeśli nie chce nam się wpisywać hasła za każdym razem, gdy wydajemy to powyższe polecenie, to zawsze
możemy [zaimplementować sobie klucze SSH][1].

[1]: /post/dd-wrt-dostep-do-routera-telnet-ssh-panel-web/
