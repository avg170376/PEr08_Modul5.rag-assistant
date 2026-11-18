# RAG-ассистент по документации

Консольный ассистент с поиском по локальной базе знаний (Retrieval-Augmented Generation). Вопрос ищется в фрагментах `docs.txt`, затем языковая модель отвечает по найденному контексту. Повторные ответы сохраняются в SQLite.

Два независимых режима:

| Папка | Модель | Эмбеддинги | Ключи |
|---|---|---|---|
| `assistant_api` | OpenAI `gpt-4o-mini` | OpenAI | `OPENAI_API_KEY` |
| `assistant_giga` | GigaChat | GigaChat | `GIGACHAT_AUTH_KEY`, `GIGACHAT_RQUID` |

Оценка качества OpenAI-режима — `assistant_api/evaluate_ragas.py` (метрики RAGAS: Faithfulness и Context Precision).

## Требования

- Python 3.11 (проверялось на 3.11.9)
- Интернет: эмбеддинги и ответы запрашиваются у OpenAI или GigaChat

## Установка

```powershell
py -3.11 -m venv venv_py311
.\venv_py311\Scripts\activate
python -m pip install --upgrade pip
pip install -r requirements.txt
```

Создайте в корне файл `.env` и заполните ключи нужного режима:

```
OPENAI_API_KEY=

GIGACHAT_RQUID=
GIGACHAT_AUTH_KEY=
```

`GIGACHAT_AUTH_KEY` — Authorization Key из личного кабинета [GigaChat API](https://developers.sber.ru/docs/ru/gigachat/quickstart/ind-using-api) (проект → Настройки API → Получить ключ).

`GIGACHAT_RQUID` — не секрет. Это идентификатор запроса в формате UUID v4, его нужно сгенерировать самим:

```powershell
[guid]::NewGuid().ToString()
```

Файл `.env` в репозиторий не входит.

## Запуск

Команды выполняются из папки режима: пути к `data/docs.txt`, ChromaDB и кэшу относительные.

OpenAI:

```powershell
.\venv_py311\Scripts\activate
cd assistant_api
python app.py
```

GigaChat:

```powershell
.\venv_py311\Scripts\activate
cd assistant_giga
python app.py
```

В консоли:

- вопрос — обычный текст
- `stats` — число документов и записей кэша
- `clear` — очистить кэш
- `exit` или `quit` — выход

Первый запуск индексирует `data/docs.txt` в ChromaDB. Следующие запуски открывают уже созданную коллекцию.

## Оценка RAGAS

Только для режима OpenAI. Нужен `OPENAI_API_KEY`. Скрипт задаёт пять вопросов из `assistant_api/data/docs.txt` и считает Faithfulness и Context Precision.

```powershell
.\venv_py311\Scripts\activate
cd assistant_api
python evaluate_ragas.py
```

В конце печатаются средние значения метрик и итоговый балл. Эталон для Context Precision в этом скрипте — начало ответа самой модели, поэтому высокий балл показывает согласие ответа с найденным контекстом.

Версии LangChain и RAGAS в `requirements.txt` ограничены сверху: RAGAS 0.3.1 не импортируется с LangChain 1.x и `langchain-community` 0.4.
