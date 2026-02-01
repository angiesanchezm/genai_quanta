# Mini Agente con Herramientas - Quanta Assistant

## Descripción

Este proyecto implementa un mini-agente conversacional utilizando LangChain para :
1. **Responder preguntas sobre Quanta** Un juego ficticio, usando  semantic search en una KB
2. **Buscar información general en Wikipedia** para consultas generales
3. **Citar fuentes** con doc_id/title o URLs

## Arquitectura

```
Usuario → Agente (LLM) → Decisión de tool
                  |
         ┌────────┴────────┐
         |                 |
    kb_search        wikipedia_search
         |                 |
   Vector Store       Wikipedia API
   (In-Memory)
         |                 |
    Documentos         Artículos
    de Quanta          de Wikipedia
```

### Componentes Principales

#### 1. **Base de Conocimiento (KB)**
- **5 documentos** sobre el juego Quanta
- **Temas cubiertos**: 
  - Overview & Components
  - Setup & Turn Structure
  - Scoring & Endgame
  - Reactions, Events & Edge Cases
  - Tournament Format

#### 2. **Vector Store**
- **Tipo**: InMemoryVectorStore (LangChain)
- **Embeddings**: OpenAI (`text-embedding-3-small`)
- **Chunking**: RecursiveCharacterTextSplitter
  - Chunk size: 500 caracteres
  - Overlap: 50 caracteres
  - Separadores: `\n\n`, `\n`, `. `, ` `

#### 3. **Herramientas (Tools)**
- **kb_search**: Búsqueda semántica en la KB de Quanta
- **wikipedia_search**: Búsqueda de información general en Wikipedia


#### 4. **Agente**
- **Framework**: LangChain
- **LLM**: OpenAI (GPT-4o-mini por default)


## Instalación y Uso

### Requisitos
- Python 3.8+
- Jupyter Notebook o JupyterLab
- API Key de al menos un proveedor(OpenAI, Anthropic, Google) en un file .env tal como en el ejempplo de env.sh

## Flujo de decisión

El agente usa el siguiente proceso de decisión:

```
   Pregunta Usuario  
           │
  ¿Menciona Quanta,reglas,           
   mecánicas, CP, Zones, Reactions?  
           │
      ┌────┴────┐
      │si       │no
      │         │
  kb_search  wikipedia_search    

```

## Análisis Casos de Prueba

| # | Consulta | Tools | Tiempo | Respuesta | Observaciones |
|---|----------|-------|--------|--------|---------------|
| 1 | Reactions y Catalyst | `kb_search` | 6.51s |si| Query compleja con 2 partes. Recuperó DOC_04 (Reactions) como principal. Explicó bien requisitos y Catalyst. |
| 2 | Formas de obtener CP | `kb_search` | 3.81s |si| Enumeración exitosa. Identificó las 3 formas. Respuesta estructurada ideal. Formato fácil lectura. |
| 3 | Cambiar entre partidas | `kb_search` | 3.88s |si| Query específica de nicho. Recuperó DOC_05 (Tournament). Identificó regla de swap de Zone tiles. Ruteo perfecto |
| 4 | Ada Lovelace | `wikipedia_search` | 7.02s |si| Información biográfica. Wikipedia retornó artículo completo. Contexto histórico preciso. URL citada correctamente |
| 5 | Angie Sanchez (error) | `wikipedia_search` | 3.44s |si| Caso crítico de manejo de error. Wikipedia "Page not found". NO alucinó información. Respuesta amigable. |
| 6 | Compara Quanta con ajedrez | `kb_search` + `wikipedia_search` | 15.11s |si| Multi-tool query. Usó ambas herramientas de forma autonoma. Sintetizó 4 fuentes. Comparación coherente|# genai_quanta
