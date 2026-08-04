## Fase 3 · Apartado 7 — 🚩 Resolución de problemas

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Conectividad VPN (WireGuard para Windows)**
> 🧭 Índice de la fase: [[Fase_3]]
>
> **📍 Cuándo se lee:** **Cuando algo no salga.** Búscate por el síntoma.

---

> [!bug] Troubleshooting (¿No hay conexión?)
> | Problema | Causa Probable | Solución Sugerida |
> | :--- | :--- | :--- |
> | `wireguard.exe /installtunnelservice` falla con "Address already in use". | Ya hay otra interfaz VPN activa con esa IP. | Desinstala el servicio con `wireguard.exe /uninstalltunnelservice wg0` antes de volver a instalarlo. |
> | No hay ping entre `10.0.0.1` y `10.0.0.2`. | La regla `WireGuard` (51820 UDP) de la Fase 1 se deshabilitó o se creó como TCP por error. | Revisa en el NSG de Azure que la regla `WireGuard` esté en `Permitir` y su protocolo sea **UDP**. |
> | WireGuard no conecta pero el puerto está abierto. | Las llaves públicas están intercambiadas incorrectamente, o el Firewall de Windows Defender bloquea el puerto 51820/UDP. | Verifica las llaves públicas cruzadas. Comprueba también `Get-NetFirewallRule -DisplayName "WireGuard VPN"` en el servidor. |
> | El cliente no encuentra el `Endpoint`. | Escribiste mal la IP pública de Azure, o la VM no está encendida. | Comprueba la IP pública en el portal de Azure y confirma que el servicio `WireGuardTunnel$wg0` está activo (`Get-Service`). |

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_3.6_Procedimiento]] | [[Fase_3]] | [[Fase_3.8_Punto_de_Control]] |
