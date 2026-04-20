# Sekretutka — Workflow Refactor Spec v1

**Дата:** 2026-04-09  
**Фокус:** low-token redesign первых трех workflow

## Цель
Зафиксировать практическую схему, по которой можно переписывать:
- [1. Sekretutka_main.json](/Users/alinavladimirova/Documents/Claude/Projects/Секретутка/1.%20Sekretutka_main.json)
- [Sekretutka_Siri_Gateway.json](/Users/alinavladimirova/Documents/Claude/Projects/Секретутка/Sekretutka_Siri_Gateway.json)
- [1.3 Quest_Architect_Daily_Planner.json](/Users/alinavladimirova/Documents/Claude/Projects/Секретутка/1.3%20Quest_Architect_Daily_Planner.json)

Так, чтобы:
- убрать монолитных агентов;
- сократить токены;
- вынести routing и валидацию в `code`;
- оставить AI только там, где он реально дает пользу.

---

## 1. Общий шаблон

```text
[User input]
  ↓
[AI: intent or extractor, JSON only]
  ↓
[Code: validation / routing / slot filling]
  ↓
[Sheets / workflow]
  ↓
[Code or short AI final reply]
```

Если ответ можно собрать шаблоном, финальный AI-вызов не нужен.

---

## 2. Workflow: `main`

## 2.1 Новая роль
`main` больше не должен быть “Секретуткой целиком”.
Он должен быть только:
- входной точкой;
- коротким router;
- state-aware dispatcher.

## 2.2 Минимальный входной контекст
`main` читает только:
- `PROFILE_GAME` snapshot keys:
  - `HP_CURRENT`
  - `CYCLE_DAY`
  - `PLAYER_STAGE`
  - `ACTIVE_ATTEMPT_ID`
  - `LEVEL_READY`
  - `DEBUFF_NAME`
- текущий `CURRENT_STATE` и `STATE_TIMESTAMP`, если хранится отдельно
- короткие алиасы привычек / правил из `GAME_RULES`
- task counters:
  - `today_task_count`
  - `overdue_task_count`

Не читать:
- весь `GAME_RULES`
- полный `Task List`
- полные planner sheets

## 2.3 Новая схема нод

```text
Telegram Trigger
  ↓
Voice/Text normalize
  ↓
Read minimal snapshot
  ↓
Read lightweight aliases
  ↓
AI Intent Classifier
  ↓
Code Router
  ↓
Execute target workflow
  ↓
Code final response
```

## 2.4 Что делает AI
Только:
- классифицирует intent;
- извлекает 1-2 сущности;
- возвращает JSON.

## 2.5 Что делает code
- проверяет active state override;
- решает, route locked или нет;
- добирает дефолты;
- проверяет хватает ли данных;
- задает 1 короткий уточняющий вопрос;
- вызывает нужный workflow;
- собирает финальный ответ.

## 2.6 Канонические маршруты
- `log_habit` -> `Quest Architect` в `HABIT_LOG`
- `add_task` -> `Quest Architect` в `TASK_CAPTURE`
- `plan_day` -> `Quest Architect` в `PLANNER`
- `decompose_goal` -> `Quest Architect` в `GOAL_DECOMPOSER`
- `log_finance` -> `finance`
- `therapist` -> `therapist`
- `rules_onboarding` -> `game_rules`
- `rules_edit` -> `game_rules`
- `read_status` -> временный direct response в `main` до новой render-версии
- `engine_notice` -> `engine`

## 2.7 Что вырезать из текущего `main`
- большой system prompt;
- memory;
- tool-calling agent;
- reasoning про `stage`, `level`, `attempt`;
- старые сущности `PROFILE`, `GAME_STAGE`, `LEVEL_ID`;
- hardcoded привычки в prompt.

## 2.8 AI budget
- 1 короткий AI call на intent
- 0 AI calls на финальный ответ в большинстве случаев
- 1 extra short AI call только если нужен humanized reply

---

## 3. Workflow: `Siri Gateway`

## 3.1 Новая роль
`Siri Gateway` не должен иметь отдельный “умный мозг”.
Он должен быть тонким голосовым входом в ту же router-архитектуру.

## 3.2 Новая схема нод

```text
Webhook
  ↓
Normalize Siri payload
  ↓
Speech-to-text if needed
  ↓
Read minimal snapshot
  ↓
AI Intent Classifier
  ↓
Code Router
  ↓
Execute target workflow
  ↓
Code voice response compressor
  ↓
Webhook response
```

## 3.3 Что общее с `main`
Один и тот же classifier pattern:
- те же intents
- те же сущности
- те же routing rules

Разница только в выходе:
- для Siri ответ максимум 2 предложения;
- без markdown;
- без статус-баров;
- без лишних деталей.

## 3.4 Что вырезать из текущего `Siri Gateway`
- отдельный большой system prompt;
- чтение полного `GAME_RULES`;
- старый `PROFILE`;
- отдельный голосовой агент;
- placeholder spreadsheet ids.

## 3.5 Voice response policy
Code после workflow делает:
- чистку markdown;
- сжатие до 1-2 предложений;
- вынос только результата и одного next step.

Пример:
- "Записала расход 250 бат на еду. Если хочешь, сразу скажи следующий платеж."

## 3.6 AI budget
- 1 короткий intent call
- 0 AI calls на финальный ответ

---

## 4. Workflow: `Quest Architect`

## 4.1 Новая роль
Один workflow допустим, но только с тремя детерминированными ветками:
- `TASK_CAPTURE`
- `PLANNER`
- `GOAL_DECOMPOSER`

Опционально:
- `EVENING_REVIEW`

## 4.2 Новая схема нод

```text
Input(mode, user_message)
  ↓
Read compact context
  ↓
Switch by mode
  ↓
AI extractor/synthesizer
  ↓
Code validation
  ↓
Sheets write
  ↓
Code final response
```

## 4.3 Ветка `TASK_CAPTURE`
Использовать, когда игрок говорит:
- "добавь задачу"
- "надо сделать"
- "запиши"

### AI получает
- `user_message`
- минимальный task taxonomy

### AI возвращает
- title
- due/start date hints
- category
- priority
- recurring

### Code делает
- дефолты;
- validation;
- 1 уточнение при нехватке ключевого слота;
- запись в `Task List`;
- optional event row в `GAME_LOG`.

### AI budget
- 0-1 calls

## 4.4 Ветка `PLANNER`
Использовать, когда игрок хочет:
- спланировать день;
- разобрать приоритеты;
- сделать утренний или вечерний check-in.

### AI получает
- `hp_current`
- `cycle_day`
- `today_task_count`
- `overdue_task_count`
- `task_candidates` в сжатом виде
- `active_goals_summary`

### AI возвращает
- до 3 фокусов;
- до 2 вопросов;
- при необходимости `suggest_decomposition = true`

### Code делает
- может создавать/обновлять задачи только после явного подтверждения;
- формирует итоговый короткий план;
- логирует relevant events в `GAME_LOG`, если это предусмотрено.

### AI budget
- 1 call

## 4.5 Ветка `GOAL_DECOMPOSER`
Использовать:
- после онбординга;
- когда игрок просит разбить цель;
- когда planner понимает, что цель слишком большая.

### AI получает
- только одну цель;
- возможно один срок;
- возможно подсказку по треку.

### AI возвращает
- `goal_summary`
- `steps[]` максимум 10
- `shared_deadline`
- `reward_suggestion`

### Code делает
- очищает старый goal block при необходимости;
- раскладывает шаги по колонкам:
  - `G, L, Q, V, AA, AF, AK, AP, AU, AZ`
- раскладывает deadlines:
  - `H, M, R, W, AB, AG, AL, AQ, AV, BA` строка `37`
- раскладывает rewards:
  - те же колонки строка `38`
- не пишет в другие зоны `PROFILE_GAME`.

### AI budget
- 1 call

## 4.6 Что вырезать из текущего `Quest Architect`
- старые `RESOURCE_LOG & BOSS_LOG`
- `KPI&HABBIT_LOG` как основной event source
- heavy agent behavior
- длинный reasoning на каждый запрос
- попытки вести “полную игру” вместо своей узкой зоны

---

## 5. Event strategy

## 5.1 Канон
Новый источник игровых событий — `GAME_LOG`.

## 5.2 Что это значит для первых трех workflow
- `main` сам обычно не пишет events
- `Siri Gateway` сам не пишет events
- `Quest Architect` пишет только свои релевантные события

Примеры:
- `TASK_CREATED`
- `TASK_STATUS_CHANGED`
- `GOAL_DECOMPOSED`
- `PLANNER_SESSION_STARTED`
- `PLANNER_SESSION_COMPLETED`

Точный словарь `EVENT_TYPE` еще нужно канонизировать отдельно.

---

## 6. Compression strategy

## 6.1 Что обязательно сжимать перед AI
- full rules -> aliases / mini dictionaries
- full task list -> counters + top 3 candidates
- profile snapshot -> 6-8 key fields
- history -> 0-3 relevant items max

## 6.2 Что вообще не отправлять в AI
- необязательные sheet rows
- старые logs
- сырые формулы
- UI coordinates

## 6.3 Когда AI можно убрать совсем
- если intent понятен regex-ом;
- если действие шаблонное;
- если ответ стандартный;
- если route уже заблокирован `current_state`.

---

## 7. Migration order

1. Переписать `main` под short intent router
2. Переписать `Siri Gateway` на тот же classifier pattern
3. Переписать `Quest Architect` на 3 режима
4. Потом уже тянуть за ними:
   - `track_progress`
   - `game_rules`
   - `engine`
   - `therapist`
   - `lk_render`

---

## 8. Definition of done

Рефакторинг первых трех workflow считается успешным, если:
- нет больших агентных prompt-ов;
- нет memory в `main`;
- нет полного `GAME_RULES` в prompt;
- `Siri` отвечает в 1-2 предложениях без отдельного агента;
- `Quest Architect` работает по mode-based веткам;
- decomposition ограничен 10 шагами;
- routing и validation живут в `code`;
- AI отвечает JSON-структурой, а не свободным полотном.
