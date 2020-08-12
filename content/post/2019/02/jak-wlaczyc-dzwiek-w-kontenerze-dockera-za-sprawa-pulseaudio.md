---
author: Morfik
categories:
- Linux
date: "2019-02-16T13:22:40Z"
published: true
status: publish
tags:
- docker
- debian
- pulseaudio
title: Jak włączyć dźwięk w kontenerze Docker'a za sprawą PulseAudio
---

Jakiś czas temu postanowiłem przetestować sposób zamknięcia graficznych aplikacji w kontenerze
Docker'a. Całe rozwiązanie zostało opisane na
przykładzie [skonteneryzowania przeglądarki Firefox]({{< baseurl >}}/post/uruchamianie-graficznych-aplikacji-w-kontenerach-dockera/).
Ten opisany w podlinkowanym artykule pomysł był nawet całkiem przyzwoity ale nie nadaje się on, gdy
w grę wchodzą programy odtwarzające dźwięk. No może to za dużo powiedziane, że się nie nadaje, ale
z pewnością brakuje mu jednego istotnego elementu. Nawet ta przykładowa przeglądarka internetowa
jest w stanie odtwarzać dźwięki jeśli się odwiedzi stosowną stronę WWW. Standardowo jednak nic nie
usłyszymy w głośnikach, gdy odpalimy dajmy na to stronę YouTube i puścimy jakiś materiał video.
Dlatego też wypadałoby skonfigurować dźwięk i przesłać go do serwera PulseAudio, który będzie
odpalony na naszym linux'owym hoście. Kiedyś już tego typu rozwiązanie nawet opisywałem na
przykładzie [zintegrowania PulseAudio z kontenerami LXC]({{< baseurl >}}/post/pulseaudio-i-przesylanie-dzwieku-przez-siec/).
Okazuje się, że tamto rozwiązanie znajduje również zastosowanie w przypadku Docker'a. Trzeba tylko
nieco inaczej skonfigurować kontener i właśnie tej kwestii będzie dotyczył niniejszy wpis.

<!--more-->
## Konfiguracja dźwięku w kontenerze Docker'a

By skonfigurować kontener Docker'a, tak by był on w stanie odtwarzać dźwięk, trzeba w nim
doinstalować pakiet `libpulse0` . Edytujemy zatem plik `dockerfile` i dopisujemy do linijki z `RUN`
wspomniany wyżej pakiet:

    ...
    RUN apt-get update && apt-get install -y \
        firefox \
        libpulse0 \
    && rm -rf /var/lib/apt/lists/*
    ....

Musimy teraz przebudować jeszcze kontener:

    $ docker-compose build

Kontener posiada już wszystkie niezbędne elementy ale musimy jeszcze wskazać tej doinstalowanej
bibliotece PulseAudio, gdzie ma szukać serwera, do którego będzie przesyłać dźwięk przez sieć.
W tym celu trzeba wyeksportować dodatkową zmienną `PULSE_SERVER` za pomocą pliku
`docker-compose.yml` :

    ...
    services:
      browser:
        ...
        environment:
          - DISPLAY=$DISPLAY
          - PULSE_SERVER=192.168.42.247
        ...

Zmienna `PULSE_SERVER` przyjmuje w argumencie adres IP serwera, do którego będzie przesyłany dźwięk.
Zwykle podajemy tutaj adres IP hosta. Nic nie stoi na przeszkodzie by przesłać dźwięk z kontenera
Docker'a na dowolną maszynę mającą linux'a na pokładzie. Wystarczy tylko podać jej adres IP.

## Konfiguracja PulseAudio na potrzeby Docker'a

Musimy jeszcze odpowiednio skonfigurować serwer PulseAudio, by był on w stanie odebrać przesyłane
zapytania od klientów. Standardowo PulseAudio nie akceptuje żadnych połączeń sieciowych. Trzeba
zatem edytować plik `/etc/pulse/default.pa` i dopisać w nim poniższą linijkę:

    load-module module-native-protocol-tcp auth-ip-acl=10.10.2.0/24

Trzeba tutaj określić sieć lub adres IP w argumencie `auth-ip-acl` . W tym przypadku jest podana
sieć 10.10.2.0/24 i tylko z niej serwer PulseAudio będzie akceptował połączenia.

Restartujemy na koniec jeszcze serwer dźwięku:

    $ pulseaudio -k
    $ pulseaudio -D

I odpalamy kontener. Jeśli wszystko dobrze skonfigurowaliśmy, to odpalając przykładowy film na YT,
powinniśmy usłyszeć dźwięk. Poniżej fotka obrazująca całe przedsięwzięcie:

![]({{< baseurl >}}/img/2019/02/001-docker-debian-linux-pulseaudio-audio-network.png#huge)

Po prawej stronie widać systemowy mikser dźwięku, w którym widoczna jest pozycja `AudioIPC server`
i to ona właśnie odpowiada za obieranie dźwięku z kontenera Firefox'a. Jeśli jednak z jakiegoś
powodu go nie słyszymy, to być może nie mamy dodanych stosownych reguł w zaporze sieciowej. Serwer
dźwięku PulseAudio nasłuchuje na porcie TCP/4713 i trzeba naszemu hostowi umożliwić komunikację na
nim.
