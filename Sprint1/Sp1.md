---
layout: default
title: "SISTEMES D'INICI"
---
# SISTEMES D'INICI

## Índex

**1- SystemV vs Upstart vs Systemd**
* 1.1- Runlevels o Targets?
* 1.2- Quin el nostre SO?

**2- SystemV**
* 2.1- Directoris
* 2.2- Procés arrencada

**3- Systemd**
* 3.1- Directoris
* 3.2- systemctl
* 3.3- dependències
* 3.4- Modificar target provisional
* 3.5- Modificar target definitiu
* 3.6- Afegir/treure serveis target
* 3.7- Creem nou target
* 3.8- Creem nou servei
---


<img width="1024" height="572" alt="image" src="https://github.com/user-attachments/assets/d6d2a5fa-8d2d-4d93-aa4a-6054b6f91b4d" />

---

## Conceptes

* **Kernel** -> gestiona processos
* **Aplicació** -> programa interactua usuari i executa 1r pla
* **Servei** -> programa associat SO i 2n pla
* **Procés** -> f(x) intern del SO
  * *Nota:* Aplicacions i serveis -> generen processos (sincronitzar i planificar)

---

## Nivells d'execució

* **0** - power off
* **1** - rescue -> 1 usuari (dimonis mínims)
* **2-5** -> multiusuari, xarxa, sense...
* **6** -> reboot

---

## Comandes d'aturada

* `/etc/init.d/cron stop`
* `service cron stop`
* `systemctl stop cron`
