# Evaluación de Agentes de IA

Materiales para la sesión de evaluación de agentes. El módulo combina teoría, un demo funcional y un workshop práctico donde cada estudiante construye y evalúa su propio agente.

---

## Estructura del módulo

| # | Documento | Contenido | Tiempo estimado |
|---|---|---|---|
| [01](01-intro.md) | Introducción | ¿Por qué evaluar agentes? Arquitectura del agente. Caso de estudio: Web Search Agent | 20 min |
| [02](02-model-vs-system.md) | Modelos vs sistemas | Benchmarks, determinismo, tipos de evaluación, observabilidad y trazas | 20 min |
| [03](03-metrics-llm-as-judge.md) | Métricas y LLM-as-judge | Evaluación humana, BLEU/ROUGE, LLM-as-judge, Correctness/Efficiency/Faithfulness, sesgos | 30 min |
| [04](04-evaluators-deep-dive.md) | Evaluadores: deep dive | agentevals completo — trajectory match (4 modos), LLM-as-judge variants, graph trajectory | 30 min |
| [05](03-workshop.md) | Workshop | Construye y evalúa tu propio agente | 90 min |

**Duración total:** ~3 horas

---

## Notebooks

| Notebook | Descripción |
|---|---|
| [`agent_evals.ipynb`](agent_evals.ipynb) | Demo completo — Web Search Agent con 6 evaluadores y LangSmith |
| [`workshop_agent_evals.ipynb`](workshop_agent_evals.ipynb) | Workshop para estudiantes — scaffolded con TODOs |

---

## Setup

```bash
# Instalar dependencias
pip install -r requirements.txt

# Variables de entorno (.env)
NVIDIA_API_KEY=nvapi-...       # o GROQ_API_KEY
LANGSMITH_API_KEY=lsv2-...     # opcional, para el dashboard
LANGCHAIN_TRACING_V2=true      # opcional
LANGCHAIN_PROJECT=agent-evals  # opcional
```

---

## Figuras

Todas las figuras están en la carpeta `figs/`:

| Figura | Usada en |
|---|---|
| `agent_loop.png` | 01 |
| `research_agent_eval.png` | 01 |
| `agent_web_search.svg` | 01, 02 |
| `data_analyzer_agent.png` | 01 |
| `llm_model_eval.png` | 02 |
| `llm_system_eval.png` | 02 |
| `web_search_agent_graph.png` | 02 |
| `prompt_regression.svg` | 02 |

---

## Flujo de la sesión

```
01 Intro (20 min)
    ↓
02 Modelos vs sistemas (20 min)
    ↓
03 Métricas + LLM-as-judge (30 min)
    ↓  ← pausa recomendada
04 Deep dive evaluadores (30 min)
    ↓
Demo en vivo: agent_evals.ipynb (20 min)
    ↓
05 Workshop (90 min)
```

---

## Conceptos clave por sección

**01–02:** agente = LLM + tools + loop · trayectoria · evaluación end-to-end vs por componentes

**03:** evaluación humana (Kappa de Cohen) · ROUGE ≈ `evaluate_retrieval` · LLM-as-judge · correctness / efficiency / faithfulness · position bias · verbosity bias · self-enhancement bias

**04:** `trajectory_match` (strict / unordered / subset / superset) · `tool_args_match_mode` · `graph_trajectory` · async · cuándo usar cada evaluador

**Workshop:** `evaluate_tool_use` (TODO) · `evaluate_retrieval` · `multidim_judge` · LangSmith · ciclo evaluar → mejorar → evaluar
