# Top SaaS Dashboard - Plan de Desarrollo

## Descripción General

**Top SaaS Dashboard** es una aplicación full-stack diseñada para inversionistas que desean analizar y evaluar métricas de las 100 empresas SaaS más importantes del mundo. La aplicación proporciona visualización interactiva de datos clave como valoración, ingresos anuales, financiamiento total, y permite filtrar empresas por industria y ubicación geográfica.

### Problema que resuelve
Los inversionistas necesitan una forma rápida y eficiente de:
- Comparar métricas financieras de empresas SaaS top
- Identificar tendencias por industria y ubicación
- Evaluar oportunidades de inversión basadas en datos concretos

### Para quién es
- Inversionistas de capital de riesgo (VC)
- Analistas financieros especializados en tecnología
- Fondos de inversión enfocados en SaaS
- Emprendedores que buscan benchmarks del mercado

### Por qué es valioso
- **Centralización de datos**: Información de múltiples fuentes en un solo dashboard
- **Visualización clara**: Métricas presentadas de forma comprensible y comparable
- **Filtrado inteligente**: Búsqueda rápida por criterios relevantes
- **Datos normalizados**: Información estructurada y validada desde la base de datos

---

## Funcionalidades Principales

### 1. Listado de Empresas con Métricas Completas

**Qué hace:**
- Muestra un grid/tabla con las 100 empresas SaaS principales
- Presenta 8 campos clave por empresa:
  - Nombre de la empresa
  - Industria/Categoría
  - Ubicación (Ciudad, País)
  - Productos/Servicios
  - Año de fundación
  - Total de financiamiento (en USD)
  - Ingresos anuales recurrentes - ARR (en USD)
  - Valoración de la empresa (en USD)

**Por qué es importante:**
- Permite comparación rápida entre empresas
- Ofrece visión completa del panorama SaaS
- Facilita identificación de patrones y outliers

**Cómo funciona:**
- Frontend fetching de datos desde API REST
- Backend consulta PostgreSQL con joins a tablas relacionadas
- Paginación de 20 empresas por página
- Ordenamiento por defecto: valoración descendente

### 2. Filtrado por Industria

**Qué hace:**
- Dropdown/select con lista de todas las industrias disponibles
- Filtra el listado mostrando solo empresas de la industria seleccionada
- Combina con otros filtros activos

**Por qué es importante:**
- Los inversionistas se especializan en sectores específicos
- Permite análisis comparativo dentro de una vertical
- Facilita identificación de líderes por industria

**Cómo funciona:**
- Endpoint `/api/v1/industries` provee lista de industrias únicas
- Query param `?industry_id=X` en endpoint de empresas
- Frontend sincroniza filtro con URL (bookmarkable)
- Backend aplica filtro en SQL con `WHERE industry_id = ?`

### 3. Filtrado por Ubicación

**Qué hace:**
- Dropdown/select con lista de todas las ubicaciones (ciudad, país)
- Filtra el listado mostrando solo empresas de la ubicación seleccionada
- Combina con otros filtros activos

**Por qué es importante:**
- Análisis geográfico de concentración de SaaS
- Inversionistas con preferencia por regiones específicas
- Identificación de hubs tecnológicos

**Cómo funciona:**
- Endpoint `/api/v1/locations` provee lista de ubicaciones únicas
- Query param `?location_id=X` en endpoint de empresas
- Frontend sincroniza filtro con URL (bookmarkable)
- Backend aplica filtro en SQL con `WHERE location_id = ?`

### 4. Paginación del Listado

**Qué hace:**
- Divide el listado en páginas de 20 empresas
- Navegación con controles anterior/siguiente y números de página
- Indicador de página actual y total de páginas
- Contador de resultados totales

**Por qué es importante:**
- Mejora performance y tiempo de carga
- Facilita navegación en dataset grande
- Experiencia de usuario estándar y familiar

**Cómo funciona:**
- Query params `?page=N&page_size=20`
- Backend usa offset/limit en SQLAlchemy
- Respuesta incluye metadata: `total_count`, `page`, `page_size`, `total_pages`
- Frontend renderiza controles de paginación basados en metadata

---

## Experiencia de Usuario

### Perfiles de Usuario

**1. Inversor de Capital de Riesgo (VC Partner)**
- Necesidades: Comparar múltiples empresas rápidamente, identificar tendencias
- Nivel técnico: Medio-alto
- Frecuencia de uso: Diaria/semanal
- Dispositivos: Principalmente desktop, ocasionalmente tablet

**2. Analista Financiero**
- Necesidades: Análisis profundo de métricas, exportación de datos
- Nivel técnico: Alto
- Frecuencia de uso: Diaria
- Dispositivos: Desktop

**3. Emprendedor/Founder**
- Necesidades: Benchmarking, entender el mercado
- Nivel técnico: Variable
- Frecuencia de uso: Ocasional
- Dispositivos: Desktop y móvil

### Flujos Clave de Usuario

**Flujo 1: Exploración General**
1. Usuario accede al dashboard
2. Ve listado de empresas con métricas visibles
3. Scrollea y revisa primeras 20 empresas
4. Navega a página 2, 3... usando controles de paginación
5. Identifica empresas de interés

**Flujo 2: Búsqueda por Industria**
1. Usuario quiere ver solo empresas de "Cloud Infrastructure"
2. Abre dropdown de industrias
3. Selecciona "Cloud Infrastructure"
4. Listado se actualiza mostrando solo empresas de esa industria
5. URL se actualiza: `/companies?industry_id=5`
6. Usuario puede compartir esta URL con colegas

**Flujo 3: Búsqueda Combinada (Industria + Ubicación)**
1. Usuario ya filtró por industria "AI/ML"
2. Ahora selecciona ubicación "San Francisco, USA"
3. Listado muestra solo empresas de AI/ML en San Francisco
4. URL: `/companies?industry_id=3&location_id=12`
5. Paginación se ajusta al nuevo total de resultados

**Flujo 4: Resetear Filtros**
1. Usuario tiene múltiples filtros activos
2. Click en botón "Clear Filters" o "All Companies"
3. Filtros se resetean
4. Listado muestra todas las empresas nuevamente
5. URL: `/companies`

### Consideraciones de UI/UX

**Layout:**
- Header fijo con título y navegación
- Barra de filtros sticky debajo del header
- Grid/tabla responsive para listado de empresas
- Footer con paginación sticky

**Diseño Responsive:**
- Desktop: Grid de 1-2 columnas con tabla detallada
- Tablet: Grid de 1 columna, campos condensados
- Mobile: Cards apiladas verticalmente, campos esenciales

**Feedback Visual:**
- Loading states durante fetch de datos
- Skeleton loaders para mejor perceived performance
- Indicadores visuales de filtros activos
- Mensajes cuando no hay resultados

**Accesibilidad:**
- Labels claros en filtros
- Contraste adecuado de colores
- Navegación por teclado en controles
- Mensajes de error descriptivos

---

## Arquitectura

### Componentes del Sistema

```
┌─────────────────────────────────────────────────────────────┐
│                         Frontend                             │
│  Next.js 16 + TypeScript + Tailwind CSS (Port 3000)         │
│                                                              │
│  ┌──────────────┐  ┌────────────┐  ┌───────────────┐       │
│  │   Pages      │  │   Hooks    │  │  API Clients  │       │
│  │ /companies   │←→│ useCompanies│←→│ fetchCompanies│       │
│  │              │  │ useFilters  │  │ fetchIndustries│      │
│  └──────────────┘  └────────────┘  └───────────────┘       │
│         ↑                                    │               │
│         │                                    ↓               │
│  ┌──────────────────────────────────────────────────┐       │
│  │            Components                             │       │
│  │  - CompanyList                                    │       │
│  │  - CompanyCard                                    │       │
│  │  - FilterBar (IndustryFilter, LocationFilter)    │       │
│  │  - Pagination                                     │       │
│  └──────────────────────────────────────────────────┘       │
└─────────────────────────────────────────────────────────────┘
                            │
                            │ HTTP/REST
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                         Backend                              │
│         FastAPI + Python 3.12 + uvicorn (Port 8000)         │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │                 API Layer (Routers)                   │  │
│  │  - GET /api/v1/companies (with filters & pagination) │  │
│  │  - GET /api/v1/industries                            │  │
│  │  - GET /api/v1/locations                             │  │
│  │  - GET /api/v1/health                                │  │
│  └──────────────────────────────────────────────────────┘  │
│                            ↓                                 │
│  ┌──────────────────────────────────────────────────────┐  │
│  │              Service Layer (Business Logic)           │  │
│  │  - CompanyService (filtering, pagination logic)      │  │
│  │  - IndustryService                                   │  │
│  │  - LocationService                                   │  │
│  └──────────────────────────────────────────────────────┘  │
│                            ↓                                 │
│  ┌──────────────────────────────────────────────────────┐  │
│  │          Repository Layer (Data Access)               │  │
│  │  - CompanyRepository (async SQLAlchemy queries)      │  │
│  │  - IndustryRepository                                │  │
│  │  - LocationRepository                                │  │
│  └──────────────────────────────────────────────────────┘  │
│                            ↓                                 │
│  ┌──────────────────────────────────────────────────────┐  │
│  │                 Models (SQLAlchemy ORM)               │  │
│  │  - Company, Industry, Location, Investor             │  │
│  │  - CompanyInvestor (many-to-many)                    │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                            │
                            │ SQL (async)
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                    Database (Supabase)                       │
│                    PostgreSQL 15+                            │
│                                                              │
│  Tables: company, industry, location, investor,             │
│          company_investor                                   │
│                                                              │
│  Indexes: industry_id, location_id, company_investor joins  │
└─────────────────────────────────────────────────────────────┘
```

### Modelos de Datos

**Schemas Pydantic (Request/Response):**

```python
# Response Schemas
class IndustryRead(BaseModel):
    id: int
    name: str

class LocationRead(BaseModel):
    id: int
    city: str
    state: str | None
    country: str

class CompanyRead(BaseModel):
    id: int
    name: str
    products: str | None
    founding_year: int | None
    total_funding: int | None
    arr: int | None
    valuation: int | None
    industry: IndustryRead | None
    location: LocationRead | None

class CompanyListResponse(BaseModel):
    data: list[CompanyRead]
    total_count: int
    page: int
    page_size: int
    total_pages: int

# Query Parameters
class CompanyFilters(BaseModel):
    industry_id: int | None = None
    location_id: int | None = None
    page: int = 1
    page_size: int = 20
```

**SQLAlchemy Models:**

```python
class Industry(Base):
    __tablename__ = "industry"
    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str] = mapped_column(String(255), unique=True)
    companies: Mapped[list["Company"]] = relationship(back_populates="industry")

class Location(Base):
    __tablename__ = "location"
    id: Mapped[int] = mapped_column(primary_key=True)
    city: Mapped[str]
    state: Mapped[str | None]
    country: Mapped[str]
    companies: Mapped[list["Company"]] = relationship(back_populates="location")

class Company(Base):
    __tablename__ = "company"
    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str]
    products: Mapped[str | None]
    founding_year: Mapped[int | None]
    total_funding: Mapped[int | None]
    arr: Mapped[int | None]
    valuation: Mapped[int | None]
    industry_id: Mapped[int | None] = mapped_column(ForeignKey("industry.id"))
    location_id: Mapped[int | None] = mapped_column(ForeignKey("location.id"))
    industry: Mapped["Industry"] = relationship(back_populates="companies")
    location: Mapped["Location"] = relationship(back_populates="companies")
```

**TypeScript Types (Frontend):**

```typescript
export interface Industry {
  id: number;
  name: string;
}

export interface Location {
  id: number;
  city: string;
  state: string | null;
  country: string;
}

export interface Company {
  id: number;
  name: string;
  products: string | null;
  founding_year: number | null;
  total_funding: number | null;
  arr: number | null;
  valuation: number | null;
  industry: Industry | null;
  location: Location | null;
}

export interface CompanyListResponse {
  data: Company[];
  total_count: number;
  page: number;
  page_size: number;
  total_pages: number;
}

export interface CompanyFilters {
  industry_id?: number;
  location_id?: number;
  page?: number;
  page_size?: number;
}
```

### APIs e Integraciones

**API Endpoints:**

1. **GET /api/v1/companies**
   - Query params: `industry_id`, `location_id`, `page`, `page_size`
   - Response: `CompanyListResponse`
   - Función: Listar empresas con filtros y paginación

2. **GET /api/v1/industries**
   - Response: `list[IndustryRead]`
   - Función: Obtener todas las industrias para el dropdown

3. **GET /api/v1/locations**
   - Response: `list[LocationRead]`
   - Función: Obtener todas las ubicaciones para el dropdown

4. **GET /api/v1/health**
   - Response: `HealthResponse`
   - Función: Health check del backend

**Patrones de Integración:**

- **Frontend → Backend**: Fetch API con SWR para caching y revalidación
- **Backend → Database**: SQLAlchemy async sessions con dependency injection
- **Error Handling**: HTTPException en backend, error boundaries en frontend
- **CORS**: Configurado para permitir localhost:3000 en desarrollo

### Requisitos de Infraestructura

**Desarrollo:**
- Node.js 18+ para frontend
- Python 3.12+ con `uv` para backend
- PostgreSQL 15+ (Supabase)
- Puertos: 3000 (frontend), 8000 (backend)

**Base de Datos:**
- Conexión a Supabase con credenciales en `.env`
- Tablas ya creadas con scripts SQL
- Datos ya cargados (100 empresas)

**Variables de Entorno:**

Backend (`.env`):
```
DATABASE_URL=postgresql+asyncpg://user:pass@host:port/database
CORS_ORIGINS=http://localhost:3000
ENVIRONMENT=development
```

Frontend (`.env.local`):
```
NEXT_PUBLIC_API_URL=http://localhost:8000
```

---

## Hoja de Ruta de Desarrollo

### Fase 1: Fundamentos Backend (Core API)

**Alcance:**
- Configuración de conexión a base de datos con SQLAlchemy async
- Modelos SQLAlchemy para todas las tablas
- Dependency injection para sesiones de base de datos
- Schemas Pydantic para requests/responses
- Repositories para Company, Industry, Location
- Services para lógica de negocio
- Routers con endpoints básicos

**Entregables:**
1. `backend/core/database.py`: Engine, SessionLocal, get_db dependency
2. `backend/models/`: Industry, Location, Company models
3. `backend/schemas/`: Pydantic schemas para API
4. `backend/repositories/`: CompanyRepository, IndustryRepository, LocationRepository
5. `backend/services/`: CompanyService, IndustryService, LocationService
6. `backend/api/`: Routers para companies, industries, locations
7. Tests unitarios para repositories y services (>60% coverage)
8. Documentación OpenAPI actualizada en /docs

**Criterios de Aceptación:**
- ✅ Backend inicia sin errores
- ✅ GET /api/v1/industries retorna lista de industrias
- ✅ GET /api/v1/locations retorna lista de ubicaciones
- ✅ GET /api/v1/companies retorna lista paginada de empresas
- ✅ Filtros por industry_id y location_id funcionan
- ✅ Paginación funciona correctamente (offset/limit)
- ✅ Tests pasan con pytest
- ✅ Mypy y ruff checks pasan

### Fase 2: Frontend Base (UI Components)

**Alcance:**
- Tipos TypeScript para todas las entidades
- API client functions con fetch
- Custom hooks para data fetching (SWR)
- Componentes React para listado y filtros
- Integración con query params en URL
- Estilos con Tailwind CSS

**Entregables:**
1. `frontend/lib/types.ts`: Interfaces TypeScript completas
2. `frontend/lib/api.ts`: fetchCompanies, fetchIndustries, fetchLocations
3. `frontend/hooks/useCompanies.ts`: Custom hook con SWR
4. `frontend/components/CompanyList.tsx`: Listado de empresas
5. `frontend/components/CompanyCard.tsx`: Card individual de empresa
6. `frontend/components/FilterBar.tsx`: Barra de filtros
7. `frontend/components/IndustryFilter.tsx`: Dropdown de industrias
8. `frontend/components/LocationFilter.tsx`: Dropdown de ubicaciones
9. `frontend/components/Pagination.tsx`: Controles de paginación
10. `frontend/app/companies/page.tsx`: Página principal del dashboard
11. Tests con Vitest/React Testing Library (>60% coverage)

**Criterios de Aceptación:**
- ✅ Frontend inicia sin errores en localhost:3000
- ✅ Listado de empresas se renderiza correctamente
- ✅ Filtros de industria y ubicación funcionan
- ✅ Filtros se reflejan en URL (bookmarkable)
- ✅ Paginación permite navegar entre páginas
- ✅ Loading states y error states funcionan
- ✅ UI es responsive (desktop, tablet, mobile)
- ✅ Tests pasan con npm test
- ✅ ESLint y type checks pasan

### Fase 3: Integración y Refinamiento

**Alcance:**
- Integración completa frontend-backend
- Manejo de errores robusto
- Validación de datos
- Optimizaciones de performance
- Mejoras de UX

**Entregables:**
1. Error boundaries en React
2. Validación de query params
3. Debouncing en filtros (si aplicable)
4. Skeleton loaders para mejor UX
5. Mensajes de "No results found"
6. Botón "Clear filters"
7. Formateo de números (currency, thousands separator)
8. Tooltips informativos en campos
9. Tests de integración end-to-end (opcional con Playwright)
10. README actualizado con instrucciones completas

**Criterios de Aceptación:**
- ✅ Flujo completo de usuario funciona sin errores
- ✅ Errores de red se manejan gracefully
- ✅ Performance es aceptable (<500ms para requests)
- ✅ UX es pulida y profesional
- ✅ Código está documentado
- ✅ Coverage de tests >60% en ambos proyectos
- ✅ Scripts de desarrollo funcionan correctamente

---

## Riesgos y Mitigaciones

### Riesgos Técnicos

**1. Complejidad de SQLAlchemy Async**
- **Riesgo**: Curva de aprendizaje para async/await patterns con SQLAlchemy
- **Probabilidad**: Media
- **Impacto**: Medio (puede retrasar desarrollo backend)
- **Mitigación**:
  - Empezar con ejemplos simples y escalar complejidad
  - Usar documentación oficial de SQLAlchemy 2.0
  - Implementar tests unitarios desde el inicio
  - Considerar fallback a SQLAlchemy sync si async es bloqueante

**2. Problemas de CORS en Desarrollo**
- **Riesgo**: Errores de CORS al conectar frontend (3000) con backend (8000)
- **Probabilidad**: Alta
- **Impacto**: Bajo (fácil de resolver)
- **Mitigación**:
  - Configurar CORSMiddleware desde el inicio
  - Documentar configuración en README
  - Usar variables de entorno para orígenes permitidos

**3. Conexión a Supabase Inestable**
- **Riesgo**: Timeouts o errores de conexión a base de datos
- **Probabilidad**: Baja
- **Impacto**: Alto (bloquea desarrollo)
- **Mitigación**:
  - Implementar retry logic en conexiones
  - Usar connection pooling adecuado
  - Tener credenciales de backup
  - Considerar base de datos local SQLite para tests

**4. Inconsistencias en Paginación**
- **Riesgo**: Resultados duplicados o faltantes al paginar con datos cambiantes
- **Probabilidad**: Baja (dataset estático de 100 empresas)
- **Impacto**: Bajo
- **Mitigación**:
  - Implementar ordenamiento consistente (por id o valuation)
  - Documentar limitaciones conocidas
  - Si datos se vuelven dinámicos, migrar a cursor-based pagination

### Desafíos de Arquitectura

**1. Separación de Responsabilidades**
- **Desafío**: Mantener clara separación entre capas (Router/Service/Repository)
- **Mitigación**:
  - Seguir estrictamente el patrón arquitectónico definido
  - Code reviews enfocadas en arquitectura
  - Refactorizar temprano si se detectan violaciones

**2. Type Safety Frontend-Backend**
- **Desafío**: Mantener sincronizados tipos TypeScript con schemas Pydantic
- **Mitigación**:
  - Generar tipos TypeScript desde OpenAPI (herramienta futura)
  - Documentar contratos de API claramente
  - Tests de integración que validen contratos

### Definición de Versión Inicial (MVP)

**Incluido en MVP:**
- ✅ Listado de empresas con 8 campos clave
- ✅ Filtro por industria
- ✅ Filtro por ubicación
- ✅ Paginación básica (20 items/página)
- ✅ UI responsive básica
- ✅ Health check endpoint

### Limitaciones de Recursos

**Tiempo:**
- Desarrollo estimado: 3-5 días para MVP completo
- Priorizar funcionalidad sobre perfección visual
- Iterar sobre mejoras después de MVP

**Equipo:**
- Desarrollo individual (full-stack)
- No hay diseñador dedicado (usar Tailwind defaults)
- No hay QA dedicado (testing manual + automatizado básico)

**Infraestructura:**
- Desarrollo local únicamente en primera versión
- No deployment a producción en MVP
- Base de datos compartida en Supabase (free tier)

---

## Apéndice

### Decisiones Arquitectónicas Clave

**DA-001: Paginación Offset/Limit**
- **Decisión**: Usar paginación tradicional con offset/limit
- **Justificación**: Dataset pequeño (100 empresas), implementación simple, UX familiar
- **Alternativas descartadas**: Cursor-based pagination (over-engineering para este caso)
- **Impacto**: Bajo esfuerzo de implementación, escalable hasta ~1000 empresas

**DA-002: Dependency Injection para DB Sessions**
- **Decisión**: Usar FastAPI Depends() con AsyncSession y context managers
- **Justificación**: Patrón recomendado, facilita testing, maneja transacciones automáticamente
- **Alternativas descartadas**: Singleton global (dificulta testing, problemas de concurrencia)
- **Impacto**: Mayor mantenibilidad, código más testeable

**DA-003: Query Params en URL para Filtros**
- **Decisión**: Sincronizar filtros con query params usando useSearchParams de Next.js
- **Justificación**: URLs bookmarkables, navegación back/forward funciona, compartir filtros
- **Alternativas descartadas**: Estado local con useState (pierde estado al refrescar)
- **Impacto**: Mejor UX, aprovecha capacidades de Next.js App Router

### Especificaciones Técnicas

**Database Schema:**
- Referencia: `scripts/database/01-top-saas-db-creation.sql`
- Normalización: 3NF (Third Normal Form)
- Relaciones:
  - Company N:1 Industry
  - Company N:1 Location
  - Company N:M Investor (tabla junction)

**API Versioning:**
- Versión actual: v1
- Prefijo: `/api/v1/`
- Futuras versiones: `/api/v2/` sin romper v1

**Performance Targets:**
- Response time: <500ms para endpoints
- Page load: <2s para página completa
- Database queries: <100ms para listado con filtros

**Testing Strategy:**
- Backend: pytest con TestClient de FastAPI
- Frontend: Vitest + React Testing Library
- Coverage mínimo: 60% en ambos proyectos
- E2E: Opcional con Playwright (post-MVP)

### Dataset Reference

**Fuente Original:**
- Kaggle: Top 100 SaaS Companies
- URL: https://www.kaggle.com/datasets/shreyasdasari7/top-100-saas-companiesstartups
- Formato: CSV
- Registros: 100 empresas

**Campos del Dataset:**
```
Company Name, Industry/Category, Location (City, Country), 
Products/Services, Founded Year, Total Funding, Annual Revenue,
Valuation, Investors
```

**Transformaciones Aplicadas:**
- Normalización de industrias en tabla separada
- Normalización de ubicaciones en tabla separada
- Normalización de inversores en tabla separada
- Parsing de múltiples inversores a relación M:N
- Conversión de campos monetarios a BIGINT
- Estandarización de nombres de países/ciudades

---

**Documento generado el:** 6 de noviembre de 2025  
**Versión:** 1.0  
**Estado:** Aprobado para implementación
