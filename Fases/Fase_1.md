## 🏗️ Fase 1: Infraestructura Cloud (Azure IaaS — Windows Server 2025)

### Infraestructura de Servidores Cloud

> **[Módulo: SOR — Sistemas Operativos en Red]**
> **[U.T. 1, 2 y 3: Instalación de Sistemas Operativos en Red]**
> **[RA.01]** Instala sistemas operativos en red describiendo sus características e interpretando la documentación técnica.
>
> **Profesor:** Pedro Navarro Miralles
> **Correo:** p.navarromiralles2@edu.gva.es
> **Centro:** IES Jorge Juan (ALICANTE)
>
> **⏱️ Tiempo estimado:** ~1,5 - 2 horas (teoría + práctica + retos + troubleshooting)
> **Requisitos:** Azure Portal · Cliente de Escritorio Remoto (RDP) — ya viene instalado en Windows; en Mac/Linux hay que descargarlo

---

> [!important] 📹 Obligaciones de grabación (LÉEME — es igual en TODAS las fases)
> Esta práctica se **graba entera con OBS**, de principio a fin. No es un repaso al final: quiero ver **cómo lo haces tú**.
> 1. **Prepárate primero (sin grabar):** comprueba lo necesario, **léete el procedimiento entero** y **crea la entrada de apuntes de esta fase** en Obsidian: fichero `v2-1-fase-1-infraestructura-cloud-azure-iaas-windows.md` dentro de `00_Apuntes/Trimestre_N/B5_Windows_Nube/`, con la estructura de la Fase 0.1 y **vacía**. Rellenarla es cosa tuya, después.
> 2. **Arranca OBS y PRESÉNTATE:** *"Hola, me llamo [Nombre], 2.º SMR, y en este vídeo voy a explicar la Fase 1 de Boochan V2.1 — Infraestructura Cloud (Azure IaaS — Windows Server 2025)."* Y **muestra algo que demuestre que eres tú** (tu perfil de GitHub, tu Teams o tu correo `@alu.edu.gva.es`). Di qué vas a hacer.
> 3. **Graba TODO el procedimiento**, explicando cada paso en voz alta mientras lo haces.
> 4. **Timestamps SIEMPRE** en la descripción: `00:00 Presentación` + uno por cada paso.
> 5. **Al terminar:** nombra el vídeo `V2.1 · Fase 1 — Infraestructura Cloud (Azure IaaS — Windows Server 2025)`, súbelo a tu playlist de YouTube **`B5_Windows_Nube`** (No listado) y **copia su enlace**.
> 6. **~8-10 min.** Esta fase es más larga que las de prerrequisitos: ve al grano, pero no te saltes pasos. Si se te va mucho, **pártela en dos vídeos** y ponlos los dos en la entrada.
> 7. **El enlace del vídeo va DENTRO de tu entrada de apuntes**, en el apartado `Enlace al vídeo explicativo`. Ahí, no en un papel.
> 8. **La entrega va por la TAREA de Teams.** Abriré una tarea que cubrirá **esta fase y otras**; te llegará notificación con fecha límite.

---

### 🎯 Objetivos de la fase

Al terminar esta fase serás capaz de:

- [ ] Explicar qué es IaaS y por qué usamos Azure en lugar de un servidor físico en el aula.
- [ ] Crear un Grupo de Recursos y una máquina virtual **Windows Server 2025** dimensionada de forma realista para un controlador de dominio con interfaz gráfica.
- [ ] Configurar el **NSG** con las 12 reglas de todo el proyecto BoochanV2.1, entendiendo para qué sirve cada una aunque algunas no se usen todavía.
- [ ] Conocer el usuario administrador local (`azureadmin`) que sustituye al login SSH del Ubuntu original.
- [ ] Conectarte al servidor por **RDP (Escritorio Remoto)** desde tu propio ordenador.
- [ ] Conocer el nombre de dominio (`BOOCHAN.SPACE`) que usará todo el proyecto BoochanV2.1.

---

### 🎯 ¿Dónde Estamos?

> [!info] El Punto de Partida
> No vienes de una fase anterior — esta es la base. Necesitas un servidor que esté **disponible 24/7**, que no dependa de tu ordenador personal, que sea escalable, profesional y seguro. En BoochanV2 (Ubuntu + Samba) ese servidor vivía en Azure; en **BoochanV2.1** vamos a construir exactamente el mismo tipo de servidor cloud, pero con **Windows Server 2025** y su rol nativo de directorio (AD DS), sin Samba de por medio.

> [!warning] El Problema
> Instalar un servidor físico en la clase es caro, requiere mantenimiento constante y no es escalable. Además, Windows Server con interfaz gráfica y Active Directory consume bastantes más recursos que un Ubuntu Server headless: no vale con "la VM más barata que encuentres", hay que dimensionarla con cabeza. La nube resuelve la disponibilidad; nosotros resolvemos el dimensionado.

> [!success] Objetivo de esta Fase
> Crear una **máquina virtual en Azure** llamada `WindowsServer`, con **Windows Server 2025 Datacenter: Azure Edition**, dimensionada en `Standard_B4ms` (4 vCPU, 16 GB RAM). Este servidor será, en las próximas fases, tu controlador de dominio Active Directory nativo. Lo protegerás con un **NSG (firewall cloud)** que bloquea internet y abre solo los puertos imprescindibles para todo el proyecto.

> [!tip] Hoja de Ruta
> 1. Crear el Grupo de Recursos y la VM `WindowsServer` en Azure (Windows Server 2025, `Standard_B4ms`)
> 2. Configurar el NSG con las 12 reglas de todo el itinerario BoochanV2.1
> 3. Obtener la IP pública del servidor
> 4. Conectarte por **RDP** desde tu PC con el usuario `azureadmin`
> 5. Verificar acceso a internet y comprobar la RAM base con el `Administrador de tareas` (línea base para comparar en fases futuras)
> 6. Anotar el dominio de todo el proyecto: `BOOCHAN.SPACE`
>
> **Resultado Final:** Un servidor Windows Server 2025 en la nube, listo, accesible por RDP, y protegido perimetralmente.
> **Siguiente:** Fase 2 (Preparación Inicial del Servidor) — daremos identidad al servidor (nombre, red, actualizaciones) y prepararemos el terreno para el rol AD DS.

---

### 📚 Fundamento Teórico Avanzado

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

### 🛠️ Procedimiento Práctico (BoochanV2.1)

> [!example] 🎬 Antes de empezar (todavía SIN grabar, y luego arranca)
> Ya conoces el método desde los prerrequisitos, así que va solo el recordatorio:
> 1. **Crea la entrada de apuntes** de esta fase (`v2-1-fase-1-infraestructura-cloud-azure-iaas-windows.md`) con su estructura, vacía.
> 2. **Léete los 5 pasos** del procedimiento enteros, para no atascarte a mitad del vídeo.
> 3. Ten **OBS** listo y comprueba **pantalla y micrófono**.
>
> Cuando lo tengas: **arranca la grabación, preséntate y muestra tu identidad**. A partir de ahí, **todo queda grabado** — incluido cualquier paso previo de preparación que venga a continuación.

> [!example] Paso 1: Creación de la Máquina Virtual en Azure
> Entra en **portal.azure.com** con las credenciales que te haya proporcionado tu profesor (identidad digital del alumno). Una vez dentro:
>
> 1. En la **barra de búsqueda** superior, escribe `Máquinas virtuales` y haz clic en el primer resultado.
> 2. Pulsa el botón **`+ Crear`** → **`Máquina virtual de Azure`**.
> 3. Rellena el formulario con **exactamente** estos valores — son los mismos para todas las fases del proyecto, no los cambies:
>
> | Campo | Valor |
> | :--- | :--- |
> | **Grupo de recursos** | Crea uno nuevo → `rg-boochan-[tunombre]` |
> | **Nombre de la máquina virtual** | `WindowsServer` |
> | **Región** | La que indique tu profesor |
> | **Imagen** | `Windows Server 2025 Datacenter: Azure Edition - x64 Gen2` |
> | **Tamaño** | `Standard_B4ms` (4 vCPUs, 16 GB RAM) |
> | **Nombre de usuario administrador** | `azureadmin` |
> | **Contraseña** | `P@ssw0rd.SOR.2026` *(¡anótala, la necesitarás en cada práctica!)* |
> | **Puertos de entrada públicos** | Permitir puertos seleccionados → **RDP (3389)** |
>
> 4. En la pestaña **`Discos`**, deja el tipo de disco del sistema operativo por defecto (**SSD Premium**) y el tamaño que proponga la imagen (en torno a 127 GB — es el necesario para Windows Server con Desktop Experience más margen para el rol AD DS).
> 5. En la pestaña **`Redes`**, deja todos los valores por defecto. Azure crea automáticamente la red y un NSG básico con el puerto RDP (3389) ya abierto, porque lo marcaste en el paso 3.
> 6. Pulsa **`Revisar y crear`** y luego **`Crear`**. Espera 3-5 minutos hasta que el despliegue termine (Windows Server tarda más en aprovisionarse que Ubuntu Server).
>
> > [!important] 💡 ¿Qué es el "Grupo de Recursos"?
> > Piensa en él como una **carpeta de proyecto**. Agrupa todos los componentes de tu servidor (la VM, el disco duro, la red...) para que al final del curso puedas borrarlos todos juntos con un solo clic, evitando costes innecesarios.
>
> > [!important] 💡 ¿Por qué `Standard_B4ms` y no la `Standard_B2s` que usaba Ubuntu?
> > La serie B son máquinas "ráfaga" (burstable), económicas y pensadas para cargas que no consumen CPU al 100% todo el rato — perfectas para un laboratorio docente. Pero **4 GB de RAM (B2s) se quedan cortos** en cuanto Windows Server arranca con Desktop Experience y, más adelante, con el rol AD DS instalado: el sistema iría a trompicones y las prácticas serían frustrantes. `Standard_B4ms` (4 vCPU, 16 GB RAM) da margen de sobra para todo el itinerario sin disparar el coste, ya que sigue siendo de la serie económica "ráfaga".
>
> > [!warning] ⚠️ No confundas el usuario administrador con una cuenta de dominio
> > `azureadmin` es una cuenta **local** del servidor, creada por Azure al desplegar la VM — no tiene nada que ver todavía con el dominio `BOOCHAN.SPACE` que crearemos en una fase posterior con el rol AD DS. Es el equivalente exacto al usuario `boochan` que se usaba por SSH en la versión Ubuntu: sirve para entrar y administrar el sistema operativo, no para autenticarte contra Active Directory.

> [!example] Paso 2: Configuración del Cortafuegos Perimetral (NSG)
> Con la VM ya creada, añadimos el resto de las reglas de seguridad que protegerán el servidor durante todo el proyecto BoochanV2.1. Azure ya creó automáticamente la regla de RDP (3389) al marcar la casilla en el Paso 1; nos falta añadir las 11 restantes.
>
> 1. En el panel de tu VM, haz clic en **`Configuración de red`** en el menú izquierdo.
> 2. Haz clic en el nombre de tu NSG (aparece como un enlace azul).
> 3. En el menú izquierdo del NSG, haz clic en **`Reglas de seguridad de entrada`**.
>
> > [!warning] ⚠️ ¡NO borres la regla de RDP (3389) por defecto!
> > Al desplegar la máquina, Azure creó una regla automática permitiendo el puerto 3389. **No la borres todavía.** Sin ella no podrás conectarte al servidor y te quedarás fuera de tu propia máquina.
>
> 4. Pulsa **`+ Agregar`** y, para **cada fila** de la tabla siguiente, rellena el formulario así:
>    - **Origen:** `Any`
>    - **Intervalos de puertos de destino:** el número (o rango) de la columna "Puerto"
>    - **Protocolo:** según la columna "Protocolo" (si pone TCP/UDP, crea **dos reglas**, una para cada protocolo, con el mismo puerto)
>    - **Acción:** `Permitir`
>    - **Prioridad:** el número de la columna "Prioridad"
>    - **Nombre:** el texto de la columna "Nombre"
> 5. Pulsa **`Agregar`** después de cada regla:
>
> | Prioridad | Nombre | Puerto | Protocolo | Para qué sirve |
> | :--- | :--- | :--- | :--- | :--- |
> | **100** | Gestion_Web | 9090 | TCP | Reservado para un panel de gestión web opcional. En la versión Ubuntu era Cockpit; Windows Server no lo usa por defecto, pero mantenemos el puerto reservado por coherencia con el resto del proyecto. |
> | **200** | RDP | 3389 | TCP | Acceso remoto de administración de Windows Server (ya creada por Azure en el Paso 1; verifícala, no la dupliques). |
> | **300** | Kerberos_Auth | 88 | TCP/UDP | Autenticación segura de usuarios del dominio. |
> | **310** | DNS_Query | 53 | TCP/UDP | Resolución de nombres, vital para Active Directory. |
> | **320** | RPC_Endpoint | 135 | TCP | Mapeador de puntos finales RPC de Windows. |
> | **330** | LDAP_Auth | 389 | TCP/UDP | Consulta del directorio de usuarios. |
> | **340** | SMB_Files | 445 | TCP | Acceso a carpetas compartidas. |
> | **350** | LDAPS | 636 | TCP | LDAP cifrado y seguro. |
> | **360** | RPC_Dinamico | 49152-65535 | TCP | Rango de puertos dinámicos requeridos por AD DS. |
> | **370** | Kerberos_Pass | 464 | TCP/UDP | Cambio de contraseñas de usuarios del dominio. |
> | **380** | NTP_Time | 123 | UDP | Sincronización horaria, crucial para Kerberos. |
> | **390** | WireGuard | 51820 | UDP | Túnel VPN de red privada para conectar el aula (Fase 3). |
>
> > [!info] 💡 ¿Por qué abrimos ya los 12 puertos si algunos no se usan hasta fases posteriores?
> > A diferencia de BoochanV2 (donde el NSG se iba ampliando fase a fase), en BoochanV2.1 preparamos el NSG completo desde el principio para que puedas centrarte en la configuración de Windows Server en las siguientes fases sin volver constantemente al portal de Azure. Aun así, **cada puerto abierto debe poder justificarse**: si en algún momento no supieras para qué sirve uno de ellos, repasa esta tabla.
>
> > [!warning] ⚠️ El puerto 3389 es provisional
> > En la **Fase 3**, cuando la VPN esté funcionando, cerraremos el puerto 3389 al mundo exterior y solo permitiremos RDP desde dentro de la red privada (VPN). Por ahora lo dejamos abierto a `Any` porque sin VPN no podríamos entrar al servidor.

> [!example] Paso 3: Primera Conexión al Servidor (RDP)
> Vuelve al panel principal de tu VM y localiza el campo **`Dirección IP pública`**. **Anótala**: la necesitarás en todas las fases del proyecto.
>
> **En Windows (cliente ya instalado):**
> 1. Pulsa `Windows + R`, escribe `mstsc` y pulsa Enter (o busca "Conexión a Escritorio remoto" en el menú Inicio).
> 2. En el campo **Equipo**, escribe la IP pública que anotaste.
> 3. Pulsa **`Conectar`**.
> 4. Cuando te pida credenciales, introduce:
>    - **Usuario:** `azureadmin` (o `WindowsServer\azureadmin` si te lo pide con el nombre del equipo delante)
>    - **Contraseña:** `P@ssw0rd.SOR.2026`
> 5. Aparecerá una advertencia de certificado ("No se puede verificar la identidad del equipo remoto"). Es normal la primera vez porque el certificado es autofirmado — pulsa **`Sí`** para continuar.
>
> **En Mac / Linux:**
> Instala el cliente **Microsoft Remote Desktop** (Mac App Store) o `Remmina`/`rdesktop` (Linux), y repite los mismos pasos: IP pública, usuario `azureadmin`, contraseña.
>
> Si al final ves el escritorio de Windows Server con el `Administrador del servidor` abierto automáticamente, **ya estás dentro de tu servidor**. A partir de aquí, todo lo que hagas con el ratón y el teclado se ejecuta en Azure, a cientos de kilómetros.
>
> > [!warning] ⚠️ ¿Por qué aquí NO vale el `P@ssw0rd` del resto del módulo?
> > En todas las máquinas locales de este módulo la contraseña es `P@ssw0rd`. Aquí no, y el motivo es interesante: **Azure la rechaza, por dos razones a la vez.**
> > 1. **Longitud.** El portal de Azure exige **entre 12 y 72 caracteres** para la contraseña del administrador de una VM. `P@ssw0rd` tiene 8. Ni lo intentes: el formulario no te deja pasar de pantalla.
> > 2. **Lista negra.** Aunque tuviera 12, Azure mantiene una **lista de contraseñas prohibidas** con las más usadas del mundo — y `P@ssw0rd` es de las primeras de esa lista. Microsoft directamente no te deja poner en internet una máquina con una contraseña que un atacante prueba en el primer segundo.
> >
> > Por eso aquí usamos **`P@ssw0rd.SOR.2026`**: 17 caracteres, con mayúsculas, minúsculas, números y símbolos.
> >
> > **La lección, que es la de verdad:** la misma contraseña que es aceptable en tu portátil aislado **es inaceptable en cuanto la máquina tiene IP pública**, y el proveedor te lo impone aunque tú no quieras. Cuando el servidor deja de estar escondido, las reglas cambian solas.
> >
> > ⚠️ **No la confundas con la del dominio.** Esta es la contraseña del **administrador de la VM** (`azureadmin`). La del **Administrator del dominio** que crearás en la Fase 4 sigue siendo `P@ssw0rd`, porque el dominio vive dentro de la red privada y ahí manda la política de Active Directory, no la de Azure. **Son dos contraseñas distintas para dos cosas distintas. Anota las dos.**
>
> > [!tip] 💡 ¿Qué es RDP?
> > RDP (Remote Desktop Protocol) es como un **"mando a distancia" cifrado que te lleva la pantalla entera** del servidor a tu monitor. Desde tu ratón y teclado del aula estás controlando un ordenador de Azure, a cientos de kilómetros, como si tuvieras el monitor del servidor delante. Todo el tráfico viaja cifrado.
>
> > [!important] 💡 El puerto 3389
> > Ahora nos conectamos por el puerto 3389, que es el que Azure abrió por defecto al marcar la casilla RDP al crear la VM. En la **Fase 3**, una vez configurada la VPN, cerraremos este puerto al mundo exterior y solo se podrá acceder desde dentro de la red privada.

> [!tip] 💡 ¿Cómo verifico si los puertos están "vivos" dentro del servidor?
> Una vez que tengas servicios corriendo en las siguientes fases, puedes usar este comando de PowerShell (como administrador) para ver qué puertos están escuchando en tu servidor:
> ```powershell
> # Muestra todas las conexiones TCP en estado de escucha con su proceso asociado
> Get-NetTCPConnection -State Listen | Sort-Object LocalPort
> ```

> [!example] Paso 4: Verificación de Internet y Medida de RAM Base
> Ya dentro del servidor por RDP:
>
> 1. Abre PowerShell y comprueba la salida a Internet:
>    ```powershell
>    ping google.com
>    ```
>    Deberías recibir respuestas sin pérdida de paquetes.
> 2. Abre el **`Administrador de tareas`** (`Ctrl+Shift+Esc`) → pestaña **`Rendimiento`** → **`Memoria`**. Anota cuánta RAM está en uso ahora mismo, con el sistema recién instalado y sin roles adicionales. Guarda ese dato — lo compararás en la Fase 4, cuando el rol AD DS esté instalado y funcionando, para ver cuánta RAM consume el dominio.
> 3. Anota también el dominio de todo el proyecto, lo necesitarás desde la Fase 4 en adelante:
>
> | Concepto | Valor en BoochanV2.1 |
> | :--- | :--- |
> | **Nombre NetBIOS** | `BOOCHAN` |
> | **Realm (dominio completo)** | `BOOCHAN.SPACE` |
> | **Nombre del servidor** | `WindowsServer` |
> | **Usuario administrador local** | `azureadmin` |
> | **Grupo de Recursos** | `rg-boochan-[tunombre]` |

---

> [!example] 🔌 Paso 5 — EJERCICIO DE VERIFICACIÓN: comprueba tu red desde fuera
> Hasta aquí has configurado la red **y has confiado en que el panel dice la verdad**. Ahora vas a comprobarlo con fuentes **externas e independientes**, que es como se hace de verdad.
>
> > [!info] ¿Qué es una API y por qué la usa un administrador?
> > Una **API** es una web hecha para que la consulte un programa en vez de una persona: en vez de devolverte una página con colores, te devuelve **datos limpios** en formato JSON.
> >
> > ¿Y para qué la quiere un administrador de sistemas? Para **comprobar desde fuera lo que desde dentro no puede ver**. Tu servidor te dirá siempre lo que él cree de sí mismo; un servicio externo te dice **lo que se ve realmente**. Y esa diferencia, cuando aparece, es justo donde está el problema que llevas dos horas buscando.
> >
> > Se consultan con **`curl`**, que ya has usado y que viene instalado en todas partes. Sin programar y sin instalar nada.
>
> **a) Verifica tu cálculo de subred.** Tu red es **`10.0.0.0/24`** (la red virtual (VNet) de Azure).
> Primero, **a mano y sin ayuda**, escribe en tu entrada de apuntes: máscara decimal, dirección de red, broadcast, número de hosts asignables, primero y último.
> Ahora compruébalo:
> ```bash
> curl "https://networkcalc.com/api/ip/10.0.0.0/24"
> ```
> Si no coincide, **no borres tu respuesta**: déjala y explica en el vídeo dónde te equivocaste. Eso enseña más que acertar.
>
> **b) Tu servidor SÍ tiene IP pública. Averigua de quién es.** Desde dentro del servidor:
> ```bash
> curl "https://api.ipify.org?format=json"
> ```
> Compárala con la que te muestra el panel de Azure: **tienen que coincidir**.
>
> Ahora pregunta **quién es el dueño de esa IP**:
> ```bash
> curl "http://ip-api.com/json/TU_IP_PUBLICA?fields=query,country,isp,org,as"
> ```
>
> > [!success] 🤔 Mira bien la respuesta
> > No sale tu nombre: sale **Microsoft**, con su número de **AS** y el país del centro de datos.
> > **Eso es "estar en la nube"**, dicho con datos: tu servidor vive dentro de la infraestructura de Microsoft, y para el resto de Internet es una máquina más de las suyas.
> > **Explica en el vídeo:** ¿en qué país está físicamente tu servidor? ¿Coincide con el que elegiste al crearlo?
>
> > [!question] Lo que va a tu entrada de apuntes
> > 1. ¿Coincidió tu cálculo de subred con el de la API? Si no, ¿en qué fallaste?
> > 2. ¿Cuál es la IP privada de tu servidor y cuál la pública? ¿Por qué no son la misma?
> > 3. ¿Por qué una comprobación hecha **desde el propio servidor** vale menos que una hecha desde fuera?
>
> > [!note] 📌 Para saber más
> > La teoría completa de esto está en la práctica **B1.9b — Verificar tu red con APIs públicas** del Bloque 1. Aquí lo aplicas a tu servidor de verdad.
> > Y una consecuencia que conviene que asumas ya: **tu servidor es alcanzable desde cualquier punto del planeta.** En cuanto lo enciendes empieza a recibir intentos de conexión de desconocidos. Por eso las siguientes fases dedican tanto tiempo al cortafuegos y a la VPN.

---

### 🚩 Resolución de Problemas y Evaluación

> [!bug] Tabla de Troubleshooting (¿Algo no funciona?)
> | Problema | Causa Probable | Solución Sugerida |
> | :--- | :--- | :--- |
> | No puedo conectar por RDP ("No se puede establecer la conexión"). | La VM no ha terminado de arrancar/aprovisionarse. | Espera 3-5 minutos (Windows Server tarda más que Ubuntu) y vuelve a intentarlo. |
> | RDP se conecta pero rechaza las credenciales. | Usuario o contraseña incorrectos, o Bloq Mayús activado. | Comprueba que escribes exactamente `azureadmin` y la contraseña tal cual, respetando mayúsculas/símbolos. |
> | Aparece "Este equipo no puede conectarse al equipo remoto" o error de licencia/protocolo. | El puerto 3389 no está abierto en el NSG, o la regla se borró por error. | Revisa **Configuración de red → NSG → Reglas de seguridad de entrada** y confirma que la regla RDP (3389) existe y está en `Permitir`. |
> | El servidor **no responde al ping**. | El protocolo ICMP está bloqueado por defecto en Azure para tráfico entrante. | Es normal por seguridad. No abras el ping de entrada; usa RDP o prueba de puertos TCP para verificar conectividad. |
> | La pantalla de RDP se ve muy lenta o pixelada. | Configuración de calidad de la conexión demasiado alta para el ancho de banda del aula. | En el cliente RDP, antes de conectar, baja la calidad de color/experiencia en las opciones avanzadas de la conexión. |

> [!help] Preguntas Críticas (Autoevaluación del alumno)
> 1. ¿Por qué Microsoft Azure es responsable del hardware pero tú eres el responsable del Sistema Operativo?
> 2. ¿Qué ocurre exactamente si dejas el puerto de administración (3389) abierto a **"Cualquiera" (0.0.0.0/0)** de forma indefinida?
> 3. ¿Por qué RDP transmite el escritorio completo mientras que SSH (usado en la versión Ubuntu) solo transmite texto? ¿Qué ventajas e inconvenientes tiene cada enfoque?
> 4. ¿Por qué el usuario `azureadmin` no es todavía una cuenta del dominio `BOOCHAN.SPACE`?
> 5. 🔬 **Reto práctico:** Entra en el NSG de Azure y **deshabilita** temporalmente la regla del puerto 9090 (sin borrarla, solo desactívala). Intenta después abrir `https://<TU_IP_PUBLICA>:9090` desde el navegador. ¿Qué ocurre? Vuelve a habilitarla. ¿Qué has comprobado con este experimento sobre el papel del NSG?
> 6. 🔬 **Reto práctico:** Anota en el `Administrador de tareas` cuánta RAM usa el sistema base recién instalado. Guarda ese dato — lo compararás en la Fase 4, cuando el rol AD DS esté corriendo, para ver cuánta RAM consume el dominio.

---

> [!caution] 🛑 Auditoría de Seguridad — Tarea pendiente tras la Fase 3
> Una vez que la VPN esté funcionando, cerrarás el acceso RDP directo desde Internet. **No lo hagas ahora**: sin VPN activa te quedarías sin acceso al servidor.
>
> **Acción — Cerrar el puerto 3389 al mundo exterior en el NSG de Azure:**
> Vuelve a **Configuración de red** → NSG → **Reglas de seguridad de entrada**. Localiza la regla `RDP` (puerto 3389, origen `Any`) y cámbiala para que el **Origen** sea únicamente el rango de la red privada de la VPN (o elimínala si vas a acceder siempre a través del túnel WireGuard).
>
> Esto es aplicar seguridad "Zero Trust": nadie en Internet puede llegar al servidor por RDP; solo quien esté dentro de la VPN.

---

### ✅ Checklist Final de la Fase 1

- [ ] Grupo de Recursos `rg-boochan-[tunombre]` creado en Azure.
- [ ] VM `WindowsServer` creada con imagen `Windows Server 2025 Datacenter: Azure Edition - x64 Gen2`.
- [ ] Tamaño de la VM: `Standard_B4ms` (4 vCPU, 16 GB RAM).
- [ ] Usuario administrador local `azureadmin` y contraseña anotados en lugar seguro.
- [ ] NSG configurado con las 12 reglas de todo el proyecto (Gestion_Web, RDP, Kerberos_Auth, DNS_Query, RPC_Endpoint, LDAP_Auth, SMB_Files, LDAPS, RPC_Dinamico, Kerberos_Pass, NTP_Time, WireGuard).
- [ ] IP pública del servidor anotada.
- [ ] Conexión por RDP realizada con éxito desde tu propio ordenador.
- [ ] `ping google.com` funciona desde dentro del servidor.
- [ ] RAM base anotada desde el `Administrador de tareas` (para comparar en la Fase 4).
- [ ] Dominio del proyecto anotado: NetBIOS `BOOCHAN`, Realm `BOOCHAN.SPACE`.

> **Siguiente paso:** Fase 2 — Preparación Inicial del Servidor, donde revisaremos la configuración inicial de Windows Server (nombre de host, red, actualizaciones) y prepararemos el terreno para el rol AD DS.

---

### ✅ Entregables y cierre

> [!abstract] Qué tienes que tener hecho al acabar esta fase
> | Entregable | Dónde vive | Qué debe contener |
> | :--- | :--- | :--- |
> | **Entrada de apuntes** | `00_Apuntes/Trimestre_N/B5_Windows_Nube/v2-1-fase-1-infraestructura-cloud-azure-iaas-windows.md` | Estructura completa + **respuestas a las Preguntas Críticas y al 🔬 Reto** + **enlace del vídeo** |
> | **Vídeo** | Playlist `B5_Windows_Nube` (No listado) | Nombrado `V2.1 · Fase 1 — Infraestructura Cloud (Azure IaaS — Windows Server 2025)`, con presentación, identidad y timestamps |
> | **Repositorio** | Tu repo de apuntes en GitHub | La entrada, subida con `git add` → `commit` → `push` |
>
> > [!danger] ⚠️ Las respuestas van en la ENTRADA, no en un documento aparte
> > Las **Preguntas Críticas** y el **🔬 Reto** de más arriba no son decorativos: son la parte de la fase que demuestra que has entendido lo que has hecho, y no solo que has sabido copiar comandos. Se contestan **con tus palabras**, en el apartado `Respuesta a las preguntas` de tu entrada.
> > Una fase con el procedimiento perfecto y las preguntas en blanco está **incompleta**.
>
> > [!info] 🏷️ Por qué el nombre lleva `V2.1` delante
> > Porque el proyecto Boochan existe en **varias versiones** (VirtualBox, Hyper-V, Azure, AWS…) y algunas comparten bloque y playlist. Sin la etiqueta, la Fase 4 de Azure y la de AWS se llamarían **exactamente igual** y no habría forma de distinguirlas. Con ella, tu carpeta y tu playlist dicen siempre **qué versión hiciste**.
>
> > [!success] 🎯 Criterio de éxito
> > Abro tu repositorio, encuentro la entrada de esta fase, y dentro está: qué has hecho, qué has entendido, qué dudas te han quedado y el enlace al vídeo donde se te ve haciéndolo. Si falta el enlace o faltan las respuestas, la fase **no cuenta como entregada**.
>
> > [!tip] 💡 ¿Y si la fase te ha llevado tres clases?
> > **Una fase, una entrada.** No creas un fichero por día: abres el mismo y sigues escribiendo. Haz `commit` y `push` **al terminar cada sesión**, para no perder nunca más de un día de trabajo.
