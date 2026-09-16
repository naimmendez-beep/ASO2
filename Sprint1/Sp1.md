---
layout: default
title: "SISTEMES D'INICI"
---
# SISTEMES D'INICI

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
