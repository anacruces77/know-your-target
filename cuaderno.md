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

## Técnica 4 · Huella tecnológica (Netcraft)

2026-11-02 18:48 · huella tecnológica de accounts.snapchat.com con Netcraft Site Report (consulta a la base de datos de Netcraft, sin tráfico al objetivo)
$ (navegador) https://sitereport.netcraft.com/?url=https://accounts.snapchat.com
-> Red: IPv4 34.149.46.130 (sin IPv6), AS396982, rDNS 130.46.149.34.bc.googleusercontent.com; netblock Google LLC (GOOGL-2, 34.128.0.0-34.191.255.255, ARIN), hosting Google, EE. UU.
-> DNS: snapchat.com en AWS Route 53 (ns-220.awsdns-27.com), registrador MarkMonitor, DNSSEC activado. DNS en AWS y servidor en Google Cloud (multinube)
-> TLS: certificado DigiCert Global G2 TLS RSA SHA256 2020 CA1, validación OV a Snap Inc. (California); CN *.snap.com, SAN *.snapchat.com y unos 30 dominios más de Snap (lista plegada en Netcraft; fuera de alcance salvo los listados); válido 2026-09-25 a 2027-04-11, 3 SCT
-> TLS: TLSv1.3, TLS_AES_256_GCM_SHA384, RSA 2048, PFS sí, ALPN h2; sin SSLv3; Heartbleed no (ya no ofrece Heartbeat); OCSP stapling sin respuesta
-> Tecnología (Netcraft, rastreo de hace 24 días): Server "API Gateway"; Google Compute Engine + Google HTTPS Load Balancer + Envoy; cabeceras HSTS (incluye subdominios), CSP, X-Content-Type-Options, X-XSS-Protection; sin trackers; sin SPF/DMARC propios (correo cubierto en técnica 2)
-> Captura: tecnica-04-huella/capturas/netcraft-accounts.snapchat.com.jpeg
siguiente: Netcraft del siguiente host en alcance

2026-11-02 18:59 · huella tecnológica de app.snapchat.com con Netcraft Site Report (consulta a la base de datos de Netcraft, sin tráfico al objetivo)
$ (navegador) https://sitereport.netcraft.com/?url=https://app.snapchat.com
-> Red: IPv4 34.149.130.113 (sin IPv6), AS396982, rDNS 113.130.149.34.bc.googleusercontent.com; mismo netblock que accounts (Google LLC, GOOGL-2, 34.128.0.0-34.191.255.255); DNS igual (Route 53, MarkMonitor, DNSSEC)
-> TLS: certificado DV de Google Trust Services (WR3), sin organización; CN app.snapchat.com, SAN app.snapchat.com y gcs.sc-cdn.net (sc-cdn.net fuera de alcance); válido 2026-10-15 a 2027-01-13 (~90 días, renovación automática), 2 SCT
-> TLS: TLSv1.3, TLS_AES_256_GCM_SHA384, RSA 2048, PFS sí, ALPN h2; sin SSLv3; Heartbleed no; OCSP stapling sin respuesta
-> Tecnología (Netcraft, rastreo de hace 9 días): Server "Google Frontend"; Google Compute Engine; Netcraft no detecta Envoy ni cabeceras de seguridad; sin trackers
-> Comparación con accounts: misma red y DNS, pero distinto frontal (Google Frontend frente a API Gateway + Envoy) y distinto certificado (DV gestionado por Google frente a OV DigiCert *.snap.com). Compatible con App Engine, como indica el CSV, sin confirmación explícita de Netcraft
-> Captura: tecnica-04-huella/capturas/netcraft-app.snapchat.com.jpeg
siguiente: Netcraft de un host de *.sc-core.net o *.sc-corp.net

2026-11-02 19:08 · huella tecnológica de developer-portal.sc-corp.net con Netcraft Site Report. Host elegido de tecnica-03-subdominios/procesado/subdominios_en_scope.txt (308 hosts bajo *.sc-core.net y *.sc-corp.net), filtrando por nombres de servicio público; host de producción con nombre de web pública
$ (navegador) https://sitereport.netcraft.com/?url=https://developer-portal.sc-corp.net
-> Background: título "Sign in - Google Accounts". El rastreador de Netcraft llegó a la pantalla de inicio de sesión de Google, no al portal: acceso protegido por autenticación de Google (compatible con Identity-Aware Proxy o App Engine con login obligatorio; Netcraft no lo confirma). Visto por primera vez en octubre de 2018
-> Red: IPv4 74.125.193.121, AS15169 (red principal de Google, no Google Cloud AS396982 como accounts y app), rDNS di-in-f121.1e100.net (frontal de Google); IPv6 2a00:1450:400b:c01::79 (Google Ireland, "EU metro frontend", RIPE). Primer host de los analizados con IPv6
-> DNS: sc-corp.net en AWS Route 53 (ns-1038.awsdns-01.org), registrador MarkMonitor (organización DNStination Inc.), DNSSEC activado. Mismo patrón que snapchat.com
-> TLS: certificado DV de Google Trust Services (WR3), solo developer-portal.sc-corp.net; válido 2026-10-27 a 2027-01-25 (~90 días), 2 SCT; TLSv1.3, TLS_AES_256_GCM_SHA384, RSA 2048, PFS sí, h2; sin SSLv3; nunca vulnerable a Heartbleed; sin cabecera Server
-> Tecnología (Netcraft, rastreo de hoy): JavaScript, Google Hosted Libraries, gzip, HTML5, CSP, HSTS, X-Frame-Options Deny, X-Content-Type-Options. Probablemente corresponden a la página de login de Google y no al portal (aparecen a la vez X-XSS-Protection Block y Disabled, señal de varias respuestas en la redirección)
-> Captura: tecnica-04-huella/capturas/netcraft-developer-portal.sc-corp.net.jpeg
siguiente: Netcraft de playcanvas.com

2026-11-02 19:11 · huella tecnológica de playcanvas.com con Netcraft Site Report. Host concreto en alcance (adquisición de Snap con infraestructura propia; dev*.playcanvas.com excluido y no consultado)
$ (navegador) https://sitereport.netcraft.com/?url=https://playcanvas.com
-> Background: "PlayCanvas | Open Source WebGL & WebGPU Game Engine", visto por primera vez en marzo de 2011
-> Red: IPv4 18.66.171.80 (sin IPv6), AS16509, rango AMAZO-CF (Amazon); rDNS server-18-66-171-80.dub56.r.cloudfront.net: nodo de CloudFront en Dublín. La IP es del CDN y el servidor de origen queda oculto. Infraestructura en AWS, a diferencia de los hosts de Snap en Google
-> DNS: Route 53 (ns-579.awsdns-08.net), MarkMonitor (DNStination Inc.), DNSSEC activado. Mismo patrón que snapchat.com y sc-corp.net
-> TLS: certificado DV de Amazon (Amazon RSA 2048 M01, compatible con AWS Certificate Manager); SAN playcanvas.com y *.playcanvas.com (el comodín cubre dev*.playcanvas.com, excluido del programa y no consultado); válido 2026-09-14 a 2027-03-30, 3 SCT
-> TLS: TLSv1.3, TLS_AES_128_GCM_SHA256 (los otros hosts usaban AES-256), RSA 2048, PFS sí, h2; sin SSLv3; Heartbleed no; OCSP stapling válido (único de los cuatro hosts)
-> Correo: SPF propio con include:_spf.google.com (Google Workspace), include:spf.mandrillapp.com (Mandrill/Mailchimp) y -all; sin DMARC
-> Tecnología (Netcraft, rastreo de hoy): CloudFront, Route 53, JavaScript, Google Tag Manager (1 tracker), gzip, HTML5 con canvas y video; cabeceras X-Frame-Options Deny, CSP, Referrer-Policy, X-Content-Type-Options, X-XSS-Protection. Netcraft no detecta HSTS
-> Captura: tecnica-04-huella/capturas/netcraft-playcanvas.com.jpeg
siguiente: cerrar técnica 4 (4 hosts de 4 zonas) y pasar a técnica 5

2026-11-02 19:15 · Ver directivas de rastreo de accounts.snapchat.com
$ curl -sS -i --max-time 15 https://accounts.snapchat.com/robots.txt -o tecnica-05-ficheros/accounts.snapchat.com_robots.txt
-> HTTP/2 200. Allow: / sin Disallow (no revela rutas). Directiva Host (legado Yandex). Sitemap: /sitemap.xml (pendiente).
   Cabeceras confirman T4: server "API Gateway", Envoy, via 1.1 google. HSTS con includeSubDomains, sin preload.
   ETag W/"tamaño-mtime" en hex → probablemente Node.js (Express/serve-static).
siguiente: /.well-known/security.txt en accounts.snapchat.com

2026-11-02 19:17 · Buscar security.txt (ubicación RFC 9116) en accounts.snapchat.com
$ curl -sS -i --max-time 15 https://accounts.snapchat.com/.well-known/security.txt -o tecnica-05-ficheros/accounts.snapchat.com_well-known_security.txt
-> HTTP/2 404 (página de error HTML de la app). No hay security.txt en /.well-known/.
   404 servido por la app (no por estáticos): HSTS y "via: 1.1 google" duplicados → dos capas tras el gateway.
   CSP revela dependencias y hosts: staging-us-central1-gcp.api.snapchat.com, chat-gold.sc-corp.net (en comodín),
   sentry.sc-prod.net, *.snap-dev.net, hosts preprod/hotfix de snapchat.com, localhost:3000 en form-action,
   http://softserve.snapchat.com (sin TLS) en frame-ancestors. Terceros: Arkose Labs, reCAPTCHA, GA/GTM, Sentry, Contentful, Bitmoji.
   Solo documentados; no se contactan (varios fuera de alcance).
   Otros: X-Frame-Options allow-from (obsoleto), cookie sc-wcid sin HttpOnly/SameSite, build frontend 1793222690.
pendiente: cruzar hosts de la CSP con la lista de T3 (local, sin peticiones)
siguiente: /security.txt (ubicación legacy) en accounts.snapchat.com

2026-11-02 19:18 · Buscar security.txt (ubicación legacy) en accounts.snapchat.com
$ curl -sS -i --max-time 15 https://accounts.snapchat.com/security.txt -o tecnica-05-ficheros/accounts.snapchat.com_security.txt
-> HTTP/2 404, misma página de error de la app. Sin security.txt en ninguna ubicación
   (coherente con reportes vía HackerOne). CSP idéntica a la anterior → política global de la app.
   Solo cambian alt-svc (orden) y trace-id → probablemente otro nodo del balanceador.
siguiente: commit del host; después robots.txt en app.snapchat.com

2026-11-02 19:20 · Ver directivas de rastreo de app.snapchat.com
$ curl -sS -i --max-time 15 https://app.snapchat.com/robots.txt -o tecnica-05-ficheros/app.snapchat.com_robots.txt
-> HTTP/2 404, página de error genérica de Google (no de Snapchat). server "Google Frontend": confirma T4, servicio en App Engine/Cloud Run sin capa web propia.
   x-snapchat-notice "Snapchat Private APIs - Unauthorized use is prohibited" → host de API, sin contenido indexable; no tiene robots.txt.
   x-snapchat-request-id en hex, parte decodificable: "feelinsonice-hrd" (probablemente ID del proyecto App Engine; -hrd = High Replication Datastore, proyecto antiguo),
   "master" (probablemente versión) y 1793379376 (= 2026-10-30 16:56 UTC, probablemente último despliegue). Interpretación, no confirmado.
   Respuesta sin HSTS, CSP ni X-Content-Type-Options (habitual en 404 de API; observación).
siguiente: /.well-known/security.txt en app.snapchat.com

2026-11-02 19:21 · Verificar decodificación del x-snapchat-request-id (local, sin peticiones al objetivo)
$ echo 6665656c696e736f6e6963652d687264 | xxd -r -p
$ echo 6d6173746572 | xxd -r -p
$ echo 31373933333739333736 | xxd -r -p
$ date -u -d @1793379376
-> "feelinsonice-hrd", "master", "1793379376" → 2026-10-30 16:56:16 UTC (17:56 hora local).
   Decodificación confirmada; el significado de cada campo (proyecto, versión, despliegue) sigue siendo interpretación.
siguiente: /.well-known/security.txt en app.snapchat.com

2026-11-02 19:22 · Buscar security.txt (ubicación RFC 9116) en app.snapchat.com
$ curl -sS -i --max-time 15 https://app.snapchat.com/.well-known/security.txt -o tecnica-05-ficheros/app.snapchat.com_well-known_security.txt
-> HTTP/2 404, misma página de error genérica de Google Frontend. Sin security.txt en /.well-known/.
   x-snapchat-request-id: mismo proyecto (feelinsonice-hrd), versión (master) y despliegue (1793379376) que la petición anterior.
   Primeros 4 bytes cambian 6ae8d483 → 6ae8d4d1: diferencia 78 = segundos entre las dos peticiones (18:20:51 → 18:22:09).
   El final del ID también cambia (…0132 → …0146), significado desconocido.
siguiente: verificar en local que los primeros 4 bytes son el timestamp de la petición

2026-11-02 HH:MM · Verificar que el inicio del x-snapchat-request-id es el timestamp de la petición (local, sin peticiones)
$ date -u -d @$((16#6ae8d4d1))
-> (resultado). Coincide con la cabecera date → formato del ID: timestamp + proyecto + versión + despliegue, compatible con IDs de log de App Engine.
siguiente: /security.txt (ubicación legacy) en app.snapchat.com

2026-11-02 19:22 · Verificar que el inicio del x-snapchat-request-id es el timestamp de la petición (local, sin peticiones)
$ date -u -d @$((16#6ae8d4d1))
-> Mon Nov 2 18:22:09 UTC 2026, coincide con la cabecera date. Formato del ID: timestamp + proyecto + versión + despliegue, compatible con IDs de log de App Engine.
siguiente: /security.txt (ubicación legacy) en app.snapchat.com

2026-11-02 19:23 · Buscar security.txt (ubicación legacy) en app.snapchat.com
$ curl -sS -i --max-time 15 https://app.snapchat.com/security.txt -o tecnica-05-ficheros/app.snapchat.com_security.txt
-> HTTP/2 404, error genérico de Google Frontend. Sin security.txt en ninguna ubicación.
   Request-id empieza por 6ae8d50f: +62 respecto al anterior = 62 s entre peticiones (18:22:09 → 18:23:11). Tercera muestra coherente: estructura del ID confirmada.
   Final del ID: …0132, …0146, …0160 en las tres peticiones; significado desconocido.
siguiente: commit del host; después robots.txt en developer-portal.sc-corp.net

2026-11-02 19:24 · Ver directivas de rastreo de developer-portal.sc-corp.net
$ curl -sS -i --max-time 15 https://developer-portal.sc-corp.net/robots.txt -o tecnica-05-ficheros/developer-portal.sc-corp.net_robots.txt
-> HTTP/2 302 a accounts.google.com (no seguida). Cuerpo "Invalid IAP credentials: empty token".
   Confirma hipótesis de T4: Google Identity-Aware Proxy (x-goog-iap-generated-response: true, cookie GCP_IAP_XSRF_NONCE Secure+HttpOnly 10 min,
   redirect_uri en iap.googleapis.com). IAP intercepta todas las rutas: robots.txt no accesible sin autenticación; sin cabeceras del backend (server, HSTS).
   Location: client_id OAuth 993232849302-… (público por diseño; 993232849302 probablemente número de proyecto GCP), scope "openid email", PKCE S256.
   state opaco, no decodificado.
siguiente: /.well-known/security.txt en developer-portal.sc-corp.net

2026-11-02 19:26 · Buscar security.txt (ubicación RFC 9116) en developer-portal.sc-corp.net
$ curl -sS -i --max-time 15 https://developer-portal.sc-corp.net/.well-known/security.txt -o tecnica-05-ficheros/developer-portal.sc-corp.net_well-known_security.txt
$ grep -iE "^HTTP|x-goog-iap|^set-cookie|Invalid IAP" tecnica-05-ficheros/developer-portal.sc-corp.net_well-known_security.txt
-> HTTP/2 302 generado por IAP (x-goog-iap-generated-response: true), mismo cuerpo "Invalid IAP credentials: empty token".
   Cookie GCP_IAP_XSRF_NONCE con nombre distinto al de la petición anterior → nonce único por intento. /.well-known/ también protegido.
siguiente: /security.txt (ubicación legacy) en developer-portal.sc-corp.net

2026-11-02 19:28 · Buscar security.txt (ubicación legacy) en developer-portal.sc-corp.net
$ curl -sS -i --max-time 15 https://developer-portal.sc-corp.net/security.txt -o tecnica-05-ficheros/developer-portal.sc-corp.net_security.txt
$ grep -iE "^HTTP|x-goog-iap|^set-cookie|Invalid IAP" tecnica-05-ficheros/developer-portal.sc-corp.net_security.txt
-> HTTP/2 302 generado por IAP, mismo cuerpo, nonce nuevo. Las 3 rutas interceptadas por IAP:
   ningún fichero accesible sin autenticación (portal interno correctamente protegido).
siguiente: commit del host; después robots.txt en playcanvas.com
