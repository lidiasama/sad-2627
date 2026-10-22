## Parte A · ¿Está actualizado el sistema?
### A1. Anotad qué versión de sistema operativo y kernel tenéis:

``` lsb_release -a```
``` uname -r```

<img width="445" height="217" alt="image" src="https://github.com/user-attachments/assets/4331e074-51f8-4890-8d04-89fe2eb52c14" />

- Versión sistema operativo: Ubuntu 24.04 LTS (noble)
- Kernel: 7.0.0-38-generic
- 
### A2. Pedid al sistema la lista de paquetes disponibles y qué se puede actualizar:

```sudo apt update```
```apt list --upgradable```

- Anotad: ¿cuántos paquetes tienen actualización pendiente? 96 paquetes

  <img width="765" height="87" alt="image" src="https://github.com/user-attachments/assets/01e93e87-f421-425b-960f-bf49da1d2316" />


### A3. Simulad la actualización sin aplicarla:

``` sudo apt-get -s upgrade | head -30```
La opción -s es "simular". Fijaos en que dice qué instalaría, pero no toca nada.

<img width="816" height="712" alt="image" src="https://github.com/user-attachments/assets/6058a6d2-52de-44e3-8602-4ccdf7e95165" />


### A4. ¿Y si el sistema ya no recibe parches? Buscad en https://endoflife.date la fecha de fin de soporte de:

La versión de Ubuntu/Debian de vuestro Torrent-Auditor.
Ubuntu 8.04, la que tiene Torrent-Vulnerable (Metasploitable2).

<img width="913" height="127" alt="image" src="https://github.com/user-attachments/assets/86047543-7b30-46a4-bb48-985874a4dd65" />

<img width="887" height="123" alt="image" src="https://github.com/user-attachments/assets/e3582bf7-d1bf-44a8-b5b0-4f10813c2bad" />


Rellenad:
Sistema | Fecha de fin de soporte | ¿Recibe parches hoy?
--------|-------------------------|---------------------
Torrent-Auditor | Abril 2029 | Sí
Torrent-Vulnerable | Mayo 2013 | No


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

  <img width="710" height="118" alt="image" src="https://github.com/user-attachments/assets/40ac5fac-0dce-4315-b6ea-ceb821979520" />

### C2. Calculad el hash:

``` sha256sum hello-2.12.1.tar.gz ``` 
  <img width="810" height="67" alt="image" src="https://github.com/user-attachments/assets/ce35e3e1-ec03-4210-bc5a-61bdf5a3ecde" />

Comparadlo con vuestro compañero de al lado. ¿Sale igual? Debería.
- Sí, sale exactamente igual al de tus compañeros, ya que cualquier descarga íntegra del archivo original produce la misma cadena matemática.

### C3. Verificad la firma usando la lista de claves de GNU:

```gpg --no-default-keyring --keyring ./gnu-keyring.gpg --verify hello-2.12.1.tar.gz.sig hello-2.12.1.tar.gz ```
   <img width="807" height="202" alt="image" src="https://github.com/user-attachments/assets/48d6c18b-5ccb-4ec0-869d-26756db972e4" />


Deberíais ver Good signature from .... También saldrá un aviso WARNING: This key is not certified with a trusted signature. Es normal, significa que vosotros no habéis dicho personalmente que esa clave sea de confianza. Anotad:

Nombre de quien firmó: Reuben Thomas <rrt@sc3d.org>
Fecha de la firma: Sun 29 May 2022 11:05:00 PM UTC
Identificador de la clave: 3FD37230

### C4. El experimento: ¿qué pasa si alguien toca el fichero? Haced una copia y añadid un carácter:

``` cp hello-2.12.1.tar.gz hello-manipulado.tar.gz ```
``` echo "x" >> hello-manipulado.tar.gz ``` 
``` sha256sum hello-2.12.1.tar.gz hello-manipulado.tar.gz ```
``` cp hello-2.12.1.tar.gz.sig hello-manipulado.tar.gz.sig ```
``` gpg --no-default-keyring --keyring ./gnu-keyring.gpg --verify hello-manipulado.tar.gz.sig hello-manipulado.tar.gz ```
  <img width="807" height="110" alt="image" src="https://github.com/user-attachments/assets/2802ec22-6fd1-476c-8a37-589ea6c846e2" />

Anotad: 
- ¿cómo cambia el hash? ¿Qué dice ahora gpg? (Debe decir BAD signature.)
  Cambia por completo, con un efecto avalancha, produciendo una cadena totalmente distinta con solo añadir una letra. Ahora muestra BAD signature, indicando que la firma digital ya no es válida porque el contenido del archivo fue modificado tras firmarse.
  
### C5. Pregunta C: completad:
Situación | ¿Basta con el hash? | ¿Hace falta la firma? ¿Por qué?
---------|--------------------|-------------------------------------
El fichero se corrompió al descargarlo | Sí | No, el hash garantiza la integridad física. si el archivo se descarga a medias o pierde bytes, el hash local no coincidirá con el esperado y detectará el fallo
Un atacante cambió el fichero y el hash publicado en la misma web | No | Si, si el atacante tiene acceso para modificar la web, pondrá un ejecutable malicioso junto con el nuevo hash. Por lo que el hash por si solo no sirve porque coincidirá; la firma es imprescindible para garantizar la autenticidad
Descargas desde un espejo (mirror) que no es de confianza | No | Si, un mirror no oficial puede alojar un binario malicioso y mostrar su propio hash alterado. Hace falta la firma digital para verificar contra el llavero oficial de claves publicas que el archivo proviene legitimamente 

## Parte D · Política de actualizaciones para Torrent
Torrent tiene tres socios y nadie se encarga del mantenimiento. Redactad una política de máximo 6 líneas que responda:

1. ¿Quién se encarga de las actualizaciones y con qué frecuencia?
2. ¿Se actualiza todo automáticamente o primero se prueba? (Pensad en el equipo del servidor de impresión y en el ERP.)
3. ¿De dónde se puede instalar software y quién lo autoriza?
4. ¿Qué se hace con el equipo que ya no recibe parches (EOL)?

Se contrata a un proveedor externo de mantenimiento IT mensual, asignando internamente a Lucia Peris la supervisión quincenal de los informes. Los parches de seguridad críticos en cliente se aplican automáticamente, mientras que las actualizaciones de servidor y ERP deben validarse previamente en pruebas. Solo se permite la instalación desde repositorios oficiales y tiendas corporativas aprobadas, requiriendo autorización expresa de la dirección previa comprobación de firmas. Cualquier sistema descatalogado que no reciba actualizaciones debe ser migrado y/o sustituido.

Para ver la opción automática en Ubuntu/Debian (solo mirad, no hace falta activarla):
``` apt policy unattended-upgrades ```
``` cat /etc/apt/apt.conf.d/20auto-upgrades 2>/dev/null ``` 

  <img width="755" height="246" alt="image" src="https://github.com/user-attachments/assets/4b6a35db-aec6-450b-9b2a-150ad2dc21f6" />

Debate: actualizar automáticamente reduce el riesgo de que Torrent esté desactualizado, pero una actualización puede romper el ERP un viernes por la tarde. ¿Qué ganáis y qué perdéis con cada opción?
Actualizar automáticamente aporta seguridad inmediata, ya que corrige vulnerabilidades sin intervención humana, pero pone en riesgo la producción si un parche rompe el ERP. Validar las actualizaciones en un entorno aislado garantiza una mayor estabilidad y control, aunque mantiene los sistemas expuestos a vulnerabilidades conocidas mientras se comprueba. La solución sería automatizar los parches del sistema operativo y validar controlando el software del negocio

Windows: los mismos conceptos existen allí con otros nombres. winver (versión), Windows Update (parches), Get-FileHash fichero (hash) y Get-AuthenticodeSignature fichero (firma de un ejecutable).
