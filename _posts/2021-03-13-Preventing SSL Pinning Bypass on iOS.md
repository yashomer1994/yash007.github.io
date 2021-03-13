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

[](#header-2)**Why SSL Pinning ?**
---

When the application tries to communicate with the server or other applications , it doesen't validate which certificate is secured and trusted and which isn't. Let's take a Scenario to better understand the use of SSL Pinning.

---
**Scenario**

---
---

Attacker is able to generate a Self-Signed Certificates and install it to the Operating system trust's store to setup MITM attack to bypass SSL.

Impact :

    - Tamper the SSL Sessions and reverse engineer the app protocol or extracting the API Requests Acceess.

    - The root CAs which are trusted by the devices can also get compromised and can be used for generating certificates.

--- 

--- 
[](#header-3)**Prevent SSL Pinning Bypass**
---

There several Mechanism to prevent the SSL Pinning Bypass by means of **Chain Of Trust** : 

Certificate Pinning Knowing in advance the certificate of the server our application is communication with, we can hard-code  into the application itself and refuse communicating unless it is a perfect match.

---

--- 
[](#header-4)**How To Pin?**
---

This technqiue is to harden the security of Applications by adding extra layer of identity check performed by Application Itself.

Methods : 
     
- **Certificate**: There's drawback of using feature is Certificate have expire after certain period of time. When Implmenting Certificate Pinning in application , need to take care of Certificate while updating the Application.

- **Public Key**: Certificates Updated Regularly but the public key remains the same or you have the ability to keep it same. Therefore pinning the key makes the design more flexible, but a bit trickier to implement, as now we have to extract the key from the certificate, both at pinning time and at every connection.

        - Create a sha256 hashes and store them. It makes it easier to manage due to size and it allows shipping an application with a hash of a future certificate or key without exposing them ahead of time.








