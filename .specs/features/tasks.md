# Lista de Tareas del Proyecto - Top SaaS Dashboard

Este documento desglosa el trabajo definido en `planning.md` en tareas accionables para el desarrollo del MVP.

---

## Fase 0: Diseño y Documentación de Arquitectura

### Decisiones Arquitectónicas (ADRs)

- [ ] Crear ADR-001: Arquitectura por Capas para Backend
    - [ ] Documentar decisión de usar patrón Router → Service → Repository → Model
    - [ ] Justificar separación de responsabilidades y mantenibilidad
    - [ ] Describir consecuencias: mayor estructura inicial, mejor testabilidad
    - [ ] Guardar en `docs/adrs/001-arquitectura-por-capas-backend.md`

- [ ] Crear ADR-002: Supabase Python Client para Acceso a Datos
    - [ ] Documentar decisión de usar Supabase Python Client en lugar de SQLAlchemy ORM
    - [ ] Justificar uso de cliente directo con PostgreSQL REST API de Supabase
    - [ ] Describir consecuencias: simplicidad, menos boilerplate, integración directa con Supabase
    - [ ] Incluir alternativa descartada (SQLAlchemy ORM con mayor complejidad)
    - [ ] Guardar en `docs/adrs/002-supabase-python-client.md`

- [ ] Crear ADR-003: Estrategia de Manejo de Errores
    - [ ] Documentar patrón de errores: HTTPException (backend) + Error Boundaries (frontend)
    - [ ] Justificar separación de concerns entre capas
    - [ ] Describir consecuencias: errores centralizados, mejor UX
    - [ ] Incluir estructura de respuestas de error estándar
    - [ ] Guardar en `docs/adrs/003-estrategia-manejo-errores.md`

### Diagramas de Arquitectura C4

- [ ] Crear Diagrama C4 - Nivel 1 (Contexto)
    - [ ] Definir sistema Top SaaS Dashboard como caja principal
    - [ ] Identificar actores externos: Inversores, Analistas, Emprendedores
    - [ ] Identificar sistemas externos: Supabase PostgreSQL
    - [ ] Mostrar interacciones principales (usuarios consultan métricas, sistema lee de DB)
    - [ ] Usar MermaidJS para generar diagrama
    - [ ] Guardar código Mermaid en `docs/architecture/c4-level1-context.md`

- [ ] Crear Diagrama C4 - Nivel 2 (Contenedores)
    - [ ] Definir contenedor Frontend (Next.js, TypeScript, Tailwind CSS, Port 3000)
    - [ ] Detallar subcapas Frontend: Pages, Components, Hooks, API Clients
    - [ ] Definir contenedor Backend (FastAPI, Python 3.12, uvicorn, Port 8000)
    - [ ] Detallar subcapas Backend: Routers, Services, Repositories (Supabase Client)
    - [ ] Definir contenedor Database (PostgreSQL 15+, Supabase)
    - [ ] Mostrar protocolos de comunicación: HTTP/REST (Frontend↔Backend), Supabase REST API (Backend↔Database)
    - [ ] Usar MermaidJS para generar diagrama detallado
    - [ ] Guardar código Mermaid en `docs/architecture/c4-level2-containers.md`

### Diagrama de Base de Datos

- [ ] Crear Diagrama Entidad-Relación (ER) completo
    - [ ] Incluir tabla `industry` con atributos (id, name, created_at, updated_at, created_by, updated_by)
    - [ ] Incluir tabla `location` con atributos (id, city, state, country, created_at, updated_at, created_by, updated_by)
    - [ ] Incluir tabla `company` con atributos (id, name, products, founding_year, total_funding, arr, valuation, employees, g2_rating, industry_id, location_id, created_at, updated_at, created_by, updated_by)
    - [ ] Incluir tabla `investor` con atributos (id, name, created_at, updated_at, created_by, updated_by)
    - [ ] Incluir tabla `company_investor` (junction table) con atributos (company_id, investor_id)
    - [ ] Definir relaciones: company N:1 industry, company N:1 location, company N:M investor
    - [ ] Marcar claves primarias y foráneas
    - [ ] Incluir índices existentes (idx_company_industry, idx_company_location, idx_company_investor_company, idx_company_investor_investor)
    - [ ] Usar MermaidJS para generar diagrama ER
    - [ ] Guardar código Mermaid en `docs/database/entity-relationship-diagram.md`

### Documentación de Arquitectura

- [ ] Documentar decisiones de arquitectura en README principal
    - [ ] Sección "Arquitectura" con referencia a diagramas C4
    - [ ] Sección "Base de Datos" con referencia a diagrama ER
    - [ ] Links a ADRs relevantes

- [ ] Crear documento de referencia de API
    - [ ] Listar endpoints principales: GET /api/v1/companies, GET /api/v1/industries, GET /api/v1/locations
    - [ ] Documentar query parameters y response models
    - [ ] Incluir ejemplos de requests/responses
    - [ ] Guardar en `docs/api/endpoints-reference.md`

---

## Fase 1: Fundamentos Backend (Core API)

### Configuración Inicial de Backend

- [ ] Configurar estructura de carpetas del backend
    - [ ] Crear directorio `src/backend/core/` para configuración central
    - [ ] Crear directorio `src/backend/schemas/` para schemas Pydantic
    - [ ] Crear directorio `src/backend/repositories/` para capa de acceso a datos con Supabase
    - [ ] Crear directorio `src/backend/services/` para lógica de negocio
    - [ ] Crear directorio `src/backend/api/` para routers (ya existe)
    - [ ] Crear directorio `tests/backend/` para tests unitarios

- [ ] Configurar dependencias del backend con `uv`
    - [ ] Verificar `pyproject.toml` tiene supabase>=2.0, pydantic-settings
    - [ ] Ejecutar `uv add supabase` para instalar cliente de Supabase
    - [ ] Ejecutar `uv sync` para instalar dependencias
    - [ ] Verificar que `uv run fastapi dev` inicia correctamente

- [ ] Configurar variables de entorno
    - [ ] Crear archivo `.env.example` con template de variables necesarias
    - [ ] Documentar variables requeridas: SUPABASE_URL, SUPABASE_KEY, CORS_ORIGINS, ENVIRONMENT
    - [ ] Actualizar `.gitignore` para excluir `.env`

### Base de Datos - Conexión y Configuración

- [ ] Revisar esquema de base de datos existente
    - [ ] Analizar script `scripts/database/01-top-saas-db-creation.sql`
    - [ ] Verificar tablas creadas: company, industry, location, investor, company_investor
    - [ ] Confirmar relaciones y tipos de datos
    - [ ] Identificar índices existentes

- [ ] Implementar conexión a Supabase
    - [ ] Crear `src/backend/core/supabase.py`
    - [ ] Configurar cliente Supabase con URL y API Key desde variables de entorno
    - [ ] Implementar `get_supabase_client()` dependency para FastAPI
    - [ ] Usar Client singleton pattern para reutilizar conexión
    - [ ] Agregar logging para operaciones de Supabase

### Schemas Pydantic (Request/Response)

- [ ] Implementar schemas para `Industry`
    - [ ] Crear `src/backend/schemas/industry.py`
    - [ ] Definir `IndustryRead` con id, name
    - [ ] Configurar `ConfigDict` con `from_attributes=True` para ORM compatibility
    - [ ] Incluir docstrings descriptivos

- [ ] Implementar schemas para `Location`
    - [ ] Crear `src/backend/schemas/location.py`
    - [ ] Definir `LocationRead` con id, city, state (nullable), country
    - [ ] Configurar `ConfigDict` con `from_attributes=True`
    - [ ] Incluir docstrings descriptivos

- [ ] Implementar schemas para `Company`
    - [ ] Crear `src/backend/schemas/company.py`
    - [ ] Definir `CompanyRead` con todos los campos (id, name, products, founding_year, total_funding, arr, valuation, industry, location)
    - [ ] Usar nested schemas: `IndustryRead`, `LocationRead` (nullable)
    - [ ] Configurar `ConfigDict` con `from_attributes=True`
    - [ ] Incluir docstrings descriptivos

- [ ] Implementar schemas para paginación y filtros
    - [ ] Crear `src/backend/schemas/pagination.py`
    - [ ] Definir `CompanyListResponse` con data (list), total_count, page, page_size, total_pages
    - [ ] Definir `CompanyFilters` con industry_id (optional), location_id (optional), page (default 1), page_size (default 20)
    - [ ] Incluir validaciones (page >= 1, page_size entre 1 y 100)
    - [ ] Incluir docstrings descriptivos

- [ ] Crear archivo `__init__.py` para schemas
    - [ ] Importar todos los schemas en `src/backend/schemas/__init__.py`
    - [ ] Facilitar importaciones desde otros módulos

### Repositories (Capa de Acceso a Datos)

- [ ] Implementar `IndustryRepository`
    - [ ] Crear `src/backend/repositories/industry_repository.py`
    - [ ] Método `get_all()`: retorna lista de todas las industrias usando `supabase.table("industry").select("*").order("name")`
    - [ ] Método `get_by_id(industry_id: int)`: retorna industria por ID usando `.eq("id", industry_id).single()`
    - [ ] Inyectar Supabase client como dependencia
    - [ ] Manejo de errores de Supabase API
    - [ ] Incluir type hints completos y docstrings

- [ ] Implementar `LocationRepository`
    - [ ] Crear `src/backend/repositories/location_repository.py`
    - [ ] Método `get_all()`: retorna lista de todas las ubicaciones usando `supabase.table("location").select("*").order("country,city")`
    - [ ] Método `get_by_id(location_id: int)`: retorna ubicación por ID
    - [ ] Inyectar Supabase client como dependencia
    - [ ] Manejo de errores de Supabase API
    - [ ] Incluir type hints completos y docstrings

- [ ] Implementar `CompanyRepository`
    - [ ] Crear `src/backend/repositories/company_repository.py`
    - [ ] Método `get_all(filters: CompanyFilters)`: retorna lista paginada con filtros
    - [ ] Usar Supabase query builder: `supabase.table("company").select("*, industry(*), location(*)")`
    - [ ] Implementar filtrado por industry_id si está presente usando `.eq("industry_id", X)`
    - [ ] Implementar filtrado por location_id si está presente usando `.eq("location_id", X)`
    - [ ] Implementar paginación con `.range(start, end)` basado en page y page_size
    - [ ] Implementar ordenamiento con `.order("valuation", desc=True)`
    - [ ] Método `count(filters)`: retorna total de empresas usando `.count()`
    - [ ] Método `get_by_id(company_id: int)`: retorna empresa por ID con joins
    - [ ] Inyectar Supabase client como dependencia
    - [ ] Manejo de errores de Supabase API
    - [ ] Incluir type hints completos y docstrings

- [ ] Crear archivo `__init__.py` para repositories
    - [ ] Importar todos los repositories en `src/backend/repositories/__init__.py`

### Services (Lógica de Negocio)

- [ ] Implementar `IndustryService`
    - [ ] Crear `src/backend/services/industry_service.py`
    - [ ] Método `get_all_industries()`: llama a repository y retorna lista
    - [ ] Inyectar `IndustryRepository` como dependencia
    - [ ] Incluir manejo de errores (log y re-raise como HTTPException si necesario)
    - [ ] Incluir type hints completos y docstrings

- [ ] Implementar `LocationService`
    - [ ] Crear `src/backend/services/location_service.py`
    - [ ] Método `get_all_locations()`: llama a repository y retorna lista
    - [ ] Inyectar `LocationRepository` como dependencia
    - [ ] Incluir manejo de errores (log y re-raise como HTTPException si necesario)
    - [ ] Incluir type hints completos y docstrings

- [ ] Implementar `CompanyService`
    - [ ] Crear `src/backend/services/company_service.py`
    - [ ] Método `get_companies(filters: CompanyFilters)`: retorna `CompanyListResponse`
    - [ ] Llamar a `CompanyRepository.get_all()` y `CompanyRepository.count()`
    - [ ] Calcular `total_pages` basado en total_count y page_size
    - [ ] Construir `CompanyListResponse` con metadata de paginación
    - [ ] Inyectar `CompanyRepository` como dependencia
    - [ ] Incluir validación de filtros (page válido, page_size dentro de límites)
    - [ ] Incluir manejo de errores (404 si página fuera de rango, 500 para errores de Supabase)
    - [ ] Incluir type hints completos y docstrings

- [ ] Crear archivo `__init__.py` para services
    - [ ] Importar todos los services en `src/backend/services/__init__.py`

### API Routers (Endpoints)

- [ ] Implementar router para `industries`
    - [ ] Crear `src/backend/api/industries.py`
    - [ ] Endpoint `GET /api/v1/industries`: retorna `list[IndustryRead]`
    - [ ] Inyectar `IndustryService` usando `Depends()`
    - [ ] Incluir response_model y status_code en decorator
    - [ ] Incluir docstring con descripción del endpoint
    - [ ] Configurar tags para documentación OpenAPI

- [ ] Implementar router para `locations`
    - [ ] Crear `src/backend/api/locations.py`
    - [ ] Endpoint `GET /api/v1/locations`: retorna `list[LocationRead]`
    - [ ] Inyectar `LocationService` usando `Depends()`
    - [ ] Incluir response_model y status_code en decorator
    - [ ] Incluir docstring con descripción del endpoint
    - [ ] Configurar tags para documentación OpenAPI

- [ ] Implementar router para `companies`
    - [ ] Crear `src/backend/api/companies.py`
    - [ ] Endpoint `GET /api/v1/companies`: retorna `CompanyListResponse`
    - [ ] Recibir query params usando `CompanyFilters` con `Depends()`
    - [ ] Inyectar `CompanyService` usando `Depends()`
    - [ ] Incluir response_model y status_code en decorator
    - [ ] Incluir docstring descriptivo con ejemplos de uso
    - [ ] Configurar tags para documentación OpenAPI
    - [ ] Manejar errores con HTTPException (400 para bad request, 500 para server error)

- [ ] Registrar routers en aplicación principal
    - [ ] Actualizar `src/backend/main.py`
    - [ ] Importar routers: industries_router, locations_router, companies_router
    - [ ] Registrar con `app.include_router()` usando prefijo `/api/v1`
    - [ ] Configurar tags apropiados para cada router

### Testing Backend

- [ ] Configurar estructura de tests para backend
    - [ ] Crear `tests/backend/conftest.py` con fixtures comunes
    - [ ] Fixture para Supabase client de prueba (mock o base de datos de test)
    - [ ] Fixture para TestClient de FastAPI
    - [ ] Fixture para datos de prueba (sample companies, industries, locations)
    - [ ] Considerar usar pytest-mock para mockear llamadas a Supabase

- [ ] Tests unitarios para `IndustryRepository`
    - [ ] Crear `tests/backend/repositories/test_industry_repository.py`
    - [ ] Test para `get_all()`: verifica que retorna todas las industrias
    - [ ] Test para `get_by_id()`: verifica que retorna industria correcta
    - [ ] Test para `get_by_id()` con ID inexistente: verifica manejo de error
    - [ ] Mockear respuestas de Supabase client

- [ ] Tests unitarios para `LocationRepository`
    - [ ] Crear `tests/backend/repositories/test_location_repository.py`
    - [ ] Test para `get_all()`: verifica ordenamiento por country, city
    - [ ] Test para `get_by_id()`: verifica que retorna ubicación correcta
    - [ ] Test para `get_by_id()` con ID inexistente: verifica manejo de error
    - [ ] Mockear respuestas de Supabase client

- [ ] Tests unitarios para `CompanyRepository`
    - [ ] Crear `tests/backend/repositories/test_company_repository.py`
    - [ ] Test para `get_all()` sin filtros: verifica paginación básica
    - [ ] Test para `get_all()` con filtro industry_id: verifica query correcto
    - [ ] Test para `get_all()` con filtro location_id: verifica query correcto
    - [ ] Test para `get_all()` con filtros combinados: verifica filtros múltiples
    - [ ] Test para `get_all()` con paginación: verifica range correcto
    - [ ] Test para `count()`: verifica conteo correcto con y sin filtros
    - [ ] Test para `get_by_id()`: verifica que incluye joins (industry, location)
    - [ ] Mockear respuestas de Supabase client

- [ ] Tests unitarios para `IndustryService`
    - [ ] Crear `tests/backend/services/test_industry_service.py`
    - [ ] Test para `get_all_industries()`: verifica que llama a repository correctamente
    - [ ] Mock de IndustryRepository para aislar lógica de servicio
    - [ ] Verificar manejo de errores si repository falla

- [ ] Tests unitarios para `LocationService`
    - [ ] Crear `tests/backend/services/test_location_service.py`
    - [ ] Test para `get_all_locations()`: verifica que llama a repository correctamente
    - [ ] Mock de LocationRepository para aislar lógica de servicio
    - [ ] Verificar manejo de errores si repository falla

- [ ] Tests unitarios para `CompanyService`
    - [ ] Crear `tests/backend/services/test_company_service.py`
    - [ ] Test para `get_companies()`: verifica construcción de CompanyListResponse
    - [ ] Test para cálculo de total_pages: verifica matemática correcta
    - [ ] Test para validación de filtros: verifica que rechaza valores inválidos
    - [ ] Mock de CompanyRepository para aislar lógica de servicio
    - [ ] Verificar manejo de errores (página fuera de rango, errores de Supabase)

- [ ] Tests de integración para endpoints
    - [ ] Crear `tests/backend/api/test_industries_endpoint.py`
    - [ ] Test GET /api/v1/industries: verifica status 200 y estructura de respuesta
    - [ ] Usar TestClient con mocks de Supabase

- [ ] Tests de integración para endpoints
    - [ ] Crear `tests/backend/api/test_locations_endpoint.py`
    - [ ] Test GET /api/v1/locations: verifica status 200 y estructura de respuesta
    - [ ] Usar TestClient con mocks de Supabase

- [ ] Tests de integración para endpoints
    - [ ] Crear `tests/backend/api/test_companies_endpoint.py`
    - [ ] Test GET /api/v1/companies sin filtros: verifica paginación default
    - [ ] Test GET /api/v1/companies?industry_id=X: verifica filtro funciona
    - [ ] Test GET /api/v1/companies?location_id=X: verifica filtro funciona
    - [ ] Test GET /api/v1/companies?industry_id=X&location_id=Y: verifica filtros combinados
    - [ ] Test GET /api/v1/companies?page=2&page_size=10: verifica paginación personalizada
    - [ ] Test con parámetros inválidos: verifica status 400 o 422
    - [ ] Usar TestClient con mocks de Supabase

- [ ] Verificar coverage de tests
    - [ ] Ejecutar `uv run pytest --cov=backend --cov-report=html`
    - [ ] Verificar que coverage es >= 60%
    - [ ] Identificar áreas sin cobertura y agregar tests si necesario

### Validación de Fase 1

- [ ] Verificar que backend inicia sin errores
    - [ ] Ejecutar `uv run fastapi dev`
    - [ ] Verificar que servidor inicia en http://localhost:8000
    - [ ] Verificar logs no muestran errores de conexión a Supabase

- [ ] Verificar endpoints en documentación OpenAPI
    - [ ] Acceder a http://localhost:8000/docs
    - [ ] Verificar que GET /api/v1/industries está documentado
    - [ ] Verificar que GET /api/v1/locations está documentado
    - [ ] Verificar que GET /api/v1/companies está documentado con query params
    - [ ] Probar endpoints desde Swagger UI

- [ ] Verificar linting y type checking
    - [ ] Ejecutar `uv run ruff check .` sin errores
    - [ ] Ejecutar `uv run ruff format .` para formatear código
    - [ ] Ejecutar `uv run mypy .` sin errores de tipos

- [ ] Verificar que todos los tests pasan
    - [ ] Ejecutar `uv run pytest -v`
    - [ ] Verificar que no hay tests fallidos
    - [ ] Revisar reporte de coverage

---

## Fase 2: Frontend Base (UI Components)

### Configuración Inicial de Frontend

- [ ] Configurar estructura de carpetas del frontend
    - [ ] Crear directorio `src/frontend/components/` para componentes React (si no existe)
    - [ ] Crear directorio `src/frontend/hooks/` para custom hooks (si no existe)
    - [ ] Crear subdirectorio `src/frontend/components/companies/` para componentes específicos
    - [ ] Crear subdirectorio `src/frontend/components/filters/` para componentes de filtrado
    - [ ] Crear subdirectorio `src/frontend/components/ui/` para componentes UI reutilizables

- [ ] Verificar dependencias del frontend
    - [ ] Confirmar que package.json incluye: next, react, typescript, tailwindcss, swr
    - [ ] Ejecutar `npm install` para instalar dependencias
    - [ ] Verificar que `npm run dev` inicia correctamente en http://localhost:3000

- [ ] Configurar variables de entorno del frontend
    - [ ] Crear archivo `.env.local.example` con template
    - [ ] Documentar variable NEXT_PUBLIC_API_URL (default: http://localhost:8000)
    - [ ] Actualizar `.gitignore` para excluir `.env.local`

### TypeScript Types (Tipos)

- [ ] Actualizar tipos existentes en `src/frontend/lib/types.ts`
    - [ ] Verificar interface `Industry` (id, name)
    - [ ] Agregar interface `Location` (id, city, state nullable, country)
    - [ ] Agregar interface `Company` (id, name, products, founding_year, total_funding, arr, valuation, industry nullable, location nullable)
    - [ ] Agregar interface `CompanyListResponse` (data, total_count, page, page_size, total_pages)
    - [ ] Agregar interface `CompanyFilters` (industry_id optional, location_id optional, page optional, page_size optional)
    - [ ] Incluir JSDoc comments para documentación
    - [ ] Exportar todas las interfaces

### API Clients (Fetching)

- [ ] Actualizar API client en `src/frontend/lib/api.ts`
    - [ ] Verificar configuración de base URL desde NEXT_PUBLIC_API_URL
    - [ ] Función `fetchIndustries()`: GET /api/v1/industries retorna Industry[]
    - [ ] Función `fetchLocations()`: GET /api/v1/locations retorna Location[]
    - [ ] Función `fetchCompanies(filters: CompanyFilters)`: GET /api/v1/companies retorna CompanyListResponse
    - [ ] Construir query string desde filters (URLSearchParams)
    - [ ] Incluir manejo de errores (throw error con mensaje descriptivo)
    - [ ] Incluir type guards o validación de respuestas
    - [ ] Incluir logging para debugging (opcional)

### Custom Hooks

- [ ] Crear custom hook `useIndustries`
    - [ ] Crear `src/frontend/hooks/useIndustries.ts`
    - [ ] Usar SWR para fetch de /api/v1/industries
    - [ ] Key: '/api/v1/industries'
    - [ ] Fetcher: fetchIndustries
    - [ ] Retornar: { industries, isLoading, error }
    - [ ] Configurar opciones SWR (revalidateOnFocus: false)
    - [ ] Incluir type hints completos

- [ ] Crear custom hook `useLocations`
    - [ ] Crear `src/frontend/hooks/useLocations.ts`
    - [ ] Usar SWR para fetch de /api/v1/locations
    - [ ] Key: '/api/v1/locations'
    - [ ] Fetcher: fetchLocations
    - [ ] Retornar: { locations, isLoading, error }
    - [ ] Configurar opciones SWR (revalidateOnFocus: false)
    - [ ] Incluir type hints completos

- [ ] Crear custom hook `useCompanies`
    - [ ] Crear `src/frontend/hooks/useCompanies.ts`
    - [ ] Recibir filters como parámetro
    - [ ] Usar SWR para fetch de /api/v1/companies con filtros
    - [ ] Key dinámica basada en filtros (serialize filters a string)
    - [ ] Fetcher: () => fetchCompanies(filters)
    - [ ] Retornar: { companies, pagination, isLoading, error, mutate }
    - [ ] Configurar opciones SWR (revalidateOnFocus: false, keepPreviousData: true)
    - [ ] Incluir type hints completos

### Componentes UI Reutilizables

- [ ] Crear componente `LoadingSpinner`
    - [ ] Crear `src/frontend/components/ui/LoadingSpinner.tsx`
    - [ ] Usar Tailwind CSS para animación de spinner
    - [ ] Props: size (small, medium, large)
    - [ ] Exportar como componente funcional

- [ ] Crear componente `ErrorMessage`
    - [ ] Crear `src/frontend/components/ui/ErrorMessage.tsx`
    - [ ] Props: message (string), onRetry (optional callback)
    - [ ] Mostrar mensaje de error con estilo apropiado
    - [ ] Botón "Retry" si onRetry está presente
    - [ ] Usar Tailwind CSS para estilos

- [ ] Crear componente `EmptyState`
    - [ ] Crear `src/frontend/components/ui/EmptyState.tsx`
    - [ ] Props: title, description, action (optional)
    - [ ] Mostrar mensaje cuando no hay resultados
    - [ ] Usar Tailwind CSS para estilos centrados

### Componentes de Filtros

- [ ] Crear componente `IndustryFilter`
    - [ ] Crear `src/frontend/components/filters/IndustryFilter.tsx`
    - [ ] Props: selectedIndustryId (nullable), onIndustryChange (callback)
    - [ ] Usar hook useIndustries para cargar opciones
    - [ ] Renderizar <select> con opción "All Industries" + lista de industrias
    - [ ] Mostrar LoadingSpinner mientras carga
    - [ ] Mostrar ErrorMessage si falla carga
    - [ ] Usar Tailwind CSS para estilos de select
    - [ ] Incluir label accesible

- [ ] Crear componente `LocationFilter`
    - [ ] Crear `src/frontend/components/filters/LocationFilter.tsx`
    - [ ] Props: selectedLocationId (nullable), onLocationChange (callback)
    - [ ] Usar hook useLocations para cargar opciones
    - [ ] Renderizar <select> con opción "All Locations" + lista de ubicaciones
    - [ ] Formato de opción: "City, Country" (incluir state si existe)
    - [ ] Mostrar LoadingSpinner mientras carga
    - [ ] Mostrar ErrorMessage si falla carga
    - [ ] Usar Tailwind CSS para estilos de select
    - [ ] Incluir label accesible

- [ ] Crear componente `FilterBar`
    - [ ] Crear `src/frontend/components/filters/FilterBar.tsx`
    - [ ] Props: filters (CompanyFilters), onFiltersChange (callback)
    - [ ] Incluir IndustryFilter y LocationFilter
    - [ ] Incluir botón "Clear Filters" que resetea todos los filtros
    - [ ] Layout horizontal en desktop, vertical en mobile
    - [ ] Usar Tailwind CSS para layout responsive
    - [ ] Sticky positioning debajo del header

### Componentes de Empresas

- [ ] Crear componente `CompanyCard`
    - [ ] Crear `src/frontend/components/companies/CompanyCard.tsx`
    - [ ] Props: company (Company)
    - [ ] Mostrar nombre de empresa como título destacado
    - [ ] Mostrar industria (si existe)
    - [ ] Mostrar ubicación en formato "City, Country"
    - [ ] Mostrar productos/servicios (truncar si es muy largo)
    - [ ] Mostrar founding_year
    - [ ] Mostrar total_funding con formato de moneda (ej. "$X.XB" o "$XM")
    - [ ] Mostrar arr con formato de moneda
    - [ ] Mostrar valuation con formato de moneda (destacado)
    - [ ] Usar Tailwind CSS para card con sombra, padding, border
    - [ ] Responsive: ajustar layout para mobile
    - [ ] Incluir estados hover con transiciones suaves

- [ ] Crear helper para formateo de números
    - [ ] Crear `src/frontend/lib/formatters.ts`
    - [ ] Función `formatCurrency(value: number | null)`: retorna string con formato "$X.XB", "$XM", "$XK" o "-"
    - [ ] Función `formatNumber(value: number | null)`: retorna string con separadores de miles o "-"
    - [ ] Función `formatYear(year: number | null)`: retorna string "YYYY" o "-"
    - [ ] Incluir tests básicos si es posible

- [ ] Crear componente `CompanyList`
    - [ ] Crear `src/frontend/components/companies/CompanyList.tsx`
    - [ ] Props: companies (Company[]), isLoading (boolean), error (Error | null)
    - [ ] Mostrar LoadingSpinner si isLoading=true
    - [ ] Mostrar ErrorMessage si error existe
    - [ ] Mostrar EmptyState si companies está vacío
    - [ ] Renderizar grid de CompanyCard para cada empresa
    - [ ] Grid responsive: 1 columna en mobile, 2 en tablet, 2-3 en desktop
    - [ ] Usar Tailwind CSS para grid layout con gap apropiado

### Componente de Paginación

- [ ] Crear componente `Pagination`
    - [ ] Crear `src/frontend/components/ui/Pagination.tsx`
    - [ ] Props: currentPage, totalPages, pageSize, totalCount, onPageChange (callback)
    - [ ] Mostrar botón "Previous" (disabled si currentPage=1)
    - [ ] Mostrar botón "Next" (disabled si currentPage=totalPages)
    - [ ] Mostrar números de página (simplificado: mostrar 1 ... currentPage-1, currentPage, currentPage+1 ... totalPages)
    - [ ] Mostrar texto "Showing X-Y of Z results"
    - [ ] Usar Tailwind CSS para estilos de botones y layout
    - [ ] Accesibilidad: aria-labels apropiados

### Página Principal de Companies

- [ ] Crear página de empresas
    - [ ] Crear `src/frontend/app/companies/page.tsx`
    - [ ] Usar 'use client' directive (Client Component)
    - [ ] Usar hooks de Next.js: useSearchParams, useRouter, usePathname
    - [ ] Leer filtros desde query params (industry_id, location_id, page)
    - [ ] Construir objeto CompanyFilters desde query params
    - [ ] Usar hook useCompanies con filtros
    - [ ] Renderizar FilterBar con callbacks que actualicen query params
    - [ ] Renderizar CompanyList con datos de useCompanies
    - [ ] Renderizar Pagination con callbacks que actualicen query params
    - [ ] Función helper updateQueryParams() que construye nueva URL y hace router.push()
    - [ ] Layout: Header → FilterBar (sticky) → CompanyList → Pagination (sticky)
    - [ ] Usar Tailwind CSS para layout general

- [ ] Actualizar navegación principal
    - [ ] Actualizar `src/frontend/app/page.tsx` (home)
    - [ ] Agregar link/botón "View Companies Dashboard" que navegue a /companies
    - [ ] Mantener sección existente de Backend Status
    - [ ] Mejorar hero section con descripción del dashboard

- [ ] Crear layout para aplicación (si no existe)
    - [ ] Verificar `src/frontend/app/layout.tsx`
    - [ ] Header fijo con título "Top SaaS Dashboard" y navegación
    - [ ] Link a "/" (Home) y "/companies" (Dashboard)
    - [ ] Footer con información técnica (ya existe)
    - [ ] Usar Tailwind CSS para estilos

### Testing Frontend

- [ ] Configurar testing con Vitest (si no está configurado)
    - [ ] Verificar `vitest.config.ts` existe
    - [ ] Configurar alias de importación (@/components, @/lib)
    - [ ] Configurar environment: jsdom

- [ ] Tests para componente CompanyCard
    - [ ] Crear `tests/frontend/components/CompanyCard.test.tsx`
    - [ ] Test: renderiza correctamente con todos los datos
    - [ ] Test: maneja valores null/undefined gracefully (muestra "-")
    - [ ] Test: formatea currency correctamente
    - [ ] Usar React Testing Library

- [ ] Tests para componente IndustryFilter
    - [ ] Crear `tests/frontend/components/IndustryFilter.test.tsx`
    - [ ] Test: muestra loading state mientras carga
    - [ ] Test: renderiza opciones correctamente después de cargar
    - [ ] Test: llama a onIndustryChange cuando selecciona opción
    - [ ] Mock de useIndustries hook

- [ ] Tests para componente LocationFilter
    - [ ] Crear `tests/frontend/components/LocationFilter.test.tsx`
    - [ ] Test: muestra loading state mientras carga
    - [ ] Test: renderiza opciones correctamente después de cargar
    - [ ] Test: llama a onLocationChange cuando selecciona opción
    - [ ] Mock de useLocations hook

- [ ] Tests para formatters
    - [ ] Crear `tests/frontend/lib/formatters.test.ts`
    - [ ] Test formatCurrency: valores grandes ($1.5B), medianos ($500M), pequeños ($50K)
    - [ ] Test formatCurrency: maneja null retornando "-"
    - [ ] Test formatNumber: incluye separadores de miles
    - [ ] Test formatYear: maneja null retornando "-"

- [ ] Verificar coverage de tests frontend
    - [ ] Ejecutar `npm test -- --coverage`
    - [ ] Verificar que coverage es >= 60%
    - [ ] Identificar componentes sin cobertura y agregar tests si necesario

### Validación de Fase 2

- [ ] Verificar que frontend inicia sin errores
    - [ ] Ejecutar `npm run dev`
    - [ ] Verificar que servidor inicia en http://localhost:3000
    - [ ] Verificar consola del navegador no muestra errores

- [ ] Verificar integración con backend
    - [ ] Asegurar backend está corriendo en http://localhost:8000
    - [ ] Navegar a http://localhost:3000/companies
    - [ ] Verificar que listado de empresas se carga correctamente
    - [ ] Verificar que filtros de industria y ubicación funcionan
    - [ ] Verificar que paginación funciona

- [ ] Verificar linting y type checking frontend
    - [ ] Ejecutar `npm run lint` sin errores
    - [ ] Ejecutar `npm run type-check` (o `npx tsc --noEmit`) sin errores de tipos

- [ ] Verificar responsive design
    - [ ] Probar en viewport mobile (375px)
    - [ ] Probar en viewport tablet (768px)
    - [ ] Probar en viewport desktop (1440px)
    - [ ] Verificar que layout se adapta correctamente

---

## Fase 3: Integración y Refinamiento

### Integración Frontend-Backend

- [ ] Verificar flujo completo de datos
    - [ ] Backend ejecutándose en puerto 8000
    - [ ] Frontend ejecutándose en puerto 3000
    - [ ] Navegar a /companies y verificar datos se cargan
    - [ ] Aplicar filtro de industria, verificar request a backend
    - [ ] Aplicar filtro de ubicación, verificar request a backend
    - [ ] Cambiar página, verificar request con paginación correcta

- [ ] Verificar configuración CORS
    - [ ] Revisar que CORSMiddleware en backend permite http://localhost:3000
    - [ ] Verificar en DevTools del navegador que no hay errores CORS
    - [ ] Probar desde diferentes navegadores (Chrome, Firefox)

### Manejo de Errores Robusto

- [ ] Implementar Error Boundary en React
    - [ ] Crear `src/frontend/components/ErrorBoundary.tsx`
    - [ ] Capturar errores de componentes hijos
    - [ ] Mostrar UI de fallback con mensaje de error
    - [ ] Incluir botón "Reload Page"
    - [ ] Logging de errores (console.error o servicio externo)

- [ ] Aplicar Error Boundary a página de empresas
    - [ ] Envolver CompanyList en ErrorBoundary en /companies page
    - [ ] Probar lanzando error intencional para verificar funciona

- [ ] Mejorar manejo de errores en API calls
    - [ ] Actualizar fetchCompanies, fetchIndustries, fetchLocations
    - [ ] Incluir try/catch para errores de red
    - [ ] Retornar objetos de error descriptivos
    - [ ] Mostrar mensajes de error user-friendly en ErrorMessage component

### Validación de Datos

- [ ] Validación en backend de query params
    - [ ] Verificar que CompanyFilters en backend valida page >= 1
    - [ ] Verificar que CompanyFilters valida page_size entre 1 y 100
    - [ ] Retornar 422 Unprocessable Entity si validación falla
    - [ ] Incluir mensajes de error descriptivos en response

- [ ] Validación en frontend antes de enviar requests
    - [ ] En updateQueryParams(), validar que page es número positivo
    - [ ] En updateQueryParams(), validar que industry_id y location_id son números válidos
    - [ ] Sanitizar valores antes de construir query string

### Optimizaciones de Performance

- [ ] Implementar skeleton loaders
    - [ ] Crear `src/frontend/components/ui/CompanySkeleton.tsx`
    - [ ] Skeleton con forma de CompanyCard (placeholder rectangles con animación pulse)
    - [ ] Usar en CompanyList cuando isLoading=true
    - [ ] Mostrar 6-8 skeletons para simular carga

- [ ] Optimizar renders de React
    - [ ] Usar React.memo en CompanyCard si es necesario
    - [ ] Verificar que callbacks en FilterBar usan useCallback
    - [ ] Verificar que no hay re-renders innecesarios (React DevTools Profiler)

- [ ] (Opcional) Implementar debouncing en filtros
    - [ ] Si se agregan filtros de texto, usar debounce para evitar requests excesivos
    - [ ] No aplicable actualmente (solo selects), pero documentar para futuro

### Mejoras de UX

- [ ] Implementar mensaje "No results found"
    - [ ] Usar componente EmptyState cuando CompanyList recibe array vacío
    - [ ] Mensaje: "No companies found matching your filters"
    - [ ] Botón "Clear Filters" en EmptyState

- [ ] Implementar botón "Clear Filters"
    - [ ] En FilterBar, agregar botón "Clear All Filters"
    - [ ] Click en botón navega a /companies (sin query params)
    - [ ] Resetea todos los filtros a valores default
    - [ ] Mostrar solo si hay filtros activos

- [ ] Formateo de números mejorado
    - [ ] Verificar que formatCurrency muestra correctamente: $1.5B, $500M, $50K
    - [ ] Incluir separadores de miles en números grandes
    - [ ] Color coding: valuation en verde/destacado, funding en azul, arr en morado (opcional)

- [ ] (Opcional) Agregar tooltips informativos
    - [ ] Tooltip en "Total Funding": explicar qué significa
    - [ ] Tooltip en "ARR": explicar Annual Recurring Revenue
    - [ ] Tooltip en "Valuation": explicar valoración de empresa
    - [ ] Usar biblioteca de tooltips (ej. Radix UI) o CSS puro

### Documentación

- [ ] Actualizar README principal
    - [ ] Sección "Getting Started" con pasos para ejecutar proyecto
    - [ ] Instrucciones para backend: `cd src/backend && uv run fastapi dev`
    - [ ] Instrucciones para frontend: `cd src/frontend && npm run dev`
    - [ ] Requisitos previos: Node.js 18+, Python 3.12+, uv instalado
    - [ ] Variables de entorno necesarias (con ejemplos)
    - [ ] Links a documentación de arquitectura y API

- [ ] Crear guía de desarrollo
    - [ ] Crear `docs/development-guide.md`
    - [ ] Estructura del proyecto (organización de carpetas)
    - [ ] Convenciones de código (naming, patterns)
    - [ ] Cómo agregar nuevos endpoints
    - [ ] Cómo agregar nuevos componentes
    - [ ] Proceso de testing

- [ ] Documentar scripts de desarrollo
    - [ ] Verificar que scripts en `scripts/dev/` funcionan correctamente
    - [ ] Documentar `run-backend.sh` y `run-frontend.sh`
    - [ ] Crear versión PowerShell si es necesario (`.ps1`)

### Testing de Integración

- [ ] (Opcional) Configurar Playwright para tests E2E
    - [ ] Instalar Playwright: `npm install -D @playwright/test`
    - [ ] Configurar `playwright.config.ts`
    - [ ] Crear test básico de flujo completo

- [ ] (Opcional) Test E2E: Flujo de exploración general
    - [ ] Navegar a /companies
    - [ ] Verificar que empresas se cargan
    - [ ] Verificar que se pueden ver al menos 20 empresas
    - [ ] Click en página 2 de paginación
    - [ ] Verificar que URL cambia a ?page=2

- [ ] (Opcional) Test E2E: Flujo de filtrado por industria
    - [ ] Navegar a /companies
    - [ ] Seleccionar industria del dropdown
    - [ ] Verificar que URL cambia a ?industry_id=X
    - [ ] Verificar que listado se actualiza

- [ ] (Opcional) Test E2E: Flujo de filtros combinados
    - [ ] Navegar a /companies
    - [ ] Seleccionar industria
    - [ ] Seleccionar ubicación
    - [ ] Verificar que URL tiene ambos query params
    - [ ] Verificar que resultados se filtran correctamente

### Validación Final de Fase 3

- [ ] Verificar flujo completo funciona sin errores
    - [ ] Iniciar backend y frontend
    - [ ] Completar flujo de exploración general
    - [ ] Completar flujo de búsqueda por industria
    - [ ] Completar flujo de búsqueda combinada
    - [ ] Completar flujo de resetear filtros
    - [ ] No debe haber errores en consola de navegador
    - [ ] No debe haber errores en logs de backend

- [ ] Verificar performance
    - [ ] Usar DevTools Network tab
    - [ ] Verificar que GET /api/v1/companies responde en <500ms
    - [ ] Verificar que página completa carga en <2s (con cache frío)
    - [ ] Verificar que no hay waterfall requests innecesarios

- [ ] Verificar UX es pulida
    - [ ] Loading states son visibles y claros
    - [ ] Error states se manejan gracefully
    - [ ] Transiciones son suaves (no hay flashes)
    - [ ] Responsive design funciona en mobile/tablet/desktop
    - [ ] Colores y contraste son accesibles

- [ ] Verificar código está documentado
    - [ ] README tiene instrucciones completas
    - [ ] Componentes principales tienen docstrings/comments
    - [ ] API endpoints tienen descripciones en OpenAPI
    - [ ] ADRs documentan decisiones clave

- [ ] Verificar coverage de tests
    - [ ] Backend: `uv run pytest --cov` muestra >= 60%
    - [ ] Frontend: `npm test -- --coverage` muestra >= 60%
    - [ ] Identificar gaps críticos y agregar tests si necesario

- [ ] Verificar scripts de desarrollo funcionan
    - [ ] `scripts/dev/run-backend.sh` inicia backend correctamente
    - [ ] `scripts/dev/run-frontend.sh` inicia frontend correctamente
    - [ ] Ambos scripts manejan errores apropiadamente

---

## Notas Finales

### Criterios de Éxito del MVP

El MVP estará completo cuando:
- ✅ Todas las tareas de las 3 fases estén marcadas como completadas
- ✅ Backend expone 3 endpoints funcionales: /companies, /industries, /locations
- ✅ Frontend muestra listado de empresas con filtros y paginación
- ✅ Filtros sincronizan con URL (bookmarkable)
- ✅ Tests unitarios cubren >= 60% del código en backend y frontend
- ✅ Aplicación funciona localmente sin errores
- ✅ Documentación permite a nuevo desarrollador ejecutar proyecto

### Priorización de Tareas

**Críticas (P0):**
- Fase 1 completa (backend core)
- Fase 2 completa (frontend base)
- Integración básica funcional

**Importantes (P1):**
- Testing con coverage >= 60%
- Manejo de errores robusto
- Documentación básica

**Opcionales (P2):**
- Tests E2E con Playwright
- Tooltips informativos
- Optimizaciones avanzadas de performance

### Estimación de Esfuerzo

- **Fase 0 (Diseño)**: 4-6 horas
- **Fase 1 (Backend)**: 12-16 horas
- **Fase 2 (Frontend)**: 12-16 horas
- **Fase 3 (Integración)**: 6-8 horas
- **Total estimado**: 34-46 horas (~4-6 días de trabajo)

### Dependencias Críticas

- Base de datos Supabase debe estar accesible y con datos cargados
- Variables de entorno deben estar configuradas correctamente (SUPABASE_URL, SUPABASE_KEY)
- Puertos 3000 y 8000 deben estar disponibles
- Node.js 18+ y Python 3.12+ deben estar instalados
- Gestor `uv` debe estar instalado para backend
