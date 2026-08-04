## Fase 1 · Apartado 7 — 🚩 Resolución de problemas

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Infraestructura Cloud (Azure IaaS — Windows Server 2025)**
> 🧭 Índice de la fase: [[Fase_1]]
>
> **📍 Cuándo se lee:** **Cuando algo no salga.** Búscate por el síntoma.

---

> [!bug] Tabla de Troubleshooting (¿Algo no funciona?)
> | Problema | Causa Probable | Solución Sugerida |
> | :--- | :--- | :--- |
> | No puedo conectar por RDP ("No se puede establecer la conexión"). | La VM no ha terminado de arrancar/aprovisionarse. | Espera 3-5 minutos (Windows Server tarda más que Ubuntu) y vuelve a intentarlo. |
> | RDP se conecta pero rechaza las credenciales. | Usuario o contraseña incorrectos, o Bloq Mayús activado. | Comprueba que escribes exactamente `azureadmin` y la contraseña tal cual, respetando mayúsculas/símbolos. |
> | Aparece "Este equipo no puede conectarse al equipo remoto" o error de licencia/protocolo. | El puerto 3389 no está abierto en el NSG, o la regla se borró por error. | Revisa **Configuración de red → NSG → Reglas de seguridad de entrada** y confirma que la regla RDP (3389) existe y está en `Permitir`. |
> | El servidor **no responde al ping**. | El protocolo ICMP está bloqueado por defecto en Azure para tráfico entrante. | Es normal por seguridad. No abras el ping de entrada; usa RDP o prueba de puertos TCP para verificar conectividad. |
> | La pantalla de RDP se ve muy lenta o pixelada. | Configuración de calidad de la conexión demasiado alta para el ancho de banda del aula. | En el cliente RDP, antes de conectar, baja la calidad de color/experiencia en las opciones avanzadas de la conexión. |

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_1.6_Procedimiento]] | [[Fase_1]] | [[Fase_1.8_Punto_de_Control]] |
