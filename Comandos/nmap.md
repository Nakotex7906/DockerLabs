# Comandos de nmap

Guía de referencia rápida con los comandos de `nmap` más usados en máquinas de práctica tipo DockerLabs. Cada sección explica qué hace cada flag, cuándo conviene usarla y trae el comando listo para copiar (reemplaza `<IP>` por la IP objetivo, por ejemplo `172.17.0.2`).

---

## 1. Escaneo inicial rápido (primer contacto)

Cuando entras a una máquina nueva, este suele ser el primer comando: barre todos los puertos TCP rápido para saber por dónde seguir.

- `-p-` → escanea **todos** los 65535 puertos TCP (por defecto nmap solo revisa los ~1000 más comunes).
- `-T4` → plantilla de temporización agresiva (más rápido, ideal en labs locales donde no hay riesgo de tumbar la red).
- `-oN` → guarda la salida en formato "normal" (texto legible) en un archivo.
- `-v` → modo verboso, muestra progreso en tiempo real.

```bash
nmap -p- -T4 -v -oN todos_los_puertos.txt <IP>
```

---

## 2. Escaneo de reconocimiento profundo (segunda pasada)

Una vez que sabes qué puertos están abiertos (paso 1), corres esto **solo sobre esos puertos** para sacar todo el detalle: versiones, servicios y vulnerabilidades básicas.

- `-sС` → corre los scripts NSE por defecto (equivalente a `--script=default`), útil para enumeración básica (banners, títulos HTTP, etc).
- `-sV` → detección de versión de los servicios (ej. saber si es Apache 2.4.41, OpenSSH 8.2, etc).
- `-O` → intenta detectar el sistema operativo del objetivo (requiere permisos root/sudo).
- `-p` → especifica los puertos exactos a escanear (los que ya detectaste antes).

```bash
nmap -sC -sV -O -p 22,80,139,445 -oN escaneo_detallado.txt <IP>
```

---

## 3. Escaneo agresivo "todo en uno"

Combina detección de SO, versiones, scripts default y traceroute en un solo comando. Es cómodo pero más ruidoso y lento que hacerlo en dos pasos (secciones 1 y 2).

- `-A` → activa a la vez `-O` (SO), `-sV` (versiones), `-sC` (scripts default) y traceroute.
- `-Pn` → **no** hace ping antes de escanear; trata al host como si estuviera activo. Muy útil en DockerLabs porque muchos contenedores bloquean el ICMP y nmap los marcaría como "down" sin este flag.

```bash
nmap -A -Pn -p- -T4 <IP>
```

---

## 4. Tipos de escaneo de puertos

- `-sS` → **SYN scan** ("escaneo sigiloso" o *half-open*). Envía un SYN y nunca completa el handshake TCP. Es el escaneo por defecto cuando corres nmap con privilegios de root, y el más usado porque es rápido y deja menos rastro en logs.

  ```bash
  sudo nmap -sS <IP>
  ```

- `-sT` → **Connect scan**. Completa el handshake TCP completo (usa la llamada `connect()` del sistema). Se usa cuando no tienes privilegios root/sudo, porque `-sS` los requiere.

  ```bash
  nmap -sT <IP>
  ```

- `-sU` → **UDP scan**. Escanea puertos UDP (DNS, SNMP, DHCP, TFTP, etc). Es mucho más lento que el TCP, por eso normalmente se limita el rango de puertos.

  ```bash
  sudo nmap -sU --top-ports 20 <IP>
  ```

- `-sn` → **Ping scan** (antes `-sP`). Solo descubre si el host está vivo, sin escanear puertos. Sirve para barrer una subred completa y ver qué IPs responden.

  ```bash
  nmap -sn 172.17.0.0/24
  ```

---

## 5. Especificación de puertos

- `-p <lista>` → puertos específicos, separados por coma o en rango.

  ```bash
  nmap -p 21,22,80,443 <IP>
  ```

- `-p-` → todos los puertos (1-65535).

  ```bash
  nmap -p- <IP>
  ```

- `-p 1-1000` → rango explícito de puertos.

  ```bash
  nmap -p 1-1000 <IP>
  ```

- `--top-ports <n>` → escanea los N puertos más comunes según la base de datos de nmap (útil para un vistazo rápido en UDP).

  ```bash
  nmap --top-ports 100 <IP>
  ```

- `-F` → modo rápido, escanea solo los 100 puertos más comunes (equivalente a `--top-ports 100` pero más corto de escribir).

  ```bash
  nmap -F <IP>
  ```

---

## 6. Scripts NSE (Nmap Scripting Engine)

Los scripts NSE son la parte más potente de nmap para CTF: automatizan enumeración y hasta detección de vulnerabilidades conocidas.

- `--script=default` (o `-sC`) → set de scripts seguros y básicos, se corren automáticamente con `-A`.
- `--script=vuln` → corre todos los scripts que buscan vulnerabilidades conocidas (CVEs) en los servicios detectados. Muy útil como paso inicial para ver si hay "fruta fácil".

  ```bash
  nmap -sV --script=vuln -p <puertos> <IP>
  ```

- `--script=<nombre>` → corre un script puntual. Ejemplos típicos en DockerLabs:
  - Enumerar recursos SMB compartidos:

    ```bash
    nmap --script smb-enum-shares -p 445 <IP>
    ```

  - Enumerar directorios/archivos HTTP comunes:

    ```bash
    nmap --script http-enum -p 80 <IP>
    ```

  - Sacar usuarios vía SMB (si el servicio lo permite):

    ```bash
    nmap --script smb-enum-users -p 445 <IP>
    ```

- `--script-args=<args>` → pasa argumentos extra a un script (ej. credenciales, wordlists).

  ```bash
  nmap --script http-enum --script-args http-enum.basepath=/ -p 80 <IP>
  ```

---

## 7. Control de tiempo y rendimiento

- `-T<0-5>` → plantillas de velocidad, de más lenta/sigilosa a más rápida/ruidosa:
  - `-T0` paranoico (evasión extrema de IDS, extremadamente lento)
  - `-T1` sigiloso
  - `-T2` educado
  - `-T3` normal (por defecto)
  - `-T4` agresivo (el más usado en labs, buen balance velocidad/estabilidad)
  - `-T5` insano (máxima velocidad, puede perder resultados)

  ```bash
  nmap -T4 -p- <IP>
  ```

- `--min-rate <n>` → fuerza a nmap a mandar al menos N paquetes por segundo, para acelerar escaneos grandes (`-p-`) que de otra forma tardarían mucho.

  ```bash
  nmap -p- --min-rate 5000 <IP>
  ```

---

## 8. Formatos de salida

Guardar la salida es buena práctica en cualquier lab para no repetir escaneos.

- `-oN <archivo>` → salida en texto plano, legible directamente.
- `-oX <archivo>` → salida en XML (útil para parsear con otras herramientas).
- `-oG <archivo>` → salida "grepable" (formato antiguo, fácil de filtrar con `grep`/`awk`).
- `-oA <base>` → guarda los tres formatos anteriores a la vez, usando `<base>` como nombre común.

  ```bash
  nmap -sC -sV -p- -oA resultado_completo <IP>
  ```

---

## 9. Evasión y otras opciones útiles

- `-Pn` → no verifica con ping si el host está arriba, asume que sí lo está (clave cuando el objetivo bloquea ICMP, muy común en contenedores Docker).
- `-n` → no resuelve DNS inverso, acelera el escaneo cuando no te importa el hostname.
- `-f` → fragmenta los paquetes para intentar evadir firewalls/IDS básicos.
- `-D <decoy1,decoy2,ME>` → lanza el escaneo mezclado con IPs "señuelo" para dificultar rastrear el origen real.

  ```bash
  nmap -D RND:10 <IP>
  ```

- `--reason` → muestra por qué nmap concluyó que un puerto está abierto/cerrado/filtrado (útil para entender resultados raros).

---

## 10. Combo recomendado para DockerLabs

Flujo típico que funciona bien en la mayoría de las máquinas de la plataforma:

```bash
# Paso 1: descubrir todos los puertos abiertos, rápido
nmap -p- -T4 -Pn -v -oN paso1_puertos.txt <IP>

# Paso 2: usar esos puertos para un escaneo detallado con versiones y scripts
nmap -sC -sV -Pn -p <puertos_encontrados> -oN paso2_detalle.txt <IP>

# Paso 3 (opcional): buscar vulnerabilidades conocidas sobre esos servicios
nmap --script=vuln -Pn -p <puertos_encontrados> -oN paso3_vulns.txt <IP>
```

---

### Notas rápidas

- `sudo` es necesario para `-sS`, `-O` y algunos scripts que arman paquetes en crudo.
- En redes Docker (como las de DockerLabs), el flag `-Pn` casi siempre es imprescindible.
- Guarda siempre la salida (`-oN`/`-oA`): en un CTF vas a querer volver a revisar resultados anteriores.
