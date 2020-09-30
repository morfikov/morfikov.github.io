---
author: Morfik
categories:
- Linux
date: "2015-06-04T15:51:37Z"
date_gmt: 2015-06-04 14:51:37 +0200
published: true
status: publish
tags:
- email
- pliki
- katalogi
- thunderbird
title: Kompaktowanie katalogów w Thunderbird
---

Postanowiłem się w końcu wziąć za porządki związane z wiadomościami pocztowymi i RSS'ami, bo katalog
Thunderbird'a już zajmował prawie 650 MiB. Jakby nie patrzeć, to trochę dużo, biorąc pod uwagę, że
są to głównie wiadomości tekstowe. W sumie to miałem tam archiwum wszystkich maili z 4-5 ostatnich
lat i trochę się tego nazbierało. Nie byłoby tego wpisu gdyby nie fakt, że nawet usunięcie 120 tyś.
wiadomości praktycznie nie wpłynęło na zajmowane przez nie przestrzeni na dysku.

<!--more-->
## Czym jest kompaktowanie folderów i jak je włączyć

Jak [możemy wyczytać tutaj](http://kb.mozillazine.org/Thunderbird_:_Tips_:_Compacting_Folders) ,
Thunderbird wykorzystuje kompaktowanie folderów w celu lepszej wydajności. Polega to na tym, że po
usunięciu wiadomości, te może i znikają nam sprzed oczu ale nie są kasowane, przynajmniej nie w
takim wymiarze tego słowa jakbyśmy przypuszczali. Nie chodzi też o przenoszenie wiadomości do kosza
i jego opróżnianie, bo to zwyczajnie nic nie da. Te wszystkie skasowane wiadomości będą ciągle
obecne po usunięciu, aż do momentu kompaktowania konkretnych folderów.

Kompaktowanie z kolei nie ma nic wspólnego z archiwizacją, tj. kompresją danych w celu
zaoszczędzenia miejsca. Wręcz odwrotnie właśnie, im częściej będziemy kompaktować foldery, tym
mniej będą one zajmować miejsca, co przyczyni się do poprawy wydajności samej aplikacji. Poza tym,
jeśli nie wykonujemy tej czynności regularnie, możemy doświadczyć dziwnych zjawisk, jak np.
powracanie usuniętych wiadomości, lub też nieprawidłowe daty w wyniku zagubienia się nagłówka takiej
wiadomości. Thunderbird potrafi automatycznie przeprowadzać kompaktowanie folderów, z tym, że trzeba
mieć zaznaczoną odpowiednią opcję (Edit => Preferences => Advanced => Network & Disk Space):

![](/img/2015/06/1.thunderbird-kompaktowanie-folderow.png#big)

U mnie jakimś cudem była ona odznaczona i dlatego właśnie ten katalog się rozrósł tak potwornie. Po
dokonaniu kompaktowania, z tych 650MiB zjechałem do nieco ponad 200MiB. Także jest spora różnica.

By nie być ciągle nurtowanym pytaniem o to czy wyrażamy zgodę na automatycznie kompaktowanie
folderów, możemy przestawić pewną opcję w Thunderbird, by pozbyć się tego monitu. W tym celu
przechodzimy pod Edit => Preferences => Advanced => General:

![](/img/2015/06/2.thunderbird-config-editor.png#big)

Tam klikamy w `Config Editor` i otworzy nam się okienko z szeregiem parametrów do ustawienia.
Wyszukujemy `mail.purge.ask` i przestawiamy jego wartość na `False` :

![](/img/2015/06/3.thunderbird-about-config.png#huge)

Jeśli korzystamy z protokołu IMAP, to powinniśmy także zaznaczyć dwie inne opcje, z tym, że te
akurat są specyficzne dla tego typu kont i można je znaleźć w Edit => Account Settings, a
konkretnie, po wybraniu konta przechodzimy do `Server Settings` . I tam zaznaczamy te dwie poniższe
opcje:

![](/img/2015/06/4.thunderbird-kompaktowanie-imap.png#small)

Dzięki nim, folder odebranych i kosz będą automatycznie kompaktowane przy zamykaniu Thunderbird'a.

## Odchudzanie pliku global-messages-db.sqlite

Kompaktowanie nie wpływa jednak na rozmiar pliku `global-messages-db.sqlite` . Jest on [używany do
indeksowania wiadomości](https://support.mozilla.org/en-US/kb/rebuilding-global-database), tak by
szybciej i łatwiej było można znaleźć interesujące nas informacje. Z tym, że zasada jest zawsze taka
sama, im więcej mamy wiadomości, tym większy będzie ten plik i co jakiś czas trzeba przebudować bazę
w nim przechowywaną. By tego dokonać, musimy pierw usunąć ten plik z katalogu Thunderbird'a. Baza
zostanie utworzona na nowo po uruchomieniu aplikacji, zaś postęp prac możemy obserwować przez
menadżer aktywności (Tools => Activity Manager):

![](/img/2015/06/5.thunderbird-activity-manager.png#big)

U mnie po przebudowanie globalnej bazy indeksów, rozmiar pliku `global-messages-db.sqlite`
zmniejszył się ze 168MiB do niecałych 7MiB.
