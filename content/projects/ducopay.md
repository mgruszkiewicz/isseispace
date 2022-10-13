---
title: "Ducopay"
date: 2022-08-14T00:24:56+02:00
draft: false
cover: "/img/projects/ducopay/firefox_OEhjHgplgP.png"
tags: ['2021', 'php', 'symfony']
---
## What it is?
Ducopay is a simple payment gateway for a internet currency called [Duinocoin](https://duinocoin.com). It features "p2p" payment method. In the first version, the coins were send at first to our account, tax was subtracted, and the rest was send to seller, but later it was removed, and the payment goes directly to seller.

It features very simple API and working mechanism
1. You create new transaction (as a result you get a transaction ID)
2. Then shop redirect you to Ducopay website for payment
3. On Ducopay page after successful payment Duinocoin transaction ID is send to backend for verification
4. If Duinocoin transaction is valid, the Ducopay server sends a callback to shop
5. Shop mark order as 'payment successful'

## Integrations
I created some demo applications in PHP and Python (with Flask), plus Wordpress Woocommerce plugin.

Go to Ducopay Docs to learn more: [https://docs.ducopay.com/example/](https://docs.ducopay.com/example/)  
Link to Ducopay: [https://ducopay.com](https://ducopay.com)

Ducopay was written using PHP framework Symfony