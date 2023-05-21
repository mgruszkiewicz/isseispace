---
title: "'Odceglanie' NAS'a Buffalo LS210/LS220"
date: 2019-11-06T00:24:56+02:00
draft: false
cover: "images/2019-11-06-buffalo/cover.webp"
tags: ['pl', 'embedded', 'nas']
---


Ten poradnik powinien działać dla Buffalo LS210/LS220 i rozwiązać problem z siedmiokrotnym miganiem czerwonej diody podczas rozruchu.

Na początku informuję że nie odpowiadam za ewentualne uszkodzenia oraz utratę danych! **Przed rozpoczęciem, jeżeli nie mamy kopii zapasowej, należy taką utworzyć** - więcej informacji jak odczytać dyski z NAS'a jest na dole

W przypadku w którym NAS nie jest w ogóle widoczny w sieci oraz w NAS Navigatorze należy wgrać podstawowy system od początku używając TFTP. Domyślnie NAS nada sobie adres IP 192.168.11.150/24\. Adres serwera TFTP (swojego komputera) należy ustawić na 192.168.11.1/24 i upewnić się aby wyłączyć zaporę systemu a następnie podłączyć NAS bezpośrednio do komputera kablem sieciowym.

W oknie serwera TFTP powinien pojawić się komunikat `listening On: 192.168.11.1:69`

Jeżeli się nie wyświetla, lub wyświetla się inny adres IP należy sprawdzić konfigurację adresów IP (lub spróbować odłączyć i podłączyć przewód sieciowy, może też być konieczne aby przed zmianą IP być podłączonym do obecnej sieci jeżeli adres się nie zmienia na ten który ustawiliśmy)

Następnie należy przytrzymać czerwony przycisk "function" z tyłu NAS i podłączyć zasilanie. Należy trzymać przycisk function do czasu kiedy dioda z przodu NAS zacznie migać na biało, następnie puścić przycisk i nadusić go jeszcze raz. W konsoli powinny pokazywać się komunikaty

`Client 192.168.11.150 ... Block Served`  
`Client 192.168.11.150 ... Block Served`

Teraz NAS powinien być w EM Mode (Emergency mode) i pokazywać się w NAS Navigatorze.

![](https://i.issei.space/ai389.jpg)

Następnie należy uruchomić program LSUpdater.exe, poczekać aż wyszuka naszego NAS'a i kliknąć przycisk Update.

![](https://i.issei.space/yp3di.png)

![](https://i.issei.space/8x00q.png) _[źródło](http://commonmanrants.blogspot.com/2014/01/buffalo-linkstation-partition-not-found.html)_

Proces ten będzie trwał kilka minut, po pomyślnym ukończeniu należy podłączyć NAS do sieci aby dostał adres przez DHCP. Po chwili serwer powinien być widoczny w NAS Navigatorze z jego adresem IP.

Linki do pobrania wszystkich potrzebnych plików: [Download](https://files.s2.issei.space/blog/Buffalo_ls2xx_recovery.zip) | [Mirror [mega.nz]](https://mega.nz/#!SRczCaxC!zZGums_mUh5XQUXvyBnLmz7GAaAZKwwJhxtsj87n0Mw) | [Mirror [GDrive]](https://drive.google.com/file/d/1jv4Rsm0fKDjrGTUyFFXdOGSFcFRLr43F/view?usp=sharing)

Suma kontrolna: `SHA1: BA3E87B4C61AAA85D8A58321E35D7B68BEBF97DE`

Dodatkowe informacje:

*   Dane na dyskach są w formacie EXT4 oraz RAID jest tworzony przez MD, więc bez problemu można RAID lub pojedynczy dysk odczytać przez linuxa lub z zastosowaniem przykładowo [Ext2Fsd](http://www.ext2fsd.com/) pod Windowsem.

