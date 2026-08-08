## Fase 3 · Apartado 5 — 📚 Fundamento teórico

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Conectividad VPN (WireGuard para Windows)**
> 🧭 Índice de la fase: [[Fase_3]]
>
> **📍 Cuándo se lee:** **Antes de teclear.** Los conceptos que necesitas.

---

> [!abstract] 1. El Dilema de la Nube
> La conectividad en la nube presenta un gran reto: queremos administrar nuestro servidor desde cualquier parte, pero no queremos exponerlo a ataques de todo el mundo. La solución es crear un **Túnel VPN P2P (Peer-to-Peer)**.

> [!info] 2. ¿Qué es WireGuard? (y qué cambia en Windows respecto a Linux)
> A diferencia de protocolos antiguos (como OpenVPN), WireGuard funciona a muy bajo nivel del sistema operativo. En Linux se integra directamente en el Kernel; en **Windows**, la aplicación oficial `WireGuard.exe` incluye su propio driver de red (Tunnel Service) que consigue un rendimiento equivalente, gestionado a través de una interfaz gráfica o de `wireguard.exe /installtunnelservice`. Utiliza **criptografía de curva elíptica**, asegurando que los datos viajen por un canal 100% blindado.

> [!important] 3. Intercambio de Llaves
> El servidor y el cliente se reconocen mediante un intercambio de llaves:
> *   **Llave Pública:** Se puede compartir (es como la dirección de tu casa).
> *   **Llave Privada:** Es el secreto absoluto. Solo quien posee la llave privada puede descifrar el tráfico que le llega.

> [!note] 4. Dos redes, dos propósitos: no confundas la IP privada de Azure con la del túnel
> En este proyecto conviven dos rangos de IP que no deben mezclarse:
> *   **`10.0.0.x` (real, de Azure):** La IP privada que el servidor tiene en la subred de la VNet de Azure (la fijaste en la Fase 2). Es la "tarjeta de red física" virtual de la VM.
> *   **`10.0.0.1` / `10.0.0.2` (virtual, del túnel WireGuard):** La red que crea la propia VPN al levantar la interfaz `wg0`. Aunque coincide en apariencia con el rango de Azure, es una interfaz de red completamente distinta, gestionada por WireGuard, que viaja **encapsulada y cifrada** dentro del tráfico real de la VNet. A partir de esta fase, cuando veas `10.0.0.1` en un comando, será casi siempre la IP del túnel, no la de la NIC de Azure.

### 📖 Diccionario de Conceptos Clave

> [!quote] Terminología VPN
> - **Cifrado Asimétrico:** Sistema que usa una llave para cerrar (pública) y otra distinta para abrir (privada).
> - **wg0.conf:** El "cerebro" o archivo maestro que define la red virtual y quién puede entrar en ella.
> - **Peer:** Cada uno de los extremos de la conexión (tu PC del aula y el Servidor en Azure son "Peers").
> - **Endpoint:** La dirección IP pública real del servidor a la que se conecta el túnel.
> - **Tunnel Service:** El servicio de Windows que WireGuard instala para mantener el túnel activo incluso sin sesión de usuario iniciada, similar a `systemctl enable wg-quick@wg0` en Linux.

---

### 🔓 Puertos: qué ya está abierto y qué falta

> [!example] 🎬 Antes de empezar (todavía SIN grabar, y luego arranca)
> Ya conoces el método desde los prerrequisitos, así que va solo el recordatorio:
> 1. **Crea la entrada de apuntes** de esta fase (`b5-azure-3-conectividad-vpn-wireguard-para-windows.md`) con su estructura, vacía.
> 2. **Léete los 5 pasos** del procedimiento enteros, para no atascarte a mitad del vídeo.
> 3. Ten **OBS** listo y comprueba **pantalla y micrófono**.
>
> Cuando lo tengas: **arranca la grabación, preséntate y muestra tu identidad**. A partir de ahí, **todo queda grabado** — incluido cualquier paso previo de preparación que venga a continuación.

> [!example] El puerto de WireGuard ya está abierto desde la Fase 1
> A diferencia de BoochanV2 (donde el NSG se iba ampliando fase a fase), en BoochanV2.1 el NSG completo del proyecto se preparó de una sola vez en la Fase 1 — incluido el puerto que esta fase necesita:
>
> | Prioridad | Nombre | Puerto | Protocolo | Para qué sirve ahora |
> | :--- | :--- | :--- | :--- | :--- |
> | **390** | WireGuard | 51820 | **UDP** | Canal cifrado del túnel VPN entre el aula y el servidor. |
>
> No necesitas volver al portal de Azure para esta fase. Si quieres comprobarlo, entra en **`Configuración de red`** de tu VM → tu NSG → **`Reglas de seguridad de entrada`** y confirma que la regla `WireGuard` (51820/UDP) está en estado `Permitir`.
>
> > [!warning] ⚠️ Este puerto es UDP, no TCP
> > Es el error más habitual al configurar WireGuard. WireGuard usa UDP porque necesita velocidad, no garantía de orden — igual que una videollamada. Si por error la regla se creó como TCP, la VPN no conectará aunque todo lo demás esté perfectamente configurado.

> [!info] El cierre del RDP público llega más adelante, en la Auditoría Final
> Podría parecer lógico cerrar ahora mismo el puerto 3389 al mundo exterior, ya que el túnel VPN va a dejarlo obsoleto. En este proyecto lo posponemos deliberadamente hasta la **Auditoría Final**, al término de todas las fases: así puedes seguir usando la conexión RDP directa por la IP pública como red de seguridad mientras construyes el resto de la infraestructura (dominio, usuarios, almacenamiento, cliente Windows 11), y solo al final aplicas el modelo **Zero Trust** completo, con las dos capas de defensa (NSG + Firewall de Windows) documentadas y verificadas de una sola vez. Por ahora, comprueba que el túnel funciona y sigue usando el acceso que te resulte más cómodo.

---

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_3.4_Donde_Estamos]] | [[Fase_3]] | [[Fase_3.6_Procedimiento]] |
