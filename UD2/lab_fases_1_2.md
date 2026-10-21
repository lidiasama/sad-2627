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
8009 | jsvc | No | Apagar
33417 | rmiregisty | No | Apagar
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
1524 | ingreslock | No | Apagar
21 | ftp | No | Apagar
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

## PASOS A SEGUIR PARA TUMBAR UN SERVICIO
### Diagnóstico inicial: El punto de partida común

- En Torrent-Vulnerable, consulta el puerto: ``` sudo netstat -tulpn | grep ':PUERTO ' ```
- Fíjate en la última columna (PID/Program name):

#### Caso 1: Si pone el nombre directo del servicio:
- Comprueba si tiene script en /etc/init.d/ con: ``` ls /etc/init.d/ | grep NOMBRE ```
   - Comprobar si el nombre no coincide (como jsvc): 
      - Opción A: (averiguar el paquete con dpkg):
        1. Prueba con: ``` dpkg -S $(which NOMBRE) ```
        2. Si 'which' no devuelve nada: averigua su ruta por el PID con: ``` ls -l /proc/PID/exe ```. Y luego consulta el paquete: ```dpkg -S /ruta/obtenida```
      - Opción B: busca si algún script lo invoca: ``` grep -l 'NOMBRE' /etc/init.d/* 2>/dev/null ```
     - Si SÍ aparece un archivo en init.d: Aplica el **Procedimiento 1**
     - Si NO aparece nada en init.d: Aplica el **Procedimiento 3.**

#### Caso 2: Si pone xinetd o inetd
- Aplica directamente el **Procedimiento 2**

### PROCEDIMIENTO 1: SERVICIO PERMANENTE CON SCRIPT EN /etc/init.d/
1. Identifica (en Torrent-Vulnerable):
- Ejecuta: ``` sudo netstat -tulpn | grep ':PUERTO ' ```
   - Verificación: que en la última columna aparece su propio ejecutable y PID, no xinetd.

2. Averiguar cómo arranca (en Torrent-Vulnerable):
- Ejecuta: ``` ls /etc/init.d/ | grep NOMBRE ``` y ```  ls -l /etc/rc2.d/ | grep NOMBRE ```
   - Verificación: existe el script en init.d y un enlace simbólico que empieza por S en rc2.d

3. Parar ahora (en Torrent-Vulnerable):
- Ejecuta: ``` sudo /etc/init.d/NOMBRE_DEL_SCRIPT stop ```
   - Verificación: devuelve un mensaje de confirmación con [ OK ] y al repetir ´´´ sudo netstat -tulpn | grep ':PUERTO ' ``` la salida queda vacía.
   - Si devuelve ``` not implemented ``` : el script no implementa la parada. Se fuerza la detención en memoria:
      - Opción A (por nombre de ejecutable): ```sudo killall -9 NOMBRE_EXACTO_BINARIO ```
      - Opción B (por PID, infalible): ```sudo kill -9 PID```
      - Verificación: ```sudo netstat -tulpn | grep ':PUERTO ' ``` debe quedar completamente en blanco.

4. Impedir tras reiniciar (en Torrent-Vulnerable):
- Ejecuta: ``` sudo update-rc.d -f NOMBRE_DEL_SCRIPT remove ```
   - Verificación: El comando informa de la retirada de enlaces. Al hacer ``` ls /etc/rc2.d/ | grep NOMBRE ```  ya no devuelva nada.

5. Comprobar (en Torrent-Auditor):
- Ejecuta: ``` sudo nmap -p PUERTO 192.168.1.101 ```
   - Verificación: el estado del puerto figure como "closed" (si pone filtered solo está oculto con cortafuegos, no apagado).


### PROCEDIMIENTO 2: SERVICIO A DEMANDA BAJO EL PORTERO (xinetd / inetd)
1. Identificar (en Torrent-Vulnerable):
- Ejecuta: ``` sudo netstat -tulpn | grep ':PUERTO ' ```
   - Verificación: En la columna derecha figura explícitamente xinetd o inetd

2. Averiguar dónde está su fichero (en Torrent-Vulnerable):
   - (En los ficheros del portero no suele figurar el número de puerto, sino su nombre de servicio. Por eso se divide en dos fases:)
- Paso 2.1. Averiguar el nombre oficial del servicio (Agenda del sistema / Escaneo)
   - Opción A. Ejecuta: ``` grep -w 'PUERTO/tcp' /etc/services ```
   - La primera columna te da el nombre exacto del servicio
   - Opción B.(Preguntárselo al servicio desde Torrent-Auditor) Ejecuta: ``` sudo nmap -sV -p PUERTO 192.168.1.101 ```

- Paso 2.2. Localizar dónde se define dentro del portero
   - Ejecuta: ``` sudo grep -rn -w 'NOMBRE' /etc/inetd.conf /etc/xinetd.conf /etc/xinetd.d/ 2>/dev/null ```
   - Verificación: Muestra la ruta exacta y el número de línea donde se define el servicio.
   - (Si no arrojase nada, busca por si acaso por el número con sudo grep -rn -w 'PUERTO' /etc/inetd.conf /etc/xinetd.conf /etc/xinetd.d/* 2>/dev/null)

3. Parar e impedir a la vez (en Torrent-Vulnerable): 
- Si el resultado estaba en /etc/inetd.conf:
   - Ejecuta: ``` sudo cp /etc/inetd.conf /root/inetd.conf.bak```, ``` sudo sed -i 's/^SERVICIO/#SERVICIO/' /etc/inetd.conf ```  y ``` sudo /etc/init.d/xinetd reload ```
   - Luego recarga el portero con: ``` sudo /etc/init.d/xinetd reload ```
   - Comprueba que se comentó con: ```grep '^#SERVICIO' /etc/inetd.conf ```

- Si el resultado estaba en /etc/xinetd.d/NOMBRE:
   - Ejecuta: ```sudo cp /etc/xinetd.d/SERVICIO /root/SERVICIO.bak```, ``` sudo sed -i 's/disable.*/disable = yes/' /etc/xinetd.d/SERVICIO ``` y  ``` sudo /etc/init.d/xinetd reload ```
   - Luego recarga el portero con: ```sudo /etc/init.d/xinetd reload```
   - Comprueba el cambio con: ``` grep -H disable /etc/xinetd.d/SERVICIO ``` 

 
  - Notas:
     - Si tras recargar xinetd sigue abierto el puerto, probad restart en vez de reload: ``` sudo /etc/init.d/xinetd restart ```
     - Si os equivocáis o rompéis algo: restaurad la copia de seguridad de /root (.bak) y recargad xinetd

4. Comprobar (en Torrent-Auditor):
- Ejecuta: ``` sudo nmap -p PUERTO 192.168.1.101 ```
   - Verifica que el puerto objetivo salga en estado "closed".

### PROCEDIMIENTO 3: SERVICIO PERMANENTE SIN SCRIPT EN /etc/init.d/
1. Identificar (en Torrent-Vulnerable):
- Ejecuta: ``` sudo netstat -tulpn | grep ':PUERTO ' ```
   - Verificación:  muestra un ejecutable propio (ej: rmiregistry) y no xinetd.   

2. Averiguar de dónde arranca (en Torrent-Vulnerable):
- Paso 2.1: Comprueba que no existe en init.d ejecutando: ```ls /etc/init.d/ | grep NOMBRE ```
   - Verificación: la salida está totalmente vacía.  

- Paso 2.2: Busca la línea de arranque en /etc/rc.local ejecutando: ``` grep -n 'NOMBRE' /etc/rc.local ```
   - Verificación: el número de línea donde está escrita la orden de inicio.

- Paso 2.3: Si no saliera en rc.local, comprobar crontab con: ``` sudo crontab -l | grep '@reboot' ```

- Paso 2.4: Si no aparece en ninguno de los anteriores, verifica si se lanzó a mano en la sesión: ``` ps -ef --forest | grep -B 2 'NOMBRE' ```

3. Parar e impedir (en Torrent-Vulnerable):
Paso 3.1. Comentar la orden de arranque con copia de seguridad y sed:
- Si estaba en /etc/rc.local: ``` sudo cp /etc/rc.local /root/rc.local.bak```, ```sudo sed -i 's/^.*NOMBRE/#&/' /etc/rc.local ```
   - Verificación: ejecuta cat /etc/rc.local | grep NOMBRE y confirma que la línea empieza por #
 
- Si estaba en crontab: ``` sudo crontab -e ```-> Pon # delante de la línea NOMBRE, guarda y sal

- Si se lanzó a mano debemos tener el PID y qué terminal o usuario lo inició. Para ello...
   - Ejecutamos: ```ps -ef --forest | grep -B 2 'NOMBRE'```
   - Como no hay script /etc/init.d/ para hacer stop, la orden directa en Linux para terminar un proceso interactivo es matarlo por su PID o por su nombre:
   - Ejecutamos: ```sudo kill -9 PID``` o ```sudo killall -9 NOMBRE```
   - Verificación: Ejecuta sudo netstat -tulpn | grep ':PUERTO ' y confirma que la salida queda completamente vacía.

Paso 3.2. Parar el proceso y asegurar que no vuelve
- Ejecuta: ``` sudo reboot ```

- 

4. Comprobar tras reiniciar:
- Dentro (Torrent-Vulnerable): ```sudo netstat -tulpn | grep ':PUERTO ' ``` 
   - Verificación: Salida vacía.    
- Fuera (Torrent-Auditor): ``` sudo nmap -p PUERTO 192.168.1.101 ``` 
   - Estado closed


### Puerto 512
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

### Puerto 513
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

Verificación: Comprueba con ``` grep '^#login' /etc/inetd.conf ```su que la línea comienza por #.

<img width="717" height="32" alt="image" src="https://github.com/user-attachments/assets/3663f04e-0d7b-4b34-af69-57a6366863c4" />

PASO 5: Comprueba
Dentro (Torrent-Vulnerable): ```sudo netstat -tulpn | grep ':513 ' ```
<img width="578" height="50" alt="image" src="https://github.com/user-attachments/assets/a781a6bb-a9a3-4b65-b08e-74c683abaa10" />

Fuera (Torrent-Auditor): ``` sudo nmap -p 513 192.168.1.101 ```
<img width="707" height="220" alt="image" src="https://github.com/user-attachments/assets/7afebb36-12d4-42af-86ac-a50c9381ddd2" />

### Puerto 2049
PASO 1:IDENTIFICA(En Torrent-Vulnerable). Comprueba el estado del socket y quién es su dueño en el sistema: ``` sudo netstat -tulpn | grep ':2049 ' ```
<img width="691" height="48" alt="image" src="https://github.com/user-attachments/assets/ce8f8cab-ded3-4390-836a-f3c9254ce592" />

En la columna PID/Program name no aparece xinetd, sino un guion (-). Esto ocurre porque NFS opera a través de hilos del propio núcleo (kernel threads). Al no estar gestionado por un portero (inetd/xinetd), se clasifica como un servicio permanente

PASO 2: AVERIGUA (¿Qué servicio es y cómo arranca?)
Para saber el nombre oficial del protocolo (en Torrent-Vulnerable): ``` grep -w 2049/tcp /etc/services ```
<img width="666" height="36" alt="image" src="https://github.com/user-attachments/assets/bfd95cdb-962a-4e4b-8b17-ec6330ce8a7a" />

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


### Puerto 8009 y 8180
PASO 1: IDENTIFICA (En Torrent-Vulnerable). Comprueba el estado del socket y quién es su dueño en el sistema: ``` sudo netstat -tulpn | grep ':8009 ' ```
<img width="951" height="42" alt="image" src="https://github.com/user-attachments/assets/862b7aa5-d23e-4a22-9a7f-721bdab89d60" />

Verificación: Verás que el dueño en la última columna es .../jsvc. Al ser su propio programa y no xinetd, confirmamos que es un servicio permanente.

PASO 2: AVERIGUA (¿Qué servicio es y cómo arranca?)
Como solo conocemos la palabra jsvc, buscamos qué script dentro de /etc/init.d/ lo invoca ``` grep -l 'jsvc' /etc/init.d/* ``` 
<img width="671" height="41" alt="image" src="https://github.com/user-attachments/assets/34d4cc84-7207-4cbc-969b-3af822061f22" />

El sistema confirma directamente que el script gestor es tomcat5.5.
A continuación, comprueba los enlaces de arranque en el runlevel 2: ``` ls -l /etc/rc2.d/ | grep tomcat ```
<img width="831" height="42" alt="image" src="https://github.com/user-attachments/assets/1758a97c-2f55-4081-8b73-d138a190be92" />

Verificación: Verás el enlace simbólico con S (S90tomcat5.5), confirmando que arranca solo al encender el equipo.

Ahora toca detener el proceso activo en memoria. Ejecuta el script oficial de parada con el nombre descubierto: ``` sudo /etc/init.d/tomcat5.5 stop ```
<img width="945" height="46" alt="image" src="https://github.com/user-attachments/assets/da3b6d4c-a5c1-48eb-94cb-91b38ee6cba5" />

Verificación: La consola responderá con un mensaje de parada (Stopping Tomcat servlet engine... [ OK ])

PASO 4: IMPIDE (Torrent-Vulnerable). Borra los enlaces automáticos de /etc/rc*.d/ para que no se reactive tras un reinicio: ``` sudo update-rc.d -f tomcat5.5 remove ```
<img width="681" height="182" alt="image" src="https://github.com/user-attachments/assets/f23fb4e6-16a5-4d0b-9265-d2b22706b430" />

Verificación: El comando listará los enlaces eliminados. Si compruebas con ``` ls /etc/rc2.d/ | grep tomcat ``` la salida quedará vacía.

Paso 5: COMPRUEBA.
Se verifican de manera conjunta los puertos 8009 y 8180 porque pertenecen a un único proceso (jsvc / Tomcat) que mantiene abiertos dos sockets de red simultáneamente. Al aplicar la orden de parada sobre el demonio se destruye dicho proceso en memoria y ambos puertos pasan a quedar cerrados a la vez.

1. Comprobación interna (en Torrent-Vulnerable): ``` sudo netstat -tulpn | grep -E ':8009 |:8180 ' ``` 
<img width="783" height="40" alt="image" src="https://github.com/user-attachments/assets/e7d2d44e-8106-4cb0-991a-090f3c18d9c0" />

Verificación: No devuelve nada

2. Comprobación externa (en Torrent-Auditor): ``` sudo nmap -p 8009,8180 192.168.1.101 ```
<img width="687" height="503" alt="image" src="https://github.com/user-attachments/assets/fa466f15-a53a-42ab-8ee4-ae9f54edd86d" />

Verificación: Ambos puertos deben figurar obligatoriamente como closed:

### Puerto 33417 y 1099
PASO 1: IDENTIFICA (En Torrent-Vulnerable). Comprobar que 33417 y 1099 comparten el mismo PID ``` sudo netstat -tulpn | grep ':33417' ```
<img width="976" height="58" alt="image" src="https://github.com/user-attachments/assets/6623f71e-198a-451b-badf-5e7bf0d637d2" />

Como no es xinetd, confirmamos que es un servicio permanente / proceso propio.

PASO 2: AVERIGUA. Hay que localizar de dónde sale el arranque. 
Primero miramos si tiene script en /etc/init.d/: ``` ls /etc/init.d/ | grep 'rmiregistry' ```
<img width="741" height="37" alt="image" src="https://github.com/user-attachments/assets/4e11c6a9-6ace-40b0-a9cb-8d561181b248" />

Si no tiene script en init.d (está en rc.local): ``` grep -n 'rmiregistry' /etc/rc.local ```
<img width="717" height="35" alt="image" src="https://github.com/user-attachments/assets/67ff40c0-c001-446a-b096-c32c85b4b4fe" />

Verificación: Te mostrará el número de línea donde está escrito el comando de arranque dentro de /etc/rc.local.

PASO 3 y 4: PARA E IMPIDE (Torrent-Vulnerable).
Si está en /etc/rc.local. Para impedir que vuelva a ejecutarse al reiniciar, se comenta la línea con #: ``` sudo sed -i 's/^.*rmiregistry/#&/' /etc/rc.local ```
<img width="841" height="63" alt="image" src="https://github.com/user-attachments/assets/2bf60030-83c0-45a9-a8e9-109e841252f1" />

Verificación: Ejecuta ``` cat /etc/rc.local | grep rmiregistry ``` y comprueba que la línea empieza por #.
Al haber quitado el arranque de /etc/rc.local, la forma para aplicar el cambio y verificarlo es reiniciar la máquina: ``` reboot ```

PASO 5: COMPRUEBA
Dentro (Torrent-Vulnerable): ``` sudo netstat -tulpn | grep -E ':1099 |:33417 ' ``` (no debe mostrar nada).
<img width="685" height="32" alt="image" src="https://github.com/user-attachments/assets/a13ce6d9-f45d-40d7-bfb3-f3870f0b3941" />

Fuera (Torrent-Auditor): ``` sudo nmap -p 1099 192.168.1.101 ``` (debe mostrar closed).
<img width="682" height="427" alt="image" src="https://github.com/user-attachments/assets/386d6825-ef5d-4d12-8773-1b6e457d4f02" />

### Puerto 6697
Diagnóstico inicial: 
PASO 1: En Torrent-Vulnerable, consulta el puerto: 
- Ejecuta: '''sudo netstat -tulpn | grep 6697
<img width="958" height="43" alt="image" src="https://github.com/user-attachments/assets/36bb22de-9f2f-45de-82d7-a96578a4f11d" />

- Nombre del programa unrealircd

PASO 2: Comprueba si tiene script en /etc/init.d/ 
- Ejecuta: ``` ls /etc/init.d/ | grep NOMBRE ```
<img width="740" height="41" alt="image" src="https://github.com/user-attachments/assets/1975e3b2-578d-4112-b3c6-b9cc190f1916" />

PASO 3: Averiguar de dónde arranca (en Torrent-Vulnerable)
- Paso 2.1: Comprueba que no existe en init.d ejecutando: ``` ls /etc/init.d/ | grep unrealircd ```
<img width="755" height="45" alt="image" src="https://github.com/user-attachments/assets/bbfcac17-1c25-46e0-956f-0a9ab9581c62" />

- Paso 2.2: Busca la línea de arranque en /etc/rc.local ejecutando: ``` grep -n 'unrelircd' /etc/rc.local ```
  <img width="742" height="40" alt="image" src="https://github.com/user-attachments/assets/186de26f-021c-4bd9-8407-bb45720228c5" />

PASO 4: Parar e impedir (en Torrent-Vulnerable):
- Paso 4.1. Comentar la orden de arranque con copia de seguridad y sed:
   - ``` sudo cp /etc/rc.local /root/rc.local.bak```
   - ```sudo sed -i 's/^.*unrealircd/#&/' /etc/rc.local ```
   - <img width="820" height="42" alt="image" src="https://github.com/user-attachments/assets/4b1100e2-f0f5-4e14-8692-09224bae8b25" />
   - Verificación: ejecuta ```cat /etc/rc.local | grep unrealircd ``` y confirma que la línea empieza por #
   - <img width="747" height="45" alt="image" src="https://github.com/user-attachments/assets/c2652ef2-b751-43e6-8170-c129fe20965f" />

- Paso 4.2. Parar el proceso y asegurar que no vuelve
   - Ejecuta: ``` sudo reboot ```
 
PASO 5: Comprobar tras reiniciar:
- Dentro (Torrent-Vulnerable):
   - Ejecuta ```sudo netstat -tulpn | grep ':6697 '  ```
   - <img width="722" height="41" alt="image" src="https://github.com/user-attachments/assets/810313ee-8b83-4f73-add4-dda1d890e7ec" />
   - Verificación: Salida vacía.
 
- Fuera (Torrent-Auditor):
   - Ejecuta: ```sudo nmap -p 6697 192.168.1.101```
   - <img width="681" height="183" alt="image" src="https://github.com/user-attachments/assets/61d2f818-c21e-4b80-b3c2-aa2228c9e812" />
   - Estado closed

### Puerto 3306 
PASO 1: Diagnóstico inicial
- En Torrent-Vulnerable, consulta el puerto: ``` sudo netstat -tulpn | grep ':3306 '```
- <img width="935" height="45" alt="image" src="https://github.com/user-attachments/assets/7c255be8-e499-43b0-967d-25376f72d0b7" />

PASO 2: Comprueba si tiene script en /etc/init.d/ 
- Ejecuta: ``` ls /etc/init.d/ | grep mysqld ```
- <img width="665" height="38" alt="image" src="https://github.com/user-attachments/assets/9639b134-25ef-4e8f-b876-aa0d5be88abd" />
- Si el nombre no coincide (como jsvc): localiza el script que lo invoca con: ```grep -l 'NOMBRE' /etc/init.d/* ```
   - <img width="670" height="42" alt="image" src="https://github.com/user-attachments/assets/989c86ad-3a9e-4717-b697-79c6bffe383c" />

PASO 3: Averiguar cómo arranca (en Torrent-Vulnerable):
- Ejecuta:``` ls /etc/init.d/ | grep mysql ``` y  ```ls -l /etc/rc2.d/ | grep mysql ```
- <img width="910" height="165" alt="image" src="https://github.com/user-attachments/assets/92a4e948-cc84-477a-a560-d206cc5608d2" />
   - Verificación: existe el script en init.d y 3 enlaces simbólicos que empieza por S en rc2.d
 
PASO 4: Parar ahora (en Torrent-Vulnerable):
- Ejecuta: ``` sudo /etc/init.d/mysql stop ```
- <img width="1040" height="47" alt="image" src="https://github.com/user-attachments/assets/8eee6457-5504-4d46-8046-8d734a68d7f6" />
   - Verificación: devuelve un mensaje de confirmación con [ OK ] y al repetir ´´´ sudo netstat -tulpn | grep ':3306 ' ``` la salida queda vacía.

PASO 5: Impedir tras reiniciar (en Torrent-Vulnerable):
- Ejecuta: ```sudo update-rc.d -f mysql remove ```
- <img width="712" height="187" alt="image" src="https://github.com/user-attachments/assets/b52a63a8-91d3-45dd-8c53-65370a3b469c" />
   - Verificación: El comando informa de la retirada de enlaces. Al hacer ls /etc/rc2.d/ | grep NOMBRE ya no devuelva nada.
 
PASO 6: Comprobar (en Torrent-Auditor):
- Ejecuta: ```sudo nmap -p 3306 192.168.1.101```
- <img width="656" height="253" alt="image" src="https://github.com/user-attachments/assets/eb7394a2-f438-4257-9417-a943ae2c4668" />
   - Verificación: el estado del puerto figure como "closed" (si pone filtered solo está oculto con cortafuegos, no apagado).
 
### Puerto 139
Diagnóstico inicial: En Torrent-Vulnerable, consulta el puerto:
- Ejecuta: ```sudo netstat -tulpn | grep ':139 ' ```
   - PID: 4365
   - Nombre: smbd

PASO 1: Comprueba si tiene script en /etc/init.d/ 
- Ejecuta: ```ls /etc/init.d/ | grep smbd ```
- Comprobar si el nombre no coincide (como jsvc): localiza el script que lo invoca con: ```grep -l 'smbd' /etc/init.d/*```
- <img width="653" height="52" alt="image" src="https://github.com/user-attachments/assets/14433bc2-a898-4b4a-aebe-908b3630ce51" />
   - Nombre: samba

PASO 2: Averiguar cómo arranca (en Torrent-Vulnerable):
- Ejecuta: ```ls /etc/init.d/ | grep samba ``` y ``` ls -l /etc/rc2.d/ | grep samba ```
   -<img width="720" height="87" alt="image" src="https://github.com/user-attachments/assets/40923753-5712-422b-85c2-af4b70569d02" />
   - Verificación: existe el script en init.dy un enlace simbólico que empieza por S en rc2.d

PASO 3: Parar ahora (en Torrent-Vulnerable):
- Ejecuta: ``` sudo /etc/init.d/samba stop ```
- <img width="726" height="62" alt="image" src="https://github.com/user-attachments/assets/55e32ee0-c7a2-449f-b8d5-fbe4a66c2176" />
   - Verificación: devuelve un mensaje de confirmación con [ OK ] y al repetir ´´´ sudo netstat -tulpn | grep ':139 ' ``` la salida queda vacía.

PASO 4: Impedir tras reiniciar (en Torrent-Vulnerable):
- Ejecuta: ``` sudo update-rc.d -f samba remove ```
- <img width="690" height="217" alt="image" src="https://github.com/user-attachments/assets/a8b2aa1c-1847-40bc-bd14-8bfe6b85c25f" />
   - Verificación: El comando informa de la retirada de enlaces. Al hacer ls /etc/rc2.d/ | grep samba ya no devuelva nada

PASO 5: Comprobar (en Torrent-Auditor):
- Ejecuta:sudo nmap -p 139 192.168.1.101
- <img width="521" height="71" alt="image" src="https://github.com/user-attachments/assets/f10a064f-4840-4b5b-bf86-7cebd124c542" />
   - Verificación: el estado del puerto figura como "cerrado" (si pone filtrado solo está oculto con cortafuegos, no apagado).


## Puerto 21
PASO 1: Identificar (en Torrent-Vulnerable):
- Ejecuta: ``` sudo netstat -tulpn | grep ':21 ' ```
   - Verificación: En la columna derecha figura explícitamente xinetd

PASO 2: Averiguar dónde está su fichero (en Torrent-Vulnerable):
Paso 2.1: Averiguar el nombre oficial del servicio
 - Ejecuta: grep -w '21/tcp' /etc/services
 - <img width="682" height="39" alt="image" src="https://github.com/user-attachments/assets/13664ee2-f698-431f-bcc3-b5cfbc01d5bd" />
 - La primera columna te da el nombre exacto del servicio

Paso 2.2. Localizar dónde se define dentro del portero
- Ejecuta: ``` sudo grep -rn -w 'NOMBRE' /etc/inetd.conf /etc/xinetd.conf /etc/xinetd.d/ 2>/dev/null ```
- <img width="1194" height="58" alt="image" src="https://github.com/user-attachments/assets/791356b7-ce1e-4912-89ed-80835d63c5d0" />
- Verificación: Muestra la ruta exacta y el número de línea donde se define el servicio. En este caso en inetd.conf esta comentada ya la línea, por lo que quien enciende el servicio es en xinetd. Además vemos que no se llama ftp sino vsftpd

PASO 3: Parar e impedir a la vez (en Torrent-Vulnerable):
- Si el resultado estaba en /etc/xinetd.d/NOMBRE:
   - Ejecuta: ``` sudo cp /etc/xinetd.d/vsftpd /root/vsftpd.bak```, ```sudo sed -i 's/disable.*/disable = yes/' /etc/xinetd.d/vsftpd ``` y ```sudo /etc/init.d/xinetd reload```
   - <img width="822" height="19" alt="image" src="https://github.com/user-attachments/assets/0f5e7330-ae83-48a7-abaf-973aeefb1bb8" />
   - <img width="957" height="22" alt="image" src="https://github.com/user-attachments/assets/4c484618-6d7c-4c2f-bb6e-ea5f29b00395" />
   - <img width="979" height="43" alt="image" src="https://github.com/user-attachments/assets/e96048c9-7d7f-43f1-b246-fb533cfa4475" />
   - Comprueba el cambio con: ```grep -H disable /etc/xinetd.d/SERVICIO```
   - <img width="740" height="40" alt="image" src="https://github.com/user-attachments/assets/86544611-2b3c-4e01-b142-d042272a3b3e" />

PASO 4: Comprobar (en Torrent-Auditor):
- Ejecuta:``` sudo nmap -p PUERTO 192.168.1.101```
- <img width="689" height="85" alt="image" src="https://github.com/user-attachments/assets/8704650d-781a-47d1-8207-7b34c540f00e" />
- Verifica que el puerto objetivo salga en estado "closed".

## puerto 53
PASO 1: Identifica (en Torrent-Vulnerable):
- Ejecuta: ``` sudo netstat -tulpn | grep ':53 ' ```
   - Verificación: En la columna derecha figura explícitamente named
- Comprueba si tiene script en /etc/init.d/ con: ``` ls /etc/init.d/ | grep named ```
- Comprobar si el nombre no coincide:
   - El método más confiable: ``` dpkg -S $(which named) ``` Te dice directamente a qué paquete y servicio pertenece
   - <img width="662" height="42" alt="image" src="https://github.com/user-attachments/assets/c2994349-24a9-4ac6-892e-5f2582c50786" />

PASO 2: Averiguar cómo arranca (en Torrent-Vulnerable):
- Ejecuta: ```ls /etc/init.d/ | grep bind9 ``` y ``` ls -l /etc/rc2.d/ | grep NOMBRE```
- <img width="729" height="83" alt="image" src="https://github.com/user-attachments/assets/6c96ea56-8a57-41c0-93e7-391e489ec66b" />
   - Verificación: existe el script en init.d y un enlace simbólico que empieza por S en rc2.d
 
PASO 3: Parar ahora (en Torrent-Vulnerable):
- Ejecuta: ``` sudo /etc/init.d/bind9 stop ```
- <img width="639" height="42" alt="image" src="https://github.com/user-attachments/assets/a5ddc0fc-2aab-4a44-ad94-4eaf82fc4c5d" />
   - Verificación: devuelve un mensaje de confirmación con [ OK ] y al repetir ``` sudo netstat -tulpn | grep ':53 ' ``` la salida queda vacía.

PASO 4: Impedir tras reiniciar (en Torrent-Vulnerable):
- Ejecuta:sudo update-rc.d -f bind9 remove
- <img width="675" height="224" alt="image" src="https://github.com/user-attachments/assets/c245b415-972a-47c2-8abd-8ef500b92bf9" />
   - Verificación: El comando informa de la retirada de enlaces. Al hacer ``` ls /etc/rc2.d/ | grep bind9 ``` ya no devuelva nada.

PASO 5: Comprobar (en Torrent-Auditor):
- Ejecuta: ``` sudo nmap -p PUERTO 192.168.1.101```
- <img width="654" height="89" alt="image" src="https://github.com/user-attachments/assets/1a8bb78e-2366-41e2-b875-aa1cba734f91" />+
   - Verificación: el estado del puerto figura como "cerrado" 

## Puerto 25
PASO 1: Identifica (en Torrent-Vulnerable):
- Ejecuta: ``` sudo netstat -tulpn | grep ':53 ' ```
   - Verificación: En la columna derecha figura explícitamente master
   - Comprueba si tiene script en /etc/init.d/ con: ``` ls /etc/init.d/ | grep master ```
   - Comprobar si el nombre no coincide: 
      - Opción A: (averiguar el paquete con dpkg):
        1. Prueba con: ``` dpkg -S $(which master) ```
        2. Si 'which' no devuelve nada: averigua su ruta por el PID con: ``` ls -l /proc/PID/exe ```. Y luego consulta el paquete: ```dpkg -S /ruta/obtenida```
           <img width="877" height="250" alt="image" src="https://github.com/user-attachments/assets/2786fb30-4416-42fc-ac91-f6a7b346ee6d" />

PASO 2: Averiguar cómo arranca (en Torrent-Vulnerable):
- Ejecuta: ``` ls /etc/init.d/ | grep postfix ``` y  ```ls -l /etc/rc2.d/ | grep postfix ``` 
- <img width="792" height="85" alt="image" src="https://github.com/user-attachments/assets/a6d80b3a-dc1f-4395-b74d-be19610e7185" />
   - Verificación: existe el script en init.dy un enlace simbólico que empieza por S en rc2.d

PASO 3: Parar ahora (en Torrent-Vulnerable):
- Ejecuta: ``` sudo /etc/init.d/postfix stop```
- <img width="975" height="43" alt="image" src="https://github.com/user-attachments/assets/352f4811-07af-4195-a15b-eed2c75178a0" />
   - Verificación: devuelve un mensaje de confirmación con [ OK ] y al repetir ``` sudo netstat -tulpn | grep ':25 ' ``` la salida queda vacía.

PASO 4: Impedir tras reiniciar (en Torrent-Vulnerable):
- Ejecuta: ``` sudo update-rc.d -f postfix remove ```
- <img width="709" height="176" alt="image" src="https://github.com/user-attachments/assets/00803d43-bb14-459d-933e-b063c8cbdd67" />
   - Verificación: El comando informa de la retirada de enlaces. Al hacer ``` ls /etc/rc2.d/ | grep postfix ``` ya no devuelva nada.

PASO 5: Comprobar (en Torrent-Auditor):
- Ejecuta: ``` sudo nmap -p 25 192.168.1.101 ```
- <img width="649" height="78" alt="image" src="https://github.com/user-attachments/assets/2ca38d07-fe7a-43a3-8027-d5e22ea53ef1" />
   - Verificación: el estado del puerto figura como "cerrado" (si pone filtrado solo está oculto con cortafuegos, no apagado).

## FINAL
<img width="967" height="112" alt="image" src="https://github.com/user-attachments/assets/f150d787-02a6-4675-bcf4-abc4c6fc51d3" />


#### Contramedida 2 — Bloquear el ping
PASO 1: Aplicar la regla de filtrado con iptables (En Torrent-Vulnerable)
- Ejecuta ```sudo iptables -A INPUT -p icmp --icmp-type echo-request -j DROP ```

PASO 2: Comprobar que la regla está activa (En Torrent-Vulnerable)
- Ejecuta: ``` sudo iptables -L INPUT -v -n ```
- <img width="1062" height="83" alt="image" src="https://github.com/user-attachments/assets/03bddc9a-2c0d-4c58-83c2-919787d409a1" />
- Verificación: Deberás ver una línea con target DROP, prot icmp, y en source/destination 0.0.0.0/0.

PASO 3:Comprobar desde fuera (En Torrent-Auditor)
- Ejecuta: ``` ping -c 3 192.168.1.101 ```
- <img width="696" height="117" alt="image" src="https://github.com/user-attachments/assets/e34a8a17-6bf2-4198-a4fe-dbee09907188" />
- Resultado esperado: No debe haber respuesta.

### B4. Medición DESPUÉS
En Torrent-Auditor:
``` sudo nmap -sV <IP> -oN despues.txt ```
``` diff antes.txt despues.txt ```

Rellenad la tabla:

Aspecto | Antes | Contramedida aplicada | Después
--------|-------|-----------------------|---------
Puertos abiertos | 22 puertos abiertos | Contramedida 1: parada de servicios permanentes con /etc/init.d/ (stop + update-rc.d remove), desactivación en /etc/xinetd.d/ (disable = yes) y neutralización de procesos persistentes | Únicamente 2 puertos abiertos
Servicios / versiones visibles | 22 servicios visibles con sus banners expuestos | Contramedida 1: apagado y deshabilitación selectiva de todos los demonios que no fueran Apache ni OpenSSH (conforme a la suposición de trabajo) | Solo 2 servicios visibles: ssh y Apache
Respuesta al ping | Responde con latencia visible (0% packet loss) | Contramedida 2: regla de filtrado en cortafuegos | No responde (100% packet loss)

<img width="837" height="744" alt="image" src="https://github.com/user-attachments/assets/215f8eb6-5acc-46ed-8044-dfb790810559" />
- Las líneas con el símbolo < corresponden a lo que había en antes.txt
- Las líneas con el símbolo > representan lo que ve nmap ahora en despues.txt

### B5. Preguntas finales
1. ¿Qué contramedida ha reducido más lo que ve el atacante? ¿Por qué?
Apagar y deshabilitar los servicios innecesarios porque elimina de raís la superficie del ataque. Como se puede comprobar en la Medición de Despues, el atacante pasó de ver 22 puertos abiertos con todos sus servicios y versiones a solo 2 puertos.
Sin embargo la contramedida 2 solo entorpece el descubrimiento inicial pero cualquier atacante que lance un escaneo nmap podría seguir descubriendo todos los servicios abiertos.

2. Si solo ocultáis un banner pero el servicio sigue activo, ¿la vulnerabilidad sigue ahí? Razonad la respuesta.
Si, la vulnerabilidad sigue presente. Porque ocultar el banner es seguridad por oscuridad. Aunque evita que un escáner como nmap clasifique a simple vista, el código vulnerable y sin parchear sigue en el servidor. Si un atacante lanza a ciegas un exploit el servidor lo procesará igual y funcionará con éxito, independientemente de si esta en el banner o no.

3. Torrent-Vulnerable usa un sistema operativo sin soporte desde hace años. ¿Puede una contramedida de estas sustituir a actualizarlo? ¿Qué haríais en una empresa real?
No, ninguna contramedida perimetral o de configuración sustituye la actualización del sistema operativo. Porque aunque se cierren todos los puertos que no se necesitan, los servicios que la empresa necesita (ssh y apache) continúan con versiones con fallos de seguridad.
En una empresa real habría que migrar y/o actualizar el servidor y las aplicaciones a distribuciones LTS.

5. De todas las contramedidas de las partes A y B, ¿cuáles evitan que el atacante encuentre información y cuáles solo ayudan a detectar que lo están intentando? (Aún no las hemos aplicado todas, pero pensad en cuáles serían de cada tipo.)
Evitar que encuentre información: apagar y deshabilitar servicios no necesarios, bloquear el ping, ocultar banners y tener cortafuegos en los puertos
Ayudar a detectar que lo intentan: la regla de registro de escaneos, inspección de registros del sistema y sistemas de detección de intrusos.


Teclado en español:
sudo loadkeys es
