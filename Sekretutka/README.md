# Секретутка — статус архитектуры и workflow

**Версия:** v4.2  
**Дата:** 2026-04-10  
**Статус:** ядро канона зафиксировано: `PRD + README` задают логику, живая CRM задает схему данных, а workflow-json приводятся к этому контуру.

---

## Канон
Игровым каноном сейчас считаются:
- [Sekretutka_PRD_v1.md](/Users/alinavladimirova/Documents/Claude/Projects/Секретутка/Sekretutka_PRD_v1.md)
- [Секретутка_Правила_Игры.docx](/Users/alinavladimirova/Documents/Claude/Projects/Секретутка/Секретутка_Правила_Игры.docx)
- живая Google Sheets CRM

Правило при конфликте:
- продуктовая логика и workflow-канон берутся из `PRD + README`;
- схема колонок, sheet names, ranges и layout берутся из живой CRM;
- [Sekretutka_Architecture_v3.html](/Users/alinavladimirova/Documents/Claude/Projects/Секретутка/Sekretutka_Architecture_v3.html) используется только как secondary reference до полной синхронизации;
- старые workflow не переопределяют канон и считаются legacy, пока не доказано обратное.

---

## Целевая модель

### Основные листы
- `GAME_RULES` — реестр правил, игровых сущностей, option sets, level names, habits, KPI, bosses, debuffs, shop и пользовательских игровых настроек.
- `GAME_LOG` — единый журнал игровых событий.
- `PROFILE_GAME` — гибридный лист: top goals + goal decomposition + snapshot игрока.
- `Task List` — единственный источник правды по задачам.
- `BELIEFS` — инсайты Therapist.
- `WALLET_Monthly Budget` — источник финансовых данных.

Технические ID-колонки в живой CRM:
- `Task List!Q = TASK_ID`
- `WALLET_Monthly Budget!L = ENTRY_ID`

### Игровые слои
- `Bio-Layer` — HP, цикл, симптомы, риск и восстановление.
- `Player Status` — `XP_TOTAL`, lifetime-опыт игрока.
- `Attempt` — период консистентного присутствия в игре.
- `Stage` — автоматически считается внутри текущего attempt.
- `Level` — прогрессия по уровню с ручным подтверждением level up.

### Канонические правила
- пока в `GAME_LOG` нет события `ONBOARDING_COMPLETED`, игра считается не начавшейся;
- ручные записи пользователя в листах до онбординга допустимы и считаются `pre-game data`, а не активным игровым состоянием;
- `pre-game data` можно читать и использовать как контекст, но оно не запускает attempt, не создает rule activation и не влияет на честный игровой расчет без подтвержденного workflow;
- первый attempt стартует сразу после онбординга;
- attempt имеет только статусы `ACTIVE` и `FAILED`;
- 25 часов молчания = `STUCK_TIME` (alert, требует помощи от Therapist, не меняет статус попытки);
- 5 дней без успешно записанной активности = `FAILED`;
- при провале attempt сгорает `XP_GOLD`, `XP_TOTAL` сохраняется;
- stage считается по прогрессу внутри текущего attempt:
  - `0–499` -> `EARLY`
  - `500–3499` -> `MID`
  - `3500+` -> `LATE`
- `TOTAL_PROGRESS` считается по прогрессу внутри текущего уровня, а не по lifetime XP;
- `LEVEL_READY = TRUE`, если:
  - `TOTAL_PROGRESS >= 1.0`
  - `BALANCE_THB >= 1 000 000`
- level up происходит только после подтверждения игроком.

---

## `PROFILE_GAME`

### Что это
`PROFILE_GAME` — не чистый snapshot-лист и не чистый dashboard.
Он совмещает:
- narrative/top goals;
- decomposition goals;
- snapshot игрока;
- отдельные derived значения.

### Goal zone
Канонические top goals сейчас живут в:
- `C5`
- `C7`
- `C9`
- `C11`
- `C13`
- `C15`
- `C17`
- `C19`
- `C21`
- `C23`

Важно:
- top goals живут в `PROFILE_GAME`, а не в `Task List`;
- `Task List` не должен хранить title-level goals как задачи;
- в `Task List` попадает только реальная task/decomposition-сущность.

### Goal decomposition zone
Quest Architect / Goblin tools может писать только сюда:
- action steps: `G/L/Q/V/AA/AF/AK/AP/AU/AZ`
- строки: `17,19,21,23,25,27,29,31,33,35`
- deadline: строка `37` в колонках `H/M/R/W/AB/AG/AL/AQ/AV/BA`
- reward: строка `38` в тех же колонках

Ограничение:
- максимум 10 подзадач на одну goal card.

### Snapshot zone
Фактические snapshot keys в CRM:
- `IS_DATA_STALE`
- `CYCLE_MODIFIER`
- `CYCLE_DAY`
- `HP_MAX`
- `HP_CURRENT`
- `BALANCE_THB`
- `TREASURE_HORIZON_DAYS`
- `XP_TOTAL`
- `XP_GOLD`
- `SPRINT_DAYS`
- `PLAYER_STAGE`
- `ACTIVE_ATTEMPT_ID`
- `XP_AT_LEVEL_START`
- `LEVEL_START_TS`
- `LEVEL_NAME`
- `RISK_MODE`
- `TOTAL_PROGRESS`
- `LEVEL_READY`
- `BELIEFS_FROM_LEVEL_START`
- `MONEY_GROWTH_LEVEL`
- `QUESTS_DONE_LEVEL`
- `DEBUFF_EXPIRES`
- `DEBUFF_NAME`
- `BOSS_NAME`

UI-правило для pre-game:
- до `ONBOARDING_COMPLETED` ячейка `D35` используется как мягкий маркер состояния мира и показывает `Подготовка мира`
- после старта игры `D35` возвращается к нормальному отображению `LEVEL_NAME`
- этот маркер не заменяет `GAME_LOG` как источник правды для старта игры

---

## EVENT_TYPE Registry

GAME_LOG использует единый EVENT_TYPE dictionary для всех событий. Реестр теперь задает не только active-set workflow, но и каноническую механику attempt/activity.

### Rules Layer
- `RULE_CREATED` — правило или базовая игровая сущность создана
- `RULE_UPDATED` — правило изменено
- `COMMIT_LOG` — игровой коммит принят и зафиксирован
- `ONBOARDING_COMPLETED` — первичная настройка игры завершена
- `ONBOARDING_CHECKPOINT` — промежуточный чекпоинт онбординга сохранён в `GAME_LOG`

### Engine Layer
- `ATTEMPT_STARTED` — попытка начата или возобновлена
- `ATTEMPT_FAILED` — попытка провалена после периода тишины
- `STUCK_TIME_DETECTED` — обнаружено 25+ часов тишины
- `STUCK_TRIGGERED` — подтверждён застой, требуется ответная логика
- `CONSISTENCY_CHECKED` — выполнена ежедневная служебная проверка
- `LEVEL_READY_REACHED` — достигнут порог готовности к переходу уровня
- `STAGE_CHANGED` — игровая стадия изменена
- `THERAPIST_REQUIRED` — ENGINE требует терапевтический сценарий
- `SPRINT_STARTED` — начат новый спринт
- `SPRINT_ENDED` — спринт завершён по длине или циклу
- `BOSS_ACTIVATED` — на спринт активирован текущий босс
- `TREASURE_HORIZON_REACHED` — достигнут горизонт сокровища без успешного перехода

### Finance Layer
- `GOLD_GAINED` — зафиксирован приход денег
- `GOLD_LOST` — зафиксирован расход денег

### Quest Architect Layer
- `HABIT_DONE` — привычка выполнена
- `HABIT_SKIPPED` — привычка пропущена
- `TASK_CREATED` — новая задача записана в `Task List`
- `TASK_UPDATED` — существующая задача обновлена
- `TASK_STATUS_CHANGED` — статус задачи изменён
- `TASK_COMPLETED` — завершение задачи зафиксировано как отдельное событие
- `DAY_PLAN_UPDATED` — план дня обновлён
- `GOAL_DECOMPOSED` — goal card разложена на шаги

### Therapist Layer
- `BELIEF_RECORDED` — новое убеждение или инсайт записаны
- `HP_GAIN` — восстановление HP; два источника: (1) Therapist FINAL — `hp_cost` прибавляется назад как дельта; (2) Quest VITALITY-привычки — `reward_value × count` при `track=VITALITY` или `resource_type=HP`; HP не превышает 100 (cap по формуле в `PROFILE_GAME`)
- `HP_SPENT` — HP списана как стоимость терапевтической работы
- `BOSS_DEFEATED` — активный босс спринта побеждён; записывается Therapist в режиме FINAL при наличии `active_boss_name` в контексте сессии
- `THERAPY_DISCUSSION` — промежуточная терапевтическая беседа
- `THERAPY_SESSION_COMPLETED` — финальная терапевтическая сессия завершена
- `LEVEL_UP_INTERVIEW_STARTED` — начат сценарий проверки level-up
- `LEVEL_UP_APPROVED` — переход уровня одобрен
- `LEVEL_UP_HELD` — переход уровня удержан
- `FIVE_FAILURES_ANALYZED` — завершён разбор серии провалов
- `TARGET_REVIEWED` — цель пересмотрена; решение хранится в `STATUS` (`KEEP`, `ADJUST`, `REPLACE`)
- `THERAPY_REFRAME_STARTED` — начат рефрейм базового правила
- `THERAPY_REFRAME_COMPLETED` — рефрейм завершён

### Reminder Audit
- `TASK_REMINDED` — hourly `ENGINE` обнаружил задачу с валидным временем старта в окне `[now+30min, now+90min]`, отправил Telegram-уведомление и записал audit-событие в `GAME_LOG`; **не считается activity**

### Service Layer
- `SERVICE_EFFECT_RECORDED` — значимый сервисный эффект зафиксирован без запуска игровой механики

Рекомендуемый базовый набор полей в `GAME_LOG`:
- `TS`
- `EVENT_TYPE`
- `SOURCE_WORKFLOW`
- `ENTITY_GROUP`
- `ENTITY_ID`
- `STATUS`
- `VALUE`
- `RESOURCE_TYPE`
- `DELTA`
- `DATE_REF`
- `STAGE_AT_TIME`
- `LEVEL_AT_TIME`
- `ATTEMPT_ID`
- `CONTEXT_JSON`
- `COMMENT`

### Attempt activity rule
Игровая активность для `attempt` считается только по `EVENT_TYPE`, а не по любому факту записи в таблицы.

События, которые считаются activity:
- `ONBOARDING_COMPLETED`
- `ATTEMPT_STARTED`
- `RULE_CREATED`
- `RULE_UPDATED`
- `GOLD_GAINED`
- `GOLD_LOST`
- `HABIT_DONE`
- `HABIT_SKIPPED`
- `TASK_CREATED`
- `TASK_UPDATED`
- `TASK_STATUS_CHANGED`
- `TASK_COMPLETED`
- `DAY_PLAN_UPDATED`
- `GOAL_DECOMPOSED`
- `BELIEF_RECORDED`
- `HP_GAIN`
- `BOSS_DEFEATED`
- `THERAPY_DISCUSSION`
- `THERAPY_SESSION_COMPLETED`
- `TARGET_REVIEWED`
- `THERAPY_REFRAME_COMPLETED`
- `LEVEL_UP_INTERVIEW_STARTED`
- `LEVEL_UP_APPROVED`
- `LEVEL_UP_HELD`

События, которые не считаются activity:
- `ONBOARDING_CHECKPOINT`
- `CONSISTENCY_CHECKED`
- `STUCK_TIME_DETECTED`
- `STUCK_TRIGGERED`
- `ATTEMPT_FAILED`
- `LEVEL_READY_REACHED`
- `STAGE_CHANGED`
- `THERAPIST_REQUIRED`
- `HP_SPENT`
- `FIVE_FAILURES_ANALYZED`
- `THERAPY_REFRAME_STARTED`
- `TASK_REMINDED`
- `SERVICE_EFFECT_RECORDED`

### Дополнительный канон ENGINE
- `onboarding_completed` должен определяться по `GAME_LOG`, а не хардкодом;
- каноническая проверка: `logRows.some(r => r.EVENT_TYPE === 'ONBOARDING_COMPLETED')`;
- при `ATTEMPT_FAILED` ENGINE должен дополнительно писать `GOLD_LOST` с `RESOURCE_TYPE = XP_GOLD` и отрицательным `DELTA`, если штраф не был списан ранее для этого fail;
- штраф за fail берётся из базового правила `FAIL_PENALTY` / `fail_penalty_xp_gold` в `GAME_RULES`;
- если штраф задан как `ALL`, списывается текущий `XP_GOLD`, но ресурс не должен уходить в минус;
- при расчётах ENGINE локально фильтрует `GAME_LOG`: всегда сохраняет structural events (`ONBOARDING_COMPLETED`, `ATTEMPT_STARTED`, `ATTEMPT_FAILED`, `SPRINT_STARTED`, `SPRINT_ENDED`, `LEVEL_UP_APPROVED`, `THERAPIST_REQUIRED`) и все события за последние 90 дней;
- `GAME_LOG` фильтруется в коде после чтения, без серверной пагинации Google Sheets на первом этапе.

## Каноника Sprint / Boss / Treasure

### Sprint
- `Sprint` — это ограниченный по длине тактический цикл внутри активной игры;
- `attempt` отвечает за живой контакт игрока с игрой, а `sprint` — за текущий тактический отрезок внутри этого контакта;
- `SPRINT_STARTED` открывает новый спринт;
- `SPRINT_ENDED` закрывает текущий спринт по длине или циклу;
- в каждый момент времени у игрока должен быть не более одного открытого спринта;
- завершение спринта не означает автоматическую победу над боссом.

### Boss
- `Boss` — это формализованное препятствие текущего спринта: паттерн, риск, саботаж или ограничитель, явно заданный в `GAME_RULES`;
- на один спринт активируется один текущий босс;
- выбор босса должен быть детерминированным и воспроизводимым;
- технический `status` workflow не должен менять выбранного босса;
- босс считается закрытым только после события `BOSS_DEFEATED`;
- `BOSS_DEFEATED` записывает Therapist в режиме FINAL при наличии `active_boss_name` в контексте; ENGINE не пишет это событие;
- победа над боссом не равна просто окончанию спринта и должна опираться на отдельное boss-condition правило в `GAME_RULES`;
- для production в `GAME_RULES` должно быть больше `3` активных boss-правил; рекомендуемый минимум — `4`.

### Treasure
- `Treasure` — это целевая ценность текущего уровня;
- `Treasure horizon` — это дата или срок, к которому текущий treasure-cycle должен быть завершён, пересобран или честно переопределён;
- `TREASURE_HORIZON_REACHED` означает не конец игры, а стратегический сбой текущей рамки цели;
- после `TREASURE_HORIZON_REACHED` допустимы штраф, `THERAPIST_REQUIRED` и возврат к пересмотру цели и планирования.

### Planned Second Wave
Согласовано как канон, даже если ещё не полностью внедрено в workflow-json:
- добавить обязательный onboarding-step `FAIL_PENALTY` с записью базового правила штрафа в `GAME_RULES`;
- в `main` читать последние строки `BELIEFS` и передавать в summary-контекст как `beliefs_summary`;
- в ENGINE ввести lifecycle спринта:
  - читать последний `SPRINT_STARTED`;
  - сравнивать длину спринта с правилом из `GAME_RULES`;
  - при завершении писать `SPRINT_ENDED` и новый `SPRINT_STARTED`;
  - при новом спринте активировать одного босса через `BOSS_ACTIVATED`;
- в ENGINE ввести контроль treasure horizon:
  - читать `GOAL_HORIZON` из `GAME_RULES`;
  - при достижении горизонта писать `TREASURE_HORIZON_REACHED`;
  - вместе с ним писать штрафной `GOLD_LOST` и `THERAPIST_REQUIRED`.

Правило:
- новый `attempt` может стартовать только после `ONBOARDING_COMPLETED`;
- `ONBOARDING_COMPLETED` может сразу же создать `ATTEMPT_STARTED`;
- после fail новый `attempt` стартует любым следующим non-service activity event;
- service effects можно логировать, но они не продлевают `attempt` и не запускают новый.

Источник: `EVENT_TYPE_Registry.json`

---

## Статусы файлов

### Canonical docs
- `Sekretutka_PRD_v1.md`
- `README.md`
- `Секретутка_Правила_Игры.docx`
- живая Google Sheets CRM

### Active workflows
- `1. Sekretutka_main.json`
- `1.0 Sekretutka_Game Rules.json`
- `1.1 Sekretutka_ENGINE_updated.json`
- `1.2. Sekretutka_Get Schedule Context.json`
- `1.3 Quest_Architect_Daily_Planner.json`
- `1.4. Sekretutka_finance.json`
- `1.5 Sekretutka_Therapist.json`
- `Sekretutka_Siri_Gateway.json`

### Legacy / deferred
- `1.6. Sekretutka_Notifier.json` — временный legacy workflow; reminder-логика должна быть перенесена в `ENGINE`, после миграции workflow архивируется
- `1.6 Sekretutka_track_progress.json` — удалён; reminder-ветка отдельно больше не развивается
- `1.7. Sekretutka_LK_Render.json` — ✅ обновлён: читает `PROFILE_GAME` и `Task List` из основной таблицы, бот `@Secretutka_AV_bot`
- `Sekretutka_Architecture_v3.html` — secondary reference until synced
- `Секретутка_Правила_Игры.backup_before_update.docx` — архивная копия, не канон

### Промпты
- `Sekretutka_Main_Prompt_REFINED.md`
- `SekretutkaGameRulesManager_Prompt_REFINED.md`
- `Sekretutka_Quest_Architect_Prompt_REFINED.md`
- `Sekretutka_Therapist_Prompt_REFINED.md`

### Документы
- `Sekretutka_PRD_v1.md`
- `Секретутка_Правила_Игры.docx`
- `Sekretutka_Architecture_v3.html`

---

## Что в workflow устарело
Ниже — не канон, а долг на рефакторинг:
- ссылки на `STAGE_RULES`;
- ссылки на `RESOURCE_LOG & BOSS_LOG`;
- старая модель `PROFILE` вместо `PROFILE_GAME`;
- ручное или полу-ручное понимание stage;
- старое различение `GAME_STAGE` и `PLAYER_STAGE`;
- старые допущения про onboarding flags в `PROFILE`;
- старая привычечная логика через отдельный workflow и теневые логи вместо `Quest Architect + GAME_LOG` как единого event-слоя.

Важно:
- общая orchestration-логика workflow может быть полезной;
- привязка к данным и полям сейчас считается ненадежной, пока не обновлена под новый канон.

## Канонические маршруты ядра
- `log_habit -> Quest Architect / HABIT_LOG`
- `add_task -> Quest Architect / TASK_CAPTURE`
- `plan_day -> Quest Architect / PLANNER`
- `decompose_goal -> Quest Architect / GOAL_DECOMPOSER`
- `log_finance -> finance`
- `therapist -> therapist`
- `rules_onboarding / rules_edit -> game_rules`
- `read_status -> temporary direct response in main`
- `engine_notice -> engine-aware internal route in main`

Примечания:
- отдельный `track_progress` удалён; `track_habit` в Siri теперь вызывает Quest Architect;
- `Siri Gateway` и `LK_Render` обновлены под канонические имена листов и основную таблицу;
- привычки, задачи, планирование и decomposition живут в одном рабочем слое `Quest Architect`.
- `Get Schedule Context` — единственное допустимое read-only исключение для контекста и памяти о существующих задачах;
- `main` может логировать значимые сервисные эффекты через `SERVICE_EFFECT_RECORDED`;
- `Get Schedule Context` по умолчанию ничего не пишет и не обязан оставлять service-log.

## Workflow Write Contracts

### `1. Sekretutka_main.json`
- роль: intent router и orchestration layer
- читает: `PROFILE_GAME`, `GAME_RULES`, `GAME_LOG`, schedule context
- пишет: только при значимом сервисном эффекте
- канонический `EVENT_TYPE`: `SERVICE_EFFECT_RECORDED` опционально
- не должен: создавать игровую активность только самим фактом маршрутизации
- текущая реализация service-log: `main` пишет `SERVICE_EFFECT_RECORDED` только для осмысленных read-only запросов про статус, расписание, список задач и план дня, если сервисный контекст реально был собран и использован в ответе;
- `SERVICE_EFFECT_RECORDED` в `main` фиксирует сервисный источник (`GET_SCHEDULE_CONTEXT`), тип эффекта `READ`, дату, stage, `ATTEMPT_ID` и краткий `context_json` c агрегатами вроде количества задач, overdue и размера `beliefs_summary`.

### `1.0 Sekretutka_Game Rules.json`
- роль: onboarding + rules edit
- читает: `GAME_LOG` для checkpoint и resume
- пишет: `GAME_RULES`, `GAME_LOG`
- обязательные события: `ONBOARDING_CHECKPOINT`, `ONBOARDING_COMPLETED`, `RULE_CREATED`, `RULE_UPDATED`
- допускается: сразу после `ONBOARDING_COMPLETED` создать `ATTEMPT_STARTED`
- planned дополнение: обязательный шаг `FAIL_PENALTY` в FSM между `STUCK_RULE` и `LOW_HP_RULES`

### `1.1 Sekretutka_ENGINE_updated.json`
- роль: системный пересчёт состояния игры и единый cron-оркестратор
- читает: `PROFILE_GAME`, `GAME_RULES`, `GAME_LOG`, `WALLET_Monthly Budget`, `Task List`
- пишет: `GAME_LOG`
- обязательные события по ситуации: `CONSISTENCY_CHECKED`, `STUCK_TIME_DETECTED`, `STUCK_TRIGGERED`, `ATTEMPT_FAILED`, `ATTEMPT_STARTED`, `STAGE_CHANGED`, `LEVEL_READY_REACHED`, `THERAPIST_REQUIRED`, `TASK_REMINDED`
- критическое правило: activity считается только по `EVENT_TYPE`
- реализовано: `GOLD_LOST` при fail, 90-day local filtering, `SPRINT_STARTED`, `SPRINT_ENDED`, `BOSS_ACTIVATED`, `TREASURE_HORIZON_REACHED`
- целевой cron-канон: один workflow с разными `run_mode`, а не набор отдельных cron-workflow
- канонические `run_mode`: `HOURLY`, `MORNING`, `EVENING`, `MIDNIGHT`
- `HOURLY` делает только тихий audit/reminder scan и не дублирует утренние или вечерние сценарии
- reminder-логика принадлежит `ENGINE`, а не отдельному `Notifier`
- `TASK_REMINDED` живёт только в `GAME_LOG`; `ENGINE` не должен менять предметный статус задачи на `🔔 Reminded`
- reminder использует стабильный `ENTITY_ID = row_number` из `Task List`, а не матчинг по title
- источник времени для reminder нормализуется так: сначала точное время в `Task List`, затем slot из daily sheet; если времени нет, reminder не отправляется
- чтение данных должно идти по bounded/incremental strategy: hourly-run читает ограниченные диапазоны и хвосты, deep rebuild допускается только специальным ручным запуском
- выбор босса в `ENGINE` должен быть детерминированным, через hash-based `chooseBossRule(seed)`, а не через `Math.random`;
- seed босса не должен зависеть от технического `status` (`STARTED` / `RESTARTED`): один и тот же спринт должен получать одного и того же босса;
- текущая безопасная привязка seed: стабильная идентичность спринта, минимум `attemptId + sprint_start_date`; если отдельного `sprint_id` пока нет, нельзя использовать `status` как часть seed.
- чтобы ротация боссов не была слишком предсказуемой, в `GAME_RULES` должно быть больше `3` активных boss-правил; минимальный целевой набор для production — `4+` босса.

### `1.2. Sekretutka_Get Schedule Context.json`
- роль: service/read-only context provider
- читает: `PROFILE_GAME`, `Task List`, `WALLET_Monthly Budget`, calendars
- пишет: ничего
- не должен: влиять на attempt/stage/level

### `1.3 Quest_Architect_Daily_Planner.json`
- роль: tasks, planner, goal decomposition, habit log, evening review
- читает: `PROFILE_GAME`, `Task List`, daily planner sheets
- пишет: `Task List`, `PROFILE_GAME`, daily planner sheets, `GAME_LOG`
- обязательные события: `HABIT_DONE`, `HABIT_SKIPPED`, `TASK_CREATED`, `TASK_UPDATED`, `TASK_STATUS_CHANGED`, `TASK_COMPLETED`, `DAY_PLAN_UPDATED`, `GOAL_DECOMPOSED`
- implementation status: `TASK_CREATED`, `TASK_UPDATED`, `TASK_STATUS_CHANGED`, `TASK_COMPLETED`, `DAY_PLAN_UPDATED`, `GOAL_DECOMPOSED` уже внедрены
- `Task List` использует `appendOrUpdate` по `TASK_ID`, а не по title и не по `row_number`

### `1.4. Sekretutka_finance.json`
- роль: запись финансов
- читает: `WALLET_Monthly Budget`
- пишет: `WALLET_Monthly Budget`, `GAME_LOG`
- обязательные события: `GOLD_GAINED`, `GOLD_LOST`
- важное правило: каждая валидная финансовая операция должна сохраняться в `WALLET_Monthly Budget`, а независимые транзакции не должны теряться, сливаться или перезаписываться
- `WALLET_Monthly Budget` использует `appendOrUpdate` по `ENTRY_ID`, а не по `TYPE`

### `1.5 Sekretutka_Therapist.json`
- роль: discussion, insight capture, level-up interview, reframe
- читает: `PROFILE_GAME`, `BELIEFS`, `GAME_RULES`
- пишет: `BELIEFS`, `GAME_LOG`, `GAME_RULES` при reframe
- обязательные события: `THERAPY_DISCUSSION`, `BELIEF_RECORDED`, `HP_SPENT`, `HP_GAIN`, `BOSS_DEFEATED`, `THERAPY_SESSION_COMPLETED`, `LEVEL_UP_INTERVIEW_STARTED`, `LEVEL_UP_APPROVED`, `LEVEL_UP_HELD`, `FIVE_FAILURES_ANALYZED`, `TARGET_REVIEWED`, `THERAPY_REFRAME_STARTED`, `THERAPY_REFRAME_COMPLETED`, `RULE_UPDATED`

### Implementation Status
- docs and registry: обновлены под канон second-wave решений;
- `ENGINE`: activity-by-event уже внедрён, second-wave logic пока зафиксирована как planned;
- `Quest Architect`: часть нового event-layer уже внедрена;
- `main`: `BELIEFS` summary ещё не внедрён;
- `Game Rules`: `FAIL_PENALTY` как отдельный onboarding-step ещё не внедрён.

### `Sekretutka_Siri_Gateway.json`
- active entrypoint; прямую запись делает через sub-workflows: Quest Architect и finance
- читает: `PROFILE_GAME`, `GAME_RULES` из основной таблицы (`1Dd0BJfB2YwEC5rY90TXvSWPtKQDVav32x0McNKOr-U4`)
- инструменты агента: `add_task` → Quest Architect, `track_habit` → Quest Architect, `add_finance` → finance
- поля из `PROFILE_GAME`: `HP_CURRENT`, `XP_GOLD`, `CYCLE_DAY`
- голосовой режим: ответ максимум 2 предложения, без markdown

### `1.6. Sekretutka_Notifier.json`
- legacy reminder workflow до переноса в `ENGINE`
- не является целевым каноном
- после миграции должен быть архивирован, а не развиваться как отдельный слой

### `1.7. Sekretutka_LK_Render.json`
- исключение из write-rule (render-only)
- читает: `PROFILE_GAME` и `Task List` из основной таблицы (headerRow=6, firstDataRow=7)
- фильтр задач: `DUE_DATE = today`, статус `✅ Completed` исключается в JS
- поля из `PROFILE_GAME`: `HP_CURRENT`, `PLAYER_STAGE`, `TOTAL_PROGRESS`, `LEVEL_READY`, `BOSS_NAME`, `DEBUFF_NAME`, `DEBUFF_EXPIRES`
- поля из `Task List`: `TASK`, `PRIORITY`, `XP| HP`
- отправка через бот `@Secretutka_AV_bot`; триггер: `/lk` OR `/status`

---

## Онбординг
Онбординг должен:
1. начинаться с `/start` или любого первого сообщения без правил, и сразу показывать промо-описание игры в стиле Секретутки;
2. давать явный вход в настройку:
   - текстовыми подтверждениями старта;
   - кнопкой `Переход к настройке` в Telegram reply keyboard;
3. после подтверждения старта переходить не сразу в сбор данных, а в короткий `ONBOARDING_INTRO`, который объясняет, что будет дальше;
4. сохранять checkpoint онбординга по `session_id` в `GAME_LOG` через `ONBOARDING_CHECKPOINT`, чтобы следующий ответ пользователя продолжал текущий шаг;
5. проходить онбординг в микрошаговом формате `1 вопрос = 1 ответ`;
6. собрать базовые игровые правила и сущности;
7. записать их в `GAME_RULES`;
8. записать стартовые события в `GAME_LOG`;
9. создать первый `attempt_id = 1` со статусом `ACTIVE`;
10. завершиться предложением разложить цель на подзадачи через Quest Architect / Goblin tools.

### FSM 39
Канонический порядок onboarding FSM теперь такой:
1. `PROMO`
2. `ONBOARDING_INTRO`
3. `HP_CALIBRATION`
4. `BIO_CYCLE_DAY`
5. `BIO_BODY_STATE`
6. `BIO_ACUTE_LIMITS`
7. `REALITY_DAY`
8. `REALITY_FIXED_COMMITMENTS`
9. `REALITY_ENERGY_ESTIMATE`
10. `SPRINT_LENGTH`
11. `SPRINT_RHYTHM`
12. `TRACKS_INTRO_AND_TASK_CRAFT`
13. `TASK_FLAGS_CRAFT`
14. `TRACK_TASK_SOCIAL`
15. `TASK_FLAGS_SOCIAL`
16. `TRACK_TASK_VITALITY`
17. `TASK_FLAGS_VITALITY`
18. `HABIT_1`
19. `HABIT_1_RULES`
20. `HABIT_2`
21. `HABIT_2_RULES`
22. `HABIT_3`
23. `HABIT_3_RULES`
24. `KPI_1`
25. `KPI_2`
26. `KPI_3`
27. `GOAL_NAME`
28. `GOAL_TREASURE`
29. `GOAL_HORIZON`
30. `BOSSES_INTRO`
31. `BOSS_1`
32. `BOSS_2`
33. `BOSS_TRIGGERS`
34. `STUCK_RULE`
35. `FAIL_PENALTY`
36. `LOW_HP_RULES`
37. `SHOP`
38. `LEVEL_NAMES`
39. `MONEY_NET_THRESHOLD`
40. `CONTRACT_CONFIRM`

Уточнение по boss pool:
- текущие шаги `BOSS_1` и `BOSS_2` недостаточны как финальный production-минимум;
- до рабочей версии в `GAME_RULES` должно быть заведено больше `3` активных боссов;
- рекомендуемый минимум: `4` отдельных boss-правила, чтобы hash-based выбор не был слишком предсказуемым.

Правила порядка:
- блок про треки и задачи идет раньше привычек;
- вопрос про `BASELINE_HABITS_INTRO` убран;
- `Craft` и `Social` задачи используют `XP_Gold`, `Vitality` задачи используют `HP`;
- KPI идут после задач и привычек, а цель уровня идет сразу после KPI.

Правила формулировок:
- `BIO_CYCLE_DAY`: если день цикла неизвестен, игрок может написать дату начала недавней менструации;
- сущности с ценой собираются сразу в человекопонятном формате:
  - `Привычка — награда`
  - `Задача — награда/цена`
  - `Награда — стоимость`
  - `Босс — стоимость`

Онбординг не должен:
- вручную переписывать формульный snapshot `PROFILE_GAME`;
- жить по старой FSM, если она противоречит канону;
- подменять `Task List` или `GAME_LOG`.

Дополнительный канон:
- fail-attempt сжигает `XP_Gold`, а не `XP_TOTAL`;
- `XP_TOTAL = cumulative(0.5 × XP_Gold gains + 0.5 × HP-positive gains)`;
- `STUCK` — это отдельное логируемое состояние контакта с боссом, а не fail;
- правило `2 из 3 (сон / еда / вода)` влияет на consistency attempt;
- goal сама по себе не начисляет ресурс, награда за нее задается через `SHOP` / goal-card reward;
- rules делятся на базовые и тактические.
- подтверждение старта должно принимать не только `ок`, но и естественные варианты вроде `давай`, `хочу`, `готова`, `погнали`, `поехали`, `Переход к настройке` и близкие формулировки.

---

## Migration Spec: Reminder -> ENGINE

Целевое состояние:
- reminder-логика больше не живёт отдельным workflow;
- `ENGINE` становится единым cron-оркестратором;
- `TASK_REMINDED` остаётся service/audit event в `GAME_LOG` и не считается activity;
- статус задачи остаётся предметным (`Pending`, `In Progress`, `Under Review` и т.д.), напоминание не меняет `Task List.STATUS`.

Что переносится в `ENGINE`:
- hourly reminder window `[now+30min, now+90min]`;
- фильтр активных task-statuses без отдельного reminder-status;
- отправка Telegram reminder;
- запись канонического `TASK_REMINDED` в `GAME_LOG`.

Что удаляется из старого `Notifier`:
- отдельный hourly trigger;
- любая логика смены `Task List.STATUS`;
- матчинг по `TASK` как по ключу;
- неканоническая короткая запись в `GAME_LOG`.

Минимальный техдолг перед переносом:
- reminder использует `row_number` как стабильный `ENTITY_ID`;
- reminder не отправляется без валидного времени старта;
- `ENGINE` переводится на bounded/incremental reads;
- `run_mode` в `ENGINE` разводятся так, чтобы `HOURLY` не конфликтовал с `MORNING` и `EVENING`.

Минимальный bounded/incremental канон:
- `PROFILE_GAME`: фиксированный snapshot-range;
- `Task List`: только рабочие колонки и разумный диапазон строк;
- `GAME_LOG`: хвост последних строк или bounded-range;
- `BELIEFS`: последние строки, а не весь лист;
- `WALLET_Monthly Budget`: bounded-range или последние месяцы;
- отдельный deep rebuild допускается только специальным ручным запуском, а не каждым hourly cron.

### Manual QA Checklist
Этот чек-лист нужен для быстрой ручной проверки живого контура `main -> onboarding -> activation -> engine`.

1. `Старт до игры`
- Сообщение: `старт`
- Ожидание:
  - `main` уводит в `rules_onboarding`
  - бот показывает стартовый onboarding promo
  - игра еще не считается начатой
  - `engine` в этом состоянии возвращает `pre_game_mode = true`

2. `Продолжение онбординга`
- Действие:
  - пройти первые 2-3 шага онбординга
  - прервать сценарий
  - затем написать `продолжим`
- Ожидание:
  - workflow подхватывает последний `ONBOARDING_CHECKPOINT`
  - бот возвращается не в начало, а на последний шаг

3. `Переход через Money Net`
- Действие:
  - дойти до шага `38`
  - ввести 4 названия уровней
- Ожидание:
  - следующий шаг — вопрос про `Money Net X`
  - после ввода числа, например `120000`, показывается контракт
  - в контракте есть строка про `Money Net ... THB в месяц 3 месяца подряд`

4. `Подтверждение контракта`
- Сообщение: `да, подтверждаю`
- Ожидание:
  - сначала пишутся строки в `GAME_RULES`
  - затем пишутся события в `GAME_LOG`
  - в `GAME_LOG` появляются `ATTEMPT_STARTED` и `ONBOARDING_COMPLETED`
  - только после этого пользователь получает сообщение об активации

5. `Статус после активации`
- Сообщение: `покажи статус`
- Ожидание:
  - `main` больше не отвечает как для pre-game режима
  - статус показывает живую игру: HP, stage, attempt, active tasks, overdue

6. `Fail по тишине`
- Подготовка:
  - сделать так, чтобы с последней meaningful activity прошло `>= 5` дней
- Действие:
  - запустить `engine`
- Ожидание:
  - создается `ATTEMPT_FAILED`
  - попытка становится неактивной
  - в тексте есть честное описание провала и штрафа

7. `Возврат после fail`
- Подготовка:
  - после fail записать новую meaningful activity
- Действие:
  - снова запустить `engine`
- Ожидание:
  - создается новый `ATTEMPT_STARTED`
  - `xp_total_at_start` фиксируется от текущего `XP_TOTAL`
  - stage снова начинается от нового attempt

8. `Therapist после 5 fail`
- Подготовка:
  - в `GAME_LOG` есть 5 событий `ATTEMPT_FAILED`
- Действие:
  - запустить `engine`
- Ожидание:
  - создается `THERAPIST_REQUIRED`
  - `recommended_next_workflow = THERAPIST`
  - planner/evening review не должны форситься поверх therapist-маршрута

9. `Level-ready и финальный ап`
- Подготовка:
  - `TOTAL_PROGRESS >= 1`
  - `BALANCE_THB >= 1 000 000`
  - отдельно есть 3 месяца `Money Net >= X`
- Ожидание:
  - сначала возникает `LEVEL_READY_REACHED`
  - `final_level_up_eligible` считается отдельно
  - threshold для final check берется из `final_level_up` правила текущего `level_scope`

Что считать провалом теста:
- bot говорит, что игра началась, без `ONBOARDING_COMPLETED`
- bot показывает успешную активацию, но в таблицах нет соответствующих записей
- после шага `38` онбординг не спрашивает `Money Net`
- контракт не показывает `Money Net X`
- `main` и `engine` по-разному трактуют pre-game / active game
- `engine` использует не тот `final_level_up` threshold для текущего уровня

### Bio-layer и recovery
У игры есть два параллельных слоя:
- `Bio-layer`: `HP`, фазы цикла и recovery-режимы;
- `Game-layer`: спринты, KPI, задачи и цель уровня.

`Bio-layer` всегда накладывается поверх `Game-layer`.
Это значит:
- нельзя “взять себя в руки” при низком `HP`;
- работа возможна только при энергии, в том числе эта игра;
- `HP = 100` — это максимально доступное здоровье в системе, а не “норма плюс запас”.

Канонические режимы:
- `NORMAL`: `HP >= 100`
  - стандартные штрафы и награды;
  - спринты и сокровища доступны;
  - тон зависит от фазы цикла, а не от FSM.
- `RECOVERY_ACTIVE`: `HP < 60`
  - система предупреждает, но не запрещает;
  - спринты, сокровища и боссы доступны;
  - цена риска: более сильные откаты `HP` и более долгий путь восстановления;
  - выход: вручную или через стабилизацию `HP`.
- `RECOVERY_PASSIVE`: `HP < 30`
  - система прямо называет состояние истощением;
  - боссы и спринты не блокируются;
  - штрафы по `XP` остаются стандартными;
  - тело платит больше: сильнее откаты `HP`, медленнее восстановление;
  - выход: при `HP >=` порога возврата, например `35`.

---

## Google Sheets
Основная таблица:
- `1Dd0BJfB2YwEC5rY90TXvSWPtKQDVav32x0McNKOr-U4`

Критичные листы:
- `Task List`
- `PROFILE_GAME`
- `GAME_RULES`
- `GAME_LOG`
- `BELIEFS`
- `WALLET_Monthly Budget`
- daily sheets
- `Weekly planner`

Timezone игры:
- канонически `Asia/Bangkok`

Настройка timezone самой таблицы делается отдельно вручную.

---

## Recent Fixes & Implementation Updates (2026-04-15)

### 1. Format Final Response Logic Fix

**Problem:** Execute Workflow sub-calls return ALL terminal nodes' outputs, not just the intended reply. When Append nodes (Sheets writes) executed in parallel with Return Response nodes, the Format Final Response would blindly take the first item `$json` — which was often the Sheets append result, not the user-facing reply.

**Solution:** Updated Format Final Response to intelligently search for the correct response item instead of assuming it's first:
```javascript
const targetItem = $allInputItems.find(i => i.json && (i.json.reply_to_user || i.json.output));
```

This ensures the response always contains the human-readable reply text, not Sheets metadata.

**Implementation:** 
- `1. Sekretutka_main` node `Format Final Response`
- Fallback: if no `reply_to_user` found, uses first item with `output` field

---

### 2. Queue-based Multi-Task Handling (States 11-16)

**Problem:** Original design asked for task name in state 11, then details in state 12. If a user had multiple tasks per track, the system couldn't handle them without re-asking the same details for each.

**Solution:** Implemented a queue-based collection + loop:
- **State 11** (per track): Collect ALL tasks as line-separated list → parse each with `parseDashFormat()` → save to `task_queue`
- **State 12** (loop): Ask details for each task one-by-one, displaying task name dynamically → on last task, merge into `task_examples` and advance
- Repeat for tracks Social (states 13-14) and Vitality (states 15-16)

**Key prompts:**
- **State 11 intro:** Includes visual explanation of difference between tasks/habits/KPI with bold emphasis
- **State 12 dynamic:** "Для задачи '**{task_name}**': она разовая или повторяющаяся?"
- **State 12 loop logic:** If not last task → show state 12 again with next task; if last task → merge and advance

**Implementation:**
- `1.0 Sekretutka_Game Rules` → Onboarding FSM Router states 11-16
- Draft structure: `craft_task_queue`, `social_task_queue`, `vitality_task_queue`
- Each detail state appends to `task_examples` array

---

### 3. Track Recognition Enhancement (States 18, 20, 22)

**Changes:**
1. **Simplified input:** Now asks ONLY for track, not period or target
2. **Multi-format recognition:** Supports emoji, English keywords, and Russian keywords
3. **Expanded keyword sets:**

   **CRAFT** (💪💼💰📊🎯🚀📈🛠💡):
   - English: craft, money, income, work, job, skill, experience, career, business, project, sales, marketing, content, training, creativity
   - Русский: крафт, деньги, доход, работа, дело, навык, опыт, карьера, бизнес, проект, продажи, маркетинг, контент, обучение, творчество

   **VITALITY** (🫀💚🏃🧘🏋🌿):
   - English: vitality, body, health, recovery, sport, sleep, movement, nutrition, meditation, yoga, training, energy, wellbeing
   - Русский: тело, здоровье, восстановление, спорт, сон, движение, питание, медитация, йога, тренировка, энергия, самочувствие

   **SOCIAL** (🤝❤️👥🫂💕🌐👯‍♀️💃):
   - English: social, relationships, family, people, communication, friendship, love, partner, parents, children, colleagues, networking, influence, community
   - Русский: отношения, семья, люди, общение, дружба, любовь, партнер, родители, дети, коллеги, нетворкинг, влияние, сообщество

**Fallback parsing:** Regex pattern matching for emoji + Cyrillic keywords  
**AI hints:** FSM Router stateMeta includes full keyword examples and emoji for each track

**Implementation:**
- `1.0 Sekretutka_Game Rules`:
  - Onboarding FSM Router states 18, 20, 22 with updated stateMeta descriptions
  - Onboarding Draft Merge node with regex fallback for each track
  - AI Extractor instruction: "Map answer to exactly one of 3 values: CRAFT (...), VITALITY (...), SOCIAL (...). Always return the English name in UPPERCASE."

---

### 4. Telegram Markdown Escaping

**Problem:** Identifiers like `XP_Gold` and `RECOVERY_ACTIVE` contain underscores that break Telegram Markdown V1 parser (underscore = italic marker). System sends "can't parse entities: Can't find end of the entity" errors.

**Solution:** Escape underscores in text output with backslash: `XP\_Gold`, `RECOVERY\_ACTIVE`, etc.

**Implementation:**
- Global rule: Escape underscores + asterisks + backticks + square brackets in all user-facing text
- Preserve Markdown formatting: `*bold*`, `_italic_` are valid; `XP_Gold` becomes `XP\_Gold`
- Applied throughout: FSM Router prompts, Draft Merge outputs, engine responses

**Technical detail:**
- `parse_mode: "Markdown"` explicitly set in Telegram send node
- No global regex escape on all text (would break valid formatting) — selective escaping only for identifiers

---

### 5. Onboarding State Machine (State 0 Restart Behavior)

**Behavior:** When user restarts onboarding mid-session:
- Route Decision detects `!onboardingCompleted || !hasRules` → forces onboarding
- Hydrate Onboarding Session finds no checkpoint → initializes clean `state=0`, `draft={}`
- FSM Router awaits user confirmation ("давай", "ок", etc.) before advancing to state 1

**Confirm patterns accepted:** давай, ок, готова, погнали, поехали, хочу, начнём, переход к настройке, and natural variations

---

## Следующий технический шаг
После синхронизации документации workflow-json нужно привести к новому канону:
1. `main` должен обрабатывать `/start` как промо-вход в игру, а не как техничное сообщение про отсутствие правил;
2. `main` должен добавлять кнопку `Переход к настройке` только на входе в rules-flow;
3. `Game Rules` workflow должен хранить checkpoint онбординга в `GAME_LOG`;
4. `ENGINE` нужно держать синхронным с `PROFILE_GAME`, `GAME_RULES`, `GAME_LOG`;
3. обновить `Quest Architect` под decomposition rules и единый event layer;
4. удалить зависимости от старых листов и старых имен полей;
5. зафиксировать canonical `EVENT_TYPE`.
