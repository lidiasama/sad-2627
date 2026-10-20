# Atacando y defendiendo Estudio Técnico Torrent SL

Abrid el documento "Datos públicos Estudio Técnico Torrent S.L.". Contiene seis fuentes públicas, todas ficticias, sobre la empresa. Vosotros sois el atacante que acaba de encontrarlas.
## Paso 1. Leed las seis fuentes con ojos de atacante y encontrad al menos 10 datos que os servirían para atacar a la empresa. Anotadlos en esta tabla:
Dato encontrado | Fuente (1-6) | ¿Para qué le sirve al atacante? | Contramedida propuesta
----------------|--------------|---------------------------------|------------------------
correos electrónicos | 1 y 3 | Para atacar por SMTP spoofing o phishing | Implementar filtros antispam/antiphishing
tienen ubuntu 8.04 (fuera de soporte desde hace años) | 2 | pueden buscar vulnerabilidades públicas y ejecutar código en remoto | Migrar el servidor a una distribución LTS con soporte activo
texto en nota adhesiva | 5 | Pueden obtener contraseñas de red o de accesos por observación (shoulder surfing) | Implantar una política de mesas prohibiendo anotaciones de claves 
del 1 al 15 de agosto el correo operativo es el de Lucia | 1, 4 y 6 | pueden lanzar ataques de fraudes, engaños o extorsiones por suplantación de identidad para desviar pagos | Tener un procedimiento de verificación por un canal de seguro para cobros y pagos
no tienen registros DKIM ni DMARC | 4 | Pueden realizar SMTP spoofing creando un falso flujo de datos que los servidores de correo aceptan | Configurar registros SPF e implementar DKIM y políticas DMARC
última actualización del diseño o del contenido de su web fue en 2019 | 1 | Pueden localizar fallos sin parchear e inyectar código o modificar el contenido de la web | Actualizaría constantemente el software y el gestor de web
equipos con Windows 7 (sin parches de seguridad desde 2020) | 2 | Pueden explotar vulnerabilidades para introducir malware y lograr la conexión de equipos no autorizados | Tendría el sistema operativo actualizado y aislaría los equipos obsoletos
NAS de copias de seguridad expuesto | 2 y 4 | Puede facilitar la conexión no autorizada y el despliegue de virus para el secuestro o destrucción de las copias | Retiraria la IP pública del DNS y permitir acceso únicamente mediante VPN
dirección física disponible | 1 y 2 | trashing | Pueden hacer un reconocimiento físico para actividades de trashing | Destruiría la documentación en papel
datos personales | 3 | Puede facilitar actividad de reconocimiento en sistemas o ingeniería social | Restringiría información pública mediante protección de privacidad

Pista: acordaos de lo que hemos visto sobre ingeniería social, spoofing y seguridad física. Hay datos que solo se ven si los miráis con esos conocimientos.

## Paso 2. Con lo que habéis encontrado, escribid en 5 líneas como máximo el plan de ataque que montaríais contra Estudio Torrent.

1. Aprovecharía los datos WHOIS, DNS y redes sociales para identificar IP de los servidores y la clave personal.
2. Del 1 al 15 de agosto, realizaría un ataque de SMTP spoofing enviando un correo a Lucía con un malware adjunto camuflado.
3. Al abrir el archivo en Windows 7, se introduciría el malware para lograr la conexión no autorizada.
4. Mediante un escáner de puertos y exploits dirigidos a servicios de Ubuntu 8.04, obtendría el acceso con privilegios de administrador.
5. Y con acceso a la red y a la IP del NAS, ejecutaría un ransomware que cifraría los archivos y las copias de seguridad, provocando una denegación del servicio y una extorsión económica.

## Paso 3. De todas las contramedidas de vuestra tabla, elegid las 3 más importantes y justificad por qué esas y no otras.
Retirar la IP pública del DNS y permitir acceso únicamente mediante VPN: porque tener el almacenamiento de respaldo expuesto directamente a Internet con un IP pública permite que cualquier atacante externo comprometa la información confidencial o destruya y cifre las copias. La empresa puede perder la capacidad de recuperar la información ante cualquier atacante, provocando la no disponibilidad y la continuidad del negocio.
Tener el sistema operativo actualizado y aislar los equipos obsoletos: porque, tanto el servidor como los otros equipos están con sistemas obsoletos que acumulan fallos críticos sin parches de seguridad. Y ocultar no es arreglar, las medidas de ocultación no eliminan la vulnerabilidad si el código vulnerable sigue en memoria. Por lo que actualizar el software es la única contramedida definitiva que evita la conexión no autorizada.
Configurar registros SPF e implementar DKIM y políticas DMARC: porque con la configuración permisiva en SPF y la ausencia de DKIM y DMARC permite a cualquier externo enviar correos suplantando el dominio legítimo de la empresa sin ser bloqueado por los servidores de destino. De esta manera el correo sería la vía principal para introducir malware.
