## Fase 1 · Apartado 4 — 🎯 ¿Dónde estamos?

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Infraestructura Cloud (Azure IaaS — Windows Server 2025)**
> 🧭 Índice de la fase: [[Fase_1]]
>
> **📍 Cuándo se lee:** **Antes de empezar.** De dónde vienes y a dónde llegas.

---

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

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_1.3_Obligaciones_Grabacion]] | [[Fase_1]] | [[Fase_1.5_Fundamento_Teorico]] |
