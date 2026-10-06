# Laboratorio fases 1 y 2 — Estudio Torrent

## A1. whois y nslookup
No es necesario

## B. Antes y después
### B1. Preparación
1. Arrancad Torrent-Auditor y Torrent-Vulnerable.
2. Antes de tocar nada, haced una instantánea de Torrent-Vulnerable (en VirtualBox: menú Máquina → Tomar instantánea). Vais a modificarla y luego la necesitaremos tal como estaba.
3. Anotad la IP de Torrent-Vulnerable (ip a dentro de ella).
4. Desde Torrent-Auditor, comprobad que la veis: ping -c 3 <IP>.

### B2. Medición ANTES
En Torrent-Auditor: sudo nmap -sV <IP> -oN antes.txt
Apuntad: 
- Cuántos puertos abiertos hay.
- Qué servicios y versiones aparecen.
- Si el ping responde.

### B3. Contramedidas (en Torrent-Vulnerable, con sudo)
Usuario msfadmin, contraseña msfadmin.

Suposición de trabajo: este servidor solo tiene que ofrecer servicio web (puerto 80) y administración remota por SSH (puerto 22). Todo lo demás sobra.

#### Contramedida 1 — Inventariar y apagar servicios innecesarios
1. Listad lo que escucha: sudo netstat -tulpn

2. Haced una tabla: puerto, servicio, ¿es necesario según la suposición de trabajo?, decisión.
Puerto | Servicio | Es necesario | Decisión
-------|----------|--------------|----------


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
