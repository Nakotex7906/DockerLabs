# Comandos útiles para resolver laboratorios de CTF

## Ping

Permite comprobar si una máquina está activa. Además, el valor del **TTL** puede dar una pista sobre el sistema operativo.

- **TTL ≈ 64:** Linux
- **TTL ≈ 128:** Windows

### Comando

```bash
ping -c 4 IP
```

**Parámetros**

- `-c 4` → Envía únicamente 4 paquetes ICMP.

---

# Nmap

Se utiliza para descubrir los puertos abiertos y los servicios que se están ejecutando en una máquina.

### Comando

```bash
nmap -sV IP
```

**Parámetros**

- `-sV` → Detecta la versión de los servicios encontrados.

---

# Hydra

Herramienta para realizar ataques de fuerza bruta contra distintos servicios, como SSH, FTP, HTTP, entre otros.

### Ejemplo

Si ya se conoce el usuario (`borazuwarah`):

```bash
hydra -l borazuwarah -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2/ -t 4 -v
```

**Parámetros**

- `-l` → Usuario.
- `-P` → Diccionario de contraseñas.
- `-t 4` → Ejecuta hasta 4 intentos en paralelo.
- `-v` → Muestra información detallada del proceso.

---

# ExifTool

Permite visualizar y modificar los metadatos de un archivo.

Es muy útil en retos de **esteganografía**, ya que puede revelar información oculta.

### Comando

```bash
exiftool archivo
```

Ejemplo:

```bash
exiftool imagen.jpg
```

---

# Gobuster

Se utiliza para descubrir directorios y archivos ocultos en un servidor web.

### Comando

```bash
gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirb/common.txt -x html,php,txt
```

**Parámetros**

- `dir` → Modo de búsqueda de directorios.
- `-u` → URL objetivo.
- `-w` → Diccionario de palabras.
- `-x` → Busca archivos con las extensiones indicadas.

---

# Métodos para obtener una shell como root

## `sudo -i`

Método más recomendado.

- Simula un inicio de sesión completo como root.
- Carga el entorno de root.
- Cambia al directorio `/root`.
- Solicita la contraseña del usuario (si tiene permisos sudo).

```bash
sudo -i
```

---

## `sudo su -`

Equivalente práctico a `sudo -i`.

- Ejecuta `su -` mediante `sudo`.
- Carga el entorno completo de root.
- Cambia al directorio `/root`.
- Solicita la contraseña del usuario.

```bash
sudo su -
```

---

## `sudo su`

Abre una shell como root, pero mantiene parte del entorno del usuario.

- Conserva el directorio actual.
- Mantiene parte de las variables de entorno.
- Solicita la contraseña del usuario.

```bash
sudo su
```

---

## `su -`

Método tradicional para cambiar al usuario root.

- Requiere que la cuenta root tenga una contraseña configurada.
- Solicita la contraseña de **root**.
- Carga un entorno limpio.
- Cambia al directorio `/root`.

```bash
su -
```
