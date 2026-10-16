# Entendiendo y clasificando amenazas

## Parte A · Entendiendo las amenazas

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

## Parte B · Clasificar amenazas
Torrent ha tenido una semana movida. Clasificad cada incidente con el término más adecuado de esta lista (podéis usar alguno más de una vez):

virus · gusano · troyano · ransomware · spyware · keylogger · adware · rootkit · backdoor · bomba lógica · bot/zombi · phishing · pharming · hoax (bulo)

Para cada caso, además, responded: ¿Se replica solo? ¿Necesita que el usuario haga algo?
num | Indicente en Torrent | Tipo | ¿Se replica solo? | ¿Necesita al usuario?
--|----------------------|------|-------------------|----------------------
1 | Un socio abre factura_marzo.pdf.exe; poco después todo el disco compartido está cifrado y aparece un mensaje pidiendo 800 € | Ransomware | No | Sí
2 | Un "optimizador gratuito" de PC que, además, envía todo lo que se teclea a un servidor externo | Keylogger | No | Sí
3 | Sin que nadie haga nada, un programa salta de portátil en portátil a través de la wifi del estudio | Gusano | Sí | No
4 | Al abrir ciertos ejecutables, éstos infectan a su vez a otros ficheros del equipo | Virus | No | Sí
5 | Tras instalar un editor de PDF gratuito, aparecen ventanas de publicidad constantemente | Adware | No | Sí
6 | Un antiguo colaborador dejó un código que borrará los planos el 31 de diciembre | Bomba lógica | No | No
7 | El ordenador del estudio está enviando spam sin que nadie lo sepa, bajo las órdenes de un servidor externo | Bot | No | No
8 | Un conjunto de programas modificados que esconde procesos y ficheros del intruso dentro del propio sistema | Rootkit | No | No
9 | Un servicio escuchando en un puerto que permite entrar al equipo sin contraseña | Backdoor | No | No
10 | Correo: "Tu banco necesita que confirmes tus datos en este enlace" | Phising | No | No
11 | Mensaje: "Reenvía esto a 10 contactos o tu cuenta se cerrará" | Hoax | No | Sí
12 | Una web idéntica a la del ERP a la que se llega porque el DNS ha sido manipulado | Pharming | No | No
13 | Un programa que registra sin permiso los hábitos de navegación y los vende | Spyware | No | No

Pregunta A: ¿Cuáles de estos incidentes no son "malware" propiamente dicho, sino fraude o ingeniería social? ¿Por qué un antivirus no los detendría?
Los incidentes que no son malware propiamente dicho son: phising, hoax y pharming.
No los detendría porque son incidentes que ocurren por fallos humanos, que no tienen un ejecutable que poder escanear. Y el pharming tampoco lo detendría porque ocurre fuera de la jurisdicción de la empresa.
