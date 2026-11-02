Proyecto 1 · Know Your Target · Parte 1 (reconocimiento pasivo)

---

2026-10-28 10:40 · objetivo: programa Snapchat en HackerOne (lectura de la política, sin tocar el objetivo)
$ (navegador) https://hackerone.com/snapchat/policy_scopes
-> 46 activos en el alcance; última actualización del alcance 18/12/2024
siguiente: descargar el CSV del alcance y las Program guidelines

2026-10-28 11:00 · entorno: Kali del laboratorio
$ git clone https://github.com/anacruces77/know-your-target.git
-> repositorio clonado en ~/know-your-target
siguiente: descargar el alcance oficial

2026-10-28 11:50 · objetivo: programa Snapchat en HackerOne (lectura de la política, sin tocar el objetivo)
$ (navegador) Scope → Download CSV · Program guidelines → PDF
-> CSV con los activos del alcance y PDF de las reglas guardados en el repositorio
siguiente: redactar alcance.md y scope.txt a partir del CSV y las guidelines

2026-11-02 17:17 · objetivo: programa Snapchat en HackerOne (lectura de la política, sin tocar el objetivo)
$ alcance.md y scope.txt redactados entre el 28/10 y 02/11 a partir del CSV (46 activos), las guidelines y las Core Ineligible Findings de HackerOne, subidos hoy al repositorio (antes estaban vacíos)
-> sin comodín *.snapchat.com, comodines en alcance *.sc-core.net y *.sc-corp.net, 16 subdominios de snapchat.com listados uno a uno, fuera de alcance: support.snapchat.com, returns.spectacles.com, dev.playcanv.as, dev*.playcanvas.com
siguiente: técnica 1 · whois snapchat.com

2026-11-02 17:24 · objetivo: snapchat.com (dominio raíz, consulta al registro, no al objetivo)
$ whois snapchat.com | tee whois_snapchat.txt
-> connect: Network is unreachable (el puerto 43 no tiene salida desde el laboratorio). Fichero vacío borrado
siguiente: misma consulta por RDAP (HTTPS), sustituto oficial de whois

2026-11-02 17:25 · objetivo: snapchat.com (dominio raíz; consulta al registro, no al objetivo)
$ curl -s https://rdap.verisign.com/com/v1/domain/snapchat.com | jq . | tee rdap_snapchat.json
-> registrador MarkMonitor Inc. (IANA 292); alta 2012-02-28, caduca 2034-02-28, último cambio 2026-01-28; 6 bloqueos client/server (delete, transfer, update); NS en AWS Route 53; sin DNSSEC
siguiente: consultar el RDAP del registrador (MarkMonitor) para los contactos del titular

2026-11-02 17:27 · objetivo: snapchat.com (dominio raíz, consulta al registrador, no al objetivo)
$ curl -s https://rdap.markmonitor.com/rdap/domain/SNAPCHAT.COM | jq . | tee rdap_snapchat_markmonitor.json
-> titular y contacto técnico: DNStination Inc. ("Domain Administrator", San Francisco, US), titular oculto tras proxy, no aparece Snap Inc., registrador MarkMonitor Inc., NS de AWS sin cambios desde 2016-03-15. Sin DNSSEC. No se obtienen personas ni correos de Snapchat
siguiente: técnica 2 · registros DNS (NS, MX, TXT) con dig

2026-11-02 17:35 · objetivo: snapchat.com (in scope, consulta DNS pública, no toca servidores del objetivo)
$ dig snapchat.com NS +short | tee dns_ns.txt
-> 4 NS de AWS Route 53 (ns-220.awsdns-27.com, ns-530.awsdns-02.net, ns-1468.awsdns-55.org, ns-1892.awsdns-44.co.uk), coincide con RDAP
siguiente: registros MX

2026-11-02 17:35 · objetivo: snapchat.com (in scope)
$ dig snapchat.com MX +short | tee dns_mx.txt
-> 5 MX de Google (aspmx.l.google.com prioridad 10 + alternativos): correo en Google Workspace
siguiente: registros TXT (SPF y verificaciones)

2026-11-02 17:36 · objetivo: snapchat.com (in scope)
$ dig snapchat.com TXT +short | tee dns_txt.txt
-> SPF estricto (-all) con Google, Zendesk y Pardot, ~40 TXT de verificación de terceros: Microsoft 365, Atlassian, Docker, Sonatype OSSRH, Tailscale, Jamf, Adobe IdP, LogMeIn, HubSpot, Segment, OneTrust, Shopify, Dropbox, Miro, Smartsheet, Linear, Canva, OpenAI, Anthropic, Apple, Autodesk, Censys, Parallels, entre otros
siguiente: política DMARC

2026-11-02 17:36 · objetivo: _dmarc.snapchat.com (in scope)
$ dig _dmarc.snapchat.com TXT +short | tee dns_dmarc.txt
-> DMARC p=reject, pct=100, informes a dmarc-report@snapchat.com (buzón de servicio, no persona)
siguiente: técnica 3 · subdominios con subfinder y crt.sh


2026-11-02 17:54 · objetivo: sc-core.net y sc-corp.net (comodines en alcance; consulta al registro, no al objetivo)
$ curl -s https://rdap.verisign.com/net/v1/domain/sc-core.net | jq . | tee rdap_sc-core.json
$ curl -s https://rdap.verisign.com/net/v1/domain/sc-corp.net | jq . | tee rdap_sc-corp.json
-> ambos con MarkMonitor Inc. y NS en AWS Route 53 (zonas distintas a la de snapchat.com y entre sí); sc-corp.net alta 2014-07-03, caduca 2028-07-03; sc-core.net alta 2015-04-30, caduca 2028-04-30; solo 3 bloqueos client (snapchat.com tiene también los server); sin DNSSEC
siguiente: técnica 3 · subdominios con subfinder y crt.sh
