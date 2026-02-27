---
name: article-to-notebooklm
description: "Загрузка статей/веб-страниц в NotebookLM с генерацией материалов. Триггеры: 'проанализируй статью', 'скушай статью', 'загрузи статью в notebooklm', или ссылка на статью (не YouTube)."
allowed-tools: Bash, Read, Write, Edit, Glob, Grep, Task, AskUserQuestion
---

# Article → NotebookLM

## Goal
Принять URL статьи (или несколько), загрузить в NotebookLM как источник, предложить пользователю выбор материалов для генерации.

## Requirements
- NotebookLM MCP сервер (настроен и авторизован)
- Папка для скачивания артефактов: `~/article-to-notebooklm/downloads/`

## Process

### Step 1: Проверка авторизации
- Вызвать `notebook_list(max_results=1)` — если ошибка авторизации:
  1. Запустить Brave: `"/Applications/Brave Browser.app/Contents/MacOS/Brave Browser" --remote-debugging-port=18800 --no-first-run --no-default-browser-check &`
  2. Авторизоваться: `nlm login --provider openclaw --cdp-url http://127.0.0.1:18800`
  3. Обновить токены: `refresh_auth()`

### Step 2: Парсинг входа
- Определить тип: одна ссылка или несколько
- Если несколько URL — показать список, спросить подтверждение
- Отфильтровать YouTube-ссылки (для них есть отдельный скилл)

### Step 3: Категоризация
- Одна статья → один блокнот
- Несколько статей одной темы → один блокнот
- Разные темы → предложить варианты группировки (human-in-the-loop)
- Название блокнота: краткое, отражающее тему

### Step 4: Создание блокнота и загрузка источников
1. `notebook_create(title=...)`
2. Для каждого URL:
   - `source_add(notebook_id=..., source_type="url", url=<article_url>, wait=true)`
   - Если source_add с URL не сработал (paywall, JS-only):
     - Попробовать `WebFetch` для извлечения текста
     - Загрузить как `source_add(source_type="text", title="...", text=<extracted_text>, wait=true)`
3. Проверить что источники добавлены: `notebook_get(notebook_id=...)`
4. Создать папку для скачивания: `mkdir -p ~/article-to-notebooklm/downloads/`

### Step 5: Предложить действия (human-in-the-loop)
Спросить пользователя через AskUserQuestion (multiSelect=true):
- **Summary (отчёт)** — краткий обзор ключевых идей (Briefing Doc)
- **Учебный гайд** — структурированный Study Guide
- **Аудиоподкаст** — deep dive диалог
- **Презентация** — slide deck
- **Карточки** — flashcards для запоминания
- **Квиз** — тест по материалу
- **Mind Map** — визуальная карта связей
- **Оставить как есть** — только источники в блокноте

### Step 6: Генерация и скачивание
Для каждого выбранного материала:
1. `studio_create(notebook_id=..., artifact_type=..., confirm=True)`
2. Поллить `studio_status(notebook_id=...)` до завершения
3. `download_artifact(notebook_id=..., artifact_type=..., output_path=~/article-to-notebooklm/downloads/...)`
4. Сообщить путь к файлу

Маппинг выбора → artifact_type:
- Summary → report (report_format="Briefing Doc")
- Учебный гайд → report (report_format="Study Guide")
- Аудиоподкаст → audio (audio_format="deep_dive")
- Презентация → slide_deck
- Карточки → flashcards
- Квиз → quiz (question_count=10)
- Mind Map → mind_map

### Step 7: Анализ контента (self-improvement)
- Проанализировать содержание статьи на полезные инсайты
- Показать пользователю ключевые выводы
- Спросить подтверждение перед обновлением memory

## Rules
- ВСЕГДА проверять авторизацию NotebookLM перед началом (Step 1)
- YouTube-ссылки → перенаправлять на скилл youtube-to-notebooklm
- Если URL не открывается (paywall/JS) — fallback на WebFetch + text source
- Показывать прогресс на каждом этапе
- Предлагать несколько вариантов, не один
- Не создавать блокнот если все URL не загрузились — сообщить об ошибке
