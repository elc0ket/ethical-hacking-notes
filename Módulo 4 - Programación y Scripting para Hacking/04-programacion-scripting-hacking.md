# Python Fundamentals

## ¿Por qué Python para Hacking?

|Razón|Beneficio|
|---|---|
|Sintaxis simple y legible|Menos líneas de código, más rápido de escribir|
|Librerías potentes|socket, scapy, requests, pycryptodome, impacket|
|Multiplataforma|Linux, Windows, Mac sin cambios|
|Comunidad de seguridad enorme|Cientos de herramientas y ejemplos disponibles|
|Rapidez de desarrollo|Scripts propios en minutos|
|Usado en frameworks reales|Metasploit, Impacket, Empire, Burp Suite extensiones|

---

## Instalación y Setup

```bash
# Verificar Python instalado
python3 --version
python --version

# Instalar pip (gestor de paquetes)
sudo apt install python3-pip

# Actualizar pip
pip3 install --upgrade pip

# Instalar librerías comunes para hacking
pip3 install requests scapy pycryptodome python-nmap
```

---

# Sintaxis Básica

## Variables y Tipos de Datos

```python
# Variables (sin declaración de tipo)
nombre = "John"
edad = 25
altura = 1.75
es_hacker = True

# Tipos de datos
tipo_string = "Texto"
tipo_int = 42
tipo_float = 3.14
tipo_bool = True
tipo_none = None
```

### Imprimir y entrada

```python
# Imprimir
print("Hola Mundo")
print(f"Mi nombre es {nombre}")              # f-string (Python 3.6+)
print("Tengo {} años".format(edad))         # format()

# Input del usuario
nombre = input("¿Cuál es tu nombre? ")
edad = int(input("¿Cuántos años tienes? "))
```

---

## Operadores

|Operador|Ejemplo|Resultado|
|---|---|---|
|`+`|5 + 3|8|
|`-`|10 - 4|6|
|`*`|3 * 4|12|
|`/`|10 / 3|3.333...|
|`//`|10 // 3|3 (división entera)|
|`%`|10 % 3|1 (módulo)|
|`**`|2 ** 3|8 (potencia)|

|Comparación|Ejemplo|Resultado|
|---|---|---|
|`==`|5 == 5|True|
|`!=`|5 != 3|True|
|`>`|10 > 5|True|
|`<`|3 < 5|True|
|`>=`|5 >= 5|True|
|`<=`|3 <= 5|True|

|Lógico|Ejemplo|Resultado|
|---|---|---|
|`and`|True and False|False|
|`or`|True or False|True|
|`not`|not True|False|

|Pertenencia|Ejemplo|Resultado|
|---|---|---|
|`in`|2 in [1,2,3]|True|
|`not in`|5 not in [1,2,3]|True|

---

## Strings (Cadenas)

```python
texto = "Python"

# Operaciones básicas
concatenar = "Hola" + " " + "Mundo"         # "Hola Mundo"
repetir = "Ha" * 3                          # "HaHaHa"
longitud = len("Python")                    # 6
```

### Indexing y Slicing

```python
texto = "Python"

# Indexing (comienza en 0)
print(texto[0])      # P
print(texto[-1])     # n (último)

# Slicing
print(texto[0:3])    # Pyt
print(texto[2:])     # thon
print(texto[:4])     # Pyth
print(texto[-3:])    # hon
```

### Métodos de Strings

|Método|Ejemplo|Resultado|
|---|---|---|
|`upper()`|"python".upper()|"PYTHON"|
|`lower()`|"PYTHON".lower()|"python"|
|`capitalize()`|"python".capitalize()|"Python"|
|`title()`|"python hacking".title()|"Python Hacking"|
|`replace()`|"python".replace("py", "Py")|"Python"|
|`split()`|"a b c".split()|['a', 'b', 'c']|
|`find()`|"python".find("on")|4|
|`startswith()`|"python".startswith("py")|True|
|`endswith()`|"python".endswith("on")|True|
|`strip()`|" texto ".strip()|"texto"|

### f-strings (formatted strings)

```python
nombre = "Alice"
edad = 25

print(f"Me llamo {nombre} y tengo {edad} años")
print(f"Edad en hexadecimal: {edad:x}")      # 19
print(f"Dos decimales: {3.14159:.2f}")       # 3.14
```

---

# Estructuras de Datos

## Listas

```python
# Declaración
lista = [1, 2, 3, 4, 5]
mixta = [1, "dos", 3.0, True]
vacia = []

# Acceso
print(lista[0])      # 1
print(lista[-1])     # 5

# Slicing
print(lista[1:4])    # [2, 3, 4]
```

### Métodos de Listas

|Método|Efecto|
|---|---|
|`append(x)`|Agregar al final|
|`insert(i, x)`|Insertar en índice i|
|`remove(x)`|Eliminar valor|
|`pop()`|Quitar último elemento|
|`extend(lista)`|Agregar múltiples elementos|
|`sort()`|Ordenar ascendente|
|`reverse()`|Invertir orden|
|`clear()`|Vaciar lista|
|`index(x)`|Encontrar índice de x|
|`count(x)`|Contar ocurrencias|

### List Comprehension

```python
cuadrados = [x**2 for x in range(1, 6)]      # [1, 4, 9, 16, 25]
pares = [x for x in range(10) if x % 2 == 0]  # [0, 2, 4, 6, 8]
```

### Iterar listas

```python
# Iteración simple
for item in lista:
    print(item)

# Con índice
for i, item in enumerate(lista):
    print(f"Índice {i}: {item}")
```

---

## Tuplas (Inmutables)

```python
# Similar a listas pero NO se pueden modificar
tupla = (1, 2, 3)
coordenadas = (10.5, 20.3)

# Acceso
print(tupla[0])     # 1

# Útil para múltiples returns
def get_coordinates():
    return 10, 20

x, y = get_coordinates()
```

---

## Diccionarios

```python
# Pares clave-valor
persona = {
    "nombre": "John",
    "edad": 25,
    "ciudad": "Bogotá"
}

# Acceso
print(persona["nombre"])              # John
print(persona.get("edad"))            # 25
print(persona.get("pais", "N/A"))     # N/A (default si no existe)

# Modificar y agregar
persona["edad"] = 26
persona["email"] = "john@example.com"

# Eliminar
del persona["ciudad"]
persona.pop("email")
```

### Métodos y operaciones de Diccionarios

```python
# Ver claves, valores, pares
print(persona.keys())       # dict_keys(['nombre', 'edad'])
print(persona.values())     # dict_values(['John', 26])
print(persona.items())      # dict_items([('nombre', 'John'), ('edad', 26)])

# Iterar
for clave, valor in persona.items():
    print(f"{clave}: {valor}")

# Verificar existencia
if "nombre" in persona:
    print("Existe")

# Dict comprehension
cuadrados = {x: x**2 for x in range(1, 6)}  # {1:1, 2:4, 3:9, 4:16, 5:25}
```

---

## Sets (Conjuntos)

```python
# Colección no ordenada de elementos únicos
conjunto = {1, 2, 3, 4, 5}
conjunto2 = {4, 5, 6, 7, 8}

# Elimina duplicados automáticamente
lista = [1, 2, 2, 3, 3, 3]
sin_duplicados = list(set(lista))  # [1, 2, 3]

# Operaciones de conjuntos
union = conjunto | conjunto2                # {1,2,3,4,5,6,7,8}
interseccion = conjunto & conjunto2         # {4, 5}
diferencia = conjunto - conjunto2           # {1, 2, 3}
diferencia_simetrica = conjunto ^ conjunto2  # {1,2,3,6,7,8}

# Métodos
conjunto.add(6)           # Agregar
conjunto.remove(1)        # Eliminar (error si no existe)
conjunto.discard(10)      # Eliminar (no error si no existe)
```

---

# Control de Flujo

## if / elif / else

```python
edad = 18

if edad < 18:
    print("Menor de edad")
elif edad == 18:
    print("Justo 18")
else:
    print("Mayor de edad")

# Operador ternario
mensaje = "Mayor" if edad >= 18 else "Menor"

# Múltiples condiciones
if edad >= 18 and edad < 65:
    print("Adulto en edad laboral")

# Verificar vacío
lista = []
if not lista:
    print("Lista vacía")

# Verificar None
valor = None
if valor is None:
    print("Es None")
```

---

## Loops - for

```python
# Iterar sobre lista
for i in [1, 2, 3, 4, 5]:
    print(i)

# Iterar sobre rango
for i in range(5):           # 0, 1, 2, 3, 4
    print(i)

for i in range(1, 6):        # 1, 2, 3, 4, 5
    print(i)

for i in range(0, 10, 2):    # 0, 2, 4, 6, 8 (incremento)
    print(i)

# Iterar sobre string
for letra in "Python":
    print(letra)

# Iterar sobre diccionario
persona = {"nombre": "John", "edad": 25}
for clave, valor in persona.items():
    print(f"{clave}: {valor}")

# break y continue
for i in range(10):
    if i == 5:
        break           # Salir del loop
    if i % 2 == 0:
        continue        # Saltar a siguiente iteración
    print(i)
```

---

## Loops - while

```python
# while básico
contador = 0
while contador < 5:
    print(contador)
    contador += 1

# while con break
while True:
    respuesta = input("¿Continuar? (s/n): ")
    if respuesta.lower() == 'n':
        break
    print("Continuando...")

# while con else
intentos = 0
max_intentos = 3
while intentos < max_intentos:
    password = input("Contraseña: ")
    if password == "secreto":
        print("Acceso concedido")
        break
    intentos += 1
else:
    print("Máximo de intentos alcanzado")
```

---

# Funciones

```python
# Función básica
def saludar():
    print("¡Hola!")

saludar()

# Función con parámetros
def saludar_nombre(nombre):
    print(f"¡Hola, {nombre}!")

saludar_nombre("Alice")

# Función con return
def sumar(a, b):
    return a + b

resultado = sumar(5, 3)  # 8

# Múltiples returns
def dividir(a, b):
    if b == 0:
        return None, "Error: división por cero"
    return a / b, None

resultado, error = dividir(10, 2)
```

### Parámetros avanzados

```python
# Parámetros por defecto
def saludar(nombre="Mundo"):
    print(f"¡Hola, {nombre}!")

saludar()          # ¡Hola, Mundo!
saludar("Alice")   # ¡Hola, Alice!

# *args (argumentos variables)
def sumar_todos(*numeros):
    return sum(numeros)

print(sumar_todos(1, 2, 3, 4, 5))  # 15

# **kwargs (keyword arguments)
def imprimir_info(**info):
    for clave, valor in info.items():
        print(f"{clave}: {valor}")

imprimir_info(nombre="John", edad=25, ciudad="Bogotá")

# Lambda (funciones anónimas)
cuadrado = lambda x: x**2
print(cuadrado(5))  # 25

# Lambda con map
numeros = [1, 2, 3, 4, 5]
cuadrados = list(map(lambda x: x**2, numeros))

# Lambda con filter
pares = list(filter(lambda x: x % 2 == 0, numeros))
```

---

# Módulos, Librerías y Archivos

## Módulos

```python
# Importar módulo completo
import os
print(os.getcwd())

# Importar con alias
import datetime as dt
print(dt.datetime.now())

# Importar función específica
from math import sqrt, pi
print(sqrt(16))  # 4.0
print(pi)        # 3.14159...
```

---

## Manejo de Archivos

```python
# Leer archivo
with open("archivo.txt", "r") as f:
    contenido = f.read()
    print(contenido)

# Leer línea por línea
with open("archivo.txt", "r") as f:
    for linea in f:
        print(linea.strip())

# Leer todas las líneas en lista
with open("archivo.txt", "r") as f:
    lineas = f.readlines()

# Escribir archivo (sobrescribe)
with open("archivo.txt", "w") as f:
    f.write("Primera línea\n")
    f.write("Segunda línea\n")

# Append (agregar al final)
with open("archivo.txt", "a") as f:
    f.write("Nueva línea\n")

# Verificar si existe
import os
if os.path.exists("archivo.txt"):
    print("Existe")

# Crear directorio
os.makedirs("nuevo_directorio", exist_ok=True)

# Listar archivos
archivos = os.listdir(".")
for archivo in archivos:
    print(archivo)
```

---

## Manejo de Excepciones

```python
# try/except básico
try:
    resultado = 10 / 0
except ZeroDivisionError:
    print("Error: división por cero")

# Múltiples excepciones
try:
    numero = int(input("Número: "))
    resultado = 10 / numero
except ValueError:
    print("Error: no es un número")
except ZeroDivisionError:
    print("Error: división por cero")

# Capturar cualquier excepción
try:
    pass
except Exception as e:
    print(f"Error: {e}")

# finally (siempre se ejecuta)
try:
    archivo = open("archivo.txt", "r")
    contenido = archivo.read()
except FileNotFoundError:
    print("Archivo no encontrado")
finally:
    archivo.close()

# else (si no hay excepción)
try:
    numero = int(input("Número: "))
except ValueError:
    print("Error")
else:
    print("Número válido")
```

---

# Script de Ejemplo: Port Scanner Básico

```python
#!/usr/bin/env python3
# port_scanner.py - Escáner de puertos simple

import socket
import sys

def scan_port(target, port):
    try:
        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        sock.settimeout(1)
        result = sock.connect_ex((target, port))

        if result == 0:
            return True
        else:
            return False
    except socket.error:
        return False
    finally:
        sock.close()

def main():
    if len(sys.argv) != 2:
        print("Uso: python3 port_scanner.py <target>")
        sys.exit(1)

    target = sys.argv[1]
    puertos_comunes = [21, 22, 23, 25, 53, 80, 110, 143, 443, 445, 3389, 8080]

    print(f"[*] Escaneando {target}...")
    print()

    for puerto in puertos_comunes:
        if scan_port(target, puerto):
            print(f"[+] Puerto {puerto} ABIERTO")

    print()
    print("[*] Escaneo completado")

if __name__ == "__main__":
    main()
```

**Uso:**

```bash
chmod +x port_scanner.py
python3 port_scanner.py 192.168.1.100
```