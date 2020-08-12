---
author: Morfik
categories:
- Linux
date: "2016-05-23T20:09:42Z"
date_gmt: 2016-05-23 18:09:42 +0200
published: true
status: publish
tags:
- dns
- systemd
- sieć
- opendns
title: Konfiguracja DDNS dla OpenDNS
---

Ludzkość w dalszym ciągu siedzi na przestarzałym już od prawie 20 lat protokole IPv4. Nie widać, też
by w najbliższym czasie coś miało się w tej kwestii zmienić. Można, co prawda, wykupić sobie stały
adres IP ale to kosztuje, no i płacimy za coś co powinniśmy mieć w standardzie, gdyby ludzie w końcu
zaczęli korzystać z IPv6. Niemniej jednak, by te wszystkie nasze maszyny podłączyć jakoś do sieci,
potrzebne nam są prywatne adresy IP + NAT lub też dynamicznie zmieniające się adresy publiczne.
Bywają też przypadki, że mamy przydzielane dynamicznie adresy z puli prywatnej, np. w wyniku dbania
o prywatność w sieciach WiFi przez [generowanie sobie przy każdym połączeniu losowego adresu
MAC]({{< baseurl >}}/post/jak-przypisac-losowy-adres-mac-interfejsu/). Zwykle w takiej sytuacji
zmienia nam się adres zewnętrzny (publiczny), który wskazuje na jeden z adresów naszego ISP. Taki
zmieniający się adres powoduje problemy przy konfiguracji poszczególnych usług sieciowych, np. gdy w
grę wchodzi konfiguracja filtra DNS, który jest zapewniany przez OpenDNS. By tego typu niedogodności
rozwiązać, możemy posłużyć się [DDNS (dynamic DNS)](https://pl.wikipedia.org/wiki/DDNS). Za każdym
razem, gdy adres IP ulega zmianie, klient DDNS informuje o tym fakcie skonfigurowane usługi. W tym
artykule przyjrzymy się nieco bliżej temu mechanizmowi.

<!--more-->
## OpenDNS na zmiennym adresie IP

Przed czytaniem dalszej części tego wpisu, przydałoby się posiadać już kontro w serwisie
[OpenDNS](https://www.opendns.com/). Utworzenie konta i dostosowanie ustawień raczej nie powinno
sprawić problemów, zatem nie będziemy tutaj tego procesu opisywać. To co nas interesuje, to aktualny
adres IP, który zostanie nam zwrócony tuż po zalogowaniu się. W tym przypadku mamy:

![]({{< baseurl >}}/img/2016/05/1.ddns-opendns.png)

Niemniej jednak, jeśli przejdziemy na zakładkę SETTINGS, to figuruje tam poniższa sieć:

![]({{< baseurl >}}/img/2016/05/2.ddns-opendns.png)

Widzimy na powyższym obrazku dwa adresy IP. Ten na górze jest aktualnie przydzielony w konfiguracji,
zaś ten na dole został automatycznie wykryty prze OpenDNS. Nie pasują one do siebie i w tym cały
problem. W takim przypadku, usługa DNS jest skonfigurowana nie na ten adres, który potrzeba. W
efekcie, wszystkie przyblokowane przez nas domeny nie mają żadnej mocy. W dalszym ciągu możemy
korzystać z usług OpenDNS ale nie mamy żadnego wpływu na konfigurację domen. Musimy zatem nauczyć
nasz system, by zaktualizował automatycznie ten powyższy adres, gdy tylko adres IP ulegnie zmianie.
W debianie możemy to zrobić za sprawą pakietu `ddclient` . Jest on standardowo w repozytoriach i z
jego instalacja nie powinno być problemów. Być może będziemy także musieli doinstalować pakiet
`libio-socket-ssl-perl` , by włączyć wsparcie dla szyfrowanej komunikacji.

## Usługa dla systemd

W pakiecie `ddclient` nie ma standardowo żadnej usługi pod systemd. Możemy ją sobie zrobić sami
przez utworzenie pliku `/etc/systemd/system/ddclient.service` i dodanie w nim poniższej zawartości:

    [Unit]
    Description=Dynamic DNS service update utility (ddclient)
    Documentation=man:ddclient(8)
    Wants=network-online.target
    Before=multi-user.target
    After=network-online.target
    Conflicts=shutdown.target

    [Service]
    Type=forking
    RemainAfterExit=yes
    ExecStart=/usr/sbin/ddclient
    PIDFile=/var/run/ddclient.pid

    [Install]
    WantedBy=multi-user.target

Dodajemy teraz tę usługę do autostartu systemu:

    # systemctl daemon-reload
    # systemctl enable ddclient.service

## Konfiguracja klienta DDNS

Konfiguracja tego klienta DDNS znajduje się w pliku `/etc/ddclient.conf` . To w nim musimy dodać
konfigurację usługi OpenDNS. [Zgodnie z informacją pod tym
linkiem](https://support.opendns.com/hc/en-us/articles/227987727), musimy dopisać poniższe linijki:

    # cat /etc/ddclient.conf
    syslog=yes
    #verbose=yes
    #mail=morfik
    #mail-failure=morfik
    pid=/var/run/ddclient.pid
    daemon=600

    use=web, web=checkip.dyndns.com/, web-skip='IP Address'
    #use=if, if=eth1
    ssl=yes
    server=updates.opendns.com
    protocol=dyndns2
    login=
    password=
    home

Pierwsza część składa się z opcji dotyczących demona DDNS i raczej nic tutaj nie trzeba zmieniać.
Opcja `daemon` to interwał (w sekundach), co który dokonuje się aktualizacja adresu IP. Druga część
zaś dotyczy konfiguracji OpenDNS. Linijka z `use=web` określaja z jakiej usługi do sprawdzania
adresu IP będziemy korzystać. W tym przypadku mamy `dyndns.com` . Jeśli chcemy by demon operował
tylko na określonym interfejsie, to podajemy go w linijce z `use=if` . Upewnijmy się, że korzystamy
z szyfrowanego połączenia( `ssl=yes` ). Opcje `server` i `protocol` określają gdzie przesyłać
zapytania oraz z jakiego protokołu będziemy korzystać. W `login` oraz `password` podajemy dane do
logowania w serwisie OpenDNS. Ostatnia zaś opcja, tj. `home` , to nazwa naszej sieci w ustawieniach
OpenDNS. Można ją odczytać z panelu na stronie. Upewnijmy się także, że ten powyższy plik posiada
odpowiednie uprawnienia:

    # chmod 600 /etc/ddclient.conf

Wracamy teraz do panelu dostępnego na stronie OpenDNS. Wybieramy kolejno zakładkę SETTINGS, naszą
sieć oraz z menu po lewej stronie "Advanced Settings". Zaznaczamy tutaj opcję: `Enable dynamic IP
update ` :

![]({{< baseurl >}}/img/2016/05/3.ddns-opendns.png)

Sprawdzamy teraz całą konfigurację poniższym poleceniem:

    # ddclient -daemon=0 -debug -verbose -noquiet

W terminalu powinien się pojawić dość obszerny komunikat, a w logu ta oto poniższa wiadomość:

![]({{< baseurl >}}/img/2016/05/5.ddns-opendns.png)

Jeśli adres IP uległ zmianie, tak jak to możemy zaobserwować na powyższym obrazku, oznacza, to, że
wszystko działa jak należy. Możemy także przejść na stronę OpenDNS i sprawdzić czy tam w
konfiguracji jest uwzględniony nowy adres IP:

![]({{< baseurl >}}/img/2016/05/4.ddns-opendns.png)

## DNS-O-Matic od OpenDNS

W przypadku, gdy mamy do czynienia z szeregiem usług, które wymagają aktualizacji adresu IP, możemy
zaprzęgnąć do pracy z [DNS-O-Matic](http://www.dnsomatic.com/) od OpenDNS. Wykorzystuje ono klienta
DDNS, tego samego co wyżej sobie skonfigurowaliśmy, z tą różnicą, że upraszcza nieco zarządzanie
usługami. Operowanie na DNS-O-Matic sprowadza się do dodania odpowiedniej usługi:

![]({{< baseurl >}}/img/2016/05/6.ddns-ddclient-dns-o-matic.png)

Następnie konfigurujemy sobie tą usługę. W tym przypadku dodałem OpenDNS:

![]({{< baseurl >}}/img/2016/05/7.ddns-ddclient-dns-o-matic.png)

Po skonfigurowaniu, usługa powinna oczekiwać na pierwszy update:

![]({{< baseurl >}}/img/2016/05/8.ddns-ddclient-dns-o-matic.png)

W tej chwili musimy nieco zmienić konfigurację klienta `ddclient` w pliku `/etc/ddclient.conf`.
Przepisujemy ją do poniższej postaci:

    syslog=yes
    #verbose=yes
    #mail=morfik
    #mail-failure=morfik
    pid=/var/run/ddclient.pid
    daemon=600

    use=web, web=myip.dnsomatic.com
    protocol=dyndns2
    #use=if, if=bond0
    ssl=yes
    server=updates.dnsomatic.com
    login=
    password=
    all.dnsomatic.com

I ponownie testujemy czy aby ustawienia są poprawne przy pomocy poniższego polecenia:

    # ddclient -daemon=0 -verbose -noquiet

Jeśli połączenie zakończy się sukcesem, to na stronie powinniśmy ujrzeć poniższy komunikat:

![]({{< baseurl >}}/img/2016/05/9.ddns-ddclient-dns-o-matic.png)
