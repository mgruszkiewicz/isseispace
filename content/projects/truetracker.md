---
title: "TrueTracker"
date: 2022-08-14T00:24:56+02:00
draft: false
cover: "images/projects/truetracker/mainpage.png"
tags: ['2021', 'php', 'symfony']
---

## What it is?
TrueTracker is my first PHP project created using Symfony framework. It is a simple RMA portal.
 
1. Repair shop register repair request (input serial numbers, customer details)
2. Repair shop give customer repair tracking code
3. Customer can check status of repair on portal
4. Repair shop can add notes, update repair status
5. System can send Email notifications about change in status.

Frontend is using Bootstrap 5 + jQuery
## Features

* Display repair status for customer
* Generate QR Code for quick lookup
* Ability to attach pictures and description to status
* Search bar for shop - can filter by customer
* Different permissions level (employee, admin)
* Generate repair raport to PDF
* Status messages can be customized
* Send email notification about status change
* Email notification HTML can be changed from admin page
* Automatic updates

## Screenshots
![user list](images/projects/truetracker/firefox_8TiVib6zRY.png)
List of users  

![example email notification](images/projects/truetracker/example_email.png)
Example email notification  

![example report](images/projects/truetracker/example_raport.png)
Example of exported status report  

![searchbar](images/projects/truetracker/searchbar.png)
Search bar can search by serial number, customer email or phone number, repair order descriptions  

![repair orders](images/projects/truetracker/repair_order.png)
List of repair orders  

![predefined states](images/projects/truetracker/firefox_oWZE66Z0RX.png)
Predefined states  

![adding new status to repair](images/projects/truetracker/add_new_status.png)
Adding new status to repair order. Status can be visible for customer or it can be set to interal (only employees can see them)

![repair order view for shop](images/projects/truetracker/shop_repair_view.png)
Repair order view for shop  

![repair view for customer](images/projects/truetracker/repair_view_customer.png)
Repair order view for customer  
