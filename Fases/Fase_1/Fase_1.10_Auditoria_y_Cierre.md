## Fase 1 · Apartado 10 — 🏁 Auditoría y cierre

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Infraestructura Cloud (Azure IaaS — Windows Server 2025)**
> 🧭 Índice de la fase: [[Fase_1]]
>
> **📍 Cuándo se lee:** **Lo último.** No pases a la fase siguiente sin repasarlo.

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

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_1.9_Preguntas]] | [[Fase_1]] | **Fase 2** |
