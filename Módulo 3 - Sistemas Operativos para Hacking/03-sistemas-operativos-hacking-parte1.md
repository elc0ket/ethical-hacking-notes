# Linux Avanzado

## ¿Por qué Linux para Hacking?

Kali Linux, Parrot OS y BlackArch vienen con cientos de herramientas preinstaladas; control total del sistema, scripting potente, comunidad de seguridad grande, open source y gratuito. Además, la gran mayoría de servidores en Internet corren Linux, por lo que dominarlo es imprescindible tanto para atacar como para moverse en un sistema comprometido.

---

## Navegación Avanzada

```bash
pwd                          # directorio actual

ls -la                       # todo, formato largo
ls -lh                       # tamaños legibles
ls -lt / ls -lS              # por fecha / por tamaño
ls -R                        # recursivo

cd /path/to/dir / cd .. / cd - / cd ~ / cd /

mkdir -p dir1/dir2/dir3       # crea padres si no existen

cp -r origen/ destino/        # recursivo
cp -v archivo destino         # verbose

mv archivo.txt nuevo_nombre.txt

rm -rf directorio/            # forzado, sin preguntar
rm -i archivo.txt             # interactivo

cat archivo.txt
head -n 20 archivo.txt
tail -f /var/log/syslog       # seguir cambios en tiempo real
less archivo.txt              # paginado (q para salir)
```

### Wildcards

|Comodín|Significado|Ejemplo|
|---|---|---|
|`*`|Cualquier cantidad de caracteres|`ls *.txt`|
|`?`|Un solo carácter|`ls file?.txt` → file1.txt, fileA.txt (no file10.txt)|
|`[]`|Rango/conjunto de caracteres|`ls file[1-3].txt` / `ls file[abc].txt`|
|`{}`|Lista de opciones|`ls file{1,5,9}.txt`|

---

## Búsqueda de Archivos

### find (el más potente)

```bash
find / -name "*.conf"              # por nombre, desde raíz
find /home -iname "*.TXT"          # case-insensitive

find / -type f -name "*.log"       # solo archivos
find / -type d -name "config"      # solo directorios
find / -type l                     # solo symlinks

find / -size +100M                 # mayor a 100MB
find / -size -1k                   # menor a 1KB

find / -perm 777                   # permisos exactos
find / -perm -u=s                  # con SUID bit
find / -perm -4000 -type f 2>/dev/null   # binarios SUID — clásico en privesc

find / -user root                  # propiedad de root
find / -group www-data

find / -mtime -7                   # modificado en los últimos 7 días
find / -mmin -60                   # modificado en los últimos 60 minutos

find / -type f -name "*.conf" -size +1M   # combinar condiciones

find / -name "*.tmp" -exec rm {} \;               # ejecutar comando en resultados
find / -type f -name "*.log" -exec grep "error" {} \;
```

### locate / which / whereis

```bash
sudo updatedb          # actualizar la base de locate
locate -i password
locate "*.conf"

which nmap             # ruta del ejecutable
whereis python3        # ejecutable + man + fuentes
```

---

## Manipulación de Texto

### grep

```bash
grep -i "error" archivo.txt          # case-insensitive
grep -n "error" archivo.txt          # con número de línea
grep -r "password" /etc/             # recursivo
grep -v "success" archivo.txt        # invertir (líneas SIN el patrón)
grep -c "error" archivo.txt          # contar ocurrencias
grep -A 3 / -B 3 / -C 3 "error" archivo.txt   # contexto después/antes/ambos
grep -l "password" *.txt             # solo nombres de archivo con match

grep "^Error" archivo.txt            # empieza con
grep "error$" archivo.txt            # termina con
```

### sed (stream editor)

```bash
sed 's/old/new/' archivo.txt              # primera ocurrencia por línea (no modifica el archivo)
sed 's/old/new/g' archivo.txt             # todas las ocurrencias
sed -i 's/old/new/g' archivo.txt          # modificar in-place
sed '1,5d' archivo.txt                    # eliminar líneas 1-5
sed '/pattern/d' archivo.txt              # eliminar líneas que matchean
sed -n '10,20p' archivo.txt               # imprimir solo líneas 10-20
```

### awk

```bash
awk '{print $1}' archivo.txt              # primera columna
awk -F':' '{print $1}' /etc/passwd        # con delimitador → lista de usuarios
awk '$3 > 100 {print $1}' archivo.txt     # condicional
awk '{sum+=$1} END {print sum}' archivo.txt   # sumar columna

# IPs más frecuentes de un log de acceso
awk '{print $1}' /var/log/apache2/access.log | sort | uniq -c | sort -rn
```

### cut / sort / uniq / wc

```bash
cut -d':' -f1 /etc/passwd        # campo 1 por delimitador
cut -c1-5 archivo.txt            # por posición de caracter

sort -n archivo.txt              # numérico
sort -u archivo.txt              # único
sort archivo.txt | uniq -c       # contar duplicados (requiere ordenar antes)
sort archivo.txt | uniq -d       # solo duplicados

wc -l archivo.txt                # contar líneas
```

---

## Pipes y Redirecciones

```bash
comando > archivo.txt        # stdout, sobrescribe
comando >> archivo.txt       # stdout, append
comando 2> error.txt         # stderr
comando > salida.txt 2> error.txt     # separados
comando &> todo.txt          # ambos al mismo archivo
comando &> /dev/null         # descartar toda salida
comando < archivo.txt        # entrada desde archivo

comando1 | comando2 | comando3        # encadenar

ps aux | grep "username" | wc -l                                    # contar procesos de un usuario
history | awk '{print $2}' | sort | uniq -c | sort -rn | head -10   # comandos más usados
cat access.log | awk '{print $1}' | sort | uniq -c | sort -rn | head -10   # IPs más frecuentes

comando | tee archivo.txt        # salida a archivo Y a pantalla
comando | tee -a archivo.txt     # append
```

---

## Comandos de Red

```bash
ip addr / ip a                          # IPs de interfaces
sudo ip addr add 192.168.1.100/24 dev eth0
sudo ip link set eth0 up / down

ip route                                # tabla de routing
sudo ip route add default via 192.168.1.1

cat /etc/resolv.conf                    # servidores DNS configurados
hostname / sudo hostname nuevo

ping -c 4 google.com
traceroute -n google.com                # sin resolución DNS

ss -tuln                                # puertos escuchando
ss -tunapl                               # conexiones + procesos
ip neigh                                # tabla ARP (equivalente moderno de arp -a)

wget -O nombre.zip https://ejemplo.com/archivo.zip
curl -L https://ejemplo.com             # seguir redirects

scp archivo.txt usuario@192.168.1.100:/path/
scp -r directorio/ usuario@192.168.1.100:/path/
```

---

## Gestión de Paquetes

### APT (Debian/Ubuntu/Kali)

```bash
sudo apt update && sudo apt full-upgrade -y
sudo apt install -y nmap
sudo apt purge nmap                # elimina también configuraciones
apt show nmap
apt list --installed / dpkg -l
sudo apt autoremove                # dependencias huérfanas
```

### YUM/DNF (Red Hat/CentOS/Fedora)

```bash
sudo yum update / sudo dnf update
sudo yum install nmap
yum list installed
```

---

# Sistema de Archivos Linux (FHS)

```
/
├── bin/     binarios esenciales (ls, cat, cp)
├── boot/    arranque (kernel, GRUB)
├── dev/     dispositivos
├── etc/     configuración del sistema
├── home/    directorios de usuarios
├── lib/     librerías compartidas
├── media/   medios removibles (USB)
├── mnt/     montaje temporal
├── opt/     software opcional/third-party
├── proc/    info de procesos (virtual)
├── root/    home de root
├── run/     datos en runtime (PIDs, sockets)
├── sbin/    binarios de administración
├── srv/     datos de servicios (web, FTP)
├── sys/     info de kernel y hardware (virtual)
├── tmp/     temporales
├── usr/     programas de usuario (bin/, lib/, local/, share/)
└── var/     datos variables (log/, www/, tmp/)
```

---

## Directorios Relevantes para Pentesting

### /etc/ — configuración

|Archivo|Contenido|
|---|---|
|`/etc/passwd`|Usuarios del sistema|
|`/etc/shadow`|Contraseñas hasheadas (solo root)|
|`/etc/group`|Grupos|
|`/etc/sudoers`|Configuración de sudo|
|`/etc/hosts`|Resolución de nombres local|
|`/etc/resolv.conf`|Servidores DNS|
|`/etc/ssh/`|Configuración SSH|
|`/etc/crontab`|Tareas programadas|
|`/etc/rc.local`|Scripts de inicio|

```bash
cat /etc/passwd
# formato: username:x:UID:GID:comment:home:shell

grep "bash$" /etc/passwd                          # usuarios con shell interactiva
awk -F':' '$3 >= 1000 {print $1}' /etc/passwd     # usuarios "humanos" (UID >= 1000)
```

### /var/log/ — logs

```bash
/var/log/syslog        # Debian/Ubuntu
/var/log/messages      # Red Hat/CentOS
/var/log/auth.log      # autenticación (SSH, sudo)
/var/log/apache2/  /var/log/nginx/  /var/log/mysql/
/var/log/kern.log       # kernel
/var/log/dmesg          # mensajes del kernel al boot

tail -f /var/log/auth.log
grep -i "failed" /var/log/auth.log
```

### /proc/ — información de procesos (virtual)

```bash
cat /proc/cpuinfo
cat /proc/meminfo
cat /proc/version
cat /proc/[PID]/cmdline    # comando que inició el proceso
cat /proc/[PID]/environ    # variables de entorno del proceso
```

### /tmp/ — temporales

Escribible por cualquier usuario — habitual para subir exploits o payloads.

```bash
ls -ld /tmp
# drwxrwxrwt  ← la 't' final es el sticky bit: cada usuario solo puede borrar sus propios archivos
```

### /home/ — usuarios

```
/home/usuario/.bashrc          /home/usuario/.bash_history
/home/usuario/.ssh/            /home/usuario/.ssh/id_rsa
/home/usuario/.mysql_history
```

---

## Checklist de Archivos Sensibles (referencia de pentesting)

```
# Credenciales / configuración
/etc/passwd  /etc/shadow  /etc/group  /etc/sudoers  /etc/ssh/sshd_config
/root/.ssh/id_rsa  /home/*/.ssh/id_rsa  /home/*/.bash_history

# Bases de datos
/var/lib/mysql/  /var/lib/postgresql/

# Web
/var/www/html/  /usr/share/nginx/html/
/var/www/html/wp-config.php   # WordPress
/var/www/html/config.php      # Drupal

# Backups
/var/backups/  /tmp/backup*  *.sql  *.bak  *.old

# Scripts y cron
/etc/crontab  /etc/cron.d/  /etc/cron.daily/  /etc/cron.hourly/
/var/spool/cron/crontabs/

# Binarios SUID (privesc)
find / -perm -4000 -type f 2>/dev/null
```

---

## Links

|Tipo|Comando|Comportamiento|
|---|---|---|
|**Hard link**|`ln archivo.txt hardlink.txt`|Apunta al inode; si se borra el original, el hard link sigue funcionando|
|**Symlink**|`ln -s archivo.txt symlink.txt`|Apunta al nombre; si se borra el original, el symlink queda roto (`lrwxrwxrwx ... symlink.txt -> archivo.txt`)|

---

## Montaje de Filesystems

```bash
mount              # ver sistemas montados
df -h               # uso de disco

sudo mount /dev/sdb1 /mnt/usb
sudo umount /mnt/usb
sudo mount -o loop imagen.iso /mnt/iso

cat /etc/fstab      # montaje automático al boot
```