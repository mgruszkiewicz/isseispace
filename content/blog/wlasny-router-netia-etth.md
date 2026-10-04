---
title: "Podłączenie własnego routera Netia ETTH"
date: 2025-03-30T13:38:41+02:00
draft: false
cover: "images/2025-03-30-netia/cover.webp"
tags: ['pl', 'networks']
---

Ostatnio zmieniłem operatora na Netie, ze względu na oferowanie internetu po ETTH, co pozwala wyeliminować totalnie urządzenie od dostawcy (a nie jak w przypadku np. UPC/Play po DOCSIS gdzie trzeba przestawić ich wspaniałego connectboxa w tryb bridge). Według instrukcji na internecie, również powinienem bez problemu otrzymać dane PPPoE.

W moim przypadku (prawdopodobnie że jest to sieć `internetia`?), w portalu netiaonline nie miałem podanych danych logowania PPPoE (tak jak to powinno być według instrukcji). Instalator również nie miał przy sobie na umowie tych danych, oraz po sprawdzeniu przez siebie w portalu, również nie mógł uzyskać tych danych. Po kontakcie na infolinii, jedyne co dostałem SMSem, to nieprzydatne dla mnie informacje co do ustawień ADSL, nadal bez danych do logowania.

Zastanawiało mnie że instalator, jedyne co zrobił po przyjściu, to podłączył router i już internet śmigał, co podpowiadało mi, że występuje jedynie filtrowanie po adresie MAC

Metodą prób i błędów, doszedłem do tego, że aby podpiąć swój router do sieci ETTH w takim wypadku należy
1. sklonować adres MAC WANu z routera który dostaliśmy od operatora (w przypadku Huawei DN8245X6-10, znajduje się od na naklejce od spodu urządzenia)
2. ~~ustawić dane logowania PPPoE na WAN na:  
login: internet  
hasło: internet~~  
Wychodzi na to, że nie zawsze PPPoE jest wymagane - u mnie na ETTH-IN wystarczy tylko sklonować adres MAC
```
root@OpenWrt:~# cat /etc/config/network | grep -A5 wan
config interface 'wan'
	option device 'wan'
	option proto 'dhcp'
	option macaddr '14:49:20:XX:XX:XX'
	option peerdns '0'
[...]
root@OpenWrt:~# ifconfig | grep -A5 wan
wan       Link encap:Ethernet  HWaddr 14:49:20:A7:D2:83
          inet addr:100.124.xxx.xxx  Bcast:100.124.xxx.255  Mask:255.255.255.0
          inet6 addr: fe80::1649:20ff:fea7:d283/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:98383500 errors:0 dropped:356456 overruns:0 frame:0
          TX packets:47658079 errors:0 dropped:0 overruns:0 carrier:0
```

Po tej operacji, mój mikrotik (i później router na openwrt) dostał adres z DHCP - niestety przez ostatnie zmiany w sieci Netii, adres IP zza CGNAT zamist publiczny :(
