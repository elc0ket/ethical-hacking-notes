# Usuarios, Procesos y Bash Scripting

## Usuarios en Linux

|Tipo|UID|Ejemplo|
|---|---|---|
|Root|0|Superusuario, control total|
|System|1-999|Servicios (www-data, mysql)|
|Normal|≥1000|Usuarios humanos|

```bash
whoami / id / id username
who / w / last                      # sesiones activas / historial de logins

sudo useradd -m -s /bin/bash username     # crear con home y shell
sudo adduser username                     # interactivo (Debian/Ubuntu)
sudo passwd username

sudo usermod -aG sudo username      # agregar a grupo
sudo usermod -s /bin/zsh user       # cambiar shell
sudo usermod -L / -U username       # bloquear / desbloquear

sudo userdel -r username            # elimina también el home

su - username                        # cambiar de usuario con su entorno
sudo -u username comando             # ejecutar como otro usuario
sudo -i / sudo su / su -             # convertirse en root
```

### /etc/passwd

```
username:x:UID:GID:comentario:home:shell
root:x:0:0:root:/root:/bin/bash
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
```

`x` en el campo de password indica que el hash real está en `/etc/shadow`.

### /etc/shadow (solo root)

```
username:$hash:lastchange:min:max:warn:inactive:expire
```

Prefijo del hash: `$1$` MD5 · `$2a$` Blowfish · `$5$` SHA-256 · `$6$` SHA-512. `!` o `*` en vez de hash = cuenta bloqueada.

### Grupos

```bash
groups / groups username
sudo groupadd developers
sudo usermod -aG groupname username     # agregar (o: gpasswd -a)
sudo gpasswd -d username groupname      # remover
cat /etc/group                          # groupname:x:GID:usuario1,usuario2
```

---

## Permisos

`r`=4, `w`=2, `x`=1.

```bash
ls -l archivo.txt
# -rw-r--r-- 1 john users 1234 ene 27 23:00 archivo.txt
#  │└┬┘└┬┘└┬┘
#  │ │  │  └ otros
#  │ │  └──── grupo
#  │ └─────── propietario
#  └───────── tipo: - regular, d directorio, l symlink, c/b device, s socket, p pipe
```

### chmod

```bash
# Simbólico
chmod u+x archivo.sh / chmod g+w archivo.txt / chmod o+r archivo.txt / chmod a+x archivo.sh
chmod u=rwx,g=rx,o=r archivo.txt

# Octal
chmod 755 script.sh      # rwxr-xr-x — ejecutable
chmod 644 archivo.txt    # rw-r--r-- — archivo normal
chmod 600 id_rsa         # rw------- — archivo privado
chmod 700 directorio/    # rwx------ — directorio privado

chmod -R 755 directorio/  # recursivo
```

### chown / chgrp

```bash
sudo chown username:groupname archivo.txt
sudo chown -R username:groupname directorio/
sudo chgrp groupname archivo.txt
```

### Permisos especiales

|Permiso|Valor octal|Efecto|Buscar|
|---|---|---|---|
|**SUID**|4 (`chmod u+s` / `4755`)|El archivo se ejecuta con los permisos del propietario, no de quien lo lanza — clave en privesc: un SUID de root ejecutado por cualquier usuario corre como root|`find / -perm -4000 -type f 2>/dev/null`|
|**SGID**|2 (`chmod g+s` / `2755`)|En archivos: ejecuta con permisos del grupo. En directorios: los archivos nuevos heredan el grupo del directorio|`find / -perm -2000 -type f 2>/dev/null`|
|**Sticky bit**|1 (`chmod +t` / `1777`)|Solo el propietario puede borrar sus propios archivos dentro del directorio (uso típico: `/tmp`)|`ls -ld /tmp` → `drwxrwxrwt`|

### umask

```bash
umask            # default 0022
# Permisos resultantes: archivos = 666-umask, directorios = 777-umask
umask 077         # archivos 600, directorios 700 (temporal)
echo "umask 077" >> ~/.bashrc   # permanente
```

---

## sudo

```bash
sudo comando
sudo -u username comando
sudo -i / sudo su                  # shell root interactiva
sudo !!                             # repetir el comando anterior con sudo
sudoedit /etc/hosts                 # editar archivo protegido
sudo -l                             # ver qué puede ejecutar el usuario actual
```

### /etc/sudoers (editar SIEMPRE con `visudo`)

```
usuario HOST=(USER) COMMANDS

john ALL=(ALL:ALL) ALL              # acceso total
alice ALL=(ALL) NOPASSWD: ALL       # sin contraseña
bob ALL=(ALL) /usr/bin/apt          # solo un binario

%sudo ALL=(ALL:ALL) ALL             # por grupo
%admin ALL=(ALL) NOPASSWD: ALL
```

---

# Procesos y Servicios

## Ver y gestionar procesos

```bash
ps aux                              # todos los procesos (a=todos los usuarios, u=formato user, x=sin TTY)
ps aux --sort=-%mem | head          # top por memoria
ps aux --sort=-%cpu | head          # top por CPU
pstree / ps auxf                    # árbol de procesos
top / htop                          # interactivo en tiempo real

pgrep -l apache2                    # PID por nombre
sudo lsof -p PID                     # archivos abiertos por el proceso
sudo lsof -i -P -n | grep PID        # puertos usados por el proceso
sudo ss -tulpn | grep PID
```

### Señales

|Señal|Nº|Efecto|
|---|---|---|
|SIGHUP|1|Hangup (reiniciar servicio)|
|SIGINT|2|Interrupt (Ctrl+C)|
|SIGKILL|9|Terminación forzada, no puede ignorarse|
|SIGTERM|15|Terminación normal (default de `kill`)|
|SIGSTOP|19|Pausar|
|SIGCONT|18|Continuar|

```bash
kill PID / kill -9 PID
pkill apache2 / pkill -9 apache2
pkill -u username                   # matar todos los procesos de un usuario
```

### Prioridad y background

```bash
nice -n 10 comando                  # menor prioridad al iniciar
sudo renice -10 -p PID              # mayor prioridad (root) sobre proceso existente

comando &                           # ejecutar en background
jobs / fg %1 / bg %1                # ver / traer a foreground / enviar a background
# Ctrl+Z pausa el proceso en foreground actual

nohup comando > output.txt 2>&1 &   # desconectar del terminal
```

---

## Servicios (systemd)

```bash
sudo systemctl start|stop|restart|reload apache2
systemctl status apache2
sudo systemctl enable|disable apache2      # inicio automático
systemctl is-enabled apache2 / is-active apache2

systemctl list-units --type=service --state=running
systemctl list-unit-files --type=service --state=enabled
systemctl --failed
```

### journalctl

```bash
sudo journalctl -u apache2 -n 50
sudo journalctl -u apache2 -f                          # tiempo real
sudo journalctl --since "1 hour ago"
sudo journalctl --since "2026-01-27" --until "2026-01-28"
sudo journalctl -p err                                  # por prioridad
```

Servicios comunes a revisar: `ssh`, `apache2`, `nginx`, `mysql`, `postgresql`, `ufw`.

---

## Tareas Programadas (cron)

```bash
crontab -e / crontab -l / crontab -r
sudo crontab -u username -e
```

```
MIN HOUR DAY MONTH WEEKDAY COMMAND

*/5 * * * *   /path/to/script.sh    # cada 5 minutos
30 2 * * *    /path/to/backup.sh    # todos los días 2:30 AM
0 9 * * 1     /path/to/script.sh    # lunes 9:00 AM
0 0 1 * *     /path/to/script.sh    # día 1 de cada mes
0 9-17 * * *  /path/to/script.sh    # cada hora de 9 a 17
0 9,17 * * *  /path/to/script.sh    # a las 9 y a las 17
```

Caracteres: `*` cualquier valor · `,` lista · `-` rango · `/` incremento.

**Cron del sistema:** `/etc/crontab`, `/etc/cron.d/`, `/etc/cron.{hourly,daily,weekly,monthly}/`. **Logs:** `sudo grep CRON /var/log/syslog` o `sudo journalctl -u cron`.

### at (tarea única)

```bash
echo "/path/to/script.sh" | at now + 1 hour
atq                  # ver pendientes
atrm JOB_NUMBER       # eliminar
```

---

# Bash Scripting para Pentesters

## Estructura básica

```bash
#!/bin/bash
echo "¡Hola Mundo!"
```

```bash
chmod +x script.sh && ./script.sh    # o: bash script.sh
```

## Variables

```bash
nombre="John"
ip="192.168.1.100"
echo "IP: ${ip}"

echo "$USER $HOME $PATH $SHELL"      # variables de entorno
echo "$0 $1 $2 $@ $# $$ $?"          # $0 script, $1/$2 args, $@ todos, $# nº args, $$ PID, $? exit code del último comando
```

## Input del usuario

```bash
read -p "Ingresa tu edad: " edad
read -sp "Contraseña: " password; echo    # -s no muestra el texto
read -t 5 -p "Responde en 5s: " respuesta  # con timeout
```

## Condicionales

```bash
if [ -f archivo.txt ]; then
    echo "existe"
else
    echo "no existe"
fi
```

```
|Comparación numérica||Comparación de strings||
|---|---|---|---|
|`-eq`|igual|`=`|igual|
|`-ne`|distinto|`!=`|distinto|
|`-gt` / `-lt`|mayor / menor|`-z`|vacío|
|`-ge` / `-le`|mayor-igual / menor-igual|`-n`|no vacío|
```

**Test de archivos:** `-e` existe · `-f` es archivo · `-d` es directorio · `-r`/`-w`/`-x` legible/escribible/ejecutable · `-s` no vacío · `-L` es symlink.

```bash
if [ $edad -ge 18 ] && [ $edad -le 65 ]; then echo "adulto en edad laboral"; fi
if [ ! -f archivo.txt ]; then echo "no existe"; fi

# [[ ]] moderno, admite patrones y regex
if [[ $nombre == J* ]]; then echo "empieza con J"; fi
if [[ $email =~ @.*\.com$ ]]; then echo "email válido"; fi
```

### case

```bash
case $1 in
    start)   echo "Iniciando..." ;;
    stop)    echo "Deteniendo..." ;;
    restart) echo "Reiniciando..." ;;
    *)       echo "Uso: $0 {start|stop|restart}"; exit 1 ;;
esac
```

## Loops

```bash
for i in {1..10}; do echo "$i"; done
for i in {0..20..2}; do echo "$i"; done          # incremento de 2
for ((i=1; i<=10; i++)); do echo "$i"; done       # estilo C
for archivo in *.txt; do echo "$archivo"; done
for ip in 192.168.1.{1..254}; do
    ping -c 1 -W 1 $ip > /dev/null 2>&1 && echo "$ip activo"
done

contador=1
while [ $contador -le 5 ]; do echo "$contador"; ((contador++)); done
while read linea; do echo "$linea"; done < archivo.txt
while ! nc -z localhost 80; do echo "esperando..."; sleep 2; done   # esperar a que un servicio arranque
```

## Funciones

```bash
escanear_puerto() {
    local ip=$1
    local puerto=$2
    if nc -z -w1 $ip $puerto 2>/dev/null; then
        echo "Puerto $puerto ABIERTO en $ip"
    else
        echo "Puerto $puerto CERRADO en $ip"
    fi
}
escanear_puerto 192.168.1.1 80
```

---

## Scripts de Referencia para Pentesting

### Ping sweep

```bash
#!/bin/bash
# ping_sweep.sh <red>   → ej: ./ping_sweep.sh 192.168.1

[ $# -ne 1 ] && { echo "Uso: $0 <red>"; exit 1; }
red=$1

echo "[*] Escaneando $red.0/24..."
for host in {1..254}; do
    ip="$red.$host"
    ping -c 1 -W 1 $ip > /dev/null 2>&1 && echo "[+] $ip ACTIVO"
done
```

### Escáner de puertos comunes

```bash
#!/bin/bash
# port_scanner.sh <IP>

[ $# -ne 1 ] && { echo "Uso: $0 <IP>"; exit 1; }
target=$1
puertos_comunes=(21 22 23 25 53 80 110 143 443 445 3306 3389 8080)

echo "[*] Escaneando $target..."
for puerto in "${puertos_comunes[@]}"; do
    timeout 1 bash -c "echo >/dev/tcp/$target/$puerto" 2>/dev/null
    [ $? -eq 0 ] && echo "[+] Puerto $puerto ABIERTO"
done
```

### Automatizar escaneo Nmap en fases

```bash
#!/bin/bash
# auto_nmap.sh <target>

[ $# -ne 1 ] && { echo "Uso: $0 <target>"; exit 1; }
target=$1
output_dir="scan_$(date +%Y%m%d_%H%M%S)"
mkdir -p $output_dir

nmap -sn $target -oN $output_dir/hosts.txt > /dev/null 2>&1
nmap -sS -T4 $target -oN $output_dir/scan_fast.txt > /dev/null 2>&1
nmap -sS -p- $target -oN $output_dir/scan_full.txt > /dev/null 2>&1
nmap -sV $target -oN $output_dir/versions.txt > /dev/null 2>&1
nmap --script vuln $target -oN $output_dir/vulns.txt > /dev/null 2>&1

echo "[*] Resultados en $output_dir/"
```

### Extraer URLs de un archivo

```bash
#!/bin/bash
# extract_urls.sh <archivo>

[ $# -ne 1 ] && { echo "Uso: $0 <archivo>"; exit 1; }
[ ! -f "$1" ] && { echo "Error: archivo no existe"; exit 1; }

grep -Eo 'https?://[^[:space:]]+' "$1" | sort -u
```

### Monitor de logs (autenticación)

```bash
#!/bin/bash
# log_monitor.sh — vigila /var/log/auth.log por patrones de interés

logfile="/var/log/auth.log"
keywords=("Failed password" "Accepted password" "Invalid user")

tail -f $logfile | while read linea; do
    for keyword in "${keywords[@]}"; do
        if echo "$linea" | grep -q "$keyword"; then
            echo "[$(date '+%Y-%m-%d %H:%M:%S')] $linea"
        fi
    done
done
```