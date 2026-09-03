# Asistente Experto en Gastronomía
### Proyecto Final — IA Generativa con Gemini, RAG y LangGraph

---

## 1. Descripción breve

Agente conversacional especializado en gastronomía que combina **Gemini 2.5 Flash Lite** como modelo de lenguaje, **RAG** (Retrieval-Augmented Generation) con ChromaDB para fundamentar las respuestas en una base de conocimiento propia, y **LangGraph** para estructurar la lógica del agente como un grafo de nodos con memoria de conversación.

---

## 2. Dominio elegido: Gastronomía

- **Cocinas del mundo**: paella valenciana (española) y sushi (japonesa)
- **Técnicas culinarias**: técnicas clásicas francesas (mise en place, emulsión, salsas madre)
- **Repostería y chocolatería**: tipos de chocolate, temperado, origen del cacao
- **Enología y maridaje**: principios de maridaje vino-comida, temperaturas de servicio
- **Cocina molecular**: esferificación, nitrógeno líquido, sous vide, espumas (Adrià, Blumenthal)

---

## 3. Stack utilizado

| Componente | Tecnología |
|---|---|
| LLM | Gemini 2.5 Flash Lite (`langchain_google_genai`) |
| Embeddings | Gemini Embedding 001 (API REST directa) |
| Vector store | ChromaDB (`langchain_chroma`) |
| Orquestación del agente | LangGraph (`langgraph`) |
| Memoria de conversación | `add_messages` de LangGraph |
| UI interactiva | ipywidgets |
| Gestión de credenciales | python-dotenv |
| Entorno | Jupyter Notebook / VS Code |

---

## 4. Guía de ejecución

### Prerrequisitos

```bash
pip install langchain langchain-google-genai langchain-chroma langgraph ipywidgets python-dotenv requests
```

### Configurar API key

Crear un archivo `.env` en la misma carpeta que el notebook:

```
GEMINI_API_KEY=tu_api_key_aqui
```

> La API key de Gemini se obtiene en [Google AI Studio](https://aistudio.google.com).

### Ejecutar el notebook

Abrir `asistente_gastronomia.ipynb` y ejecutar todas las celdas en orden:

1. **Celda 2** — carga la API key desde `.env`
2. **Celda 4** — define los 6 documentos de la base de conocimiento
3. **Celda 5** — crea y persiste ChromaDB en `./chroma_gastronomia/`
4. **Celda 7** — define el system prompt
5. **Celda 9** — configura el LLM (Gemini) y el retriever
6. **Celda 10** — construye y compila el grafo LangGraph
7. **Celda 12** — define la función `preguntar()`
8. **Celda 14** — ejecuta 5 preguntas de demostración
9. **Celda 16** — abre el chat interactivo con ipywidgets

> **Alternativa sin cuota de Gemini**: descomentar el bloque de `ChatOllama` en la celda 9. Requiere tener [Ollama](https://ollama.com) instalado y haber ejecutado `ollama pull gemma3:4b`.

---

## 5. Justificación del system prompt

El system prompt define la identidad, el tono, las capacidades y los límites del agente:

```
Eres Chef AI, un asistente experto en gastronomía con profundo conocimiento
en cocina internacional, técnicas culinarias, ingredientes, maridajes y cultura gastronómica.
```

**Decisiones de diseño:**

- **Rol explícito ("Chef AI")**: anclar la identidad evita que el modelo divague hacia otros dominios y da coherencia a las respuestas.
- **Tono "chef experimentado con alumno"**: establece un registro profesional pero accesible, sin ser frío ni informal.
- **Precisión en recetas**: la instrucción de ser específico con cantidades y pasos obliga al modelo a dar respuestas útiles, no genéricas.
- **Placeholder `{context}`**: se inyectan en cada turno los fragmentos recuperados de ChromaDB, forzando que las respuestas se basen en la base de conocimiento y no en la memoria paramétrica del LLM.
- **Honestidad explícita**: "si no tienes información suficiente, dilo" reduce las alucinaciones en un dominio donde los detalles técnicos importan.
- **Límite de dominio**: la restricción a gastronomía mantiene el foco del agente y es coherente con el corpus de documentos disponible.

---

## 6. Arquitectura del grafo

```
                    ┌───────────────────────────┐
                    │        AgentState          │
                    │  - messages (historial)    │
                    │  - context  (RAG)          │
                    │  - question (pregunta)     │
                    └───────────────────────────┘
                                 │
                    ┌────────────▼────────────┐
     ENTRADA ──────►│  Nodo: recuperar        │
                    │  (recuperar_contexto)   │
                    │                         │
                    │  Busca en ChromaDB los  │
                    │  3 docs más relevantes  │
                    │  y los guarda en state  │
                    └────────────┬────────────┘
                                 │
                    ┌────────────▼────────────┐
                    │  Nodo: generar          │
                    │  (generar_respuesta)    │
                    │                         │
                    │  Construye el prompt:   │
                    │  SystemMsg(context) +   │
                    │  historial +            │
                    │  HumanMsg(pregunta)     │
                    │                         │
                    │  Llama a Gemini y       │
                    │  acumula en messages    │
                    └────────────┬────────────┘
                                 │
                               END
```

**Flujo de memoria**: en cada turno, `messages` acumula `HumanMessage` + `AIMessage`. El historial completo se pasa al nodo `generar` en la siguiente vuelta, dando al modelo contexto de la conversación anterior.

---

## 7. Dependencias

```
langchain>=0.3
langchain-google-genai>=2.0
langchain-chroma>=0.1
langgraph>=0.2
chromadb>=0.5
ipywidgets>=8.0
python-dotenv>=1.0
requests>=2.31
```
