# 🚀 BoochanV2.1 — Infraestructura de Servidores Cloud sobre Azure (Windows Server 2025 + AD DS nativo)

> **Módulo:** Sistemas Operativos en Red (SOR) · 2.º Curso SMR
> **Profesor:** Pedro Navarro Miralles · IES Jorge Juan (Alicante)
> **Correo:** p.navarromiralles2@edu.gva.es
> **Entorno:** Microsoft Azure (IaaS) — hermana de BoochanV2 (Ubuntu + Samba AD DC sobre Azure) y de BoochanV1.1 (AD DS nativo sobre Hyper-V local)
> **RA cubiertos:** RA.01, RA.02, RA.03, RA.04, RA.05, RA.06
> **⏱️ Tiempo estimado total:** ~13-14 horas repartidas en 9 sesiones (ver desglose por fase más abajo)

---

## ¿Qué es este proyecto?

BoochanV2.1 es un itinerario práctico de **8 fases + auditoría final** en el que el alumno construye, desde cero y **en la nube real de Microsoft Azure**, una infraestructura profesional completa: un servidor **Windows Server 2025** con **Controlador de Dominio (AD DS nativo)**, **VPN WireGuard**, **cuotas de disco (FSRM)**, **permisos avanzados (NTFS + Access-Based Enumeration)** y un **cliente Windows 11 físico del aula** integrado en el dominio a través de un túnel cifrado — todo ello desplegado como recursos reales de una cuenta cloud, protegidos por un **NSG (Network Security Group)** perimetral.

Es la versión **100% Microsoft, en la nube** del proyecto Boochan. La teoría, la estructura del itinerario y los conceptos son los mismos que en BoochanV1 (VirtualBox/Ubuntu), BoochanV1.1 (Hyper-V/Windows Server) y BoochanV2 (Azure/Ubuntu), pero aquí se sustituye Samba AD DC por el rol **AD DS nativo** de Windows Server, manteniendo la misma infraestructura Azure que ya usaba BoochanV2.

---

## Relación con BoochanV2 (Azure + Ubuntu + Samba)

BoochanV2.1 comparte con BoochanV2 el **mismo escenario cloud completo**: misma cuenta Azure, mismo Grupo de Recursos, mismo dominio Active Directory (`BOOCHAN` / `BOOCHAN.SPACE`), mismo NSG con los mismos 12 puertos perimetrales, misma red privada `10.0.0.0/24`, y la misma VPN WireGuard para blindar el acceso remoto. Un alumno que complete cualquiera de las dos versiones ha aprendido exactamente los mismos conceptos de administración de sistemas en red en la nube — solo cambia el fabricante y la implementación técnica del Controlador de Dominio.

La diferencia de fondo es que BoochanV2 usa **Samba AD DC sobre Ubuntu Server 26.04**, una reimplementación de código abierto de Active Directory que exige ensamblar manualmente piezas independientes (`samba-tool`, `winbind` como traductor SID↔UID/GID, Loop Devices para las cuotas, `setfacl` + Access Based Enumeration emulada en `smb.conf`, y un script externo `provision_boochan.sh` clonado de un repositorio Git). BoochanV2.1 usa el **producto original de Microsoft**: el rol AD DS se promociona con un único cmdlet nativo (`Install-ADDSForest`), no hace falta ningún traductor de identidades porque usuarios, sistema de archivos y controlador de dominio hablan el mismo idioma (SID) de principio a fin, las cuotas se aplican directamente sobre NTFS con FSRM sin discos virtuales de por medio (`Loop Devices` + `fstab`), y Access-Based Enumeration es una función nativa del recurso SMB en lugar de una imitación (`access based share enum = yes` en `smb.conf`). Ninguna de las dos filosofías es "mejor" en abstracto: Samba es gratuito y corre sobre Linux; AD DS requiere licencia de Windows Server pero ofrece una experiencia mucho más asistida y con menos piezas artesanales que puedan fallar a medio camino.

También cambia el dimensionado de la máquina: BoochanV2 usa `Standard_B2s` (2 vCPU, 4 GB RAM), suficiente para un Ubuntu Server headless; BoochanV2.1 necesita `Standard_B4ms` (4 vCPU, 16 GB RAM) porque Windows Server con Desktop Experience y el rol AD DS consumen bastantes más recursos que un Linux sin interfaz gráfica. El puerto de administración remota también cambia de naturaleza: SSH (puerto 2222/22, terminal de texto) da paso a **RDP** (puerto 3389, escritorio gráfico completo).

## Relación con BoochanV1.1 (Hyper-V + Windows Server, local)

Frente a la versión local, BoochanV2.1 comparte con BoochanV1.1 el **mismo Controlador de Dominio AD DS nativo**: mismo cmdlet `Install-ADDSForest`, misma familia de comandos PowerShell/AD DS/FSRM, mismos conceptos de SID nativo sin traducción, misma Access-Based Enumeration real de Windows Server. Un alumno que haya hecho BoochanV1.1 reconocerá casi todos los comandos de esta versión — la diferencia no está en "qué se administra", sino en "dónde vive el servidor".

La diferencia de fondo es la infraestructura subyacente: BoochanV1.1 usa **Hyper-V** (el hipervisor de Windows 11 Pro/Enterprise/Education) para crear dos máquinas virtuales — servidor y cliente — dentro del propio portátil del alumno, sin cuenta cloud, sin coste y sin depender de la conexión del aula, pero también sin disponibilidad 24/7 ni acceso fuera del aula. BoochanV2.1 despliega el servidor como una **VM real en Azure** (con IP pública, NSG, coste asociado a la cuenta del profesor) y usa el **PC físico del aula como cliente Windows 11** — no una máquina virtual anidada, sino el propio ordenador del alumno unido al dominio a través de un túnel WireGuard sobre Internet. Esto introduce capas de complejidad ausentes en el laboratorio local: el NSG de Azure como muralla perimetral adicional (BoochanV1.1 solo tenía el Firewall de Windows Defender local), la latencia real de Internet, y la necesidad de una VPN funcional para que cliente y servidor —separados físicamente por cientos de kilómetros— se comporten como si estuvieran en la misma red.

---

## ⚠️ Antes de empezar: requisitos del proyecto (LÉEME)

- **Cuenta Azure activa y gestionada por el profesor.** A diferencia de BoochanV1.1 (gratis, en el propio portátil), este proyecto **tiene coste económico real** asociado al Grupo de Recursos de Azure. El alumno no crea ni paga la cuenta; el profesor la gestiona y debe indicar las credenciales de acceso al Azure Portal antes de empezar la Fase 1.
- **Apagar o eliminar los recursos al terminar cada sesión** si así lo indica el profesor — una VM `Standard_B4ms` (4 vCPU, 16 GB RAM) encendida sin usar genera coste innecesario.
- **Windows 11 (PC físico del aula) como cliente**, necesario a partir de la Fase 8. No requiere ninguna característica de virtualización especial (a diferencia de BoochanV1.1, que exigía Windows 11 Pro/Enterprise/Education para Hyper-V) — cualquier edición de Windows 11 sirve, porque el cliente es el propio PC, no una VM anidada.
- **Cliente de Escritorio Remoto (RDP)**: ya viene instalado en Windows; en Mac hay que descargar "Microsoft Remote Desktop" desde la App Store; en Linux, `Remmina` o `rdesktop`.
- **WireGuard para Windows**, necesario desde la Fase 3, tanto en el servidor como en el PC del aula.
- **Conexión a Internet estable en el aula.** A diferencia de BoochanV1.1 (que no depende de la conexión del instituto porque todo vive en el propio portátil), aquí tanto el acceso RDP inicial como el túnel VPN dependen de que el aula tenga salida a Internet.

---

## 🗺️ Índice de fases

| Fase | Título | Concepto Azure / Windows Server clave |
|------|--------|-------------------------------------|
| [1](Fases/Fase_1.md) | Infraestructura Cloud (Azure IaaS) | VM `WindowsServer` (`Standard_B4ms`), NSG con 12 reglas, usuario `azureadmin`, RDP |
| [2](Fases/Fase_2.md) | Preparación Inicial del Servidor | `Rename-Computer`, IP privada estática desde el portal de Azure, DNS a `127.0.0.1`, Windows Update |
| [3](Fases/Fase_3.md) | Conectividad VPN (WireGuard para Windows) | Túnel cifrado servidor↔aula sobre Internet real, Tunnel Service, `PersistentKeepalive` |
| [4](Fases/Fase_4.md) | Aprovisionamiento del Dominio (AD DS nativo) | `Install-ADDSForest`, NTDS, Kerberos, DNS integrado, sin script externo |
| [5](Fases/Fase_5.md) | Gestión de Identidades (Usuarios y Grupos) | `New-ADUser`/`New-ADGroup`, SID nativo sin traducción, OUs |
| [6](Fases/Fase_6.md) | Almacenamiento con Cuotas (FSRM) | File Server Resource Manager, cuota directa sobre NTFS del disco de la VM, sin discos virtuales |
| [7](Fases/Fase_7.md) | Seguridad Avanzada (NTFS + ABE) | `icacls`, herencia `(OI)(CI)`, Access-Based Enumeration nativa de SMB |
| [8](Fases/Fase_8.md) | Integración del Cliente (Windows 11) | PC físico del aula unido vía VPN, `Add-Computer`, RSAT, mapeo de unidades |
| [Final](Fases/Auditoria_Final.md) | Auditoría Final y Hardening | Zero Trust en dos capas: NSG de Azure + Firewall de Windows Defender local |

### Resumen de cada fase

**[Fase 1 — Infraestructura Cloud (Azure IaaS)](Fases/Fase_1.md):** se crea el Grupo de Recursos `rg-boochan-[tunombre]` y la VM `WindowsServer` (Windows Server 2025 Datacenter: Azure Edition, `Standard_B4ms` — 4 vCPU, 16 GB RAM) con el usuario administrador local `azureadmin`. Se configura de una sola vez el **NSG completo del proyecto** con las 12 reglas que se usarán en todas las fases siguientes, y se realiza la primera conexión por **RDP**. Se fija el nombre del proyecto: `BOOCHAN` / `BOOCHAN.SPACE`.

**[Fase 2 — Preparación Inicial del Servidor](Fases/Fase_2.md):** a diferencia de Ubuntu (donde había que purgar Samba preinstalado), Windows Server no trae nada que estorbar — el trabajo es dar identidad. Se renombra el equipo a `WindowsServer` (`Rename-Computer`), se verifica la IP privada asignada por Azure en el rango `10.0.0.x` y se fija como **estática desde el propio portal de Azure** (no dentro de Windows, a diferencia de Hyper-V), se prepara el DNS local a `127.0.0.1` (anticipando la Fase 4) y se parchea el sistema con Windows Update antes de instalar ningún rol crítico.

**[Fase 3 — Conectividad VPN (WireGuard para Windows)](Fases/Fase_3.md):** se instala WireGuard para Windows y se construye un túnel cifrado punto a punto entre el servidor en Azure y el PC del aula, sobre Internet real (a diferencia de BoochanV1.1, donde el conmutador interno de Hyper-V ya estaba aislado por diseño). El puerto UDP 51820 ya se abrió en el NSG en la Fase 1. El cierre definitivo del RDP público se pospone deliberadamente hasta la Auditoría Final.

**[Fase 4 — Aprovisionamiento del Dominio (AD DS nativo)](Fases/Fase_4.md):** un único cmdlet, `Install-ADDSForest`, sustituye por completo al script `provision_boochan.sh` de BoochanV2: instala la base de datos NTDS, configura Kerberos, activa el DNS integrado y el nivel funcional `Win2025`, sin script externo ni repositorio Git que clonar. Se provisiona el dominio `BOOCHAN.SPACE` (NetBIOS `BOOCHAN`) — el mismo Realm que usa BoochanV2.

**[Fase 5 — Gestión de Identidades (Usuarios y Grupos)](Fases/Fase_5.md):** se crean las OUs `Departamentos`/`Policia`/`Bomberos`, los grupos de seguridad `Policia` y `Bomberos`, y los usuarios `user1` y `user2` con `New-ADUser`/`New-ADGroup`. La simplificación clave: no existe ningún paso equivalente a `winbind` — los usuarios de AD ya son objetos nativos del mismo ecosistema que NTFS, sin traducción de identidades.

**[Fase 6 — Almacenamiento con Cuotas (FSRM)](Fases/Fase_6.md):** se instala el rol **FSRM** y se aplican cuotas físicas de 5 GB directamente sobre dos carpetas NTFS reales (`C:\ShareData\Prueba1` y `C:\ShareData\Prueba3`) del disco del sistema operativo de la propia VM, con `New-FsrmQuotaTemplate`/`New-FsrmQuota`, sin crear ningún disco virtual (Loop Device) como en BoochanV2 — ni riesgo de romper el arranque por un `fstab` mal escrito, ni necesidad de añadir un disco de datos adicional en Azure.

**[Fase 7 — Seguridad Avanzada (NTFS + ABE)](Fases/Fase_7.md):** se rompe la herencia de permisos de `Prueba3` con `icacls /inheritance:r` y se concede acceso exclusivo al grupo `Policia` con herencia `(OI)(CI)`, y se activa **Access-Based Enumeration nativa** (`New-SmbShare -FolderEnumerationMode AccessBased`) — no una imitación como en Samba, sino la función real de Windows Server, sin tocar ningún archivo de configuración ni reiniciar ningún servicio.

**[Fase 8 — Integración del Cliente (Windows 11)](Fases/Fase_8.md):** el **PC físico del aula** (no una VM anidada, a diferencia de BoochanV1.1) activa el túnel WireGuard, sincroniza su reloj (`w32tm /resync /force`), se une al dominio con `Add-Computer -DomainName "BOOCHAN.SPACE"` y mapea las carpetas compartidas — demostrando que el modelo de permisos NTFS + ABE se respeta desde un cliente real separado físicamente del servidor por cientos de kilómetros, conectado únicamente por el túnel cifrado.

**[Auditoría Final — Hardening](Fases/Auditoria_Final.md):** cierre de seguridad con el principio Zero Trust aplicado en **dos capas independientes**: el **NSG de Azure** (restringiendo el origen de casi todas las reglas de `Any` a `10.0.0.0/24`, la red de la VPN) y el **Firewall de Windows Defender con Seguridad Avanzada** dentro del propio servidor (`DefaultInboundAction Block` en el perfil de Dominio). El puerto WireGuard `51820/udp` es la única excepción que se mantiene abierta a `Any` en el NSG — es la puerta de entrada al propio túnel.

---

## 📊 Datos clave del proyecto

| Concepto | Valor en BoochanV2.1 |
| :--- | :--- |
| **Nombre NetBIOS** | `BOOCHAN` |
| **Realm (dominio completo)** | `BOOCHAN.SPACE` |
| **FQDN del servidor** | `WindowsServer.BOOCHAN.SPACE` |
| **VM del servidor** | `WindowsServer` — `Standard_B4ms` (4 vCPU, 16 GB RAM), Windows Server 2025 Datacenter: Azure Edition |
| **IP privada del servidor (Azure, VNet)** | `10.0.0.x` (rango de la subred de Azure, fijada como estática desde el portal en la Fase 2) |
| **Red del túnel VPN (WireGuard)** | `10.0.0.0/24` — coincide con el rango de Azure a propósito; el túnel encapsula y cifra el tráfico dentro de ese mismo rango lógico |
| **Usuario administrador local del servidor** | `azureadmin` |
| **Usuario Administrador del dominio** | `BOOCHAN\Administrator` |
| **Grupo de Recursos de Azure** | `rg-boochan-[tunombre]` |
| **Usuarios de dominio de ejemplo** | `user1` (grupo `Policia`) · `user2` (grupo `Bomberos`) |
| **Sistema operativo servidor** | Windows Server 2025 Datacenter: Azure Edition (Desktop Experience) |
| **Sistema operativo cliente** | Windows 11 (PC físico del aula) |
| **Protocolo de administración remota** | RDP (puerto 3389) — sustituye al SSH de la versión Ubuntu |
| **NSG (Network Security Group)** | 12 reglas: Gestion_Web (9090), RDP (3389), Kerberos_Auth (88), DNS_Query (53), RPC_Endpoint (135), LDAP_Auth (389), SMB_Files (445), LDAPS (636), RPC_Dinamico (49152-65535), Kerberos_Pass (464), NTP_Time (123), WireGuard (51820) |

---

## ⚖️ Comparativa breve: Samba AD DC (V2) vs. AD DS nativo (V2.1)

| Concepto en BoochanV2 (Samba/Ubuntu) | Equivalente en BoochanV2.1 (AD DS/Windows Server) |
| :--- | :--- |
| Samba AD DC (`samba-tool domain provision`) | **AD DS nativo** (`Install-ADDSForest`) |
| Script externo `provision_boochan.sh` clonado de un repositorio Git | Cmdlet nativo único, sin script ni repositorio externo |
| `winbind` (traductor SID ↔ UID/GID) | No existe — el SID es nativo en todo el ecosistema Windows |
| `samba-tool user create --uid=... --gid-number=...` | `New-ADUser` sin ningún identificador manual — el SID lo asigna el propio controlador de dominio |
| `setfacl` / `getfacl` (ACLs de Linux) | `icacls` / `Set-Acl` (ACLs NTFS) |
| Access Based Enumeration **emulada** en `smb.conf` (`access based share enum = yes`) | Access-Based Enumeration **nativa** (`New-SmbShare -FolderEnumerationMode AccessBased`) |
| Loop Devices (`dd` + `mkfs.ext4` + `fstab` con `loop`) para las cuotas | **FSRM** (`New-FsrmQuota`) directamente sobre NTFS del disco de la VM, sin discos virtuales |
| SSH (puerto 2222/22) como acceso remoto | **RDP** (puerto 3389) — escritorio gráfico completo, no solo terminal |
| Ubuntu Server headless (`Standard_B2s`, 2 vCPU / 4 GB RAM) | Windows Server con Desktop Experience (`Standard_B4ms`, 4 vCPU / 16 GB RAM) |
| Purga agresiva de Samba/CUPS preinstalados (`apt purge`) | No hace falta — Windows Server no trae ningún rol activado de fábrica |
| `chattr +i /etc/resolv.conf` para "blindar" el DNS | El propio DNS Server integrado de AD DS se convierte en la fuente de verdad, sin fichero que proteger |
| `nano` / edición manual de `smb.conf` | PowerShell (cmdlets sobre objetos, no ficheros de configuración planos) |

---

## 📂 Estructura de la carpeta

```
BoochanV2.1/
├── Manual_BoochanV2.1.md         ← este documento (punto de entrada)
├── Fases/
│   ├── Fase_1.md … Fase_8.md     ← las 8 fases del itinerario
│   ├── Auditoria_Final.md        ← cierre de seguridad (hardening NSG + Firewall de Windows Defender)
│   └── Solucionario/             ← respuestas y retos resueltos (1 por fase)
└── 99_Recursos/
    ├── Diccionario_Comandos_Sistema.md    ← comandos PowerShell / AD DS / FSRM
    ├── Comandos_Azure_CLI_Portal.md       ← específico de BoochanV2.1 (gestión de la VM/NSG en Azure)
    └── Guía_Errores_y_Resolución.md       ← catálogo de errores por fase
```

---

## 🧭 Recomendación de uso

1. Lee este manual y la advertencia de requisitos (cuenta Azure gestionada por el profesor, coste asociado, Windows 11 del aula, WireGuard).
2. Sigue las fases **en orden** — son dependientes entre sí (las Fases 4, 5, 7 y 8 son secuenciales; la Fase 8 requiere las Fases 1-7 completas y un PC físico del aula disponible).
3. Si algo falla, antes de bloquearte consulta **[99_Recursos/Guía_Errores_y_Resolución.md](99_Recursos/Guía_Errores_y_Resolución.md)**, organizada por fase, e incluye los problemas específicos de Azure (NSG mal configurado, IP dinámica no fijada, coste de recursos olvidados encendidos).
4. Para repasar cmdlets de PowerShell/AD DS/FSRM consulta **[99_Recursos/Diccionario_Comandos_Sistema.md](99_Recursos/Diccionario_Comandos_Sistema.md)**; para la gestión de la VM y el NSG desde el portal o Azure CLI, consulta **[99_Recursos/Comandos_Azure_CLI_Portal.md](99_Recursos/Comandos_Azure_CLI_Portal.md)**.
5. Al terminar cada sesión, si el profesor lo indica, apaga la VM (`Stop-VM`/desasignar desde el portal) para no generar coste innecesario mientras no se usa — a diferencia de BoochanV1.1, aquí el "servidor" sigue existiendo (y costando) aunque cierres tu portátil.

---

> **Nota sobre IPs:** a lo largo del proyecto conviven **dos** rangos relevantes, no los confundas: la **IP privada de la VNet de Azure `10.0.0.x`** (la tarjeta de red real de la VM, fijada como estática en la Fase 2) y la **IP pública de Azure** (asignada en la Fase 1, usada para la primera conexión RDP y como `Endpoint` del túnel WireGuard). La red del **túnel WireGuard** reutiliza deliberadamente el mismo rango lógico `10.0.0.0/24` que la VNet de Azure — es una capa de cifrado adicional que viaja encapsulada dentro del tráfico real de Internet, no una red física distinta como ocurría en BoochanV1.1 con sus `10.10.10.0/24` (conmutador interno) y `10.20.20.0/24` (túnel) separados.
