# Práctica 1 - Perímetro y zonas
Torrent S.L. quiere ordenar su red antes de abrir ningún acceso remoto. En esta práctica decides dónde va cada servicio y qué protocolo se usa en cada caso, y lo justificas.

## Minirreto 1: coloca cada servicio en su zona
**Zonas disponibles**
Zona | Confianza | Idea
-----|-----------|------
Internet | No confiable | Cualquiera, sin control
DMZ | Semiconfiable | Servicios expuestos a Internet, aislados de la LAN
LAN | Confiable | Usuarios y equipos propios
Zona crítica | Restringida | Archivos de alto valor con pocos accesos

Si un servicio no encaja en ninguna, créale una zona propia y defiende por qué.
**Servicios de Torrent**
Servicio | Zona | Justificación (una frase)
---------|------|----------------------
Web corporativa pública | DMZ | Debe estar disponible desde Internet
Servidor de correo | LAN | 
Pasarela VPN | DMZ | 
Proxy (saliente e inverso) | DMZ | 
HMI de planta y carpeta de planos | Zona crítica | Son archivos muy controlados
Autoridad de certificación (srv-ca) |  Zona crítica | 
Servidor de copias de seguridad | Zona crítica | 
Portátiles de ingenieros y operarios | LAN | 
Wi-Fi de invitados | LAN | 
Servidor de registros y monitorización | LAN | 
