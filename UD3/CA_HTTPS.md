# Certificados y HTTPS: de nuestra propia CA a Let’s Encrypt
## 0. Contexto y objetivos
El servidor de Torrent S.L. (el que habéis ido montando con usuarios, ACL, contraseñas y MFA) publica una página web interna con el estado de la planta. Ahora mismo se sirve por HTTP: cualquiera conectado a la wifi del estudio podría leer lo que viaja, o incluso modificarlo por el camino. La solución es HTTPS, y para que funcione hace falta un certificado digital
Un certificado responde a una pregunta: «¿cómo sé que este servidor es realmente quien dice ser?». La respuesta depende de quién lo haya firmado y de si tu navegador se fía de esa entidad. Hoy lo vais a ver de dos formas distintas sobre el mismo servidor Apache:
- Parte 1: montáis la autoridad de certificación (CA) de Torrent S.L. y firmáis vosotros el certificado del servidor
- Parte 2: pedís un certificado real y gratuito a Let’s Encrypt, una CA pública, y comparáis las dos situaciones.

## 0.1 Antes de empezar: qué necesitas
- Una máquina virtual Linux de servidor (Debian o Ubuntu Server) con usuario con permisos de sudo. Puedes usar Torrent-Auditor.
- Salida a Internet desde la VM, para instalar paquetes y, en la Parte 2, hablar con Let’s Encrypt y DuckDNS
- Poder llegar desde el navegador del equipo a la VM. Hay dos configuraciones válidas de VirtualBox:

Configuración | Cómo queda
---------------|----------
A. Un adaptador en modo puente | La VM recibe una IP de la red del aula, sale a Internet y el equipo llega directamente a ella. (NO SUELE FUNCIONAR EN EL AULA)
B. Dos adaptadores | Adaptador 1 en NAT (salida a Internet) y adaptador 2 en «Solo anfitrión» (para llegar desde el equipo). «Red interna» solo comunica máquinas virtuales entre sí, no sirve para llegar desde el navegador del equipo.
<img width="677" height="265" alt="image" src="https://github.com/user-attachments/assets/d1699f13-23d4-4a7b-af0d-955a75e384cd" />
<img width="572" height="195" alt="image" src="https://github.com/user-attachments/assets/9ae037c6-39ce-43a5-8d20-dfd7762c7fbb" />


Averigua qué IP usa tu equipo para llegar a la VM y compruébala. A partir de ahora la llamaremos IP_SERVIDOR (cámbiala por la tuya en todos los comandos):
- ```ip br -4 a```
  <img width="417" height="77" alt="image" src="https://github.com/user-attachments/assets/f2b0adfd-0ee1-4f72-84c8-726eb269e7c8" />

- ```ping -c 2 1.1.1.1```
- ```ping -c 2 debian.org```
 <img width="893" height="297" alt="image" src="https://github.com/user-attachments/assets/ba18d1f3-377c-43ae-b627-e015ab27d07c" />
 
  - Qué deberías ver: la IP de tu adaptador y respuestas en los dos ping. El primero prueba que hay salida a Internet; el segundo, que además funciona la resolución de nombres (DNS).

## PARTE 1. Nuestra propia autoridad de certificación
### 1.1 Cómo funciona
Una autoridad de certificación (CA) es una entidad de confianza que firma certificados. Un certificado de servidor contiene tres cosas: la clave pública del servidor, su identidad (nombre o IP) y la firma de la CA. Un navegador acepta el certificado si está firmado por una CA que ya figura en su lista de entidades de confianza

Vamos a crear cinco ficheros. Fíjate en quién puede ver cada uno, porque ahí está la seguridad de todo el sistema:

Fichero | Qué es | Quién debe tenerlo
-------|----------|-------------
ca.key | Clave privada de la CA | Solo la CA. Si se filtra, cualquiera puede emitir certificados falsos en nombre de Torrent s.l
ca.crt | Certificado de la CA (su clave pública) | Todos los clientes que deban confiar en ella
srv.key | Clave privada del servidor web | Solo el servidor
srv.csr | Petición de certificado | Se envía a la CA para que la firme
srv.crt | Certificado del servidor, firmado por la CA | Se publica: lo envía el servidor a cada cliente

### 1.2 Paso a paso
#### Paso 0 - Instalar Apache y comprobar que funciona (todavía sin cifrar)
Instalamos el servidor web, la herramienta openssl con la que crearemos todo, curl para hacer pruebas y cron (lo necesitaremos en la Parte 2).
- ``` sudo apt update ```
- ``` sudo apt install -y apache2 openssl curl cron ```

Comprueba que Apache está corriendo y escuchando en el puerto 80:
- ```systemctl status apache2 --no-pager ```
- ``` sudo ss -tlnp | grep ':80' ```
  <img width="873" height="385" alt="image" src="https://github.com/user-attachments/assets/4c03d2c5-5b1f-4403-a660-3efb855ea6aa" />

Prueba en el navegador: abre http://IP_SERVIDOR desde el navegador del equipo anfitrión. Debe mostrar la página por defecto de Apache sin cifrar.




  
