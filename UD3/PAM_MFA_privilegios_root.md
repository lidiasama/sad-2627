# 0. Contexto y objetivos
En la sesión anterior configurasteis ACL para decidir qué puede leer, escribir o ejecutar cada grupo (operarios, ingenieria, supervisores) sobre hmi_planta y planos/. Eso resuelve el «qué puede hacer cada uno», pero da por hecho que el sistema sabe con certeza quién es quién. Hoy atacamos ese segundo problema: cómo nos aseguramos de que alberto es realmente alberto, y de que ni siquiera el personal de sistemas usa la cuenta root para el día a día.

Trabajaréis sobre la misma máquina y los mismos usuarios/grupos de la práctica de ACL. Como solo disponéis de openssh-server (sin claves públicas configuradas), la autenticación seguirá siendo por contraseña, reforzada con un segundo factor.
