# Platziflix - Proyecto Multi-plataforma

## Arquitectura del Sistema

Platziflix es una plataforma de cursos online con arquitectura multi-plataforma:

```
                          ┌─────────────────────────────────┐
                          │         BACKEND (FastAPI)        │
                          │         localhost:8000           │
                          │  CourseService + Models + Alembic│
                          │  ┌──────────────────────────┐   │
                          │  │  PostgreSQL 15 (Docker)   │   │
                          │  │  platziflix_db            │   │
                          │  └──────────────────────────┘   │
                          └──────────────┬──────────────────┘
                                         │ REST API (JSON)
                    ┌────────────────────┼───────────────────┐
                    │                    │                   │
          ┌─────────▼──────┐   ┌─────────▼──────┐  ┌────────▼───────┐
          │   FRONTEND      │   │    ANDROID      │  │      iOS       │
          │  Next.js 15     │   │    Kotlin       │  │     Swift      │
          │  localhost:3000 │   │  Jetpack Compose│  │    SwiftUI     │
          └─────────────────┘   └─────────────────┘  └────────────────┘
```

---

## Stack Tecnológico

### Backend
- **Framework**: FastAPI 0.104.0
- **Base de datos**: PostgreSQL 15
- **ORM**: SQLAlchemy 2.0
- **Validación**: Pydantic 2.0
- **Migraciones**: Alembic 1.13.0
- **Container**: Docker + Docker Compose
- **Dependencias**: UV (Python 3.11+)
- **Puerto**: 8000

### Frontend
- **Framework**: Next.js 15.3.3 (App Router)
- **React**: 19.0.0
- **Lenguaje**: TypeScript 5 (strict)
- **Estilos**: SCSS + CSS Modules
- **Testing**: Vitest 3.2.3 + React Testing Library 16
- **Build**: Turbopack
- **Fonts**: Geist Sans & Geist Mono
- **Puerto**: 3000

### Mobile Android
- **Lenguaje**: Kotlin
- **UI**: Jetpack Compose
- **Network**: Retrofit 2.9.0 + OkHttp + Gson 2.10.1
- **Async**: Coroutines 1.7.3 + StateFlow
- **Imágenes**: Coil 2.5.0
- **Min SDK**: 24 / Target SDK: 35

### Mobile iOS
- **Lenguaje**: Swift
- **UI**: SwiftUI
- **Network**: URLSession + async/await
- **State**: @Published + Combine
- **Min iOS**: 14+

---

## Estructura del Proyecto

```
claude-code/
├── Backend/
│   ├── app/
│   │   ├── main.py                    # FastAPI entry point + todos los endpoints
│   │   ├── core/config.py             # Settings y variables de entorno
│   │   ├── db/
│   │   │   ├── base.py                # Engine y Session SQLAlchemy
│   │   │   └── seed.py                # Script de datos de prueba
│   │   ├── models/
│   │   │   ├── base.py                # BaseModel: id, created_at, updated_at, deleted_at
│   │   │   ├── course.py              # Course + avg_rating property
│   │   │   ├── teacher.py             # Teacher
│   │   │   ├── lesson.py              # Lesson (mapped como "Class" en API)
│   │   │   ├── course_teacher.py      # Association table M2M
│   │   │   └── course_rating.py       # CourseRating (1-5, soft delete)
│   │   ├── schemas/rating.py          # Pydantic schemas para ratings
│   │   ├── services/course_service.py # Toda la lógica de negocio
│   │   ├── alembic/versions/          # Migraciones de DB
│   │   └── tests/                     # Tests de endpoints y servicios
│   ├── Dockerfile
│   ├── docker-compose.yml
│   ├── Makefile
│   └── pyproject.toml
│
├── Frontend/
│   └── src/
│       ├── app/
│       │   ├── page.tsx               # Home: grid de cursos (Server Component)
│       │   ├── course/[slug]/page.tsx # Detalle de curso
│       │   └── classes/[class_id]/page.tsx # Reproductor de video
│       ├── components/
│       │   ├── Course/                # Tarjeta de curso
│       │   ├── CourseDetail/          # Layout de detalle
│       │   ├── StarRating/            # Componente de estrellas (readonly)
│       │   └── VideoPlayer/           # Reproductor HTML5
│       ├── services/ratingsApi.ts     # Cliente HTTP con timeout y error handling
│       └── types/
│           ├── index.ts               # Course, Class, CourseDetail interfaces
│           └── rating.ts              # Rating types + type guards
│
└── Mobile/
    ├── PlatziFlixAndroid/
    │   └── app/src/main/java/com/espaciotiago/platziflixandroid/
    │       ├── data/                  # DTOs, Mappers, Repositories, Network
    │       ├── domain/                # Modelos de dominio + Repository interface
    │       ├── presentation/          # ViewModel, Screens, Components
    │       ├── di/AppModule.kt        # DI manual (toggle USE_MOCK_DATA)
    │       └── ui/theme/              # Design system
    └── PlatziFlixiOS/
        └── PlatziFlixiOS/
            ├── Data/                  # DTOs (Codable), Mappers, RemoteRepository
            ├── Domain/                # Models (Identifiable/Equatable) + Protocol
            ├── Presentation/          # ViewModel (@MainActor) + Views
            └── Services/              # NetworkManager, APIEndpoint, NetworkError
```

---

## Modelo de Datos

### Entidades y Relaciones

```
Course ──< course_teachers >── Teacher
  │
  └──< Lesson (slug, video_url)
  │
  └──< CourseRating (user_id, rating 1-5, soft delete)
```

### BaseModel (heredado por todos los modelos)
- `id` (PK), `created_at`, `updated_at`, `deleted_at` (soft delete)

### Course
- `name`, `description`, `thumbnail` (URL), `slug` (unique, indexed)
- Properties computadas: `average_rating`, `total_ratings`

### Teacher
- `name`, `email` (unique, indexed)

### Lesson
- `course_id` (FK), `name`, `description`, `slug`, `video_url`

### CourseRating
- `course_id` (FK), `user_id` (int, sin FK por ahora), `rating` (1-5)
- UNIQUE constraint en `(course_id, user_id)` donde `deleted_at IS NULL`

---

## API Endpoints

### Cursos
- `GET /` — Bienvenida
- `GET /health` — Health check + conectividad DB
- `GET /courses` — Lista todos los cursos con rating stats
- `GET /courses/{slug}` — Detalle: profesores + clases + rating stats
- `GET /classes/{class_id}` — Detalle de clase con video URL

### Ratings
- `POST /courses/{course_id}/ratings` — Crear o actualizar rating (upsert, 201 en create)
- `GET /courses/{course_id}/ratings` — Todos los ratings activos
- `GET /courses/{course_id}/ratings/stats` — Stats agregadas (avg, total, distribución 1-5)
- `GET /courses/{course_id}/ratings/user/{user_id}` — Rating de un usuario (204 si no existe)
- `PUT /courses/{course_id}/ratings/{user_id}` — Actualizar rating (404 si no existe)
- `DELETE /courses/{course_id}/ratings/{user_id}` — Soft delete (204)

### Schemas de respuesta (Pydantic)
```python
RatingRequest:  { user_id: int > 0, rating: int 1-5 }
RatingResponse: { id, course_id, user_id, rating, created_at, updated_at }
RatingStatsResponse: { average_rating: float, total_ratings: int, rating_distribution: dict }
```

---

## Patrones de Desarrollo

### Patrón común en todas las plataformas

```
API Response (JSON)
      ↓
    DTO / Pydantic Schema
      ↓
    Mapper
      ↓
  Domain Model
      ↓
  ViewModel / Service
      ↓
       UI
```

### Backend
- **Service Layer**: `CourseService` concentra toda la lógica de negocio
- **Soft Deletes**: Filtro `deleted_at IS NULL` en todas las queries
- **Upsert en ratings**: POST crea o actualiza según si existe rating activo
- **Aggregation en DB**: stats se calculan con SQL (no en Python) para performance

### Frontend
- **Server Components**: data fetching con `fetch()` directo, `cache: "no-store"`
- **No client state** para datos principales — solo Server Components
- **ratingsApi.ts**: fetch con timeout 10s + AbortController + manejo de 404
- **Type guards**: `isCourseRating()`, `isRatingStats()`, `isValidRating()`

### Android (MVVM + Clean Architecture)
- **Capas**: `data/` → `domain/` → `presentation/`
- **Estado**: `CourseListUiState` via StateFlow, eventos via `CourseListUiEvent`
- **Toggle mock**: `USE_MOCK_DATA = false` en `AppModule.kt`
- **MVI**: ViewModel recibe eventos, emite estado

### iOS (MVVM + Clean Architecture)
- **Capas**: `Data/` → `Domain/` → `Presentation/`
- **ViewModel**: `@MainActor ObservableObject` con `@Published`
- **Search**: debounce 300ms en `$searchText`
- **Previews**: Mock data estático en extensión de `Course`

---

## Convenciones de Naming

| Plataforma | Convención |
|---|---|
| Python (Backend) | `snake_case` |
| TypeScript (Frontend) | `camelCase` variables, `PascalCase` componentes/tipos |
| Kotlin (Android) | `camelCase` variables, `PascalCase` clases |
| Swift (iOS) | `camelCase` variables, `PascalCase` tipos |

---

## Comandos de Desarrollo

### Backend
```bash
cd Backend
make start              # Iniciar Docker Compose (DB + API)
make stop               # Detener containers
make migrate            # Alembic upgrade head
make create-migration   # Generar nueva migración
make seed               # Poblar datos de prueba
make seed-fresh         # Limpiar y re-seedear
make logs               # Ver logs de todos los servicios
make clean              # Limpieza completa
```

### Frontend
```bash
cd Frontend
yarn dev          # Servidor de desarrollo (Turbopack)
yarn build        # Build de producción
yarn test         # Ejecutar tests con Vitest
yarn lint         # ESLint
```

---

## URLs del Sistema

- **Backend API**: http://localhost:8000
- **API Docs (Swagger)**: http://localhost:8000/docs
- **Frontend Web**: http://localhost:3000

---

## Base de Datos

- **Usuario**: `platziflix_user`
- **Password**: `platziflix_password`
- **Database**: `platziflix_db`
- **Puerto**: `5432`
- **Migraciones**: `Backend/app/alembic/versions/`

---

## Funcionalidades Implementadas

- ✅ Catálogo de cursos con grid estilo Netflix
- ✅ Detalle de cursos (profesores, lecciones, clases)
- ✅ Navegación por slug SEO-friendly
- ✅ Reproductor de video integrado
- ✅ Health checks de API y DB
- ✅ Sistema de ratings (CRUD completo con soft delete)
- ✅ Componente StarRating con soporte de media estrella
- ✅ Apps móviles nativas (Android + iOS)
- ✅ Testing en todos los componentes

---

## Reglas de Desarrollo

1. **Docker obligatorio** para el backend — siempre verificar que el contenedor esté corriendo antes de ejecutar comandos
2. Usar los comandos del **Makefile** para operaciones de backend (no ejecutar directamente)
3. **TypeScript strict** en Frontend — no usar `any`
4. **Testing requerido** para nuevas funcionalidades en todas las plataformas
5. **Migraciones obligatorias** para cualquier cambio de esquema DB
6. **API REST** como única fuente de datos para Frontend y Mobile
7. Los comandos del backend deben ejecutarse **dentro del contenedor Docker** — revisar Makefile antes de ejecutar
