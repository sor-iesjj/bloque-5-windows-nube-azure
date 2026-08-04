## Fase 3 · Apartado 4 — 🎯 ¿Dónde estamos?

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Conectividad VPN (WireGuard para Windows)**
> 🧭 Índice de la fase: [[Fase_3]]
>
> **📍 Cuándo se lee:** **Antes de empezar.** De dónde vienes y a dónde llegas.

---

> [!info] Vienes de Fase 2
> Completaste la preparación inicial del servidor y le diste identidad (`WindowsServer`), con IP privada fija en el rango `10.0.0.x` de Azure. Ahora tienes un servidor limpio, identificado, actualizado. Pero hay un problema crítico: está expuesto a internet público. El puerto RDP (3389) está abierto a todo el mundo desde la Fase 1 — bots intentarán conectarse miles de veces al día para adivinar contraseñas de administrador.

> [!warning] El Problema
> Sin una VPN privada, tu servidor es vulnerable a ataques de fuerza bruta contra el Escritorio Remoto. Cualquiera en internet puede intentar adivinar credenciales de administrador. Además, en las próximas fases necesitarás que el aula acceda al servidor desde cualquier lugar, pero solo el aula — no todo el mundo. Necesitas un túnel privado cifrado que solo tú controles.

> [!success] Objetivo de esta Fase
> Instalar **WireGuard para Windows**: una VPN ligera y moderna que crea un túnel P2P cifrado entre tu PC del aula (`10.0.0.2`) y el servidor (`10.0.0.1`). Este túnel es tu "puerta trasera" secreta — solo quien tenga las llaves criptográficas puede entrar. El puerto UDP 51820 que necesita ya está abierto desde la Fase 1; en esta fase construyes y verificas el túnel de extremo a extremo. **El cierre definitivo del RDP público** (Zero Trust) se aplicará más adelante, en la Auditoría Final, una vez que todos los servicios del proyecto estén desplegados y probados.

> [!tip] Hoja de Ruta
> 1. Comprobar que el puerto 51820 UDP ya está abierto en el NSG de Azure (lo abriste en la Fase 1)
> 2. Instalar WireGuard en el servidor
> 3. Generar pares de llaves criptográficas (servidor + tu PC)
> 4. Crear archivo de configuración `wg0.conf` en el servidor
> 5. Crear perfil VPN para tu PC del aula
> 6. Activar el túnel y verificar con `ping 10.0.0.1` desde tu PC
>
> **Resultado Final:** Túnel VPN cifrado y verificado entre el aula y el servidor. El endurecimiento final del acceso (cerrar RDP público) se hará en la Auditoría Final del proyecto, no aquí.
> **Siguiente:** Fase 4 (Dominio) — provisionar Active Directory Domain Services. Ahora que hay conexión VPN segura disponible, puedes instalar servicios críticos.

---

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_3.3_Obligaciones_Grabacion]] | [[Fase_3]] | [[Fase_3.5_Fundamento_Teorico]] |
