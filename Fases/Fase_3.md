## 🔒 Fase 3: Conectividad VPN (WireGuard para Windows)

### Infraestructura de Servidores Cloud (Azure + Windows Server 2025)

> **[Módulo: SOR — Sistemas Operativos en Red]**
> **[U.T. 9: Gestión remota e Integración en Red]**
> **[RA.05]** Realiza tareas de monitorización y uso del sistema operativo en red.
>
> **Profesor:** Pedro Navarro Miralles  
> **Correo:** p.navarromiralles2@edu.gva.es  
> **Centro:** IES Jorge Juan (ALICANTE)
>
> **⏱️ Tiempo estimado:** ~2 horas (teoría + práctica + retos + troubleshooting)  
> **Requisitos:** 16 GB RAM (VM `Standard_B4ms`) | WireGuard para Windows (PC del aula y servidor) | Azure Portal | RDP

---

### 🎯 ¿Dónde Estamos?

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

### 📚 Fundamento Teórico

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

### 🛠️ Procedimiento Práctico (BoochanV2.1)

> [!example] Paso 1: Instalación y Generación de Llaves Criptográficas del Servidor
> Descarga e instala WireGuard en el servidor desde dentro de la sesión RDP.
>
> > [!info] 📚 Diccionario de Comandos: Para entender la sintaxis exacta de `wg.exe` y repasar otros comandos, consulta el [[Diccionario_Comandos_Sistema]].
>
> 1. Desde dentro de la VM, abre un navegador y ve a `https://www.wireguard.com/install/`.
> 2. Descarga el instalador oficial para Windows (`.msi`) y ejecútalo con permisos de administrador, aceptando las opciones por defecto.
> 3. Al terminar, se abrirá la aplicación **WireGuard** con una lista vacía de túneles.
>
> Genera las llaves del servidor por línea de comandos, para mantener el paralelismo con BoochanV2 y facilitar la reproducibilidad:
> ```powershell
> # Crea la carpeta de configuración si no existe
> New-Item -ItemType Directory -Path "C:\WireGuard" -Force
> cd C:\WireGuard
>
> # Genera la llave privada y, a partir de ella, la llave pública
> wg genkey | Out-File -Encoding ascii privatekey
> Get-Content privatekey | wg pubkey | Out-File -Encoding ascii publickey
> ```
> Ahora **lee y anota** la llave pública del servidor. La necesitarás cuando configures el cliente en el Paso 3:
> ```powershell
> Get-Content C:\WireGuard\publickey
> ```
>
> > [!tip] 💡 ¿Qué hace este comando?
> > - **El Pipe (`|`):** Igual que en Linux, encadena comandos: la salida de uno entra directamente al siguiente.
> > - **`wg genkey` / `wg pubkey`:** Son los mismos binarios `wg` que en Linux — WireGuard incluye herramientas de línea de comandos idénticas en ambas plataformas.
> > - **Permisos del archivo:** A diferencia de Linux (`umask 077`), Windows no gestiona permisos de archivo con ese mecanismo. Por buena práctica, asegúrate de que la carpeta `C:\WireGuard` no está compartida ni es accesible por otros usuarios del sistema.

> [!example] Paso 2: Configuración del Túnel en el Servidor (`wg0.conf`)
> Crea el archivo de configuración del túnel:
>
> > [!info] 📚 Recurso: Para editar texto rápido en Windows Server usa el Bloc de notas (`notepad`) — no existe `nano` como en Linux; abre el fichero con `notepad C:\WireGuard\wg0.conf`.
>
> ```powershell
> notepad C:\WireGuard\wg0.conf
> ```
> Escribe este contenido. Sustituye `<CONTENIDO_DE_TU_PRIVATEKEY>` por el valor del archivo `privatekey`:
> ```ini
> [Interface]
> PrivateKey = <CONTENIDO_DE_TU_PRIVATEKEY>
> Address = 10.0.0.1/24
> ListenPort = 51820
>
> [Peer]
> PublicKey = <LLAVE_PÚBLICA_DEL_CLIENTE_AULA>
> AllowedIPs = 10.0.0.2/32
> ```
> Guarda y cierra Notepad. Deja el campo `<LLAVE_PÚBLICA_DEL_CLIENTE_AULA>` como está por ahora; lo completarás en el Paso 4 una vez que generes las llaves del cliente.
>
> Abre el puerto UDP en el Firewall de Windows para que el tráfico WireGuard pueda llegar al servidor:
> ```powershell
> New-NetFirewallRule -DisplayName "WireGuard VPN" -Direction Inbound -Protocol UDP -LocalPort 51820 -Action Allow
> ```

> [!example] Paso 3: Instalación y Configuración del Cliente (PC del Aula)
> El túnel VPN necesita dos extremos configurados. Ahora le toca al **PC de tu aula**:
>
> **1. Instala la aplicación WireGuard en tu PC:**
> - **Windows:** Ve a `wireguard.com/install`, descarga el instalador `.exe` y ejecútalo.
> - **Mac:** Búscalo en la App Store buscando "WireGuard" o descárgalo desde `wireguard.com/install`.
>
> **2. Crea un nuevo túnel y obtén las llaves del cliente:**
> - Abre la aplicación WireGuard.
> - Haz clic en **"Agregar túnel"** → **"Crear nuevo túnel vacío"** (en Mac: icono `+`).
> - WireGuard genera automáticamente las llaves del cliente. Verás la **Clave Pública** del cliente en la parte superior del cuadro de configuración.
> - **Copia y anota esa Clave Pública**: la necesitarás en el servidor.
>
> **3. Completa el archivo de configuración del cliente** con este contenido:
> ```ini
> [Interface]
> PrivateKey = <SE_RELLENA_AUTOMÁTICAMENTE_por_WireGuard>
> Address = 10.0.0.2/32
> DNS = 10.0.0.1
>
> [Peer]
> PublicKey = <LLAVE_PÚBLICA_DEL_SERVIDOR_del_Paso_1>
> AllowedIPs = 10.0.0.0/24
> Endpoint = TU_IP_PUBLICA_AZURE:51820
> PersistentKeepalive = 25
> ```
>
> > [!important] 💡 ¿Qué es `PersistentKeepalive`?
> > Azure cierra las conexiones que están inactivas. Este parámetro hace que el cliente envíe un pequeño "pulso" cada 25 segundos para mantener el túnel vivo aunque no haya tráfico real. Sin esta línea, la VPN se desconectaría sola a los pocos minutos.

> [!example] Paso 4: Intercambio de Llaves y Activación
> Vuelve a la sesión RDP del servidor y completa el archivo `wg0.conf` con la llave pública del cliente que anotaste en el Paso 3:
> ```powershell
> notepad C:\WireGuard\wg0.conf
> ```
> Sustituye `<LLAVE_PÚBLICA_DEL_CLIENTE_AULA>` por la llave pública real de tu PC. Guarda y cierra.
>
> > [!caution] ⚠️ Atención al Portapapeles (Copia-Pega)
> > Al borrar el texto de ejemplo `<LLAVE...>`, asegúrate de eliminar también los símbolos `<` y `>`. Un espacio extra, un salto de línea invisible o una letra comida arruinará la conexión VPN de forma silenciosa.
> >
> > **Antes de guardar**, verifica que la clave quedó bien pegada ejecutando:
> > ```powershell
> > Select-String -Path C:\WireGuard\wg0.conf -Pattern "PublicKey"
> > ```
> > La salida debe ser una sola línea limpia, sin espacios al principio ni al final, parecida a esto:
> > ```
> > PublicKey = aBcDeFgHiJkLmNoPqRsTuVwXyZ1234567890abcde=
> > ```
> > Si ves dos líneas, espacios raros o caracteres `<` o `>` sueltos, vuelve a editar el archivo antes de continuar.
>
> Ahora instala el túnel como servicio de Windows y actívalo:
> ```powershell
> # Instala el túnel como servicio (arranca automáticamente con el sistema, equivalente a systemctl enable)
> wireguard.exe /installtunnelservice C:\WireGuard\wg0.conf
> ```
>
> > [!tip] 💡 ¿Qué hace `/installtunnelservice`?
> > Es el equivalente Windows a `sudo wg-quick up wg0` seguido de `sudo systemctl enable wg-quick@wg0` en un solo comando: levanta el túnel inmediatamente **y** lo registra como servicio persistente que arrancará automáticamente en cada reinicio del servidor.
>
> **En el PC cliente (aula):** Activa el túnel haciendo clic en el botón **"Activar"** de la aplicación WireGuard.
>
> Verifica que el túnel está activo. En el servidor:
> ```powershell
> wg show
> ```
> Y desde el terminal de tu PC del aula:
> ```powershell
> # Si recibes respuestas, el túnel funciona correctamente
> ping 10.0.0.1
> ```
>
> > [!important] 🔒 VPN activa: ya puedes preferir el túnel para administrar el servidor
> > El túnel funciona. A partir de ahora puedes conectarte por RDP usando la IP del túnel en lugar de la IP pública de Azure — es la ruta más segura y la que usarán las siguientes fases como referencia:
> > ```
> > mstsc /v:10.0.0.1
> > ```
> > La IP pública seguirá funcionando también hasta la Auditoría Final, donde se cerrará definitivamente el acceso directo. No hace falta que hagas nada más de seguridad en esta fase.

---

### 🚩 Resolución de Problemas y Evaluación

> [!bug] Troubleshooting (¿No hay conexión?)
> | Problema | Causa Probable | Solución Sugerida |
> | :--- | :--- | :--- |
> | `wireguard.exe /installtunnelservice` falla con "Address already in use". | Ya hay otra interfaz VPN activa con esa IP. | Desinstala el servicio con `wireguard.exe /uninstalltunnelservice wg0` antes de volver a instalarlo. |
> | No hay ping entre `10.0.0.1` y `10.0.0.2`. | La regla `WireGuard` (51820 UDP) de la Fase 1 se deshabilitó o se creó como TCP por error. | Revisa en el NSG de Azure que la regla `WireGuard` esté en `Permitir` y su protocolo sea **UDP**. |
> | WireGuard no conecta pero el puerto está abierto. | Las llaves públicas están intercambiadas incorrectamente, o el Firewall de Windows Defender bloquea el puerto 51820/UDP. | Verifica las llaves públicas cruzadas. Comprueba también `Get-NetFirewallRule -DisplayName "WireGuard VPN"` en el servidor. |
> | El cliente no encuentra el `Endpoint`. | Escribiste mal la IP pública de Azure, o la VM no está encendida. | Comprueba la IP pública en el portal de Azure y confirma que el servicio `WireGuardTunnel$wg0` está activo (`Get-Service`). |

> [!help] Preguntas Críticas (Autoevaluación)
> 1. ¿Por qué la llave privada **NUNCA** debe salir de tu servidor ni enviarse por correo?
> 2. ¿Qué diferencia hay entre instalar WireGuard "solo activado en la app" y hacerlo como Tunnel Service con `/installtunnelservice`?
> 3. ¿Para qué sirve el parámetro `AllowedIPs` en la configuración del Peer?
> 4. 🔬 **Reto práctico:** Con el túnel activo, ejecuta `wg show` en el servidor y localiza la línea `latest handshake`. ¿Hace cuántos segundos fue el último intercambio? Ahora desactiva el túnel desde tu PC del aula y vuelve a ejecutar el comando 30 segundos después. ¿Qué cambió en esa línea?
> 5. 🔬 **Reto práctico:** Con el túnel WireGuard **desactivado** en tu PC, intenta conectarte al servidor por RDP usando la IP pública de Azure. ¿Puedes entrar? Deberías poder — en esta fase el RDP público sigue abierto a propósito. Reflexiona: ¿qué riesgo concreto sigue existiendo mientras no llegues a la Auditoría Final del proyecto, y por qué crees que este itinerario pospone el cierre del 3389 en vez de hacerlo ya?

---

> [!caution] 🛑 Auditoría y Seguridad (RA.05)
> Las llaves privadas son la **identidad** de tu servidor. Si un atacante las copia, podrá entrar en tu red privada como si fuera tú. **Validación:** El alumno debe demostrar el `ping 10.0.0.1` desde el cliente del aula y el `wg show` en el servidor mostrando el peer conectado.
