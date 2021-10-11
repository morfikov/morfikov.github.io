---
author: Morfik
categories:
- RaspberryPi
date:    2020-08-22 14:35:00 +0200
lastmod: 2021-09-04 16:00:00 +0200
published: true
status: publish
tags:
- raspberry-pi-4b
- argon-one-pi4
- kodi
- xbmc
- libreelec
GHissueID: 38
title: Nie działa wentylator i przycisk Power w Argon One Pi4 na LibreELEC (Raspberry Pi 4B)
---

Ostatnio wpadł mi w łapki [minikomputer Raspberry Pi 4B][1] (wersja z 2G RAM). Chciałem wykorzystać
go w roli serwera multimediów na bazie Kodi/XBMC i podpiąć go pod TV. Jako, że procesory tych
malinek czasem potrafią się mocniej rozgrzewać, to do całego zestawu wziąłem również [aluminiową
obudowę z wentylatorem Argon One Pi4][2]. Po pobraniu [najnowszego Raspberry Pi OS][3],
zainstalowaniu go oraz dociągnięciu skryptu producenta obudowy, przyszła pora na poddanie tego RPI
stres testowi, w którym temperatura CPU nie wzrosła w zasadzie powyżej 52°C. Niemniej jednak,
ten Raspberry Pi ma realizować dwa zadania, tj. serwować filmy z domowej wideoteki oraz umożliwić
oglądanie VoD/strimingu video, w tym też materiały na YouTube. Dlatego właśnie potrzebny jest
Kodi/XBMC, który nie wymagałby większych nakładów pracy przy konfiguracji tej zabawki. Tak właśnie
natrafiłem na [projekt LibreELEC][4], który to dostarcza system operacyjny na bazie linux'a wraz z
wbudowanym Kodi uruchamiającym się automatycznie bez zbędnej dodatkowej konfiguracji. Problem z
tym LibreELEC jest natomiast taki, że standardowo nie da rady ani zaprogramować przycisku Power w
obudowie Argon One Pi4, ani ustawić progów temperatury dla jej wentylatora. Niemniej jednak,
istnieje sposób by to zrobić i właśnie temu zagadnieniu będzie poświęcony niniejszy wpis.

<!--more-->
## Oprogramowanie dla obudowy Argon One Pi4

W pudełku, w którym była obudowa Argon One Pi4, znajdowała się także niezbyt obszerna instrukcja z
informacją na temat konfiguracji przycisku Power oraz wentylatora. Producent obudowy podaje także
krótkie polecenie, które trzeba uruchomić w terminalu na Raspberry Pi 4B:

    $ curl https://download.argon40.com/argon1.sh | bash

To rozwiązanie jak najbardziej zadziała na standardowym systemie maliny, tj. Raspberry Pi OS
(kiedyś znany jako Raspbian). Niemniej jednak, w przypadku LibreELEC, jego obrazy są tak
skonfigurowane, by domyślnie nie można było w systemie jako takim wprowadzać żadnych zmian (główny
system plików jest [na bazie SquashFS][9], tj. ten sam, który stosowany jest np. w systemach live z
linux'ami). Takie podejście naturalnie poprawia bezpieczeństwo naszego domowego centrum multimediów
ale niestety z racji, że system plików SquashFS jest tylko do odczytu, to powoduje on problemy z
obudową Argon One Pi4.

W przypadku uruchomienia tego powyższego skryptu, zostaną nam zwrócone takie oto błędy:

    LibreELEC:~ # curl https://download.argon40.com/argon1.sh | bash

    bash: syntax error: unexpected "("

    curl: (23) Failed writing body (538 != 1291)

## Jak pogodzić Argon One Pi4 z LibreELEC

Szukając informacji na temat przerobienia obrazu LibreELEC, znalazłem przez przypadek rozwiązanie,
które jest w stanie zaradzić zaistniałemu problemowi z obudową Argon One Pi4. Chodzi generalnie o
zainstalowanie `Raspberry Pi Tools` oraz `System Tools` z poziomu Kodi. Te dwa zestawy narzędzi
możemy wgrać przechodząc kolejno do `Dodatki` (Addons) > `Zainstaluj z repozytorium` (Install from
Repository) > `LibreELEC Add-ons` > `Programy` (Program Addons), tak jak to zostało zobrazowane na
poniższych fotkach:

![](/img/2020/08/001-argon-one-pi4-raspberry-pi-libreelec-kodi-addons-config.png#huge)

![](/img/2020/08/002-argon-one-pi4-raspberry-pi-libreelec-kodi-addons-config.png#huge)

![](/img/2020/08/003-argon-one-pi4-raspberry-pi-libreelec-kodi-addons-config.png#huge)

![](/img/2020/08/004-argon-one-pi4-raspberry-pi-libreelec-kodi-addons-config.png#huge)

![](/img/2020/08/005-argon-one-pi4-raspberry-pi-libreelec-kodi-addons-config.png#huge)

![](/img/2020/08/006-argon-one-pi4-raspberry-pi-libreelec-kodi-addons-config.png#huge)

![](/img/2020/08/007-argon-one-pi4-raspberry-pi-libreelec-kodi-addons-config.png#huge)

![](/img/2020/08/008-argon-one-pi4-raspberry-pi-libreelec-kodi-addons-config.png#huge)

Następnie musimy pobrać nieco [zmieniony skrypt producenta obudowy][5]. W tym przypadku należy
wpisać w terminalu na Raspberry Pi 4B to poniższe polecenie:

    LibreELEC:~ # curl https://download.argon40.com/argonone-setup-libreelec.sh | bash

![](/img/2020/08/009-argon-one-pi4-raspberry-pi-libreelec-script-exec-config.png#huge)

Uruchomienie skryptu `argonone-setup-libreelec.sh` powoduje utworzenie szeregu plików w katalogu
`/storage/` . W ten sposób w tym folderze pojawi się skrypt `argonone-config` , przy pomocy którego
możemy skonfigurować zachowanie wentylatora obudowy Argon One Pi4. W `/storage/` zostanie także
umieszczony inny skrypt, tj. `argonone-uninstall` , dzięki któremu to możemy odinstalować wszystkie
wgrane pliki i prawie przywrócić stan systemu LibreELEC do tego sprzed wprowadzania w nim zmian (by
to zrobić w pełni, potrzebna będzie manualna interwencja). Dalej, do katalogu `/storage/.config/`
powędrują dwa skrypty python'a, tj. `argononed-poweroff.py` odpowiedzialny za poprawne zamknięcie
systemu oraz `argononed.py` , którego zadaniem będzie monitorowanie przycisku Power oraz ogarnianie
pracy wentylatora. By te dwa ostatnie zadania mogły być realizowane, do katalogu
`/storage/.config/system.d/` zostanie wgrana usługa dla systemd w postaci pliku
`argononed.service` . Domyślnie też zostanie utworzone dowiązanie w
`/storage/.config/system.d/multi-user.target.wants/argononed.service` , przez co ta usługa będzie
podnoszona na starcie naszego Raspberry Pi 4B. Do katalogu `/storage/.config/` zostanie także
wgrany skrypt `shutdown.sh` , który będzie wywoływany przy zamykaniu/restartowaniu systemu. Jego
zadaniem jest właściwie przekierowanie obsługi tego procesu do skryptu `argononed-poweroff.py` . Z
kolei by wentylator działał tak jak tego od niego oczekujemy, tj. zmieniał prędkość swoich obrotów
wraz ze zmianą temperatury CPU, potrzebny nam jest jeszcze plik konfiguracyjny `argononed.conf` ,
który siedzi w katalogu `/storage/` .

Reasumując, pobrany skrypt producenta obudowy Argon One Pi4 `argonone-setup-libreelec.sh`
wygeneruje na Raspberry Pi 4B te poniższe pliki:

    /storage/argonone-config
    /storage/argonone-uninstall
    /storage/argononed.conf
    /storage/.config/argononed-poweroff.py
    /storage/.config/argononed.py
    /storage/.config/system.d/argononed.service
    /storage/.config/system.d/multi-user.target.wants/argononed.service
    /storage/.config/shutdown.sh

### Parametry dtparam=i2c=on oraz enable_uart=1 w /flash/config.txt

Poza tymi plikami, skrypt `argonone-setup-libreelec.sh` wprowadza zmiany na partycji `/flash/` ,
która jest zamontowana standardowo w trybie tylko do odczytu. Zmiany te dotyczą [pliku
config.txt][10], który to w zasadzie robi za BIOS w komputerach Raspberry Pi.

Te dwie poniższe linijki będą dodane na końcu pliku `config.txt` :

    [all]
    ...
    dtparam=i2c=on
    enable_uart=1

Linijka z `dtparam=` ma za zadanie zmienić nieco konfigurację Raspberry Pi 4B i włączyć w nim
obsługę [szyny I2C][12], tak by nasza malina była w stanie komunikować się z obudową Argon One Pi4:

    LibreELEC:~ # i2cdetect -l
    i2c-1   i2c             bcm2835 I2C adapter                     I2C adapter

    LibreELEC:~ # i2cdetect -y 1
         0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
    00:          -- -- -- -- -- -- -- -- -- -- -- -- --
    10: -- -- -- -- -- -- -- -- -- -- 1a -- -- -- -- --
    20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
    30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
    40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
    50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
    60: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
    70: -- -- -- -- -- -- -- --

Bez `i2c=on` , polecenie `i2cdetect -l` nic nie zwróci, a skrypt `/storage/.config/argononed.py`
się wykrzaczy i zakończy poniższym błędem:

    LibreELEC:~ # /storage/.config/argononed.py
    Traceback (most recent call last):
      File "/storage/.config/argononed.py", line 12, in <module>
        bus = smbus.SMBus(1)
    IOError: [Errno 2] No such file or directory

Zatem bez `i2c=on` nie zadziała nam ani wentylator, ani przycisk Power w obudowie Argon One Pi4.

Parametr `enable_uart` zdaje się być zbędny jeśli chodzi o samą konfigurację wentylatora i
przycisku Power w obudowie Argon One Pi4. Niemniej jednak `enable_uart=1` aktywuje konsolę
szeregową UART ([Universal Asynchronous Receiver-Transmitter][11]). Powinno się zatem pojawić
urządzenie `/dev/ttyS0` albo `/dev/ttyAMA0` w zależności od tego, którą wersję Raspberry Pi
posiadamy. Przynajmniej tak piszą w dokumentacji. W przypadku mojego Raspberry Pi 4B, pojawiły się
oba te urządzenia.

Te dwie linijki z pliku `/flash/config.txt` przy ewentualnym odinstalowaniu oprogramowania
dostarczonego za sprawą skryptu `argonone-setup-libreelec.sh` nie są usuwane. Jeśli chcielibyśmy je
usunąć, to ręcznie musimy edytować plik `/flash/config.txt` uprzednio montując partycję `/flash/` w
trybie do zapisu:

    LibreELEC:~ # mount -o remount,rw /flash

Po skończonej edycji, montujemy tę partycję ponownie w tryb do odczytu:

    LibreELEC:~ # mount -o remount,ro /flash

Wszelkie zmiany dokonane w obrębie partycji `/flash/` będą aplikowane dopiero po ponownym
uruchomieniu systemu.

### Problemy z uruchamianiem usługi na LibreELEC 10

Parę dni temu została wypuszczona [nowa stabilna wersja LibreELEC][13], tj. v10 Matrix. Szereg
rzeczy w tej nowszej wersji zostało zmienionych, przez co skrypt do obsługi obudowy Argon One Pi4
przestał działać. Można ten problem poprawić w bardzo [prosty sposób][14]. Najpierw edytujemy plik
`/flash/config.txt` i dostosowujemy jego zawartość, tak by uwzględniała te dwie poniższe linijki:

    dtparam=i2c_vc=on
    dtparam=i2c_arm=on

Następnie edytujemy plik usługi systemd, tj. `/storage/.config/system.d/argononed.service` i
zmieniamy w nim linijkę z `StartExec` :

	[Unit]
	Description=Argon One Fan and Button Service
	After=multi-user.target
	[Service]
	Type=simple
	Restart=always
	RemainAfterExit=true
	#ExecStart=/usr/bin/python /storage/.config/argononed.py
	ExecStart=/bin/sh -c ". /etc/profile; exec /usr/bin/python /storage/.config/argononed.py"
	[Install]
	WantedBy=multi-user.target

## Konfiguracja pracy wentylatora Argon One Pi4

Po zrestartowaniu Raspberry Pi 4B, przycisk Power powinien zacząć działać w sposób opisany w
instrukcji obsługi obudowy. Zatem, gdy Raspberry Pi 4B jest wyłączony, to krótkie przyciśnięcie
przycisku Power włączy to urządzenie. Podczas pracy maliny, można ją zresetować przyciskając
przycisk Power dwa razy w krótkim odstępie czasu. Jeśli zaś przycisk Power zostanie wciśnięty i
przytrzymany od 3-5 sekund, to zostanie zainicjowana procedura wyłączenia systemu LibreELEC oraz
odcięcia zasilania od Raspberry Pi 4B. A jak ten przycisk przytrzymamy powyżej 5 sekund, to ten
nasz mały komputer zostanie siłowo wyłączony, co może uszkodzić systemy plików wszystkich
dysków/partycji, które mają je zamontowane w trybie do zapisu. Także lepiej nie trzymać tego
przycisku zbyt długo.

Jeśli zaś chodzi o pracę wentylatora obudowy Argon One Pi4, to przy domyślnej konfiguracji
dostarczonej przez skrypt `argonone-setup-libreelec.sh` , ten wiatrak będzie w zasadzie wyłączony
do momentu, aż temperatura procesora przekroczy 55°C. W takim przypadku, system ustawi moc
wentylatora na 10%. Jeśli temperatura CPU sięgnie 60°C, to ta moc zostanie zwiększona do 55%.
Gdyby zdarzyło się tak, że temperatura procesora przekroczy 65°C, to wentylator będzie działał
na pełnych obrotach i będzie przy tym dość głośny.

W przypadku, gdy te domyślne progi przełączania wentylatora nam nie odpowiadają i uważamy, że
temperatura CPU 55+°C to za dużo, to zawsze możemy zmienić konfigurację pracy wiatraka w
obudowie Argon One Pi4. Trzeba jednak mieć na uwadze, że może i zbyt wysoka temperatura jest w
stanie uszkodzić CPU ale wbudowane w Raspberry Pi 4B zabezpieczenia przed przegrzaniem się
procesora (CPU/GPU throttling) zaczynają się aktywować dopiero po przekroczeniu 80°C. W ten sposób,
CPU będzie działał ze 100% mocy do momentu osiągnięcia tych 80°C. Po ich przekroczeniu
zostanie załączony CPU throttling, który [obniży taktowanie procesora z 1,5 GHz do 1 GHz][8]. Jeśli
temperatura w dalszym ciągu będzie rosła, to po przekroczeniu 85°C aktywowany zostanie GPU
throttling. Zatem 55°C wydaje się być w miarę optymalnym rozwiązaniem, by ten wentylator
powoli zaczął się rozkręcać. Tak czy inaczej, progi pracy wiatraka można zmienić przez dostosowanie
wartości w pliku `/storage/argononed.conf` , przykładowo:

    55=10
    60=55
    65=100

Przed znakiem równości mamy temperatury CPU (w °C), przy których praca wentylatora będzie ulegać
zmianie, zaś po znaku równości jest wartość mocy wentylatora wyrażona w procentach. Wentylator
można regulować z dokładnością do 1%. Tych linijek, co są wyżej, może być więcej niż 3, co zezwala
na bardziej dokładne sprofilowanie wiatraka obudowy.

Aktualną temperaturę procesora Raspberry Pi 4B można podejrzeć w pliku
`/sys/class/thermal/thermal_zone0/temp` :

    # cat /sys/class/thermal/thermal_zone0/temp
    38459

By otrzymać wynik w °C, trzeba tę powyższą wartość podzielić przez 1000. W ten sposób otrzymujemy
~38,5°C . Zawsze też możemy [skorzystać z vcgencmd][7]:

    LibreELEC:~ # vcgencmd measure_temp
    temp=38.0'C

Temperaturę CPU naszej maliny możemy także sprawdzić w Kodi:

![](/img/2020/08/010-argon-one-pi4-raspberry-pi-libreelec-kodi-status-temp.png#huge)

## Podsumowanie

Raspberry Pi 4B potrafi się bardzo szybko rozgrzać do tych granicznych temperatur przy mocniejszym
obciążeniu CPU/GPU w przypadku, gdy nie stosujemy żadnego chłodzenia, nawet tego pasywnego w
postaci doczepionych do układów radiatorów. Może i ta obudowa Argon One Pi4 dla Raspberry Pi 4B do
najtańszych nie należy ale dość przyzwoicie jest w stanie schłodzić rozgrzaną malinę i nie dopuścić
by CPU/GPU throttling zdławił jej procesor, czego efektem była by dość spora utrata mocy (ok ~30%).
Niemniej jednak, przeciętne materiały video z domowej kolekcji filmów puszczanych za sprawą
LibreELEC/Kodi raczej nie powinny aż tak mocno utylizować zasobów tej maszyny. Dlatego też jeśli
nie mamy zamiaru się bawić w OC, to lepszym wyjściem będzie poszukanie tańszej obudowy, a nadwyżkę
hajsu przeznaczyć na więcej pamięci operacyjnej RAM w Raspberry Pi 4B.


[1]: https://botland.com.pl/pl/moduly-i-zestawy-raspberry-pi-4b/14646-raspberry-pi-4-model-b-wifi-dualband-bluetooth-2gb-ram-15ghz-765756931175.html
[2]: https://botland.com.pl/pl/obudowy-do-raspberry-pi-4b/15391-obudowa-do-raspberry-pi-4b-aluminiowa-z-wentylatorem-argon-one-szara.html
[3]: https://www.raspberrypi.org/downloads/raspberry-pi-os/
[4]: https://libreelec.tv/
[5]: https://download.argon40.com/argonone-setup-libreelec.sh
[6]: https://www.argon40.com/learn/index.php/2020/03/10/argon-one-installation-guide-for-libreelec/
[7]: https://www.raspberrypi.org/documentation/raspbian/applications/vcgencmd.md
[8]: https://www.raspberrypi.org/blog/thermal-testing-raspberry-pi-4/
[9]: https://en.wikipedia.org/wiki/SquashFS
[10]: https://www.raspberrypi.org/documentation/configuration/config-txt/
[11]: https://en.wikipedia.org/wiki/Universal_asynchronous_receiver-transmitter
[12]: https://en.wikipedia.org/wiki/I%C2%B2C
[13]: https://libreelec.tv/2021/08/26/libreelec-matrix-10-0/
[14]: https://forum.libreelec.tv/thread/23933-argon40-case-python-script-for-fan-control-problem-with-le-10-beta/?postID=155141#post155141
