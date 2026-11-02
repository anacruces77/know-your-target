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

2026-11-02 18:03 · objetivo: alcance del programa (preparación, sin tocar el objetivo)
$ listas generadas a partir del CSV en tecnica-03-subdominios/procesado/ (dominios_scope_wildcard, dominios_mapeo, hosts_scope_exactos, excluidos, dominios_raiz)
-> 2 comodines en alcance, 6 dominios solo para mapeo, 30 hosts concretos en alcance, 3 exclusiones
siguiente: subfinder sobre los 8 dominios raíz

2026-11-02 18:05 · objetivo: entorno (comprobación de la herramienta)
$ subfinder -version ; subfinder -ls ; cat ~/.config/subfinder/provider-config.yaml
-> subfinder v2.14.0 (aviso de versión desactualizada), 50 fuentes, ninguna clave API configurada: solo ~13 fuentes gratuitas disponibles
siguiente: primera ejecución sin claves

2026-11-02 18:12 · objetivo: 8 dominios raíz (fuentes pasivas, no toca el objetivo)
$ subfinder -dL procesado/dominios_raiz.txt -all -silent -o raw/subfinder_sinclaves_2026-11-02.txt
-> 774 resultados. Duplicados por mayúsculas y registros DNS de servicio (_dmarc, _tcp) mezclados con hosts; ruido bajo snappublisher.snapchat.com
siguiente: repetir con atribución de fuentes

2026-11-02 18:18 · objetivo: 8 dominios raíz (fuentes pasivas)
$ subfinder -dL procesado/dominios_raiz.txt -all -cs -oJ -silent -o raw/subfinder_fuentes_2026-11-02.json
-> 808 resultados (34 más que a las 18:12: las fuentes pasivas varían entre ejecuciones). Solo 6 fuentes con datos: thc 442, rapiddns 273, crtsh 271, hackertarget 174, waybackarchive 33, hudsonrock 18. Por dominio: snapchat.com 359, sc-corp.net 238, sc-core.net 74, bitstrips.com 51, playcanvas.com 36, spectacles.com 29, bitmoji.com 17, playcanv.as 4
siguiente: consulta directa a crt.sh

2026-11-02 18:23 · objetivo: 8 dominios raíz (logs públicos de Certificate Transparency, no toca el objetivo)
$ curl -s "https://crt.sh/?q=%25.<dominio>&output=json" (bucle, 5 s entre peticiones)
-> 8 JSON válidos. Certificados: snapchat.com 2989, spectacles.com 2189, playcanvas.com 453, bitmoji.com 104, bitstrips.com 91, playcanv.as 22, sc-corp.net 20, sc-core.net 2
siguiente: extraer nombres únicos

2026-11-02 18:28 · objetivo: resultados de crt.sh (procesado local)
$ jq -r '.[].name_value' raw/crtsh_*.json | tr 'A-Z' 'a-z' | sed 's/^\*\.//' | sort -u
-> 292 nombres: 290 de los dominios objetivo + 2 de team--snapchat.com (dominio ajeno, propiedad no verificada, no se analiza). sc-core.net y sc-corp.net solo devuelven el dominio raíz (certificados comodín): CT no revela hosts internos
siguiente: fusionar con subfinder y comparar

2026-11-02 18:29 · objetivo: resultados de subfinder y crt.sh (procesado local)
$ fusión, normalización a minúsculas y comparación con comm
-> 803 subdominios únicos. Solo subfinder 523, solo crt.sh 9 (8 dominios raíz + 1 subdominio real: msg-app-x9dwa4.playcanvas.com), ambas 281, 10 registros _ descartados. El DNS pasivo aporta mucho más que CT en este objetivo
siguiente: separar superficie en alcance y fuera de alcance

2026-11-02 18:31 · objetivo: 803 subdominios (filtrado contra el alcance)
$ comodines + hosts exactos, menos exclusiones
-> 338 en alcance (308 bajo comodines + 30 de 30 hosts concretos), 465 fuera de alcance. Aparecen las 3 exclusiones explícitas, descartadas. Error corregido: el filtro dev* eliminaba developer.playcanvas.com, que está en alcance (la entrada explícita prevalece sobre dev*.playcanvas.com)
siguiente: análisis de nomenclatura

2026-11-02 18:32 · objetivo: 338 subdominios en alcance (análisis de nombres, sin tocar el objetivo)
$ recuento de palabras clave, zonas y sufijos de entorno
-> hipótesis por nomenclatura, no verificadas: sc-core.net = red de oficinas (34 *.qsys, salas Q-SYS de audio/vídeo; sedes lon03, ams, oxf02, pdx01, ein100; Windows en EC2 ec2amaz-*; fortimanager, wifienroll). sc-corp.net = herramientas internas (jira, wiki, gerrit, github, spinnaker, ssh-ca, secproxy, identity-broker) en AWS, GCP, Oracle Cloud y App Engine. Muchos más nombres de entornos no productivos (dev, stage, staging, beta) que de prod (22)
siguiente: técnica 4 · huella tecnológica con Netcraft
