# Alcance · Programa de Snapchat (Snap Inc.) en HackerOne

## 1. Programa

| Campo | Valor |
|---|---|
| Organización | Snap Inc. (Snapchat, Bitmoji, PlayCanvas, Spectacles) |
| Plataforma | HackerOne |
| Tipo | Bug bounty público (con recompensa económica) |
| Enlace al alcance | https://hackerone.com/snapchat/policy_scopes |
| Enlace a la política | https://hackerone.com/snapchat |
| Fecha de consulta | 2026-10-28 |
| Última actualización del alcance según la plataforma | 18/12/2024 |
| Evidencias en el repositorio | `scopes_for_snapchat_at_2026-10-28_10_46_12_UTC.csv` y `snapchat-guidelines.pdf` |

## 2. Qué está dentro del alcance

El CSV oficial contiene 46 activos. Los organizo en dos niveles, tal y como hace el propio programa (*Tier A · Core Assets* y *Tier B · Non Core*).

### 2.1 Comodines

| Activo | Descripción del programa | Severidad máx. |
|---|---|---|
| `*.sc-core.net` | Servicios internos de Snapchat | Critical |
| `*.sc-corp.net` | (sin descripción) | Critical |

### 2.2 Dominios de Snapchat · Tier A (core)

| Activo | Descripción del programa | Severidad máx. |
|---|---|---|
| `web.snapchat.com` | Snapchat en la web | Critical |
| `accounts.snapchat.com` | Gestión de cuentas | Critical |
| `app.snapchat.com` | Aplicación principal del servidor (Google App Engine) | Critical |
| `business.snapchat.com` | Business Manager | Critical |
| `store.snapchat.com` | Tienda de merchandising de Bitmoji | Critical |
| `lensstudio.snapchat.com/api/` | Código fuente (JavaScript) | Critical |
| `ads.snapchat.com` | Plataforma de anuncios | High |
| `create.snapchat.com` | Creación de Geofilters | High |
| `geofilters.snapchat.com` | Compra de Geofilters | High |
| `kit.snapchat.com` | Snap Kit (web y SDK) | High |
| `my.snapchat.com` | Spotlight en la web | High |
| `snappublisher.snapchat.com` | Herramienta de publicación | High |
| `businesshelp.snapchat.com` | Instancia de Salesforce | High |
| `scan.snapchat.com` | Creación de Snapcodes | Low |
| `map.snapchat.com` | Snap Map | Low |
| `story.snapchat.com` | Historias | Low |
| `spectacles.com` | Tienda de Spectacles | Low |

### 2.3 Dominios · Tier B (no core)

| Activo | Severidad máx. |
|---|---|
| `www.bitmoji.com`, `www.bitstrips.com` | Medium |
| `playcanvas.com`, `blog.`, `code.`, `developer.`, `forum.`, `launch.`, `login.`, `msg.`, `relay.`, `rt.`, `store.playcanvas.com` | Medium |
| `playcanv.as` | Medium |

### 2.4 Activos que no son dominios

No entran en el `scope.txt` porque no se investigan por DNS, pero forman parte del programa:

- Apps móviles: `com.snapchat.android` (Android), `com.toyopagroup.picaboo` (iOS), `com.bitstrips.imoji` (Bitmoji, iOS y Android).
- Lens Studio (ejecutable descargable).

## 3. Qué está fuera del alcance

| Activo | Motivo según el programa |
|---|---|
| `support.snapchat.com` | Web estática de soporte |
| `returns.spectacles.com` | Gestionada por un tercero (NetSuite) |
| `dev.playcanv.as` | Entorno de desarrollo |
| `dev*.playcanvas.com` | Entornos de desarrollo |
| Spectacles y su estuche de carga | Hardware |

## 4. Observaciones importantes para el proyecto

1. **No existe un comodín `*.snapchat.com`.** Los subdominios de `snapchat.com` están autorizados uno a uno. Cualquier subdominio que encuentre durante el reconocimiento y no aparezca en la lista anterior se considera **fuera de alcance**, aunque pertenezca a Snapchat.
2. **El dominio raíz `snapchat.com` no está listado**, así que tampoco está en alcance. Lo uso solo como punto de partida de las consultas pasivas (whois, DNS, crt.sh), que no envían tráfico al objetivo.
3. **`app.snapchat.com` se aloja en Google App Engine** según el propio programa (`feelinsonice-hrd.appspot.com`). Ese hostname no figura como activo, así que no lo incluyo en el `scope.txt`.

## 5. Reglas que afectan a este proyecto

La página de política del programa es breve: contiene la tabla de recompensas por nivel (Tier A y Tier B) y una única exclusión general, *"Core Ineligible Findings are out of scope"*, que remite a la lista estándar de HackerOne:
https://docs.hackerone.com/en/articles/8494488-core-ineligible-findings (consultada el 2026-10-28).

Además, la página indica que el programa cumple los *Platform Standards* de HackerOne, por lo que se aplican también las normas generales de la plataforma.

| Pregunta | Qué dice el programa |
|---|---|
| ¿Permite herramientas automáticas? | No lo menciona expresamente. En este proyecto no se usa ninguna herramienta que envíe tráfico al objetivo, así que no afecta. |
| ¿Límite de peticiones? | No fija un número. Sí excluye como *hazardous testing* cualquier tráfico excesivo o prueba que pueda afectar a la disponibilidad (DoS/DDoS). |
| ¿Prohíbe contactar con empleados? | Sí. La ingeniería social (phishing, abrir solicitudes de soporte, etc.) se considera *hazardous testing* y no debe intentarse sin autorización expresa. |
| ¿Requisito de identificación? | No exige cabecera ni identificador concreto en la política publicada. |
| Otras exclusiones relevantes | Hallazgos sin impacto real o de *hardening* opcional no son válidos: configuración de SPF/DKIM/DMARC, flags de cookies, la mayoría de problemas de rate limiting, clickjacking en páginas no sensibles, self-XSS, etc.

