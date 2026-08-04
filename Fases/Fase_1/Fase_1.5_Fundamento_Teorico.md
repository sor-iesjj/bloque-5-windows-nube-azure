## Fase 1 · Apartado 5 — 📚 Fundamento teórico

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Infraestructura Cloud (Azure IaaS — Windows Server 2025)**
> 🧭 Índice de la fase: [[Fase_1]]
>
> **📍 Cuándo se lee:** **Antes de teclear.** Los conceptos que necesitas.

---

> [!info] ¿Por qué usamos la Nube (IaaS)?
> El concepto de **IaaS (Infraestructura como Servicio)** es el primer pilar de la administración moderna. Tradicionalmente instalaríamos Windows Server introduciendo un DVD o un USB en una máquina física en el aula. En este proyecto damos el salto profesional: en lugar de tocar un ordenador físico, alquilamos recursos en centros de datos masivos de Microsoft.

> [!abstract] 1. La "Magia" del Hipervisor y la Virtualización
> El **Hipervisor** es una capa de software de bajo nivel que gestiona los recursos de un servidor físico real y te entrega una porción exacta de CPU, RAM y disco.
> - **Tu servidor:** Cree que es un ordenador físico completo.
> - **La Realidad:** Es un archivo ejecutándose dentro de otro ordenador gigante. Esto permite que un solo superordenador de Azure albergue cientos de servidores de alumnos de forma aislada.

> [!warning] 2. El Modelo de Responsabilidad Compartida
> Trabajar en la nube no significa que "todo es mágico y seguro". Azure funciona bajo este modelo:
> * **Responsabilidad de Microsoft:** Seguridad física del centro de datos, electricidad y conexión a Internet.
> * **Tu Responsabilidad (El Administrador):** Eres el responsable absoluto de lo que ocurre **dentro** de tu máquina virtual. Si configuras mal una contraseña o dejas un puerto abierto, los hackers entrarán. ¡Microsoft no te protegerá de tus propios errores de configuración!

> [!important] 3. Windows Server con Interfaz Gráfica: por qué pesa más que Ubuntu Server
> En BoochanV2 (Ubuntu) usábamos un servidor **"Headless"** (sin entorno gráfico) para ahorrar RAM. Aquí hacemos una excepción deliberada: instalamos **Windows Server 2025 con Desktop Experience** (interfaz gráfica), porque es tu primera vez administrando Active Directory de forma nativa y necesitas ver herramientas como `Administrador del servidor`, `Usuarios y equipos de Active Directory` o el `Administrador de DNS` mientras aprendes.
> * **Consecuencia práctica:** Windows Server + Desktop Experience + AD DS necesita bastante más RAM que un Ubuntu Server headless. Por eso en BoochanV2.1 usamos `Standard_B4ms` (4 vCPU, **16 GB** de RAM) en lugar de la `Standard_B2s` (2 vCPU, 4 GB) que bastaba en la versión Ubuntu.

> [!note] 4. Seguridad Perimetral y Protocolos (TCP vs UDP)
> Antes de encender el servidor, lo protegemos con un **NSG (Network Security Group)**, que actúa como la muralla del castillo. Solo abriremos las "puertas" (puertos) necesarias usando estos protocolos:
> * **TCP (Transmission Control Protocol):** Para administrar el servidor (RDP) y archivos (SMB). TCP exige confirmación de entrega. Si un dato se pierde, se vuelve a pedir. Es **fiable pero más lento**.
> * **UDP (User Datagram Protocol):** Para la VPN (WireGuard) y la hora (NTP). UDP dispara los paquetes a máxima velocidad sin preguntar nada. Es **rapidísimo pero menos fiable**.

> [!important] 5. RDP en vez de SSH: la puerta de entrada cambia de naturaleza
> En Ubuntu, administrábamos el servidor con SSH (línea de comandos cifrada, puerto 22/2222). En Windows Server, la herramienta equivalente para administración remota gráfica es **RDP (Remote Desktop Protocol)**, puerto **3389**. La diferencia es de fondo, no solo de puerto:
> * SSH te da una terminal de texto remota.
> * RDP te da **el escritorio completo del servidor**, como si estuvieras sentado delante de él — puedes abrir el `Administrador del servidor`, hacer clic con el ratón, ver ventanas.
> * Esto es coherente con la decisión de instalar Windows Server con Desktop Experience: si vas a usar herramientas gráficas, necesitas un protocolo que te lleve el escritorio entero, no solo una consola.

> [!important] 6. ¿Por qué seguimos usando `BOOCHAN.SPACE` y no un dominio `.LOCAL`?
> A diferencia de un laboratorio local (Hyper-V, sin salida a Internet, donde tendría sentido un dominio `.LOCAL`), aquí el servidor tiene una **IP pública real** en Azure. Mantenemos el mismo dominio real que ya usa BoochanV2: NetBIOS `BOOCHAN`, Realm **`BOOCHAN.SPACE`**. Es el mismo proyecto, la misma identidad de red, solo cambia el sistema operativo del controlador de dominio.

### 📖 Diccionario de Conceptos Clave

> [!quote] Terminología Profesional (Para no perderse)
> - **Instancia:** Una máquina virtual activa y ejecutándose en la nube.
> - **Provisionamiento:** El proceso de preparar y equipar un servidor con todo lo necesario para funcionar.
> - **NSG (Network Security Group):** Un firewall o muro lógico que decide qué tráfico de Internet entra a tu servidor.
> - **RDP (Remote Desktop Protocol):** Protocolo de Microsoft que transmite el escritorio completo de un servidor remoto a tu pantalla, cifrado, para administrarlo como si estuvieras delante.
> - **Usuario administrador local:** Cuenta creada al desplegar la VM (no es una cuenta de dominio todavía) con permisos totales sobre el servidor. Es el equivalente Windows al usuario SSH `boochan` de la versión Ubuntu.
> - **Datacenter Azure Edition:** Variante de Windows Server Datacenter optimizada específicamente para ejecutarse como máquina virtual en Azure, con integración mejorada de actualizaciones y "hotpatching".

---

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_1.4_Donde_Estamos]] | [[Fase_1]] | [[Fase_1.6_Procedimiento]] |
