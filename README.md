# 📚 Sistema de Gestión de Autores y Publicaciones - Documentación Completa

> **Proyecto Integrador:** Arquitectura de Microservicios con Modelado BPMN y Frontend React

## 🎯 Visión General

Este proyecto desarrolla una **solución moderna y escalable** para que una editorial digital gestione autores y publicaciones de manera desacoplada. Implementa dos microservicios independientes que se comunican vía REST API, un frontend React intuitivo, y un flujo editorial modelado en BPMN con Camunda.

### Tecnologías Stack
- **Backend:** Node.js + Express + Sequelize
- **BD:** PostgreSQL (duplicada por servicio)
- **Frontend:** React + Vite + React Router
- **Despliegue:** Docker Compose
- **Modelado:** Camunda Modeler (BPMN 2.0)

---

## 📁 Estructura del Workspace

```
Tarea 3/
├── 📋 README.md                    ← Este archivo
├── 📋 PLAN_EJECUCION.md             ← Índice de 49 pasos
├── 📋 ESPECIFICACIONES.md           ← Reqs técnicas detalladas
├── 📋 ARQUITECTURA.md               ← Análisis de diseño
├── 📋 INICIO_RAPIDO.md              ← Guía rápida implementación
│
├── 📁 servidor/                     ← Backend Microservicios
│   ├── 📁 authors-service/
│   │   ├── src/
│   │   │   ├── controllers/         ← Manejo HTTP
│   │   │   ├── services/            ← Lógica de negocio
│   │   │   ├── repositories/        ← Acceso a datos
│   │   │   ├── models/
│   │   │   │   ├── entities/        ← Clases (Person, Author)
│   │   │   │   └── Author.sequelize.js
│   │   │   ├── routes/              ← Definición de rutas
│   │   │   ├── middleware/          ← Validación, errores
│   │   │   ├── config/
│   │   │   │   └── database.js
│   │   │   └── server.js            ← Punto de entrada
│   │   ├── package.json
│   │   ├── .env                     ← Variables local
│   │   ├── .gitignore
│   │   ├── Dockerfile               ← Para Docker
│   │   └── README.md
│   │
│   ├── 📁 publications-service/
│   │   ├── src/
│   │   │   ├── controllers/
│   │   │   ├── services/
│   │   │   ├── repositories/
│   │   │   ├── models/
│   │   │   │   ├── entities/        ← Clases (Content, Publication)
│   │   │   │   └── Publication.sequelize.js
│   │   │   ├── clients/
│   │   │   │   └── AuthorsServiceClient.js  ← Adapter Pattern
│   │   │   ├── strategies/
│   │   │   │   └── StatusTransitionStrategy.js ← Strategy Pattern
│   │   │   ├── routes/
│   │   │   ├── middleware/
│   │   │   ├── config/
│   │   │   └── server.js
│   │   ├── package.json
│   │   ├── .env
│   │   ├── .gitignore
│   │   ├── Dockerfile
│   │   └── README.md
│   │
│   ├── docker-compose.yml           ← Orquestación completa
│   ├── .env.example                 ← Template variables
│   └── .gitignore
│
├── 📁 cliente/
│   └── 📁 frontend/                 ← Aplicación React
│       ├── src/
│       │   ├── components/
│       │   │   ├── Authors/
│       │   │   │   ├── AuthorList.jsx
│       │   │   │   ├── AuthorForm.jsx
│       │   │   │   └── AuthorDetail.jsx
│       │   │   ├── Publications/
│       │   │   │   ├── PublicationList.jsx
│       │   │   │   ├── PublicationForm.jsx
│       │   │   │   ├── PublicationDetail.jsx
│       │   │   │   └── StatusChanger.jsx
│       │   │   └── Common/
│       │   │       ├── Layout.jsx
│       │   │       ├── Toast.jsx
│       │   │       └── LoadingSpinner.jsx
│       │   ├── services/
│       │   │   ├── api.js            ← Configuración axios
│       │   │   ├── authorsService.js
│       │   │   └── publicationsService.js
│       │   ├── hooks/
│       │   │   ├── useApi.js
│       │   │   └── usePagination.js
│       │   ├── styles/
│       │   ├── App.jsx
│       │   └── main.jsx
│       ├── .env
│       ├── .gitignore
│       ├── package.json
│       ├── vite.config.js
│       ├── Dockerfile                ← Multi-stage build
│       └── index.html
│
├── 📁 bpmn/
│   ├── editorial_process.bpmn        ← Archivo Camunda
│   ├── simulaciones.png              ← Screenshots
│   └── README.md                     ← Explicación proceso
│
└── 📁 docs/                          ← Documentación adicional
    ├── PATRONES_DISEÑO.md
    ├── PRINCIPIOS_SOLID.md
    ├── DIAGRAMA_ARQUITECTURA.md
    └── TROUBLESHOOTING.md
```

---

## 📖 Documentos Principales Explicados

### 1. **ESPECIFICACIONES.md** (Leer PRIMERO)
- Resumen ejecutivo del proyecto
- Requisitos técnicos detallados
- Endpoints API con ejemplos JSON
- Estados editoriales y transiciones
- Elementos BPMN requeridos
- Checklist de entregables

**Cuándo leer:** Al inicio, para entender completamente qué se debe construir

### 2. **ARQUITECTURA.md** (Referencia técnica)
- Diagrama de componentes
- Jerarquía de clases (Person → Author, Content → Publication)
- Patrones de diseño implementados
  - Repository Pattern
  - Adapter Pattern
  - Strategy Pattern
  - Factory Pattern
  - Dependency Injection
- Principios SOLID con ejemplos
- Manejo de errores y DTOs

**Cuándo leer:** Durante desarrollo, para entender cómo estructurar código

### 3. **PLAN_EJECUCION.md** (Guía paso a paso)
- 49 pasos organizados en 9 fases
- Cada paso indica exactamente qué hacer
- Checklist para validar progreso
- Resumen de entregas

**Cuándo leer:** Seguir durante implementación, paso por paso

### 4. **INICIO_RAPIDO.md** (Get started en 1 hora)
- Requisitos previos con links
- Setup de cada microservicio
- Creación de modelos y controladores
- Docker Compose básico
- Comandos de prueba

**Cuándo leer:** Para configuración inicial rápida

---

## 🚀 Cómo Empezar (5 Pasos)

### Paso 1: Revisar Documentación (30 min)
```bash
# En este orden:
1. Leer ESPECIFICACIONES.md (entiende qué)
2. Leer ARQUITECTURA.md (entiende cómo)
3. Leer INICIO_RAPIDO.md (entiende por dónde)
```

### Paso 2: Instalar Requisitos Previos (15 min)
```bash
# Verificar instalación
node --version      # v18+
npm --version
docker --version
docker-compose --version

# Opcional (para BPMN)
# Descargar Camunda Modeler desde https://camunda.com/download/modeler/
```

### Paso 3: Setup Inicial (20 min)
```bash
# Authors Service
cd servidor/authors-service
npm init -y
npm install express dotenv cors pg sequelize
npm install --save-dev nodemon

# Publications Service
cd servidor/publications-service
npm init -y
npm install express dotenv cors pg sequelize axios
npm install --save-dev nodemon

# Frontend
cd cliente
npm create vite@latest frontend -- --template react
cd frontend && npm install
```

### Paso 4: Crear Estructura (15 min)
```bash
# Seguir INICIO_RAPIDO.md secciones 1-4
# - Crear carpetas src/
# - Crear archivos .env
# - Crear server.js básico
# - Crear docker-compose.yml
```

### Paso 5: Desarrollo Iterativo (Semanas siguientes)
```bash
# Seguir PLAN_EJECUCION.md paso por paso
# Implementar modelos → Services → Controllers → Tests
# Luego: Frontend React
# Finalmente: BPMN y documentación
```

---

## 📊 Mapeo de Requisitos vs Documentos

| Requisito | ESPECIFICACIONES | ARQUITECTURA | PLAN_EJECUCION | INICIO_RAPIDO |
|-----------|------------------|--------------|-----------------|---------------|
| Clases abstractas | ✅ Sección 1.2, 2.2 | ✅ Código completo | ✅ Paso 6, 13 | ✅ Paso 5-6 |
| API Endpoints | ✅ Tablas endpoints | ✅ Ejemplos JSON | ✅ Paso 11, 19 | ✅ Paso 5 |
| ORM (Sequelize) | ✅ Sección 1.3 | ✅ Diagrama | ✅ Paso 7, 14 | ✅ Paso 5 |
| Repository Pattern | ✅ Sección 6.1 | ✅ Ejemplos código | ✅ Paso 8, 16 | ✅ Paso 5 |
| Adapter Pattern | ✅ Sección 6.2 | ✅ Implementación | ✅ Paso 15 | - |
| Strategy Pattern | ✅ Sección 6.3 | ✅ Ejemplos | ✅ Paso 9, 17 | - |
| React Frontend | ✅ Sección 3 | ✅ Componentes | ✅ Paso 21-27 | ✅ Paso 3 |
| Docker Compose | ✅ Sección 5 | ✅ YAML ejemplo | ✅ Paso 35 | ✅ Paso 4 |
| BPMN Modelado | ✅ Sección 4 | - | ✅ Paso 28-31 | - |
| SOLID Principles | ✅ Sección 7 | ✅ Explicado | ✅ Paso 42 | - |
| Testing | ✅ Checklist | - | ✅ Paso 12, 20, 46-48 | - |

---

## 🔑 Conceptos Clave

### Microservicios
- 2 servicios independientes: Autores y Publicaciones
- Cada uno con su propia BD (PostgreSQL)
- Se comunican vía HTTP REST (síncrono)
- Publications depende de Authors

### Clases y Herencia
```
Person (abstracta) → Author (concreta)
Content (abstracta) → Publication (concreta)
```

### Patrones de Diseño
1. **Repository** - Abstrae acceso a datos
2. **Adapter** - Comunica con servicio externo
3. **Strategy** - Encapsula validaciones
4. **Factory** (opcional) - Crea DTOs
5. **Dependency Injection** - Desacopla dependencias

### Principios SOLID
- **S**ingle: Una responsabilidad por clase
- **O**pen: Abierto a extensión, cerrado a modificación
- **L**iskov: Clases derivadas respetan base
- **I**nterface: Segregación de responsabilidades
- **D**ependency: Inyección de dependencias

### BPMN
- Modelado visual del proceso editorial
- Roles: Autor, Editor, Revisor
- Estados de publicación: DRAFT → PUBLISHED o REJECTED
- Simulación con Token Simulation

---

## 🛠️ Herramientas Recomendadas

### Para Desarrollo
- **VS Code** - Editor (recomendado)
- **Node.js 18+** - Runtime
- **PostgreSQL** - Para desarrollo local (opcional)
- **Git** - Control de versiones

### Para Testing
- **Postman** o **Insomnia** - Probar APIs
- **Jest** - Tests unitarios
- **React Testing Library** - Tests componentes

### Para BPMN
- **Camunda Modeler** - Diseñar procesos
- **Token Simulation** - Simular flujos

### Para Documentación
- **PlantUML** o **Draw.io** - Diagramas
- **Markdown** - Documentación código

---

## ✅ Checklist de Verificación

Antes de considerar el proyecto completo:

**Backend:**
- [ ] Authors Service compilando sin errores
- [ ] Publications Service compilando sin errores
- [ ] Endpoints testables con Postman
- [ ] BD sincronizadas con Sequelize
- [ ] Inter-service communication funcionando
- [ ] Error handling implementado
- [ ] Tests pasando (>70% cobertura)

**Frontend:**
- [ ] React proyecto corriendo sin errores
- [ ] Componentes renderizando correctamente
- [ ] Conecta con ambos microservicios
- [ ] CRUD completo funciona
- [ ] Notificaciones Toast visibles
- [ ] Responsive en mobile

**BPMN:**
- [ ] Diagrama dibujado en Camunda
- [ ] 3 escenarios simulados
- [ ] Capturas de pantalla guardadas
- [ ] Documentado en `bpmn/README.md`

**Despliegue:**
- [ ] docker-compose.yml completo
- [ ] Dockerfiles para cada servicio
- [ ] `docker-compose up --build` sin errores
- [ ] Healthchecks pasando
- [ ] Puertos accesibles
- [ ] Volúmenes persistiendo datos

**Documentación:**
- [ ] README.md actualizado
- [ ] Patrones documentados (dónde, por qué, cómo)
- [ ] Diagrama de arquitectura incluido
- [ ] Endpoints documentados
- [ ] Instrucciones de setup clara
- [ ] Explicación de SOLID principles

---

## 🆘 Soporte y Troubleshooting

### Problemas Comunes

**Puerto ya en uso:**
```bash
# Windows
netstat -ano | findstr :3001
taskkill /PID <PID>

# Linux/Mac
lsof -i :3001
kill -9 <PID>
```

**BD no conecta:**
```bash
# Verificar credenciales en .env
# Verificar PostgreSQL corriendo
docker ps | grep postgres
```

**Docker compose error:**
```bash
# Borrar y reconstruir
docker-compose down -v
docker-compose up --build
```

**Frontend no conecta API:**
```bash
# Verificar URLs en api.js
# Verificar CORS habilitado en Backend
# Verificar servicios levantados: docker-compose ps
```

Ver `docs/TROUBLESHOOTING.md` para más

---

## 📚 Referencias y Enlaces

- [Node.js Docs](https://nodejs.org/docs/)
- [Express.js Guide](https://expressjs.com/)
- [Sequelize ORM](https://sequelize.org/)
- [React Docs](https://react.dev/)
- [Vite Guide](https://vitejs.dev/)
- [Docker Docs](https://docs.docker.com/)
- [Camunda Docs](https://docs.camunda.org/)
- [SOLID Principles](https://en.wikipedia.org/wiki/SOLID)

---

## 📋 Progreso del Proyecto

**Fases:**
1. ✅ Configuración inicial (Estructura, .env, Dockerfiles)
2. ⏳ Microservicio Autores (Models, Service, API)
3. ⏳ Microservicio Publicaciones (Models, Service, API, Client)
4. ⏳ Frontend React (Componentes, Servicios, Rutas)
5. ⏳ BPMN (Modelado, Simulación)
6. ⏳ Integración (Docker Compose, Testing)
7. ⏳ Documentación (Patrones, SOLID, Readme)
8. ⏳ Pruebas finales (E2E, Performance)
9. ⏳ Entrega (Empaquetado, Presentación)

---

## 📞 Contacto y Preguntas

Si tienes dudas sobre:
- **Arquitectura** → Consulta `ARQUITECTURA.md`
- **Pasos implementación** → Sigue `PLAN_EJECUCION.md`
- **Requisitos** → Lee `ESPECIFICACIONES.md`
- **Inicio rápido** → Usa `INICIO_RAPIDO.md`
- **Problemas** → Revisa `docs/TROUBLESHOOTING.md`

---

## 📄 Información del Proyecto

- **Versión:** 1.0
- **Fecha:** 4 de febrero de 2026
- **Equipo:** Máximo 3 personas
- **Plazo:** 3 semanas
- **Tecnologías:** Node.js, React, PostgreSQL, Docker
- **Arquitectura:** Microservicios
- **Estado:** Documentación completa, listo para desarrollo

---

**¡Bienvenido al proyecto! Sigue los documentos en orden y tendrás éxito. 🚀**

```
Inicio Recomendado:
1. Este README.md (visión general) ✓
2. ESPECIFICACIONES.md (requisitos)
3. ARQUITECTURA.md (diseño)
4. INICIO_RAPIDO.md (primeros pasos)
5. PLAN_EJECUCION.md (implementación paso a paso)
```

---

## 📊 Diagrama de Flujo General

```
Usuario en Frontend (React)
    │
    ├── Crear/Ver Autores
    │   └── HTTP → Authors Service (3001)
    │       └── BD PostgreSQL authors_db
    │
    ├── Crear/Ver Publicaciones
    │   └── HTTP → Publications Service (3002)
    │       ├─ Valida autor (HTTP → Authors Service)
    │       └─ BD PostgreSQL publications_db
    │
    └── Ver Proceso Editorial (BPMN)
        └── Visualización en Camunda Modeler
```

---

**Documento Principal: README.md v1.0**  
**Generado por:** GitHub Copilot  
**Estado:** Listo para desarrollo
