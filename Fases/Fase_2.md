## 🧹 Fase 2: Preparación Inicial del Servidor

### Infraestructura de Servidores Cloud (Azure + Windows Server 2025)

> **[Módulo: SOR — Sistemas Operativos en Red]**
> **[U.T. 5: Administración en Windows Server - Instalación y Configuración]**
> **[RA.02]** Gestiona usuarios y grupos, interpretando especificaciones y aplicando herramientas del sistema.
>
> **Profesor:** Pedro Navarro Miralles  
> **Correo:** p.navarromiralles2@edu.gva.es  
> **Centro:** IES Jorge Juan (ALICANTE)
>
> **⏱️ Tiempo estimado:** ~1,25 horas (teoría + práctica + retos + troubleshooting)  
> **Requisitos:** 16 GB RAM (VM `Standard_B4ms`) | Conectividad internet | Escritorio Remoto (RDP)

---

### 🎯 ¿Dónde Estamos?

> [!info] Vienes de Fase 1
> Creaste una máquina virtual **Windows Server 2025 Datacenter** en Azure (`Standard_B4ms`). Está encendida, accesible por Escritorio Remoto, protegida por un NSG que solo abre el puerto 3389. Pero viene "de fábrica": nombre genérico tipo `WIN-XXXXXXXXXXX`, sin actualizaciones aplicadas, y sin que nadie le haya confirmado cuál es su identidad definitiva dentro del proyecto.

> [!warning] El Problema
> A diferencia de Ubuntu Server (donde en BoochanV2 había que **purgar** Samba, CUPS y demonios heredados que ocupaban puertos), Windows Server no trae ningún servicio de directorio ni de archivos preinstalado que estorbe. El "problema" aquí es de otra naturaleza: el servidor tiene un nombre aleatorio que nadie reconocería en un log ni en una consulta DNS, y puede tener vulnerabilidades sin parchear desde el día en que se generó la imagen. Además, en Azure, la IP privada la asigna el propio DHCP de la plataforma — si no la fijamos, Azure sigue dándonos la misma en cada reinicio, pero conviene comprobarlo y dejarlo documentado antes de construir nada encima.

> [!success] Objetivo de esta Fase
> **Identidad:** Renombrar el servidor a `WindowsServer`, el nombre que usará todo el proyecto BoochanV2.1. **Red:** Verificar y anotar la IP privada que Azure asignó al servidor (rango `10.0.0.x`), y fijarla como estática desde el propio portal de Azure para que nunca cambie entre reinicios. **Higiene:** Aplicar Windows Update para partir de un sistema parcheado antes de instalar ningún rol crítico como AD DS (Fase 4).

> [!tip] Hoja de Ruta
> 1. Conectarse al servidor por Escritorio Remoto (RDP) con la IP pública de Azure
> 2. Renombrar el equipo con `Rename-Computer` a `WindowsServer`
> 3. Verificar la IP privada asignada por Azure con `Get-NetIPConfiguration` (rango `10.0.0.x`)
> 4. Fijar esa IP como **estática** desde el portal de Azure (pestaña `Configuración de red` → IP privada)
> 5. Apuntar el DNS del propio adaptador a `127.0.0.1` (se explica por qué, aunque el DNS real no existirá hasta la Fase 4)
> 6. Instalar actualizaciones de Windows Update
> 7. Reiniciar y verificar identidad y red
>
> **Resultado Final:** Servidor con nombre `WindowsServer`, IP privada fija en el rango `10.0.0.x` de Azure, y parches de seguridad al día.
> **Siguiente:** Fase 3 (Conectividad VPN) — instalarás WireGuard para Windows para blindar el acceso remoto al servidor.

---

### 📚 Fundamento Teórico Avanzado

> [!abstract] 1. "De fábrica" en Windows Server vs. Linux: no hay que purgar, hay que nombrar
> En BoochanV2 (Ubuntu), la instalación por defecto traía Samba básico, CUPS y otros demonios que había que **purgar agresivamente** porque ocupaban puertos que el futuro Controlador de Dominio necesitaría (el temido conflicto del puerto 445). **Windows Server no tiene ese problema:** la instalación no trae ningún rol activado por defecto — ni siquiera AD DS, DNS o el propio Escritorio Remoto están instalados hasta que tú los añades explícitamente con `Install-WindowsFeature`. El trabajo de esta fase no es "demoler", es **dar identidad**: nombre de equipo e IP privada consolidada, los dos datos que todo lo demás (Fase 3, Fase 4) dará por hecho que ya existen.

> [!warning] 2. Por qué el nombre del equipo importa tanto como el FQDN en Linux
> En BoochanV2 configurabas `/etc/hosts` para que el servidor supiera su FQDN completo (`UbuntuServer.BOOCHAN.SPACE`). En Windows Server el equivalente conceptual es el **nombre de equipo** (Computer Name). Cuando en la Fase 4 promociones este servidor a Controlador de Dominio con `Install-ADDSForest`, Windows construirá automáticamente el FQDN del propio servidor concatenando el nombre de equipo con el Realm del dominio: `WindowsServer.BOOCHAN.SPACE`. Si en ese momento el nombre de equipo sigue siendo el genérico `WIN-XXXXXXXXXXX`, el dominio se creará igualmente, pero el servidor tendrá un nombre absurdo para siempre — cambiarlo después de promocionar a Controlador de Dominio es mucho más complicado (requiere herramientas adicionales y reinicios en cadena). Por eso se hace **ahora**, antes de instalar ningún rol.

> [!tip] 3. IP dinámica de Azure vs. IP estática "dentro" de la VM
> En BoochanV2, la IP privada Ubuntu la asigna Azure por DHCP y el alumno simplemente la consulta con `hostname -I` para usarla. En Windows Server pasa exactamente lo mismo a nivel de sistema operativo: **no vamos a fijar la IP a mano dentro de Windows** con `New-NetIPAddress` (eso es lo que hacíamos en el laboratorio local de Hyper-V, donde no existía ningún DHCP). En Azure, fijar la IP se hace desde el lado de la plataforma: el portal permite marcar la asignación de la IP privada de la tarjeta de red (NIC) como **"Estática"** en lugar de "Dinámica", de modo que Azure siempre entregue la misma dirección, sin tocar la configuración de red dentro del propio Windows. Es un cambio de paradigma importante: en la nube, la identidad de red se fija en la infraestructura, no en el sistema operativo huésped.

> [!info] 4. ¿Dónde queda el "`/etc/hosts`" de Windows?
> Windows Server también tiene un archivo equivalente, `C:\Windows\System32\drivers\etc\hosts`, pero **no lo vamos a tocar en esta fase, y probablemente nunca en este proyecto**. La razón es que, en cuanto promociones el servidor a Controlador de Dominio en la Fase 4, el propio servicio **DNS Server** (que se instala junto con AD DS) se convertirá en la fuente de verdad para resolver `WindowsServer.BOOCHAN.SPACE` y cualquier otro nombre del dominio — de forma dinámica y automática, sin mantenimiento manual de ficheros.

> [!important] 5. Windows Update antes de instalar roles críticos
> Instalar AD DS (Fase 4) sobre un sistema sin parchear es una mala práctica que en producción podría dejar el Controlador de Dominio expuesto a vulnerabilidades conocidas desde el primer día. El hábito profesional correcto es siempre el mismo: **actualizar primero, instalar roles después.**

### 📖 Diccionario de Conceptos Clave

> [!quote] Terminología Profesional
> - **Rename-Computer:** Cmdlet de PowerShell que cambia el nombre NetBIOS/DNS del equipo. Requiere reinicio para aplicarse.
> - **Get-NetIPConfiguration:** Cmdlet que muestra de un vistazo la IP, la puerta de enlace y el DNS de cada adaptador de red.
> - **Set-DnsClientServerAddress:** Cmdlet que fija qué servidor DNS debe consultar un adaptador de red.
> - **IP privada estática (Azure):** Ajuste de la tarjeta de red virtual (NIC) en el portal de Azure que evita que la IP interna cambie al reiniciar la VM. Equivale conceptualmente al fichero `/etc/netplan/` de Ubuntu, pero se configura fuera del sistema operativo.

---

### 🛠️ Procedimiento Práctico (BoochanV2.1)

> [!important] 🔌 Antes de empezar: Conéctate al servidor
> Todos los comandos de esta fase se ejecutan **dentro de tu servidor en Azure**, no en tu PC del aula. Abre el cliente de Escritorio Remoto de Windows (`mstsc`) o la app "Conexión a Escritorio remoto" en Mac, y conéctate con la IP pública que anotaste en la Fase 1:
> ```
> Equipo: TU_IP_PUBLICA
> Usuario: azureadmin
> ```
> Cuando veas el escritorio de Windows Server, abre PowerShell **como Administrador** (clic derecho sobre el icono → `Ejecutar como administrador`) y ya estás listo para continuar.

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

### 🚩 Resolución de Problemas y Evaluación

> [!bug] Troubleshooting (¿Algo no va bien?)
> | Problema | Causa Probable | Solución Sugerida |
> | :--- | :--- | :--- |
> | `Rename-Computer` no pide reinicio y el nombre no cambia. | Se ejecutó sin permisos de administrador. | Cierra PowerShell y vuelve a abrirlo con `Ejecutar como administrador`. Repite el comando. |
> | Tras el `-Restart` no puedo reconectar por RDP. | La VM todavía está reiniciando. | Espera 1-2 minutos y vuelve a intentar la conexión con `mstsc`. |
> | El portal de Azure no deja marcar la IP como "Estática". | La VM está apagada, o el cambio se intentó desde el recurso equivocado (NIC en vez de VM). | Asegúrate de que la VM está encendida y de entrar en `Configuración de red → Interfaz de red → Configuración IP`. |
> | `Install-Module -Name PSWindowsUpdate` falla o se queda colgado. | Problema temporal de repositorio de PowerShell Gallery no confiado. | Responde `S` (Sí) si pregunta por confiar en el repositorio, o añade `-Force` al comando. |

> [!help] Preguntas Críticas (Autoevaluación)
> 1. ¿Por qué en Windows Server no hace falta "purgar" nada, a diferencia de Ubuntu Server en BoochanV2?
> 2. ¿Qué ocurre si promocionas el servidor a Controlador de Dominio (Fase 4) sin haberlo renombrado antes? ¿Por qué es tan importante hacerlo en este orden?
> 3. ¿Por qué fijamos la IP privada como "Estática" desde el portal de Azure y no con `New-NetIPAddress` dentro de Windows, como haríamos en un laboratorio local?
> 4. 🔬 **Reto práctico:** Ejecuta `Get-NetIPConfiguration` y compara la salida con `Get-NetIPAddress`. ¿Qué información adicional aporta el primero (puerta de enlace, DNS) que el segundo no muestra directamente?
> 5. 🔬 **Reto práctico:** Ejecuta `Get-Process | Sort-Object WS -Descending | Select-Object -First 5` para ver los 5 procesos que más RAM consumen ahora mismo. Anota el total de RAM libre con `Get-Counter '\Memory\Available MBytes'`. Guarda ese dato — lo compararás con la Fase 4, cuando AD DS esté instalado, para ver cuánta RAM consume el dominio.

---

> [!caution] 🛑 Auditoría y Evaluación (RA.02)
> El alumno debe demostrar que el servidor tiene el nombre e IP correctos antes de avanzar. **Riesgo Crítico:** Si el nombre de equipo no es `WindowsServer` antes de la Fase 4, el FQDN del propio Controlador de Dominio quedará con un nombre erróneo de forma permanente.

> [!success] 🏁 Punto de Control (Antes de seguir)
> - [ ] ¿`$env:COMPUTERNAME` devuelve exactamente `WindowsServer`?
> - [ ] ¿`Get-NetIPConfiguration` muestra una IP en el rango `10.0.0.x` marcada como estática en el portal de Azure?
> - [ ] ¿El DNS del adaptador apunta a `127.0.0.1`?
> - [ ] ¿`Get-WindowsUpdate` no muestra actualizaciones pendientes?
