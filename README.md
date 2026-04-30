# PlatoScore 🎯

**Aplicación Android para la gestión de competiciones de tiro al plato.** PlatoScore permite registrar tiradas, organizar tiradores en escuadras y calcular automáticamente clasificaciones por categoría, todo ello con persistencia local mediante Room y una arquitectura MVVM limpia en Kotlin.

***

## Descripción

PlatoScore digitaliza la gestión de competiciones de tiro al plato. Con PlatoScore, el responsable puede registrar todos los datos desde su móvil Android, asignar tiradores a escuadras y obtener automáticamente el ranking por categoría (local, general, junior, senior y dama).

***

## Capturas de pantalla

| Lista de tiradas | Datos de la tirada | Escuadras |
|:----------------:|:-----------------:|:---------:|
| ![Lista de tiradas](screenshots/PantallaPrincipalPlatoScore.jpg) | ![Datos de la tirada](screenshots/InfoTiradaPlatoScore_1.jpg) | ![Escuadras](screenshots/InfoTiradaPlatoScore_2.jpg) |

| Tiradores de escuadra | Clasificaciones |
|:--------------------:|:---------------:|
| ![Tiradores de escuadra](screenshots/InfoEscuadraPlatoScore.jpg) | ![Clasificaciones](screenshots/ResultadosTiradaPlatoScore.jpg) |

***

## Características principales

- 📋 **Gestión de tiradas**: crea, edita y elimina competiciones con nombre, fecha y precios por categoría (local, general, junior, senior, dama).
- 👥 **Gestión de escuadras**: organiza los tiradores en escuadras, con relación 1-N gestionada por Room.
- 🎯 **Gestión de tiradores**: registra nombre, apellidos, DNI, número de licencia, platos rotos y categorías de cada participante.
- 🏆 **Clasificaciones automáticas**: el sistema calcula en tiempo real la posición de cada tirador en su categoría, ordenando por platos rotos de forma descendente.
- 💾 **Persistencia local**: todos los datos se almacenan en una base de datos SQLite local mediante Room, sin necesidad de conexión a internet.

***

## Arquitectura MVVM

El proyecto sigue el patrón **MVVM (Model-View-ViewModel)** recomendado por Google, con una capa de repositorios que desacopla los ViewModels del acceso directo a la base de datos.

```
UI (Activities)
    │
    ▼
ViewModels ◄─ LiveData ─► UI reactiva
    │
    ▼
Repositorios
    │
    ▼
Room DAOs ─► SQLite
```

| Capa | Clases | Responsabilidad |
|------|--------|-----------------|
| **Modelo** | `Tirada`, `Escuadra`, `Tirador`, `Resultado`, `EscuadraConTiradores` | Entidades de dominio y relaciones Room |
| **ViewModel** | `TiradaViewModel`, `EscuadraViewModel`, `TiradorViewModel`, `ResultadoViewModel` | Lógica de negocio, operaciones CRUD y generación de resultados |
| **Repositorio** | `TiradaRepository`, `EscuadraRepository`, `TiradorRepository` | Abstracción del acceso a datos, operaciones suspendidas con corrutinas |
| **Base de datos** | `PlatoScoreDatabase` (Room) | Configuración de la base de datos SQLite y acceso a los DAOs |

***

## Modelo de datos

### Tirada
| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id` | Int (PK) | Identificador único |
| `nombre` | String | Nombre de la tirada |
| `fecha` | String | Fecha de la competición |
| `precioLocal` / `precioGeneral` / `precioJunior` / `precioSenior` / `precioDama` | Float | Precio por categoría |

### Tirador
| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id` | Int (PK) | Identificador único |
| `escuadraId` | Int (FK) | Referencia a la escuadra (CASCADE delete) |
| `nombreApellidos` | String | Nombre completo |
| `platosRotos` | Int | Número de platos rotos en la tirada |
| `esLocal` / `esJunior` / `esSenior` / `esDama` | Boolean | Categorías del tirador |
| `precio` | Float | Precio aplicado según su categoría |

### `EscuadraConTiradores`
Data class que usa `@Embedded` y `@Relation` para recuperar una escuadra junto con todos sus tiradores en una sola consulta Room.

***

## Tecnologías utilizadas

- **Kotlin** — lenguaje principal.
- **Android Jetpack – Room** — ORM sobre SQLite con soporte de relaciones y LiveData.
- **Android Jetpack – ViewModel & LiveData** — arquitectura reactiva.
- **Kotlin Coroutines** — operaciones asíncronas en repositorios y ViewModels.
- **Android Studio** — entorno de desarrollo oficial.

***

## Instalación

1. Clona el repositorio:
   ```bash
   git clone https://github.com/Llarry793/PlatoScore.git
   ```
2. Abre el proyecto en Android Studio.
3. Sincroniza las dependencias de Gradle.
4. Ejecuta la app en un emulador o dispositivo físico con Android 6.0+.

***

## Posibles mejoras futuras

- Exportación de resultados a PDF o CSV.
- Sincronización con backend remoto.
- Inyección de dependencias con **Hilt**.
- Tests unitarios para `ResultadoViewModel` y tests de integración con Room.
- Migración de LiveData a **StateFlow/Flow**.

***

## Autor

Desarrollado por **Óscar** (UV) — arquitectura MVVM + Room en un contexto real de gestión deportiva.
