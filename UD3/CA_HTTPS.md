# Certificados y HTTPS: de nuestra propia CA a Let’s Encrypt
## 0. Contexto y objetivos
El servidor de Torrent S.L. (el que habéis ido montando con usuarios, ACL, contraseñas y MFA) publica una página web interna con el estado de la planta. Ahora mismo se sirve por HTTP: cualquiera conectado a la wifi del estudio podría leer lo que viaja, o incluso modificarlo por el camino. La solución es HTTPS, y para que funcione hace falta un certificado digital
Un certificado responde a una pregunta: «¿cómo sé que este servidor es realmente quien dice ser?». La respuesta depende de quién lo haya firmado y de si tu navegador se fía de esa entidad. Hoy lo vais a ver de dos formas distintas sobre el mismo servidor Apache:
- Parte 1: montáis la autoridad de certificación (CA) de Torrent S.L. y firmáis vosotros el certificado del servidor
- Parte 2: pedís un certificado real y gratuito a Let’s Encrypt, una CA pública, y comparáis las dos situaciones.


