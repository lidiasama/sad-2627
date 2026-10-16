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

Reflexiona: ¿qué pasaría si un atacante introduce una clave falsa en /usr/share/keyrings/? ¿y si intercepta o reemplaza la clave oficial de Ubuntu ubuntu-archive-keyrings.gpg?
