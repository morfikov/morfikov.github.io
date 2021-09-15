---
author: Morfik
categories:
- Android
date:    2019-01-19 21:12:11 +0100
lastmod: 2019-01-19 21:13:11 +0100
published: true
status: publish
tags:
- smartfon
- adb
- magisk
- safetynet
- root
title: Jak przy pomocy Magisk'a pogodzić SafetyNet i ADB/USB debug
---

Do tej pory zbytnio nie interesowałem się zagadnieniami dotyczącymi mechanizmu [SafetyNet][1], który
ma na celu utrudnić nieco życie użytkownikom smartfonów z Androidem lubiącym posiadać pełny dostęp
do systemu swoich urządzeń za sprawą uzyskania praw administratora (root). To co się zmieniło na
przestrzeni ostatnich paru miesięcy, to fakt, że coraz więcej aplikacji polega na tym całym
SafetyNet, a przynajmniej ja zaczynam coraz częściej korzystać z tego typu oprogramowania. Jeśli
jednak nasze urządzenie nie przejdzie testów SafetyNet, to funkcjonalność aplikacji polegających na
tym mechanizmie może zostać dość znacznie ograniczona. Przykładem może być appka Revolut i jej
odblokowanie za pomocą czytnika linii papilarnych. Bez SafetyNet trzeba podawać PIN za każdym
razem, gdy się do tej aplikacji będziemy próbowali zalogować. Zwykle do obejścia SafetyNet używa
się Magisk'a ale w pewnych sytuacjach, nawet i on nie jest w stanie z tym zdaniem sobie poradzić,
przynajmniej nie bez dodatkowej konfiguracji. Jeśli na co dzień korzystamy z opcji debugowania
ADB/USB, to może nas spotkać nie lada dylemat -- ADB/USB debug vs. SafetyNet. Okazuje się, że można
pogodzić te dwie rzeczy.

<!--more-->
## ADB debug, a poziom logowania komunikatów

Próbując obejść SafetyNet, [trafiłem na informację][2], która obwieszcza, że Magisk nie jest w
stanie sobie poradzić z ukryciem faktu włączenia narzędzi deweloperskich, do których dostęp uzyskuje
się klikając parę razy na numer kompilacji oprogramowania zainstalowanego w telefonie. Dodatkowo,
Magisk ma problemy z ukryciem informacji o wykorzystywaniu debugowania ADB/USB. Pozornie zatem może
się wydawać, że nic nie da się zrobić i jeśli zdecydujemy się korzystać z ADB/USB debug, to możemy
zapomnieć o SafetyNet i vice versa. Mi jednak nie pasowała zbytnio żadna z tych opcji, bo ja
wykorzystuję `adb` do przesyłania plików na telefon, co działa o wiele lepiej niż protokół MTP.
Dlatego też postanowiłem poszukać nieco głębiej i ustalić dlaczego włączenie narzędzi deweloperskich
(względnie też tego ADB/USB debug) psuje SafetyNet.

Okazało się, że problemem nie są same opcje deweloperskie, ani tym bardziej debugowanie ADB/USB.
Kluczową rolę odgrywa tutaj logowanie komunikatów, a te z kolei mogą zawierać pewne newralgiczne
dane, zwłaszcza jeśli poziom logowania mamy ustawiony na debugowanie. A jak wiadomo, włączenie
ADB/USB debug, jak sama nazwa wskazuje z resztą, aktywuje mechanizm logowania na tym właśnie
poziomie. Rozwiązaniem tego całego problemu na linii ADB/USB debug vs. SafetyNet jest
skonfigurowanie poziomu logowania.

## Jak skonfigurować domyślny poziom logowania w Androidzie

Domyślny poziom logowania komunikatów w Androidzie można ustawić w pliku `build.prop` dodając do
niego klucz `log.tag=` . Domyślnie wartość tego klucza wskazuje na `M` lub `D` albo też w ogóle go
nie ma, przez co domyślnie będą logowane komunikaty na najniższym poziomie. Trzeba zatem w tym
kluczu ustawić `I` (od info). Można również ustawić `W` albo `E` jeśli chcemy mieć domyślnie tylko
ostrzeżenia lub błędy. Nie będziemy oczywiście ręcznie edytować tego pliku, bo mamy do tego celu
bardzo przydatny moduł Magisk'a zwany [MagiskHide Props Config][3].

### Moduł MagiskHide Props Config

Zadaniem modułu MagiskHide Props Config jest, jak nazwa wskazuje, konfiguracja kluczy, które można
ustawić przez plik `build.prop` . Moduł znajduje się w repozytorium Magisk'a i raczej nie powinno
być problemów z jego instalacją. Natomiast jeśli chodzi o konfigurację, to musimy pobrać sobie
[plik konfiguracyjny][4] i odpowiednio go przerobić.

Poniżej znajduje się mój plik:

    #!/system/bin/sh

    # MagiskHide Props Config
    # By Didgeridoohan @ XDA Developers

    CONFFINGERPRINT=""

    CONFPROPFILES=true

    CONFDEBUGGABLE=""
    CONFSECURE=""
    CONFTYPE=""
    CONFTAGS=""
    CONFSELINUX="0"

    CONFPROPS=""
    CONFPROPSPOST="
    log.tag=I
    "
    CONFPROPSLATE="
    log.tag=I
    "
    PROPOPTION=replace

    CONFDELPROPS=""
    DELPROPOPTION=replace

    CONFLATE=false
    CONFCOLOUR=enabled
    CONFWEB=enabled

    # =================================================================
    # ========================== Instructions =========================
    # =================================================================
    # Set the above variables to the desired prop/configuration values.

    # CONFFINGERPRINT should be set to the fingerprint of a ROM that passes
    # the ctsProfile check. See the prints.sh file for usable prints,
    # or the documentation for information on how to find one.
    # Note that Android builds after March 16 2018 often also need to match the Android
    # security patch date. Use the CONFPROPS setting to set ro.build.version.security_patch
    # to the matching date (example: 2018-10-05).

    # CONFPROPFILES should be set to "true" if you want to mask the file
    # values in build.prop and default.prop. For better root hiding.

    # The MagiskHide prop variables can be set as follows:
    # CONFDEBUGGABLE - 0 or 1 (set to "0" by MagiskHide - sensitive value is "1")
    # CONFSECURE - 0 or 1 (set to "1" by MagiskHide - sensitive value is "0")
    # CONFTYPE - user or userdebug (set to "user" by MagiskHide - sensitive value is "userdebug")
    # CONFTAGS - release-keys or test-keys (set to "release-keys" by MagiskHide - sensitive value is "test-keys")
    # CONFSELINUX - 0 or 1 (set to "0" by MagiskHide - sensitive value is "1")

    # CONFPROPS should contain any custom props and the value you want the module to set.
    # Any props you've previously edited in build.prop, and more, can be set like this.
    # Add them to the CONFPROPS variable according to the following example:
    # CONFPROPS="
    # ro.sf.lcd_density=320
    # ro.config.media_vol_steps=30
    # net.tethering.noprovisioning=true
    # "
    # Please observe that if the prop you're trying to set contains spaces, you'll
    # need to replace those spaces with "_sp_" (without the quotation marks).
    #
    # If you want a specific prop to run in either post-fs-data or late_start service,
    # use either CONFPROPSPOST or CONFPROPSLATE instead. Any props added to CONFPROPS
    # will run in the boot stage currently set in the module options (see CONFLATE below).
    #
    # With PROPOPTION you can decide if the current custom prop list should
    # be replaced, added to or preserved. Add the corresponding words "replace",
    # "add", or "preserve". The default option is to replace the list.
    # This option supersedes the preserve option described below, but only
    # for the CONFPROPS variables.

    # CONFDELPROPS is a list of props you want to remove from your system.
    # Be very careful when using this option, removing the wrong props might
    # cause issues.
    # Add the props you want to remove to the CONFDELPROPS variable according to
    # the following example:
    # CONFDELPROPS="
    # ro.sf.lcd_density
    # ro.config.media_vol_steps
    # net.tethering.noprovisioning
    # "
    #
    # With DELPROPOPTION you can decide if the current custom prop list should
    # be replaced, added to or preserved. Add the corresponding words "replace",
    # "add", or "preserve". The default option is to replace the list.
    # This option supersedes the preserve option described below, but only
    # for the CONFDELPROPS variable.

    # CONFLATE is by default set to "false". This loads the boot script during the
    # post-fs-data mode. If the setting is changed to "true", the boot script
    # will instead be loaded later during boot, in the late_start service mode. This is
    # useful if the module's boot script seems to be causing issues during boot.
    #
    # CONFCOLOUR and CONFWEB are the options for colour and automatic fingerprints
    # list update. See the module documentation for more details. Set to "enabled" or "disabled".

    # If any variables are left unset, that particular prop/configuration
    # will be cleared and the device/MagiskHide default values will be used.
    # If you want to keep any current module settings, add "preserve" to the variable.
    # Example:
    # CONFFINGERPRINT=preserve

    # When placed in /cache or the root of your internal storage, the module will load these
    # values during boot and the configuration file will be deleted. Keep a backup of the
    # file if you want to reuse it at a later time (clean ROM flash, etc).

    # For more information, see the documentation:
    # https://github.com/Magisk-Modules-Repo/MagiskHide-Props-Config/blob/master/README.md
    # and the support thread @ XDA Developers:
    # https://forum.xda-developers.com/apps/magisk/module-magiskhide-props-config-t3789228

Kluczowe w tym pliku są zmienne `CONFPROPS` , `CONFPROPSPOST` oraz `CONFPROPSLATE` . W tym
przypadku wszystkie trzy mają ustawiony `log.tag=I` . Tylko w ten sposób mój system był w stanie
honorować to ustawienie -- w innych przypadkach zwyczajnie je sobie nadpisywał, przez co SafetyNet
nie przechodził.

Tak uzupełniony plik zapisujemy pod nazwą `propsconf_conf` w katalogu `/cache/` na smartfonie
(wymagany root) i uruchamiamy ponownie urządzenie. Raz zaaplikowana konfiguracja dla modułu będzie
honorowana między restartami telefonu, choć plik `propsconf_conf` z katalogu `/cache/` zostanie
usunięty. Dlatego też dobrze jest mieć aktualną jego wersję na flash'u smartfona, by w razie czego
łatwo móc sobie zmienić ustawienia modułu.

## Sprawdzenie SafetyNet

Teraz już w zasadzie pozostaje nam do sprawdzenia czy nasz smartfon przechodzi testy SafetyNet. I
jak można zobaczyć na poniższej fotce, mój telefon radzi sobie z tym zadaniem całkiem nieźle.

![](/img/2019/01/001-magisk-safetynet-check-adb-usb-debug.png#medium).


[1]: https://lineageos.org/Safetynet/
[2]: https://www.didgeridoohan.com/magisk/MagiskHideBasics
[3]: https://forum.xda-developers.com/apps/magisk/module-magiskhide-props-config-t3789228
[4]: https://github.com/Magisk-Modules-Repo/MagiskHidePropsConf/blob/master/common/propsconf_conf
