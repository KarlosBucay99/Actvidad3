# 📋 Instrucciones de Ejecución Completa - Sistema de Autores y Publicaciones

## Estado Actual (5 de febrero de 2026)

✅ **Sistema completamente funcional y poblado con datos**

---

## 🚀 EJECUCIÓN DESDE CERO (Limpieza Total + Inicio)

Este proceso elimina todo y comienza desde cero. Sigue los pasos exactamente en orden.

### Requisitos Previos
- Docker instalado y ejecutándose
- `docker-compose` disponible
- Acceso a terminal (cmd, PowerShell o bash)
- Estar en la carpeta: `C:\Users\KarlosPC\Desktop\Tarea 3`

---

## FASE 1: LIMPIEZA TOTAL (Opcional - Solo si Necesitas Empezar Completamente de Cero)

Si quieres eliminar todo y comenzar desde cero, ejecuta estos comandos:

```bash
# 1. Bajar todos los servicios de compose
docker-compose down -v --remove-orphans

# 2. Eliminar contenedores residuales
docker rm -f db-authors finanzas authors-service publications-service frontend 2>nul

# 3. Eliminar volúmenes de datos
docker volume rm data-authors 2>nul

# 4. Listar lo que quedó (verificación)
docker ps -a
docker volume ls
```

**Nota:** Los comandos con `2>nul` son para Windows y ignoran errores si no existen los contenedores.

---

## FASE 2: CREAR INFRAESTRUCTURA DE BASE DE DATOS

### Paso 1: Crear Contenedor de BD de Autores

```bash
docker run --name db-authors -p 5432:5432 ^
  -e POSTGRES_USER=postgres ^
  -e POSTGRES_PASSWORD=cbucay0599 ^
  -e POSTGRES_DB=authors_db ^
  -v data-authors:/var/lib/postgresql/data ^
  -d postgres:15-alpine
```

**Qué hace:** Crea un contenedor PostgreSQL para la BD de autores, exponiendo puerto 5432.

Espera 5-10 segundos para que PostgreSQL inicie completamente.

### Paso 2: Crear Contenedor de BD de Publicaciones (finanzas)

```bash
docker run --name finanzas -p 5433:5432 ^
  -e POSTGRES_USER=postgres ^
  -e POSTGRES_PASSWORD=cbucay0599 ^
  -e POSTGRES_DB=finanzasbd ^
  -v data-publications:/var/lib/postgresql/data ^
  -d postgres:15-alpine
```

**Qué hace:** Crea un contenedor PostgreSQL para la BD de publicaciones, exponiendo puerto 5433.

### Paso 3: Verificar que las BDs Estén Corriendo

```bash
docker ps | grep -E "db-authors|finanzas"
```

**Resultado esperado:** Deberías ver dos contenedores listados como "Up" o "running".

---

## FASE 3: CONFIGURAR RED DE DOCKER

Docker Compose necesita que los contenedores estén en su red para comunicarse.

### Paso 1: Crear la Red de Docker Compose (si no existe)

```bash
docker network ls | grep tarea3_microservices
```

Si no aparece en la lista, créala:

```bash
docker network create tarea3_microservices
```

### Paso 2: Conectar BDs a la Red

```bash
docker network connect tarea3_microservices db-authors
docker network connect tarea3_microservices finanzas
```

Si ves el error "endpoint with name finanzas already exists in network tarea3_microservices", significa que ya están conectados. ✅ Está bien.

### Paso 3: Verificar Conectividad

```bash
docker inspect db-authors --format "{{json .NetworkSettings.Networks}}"
docker inspect finanzas --format "{{json .NetworkSettings.Networks}}"
```

Ambos deberían mostrar `tarea3_microservices` en sus networks.

---

## FASE 4: LEVANTAR MICROSERVICIOS

### Paso 1: Navegar a la Carpeta del Proyecto

```bash
cd "C:\Users\KarlosPC\Desktop\Tarea 3"
```

Verifica que estés en la carpeta correcta:
```bash
dir | grep docker-compose.yml
```

Deberías ver `docker-compose.yml` listado.

### Paso 2: Iniciar Servicios con Docker Compose

```bash
docker-compose up -d
```

**Qué hace:** 
- Levanta `authors-service` en puerto 3001
- Levanta `publications-service` en puerto 3002
- Levanta `frontend` en puerto 3000

Espera 15-20 segundos para que todos los servicios se inicien completamente.

### Paso 3: Verificar que los Servicios Estén Corriendo

```bash
docker-compose ps
```

**Resultado esperado:**
```
NAME                 STATUS
authors-service      Up (healthy)
publications-service Up (healthy)
frontend             Up
```

---

## FASE 5: VERIFICACIÓN Y PRUEBAS

### Paso 1: Verificar Health de los Servicios

```bash
curl http://localhost:3001/health
```

**Resultado esperado:**
```json
{"status":"Authors Service OK","database":"connected","timestamp":"..."}
```

```bash
curl http://localhost:3002/health
```

**Resultado esperado:**
```json
{"status":"healthy","service":"publications-service","timestamp":"..."}
```

### Paso 2: Verificar Datos Poblados

Listar autores:
```bash
curl http://localhost:3001/authors
```

**Resultado esperado:** JSON con 5 autores (total: 5)

Listar publicaciones:
```bash
curl http://localhost:3002/publications
```

**Resultado esperado:** JSON con 5 publicaciones (total: 5)

### Paso 3: Acceder a la Aplicación

Abre tu navegador:
- **Frontend:** http://localhost:3000
- **API Autores:** http://localhost:3001/authors
- **API Publicaciones:** http://localhost:3002/publications

---

## 🎨 EJECUTAR LA INTERFAZ GRÁFICA

### Opción 1: Frontend Incluido en docker-compose (Recomendado)

La interfaz gráfica se levanta automáticamente cuando ejecutas:

```bash
docker-compose up -d
```

Luego accede en tu navegador:
```
http://localhost:3000
```

**Qué verás:**
- Página de inicio con navegación
- Sección de Autores (listar, crear, ver detalles)
- Sección de Publicaciones (listar, crear, ver detalles)
- Interfaz conectada a los APIs en puertos 3001 y 3002

### Opción 2: Ejecutar Frontend Localmente (Para Desarrollo)

Si quieres ejecutar el frontend sin Docker (para desarrollo):

```bash
# 1. Navegar a la carpeta del frontend
cd "C:\Users\KarlosPC\Desktop\Tarea 3\cliente\frontend"

# 2. Instalar dependencias (si aún no lo hiciste)
npm install

# 3. Iniciar el servidor de desarrollo
npm run dev
```

**Resultado:**
```
  VITE v4.x.x  ready in xxx ms

  ➜  Local:   http://localhost:5173/
  ➜  press h to show help
```

Abre en tu navegador: `http://localhost:5173`

**Nota:** El frontend en desarrollo (puerto 5173) se conectará automáticamente a los APIs en puertos 3001 y 3002 (Docker).

### Opción 3: Ver el Estado del Frontend en Docker

```bash
# Ver logs del frontend
docker-compose logs frontend

# Ver si está corriendo
docker ps | grep frontend

# Reiniciar solo el frontend
docker-compose down frontend
docker-compose up -d frontend
```

---

## CHECKLIST DE VERIFICACIÓN RÁPIDA

Ejecuta esto para confirmar que TODO está funcionando:

```bash
# Verificar contenedores
docker ps | grep -E "db-authors|finanzas|authors|publications|frontend"

# Verificar health
curl -s http://localhost:3001/health | find "Authors Service OK" && echo "✓ Authors Service"
curl -s http://localhost:3002/health | find "healthy" && echo "✓ Publications Service"

# Verificar datos
curl -s http://localhost:3001/authors | find '"total":5' && echo "✓ 5 Autores"
curl -s http://localhost:3002/publications | find '"total":5' && echo "✓ 5 Publicaciones"
```

---

## 📊 Datos Poblados en el Sistema

### 5 Autores Incluidos
1. **Gabriel García Márquez** - Colombia (ID: 1)
2. **Isabel Allende** - Chile (ID: 2)
3. **Jorge Luis Borges** - Argentina (ID: 3)
4. **Mario Vargas Llosa** - Perú (ID: 4)
5. **Laura Esquivel** - México (ID: 5)

### 5 Publicaciones Incluidas
| ID | Título | Autor | Estado |
|---|---|---|---|
| 1 | Cien años de soledad | Gabriel García Márquez | DRAFT |
| 2 | La casa de los espíritus | Isabel Allende | DRAFT |
| 3 | Ficciones | Jorge Luis Borges | DRAFT |
| 4 | Conversación en La Catedral | Mario Vargas Llosa | DRAFT |
| 5 | Como agua para chocolate | Laura Esquivel | DRAFT |

*Los datos se crean automáticamente cuando inicias los servicios por primera vez.*

---

## OPERACIONES COMUNES DESPUÉS DE LA INSTALACIÓN

### Detener Todos los Servicios

```bash
# Detener sin eliminar volúmenes (preserva datos)
docker-compose down

# Detener y eliminar todo incluyendo volúmenes
docker-compose down -v
```

### Reiniciar Servicios (Si Ya Existen)

```bash
# Si todo está creado, solo haz esto:
docker-compose up -d

# Luego verifica:
docker-compose ps
curl http://localhost:3001/health
```

### Ver Logs de los Servicios

```bash
# Logs del servicio de autores
docker-compose logs authors-service

# Logs del servicio de publicaciones
docker-compose logs publications-service

# Logs del frontend
docker-compose logs frontend

# Logs de todos en tiempo real
docker-compose logs -f
```

### Conectarse a las Bases de Datos Directamente

```bash
# Conectar a BD de autores
docker exec -it db-authors psql -U postgres -d authors_db

# Conectar a BD de publicaciones
docker exec -it finanzas psql -U postgres -d finanzasbd
```

Luego en psql puedes hacer:
```sql
\dt                    -- Ver tablas
SELECT * FROM authors; -- Ver autores
SELECT * FROM publications; -- Ver publicaciones
```

---

## 🔌 Puertos y URLs

| Servicio | URL | Puerto |
|----------|-----|--------|
| Frontend | http://localhost:3000 | 3000 |
| Authors API | http://localhost:3001 | 3001 |
| Authors API Health | http://localhost:3001/health | 3001 |
| Publications API | http://localhost:3002 | 3002 |
| Publications API Health | http://localhost:3002/health | 3002 |
| Authors BD | localhost:5432 | 5432 |
| Publications BD (finanzas) | localhost:5433 | 5433 |

---

## 📝 Endpoints Importantes

### Authors API (Puerto 3001)
```bash
# Listar autores
GET /authors?page=1&limit=10

# Obtener autor específico
GET /authors/:id

# Crear autor
POST /authors
Body: {"name":"...", "email":"...", "birthDate":"...", "nationality":"..."}

# Actualizar autor
PUT /authors/:id

# Eliminar autor
DELETE /authors/:id

# Verificar salud
GET /health
```

### Publications API (Puerto 3002)
```bash
# Listar publicaciones
GET /publications?page=1&limit=10

# Obtener publicación específica
GET /publications/:id

# Crear publicación
POST /publications
Body: {"title":"...", "description":"...", "content":"...", "authorId":1, "status":"DRAFT"}

# Cambiar estado
PATCH /publications/:id/status
Body: {"newStatus":"REVIEW"}

# Verificar salud
GET /health
```

---

## 🛑 Detener Servicios

```bash
# Detener todos los servicios
docker-compose down

# Mantener volúmenes (preservar datos)
docker-compose down --remove-orphans

# Limpiar todo incluyendo volúmenes
docker-compose down -v
```

---

## 🔧 Solución de Problemas

### Problema 1: "Error response from daemon: port is already allocated"

**Causa:** Los puertos 3001, 3002, 3000, 5432 o 5433 ya están en uso.

**Solución:**

```bash
# En Windows - Encontrar qué está usando el puerto
netstat -ano | findstr :3001

# Eliminar el proceso (reemplaza <PID> con el número)
taskkill /PID <PID> /F

# O mejor aún, limpiar todo
docker-compose down -v --remove-orphans
docker rm -f db-authors finanzas 2>nul
docker volume rm data-authors data-publications 2>nul
```

Luego vuelve a ejecutar desde la FASE 2.

### Problema 2: "Los servicios no se conectan a la BD"

**Causa:** Los contenedores no están en la red correcta.

**Verificación:**
```bash
docker inspect db-authors --format "{{json .NetworkSettings.Networks}}"
docker inspect finanzas --format "{{json .NetworkSettings.Networks}}"
```

Ambos deben mostrar `tarea3_microservices`.

**Solución:**
```bash
docker network connect tarea3_microservices db-authors
docker network connect tarea3_microservices finanzas
```

### Problema 3: "Connection refused" al hacer curl

**Causa:** Los servicios aún no han iniciado completamente.

**Solución:**
```bash
# Espera 20 segundos y luego intenta nuevamente
docker-compose ps  -- Ver estado de los servicios

# Si alguno no está "Up", revisa los logs:
docker-compose logs authors-service
docker-compose logs publications-service
```

### Problema 4: "password authentication failed for user 'postgres'"

**Causa:** La contraseña no coincide entre los containers.

**Solución:**
```bash
# Ejecutar dentro del contenedor para cambiar la contraseña
docker exec db-authors psql -U postgres -d authors_db -c "ALTER USER postgres WITH PASSWORD 'cbucay0599';"
docker exec finanzas psql -U postgres -d finanzasbd -c "ALTER USER postgres WITH PASSWORD 'cbucay0599';"

# Luego reinicia los servicios
docker-compose down
docker-compose up -d
```

### Problema 5: Base de datos vacía (sin datos)

**Causa:** Los datos no se migraron correctamente al iniciar.

**Verificación:**
```bash
docker exec db-authors psql -U postgres -d authors_db -c "SELECT COUNT(*) FROM authors;"
docker exec finanzas psql -U postgres -d finanzasbd -c "SELECT COUNT(*) FROM publications;"
```

**Solución:** Vuelve a limpiar todo y comienza desde la FASE 1 (LIMPIEZA TOTAL).

### Problema 6: Entre en un Bucle Infinito de Errores

**Solución Nuclear (Elimina ABSOLUTAMENTE TODO):**

```bash
# Bajar compose
docker-compose down -v --remove-orphans

# Eliminar contenedores
docker rm -f $(docker ps -a -q) 2>nul

# Eliminar volúmenes
docker volume rm $(docker volume ls -q) 2>nul

# Eliminar redes personalizadas
docker network rm tarea3_microservices microservices 2>nul

# Verificar que esté limpio
docker ps -a
docker volume ls
docker network ls
```

Luego comienza desde la FASE 2.

---

## ✅ Comandos Finales de Validación

Una vez completes todas las FASES, ejecuta esto para confirmar que todo está perfecto:

```bash
# 1. Verificar contenedores levantados
docker ps

# 2. Testear health de servicios
curl http://localhost:3001/health && echo. && echo "✓ Authors Service OK"
curl http://localhost:3002/health && echo. && echo "✓ Publications Service OK"

# 3. Verificar datos
curl -s http://localhost:3001/authors | find "total" && echo "✓ Autores poblados"
curl -s http://localhost:3002/publications | find "total" && echo "✓ Publicaciones pobladas"

# 4. Ver estadísticas finales
docker stats --no-stream
```

**Si todo muestra "✓", ¡el sistema está completamente funcional!** 🎉

---

## 📞 Referencia Rápida de URLs

### Acceso al Sistema
| Componente | URL | Descripción |
|---|---|---|
| Frontend | http://localhost:3000 | Interfaz gráfica (React) |
| Authors API | http://localhost:3001 | API REST de autores |
| Publications API | http://localhost:3002 | API REST de publicaciones |

### Health Checks
| Servicio | URL |
|---|---|
| Authors | http://localhost:3001/health |
| Publications | http://localhost:3002/health |

### Datos
| Recurso | URL | Método |
|---|---|---|
| Listar Autores | http://localhost:3001/authors | GET |
| Obtener Autor | http://localhost:3001/authors/:id | GET |
| Crear Autor | http://localhost:3001/authors | POST |
| Listar Publicaciones | http://localhost:3002/publications | GET |
| Obtener Publicación | http://localhost:3002/publications/:id | GET |
| Crear Publicación | http://localhost:3002/publications | POST |

---

## 🎯 Resumen de Puertos

| Servicio | Puerto | Host |
|---|---|---|
| Frontend | 3000 | localhost:3000 |
| Authors Service | 3001 | localhost:3001 |
| Publications Service | 3002 | localhost:3002 |
| BD Autores | 5432 | localhost:5432 |
| BD Publicaciones | 5433 | localhost:5433 |

---

**¡Listo para usar! 🚀**
