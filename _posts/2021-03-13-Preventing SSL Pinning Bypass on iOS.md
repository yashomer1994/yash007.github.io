---
layout: post
title:  "Preventing SSL Pinning Bypass on iOS"
date:   2021-03-13
categories: swift iOS Apple
title: Preventing SSL Pinning Bypass on iOS
---

In this recent years, Attacker first step to reverse engineer the Mobile application is to bypass SSL/TLS Pinning, so to get start to prevent the app from bypassed i will start explaining from basics to advanced in several series of blogs.

--- 

[](#header-1)**Introduction to SSL Pinning**
---

SSL Pinning is a Mechanism used in Mobile Applications to Secure the Transmission between the Client and Server from **MITM** Attacks. SSL technique  is used by Applications to accept the  default to trusted certificates by Operating System which is stored in certificate store.

[](#header-1)**Why SSL Pinning ?**
---

When the application tries to communicate with the server or other applications , it doesen't validate which certificate is secured and trusted and which isn't. Let's take a Scenario to better understand the use of SSL Pinning.

---
**Scenario**
---

Attacker is able to generate a Self-Signed Certificates and install it to the Operating system trust's store to setup MITM attack to bypass SSL.

Impact :

    - Tamper the SSL Sessions and reverse engineer the app protocol or extracting the API Requests Acceess.

    - The root CAs which are trusted by the devices can also get compromised and can be used for generating certificates.

--- 



