# LLM, агенты и harness

От предсказания токена до приложения с инструментами, поиском и памятью. Раздел объясняет границы модели и внешнего кода, а также способы проверки результата.

Материал адаптирован из исходного конспекта «LLM, Agents и Harness — от токена до автономного агента». Примеры контрактов условные; раздел не привязан к SDK или конкретной модели.

## Порядок чтения

| Файл | Что внутри |
|---|---|
| [Основы LLM](./01-llm-fundamentals.md) | Токены, embeddings, Transformer, sampling, контекст и KV-cache |
| [Обучение модели](./02-training.md) | Pretraining, SFT, RLHF/RLAIF, DPO и роли сообщений |
| [Inference](./03-inference.md) | Передача истории, streaming, лимиты, prompt caching и стоимость |
| [Tool use](./04-tool-use.md) | Схемы функций, выполнение, результаты, параллельность и deferred tools |
| [Harness](./05-harness.md) | Агентный цикл, состояния, разрешения, служебный контекст и compaction |
| [Агенты и подагенты](./06-agents.md) | Workflow, fresh/fork, изоляция контекста и оркестрация |
| [MCP](./07-mcp.md) | Host/client/server, инструменты, ресурсы и транспорты |
| [RAG и память](./08-rag-and-memory.md) | Chunking, поиск, long context, fine-tuning и persistent memory |
| [Надёжность](./09-reliability.md) | Hallucinations, evals, prompt injection, guardrails и reasoning budget |

## Глоссарий

| Термин | Суть |
|---|---|
| Токен | Элемент словаря, с которым работает модель |
| Embedding | Векторное представление токена или текста; назначение зависит от модели |
| Transformer | Архитектура с механизмами attention и преобразованиями представлений |
| Self-attention | Взвешенное использование представлений доступных позиций |
| Context window | Ограничение объёма контекста для обращения к модели |
| KV-cache | Сохранённые Keys и Values прошлых позиций для инференса |
| Temperature / top-k / top-p | Настройки выбора токена из распределения |
| Pretraining | Предобучение на большом корпусе |
| SFT | Дообучение на примерах желаемого поведения |
| RLHF / RLAIF | Обучение с обратной связью людей или AI-систем |
| DPO | Прямая оптимизация по предпочтениям без отдельного RL-цикла |
| System prompt | Инструкции в системной роли API; не механизм контроля доступа |
| Prompt caching | Переиспользование вычислений для повторяющегося входа |
| Tool use / function calling | Генерация моделью структурированного запроса на внешнее действие |
| Tool call / tool result | Связанные идентификатором запрос инструмента и результат выполнения |
| Deferred tools | Загрузка полных описаний инструментов по необходимости |
| Harness | Приложение, управляющее взаимодействием модели со средой |
| Agentic loop | Цикл выбора шага, действия и обработки результата |
| Permission mode | Политика разрешения, подтверждения или запрета действий |
| System-reminder | Пример соглашения о служебной разметке в конкретном harness |
| Compaction | Сжатие истории для освобождения текущего контекста |
| Agent | Система, в которой модель динамически выбирает ход работы |
| Subagent | Отдельный агентный цикл для подзадачи |
| Fresh / fork | Запуск с явно заданным контекстом или с его унаследованным снимком |
| MCP | Протокол интеграции приложений с внешними возможностями и данными |
| RAG | Генерация с использованием извлечённых внешних данных |
| Chunking | Разделение документов на фрагменты для обработки и поиска |
| Persistent memory | Сохранённые вне контекста данные для последующих сессий |
| Hallucination | Неверное или неподтверждённое утверждение модели |
| Evals | Процедуры измерения качества и поведения системы |
| Guardrails | Ограничения и проверки поведения системы |
| Prompt injection | Попытка внедрить инструкции через недоверенный вход |
| Reasoning effort / budget | Настройка дополнительных вычислений на рассуждение |

## Первичные источники

- [Attention Is All You Need](https://arxiv.org/abs/1706.03762) — исходная архитектура Transformer.
- [Direct Preference Optimization](https://arxiv.org/abs/2305.18290) — оптимизация по предпочтениям без отдельного RL-цикла.
- [Hugging Face — Cache strategies](https://huggingface.co/docs/transformers/en/kv_cache) — KV-cache и переиспользование префиксов.
- [Anthropic — Building effective agents](https://www.anthropic.com/research/building-effective-agents) — различие workflow и агента, паттерны оркестрации.
- [MCP — Architecture](https://modelcontextprotocol.io/docs/learn/architecture) — роли и основные возможности протокола.
- [MCP — Transports, версия 2025-11-25](https://modelcontextprotocol.io/specification/2025-11-25/basic/transports) — конкретная версия транспортного контракта.
- [Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks](https://arxiv.org/abs/2005.11401) — сочетание генерации и внешнего поиска.

## Связанные разделы

- [ASP.NET Core Pipeline](../aspnet-pipeline/README.md)
- [Тестирование в .NET](../testing/README.md)
- [Микросервисы](../microservices/README.md)
