## 💾 Puntos de control: cómo volver atrás

### El seguro que te permite experimentar sin miedo

> **[Módulo: SOR — Sistemas Operativos en Red]**
> **Profesor:** Pedro Navarro Miralles · IES Jorge Juan (ALICANTE)

---

> [!warning] 📖 Cómo se usa este documento
> **Esto no es una fase y no se entrega.** Es una técnica que aplicarás **al terminar cada fase**, y una instrucción que verás repetida en el apartado 8 de todas ellas.
>
> Léelo **una vez, antes de empezar la Fase 1**.

---

### 🎯 El problema que resuelve

> [!danger] La situación que te vas a encontrar
> Estás en la Fase 4, montando el dominio. Algo sale mal, intentas arreglarlo y lo empeoras. Al cabo de una hora tu servidor está en un estado que no entiendes y que no se parece a nada de lo que describe el manual.
>
> **Sin punto de control, solo tienes una opción: empezar de cero.** Con uno, vuelves al último estado bueno en minutos y repites solo la fase que se torció.

> [!success] Y sirve para algo más: para experimentar
> Un punto de control no es solo un seguro contra catástrofes. Es lo que te permite **romper cosas a propósito** para ver qué pasa.
>
> ¿Qué ocurre si te saltas un paso? ¿Si pones la máscara mal? Hazlo, míralo, aprende, y vuelve atrás.
>
> **Un administrador que puede volver atrás experimenta. Uno que no, obedece instrucciones por miedo a romper algo.**

---

### 🛠️ Cómo se hace en Azure

> [!warning] ⚠️ En la nube NO hay "instantáneas de VirtualBox"
> El concepto es el mismo, pero el mecanismo cambia: aquí se hace una **instantánea (snapshot) del disco del sistema**.

> [!example] Tomar una instantánea del disco
> 1. **Apaga la VM** desde el portal (`Detener`), para que el disco quede consistente.
> 2. Portal de Azure → tu VM → **`Discos`** → clic en el disco del sistema → **`Crear instantánea`**.
> 3. Nómbrala **`Fase N terminada`** y guárdala en el mismo grupo de recursos.
>
> Por CLI:
> ```bash
> az snapshot create -g rg-boochan-TUNOMBRE -n "Fase-N-terminada" \
>    --source $(az vm show -g rg-boochan-TUNOMBRE -n UbuntuServer --query storageProfile.osDisk.managedDisk.id -o tsv)
> ```

> [!example] Volver atrás
> Restaurar **no es un botón**: se crea un disco nuevo a partir de la instantánea y se intercambia por el actual (`Cambiar disco del SO`). Es más laborioso que en local — otra razón para no romper cosas alegremente en la nube.

> [!danger] 💰 Las instantáneas CUESTAN dinero
> A diferencia de VirtualBox, aquí ocupas almacenamiento facturable mientras existan. **Borra las que ya no necesites** y, al terminar el proyecto, **elimina el grupo de recursos entero** — se lleva por delante VM, discos e instantáneas de una vez.
---

> [!summary] 🎓 Qué has aprendido
> Que **antes de tocar algo importante, se guarda el estado al que poder volver**. Cambia el nombre según dónde estés —instantánea, punto de control, snapshot, imagen— pero la idea es la misma en todas partes, y en el código se llama `commit`.
>
> **No avances sin poder retroceder.**
