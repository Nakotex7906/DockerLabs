# Comandos Gobuster

Guía de referencia rápida con los modos y flags más usados de `gobuster`. Cada sección explica qué hace cada flag, cuándo conviene usarla y trae el comando listo para copiar (reemplaza `<IP>`, `<dominio>` y las rutas de wordlist según tu entorno; en Kali suelen estar en `/usr/share/wordlists/`).

---

## 1. Los 4 modos de gobuster

Gobuster no es un solo comando, es una herramienta con **subcomandos** (modos). Los que más vas a usar en DockerLabs son:

- `dir` → fuerza bruta de directorios/archivos en un sitio web.
- `dns` → fuerza bruta de subdominios.
- `vhost` → fuerza bruta de virtual hosts (nombres de dominio que responden distinto en el mismo servidor/IP).
- `fuzz` → fuzzing genérico usando el marcador `FUZZ` en cualquier parte de la URL, headers o body.

```bash
gobuster <modo> [flags]
```

---

## 2. Modo `dir` — enumeración de directorios y archivos

El más usado en la mayoría de los retos web de DockerLabs, para encontrar rutas ocultas (paneles de admin, backups, archivos de configuración, etc).

- `-u` → URL objetivo.
- `-w` → ruta a la wordlist a usar.
- `-x` → extensiones a probar además del nombre "seco" (ej. `.php,.txt,.bak`).
- `-t` → cantidad de threads (hilos) en paralelo, sube la velocidad.
- `-o` → guarda la salida en un archivo.

```bash
gobuster dir -u http://<IP> -w /usr/share/wordlists/dirb/common.txt -x php,txt,bak -t 50 -o gobuster_dir.txt
```

---

## 3. Modo `dns` — enumeración de subdominios

Útil cuando el reto trabaja con nombres de dominio (por ejemplo tienes que agregar entradas al `/etc/hosts`) y quieres descubrir subdominios ocultos.

- `-d` → dominio base a atacar.
- `-w` → wordlist de posibles subdominios.
- `-i` → muestra también las IPs resueltas de cada subdominio encontrado.

```bash
gobuster dns -d <dominio> -w /usr/share/wordlists/subdomains-top1million-5000.txt -i -o gobuster_dns.txt
```

---

## 4. Modo `vhost` — enumeración de virtual hosts

Cuando un mismo servidor/IP sirve contenido distinto según el header `Host:` (muy común en máquinas que "esconden" un segundo sitio en el mismo puerto 80/443).

- `-u` → URL base (normalmente la IP).
- `-w` → wordlist de posibles nombres de vhost.
- `--append-domain` → agrega automáticamente el dominio base a cada palabra de la wordlist antes de probarla.

```bash
gobuster vhost -u http://<dominio> -w /usr/share/wordlists/subdomains-top1million-5000.txt --append-domain -o gobuster_vhost.txt
```

---

## 5. Modo `fuzz` — fuzzing genérico

Para casos donde la parte a "adivinar" no es un directorio simple, sino un parámetro, un valor de header, o parte de la URL en cualquier posición. Se marca el punto exacto con la palabra `FUZZ`.

- `-u` → URL con el marcador `FUZZ` en la posición que quieres fuzzear.
- `-w` → wordlist a usar.

```bash
gobuster fuzz -u "http://<IP>/index.php?page=FUZZ" -w /usr/share/wordlists/dirb/common.txt
```

---

## 6. Flags generales (aplican a casi todos los modos)

- `-w <wordlist>` → wordlist a usar (obligatorio en todos los modos).
- `-t <n>` → número de threads en paralelo (por defecto 10; subirlo a 50-100 acelera bastante en labs locales).
- `-o <archivo>` → guarda los resultados en un archivo.
- `-q` → modo silencioso (quiet), no muestra el banner ni la configuración al inicio.
- `-v` → modo verboso, muestra más detalle de cada intento.
- `-z` → oculta la barra de progreso (útil si vas a redirigir la salida a un archivo o pipe).

---

## 7. Flags específicos del modo `dir` (los más usados en CTF)

- `-x <ext1,ext2>` → extensiones de archivo a agregar a cada palabra probada (ej. `.php,.html,.txt,.bak,.zip`).
  ```bash
  gobuster dir -u http://<IP> -w wordlist.txt -x php,txt,zip
  ```
- `-s <códigos>` → lista blanca de códigos de estado HTTP a mostrar (por defecto muestra 200, 204, 301, 302, 307, 401, 403).
  ```bash
  gobuster dir -u http://<IP> -w wordlist.txt -s 200,301,403
  ```
- `-b <códigos>` → lista negra de códigos de estado a **ocultar** de los resultados (ej. para filtrar el ruido de 404).
  ```bash
  gobuster dir -u http://<IP> -w wordlist.txt -b 404
  ```
- `-c <cookie>` → envía una cookie de sesión, útil cuando el directorio a explorar requiere estar autenticado.
  ```bash
  gobuster dir -u http://<IP> -w wordlist.txt -c "PHPSESSID=abc123"
  ```
- `-H "<header>"` → agrega un header HTTP personalizado a cada petición.
  ```bash
  gobuster dir -u http://<IP> -w wordlist.txt -H "Authorization: Bearer <token>"
  ```
- `-r` → sigue redirecciones (por defecto gobuster no sigue redirects 301/302).
  ```bash
  gobuster dir -u http://<IP> -w wordlist.txt -r
  ```
- `-k` → ignora errores de certificado SSL/TLS (para HTTPS con certificados autofirmados, muy común en labs).
  ```bash
  gobuster dir -u https://<IP> -w wordlist.txt -k
  ```
- `--add-slash` → agrega un `/` al final de cada petición, útil para detectar directorios correctamente.
- `-e` → muestra las URLs completas en vez de solo la ruta relativa.

---

## 8. Combo recomendado para DockerLabs

Flujo típico para la parte web de una máquina de la plataforma:

```bash
# Paso 1: enumeración rápida de directorios y archivos comunes
gobuster dir -u http://<IP> -w /usr/share/wordlists/dirb/common.txt -t 50 -o paso1_dirs.txt

# Paso 2: si encuentras un panel/tecnología específica (ej. WordPress, PHP), repite
# con extensiones y una wordlist más grande, filtrando el ruido de 404
gobuster dir -u http://<IP> -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-medium-files.txt \
  -x php,txt,bak,zip -t 50 -b 404 -o paso2_dirs_grande.txt

# Paso 3 (si el reto usa dominios/vhosts): buscar subdominios o vhosts ocultos
gobuster vhost -u http://<dominio> -w /usr/share/wordlists/subdomains-top1million-5000.txt --append-domain -o paso3_vhosts.txt
```

---

### Notas rápidas
- Si gobuster tira muchos falsos positivos (todo responde 200), el sitio probablemente usa una página de error personalizada: revisa el tamaño de respuesta y filtra con `-b` o usa `--exclude-length`.
- Wordlists típicas en Kali/DockerLabs: `/usr/share/wordlists/dirb/common.txt` (rápida) y las de `seclists` en `/usr/share/wordlists/seclists/Discovery/Web-Content/` (más completas, más lentas).
- Combina siempre con lo que ya sacaste de `nmap` (puerto 80/443, tecnología detectada) para elegir mejor la wordlist y extensiones.