## Historia de la Ciberseguridad — Línea de Tiempo

### 1960-1970: Los orígenes

|Año|Evento|
|---|---|
|1960s|**Phreaking**: John Draper (Captain Crunch) descubre que un silbato de juguete emite 2600 Hz, la frecuencia usada por AT&T para controlar líneas — permitía llamadas gratuitas. Nace el concepto de "hackear sistemas".|
|1971|**Creeper**, primer programa autorreplicante (Bob Thomas), mostraba _"I'M THE CREEPER: CATCH ME IF YOU CAN"_. Se creó **Reaper** para eliminarlo — el primer antivirus.|
|1975|Se funda el **Homebrew Computer Club** en California; Wozniak y Jobs presentan el Apple I. Nace la cultura hacker moderna (compartir conocimiento libremente).|

### 1980: la era dorada del hacking

|Año|Evento|
|---|---|
|1983|La película **WarGames** populariza el hacking (joven hackea NORAD, casi provoca una guerra) — impulsa la conciencia de ciberseguridad en el gobierno de EE.UU.|
|1986|**Computer Fraud and Abuse Act (CFAA)**, primera ley importante contra el cibercrimen en EE.UU., respuesta directa a WarGames.|
|1988|**Morris Worm** (Robert Tappan Morris): infectó ~6.000 equipos (10% de Internet de la época) por un error de programación, sin intención maliciosa. Primer condenado bajo la CFAA (libertad condicional, servicio comunitario, multa).|

### 1990: explosión de Internet y malware

|Año|Evento|
|---|---|
|1990|Guerra cibernética entre grupos: **Legion of Doom (LOD)** vs **Masters of Deception (MOD)**, exploraban sistemas telefónicos y redes.|
|1994|Microsoft desarrolla **PPTP**, el primer protocolo VPN.|
|1995|**Kevin Mitnick** capturado por el FBI tras hackear Motorola, Nokia y Sun Microsystems; 5 años de prisión, hoy consultor de seguridad.|
|1999|**Melissa Virus** (macro de Word, David L. Smith): ~1 millón de equipos infectados, ~$80M en daños.|

### 2000: ciberguerra y hacktivismo

|Año|Evento|
|---|---|
|2000|**MafiaBoy** (Michael Calce, 15 años) tumba Yahoo!, eBay, Amazon y CNN con DDoS — ~$1.7 mil millones en daños.|
|2003|Nace **Anonymous** en 4chan, colectivo hacktivista descentralizado.|
|2008|**Conficker Worm**: 9-15 millones de PCs infectados, botnet masiva, creador nunca identificado.|

### 2010: ataques sofisticados y APTs

|Año|Evento|
|---|---|
|2010|**Stuxnet** (presuntamente EE.UU./Israel) destruye ~1.000 centrifugadoras nucleares iraníes: 4 zero-days, propagación por USB, primera arma cibernética con daño físico conocido.|
|2013|**Edward Snowden** revela el programa de vigilancia masiva PRISM de la NSA.|
|2014|**Sony Pictures Hack** (Guardians of Peace, vinculado a Corea del Norte): robo y filtración de datos, respuesta a la película "The Interview".|
|2016|**Mirai Botnet**: infecta dispositivos IoT, ataca Dyn DNS y tumba Twitter/Netflix/Reddit; código publicado públicamente.|
|2017|**WannaCry**: ~230.000 equipos cifrados en 150 países vía EternalBlue (NSA); detenido por el kill switch de Marcus Hutchins.|
|2017|**Equifax**: 147 millones de personas afectadas, vulnerabilidad conocida de Apache Struts sin parchear; CEO renuncia, multa de $700M.|

### 2020 en adelante: ransomware, APTs y ciberguerra

|Año|Evento|
|---|---|
|2020|**SolarWinds**: cadena de suministro comprometida (backdoor SUNBURST), ~18.000 organizaciones afectadas incluyendo gobierno de EE.UU.|
|2021|**Colonial Pipeline** (grupo DarkSide): ransomware sobre infraestructura crítica, escasez de gasolina en la Costa Este de EE.UU., rescate de $4.4M.|
|2021|**Log4Shell (CVE-2021-44228)**: RCE crítico en Log4j, severidad 10/10 CVSS, afectó millones de servidores.|
|2022|**Hackeo de LastPass**: bóvedas cifradas robadas, migración masiva a Bitwarden.|
|2023-2024|Auge de IA en ciberseguridad: deepfakes, phishing generado por IA y malware adaptativo del lado ofensivo; detección automatizada y ML del lado defensivo.|

---

## Hackers Históricos de Referencia

|Nombre|Apodo|Notas|
|---|---|---|
|**Kevin Mitnick** (1963-2023)|The Condor|Hackeó Motorola/Nokia/Sun, 5 años de prisión, luego consultor y autor de _Ghost in the Wires_|
|**Adrian Lamo** (1981-2018)|The Homeless Hacker|Hackeó NYT/Microsoft/Yahoo!, reportó a Chelsea Manning (WikiLeaks), luego periodista de seguridad|
|**Gary McKinnon**|Solo|Hackeó 97 sistemas militares/NASA buscando evidencia de OVNIs; evitó extradición por motivos médicos|
|**Albert Gonzalez**|—|Robó 170M de números de tarjeta (TJX, Heartland), 20 años de prisión, ~$200M de ganancia|
|**Anonymous**|—|Colectivo hacktivista desde 2003 (OpChanology, OpPetrol, OpISIS)|
|**Lizard Squad**|—|DDoS a PSN y Xbox Live (2014), miembros menores de edad arrestados|

---

## Tipos de Hackers y Motivaciones

### Por ética

|Tipo|Definición|Notas|
|---|---|---|
|**White Hat**|Hackea legalmente, con permiso explícito|Certificaciones (CEH, OSCP, GPEN), reporta responsablemente. Ej.: Dan Kaminsky, Charlie Miller|
|**Black Hat**|Hackea para beneficio propio o dañar|Roba datos/dinero, instala malware; consecuencias legales severas|
|**Grey Hat**|Hackea sin permiso, pero sin intención de dañar|Reporta vulnerabilidades encontradas sin autorización previa — técnicamente ilegal aunque bienintencionado|

### Por motivación

|Perfil|Objetivo|Ejemplos/notas|
|---|---|---|
|**Financieros**|Dinero|Robo de tarjetas, ransomware, fraude bancario. Ej.: Carbanak (~$1.000M robados)|
|**Curiosos/Exploradores**|Conocimiento|Sin ánimo de lucro ni daño; muchos derivan a White Hat. Sigue siendo ilegal sin permiso|
|**Hacktivistas**|Causas políticas/sociales|DDoS, defacement, leaks, doxing. Ej.: Anonymous, LulzSec, Syrian Electronic Army|
|**State-Sponsored (APT)**|Espionaje y ciberguerra|Financiados por gobiernos, muy sofisticados, largo plazo|
|**Script Kiddies**|Ego/diversión|Usan herramientas ajenas (LOIC, exploits de GitHub) sin entenderlas; mismas consecuencias legales|
|**Insider Threats**|Interno (malicioso o negligente)|~30% de brechas involucran insiders; ej. Edward Snowden|

**Grupos APT conocidos:**

|Grupo|País (presunto)|Objetivos|
|---|---|---|
|APT28 (Fancy Bear)|Rusia|Gobiernos occidentales, OTAN|
|APT29 (Cozy Bear)|Rusia|Gobiernos, think tanks|
|APT1 (Unit 61398)|China|Propiedad intelectual|
|Lazarus Group|Corea del Norte|Financiero, infraestructura|
|Equation Group|EE.UU. (NSA)|Inteligencia global|

**Operaciones asociadas:** Stuxnet (EE.UU./Israel vs Irán), Sony Pictures (Corea del Norte), SolarWinds (Rusia), DNC Hack 2016 (Rusia).

---

## Distribución de Habilidad (referencia orientativa)

|Nivel|% aproximado|Perfil|
|---|---|---|
|Elite Hackers|~1%|APTs, 0-days, desarrollo de malware propio|
|Advanced|~9%|Pentesters, Red Teams|
|Intermediate|~30%|Bug bounty, jugadores de CTF|
|Script Kiddies|~60%|Usan herramientas de terceros sin entenderlas|

---

## Carreras en Hacking Ético

|Rol|Descripción|Certificaciones|Rango salarial (USD/año)|
|---|---|---|---|
|**Pentester**|Simula ataques para encontrar vulnerabilidades|OSCP, CEH, GPEN|$70K–$150K|
|**Red Team Operator**|Simula un ataque real completo (adversario avanzado), no solo busca el máximo de vulnerabilidades|OSCP, OSEP, OSCE, CRTO|$100K–$180K|
|**Bug Bounty Hunter**|Encuentra vulnerabilidades en programas de recompensas (HackerOne, Bugcrowd, Synack, Intigriti)|—|Variable ($0–$1M+)|
|**Security/SOC Analyst**|Monitorea SIEM, analiza logs, responde a incidentes|Security+, CySA+, GCIH|$60K–$110K|
|**Malware Analyst / Reverse Engineer**|Analiza malware (IDA Pro, Ghidra, x64dbg)|GREM, GCFA|$90K–$160K|
|**Forensics Investigator**|Investiga incidentes, recopila evidencia digital, cadena de custodia|GCFE, EnCE, CHFI|$70K–$130K|

Top bug bounty payouts de referencia: Apple y Google hasta $1M, Microsoft hasta $250K, Facebook hasta $40K.

---

# Principios Fundamentales de Seguridad

## Tríada CIA

|Pilar|Definición|Controles típicos|Ejemplo de violación|
|---|---|---|---|
|**Confidencialidad**|Solo personas autorizadas acceden a la información|Cifrado (AES, RSA), control de acceso, 2FA, VPN, clasificación de info|Equifax 2017 (147M de registros expuestos)|
|**Integridad**|La información no se modifica sin autorización|Hash (SHA-256), firmas digitales, control de versiones, checksums, backups inmutables|Stuxnet (alteró el control de centrifugadoras)|
|**Disponibilidad**|La información está accesible cuando se necesita|Redundancia (RAID), balanceo de carga, UPS, backups, DRP, protección DDoS|WannaCry 2017 (cifró sistemas del NHS)|

---

## AAA Framework

|Fase|Pregunta|Ejemplo|
|---|---|---|
|**Authentication**|¿Quién eres?|Usuario + contraseña|
|**Authorization**|¿Qué puedes hacer?|Permisos y roles|
|**Accounting**|¿Qué hiciste?|Logs y auditoría|

---

## No Repudio (Non-Repudiation)

No se puede negar haber realizado una acción: firmas digitales con certificado, logs con timestamp, transacciones en blockchain. Ej.: una firma digital en un contrato impide alegar después "yo no lo firmé".

---

## Principio de Mínimo Privilegio

Los usuarios solo deben tener los permisos mínimos necesarios para su función.

```
✅ Correcto:  usuario de contabilidad puede ver facturas y crear
              reportes, pero NO puede borrar la base de datos.
❌ Incorrecto: usuario de contabilidad con permisos de admin total.
```

Beneficio: limita el daño si la cuenta se ve comprometida.

---

## Defensa en Profundidad

Múltiples capas de seguridad independientes, de modo que si una falla, las demás siguen protegiendo:

```
Capa 7: Políticas y procedimientos
Capa 6: Educación de usuarios
Capa 5: Datos (cifrado)
Capa 4: Aplicaciones (WAF)
Capa 3: Host (antivirus)
Capa 2: Red (firewall)
Capa 1: Física (guardias, cámaras)
```

Analogía: un castillo medieval — foso, muralla exterior, guardias, torres, muralla interior, sala del tesoro cifrada.

---

## Seguridad por Diseño (Security by Design)

Integrar la seguridad desde el inicio del desarrollo, no como parche posterior.

```
Enfoque incorrecto:  desarrollar → lanzar a producción → parchear
                      vulnerabilidades sobre la marcha.

Enfoque correcto:    diseñar con seguridad en mente → threat
                      modeling en el diseño → code review de
                      seguridad → pentesting antes de lanzar →
                      lanzamiento con seguridad incorporada.
```