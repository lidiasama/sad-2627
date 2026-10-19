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
Suposición de trabajo: este servidor solo tiene que ofrecer servicio web (puerto 80) y administración remota por SSH (puerto 22). Todo lo demás sobra.

#### Contramedida 1 — Inventariar y apagar servicios innecesarios
1. Listad lo que escucha:
   ``` sudo netstat -tulpn ```

3. Haced una tabla: puerto, servicio, ¿es necesario según la suposición de trabajo?, decisión.

Puerto | Servicio | Es necesario | Decisión
-------|----------|--------------|----------
512 | exec | No | Apagar
513 | login | No | Apagar
2049 | nfs | No | Apagar
514 | shell? | No | Apagar
47587 | rmiregisty | No | Apagar
8009 | jsvc | No | Apagar
6697 | unrealircd | No | Apagar
37834 | rpc.mountd | No | Apagar
3306 | mysqld | No | Apagar
1099 | rmiregistry | No | Apagar
6667 | unrealircd | No | Apagar
139 | smbd | No | Apagar
5900 | Xtightvnc | No | Apagar
45260 |   | No | Apagar
44559 | rpc.statd | No | Apagar
111 | portmap | No | Apagar
6000 | Xtightvnc | No | Apagar
80 | Apache | Sí | Mantener
8787 | ruby | No | Apagar
8180 | jsvc | No | Apagar
1524 |  | No | Apagar
21 | | No | Apagar
5432 | postgres | No | Apagar
25 | master | No | Apagar
445 | smbd | No | Apagar
2121 | profttpd | No | Apagar
3632 | distccd | No | Apagar
53 | named | No | Apagar
22 | sshd | Si | Mantener
5432 | postgres | No | Apagar
953 | named | No | Apagar

4. Apagad los servicios que sobran. El profesor os dará el método para cada uno (unos se paran con su script de /etc/init.d/, otros se desactivan en un archivo de configuración).

##### Puerto 512
Vamos a averiguar que servicio y archivo es:

PASO 1: Preguntárselo al servicio (el más fiable). Ve a la máquina Torrent-Auditor. Ejecuta: ``` sudo nmap -sV -p 512 192.168.1.101 ```

En este caso la IP es la IP del host vulnerable y 512 es el puerto del servicio que queremos conocer.

<img width="962" height="245" alt="image" src="https://github.com/user-attachments/assets/b46eb227-c940-400e-aa21-42c301de451d" />

Aquí podemos confirmar que es el Servicio exec

PASO 2: Ahora que tenemos el nombre del servicio vamos a averiguar donde está el servicio, si en un archivo de xinetd.d o en una linea dentro de inetd.conf

<img width="900" height="65" alt="image" src="https://github.com/user-attachments/assets/b9e02c3a-f6bb-4a99-ad9c-5673ecba5ce5" />

Aquí podemos observar que cuando buscamos si existe un archivo con el nombre del servicio en xinetd no sale nada, pero cuando buscamos si existe una línea con el nombre del servicio dentro de inetd si que nos aparece.

PASO 3: Vamos a apagar el puerto 512. Como tiene un portero cuando lo apagamos ya no vuelve.

Para ello debemos comentar la línea en inetd.conf. En la máquina Torrent-Vulnerable, ejecuta para poner la almohadilla "#" al inicio de la línea de exec: ``` sudo sed -i 's/^exec/#exec/' /etc/inetd.conf ```

Verificación: Ejecuta ``` grep exec /etc/inetd.conf ``` y comprueba que la línea ahora comienza por #exec

<img width="722" height="52" alt="image" src="https://github.com/user-attachments/assets/c8a1763b-3653-4024-87c8-b20f82227319" />


PASO 4: Recargar el portero. Recarga el servicio usando su script en /etc/init.d/ para que relea el fichero sin tumbar las demás conexiones: ``` sudo /etc/init.d/xinetd reload ```

<img width="718" height="35" alt="image" src="https://github.com/user-attachments/assets/e576fa70-5553-47f0-ad46-d56f633be7a6" />

PASO 5: Comprobamos que el puerto esta cerrado y que los demás servicios siguen vivos
Con ''' sudo netstat -tulpn | grep ':512' ``` no debe devolver nada
<img width="652" height="32" alt="image" src="https://github.com/user-attachments/assets/6d1fa62d-fcbf-4870-b1c5-51c4c1528528" />

Abre la terminal en la máquina Torrent-Auditor y lanza el escaneo apuntando a la IP de Torrent-Vulnerable (192.168.1.101): ``` Sudo nmap -p 512 192.168.1.101 ``` 
<img width="693" height="185" alt="image" src="https://github.com/user-attachments/assets/7b7af9b5-c69c-40dc-af64-32e4b31252e4" />

El puerto debe figurar como closed

##### Puerto 513
PASO 1: Identifica (en Torrent-Vulnerable), consultas quién tiene el puerto: ``` sudo netstat -tulpn | grep ':513 ' ```
<img width="677" height="50" alt="image" src="https://github.com/user-attachments/assets/c702b684-2d5c-4fa7-9695-f7834e0bd69f" />

Verificación: Te devuelve que el dueño es 4455/xinetd. Con esto ya sabes que estás ante un servicio a demanda

PASO 2: Saber qué servicio es.

Para saber qué servicio maneja ese puerto 513 en Torrent-Vulnerable ejecutamos: ``` grep -w 513/tcp /etc/services ```
<img width="600" height="35" alt="image" src="https://github.com/user-attachments/assets/f58e549e-150c-45c9-93ba-17211d99805f" />

Resultado: Devuelve login 513/tcp. Ya sabes que el nombre es login.

PASO 3: Averigua dónde vive (en Torrent-Vulnerable).
Ahora que sabes que se llama login, ejecutas el comando: ``` sudo grep -n -w login /etc/inetd.conf /etc/xinetd.conf /etc/xinetd.d/* 2>/dev/null ```
<img width="722" height="65" alt="image" src="https://github.com/user-attachments/assets/b2a11dec-db7b-4eec-9342-948b9b9d1af8" />

Verificación: Te muestra que está en /etc/inetd.conf en la línea de login.

PASO 4: Comentar la línea y recargar
En los porteros, parar e impedir es un solo paso. Ejecutas en Torrent-Vulnerable: ```sudo cp /etc/inetd.conf /root/inetd.conf.bak ``` , ```sudo sed -i 's/^login/#login/' /etc/inetd.conf ``` , ``` sudo /etc/init.d/xinetd reload ```

Verificación: Comprueba con grep '^#login' /etc/inetd.conf que la línea comienza por #.

<img width="717" height="32" alt="image" src="https://github.com/user-attachments/assets/3663f04e-0d7b-4b34-af69-57a6366863c4" />

PASO 5: Comprueba
Dentro (Torrent-Vulnerable): ```sudo netstat -tulpn | grep ':513 ' ```
<img width="578" height="50" alt="image" src="https://github.com/user-attachments/assets/a781a6bb-a9a3-4b65-b08e-74c683abaa10" />

Fuera (Torrent-Auditor): ``` sudo nmap -p 513 192.168.1.101 ```
<img width="707" height="220" alt="image" src="https://github.com/user-attachments/assets/7afebb36-12d4-42af-86ac-a50c9381ddd2" />

##### Puerto 2049
PASO 1:IDENTIFICA(En Torrent-Vulnerable). Comprueba el estado del socket y quién es su dueño en el sistema: ``` sudo netstat -tulpn | grep ':2049 ' ```
<img width="691" height="48" alt="image" src="https://github.com/user-attachments/assets/ce8f8cab-ded3-4390-836a-f3c9254ce592" />

En la columna PID/Program name no aparece xinetd, sino un guion (-). Esto ocurre porque NFS opera a través de hilos del propio núcleo (kernel threads). Al no estar gestionado por un portero (inetd/xinetd), se clasifica como un servicio permanente

PASO 2: AVERIGUA (¿Qué servicio es y cómo arranca?)
Para saber el nombre oficial del protocolo (en Torrent-Vulnerable): ``` grep -w 2049/tcp /etc/services ```
<img width="666" height="36" alt="image" src="https://github.com/user-attachments/assets/bfd95cdb-962a-4e4b-8b17-ec6330ce8a7a" />

Resultado: nfs 2049/tcp. El servicio es NFS (Network File System).



#### Contramedida 2 — Bloquear el ping
sudo iptables -A INPUT -p icmp --icmp-type echo-request -j DROP

Comprobad desde Torrent-Auditor: ping -c 3 <IP>.
Ahora localizamos su script de inicio (en Torrent-Vulnerable): ``` ls /etc/init.d/ | grep nfs ```
<img width="621" height="83" alt="image" src="https://github.com/user-attachments/assets/6e3be3ef-1518-4ac8-b60b-d99b71b05d03" />

Resultado: Aparece el script gestor: nfs-kernel-server (y nfs-common).
Ahora verificamos sus enlaces de arranque: ``` ls -l /etc/rc2.d/ | grep nfs ``` 
<img width="725" height="77" alt="image" src="https://github.com/user-attachments/assets/2681a92b-2a7f-493c-9f75-50c00969e483" />

Resultado: Verás enlaces simbólicos que empiezan por S (como S20nfs-kernel-server), lo que indica que el sistema lo arranca automáticamente al encender el equipo.

PASO 3:PARA(En Torrent-Vulnerable)
Detén el demonio activo en memoria para liberar el puerto inmediatamente (sin usar killall): ``` sudo /etc/init.d/nfs-kernel-server stop ``` y ```sudo /etc/init.d/nfs-common stop ```
<img width="722" height="55" alt="image" src="https://github.com/user-attachments/assets/475797a3-958d-4b5c-87cb-596089814af1" />
<img width="710" height="33" alt="image" src="https://github.com/user-attachments/assets/8ad3ce01-44d2-416a-8895-966ac655984f" />

Verificación: La consola devolverá la confirmación de parada

PASO 4: IMPIDE (En Torrent-Vulnerable)
Elimina los enlaces automáticos en /etc/rc*.d/ para que no vuelva a levantarse tras reiniciar: ``` sudo update-rc.d -f nfs-kernel-server remove ``` y ``` sudo update-rc.d -f nfs-common remove ```
<img width="721" height="148" alt="image" src="https://github.com/user-attachments/assets/54aa7346-69fe-406c-8d29-b86b44b229dd" />

Verificación: El comando indicará ´´´ Removing any system startup links for /etc/init.d/nfs-kernel-server .... ``` Si vuelves a consultar con ls /etc/rc2.d/ | grep nfs, ya no debe aparecer ningún enlace con S <img width="572" height="20" alt="image" src="https://github.com/user-attachments/assets/42dc6579-c5ca-47dd-8588-15834ac981fd" />

PASO 5: COMPRUEBA
Desde Torrent-Vulnerable si el socket en el puerto 2049 sigue existiendo: ``` sudo netstat -tulpn | grep ':2049 ' ```
<img width="626" height="32" alt="image" src="https://github.com/user-attachments/assets/8ecc565e-edfe-4ce8-9ba3-8179ed68d71a" />

Verificación: La salida debe quedar vacía (no devuelve ninguna línea).

Desde Torrent-Auditor realiza el escaneo hacia la IP objetivo: ``` sudo nmap -p 2049 192.168.1.101 ```
<img width="882" height="226" alt="image" src="https://github.com/user-attachments/assets/2f6fafaf-7050-4a04-beb4-598819e8a17d" />

Verificación: La columna STATE debe reportar obligatoriamente closed

### B4. Medición DESPUÉS
En Torrent-Auditor:
``` sudo nmap -sV <IP> -oN despues.txt 
diff antes.txt despues.txt ```

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
