# 📋 ESPECIFICACIONES DEL PROYECTO

## 📌 RESUMEN EJECUTIVO

**Proyecto:** Sistema de Microservicios para Gestión de Autores y Publicaciones  
**Plazo:** Hasta 3 semanas  
**Equipo:** Máximo 3 personas  
**Lenguajes:** JavaScript/Node.js (Backend), React (Frontend)  
**Despliegue:** Docker Compose  

---

## 🎯 OBJETIVOS

1. **Construir solución funcional** con dos microservicios desacoplados (Autores y Publicaciones)
2. **Implementar clases abstractas y derivadas** en cada microservicio (Herencia OOP)
3. **Crear frontend React** para operaciones CRUD básicas
4. **Modelar proceso BPMN** en Camunda (sin integración con microservicios)
5. **Desplegar con Docker Compose** de forma reproducible
6. **Aplicar principios SOLID** y patrones de diseño
7. **Incluir pruebas básicas** para validar funcionalidad

---

## 📊 ESPECIFICACIÓN TÉCNICA

### 1. MICROSERVICIO DE AUTORES

#### 1.1 Responsabilidad
Administrar el ciclo de vida de autores (registro, consulta, actualización, eliminación)

#### 1.2 Requisitos de Dominio

**Clases Requeridas:**
- `Person` (Clase Abstracta Base)
  - Propiedades: `id`, `name`, `email`, `birthDate`, `createdAt`
  - Método abstracto: `validate()`

- `Author` (Clase Derivada Concreta)
  - Extiende: `Person`
  - Propiedades adicionales: `bio`, `expertise`, `nationality`
  - Implementa: `validate()` para reglas específicas de autor

#### 1.3 Persistencia
- **ORM:** Sequelize
- **Base de Datos:** PostgreSQL
- **IP:Puerto:** `localhost:5432` (desarrollo) / `db-authors` en Docker

#### 1.4 API REST Endpoints

| Método | Ruta | Descripción | Status |
|--------|------|-------------|--------|
| POST | `/authors` | Crear nuevo autor | 201 |
| GET | `/authors` | Listar autores (paginación) | 200 |
| GET | `/authors/{id}` | Obtener autor específico | 200 |
| PUT | `/authors/{id}` | Actualizar autor | 200 |
| DELETE | `/authors/{id}` | Eliminar autor | 204 |
| GET | `/health` | Verificar servicio | 200 |

#### 1.5 Ejemplo de Payload

**POST /authors**
```json
{
  "name": "Gabriel García Márquez",
  "email": "gabriel@example.com",
  "birthDate": "1927-03-06",
  "bio": "Escritor colombiano, autor de Cien años de soledad",
  "expertise": "Realismo mágico",
  "nationality": "Colombia"
}
```

**Response 201**
```json
{
  "id": 1,
  "name": "Gabriel García Márquez",
  "email": "gabriel@example.com",
  "birthDate": "1927-03-06",
  "bio": "Escritor colombiano...",
  "expertise": "Realismo mágico",
  "nationality": "Colombia",
  "createdAt": "2026-02-04T10:30:00Z"
}
```

#### 1.6 Restricción importante
**El servicio de Autores NO debe consultar al servicio de Publicaciones** (evitar dependencia circular)

---

### 2. MICROSERVICIO DE PUBLICACIONES

#### 2.1 Responsabilidad
Administrar publicaciones y su estado editorial, con validación de autores existentes

#### 2.2 Requisitos de Dominio

**Clases Requeridas:**
- `Content` (Clase Abstracta Base)
  - Propiedades: `id`, `title`, `description`, `createdAt`, `updatedAt`
  - Método abstracto: `validate()`

- `Publication` (Clase Derivada Concreta)
  - Extiende: `Content`
  - Propiedades:
    - `authorId` (referencia a Authors Service)
    - `status` (DRAFT | IN_REVIEW | APPROVED | PUBLISHED | REJECTED)
    - `reviewComments` (string)
    - `publishedDate` (date nullable)
  - Métodos:
    - `updateStatus(newStatus)` ← Valida transiciones
    - `canTransitionTo(status)` ← Regla de negocio
    - `isPublished()` ← Helper

#### 2.3 Persistencia
- **ORM:** Sequelize
- **Base de Datos:** PostgreSQL
- **IP:Puerto:** `localhost:5433` (desarrollo) / `db-publications` en Docker

#### 2.4 API REST Endpoints

| Método | Ruta | Descripción | Status |
|--------|------|-------------|--------|
| POST | `/publications` | Crear publicación (valida autor) | 201 |
| GET | `/publications` | Listar publicaciones (filtrable) | 200 |
| GET | `/publications/{id}` | Obtener publicación | 200 |
| GET | `/publications/{id}/detail` | Obtener con datos del autor | 200 |
| PATCH | `/publications/{id}/status` | Cambiar estado editorial | 200 |
| DELETE | `/publications/{id}` | Eliminar publicación | 204 |
| GET | `/health` | Verificar servicio | 200 |

#### 2.5 Estados Editorial (Transiciones Permitidas)

```
DRAFT ──→ IN_REVIEW ──→ APPROVED ──→ PUBLISHED
  ↑         │            ↓
  └─────────┴───────── REJECTED
```

**Tabla de Transiciones Válidas:**
```
DRAFT:       → IN_REVIEW
IN_REVIEW:   → APPROVED | REJECTED | DRAFT (revisar)
APPROVED:    → PUBLISHED | REJECTED | IN_REVIEW
PUBLISHED:   → REJECTED (retracción)
REJECTED:    → DRAFT (reescribir)
```

#### 2.6 Ejemplo de Payload

**POST /publications**
```json
{
  "title": "Cien años de soledad",
  "description": "Novela maestra de la literatura latinoamericana",
  "authorId": 1
}
```

**Response 201**
```json
{
  "id": 1,
  "title": "Cien años de soledad",
  "description": "Novela maestra...",
  "authorId": 1,
  "status": "DRAFT",
  "reviewComments": null,
  "publishedDate": null,
  "createdAt": "2026-02-04T10:35:00Z"
}
```

**PATCH /publications/1/status**
```json
{
  "newStatus": "IN_REVIEW",
  "reviewComments": "Enviado a revisión editorial"
}
```

**GET /publications/1/detail** (Respuesta Enriquecida)
```json
{
  "id": 1,
  "title": "Cien años de soledad",
  "status": "APPROVED",
  "author": {
    "id": 1,
    "name": "Gabriel García Márquez",
    "email": "gabriel@example.com"
  },
  "createdAt": "2026-02-04T10:35:00Z"
}
```

#### 2.7 Validaciones Obligatorias

- **Al crear publicación:**
  - El `authorId` debe existir (llamada a Authors Service)
  - Si no existe: responder `404` con mensaje claro
  - Si timeout: responder `503` con reintentos internos
  - Si error: registrar en logs

#### 2.8 Comunicación Inter-Servicios

**Flujo Synchronous:**
```
Publications Service  
    │  
    ├─ HTTP GET /authors/{authorId}  
    │  ├─ Authors Service responde  
    │  ├─ Para procesar la solicitud  
    │  └─ Si falla: error 503 o 404
```

**Implementación:**
- Cliente HTTP con retry logic (máximo 3 intentos)
- Timeout configurado (ej: 5 segundos)
- Manejo de excepciones y errores
- Logging de llamadas

---

### 3. FRONTEND REACT

#### 3.1 Requisitos

**Tecnología:**
- Framework: React 18+
- Build: Vite
- Router: react-router-dom
- HTTP Client: axios
- Estilos: CSS Modules o Styled Components

#### 3.2 Páginas y Componentes

**Página: Gestión de Autores**
- [ ] `AuthorList` - Tabla paginable de autores
  - Columnas: ID, Nombre, Email, Acciones
  - Botón: "Nuevo Autor"
  - Iconos: Edit (editar), Delete (eliminar), View (ver detalle)

- [ ] `AuthorForm` - Formulario crear/editar
  - Campos: Nombre*, Email*, Bio, Expertise, Nationality
  - Validaciones del lado del cliente
  - Botones: Guardar, Cancelar

- [ ] `AuthorDetail` - Vista de detalle
  - Mostrar todos los datos del autor
  - Botones: Editar, Eliminar, Volver

**Página: Gestión de Publicaciones**
- [ ] `PublicationList` - Tabla filtrable
  - Columnas: ID, Título, Autor (nombre), Estado, Acciones
  - Filtros: Por estado, por autor
  - Botón: "Nueva Publicación"

- [ ] `PublicationForm` - Crear/editar publicación
  - Campos: Título*, Descripción, Autor (dropdown)*
  - Validar que el autor exista (fetch al abrir form)
  - Botones: Guardar, Cancelar

- [ ] `PublicationDetail` - Vista detallada
  - Mostrar título, descripción, autor completo
  - Mostrar estado actual
  - Botón: "Cambiar Estado"

- [ ] `StatusChanger` - Modal para cambiar estado
  - Selector: estados válidos según transiciones
  - Campo opcional: comentarios de revisión
  - Validar transiciones permitidas
  - Historial de cambios (si aplica)

**Componentes Comunes:**
- [ ] `Layout` - Header con navegación
  - Links a: Autores, Publicaciones, Home
- [ ] `Toast` - Notificaciones
  - Tipos: Success, Error, Warning, Info
  - Auto-close después de 3 segundos
- [ ] `LoadingSpinner` - Indicador de carga
- [ ] `ErrorBoundary` - Captura de errores

#### 3.3 Rutas (React Router)

```
/                         → Home
/authors                  → AuthorList
/authors/new              → AuthorForm (create)
/authors/:id              → AuthorDetail
/authors/:id/edit         → AuthorForm (edit)

/publications             → PublicationList
/publications/new         → PublicationForm (create)
/publications/:id         → PublicationDetail
/publications/:id/edit    → PublicationForm (edit)

/404                      → Not Found
```

#### 3.4 Servicios API (axios)

**`authorsService.js`:**
```javascript
export const getAuthors = (page = 1, limit = 10) => axios.get('/authors', { params: { page, limit } })
export const getAuthorById = (id) => axios.get(`/authors/${id}`)
export const createAuthor = (data) => axios.post('/authors', data)
export const updateAuthor = (id, data) => axios.put(`/authors/${id}`, data)
export const deleteAuthor = (id) => axios.delete(`/authors/${id}`)
```

**`publicationsService.js`:**
```javascript
export const getPublications = (filters) => axios.get('/publications', { params: filters })
export const getPublicationById = (id) => axios.get(`/publications/${id}`)
export const getPublicationDetail = (id) => axios.get(`/publications/${id}/detail`)
export const createPublication = (data) => axios.post('/publications', data)
export const updatePublicationStatus = (id, newStatus, comments) => 
  axios.patch(`/publications/${id}/status`, { newStatus, reviewComments: comments })
export const deletePublication = (id) => axios.delete(`/publications/${id}`)
```

---

### 4. MODELADO BPMN EN CAMUNDA

#### 4.1 Procesos a Modelar

**Proceso: "Editorial Publication Workflow"**

#### 4.2 Elementos Requeridos

**Participantes (Pools/Lanes):**
1. **Autor** - Inicia proceso, responde a cambios solicitados
2. **Editor** - Coordina el flujo
3. **Revisor** - Evalúa y aprueba/rechaza

#### 4.3 Flujo del Proceso

```
┌─────────────────────────────────────────────────────────┐
│  Autor        │  Editor         │  Revisor              │
├─────────────────────────────────────────────────────────┤
│               │                 │                       │
│  ◯ Inicio     │                 │                       │
│  │            │                 │                       │
│  ├─→ [Crear Borrador]           │                       │
│  │            │                 │                       │
│  │            ├─→ [Enviar a Revisión]                   │
│  │            │                 │                       │
│  │            │                 ├─→ [Revisar Texto] ▭   │
│  │            │                 │        (User Task)    │
│  │            │                 │                       │
│  │            │          ◇ [¿Aprobado?]                 │
│  │            │         ╱   │   ╲                       │
│  │            │        /    │    \                      │
│  │        [NO]│       /     │     \[SI]                 │
│  │◀───────────┴──────/      │      └─→ [Preparar Pub]   │
│  │   [Requiere       │      │          │                │
│  │    Cambios]       │      │          ├─→ [Publicar]   │
│  │                   │  [NO]│          │                │
│  │                   │      └─→ [Notificar Rechazo]     │
│  │                   │          │                       │
│  │ [Reescribir]      │          ◯ [Fin - Rechazado]    │
│  │ │                 │                                  │
│  └─→ ────────────────→ [Re-enviar] ──┐                  │
│                                       │                 │
│                    ┌──────────────────┘                  │
│                    │                                     │
│                    ├─→ [Completar Revisión] ────┐       │
│                       (depende de nuevas cambios)│       │
│                                                  │       │
│                              ┌───────────────────┘       │
│                              │                           │
│                              └─→ [Publicar]             │
│                                 │                       │
│                                 ◎ [Fin - Publicado]    │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

#### 4.4 Elementos BPMN Utilizados

- **1 Evento de Inicio** (`◯`) - "Crear Borrador"
- **2 Eventos de Fin** (`◎`) - "Publicado", "Rechazado"
- **1+ Gateway Exclusivo (XOR)** (`◇`) - Decisión de aprobación
- **5+ Tareas Humanas** (User Task) - Actividades manuales
- **Flujos de Secuencia** - Conectores entre elementos
- **Variables de Proceso:**
  - `publicationId: string`
  - `authorId: number`
  - `aprobado: boolean`
  - `requiereCambios: boolean`
  - `comentariosRevision: string`

#### 4.5 Escenarios de Simulación

**Escenario 1: Aprobación Directa**
```
Crear Borrador → Enviar Revisión → Revisar (✓ Aprobado) 
→ Preparar → Publicar → FIN(Publicado)
```

**Escenario 2: Rechazo**
```
Crear Borrador → Enviar Revisión → Revisar (✗ Rechazado)
→ Notificar → FIN(Rechazado)
```

**Escenario 3: Cambios Necesarios (Opcional)**
```
Crear Borrador → Enviar Revisión → Revisar (⚠ Requiere Cambios)
→ Devolver a Autor → Reescribir → Re-enviar Revisión
→ Revisar (✓ Aprobado) → Preparar → Publicar → FIN(Publicado)
```

---

### 5. DESPLIEGUE CON DOCKER

#### 5.1 Servicios en docker-compose.yml

```yaml
services:
  db-authors          # PostgreSQL para Autores
  db-publications     # PostgreSQL para Publicaciones
  authors-service     # Microservicio Autores (Node.js)
  publications-service # Microservicio Publicaciones (Node.js)
  frontend            # React (Vite/nginx)

networks:
  microservices       # Red compartida para comunicación

volumes:
  data-authors        # Persistencia BD Autores
  data-publications   # Persistencia BD Publicaciones
```

#### 5.2 Configuración Requerida

**Variables de Entorno:**
```
AUTHOR_SERVICE_URL=http://authors-service:3001
POSTGRES_PASSWORD=secure_password
NODE_ENV=production
DBPort=5432
```

**Puertos Expuestos:**
- `3000:3000` - Frontend React
- `3001:3001` - Authors Service API
- `3002:3002` - Publications Service API
- `5432:5432` - PostgreSQL Authors DB
- `5433:5433` - PostgreSQL Publications DB

**Healthchecks:**
```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:PORT/health"]
  interval: 10s
  timeout: 5s
  retries: 3
```

**Dependencias:**
```
publications-service 
  ├─ depends_on: db-publications (healthy)
  ├─ depends_on: authors-service (healthy)
authors-service
  ├─ depends_on: db-authors (healthy)
frontend
  └─ depends_on: authors-service, publications-service
```

#### 5.3 Comandos Docker

```bash
# Build y run
docker-compose up --build

# Ver logs
docker-compose logs -f authors-service

# Detener
docker-compose down

# Limpiar volúmenes (⚠️ Borra datos)
docker-compose down -v
```

---

### 6. PATRONES DE DISEÑO (Mínimo 3)

#### 6.1 Repository Pattern

**Ubicación:** `{servicio}/src/repositories/*.js`

**Descripción:**
Abstraer lógica de acceso a datos, permitir cambiar BD sin afectar servicio

**Ejemplo:**
```javascript
class AuthorRepository {
  async findById(id) { /* consulta BD */ }
  async create(data) { /* escribe BD */ }
}
```

**Beneficio:** Lógica del negocio desacoplada de BD

---

#### 6.2 Adapter Pattern

**Ubicación:** `publications-service/src/clients/AuthorsServiceClient.js`

**Descripción:**
Adaptar interfaz HTTP del servicio externo a uso interno

**Ejemplo:**
```javascript
class AuthorsServiceClient {
  async getAuthorById(id) {
    // Maneja HTTP, errores, timeouts
    // Devuelve objeto JavaScript limpio
  }
}
```

**Beneficio:** Aislar cambios en servicio externo

---

#### 6.3 Strategy Pattern

**Ubicación:** 
- `authors-service/src/strategies/ValidationStrategy.js`
- `publications-service/src/strategies/StatusTransitionStrategy.js`

**Descripción:**
Encapsular algoritmos intercambiables (validaciones, transiciones)

**Ejemplo:**
```javascript
class StatusTransitionStrategy {
  canTransition(current, target) {
    const allowed = { DRAFT: ['IN_REVIEW'], ... };
    return allowed[current]?.includes(target);
  }
}
```

**Beneficio:** Agregar reglas sin modificar código existente

---

#### 6.4 (Opcional) Factory Pattern

**Ubicación:** `{servicio}/src/factories/DTOFactory.js`

**Descripción:**
Crear objetos DTO de manera consistente

---

### 7. PRINCIPIOS SOLID

#### 7.1 Single Responsibility (SRP)

✅ **Controllers** - Solo manejar HTTP  
✅ **Services** - Solo lógica de negocio  
✅ **Repositories** - Solo acceso a datos  
✅ **Entities** - Solo reglas del dominio  

#### 7.2 Open/Closed (OCP)

✅ Extensible para nuevas validaciones sin modificar código  
✅ Clases base abstractas para extensión

#### 7.3 Liskov Substitution (LSP)

✅ `Author extends Person` respeta contrato de base  
✅ `Publication extends Content` respeta contrato de base

#### 7.4 Interface Segregation (ISP)

✅ DTOs específicos para cada operación  
✅ Interfaces mínimas y focalizadas

#### 7.5 Dependency Inversion (DIP)

✅ Inyección de dependencias en constructores  
✅ Servicios dependen de abstracciones (Repositories) no implementaciones

---

## ✅ CHECKLIST DE ENTREGABLES

### Código Fuente
- [ ] Microservicio Authors (Node.js + Express + Sequelize)
  - [ ] Clases Person (abstracta) y Author (derivada)
  - [ ] AuthorRepository (Repository Pattern)
  - [ ] AuthorService con lógica de negocio
  - [ ] AuthorController con manejo HTTP
  - [ ] DTOs para entrada/salida
  - [ ] Todos los endpoints funcionando
  - [ ] Tests unitarios
  
- [ ] Microservicio Publications (Node.js + Express + Sequelize)
  - [ ] Clases Content (abstracta) y Publication (derivada)
  - [ ] PublicationRepository
  - [ ] PublicationService con validaciones
  - [ ] AuthorsServiceClient (Adapter Pattern)
  - [ ] Transiciones de estado (Strategy Pattern)
  - [ ] PublicationController con manejo HTTP
  - [ ] DTOs enriquecidos
  - [ ] Todos los endpoints funcionando
  - [ ] Tests unitarios
  
- [ ] Frontend React
  - [ ] Componentes de Autores (List, Form, Detail)
  - [ ] Componentes de Publicaciones (List, Form, Detail, StatusChanger)
  - [ ] Servicios API (authorsService, publicationsService)
  - [ ] Navegación con React Router
  - [ ] Notificaciones (Toast)
  - [ ] Manejo de errores y loading
  - [ ] Responsive design
  - [ ] Tests de componentes

### Despliegue
- [ ] Dockerfile para Authors Service
- [ ] Dockerfile para Publications Service
- [ ] Dockerfile para Frontend (multi-stage)
- [ ] docker-compose.yml con todos servicios
- [ ] .env configurado
- [ ] .gitignore apropiado
- [ ] Verificación: `docker-compose up --build` sin errores

### BPMN
- [ ] Diagrama en Camunda Modeler (`bpmn/editorial_process.bpmn`)
- [ ] 3+ Participantes (Autor, Editor, Revisor)
- [ ] 1 evento inicio, 2 eventos fin
- [ ] Flujo con transiciones validadas
- [ ] Gateway XOR para decisión aprobación
- [ ] Variables de simulación
- [ ] Token Simulation ejecutada (3 escenarios)
- [ ] Capturas de pantalla de simulaciones

### Documentación
- [ ] README.md en raíz del proyecto
  - [ ] Descripción general
  - [ ] Requisitos previos
  - [ ] Instrucciones instalación
  - [ ] Cómo correr con Docker
  - [ ] Endpoints documentados
- [ ] Diagrama de arquitectura (UML/PlantUML)
- [ ] Documento explicando patrones de diseño
  - [ ] Dónde se aplica cada patrón
  - [ ] Por qué se eligió
  - [ ] Ubicación en código
- [ ] Documento explicando principios SOLID aplicados
- [ ] Estructura de carpetas explicada

### Testing
- [ ] Tests unitarios para AuthorService
- [ ] Tests unitarios para PublicationService
- [ ] Tests de integración para APIs
- [ ] Tests de componentes React
- [ ] Cobertura >70%

---

## 🚀 CÓMO USAR ESTA DOCUMENTACIÓN

1. **Primero:** Lee este documento (Especificaciones)
2. **Luego:** Sigue `INICIO_RAPIDO.md` para configuración inicial
3. **Durante:** Consulta `PLAN_EJECUCION.md` paso a paso
4. **Referencia:** `ARQUITECTURA.md` para detalles técnicos y patrones

---

## 📞 NOTAS IMPORTANTES

- ✅ Microservicios **no deben** ser acoplados innecesariamente
- ✅ Publications Service **sí puede** llamar Authors Service (dependencia controlada)
- ✅ Authors Service **no puede** llamar Publications Service (evitar ciclos)
- ✅ Usar ORM (Sequelize) para acceso a datos
- ✅ Validar entrada en Controllers y Services
- ✅ Manejo consistente de errores
- ✅ DTOs para separar modelos y respuestas HTTP
- ✅ Logs informativos para debugging

---

**Versión:** 1.0  
**Fecha:** 4 de febrero de 2026  
**Autor:** GitHub Copilot  
**Estado:** Aprobado para desarrollo
