## Fase 4 · Apartado 7 — 🚩 Resolución de problemas

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Aprovisionamiento del Dominio (AD DS nativo)**
> 🧭 Índice de la fase: [[Fase_4]]
>
> **📍 Cuándo se lee:** **Cuando algo no salga.** Búscate por el síntoma.

---

> [!bug] Troubleshooting (¿El dominio no nace?)
> | Problema | Causa Probable | Solución Sugerida |
> | :--- | :--- | :--- |
> | Error "El nombre de equipo del equipo local no coincide con..." o similar. | El servidor no se renombró correctamente en la Fase 2, o el reinicio del renombrado no se completó antes de instalar AD DS. | Ejecuta `$env:COMPUTERNAME` y confirma que devuelve `WindowsServer`. Si no, repite el Paso 1 de la Fase 2. |
> | Error "La dirección IP no está configurada estáticamente". | La IP privada de la NIC en Azure sigue en modo `Dinámica` en vez de `Estática`. | Repite el Paso 3 de la Fase 2 (fijar la IP estática desde el portal de Azure) antes de reintentar. |
> | `Install-ADDSForest` se detiene con un error de prerrequisitos (`Test-ADDSForestInstallation`). | Falta de RAM, o el nombre NetBIOS ya existe en la red (poco probable en un entorno de alumno aislado). | Revisa el mensaje de error completo — PowerShell describe exactamente qué prerrequisito falló. |
> | El servidor no responde tras el reinicio automático. | El reinicio tarda más de lo habitual mientras se inicializan los servicios de AD DS por primera vez. | Espera 2-3 minutos adicionales antes de intentar reconectar por RDP. Es normal en el primer arranque tras la promoción. |
> | `Resolve-DnsName` no encuentra el registro SRV de Kerberos. | El servicio DNS no terminó de configurarse, o el adaptador de red no apunta al DNS correcto. | Ejecuta `Get-Service DNS` y confirma que está `Running`. Revisa `Get-DnsClientServerAddress` como en el Paso 4. |
> | No puedo reconectar por RDP tras el reinicio automático de la promoción. | El servidor tarda unos minutos en levantar los servicios de AD DS tras el reinicio. **El RDP público por la IP de Azure sigue abierto** (no se cierra hasta la Auditoría Final), así que puedes reconectar por cualquiera de las dos vías. | Espera 2-3 minutos y reconecta por la IP pública (`mstsc /v:<IP_pública>`) o, si lo prefieres, por el túnel (`mstsc /v:10.0.0.1`). Si el túnel no responde, verifica `wg show` y que el servicio `WireGuardTunnel$wg0` siga `Running` tras el reinicio. |

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_4.6_Procedimiento]] | [[Fase_4]] | [[Fase_4.8_Punto_de_Control]] |
