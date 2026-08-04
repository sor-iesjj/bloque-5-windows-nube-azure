## Fase 1 · Apartado 6 — 🛠️ Procedimiento práctico

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Infraestructura Cloud (Azure IaaS — Windows Server 2025)**
> 🧭 Índice de la fase: [[Fase_1]]
>
> **📍 Cuándo se lee:** **Con la VM delante.** Aquí está el trabajo.

---

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

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_1.5_Fundamento_Teorico]] | [[Fase_1]] | [[Fase_1.7_Resolucion_Problemas]] |
