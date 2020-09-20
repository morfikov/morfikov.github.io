---
author: Morfik
categories:
- Linux
date: "2015-06-15T22:28:17Z"
date_gmt: 2015-06-15 20:28:17 +0200
published: true
status: publish
tags:
- system-plików
- smart
- hdd
- ssd
- ext4
title: Bad sektor w dzienniku systemu plików ext4
---

Parę dni temu opisywałem jak udało mi się [realokować uszkodzony sektor][1] z dysku, który już
przepracował dość długi okres czasu. Nie było to znowu jakoś specjalnie trudne, z tym, że cały
problem dotyczył jakiegoś losowego sektora gdzieś w środku partycji. Jako, że domyślnym systemem
plików na linuxie są te z rodziny `ext` (ext2, ext3, ext4) , oraz, że [trzecia wersja][2] tego
systemu plików została wyposażona w dziennik (journal), to trzeba by się zastanowić, co w przypadku
gdy taki uszkodzony sektor trafi się właśnie w dzienniku tego systemu plików?

<!--more-->
## Gdzie się kończy dziennik systemu plików ext4

W manualu narzędzia `mkfs.ext4` możemy odnaleźć następujące zdanie:

> The size of the journal must be at least 1024 filesystem blocks (i.e., 1MB if using 1k blocks, 4MB
> if using 4k blocks, etc.) and may be no more than 10,240,000 filesystem blocks or half the total
> file system size (whichever is smaller)
>
> [man mkfs.ext4][3]

Zatem dziennik systemu plików ext4 nie może mieć mniej niż 4MiB (domyślnie blok ma 4K) i nie może
być większy niż połowa systemu plików na partycji. Ile zatem taki dziennik zajmuje w naszym
przypadku? Domyślnie jest to `128MiB` , natomiast jeśli nie jesteśmy pewni, to zawsze możemy
sprawdzić przy pomocy `dumpe2fs` :

    # dumpe2fs /dev/sda2 | grep ^Journal
    dumpe2fs 1.42.13 (17-May-2015)
    Journal inode:            8
    Journal backup:           inode blocks
    Journal features:         journal_incompat_revoke
    Journal size:             32M
    Journal length:           8192
    Journal sequence:         0x0000035f
    Journal start:            0
    2

Zatem w tym przypadku, dziennik zaczyna się na bloku `0` tej partycji i rozciąga się na kolejne
`8192` bloki zajmując łącznie `32MiB`. Zatem jeśli będzie problem z odczytem jakiegoś sektora
poniżej 8192 lub 8192\*(4096/512)=65536, w zależności od tego czy dysk posiada sektory 512 czy 4096
bajtowe, wtedy obrywa dziennik systemu plików ext4. Jako, że to na nim się opiera bezpieczeństwo
plików, to prawdopodobnie system zacznie sypać błędami próbując odczytać ten dziennik, a to może
uniemożliwić nawet start systemu.

## Usuwanie i tworzenie dziennika

W przypadku gdyby jednak oberwało się dziennikowi systemu plików ext4, nie ma innej rady jak usunąć
go i stworzyć jeszcze raz. Usuwanie dziennika sprowadza się do wydania poniższego polecenia, np. z
jakiegoś systemu live:

    # tune2fs -O ^has_journal /dev/sda2

Po tym kroku postępujemy dokładnie tak jak w przypadku zwykłego bad sektora. Gdy już realokujemy
sektor, trzeba stworzyć dziennik na nowo:

    # tune2fs -j /dev/sda2

I to w zasadzie wszystko.


[1]: {{< baseurl >}}/post/uszkodzony-sektor-na-dysku-i-jego-realokacja/
[2]: https://en.wikipedia.org/wiki/Ext3
[3]: http://manpages.ubuntu.com/manpages/xenial/en/man8/mkfs.ext4.8.html
