# СЕКРЕТУТКА — PRODUCT REQUIREMENTS DOCUMENT

## 1. Статус документа
Этот PRD описывает три слоя одновременно:
- целевой канон игры;
- фактическую CRM-схему, которая сейчас является самым свежим источником правды по структуре данных;
- разрыв между целевой архитектурой и текущими workflow.

Каноном для игровой логики считаются:
- [Секретутка_Правила_Игры.docx](/Users/alinavladimirova/Documents/Claude/Projects/Секретутка/Секретутка_Правила_Игры.docx);
- [Sekretutka_Architecture_v3.html](/Users/alinavladimirova/Documents/Claude/Projects/Секретутка/Sekretutka_Architecture_v3.html);
- этот PRD;
- актуальная Google Sheets CRM.

Каноном для схемы колонок считаются живые листы CRM, даже если старые документы описывали их иначе.

---

## 2. Видение продукта
Секретутка — это личный Telegram-ассистент Алины на базе n8n и Google Sheets, который:
- фиксирует задачи, привычки, KPI, деньги, инсайты и игровые изменения как структурированные данные;
- ведет игру как честную систему состояний, в которой ничего не считается записанным без подтвержденного workflow;
- отделяет правила мира, историю событий, витрины и snapshot игрока;
- помогает игроку задать цель игры и поддерживать консистентность через попытки, уровни и ежедневные действия.

Базовый принцип архитектуры:
- `GAME_RULES` = реестр правил мира, option sets, пользовательских настроек и игровых сущностей;
- `GAME_LOG` = единая история игровых событий;
- `PROFILE_GAME` = комбинированный лист целей и snapshot игрока, который в основном собирается формулами;
- `Task List` = единственный источник правды по задачам;
- `Dashboard`, daily planner и weekly planner = витрины и представления.

Технические ID-колонки в живой CRM:
- `Task List!Q = TASK_ID`
- `WALLET_Monthly Budget!L = ENTRY_ID`

### 2.1 Pre-game mode
До завершения онбординга система работает в режиме подготовки мира:
- пользователь может вручную заполнить любые листы CRM;
- такие записи считаются `pre-game data`;
- `pre-game data` можно читать как контекст, но оно не должно само по себе запускать игру;
- `GAME_RULES` до старта игры может оставаться пустым;
- канонический признак старта игры — наличие `ONBOARDING_COMPLETED` в `GAME_LOG`.
- мягкий UI-маркер pre-game живёт в `PROFILE_GAME!D35`: до старта там показывается `Подготовка мира`, после старта — нормальное `LEVEL_NAME`.

---

## 3. Продуктовые принципы
1. **Один источник правды на один тип сущности**
   - задачи живут в `Task List`
   - игровые правила и конфиги живут в `GAME_RULES`
   - история событий живет в `GAME_LOG`
   - текущий snapshot игрока показывается в `PROFILE_GAME`
2. **Игрок задает правила игры, но не все игровые состояния вручную**
   - игрок задает treasure, narrative goals, level names, habits, KPI, bosses, rewards и часть правил мира;
   - система автоматически вычисляет attempts, stage, readiness и level progression.
3. **Система остается честной**
   - если запись не подтверждена workflow, Секретутка не говорит, что она состоялась;
   - тишина и активность считаются только по успешно записанным игровым событиям.
4. **Витрины не дублируют источники правды**
   - `Dashboard`, daily planner, weekly planner и kanban-виды не являются мастер-таблицами.
5. **Все игровое состояние должно быть объяснимым**
   - у каждого вычисляемого поля должен быть понятный источник;
   - stage, attempt и level должны выводиться из логики и событий, а не из скрытых ручных правок.
6. **Pre-game данные допустимы, но не равны игровому состоянию**
   - до `ONBOARDING_COMPLETED` ручные записи не считаются активным attempt;
   - ручное заполнение листов не заменяет activation workflow;
   - без подтвержденного workflow данные не должны менять честную игровую механику.

---

## 4. Каноническая игровая модель

### 4.1 Bio-Layer
Bio-Layer — это слой состояния тела и давления:
- `CYCLE_DAY`
- `CYCLE_MODIFIER`
- `HP_CURRENT`
- симптомы, дебаффы, состояние восстановления

У игры есть два параллельных слоя:
- `Bio-layer`: `HP`, цикл, симптомы и recovery-режимы;
- `Game-layer`: спринты, KPI, задачи и цель уровня.

Bio-Layer всегда накладывается поверх Game-Layer.
Это значит:
- нельзя “взять себя в руки” при низком `HP`;
- работа возможна только при энергии, в том числе эта игра;
- `HP = 100` — это максимально доступное здоровье в системе;
- тон, нагрузка и советы зависят от `Bio-layer`, а не только от FSM.

Bio-Layer не:
- определяет уровень;
- определяет stage напрямую;
- заменяет attempts и progression logic.

#### 4.1.1 Recovery modes
Канонические режимы:

`NORMAL`
- `HP >= 100`
- стандартные штрафы и награды;
- спринты и сокровища доступны;
- тон зависит от фазы цикла, а не от FSM.

`RECOVERY_ACTIVE`
- `HP < 60`
- система предупреждает, но не запрещает;
- спринты, сокровища и боссы доступны;
- цена риска: более сильные откаты `HP` и более долгий путь восстановления;
- выход: вручную или через стабилизацию `HP`.

`RECOVERY_PASSIVE`
- `HP < 30`
- система прямо называет состояние истощением;
- боссы и спринты не блокируются;
- штрафы по `XP` стандартные;
- тело платит больше: сильнее откаты `HP`, дольше восстановление;
- выход: при `HP >=` порога возврата, например `35`.

#### 4.1.2 HP mechanics
HP восстанавливается через два источника:
- **Therapist FINAL**: по завершении финальной терапевтической сессии значение `hp_cost` из ответа агента прибавляется обратно как событие `HP_GAIN` с `DELTA = hp_cost`;
- **VITALITY-привычки**: при выполнении привычки с `track = VITALITY` или `resource_type = HP` значение `reward_value × count` записывается как `HP_GAIN` событие в `GAME_LOG`.

HP тратится:
- через событие `HP_SPENT` при открытии терапевтической сессии (значение `hp_cost` в момент входа в режим).

Ограничение: `HP` не может превышать `100` — ограничение применяется формулой в `PROFILE_GAME`; workflow пишет сырые дельты, а cap соблюдается на уровне snapshot.

### 4.2 Player Status
`Player Status = XP_TOTAL`

Это lifetime-показатель опыта игрока:
- может расти;
- не сгорает при провале attempt;
- не равен stage;
- не равен level.

### 4.3 Attempt
Attempt — это период консистентного присутствия в игре.

Канонические правила:
- если в `GAME_LOG` нет `ONBOARDING_COMPLETED`, attempt еще не существует;
- attempt имеет только два статуса: `ACTIVE`, `FAILED`;
- первый attempt стартует сразу после завершения онбординга;
- новый attempt стартует только после нового возвращения игрока в игру;
- возвращением в игру считается любое успешно записанное игровое событие после провала или паузы.

#### Что считается активностью внутри attempt
Активность считается только по успешно записанным событиям в системе, например:
- создание или обновление задачи;
- изменение статуса задачи;
- лог привычки;
- обновление плана дня;
- decomposition цели;
- лог KPI;
- финансовая запись;
- запись инсайта через Therapist;
- терапевтическая discussion-сессия;
- изменения правил, если они реально сохранились;
- пересмотр цели;
- сценарии level-up interview.

Простое сообщение в чат без успешной записи не сбрасывает счетчик тишины.

Каноническое правило:
- activity считается только по `EVENT_TYPE` в `GAME_LOG`;
- service-log не считается игровой активностью;
- read-only вызовы и orchestration без значимого эффекта не продлевают attempt.

#### Промежуточное состояние: STUCK_TIME
`STUCK_TIME` — это alert-сигнал, который возникает при 25+ часов молчания во время ACTIVE attempt.

Характеристики:
- `STUCK_TIME` не меняет статус attempt (он остается `ACTIVE`);
- `STUCK_TIME` инициирует контакт помощи от Therapist;
- генерирует событие `STUCK_TIME_DETECTED` в `GAME_LOG`;
- дает игроку возможность получить поддержку до того, как attempt будет провален;
- отличается от 5-дневного провала attempt тем, что это раннее предупреждение.

#### Провал attempt
Attempt считается проваленным, если 5 дней подряд нет успешно записанной игровой активности.

При провале attempt:
- `ATTEMPT_STATUS = FAILED`;
- `PLAYER_STAGE` сбрасывается в `EARLY`;
- `XP_GOLD` сгорает;
- `XP_TOTAL` сохраняется;
- в snapshot остается последний `ACTIVE_ATTEMPT_ID`;
- новый attempt стартует только при следующем реальном возвращении игрока в систему.

### 4.4 Stage
Stage — это автоматически вычисляемый статус внутри текущего attempt.

Канонические значения:
- `EARLY`
- `MID`
- `LATE`

Stage считается не по lifetime XP, а по прогрессу внутри текущего attempt:

```text
STAGE_XP = XP_TOTAL - XP_AT_ATTEMPT_START
```

Пороги:
- `0–499` -> `EARLY`
- `500–3499` -> `MID`
- `3500+` -> `LATE`

Правила:
- stage всегда начинается с `EARLY` в начале нового attempt;
- stage не задается игроком вручную;
- stage должен быть объясним через snapshot и стартовые точки текущего attempt.

### 4.4.1 XP model
`XP_Gold` — основной игровой ресурс консистентности и прогресса.

Правила:
- системные провалы и fail-attempt сжигают `XP_Gold`, а не `XP_TOTAL`;
- игрок во время онбординга сам задает штрафы за системные провалы именно в `XP_Gold`;
- bosses и другие отрицательные игровые эффекты выражаются через отрицательные значения.

`XP_TOTAL` — lifetime-метрика опыта.

Каноническая модель:

```text
XP_TOTAL = cumulative(0.5 × XP_Gold gains + 0.5 × HP-positive gains)
```

Важно:
- `XP_TOTAL` не сгорает при fail;
- `STAGE_XP` считается от `XP_TOTAL` внутри текущего attempt;
- `XP_Gold` и `XP_TOTAL` не взаимозаменяемы.

### 4.5 Level
Level состоит из двух связанных сущностей:
- `Level Number` — системный прогрессионный уровень;
- `Level Name` — системное имя уровня, выбранное игроком во время онбординга и сохраненное в `GAME_RULES`.

#### Level Number
Это финансово-прогрессионный уровень:
- растет только вперед;
- не откатывается назад;
- повышается только после подтверждения игроком.

#### Level Name
Это не свободное текущее настроение игрока, а имя системного уровня.
Минимально обязательная конфигурация:
- в онбординге игрок задает имена как минимум для 4 уровней;
- в будущем система может иметь больше 4 уровней;
- уровень отображается как связка `Level Number + Level Name`.

### 4.6 TOTAL_PROGRESS
`TOTAL_PROGRESS` — индикатор готовности к level up, а не отдельный вид уровня.

Формула:

```text
TOTAL_PROGRESS = 0.4×(Money_Growth_From_Level_Start/1 000 000)
               + 0.3×(Done_Tasks_From_Level_Start/90)
               + 0.2×(XP_From_Level_Start/5000)
               + 0.1×(Beliefs_From_Level_Start/3)
```

Важно:
- используется `XP_From_Level_Start`, а не весь `XP_TOTAL`;
- это прогресс внутри текущего уровня, а не lifetime-показатель.

### 4.7 LEVEL_READY
`LEVEL_READY = TRUE`, если одновременно выполняются оба условия:
- `TOTAL_PROGRESS >= 1.0`
- запас `THB >= 1 000 000`

Это не автопереход.
Когда `LEVEL_READY = TRUE`, система:
- сообщает игроку о готовности;
- ждет подтверждения;
- только после подтверждения делает level up.

### 4.7.1 Финальный level up
`LEVEL_READY` — readiness-сигнал, а не финальный ап уровня.

Финальный ап требует отдельной финансовой проверки:
- `Money Net >= X` в месяц;
- минимум `3` месяца подряд;
- учитывается доход плюс экономия;
- разовый всплеск не считается достаточным.

Итого:
- `TOTAL_PROGRESS + BALANCE_THB` отвечают за readiness;
- трехмесячная money-net проверка отвечает за финальное подтверждение перехода.

### 4.8 Что сбрасывается при level up
При подтвержденном переходе на новый уровень пересобирается база уровня:
- `LEVEL_START_TS`
- `XP_AT_LEVEL_START`
- `MONEY_GROWTH_LEVEL`
- `QUESTS_DONE_LEVEL`
- `BELIEFS_FROM_LEVEL_START`
- любые другие счетчики "from level start"

### 4.9 Goals и decomposition
В онбординге игрок обязан задать хотя бы одну цель.

Цель живет в двух режимах:
- narrative/top goal в `PROFILE_GAME`;
- структурированные игровые параметры и правила в `GAME_RULES`.

Онбординг должен завершаться предложением:
- разложить хотя бы одну цель на подзадачи;
- передать это в Quest Architect / Goblin tools;
- заполнить не более 10 action steps.

### 4.10 Risk Mode
`RISK_MODE` — это не постоянная стартовая роль игрока, а динамическое правило игры.

Сценарий:
- у игрока падает `HP`;
- система предлагает восстановление и более мягкий режим;
- игрок может осознанно выбрать продолжить крафтить ценой здоровья;
- это и есть игровой риск, который должен фиксироваться как часть правил/событий системы.

### 4.11 STUCK Detection System
Система имеет два механизма обнаружения залипания:

#### 4.11.1 STUCK_TIME: Молчание (25+ часов)
`STUCK_TIME` — это alert-сигнал о времени, прошедшем с последнего успешного события.

Правило:
- если 25+ часов молчания во время ACTIVE attempt;
- система генерирует `STUCK_TIME_DETECTED` событие;
- инициирует контакт с Therapist для получения помощи;
- не меняет статус attempt (остается `ACTIVE`);
- не заменяет fail-логику по 5 дням.

#### 4.11.2 STUCK_TRIGGERED: Денежный блок (X THB за Y дней)
`STUCK_TRIGGERED` — это состояние отсутствия денежного прогресса несмотря на активность.

Правило:
- `STUCK_TRIGGERED` возникает, если нет денежного прогресса `X` за `Y` дней;
- при этом у игрока были действия по цели: задачи или KPI;
- параметры `X/Y` задает игрок в правилах во время онбординга;
- логируется как `STUCK_TRIGGERED` событие в `GAME_LOG`;
- инициирует специальный контакт с боссом / финансовым анализом;
- не завершает attempt автоматически.

Различие:
- `STUCK_TIME` = нет событий, срочная помощь;
- `STUCK_TRIGGERED` = есть события, но нет денежного результата.

### 4.12 Energy constraint
Работа возможна только при энергии.

Каноническое правило:
- `2 из 3 (сон / еда / вода)` = день засчитан;
- `1 из 3` = тоже допустимый день;
- `0 из 3` = нейтрально.

Это правило влияет на `attempt consistency`.

Дополнительные safety thresholds являются player-defined rules:
- при `HP < 50` нельзя усиливать цели и запускать тяжелый авто-анализ;
- при `HP < 30` система сокращает штрафы, текст и рекомендации.

---

## 5. Целевые листы и модели данных

## 5.1 `PROFILE_GAME`
`PROFILE_GAME` — это гибридный лист:
- сверху он хранит narrative goals и их decomposition-карточки;
- слева внизу он показывает snapshot игрока;
- большая часть snapshot должна собираться формулами из других листов;
- прямые записи workflow сюда допускаются только в строго оговоренных зонах.

### 5.1.1 Goal zone
Верхняя часть листа используется как goal-tracker.

Канонические top goals сейчас живут в колонке `C`:
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

Это narrative/top goals игрока.

### 5.1.2 Goal decomposition zone
Quest Architect в режиме Goblin tools имеет прямое право писать только в зону подзадач для goal cards.

Канонические колонки action steps:
- `G`, `L`, `Q`, `V`, `AA`, `AF`, `AK`, `AP`, `AU`, `AZ`

Канонические строки action steps:
- `17`, `19`, `21`, `23`, `25`, `27`, `29`, `31`, `33`, `35`

Итого:
- максимум 10 подзадач на goal card;
- одна подзадача на одну ячейку;
- Goblin tools не должен писать в другие части snapshot.

Дедлайн по каждой goal card:
- `H`, `M`, `R`, `W`, `AB`, `AG`, `AL`, `AQ`, `AV`, `BA`
- строка `37`

Награда по каждой goal card:
- те же колонки
- строка `38`

### 5.1.3 Snapshot zone
Нижний левый блок листа содержит snapshot игрока в key/value формате.
Фактическая CRM-схема является каноном и важнее старых PRD-описаний.

Ключи, уже присутствующие в CRM:
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

Дополнительное UI-правило:
- `D35` используется как мягкий pre-game label;
- до `ONBOARDING_COMPLETED` там показывается `Подготовка мира`;
- после старта игры ячейка возвращается к нормальному отображению `LEVEL_NAME`;
- `D35` не является источником правды для lifecycle игры и не заменяет `GAME_LOG`.

### 5.1.4 Правило записи в `PROFILE_GAME`
По умолчанию workflow не пишет snapshot-поля напрямую.

Источники значений:
- задачи и completion -> из `Task List` и формул;
- привычки / KPI / rules events -> из `GAME_LOG` и формул;
- деньги -> из `WALLET_Monthly Budget` и формул;
- beliefs -> из `BELIEFS` и формул;
- stage / level readiness / stale logic -> формулы и derived logic;
- goals и decomposition -> прямые записи в goal zone.

Прямые workflow-записи в `PROFILE_GAME` допускаются только для:
- top goals;
- goal decomposition block Quest Architect / Goblin tools;
- точечных технических сценариев, которые будут отдельно описаны при внедрении.

## 5.2 `GAME_RULES`
`GAME_RULES` — единый реестр правил, игровых сущностей и пользовательских настроек.

### Каноническая схема CRM
- `CREATED_TS`
- `ACTIVE`
- `RULE_GROUP`
- `TYPE`
- `ID_NAME`
- `RPG_NAME`
- `TRACK`
- `STAGE_SCOPE`
- `LEVEL_SCOPE`
- `ATTEMPT_SCOPE`
- `VALUE`
- `RESOURCE_TYPE`
- `TARGET_VALUE`
- `PERIOD`
- `TARGET_JSON`
- `EFFECT`
- `EFFECT_TARGET`
- `TRIGGER_JSON`
- `FLAVOR_TEXT`
- `DESC`
- `SOURCE`
- `SORT_ORDER`
- `NOTES`

### Что хранится в `GAME_RULES`
- системные правила progression;
- stage thresholds;
- level naming map;
- rules для attempts;
- habits;
- KPI;
- bosses;
- debuffs;
- reward shop;
- narrative/game declarations игрока;
- параметры treasure и horizon;
- sprint/KPI settings;
- risk-related rules.
- task economics;
- distinction between base rules and tactical rules.

### Что важно зафиксировать
- схема CRM уже канонична;
- старые PRD-описания колонок не применяются, если расходятся с CRM;
- если для новой модели понадобятся дополнительные колонки, они должны добавляться как расширение этой схемы, а не как откат к старой структуре.

### Базовые и тактические правила
Для онбординга и `RULES_EDIT` правила делятся на два класса.

#### Базовые
Меняются только через therapist / reframe:
- структура ресурсов (`XP_Gold`, `HP`, `XP_TOTAL`);
- штрафы за системные провалы;
- fail-threshold;
- `STUCK` thresholds;
- recovery / risk rules;
- energy safety thresholds;
- правила level readiness и финального level up;
- правило reframe после `5` fails.

#### Тактические
Могут меняться свободно, с фиксацией в `GAME_LOG`:
- текущая цель уровня;
- sprint length;
- KPI;
- habits;
- tasks and focus rules;
- reward shop items;
- level names;
- текущие bosses / debuffs, если они не меняют safety-ядро;

## 5.3 `GAME_LOG`
`GAME_LOG` — единый журнал событий.

### Каноническая схема CRM
- `TS`
- `EVENT_TYPE`
- `SOURCE_WORKFLOW`
- `ENTITY_GROUP`
- `ENTITY_ID`
- `ENTITY_NAME`
- `TRACK`
- `STATUS`
- `VALUE`
- `RESOURCE_TYPE`
- `DELTA`
- `PERIOD`
- `DATE_REF`
- `STAGE_AT_TIME`
- `LEVEL_AT_TIME`
- `ATTEMPT_ID`
- `CONTEXT_JSON`
- `COMMENT`

### Роль `GAME_LOG`
`GAME_LOG` хранит:
- события попыток;
- события привычек;
- task-related игровые события;
- изменения в правилах;
- примененные дебаффы и боссы;
- игровые финансовые и ресурсные эффекты;
- level и stage transitions;
- служебные игровые подтверждения активности.
- `COMMIT_LOG` как тип событий принятого игрового коммита.

### EVENT_TYPE
Полный канонический словарь `EVENT_TYPE` должен быть свернут отдельно по листу `GAME_RULES` и текущим workflow.

На дату этого документа канон должен отражать active workflows и минимум включает:
- rules: `RULE_CREATED`, `RULE_UPDATED`, `COMMIT_LOG`, `ONBOARDING_COMPLETED`, `ONBOARDING_CHECKPOINT`;
- finance: `GOLD_GAINED`, `GOLD_LOST`;
- engine: `ATTEMPT_STARTED`, `ATTEMPT_FAILED`, `STUCK_TIME_DETECTED`, `STUCK_TRIGGERED`, `CONSISTENCY_CHECKED`, `LEVEL_READY_REACHED`, `STAGE_CHANGED`, `THERAPIST_REQUIRED`, `SPRINT_STARTED`, `SPRINT_ENDED`, `BOSS_ACTIVATED`, `TREASURE_HORIZON_REACHED`, `TASK_REMINDED`;
- quest architect: `HABIT_DONE`, `HABIT_SKIPPED`, `TASK_CREATED`, `TASK_UPDATED`, `TASK_STATUS_CHANGED`, `TASK_COMPLETED`, `DAY_PLAN_UPDATED`, `GOAL_DECOMPOSED`;
- therapist: `BELIEF_RECORDED`, `HP_GAIN`, `HP_SPENT`, `BOSS_DEFEATED`, `THERAPY_DISCUSSION`, `THERAPY_SESSION_COMPLETED`, `LEVEL_UP_INTERVIEW_STARTED`, `LEVEL_UP_APPROVED`, `LEVEL_UP_HELD`, `FIVE_FAILURES_ANALYZED`, `TARGET_REVIEWED`, `THERAPY_REFRAME_STARTED`, `THERAPY_REFRAME_COMPLETED`.
- service: `SERVICE_EFFECT_RECORDED`.

Правило нормализации:
- `EVENT_TYPE` не должен кодировать вариант решения, если его можно хранить в `STATUS`;
- пример: `TARGET_KEEP / TARGET_ADJUST / TARGET_REPLACE` нормализуются в `TARGET_REVIEWED` + `STATUS = KEEP|ADJUST|REPLACE`.

### Attempt activity and non-activity
Для lifecycle attempt события делятся на две группы.

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

Следствие:
- service effects можно логировать, но они не продлевают attempt;
- новый attempt стартует любым следующим non-service activity event;
- `ONBOARDING_COMPLETED` может сразу же создать `ATTEMPT_STARTED`.

### Каноника `SPRINT / BOSS / TREASURE`

`Sprint`:
- это ограниченный по длине тактический цикл внутри активной игры;
- не равен `attempt`: `attempt` показывает жив ли контакт игрока с игрой, а `sprint` показывает текущий тактический отрезок внутри этого контакта;
- `SPRINT_STARTED` открывает новый спринт;
- `SPRINT_ENDED` закрывает текущий спринт по длине или циклу;
- в каждый момент времени у игрока должен быть не более одного открытого спринта.

`Boss`:
- это формализованное препятствие текущего спринта, явно заданное в `GAME_RULES`;
- на один спринт активируется один текущий босс;
- выбор босса должен быть детерминированным и воспроизводимым;
- технический `status` workflow не должен менять выбранного босса;
- босс считается закрытым только через `BOSS_DEFEATED`;
- `BOSS_DEFEATED` записывает Therapist в режиме FINAL, если в контексте присутствует `active_boss_name`; ENGINE не пишет это событие;
- победа над боссом не равна окончанию спринта и должна опираться на отдельное boss-condition правило;
- для production в `GAME_RULES` должно быть больше `3` активных boss-правил; рекомендуемый минимум — `4`.

`Treasure`:
- это целевая ценность уровня;
- `treasure horizon` — это дата или срок, к которому текущий treasure-cycle должен быть завершён, пересобран или честно переопределён;
- `TREASURE_HORIZON_REACHED` означает стратегический сбой текущей рамки цели, а не автоматический конец всей игры;
- после `TREASURE_HORIZON_REACHED` допустимы штраф, `THERAPIST_REQUIRED` и возврат к пересмотру цели и планирования.

### Дополнительные правила ENGINE
- `onboarding_completed` определяется через `GAME_LOG`, а не через хардкод и не через один только snapshot;
- базовая проверка: `logRows.some(r => r.EVENT_TYPE === 'ONBOARDING_COMPLETED')`;
- при новом `ATTEMPT_FAILED` ENGINE обязан дополнительно писать `GOLD_LOST` для сгорания `XP_GOLD`;
- этот `GOLD_LOST` должен использовать `RESOURCE_TYPE = XP_GOLD`, отрицательный `DELTA` и не должен дублироваться при повторном прогоне;
- значение штрафа берётся из правила `FAIL_PENALTY` / `fail_penalty_xp_gold` в `GAME_RULES`;
- если штраф задан как `ALL`, система списывает весь доступный `XP_GOLD`, но не уходит в отрицательный остаток;
- ENGINE после чтения `GAME_LOG` локально фильтрует события: всегда оставляет structural events и окно последних 90 дней для остального потока.

Structural events для local filtering:
- `ONBOARDING_COMPLETED`
- `ATTEMPT_STARTED`
- `ATTEMPT_FAILED`
- `SPRINT_STARTED`
- `SPRINT_ENDED`
- `LEVEL_UP_APPROVED`
- `THERAPIST_REQUIRED`

## 5.4 `Task List`
`Task List` остается единственным источником правды по задачам.

Правила:
- новая задача сначала создается в `Task List`;
- daily planner строится из `Task List`;
- effective date:
  - если есть `Due Date`, используется `Due Date`;
  - иначе используется `Start Date`.

### Экономика задач
Каждая задача относится к одному из треков:
- `Craft` — заработок денег, опыта, знаний;
- `Vitality` — тело, здоровье, энергия, красота;
- `Social` — отношения, влияние, признание, любовь, дружба.

Каждая задача имеет одно значение в колонке `XP| HP`.

Правила:
- положительное значение = награда;
- отрицательное значение = цена / штраф / boss effect;
- тип ресурса выводится из track и rules;
- для `Vitality` это по умолчанию `HP`;
- для `Craft` это по умолчанию `XP_Gold`;
- для `Social` ресурс определяется rules игрока.

## 5.5 `BELIEFS`
`BELIEFS` — лист инсайтов Therapist.

Он является источником для:
- narrative change;
- терапевтических событий;
- `BELIEFS_FROM_LEVEL_START` через формулы.

## 5.6 `WALLET_Monthly Budget`
`WALLET_Monthly Budget` — источник финансовых транзакций.

Именно он, а не snapshot, является базой для:
- `BALANCE_THB`
- `BALANCE_RUB`
- `BALANCE_USD`
- `MONEY_GROWTH_LEVEL`

Каноническое уточнение:
- каждая валидная финансовая операция должна сохраняться в `WALLET_Monthly Budget`;
- реализация записи может быть любой, если независимые транзакции не теряются, не сливаются и не перезаписываются;
- money logic для первого level-up учитывает только данные после `LEVEL_START_TS`.

---

## 6. Онбординг
Онбординг — это не просто анкета, а запуск игровой системы.

Цели онбординга:
- после `/start` или первого контакта сразу объяснить, что это за игра, в тоне Секретутки;
- дать мягкий и понятный переход в настройку игры;
- зафиксировать хотя бы одну цель;
- собрать базовые игровые правила;
- создать начальный набор rules;
- записать стартовые события в `GAME_LOG`;
- запустить первый attempt;
- закончить приглашением разложить цель на подзадачи.

### 6.1 Что должен собрать онбординг
- bio-context игрока;
- sprint length;
- task economics and examples;
- habits;
- KPI;
- bosses;
- debuffs / risk logic;
- reward shop;
- хотя бы одну top goal / treasure;
- horizon;
- названия как минимум 4 уровней;
- rules, по которым считается attempt consistency;
- старт нового attempt.

Рекомендуемый порядок блоков теперь микрошаговый: `1 вопрос = 1 ответ`.

Канонический FSM onboarding:
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
- текущие шаги `BOSS_1` и `BOSS_2` достаточны только как старт онбординга, но не как production-минимум;
- до рабочей версии в `GAME_RULES` должно быть больше `3` активных боссов;
- рекомендуемый минимум: `4` отдельных boss-правила, чтобы hash-based выбор не был слишком предсказуемым.

Дополнение по шагу `FAIL_PENALTY`:
- это обязательный шаг онбординга;
- Секретутка спрашивает, сколько `XP_GOLD` сгорает при провале attempt;
- ответ должен поддерживать как минимум fixed number и режим `ALL`;
- правило записывается в `GAME_RULES` как базовое правило штрафа fail-attempt.

### 6.2 Planned second-wave implementation
Следующие решения согласованы как часть канона, даже если ещё не во всех workflow реализованы:
- `main` должен читать последние строки из `BELIEFS` и передавать в основной routing/context слой как `beliefs_summary`;
- ENGINE должен вести lifecycle спринта:
  - читать последний `SPRINT_STARTED`;
  - учитывать длину спринта из `GAME_RULES`;
  - при истечении длины писать `SPRINT_ENDED` и новый `SPRINT_STARTED`;
  - при старте спринта активировать босса через `BOSS_ACTIVATED`;
- ENGINE должен контролировать treasure horizon:
  - читать `GOAL_HORIZON` из `GAME_RULES`;
  - при достижении горизонта писать `TREASURE_HORIZON_REACHED`;
  - вместе с этим писать штрафной `GOLD_LOST` и `THERAPIST_REQUIRED`.

### 6.1A UX-вход в онбординг
Первый вход должен работать так:
- если правил игры еще нет, `main` отправляет промо-текст о том, что такое Секретутка;
- вместе с промо-текстом Telegram показывает кнопку `Переход к настройке`;
- игрок может подтвердить старт как кнопкой, так и естественной фразой.

Подтверждением старта считаются не только `ок` / `да`, но и разговорные формулировки:
- `давай`;
- `хочу`;
- `готова` / `готов`;
- `поехали` / `погнали`;
- `старт`, `стартуем`, `начинаем`;
- `перейти к настройке`, `переход к настройке`, `к настройке`, `настройка`;
- близкие английские формы вроде `go`, `go ahead`, `let's go`.

Если игрок подтверждает старт, система должна переходить в `ONBOARDING_INTRO`, а затем в первый сбор данных, а не повторять промо.
После `PROMO` второй ответ игрока должен вести в `ONBOARDING_INTRO`, а не сразу в сбор данных.

### 6.2 Что должен сделать workflow онбординга
После подтвержденного прохождения онбординга workflow должен:
1. записать нужные правила и игровые сущности в `GAME_RULES`;
2. записать стартовые события в `GAME_LOG`;
3. записать `ONBOARDING_COMPLETED`;
4. сразу после этого иметь право создать `ATTEMPT_STARTED` для `attempt_id = 1` со статусом `ACTIVE`;
5. закончить предложением разложить одну из целей на подзадачи в goal block.

Во время прохождения workflow должен сохранять checkpoint онбординга в `GAME_LOG`, чтобы:
- помнить `current_state`;
- помнить `draft_json`;
- не сбрасывать пользователя в начало после каждого нового сообщения;
- продолжать диалог по `session_id`.

Во время онбординга система должна просить игрока задавать сущности сразу вместе с ценой / наградой там, где это применимо:
- `Привычка — награда`
- `Задача — награда/цена`
- `Награда — стоимость`
- `Босс — стоимость`

Отдельные правила сбора:
- `BIO_CYCLE_DAY` допускает как номер дня цикла, так и дату начала недавней менструации;
- вопрос про базовые привычки как отдельный state убран;
- блок про треки и задачи идет раньше привычек;
- для задач:
  - `Craft` -> `XP_Gold`
  - `Social` -> `XP_Gold`
  - `Vitality` -> `HP`
- KPI формулируются как повторяющиеся действия с минимальным порогом за спринт и честной наградой за выполнение;
- `GOAL_NAME` идет сразу после KPI.

Goal сама по себе не начисляет ресурс автоматически.
Вместо этого игроку доступна награда за goal через `SHOP` и goal-card reward.

Отдельное обязательное правило:
- перед контрактом онбординг обязан спросить `Money Net X` для текущего уровня;
- это player-defined финансовый порог, по которому потом считается финальный ап уровня;
- contract summary должен показывать этот порог явно;
- без ответа на `Money Net X` онбординг не считается завершенным.

### 6.3 Чего онбординг не должен делать
- не должен вручную переписывать формульный snapshot `PROFILE_GAME`;
- не должен подменять `Task List`;
- не должен вводить отдельную старую FSM-логику, если она противоречит канону игры.

### 6.4 Manual QA сценарии
Этот набор сценариев нужен для ручной проверки, что `main`, onboarding и `engine` остаются согласованными.

1. `Старт до игры`
- Сообщение: `старт`
- Ожидание:
  - `main` уводит в onboarding
  - пользователь получает promo/start текст
  - игра еще не считается начатой
  - `engine` возвращает `pre_game_mode = true`

2. `Resume onboarding`
- Действие:
  - пройти несколько шагов
  - прервать диалог
  - затем написать `продолжим`
- Ожидание:
  - workflow поднимает последний `ONBOARDING_CHECKPOINT`
  - диалог продолжается с последнего шага, а не с `PROMO`

3. `LEVEL_NAMES -> MONEY_NET_THRESHOLD -> CONTRACT`
- Действие:
  - дойти до шага `LEVEL_NAMES`
  - ввести 4 названия уровней
- Ожидание:
  - следующий шаг — `MONEY_NET_THRESHOLD`
  - после ответа на него показывается контракт
  - в контракте присутствует строка про `Money Net ... THB в месяц 3 месяца подряд`

4. `Подтверждение контракта`
- Сообщение: `да, подтверждаю`
- Ожидание:
  - сначала пишутся rows в `GAME_RULES`
  - затем пишутся события в `GAME_LOG`
  - в `GAME_LOG` появляются `ATTEMPT_STARTED` и `ONBOARDING_COMPLETED`
  - только после этого пользователь получает текст об активации

5. `Статус после активации`
- Сообщение: `покажи статус`
- Ожидание:
  - `main` больше не отвечает как для pre-game режима
  - показывает HP, stage, attempt, active tasks, overdue

6. `Fail по тишине`
- Подготовка:
  - с последней meaningful activity прошло `>= 5` дней
- Действие:
  - запустить `engine`
- Ожидание:
  - создается `ATTEMPT_FAILED`
  - попытка становится неактивной
  - в ответе есть честное описание провала и штрафа

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
  - в `GAME_LOG` накоплены 5 событий `ATTEMPT_FAILED`
- Действие:
  - запустить `engine`
- Ожидание:
  - создается `THERAPIST_REQUIRED`
  - `recommended_next_workflow = THERAPIST`
  - planner/evening review не должны форситься поверх therapist-маршрута

9. `LEVEL_READY vs финальный ап`
- Подготовка:
  - `TOTAL_PROGRESS >= 1`
  - `BALANCE_THB >= 1 000 000`
  - есть 3 месяца `Money Net >= X`
- Ожидание:
  - сначала возникает `LEVEL_READY_REACHED`
  - `final_level_up_eligible` считается отдельно
  - threshold для финального апа берется из `final_level_up` правила текущего `level_scope`

Что считать провалом ручного теста:
- игра считается начатой без `ONBOARDING_COMPLETED`
- activation text возвращается, а запись в листы не произошла
- после `LEVEL_NAMES` не спрашивается `Money Net X`
- contract summary не показывает `Money Net X`
- `main` и `engine` по-разному трактуют pre-game / active game
- `engine` берет не тот `final_level_up` threshold для текущего уровня

---

## 7. Workflow-логика

### 7.1 Main
Main workflow:
- понимает намерение;
- маршрутизирует запрос;
- возвращает только честный ответ;
- не должен сам выдумывать подтверждение записи.

Дополнительно main должен:
- распознавать `/start` как промо-вход в игру;
- если правил нет, направлять пользователя в `ONBOARDING`;
- показывать кнопку `Переход к настройке` только на входе в rules-flow;
- передавать стабильный `session_id` из Telegram `chat_id`;
- не подменять собой onboarding FSM.

Write contract:
- читает: `PROFILE_GAME`, `GAME_RULES`, `GAME_LOG`, service context;
- по умолчанию не пишет;
- может логировать только `SERVICE_EFFECT_RECORDED`, если был значимый сервисный эффект;
- `SERVICE_EFFECT_RECORDED` в `main` допустим только для реально использованного read-only контекста, а не для любого вызова маршрутизатора;
- service-log из `main` должен фиксировать сервисный источник, тип эффекта `READ`, дату, `STAGE_AT_TIME`, `ATTEMPT_ID` и компактный `CONTEXT_JSON` c агрегатами контекста;
- не должен создавать игровую активность самим фактом маршрутизации.

### 7.2 Quest Architect
Quest Architect отвечает за:
- задачи;
- planning;
- decomposition goals;
- daily / evening review;
- при необходимости прямую запись в goal decomposition zone.

Он не должен:
- напрямую вести игровой snapshot за пределами goal zone;
- подменять `GAME_LOG` собственной теневой системой событий.

Write contract:
- пишет в `Task List`, goal zone `PROFILE_GAME`, daily planner и `GAME_LOG`;
- обязан создавать task/habit/planner/decomposition события как часть event-sourcing слоя;
- top goals не должны дублироваться в `Task List` как задачи.

### 7.3 Therapist
Therapist:
- работает с эмоциональными сообщениями и инсайтами;
- пишет в `BELIEFS`;
- создает события, из которых формулами считается игровой прогресс.

Write contract:
- discussion тоже должна писать в таблицу;
- reframe rules работает только через новую версию правила + деактивацию старой;
- `BELIEFS` считается живым operational log, а не архивом.

### 7.4 ENGINE
ENGINE:
- делает тихие системные пересчеты;
- следит за тишиной и провалом attempts;
- считает readiness;
- владеет reminder-audit логикой;
- собирает единое Telegram-сообщение;
- не должен полагаться на устаревшие листы вроде `STAGE_RULES` или `RESOURCE_LOG & BOSS_LOG`.

Write contract:
- читает `PROFILE_GAME`, `GAME_RULES`, `GAME_LOG`, `Task List`, `WALLET_Monthly Budget`;
- пишет только в `GAME_LOG`;
- считает activity только по `EVENT_TYPE`;
- financial и therapist events обязаны продлевать attempt наравне с task/habit events.
- `TASK_REMINDED` принадлежит `ENGINE` и является audit-event, а не task-status;
- `ENGINE` не должен менять `Task List.STATUS` на `🔔 Reminded`;
- reminder должен использовать `row_number` как стабильный `ENTITY_ID`;
- источник времени reminder нормализуется так: сначала точное время в `Task List`, затем slot из daily sheet; если времени нет, reminder не отправляется;
- единый cron-канон `ENGINE`: `HOURLY`, `MORNING`, `EVENING`, `MIDNIGHT`;
- `HOURLY` делает только тихий audit/reminder scan и не дублирует утренние или вечерние сценарии;
- чтение данных должно идти по bounded/incremental strategy: hourly-run читает ограниченные диапазоны и хвосты, а deep rebuild допускается только отдельным ручным запуском;
- выбор босса должен быть детерминированным, через hash-based `chooseBossRule(seed)`, а не случайным `Math.random`;
- seed босса должен быть привязан к стабильной идентичности спринта и не должен зависеть от технического `status` (`STARTED` / `RESTARTED`);
- минимально допустимый seed до появления отдельного `sprint_id`: `attemptId + sprint_start_date`.
- для production в `GAME_RULES` должно быть больше `3` активных boss-правил; рекомендуемый минимум — `4`, иначе ротация при hash-based выборе становится слишком предсказуемой.

### 7.5 Finance
Finance workflow:
- пишет в `WALLET_Monthly Budget` и `GAME_LOG`;
- обязан создавать `GOLD_GAINED` или `GOLD_LOST`;
- финансовая запись считается игровой активностью;
- `appendOrUpdate` допустим, если match идёт только по `ENTRY_ID`;
- `TYPE` не является допустимым ключом для upsert;
- техническая реализация может быть любой, если каждая валидная транзакция сохраняется как независимая операция и не происходит потери, слияния или перезаписи независимых записей.

### 7.6 Service workflows
Service workflows вроде `Get Schedule Context`:
- могут читать нужный контекст для памяти о задачах, плане и кошельке;
- не должны менять attempt/stage/level;
- по умолчанию ничего не пишут;
- не должны засорять `GAME_LOG` полным трассировочным логом каждого read-only вызова.

### 7.7 Migration spec: Reminder -> ENGINE
Целевой канон:
- отдельный `Notifier` перестаёт быть самостоятельным рабочим слоем;
- reminder-логика переезжает в `ENGINE`;
- `TASK_REMINDED` остаётся service/audit event и не считается activity;
- статус задачи остаётся предметным и не меняется на reminder-status.

Что переносится в `ENGINE`:
- hourly reminder window `[now+30min, now+90min]`;
- фильтр активных task-statuses без отдельного `🔔 Reminded`;
- отправка Telegram reminder;
- запись `TASK_REMINDED` в `GAME_LOG`.

Что должно исчезнуть из legacy `Notifier`:
- отдельный hourly trigger;
- любая смена `Task List.STATUS` ради напоминания;
- матчинг по title как по ключу;
- неканоническая короткая запись в `GAME_LOG`.

Минимальный техдолг перед переносом:
- использовать `row_number` как `ENTITY_ID` reminder-события;
- не отправлять reminder без валидного времени старта;
- перевести `ENGINE` на bounded/incremental reads;
- развести `run_mode`, чтобы `HOURLY` не конфликтовал с `MORNING` и `EVENING`.

### 7.7.1 Task upsert canon
Для `Task List`:
- разрешён `appendOrUpdate`, если match идёт только по `TASK_ID`;
- `TASK_ID` живёт в `Task List!Q`;
- `row_number` используется только как временная ссылка для audit/reminder и не является каноническим ключом задачи;
- title задачи не является допустимым upsert-key.

---

## 8. Текущее состояние реализации

### 8.1 Что уже совпадает с целью
- есть живые листы `GAME_RULES`, `GAME_LOG`, `PROFILE_GAME`, `Task List`, `BELIEFS`, `WALLET_Monthly Budget`;
- `PROFILE_GAME` уже совмещает goal-tracker и snapshot;
- data entry по задачам и финансам уже живет в отдельных рабочих листах;
- есть выделенная зона под decomposition goals.

### 8.2 Что пока не внедрено
- `GAME_RULES` и `GAME_LOG` почти пустые и ждут реального онбординга и дальнейшей игры;
- единая event-sourcing логика еще не стала реальным основанием для всех workflow;
- часть workflow пока не покрывает весь канонический `EVENT_TYPE` set;
- отдельный legacy `Notifier` еще не влит в `ENGINE`;
- старые workflow продолжают ссылаться на устаревшие листы и поля.

---

## 9. Разрыв между целью и текущими workflow
На текущий момент устарели или требуют рефакторинга следующие допущения:
- отдельный лист `STAGE_RULES`;
- логика `RESOURCE_LOG & BOSS_LOG` как главного event-слоя;
- старая схема `PROFILE` вместо `PROFILE_GAME`;
- ручное назначение stage;
- старое понимание level name как пользовательского текущего выбора;
- onboarding, который пишет в старые поля и не запускает корректный `GAME_LOG`;
- `Quest Architect`, который все еще опирается на старые сущности и не описан как goal decomposer для `PROFILE_GAME`.

Следствие:
- документы должны быть приведены к CRM;
- prompts должны быть приведены к канону;
- workflow-json требуют отдельного технического обновления после синхронизации документации.

---

## 10. Критерии готовности новой модели
Новая модель считается внедренной, когда одновременно выполняются условия:
1. `PROFILE_GAME` описан и используется как реальный гибрид goals + snapshot.
2. `GAME_RULES` использует каноническую CRM-схему и хранит игровые сущности.
3. `GAME_LOG` становится единым журналом подтвержденных игровых событий.
4. attempt lifecycle работает по правилу "5 дней тишины -> FAILED".
5. stage считается автоматически по `STAGE_XP` внутри текущего attempt.
6. `LEVEL_READY` считается по формуле и подтверждается игроком вручную.
7. онбординг создает первый attempt и заканчивается handoff на decomposition хотя бы одной цели.
8. README и prompt-файлы не противоречат этому PRD.
