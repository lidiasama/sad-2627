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
  - De planos/ será Victor
  - Asignar propietario y grupo a planos/
    - ```sudo chown victor planos/```
      <img width="617" height="27" alt="image" src="https://github.com/user-attachments/assets/e34c2806-fbca-4239-b27f-b045d90f99ed" />

  - De hmi_planta será Lucia
  - Asignar propietario y grupo a hmi_planta
    - ```sudo chown lucia hmi_planta```
     <img width="638" height="26" alt="image" src="https://github.com/user-attachments/assets/5ce6af34-e7d9-42c6-b7e6-78077c597f2b" />
 
   
5.  Aplica con chmod unos permisos de partida restrictivos (700) de forma que el resto de acceso tenga que concederse explícitamente mediante ACL
- Asegurar permisos restrictivos 700 en los elementos raíz del ejercicio
  - ```sudo chmod 700 /srv/torrentsl/hmi_planta```
  - ```sudo chmod 700 /srv/torrentsl/planos```
  <img width="747" height="46" alt="image" src="https://github.com/user-attachments/assets/8349e47e-916e-4f83-a3fe-5183dcb11a85" />

- Asignar 700 a los ficheros dentro del directorio planos/
  - ```sudo chmod 700 /srv/torrentsl/planos/*```
  <img width="660" height="157" alt="image" src="https://github.com/user-attachments/assets/b562191d-17d1-4d64-ac3c-6e21397e7e8f" />


### Tarea 3 - Configurar las ACL con setfacl
Para cada fila que hayas completado en la tabla de la sección 3, añade una entrada ACL, correspondiente. Recuerda la sintaxis básica:

6. Aplica las ACL necesarias sobre hmi_planta.
- Grupo operarios: ejecutar (--x)
  - ```sudo setfacl -m g:operarios:--x hmi_planta```
- Grupo ingenieria: sin acceso (---)
  - ```sudo setfacl -m g:ingenieria:--- hmi_planta```
- Usuario nestor: leer y ejecutar (r-x)
  - ```sudo setfacl -m u:nestor:r-x hmi_planta```
- Usuario lucia: control total (rwx)
  - ```sudo setfacl -m u:lucia:rwx hmi_planta```
    <img width="760" height="92" alt="image" src="https://github.com/user-attachments/assets/e6892b0a-7d6a-4b4f-9055-e9cf1451c2e4" />


7. Aplica las ACL necesarias sobre la carpeta planos/ y, si procede, sobre los ficheros que ya contiene.

- Grupo operarios: sin acceso al directorio (---)
  - ```sudo setfacl -m g:operarios:--- planos/```
- Grupo ingenieria: sin acceso al directorio (---)
  - ```sudo setfacl -m g:ingenieria:--- planos/```
- Grupo supervisores: leer y entrar/recorrer la carpeta (r-x)
  - ```sudo setfacl -m g:supervisores:r-x planos/```
- Usuario victor: control total sobre el directorio (rwx)
  - ```sudo setfacl -m u:victor:rwx planos/```
    <img width="767" height="93" alt="image" src="https://github.com/user-attachments/assets/38a9fe2b-16a7-4328-a053-6296c5c9c80a" />


- Permisos de paso imprescindibles para que alberto, oscar y lucia puedan atravesar la carpeta planos/ hasta sus ficheros asignados:
  - sudo setfacl -m u:alberto:--x planos/
  - sudo setfacl -m u:oscar:--x planos/
  - sudo setfacl -m u:lucia:--x planos/
  <img width="707" height="76" alt="image" src="https://github.com/user-attachments/assets/eca8cd96-3f06-4eaf-b234-fad05f3f01e5" />

Ficheros individuales:
- alberto: lectura en autoclave.dwg (r--)
  - sudo setfacl -m u:alberto:r-- planos/autoclave.dwg
    <img width="846" height="22" alt="image" src="https://github.com/user-attachments/assets/6edccaba-08ff-40a5-ac0c-8b49a99917c0" />

- oscar: lectura en caldera.dwg (r--)
  - sudo setfacl -m u:oscar:r-- planos/caldera.dwg
    <img width="807" height="22" alt="image" src="https://github.com/user-attachments/assets/ab525bbc-9293-4415-96a3-aff44792cefb" />

- lucia: lectura y escritura en autoclave.dwg (rw-)
  - sudo setfacl -m u:lucia:rw- planos/autoclave.dwg
  <img width="803" height="25" alt="image" src="https://github.com/user-attachments/assets/5f0688b2-a23e-407b-a53f-735e8c58161e" />

8. Valora si alguno de los permisos del diagrama debería aplicarse también como ACL por defecto (setfacl-d), para que los ficheros nuevos que se creen en planos/ hereden automáticamente esos permisos.
  - sudo setfacl -d -m g:supervisores:r-x planos/
  - sudo setfacl -d -m u:victor:rwx planos/
    <img width="840" height="47" alt="image" src="https://github.com/user-attachments/assets/ff0046c2-0d4c-4496-811b-9f9ef0e16562" />


### Tarea 4 - Verificación
9. Comprueba el resultado con getfacl sobre cada recurso:
  - getfacl hmi_planta
    <img width="541" height="270" alt="image" src="https://github.com/user-attachments/assets/81fa2f14-5d03-4b3a-ad12-db953af962ae" />

  - getfacl -R planos/
    <img width="425" height="761" alt="image" src="https://github.com/user-attachments/assets/7c0edd5d-6bdf-4917-b003-c6b9e52da5e7" />

10. Accede como cada uno de los nueve usuarios (su -l usuario) y comprueba que puede hacer exactamente lo que el diagrama indica, y nada más.

- alberto: 
  - Permitido: Ejecutar hmi_planta y leer planos/autoclave.dwg.
  - Denegado: Modificar hmi_planta, listar el contenido de planos/, leer caldera.dwg o modificar autoclave.dwg
  <img width="787" height="507" alt="image" src="https://github.com/user-attachments/assets/e244d844-cb74-408a-b231-fa050a20a905" />

- oscar: 
  - Permitido: Ejecutar hmi_planta y leer planos/caldera.dwg.
  - Denegado: Leer autoclave.dwg, leer destilacion.dwg o modificar ficheros.
  <img width="710" height="375" alt="image" src="https://github.com/user-attachments/assets/3835a47a-6779-4e42-8c9f-085a14ee2ea0" />

- hugo:
  - Permitido: Únicamente ejecutar hmi_planta.
  - Denegado: Cualquier acceso a la carpeta planos/ y a sus archivos.
  	<img width="605" height="218" alt="image" src="https://github.com/user-attachments/assets/20ae17ac-a591-4edc-854a-45689f2fea77" />

- nestor: 
  - Permitido: Leer y ejecutar hmi_planta (r-x).
  - Denegado: Modificar hmi_planta y cualquier acceso a la carpeta planos/
  <img width="663" height="412" alt="image" src="https://github.com/user-attachments/assets/552e7299-6bce-4758-99e9-2a4725b56a30" />

- victor:
  - Permitido: Control total (rwx) sobre la carpeta planos/ (crear, listar, borrar ficheros).
  - Denegado: Acceso a hmi_planta
  <img width="720" height="335" alt="image" src="https://github.com/user-attachments/assets/1c56d4f3-ba93-4a69-a0dd-e50a9ea3c2dc" />

- lucia:
  - Permitido: Control total (rwx) sobre hmi_planta y lectura/escritura (rw-) sobre planos/autoclave.dwg.
  - Denegado: Acceso directo a otros ficheros dentro de planos/ (caldera.dwg o destilacion.dwg)
  <img width="832" height="462" alt="image" src="https://github.com/user-attachments/assets/362ec5c4-d8ff-4b45-81b0-2b4d04b8d73d" />

- carlos, mary y john:
  - Permitido: Entrar y listar la carpeta planos/ (r-x), y consultar los ficheros planos con lectura.
  - Denegado: Modificar o crear ficheros dentro de planos/ y acceder a hmi_planta (según tabla de permisos).
    <img width="668" height="397" alt="image" src="https://github.com/user-attachments/assets/8d676307-b4d3-4a59-ac06-83bdfe2f38b7" />


## 6. Preguntas de reflexión:
1. ¿Por qué no bastan los permisos clásicos de Unix (usuario / grupo / resto) para implementar el esquema del diagrama? Pon un ejemplo concreto de este caso.
Porque

2. ¿Qué es la máscara (mask) de una ACL y cómo afecta a los permisos efectivos de un usuario o grupo, aunque su entrada de ACL indique más permisos?


3. ¿Qué diferencia hay entre una ACL de acceso y una ACL por defecto? ¿Cuándo se aplica cada una?


4. Si mañana se incorpora un nuevo operario, ¿qué pasos tendrías que repetir para darle exactamente el mismo acceso que al resto de operarios?



