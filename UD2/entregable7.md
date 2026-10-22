# UD02 · Práctica: malware en un entorno controlado, y cómo detectarlo y eliminarlo

## Parte A · Hecha en Entregable 5
Malware | Resumen
--------|--------
Adware | Software publicitario que muestra o descarga anuncios automática e invasivamente en la pantalla, a menudo en aplicaciones gratuitas
Backdoor | Mecanismo oculto que permite acceder a un sistema informático eludiendo los controles de autenticación o seguridad
Bomba Fork | Es un ataque por denegación de servicio donde un proceso se replica continuamente a sí mismo hasta agotar los recursos de CPU y memoria del sistema
Bots | Son programas automatizados diseñados para realizar tareas repetitivas, se suelen formar redes (botnets) controladas para atacar coordinadamente
Bug | Error, fallo o defecto en el diseño o programación de un software que produce resultados inesperados o comportamientos erróneos
Caballo de Troya | Es un malware disfrazado de software legítimo o inofensivo que permite el acceso no autorizado o la realización de acciones dañinas sin conocimiento del usuario
Cookies | Son ficheros de texto guardados por el navegador para recordar información de sesión, preferencias o rastrear la actividad web del usuario
Crackers | Son personas que vulneran la seguridad de sistemas, redes o software con fines maliciosos, lucrativos o destructivos
Cryptovirus / Ransomware | Código malicioso que secuestra y cifra los archivos o el sistema completo de la víctima, exigiendo un rescate para restablecer el acceso
Exploits | Son fragmentos de sotfware, datos o comandos diseñados para aprovechar vulnerabilidades y fallos de seguridad existentes en un sistema o aplicación
Hijacker | Sotfware malicioso que secuestra funciones del sistema o del navegador web
Hoax, Joke o Bulo | Mensajes falsos o cadenas engañosas que buscan desinformar, provocar pánico o hacer que los usuarios saturen redes compartiendo información errónea
Keystroke / Keylogger | Herramienta de espionaje que registra cada pulsación del teclado para robar contraseñas y datos sensibles
Ladilla virtual | Es un termino coloquial para referirse a pequeños programas parásitos molestos que se adhieren a carpetas o procesos
Leadfrog | Técnica de intrusión que salta a través de hosts o servidores intermedios para dificultar el rastreo del origen
Parásito informático | Malware que necesita acoplarse e infectar archivos ejecutables o sectores de arranque ya existentes en el equipo anfitrión para funcionar y propagarse
Pharming | Técnica que manipula la resolución de nombres DNS o el archivo hosts para redirigir al usuario
Phishing | Método de ingeniería social donde los atacantes se hacen pasar por entidades legítimas para robar credenciales y datos confidenciales
Rabbit o conejos | Malware antiguo similar a una bomba fork; se replica a sí mismo sin control hasta saturar el disco o la memoria y paralizar la máquina
Riskware | Programas legítimos que no son malware en sí, pero cuyas características o configuración pueden entrañar riesgos si un atacante los aprovecha, por ejemplo herramientas de acceso remoto
Rootkit | Conjunto de herramientas diseñado para ocultar la presencia de intrusos o malware en el sistema
Scumware o Escoria | Término despectivo para referirse al software invasivo no deseado que altera la experiencia del usuario, inyecta anuncios o recopila datos sin permiso claro
Spam | Mensajes no solicitados enviados masivamente
Spyware | Software espía que recopila información personal, hábitos de navegación y credenciales de la víctima en segundo plano
Pop-ups o Ventanas emergentes | Ventanas que aparecen de manera súbita en el navegador o sistema
Worms o gusanos | Malware con la capacidad de autoreplicarse e infectar otros dispositivos a través de la red de forma autónoma, sin necesidad de intervención del usuario ni de un archivo portador

Pregunta A: ¿Cuáles de estos incidentes no son "malware" propiamente dicho, sino fraude o ingeniería social? ¿Por qué un antivirus no los detendría? 
Los incidentes que no son malware propiamente dicho son: phising, hoax y pharming. No los detendría porque son incidentes que ocurren por fallos humanos, que no tienen un ejecutable que poder escanear. Y el pharming tampoco lo detendría porque ocurre fuera de la jurisdicción de la empresa.

## Parte B · Preparar un entorno controlado (CE d)
Antes de analizar nada sospechoso, se prepara el laboratorio de análisis. Hacedlo en este orden:

1. Apagad Torrent-Auditor. En VirtualBox, comprobad que no tiene carpetas compartidas ni portapapeles/arrastrar y soltar habilitados con el equipo anfitrión.

<img width="1025" height="632" alt="image" src="https://github.com/user-attachments/assets/465585b1-0982-466a-bccb-72bb852d3f77" />

2. Haced un snapshot con el nombre limpio-antes-malware.

<img width="712" height="288" alt="image" src="https://github.com/user-attachments/assets/aae9e857-6410-4da7-9b84-6b560d859d00" />

3. Dejad el adaptador NAT activo solo durante la Parte D (instalar ClamAV y actualizar firmas). Se desactiva antes de ejecutar el programa sospechoso (Parte F).

<img width="1000" height="487" alt="image" src="https://github.com/user-attachments/assets/d9725daa-aca6-4441-9551-a71b26e6fda5" />


- Rellenad en vuestro documento:

Medida de aislamiento | ¿Hecho? | ¿Qué riesgo evita?
----------------------|---------|-------------------
Sin carpetas compartidas ni portapapeles | Si | Evita la fuga hacia el sistema anfitrión, impidiendo que el software malicioso acceda a archivos reales
Snapshot previo | Si | Permite revertir la MV a un estado limpio si el malware daña el sistema
Sin red (o red interna aislada) al ejecutar | Si | Impide que el malware se comunique con servidores de comando y control, propague tráfico malicioso por la red local o exfiltre información hacia internet
Máquina que no contiene datos personales ni credenciales | Si | Evita el robo de identidad, filtración de credenciales de usuario o secuestro de datos reales

- Idea clave: un sandbox sirve en los dos sentidos: lo que pasa dentro no sale, y lo de fuera no influye en el análisis.

## Parte C · El fichero EICAR: una amenaza de mentira (CE e)
EICAR es una cadena de texto estándar de 68 caracteres que todos los antivirus detectan como si fuera un virus. No hace nada, pero permite comprobar que el antivirus funciona.
1. En el navegador del anfitrión abrid la página oficial de EICAR (https://www.eicar.org, sección Anti-Malware Testfile). Localizad la cadena de texto de prueba.ç
2. Copiadla en un fichero de la VM:

``` nano ~/eicar.txt```     pegad la cadena, guardad y salid 

<img width="817" height="58" alt="image" src="https://github.com/user-attachments/assets/e00d9edc-4de4-422d-8f1e-5c5ce120102f" />

``` ls -l ~/eicar.txt ```

<img width="715" height="47" alt="image" src="https://github.com/user-attachments/assets/0a26bb2e-1754-4a79-a767-298a2e2a3edb" />


3. Calculad el hash y buscadlo (solo el hash, nunca subáis ficheros desconocidos) en https://www.virustotal.com:

  <img width="893" height="347" alt="image" src="https://github.com/user-attachments/assets/5a41524c-40f2-4910-89c7-fd394caeeb80" />


```sha256sum ~/eicar.txt ``` 

  <img width="817" height="75" alt="image" src="https://github.com/user-attachments/assets/75dcc498-6eb1-4ebc-b0e3-2c88d65c70c1" />

Nombre de fichero identificado en la base de datos: 01_standard__eicar_lf.txt (la terminación _lf indica precisamente el salto de línea Line Feed que hizo que pesara 69 bytes)

- Anotad: ¿cuántos motores lo detectan? 63 de 67 motores

## Parte D · Instalar y usar ClamAV (CE e)
### D1. Instalación (con NAT activo):

```sudo apt update```

  <img width="815" height="296" alt="image" src="https://github.com/user-attachments/assets/c976d1bb-5db5-426c-a792-3b7c8de5f88c" />

```sudo apt install -y clamav ```

  <img width="805" height="200" alt="image" src="https://github.com/user-attachments/assets/b4cd9d57-df68-4007-9a36-85b99989bb9f" />



### D2. Actualizar las firmas (la "base de datos de virus"). Si falla con un mensaje de bloqueo, parad antes el servicio automático:

```sudo systemctl stop clamav-freshclam```
```sudo freshclam```
```sudo systemctl start clamav-freshclam```

<img width="803" height="532" alt="image" src="https://github.com/user-attachments/assets/8041f7ee-1947-43ae-bdfd-e8b1f8338cbb" />

- Anotad la fecha de la base de datos: ``` clamscan --version.```
- Version: ClamAV 1.2.4/28161/Thu Oct 22 06:24:11 2026
  
<img width="463" height="47" alt="image" src="https://github.com/user-attachments/assets/b02bfb1f-2ee3-4ddc-81be-15d43705ecd2" />


### D3. Escanear el fichero EICAR:
``` clamscan ~/eicar.txt ```
<img width="802" height="380" alt="image" src="https://github.com/user-attachments/assets/b09e5afa-3d13-424e-978a-bf9f70dfb48f" />

Veréis una línea terminada en FOUND y un resumen. Anotad el nombre con el que ClamAV bautiza la amenaza y el número de ficheros infectados.
- Nombre con el que ClamAV bautiza la amenaza: Eicar-Signature
- Número de ficheros infectados: 1

### D4. Escanear una carpeta entera, mostrando solo lo infectado:
```clamscan -r -i -v ~```
<img width="457" height="383" alt="image" src="https://github.com/user-attachments/assets/15bccfcb-170f-4526-9c31-796a88696890" />

Resultado: Detectará y mostrará únicamente la línea de detección de ~/eicar.txt junto al resumen final

Opciones útiles:
Opción | Qué hace
-------|---------
-r | recursivo
-i | muestra solo ficheros infectados
-v | modo verboso
--move=DIR | mueve los infectados a una carpeta (cuarentena)
--remove | Borra los infectados (peligroso con falsos positivos)
-l fichero-log | guarda un registro

### D5. Cuarentena en vez de borrar:
```mkdir -p ~/cuarentena```
```clamscan -r -i -v --move=$HOME/cuarentena ~```
```ls -l ~/cuarentena```
<img width="572" height="70" alt="image" src="https://github.com/user-attachments/assets/db13dd8c-8876-45f2-8fe9-ce115e1e7333" />

- Comprobación: Al listar ~/cuarentena, debe aparecer el archivo eicar.txt trasladado desde tu carpeta de usuario.

- Pregunta D: ¿Por qué suele ser mejor mover a cuarentena que borrar directamente? ¿Qué pasaría si ClamAV marcara por error un fichero importante del ERP de Torrent?
Suele ser mejor porque neutraliza la amenaza al aislarla del sistema sin provocar una pérdida de la información, lo que permite conservar el archivo para poder analizarlo o recuperarlo.
Si ClamAV marcara por error un fichero de ERP importante y este se eliminara directamente, el sistema de gestión dejaría de funcionar y paralizaria la facturación y el funcionamiento de la empresa, pero si lo mueve a cuarentena solo se produce un fallo temporal que se puede recolver facilmente devolviendo el archivo.


### D6. Detectar rootkits (herramienta complementaria, también de la familia "paliativa"):
- ```sudo apt install -y rkhunter```
  <img width="743" height="125" alt="image" src="https://github.com/user-attachments/assets/164c4050-6896-440c-b5d3-50f36edb83d4" />
  
- Al instalarlo nos sale una ventana de configuración del gestor de paquetes de Debian/Ubuntu (debconf). Ha aparecido porque al instalar rkhunter. Debemos salir.
<img width="850" height="677" alt="image" src="https://github.com/user-attachments/assets/10e70f80-3589-4e2f-a05b-85f075907ca3" />

- ```sudo rkhunter --update```
  <img width="875" height="50" alt="image" src="https://github.com/user-attachments/assets/f0cd6827-c1ff-40f9-91c3-1abf0d384887" />

  Este error ocurre porque en Ubuntu/Debian el archivo de configuración de rkhunter viene por defecto con la descarga de actualizaciones desactivada mediante una directiva inválida.
  Sigue estos pasos para solucionarlo:
  1. Modificar la configuración de rkhunter: ``` sudo nano /etc/rkhunter.conf ``` Añadiendo # a la línea " WEB_CMD="/bin/false" " 
  <img width="723" height="382" alt="image" src="https://github.com/user-attachments/assets/345a8912-93d9-4e35-94ed-0810ae35c63d" />

  2. Ejecuta de nuevo
    <img width="845" height="256" alt="image" src="https://github.com/user-attachments/assets/4585860c-4c19-48ff-b8ff-8bf2169f8b14" />

    - La actualización remota de firmas de rkhunter falla debido a que los repositorios y espejos predeterminados de la versión empaquetada ya no se encuentran operativos. No obstante, la herramienta mantiene su base de datos local preinstalada para realizar el análisis.

- ```sudo rkhunter --propupd```
  <img width="512" height="75" alt="image" src="https://github.com/user-attachments/assets/41da10ba-3877-406b-85cc-85bf5f40e0c6" />

- ```sudo rkhunter --check --sk --rwo```
<img width="882" height="181" alt="image" src="https://github.com/user-attachments/assets/7a554609-9fbe-4617-9eee-a19df8b0081e" />

  - Avisos detectados:
    - The command '/usr/bin/lwp-request' has been replaced by a script: Perl script text executable: alerta de que un binario esperado ha sido reemplazado por un script de Perl.
    - User 'postfix' has been added to the passwd file: se detecta un nuevo usuario creado en el archivo de cuentas del sistema (/etc/passwd)
    - Group 'postfix' has been added to the group file: se detecta un nuevo grupo en /etc/group
    - Changes found in the group file for group 'postdrop': cambios en los grupos del sistema relacionados con el componente postdrop
    - Hidden file found: /etc/.resolv.conf.systemd-resolved.bak: presencia de un archivo oculto (empieza por .) de respaldo de configuración de red.
    - Hidden file found: /etc/.updated: presencia de un archivo oculto utilizado como marca de tiempo de actualización del sistema
  - ¿Son preocupantes o no?
    - No son preocupantes: se trata en su totalidad de falsos positivos.
    - Los avisos de postfix y postdrop: Aparecieron porque hace unos minutos, al instalar rkhunter, se desplegó el paquete postfix como dependencia (la pantalla azul de configuración que te apareció antes). El instalador oficial del sistema creó de forma legítima ese usuario y grupos de servicio para operar el correo local.
    - El script /usr/bin/lwp-request: En las versiones modernas de Ubuntu/Debian, lwp-request se distribuye oficialmente como un script ejecutable en lenguaje Perl dentro del paquete estándar libwww-perl, por lo que es el comportamiento esperado de la distribución.
    - Los archivos ocultos en /etc: Tanto .resolv.conf.systemd-resolved.bak como .updated son archivos auxiliares legítimos generados por systemd para gestionar el servicio DNS local y registrar marcas temporales del gestor de paquetes.
  
- --propupd guarda una referencia (los hashes de los ficheros importantes). Es la misma idea del hash del sistema de la fase 4.
- --rwo muestra solo avisos (warnings).
- El informe completo queda en /var/log/rkhunter.log.

 - Ojo: rkhunter suele dar falsos positivos en un sistema recién instalado. Anotad cuáles salen y razonad si son preocupantes o no.

## Parte E · Análisis estático: mirar sin ejecutar (CE d)
Ya sabéis que el análisis estático examina un fichero sin ejecutarlo. Probadlo con el fichero que os dará el profesor, sospechoso.sh:
```file sospechoso.sh```
```sha256sum sospechoso.sh```
```cat sospechoso.sh```

Contestad por escrito, antes de ejecutarlo:
1. ¿Qué crees que hace cada línea?
2. ¿Crea ficheros? ¿Abre puertos? ¿Modifica algo para sobrevivir a un reinicio?
3. Si fuera un .exe binario, ¿qué habría que hacer en su lugar? (Pista: no se puede leer, habría que usar herramientas como strings o un desensamblador.)

## Parte F · Análisis dinámico: observar qué hace (CE d)
1. Desactivad el adaptador NAT de la VM (o pasadla a red interna sin otros equipos).
2. Haced un snapshot: antes-de-ejecutar.
3. Tomad una foto del estado inicial (guardad la salida en ficheros para comparar luego):

```ps aux > ~/antes_procesos.txt```
```ss -tulpn > ~/antes_puertos.txt```
```crontab -l > ~/antes_cron.txt 2>&1```
```ls -la /tmp > ~/antes_tmp.txt```

4. Ejecutad el programa como usuario normal (no como root):
```bash sospechoso.sh```

5. Repetid la foto y comparad:
```ps aux > ~/despues_procesos.txt```
```ss -tulpn > ~/despues_puertos.txt```
```crontab -l > ~/despues_cron.txt 2>&1```
```ls -la /tmp > ~/despues_tmp.txt```

```diff ~/antes_procesos.txt ~/despues_procesos.txt```
```diff ~/antes_puertos.txt ~/despues_puertos.txt```
```diff ~/antes_cron.txt ~/despues_cron.txt```
```diff ~/antes_tmp.txt ~/despues_tmp.txt```

6. Rellenad el informe de comportamiento:
Qué hace | Evidencia (comando y salida) | ¿Qué fase del ataque recuerda?
--------|-------------------------------|-----------------------------
Crea ficheros o carpetas |
Abre algún puerto o proceso |
Intenta persistir |

7. Comprobad la persistencia: reiniciad la VM y volved a mirar si el comportamiento reaparece.

8. Erradicación: documentad cómo lo eliminaríais (parar el proceso, quitar la entrada de arranque, borrar los ficheros) y verificad con los mismos comandos del paso 3 que ha desaparecido. Después podéis volver al snapshot.

- Pregunta F: Habéis encontrado el comportamiento comparando "antes" y "después". ¿Cuál es el equivalente en un servidor real de Torrent si nadie hizo la foto inicial? ¿Qué relación tiene con el inventario de servicios que hicisteis en la práctica anterior?

## Parte G · Preventivo o paliativo (CE c, e)
Clasificad cada medida:

Medida | Preventiva | Paliativa
------|-------------|-----------
Mantener el sistema actualizado		
ClamAV escaneando a demanda un disco		
Descargar software solo de repositorios oficiales		
Copia de seguridad semanal desconectada de la red		
Cuenta de usuario sin privilegios de administrador		
rkhunter tras sospechar una intrusión		
Formación del personal contra el phishing


