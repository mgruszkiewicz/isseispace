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
2. ustawić dane logowania PPPoE na WAN na:  
login: internet  
hasło: internet  

Po tej operacji, mój mikrotik dostał adres z DHCP - niestety przez ostatnie zmiany w sieci Netii, adres IP zza CGNAT zamist publiczny :(