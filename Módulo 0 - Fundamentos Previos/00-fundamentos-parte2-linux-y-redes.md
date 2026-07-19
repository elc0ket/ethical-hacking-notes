## ¿Qué es Linux?

Sistema operativo de código abierto basado en Unix (Linus Torvalds, 1991). El más usado en servidores, supercomputadoras y Android.

### Distribuciones

|Distribución|Uso principal|
|---|---|
|**Ubuntu**|Principiantes, Desktop|
|**Debian**|Servidores, base de Ubuntu|
|**Kali Linux**|Hacking ético / pentesting|
|**Parrot OS**|Pentesting, más ligero que Kali|
|**CentOS/Rocky**|Servidores corporativos|
|**Arch Linux**|Usuarios avanzados, minimalista|
|**Linux Mint**|Desktop alternativa, amigable|

---

## Estructura de Directorios

```
/                           ← Raíz del sistema
├── bin/                    ← Binarios esenciales (ls, cd, cat)
├── boot/                   ← Archivos de arranque (kernel)
├── dev/                    ← Dispositivos (discos, USB)
├── etc/                    ← Archivos de configuración
├── home/                   ← Carpetas de usuarios
├── root/                   ← Carpeta del usuario root
├── tmp/                    ← Archivos temporales
├── usr/                    ← Programas de usuario (bin/, lib/, local/)
├── var/                    ← Datos variables (log/, www/, tmp/)
├── opt/                    ← Software opcional
├── proc/                   ← Información de procesos
└── sys/                    ← Información del sistema
```

|Directorio|Descripción|
|---|---|
|`/home/usuario`|Archivos personales del usuario|
|`/etc`|Configuraciones del sistema|
|`/var/log`|Logs y registros|
|`/tmp`|Archivos temporales (se borran al reiniciar)|
|`/usr/bin`|Comandos y programas|
|`/root`|Carpeta del superusuario|

---

## Terminal

**Shells comunes:** bash (más común), zsh, sh, fish.

**Prompt:**

```
usuario@hostname:~/carpeta$
```

`usuario` · `hostname` · `~/carpeta` (directorio actual, `~` = home) · `$` usuario normal / `#` root.

---

## Comandos - Navegación

|Comando|Uso|
|---|---|
|`pwd`|Directorio actual|
|`ls` / `ls -l` / `ls -a` / `ls -la`|Listar / detallado / ocultos / ambos|
|`ls -lS` / `ls -lt` / `ls -lh` / `ls -R`|Por tamaño / fecha / formato humano / recursivo|
|`cd carpeta` / `cd ~` / `cd /`|Navegar / home / raíz|
|`cd ..` / `cd ../..` / `cd -`|Subir nivel(es) / directorio anterior|
|`tree` / `tree -L 2` / `tree -d`|Árbol de directorios / N niveles / solo carpetas|

**Lectura de `ls -la`:**

```
drwxr-xr-x  2 usuario grupo 4096 ene 27 22:00 Documents
-rw-r--r--  1 usuario grupo  157 ene 27 22:00 archivo.txt
```

`d`=directorio, `-`=archivo · `rwx`=permisos · propietario/grupo · tamaño (bytes) · fecha de modificación.

---

## Comandos - Archivos

|Comando|Uso|
|---|---|
|`cat archivo.txt`|Ver contenido|
|`cat > nuevo.txt` (Ctrl+D para guardar)|Crear archivo con contenido|
|`less archivo.txt`|Paginador (`/palabra` buscar, `q` salir)|
|`head -n 5 archivo.txt` / `tail -n 20 archivo.txt`|Primeras / últimas N líneas|
|`tail -f /var/log/syslog`|Ver log en tiempo real|
|`touch archivo.txt`|Crear archivo vacío / actualizar fecha|
|`mkdir -p Carpeta1/Carpeta2/Carpeta3`|Crear estructura anidada|
|`cp -r origen/ destino/`|Copiar carpeta completa|
|`cp -v` / `cp -u`|Verbose / solo si destino más viejo|
|`mv origen destino`|Mover o renombrar|
|`rm -rf Carpeta/`|Eliminar carpeta y contenido, forzado|
|`grep -rni "palabra" ruta/`|Buscar recursivo, sin mayúsculas, con nº de línea|
|`grep -c` / `grep -v`|Contar coincidencias / invertir búsqueda|
|`find /home -iname "*.txt"`|Buscar por nombre (sin distinguir mayúsculas)|
|`find /home -mtime -7`|Modificados hace menos de 7 días|
|`find /home -size +100M`|Mayores a 100 MB|
|`find /home -name "*.log" -exec rm {} \;`|Buscar y ejecutar comando|

⚠️ **`rm -rf /` nunca se ejecuta** — borra el sistema completo.

---

## Comandos - Sistema

|Comando|Uso|
|---|---|
|`uname -a` / `-r` / `-m`|Info completa / versión kernel / arquitectura|
|`whoami` / `id`|Usuario actual / UID, GID y grupos|
|`df -h` / `df -i`|Espacio en disco (formato humano) / inodos|
|`du -sh` / `du -h --max-depth=1`|Tamaño total de carpeta / por subcarpeta|
|`free -h`|Memoria RAM (formato humano)|
|`ps aux` / `ps auxf`|Todos los procesos / en árbol|
|`top` / `htop`|Monitor de procesos (interactivo)|
|`kill 1234` / `kill -9 1234`|Terminar proceso / forzar|
|`killall -9 nombre`|Forzar cierre por nombre|

**Señales comunes:** `SIGTERM (15)` terminación suave (default) · `SIGKILL (9)` forzada · `SIGHUP (1)` recargar config.

---

## Permisos

`u` (usuario/propietario), `g` (grupo), `o` (otros). `r`=4, `w`=2, `x`=1.

```
-rwxr-xr-x  1 usuario grupo 8192 ene 27 22:00 script.sh
  │   │   │
  │   │   └── Otros: r-x
  │   └────── Grupo: r-x
  └────────── Usuario: rwx
```

Tipos de archivo: `-` regular · `d` directorio · `l` enlace simbólico · `b`/`c` dispositivo de bloques/caracteres.

### chmod

```bash
chmod 755 archivo.sh      # rwx / r-x / r-x
chmod 600 archivo.txt     # solo usuario lee/escribe
chmod +x script.sh        # hacer ejecutable
chmod -R 755 MiCarpeta/   # recursivo

# Modo simbólico
chmod u+x archivo.sh
chmod g-w archivo.txt
chmod o+r archivo.txt
chmod a+rwx archivo.txt
```

### chown / chgrp

```bash
sudo chown usuario:grupo archivo.txt
sudo chown -R usuario:grupo /var/www/
sudo chgrp -R grupo /var/www/
```

---

## Comandos Avanzados

|Comando|Uso|
|---|---|
|`sudo comando` / `sudo su` / `sudo -i`|Ejecutar como root / convertirse en root|
|`sudo -u usuario comando`|Ejecutar como otro usuario|
|`apt update && apt upgrade`|Actualizar índices y paquetes|
|`apt install/remove/purge paquete`|Instalar / quitar / quitar con configs|
|`wget -O nombre.zip URL` / `wget -c URL`|Descargar con nombre / reanudar|
|`curl -O URL` / `curl -I URL` / `curl -L URL`|Descargar / ver headers / seguir redirecciones|
|`tar -czvf archivo.tar.gz carpeta/`|Comprimir (gzip)|
|`tar -xzvf archivo.tar.gz -C /destino/`|Extraer en carpeta destino|
|`tar -tzvf archivo.tar.gz`|Ver contenido sin extraer|
|`zip -r carpeta.zip carpeta/` / `unzip archivo.zip -d /destino/`|Comprimir / extraer ZIP|
|`alias ll='ls -la'` → `echo "alias ll='ls -la'" >> ~/.bashrc`|Atajo temporal → permanente|
|`history` / `!123` / `!!` / `Ctrl+R`|Historial / ejecutar por nº / último / buscar|
|`man comando`|Manual|
|`echo $HOME` / `$USER` / `$PATH`|Variables de entorno|

---

# Redes e Internet Básico

## Componentes de Red

|Componente|Función|
|---|---|
|**ISP**|Proveedor de acceso a Internet|
|**Módem**|Convierte señal digital ↔ analógica, conecta con el ISP|
|**Router**|Dirige tráfico entre dispositivos, asigna IPs (DHCP), firewall básico|
|**Switch**|Conecta dispositivos por cable (capa 2)|
|**Access Point**|Punto de acceso WiFi (a menudo integrado en el router)|

Red doméstica típica: `Internet → Módem → Router (ej. 192.168.1.1) → dispositivos (192.168.1.x)`.

---

## Direcciones IP

### IPv4

Formato `192.168.1.100` (4 octetos, 0-255 cada uno).

|Clase|Rango|Uso|
|---|---|---|
|A|1.0.0.0 – 126.255.255.255|Redes grandes|
|B|128.0.0.0 – 191.255.255.255|Redes medianas|
|C|192.0.0.0 – 223.255.255.255|Redes pequeñas|

**Rangos privados (no rutables en Internet):**

|Rango|Descripción|
|---|---|
|`10.0.0.0 – 10.255.255.255`|Clase A privada|
|`172.16.0.0 – 172.31.255.255`|Clase B privada|
|`192.168.0.0 – 192.168.255.255`|Clase C privada (la más común)|
|`127.0.0.1`|Localhost|

### IPv6

Formato: 8 grupos hex, ej. `2001:0db8:0000:0000:0000:0000:0000:0001` → simplificado `2001:db8::1`.

---

## Puertos

Rango 0–65535.

|Rango|Tipo|
|---|---|
|0–1023|Bien conocidos (servicios estándar)|
|1024–49151|Registrados|
|49152–65535|Dinámicos/temporales|

**Puertos comunes** (referencia útil para reconocimiento/pentesting):

|Puerto|Servicio|
|---|---|
|20/21|FTP|
|22|SSH|
|23|Telnet (inseguro, obsoleto)|
|25|SMTP|
|53|DNS|
|80|HTTP|
|110|POP3|
|143|IMAP|
|443|HTTPS|
|3306|MySQL|
|3389|RDP|
|5432|PostgreSQL|
|8080|HTTP alternativo|

---

## Modelo TCP/IP

```
4. Aplicación      ← HTTP, FTP, DNS, SMTP
5. Transporte      ← TCP, UDP
6. Internet        ← IP, ICMP, ARP
7. Acceso a Red    ← Ethernet, WiFi
```

```
||TCP|UDP|
|---|---|---|
|Conexión|Orientado a conexión|Sin conexión|
|Confiabilidad|Reenvía paquetes perdidos|No confiable|
|Velocidad|Más lento|Más rápido|
|Orden|Garantizado|No garantizado|
|Uso típico|HTTP, FTP, SSH|DNS, streaming, VoIP|
```

---

## DNS

Traduce nombres de dominio a IPs. Jerarquía: `. (raíz) → .com/.net/.org → dominio → subdominio`.

|Registro|Descripción|Ejemplo|
|---|---|---|
|A|IPv4|google.com → 142.250.65.206|
|AAAA|IPv6|google.com → 2001:db8::1|
|MX|Servidor de correo|gmail.com → smtp.gmail.com|
|CNAME|Alias|www → ejemplo.com|
|TXT|Texto arbitrario|Verificación, SPF|
|NS|Servidor de nombres|Delegación DNS|

**DNS públicos:** Google `8.8.8.8` / `8.8.4.4` · Cloudflare `1.1.1.1` / `1.0.0.1` · OpenDNS `208.67.222.222` / `208.67.220.220`.

---

## HTTP vs HTTPS

|HTTP|HTTPS|
|---|---|
|Puerto 80|Puerto 443|
|Sin cifrar|Cifrado (SSL/TLS)|

Certificado SSL/TLS: emitido por una CA, contiene dominio, clave pública y fecha de expiración.

---

## Comandos de Diagnóstico de Red

|Comando (Linux)|Equivalente Windows|Uso|
|---|---|---|
|`ip addr` / `ifconfig`|`ipconfig /all`|Ver configuración de red local|
|`ping -c 4 google.com`|`ping -n 4 google.com`|Probar conectividad (N paquetes)|
|`traceroute google.com`|`tracert google.com`|Ver ruta/saltos hasta el destino|
|`nslookup google.com` / `dig google.com`|`nslookup google.com`|Consultar DNS|
|`nslookup google.com 8.8.8.8`|—|Consultar contra DNS específico|
|`ss -tuln` / `netstat -tuln`|`netstat -an`|Ver puertos abiertos y conexiones activas|

**Lectura de ping:** `time=15ms` → latencia (menor es mejor) · `TTL=117` → saltos restantes.