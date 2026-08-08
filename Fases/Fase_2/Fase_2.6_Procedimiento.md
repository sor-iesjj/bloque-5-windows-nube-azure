## Fase 2 · Apartado 6 — 🛠️ Procedimiento práctico

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Preparación Inicial del Servidor**
> 🧭 Índice de la fase: [[Fase_2]]
>
> **📍 Cuándo se lee:** **Con la VM delante.** Aquí está el trabajo.

---

> [!important] 🔌 Antes de empezar: Conéctate al servidor
> Todos los comandos de esta fase se ejecutan **dentro de tu servidor en Azure**, no en tu PC del aula. Abre el cliente de Escritorio Remoto de Windows (`mstsc`) o la app "Conexión a Escritorio remoto" en Mac, y conéctate con la IP pública que anotaste en la Fase 1:
> ```
> Equipo: TU_IP_PUBLICA
> Usuario: azureadmin
> ```
> Cuando veas el escritorio de Windows Server, abre PowerShell **como Administrador** (clic derecho sobre el icono → `Ejecutar como administrador`) y ya estás listo para continuar.

> [!example] 🎬 Antes de empezar (todavía SIN grabar, y luego arranca)
> Ya conoces el método desde los prerrequisitos, así que va solo el recordatorio:
> 1. **Crea la entrada de apuntes** de esta fase (`b5-azure-2-preparacion-inicial-del-servidor.md`) con su estructura, vacía.
> 2. **Léete los 6 pasos** del procedimiento enteros, para no atascarte a mitad del vídeo.
> 3. Ten **OBS** listo y comprueba **pantalla y micrófono**.
>
> Cuando lo tengas: **arranca la grabación, preséntate y muestra tu identidad**. A partir de ahí, **todo queda grabado** — incluido cualquier paso previo de preparación que venga a continuación.

> [!example] Paso 1: Renombrar el Equipo
> > [!info] 📚 Diccionario de Comandos: Para entender la sintaxis exacta y ver ejemplos de los cmdlets de red y sistema que usaremos en esta fase, consulta el [[Diccionario_Comandos_Sistema]].
>
> ```powershell
> # Comprueba el nombre actual (genérico, tipo WIN-XXXXXXXXXXX)
> $env:COMPUTERNAME
>
> # Renombra el equipo y reinicia para aplicar el cambio
> Rename-Computer -NewName "WindowsServer" -Restart
> ```
>
> > [!tip] 💡 ¿Qué hace este comando?
> > - **`Rename-Computer`:** Cambia el nombre del equipo tanto a nivel NetBIOS como en la configuración de red interna de Windows. A diferencia de Linux (donde bastaba editar `/etc/hostname`), Windows **exige un reinicio completo** para que el nuevo nombre se propague a todos los servicios del sistema.
> > - **`-Restart`:** Evita el paso manual de reiniciar tú mismo; la VM se reiniciará automáticamente en cuanto el cmdlet aplique el cambio.
> >
> > La VM tardará uno o dos minutos en reiniciarse. **Perderás la sesión de Escritorio Remoto** — es normal. Espera un minuto y vuelve a conectarte con `mstsc` a la misma IP pública.

> [!example] Paso 2: Verificación de la IP Privada asignada por Azure
> Tras el reinicio, vuelve a abrir PowerShell como Administrador. Comprueba qué IP privada te ha asignado Azure:
> ```powershell
> # Muestra IP, puerta de enlace y DNS del adaptador de red
> Get-NetIPConfiguration
> ```
>
> > [!tip] 💡 ¿Qué IP anoto?
> > Busca la dirección que empieza por **`10.0.0.`** — es la IP privada de la subred de Azure (por ejemplo, `10.0.0.4`). Es la que usarán todas las fases siguientes para identificar internamente al servidor. Anótala junto a la IP pública que ya tenías de la Fase 1.
>
> Verifica también el nombre completo del equipo:
> ```powershell
> # Debe devolver: WindowsServer
> $env:COMPUTERNAME
> ```

> [!example] Paso 3: Fijar la IP Privada como Estática (desde el Portal de Azure)
> Azure asigna la IP privada por DHCP dentro de su propia infraestructura, no dentro de Windows. Para garantizar que esa IP nunca cambie (algo imprescindible para un futuro Controlador de Dominio), fíjala como estática desde el portal:
>
> 1. Entra en **portal.azure.com** → tu máquina virtual → **`Configuración de red`**.
> 2. Haz clic en la pestaña **`Interfaz de red`** (Network Interface) y luego en **`Configuración IP`**.
> 3. Selecciona la configuración IP (normalmente `ipconfig1`) y localiza el apartado **`Asignación`** de la dirección IP privada.
> 4. Cambia el valor de `Dinámica` a **`Estática`**. Azure propondrá automáticamente la misma IP que ya tenía asignada (la que anotaste en el Paso 2) — no la cambies, solo confirma.
> 5. Pulsa **`Guardar`**.
>
> > [!important] 💡 ¿Por qué no usar `New-NetIPAddress` dentro de Windows, como haríamos en Hyper-V?
> > En un laboratorio local (Hyper-V) no hay ningún DHCP gestionando la red interna, así que hay que fijar la IP a mano dentro del propio sistema operativo. En Azure, el DHCP lo gestiona la plataforma cloud: si fijaras la IP manualmente dentro de Windows con `New-NetIPAddress`, entrarías en conflicto con la configuración que Azure espera tener bajo su control, y en el peor caso perderías la conectividad de red al reiniciar. La forma correcta en la nube es siempre "estática desde la plataforma", nunca "estática desde el huésped".

> [!example] Paso 4: Preparar el DNS del Adaptador para la Fase 4
> Fija el DNS del adaptador de red al propio servidor, en preparación para cuando instales el rol DNS en la Fase 4:
> ```powershell
> # Averigua el índice del adaptador de red activo
> Get-NetAdapter
>
> # Sustituye 4 por el ifIndex real de tu adaptador (normalmente el único que aparece "Up")
> Set-DnsClientServerAddress -InterfaceIndex 4 -ServerAddresses "127.0.0.1"
> ```
>
> > [!important] 💡 ¿Por qué apuntar el DNS a `127.0.0.1` si todavía no hay servicio DNS instalado?
> > Es una preparación deliberada para la Fase 4. Cuando promociones el servidor a Controlador de Dominio con `Install-ADDSForest -InstallDNS`, el propio asistente instalará y activará el servicio **DNS Server** en esta misma máquina. Dejar el adaptador ya apuntando a `127.0.0.1` desde ahora evita un paso de reconfiguración posterior y refuerza la idea de que, en un dominio Active Directory, **el Controlador de Dominio se consulta siempre a sí mismo** para resolver nombres del dominio — el mismo principio que en BoochanV2 se lograba con `chattr +i` sobre `resolv.conf`. Hasta la Fase 4, este ajuste no tiene efecto práctico, pero deja el terreno preparado.
>
> Verifica la configuración aplicada:
> ```powershell
> Get-DnsClientServerAddress -InterfaceIndex 4
> ```

> [!example] Paso 5: Instalación de Actualizaciones de Windows Update
> Antes de instalar cualquier rol crítico como AD DS (Fase 4), parchea el sistema. El módulo `PSWindowsUpdate` no viene instalado por defecto; lo instalamos desde el repositorio oficial de PowerShell Gallery:
> ```powershell
> # Instala el proveedor NuGet si el sistema lo solicita
> Install-PackageProvider -Name NuGet -MinimumVersion 2.8.5.201 -Force
>
> # Instala el módulo de gestión de Windows Update
> Install-Module -Name PSWindowsUpdate -Force
>
> # Busca e instala todas las actualizaciones disponibles, reiniciando si hace falta
> Get-WindowsUpdate -Install -AcceptAll -AutoReboot
> ```
>
> > [!caution] ⚠️ Este proceso puede tardar bastante
> > Dependiendo de cuántas actualizaciones haya pendientes desde la imagen base de Azure, este paso puede tardar entre 10 y 30 minutos, e incluir uno o varios reinicios automáticos. Es normal. No apagues la VM manualmente durante este proceso — perderás la sesión RDP, espera unos minutos y vuelve a conectarte.
>
> Cuando termine, verifica que no quedan actualizaciones pendientes:
> ```powershell
> Get-WindowsUpdate
> ```
> Si la lista sale vacía, el sistema está al día.

> [!example] Paso 6: Verificación Final de Identidad y Red
> Confirma que todo quedó correctamente aplicado:
> ```powershell
> # Debe devolver: WindowsServer
> $env:COMPUTERNAME
>
> # Debe mostrar la IP 10.0.0.x fija en el adaptador
> Get-NetIPAddress -AddressFamily IPv4 | Where-Object { $_.IPAddress -like "10.0.0.*" }
> ```

---

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_2.5_Fundamento_Teorico]] | [[Fase_2]] | [[Fase_2.7_Resolucion_Problemas]] |
