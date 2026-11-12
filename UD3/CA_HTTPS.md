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
<img width="881" height="276" alt="image" src="https://github.com/user-attachments/assets/c585b4bd-4d87-49d3-9e0d-6d50e431a0c6" />

#### Paso 1 — Crear una carpeta de trabajo para la CA
Separamos todo lo de la CA en su propia carpeta, lejos de la configuración de Apache. La subcarpeta private contendrá claves privadas, así que solo tu usuario puede entrar en ella (permisos 700).
- ```mkdir -p ~/ca/private ~/ca/certs```
- ```chmod 700 ~/ca/private```
- ```cd ~/ca```
  - Qué deberías ver: no hay salida. Comprueba con ls -ld private que los permisos son drwx------
  <img width="590" height="120" alt="image" src="https://github.com/user-attachments/assets/f85f4ad1-b7af-44dc-99ab-fa0acc046966" />

#### Paso 2 — Generar la clave privada de la CA
genrsa crea una clave privada RSA; 4096 bits es una longitud generosa porque esta clave va a vivir mucho tiempo.
La opción -aes256 cifra el fichero con una contraseña (passphrase): si alguien copia ca.key, no puede usarla sin esa frase. Cuidado: si la olvidas no se puede recuperar.
- ```openssl genrsa -aes256 -out private/ca.key 4096```
  - Qué deberías ver: te pide dos veces una frase de paso (Enter PEM pass phrase). Elige una larga y guárdala
    <img width="741" height="75" alt="image" src="https://github.com/user-attachments/assets/627428f5-b9e4-424f-972b-01b88160c332" />

- **¿Por qué conviene proteger con contraseña la clave privada de una CA aunque esté en un
directorio con permisos restrictivos?**
- Por el principio de defensa en profundidad. Los permisos 700 solo protegen frente a otros usuarios del sistema, si alguien tiene acceso con permisos de administración puede hacerse una copia pero sin la contraseña no podría usarla.

#### Paso 3 — Crear el certificado raíz de la CA (autofirmado)
Ahora la CA se presenta al mundo: crea su propio certificado y lo firma con su propia clave. Eso se llama certificado autofirmado: el emisor y el sujeto son la misma entidad. Es lo normal en una CA raíz, porque no hay nadie por encima que la avale.
- ```openssl req -x509 -new -key private/ca.key -sha256 -days 3650  -out certs/ca.crt -subj "/C=ES/O=Torrent S.L./CN=Torrent S.L. CA Raiz" -addext "basicConstraints=critical,CA:TRUE" -addext "keyUsage=critical,keyCertSign,cRLSign"```
  - Qué deberías ver: te pide la frase de paso del Paso 2 y termina sin mensajes.
  <img width="887" height="88" alt="image" src="https://github.com/user-attachments/assets/a9db8e92-b87a-4dfb-a5f3-12bb8b823c4f" />

- Qué significa cada opción:
  - req -x509: crea directamente un certificado (sin esta opción crearía solo una petición de certificado).
  - -days 3650: validez de 10 años. Las raíces duran mucho; los certificados de servidor, poco.
  - -subj: los datos de identidad (país, organización y nombre de la CA).
  - basicConstraints CA:TRUE: declara «este certificado pertenece a una CA y puede firmar otros».
  - keyUsage keyCertSign: limita para qué puede usarse la clave: firmar certificados

#### Paso 4 — Inspeccionar el certificado que acabas de crear
Un certificado es un documento legible. Comprueba que contiene lo que esperas.
- ```openssl x509 -in certs/ca.crt -noout -subject -issuer -dates```
  <img width="858" height="116" alt="image" src="https://github.com/user-attachments/assets/d705b4c1-ac7d-432c-90c6-69cf1eb21991" />

- ```openssl x509 -in certs/ca.crt -noout -text | grep -A1 'Basic Constraints'```
  <img width="878" height="95" alt="image" src="https://github.com/user-attachments/assets/ec61a7b1-3e5e-43cf-acb8-942ac36bab3b" />

  - Qué deberías ver: Subject e Issuer idénticos, las fechas de inicio y fin y la línea CA:TRUE.
- **¿Por qué coinciden Subject e Issuer? ¿Hasta qué fecha es válida tu CA?**
- Coinciden porque es un certificado autofirmado. Al ser una CA raíz, no hay otra entidad superior que la avale o la firma, por lo que el emisor y el sujeto son la misma entidad. Es apta hasta el 6 de noviembre de 2036 (notAfter).
- 
#### Paso 5 — Clave privada del servidor y petición de certificado (CSR)
El servidor necesita su propio par de claves. A diferencia de la CA, esta clave va sin contraseña, porque Apache tiene que poder arrancar solo sin que nadie teclee nada. Lo compensamos protegiendo el fichero con permisos.

Después creamos la CSR: es la petición que se le envía a la CA diciendo «aquí tienes mi clave pública y mi identidad, fírmame un certificado». La CSR nunca contiene la clave privada
- ```openssl genrsa -out private/srv.key 2048```
- ```openssl req -new -key private/srv.key -out srv.csr -subj```
  - Qué deberías ver: no hay mensajes de error y aparecen los ficheros private/srv.key y srv.csr (comprueba con ls)
  <img width="867" height="137" alt="image" src="https://github.com/user-attachments/assets/98b9f6c1-db72-4563-b2e2-cb2647aa74b9" />

#### Paso 6 — Indicar para qué nombres e IP es válido el certificado (SAN)
Los navegadores modernos ya no miran el nombre (CN) del certificado: comprueban el campo Subject Alternative Name (SAN). Como vas a entrar por IP, esa IP tiene que figurar ahí. Crea el fichero de extensiones y sustituye IP_SERVIDOR por la tuya:
- ```nano srv.ext```
Dentro del fichero escribe estas cuatro líneas y guarda
- ```subjectAltName = DNS:hmi.torrent.local, IP:IP_SERVIDOR```
- ```basicConstraints = CA:FALSE```
- ```keyUsage = digitalSignature, keyEncipherment```
- ```extendedKeyUsage = serverAuth```
  <img width="640" height="140" alt="image" src="https://github.com/user-attachments/assets/aced9d15-7479-4128-a936-755f9b5340ab" />

  - CA:FALSE: este certificado no puede usarse para firmar otros (solo la CA puede).
  - keyUsage / extendedKeyUsage serverAuth: limitan el uso de la clave a identificar a un servidor web.

#### Paso 7 — La CA firma el certificado del servidor
Este es el momento clave: la CA comprueba la petición y la firma con su clave privada. A partir de aquí, cualquiera que confíe en la CA puede confiar en el servidor. La opción -CAcreateserial asigna un número de serie único al certificado.
- ```openssl x509 -req -in srv.csr -CA certs/ca.crt -CAkey private/ca.key -CAcreateserial -out certs/srv.crt -days 365 -sha256 -extfile srv.ex ```
  - Qué deberías ver: te pide la frase de paso de la CA y muestra «Certificate request self-signature ok», el subject y «Getting CA Private Key»
  <img width="620" height="140" alt="image" src="https://github.com/user-attachments/assets/d5230dbc-29b8-46bc-9ba6-d8ed3ea64e8c" />

- **¿Por qué el certificado del servidor dura 365 días y el de la CA 10 años?**
- Porque la clave privada del servidor web (srv.key) está sin contraseña en un equipo conectado a la red y expuesto a internet. Si la clave del servidor se ve comprometida, el atacante solo podrá usarla 1 año. Y porque el certificado de raíz de la CA (ca.crt) debe instalarse manualmente en los clientes. Si la CA caducase cada año, habría que redistribuir el certificado en todos los equipos, sin embargo los certificados del servidor se renuevan sin tener que tocar a los clientes.

#### Paso 8 — Verificar la cadena de confianza
openssl puede comprobar que el certificado del servidor está firmado por la CA indicada.
- ```openssl verify -CAfile certs/ca.crt certs/srv.crt```
- ```openssl x509 -in certs/srv.crt -noout -subject -issuer -ext subjectAltNam```
  - Qué deberías ver: certs/srv.crt: OK. Ahora Subject e Issuer son distintos, y en el SAN aparecen el nombre y tu IP.
  <img width="888" height="185" alt="image" src="https://github.com/user-attachments/assets/703d2f4a-7af3-4d4d-9b04-eeca6ff96214" />

- **Anota: la salida de los dos comandos**

#### Paso 9 — Instalar el certificado en Apache
Copiamos el certificado y la clave a los directorios estándar del sistema y activamos el módulo SSL de Apache. La clave solo debe poder leerla root (Apache la lee al arrancar, antes de cambiar de usuario).
1. Copia el certificado y la clave privada a las rutas estándar de Ubuntu y asegura la clave para que solo la lea root:
- ```sudo cp ~/ca/certs/srv.crt /etc/ssl/certs/torrent-srv.crt```
- ```sudo cp ~/ca/private/srv.key /etc/ssl/private/torrent-srv.key```
- ```sudo chmod 600 /etc/ssl/private/torrent-srv.key```
  - Verification: Ejecuta ls -l /etc/ssl/private/torrent-srv.key. Debe mostrar permisos -rw------- con propietario root.
  <img width="746" height="47" alt="image" src="https://github.com/user-attachments/assets/c124d094-cb66-48ce-b991-68176a618291" />


2. Activar el módulo SSL en Apache
- ```sudo a2enmod ssl```
  - Verification: La terminal indicará Enabling module ssl (o avisará de que ya está activado).
  <img width="885" height="222" alt="image" src="https://github.com/user-attachments/assets/150daa05-5a3f-4dd0-ad17-2419293356a3" />

3. Modificar default-ssl.conf
- ```sudo nano /etc/apache2/sites-available/default-ssl.conf```

Baja hasta localizar las dos líneas que apuntan al certificado de pruebas (snakeoil). Sustitúyelas para que apunten a tus ficheros:
- ```SSLCertificateFile /etc/ssl/certs/torrent-srv.crt```
- ```SSLCertificateKeyFile /etc/ssl/private/torrent-srv.key```
  <img width="832" height="147" alt="image" src="https://github.com/user-attachments/assets/876d9749-1bab-4dc4-8db0-4240550cf322" />

  - Verification: Ejecuta ```grep -E 'SSLCertificateFile|SSLCertificateKeyFile' /etc/apache2/sites-available/default-ssl.conf ``` y comprueba que ambas líneas apuntan a torrent-srv.crt y torrent-srv.key.
  <img width="878" height="115" alt="image" src="https://github.com/user-attachments/assets/832c3ecf-4574-404b-af36-5a2caba62c91" />


4. Habilitar el sitio y reiniciar Apache
Activa el sitio HTTPS, comprueba la sintaxis y reinicia el servicio Apache (al activar un módulo nuevo es imprescindible reiniciar completo):
- ```sudo a2ensite default-ssl```
- ```sudo apache2ctl configtest```
- ```sudo systemctl restart apache2```
- ```sudo ss -tlnp | grep ':443'```
  - Verification: configtest debe devolver Syntax OK (los avisos sobre ServerName son inofensivos) y ss -tlnp debe mostrar un proceso apache2 escuchando en el puerto 443.
  <img width="881" height="292" alt="image" src="https://github.com/user-attachments/assets/b4f4122a-f338-4952-a687-320778177779" />

#### Paso 10 — Probar HTTPS desde la propia VM, antes de confiar en la CA
1. Prueba contra el almacén de confianza del sistema (falla esperada)
- ```curl -I https://192.168.1.100```
  <img width="807" height="65" alt="image" src="https://github.com/user-attachments/assets/ffba0893-b9b7-496f-9a82-f0257318b2a8" />

  - Resultado esperado: Devuelve un error

2. Prueba indicando manualmente nuestra CA (éxito)
Ahora se le indica a curl que confíe específicamente en ca.crt para esta consulta con el parámetro --cacert:
- ```curl -I --cacert ~/ca/certs/ca.crt https://192.168.1.100 ```
  <img width="836" height="93" alt="image" src="https://github.com/user-attachments/assets/7c5363d3-db5c-4f21-a62e-4fc23ea8c182" />

  - Resultado esperado: La cabecera HTTP devuelta con éxito

- **¿Por qué el primer comando falla y el segundo funciona?**
- curl utiliza por defecto el almacén global de certificados del sistema, donde solo figuran las CA públicas reconocidas. En el primer intento, el servidor presenta un certificado emitido por una entidad privada (nosotros), lo que provoca el rechazo por emisor no confiable. En el segundo intento, la opción --cacert entrega temporalmente la clave pública de la CA a curl, lo que le permite validar la firma del certificado del servidor y completar la cadena de confianza con éxito.

#### Paso 11 — El navegador: advertencia, huella e importación de la CA
1. Ver la advertencia inicial en Firefox
- Desde el navegador de tu ordenador físico, accede a: https://192.168.1.100
  <img width="892" height="362" alt="image" src="https://github.com/user-attachments/assets/e61dbbae-ec54-4fd2-b42c-c8e4266e0623" />

  - Verification: El navegador bloqueará el paso mostrando una advertencia de seguridad con el código de error (el navegador no tiene en su almacén a Torrent S.L. CA Raiz).

2. Publicar ca.crt y calcular su huella SHA-256 en la VM
En la terminal de tu máquina virtual, copia el certificado de la CA a la raíz web de Apache y calcula su huella digital criptográfica:
- ```sudo cp ~/ca/certs/ca.crt /var/www/html/torrent-ca.crt```
- ```openssl x509 -in ~/ca/certs/ca.crt -noout -fingerprint -sha256```
  <img width="890" height="100" alt="image" src="https://github.com/user-attachments/assets/08826d4e-9328-44c7-b72a-0be3266f57f2" />

  - Verification: La terminal imprimirá la huella digital (SHA256 Fingerprint=XX:XX:XX...). Déjala visible en la pantalla.
 
3. Descargar e importar la CA comprobando la huella
- En Firefox del equipo físico abre: http://192.168.1.100/torrent-ca.crt
- Firefox abrirá una ventana para importar la CA.
- Punto crítico: Antes de marcar ninguna casilla, haz clic en «Ver certificado» y compara la huella SHA-256 que te muestra Firefox con la que calculó OpenSSL en tu terminal.
- Si coinciden exactamente, marca la casilla «Confiar en esta CA para identificar sitios web» y pulsa en Aceptar.
  - Verification: El certificado Torrent S.L. CA Raiz quedará registrado en la lista de Autoridades de Firefox.
 
4. Recargar y comprobar el candado seguro
Vuelve a cargar en Firefox la dirección https://192.168.1.100
  <img width="640" height="92" alt="image" src="https://github.com/user-attachments/assets/6c0059c3-960f-4f75-9901-c9d0e8a37fb1" />

  - Verification: La advertencia desaparece por completo y aparece el icono del candado sin avisos. Al pulsar en el candado y ver los detalles del certificado, verás que el emisor (Issuer) es Torrent S.L. CA Raiz.

#### Paso 12 — Mirar la conexión TLS por dentro
1. Ejecutar openssl s_client con la CA
- ```openssl s_client -connect 192.168.1.100:443 -CAfile ~/ca/certs/ca.crt </dev/null 2>/dev/null | grep -E 's:|i:|Verify return code|Cipher' ```
  <img width="875" height="142" alt="image" src="https://github.com/user-attachments/assets/ce50110e-ac8d-4815-8b8d-387be08de824" />

  - Verification: La salida debe mostrar 4 líneas

2. Anotar en el informe (Versión y Cifrado)
- ```openssl s_client -connect 192.168.1.100:443 -CAfile ~/ca/certs/ca.crt </dev/null 2>/dev/null | grep -E 'Protocol|Cipher'```
  <img width="882" height="76" alt="image" src="https://github.com/user-attachments/assets/920d444b-ab46-459b-b2f9-9b46c47d5aca" />

  - Verification: Te mostrará Protocol  : TLSv1.3 (o TLSv1.2) y la suite de cifrado (Cipher). Anota ambos datos en tu informe.
  
3. Hacer el experimento del apartado Investiga
- ```openssl s_client -connect 192.168.1.100:443 </dev/null 2>/dev/null | grep -E 'Verify return code'```
  <img width="886" height="70" alt="image" src="https://github.com/user-attachments/assets/3bf552a4-0935-4639-8707-9697e32df88e" />
  
  - Verification: Ahora el código cambia de 0 (ok) a un fallo de verificación

- **Anota en tu informe: qué versión de TLS y qué cifrado se han negociado.**
  - Versión: TLSv1.3
  - Cifrado (Cipher): TLS_AES_256_GCM_SHA384

- **Investiga: ¿Qué pasaría si en el Paso 12 quitaras la opción -CAfile? Pruébalo y explica el valor de «Verify return code» que obtienes**
- Al quitar -CAfile, OpenSSL recurre exclusivamente al almacén de entidades de confianza del sistema operativo (/etc/ssl/certs). Al no estar instalada la CA privada de Torrent S.L. en ese almacén, no puede construir la cadena de confianza y la verificación criptográfica falla.

#### Paso 13 — Guardar lo hecho para la Fase 2
La CA que acabas de crear se va a reutilizar para emitir certificados de la VPN. Haz una copia de seguridad de la carpeta (sin subirla a ningún sitio) y, si quieres, un snapshot de la VM en VirtualBox.
- ```tar czf ~/ca-torrent-backup.tar.gz -C ~ ca```
  <img width="846" height="75" alt="image" src="https://github.com/user-attachments/assets/f3155e81-7a52-4cf5-82fb-eef3f0cc0642" />

  - Verificación: Ejecuta ls -lh ~/ca-torrent-backup.tar.gz. Comprueba que el archivo comprimido existe en tu directorio personal.

### 1.3 Preguntas de la Parte 1
1. Explica con tus palabras la diferencia entre ca.key, ca.crt, srv.key y srv.crt: ¿cuáles son secretos y cuáles públicos?
- ca.key es la clave privad de nuestra CA. Se utiliza para firmar los certificados que emitimos. Debe guardarse protegida con contraseña y nunca debe salir de CA.
- svr.key: es la clave privada del servidor web. El servidor la usa para cifrar y descifrar las conexiones TLS que llegan de los clientes. Se guarda sin contraseña para que Aapache pueda arrancar solo, pero con permisos estrictos de lectura exclusiva para root.
- ca.crt: es el certificado raíz de la CA, contiene su clave pública y sus datos identificativos. Se reparte e instala en el almacén de confianza de los navegadores para que confíen en los certificados que firme nuestra CA.
- srv.crt: es el certificado digital del servidor web, ya firmado por la CA. Apache lo envía en abierto a cualquier cliente que se conecte por HTTTPS para demostrar su identidad y entregarle su clave pública.

2. Alguien roba ca.key (sin contraseña). ¿Qué puede hacer con ella? ¿Qué tendría que hacer Torrent S.L. para recuperar la confianza?
- Al tener una clave privada raíz sin contraseña, puede generar y firmar certificados validos para cualquier dominio o servicio haciéndose pasar por Torrent S.L. Como los equipos de la empresa ya confían en esa CA, el atacante puede montar servidores falsos y realizar ataques sin que los navegadores muestren ninguna advertencia de seguridad, pudiendo interceptar credenciales y tráfico interno.
- La CA actual queda completamente comprometida y debe anularse. Primero debe eliminar y revocar la clave y el certificado raíz comprometidos. Después debe crear desde cero una nueva CA, desinstalar manualmente el certificado ca.crt del almacén de confianza de loa navegadores y equipos e instalar el nuevo ca.crt en los clientes y volver a emitir e instalar los certificados de los servidores.

3. ¿Por qué descargar ca.crt por HTTP y no verificar la huella es un riesgo? ¿Quién podría aprovecharlo?
- Porque el protocolo HTTP viaja en texto plano, sin cifrado ni mecanismos que garanticen la integridad del archivo. Si importamos el certificado en el navegadore sin comprobar antes su huella por un canal seguro no tenemos garantía de que el fichero recibido sea el original.
- Podría aprovecharlo cualquiera situado en la misma red o en el camino de la conexión. El atacante podría interceptar la descarga y sustituir ca.crt por su propio certificado raíz; una vez instalado en el navegador de la víctima, podría suplantar y descifrar cualquier web que el usuario visite.

4. ¿Es una buena idea que la CA y el servidor web estén en la misma máquina? ¿Qué problema de seguridad crea?
- No, es una mala práctica se seguridad. Porque rompe el principio de aislamiento de servicios. El servidor web está expuesto a la red y a posibles vulnerabilidades en sus aplicaciones. Si un atacante logra explotar un fallo en la web y escalar privilegios hasta el root del equipo, tendrá acceso directo a ca.key, comprometiendo toda la infraestructura de la empresa. En un entorno real, la CA raíz debe mantenerse aislada en una máquina independiente y desconectada de la red.

##  PARTE 2 Un certificado real y gratuito con Let’s Encrypt
### 2.1 Cómo funciona esto
Let’s Encrypt es una autoridad de certificación pública, gratuita y automática. Su certificado raíz ya viene instalado en todos los navegadores y sistemas, y por eso, a diferencia de la CA de Torrent S.L., los navegadores confían en lo que firma sin que tengas que importar nada.

Desafío | Cómo funciona | Sirve en nuestras VM?
--------|---------------|-----------------------
HTTP-01 | Let’s Encrypt se conecta a tu servidor web desde Internet y busca un fichero concreto. | No: las VM están en la red del aula y no son accesibles desde fuera. 
DNS-01 | Tú creas un registro DNS de tipo TXT con un valor que te da Let’s Encrypt. Si lo encuentra, demuestras que controlas el dominio. | Sí: no hace falta que el servidor sea accesible desde Internet

- Usaremos DNS-01 con un subdominio gratuito de DuckDNS y la herramienta acme.sh, que automatiza el proceso.

### 2.2 Paso a paso
#### Paso 1 — Crear tu subdominio en DuckDNS
1. Iniciar sesión en DuckDNS
Entra en la web oficial: https://www.duckdns.org e inicia sesión (por ejemplo, con tu cuenta de GitHub o Google).
  - Verification: Accederás al panel de control donde en la parte superior verás tu token (una cadena alfanumérica larga).

2. Registrar el subdominio y apuntar la IP
- En el recuadro sub domains, escribe un nombre único en minúsculas y sin espacios (torrent-samaniego). Pulsa en add domain.
  <img width="877" height="151" alt="image" src="https://github.com/user-attachments/assets/7a150b33-1191-4d10-9ce8-ddc27059ad24" />

- En la lista de dominios, en el campo current ip, escribe tu IP_SERVIDOR (192.168.1.100).
  <img width="787" height="447" alt="image" src="https://github.com/user-attachments/assets/4de9c2b8-7eed-46b5-83d4-a864fbcfcc4d" />

- Haz clic en el botón update ip.

3. Comprobar la resolución DNS
Vuelve a la terminal de tu máquina virtual y comprueba que el dominio resuelve a tu IP local:
- ```getent hosts torrent-samaniego.duckdns.org```
  <img width="707" height="53" alt="image" src="https://github.com/user-attachments/assets/d58a0d4f-a0c1-464a-81f4-133896cd114c" />

  - Qué deberías ver: La IP 192.168.1.100 seguida de torrent-samaniego.duckdns.org. Eso confirma que el DNS público ya apunta a tu servidor

#### Paso 2 — Instalar acme.sh (leyendo antes lo que se va a ejecutar)
Una vez confirmada la resolución DNS, pasamos a instalar la herramienta acme.sh, que se encargará de comunicarse con Let's Encrypt. Como pide el guion, entraremos como root e inspeccionaremos el script antes de ejecutarlo:
1. Pasar a root y descargar el instalador
- ``` sudo -i ```
- ```curl -o get-acme.sh https://get.acme.sh```
  <img width="811" height="201" alt="image" src="https://github.com/user-attachments/assets/384d08fa-4abb-49d8-92aa-b532041785a9" />

  - Verification: El archivo get-acme.sh aparecerá guardado en el directorio actual (ls -l get-acme.sh).

2. Inspeccionar el script
Abre el script para ver su código fuente:
- ```less get-acme.sh```

- Anota en tu informe (Paso 2): Dos cosas que hace el script de instalación: clona/descarga el repositorio desde GitHub a ~/.acme.sh y añade una tarea programada en cron para la renovación automática

3. Ejecutar la instalación con tu correo
Lanza el script indicando una dirección de correo válida
- ```sh get-acme.sh email=TU_CORREO@ejemplo.com```
  <img width="882" height="380" alt="image" src="https://github.com/user-attachments/assets/10180424-1ad4-473d-a320-5cec056497ed" />

#### Paso 3 — Elegir Let’s Encrypt como autoridad
1. Recargar el entorno para disponer del alias acme.sh:
- ```source ~/.bashrc```

2. Establecer Let's Encrypt como CA predeterminada:
- ```~/.acme.sh/acme.sh --set-default-ca --server letsencrypt```
  <img width="882" height="62" alt="image" src="https://github.com/user-attachments/assets/bfd6a7e8-f2e9-4dca-9b5c-e621038a7f71" />

  - Verificación: Si el comando finaliza sin errores y muestra la URL del directorio de Let's Encrypt (acme-v02.api.letsencrypt.org), la CA queda fijada de forma permanente para todas las solicitudes posteriores.

- **¿Por qué se ejecuta este comando?**
- Las versiones modernas de acme.sh apuntan por defecto a ZeroSSL. Al ejecutar --set-default-ca --server letsencrypt, reconfiguramos el cliente ACME para que negocie la emisión y renovación contra los servidores públicos y gratuitos de Let's Encrypt.

#### Paso 4: Configurar el token de DuckDNS y solicitar el certificado
1. Exportar el token de DuckDNS en memoria
- ```export DuckDNS_Token="PEGA_AQUI_TU_TOKEN_DE_DUCKDNS"```
  <img width="822" height="79" alt="image" src="https://github.com/user-attachments/assets/67102210-447c-4a83-9b30-6ecaab52c18f" />

  - Verificación: Ejecuta ```echo $DuckDNS_Token``` y confirma que imprime exactamente la cadena alfanumérica sin espacios ni comillas extra.
 
2. Emitir el certificado usando el desafío DNS-01
Solicita a Let's Encrypt el certificado para tu subdominio torrent-samaniego.duckdns.org indicando el plugin dns_duckdns:
- ```~/.acme.sh/acme.sh --issue --dns dns_duckdns -d torrent-samaniego.duckdns.org```
  <img width="891" height="155" alt="image" src="https://github.com/user-attachments/assets/c306e1ba-fc8a-4d28-b6ef-dc8d055ebd34" />

  - Verificación: El cliente acme.sh creará automáticamente un registro TXT (_acme-challenge.torrent-samaniego.duckdns.org) en DuckDNS, esperará unos segundos a que se propague y descargará los ficheros del certificado. El proceso finalizará mostrando en verde Cert success. junto con las rutas donde se han guardado fullchain.cer y torrent-samaniego.duckdns.org.key.

- **Qué ocurre por debajo**
- Al usar el desafío --dns dns_duckdns: No necesitas abrir puertos en el router hacia Internet: Let's Encrypt no intenta conectarse a tu IP privada (192.168.1.100), sino que consulta los servidores DNS públicos de DuckDNS para comprobar que controlas el dominio mediante el registro TXT temporal creado con tu token. A diferencia del certificado autofirmado de la Parte 1, este certificado está firmado por una Autoridad de Certificación pública reconocida universalmente (Let's Encrypt / ISRG Root X1).

#### Paso 5: Instalar el certificado de Let's Encrypt en Apache
1. Instalar los ficheros con --install-cert
- ```~/.acme.sh/acme.sh --install-cert -d torrent-samaniego.duckdns.org --key-file       /etc/ssl/private/le-torrent.key --fullchain-file /etc/ssl/certs/le-torrent-fullchain.crt --reloadcmd     "systemctl reload apache2" ```
  <img width="885" height="223" alt="image" src="https://github.com/user-attachments/assets/13047779-e5cc-421e-9c02-a0fc4f19b33e" />

  - Verificación: La consola mostrará mensajes indicando la instalación de la clave, del certificado completoy la ejecución con éxito del comando de recarga

2. Apuntar Apache al nuevo certificado en default-ssl.conf
Abre el fichero de configuración SSL de Apache:
- ```nano /etc/apache2/sites-available/default-ssl.conf```
Localiza las líneas SSLCertificateFile y SSLCertificateKeyFile (que actualmente apuntan a torrent-srv.crt y torrent-srv.key) y cámbialas para que apunten a los ficheros de Let's Encrypt:
- SSLCertificateFile /etc/ssl/certs/le-torrent-fullchain.crt
- SSLCertificateKeyFile /etc/ssl/private/le-torrent.key
  <img width="871" height="136" alt="image" src="https://github.com/user-attachments/assets/17e70cb6-bdf4-4c08-96f2-f491606fc148" />

3. Validar sintaxis y recargar Apache
Comprueba la configuración y recarga el servidor web:
- ```apache2ctl configtest```
- ```systemctl reload apache2```
  <img width="862" height="150" alt="image" src="https://github.com/user-attachments/assets/f6bab2fd-86d0-40ad-bc93-4cfaed3459c9" />

  - Verificación: configtest debe devolver Syntax OK y systemctl reload apache2 finalizará en silencio sin errores.

- **Por qué usamos --fullchain-file y --install-cert**
  - --fullchain-file: Incluye tanto tu certificado del servidor (torrent-samaniego.duckdns.org) como el certificado intermedio de Let's Encrypt (R10 / R11). Sin la cadena completa, muchos clientes y navegadores no pueden enlazar tu certificado hasta la raíz pública (ISRG Root X1)
  - --install-cert: Nunca se deben copiar a mano los ficheros desde ~/.acme.sh/. Al usar --install-cert, acme.sh guarda en su configuración interna dónde están instalados y qué comando ejecutar (--reloadcmd), automatizando todo el ciclo cuando el cron renueve el certificado a los 60 días.

#### Paso 6: Comprobar el certificado real
1. Comprobar el candado en Firefox 
- Abre en el navegador de tu ordenador físico la URL exacta de tu subdominio (recuerda usar el nombre de dominio, no la IP): https://torrent-samaniego.duckdns.org
  <img width="897" height="195" alt="image" src="https://github.com/user-attachments/assets/a6caaa86-a6ff-463f-a2a5-d6b117452f9e" />

  - Verificación: La página carga directamente con el candado verde/seguro sin ninguna advertencia, y al revisar los detalles del certificado aparece emitido por Let's Encrypt (R10 / R11).

2. Verificar la cadena completa con OpenSS
- ```openssl s_client -connect torrent-samaniego.duckdns.org:443 -servername torrent-samaniego.duckdns.org </dev/null 2>/dev/null | grep -E 's:|i:|Verify return code'```
  <img width="887" height="260" alt="image" src="https://github.com/user-attachments/assets/4b43a5cf-ee21-4d1f-95c4-dffc17f546ef" />

  - Verificación: Devolverá Verify return code: 0 (ok) sin necesidad de pasar -CAfile, demostrando que el sistema ya confía de forma nativa en la cadena de Let's Encrypt.

3. Verificar la renovación automática en cron
- ``` crontab -l | grep acme.sh```
  <img width="875" height="50" alt="image" src="https://github.com/user-attachments/assets/b6654502-2e37-4b09-bb5e-8beb571d4cad" />

  - Verificación: Aparecerá una línea programada que ejecuta /root/.acme.sh/acme.sh --cron automáticamente cada día para renovar el certificado antes de que caduque.
 
#### Preguntas y tabla comparativa
1. ¿Por qué los certificados de Let's Encrypt duran solo 90 días (frente a los 365 de la Parte 1)?
- Porque reducir la vida útil de 90 días limita el tiempo de exposición si la clave privada del servidor llega a ser robada o comprometida, y a la vez obliga a los administradores a automatizar todo el ciclo de renovación en vez de depender de tareas manuales propensas a olvidos.
  
2. ¿Por qué Firefox y OpenSSL confían en este certificado sin haber importado nada previamente?
- Porque Let's Encrypt es una CA pública cuya entidad raíz ya viene preinstalada dentro de los almacenes de confianza de los navegadores y sistemas operativos modernos.

3. ¿Qué ventaja tiene usar el desafío DNS-01 (--dns dns_duckdns) frente al desafío HTTP-01?
El reto DNI-01 valida la propiedad del dominio creando un registro temporal TST en los servidores públicos de DucksDNS. Esto permite emitir certificados reales para máquinas internas con IP privada sin tener que abrir los puertos 80 o 443 en el router.

**Tabla resumen**

Aspecto | Parte 1 (CA propia/OpenSSL) | Parte 2 (Let`s Encrypt/acme.sh)
--------|-----------------------------|-------------------------------
Emisor (Issuer) | Torrent S.L. CA Raiz | Let's Encrypt (R10/R11, pública)
Confianza en clientes | Manual, hay que importar ca.crt en cada navegador | Automática, está preinstalada de serie en todos los navegadores
Identificador válido | IP (192.168.1.100) y dominio local (hmi.torrent.local) | Nombre de dominio público (torrent-samaniego.duckdns.org)
Duración y renovación | 365 días, renovación manual firmando otro CSR | 90 días, renovación automática cada 60 días con crontab


## Parte 3: 





