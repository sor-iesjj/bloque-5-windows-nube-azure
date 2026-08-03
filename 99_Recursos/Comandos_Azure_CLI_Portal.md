# ☁️ Comandos de Azure CLI / Portal

Esta guía no existe en BoochanV1.1 (Hyper-V local), porque allí no hay ningún proveedor cloud de por medio. En **BoochanV2.1 tú eres quien crea, configura y administra recursos reales de Azure** — una VM, un NSG, una IP pública, un Grupo de Recursos — así que necesitas dominar tanto el Azure Portal (interfaz web) como, opcionalmente, la Azure CLI (línea de comandos). Úsala como referencia rápida a lo largo de la Fase 1 (creación de la VM y el NSG), la Fase 2 (fijar la IP privada) y la Auditoría Final (cierre de puertos).

> [!info] Portal vs. CLI: dos caminos, mismo resultado
> Todas las acciones de esta guía se explican primero por el **Azure Portal** (`portal.azure.com`), la vía que sigue el manual paso a paso en las Fases 1-8, porque es más visual para quien administra Azure por primera vez. Cuando existe, se indica también el comando equivalente de **Azure CLI** (`az`), útil si tu profesor te pide automatizar tareas repetitivas o si prefieres trabajar desde una terminal. Ambos caminos actúan sobre los mismos recursos — no son alternativas incompatibles.

---

## 🏗️ 1. Grupo de Recursos y Máquina Virtual

### Crear un Grupo de Recursos
> **Ruta del Portal:** Barra de búsqueda → `Grupos de recursos` → `+ Crear`

> [!example] Valor usado en BoochanV2.1 (Fase 1)
> | Campo | Valor |
> | :--- | :--- |
> | Nombre | `rg-boochan-[tunombre]` |
> | Región | La que indique tu profesor |
>
> **Azure CLI equivalente:**
> ```bash
> az group create --name rg-boochan-tunombre --location westeurope
> ```
>
> > [!important] 💡 ¿Por qué agrupar todo en un Grupo de Recursos?
> > Piensa en él como una **carpeta de proyecto**: agrupa la VM, el disco, la red y el NSG para que al final del curso el profesor (o tú, si tienes permiso) pueda borrar todos los recursos juntos con una sola operación, evitando coste residual olvidado.

### Crear la máquina virtual `WindowsServer`
> **Ruta del Portal:** Barra de búsqueda → `Máquinas virtuales` → `+ Crear` → `Máquina virtual de Azure`

> [!example] Valores usados en BoochanV2.1 (Fase 1)
> | Campo | Valor |
> | :--- | :--- |
> | Nombre de la VM | `WindowsServer` |
> | Imagen | `Windows Server 2025 Datacenter: Azure Edition - x64 Gen2` |
> | Tamaño | `Standard_B4ms` (4 vCPU, 16 GB RAM) |
> | Usuario administrador | `azureadmin` |
> | Puertos de entrada públicos | RDP (3389) |
>
> **Azure CLI equivalente (referencia, no obligatorio en el proyecto):**
> ```bash
> az vm create \
>   --resource-group rg-boochan-tunombre \
>   --name WindowsServer \
>   --image Win2025Datacenter \
>   --size Standard_B4ms \
>   --admin-username azureadmin \
>   --admin-password "P@ssw0rd.SOR.2026" \
>   --public-ip-sku Standard
> ```
>
> > [!important] 💡 ¿Por qué `Standard_B4ms` y no una VM más barata?
> > La serie B son máquinas "ráfaga" (burstable), pensadas para cargas que no consumen CPU al 100% todo el rato — ideales para un laboratorio docente. Pero Windows Server con Desktop Experience y, más adelante, el rol AD DS, necesitan bastante más RAM que un Ubuntu Server headless (que se conformaba con `Standard_B2s`, 4 GB). `Standard_B4ms` (16 GB) da margen de sobra sin disparar el coste.

### Iniciar, detener y desasignar la VM (control de coste)
> **Ruta del Portal:** Panel de tu VM → botones `Iniciar` / `Detener` en la barra superior

> [!caution] ⚠️ "Detener" no siempre significa "dejar de pagar"
> Azure factura por horas de cómputo mientras la VM está **asignada** a un servidor físico, aunque esté apagada dentro del sistema operativo. Para dejar de pagar por el cómputo (aunque el disco sigue teniendo un coste menor de almacenamiento), debes **desasignar** la VM, no solo apagarla desde dentro de Windows.
>
> - **`Stop-Computer` (dentro de Windows) o apagar desde el escritorio:** apaga el sistema operativo, pero Azure sigue reservando el hardware — **sigue habiendo coste de cómputo**.
> - **Botón `Detener` del Portal (o `az vm deallocate`):** libera el hardware asignado — **coste de cómputo detenido**, solo queda el coste (menor) del disco y la IP pública si la reservaste como estática.
>
> **Azure CLI equivalente:**
> ```bash
> az vm deallocate --resource-group rg-boochan-tunombre --name WindowsServer
> az vm start --resource-group rg-boochan-tunombre --name WindowsServer
> ```
>
> > [!tip] 💡 Hábito recomendado al terminar cada sesión
> > Si tu profesor lo indica, usa siempre el botón `Detener` del Portal (no el apagado de Windows) al final de la sesión de clase. Recuerda que al volver a `Iniciar` la VM, si la IP pública no está reservada como estática, Azure puede asignarte una IP pública **distinta** — revisa siempre el campo `Dirección IP pública` del panel de la VM antes de conectar por RDP.

---

## 🌐 2. Red: NSG, IP privada e IP pública

### Consultar y editar las reglas del NSG (Network Security Group)
> **Ruta del Portal:** Panel de tu VM → `Configuración de red` → clic en el nombre del NSG → `Reglas de seguridad de entrada`

> [!example] Configuración estándar del proyecto (Fase 1)
> Las 12 reglas del NSG completo de BoochanV2.1 se crean de una sola vez en la Fase 1 — a diferencia de BoochanV2, donde el NSG se ampliaba fase a fase. Consulta la tabla completa en [[../Fases/Fase_1|Fase 1]] o en el [[../Manual_BoochanV2.1|Manual]].
>
> **Azure CLI equivalente (una regla, de ejemplo):**
> ```bash
> az network nsg rule create \
>   --resource-group rg-boochan-tunombre \
>   --nsg-name WindowsServer-nsg \
>   --name WireGuard \
>   --priority 390 \
>   --destination-port-ranges 51820 \
>   --protocol Udp \
>   --access Allow
> ```
>
> > [!tip] 💡 Diagnóstico rápido: "no me conecta pero el puerto debería estar abierto"
> > Revisa siempre tres cosas en este orden: (1) la regla existe y está en `Permitir`, (2) el **protocolo** es el correcto (TCP vs UDP es el error más común, especialmente con WireGuard), (3) el **origen** de la regla no se restringió por error en un paso de una fase anterior (por ejemplo, si ya aplicaste parte de la Auditoría Final antes de tiempo).

### Fijar la IP privada como estática
> **Ruta del Portal:** Panel de tu VM → `Configuración de red` → pestaña `Interfaz de red` → `Configuración IP` → selecciona `ipconfig1` → cambia `Asignación` de `Dinámica` a `Estática` → `Guardar`

> [!important] 💡 Diferencia clave con un laboratorio local (Hyper-V)
> En Hyper-V, la IP se fija **dentro** del propio Windows con `New-NetIPAddress`, porque no hay ningún DHCP gestionando la red interna. En Azure, el DHCP lo gestiona la propia plataforma cloud — fijar la IP se hace **desde el lado de Azure**, marcando la asignación como "Estática" en la interfaz de red (NIC). Si intentaras fijarla manualmente dentro de Windows con `New-NetIPAddress`, entrarías en conflicto con la configuración que Azure espera controlar, arriesgando perder la conectividad al reiniciar.
>
> **Azure CLI equivalente:**
> ```bash
> az network nic ip-config update \
>   --resource-group rg-boochan-tunombre \
>   --nic-name WindowsServer-nic \
>   --name ipconfig1 \
>   --private-ip-address 10.0.0.4
> ```

### Ver la IP pública asignada
> **Ruta del Portal:** Panel principal de tu VM → campo `Dirección IP pública` (parte superior derecha del resumen)

> [!tip] 💡 ¿Reservar la IP pública como estática también?
> Por defecto, la IP pública de la VM es **dinámica**: puede cambiar si desasignas y vuelves a iniciar la VM. Para un proyecto de varias semanas como BoochanV2.1, donde el `Endpoint` de WireGuard depende de esa IP (Fase 3), es buena práctica pedir al profesor que la fije como **Estática** desde `Configuración de red → Interfaz de red → IP pública → Configuración → Asignación: Estática` — así no tienes que reconfigurar el cliente WireGuard cada vez que la VM se reinicia.

---

## 🖥️ 3. Conexión y diagnóstico remoto

### Conectar por RDP desde el Portal (descarga del archivo `.rdp`)
> **Ruta del Portal:** Panel de tu VM → botón `Conectar` (parte superior) → `RDP` → `Descargar archivo RDP`

> [!tip] 💡 Alternativa más rápida: `mstsc` directo
> No hace falta descargar el archivo `.rdp` cada vez. Una vez que conoces la IP (pública en la Fase 1-2, o del túnel VPN desde la Fase 3), basta con `Windows + R` → `mstsc` → introducir la IP directamente, o desde PowerShell: `mstsc /v:<IP>`.

### Azure Serial Console (consola de rescate si el RDP falla)
> **Ruta del Portal:** Panel de tu VM → menú izquierdo → `Solución de problemas` → `Consola serie` (Serial Console)

> [!caution] ⚠️ Tu red de seguridad si te bloqueas con el firewall
> Si en la Auditoría Final aplicas `DefaultInboundAction Block` en el Firewall de Windows Defender (o restringes el NSG) desde una IP que no está dentro del rango permitido, te quedarás fuera del servidor por RDP. La **Consola serie de Azure** es la vía de rescate: abre una consola de texto directamente sobre la VM a través de la infraestructura de Azure, sin pasar por ninguna capa de red (ni NSG ni Firewall de Windows) — es el equivalente cloud a "Conectar..." en Hyper-V Manager, la ventana de consola directa de una VM local.
>
> > [!important] Requiere habilitarla primero
> > La Consola serie necesita que el `boot diagnostics` esté activado en la VM (normalmente ya lo está por defecto) y que haya un usuario local con permisos (`azureadmin`) para iniciar sesión desde ahí. Actívala **antes** de necesitarla — comprobarlo en mitad de un bloqueo es tarde.

### Comprobar puertos en escucha dentro de la VM
> **Dentro de la VM, PowerShell como Administrador:**
> ```powershell
> Get-NetTCPConnection -State Listen | Sort-Object LocalPort
> ```
> Útil para verificar que un servicio (RDP, SMB, LDAP...) realmente está escuchando en el puerto que esperas, antes de sospechar del NSG.

---

## 💰 4. Control de costes del proyecto

> [!warning] ⚠️ Este proyecto tiene coste económico real
> A diferencia de BoochanV1.1 (gratis, en el propio portátil del alumno), cada hora que la VM `Standard_B4ms` está **asignada** (aunque esté apagada dentro de Windows) genera coste en la cuenta Azure gestionada por el profesor. Un IP pública estática reservada y sin usar también genera un pequeño coste continuo.

> [!example] Buenas prácticas de coste durante el curso
> 1. **Desasigna la VM** (no solo la apagues) al terminar cada sesión de clase, si el profesor lo indica — ver sección 1 de esta guía.
> 2. **No dupliques recursos** por error: si creaste una VM de prueba con un nombre distinto a `WindowsServer` mientras aprendías el formulario, bórrala en cuanto lo confirmes con tu profesor.
> 3. **Revisa el `Grupo de Recursos`** de vez en cuando (`Resumen → Coste acumulado` en el Portal) para tener una idea de cuánto lleva gastado el proyecto — es una competencia profesional real: en una empresa, un administrador que no vigila el coste cloud es un problema.
> 4. **Al finalizar todo el proyecto**, el profesor eliminará el Grupo de Recursos completo (`az group delete --name rg-boochan-tunombre`) para detener cualquier coste futuro — no lo hagas tú sin confirmación explícita, es una operación irreversible que borra todos los recursos del proyecto de golpe.

---

## 📋 5. Tabla resumen: comandos Azure CLI más usados en el proyecto

| Acción | Comando `az` |
| :--- | :--- |
| Crear Grupo de Recursos | `az group create --name [nombre] --location [región]` |
| Listar VMs del grupo | `az vm list --resource-group [rg] --output table` |
| Ver estado de la VM | `az vm get-instance-view --resource-group [rg] --name WindowsServer --query instanceView.statuses` |
| Iniciar VM | `az vm start --resource-group [rg] --name WindowsServer` |
| Detener y desasignar VM (parar coste) | `az vm deallocate --resource-group [rg] --name WindowsServer` |
| Ver reglas del NSG | `az network nsg rule list --resource-group [rg] --nsg-name WindowsServer-nsg --output table` |
| Crear regla de NSG | `az network nsg rule create --resource-group [rg] --nsg-name [nsg] --name [nombre] --priority [n] --destination-port-ranges [puerto] --protocol [Tcp\|Udp] --access Allow` |
| Ver IP pública actual | `az vm list-ip-addresses --resource-group [rg] --name WindowsServer --output table` |
| Eliminar todo el proyecto (⚠️ solo el profesor) | `az group delete --name [rg] --yes` |
