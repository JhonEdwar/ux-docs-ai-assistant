# 📚 UX Docs AI Assistant — n8n RAG Workflows

Asistente de IA especializado en UX y diseño de productos digitales. Responde preguntas basándose **exclusivamente** en documentos y libros de UX indexados en Pinecone, accesible vía chat web y Telegram.

---

## 🗂️ Estructura del proyecto

```
/
├── loadDataUx.json       # Ingesta y vectorización de PDFs
└── Chat_docs_ux.json     # Agente conversacional RAG
```

---

## 🔄 Workflows

### 1. `loadDataUx` — Ingesta de documentos

Carga PDFs desde Google Drive, los fragmenta y los almacena como vectores en Pinecone.

```
Google Drive (PDFs) → Document Loader → Text Splitter → Embeddings OpenAI → Pinecone
```

| Parámetro | Valor |
|-----------|-------|
| Chunk size / overlap | 600 / 100 tokens |
| Namespace | `docsUx` |
| Índice | `docsuxn8n` |
| Modelo de embeddings | `text-embedding-3-large` |

> ⚠️ Limpiar el namespace en Pinecone antes de recargar documentos para evitar duplicados.

---

### 2. `Chat_docs_ux` — Agente conversacional

Recibe preguntas por Telegram o chat web, consulta Pinecone y responde con información de los documentos.

```
Telegram / Chat UI → AI Agent → Pinecone (vector_search_ux_docs) → Respuesta
```

**Nodos principales:**

| Nodo | Descripción |
|------|-------------|
| `AI Agent` | Orquesta la lógica RAG |
| `OpenAI Chat Model` | `gpt-4.1-mini`, temperatura `0.3` |
| `Postgres Chat Memory` | Historial por sesión (ventana: 10 mensajes) |
| `vector_search_ux_docs` | Búsqueda vectorial en Pinecone (top K: 10) |
| `Embeddings OpenAI` | `text-embedding-3-large` |

---

## 🧠 Comportamiento del agente

- Solo responde con información recuperada de los vectores, nunca con conocimiento general.
- Si no encuentra información responde: *"No tengo información suficiente en los documentos disponibles."*
- Siempre indica la fuente (libro, documento, página).
- Máximo 3 puntos clave por respuesta.

---

## 🚀 Setup

### 1. Importar workflows
En n8n: **Settings → Import workflow** → cargar cada `.json`.

### 2. Configurar credenciales

| Servicio | Uso |
|----------|-----|
| OpenAI | LLM + Embeddings |
| Pinecone | Vector store |
| PostgreSQL | Chat memory |
| Telegram | Bot trigger + respuestas |
| Google Drive | Fuente de PDFs |

### 3. Crear índice en Pinecone

```
Índice:       docsuxn8n
Namespace:    docsUx
Dimensiones:  3072
Métrica:      cosine
```

### 4. Cargar documentos
1. Subir PDFs a Google Drive
2. Ejecutar el workflow `loadDataUx`
3. Verificar que los vectores aparezcan en Pinecone

### 5. Activar el agente
1. Activar el workflow `Chat_docs_ux`
2. n8n debe estar expuesto en una **URL pública HTTPS** (requerido para el webhook de Telegram)

---

## 💬 Ejemplos de uso

```
¿Qué es la usabilidad según Nielsen?
¿Cuáles son las heurísticas de usabilidad?
Dame un checklist de pruebas de usabilidad
```

---

## 🐛 Troubleshooting

| Problema | Solución |
|----------|----------|
| Agente responde con conocimiento general | Verificar que `vector_search_ux_docs` esté conectado |
| Respuestas duplicadas | Limpiar namespace `docsUx` y recargar documentos |
| Token overflow | Reducir `contextWindowLength` en Postgres Chat Memory |
| Webhook de Telegram sin respuesta | Verificar URL pública HTTPS y que el workflow esté activo |
| Embeddings no coinciden | Confirmar `text-embedding-3-large` en carga y búsqueda |

---

## 📦 Stack

- [n8n](https://n8n.io/) — Orquestación de workflows
- [OpenAI](https://openai.com/) — LLM + Embeddings
- [Pinecone](https://www.pinecone.io/) — Base de datos vectorial
- [PostgreSQL / Supabase](https://supabase.com/) — Memoria de conversación
- [Telegram Bot API](https://core.telegram.org/bots/api) — Interfaz de chat

---

## 📄 Licencia

MIT
