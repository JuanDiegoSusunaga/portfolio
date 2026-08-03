# Crypto Intelligence Engine

> Motor cuantitativo de trading algorítmico (paper trading) sobre BTC/ETH/SOL/XRP: ingesta multi-fuente 24/7, ML con validación walk-forward, gestión de riesgo estadística y operación autónoma en GCP con alertas por Telegram.

| | |
|---|---|
| **Estado** | Pausado (corrió 24/7 en GCP · may–jul 2026) |
| **Tipo** | Personal / investigación cuantitativa |
| **Repo** | Privado |
| **Modo** | Paper trading — sin dinero real por diseño |
| **Mi rol** | Diseño y desarrollo completo |

## Idea

Un sistema que observe el mercado cripto como lo haría un desk cuantitativo pequeño: datos de precio, derivados, on-chain, macro y noticias convergen en una tabla maestra de features; modelos estadísticos generan señales; un gestor de riesgo dimensiona posiciones; y todo reporta a Telegram sin intervención humana. El objetivo era aprender **si el edge existe y se puede medir** — por eso paper trading: el broker real es un stub deliberado.

## Arquitectura

```
Binance · OKX · Deribit · FRED · BigQuery on-chain · DeFiLlama · CoinGecko · RSS
        → ingesta asyncio (15 jobs cron APScheduler, un solo event loop)
        → DuckDB (~24 tablas) → master_features (ancla horaria, fuentes diarias forward-filled)
        → XGBoost (dirección 4h) + HMM (régimen) + EVT (stops) + Kelly (sizing)
        → paper trades + alertas Telegram (outbox SQLite transaccional)
```

- **Ingesta:** 17 módulos concurrentes (httpx + semáforos); DuckDB es single-writer, así que los upserts se serializan con `asyncio.to_thread` mientras el fanout HTTP corre en paralelo.
- **Capas cuantitativas:** XGBoost + SHAP con walk-forward de 5 folds y reentrenamiento semanal; HMM gaussiano de 3 regímenes; stops por teoría de valores extremos (EVT/GPD); sizing ½-Kelly con cap del 20 % por posición y tracker de edge con intervalos de Wilson; TCN en shadow mode.
- **Operación:** systemd en una VM e2-small + **watchdog externo por cron** que vigila el servicio, la frescura de 7 tablas y los heartbeats por job, alertando por Telegram sin pasar por el proceso principal. Kill-switch persistente toggleable desde Telegram (`/halt`, `/resume`) sin reiniciar.
- **Capa LLM:** agentes de research/trading (Gemini vía Vertex AI) con 8 herramientas de function-calling y **cost cap de $5 USD/día** con contabilidad de tokens en base de datos.

## Decisiones tomadas contra los datos

La parte de la que más aprendí no fue construir capas sino **apagarlas**:

- La estrategia de IV-fade se retiró tras un backtest de 5 años (profit factor 0,42 — pierde dinero consistentemente).
- El agente RL (stable-baselines3) quedó deshabilitado: Sharpe −0,58 en validación.
- El universo de trading se recortó de top-50 a 4 pares: los símbolos sin cobertura del modelo ML tenían win rate 23 % vs. 44 % de los cubiertos.
- Un rediseño de features "estacionarias" se revirtió porque empeoraba el walk-forward.

## Resiliencia ante proveedores hostiles

- Binance Futures y Bybit geo-bloquean IPs de GCP (HTTP 451/403) → migración de toda la capa de derivados a OKX en 3 días.
- El free tier de CoinGecko devuelve 429 sostenido desde GCP → fail-fast con caída a whitelist curada.
- Las alertas Telegram salen por un **outbox SQLite con reintentos**: si Telegram o la DuckDB fallan, no se pierde nada.

## Stack

- **Python 3.12** gestionado con uv (lockfile commiteado, ruff en hooks)
- **Datos:** DuckDB + Polars · Parquet como archivo frío
- **ML:** XGBoost, SHAP, hmmlearn, scipy (EVT), scikit-learn, stable-baselines3
- **Infra:** GCP Compute Engine (systemd + cron watchdog), Vertex AI Custom Jobs para entrenar el TCN, BigQuery para datos on-chain
- **Interfaz:** CLI Typer (13 sub-apps) · bot de Telegram en español (8 comandos)

## Números

| Métrica | Valor |
|---|---|
| Código | ~15.500 líneas en 89 módulos |
| Tests | 119 (seguridad, Kelly, EVT, universo, riesgo) |
| Jobs programados | 15 (de 5 min a semanales, con offsets anti-colisión) |
| Features del modelo | 38 · ~44 columnas en `master_features` |
| Datos | 5 años de backfill · ~140K velas · macro desde 2010 |
| XGBoost (walk-forward) | PF 1,18–1,29 · WR 53–55 % · accuracy direccional ~45 % |
| Riesgo | ≤20 %/posición · ≤80 % exposición · stop diario 3 % · cooldown 2 h |

## Estado y lecciones

Corrió de forma autónoma en GCP entre mayo y julio de 2026; el proyecto se pausó al cerrar la cuenta de facturación de GCP (julio 2026). El código y la metodología quedan como base para una siguiente iteración.

- **La infraestructura es el 80 %**: heartbeats, watchdog, outbox e idempotencia consumieron más diseño que los modelos — y son lo que permitió confiar en los resultados.
- **Un backtest honesto vale más que una estrategia bonita**: tres componentes se apagaron por evidencia; el sistema termina siendo tanto un filtro de ideas como un generador de señales.
- **Paper trading primero no es timidez, es método**: separa el problema de "¿hay edge?" del de "¿puedo ejecutarlo?", y el broker stub documenta explícitamente qué faltaría para ir a real.
