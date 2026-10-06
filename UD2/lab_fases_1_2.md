# Laboratorio fases 1 y 2 — Estudio Torrent

## A1. whois y nslookup
No es necesario

## B. Antes y después
### B1. Preparación
1. Arrancad Torrent-Auditor y Torrent-Vulnerable.
2. Antes de tocar nada, haced una instantánea de Torrent-Vulnerable (en VirtualBox: menú Máquina → Tomar instantánea). Vais a modificarla y luego la necesitaremos tal como estaba.
3. Anotad la IP de Torrent-Vulnerable (ip a dentro de ella).
   192.168.1.101
5. Desde Torrent-Auditor, comprobad que la veis: ping -c 3 <IP>.

   <img width="623" height="115" alt="image" src="https://github.com/user-attachments/assets/4b7c01a3-d0e5-4a9d-8063-5bb5ee43804f" />

### B2. Medición ANTES
En Torrent-Auditor: 
sudo nmap -sV <IP> -oN antes.txt
Apuntad: 
- Cuántos puertos abiertos hay: 22 puertos abiertos
- Qué servicios y versiones aparecen:

Puerto | Servicio | Versión
-------|----------|--------
21 | ftp | vsftpc 2.3.4
22 | ssh | OpenSSH 4.7p1 Debian 8ubuntu (protocol 2.0)
25 | smtp? | -
53 | domain | ISC BIND 9.4.2
80 | http | Apache httpd 2.2.8 ((Ubuntu) DAV/2)
111 | rpcbind | 2 (RPC #100000)
139 | netbios-ssn | Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445 | netbios-ssn | Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
512 | esec? | - 
513 | login? | -
514 | shell? | -
1099 | java-rmi | GNU Classpath grmiregistry
1524 | bindshell | Metasplotaible root shell
2049 | nfs | 2-4 (RPC #100003)
2121 | ccproxy-ftp? | -
3306 | msql? | -
5432 | postgresql | PostgreSQL DB 8.3.0 - 8.3.7
5900 | vnc | VNC (protocol 3.3)
6000 | X11 | (access denied)
6667 | irc | UnrealIRCd
8009 | ajp13 | Apache Jserv (Protocol v1.3)
8180 | http | Apache Tomcar/Coyote JSP engine 1.1

- Si el ping responde: Si responde

### B3. Contramedidas (en Torrent-Vulnerable, con sudo)
Usuario msfadmin, contraseña msfadmin.

Suposición de trabajo: este servidor solo tiene que ofrecer servicio web (puerto 80) y administración remota por SSH (puerto 22). Todo lo demás sobra.

#### Contramedida 1 — Inventariar y apagar servicios innecesarios
1. Listad lo que escucha: sudo netstat -tulpn

2. Haced una tabla: puerto, servicio, ¿es necesario según la suposición de trabajo?, decisión.

Puerto | Servicio | Es necesario | Decisión
-------|----------|--------------|----------
512
513
2049
514
47587 | rmiregisty | No |
8009 | jsvc | No |
6697 | unrealircd | No |
37834 | rpc.mountd | No |
3306 | mysqld | No |
1099 | rmiregistry | No |
6667 | unrealircd | No |
139 | smbd | No |
5900 | Xtightvnc | No |
45260 |  
44559 | rpc.statd | No |
111 | portmap | No |
6000 | Xtightvnc | No |
80 | Apache | Sí |
8787 | ruby | No |
8180 | jsvc | No |
1524 | 
21 |
5432 | postgres | No |
25 | master | No |
445 | smbd | No |
2121 | profttpd | No |
3632 | distccd | No |
53 | named | No |
22 | sshd | Si |
5432 | postgres | No |
953 | named | No |
2049 |
38151 |
137 | nmbd | No |
138 | nmbd | No |
911 | rpc.statd | No |
57123 | named | No |
57638 | rpc.mountd | No |
69 |
39764 | rcp.statd | No |
111 | portmap | No |
53 | named | No |
50375 | named | No |

4. Apagad los servicios que sobran. El profesor os dará el método para cada uno (unos se paran con su script de /etc/init.d/, otros se desactivan en un archivo de configuración).

#### Contramedida 2 — Bloquear el ping
sudo iptables -A INPUT -p icmp --icmp-type echo-request -j DROP

Comprobad desde Torrent-Auditor: ping -c 3 <IP>.

### B4. Medición DESPUÉS
En Torrent-Auditor:
sudo nmap -sV <IP> -oN despues.txt
diff antes.txt despues.txt

Rellenad la tabla:

Aspecto | Antes | Contramedida aplicada | Después
--------|-------|-----------------------|---------

### B5. Preguntas finales
1. ¿Qué contramedida ha reducido más lo que ve el atacante? ¿Por qué?
2. Si solo ocultáis un banner pero el servicio sigue activo, ¿la vulnerabilidad sigue ahí? Razonad la respuesta.
3. Torrent-Vulnerable usa un sistema operativo sin soporte desde hace años. ¿Puede una contramedida de estas sustituir a actualizarlo? ¿Qué haríais en una empresa real?
4. De todas las contramedidas de las partes A y B, ¿cuáles evitan que el atacante encuentre información y cuáles solo ayudan a detectar que lo están intentando? (Aún no las hemos aplicado todas, pero pensad en cuáles serían de cada tipo.)


Putty -> vt100
En MS2 -> etc/init.d/ssh -> UseDNS no

Teclado en español:
sudo loadkeys es
