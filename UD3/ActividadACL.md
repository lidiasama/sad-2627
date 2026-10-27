# Configuración de ACL en Linux
#### Caso Torrent S.L. — control de acceso 

## 1. Contexto
Torrent S.L. dispone de una aplicación de control de planta, hmi_planta, y de una carpeta compartida con los
planos de los equipos industriales, planos/, que contiene los ficheros autoclave.dwg, caldera.dwg y
destilacion.dwg.

El departamento de sistemas te ha encargado configurar el acceso a estos dos recursos para tres grupos de
personal, siguiendo exactamente el esquema de permisos representado en el siguiente diagrama. Los permisos
clásicos de Unix (propietario / grupo / resto) no son suficientes para representar este esquema: necesitarás
Listas de Control de Acceso (ACL)

## 2. Diagrama de permisos a implementar
<img width="535" height="371" alt="image" src="https://github.com/user-attachments/assets/700e5adc-1bd3-43b2-8220-7b036937d075" />

## 3. Usuarios y grupos
Grupo | Usuarios
------|---------
operarios | alberto, oscar, hugo
ingenieria | nestor, victor, lucia
supervisores | carlos, mary, john

**Tabla de permisos**
r=leer, w=escribir, x=ejecutar
Grupo/Usuario | Recurso | Permisos 
------|----------|---------
operarios | hmi_planta | --x
operatios | planos/ | --- 
ingenieria | hmi_planta | --- 
ingenieria | planos/ | --- 
supervisores | planos/ | r-x
alberto | planos/autoclave.dwg | r--
oscar | planos/caldera.dwg | r--
nestor | hmi_planta | r-x
victor | /planos | rwx
lucia | hmi_planta | rwx
lucia | /planos/autoclave.dwg | rw-


## 4. Tareas a realizar
### Tarea 1 - Preparar el entorno
1. Comprueba que el soporte de ACL está disponible en tu sistema: ```dpkg -s acl ```
- <img width="740" height="372" alt="image" src="https://github.com/user-attachments/assets/e6af8e9f-c5f1-45ca-b884-0169acfaf85e" />

3. Crea los 3 grupos y los nueve usuarios del enunciado, asignando a cada usuario su grupo correspondiente.
``` sudo useradd -m alberto ```
``` sudo useradd -m oscar ```
``` sudo useradd -m hugo ```
``` sudo useradd -m nestor ```
``` sudo useradd -m victor ```
``` sudo useradd -m lucia ```
``` sudo useradd -m carlos ```
``` sudo useradd -m mary ```
``` sudo useradd -m john ```
``` sudo groupadd operarios ```
``` sudo groupadd ingenieria ```
``` sudo groupadd supervisores ```
``` sudo usermod -a -G operarios alberto```
``` sudo usermod -a -G operarios oscar```
``` sudo usermod -a -G operarios hugo```
``` sudo usermod -a -G ingenieria nestor```
``` sudo usermod -a -G ingenieria victor```
``` sudo usermod -a -G ingenieria lucia```
``` sudo usermod -a -G supervisores carlos```
``` sudo usermod -a -G supervisores mary```
``` sudo usermod -a -G supervisores john```

5. Crea la estructura de ficheros: un fichero (o script) ejecutable llamado hmi_planta(script con echo), y una carpeta llamada planos/ con los ficheros autoclave.dwg, caldera.dwg y destilacion.dwg. Todo dentro de /srv/torrentsl/planos/
- Primero creamos la ruta y la subcarpeta: ```sudo mkdir -p /srv/torrentsl/planos ```
- Nos situamos en el directorio: ``` cd /srv/torrentsl/planos/ ```
- Creamos el script hmi_planta con un echo en /srv/torrentsl/: ```sudo nano hmi_planta```
  <img width="590" height="82" alt="image" src="https://github.com/user-attachments/assets/3d99c862-abf6-4be8-8cad-5865d5dc47b5" />

- Creamos los tres ficheros dentro de la subcarpeta /planos: ```sudo touch planos/autoclave.dwg planos/caldera.dwg planos/destilacion.dwg ```

- Verificamos con ``` ls ```
- <img width="521" height="108" alt="image" src="https://github.com/user-attachments/assets/f4f14ce8-c5e9-44b5-b26e-44a0600a13d4" />

### Tarea 2 - Permisos base (propietario/grupo/resto)
4. Decide qué usuario y qué grupo deben ser propietarios de hmi_planta y de planos/.
   
6. Aplica con chmod unos permisos de partida restrictivos (700) de forma que el resto de acceso tenga que concederse explícitamente mediante ACL

### Tarea 3 - Configurar las ACL con setfacl
Para cada fila que hayas completado en la tabla de la sección 3, añade una entrada ACL, correspondiente. Recuerda
