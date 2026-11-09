# Contraseñas, MFA y privilegios justos
## PARTE 1: Contraseñas robustas con PAM
### 1.1 ¿Por qué PAM y no tocar /etc/shadow directamente?
PAM (Pluggable Authentication Modules) es la capa que usa Linux para gestionar la autenticación de cualquier programa: login, su, passwd, sshd... Configurar la política de contraseñas en PAM significa que se aplica una sola vez y afecta a todos los programas que autentican usuarios, en vez de tener que vigilar cada aplicación por separado.

### 1.2 Instalar y activar pwquality
El módulo pam_pwquality es el que comprueba la calidad de una contraseña nueva antes de aceptarla.
- ```sudo apt update ```
- ```sudo apt install libpam-pwdquality ```

Se activa comprobando que esta línea existe en /etc/pam.d/common-password, justo antes de la línea de pam_unix.so (en Debian/Ubuntu suele instalarse ya activada; si no, añádela):
- ```password requisite pam_pwquality.so retry=3 ```
<img width="815" height="475" alt="image" src="https://github.com/user-attachments/assets/0c6086a3-7f79-4504-87c4-309df09dc283" />

Qué se considera una contraseña «de calidad» se configura en /etc/security/pwquality.conf:
Parámetro | Valor sugerido | Qué hace
----------|---------------|---------
minlen | 12 | Longitud mínima de contraseña
minclass | 3 | Nª minimo de tipos de carácter distintos
dredit / ucredit / lcredit / ocredit | -1 cada uno | Exigen al menos un digito / mayúscula / minúscula / símbolo (el signo negativo = obligatorio)
dictcheck | 1 | Rechaza contraseñas basadas en palabras de diccionario (usa cracklib)
maxrepeat | 3 | Máximo de caracteres idénticos seguidos
enforce_for_root | (sin valor, solo activarla) | Sin esta linea, root puede saltarse toda la política

### 1.3 Práctica guiada — aplicar la política
1. Edita el archivo de directivas con ``` sudo /etc/security/pwquality.conf ``` y fija los valores de la tabla anterior (descomenta la línea o añádela si no existe).
  <img width="822" height="682" alt="image" src="https://github.com/user-attachments/assets/6767084e-4787-4f10-adee-6f3fb6a84ac1" />
  <img width="856" height="625" alt="image" src="https://github.com/user-attachments/assets/88eb3c56-1987-40e1-888a-1eec58299374" />

2. Comprueba que la línea pam_pwquality.so está presente en /etc/pam.d/common-password.
  <img width="872" height="492" alt="image" src="https://github.com/user-attachments/assets/fa871340-f4e7-4e64-b702-8c910077ed0c" />

3. Como el usuario hugo (grupo operarios, creado ayer en la práctica de ACL), intenta cambiarle la contraseña por algo débil y observa el rechazo (Escribe 1234)
  <img width="777" height="70" alt="image" src="https://github.com/user-attachments/assets/73444a8a-0f5f-4271-b1ce-c21fd681378c" />

  - Verificación: El sistema la rechaza.

4. Repite con una contraseña que cumpla la política y confirma que esta vez se acepta (Escribe: Holaquetal123.)
  <img width="710" height="138" alt="image" src="https://github.com/user-attachments/assets/bc1edf6b-f074-41bb-88f1-68e836bee851" />

5. Como root, intenta ponerle a root mismo una contraseña débil: sin enforce_for_root se aceptaría sin más. Actívala y repite la prueba. (1234)
  <img width="731" height="87" alt="image" src="https://github.com/user-attachments/assets/691a19ac-0ffb-44c8-986e-7910bf571693" />
  Ahora comentando la linea:
  <img width="875" height="626" alt="image" src="https://github.com/user-attachments/assets/97004cb2-541f-4c7b-9d7a-92a2dae312bd" />
  <img width="731" height="118" alt="image" src="https://github.com/user-attachments/assets/e38da6dd-30c3-4b51-8339-ccf4739c4981" />


### 1.4 Caducidad y aviso con chage
pwquality decide si una contraseña es lo bastante fuerte al crearla; chage decide durante cuánto tiempo sigue siendo válida.

Opción | Qué hace
------|---------
-m | Días mínimos antes de poder volver a cambiarla (evita cambiarla 5 veces seguidas para recuperar la de siempre)
-M | Días máximos de validez: pasado ese tiempo, caduca
-W | Días de aviso antes de que caduque
-l | Lista la política actual aplicada a un usuario

Como ingeniería y supervisores tienen permisos de escritura sobre planos/ (ved la práctica de ACL), es razonable que su contraseña caduque antes que la de operarios:
1. Aplica la caducidad a los usuarios de ingeniería y supervisores (60 días)
- ``` for u in nestor victor lucia carlos mary john; do ```
- ```sudo chage -m 1 -M 60 -W 7 $u ```
- ``` done```
  <img width="752" height="67" alt="image" src="https://github.com/user-attachments/assets/cc5055ea-6a22-42f6-95e1-6d377c1353ba" />
  
2. Aplica la caducidad a los usuarios de operarios (90 días)
- ``` for u in alberto oscar hugo; do ```
- ``` sudo chage -m 1 -M 90 -W 7 $u ```
- ``` done ```
  <img width="670" height="68" alt="image" src="https://github.com/user-attachments/assets/7cfec2ef-ccc1-46c9-820e-736ad312e3c2" />

- Verificación: Ejecuta chage -l nestor para comprobar que ```Maximum number of days between password change ``` tiene asignado el valor 60:
  <img width="745" height="177" alt="image" src="https://github.com/user-attachments/assets/244a225e-5fe3-4851-9d59-8859bcbd11e4" />

## PARTE 2: MFA en SSH con Google Authenticator
### 2.1 El concepto: TOTP
TOTP (Time-based One-Time Password) genera un código de 6 dígitos que cambia cada 30 segundos, a partir de un secreto compartido entre el servidor y tu app (Google Authenticator, Authy, FreeOTP...) y la hora actual. Aunque alguien robe tu contraseña, sin ese secreto no puede generar el código — por eso es un segundo factor («algo que tienes»), no solo «algo que sabes».

### 2.2 Instalar el módulo PAM
Ejecuta con privilegios administrativos en el sistema:
- ```sudo apt install -y libpam-google-authenticator ```

### 2.3 Generar el secreto (lo hace cada usuario, con su propia cuenta)
1. Cambia a la sesión de carlos: ``` su -l carlos ```
2. Lanza el asistente interactivo: ``` google-authenticator ```
  <img width="893" height="491" alt="image" src="https://github.com/user-attachments/assets/65c935c8-1711-4590-85e5-34ece35e7863" />

3. El asistente hace varias preguntas; esto es lo que responderéis y por qué:
Pregunta | Respuesta recomendada
---------|--------------------
¿Tokens time-based? | y - es el estándar TOTP
¿Actualizar ~/.google_authenticator? | y - guarda la clave en su directorio personal 
¿Impedir que el mismo código se use dos veces? | y - evita reutilizar un código capturado
¿Ampliar la ventana de validez? | n - por defecto ya tolera pequeños desfases de reloj
¿Limitar los intentos (rate-limit? | y - 3 intentos cada 30 seg frena la fuerza bruta sobre el código

4. El asistente muestra un código QR en texto y una clave secreta en base32, además de 5 «scratch codes» (códigos de emergencia, de un solo uso).
Escanea el código QR que se dibuja en la terminal con la app móvil y guarda los 5 scratch codes en un sitio seguro.


### 2.4 Conectar Google Authenticator con sshd
1. Edita el archivo PAM de SSH: ```sudo nano /etc/pam.d/sshd ```
2. Al final del archivo añade esta línea (después de @include common-auth, para que primero se compruebe la contraseña y después el código): ```auth required pam_google_authenticator.so ```
  <img width="791" height="126" alt="image" src="https://github.com/user-attachments/assets/e4ff9238-6b18-4f72-9e21-7eb8ba6f29fa" />


3. Edita /etc/ssh/sshd_config con: ```sudo nano /etc/pam.d/sshd_config```, asegúrate de que estas directivas están así (el nombre exacto puede variar según la versión de OpenSSH: en las más recientes es KbdInteractiveAuthentication, en las antiguas ChallengeResponseAuthentication):
- ```KbdInteractiveAuthentication yes```
  <img width="727" height="72" alt="image" src="https://github.com/user-attachments/assets/cf7ef739-d789-4488-84f0-5a2c4f864a59" />

- ```UsePAM yes```
  <img width="742" height="106" alt="image" src="https://github.com/user-attachments/assets/045e6b3c-f947-43c3-a942-0b41b803effc" />

- ```PasswordAuthentication yes```
  <img width="467" height="87" alt="image" src="https://github.com/user-attachments/assets/12832856-9941-4a52-bc6f-8807e76075ad" />

4. Aplica los cambios reiniciando el servicio: ```sudo systemctl restart ssh```

### 2.5 Verificación
1. Sin cerrar la sesión actual, abre otra terminal y conéctate como carlos: ```ssh carlos@localhost```
2. Verificación: Debe pedirte primero la contraseña y seguidamente el código de 6 digitos ```Verification code```
  <img width="496" height="135" alt="image" src="https://github.com/user-attachments/assets/546a9d8e-a7c3-4cde-9885-9f1a3c5c2743" />

3. Introduce un código falso a propósito y comprueba que rechaza la conexión con ``` Permission denied```
  <img width="798" height="577" alt="image" src="https://github.com/user-attachments/assets/b8327b8c-01f9-4e14-aaec-f81a16281e52" />

4. Al poner bien el código si que dejará entrar
  <img width="798" height="577" alt="image" src="https://github.com/user-attachments/assets/9acaa29b-b3f7-412b-bf22-5680aa0b4ad3" />


### 2.6 ¿A quién se lo exigimos?
**El módulo pam_succeed_if permite condicionar una regla de PAM a la pertenencia a un usuario o grupo.**
**Realiza las investigaciones necesarias para crear un grupo al cual exigirle autenticación de doble factor**

Para exigir el doble factor únicamente a los miembros del grupo supervidores:
1. Abrimos de nuevo /etc/pam.d/sshd: ```sudo nano /etc/pam.d/sshd``
2. Modificamos la línea final dejándola de esta manera: ```auth [success=1 default=ignore] pam_succeed_if.so quiet user notingroup supervisores``` ```auth required pam_google_authenticator.so```
  - Explicación: si el usuario NO pertenece a supervisores, PAM se salta 1 regla y no le pide el MFA; a los miembros de supervisores sí se lo exige obligatoriamente
  <img width="727" height="137" alt="image" src="https://github.com/user-attachments/assets/97951878-808d-434b-84ab-0168e07ad14c" />

3. Guarda el fichero y reinicia el servicio con ```sudo systemctl restart ssh```

## PARTE 3 - Privilegios justos con sudo
### 3.1 Por qué no usar la cuenta root a diario
Es el mismo principio de mínimo privilegio que ya vimos con UAC en Windows: si una cuenta con la que se trabaja a diario tiene siempre todos los permisos, cualquier error o malware que la afecte tiene también todos los permisos. sudo permite conceder, comando a comando, exactamente lo que cada grupo necesita, sin compartir la contraseña de root ni dar una cuenta de administrador completa.

### 3.2 Sintaxis básica de sudoers
Las reglas de sudo nunca se editan a mano con un editor cualquiera: se usa visudo, que comprueba la sintaxis antes de guardar y evita dejar el sistema sin forma de usar sudo por un error de tecleo. Lo recomendado es crear un fichero propio en /etc/sudoers.d/ en vez de tocar /etc/sudoers directamente.
Quién | Desde dónde | Como quién | Qué comandos | Pide clave
------|--------------|-----------|--------------|----------
%ingenieria | ALL | (root) | systemctl restart / status hmi_planta | Sí
$supervisores | ALL | (root) | auditoria_planta.sh | no (NOPASSWD)
operarios | - | - | ninguno | -

El % delante del nombre indica que es un grupo, no un usuario. NOPASSWD es una excepción pensada solo para un script muy concreto y de bajo riesgo (una auditoría de solo lectura); como norma general, que sudo pida contraseña es lo deseable.

### 3.3 Práctica guiada
1. Crea un script de prueba y dale permisos de ejecución:
- ```sudo tee /usr/local/bin/auditoria_planta.sh << 'EOF' ```
- ```#!/bin/bash ```
- ```echo "[$(date)] Auditoria de planda ejecutada por $(whoami)"```
- `` EOF ```
  <img width="712" height="127" alt="image" src="https://github.com/user-attachments/assets/dc411475-3f43-4f92-b1d8-359015206077" />

- ```sudo chmod 755 /usr/local/bin/auditoria_planta.sh```
  - Verificación: Ejecuta auditoria_planta.sh y comprueba que imprime la fecha y el usuario.
  <img width="627" height="50" alt="image" src="https://github.com/user-attachments/assets/845f4d02-5dab-4db3-b719-be464f95e563" />

2. Crea el fichero de reglas de sudo usando visudo:
- ```sudo visudo -f /etc/sudoers.d/torrent ```

3. Escribe dentro estas dos lineas:
- %ingenieria ALL=(root) /usr/bin/systemctl restart hmi_planta, /usr/bin/systemctl status hmi_planta
- %supervisores ALL=(root) NOPASSWD: /usr/local/bin/auditoria_planta.sh
  <img width="827" height="65" alt="image" src="https://github.com/user-attachments/assets/6b4d0309-ad0f-474a-8000-5fd8b63f07e2" />

### 3.4. Verificación
1. Comprueba los permisos de cada usuario según su grupo:
- ```sudo -l -U nestor```
- ```sudo -l -U carlos```
- ```sudo -l -U alberto```
  <img width="852" height="292" alt="image" src="https://github.com/user-attachments/assets/c6307924-84a7-44c2-9e04-7afddf6bdac3" />

  - Verificación: nestor solo puede reiniciar/consultar hmi_planta pidiendo clave; carlos puede ejecutar la auditoria sin clave y alberto no tiene permisos administrativos.
2. Entra como alberto y prueba a reiniciar el servicio:
- ```su -l alberto```
- ```sudo systemctl restart hmi_planta```
  <img width="518" height="71" alt="image" src="https://github.com/user-attachments/assets/ee552e0e-804a-439e-9464-e727a5be6f9a" />

  - Verificación: La salida debe mostrar: alberto is not in the sudoers file. This incident will be reported
3. Revisa la traza de seguridad generada:
- ```sudo tail n 20 /var/log/auth.log```
  <img width="892" height="686" alt="image" src="https://github.com/user-attachments/assets/82a0f6a8-bff7-4d7e-8e3a-13ef568fb975" />

4. Entra como carlos y prueba el script:
- ```su -l carlos ```
- ``` sudo /usr/local/bin/auditoria_planta.sh```
 <img width="641" height="57" alt="image" src="https://github.com/user-attachments/assets/ba889442-1cca-4b01-8589-9f37f3cfd3ce" />

  - Verificación: El script se ejecuta inmediatamente sin pedir ninguna contraseña.

## Preguntas de reflexión
1. Una contraseña de 20 caracteres aleatorios es, en teoría, casi imposible de reventar por fuerza bruta. ¿Qué riesgo concreto cubre el MFA que una contraseña así, por fuerte que sea, no cubre?
2. Buscad brevemente la postura actual del NIST sobre la caducidad obligatoria de contraseñas. ¿Por qué forzar el cambio cada pocos días puede llegar a ser contraproducente para la seguridad real?
3. Aunque sudo esté perfectamente configurado, seguís necesitando las ACL de la sesión anterior. Explicad con un ejemplo por qué una capa no sustituye a la otra.
4. Un alumno pierde el móvil con Google Authenticator y no guardó los scratch codes. ¿Qué tendría que hacer el administrador del sistema para devolverle el acceso sin comprometer la seguridad del resto de cuentas?
