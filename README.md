# PlatoScore 🎯

**Aplicación Android para la gestión de competiciones de tiro al plato.** PlatoScore permite registrar tiradas, organizar tiradores en escuadras y calcular automáticamente clasificaciones por categoría, todo ello con persistencia local mediante Room y una arquitectura MVVM limpia en Kotlin.

***

## Descripción

PlatoScore nace para digitalizar y simplificar la gestión de competiciones de tiro al plato. Antes de la app, organizar una tirada implicaba papeles, cálculos manuales y errores en las clasificaciones. Con PlatoScore, el responsable de la competición puede registrar todos los datos desde su móvil Android, asignar tiradores a escuadras, registrar platos rotos y obtener automáticamente el ranking por categoría (local, general, junior, senior y dama).

***

## Características principales

- 📋 **Gestión de tiradas**: crea, edita y elimina competiciones con nombre, fecha y precios diferenciados por categoría (local, general, junior, senior, dama).
- 👥 **Gestión de escuadras**: organiza los tiradores en escuadras dentro de cada tirada, con relación 1-N gestionada por Room.
- 🎯 **Gestión de tiradores**: registra nombre, apellidos, DNI, número de licencia, platos rotos y categorías de cada participante.
- 🏆 **Clasificaciones automáticas**: el sistema calcula en tiempo real la posición de cada tirador en su categoría (local/general y subcategorías), ordenando por platos rotos de forma descendente.
- 💰 **Cálculo de puntuación económica**: el total de puntos de cada tirador se calcula multiplicando sus platos rotos por el precio de su categoría.
- 💾 **Persistencia local**: todos los datos se almacenan en una base de datos SQLite local mediante Room, sin necesidad de conexión a internet.

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
| **Modelo** | `Tirada`, `Escuadra`, `Tirador`, `Resultado`, `EscuadraConTiradores` | Entidades de dominio y relaciones Room |
| **ViewModel** | `TiradaViewModel`, `EscuadraViewModel`, `TiradorViewModel`, `ResultadoViewModel` | Lógica de negocio, operaciones CRUD y generación de resultados |
| **Repositorio** | `TiradaRepository`, `EscuadraRepository`, `TiradorRepository` | Abstracción del acceso a datos, operaciones suspendidas con corrutinas |
| **Base de datos** | `PlatoScoreDatabase` (Room) | Configuración de la base de datos SQLite y acceso a los DAOs |

***

## Modelo de datos

### Tirada
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

### Escuadra
Agrupación de tiradores dentro de una tirada.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id` | Int (PK, autoGenerate) | Identificador único |
| `tiradaId` | Int (FK) | Referencia a la tirada (CASCADE delete) |
| `numeroEscuadra` | Int | Número de escuadra dentro de la tirada |

### Tirador
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

### Relaciones Room
`EscuadraConTiradores` es una data class que utiliza `@Embedded` y `@Relation` para recuperar una escuadra junto con todos sus tiradores en una sola consulta, sin necesidad de hacer JOINs manuales.

***

## ViewModels y lógica de negocio

Cada ViewModel extiende `AndroidViewModel` y expone los datos mediante **LiveData**, permitiendo que la UI reaccione automáticamente a los cambios. Las operaciones de escritura (insert, update, delete) se ejecutan dentro de `viewModelScope.launch {}` usando corrutinas de Kotlin, evitando bloquear el hilo principal.

### ResultadoViewModel
Es el ViewModel con mayor lógica de negocio. El método `generarResultados()` recibe la lista de tiradores de una tirada y:

1. Separa los tiradores por categoría (locales, generales, juniors, seniors, damas).
2. Ordena cada grupo de forma descendente por platos rotos.
3. Asigna la posición de clasificación correspondiente a cada tirador en cada categoría a la que pertenece.
4. Calcula el total de puntos como `platosRotos × precio`.
5. Devuelve una lista de objetos `Resultado` lista para mostrar en la UI.

***

## Tecnologías utilizadas

- **Kotlin** — lenguaje principal para toda la lógica de negocio y modelos de datos.
- **Android Jetpack – Room** — ORM sobre SQLite para persistencia local, con soporte de relaciones entre entidades, consultas con LiveData y operaciones asíncronas.
- **Android Jetpack – ViewModel & LiveData** — arquitectura reactiva para separar la UI de la lógica de negocio.
- **Kotlin Coroutines** — gestión de operaciones asíncronas en los repositorios y ViewModels.
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
