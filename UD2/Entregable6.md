## Parte A · ¿Está actualizado el sistema?

## Parte B · ¿De dónde vienen los programas instalados?
### B1. Mirad de qué repositorios se descarga vuestro sistema e investiga el fichero donde están configurados los repositorios y comenta el significado de cada una de las líneas y su contenido
``` cat /etc/apt/sources.list ```

<img width="650" height="267" alt="image" src="https://github.com/user-attachments/assets/129a9f0c-0a2b-4127-8980-0f0197865bad" />

- ``` Types: deb ``` : Especifica el tipo de paquete que se va a descargar. En este caso son paquetes binarios precompilados listos para instalar. Si se requirieran los paquetes de código fuente para compilar se indicaría con ``` deb-src ```.
- ``` URIs: [http://es.archive.ubuntu.com/ubuntu/](http://es.archive.ubuntu.com/ubuntu/) ```: Es la dirección web desde donde el gestor APT descarga los paquetes y los índices de software. En este caso apunta al mirror oficial ubicado en España.
- ``` Suites: noble noble-updates noble-backports ``` : Representa las ramas o distribuciones del sistema operativo configuradas para este origen.
  - ``` noble ``` es la rama base de la versión instalada.
  - ``` noble-updates ``` son las actualizaciones recomendadas de software que se publican tras el lanzamiento del sistema.
  - ``` noble-backports ``` son versiones más recientes de ciertas aplicaciones portadas desde ramas más nuevas de Ubuntu.
- ``` Components: main restricted universe multiverse ``` : Define las secciones o áreas del repositorio según su licencia y nivel de soporte de Canonical:
  - ``` main ``` : Sotfware libre y de código abierto con soporte oficial completo de Canonical.
  - ``` restricted ``` : Sotfware propietario con soporte oficial.
  - ``` universe ``` : Software libre y de código abierto manenido por la comunidad, sin soporte comercial garantizado por Canonical.
  - ``` multiverse ``` : Sotfware restringido por temas de patentes o licencias propietarias, no soportado oficialmente.
- ``` Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg ``` : Es la ruta absoluta al llavero de claves criptográficas públicas GPG. APT utiliza esta clave para verificar la integridad y la autenticidad de los paquetes descargados, asegurando que provienen legítimamente de Ubuntu y no han sido manipulados.

- ``` Types: deb ``` : Indica paquetes binarios precompilados.
- ``` URIs: [http://security.ubuntu.com/ubuntu/](http://security.ubuntu.com/ubuntu/) ``` : Es el servidor especifico dedicado a alojar las actualizaciones urgentes de seguridad. Está separado para garantizar disponibilidad inmediata y sincronización prioritaria cuando se publica un parche crítico.
- ``` Suites: noble-security ``` : Es la rama dedicada exclusivamente a parches y correcciones críticas de vulnerabilidades (CVEs) para Ubuntu Noble.
- ``` Components: main restricted universe multiverse ``` : Aplica las actualizaciones de seguridad a todas las áreas de software habilitadas en el sistema.
- ``` Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg ``` : Es la clave GPG empleada para validar que los parches de seguridad estén firmados digitalmente por Canocical.


``` ls /etc/apt/sources.list.d/ ```
<img width="507" height="48" alt="image" src="https://github.com/user-attachments/assets/4481646a-3efe-4cd3-a8e7-162c6b9225d5" />

Los ficheros presentes en el directorio son:
- ``` ubuntu.sources ``` : Es el fichero principal de configuración de repositorios del sistema en las versiones modernas de Ubuntu. Contiene las fuentes oficiales activas en formato declarativo deb822. Es el archivo que APT lee actualmente al ejecutar un ``` sudo apt update ```.
- ``` ubuntu.sources.curtin.orig ``` : Es una copia de seguridad del archivo original. La extensión .orig asegura que el gestor APT lo ignore al buscar repositorios, dejándolo disponible únicamente como respaldo.

### B2. apt solo instala paquetes firmados por claves en las que confía. Mirad dónde están esas claves:

``` ls /etc/apt/trusted.gpg.d/ /usr/share/keyrings/ 2>/dev/null ```

<img width="797" height="296" alt="image" src="https://github.com/user-attachments/assets/af704193-d6a1-467f-938b-594a2467834f" />

**Reflexiona:** ¿qué pasaría si un atacante introduce una clave falsa en /usr/share/keyrings/? ¿y si intercepta o reemplaza la clave oficial de Ubuntu ubuntu-archive-keyrings.gpg?
Si introduce una clave falsa pero no añade un repositorio falso configurado apuntando a esa clave, no pasaría nada. En caso de hacerlo, podría instalar cualquier malware.
Si intercepta o reemplaza la clave oficial de Ubuntu, como la clave ya no coincide con Canonical, el sistema daría error y rechazaría los paquetes legítimos. Y si el atacante desvía el tráfico o hace pasar a un repositorio por uno de Ubuntu, el equipo creerá que sus paquetes son oficiales y al instalarlos se ejecutarían con permisos de administrador.

### B3. Información de un paquete concreto. Elegid openssh-server o curl:

``` apt policy openssh-client ``` 

<img width="837" height="242" alt="image" src="https://github.com/user-attachments/assets/d82dfd97-0cc3-4c29-b307-392b7c522686" />

Versión instalada: 1:9.6p1-3ubuntu13.19
Versión candidata: 1:9.6p1-3ubuntu13.19
Repositorio de origen: 
- actualizaciones de mantenimiento para Ubuntu 24.04 LTS: ``` [http://es.archive.ubuntu.com/ubuntu](http://es.archive.ubuntu.com/ubuntu) noble-updates/main ```
- parches y actualizaciones de seguridad: ``` [http://security.ubuntu.com/ubuntu](http://security.ubuntu.com/ubuntu) noble-security/main ```
- repositorio base oficial de la versión instalada: ``` [http://es.archive.ubuntu.com/ubuntu](http://es.archive.ubuntu.com/ubuntu) noble/main ```
Significado de la tabla de versiones:
- Línea de versión y estado actual ``` *** 1:9.6p1-3ubuntu13.19 500 ```:
  - Los asteriscos triples (***) señalan que esta es la versión que actualmente se encuentra instalada en el equipo.ç
  - El número 500 representa la prioridad por defecto.
- Orígenes de la versión ``` 1:9.6p1-3ubuntu13.19 ``` :
  - noble-updates/main y noble-security/main con prioridad 500: indican que esta versión actualizada está disponible tanto en el canal de actualizaciones habituales como en el de parches de seguridad de Canonical.
  - ``` 100 /var/lib/dpkg/status ```: la base de datos interna de paquetes locales (dpkg) confirma con prioridad 100 que esta versión exacta ya está registrada e instalada localmente en el disco.
- Versión base alternativa ``` 1:9.6p1-3ubuntu13 500 ```
  - ``` noble/main``` : es la versión original que se publicó inicialmente con el lanzamiento de la distribución (Ubuntu Noble 24.04). Al ser inferior a la versión con parches (13.19), el gestor no la elige como candidata

 ### B4. Buscad paquetes instalados que no proceden de ningún repositorio (instalados a mano):

``` apt list --installed 2>/dev/null | grep -i local ``` 
<img width="850" height="156" alt="image" src="https://github.com/user-attachments/assets/868e8e3c-fd86-4ffc-a21a-4ca337c47949" />

Cada línea que salga merece una pregunta: ¿quién lo instaló, de dónde, y quién lo verificó?
- krb5-locales:
  - ¿Quién lo instaló? El sistema automáticamente como dependencia (automatic).
  - ¿De dónde? De los repositorios oficiales de Ubuntu (noble-updates).
  - ¿Quién lo verificó? APT mediante la clave oficial de Canonical (ubuntu-archive-keyring.gpg)
- libencode-locale-perl:
  - ¿Quién lo instaló? El sistema automáticamente como dependencia (automatic).
  - ¿De dónde? De los repositorios oficiales de Ubuntu (noble)
  - ¿Quién lo verificó? APT mediante la clave oficial de Canonical (ubuntu-archive-keyring.gpg)
- liblocale-gettext-perl:
  - ¿Quién lo instaló? El sistema automáticamente como dependencia (automatic).
  - ¿De dónde? De los repositorios oficiales de Ubuntu (noble)
  - ¿Quién lo verificó? APT mediante la clave oficial de Canonical (ubuntu-archive-keyring.gpg)
- locales:
  - ¿Quién lo instaló? El sistema automáticamente como dependencia (automatic).
  - ¿De dónde? De los repositorios oficiales de Ubuntu (noble-updates y noble-security)
  - ¿Quién lo verificó? APT mediante la clave oficial de Canonical (ubuntu-archive-keyring.gpg)
- nessus:
  - ¿Quién lo instaló? El administrador manualmente mediante dpkg -i o similar (local)
  - ¿De dónde? De un origen externo a Ubuntu (descargado de la web del fabricante Tenable)
  - ¿Quién lo verificó?Nadie por APT; fue responsabilidad del administrador comprobar el hash SHA256 o la firma del archivo antes de instalarlo.
- python-babel-localedata:
  - ¿Quién lo instaló? El sistema automáticamente como dependencia (automatic).
  - ¿De dónde? De los repositorios oficiales de Ubuntu (noble).
  - ¿Quién lo verificó? APT mediante la clave oficial de Canonical (ubuntu-archive-keyring.gpg)

### B5. Comprobad si algún fichero de un paquete ha cambiado desde que se instaló:

``` sudo dpkg -V ```
<img width="607" height="122" alt="image" src="https://github.com/user-attachments/assets/4158dc4c-d63c-45d6-b4e5-61756c5ed18e" />

Si no sale nada, todo coincide con lo instalado.
Si salen líneas, la primera columna indica qué cambió (5 = el hash MD5 es distinto). Los ficheros de configuración (c) cambian de forma normal. Un binario (/usr/bin/...) modificado es una alarma.

### Pregunta B: dpkg -V ¿detectaría un rootkit que además hubiera modificado la base de datos de dpkg? ¿Por qué conviene comparar con una copia guardada fuera del sistema? (Pista: recordad la fase 4 de la intrusión.)
No, no lo detectaría. El comando comprueba la integridad de los ficheros del sistema calculando su hash actual y comparándolo con los valores almacenados en la base de datos local. Si un atacante ha obtenido privilegios de root e instala un rootkit, simplemente abriría la libreta de dpkg y cambiaría la huella guardada por la huella nueva de su archivo malicioso, por lo que como ambos coinciden, cuando pasásemos sudo dpkg -V diría que todo está bien y no detectaría ninguna anomalía. 

## Parte C · Verificar una descarga: hash y firma
Descargaremos un programa pequeño y real, GNU Hello, junto a su firma y la lista de claves de GNU.

### C1. Descarga:
```mkdir -p ~/verif && cd ~/verif ``` 
wget https://ftp.gnu.org/gnu/hello/hello-2.12.1.tar.gz
wget https://ftp.gnu.org/gnu/hello/hello-2.12.1.tar.gz.sig
wget https://ftp.gnu.org/gnu/gnu-keyring.gpg

### C2. Calculad el hash:

``` sha256sum hello-2.12.1.tar.gz ``` 
Comparadlo con vuestro compañero de al lado. ¿Sale igual? Debería.

### C3. Verificad la firma usando la lista de claves de GNU:

```gpg --no-default-keyring --keyring ./gnu-keyring.gpg --verify hello-2.12.1.tar.gz.sig hello-2.12.1.tar.gz ```

Deberíais ver Good signature from .... También saldrá un aviso WARNING: This key is not certified with a trusted signature. Es normal, significa que vosotros no habéis dicho personalmente que esa clave sea de confianza. Anotad:

Nombre de quien firmó: 
Fecha de la firma: 
Identificador de la clave:

### C4. El experimento: ¿qué pasa si alguien toca el fichero? Haced una copia y añadid un carácter:

``` cp hello-2.12.1.tar.gz hello-manipulado.tar.gz ```
``` echo "x" >> hello-manipulado.tar.gz ``` 
``` sha256sum hello-2.12.1.tar.gz hello-manipulado.tar.gz ```
``` cp hello-2.12.1.tar.gz.sig hello-manipulado.tar.gz.sig ```
``` gpg --no-default-keyring --keyring ./gnu-keyring.gpg --verify hello-manipulado.tar.gz.sig hello-manipulado.tar.gz ```

Anotad: ¿cómo cambia el hash? ¿Qué dice ahora gpg? (Debe decir BAD signature.)

### C5. Pregunta C: completad:
Situación | ¿Basta con el hash? | ¿Hace falta la firma? ¿Por qué?
---------|--------------------|-------------------------------------


## Parte D · Política de actualizaciones para Torrent
Torrent tiene tres socios y nadie se encarga del mantenimiento. Redactad una política de máximo 6 líneas que responda:

1. ¿Quién se encarga de las actualizaciones y con qué frecuencia?
2. ¿Se actualiza todo automáticamente o primero se prueba? (Pensad en el equipo del servidor de impresión y en el ERP.)
3. ¿De dónde se puede instalar software y quién lo autoriza?
4. ¿Qué se hace con el equipo que ya no recibe parches (EOL)?

Para ver la opción automática en Ubuntu/Debian (solo mirad, no hace falta activarla):
``` apt policy unattended-upgrades ```
``` cat /etc/apt/apt.conf.d/20auto-upgrades 2>/dev/null ``` 

Debate: actualizar automáticamente reduce el riesgo de que Torrent esté desactualizado, pero una actualización puede romper el ERP un viernes por la tarde. ¿Qué ganáis y qué perdéis con cada opción?

Windows: los mismos conceptos existen allí con otros nombres. winver (versión), Windows Update (parches), Get-FileHash fichero (hash) y Get-AuthenticodeSignature fichero (firma de un ejecutable).
