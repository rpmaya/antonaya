# Antonaya
## Propuesta Técnica

## DIAGRAMA DE ARQUITECTURA

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                         CAPA DE GESTIÓN DE CONOCIMIENTO                         │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────────────┐                    ┌─────────────────────────────┐     │
│  │  Empleado/Gestor    │                    │   Portal de Administración  │     │
│  │  Sube documentos    │───────────────────▶│   • Selección comunidad     │     │
│  │  por comunidad      │                    │   • Tipo documento          │     │
│  └─────────────────────┘                    │   • Validación              │     │
│                                             └──────────────┬──────────────┘     │
│                                                            │                    │
└────────────────────────────────────────────────────────────┼────────────────────┘
                                                             │
                                                             ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              N8N.IO - ORQUESTADOR CENTRAL                       │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────────────────────┐         ┌──────────────────────────────────┐   │
│  │   Workflow: Upload Docs     │         │   Workflow: Email Processing     │   │
│  ├─────────────────────────────┤         ├──────────────────────────────────┤   │
│  │ 1. Webhook receiver         │         │ 1. Email trigger (IMAP/Gmail)    │   │
│  │ 2. Community validator      │         │ 2. Extract sender & community    │   │
│  │ 3. Document processor       │         │ 3. Route to specific AI agent    │   │
│  │ 4. Metadata enrichment      │         │ 4. Query knowledge base          │   │
│  │ 5. Chunking by type         │         │ 5. Generate response             │   │
│  │ 6. Embedding generator      │         │ 6. Personalize for community     │   │
│  │ 7. Vector DB router         │         │ 7. Send email response           │   │
│  └──────────────┬──────────────┘         └──────────────┬───────────────────┘   │
│                 │                                       │                       │
│  ┌──────────────▼──────────────┐         ┌──────────────▼───────────────────┐   │
│  │  Community Classifier       │         │  AI Agent Selector               │   │
│  │  • Comunidad ID             │         │  • Detect community from email   │   │
│  │  • Document type            │         │  • Load specific context         │   │
│  │  • Access permissions       │         │  • Apply community rules         │   │
│  └──────────────┬──────────────┘         └──────────────┬───────────────────┘   │
│                 │                                       │                       │
└─────────────────┼───────────────────────────────────────┼───────────────────────┘
                  │                                       │
                  ▼                                       ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                     KNOWLEDGE BASE & TRAINING (Multi-Tenant)                    │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌────────────────────────┐     ┌──────────────────────────┐                    │
│  │   VECTOR DB GENERAL    │     │  VECTOR DB POR COMUNIDAD │                    │
│  ├────────────────────────┤     ├──────────────────────────┤                    │
│  │ Namespace: "general"   │     │ Namespace: "comm_001"    │ ◄─── Comunidad A   │
│  │ • Normativas comunes   │     │ • Estatutos propios      │                    │
│  │ • Leyes propiedad H.   │     │ • Actas reuniones        │                    │
│  │ • Procedimientos std   │     │ • Presupuestos           │                    │
│  │ • FAQs generales       │     │ • Proveedores            │                    │
│  └────────────────────────┘     ├──────────────────────────┤                    │
│                                 │ Namespace: "comm_002"    │ ◄─── Comunidad B   │
│  ┌────────────────────────┐     │ • Estatutos propios      │                    │
│  │  METADATA STORE        │     │ • Actas reuniones        │                    │
│  ├────────────────────────┤     │ • Deudas/Pagos           │                    │
│  │ • Community profiles   │     │ • Obras planificadas     │                    │
│  │ • Access controls      │     ├──────────────────────────┤                    │
│  │ • Document versions    │     │ Namespace: "comm_003"    │ ◄─── Comunidad C   │
│  │ • Update history       │     │ • Estatutos propios      │                    │
│  └────────────────────────┘     │ • Normas piscina         │                    │
│                                 │ • Horarios zonas comunes │                    │
│                                 └──────────────────────────┘                    │
│                                                                                 │
└──────────────────────────────────────┬──────────────────────────────────────────┘
                                       │
                                       ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                            AI PROCESSING LAYER                                  │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌────────────────────────┐     ┌────────────────────────────────────────────┐  │
│  │   AGENTE IA GENERAL    │     │        AGENTES IA POR COMUNIDAD            │  │
│  ├────────────────────────┤     ├────────────────────────────────────────────┤  │
│  │ • Contexto legal común │     │  ┌─────────────┐  ┌─────────────┐          │  │
│  │ • Respuestas estándar  │     │  │ Agente      │  │ Agente      │          │  │
│  │ • Conocimiento base    │     │  │ Comunidad A │  │ Comunidad B │  ...     │  │
│  └───────────┬────────────┘     │  └─────────────┘  └─────────────┘          │  │
│              │                  │  • Contexto específico                     │  │
│              │                  │  • Tono personalizado                      │  │
│              │                  │  • Historial comunidad                     │  │
│              │                  └────────────────────────────────────────────┘  │
│              │                                       │                          │
│              └───────────────┬───────────────────────┘                          │
│                              ▼                                                  │
│  ┌────────────────────────────────────────────────────────────────────────┐     │
│  │                    RESPONSE GENERATOR                                  │     │
│  │  • Combina respuesta general + específica                              │     │
│  │  • Aplica tono y estilo de la comunidad                                │     │
│  │  • Añade información relevante (próximas juntas, pagos, etc.)          │     │
│  └────────────────────────────────────────────────────────────────────────┘     │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────────┐
│                           CAPA DE COMUNICACIÓN                                  │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────────────┐     ┌─────────────────────┐    ┌──────────────────┐    │
│  │   Email Entrante    │     │  Email Saliente     │    │   Audit Trail    │    │
│  │  • IMAP/Gmail API   │     │  • SMTP/API         │    │  • Logs          │    │
│  │  • Auto-detección   │     │  • Templates        │    │  • Métricas      │    │
│  │    comunidad        │     │  • Attachments      │    │  • Analytics     │    │
│  └─────────────────────┘     └─────────────────────┘    └──────────────────┘    │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```
## CARACTERÍSTICAS CLAVE

1. **Multi-tenancy completo**: Cada comunidad tiene su propio espacio aislado
2. **Conocimiento compartido**: Base general + específica por comunidad
3. **Detección automática**: Identifica la comunidad del email entrante
4. **Personalización**: Respuestas adaptadas a cada comunidad
5. **Control de acceso**: Empleados solo acceden a comunidades autorizadas
6. **Trazabilidad**: Log completo de todas las interacciones


