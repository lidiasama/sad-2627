# Ataques de suplantación de identidad

## 1. SMTP spoofing

### Qué es
Es un correo de suplantación de identidad, que emplea ataques de spam o phising.
Cambian los encabezados imitándolos, intentando dar confianza para poder instalar malware o recibir información confidencial. 
Y sacan info facilmente.
Esto ocurre porque pueden cambiar el remitente al que responder

### Cómo se lleva a cabo
Se lleva a cabo porque el protocolo smtp fue diseñado sin mecanismo de autenticidad por lo que pueden cambiar la firma, encabezado y destinatario.

### Qué categoría(s) de amenaza compromete
(Confidencialidad / Integridad / Autenticidad / Disponibilidad — justificad)
Autenticidad al modificar el remitente.
Integridad y disponibilidad en cascada

### Ejemplo o caso real
100 mill de dolares a facebook y google, un atacante uso este metodo para enviar correos como proveedor para solicitar dinero.

### Medida de prevención
Los protocolos son SPF, DKIM, DMARC

### Fuente
GRUPO 4


## 2. DNS spoofing

### Qué es
Ciberataque donde se alteran los registros y el caché de los servidores DNS para redirigir el trafico a webs falsas o malisiosas.

### Cómo se lleva a cabo
Modifica los registros y el caché de los servidores DNS, cambiando las ip para interceptar datos

### Qué categoría(s) de amenaza compromete
Las 3. 

### Ejemplo o caso real
2015, se lanzó un ataque dns contra una aerolínea para acceder en vez de la web de los vuelos para ver tu vuelo, te daba un error 404 y una foto de un lagarto.

### Medida de prevención
Servicio dnssec con firma pictográfica.
https como cifrado de la conexión

### Fuente
GRUPO 2

## 3. IP spoofing

### Qué es
Es una técnica que consiste en falsificar la dirección IP de origen en los paquetes de datos para hacer creer que provienen de una fuente confiable o de otro sistema.

### Cómo se lleva a cabo
1. Todo paquete de red tiene una cabecera con la IP de origen y la de destino.
2. El ataque modifica el campo de origen y pone otra IP (la de otro equipo, una inventada o la d la propia víctima).
3. El destino recibe el paquete y cree que viene de esa IP falsa.
4. Las respuestas se envían a la IP falsificada, no al atacante. Por eso, el spoofing suele usarse cuando el atacante no necesita recibir la respueta.

### Qué categoría(s) de amenaza compromete
Vulnera principalmente la Autenticidad, de forma derivada la Integridad y Disponibilidad
El enfoque es engañar al receptor, suplantando la identidad de otro emisor, permitiendo inyectar código bajo el pseudónimo de otro emisor considerado seguro, destruyendo por completo la garantía del origen, ergo, rompe la Autenticidad. 

### Ejemplo o caso real
Un caso famoso de las primeras veces que se aplicó esta técnica fue el de Kevin Mitnick en 1994. Este quería acceder ilegalmente al ordenador del experto en seguridad Tsutomu Shimomura. Para realizarlo, falsificó la dirección IP de una máquina de confianza que el sistema del atacado reconociera. Dado que el ataque parecía provenir de una dirección IP conocida y confiable, el sistema objetivo no solicitó una contraseña. Kevin no podría recibir respuestas, puesto que iban dirigidas a la máquina real, pero adivinó los códigos de confirmación para completar el protocolo de enlace y tener acceso unidireccional. Esto le permitió extraer datos y acaparar titulares, demostrando los riesgos de confiar únicamente en las direcciones IP.

### Medida de prevención
Se puede prevenir este ataque de varias formas, entre ellos:
**Filtrado de paquetes:**Evalúa la cabecera de cada paquete IP (analizando aspectos como la dirección de origen, destino y puertos) para determinar si permite o bloquea el tráfico entrante o saliente según las reglas establecidas. Si un paquete incumple las reglas o muestra inconsistencia en sus datos, el firewall o dispositivo de red lo descarta.
**Autenticación mediante infraestructura de clave pública:** Usa un cifrado asimétrico, una clave privada para cifrar y autenticar, y una pública para descifrar. Impide que terceros deduzcan la clave privada, lo que permite verificar con seguridad a usuarios y dispositivos frente a ataques de suplantación.
**Supervisión de redes y Firewalls:** Detecta de forma temprana actividades sospechosas para mitigar daños, aunque la suplantación de IP intente ocultarlas. El firewall autentica direcciones IP y filtra el tráfico potencialmente malicioso para evitar accesos no autorizados.
**Formación en materia de seguridad:**Enseñar a los usuarios a evitar trampas como enlaces sospechosos para mitigar los datos de la suplantación de IP. 

### Fuente
https://securityjournalamericas.com/ip-spoofing/#Kevin_Mitnicks_Hack_1994 

## 4. Captura de cuentas de usuario y contraseñas

### Qué es
Vulnerabilidad activa que te coge las credenciales aprovechando fallos de seguridad como contraseñas escritas.

### Cómo se lleva a cabo
Con herramientas de trafico de red que capturan que pasa. 
Técnicas como el Phising.
Te quitan información sin que lo sepas.
Búsquedas en bbdd y diccionarios también.

### Qué categoría(s) de amenaza compromete
Intercepción. Se capturan datos mientras que están enviándose entre el punto a y punto b
Suplantación de identidad también
Confidencialidad e y potencialmente integridad y disponibilidad dependiendo de lo que se haga con el acceso.

### Ejemplo o caso real
Paso en oct de 2025, mayor exposición de datos de la histgoria.
30 conjuntos de datos 16 millones de datos. Muchas empresas colmo netflix, paypal fueron afectadas.

### Medida de prevención
Una medida de prevención es la doble autentificación.

### Fuente
Grupo 3


## Aplicado a Estudio Torrent
