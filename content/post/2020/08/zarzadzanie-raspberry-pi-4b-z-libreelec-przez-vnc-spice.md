---
author: Morfik
categories:
- RaspberryPi
- Linux
date:    2020-08-25 20:08:00 +0200
lastmod: 2020-08-25 20:08:00 +0200
published: true
status: publish
tags:
- raspberry-pi-4b
- kodi
- xbmc
- libreelec
- vnc
- spice
- sieć
- debian
title: Zarządzanie Raspberry Pi 4B z LibreELEC przez VNC/SPICE
---

Do konfiguracji [systemu LibreELEC][1], który jest zainstalowany na moim Raspberry Pi 4B nie
potrzebuję żadnego monitora czy wyświetlacza, a wszystkie prace administracyjne związane z tym
małym komputerem ogarniam po SSH. Niemniej jednak, są pewne rzeczy, których się po SSH nie da
skonfigurować. Chodzi generalnie o konfigurację Kodi i sprawy związane z domowa wideoteką. Ten
Raspberry Pi 4B jest podłączony do TV, zatem bez problemu można przez graficzny interfejs Kodi
skonfigurować to, co potrzeba. Niemniej jednak konfigurowanie Kodi przez aplikację na Androida
([Kore][2]) nie należy do najłatwiejszych zadań. Nie uśmiecha mi się też ciągle latać z myszą i
klawiaturą w celu podłączania ich do portów USB maliny. Nie mam też w zasadzie zewnętrznego
monitora HDMI, bo od lat używam jedynie laptopów. Pomyślałem zatem, by zrobić użytek z protokołu
[VNC][3]/[SPICE][4] i przesłać obraz z Raspberry Pi 4B przez sieć do mojego laptopa. W ten sposób
odpadłoby mi ciągłe przemieszczanie się między pokojami i byłbym w stanie podejrzeć obraz TV na
ekranie monitora w moim laptopie przy jednoczesnym zachowaniu możliwości używania myszy i
klawiatury, co by mi bardzo zaoszczędziło czas przy konfiguracji Kodi.

<!--more-->
## Dodatek Raspberry Pi VNC dla Kodi

Póki co nie udało mi się znaleźć żadnego dodatku dla Kodi, który by oferował serwer SPICE. Jedyne
czego się doszukałem to [addon Raspberry Pi VNC][5], który instaluje serwer VNC. Problem z VNC w
przypadku mojego Raspberry Pi 4B jest bardzo podobny do tego, który miałem przy maszynach
wirtualnych -- słaba wydajność, choć do konfiguracji Kodi ten protokół powinien wystarczyć.

By zainstalować ten dodatek, trzeba przejść w Kodi kolejno do `Dodatki` =>
`Zainstaluj z repozytorium` => `LibreELEC Add-ons` => `Usługi` i wybrać pozycję `Raspberry Pi VNC` :

![]({{< baseurl >}}/img/2020/08/001-raspberry-pi-libreelec-kodi-xbmc-vnc-spice-addon-install.png#huge)

Jak większość dodatków, tak samo i `Raspberry Pi VNC` posiada parę opcji, które możemy dostosować,
by zmienić nieco zachowanie samego addon'a:

![]({{< baseurl >}}/img/2020/08/002-raspberry-pi-libreelec-kodi-xbmc-vnc-spice-addon-config.png#huge)

Przy standardowych ustawieniach, obraz serwowany przez LibreELEC do klienta VNC przycina się dość
mocno, a mysz ma przy tym strasznego laga. Możemy zatem podbić FPS z domyślnych `15` na `25` , co
powinno trochę pomóc. Przy zaznaczeniu opcji `relative` wskaźnik myszy staje się niedokładny i
odbiega dość znacznie od faktycznego położenia kursora. Dlatego lepiej nie zaznaczać tej opcji.
Można by przypuszczać, że `multi-threaded` ma coś wspólnego z szeroko rozumianą wielowątkowością
(technologia HT) ale tak nie jest. Zaznaczenie tej opcji sprawi jedynie, że proces serwera VNC
będzie uruchomiony w osobnym wątku, co powinno nieco poprawić wydajność. Można niby zaznaczyć
jeszcze opcję `downscale` , co dość znacznie upłynni operowanie na Kodi ale kosztem jakości obrazu.
Ta opcja w zasadzie przydaje się, gdy mamy do czynienia z wysokiej rozdzielczości TV, bo obraz
zostanie przeskalowany do 1/4 oryginału, przez co zmniejszy się dość znacznie zapotrzebowanie na
zasoby sprzętowe oraz odciążymy trochę sieć. Przy tych odbiornikach dysponującymi mniejszą
rozdziałką, takie przeskalowanie sprawi, że na monitorze komputera będziemy mieć małe okienko. Po
jego powiększeniu czy zmaksymalizowaniu, jakość wyświetlanego kontentu dość znacznie się obniży. Do
sterowania Kodi w zupełności wystarczy ale raczej tylko do tego.

Nie wiem czy w kwestii słabej wydajności VNC na Raspberry Pi 4B z LibreELEC/Kodi da radę coś więcej
zdziałać. Procesor nie jest wykorzystywany nawet w 1/3. Ze znalezionych informacji wychodzi, że
jednak to max co można z serwera VNC wyciągnąć pod LibreELEC/Kodi, a na serwer SPICE póki co raczej
nie ma co liczyć. Tak czy inaczej, lepszy rydz nisz nic.

Po skonfigurowaniu serwera VNC, powinien on zostać automatycznie uruchomiony na Raspberry Pi 4B.
Czy tak się w istocie stało możemy naturalnie zweryfikować przy pomocy `netstat` lub `ps` .
Wystarczy zalogować się na malinę po SSH i wydać jedno z tych poniższych poleceń:

    LibreELEC:~ # ps aux | grep vnc
      983 root      0:00 /bin/sh /storage/.kodi/addons/service.system.dispmanx_vnc/bin/dispmanx_vncserver-service
      992 root      3:45 dispmanx_vncserver -p 5900 -s 0 -t 20 -a -m -d

    LibreELEC:~ # netstat -napletu | grep  vnc
    tcp        0      0 0.0.0.0:5900            0.0.0.0:*               LISTEN      992/dispmanx_vncser
    tcp        0      0 :::5900                 :::*                    LISTEN      992/dispmanx_vncser

Jak widać, mamy dwa procesy. Zwykle byłby tutaj jeden proces ale za sprawą zaznaczenia opcji
`multi-threaded` mamy ten dodatkowy dla samego serwera VNC. Ten serwer nasłuchuje zapytań z sieci
na standardowym porcie `5900` i protokole `TCP` .

## Jak podłączyć się do serwera VNC

Pozostało nam już w zasadzie podłączenie się do serwera VNC. Tylko jak to zrobić? Potrzebne nam
jest jak zwykle dodatkowe oprogramowanie. W przypadku Debiana możemy doinstalować pakiet
`virt-viewer` , który to zawiera narzędzie `remote-viewer` , przy pomocy którego możemy połączyć
się z serwerami VNC/SPICE. Po zainstalowaniu pakietu, w terminalu wpisujemy to poniższe polecenie:

    $ remote-viewer vnc://192.168.1.239:5900

W przypadku poprawnej konfiguracji maliny, powinno nam się pokazać okienko z interfejsem Kodi:

![]({{< baseurl >}}/img/2020/08/003-raspberry-pi-libreelec-kodi-xbmc-vnc-spice-remote-viewer.png#big)

## Panel webowy Kodi

Myślałem, że jakoś lepiej wypadnie sprawa z łączeniem się do Raspberry Pi 4B z wykorzystaniem
protokołu SPICE/VNC. Szkoda trochę, że SPICE póki co nie da się w ogóle zaimplementować mając
wgrany LibreELEC, a słaba wydajność VNC skłoniła mnie do włączenia obsługi panelu webowego Kodi.
Jakby nie patrzeć, Kodi może też być zarządzany z poziomu WWW. Do tej pory jednak nie interesował
mnie ten jego webowy interfejs ale postanowiłem się mu przyjrzeć nieco bliżej.

![]({{< baseurl >}}/img/2020/08/004-raspberry-pi-libreelec-kodi-xbmc-web-panel.png#huge)

No trzeba przyznać, że wygląda trochę strasznie ale można przy jego pomocy chyba skonfigurować
każdy aspekt pracy Kodi, przez co wydaje się być idealną alternatywą dla niewydolnego VNC.


[1]: https://libreelec.tv/
[2]: https://kodi.wiki/view/Kore
[3]: https://en.wikipedia.org/wiki/Virtual_Network_Computing
[4]: https://en.wikipedia.org/wiki/Simple_Protocol_for_Independent_Computing_Environments
[5]: https://github.com/patrikolausson/dispmanx_vnc
