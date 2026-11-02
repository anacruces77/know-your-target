Proyecto 1 · Know Your Target · Parte 1 (reconocimiento pasivo)
 
---
 
2026-10-28 10:40
Objetivo: programa Snapchat en HackerOne (lectura de la política, sin tocar el objetivo)
https://hackerone.com/snapchat/policy_scopes -> 46 activos en el alcance; última actualización del alcance 18/12/2024

2026-10-28 11:00 - entorno: repositorio clonado en Kali


2026-10-28 11:50
Objetivo: programa Snapchat en HackerOne (lectura de la política, sin tocar el objetivo)
Descargado del navegador: Scope → Download CSV  y Program guidelines → PDF, para tener el CSV con los activos del alcance y PDF de las reglas guardados en el repositorio


2026-11-02 17:17 
Objetivo: programa Snapchat en HackerOne (lectura de la política, sin tocar el objetivo)
$ alcance.md y scope.txt redactados entre el 28/10 y 02/11 a partir del CSV (46 activos), las guidelines y las Core Ineligible Findings de HackerOne, subidos hoy al repositorio (antes estaban vacíos)
-> sin comodín *.snapchat.com; comodines en alcance *.sc-core.net y *.sc-corp.net, 16 subdominios de snapchat.com listados uno a uno, fuera de alcance: support.snapchat.com, returns.spectacles.com, dev.playcanv.as, dev*.playcanvas.com


2026-11-02 17:27
Objetivo: snapchat.com (dominio raíz, consulta al registrador, no al objetivo)
$ curl -s https://rdap.markmonitor.com/rdap/domain/SNAPCHAT.COM | jq . | tee rdap_snapchat_markmonitor.json
-> titular y contacto técnico: DNStination Inc. ("Domain Administrator", San Francisco, US), titular oculto tras proxy, no aparece Snap Inc., registrador MarkMonitor Inc., NS de AWS sin cambios desde 2016-03-15. Sin DNSSEC. No se obtienen personas ni correos de Snapchat
