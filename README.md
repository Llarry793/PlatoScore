# PlatoScore 🎯

**Aplicación Android para la gestión de competiciones de tiro al plato.** PlatoScore permite registrar tiradas, organizar tiradores en escuadras y calcular automáticamente clasificaciones por categoría, todo ello con persistencia local mediante Room y una arquitectura MVVM limpia en Kotlin.

***

## Descripción

PlatoScore nace para digitalizar y simplificar la gestión de competiciones de tiro al plato. Antes de la app, organizar una tirada implicaba papeles, cálculos manuales y errores en las clasificaciones. Con PlatoScore, el responsable de la competición puede registrar todos los datos desde su móvil Android, asignar tiradores a escuadras, registrar platos rotos y obtener automáticamente el ranking por categoría (local, general, junior, senior y dama).

***

## Características principales

- 📋 **Gestión de tiradas**: crea, edita y elimina competiciones con nombre, fecha y precios diferenciados por categoría (local, general, junior, senior, dama).[^1]
- 👥 **Gestión de escuadras**: organiza los tiradores en escuadras dentro de cada tirada, con relación 1-N gestionada por Room.[^2]
- 🎯 **Gestión de tiradores**: registra nombre, apellidos, DNI, número de licencia, platos rotos y categorías de cada participante.[^3]
- 🏆 **Clasificaciones automáticas**: el sistema calcula en tiempo real la posición de cada tirador en su categoría (local/general y subcategorías), ordenando por platos rotos de forma descendente.[^4]
- 💰 **Cálculo de puntuación económica**: el total de puntos de cada tirador se calcula multiplicando sus platos rotos por el precio de su categoría.[^4]
- 💾 **Persistencia local**: todos los datos se almacenan en una base de datos SQLite local mediante Room, sin necesidad de conexión a internet.[^3][^1][^2]

***

## Arquitectura

El proyecto sigue el patrón **MVVM (Model-View-ViewModel)** recomendado por Google para aplicaciones Android, con una capa de repositorios que desacopla los ViewModels del acceso directo a la base de datos.

```
UI (Fragments / Activities)
        │
        ▼
   ViewModels  ◄── LiveData ──► UI reactiva
        │
        ▼
  Repositorios
        │
        ▼
   Room DAOs  ──► Base de datos SQLite local
```

### Capas del proyecto

| Capa | Clases | Responsabilidad |
|------|--------|-----------------|
| **Modelo** | `Tirada`, `Escuadra`, `Tirador`, `Resultado`, `EscuadraConTiradores` | Entidades de dominio y relaciones Room[^3][^5][^1][^6][^2] |
| **ViewModel** | `TiradaViewModel`, `EscuadraViewModel`, `TiradorViewModel`, `ResultadoViewModel` | Lógica de negocio, operaciones CRUD y generación de resultados[^7][^8][^9][^4] |
| **Repositorio** | `TiradaRepository`, `EscuadraRepository`, `TiradorRepository` | Abstracción del acceso a datos, operaciones suspendidas con corrutinas[^7][^8][^9] |
| **Base de datos** | `PlatoScoreDatabase` (Room) | Configuración de la base de datos SQLite y acceso a los DAOs[^7][^8][^9] |

***

## Modelo de datos

### Tirada[^1]
Entidad principal que representa una competición.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id` | Int (PK, autoGenerate) | Identificador único |
| `nombre` | String | Nombre de la tirada |
| `fecha` | String | Fecha de la competición |
| `precioLocal` | Float | Precio para tiradores locales |
| `precioGeneral` | Float | Precio para tiradores generales |
| `precioJunior` | Float | Precio para categoría junior |
| `precioSenior` | Float | Precio para categoría senior |
| `precioDama` | Float | Precio para categoría dama |

### Escuadra[^2]
Agrupación de tiradores dentro de una tirada.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id` | Int (PK, autoGenerate) | Identificador único |
| `tiradaId` | Int (FK) | Referencia a la tirada (CASCADE delete) |
| `numeroEscuadra` | Int | Número de escuadra dentro de la tirada |

### Tirador[^3]
Participante registrado en una escuadra.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id` | Int (PK, autoGenerate) | Identificador único |
| `escuadraId` | Int (FK) | Referencia a la escuadra (CASCADE delete) |
| `nombreApellidos` | String | Nombre completo |
| `dni` | String | DNI del tirador |
| `numeroLicencia` | String | Número de licencia federativa |
| `platosRotos` | Int | Número de platos rotos en la tirada |
| `esLocal` | Boolean | Si pertenece a la categoría local |
| `esJunior` | Boolean | Si pertenece a la categoría junior |
| `esSenior` | Boolean | Si pertenece a la categoría senior |
| `esDama` | Boolean | Si pertenece a la categoría dama |
| `precio` | Float | Precio aplicado según su categoría |

### Relaciones Room[^6]
`EscuadraConTiradores` es una data class que utiliza `@Embedded` y `@Relation` para recuperar una escuadra junto con todos sus tiradores en una sola consulta, sin necesidad de hacer JOINs manuales.

***

## ViewModels y lógica de negocio

Cada ViewModel extiende `AndroidViewModel` y expone los datos mediante **LiveData**, permitiendo que la UI reaccione automáticamente a los cambios. Las operaciones de escritura (insert, update, delete) se ejecutan dentro de `viewModelScope.launch {}` usando corrutinas de Kotlin, evitando bloquear el hilo principal.[^7][^8][^9]

### ResultadoViewModel[^4]
Es el ViewModel con mayor lógica de negocio. El método `generarResultados()` recibe la lista de tiradores de una tirada y:

1. Separa los tiradores por categoría (locales, generales, juniors, seniors, damas).
2. Ordena cada grupo de forma descendente por platos rotos.
3. Asigna la posición de clasificación correspondiente a cada tirador en cada categoría a la que pertenece.
4. Calcula el total de puntos como `platosRotos × precio`.
5. Devuelve una lista de objetos `Resultado` lista para mostrar en la UI.

***

## Tecnologías utilizadas

- **Kotlin** — lenguaje principal para toda la lógica de negocio y modelos de datos.[^5][^6][^1][^2][^3]
- **Android Jetpack – Room** — ORM sobre SQLite para persistencia local, con soporte de relaciones entre entidades, consultas con LiveData y operaciones asíncronas.[^1][^2][^3]
- **Android Jetpack – ViewModel & LiveData** — arquitectura reactiva para separar la UI de la lógica de negocio.[^8][^9][^7][^4]
- **Kotlin Coroutines** — gestión de operaciones asíncronas en los repositorios y ViewModels.[^9][^7][^8]
- **Android Studio** — entorno de desarrollo oficial para Android.

***

## Requisitos

- Android 6.0 (API 23) o superior
- Android Studio Hedgehog o posterior (para compilar desde código fuente)

***

## Instalación y uso

1. Clona el repositorio:
   ```bash
   git clone https://github.com/tu-usuario/platoscore.git
   ```
2. Abre el proyecto en Android Studio.
3. Sincroniza las dependencias de Gradle.
4. Ejecuta la app en un emulador o dispositivo físico con Android 6.0+.

***

## Posibles mejoras futuras

- Exportación de resultados a PDF o CSV para facilitar la entrega de clasificaciones oficiales.
- Sincronización con backend remoto para compartir resultados entre dispositivos.
- Implementar inyección de dependencias con **Hilt** para mejorar la testabilidad.
- Añadir tests unitarios para `ResultadoViewModel` y tests de integración con Room.
- Migrar de LiveData a **StateFlow/Flow** para alinearse con las prácticas modernas de Kotlin.
- Soporte de múltiples idiomas (internacionalización).

***

## Autor

Desarrollado por **Óscar** como proyecto Android con foco en modelado de datos relacional, arquitectura MVVM y uso de Room en un contexto real de gestión deportiva.

---

## References

1. [Tirada.kt](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/195884503/9a699467-2e4a-4c71-8f15-782714976aa2/Tirada.kt?AWSAccessKeyId=ASIA2F3EMEYE6RPTF6GM&Signature=0WUrS8UQXf1IUols2qlTZzqAs0U%3D&x-amz-security-token=IQoJb3JpZ2luX2VjEPT%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaCXVzLWVhc3QtMSJGMEQCIFf0O14uLKTxK6iSjYEiV4h%2FtZP%2FRZV8C8NpmUpGy2jpAiAq6ze9RfhbQVhshUQEQXgiAF85pb66xSQBf%2BukmRxv5Sr8BAi9%2F%2F%2F%2F%2F%2F%2F%2F%2F%2F8BEAEaDDY5OTc1MzMwOTcwNSIMK5RRYyxOCzQByZHiKtAEPkYthAc3bMJsrL4Bnf55XLrXPUpwzcHVwta6rn0TADrGHqe8wMqwWNQ%2FYRGC9r9XwZO6%2FKvEgbZDeYhyKgiCcWFqjVKkYC%2FLQCJcTdCDQC1kxR1Bu3DlMZm2J1LzxdliJVZNUfrQXGLysuRkV9PjpJQW5yMc%2FS5Nie%2FUAxXueXDBQmHAR6jAAJrz5Qxh2v7CLCviGnAD%2FMomGSOSeHBpsK%2FsFZ%2BH%2BIhaYc43G25vp63W7ibVFtA2jl0uCiJLzKhF7Jci7GFC8pY96s8xApT0V%2BK4H9RW1kE%2FSUOL0uLf%2BY3Yxsf2OYbMs0ve0ClmU5p0LlaPF%2Fn2Qj0h0HihHBTpBnjFAJ4WetgkQ6uUOKyUIuON1kyXvuZr4Cwq5peZ2m70AuJv217HYSSVrBTc5V0G1SWcmO2G3U%2BFoFsWsInc4Wg40CP3KOWeA6%2F88AIDmhj8ULs9jfxGYoFkOQ4aGxo0WCO0Lf6xsNyoq%2BVryVskoOZOt1ximF5QxtVdYeUJSWgwyIyFJbKU1x3Tmg7F4%2FOgjkkeqoSlx2DtCGmffZe6xKKO%2BQivtxH%2BcSWDT%2FuuSho4OYkblEfxsyP2O5CAzPQs2lZoSmBJG4E9NECudreA5NYrjUgxWTqmtENLcbK11QhZUGY7mHS%2FDkUoi0Mo%2FERe5h93eqOZZJSVNW87sOd1soUMIcU%2FtPrFVAjUqhZQoMItJN7ElJCBCI89hAw4yD3azvSy5RmIiiXJpxkpwGu17Qwwqu2sCNUn%2FSfqep7qfO6ZwS036nWZvOb4uOBD6y9bHjCmj4PPBjqZAeWBE19tRJfUJjvZ5NguvI%2F3VLc5Q%2BohpR1SVkIx2GfsiX%2BDLka2KQlfgGX0mVeR2smW7ph2jxd%2BtmJgq42%2FiBnST8Pn87Y0TLa1OHG7W4n046CIfsT3vWC2ddLbnATfso8l4v1x9UD%2B9mCnuLeRftDJgifrKI2loTKYtmELFxT2Yj0yKW2kTaKgIXakU7YBdqMwTgrsB%2FqO1A%3D%3D&Expires=1776342393) - package oscar.platoscore.models

import androidx.room.Entity
import androidx.room.PrimaryKey

@En...

2. [Escuadra.kt](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/195884503/e85af453-85b1-4557-9cdd-8f2819ecc681/Escuadra.kt?AWSAccessKeyId=ASIA2F3EMEYE6RPTF6GM&Signature=26JzCWdGukEJLON4dzmQecUGsP4%3D&x-amz-security-token=IQoJb3JpZ2luX2VjEPT%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaCXVzLWVhc3QtMSJGMEQCIFf0O14uLKTxK6iSjYEiV4h%2FtZP%2FRZV8C8NpmUpGy2jpAiAq6ze9RfhbQVhshUQEQXgiAF85pb66xSQBf%2BukmRxv5Sr8BAi9%2F%2F%2F%2F%2F%2F%2F%2F%2F%2F8BEAEaDDY5OTc1MzMwOTcwNSIMK5RRYyxOCzQByZHiKtAEPkYthAc3bMJsrL4Bnf55XLrXPUpwzcHVwta6rn0TADrGHqe8wMqwWNQ%2FYRGC9r9XwZO6%2FKvEgbZDeYhyKgiCcWFqjVKkYC%2FLQCJcTdCDQC1kxR1Bu3DlMZm2J1LzxdliJVZNUfrQXGLysuRkV9PjpJQW5yMc%2FS5Nie%2FUAxXueXDBQmHAR6jAAJrz5Qxh2v7CLCviGnAD%2FMomGSOSeHBpsK%2FsFZ%2BH%2BIhaYc43G25vp63W7ibVFtA2jl0uCiJLzKhF7Jci7GFC8pY96s8xApT0V%2BK4H9RW1kE%2FSUOL0uLf%2BY3Yxsf2OYbMs0ve0ClmU5p0LlaPF%2Fn2Qj0h0HihHBTpBnjFAJ4WetgkQ6uUOKyUIuON1kyXvuZr4Cwq5peZ2m70AuJv217HYSSVrBTc5V0G1SWcmO2G3U%2BFoFsWsInc4Wg40CP3KOWeA6%2F88AIDmhj8ULs9jfxGYoFkOQ4aGxo0WCO0Lf6xsNyoq%2BVryVskoOZOt1ximF5QxtVdYeUJSWgwyIyFJbKU1x3Tmg7F4%2FOgjkkeqoSlx2DtCGmffZe6xKKO%2BQivtxH%2BcSWDT%2FuuSho4OYkblEfxsyP2O5CAzPQs2lZoSmBJG4E9NECudreA5NYrjUgxWTqmtENLcbK11QhZUGY7mHS%2FDkUoi0Mo%2FERe5h93eqOZZJSVNW87sOd1soUMIcU%2FtPrFVAjUqhZQoMItJN7ElJCBCI89hAw4yD3azvSy5RmIiiXJpxkpwGu17Qwwqu2sCNUn%2FSfqep7qfO6ZwS036nWZvOb4uOBD6y9bHjCmj4PPBjqZAeWBE19tRJfUJjvZ5NguvI%2F3VLc5Q%2BohpR1SVkIx2GfsiX%2BDLka2KQlfgGX0mVeR2smW7ph2jxd%2BtmJgq42%2FiBnST8Pn87Y0TLa1OHG7W4n046CIfsT3vWC2ddLbnATfso8l4v1x9UD%2B9mCnuLeRftDJgifrKI2loTKYtmELFxT2Yj0yKW2kTaKgIXakU7YBdqMwTgrsB%2FqO1A%3D%3D&Expires=1776342393) - package oscar.platoscore.models

import androidx.room.Entity
import androidx.room.ForeignKey
impo...

3. [Tirador.kt](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/195884503/b1ec116c-c57b-4ba7-8a60-20157b50014c/Tirador.kt?AWSAccessKeyId=ASIA2F3EMEYE6RPTF6GM&Signature=rVGwOoSiLDWswipxFGHCsuygO2o%3D&x-amz-security-token=IQoJb3JpZ2luX2VjEPT%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaCXVzLWVhc3QtMSJGMEQCIFf0O14uLKTxK6iSjYEiV4h%2FtZP%2FRZV8C8NpmUpGy2jpAiAq6ze9RfhbQVhshUQEQXgiAF85pb66xSQBf%2BukmRxv5Sr8BAi9%2F%2F%2F%2F%2F%2F%2F%2F%2F%2F8BEAEaDDY5OTc1MzMwOTcwNSIMK5RRYyxOCzQByZHiKtAEPkYthAc3bMJsrL4Bnf55XLrXPUpwzcHVwta6rn0TADrGHqe8wMqwWNQ%2FYRGC9r9XwZO6%2FKvEgbZDeYhyKgiCcWFqjVKkYC%2FLQCJcTdCDQC1kxR1Bu3DlMZm2J1LzxdliJVZNUfrQXGLysuRkV9PjpJQW5yMc%2FS5Nie%2FUAxXueXDBQmHAR6jAAJrz5Qxh2v7CLCviGnAD%2FMomGSOSeHBpsK%2FsFZ%2BH%2BIhaYc43G25vp63W7ibVFtA2jl0uCiJLzKhF7Jci7GFC8pY96s8xApT0V%2BK4H9RW1kE%2FSUOL0uLf%2BY3Yxsf2OYbMs0ve0ClmU5p0LlaPF%2Fn2Qj0h0HihHBTpBnjFAJ4WetgkQ6uUOKyUIuON1kyXvuZr4Cwq5peZ2m70AuJv217HYSSVrBTc5V0G1SWcmO2G3U%2BFoFsWsInc4Wg40CP3KOWeA6%2F88AIDmhj8ULs9jfxGYoFkOQ4aGxo0WCO0Lf6xsNyoq%2BVryVskoOZOt1ximF5QxtVdYeUJSWgwyIyFJbKU1x3Tmg7F4%2FOgjkkeqoSlx2DtCGmffZe6xKKO%2BQivtxH%2BcSWDT%2FuuSho4OYkblEfxsyP2O5CAzPQs2lZoSmBJG4E9NECudreA5NYrjUgxWTqmtENLcbK11QhZUGY7mHS%2FDkUoi0Mo%2FERe5h93eqOZZJSVNW87sOd1soUMIcU%2FtPrFVAjUqhZQoMItJN7ElJCBCI89hAw4yD3azvSy5RmIiiXJpxkpwGu17Qwwqu2sCNUn%2FSfqep7qfO6ZwS036nWZvOb4uOBD6y9bHjCmj4PPBjqZAeWBE19tRJfUJjvZ5NguvI%2F3VLc5Q%2BohpR1SVkIx2GfsiX%2BDLka2KQlfgGX0mVeR2smW7ph2jxd%2BtmJgq42%2FiBnST8Pn87Y0TLa1OHG7W4n046CIfsT3vWC2ddLbnATfso8l4v1x9UD%2B9mCnuLeRftDJgifrKI2loTKYtmELFxT2Yj0yKW2kTaKgIXakU7YBdqMwTgrsB%2FqO1A%3D%3D&Expires=1776342393) - package oscar.platoscore.models

import androidx.room.Entity
import androidx.room.ForeignKey
impo...

4. [ResultadoViewModel.kt](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/195884503/68707f72-3aab-4a5f-acc2-544cf0ba0f65/ResultadoViewModel.kt?AWSAccessKeyId=ASIA2F3EMEYE6RPTF6GM&Signature=HWuY6krGRzJj0l3gYxprRopJrHA%3D&x-amz-security-token=IQoJb3JpZ2luX2VjEPT%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaCXVzLWVhc3QtMSJGMEQCIFf0O14uLKTxK6iSjYEiV4h%2FtZP%2FRZV8C8NpmUpGy2jpAiAq6ze9RfhbQVhshUQEQXgiAF85pb66xSQBf%2BukmRxv5Sr8BAi9%2F%2F%2F%2F%2F%2F%2F%2F%2F%2F8BEAEaDDY5OTc1MzMwOTcwNSIMK5RRYyxOCzQByZHiKtAEPkYthAc3bMJsrL4Bnf55XLrXPUpwzcHVwta6rn0TADrGHqe8wMqwWNQ%2FYRGC9r9XwZO6%2FKvEgbZDeYhyKgiCcWFqjVKkYC%2FLQCJcTdCDQC1kxR1Bu3DlMZm2J1LzxdliJVZNUfrQXGLysuRkV9PjpJQW5yMc%2FS5Nie%2FUAxXueXDBQmHAR6jAAJrz5Qxh2v7CLCviGnAD%2FMomGSOSeHBpsK%2FsFZ%2BH%2BIhaYc43G25vp63W7ibVFtA2jl0uCiJLzKhF7Jci7GFC8pY96s8xApT0V%2BK4H9RW1kE%2FSUOL0uLf%2BY3Yxsf2OYbMs0ve0ClmU5p0LlaPF%2Fn2Qj0h0HihHBTpBnjFAJ4WetgkQ6uUOKyUIuON1kyXvuZr4Cwq5peZ2m70AuJv217HYSSVrBTc5V0G1SWcmO2G3U%2BFoFsWsInc4Wg40CP3KOWeA6%2F88AIDmhj8ULs9jfxGYoFkOQ4aGxo0WCO0Lf6xsNyoq%2BVryVskoOZOt1ximF5QxtVdYeUJSWgwyIyFJbKU1x3Tmg7F4%2FOgjkkeqoSlx2DtCGmffZe6xKKO%2BQivtxH%2BcSWDT%2FuuSho4OYkblEfxsyP2O5CAzPQs2lZoSmBJG4E9NECudreA5NYrjUgxWTqmtENLcbK11QhZUGY7mHS%2FDkUoi0Mo%2FERe5h93eqOZZJSVNW87sOd1soUMIcU%2FtPrFVAjUqhZQoMItJN7ElJCBCI89hAw4yD3azvSy5RmIiiXJpxkpwGu17Qwwqu2sCNUn%2FSfqep7qfO6ZwS036nWZvOb4uOBD6y9bHjCmj4PPBjqZAeWBE19tRJfUJjvZ5NguvI%2F3VLc5Q%2BohpR1SVkIx2GfsiX%2BDLka2KQlfgGX0mVeR2smW7ph2jxd%2BtmJgq42%2FiBnST8Pn87Y0TLa1OHG7W4n046CIfsT3vWC2ddLbnATfso8l4v1x9UD%2B9mCnuLeRftDJgifrKI2loTKYtmELFxT2Yj0yKW2kTaKgIXakU7YBdqMwTgrsB%2FqO1A%3D%3D&Expires=1776342393) - package oscar.platoscore.viewmodels

import androidx.lifecycle.ViewModel
import oscar.platoscore.m...

5. [Resultado.kt](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/195884503/f5395404-f5ac-476d-986b-2a111274e274/Resultado.kt?AWSAccessKeyId=ASIA2F3EMEYE6RPTF6GM&Signature=FpAJ%2FDfntqNc4ES56g4QmfNpvUo%3D&x-amz-security-token=IQoJb3JpZ2luX2VjEPT%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaCXVzLWVhc3QtMSJGMEQCIFf0O14uLKTxK6iSjYEiV4h%2FtZP%2FRZV8C8NpmUpGy2jpAiAq6ze9RfhbQVhshUQEQXgiAF85pb66xSQBf%2BukmRxv5Sr8BAi9%2F%2F%2F%2F%2F%2F%2F%2F%2F%2F8BEAEaDDY5OTc1MzMwOTcwNSIMK5RRYyxOCzQByZHiKtAEPkYthAc3bMJsrL4Bnf55XLrXPUpwzcHVwta6rn0TADrGHqe8wMqwWNQ%2FYRGC9r9XwZO6%2FKvEgbZDeYhyKgiCcWFqjVKkYC%2FLQCJcTdCDQC1kxR1Bu3DlMZm2J1LzxdliJVZNUfrQXGLysuRkV9PjpJQW5yMc%2FS5Nie%2FUAxXueXDBQmHAR6jAAJrz5Qxh2v7CLCviGnAD%2FMomGSOSeHBpsK%2FsFZ%2BH%2BIhaYc43G25vp63W7ibVFtA2jl0uCiJLzKhF7Jci7GFC8pY96s8xApT0V%2BK4H9RW1kE%2FSUOL0uLf%2BY3Yxsf2OYbMs0ve0ClmU5p0LlaPF%2Fn2Qj0h0HihHBTpBnjFAJ4WetgkQ6uUOKyUIuON1kyXvuZr4Cwq5peZ2m70AuJv217HYSSVrBTc5V0G1SWcmO2G3U%2BFoFsWsInc4Wg40CP3KOWeA6%2F88AIDmhj8ULs9jfxGYoFkOQ4aGxo0WCO0Lf6xsNyoq%2BVryVskoOZOt1ximF5QxtVdYeUJSWgwyIyFJbKU1x3Tmg7F4%2FOgjkkeqoSlx2DtCGmffZe6xKKO%2BQivtxH%2BcSWDT%2FuuSho4OYkblEfxsyP2O5CAzPQs2lZoSmBJG4E9NECudreA5NYrjUgxWTqmtENLcbK11QhZUGY7mHS%2FDkUoi0Mo%2FERe5h93eqOZZJSVNW87sOd1soUMIcU%2FtPrFVAjUqhZQoMItJN7ElJCBCI89hAw4yD3azvSy5RmIiiXJpxkpwGu17Qwwqu2sCNUn%2FSfqep7qfO6ZwS036nWZvOb4uOBD6y9bHjCmj4PPBjqZAeWBE19tRJfUJjvZ5NguvI%2F3VLc5Q%2BohpR1SVkIx2GfsiX%2BDLka2KQlfgGX0mVeR2smW7ph2jxd%2BtmJgq42%2FiBnST8Pn87Y0TLa1OHG7W4n046CIfsT3vWC2ddLbnATfso8l4v1x9UD%2B9mCnuLeRftDJgifrKI2loTKYtmELFxT2Yj0yKW2kTaKgIXakU7YBdqMwTgrsB%2FqO1A%3D%3D&Expires=1776342393) - package oscar.platoscore.models

data class Resultado(
val tirador: Tirador,
val clasificacionLoc...

6. [EscuadraConTiradores.kt](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/195884503/e106bbf7-3d32-4b44-bf2d-9c459e7c22ca/EscuadraConTiradores.kt?AWSAccessKeyId=ASIA2F3EMEYE6RPTF6GM&Signature=H7XL8UcYT0KO4O%2BVY0MjK0MKTCQ%3D&x-amz-security-token=IQoJb3JpZ2luX2VjEPT%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaCXVzLWVhc3QtMSJGMEQCIFf0O14uLKTxK6iSjYEiV4h%2FtZP%2FRZV8C8NpmUpGy2jpAiAq6ze9RfhbQVhshUQEQXgiAF85pb66xSQBf%2BukmRxv5Sr8BAi9%2F%2F%2F%2F%2F%2F%2F%2F%2F%2F8BEAEaDDY5OTc1MzMwOTcwNSIMK5RRYyxOCzQByZHiKtAEPkYthAc3bMJsrL4Bnf55XLrXPUpwzcHVwta6rn0TADrGHqe8wMqwWNQ%2FYRGC9r9XwZO6%2FKvEgbZDeYhyKgiCcWFqjVKkYC%2FLQCJcTdCDQC1kxR1Bu3DlMZm2J1LzxdliJVZNUfrQXGLysuRkV9PjpJQW5yMc%2FS5Nie%2FUAxXueXDBQmHAR6jAAJrz5Qxh2v7CLCviGnAD%2FMomGSOSeHBpsK%2FsFZ%2BH%2BIhaYc43G25vp63W7ibVFtA2jl0uCiJLzKhF7Jci7GFC8pY96s8xApT0V%2BK4H9RW1kE%2FSUOL0uLf%2BY3Yxsf2OYbMs0ve0ClmU5p0LlaPF%2Fn2Qj0h0HihHBTpBnjFAJ4WetgkQ6uUOKyUIuON1kyXvuZr4Cwq5peZ2m70AuJv217HYSSVrBTc5V0G1SWcmO2G3U%2BFoFsWsInc4Wg40CP3KOWeA6%2F88AIDmhj8ULs9jfxGYoFkOQ4aGxo0WCO0Lf6xsNyoq%2BVryVskoOZOt1ximF5QxtVdYeUJSWgwyIyFJbKU1x3Tmg7F4%2FOgjkkeqoSlx2DtCGmffZe6xKKO%2BQivtxH%2BcSWDT%2FuuSho4OYkblEfxsyP2O5CAzPQs2lZoSmBJG4E9NECudreA5NYrjUgxWTqmtENLcbK11QhZUGY7mHS%2FDkUoi0Mo%2FERe5h93eqOZZJSVNW87sOd1soUMIcU%2FtPrFVAjUqhZQoMItJN7ElJCBCI89hAw4yD3azvSy5RmIiiXJpxkpwGu17Qwwqu2sCNUn%2FSfqep7qfO6ZwS036nWZvOb4uOBD6y9bHjCmj4PPBjqZAeWBE19tRJfUJjvZ5NguvI%2F3VLc5Q%2BohpR1SVkIx2GfsiX%2BDLka2KQlfgGX0mVeR2smW7ph2jxd%2BtmJgq42%2FiBnST8Pn87Y0TLa1OHG7W4n046CIfsT3vWC2ddLbnATfso8l4v1x9UD%2B9mCnuLeRftDJgifrKI2loTKYtmELFxT2Yj0yKW2kTaKgIXakU7YBdqMwTgrsB%2FqO1A%3D%3D&Expires=1776342393) - package oscar.platoscore.models

import androidx.room.Embedded
import androidx.room.Relation

dat...

7. [TiradorViewModel.kt](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/195884503/28e8d73d-2a0e-4964-af82-a6f72d92e17d/TiradorViewModel.kt?AWSAccessKeyId=ASIA2F3EMEYE6RPTF6GM&Signature=s0agJjc7lqVf6FHCCvnjxzlN8Vg%3D&x-amz-security-token=IQoJb3JpZ2luX2VjEPT%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaCXVzLWVhc3QtMSJGMEQCIFf0O14uLKTxK6iSjYEiV4h%2FtZP%2FRZV8C8NpmUpGy2jpAiAq6ze9RfhbQVhshUQEQXgiAF85pb66xSQBf%2BukmRxv5Sr8BAi9%2F%2F%2F%2F%2F%2F%2F%2F%2F%2F8BEAEaDDY5OTc1MzMwOTcwNSIMK5RRYyxOCzQByZHiKtAEPkYthAc3bMJsrL4Bnf55XLrXPUpwzcHVwta6rn0TADrGHqe8wMqwWNQ%2FYRGC9r9XwZO6%2FKvEgbZDeYhyKgiCcWFqjVKkYC%2FLQCJcTdCDQC1kxR1Bu3DlMZm2J1LzxdliJVZNUfrQXGLysuRkV9PjpJQW5yMc%2FS5Nie%2FUAxXueXDBQmHAR6jAAJrz5Qxh2v7CLCviGnAD%2FMomGSOSeHBpsK%2FsFZ%2BH%2BIhaYc43G25vp63W7ibVFtA2jl0uCiJLzKhF7Jci7GFC8pY96s8xApT0V%2BK4H9RW1kE%2FSUOL0uLf%2BY3Yxsf2OYbMs0ve0ClmU5p0LlaPF%2Fn2Qj0h0HihHBTpBnjFAJ4WetgkQ6uUOKyUIuON1kyXvuZr4Cwq5peZ2m70AuJv217HYSSVrBTc5V0G1SWcmO2G3U%2BFoFsWsInc4Wg40CP3KOWeA6%2F88AIDmhj8ULs9jfxGYoFkOQ4aGxo0WCO0Lf6xsNyoq%2BVryVskoOZOt1ximF5QxtVdYeUJSWgwyIyFJbKU1x3Tmg7F4%2FOgjkkeqoSlx2DtCGmffZe6xKKO%2BQivtxH%2BcSWDT%2FuuSho4OYkblEfxsyP2O5CAzPQs2lZoSmBJG4E9NECudreA5NYrjUgxWTqmtENLcbK11QhZUGY7mHS%2FDkUoi0Mo%2FERe5h93eqOZZJSVNW87sOd1soUMIcU%2FtPrFVAjUqhZQoMItJN7ElJCBCI89hAw4yD3azvSy5RmIiiXJpxkpwGu17Qwwqu2sCNUn%2FSfqep7qfO6ZwS036nWZvOb4uOBD6y9bHjCmj4PPBjqZAeWBE19tRJfUJjvZ5NguvI%2F3VLc5Q%2BohpR1SVkIx2GfsiX%2BDLka2KQlfgGX0mVeR2smW7ph2jxd%2BtmJgq42%2FiBnST8Pn87Y0TLa1OHG7W4n046CIfsT3vWC2ddLbnATfso8l4v1x9UD%2B9mCnuLeRftDJgifrKI2loTKYtmELFxT2Yj0yKW2kTaKgIXakU7YBdqMwTgrsB%2FqO1A%3D%3D&Expires=1776342393) - package oscar.platoscore.viewmodels

import android.app.Application
import androidx.lifecycle.Andr...

8. [EscuadraViewModel.kt](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/195884503/735025b6-8ae5-462b-89cd-2be41c6ef0eb/EscuadraViewModel.kt?AWSAccessKeyId=ASIA2F3EMEYE6RPTF6GM&Signature=b6Ki5otG%2B%2Bb8VZSmfCG6kNGMVn0%3D&x-amz-security-token=IQoJb3JpZ2luX2VjEPT%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaCXVzLWVhc3QtMSJGMEQCIFf0O14uLKTxK6iSjYEiV4h%2FtZP%2FRZV8C8NpmUpGy2jpAiAq6ze9RfhbQVhshUQEQXgiAF85pb66xSQBf%2BukmRxv5Sr8BAi9%2F%2F%2F%2F%2F%2F%2F%2F%2F%2F8BEAEaDDY5OTc1MzMwOTcwNSIMK5RRYyxOCzQByZHiKtAEPkYthAc3bMJsrL4Bnf55XLrXPUpwzcHVwta6rn0TADrGHqe8wMqwWNQ%2FYRGC9r9XwZO6%2FKvEgbZDeYhyKgiCcWFqjVKkYC%2FLQCJcTdCDQC1kxR1Bu3DlMZm2J1LzxdliJVZNUfrQXGLysuRkV9PjpJQW5yMc%2FS5Nie%2FUAxXueXDBQmHAR6jAAJrz5Qxh2v7CLCviGnAD%2FMomGSOSeHBpsK%2FsFZ%2BH%2BIhaYc43G25vp63W7ibVFtA2jl0uCiJLzKhF7Jci7GFC8pY96s8xApT0V%2BK4H9RW1kE%2FSUOL0uLf%2BY3Yxsf2OYbMs0ve0ClmU5p0LlaPF%2Fn2Qj0h0HihHBTpBnjFAJ4WetgkQ6uUOKyUIuON1kyXvuZr4Cwq5peZ2m70AuJv217HYSSVrBTc5V0G1SWcmO2G3U%2BFoFsWsInc4Wg40CP3KOWeA6%2F88AIDmhj8ULs9jfxGYoFkOQ4aGxo0WCO0Lf6xsNyoq%2BVryVskoOZOt1ximF5QxtVdYeUJSWgwyIyFJbKU1x3Tmg7F4%2FOgjkkeqoSlx2DtCGmffZe6xKKO%2BQivtxH%2BcSWDT%2FuuSho4OYkblEfxsyP2O5CAzPQs2lZoSmBJG4E9NECudreA5NYrjUgxWTqmtENLcbK11QhZUGY7mHS%2FDkUoi0Mo%2FERe5h93eqOZZJSVNW87sOd1soUMIcU%2FtPrFVAjUqhZQoMItJN7ElJCBCI89hAw4yD3azvSy5RmIiiXJpxkpwGu17Qwwqu2sCNUn%2FSfqep7qfO6ZwS036nWZvOb4uOBD6y9bHjCmj4PPBjqZAeWBE19tRJfUJjvZ5NguvI%2F3VLc5Q%2BohpR1SVkIx2GfsiX%2BDLka2KQlfgGX0mVeR2smW7ph2jxd%2BtmJgq42%2FiBnST8Pn87Y0TLa1OHG7W4n046CIfsT3vWC2ddLbnATfso8l4v1x9UD%2B9mCnuLeRftDJgifrKI2loTKYtmELFxT2Yj0yKW2kTaKgIXakU7YBdqMwTgrsB%2FqO1A%3D%3D&Expires=1776342393) - package oscar.platoscore.viewmodels

import android.app.Application
import androidx.lifecycle.Andr...

9. [TiradaViewModel.kt](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/195884503/66a6dd26-e63b-4b87-b98a-15e0229eafb7/TiradaViewModel.kt?AWSAccessKeyId=ASIA2F3EMEYE6RPTF6GM&Signature=px30JUawNRfvTXKOaURxo3Ty%2Fjc%3D&x-amz-security-token=IQoJb3JpZ2luX2VjEPT%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaCXVzLWVhc3QtMSJGMEQCIFf0O14uLKTxK6iSjYEiV4h%2FtZP%2FRZV8C8NpmUpGy2jpAiAq6ze9RfhbQVhshUQEQXgiAF85pb66xSQBf%2BukmRxv5Sr8BAi9%2F%2F%2F%2F%2F%2F%2F%2F%2F%2F8BEAEaDDY5OTc1MzMwOTcwNSIMK5RRYyxOCzQByZHiKtAEPkYthAc3bMJsrL4Bnf55XLrXPUpwzcHVwta6rn0TADrGHqe8wMqwWNQ%2FYRGC9r9XwZO6%2FKvEgbZDeYhyKgiCcWFqjVKkYC%2FLQCJcTdCDQC1kxR1Bu3DlMZm2J1LzxdliJVZNUfrQXGLysuRkV9PjpJQW5yMc%2FS5Nie%2FUAxXueXDBQmHAR6jAAJrz5Qxh2v7CLCviGnAD%2FMomGSOSeHBpsK%2FsFZ%2BH%2BIhaYc43G25vp63W7ibVFtA2jl0uCiJLzKhF7Jci7GFC8pY96s8xApT0V%2BK4H9RW1kE%2FSUOL0uLf%2BY3Yxsf2OYbMs0ve0ClmU5p0LlaPF%2Fn2Qj0h0HihHBTpBnjFAJ4WetgkQ6uUOKyUIuON1kyXvuZr4Cwq5peZ2m70AuJv217HYSSVrBTc5V0G1SWcmO2G3U%2BFoFsWsInc4Wg40CP3KOWeA6%2F88AIDmhj8ULs9jfxGYoFkOQ4aGxo0WCO0Lf6xsNyoq%2BVryVskoOZOt1ximF5QxtVdYeUJSWgwyIyFJbKU1x3Tmg7F4%2FOgjkkeqoSlx2DtCGmffZe6xKKO%2BQivtxH%2BcSWDT%2FuuSho4OYkblEfxsyP2O5CAzPQs2lZoSmBJG4E9NECudreA5NYrjUgxWTqmtENLcbK11QhZUGY7mHS%2FDkUoi0Mo%2FERe5h93eqOZZJSVNW87sOd1soUMIcU%2FtPrFVAjUqhZQoMItJN7ElJCBCI89hAw4yD3azvSy5RmIiiXJpxkpwGu17Qwwqu2sCNUn%2FSfqep7qfO6ZwS036nWZvOb4uOBD6y9bHjCmj4PPBjqZAeWBE19tRJfUJjvZ5NguvI%2F3VLc5Q%2BohpR1SVkIx2GfsiX%2BDLka2KQlfgGX0mVeR2smW7ph2jxd%2BtmJgq42%2FiBnST8Pn87Y0TLa1OHG7W4n046CIfsT3vWC2ddLbnATfso8l4v1x9UD%2B9mCnuLeRftDJgifrKI2loTKYtmELFxT2Yj0yKW2kTaKgIXakU7YBdqMwTgrsB%2FqO1A%3D%3D&Expires=1776342393) - package oscar.platoscore.viewmodels

import android.app.Application
import androidx.lifecycle.Andr...

