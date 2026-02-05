# 🏗️ ARQUITECTURA DEL PROYECTO

## Diagrama General

```
┌─────────────────────────────────────────────────────────────────┐
│                       CAPA DE PRESENTACIÓN                      │
│                                                                   │
│                   📱 Frontend React (Vite)                       │
│           http://localhost:3000 (nginx en producción)           │
│                                                                   │
│  ┌──────────────┐      ┌──────────────┐     ┌──────────────┐   │
│  │ AuthorList   │      │PublicationList│    │StatusChanger │   │
│  ├──────────────┤      ├──────────────┤     ├──────────────┤   │
│  │ AuthorForm   │      │PublForm      │     │Details       │   │
│  └──────────────┘      └──────────────┘     └──────────────┘   │
│        │                      │                     │            │
└────────┼──────────────────────┼─────────────────────┼────────────┘
         │                      │                     │
         └──────────────────────┴─────────────────────┘
                        │
                  (HTTP REST)
                        │
         ┌──────────────┴──────────────┐
         │                             │
    ┌────▼──────────┐      ┌──────────▼──────┐
    │ Authors API   │      │Publications API │
    │  :3001        │      │     :3002       │
    └────┬──────────┘      └────────┬────────┘
         │                          │
    ┌────┴──────────────────────────┼─────────────────┐
    │   CAPA DE APLICACIÓN          │                 │
    │  (Node.js + Express)          │                 │
    │                               │                 │
    │  ┌────────────────────┐       │   ┌───────────────────────┐
    │  │ AuthorController   │       │   │PublicationController  │
    │  │                    │       │   │                       │
    │  ├────────────────────┤       │   ├───────────────────────┤
    │  │ AuthorService      │       │   │PublicationService     │
    │  │ - validate()       │       │   │ - createWithValidation│
    │  │ - CRUD ops         │       │   │ - updateStatus()      │
    │  └────────────────────┘       │   │ - getWithAuthorData() │
    │                               │   └───────────────────────┘
    │  ┌────────────────────┐       │   ┌───────────────────────┐
    │  │AuthorRepository    │       │   │PublicationRepository │
    │  │(Patrón Repository) │       │   │                      │
    │  └────────────────────┘       │   │AuthorsServiceClient  │
    │                               │   │(Patrón Adapter)     │
    │                               │   └───────────────────────┘
    │                               │
    └───────────────────────────────┴─────────────────┐
        │                           │                 │
        └───────────────┬───────────┴──────────────────┘
                        │
                    (ORM - Sequelize)
                        │
         ┌──────────────┴──────────────┐
         │                             │
    ┌────▼──────────┐      ┌──────────▼──────┐
    │ PostgreSQL    │      │   PostgreSQL    │
    │ authors_db   │      │publications_db │
    │   :5432       │      │     :5433       │
    └───────────────┘      └─────────────────┘
         │                        │
    ┌────▼────────────────────────▼────┐
    │   CAPA DE PERSISTENCIA           │
    │   (Docker Volumes)               │
    │   - data-authors                 │
    │   - data-publications            │
    └──────────────────────────────────┘
```

---

## Componentes Principales

### 1. MICROSERVICIO DE AUTORES

**Stack Tecnológico:**
- Runtime: Node.js 18+
- Framework: Express.js
- ORM: Sequelize
- BD: PostgreSQL
- Validación: express-validator, class-validator
- Testing: Jest

**Estructura de Clases:**

```javascript
// Jerarquía de Herencia
┌─────────────────┐
│   abstract      │
│   Person        │ (Clase Abstracta Base)
├─────────────────┤
│ - id: number    │
│ - name: string  │
│ - email: string │
│ - birthDate     │
│ - createdAt     │
└────────┬────────┘
         │ (extends)
         │
    ┌────▼──────────┐
    │   Author      │ (Clase Concreta Derivada)
    ├───────────────┤
    │ - bio         │
    │ - expertise   │
    │ - nationality │
    │ + validate()  │
    │ + getFullInfo()
    └───────────────┘

// Capas de Aplicación
┌─────────────────────────────────┐
│    AuthorController (HTTP)      │ ← Solicitudes HTTP
├─────────────────────────────────┤
│    AuthorService (Negocio)      │ ← Lógica de negocio
├─────────────────────────────────┤
│   AuthorRepository (Datos)      │ ← Acceso a BD
├─────────────────────────────────┤
│    Sequelize Model (ORM)        │ ← Mapeo de BD
└─────────────────────────────────┘
```

### 2. MICROSERVICIO DE PUBLICACIONES

**Stack Tecnológico:**
- Runtime: Node.js 18+
- Framework: Express.js
- ORM: Sequelize
- BD: PostgreSQL
- HTTP Client: axios (para llamadas a Authors Service)
- Validación: express-validator, class-validator
- Testing: Jest

**Estructura de Clases:**

```javascript
// Jerarquía de Herencia
┌──────────────────┐
│    abstract      │
│    Content       │ (Clase Base Abstracta)
├──────────────────┤
│ - id: number     │
│ - title: string  │
│ - description    │
│ - createdAt      │
│ - updatedAt      │
└────────┬─────────┘
         │ (extends)
         │
    ┌────▼────────────────┐
    │  Publication         │ (Clase Derivada)
    ├─────────────────────┤
    │ - authorId: number  │
    │ - status: enum      │ ← DRAFT | IN_REVIEW | APPROVED | PUBLISHED | REJECTED
    │ - reviewComments    │
    │ - publishedDate     │
    │ + updateStatus()    │
    │ + canTransitionTo() │
    │ + enrich()          │ ← Llena datos del autor
    └─────────────────────┘

// Comunicación Inter-Servicios
Publications Service
    │
    ├─ AuthorsServiceClient
    │  (Patrón Adapter)
    │  └─ getAuthorById(id)
    │     ├─ HTTP GET: http://authors-service:3001/authors/{id}
    │     └─ Manejo de errores (timeout, no encontrado, etc.)
    │
    └─ PublicationService
       ├─ createPublication()
       │  └─ Valida que autor exista ANTES de crear
       └─ getPublicationWithAuthor()
          └─ Enriquece respuesta con datos del autor

// Capas de Aplicación
┌──────────────────────────────────┐
│ PublicationController (HTTP)     │ ← Solicitudes HTTP
├──────────────────────────────────┤
│ PublicationService (Negocio)     │ ← Lógica editorial
│ + Transiciones de estado         │
│ + Validaciones complejas         │
├──────────────────────────────────┤
│ PublicationRepository (Datos)    │ ← Acceso a BD
│ + AuthorsServiceClient           │ ← Llamadas a Authors Service
├──────────────────────────────────┤
│ Sequelize Model (ORM)            │ ← Mapeo de BD
└──────────────────────────────────┘
```

### 3. FRONTEND REACT

**Stack Tecnológico:**
- Framework: React 18
- Build Tool: Vite
- Routing: react-router-dom
- HTTP Client: axios
- Testing: Vitest
- UI: CSS modules + responsive design

**Estructura de Componentes:**

```
src/
├── components/
│   ├── Authors/
│   │   ├── AuthorList.jsx      ← Listado paginable
│   │   ├── AuthorForm.jsx      ← Crear/Editar autor
│   │   └── AuthorDetail.jsx    ← Vista detallada
│   ├── Publications/
│   │   ├── PublicationList.jsx ← Listado con filtros
│   │   ├── PublicationForm.jsx ← Crear nuevo
│   │   ├── PublicationDetail.jsx ← Detalle con autor
│   │   └── StatusChanger.jsx   ← Cambiar estado (DRAFT→PUBLISHED)
│   ├── Common/
│   │   ├── Layout.jsx          ← Navbar, estructura
│   │   ├── Toast.jsx           ← Notificaciones
│   │   └── Loading.jsx         ← Indicador de carga
│   └── Router/
│       └── AppRouter.jsx       ← Definición de rutas
├── services/
│   ├── api.js                  ← Configuración axios
│   ├── authorsService.js       ← Llamadas a /authors
│   └── publicationsService.js  ← Llamadas a /publications
├── hooks/
│   ├── useApi.js               ← Hook para fetching
│   └── usePagination.js        ← Lógica de paginación
├── styles/
│   └── index.css               ← Estilos globales
└── App.jsx
```

**Flujos de Usuario:**

```
CREAR AUTOR:
User → AuthorForm → authorsService.create() 
→ POST /authors → AuthorService.create() 
→ AuthorRepository.create() → Person.validate() → Author entity
→ PostgreSQL → Toast("Autor creado") → AuthorList

CREAR PUBLICACIÓN:
User → PublicationForm (selecciona autor) 
→ publicationsService.create() 
→ POST /publications 
→ PublicationService.createWithValidation() 
→ AuthorsServiceClient.getAuthorById() 
→ HTTP GET /authors/{id} → Authors Service
→ [Error: No existe] o [Válido] 
→ PublicationRepository.create() → Content.validate() → Publication
→ PostgreSQL → Toast → PublicationList

CAMBIAR ESTADO:
User → PublicationDetail → StatusChanger 
→ PATCH /publications/{id}/status
→ PublicationService.updateStatus()
→ Valida transición permitida (Strategy Pattern)
→ PublicationRepository.updateStatus()
→ PostgreSQL → Toast → Refresh detail

LISTAR PUBLICACIONES CON ENRIQUECIMIENTO:
User → PublicationList

GET /publications/1 (por ejemplo)
→ PublicationService.getPublicationWithAuthor()
→ PublicationRepository.findById() ← BD Publication
→ AuthorsServiceClient.getAuthorById(publication.authorId)
→ HTTP GET /authors/1 ← Authors Service
→ Combina datos → PublicationWithAuthorDTO
→ Frontend renderiza con nombre del autor
```

---

## Patrones de Diseño Implementados

### 1. Repository Pattern

**Localización:**
- `servidor/authors-service/src/repositories/AuthorRepository.js`
- `servidor/publications-service/src/repositories/PublicationRepository.js`

**Propósito:** Abstraer lógica de acceso a datos

```javascript
class AuthorRepository {
  async create(data) { /* ... */ }
  async findById(id) { /* ... */ }
  async findAll(options) { /* ... */ }
  async update(id, data) { /* ... */ }
  async delete(id) { /* ... */ }
}

// Uso en Service:
class AuthorService {
  constructor(authorRepository) {
    this.repo = authorRepository;
  }
  
  async registerAuthor(data) {
    // Lógica de negocio
    return this.repo.create(data); // Delega acceso a datos
  }
}
```

**Beneficio:** Cambiar de BD sin tocar lógica de negocio

---

### 2. Adapter Pattern

**Localización:**
- `servidor/publications-service/src/clients/AuthorsServiceClient.js`

**Propósito:** Adaptar interfaz HTTP/REST del servicio externo

```javascript
class AuthorsServiceClient {
  constructor(baseUrl, httpClient) {
    this.baseUrl = baseUrl;
    this.httpClient = httpClient;
  }
  
  async getAuthorById(id) {
    try {
      const response = await this.httpClient.get(
        `${this.baseUrl}/authors/${id}`
      );
      return response.data; // Adaptado a formato interno
    } catch (error) {
      if (error.response?.status === 404) {
        throw new AuthorNotFoundError(id);
      }
      throw new ExternalServiceError(error);
    }
  }
}

// Uso en Service:
class PublicationService {
  async createPublication(dataWithAuthorId) {
    const author = await this.authorsClient.getAuthorById(
      dataWithAuthorId.authorId
    );
    // Ahora en el scope local, sin depender de detalles HTTP
  }
}
```

**Beneficio:** Aislar cambios en servicio externo

---

### 3. Strategy Pattern

**Localización:**
- `servidor/authors-service/src/strategies/ValidationStrategy.js`
- `servidor/publications-service/src/strategies/StatusTransitionStrategy.js`

**Propósito:** Encapsular algoritmos intercambiables

```javascript
// Ejemplo 1: Validación
class AuthorValidationStrategy {
  validate(author) {
    if (!author.name || author.name.length < 2) {
      throw new ValidationError("Name too short");
    }
    if (!this.isValidEmail(author.email)) {
      throw new ValidationError("Invalid email");
    }
  }
}

// Ejemplo 2: Transiciones de estado
class PublicationStatusStrategy {
  canTransition(currentStatus, targetStatus) {
    const allowedTransitions = {
      DRAFT: ['IN_REVIEW'],
      IN_REVIEW: ['APPROVED', 'REJECTED', 'DRAFT'],
      APPROVED: ['PUBLISHED'],
      PUBLISHED: ['REJECTED'],
      REJECTED: ['DRAFT']
    };
    return allowedTransitions[currentStatus]?.includes(targetStatus) ?? false;
  }
}

// Uso:
class PublicationService {
  async updateStatus(id, newStatus) {
    const pub = await this.repo.findById(id);
    if (!this.statusStrategy.canTransition(pub.status, newStatus)) {
      throw new InvalidStatusTransitionError(pub.status, newStatus);
    }
    return this.repo.updateStatus(id, newStatus);
  }
}
```

**Beneficio:** Fácil agregar/cambiar reglas de negocio

---

### 4. Factory Pattern (Opcional)

**Localización:**
- `servidor/authors-service/src/factories/AuthorDTOFactory.js`

**Propósito:** Crear objetos DTOs consistentemente

```javascript
class AuthorDTOFactory {
  static createFromEntity(authorEntity) {
    return {
      id: authorEntity.id,
      name: authorEntity.name,
      email: authorEntity.email,
      bio: authorEntity.bio,
      expertise: authorEntity.expertise,
      createdAt: authorEntity.createdAt
    };
  }
}

// Uso:
const authorDTO = AuthorDTOFactory.createFromEntity(authorEntity);
```

---

### 5. Dependency Injection (DI)

**Localización:** Constructores en Services y Repositories

**Propósito:** Desacoplar dependencias

```javascript
// En lugar de:
class AuthorService {
  constructor() {
    this.repo = new AuthorRepository(); // Acoplado
  }
}

// Hacer:
class AuthorService {
  constructor(authorRepository) {
    this.repo = authorRepository; // Inyectado (desacoplado)
  }
}

// Instanciación:
const repo = new AuthorRepository();
const service = new AuthorService(repo);
```

---

## Principios SOLID

### S - Single Responsibility Principle

Cada clase tiene **una única razón de cambio:**

- `AuthorController` → Manejar HTTP
- `AuthorService` → Lógica de negocio
- `AuthorRepository` → Acceso a BD
- `Author` (Entidad) → Reglas del dominio

### O - Open/Closed Principle

Abierto a **extensión** (agregar validaciones), cerrado a **modificación** (código existente):

```javascript
// En lugar de agregar IF en AuthorService:
if (type === 'premium') { /* validación X */ }
if (type === 'basic') { /* validación Y */ }

// Extender con Strategy:
class PremiumAuthorValidation extends BaseValidation { validate() { /* X */ } }
class BasicAuthorValidation extends BaseValidation { validate() { /* Y */ } }
```

### L - Liskov Substitution Principle

Clases derivadas respetan el contrato de la base:

```javascript
// Author (derivada) puede reemplazar a Person (base)
abstract class Person {
  abstract validate();
}

class Author extends Person {
  validate() { /* implementación para Author */ }
  // Cualquier método que espere Person puede recibir Author
}
```

### I - Interface Segregation Principle

Interfaces **específicas**, no genéricas:

```javascript
// Mal: Interfaz genérica
interface IEntity {
  create(); read(); update(); delete();
}

// Bien: Interfaces segregadas
interface ICreatable { create(); }
interface IReadable { read(); }
interface IUpdatable { update(); }
interface IDeletable { delete(); }

class ReadOnlyService implements IReadable { /* */ }
```

### D - Dependency Inversion Principle

Depender de **abstracciones**, no de implementaciones:

```javascript
// Mal: 
class PublicationService {
  constructor() {
    this.client = new PostgreSQLRepository(); // Concreto
  }
}

// Bien:
class PublicationService {
  constructor(repository) { // Abstracción (interfaz)
    this.repository = repository;
  }
}
```

---

## Gestión de Errores

### Errores Personalizados en Services:

```javascript
class AuthorNotFoundError extends Error {
  constructor(id) {
    super(`Author with id ${id} not found`);
    this.statusCode = 404;
  }
}

class ValidationError extends Error {
  constructor(message) {
    super(message);
    this.statusCode = 400;
  }
}

class ExternalServiceError extends Error {
  constructor(message) {
    super(`External service error: ${message}`);
    this.statusCode = 503;
  }
}
```

### Middleware de Manejo en Controllers:

```javascript
app.use((err, req, res, next) => {
  if (err instanceof ValidationError) {
    return res.status(err.statusCode).json({
      error: err.message,
      type: 'VALIDATION_ERROR'
    });
  }
  
  if (err instanceof AuthorNotFoundError) {
    return res.status(err.statusCode).json({
      error: err.message,
      type: 'NOT_FOUND'
    });
  }
  
  // Error genérico
  res.status(500).json({
    error: 'Internal server error',
    type: 'INTERNAL_ERROR'
  });
});
```

---

## DTOs (Data Transfer Objects)

**Propósito:** Separar modelos internos de respuestas HTTP

```javascript
// Entrada
class CreateAuthorDTO {
  name; email; bio; expertise; nationality;
  
  validate() {
    if (!this.name) throw new Error("Name required");
    if (!this.isValidEmail(this.email)) throw new Error("Invalid email");
  }
}

// Salida
class AuthorResponseDTO {
  id; name; email; bio; expertise; createdAt;
  
  constructor(authorEntity) {
    this.id = authorEntity.id;
    this.name = authorEntity.name;
    // NO expone campos internos
  }
}

// Enriquecida (Publication con datos de Author)
class PublicationWithAuthorDTO {
  id; title; status; author; createdAt;
  
  constructor(publicationEntity, authorData) {
    this.id = publicationEntity.id;
    this.title = publicationEntity.title;
    this.status = publicationEntity.status;
    this.author = {
      id: authorData.id,
      name: authorData.name,
      email: authorData.email
    };
  }
}
```

---

## Comunicación Inter-Servicios

```
Flujo sincrónico (REST):

Publications Service
    │
    ├─ [Crear publicación]
    │   HTTP GET → http://authors-service:3001/authors/5
    │   ├─ Response 200: { id: 5, name: "John", ... } ✓
    │   ├─ Response 404: Author not found ✗
    │   └─ Timeout/Error 503: Authors Service down ✗
    │
    └─ [Manejo de errores]
        ├─ Retry logic (3 intentos con backoff)
        ├─ Circuit breaker (si continúa fallando)
        └─ Fallback (ej: datos cached)
```

---

## Volúmenes Docker para Persistencia

```yaml
volumes:
  data-authors:         # Datos de PostgreSQL authors_db
    driver: local
  data-publications:    # Datos de PostgreSQL publications_db
    driver: local
```

Los datos persisten incluso si se detienen los contenedores:
```bash
docker-compose down  # Servicios se detienen, BD está en volume
docker-compose up    # BD se reconstruye desde volume existente
```

---

## Resumen de Componentes

| Componente | Tipo | Responsabilidad |
|-----------|------|-----------------|
| `Person` / `Author` | Entity | Reglas del dominio |
| `AuthorService` | Service | Lógica de negocio |
| `AuthorRepository` | Repository | Acceso a datos |
| `AuthorController` | Controller | HTTP handling |
| `AuthorDTO` | DTO | Transfer de datos |
| `ValidationStrategy` | Strategy | Encapsular validaciones |
| `AuthorsServiceClient` | Adapter | Inter-service comm |
| `ErrorHandler` | Middleware | Manejo de errores |

---

**Esta arquitectura garantiza:**
✅ Escalabilidad horizontal (microservicios)
✅ Mantenibilidad (claras responsabilidades)
✅ Testabilidad (inyección de dependencias)
✅ Resiliencia (manejo de errores)
✅ Flexibilidad (patrones de diseño)
