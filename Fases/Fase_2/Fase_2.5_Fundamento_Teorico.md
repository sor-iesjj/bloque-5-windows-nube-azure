## Fase 2 · Apartado 5 — 📚 Fundamento teórico

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Preparación Inicial del Servidor**
> 🧭 Índice de la fase: [[Fase_2]]
>
> **📍 Cuándo se lee:** **Antes de teclear.** Los conceptos que necesitas.

---

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

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_2.4_Donde_Estamos]] | [[Fase_2]] | [[Fase_2.6_Procedimiento]] |
