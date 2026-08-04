## Fase 2 · Apartado 4 — 🎯 ¿Dónde estamos?

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Preparación Inicial del Servidor**
> 🧭 Índice de la fase: [[Fase_2]]
>
> **📍 Cuándo se lee:** **Antes de empezar.** De dónde vienes y a dónde llegas.

---

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

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_2.3_Obligaciones_Grabacion]] | [[Fase_2]] | [[Fase_2.5_Fundamento_Teorico]] |
