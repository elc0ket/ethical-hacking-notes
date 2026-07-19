## ¿Qué es una Computadora?

Máquina electrónica que procesa datos según instrucciones predefinidas (programas): realiza operaciones matemáticas, almacena información y se comunica con otros dispositivos.

### Componentes Principales

|Componente|Función|Notas|
|---|---|---|
|**CPU**|Ejecuta instrucciones y cálculos|Múltiples núcleos → procesamiento paralelo (Intel Core i5/i7/i9, AMD Ryzen 5/7/9)|
|**RAM**|Almacenamiento temporal de datos en uso|Se borra por completo al apagar el equipo; 8/16/32 GB típico|
|**Disco (HDD/SSD)**|Almacenamiento permanente|HDD: mecánico, más lento, más barato, más capacidad. SSD: chip flash, más rápido, más caro|
|**Motherboard**|Conecta e interconecta todos los componentes|CPU socket, RAM slots, GPU slot, SATA, USB, red, audio|
|**GPU**|Procesa gráficos e imágenes|Integrada (básica, en el CPU) vs Dedicada (potente, tarjeta separada)|
|**PSU**|Convierte AC → DC para alimentar el equipo|Potencia típica: 500-750W|

### Hardware vs Software

|Hardware|Software|
|---|---|
|Componentes físicos (CPU, RAM, disco, teclado)|Programas y datos (Windows, Chrome, Word)|
|Se puede tocar|No se puede tocar|

### Sistema Operativo (OS)

Software principal que gestiona el hardware y permite ejecutar programas. Capas: **Aplicaciones → Kernel (gestiona hardware) → Hardware**.

|SO|Notas|
|---|---|
|**Windows** (Microsoft)|Más popular en escritorio, interfaz gráfica amigable (Win 10/11)|
|**Linux** (Open Source)|Servidores y ciberseguridad, múltiples distros (Ubuntu, Kali, Debian), gratuito|
|**macOS** (Apple)|Exclusivo de Mac, basado en Unix|

---

## LAB 0.1: Identificar Componentes de tu PC

### Windows

```cmd
:: Información del sistema (GUI)
Windows + R → msinfo32 → Enter

:: Vía CMD
systeminfo
wmic cpu get name
wmic memorychip get capacity

:: Administrador de tareas
Ctrl + Shift + Esc → Rendimiento
```

### Linux

```bash
uname -a
lscpu
free -h
df -h
sudo lshw -short
```

---

## Estructura de Carpetas en Windows

```
C:
├── Windows              ← Sistema operativo (NO tocar)
├── Program Files        ← Programas instalados (64 bits)
├── Program Files (x86)  ← Programas instalados (32 bits)
├── Users                ← Archivos de usuarios
│   ├── TuNombre
│   │   ├── Desktop
│   │   ├── Documents
│   │   ├── Downloads
│   │   ├── Pictures
│   │   └── Videos
│   └── Public
└── Temp                 ← Archivos temporales
```

**Unidades comunes:** `C:` disco principal (sistema) · `D:` segunda partición/disco · `E:`, `F:` USB/DVD/discos externos.

---

## CMD - Símbolo del Sistema

### Abrir CMD

```
Windows + R → cmd → Enter          (normal)
Windows + X → Símbolo del sistema (Administrador)   (como admin)
```

### Navegación y archivos

|Comando|Uso|
|---|---|
|`dir`|Listar archivos y carpetas|
|`dir /a`|Incluir archivos ocultos|
|`dir /s`|Recursivo (subcarpetas)|
|`dir *.txt`|Filtrar por extensión|
|`dir /o:n` / `dir /o:d`|Ordenar por nombre / fecha|
|`cd`|Mostrar directorio actual|
|`cd Desktop` / `cd ..` / `cd ..\..`|Navegar|
|`cd C:\` / `cd %userprofile%`|Ir a ruta absoluta / home del usuario|
|`mkdir MiCarpeta`|Crear directorio|
|`mkdir Carpeta1\Subcarpeta1\Subcarpeta2`|Crear estructura anidada|
|`rmdir MiCarpeta`|Eliminar directorio vacío|
|`rmdir /s /q MiCarpeta`|Eliminar directorio con contenido, sin confirmación|
|`copy origen destino`|Copiar archivo|
|`move origen destino`|Mover archivo|
|`del archivo.txt` / `del /q` / `del /f`|Eliminar (silencioso / forzado)|
|`type archivo.txt`|Ver contenido|
|`type archivo.txt \| more`|Ver contenido paginado|
|`echo texto > archivo.txt`|Crear archivo con contenido (sobrescribe)|
|`echo texto >> archivo.txt`|Añadir línea (append)|
|`cls`|Limpiar pantalla|

### Red

|Comando|Uso|
|---|---|
|`ipconfig` / `ipconfig /all`|Info de red / detallada|
|`ipconfig /renew`|Renovar IP (DHCP)|
|`ipconfig /displaydns` / `/flushdns`|Ver / limpiar caché DNS|
|`ping google.com`|Probar conectividad|
|`ping -t google.com`|Ping continuo|
|`ping -n 10 google.com`|Ping con N paquetes|
|`tracert google.com`|Rastrear ruta (saltos/routers)|
|`netstat -a`|Conexiones activas|
|`netstat -n`|Sin resolución de nombres|
|`netstat -b`|Con el ejecutable asociado (requiere admin)|
|`nslookup google.com`|Consultar DNS|
|`nslookup google.com 8.8.8.8`|Consultar contra un servidor DNS específico|

### Sistema

|Comando|Uso|
|---|---|
|`systeminfo`|SO, versión, fabricante, CPU, RAM, fecha de instalación|
|`tasklist`|Procesos en ejecución|
|`tasklist /fi "username eq Usuario"`|Filtrar por usuario|
|`taskkill /im proceso.exe` / `/pid 1234`|Terminar proceso por nombre / PID|
|`taskkill /f /im chrome.exe`|Forzar terminación|
|`whoami` / `whoami /groups` / `whoami /priv`|Usuario actual, grupos, privilegios|
|`hostname`|Nombre del equipo|
|`net user`|Listar usuarios locales|

---

## PowerShell

Shell más potente que CMD, orientada a objetos.

**Abrir:** `Windows + X` → Windows PowerShell

|CMD|PowerShell equivalente|
|---|---|
|`dir`|`Get-ChildItem` / `ls`|
|`cd`|`Set-Location` / `cd`|
|`copy`|`Copy-Item`|
|`del`|`Remove-Item`|
|`type`|`Get-Content`|

```powershell
Get-ChildItem
Get-Content archivo.txt
Get-Process
Get-Service
Get-NetIPAddress
Invoke-WebRequest -Uri https://ejemplo.com/archivo.zip -OutFile archivo.zip
```

---

## Labs de Práctica

### Ejercicio 1: Navegar por el sistema

```cmd
cd C:\
dir
cd Users
dir
cd %username%
cd Desktop
dir
cd ..
```

### Ejercicio 2: Crear estructura de archivos

```cmd
mkdir C:\Lab_CMD
cd C:\Lab_CMD
mkdir Documentos
mkdir Imagenes
mkdir Backup
echo Este es un archivo de prueba > test.txt
type test.txt
copy test.txt Backup\test_backup.txt
dir /s
```

### Ejercicio 3: Comandos de red

```cmd
ipconfig
ping google.com
tracert google.com
netstat -an
nslookup whoami-labs.com
```

### Ejercicio 4: Información del sistema

```cmd
systeminfo
tasklist
whoami
hostname
net user
```
