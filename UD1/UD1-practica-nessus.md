# Informe de auditoría de vulnerabilidades 

## 1. Contexto y alcance
Se ha realizado una auditoría previa a la contratación de un servicio de mantenimiento informático para la empresa Estudio Técnico Torrent S.L.
La empresa requiere conocer el nivel de seguridad real de su insfraestructura antes de delegar la administración.

## 2. Metodología
Se ha utilizado Tenable Nessus, uno de los escáneres de vulnerabilidades estándar en la industria. Su motor permite identificar configuraciones deficientes, servicios desactualizados, puertos expuestos y vulnerabilidades asociadas a identificadores CVE.
Se ha realizado un escaneo básico de red, donde se ha configurado una tarea de análisis de red no autenticada. 

## 3. Resumen de resultados
Severidad | Puntuación | Nº vulnerabilidades
----------|------------|--------------------
Crítica | 10.0 - 9.0 | 7
Alta | 8.9 - 7.0 |  4
Media | 6.9 - 4.0 |  11
Baja | 3.9 - 0.1 |  6
Informativa | 0.0 | 110

<img width="717" height="87" alt="image" src="https://github.com/user-attachments/assets/0cafedc6-92d1-4b98-bccd-b024a0cb1210" />


## 4. Vulnerabilidades clasificadas
Vulnerabilidad | Severidad / CVSS | Origen (diseño / implantación / uso) | Breve descripción
---------------|------------------|--------------------------------------|-------------------
VNC Server 'password' Password | 10.0* | USO | El servidor VNC que se ejecuta en el host remoto está protegido con una contraseña débil. Un atacante remoto no autenticado que tenga acceso a la red puede explotar esto para tomar control del sistema
NFS Shares World Readable | 7.5 | Implantación | El servicio de carpetas compartidas en red NFS exporta directorios con permisos de lectura abiertos para cualquier dispositivo de la red. Esto permite que un equipo no autorizado pueda entrar a directorios compartidos y acceda a archivos y configuraciones sensibles del sistema.
TLS Version 1.0 Protocol Detection | 6.5 | Diseño | El servidor admite conexiones cifradas bajo el protocolo TLS 1.0, que usa algoritmos obsoletos y mecanismos de cifrado por bloques vulnerables a ataques conocidos.

## 5. Análisis en profundidad
Vulnerabilidad: VNC Server 'password' Password.

¿Qué es? Es un servicio de escritorio remoto configurado con una contraseña débil y por defecto, concretamente 'password' para controlar el sistema.

¿Cómo se puede explotar? Si una persona tiene acceso al puerto 5900 /tpc, con un visor de VNC puede introducir la contraseña y obtener el control del sistema.

¿Cómo se mitigaría? Habría que cambiar la contraseña por una más robusta, restringir el acceso al puerto y deshabilitar el servicio si no es necesario.

Referencia CVE o nombre del plugin de Nessus que la detectó: Plugin ID 61708

Severidad y puntuación: Crítica (Critical), con puntuación base CVSS v2.0 de 10.0.   
Vector CVSS v2.0: CVSS2#AV:N/AC:L/Au:N/C:C/I:C/A:C.   

Métrica | Sigla y Valor | Significado 
--------|--------------|-------------
Attack Vector |AV:N | Network (red)
Attack Complexity | AC:L (eq PR:N) | Low (baja)
Authentication (v2) / Privileges (v3) | Au:N | None
User Interaction | (implicito N) | None
Confidencialidad | C:C | Complete
Integridad | I:C | Complete
Disponibilidad | A:C | Complete

- La etiqueta Exploited by Nessus: true en el panel de detalles demuestra que el hallazgo no es un falso positivo teórico, sino un fallo validado en tiempo real al lograr autenticación exitosa sobre el puerto 5900

<img width="1169" height="691" alt="image" src="https://github.com/user-attachments/assets/2bc0e6d5-c4e0-44ce-95a4-99264bc70065" />

## 6. Recomendaciones
1. Modificar TODAS las contraseñas débiles y establecer una política de contraseñas robustas, prohibiendo anotarlas.
2. Parar y deshabilitar los servicios y puertos que no sean utilizados en el día a día.
3. Planificar la sustitución del sistema operativo Ubuntu que tienen por una versión más moderna y con parches de seguridad.

## 7. Conclusión
Si que lo firmaría, pero con ciertas condiciones. La idea de contratar una empresa es para mantener la infraestructura, pero con el estado que está el servidor un mantenimiento normal no sirve de mucho. Porque este sería para cuidar un sistema que funciona bien.

Por eso, aceptaría firmar si se cumplen las siguientes condiciones: primero se debe hacer un trabajo previo de puesta a punto para migrar el sistema a una versión más actualizada con soporte, y realizar ciertas medidas de seguridad. Y segundo, que el segundo contrato de mantenimiento no se comience hasta que no se realice otro escaneo con Nessus y se compruebe que las vulnerabilidades se han solucionado.

