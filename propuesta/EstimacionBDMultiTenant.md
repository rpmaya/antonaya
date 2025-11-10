# Estimación de Trabajo: Base de Datos y Embeddings para Sistema Multi-tenant

## 1. Setup Inicial de Supabase (4-6 horas)

### Base de Datos Relacional
**Tiempo estimado: 2-3 horas**

- Crear proyecto en Supabase
- Configurar variables de entorno y conexión
- Setup de políticas de seguridad Row Level Security (RLS) base
- Configuración de roles y permisos iniciales

### Base de Datos Vectorial
**Tiempo estimado: 2-3 horas**

- Habilitar extensión `pgvector` en Supabase
- Configurar índices HNSW para búsqueda eficiente
- Testing de conexión y queries vectoriales básicas

## 2. Diseño y Creación de Modelos (8-12 horas)

### Esquema Relacional Multi-tenant
**Tiempo estimado: 5-7 horas**

**Tablas principales:**

```sql
-- Tabla de tenants (propietarios)
tenants
  - id (uuid, PK)
  - name (text)
  - email (text)
  - created_at (timestamp)
  - settings (jsonb)

-- Tabla de fincas
fincas
  - id (uuid, PK)
  - tenant_id (uuid, FK)
  - nombre (text)
  - ubicacion (text)
  - metadata (jsonb)
  - created_at (timestamp)

-- Tabla de documentos
documentos
  - id (uuid, PK)
  - finca_id (uuid, FK)
  - tenant_id (uuid, FK)
  - tipo (text) -- facturas, contratos, informes, etc.
  - titulo (text)
  - contenido_original (text)
  - url_archivo (text)
  - metadata (jsonb)
  - created_at (timestamp)

-- Tabla de conversaciones del agente
conversaciones
  - id (uuid, PK)
  - finca_id (uuid, FK)
  - tenant_id (uuid, FK)
  - user_id (uuid)
  - created_at (timestamp)

-- Tabla de mensajes
mensajes
  - id (uuid, PK)
  - conversacion_id (uuid, FK)
  - tenant_id (uuid, FK)
  - role (text) -- user, assistant, system
  - contenido (text)
  - metadata (jsonb)
  - created_at (timestamp)
```

**Tareas:**
- Diseño del esquema ER
- Creación de migraciones SQL
- Implementación de índices para optimización
- Setup de foreign keys y constraints

### Esquema Vectorial
**Tiempo estimado: 3-5 horas**

```sql
-- Tabla de embeddings
embeddings
  - id (uuid, PK)
  - documento_id (uuid, FK)
  - finca_id (uuid, FK)
  - tenant_id (uuid, FK) -- CRÍTICO para multi-tenant
  - chunk_index (integer)
  - contenido_chunk (text)
  - embedding (vector(1536)) -- OpenAI ada-002 o similar
  - metadata (jsonb)
  - created_at (timestamp)
```

**Tareas:**
- Crear tabla con tipo vector
- Crear índice HNSW para búsqueda rápida
- Definir estrategia de chunking (tamaño, overlap)

## 3. Implementación de Row Level Security (RLS) (6-8 horas)

**Tiempo estimado: 6-8 horas**

**Políticas críticas para multi-tenant:**

```sql
-- Política para tabla fincas
CREATE POLICY "Usuarios solo ven sus fincas"
ON fincas FOR SELECT
USING (tenant_id = current_setting('app.tenant_id')::uuid);

-- Política para embeddings
CREATE POLICY "Embeddings por tenant"
ON embeddings FOR SELECT
USING (tenant_id = current_setting('app.tenant_id')::uuid);

-- Políticas INSERT, UPDATE, DELETE similares
```

**Tareas:**
- Implementar RLS en todas las tablas
- Crear función para set tenant_id en contexto
- Testing exhaustivo de aislamiento entre tenants
- Documentación de políticas de seguridad

## 4. Sistema de Generación de Embeddings (12-16 horas)

### Pipeline de Procesamiento
**Tiempo estimado: 8-10 horas**

**Componentes en Python:**

```python
# Arquitectura sugerida
- document_processor.py
  - Extracción de texto (PDF, DOCX, TXT)
  - Limpieza y normalización
  
- chunking_service.py
  - Estrategia de chunking (RecursiveCharacterTextSplitter)
  - Tamaño chunks: 500-1000 tokens
  - Overlap: 50-100 tokens
  
- embedding_service.py
  - Generación de embeddings (OpenAI, Cohere, o modelo local)
  - Batch processing para eficiencia
  - Rate limiting y manejo de errores
  
- vector_store.py
  - Inserción en Supabase
  - Validación de tenant_id
  - Manejo de duplicados
```

**Tareas:**
- Implementar extracción de texto multi-formato
- Configurar chunking óptimo según tipo documento
- Integrar API de embeddings con manejo de errores
- Crear pipeline batch para documentos existentes
- Testing con documentos reales

### Scripts de Migración y Mantenimiento
**Tiempo estimado: 4-6 horas**

```python
# Scripts necesarios
- initial_load.py
  - Carga inicial de documentos para 5 fincas
  - Validación de datos
  
- reindex_embeddings.py
  - Re-generación de embeddings si cambia modelo
  
- cleanup_orphaned.py
  - Limpieza de embeddings huérfanos
```

## 5. API y Funciones de Búsqueda (8-10 horas)

### Funciones de Base de Datos
**Tiempo estimado: 4-5 horas**

```sql
-- Función de búsqueda semántica con filtro tenant
CREATE FUNCTION search_embeddings(
  query_embedding vector(1536),
  p_tenant_id uuid,
  p_finca_id uuid DEFAULT NULL,
  match_threshold float DEFAULT 0.7,
  match_count int DEFAULT 10
)
RETURNS TABLE (
  id uuid,
  documento_id uuid,
  contenido_chunk text,
  similarity float,
  metadata jsonb
);
```

**Tareas:**
- Crear funciones PostgreSQL optimizadas
- Implementar filtros multi-tenant seguros
- Testing de performance con datos reales

### API Python para n8n
**Tiempo estimado: 4-5 horas**

```python
# Endpoints necesarios
- POST /api/embeddings/generate
  - Procesa nuevo documento
  - Valida tenant_id
  
- POST /api/search/semantic
  - Búsqueda semántica
  - Incluye tenant_id en contexto
  
- GET /api/documents/{finca_id}
  - Lista documentos de una finca
  
- DELETE /api/embeddings/{documento_id}
  - Elimina embeddings de documento
```

**Tareas:**
- Crear FastAPI o (express en caso de usar NodeJS)
- Implementar autenticación y validación tenant
- Documentar endpoints para equipo n8n
- Testing de integración

## 6. Testing y Optimización (6-8 horas)

### Testing
**Tiempo estimado: 4-5 horas**

- Unit tests para funciones críticas
- Integration tests del pipeline completo
- Testing de aislamiento multi-tenant
- Performance testing con volumen MVP (5 fincas)
- Testing de búsqueda semántica con queries reales

### Optimización
**Tiempo estimado: 2-3 horas**

- Análisis de queries lentas
- Ajuste de índices
- Optimización de chunking según resultados
- Tuning de parámetros HNSW

## 7. Documentación (4-5 horas)

**Entregables:**
- Diagrama ER actualizado
- Documentación de API
- Guía de setup y deployment
- Ejemplos de uso para equipo n8n
- Procedimientos de troubleshooting
- Estrategia de backup y recuperación

---

## RESUMEN DE ESTIMACIÓN TOTAL

| Fase | Tiempo Mínimo | Tiempo Máximo |
|------|--------------|---------------|
| Setup Inicial | 4h | 6h |
| Diseño Modelos | 8h | 12h |
| RLS Multi-tenant | 6h | 8h |
| Pipeline Embeddings | 12h | 16h |
| API Búsqueda | 8h | 10h |
| Testing/Optimización | 6h | 8h |
| Documentación | 4h | 5h |
| **TOTAL** | **48h** | **65h** |

**Estimación realista: 50-60 horas (6-8 días laborables)**

## Consideraciones Importantes

### Decisiones Técnicas Críticas

1. **Modelo de Embeddings:** ¿OpenAI, Gemini, Cohere, ...?
   - OpenAI ada-002: Más caro pero probado
   - Cohere: Buen balance costo/performance
   - Gemini ¿?

2. **Estrategia de Chunking:** Dependerá del tipo de documentos
   - Facturas: chunks pequeños (200-300 tokens)
   - Informes técnicos: chunks medianos (500-800 tokens)
   - Contratos: chunks con contexto legal (800-1000 tokens)

3. **Volumen Inicial Estimado (5 fincas):**
   - ~50-100 documentos por finca
   - ~250-500 documentos totales
   - ~2,500-10,000 chunks
   - Costo embeddings OpenAI: ~$2-8 USD inicial

### Riesgos y Mitigaciones

- **Aislamiento multi-tenant:** Prioridad máxima, testing exhaustivo necesario
- **Performance búsqueda:** Monitorear desde día 1, índices HNSW críticos
- **Costo embeddings:** Implementar caché y evitar re-generación innecesaria
- **Calidad búsqueda:** Iterar en chunking basado en feedback real

---

## Notas de Implementación

### Prioridades para el MVP

1. **Semana 1:** Setup Supabase + Esquema Relacional + RLS básico
2. **Semana 2:** Pipeline de embeddings + Búsqueda semántica
3. **Testing continuo:** Validar aislamiento multi-tenant en cada fase

### Métricas de Éxito

- Tiempo de respuesta búsqueda < 500ms
- 100% aislamiento entre tenants
- Precisión búsqueda semántica > 80% (feedback usuarios)
- Costo por documento < $0.02 USD