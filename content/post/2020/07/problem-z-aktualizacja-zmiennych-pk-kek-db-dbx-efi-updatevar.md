---
author: Morfik
categories:
- Linux
date: "2020-07-30T20:30:00Z"
published: true
status: publish
tags:
- debian
- efi
- uefi
title: Problem z aktualizacją zmiennych PK, KEK, db i dbx via efi-updatevar
---

Jakiś czas temu opisywałem [konfigurację własnych kluczy EFI/UEFI][2], którymi można zastąpić te
wbudowane standardowo w firmware naszego komputera. W tamtym artykule napotkałem jednak dość dziwny
problem, za sprawą którego nie można było zaktualizować zmiennych `PK` , `KEK` , `db` i `dbx` przy
pomocy `efi-updatevar` z poziomu działającego linux'a. Gdy próbowało się te zmienne przepisać,
dostawało się błąd typu `Operation not permitted` . Niby system został uruchomiony w trybie `Setup
Mode` ale z jakiegoś powodu odmawiał on współpracy i trzeba było te zmienne aktualizować
bezpośrednio z poziomu firmware EFI/UEFI, co było trochę upierdliwe. Szukając wtedy informacji na
ten temat, jedyne co znalazłem, to fakt, że sporo osób ma podobny problem i najwyraźniej firmware
mojego laptopa jest ździebko niedorobiony, przez co `efi-updatevar` nie mógł realizować swojego
zdania. Rzeczywistość okazała się nieco inna, a rozwiązanie samego problemu było wręcz banalne.

<!--more-->
## Czemu efi-updatevar zwraca "Operation not permitted"

Niby własne klucze EFI/UEFI udało się ostatecznie wgrać do firmware, może nieco inną metodą i
trochę na około, ale przez cały ten czas ta sytuacja nie dawała mi spokoju. Może nie szukałem
aktywnie rozwiązania problem, bo w ostatecznie te klucze udało się wymienić ale nie lubię zostawić
niedokończonych spraw samym sobie, bo prędzej czy później one wracają w najmniej oczekiwanym
momencie. Jako, że ostatnio sporo się bawię narzędziem `strace` i przy okazji parę dni temu pojawił
się dość poważny [błąd w Grub2, który obchodzi mechanizm Secure Boot][3], to postanowiłem powrócić
do tematu i trochę go głębiej podrążyć.

Poniżej jest kawałek logu `strace` z zarejestrowanym błędem:

	# strace -f -o /tmp/strace.txt -s 2000 efi-updatevar -f dbx.esl dbx
	...
	13488 openat(AT_FDCWD, "/sys/firmware/efi/efivars/dbx-d719b2cb-3d3a-4596-a3bc-dad00e67656f", O_RDWR|O_CREAT|O_TRUNC, 0644) = -1 EPERM (Operation not permitted)
	13488 write(2, "Failed to update dbx: ", 22) = 22
	13488 write(2, "Operation not permitted\n", 24) = 24

Niemniej jednak, jak się podejrzy uprawnienia tego pliku
`/sys/firmware/efi/efivars/dbx-d719b2cb-3d3a-4596-a3bc-dad00e67656f` , to mamy coś takiego:

	# ls -al /sys/firmware/efi/efivars/dbx-d719b2cb-3d3a-4596-a3bc-dad00e67656f
	-rw-r--r-- 1 root root 0 2020-07-30 17:42:36 /sys/firmware/efi/efivars/dbx-d719b2cb-3d3a-4596-a3bc-dad00e67656f

Więc w czym problem? Niby root ma uprawnienia do zapisu tego pliku ale mimo to operacja zapisu jest
niedozwolona.

Z początku nie miałem pojęcia o co może chodzić. Góglając jednak, natrafiłem na [pakiet
secureboot-db][1], którego co prawda nie ma w Debianie ale jest za to w Ubuntu. Postanowiłem sobie
ten pakiet zbudować, no i do tego celu były mi potrzebne źródła, które można było pozyskać ze
wskazanego wyżej linku. Tam z kolei już był przygotowany katalog `debian/` , a w nim plik
`secureboot-db.service` , tj. usługa dla systemd. Zajrzawszy do środka, moim oczom ukazała się
poniższa zawartość:

	...
	[Service]
	...
	ExecStartPre=-/usr/bin/chattr -i /sys/firmware/efi/efivars/KEK-8be4df61-93ca-11d2-aa0d-00e098032b8c
	ExecStartPre=-/usr/bin/chattr -i /sys/firmware/efi/efivars/db-d719b2cb-3d3a-4596-a3bc-dad00e67656f
	ExecStartPre=-/usr/bin/chattr -i /sys/firmware/efi/efivars/dbx-d719b2cb-3d3a-4596-a3bc-dad00e67656f

	...

No proszę, a co tu robi `chattr` ? Przecie to narzędzie zmienia atrybuty pliku. No właśnie
atrybuty... A ten `i` odpowiada za bit odporności (immutable bit) i gdy jest on ustawiony, to taki
plik nie może być w żaden sposób modyfikowany, nawet przez użytkownika root.

Postanowiłem zatem sprawdzić atrybuty plików w katalogu `/sys/firmware/efi/efivars/` :

	# lsattr /sys/firmware/efi/efivars/*
	...
	----i--------------- /sys/firmware/efi/efivars/KEK-8be4df61-93ca-11d2-aa0d-00e098032b8c
	----i--------------- /sys/firmware/efi/efivars/PK-8be4df61-93ca-11d2-aa0d-00e098032b8c
	----i--------------- /sys/firmware/efi/efivars/db-d719b2cb-3d3a-4596-a3bc-dad00e67656f
	----i--------------- /sys/firmware/efi/efivars/dbx-d719b2cb-3d3a-4596-a3bc-dad00e67656f
	...

I zgodnie z moimi podejrzeniami, wszystkie te wyżej widoczne pliki (każdy odpowiada innej zmiennej
EFI/UEFI) mają ustawiony bit odporności. Usunąłem zatem ten bit odporności:

	# chattr -i /sys/firmware/efi/efivars/KEK-8be4df61-93ca-11d2-aa0d-00e098032b8c
	# chattr -i /sys/firmware/efi/efivars/PK-8be4df61-93ca-11d2-aa0d-00e098032b8c
	# chattr -i /sys/firmware/efi/efivars/db-d719b2cb-3d3a-4596-a3bc-dad00e67656f
	# chattr -i /sys/firmware/efi/efivars/dbx-d719b2cb-3d3a-4596-a3bc-dad00e67656f

I co się okazało? Teraz bez większego problemu idzie te zmienne zapisać przy pomocy narzędzia
`efi-updatevar` .

## Czyszczenie zmiennych PK, KEK, db i dbx

Drugi problem, jaki napotkałem, związany był z wyczyszczeniem zmiennych `PK` , `KEK` , `db` i
`dbx` . Do tej pory informacje w tych zmiennych były zastępowane w procesie ich aktualizacji.
Czasami jednak przydałoby się te zmienne kompletnie wyczyścić, tak by nie zawierały żadnych
informacji. Można tego dokonać przez stworzenie pustego pliku z listą sygnatur EFI/UEFI:

	# > null.esl
	# ls -al null.esl
	-rw-r--r-- 1 root root 0 2020-07-30 18:32:47 null.esl

Tak stworzony plik trzeba podpisać w standardowy sposób kluczem `PK` , w którego to posiadaniu
powinniśmy już być (jeśli nie mamy tego klucza, to odsyłam do artykułu podlinkowanego we wstępie):

	# sign-efi-sig-list -c PK.crt -k PK.key PK null.esl PKnull.auth
	Timestamp is 2020-7-30 18:35:21
	Authentication Payload size 40
	Signature of size 1211
	Signature at: 40

	# ls -al PKnull.auth
	-rw------- 1 root root 1251 2020-07-30 18:35:21 PKnull.auth

Ten plik może zostać teraz wykorzystany do wyczyszczenia zmiennych `PK` , `KEK` , `db` i `dbx` .
Robimy to w poniższy sposób:

	# efi-updatevar -f PKnull.auth PK
	# efi-updatevar -f PKnull.auth KEK
	# efi-updatevar -f PKnull.auth db
	# efi-updatevar -f PKnull.auth dbx

Sprawdźmy czy zadziałało:

	# efi-readvar

	Variable PK has no entries
	Variable KEK has no entries
	Variable db has no entries
	Variable dbx has no entries
	Variable MokList has no entries

No proszę, zmienne udało się wyczyścić bez żadnego problemu, a sam błąd `Operation not permitted`
już się nie pojawia.

Po wyczyszczeniu zmiennej `PK` system automatycznie przejdzie w tryb `Setup Mode` , o czym możemy
się przekonać wydając jedno z tych dwóch poniższych poleceń:

	# mokutil --sb-state
	SecureBoot disabled
	Platform is in Setup Mode

	# bootctl | grep Setup

	   Setup Mode: setup

Możemy teraz wgrać nową zawartość do tych czterech zmiennych dokładnie w taki sam sposób jak to
było robione z poziomu firmware EFI/UEFI, z tą różnicą, że przy pomocy `efi-updatevar` :

	# efi-updatevar -f old_dbx.auth dbx
	# efi-updatevar -f compound_db.auth db
	# efi-updatevar -f compound_KEK.auth KEK
	# efi-updatevar -f PK.auth PK

I sprawdźmy czy udało się je uzupełnić:

	# efi-readvar
	Variable PK, length 853
	PK: List 0, type X509
		Signature 0, size 825, owner 90a7d962-e601-4363-a326-524f9f76132b
			Subject:
				CN=morfikov's platform key
			Issuer:
				CN=morfikov's platform key
	Variable KEK, length 861
	KEK: List 0, type X509
		Signature 0, size 833, owner 95b65092-8342-4a45-9dc2-d80233aba2b3
			Subject:
				CN=morfikov's key-exchange-key
			Issuer:
				CN=morfikov's key-exchange-key
	Variable db, length 865
	db: List 0, type X509
		Signature 0, size 837, owner 1091ff9c-7b84-4df6-8163-ed1b8aa05096
			Subject:
				CN=morfikov's kernel-signing key
			Issuer:
				CN=morfikov's kernel-signing key
	Variable dbx, length 3800
	dbx: List 0, type SHA256
		Signature 0, size 48, owner 7facc7b6-127f-4e9c-9c5d-080f98994345
			Hash:14e62a4905e19189e70828983165939afc0a331d0b415f3332b0e818a827f436
		...
	Variable MokList has no entries

Jak tylko zapiszemy zmienną `PK` , to system z `Setup Mode` automatycznie przeskoczy do
`User Mode` , co możemy zaobserwować w wyjściu  `bootctl` :

	# bootctl | grep Setup

	   Setup Mode: user

Warto w tym miejscu zaznaczyć, że z racji podpisania pliku `null.esl` kluczem `PK` , zmiany, które
ten plik wprowadza (czyści zmienne) są zawsze autoryzowane. Dlatego dobrze jest ten plik po
zakończeniu prac usunąć albo chociaż trzymać go w bezpiecznym miejscu razem z samym kluczem `PK` ,
tak by nikt nam zmiennych EFI/UEFI nie przepisał bez naszej świadomej i dobrowolnej zgody.


[1]: https://packages.ubuntu.com/groovy/secureboot-db
[2]: /jak-dodac-wlasne-klucze-dla-secure-boot-do-firmware-efi-uefi-pod-linux/
[3]: https://eclypsium.com/2020/07/29/theres-a-hole-in-the-boot/
