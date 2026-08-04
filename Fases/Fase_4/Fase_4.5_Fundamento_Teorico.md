## Fase 4 · Apartado 5 — 📚 Fundamento teórico

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Aprovisionamiento del Dominio (AD DS nativo)**
> 🧭 Índice de la fase: [[Fase_4]]
>
> **📍 Cuándo se lee:** **Antes de teclear.** Los conceptos que necesitas.

---

> [!abstract] 1. El "Cerebro" de la Red: Active Directory (AD)
> Estamos creando el **Active Directory**. Este es el "Cerebro" que gestiona la base de datos de todos los objetos de la red: usuarios, grupos y ordenadores. El rol AD DS instala tres servicios vitales para que esto funcione:
> *   **NTDS (base de datos del directorio):** El equivalente al LDAP de Samba — la base de datos jerárquica donde vive cada objeto del dominio.
> *   **Kerberos:** El sistema de "tickets" de seguridad (como un pase VIP de un festival).
> *   **DNS integrado:** El propio rol DNS Server, instalado junto con AD DS, gestiona los registros SRV que indican dónde están los servicios de red.

> [!important] 2. De lo artesanal a lo asistido: `provision_boochan.sh` vs. `Install-ADDSForest`
> En BoochanV2 (Samba), provisionar el dominio requería un **script externo** (`provision_boochan.sh`), clonado desde un repositorio Git, que ejecutaba `samba-tool domain provision`, configuraba a mano el `resolv.conf`, copiaba el `krb5.conf` generado y activaba los servicios uno por uno con `systemctl`. Era un proceso "artesanal": cada paso era responsabilidad del script y un solo error a mitad de camino podía dejar el dominio a medias.
>
> En Windows Server, **no hace falta ningún script externo ni repositorio que clonar**. `Install-ADDSForest` es un único cmdlet nativo del sistema operativo que orquesta *todo* el proceso: instala la base de datos NTDS, configura Kerberos, activa el DNS integrado, establece el nivel funcional del dominio y reinicia el servidor cuando corresponde — todo con validaciones automáticas en cada paso (`Test-ADDSForestInstallation` se ejecuta de fondo antes de aplicar ningún cambio). Es un contraste interesante entre dos filosofías de administración:
> *   **Samba AD DC (artesanal):** una **reimplementación de código abierto** de Active Directory que requiere ensamblar manualmente piezas independientes (LDAP, Kerberos, DNS, Winbind) mediante un script.
> *   **AD DS (asistido):** el **producto original de Microsoft**, con un asistente guiado (gráfico o por PowerShell) que hace ese ensamblaje por ti en un solo cmdlet.
>
> Ninguno es "mejor" en abstracto — Samba es gratuito y funciona sobre Linux; AD DS requiere licencia de Windows Server pero ofrece una experiencia de instalación mucho más asistida y menos propensa a errores humanos.

### 📖 Diccionario de Conceptos Clave

> [!quote] Terminología de Dominio
> - **Reino (Realm):** El nombre de dominio completo (ej. `BOOCHAN.SPACE`). Siempre se escribe en **MAYÚSCULAS** para que Kerberos lo entienda.
> - **NetBIOS Domain:** El nombre corto del dominio (ej. `BOOCHAN`), usado por protocolos Windows heredados y como prefijo de inicio de sesión (`BOOCHAN\usuario`).
> - **Nivel Funcional (Functional Level):** El conjunto de características avanzadas de Active Directory disponibles en un dominio o bosque, determinado por la versión mínima de Windows Server que puede actuar como Controlador de Dominio. En BoochanV2.1 usamos el nivel más alto disponible, porque no hay Controladores de Dominio antiguos con los que mantener compatibilidad.
> - **DSRM (Directory Services Restore Mode):** Modo de arranque especial de un Controlador de Dominio para tareas de recuperación de emergencia. Requiere una contraseña propia, distinta de la del Administrador del dominio.
> - **SRV Record:** Un registro DNS especial que indica qué servidor ofrece un servicio específico (ej. "el servidor de tickets está en esta IP").
> - **Provisionamiento:** El acto de generar la base de datos del dominio desde cero.

---

### 🔓 Puertos del Dominio: ya abiertos desde la Fase 1

> [!info] Los 9 puertos de Active Directory ya están en el NSG
> Active Directory es un ecosistema de servicios que se hablan entre sí: si falta un puerto, los clientes Windows no pueden autenticarse ni resolver nombres. En BoochanV2.1 no hace falta que vuelvas al portal de Azure para esta fase — todos estos puertos se abrieron de una sola vez en la Fase 1, junto con el resto del NSG del proyecto:
>
> | Prioridad | Nombre | Puerto | Protocolo | Para qué sirve ahora |
> | :--- | :--- | :--- | :--- | :--- |
> | **300** | Kerberos_Auth | 88 | TCP/UDP | Emite los "tickets" de seguridad que identifican a cada usuario del dominio. |
> | **310** | DNS_Query | 53 | TCP/UDP | Resuelve los nombres del dominio (ej. `BOOCHAN.SPACE`). |
> | **320** | RPC_Endpoint | 135 | TCP | Punto de entrada para las llamadas a procedimiento remoto de Windows. |
> | **330** | LDAP_Auth | 389 | TCP/UDP | Permite consultar el directorio de usuarios y grupos del dominio. |
> | **340** | SMB_Files | 445 | TCP | Acceso a las carpetas compartidas del servidor (SYSVOL, NETLOGON). |
> | **350** | LDAPS | 636 | TCP | Versión cifrada de LDAP — protege las consultas de usuarios en tránsito. |
> | **360** | RPC_Dinamico | 49152-65535 | TCP | Rango de puertos que Active Directory negocia dinámicamente para comunicarse. |
> | **370** | Kerberos_Pass | 464 | TCP/UDP | Gestión de cambios de contraseña de los usuarios del dominio. |
> | **380** | NTP_Time | 123 | UDP | Sincronización horaria del servidor — Kerberos falla si el reloj difiere más de 5 minutos. |
>
> Si quieres comprobarlo antes de empezar, entra en **`Configuración de red`** de tu VM → tu NSG → **`Reglas de seguridad de entrada`** y verifica que estas reglas siguen en `Permitir`. Si alguna falta o aparece deshabilitada, revisa la Fase 1 antes de continuar — sin ella, el dominio se provisionará igualmente, pero algún cliente futuro no podrá hablar con él.
>
> > [!tip] 💡 El Firewall de Windows también se reconfigura, pero automáticamente
> > A diferencia del NSG de Azure (que ya abriste tú a mano en la Fase 1), el **Firewall de Windows Defender local** del propio servidor se reconfigura solo durante `Install-ADDSForest`: el asistente crea las reglas necesarias para todo el tráfico de AD DS sin que tengas que tocarlas.

---

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_4.4_Donde_Estamos]] | [[Fase_4]] | [[Fase_4.6_Procedimiento]] |
