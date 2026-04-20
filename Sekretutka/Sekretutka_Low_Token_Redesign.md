# Sekretutka — Low-Token Workflow Redesign

## 1. Цель
Пересобрать систему так, чтобы:
- AI использовался только там, где без него нельзя;
- большая часть логики жила в `code`, Google Sheets и явных правилах;
- каждый workflow имел короткий и предсказуемый токеновый профиль;
- монолитные агенты были заменены на каскад:
  - intent detection
  - structured processing
  - короткий финальный ответ

Базовый шаблон:

```text
[Запрос от юзера]
   ↓
[AI: короткий intent JSON]
   ↓
[Sheets / Code / Rules]
   ↓
[AI: короткий финальный ответ]
```

Если `code` и lookup-логика полностью покрывают сценарий, второй AI-вызов тоже можно убрать.

---

## 2. Общие правила low-token архитектуры

### 2.1 AI не должен быть главным оркестратором
Нельзя использовать один большой агент, который:
- читает весь профиль;
- читает все правила;
- сам решает маршрут;
- сам же пишет финальный текст;
- еще и вызывает инструменты.

Это самая дорогая и самая хрупкая схема.

### 2.2 Каждый AI-вызов должен быть узким
Допустимые типы AI-вызовов:
- `Intent Classifier`
- `Structured Extractor`
- `Therapist Core`
- `Planner Synthesizer`
- `Short Response Composer`

Недопустимый тип:
- "суперагент, который знает все и решает все"

### 2.3 Сначала правила, потом модель
Если действие можно определить так:
- по ключевым словам;
- по regex;
- по точному словарю;
- по lookup в `GAME_RULES`;
- по состоянию `CURRENT_STATE`;
то это должно жить в `code`, а не в LLM.

### 2.4 В AI передается только сжатый контекст
Нельзя передавать в LLM:
- весь `GAME_RULES`;
- весь `PROFILE_GAME`;
- весь `Task List`;
- длинную memory-историю.

Нужно передавать только:
- нужные поля snapshot;
- только релевантные rows;
- короткое summary, собранное кодом.

### 2.5 Один workflow — одна роль
Каждый workflow должен отвечать за одну ясную работу:
- route
- log habit
- log finance
- run onboarding
- decompose goal
- run therapist
- render cabinet

Не смешивать это в одного "умного помощника".

---

## 3. Канонические паттерны

## 3.1 Intent-first
Использовать, когда пользователь пишет свободным текстом.

Формат intent-ответа:

```json
{
  "intent": "log_habit | add_task | plan_day | log_finance | therapist | onboarding | rules_edit | read_status | decompose_goal | unknown",
  "confidence": 0.0,
  "entities": {},
  "needs_clarification": false
}
```

Ограничения:
- максимум 1 короткий system prompt;
- максимум 1 короткий user message;
- ответ только JSON;
- без длинных reasoning-абзацев.

## 3.2 Code-first slot filling
После intent-classifier:
- `code` нормализует поля;
- `code` проверяет обязательные слоты;
- если данных не хватает, возвращается короткий уточняющий вопрос;
- если хватает, workflow идет в Sheets.

## 3.3 Response-last
Финальный AI-вызов нужен только если надо:
- красиво, но коротко объяснить результат;
- адаптировать тон под HP / cycle / state.

Если ответ шаблонный, использовать `code` вместо AI.

---

## 4. Глобальная архитектура системы

## 4.1 Main Router
Главный workflow должен стать не агентом, а маршрутизатором.

### Текущая проблема
[1. Sekretutka_main.json](/Users/alinavladimirova/Documents/Claude/Projects/Секретутка/1.%20Sekretutka_main.json) сейчас:
- держит большой system prompt;
- тянет старый `PROFILE` (историческая проблема; active-core уже переведён на `PROFILE_GAME`);
- использует агент с tool-calling;
- содержит старую игровую логику;
- платит токены за routing, reasoning и generation одновременно.

### Целевая low-token схема

```text
Telegram Trigger
  ↓
Read minimal snapshot summary
  ↓
Intent Classifier AI (короткий JSON)
  ↓
Code Router
  ↓
Target workflow
  ↓
Template or Short Response Composer
```

### Что должно читать
Не весь `PROFILE_GAME`, а только сжатый summary:
- `HP_CURRENT`
- `CYCLE_DAY`
- `PLAYER_STAGE`
- `ACTIVE_ATTEMPT_ID`
- `LEVEL_READY`
- `CURRENT_STATE`
- `ACTIVE_DEBUFF_NAME`

### Что должно исчезнуть
- большой сюжетный system prompt;
- самостоятельный reasoning про stage / level;
- tool-routing через свободный агент;
- старая логика `GAME_STAGE`, `LEVEL_ID`.

---

## 4.2 ENGINE

### Текущая проблема
[1.1 Sekretutka_ENGINE_updated.json](/Users/alinavladimirova/Documents/Claude/Projects/Секретутка/1.1%20Sekretutka_ENGINE_updated.json) уже без AI, но логика неверна:
- читает `STAGE_RULES`;
- пишет в `RESOURCE_LOG & BOSS_LOG`;
- опирается на старый `PROFILE`;
- использует старые attempt statuses;
- сжигает `XP_Gold` неверно или смешивает его с `XP_TOTAL`;
- не учитывает, что `XP_TOTAL = cumulative(0.5 × XP_Gold gains + 0.5 × HP-positive gains)`.

### Целевая роль
ENGINE должен быть полностью `code + sheets`, без AI.

### Что делает
- nightly stale check;
- attempt fail logic;
- debuff expiry;
- level readiness check;
- saturday sheet generation;
- message assembly.

### Low-token правило
Не использовать AI даже для сообщений, пока сообщения можно собрать шаблонами.

### Целевая схема

```text
CRON
  ↓
Read compact inputs
  - PROFILE_GAME snapshot block
  - GAME_LOG recent rows
  - BELIEFS recent rows
  - WALLET recent rows
  - Task List counters
  ↓
Code: derive
  - stale_days
  - attempt status
  - stage
  - readiness
  - deltas
  ↓
Append GAME_LOG events
  ↓
Optional update of goal/snapshot support rows
  ↓
Code/template final Telegram message
```

### Отдельный принцип
ENGINE не должен читать огромные таблицы полностью.
Ему нужны:
- recent events window;
- агрегаты;
- ключевые snapshot fields.

---

## 4.3 Game Rules / Onboarding

### Текущая проблема
[1.0 Sekretutka_Game Rules.json](/Users/alinavladimirova/Documents/Claude/Projects/Секретутка/1.0%20Sekretutka_Game%20Rules.json) использует AI как длинный FSM-мозг и все еще смешивает старые листы.

### Целевая low-token модель
Onboarding делится на два слоя:
- FSM state machine в `code`
- AI только для коротких вопросов и нормализации свободных ответов

Канонический порядок блоков:
- `INTRO`
- `BIO`
- `SPRINT_KPI`
- `HABITS`
- `TASKS`
- `BOSSES_DEBUFFS`
- `SHOP`
- `GOAL`
- `LEVEL_NAMES`
- `CONTRACT`
- `ACTIVATE`

### Рекомендуемая схема

```text
[state + user answer]
   ↓
[Code FSM]
   - знает текущий state
   - знает обязательные поля
   - знает следующий state
   ↓
[AI Extractor]
   - только если ответ свободный и нужен парсинг
   ↓
[Code Validation]
   ↓
[Build GAME_RULES rows]
   ↓
[Build GAME_LOG rows]
   ↓
[Short response]
```

### Где AI нужен
- распознать свободную формулировку goal;
- нормализовать level names;
- структурировать habits / KPI из естественного текста.

### Где AI не нужен
- переходы по state;
- определение обязательных полей;
- формирование rows для Sheets;
- запуск первого attempt;
- handoff на goal decomposition.

---

## 4.4 Quest Architect

### Текущая проблема
[1.3 Quest_Architect_Daily_Planner.json](/Users/alinavladimirova/Documents/Claude/Projects/Секретутка/1.3%20Quest_Architect_Daily_Planner.json) использует большой агент для задач и planning, а еще пишет в старые логи.

### Целевая роль
Quest Architect делится на 3 независимых режима:
- `task_capture`
- `planner`
- `goal_decomposer`

### 4.4.1 Task capture
Для простого запроса "добавь задачу" AI почти не нужен.

Схема:

```text
Intent: add_task
  ↓
Code slot extraction
  - task
  - recurring
  - due_date
  - category
  - track
  ↓
Clarify if needed
  ↓
Write Task List
  ↓
Template response
```

### 4.4.2 Planner
Для morning/evening planning AI нужен, но только на synthesis-слое.

Схема:

```text
Build compact planning context in code
  - today's tasks
  - overdue count
  - cycle
  - hp
  - active goals
  ↓
Planner AI
  - only propose structure/questions
  ↓
Code parse
  ↓
Task writes / updates
  ↓
Short response
```

### 4.4.3 Goal decomposer
Этот режим можно сделать очень дешевым:
- AI получает только одну цель;
- AI возвращает максимум 10 steps;
- `code` раскладывает их по ячейкам;
- reward/deadline можно спрашивать отдельно или предлагать шаблонно.

### Очень важно
Quest Architect не должен писать:
- в `RESOURCE_LOG & BOSS_LOG`;
- в старые habit-логи как главный event source.

Его записи:
- `Task List`
- goal decomposition zone в `PROFILE_GAME`
- при необходимости игровые события в `GAME_LOG`

---

## 4.5 Therapist

### Текущая проблема
[1.5 Sekretutka_Therapist.json](/Users/alinavladimirova/Documents/Claude/Projects/Секретутка/1.5%20Sekretutka_Therapist.json) — один из немногих workflow, где AI действительно нужен. Но сейчас он дорогой:
- длинный system prompt;
- memory на 30 сообщений;
- широкий контекст;
- старая игровая терминология.

### Целевая low-token модель
Therapist остается AI-heavy, но сжимается.

### Что сократить
- memory window заменить на:
  - текущая жалоба
  - 3 последних релевантных belief
  - snapshot: `cycle_day`, `hp`, `attempt_id`, `stage`
- system prompt укоротить до терапевтического ядра;
- output строго JSON.

### Что оставить AI
- только core therapeutic transformation.

### Что вынести в code
- выбор режима `discussion/final`;
- формирование контекста;
- обрезка history;
- запись в `BELIEFS`;
- запись event в `GAME_LOG`;
- расчет, надо ли списывать `HP_COST`.

---

## 4.6 Track Progress

### Текущая проблема
[1.6 Sekretutka_track_progress.json](/Users/alinavladimirova/Documents/Claude/Projects/Секретутка/1.6%20Sekretutka_track_progress.json) без AI, но использует старую архитектуру:
- `KPI&HABBIT_LOG`
- `RESOURCE_LOG & BOSS_LOG`
- жестко зашитые строки daily planner.

### Актуальный статус
Этот блок теперь исторический: отдельный `track_progress` больше не считается частью active-core.
Логика привычек перенесена в `Quest Architect / HABIT_LOG`, а сам файл рассматривается как legacy.

```text
input(item_id, value, date, status)
  ↓
lookup GAME_RULES
  ↓
code validation
  ↓
append GAME_LOG event
  ↓
optional planner tick/update
  ↓
template response
```

### Что тут вообще не нужно
- AI
- свободный reasoning
- второй лог параллельно с `GAME_LOG`

---

## 4.7 Finance

### Статус
[1.4. Sekretutka_finance.json](/Users/alinavladimirova/Documents/Claude/Projects/Секретутка/1.4.%20Sekretutka_finance.json) пользователь считает рабочим.

### Low-token оценка
Это хороший ориентир по стилю:
- AI нет;
- deterministic row write;
- минимум логики.

### Что можно улучшить позже
- проверить workbook id;
- убрать лишний read-before-write, если append index можно считать проще;
- парсинг сумм и валют вынести в upstream workflow.

Но по вашей пометке пока его не трогаем.

---

## 4.8 LK Render

### Текущая проблема
[1.7. Sekretutka_LK_Render.json](/Users/alinavladimirova/Documents/Claude/Projects/Секретутка/1.7.%20Sekretutka_LK_Render.json) сидит на старом workbook и старом профиле.

### Целевая low-token модель
LK Render должен быть полностью без AI:

```text
Read compact snapshot
Read active tasks
Read active debuffs/bosses if needed
  ↓
Code render message
  ↓
Telegram
```

Никакого AI тут не нужно.

---

## 4.9 Siri Gateway

### Текущая проблема
[Sekretutka_Siri_Gateway.json](/Users/alinavladimirova/Documents/Claude/Projects/Секретутка/Sekretutka_Siri_Gateway.json) использует голосового агента.

### Целевая low-token модель
Siri должен переиспользовать Main Router pattern:

```text
Webhook
  ↓
Parse audio text
  ↓
Intent classifier AI (короткий)
  ↓
Code route
  ↓
Workflow
  ↓
Code short voice response
```

Для Siri особенно важно:
- не тянуть длинный prompt;
- не использовать tool-calling agent;
- держать ответ в 1-2 предложениях.

---

## 5. Новый workflow-пакет

## 5.1 Обязательные workflow после рефакторинга
- `main_router`
- `engine_core`
- `onboarding_rules`
- `task_capture`
- `planner_daily`
- `goal_decomposer`
- `track_progress`
- `finance`
- `therapist`
- `lk_render`
- `siri_gateway`

## 5.2 Что можно объединить
Можно оставить общий Quest Architect workflow, но только если внутри у него будут разные deterministic ветки:
- `mode = task_capture`
- `mode = planner`
- `mode = goal_decomposer`

То же для onboarding:
- FSM в одном workflow допустима, если она кодовая, а не агентная.

---

## 6. AI budget by workflow

### `main_router`
- 1 короткий AI call на intent
- 0 или 1 короткий AI call на финальную формулировку

### `engine_core`
- 0 AI calls

### `onboarding_rules`
- 0-1 AI call на ход, только если нужен extractor

### `task_capture`
- 0 AI calls в простых случаях
- 1 короткий AI extractor в сложных

### `planner_daily`
- 1 AI call на synthesis плана
- остальное code

### `goal_decomposer`
- 1 AI call на decomposition

### `track_progress`
- historical / legacy, вне active-core

### `finance`
- 0 AI calls

### `therapist`
- 1 AI core call

### `lk_render`
- 0 AI calls

### `siri_gateway`
- 1 AI intent call
- 0 или 1 short response call

---

## 7. Приоритет переписывания с учетом токенов

1. `1. Sekretutka_main.json`
Причина:
- сейчас это самый дорогой монолитный агент;
- он загрязняет всю систему старой логикой.

2. `Sekretutka_Siri_Gateway.json`
Причина:
- сейчас тоже агентный и лишне дорогой;
- его легко перевести на intent-first.

3. `1.3 Quest_Architect_Daily_Planner.json`
Причина:
- много задач можно перевести в deterministic code;
- decomposition идеально подходит под один короткий AI call.

4. `1.0 Sekretutka_Game Rules.json`
Причина:
- onboarding сейчас агентный, а должен быть code-driven FSM.

5. `1.1 Sekretutka_ENGINE_updated.json`
Причина:
- AI там нет, но логика неправильная;
- после переписывания router/onboarding станет проще правильно переделать event logic.

6. `1.6 Sekretutka_track_progress.json`
7. `1.5 Sekretutka_Therapist.json`
8. `1.7. Sekretutka_LK_Render.json`

---

## 8. Практический принцип для всех следующих правок
Если в workflow появляется вопрос:
"Нужен ли тут AI?"

Отвечать так:
- если нужно распознать смысл свободной речи -> короткий intent/extractor AI;
- если нужно применить правило -> `code`;
- если нужно записать данные -> Sheets;
- если нужно красиво подтвердить -> шаблон или короткий final AI;
- если нужно терапевтическое преобразование или полноценное планирование -> AI оправдан.

---

## 9. Следующий шаг
Следующий рациональный шаг — не переписывать все сразу, а сначала зафиксировать token-efficient спецификацию для первых трех workflow:
1. `1. Sekretutka_main.json`
2. `Sekretutka_Siri_Gateway.json`
3. `1.3 Quest_Architect_Daily_Planner.json`

Именно они дадут самый большой выигрыш по токенам и сразу зададут правильный паттерн для остальных.
