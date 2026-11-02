# Mapa de superficie de ataque · Snapchat (HackerOne)

Proyecto 1 · Know Your Target · Parte 1 (reconocimiento pasivo, 2026-10-28 a 2026-11-02).


**Alcance:** comodines `*.sc-core.net` y `*.sc-corp.net`, más 30 hosts concretos. `snapchat.com` y `playcanvas.com` no son comodín. Exclusiones: `support.snapchat.com`, `returns.spectacles.com`, `dev.playcanv.as` y `dev*.playcanvas.com`. Detalle en [alcance.md](alcance.md) (17:17, 18:03).

---

## 1. Resumen

**La organización en cinco líneas**

1. Snap Inc. es una empresa cotizada de entre 1001 y 5000 empleados con sede en California. Opera Snapchat, Bitmoji, Spectacles y PlayCanvas, un motor de juegos web con sede en Londres (20:36, 20:44).
2. Sus dominios están registrados con MarkMonitor y el titular aparece oculto tras DNStination Inc. (snapchat.com, desde 2012). Todo el DNS está en AWS Route 53 y ninguno de los dominios tiene DNSSEC (17:25, 17:27, 17:54, 20:51).
3. La infraestructura es multinube. Los servicios de Snapchat están en Google Cloud y PlayCanvas en AWS CloudFront. El correo es Google Workspace con SPF `-all` y DMARC `p=reject` (18:48, 18:59, 19:11, 17:35, 17:36).
4. `sc-corp.net` agrupa las herramientas internas (Jira, wiki, GitHub Enterprise, Gerrit, Spinnaker, CA de SSH). La mayoría comparten una misma IP de Google Cloud y al menos una está protegida con Google IAP (18:32, 19:24, 20:16, 20:56).
5. `sc-core.net` parece la red de oficinas (35 salas de vídeo Q-SYS, FortiManager, alta de wifi) y no tiene huella pública: 0 resultados en Google y sin registro A en los hosts consultados (18:32, 19:56, 20:56).

**Lo más relevante**

- **Herramientas internas expuestas a Internet**, con muchas variantes de desarrollo y preproducción (`jira-dev`, `wiki-stage`, `secproxy-dev`, `identity-broker-stage`…), todas dentro del comodín en alcance (18:32).
- **Un entorno de staging indexado en Google**: el login de EngFlow en `engflow-cache-rbe-gcp-staging.sc-corp.net` (19:47).
- **La CSP de `accounts.snapchat.com` revela hosts de preproducción y hotfix**, además de `localhost:3000` en `form-action` (19:17, 19:35).
- **Ningún host publica `security.txt`** y Shodan solo ve los puertos 80/443, sin vulnerabilidades conocidas (19:33, 19:41).

---

## 2. Dominios y subdominios

**Totales:** 803 subdominios únicos (subfinder + crt.sh). De ellos, 338 están en alcance (308 bajo los comodines y los 30 hosts concretos) y 465 fuera de alcance. Bajo `sc-core.net` hay 35 hosts `*.qsys` (18:29, 18:31, 18:32). La lista completa no se publica (`.gitignore`).

La tabla recoge los hosts con información en el cuaderno. Las IP son las resueltas el 2026-11-02 con el resolver 8.8.8.8 (20:56).

### En alcance

| Subdominio | IP | En alcance | Para qué parece servir | Ref. |
|---|---|---|---|---|
| accounts.snapchat.com | 34.149.46.130 | Sí | Login, registro y suscripción Snapchat+. API Gateway + Envoy en Google Cloud | 18:48, 19:15, 20:03 |
| app.snapchat.com | 34.149.130.113 | Sí | API privada ("Snapchat Private APIs") en App Engine, proyecto `feelinsonice-hrd` | 18:59, 19:20, 19:21 |
| ads.snapchat.com | 34.149.46.130 | Sí | Plataforma de anuncios (por nombre). Misma IP que accounts | 20:56 |
| business.snapchat.com | 34.149.46.130 | Sí | Snapchat para empresas (por nombre). Misma IP que accounts | 20:56 |
| web.snapchat.com | 34.149.46.130 | Sí | Cliente web de Snapchat (por nombre). Misma IP que accounts | 20:56 |
| businesshelp.snapchat.com | 145.224.212.113, .122, .124 | Sí | Centro de ayuda para empresas (por nombre). Red distinta al resto, proveedor sin verificar | 20:31, 20:56 |
| story.snapchat.com | 3.163.248.4 | Sí | Historias públicas (por nombre). Proveedor sin verificar | 20:56 |
| kit.snapchat.com | 35.190.43.134 | Sí | Snap Kit, SDK para desarrolladores (por nombre) | 20:56 |
| snappublisher.snapchat.com | 108.177.15.121 | Sí | Herramienta de publicación (por nombre). Misma IP que developer-portal | 18:12, 20:56 |
| spectacles.com | 34.120.54.238 | Sí | Web de las gafas Spectacles. Shodan la asocia a la IP de accounts, dato histórico | 19:42, 20:56 |
| playcanvas.com | 108.157.128.35, .60, .118, .127 | Sí | Portada del motor PlayCanvas y galería `/explore`. AWS CloudFront | 19:11, 19:29, 20:14 |
| login.playcanvas.com | 108.157.128.35, .60, .118, .127 | Sí | Inicio de sesión de PlayCanvas (por nombre). Mismas IP que playcanvas.com | 20:34, 20:56 |
| blog.playcanvas.com | 185.199.108–111.153 | Sí | Blog (por nombre). Rango compatible con GitHub Pages, sin verificar | 20:34, 20:56 |
| developer.playcanvas.com | 185.199.108–111.153 | Sí | Documentación para desarrolladores (por nombre). Mismas IP que blog | 18:31, 20:34, 20:56 |
| code.playcanvas.com | 99.84.9.11, .12, .76, .97 | Sí | Código del motor servido a las páginas. Aparece en la CSP de playcanvas.com | 19:29, 19:35 |
| msg.playcanvas.com | 18.154.43.15, .48, .71, .119 | Sí | Mensajería en tiempo real (por nombre). En la CSP de playcanvas.com | 19:29, 19:35 |
| relay.playcanvas.com | 18.154.43.15, .48, .71, .119 | Sí | Relé de conexiones (por nombre). Mismas IP que msg | 19:29, 19:35 |
| developer-portal.sc-corp.net | 108.177.15.121 | Sí | Portal interno protegido por Google IAP (login con cuenta de Google) | 19:08, 19:24–19:28 |
| chat-gold.sc-corp.net | 35.190.16.123 | Sí | Chat interno (por nombre). Aparece en la CSP de accounts | 19:17, 19:35 |
| wiki.sc-corp.net | 35.190.16.123 | Sí | Wiki interna, citada en un repositorio oficial de Snap en GitHub | 20:16 |
| github.sc-corp.net | 35.190.16.123 | Sí | GitHub Enterprise interno (hipótesis, por instrucciones de clonado públicas) | 20:16 |
| jira.sc-corp.net | 35.190.16.123 | Sí | Jira interno (por nombre). Hay variantes `-dev`, `-stage` y `-api` | 18:32 |
| gerrit.sc-corp.net | 35.190.16.123 | Sí | Revisión de código Gerrit (por nombre) | 18:32 |
| secproxy.sc-corp.net | 35.190.16.123 | Sí | Proxy de seguridad (por nombre). Tiene subdominios propios (`*.secproxy.sc-corp.net`) | 18:32 |
| identity-broker-prod.sc-corp.net | 35.190.16.123 | Sí | Intermediario de identidad, producción (por nombre). Existe `-stage` | 18:32 |
| spinnaker.sc-corp.net | 35.227.254.174 | Sí | Despliegue continuo con Spinnaker (por nombre) | 18:32 |
| ssh-ca.sc-corp.net | 35.227.192.89 | Sí | Autoridad de certificados SSH (por nombre). Existe `-dev` | 18:32 |
| engflow-cache-rbe-gcp-staging.sc-corp.net | 136.119.114.15 | Sí | Caché de compilación remota EngFlow en staging. Login indexado en Google | 19:47 |
| stream.sc-corp.net | 167.82.38.132 | Sí | Vídeo con StreamShark (Sticker Kit, Login Kit) | 19:47 |
| fortimanager.sc-core.net | sin registro A | Sí | Gestión de cortafuegos Fortinet (por nombre). Sin IP pública | 18:32, 20:56 |
| wifienroll.sc-core.net | sin registro A | Sí | Alta de dispositivos en la wifi corporativa (por nombre). Sin IP pública | 18:32, 20:56 |

### Fuera de alcance (documentados, no contactados)

| Subdominio | IP | En alcance | Para qué parece servir | Ref. |
|---|---|---|---|---|
| support.snapchat.com | no consultada | No (exclusión) | Soporte (por nombre). Apareció en T3 y T8 | 18:31, 20:25 |
| returns.spectacles.com | no consultada | No (exclusión) | Devoluciones de Spectacles (por nombre) | 18:31 |
| api.playcanvas.com | no consultada | No | API de PlayCanvas (por nombre). Conocido en T3, no listado en el alcance | 20:34 |
| staging-us-central1-gcp.api.snapchat.com | no consultada | No | API de staging. Aparece en la CSP de accounts | 19:17 |
| softserve.snapchat.com y variantes `hotfix-dot-` / `preproduction-dot-` | no consultada | No | Servicio App Engine con entornos hotfix y preproducción. En la CSP de accounts | 19:35 |
| snap-profile-manager.snapchat.com y variantes | no consultada | No | Gestión de perfiles en App Engine. En la CSP de accounts | 19:35 |

**Patrones observados en las IP** (20:56):

- `34.149.46.130` es compartida por accounts, ads, business y web: un mismo balanceador de Google Cloud de Snap, coherente con los 19 nombres que Shodan asocia a esa IP (19:41).
- `35.190.16.123` es compartida por siete herramientas internas de sc-corp.net (chat-gold, wiki, github, jira, gerrit, secproxy, identity-broker-prod). *Hipótesis:* un frontal común que aplica la autenticación antes de llegar a cada herramienta.
- snappublisher.snapchat.com y developer-portal.sc-corp.net resuelven a la misma IP de frontal de Google (`108.177.15.121`).

---

## 3. Infraestructura y servicios

### Registro y DNS

| Dominio | Registrador / titular | Alta · caducidad | Bloqueos | NS | DNSSEC | Ref. |
|---|---|---|---|---|---|---|
| snapchat.com | MarkMonitor / DNStination Inc. (proxy) | 2012-02-28 · 2034-02-28 | 6 (client y server) | Route 53 | No | 17:25, 17:27, 20:51 |
| sc-corp.net | MarkMonitor | 2014-07-03 · 2028-07-03 | 3 (solo client) | Route 53, zona propia | No | 17:54, 20:51 |
| sc-core.net | MarkMonitor | 2015-04-30 · 2028-04-30 | 3 (solo client) | Route 53, zona propia | No | 17:54, 20:51 |
| playcanvas.com | MarkMonitor / DNStination Inc. | — | — | Route 53 | No | 19:11, 20:51 |

Netcraft indica "DNSSEC activado" para snapchat.com, sc-corp.net y playcanvas.com, pero la zona padre no tiene registro DS para ninguno de los cuatro dominios. El dato de Netcraft es erróneo (20:51).

### Hosting y servicios en la nube

| Host | Red | Frontal / servicio | Ref. |
|---|---|---|---|
| accounts.snapchat.com | Google Cloud (AS396982) | API Gateway + Envoy tras el balanceador HTTPS de Google | 18:48, 19:15 |
| app.snapchat.com | Google Cloud (AS396982) | Google Frontend, App Engine (proyecto `feelinsonice-hrd`, versión `master`) | 18:59, 19:20–19:23 |
| developer-portal.sc-corp.net | Red principal de Google (AS15169) | Google Identity-Aware Proxy con OAuth y PKCE | 19:08, 19:24 |
| playcanvas.com | Amazon (AS16509) | AWS CloudFront (nodos de Dublín y Madrid); origen oculto | 19:11, 19:29 |

- **Multinube:** el DNS está en AWS, los servicios de Snapchat en Google Cloud y PlayCanvas en AWS. `sc-corp.net` muestra nombres de AWS, GCP, Oracle Cloud y App Engine (18:32).
- **Sin verificar:** los propietarios de las redes de businesshelp, story, kit, code, msg, relay, blog, developer, spinnaker, ssh-ca, engflow y stream (20:56).

### Tecnologías

- **Backend:** el formato de los ETag de accounts.snapchat.com y playcanvas.com corresponde al módulo `etag` de Node.js (compatible con Express) (19:15, 19:30, 19:31).
- **TLS:** TLS 1.3 y PFS en los 4 hosts analizados, sin SSLv3. Hay tres familias de certificados: DigiCert OV `*.snap.com` (accounts), Google Trust Services DV de 90 días (app, developer-portal) y Amazon (playcanvas) (18:48–19:11).
- **Cabeceras:** accounts tiene HSTS con subdominios (sin preload), CSP, y la cookie `sc-wcid` sin HttpOnly ni SameSite. playcanvas.com no tiene HSTS y su CSP incluye `unsafe-inline` y `unsafe-eval` (19:15, 19:17, 19:29).
- **Ficheros públicos:** los `robots.txt` no revelan rutas y no hay `security.txt` en ninguno de los 4 hosts; los reportes van por HackerOne (19:33).

### Correo

- **snapchat.com:** Google Workspace (5 MX de Google), SPF estricto `-all` con Google, Zendesk y Pardot, y DMARC `p=reject` al 100 % (17:35, 17:36).
- **playcanvas.com:** SPF propio con Google Workspace y Mandrill (Mailchimp), `-all`, sin DMARC (19:11).

### Terceros y SaaS

- **Verificaciones TXT en snapchat.com** (unas 40): Microsoft 365, Atlassian, Docker, Sonatype, Tailscale, Jamf, Adobe, HubSpot, Segment, OneTrust, Shopify, Dropbox, Miro, OpenAI, Anthropic, Apple, Censys, entre otros (17:36).
- **Terceros en las CSP:** Arkose Labs, reCAPTCHA, Google Analytics/Tag Manager, Sentry, Contentful y Bitmoji (accounts); Stripe y S3 eu-west-1 (playcanvas) (19:17, 19:29).

### Lo que muestra Shodan (InternetDB)

- **Puertos y vulnerabilidades:** las 7 IP consultadas solo tienen abiertos 80 y 443, sin vulnerabilidades conocidas (19:41).
- **IP de accounts (`34.149.46.130`):** 19 nombres de Snap asociados (snap.com, bitmoji.com, spectacles.com, api.snapchat.com…) y la etiqueta `pqc`. *Hipótesis:* balanceador de Google Cloud propio de Snap (19:41, 19:42).
- Shodan no indica la fecha de su escaneo, así que estos datos pueden no ser actuales (19:41).

---

## 4. Correos y personas

### Patrón de correo: no determinado

No se ha podido obtener, por cuatro motivos:

1. **Hunter.io**, en el plan gratuito, enmascara la parte local de los correos y no muestra el patrón (20:36, 20:44).
2. **theHarvester**, solo con fuentes sin clave API, no encontró ningún correo personal (20:25, 20:32).
3. **El servidor de snapchat.com es catch-all** (acepta cualquier dirección), así que no se puede confirmar un buzón probando direcciones (20:36).
4. **Por diseño de la fase pasiva**, no se usó el Email Verifier de Hunter, para no contactar con los servidores de correo del objetivo (20:36).

No se propone un patrón deducido porque no se podría rastrear a un dato del cuaderno.

### Correos encontrados

| Dominio | Hunter.io (personas / genéricos) | Buzones genéricos conocidos | Ref. |
|---|---|---|---|
| snapchat.com | 91 (44 / 47) | `dmarc-report@snapchat.com` (informes DMARC) | 17:36, 20:36 |
| snap.com | 106 (64 / 42) | — | 20:44 |
| playcanvas.com | 6 (1 / 5) | `support@playcanvas.com` | 20:32, 20:44 |

- El correo corporativo indexado está sobre todo en **snap.com**, que no es un host en alcance (20:45).
- Los correos de personas no se guardan en el repositorio (`.gitignore`) (20:23, 20:45).

### Lo averiguado de la organización

- **Empresa:** cotizada, 1001–5000 empleados. Hunter da dos sedes distintas: Venice (snapchat.com) y Santa Monica (snap.com), ambas en California (20:36, 20:44).
- **PlayCanvas:** adquisición de Snap con infraestructura propia; videojuegos, 11–50 empleados, fundada en 2011, Londres (19:11, 20:44).
- **Herramientas de trabajo internas:** Jira, wiki, GitHub Enterprise, Gerrit, Spinnaker, EngFlow (compilación remota) y una CA de SSH (18:32, 19:47, 20:16).
- **Identidad y accesos:** Google IAP y un servicio de *identity broker* (19:24, 18:32).
- **Gestión de equipos y red:** Jamf y Tailscale según los TXT (17:36); FortiManager y alta de wifi en sc-core.net (18:32).
- **Oficinas** (por los códigos en sc-core.net): Londres (`lon03`), Ámsterdam (`ams`), Oxford (`oxf02`), Portland (`pdx01`) y `ein100` (posiblemente Eindhoven) (18:32).
- **Registro de dominios:** no aparecen personas ni correos de Snap; el titular está oculto tras DNStination Inc. (17:27).

---

## 5. Puntos de interés

Estos son los tres elementos que investigaría primero en la fase activa, siempre dentro de las reglas del programa.

### 1. Herramientas internas de sc-corp.net tras el frontal `35.190.16.123`

- **Qué es:** siete herramientas internas (Jira, wiki, GitHub Enterprise, Gerrit, chat, secproxy, identity broker) comparten una IP pública. El único host de esta zona que se ha probado, developer-portal, está correctamente protegido con Google IAP (19:24–19:28, 20:56).
- **Por qué:** todas están dentro del comodín en alcance y tienen muchas variantes de desarrollo y preproducción (`jira-dev`, `jira-stage-api`, `wiki-stage`, `secproxy-dev`, `identity-broker-stage`, `ssh-ca-dev`…) (18:32). El riesgo está en que una sola variante quede sin la misma protección. Si eso pasa, el impacto es alto: código fuente, documentación interna, despliegues y certificados SSH.
- **Primer paso:** comprobar si cada variante exige autenticación igual que developer-portal.

### 2. Staging de EngFlow: `engflow-cache-rbe-gcp-staging.sc-corp.net`

- **Qué es:** un entorno de staging de compilación remota, con pantalla de login indexada por Google. Es el único host de los comodines con página propia en el buscador. Hay variantes `-prod` y sin sufijo (19:47, 18:32).
- **Por qué:** los entornos de staging suelen estar menos protegidos que producción. Una caché de compilación maneja artefactos y, potencialmente, código fuente. Además, su IP (`136.119.114.15`) es distinta del frontal común de las demás herramientas (20:56), así que podría no compartir su protección.

### 3. `accounts.snapchat.com`

- **Qué es:** el servicio de login, registro y suscripción de pago. Es la función pública más sensible y comparte balanceador con ads, business y web (18:48, 20:03, 20:56).
- **Por qué:**
  - Su CSP revela hosts de staging, preproducción y hotfix, permite `localhost:3000` en `form-action` y `http://softserve.snapchat.com`, sin TLS, en `frame-ancestors` (19:17, 19:35).
  - Usa `X-Frame-Options allow-from`, que está obsoleto (19:17).
  - La cookie `sc-wcid` no tiene HttpOnly ni SameSite (19:17).
- **Primer paso:** revisar los flujos de autenticación y si esas directivas permisivas tienen efecto real. Ojo: los hosts que aparecen en la CSP están fuera de alcance y no se deben atacar.

**Otros candidatos descartados por ahora:**
- playcanvas.com: sin HSTS, con `unsafe-inline` y `unsafe-eval` (19:29).
- app.snapchat.com: API privada en App Engine (19:20).
