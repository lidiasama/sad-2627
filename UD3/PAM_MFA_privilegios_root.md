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
1. Edita /etc/security/pwquality.conf con sudo y fija los valores de la tabla anterior (descomenta la línea o añádela si no existe).
  <img width="822" height="682" alt="image" src="https://github.com/user-attachments/assets/6767084e-4787-4f10-adee-6f3fb6a84ac1" />
  <img width="856" height="625" alt="image" src="https://github.com/user-attachments/assets/88eb3c56-1987-40e1-888a-1eec58299374" />

2. Comprueba que la línea pam_pwquality.so está presente en /etc/pam.d/common-password.
  <img width="872" height="492" alt="image" src="https://github.com/user-attachments/assets/fa871340-f4e7-4e64-b702-8c910077ed0c" />

3. Como el usuario hugo (grupo operarios, creado ayer en la práctica de ACL), intenta cambiarle la contraseña por algo débil y observa el rechazo (1234)
  <img width="777" height="70" alt="image" src="https://github.com/user-attachments/assets/73444a8a-0f5f-4271-b1ce-c21fd681378c" />

4. Repite con una contraseña que cumpla la política y confirma que esta vez se acepta (Holaquetal123.)
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
1. Aplicar a ingeniería y supervisores (60 días)
- ``` for u in nestor victor lucia carlos mary john; do ```
- ```sudo chage -m 1 -M 60 -W 7 $u ```
- ``` done```
  <img width="752" height="67" alt="image" src="https://github.com/user-attachments/assets/cc5055ea-6a22-42f6-95e1-6d377c1353ba" />


3. Aplicar a operarios (90 días)
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
- ```sudo apt install libpam-google-authenticator ```

### 2.3 Generar el secreto (lo hace cada usuario, con su propia cuenta)
Conectado ya como el usuario (por ejemplo carlos, del grupo supervisores), ejecuta:
- ``` google-authenticator ```
El asistente hace varias preguntas; esto es lo que responderéis y por qué:
Pregunta | Respuesta recomendada
---------|--------------------
¿Tokens time-based? | y - es el estándar TOTP
¿Actualizar ~/.google_authenticator? | y
¿Impedir que el mismo código se use dos veces? | y - evita reutilizar un código capturado
¿Ampliar la ventana de validez? | n - por defecto ya tolera pequeños desfases de reloj
¿Limitar los intentos (rate-limit? | y - 3 intentos cada 30 seg frena la fuerza bruta sobre el código

El asistente muestra un código QR en texto y una clave secreta en base32, además de 5 «scratch codes» (códigos de emergencia, de un solo uso).

Deberás escanear el QR en la APP Google Authenticator o introducir la clave a mano.

### 2.4 Conectar Google Authenticator con sshd
Añade esta línea al final de /etc/pam.d/sshd (después de @include common-auth, para que primero se compruebe la contraseña y después el código): ```auth required pam_google_authenticator.so ```

Y en /etc/ssh/sshd_config, asegúrate de que estas directivas están así (el nombre exacto puede variar según la versión de OpenSSH: en las más recientes es KbdInteractiveAuthentication, en las antiguas ChallengeResponseAuthentication):
- ```KbdInteractiveAuthentication yes```
- ```UsePAM yes```
- ```PasswordAuthentication yes```
