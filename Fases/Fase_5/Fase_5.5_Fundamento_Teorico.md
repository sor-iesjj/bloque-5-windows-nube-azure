## Fase 5 · Apartado 5 — 📚 Fundamento teórico

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Gestión de Identidades (Usuarios y Grupos en Active Directory)**
> 🧭 Índice de la fase: [[Fase_5]]
>
> **📍 Cuándo se lee:** **Antes de teclear.** Los conceptos que necesitas.

---

> [!abstract] 1. Una Simplificación Real (no inventada)
> En **BoochanV2** (Ubuntu + Samba AD DC sobre Azure), esta misma fase requería instalar y configurar **winbind**, un servicio "traductor" que convertía el **SID** de Windows en un **UID/GID** de Linux, porque el sistema de archivos ext4 solo entiende números Unix. Aquí, sobre **Windows Server 2025 con AD DS nativo**, ese paso **no existe y no hace falta**. Los usuarios de Active Directory ya son, de fábrica, objetos nativos del mismo ecosistema que gestiona el sistema de archivos NTFS: no hay dos "idiomas" que traducir, porque el controlador de dominio, el servidor de archivos y el sistema operativo cliente son todos del mismo fabricante y hablan el mismo protocolo de identidad (SID) de principio a fin. Lo que en Samba era un puente necesario entre dos mundos, en Windows nativo es directamente el mismo mundo.

> [!info] 2. Unidades Organizativas (OU): la carpeta de las identidades
> Una **OU** es un contenedor dentro de Active Directory que permite organizar usuarios, grupos y equipos de forma jerárquica — y, sobre todo, permite aplicar políticas (GPOs) o delegar permisos administrativos a un subconjunto concreto del dominio. No confundas una OU con un grupo: la OU organiza *dónde vive* el objeto dentro del árbol de AD; el grupo organiza *a qué tiene acceso*. Un usuario pertenece a **una sola OU** (su ubicación), pero puede ser miembro de **varios grupos** (sus permisos).

### 📖 Diccionario de Conceptos Clave

> [!quote] Terminología de Identidades
> - **SID (Security Identifier):** El identificador único y permanente que Windows asigna a cada usuario, grupo o equipo del dominio. Es el equivalente funcional al UID de Linux, pero nativo de todo el ecosistema Windows — no necesita traducción.
> - **OU (Organizational Unit):** Contenedor jerárquico de AD para organizar objetos y aplicar políticas.
> - **Grupo de Seguridad:** Conjunto de usuarios al que se le asignan permisos de forma colectiva, en lugar de usuario por usuario.
> - **Ámbito Global (Global Scope):** El tipo de grupo más habitual para agrupar usuarios de un mismo dominio; es el equivalente que usaremos aquí.
> - **`New-ADUser` / `New-ADGroup` / `New-ADOrganizationalUnit`:** Los cmdlets de PowerShell (módulo `ActiveDirectory`) que sustituyen a la "navaja suiza" `samba-tool` de Samba.

---

### 🔓 Apertura de Puertos (NSG de Azure)

> [!info] ℹ️ Sin cambios en el NSG en esta fase
> Los puertos necesarios para Active Directory (LDAP 389, Kerberos 88, DNS 53, RPC 135...) ya fueron abiertos en el Grupo de Seguridad de Red (NSG) de la VM `WindowsServer` en la **Fase 4**. No tienes que añadir ninguna regla nueva en Azure: la gestión de usuarios y grupos se hace en local, en la consola RDP del servidor o por PowerShell remoto, y no abre puertos nuevos.
>
> En este proyecto administramos el servidor por **RDP**, no por PowerShell remoto — así que el puerto **5985 (WinRM)** NO forma parte de las 12 reglas del NSG. Si quisieras usar PowerShell remoto desde tu equipo (no es necesario para el itinerario), tendrías que añadir tú mismo una regla para el 5985 en el NSG y ejecutar `Enable-PSRemoting` en el servidor. Mientras administres por RDP, no hace falta.

---

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_5.4_Donde_Estamos]] | [[Fase_5]] | [[Fase_5.6_Procedimiento]] |
