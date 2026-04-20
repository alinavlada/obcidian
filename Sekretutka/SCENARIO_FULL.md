# Секретутка — Полный сценарий системы

**Версия:** на основе кода 2026-04-14  
**Формат:** каждый шаг = триггер → что происходит внутри → что пишется в таблицу → что получает юзер

---

## Легенда воркфлоу

| Аббревиатура | Воркфлоу |
|---|---|
| **MAIN** | `1. Sekretutka_main` |
| **RULES** | `1.0 Sekretutka_Game Rules` |
| **ENGINE** | `1.1 Sekretutka_ENGINE_updated` |
| **SCHED** | `1.2 Sekretutka_Get Schedule Context` |
| **QUEST** | `1.3 Quest_Architect_Daily_Planner` |
| **FIN** | `1.4 Sekretutka_finance` |
| **THER** | `1.5 Sekretutka_Therapist` |
| **NOTIF** | `1.6 Sekretutka_Notifier` (legacy) |
| **LK** | `1.7 Sekretutka_LK_Render` |

---

## СЦЕНАРИЙ 1 — Первый запуск: /start

**Условие:** GAME_LOG не содержит события `ONBOARDING_COMPLETED`

### Шаг 1.1 — Юзер пишет `/start`

**MAIN:**
1. Telegram Trigger → If LK Route → нет совпадения → If (voice?) → нет → Read PROFILE_GAME → Read GAME_RULES → Read GAME_LOG → Read BELIEFS → SCHED → Code in JavaScript
2. Code in JavaScript определяет: `isStartCommand=true`, `onboardingCompleted=false`
3. Sekretutka (AI Router) — вызывается, но Route Decision игнорирует его ответ
4. Route Decision: `isStartCommand && !onboardingCompleted` → `routeTarget='none'`, `output=startPromoText`, `is_start_promo=true`
5. Все If-ноды роутинга пропускаются → Format Final Response → `shouldForceStartPromo=true` → берёт `routeOutput`

**В таблицу:** ничего не пишется

**Юзер получает:**
> Привет! 💁‍♀️  
> Я Секретутка — твоя помощница, высокоинтеллектуальный симбиоз личного секретаря (расписание задач и отслеживание привычек), финансиста, Мастера RPG-игр и коуча-психолога. Моя миссия — вести тебя к результату, сохраняя здоровье и капитал, поэтому я помогаю превратить жизнь в игру и увидеть себя настоящую.
>
> В игре правила не прописаны — ты создаёшь их сама, а система честно фиксирует только то, что реально прошло через workflow.
>
> Давай соберём твою версию игры и запустим первый уровень?

---

### Шаг 1.2 — Юзер отвечает "давай" (или "да", "ок", "погнали" и т.д.)

**MAIN:**
1. Route Decision: `onboardingConfirmation=true`, `!onboardingCompleted=true` → `shouldForceOnboarding=true`
2. `routeTarget='rules'`, передаёт в RULES: `{mode:'ONBOARDING', current_state:0, draft_json:{}, session_id:chat_id, intro_style:'coach'}`

**RULES:**
1. Normalize Input → Mode Switch → ветка ONBOARDING
2. Read Onboarding GAME_LOG (`GAME_LOG!B2:S5000`) → Hydrate Onboarding Session
3. Hydrate: нет checkpoint → `has_onboarding_checkpoint=false`
4. Onboarding FSM Router: `state=0`, `yes("давай")=true` → action=ASK, advance to `state=1`
5. Onboarding Action Switch: ASK → Build Onboarding Checkpoint
6. Build Onboarding Checkpoint формирует строку checkpoint, `reply_to_user=prompts[1]`
7. Параллельно: Append Onboarding Checkpoint GAME_LOG + Return Response

**В GAME_LOG пишется** (через Append Onboarding Checkpoint GAME_LOG, `GAME_LOG!B:S`):
```
TS | ONBOARDING_CHECKPOINT | GAME_RULES_WORKFLOW | ONBOARDING | {chat_id} | Onboarding Session | | IN_PROGRESS | 1 | ... | {today} | ... | {checkpoint_json} | {reply_text[:400]}
```

**Юзер получает:**
> Сейчас я проведу тебя по настройке игры шаг за шагом. Сначала мы поймём твоё текущее состояние и реальную нагрузку, потом соберём привычки, задачи, KPI, цель и правила игры, а в конце зафиксируем твой контракт. Я буду задавать по одному вопросу за раз.

*(Система ждёт ответа, checkpoint state=1 сохранён в GAME_LOG)*

---

## СЦЕНАРИЙ 2 — Онбординг: состояния 0–39

> Каждый шаг: юзер отвечает → MAIN роутит в RULES → Hydrate восстанавливает state из checkpoint → FSM Router определяет action → EXTRACT → AI Extractor (GPT-4o-mini) → Draft Merge → checkpoint обновляется → юзер получает следующий вопрос

### State 1 → 2: Юзер пишет что угодно в ответ на инструкцию

FSM Router: `state=1` → всегда advance to state=2

**Юзер получает (state 2 prompt):**
> Для начала откалибруем систему. Оцени свой текущий уровень энергии по шкале от 0 до 10, где 10 — "могу свернуть горы", а 0 — "не могу встать с кровати".

---

### State 2: Юзер пишет оценку энергии ("8", "6 из 10", "восемь — горы сдвигаю")

**RULES:**
- FSM Router: `state=2` → action=EXTRACT, передаёт в AI Extractor
- AI Extractor (GPT-4o-mini): инструкция `{hp_self_score_10: number (0-10)}`, user_input="8" → возвращает `{hp_self_score_10: 8}`
- Fallback: если AI не распознал → парсим число напрямую из user_input
- Validation: если hp не 0-10 → `askSame("Нужна оценка от 0 до 10. Можно просто числом: 6.")`
- Draft Merge: `draft.bio.hp_self_score_10=8, hp_current_estimate=80`
- Build Checkpoint → Append GAME_LOG + Return Response

**Юзер получает (state 3):**
> Какой сейчас день цикла? Если не знаешь, напиши дату начала недавней менструации.

---

### State 3: Юзер пишет день цикла ("18", "23 марта", "не знаю")

- AI Extractor: `{cycle_day: number|null, last_menstruation_start: string|null, cycle_day_unknown: boolean}`
- Fallback: чистое число → `cycle_day`; дата → `last_menstruation_start`
- Draft Merge: `draft.bio.cycle_day`, `last_menstruation_start`, `cycle_day_unknown`

**Юзер получает (state 4):**
> Как сейчас ощущается тело? Можно коротко и по-человечески.

---

### State 4: Юзер описывает состояние тела ("устала, болит шея")

- AI Extractor: `{body_state: string}`
- Fallback: берём raw user_input

**Юзер получает (state 5):**
> Есть ли сейчас что-то острое: болезнь, сильная усталость, восстановление после перегруза, боль, недосып?

---

### State 5–8: Базовые вопросы нагрузки

| State | Вопрос | AI extracts |
|---|---|---|
| 5 | Острые состояния? | `acute_condition: string\|null` |
| 6 | Что заполняет дни? | `daily_main_activity: string` |
| 7 | Жёсткие обязательства? | `fixed_obligations: string` |
| 8 | Сколько сил остаётся? | `residual_energy: string` |

---

### State 9: Длина спринта ("21", "28 дней", "по циклу")

- AI Extractor: `{sprint_days: number|null, sprint_by_cycle: boolean}`
- Fallback: чистое число → `sprint_days`

**Юзер получает (state 10):**
> Как тебе комфортнее двигаться внутри спринта: равномерно, рывками, спокойно, только по будням, с разгоном в начале?

---

### State 10: Ритм спринта

- AI Extractor: `{sprint_rhythm: string}`

---

### States 11–16: Задачи по трекам (Craft/Social/Vitality) — Queue-based Multi-Task Flow

**Архитектура:** юзер вводит все задачи по одной системе (строки, разделённые переносом), система парсит все задачи в queue, потом по одной спрашивает детали каждой. После последней детали — merge всех в draft и advance.

**State 11: Сбор всех задач Craft**

Юзер получает:
> 💪 **CRAFT-задачи: профессионально значимое и финансовое**  
> Напиши каждую задачу на отдельной строке с наградой в формате: *задача — награда*.  
> Например:
> - сделать квартальный отчёт — 100
> - записать видео для курса — 80  
> - встреча с партнёром — 60
> Можешь добавить столько, сколько хочешь.

**RULES:**
- FSM Router: `state=11` → action=COLLECT_TASKS
- Parse Input: каждая строка → `parseDashFormat()` → `{task_name, xp_gold}`
- All Tasks → `craft_task_queue = [{task_name, xp_gold}, ...]`
- Advance to `state=12`, передать в prompt первую задачу: `current_craft_task = craft_task_queue[0]`
- Fallback для пустого queue: `askSame("Напиши хотя бы одну задачу")`

**State 12: Детали каждой задачи Craft (loop)**

Юзер получает (динамично подставляется имя):
> Для задачи '**сделать квартальный отчёт**': она разовая или повторяющаяся? Если повторяется, насколько часто (еженедельно, ежемесячно)?

**RULES:**
- State 12 может выполниться несколько раз (по числу задач)
- AI Extractor: `{recurring: boolean, frequency?: string}` или fallback текстовый парсинг
- Append в `task_examples[craft].push({...current_task, recurring, frequency})`
- Если это последняя задача в queue → merge, advance to state 13
- Если не последняя → loop: показать следующую задачу в state 12 ещё раз

**State 13: Сбор всех задач Social**

Юзер получает:
> 🤝 **SOCIAL-задачи: отношения, люди, общение**  
> По той же системе: каждую на строку, формат *задача — награда*.  
> Примеры:
> - позвонить маме — 50
> - встреча с друзьями — 40  
> - написать ответное письмо — 30

**State 14: Детали Social (loop)**

Аналогично state 12, но для Social задач.

**State 15: Сбор всех задач Vitality**

Юзер получает:
> 🫀 **VITALITY-задачи: тело, здоровье, восстановление**  
> По той же системе, но награда в HP (а не XP):
> - ходьба 10000 шагов — 30
> - йога сеанс — 25  
> - спорт тренировка — 40

**State 16: Детали Vitality (loop)**

Аналогично state 14.

| State | Действие | AI extracts |
|---|---|---|
| 11 | Сбор всех Craft задач | `{task_name, xp_gold}` × N |
| 12 | Детали каждой Craft | `{recurring, frequency}` |
| 13 | Сбор всех Social задач | `{task_name, xp_gold}` × N |
| 14 | Детали каждой Social | `{recurring, frequency}` |
| 15 | Сбор всех Vitality задач | `{task_name, hp_reward}` × N |
| 16 | Детали каждой Vitality | `{recurring, frequency}` |

**Fallback парсинг:**
- State 11/13/15: `parseDashFormat()` для каждой строки
- State 12/14/16: текстовый парсинг для "разовая"/"повторяющаяся"/"ежемесячно" и т.д.

---

### States 17–22: Привычки

**State 17: Привычка 1 + награда**

Юзер пишет: "медитация — 20" или "медитация, 20"

**RULES:**
- FSM Router: `state=17` → action=EXTRACT
- AI Extractor: `{habit_name: string, reward: number}`
- Fallback: `parseDashFormat()`
- Draft: `draft.habits[0] = {habit_name, reward}`
- Advance to state 18

**State 18: Трек привычки 1 (только трек, никаких других деталей)**

Юзер получает:
> К какому треку относится привычка '**медитация**'?  
> Ты можешь ответить эмодзи, словом, или своими словами:
> - **CRAFT** (💪💼💰📊🎯🚀📈🛠💡) — деньги, доход, работа, дело, навык, опыт, карьера, бизнес, проект, продажи, маркетинг, контент, обучение, творчество
> - **VITALITY** (🫀💚🏃🧘🏋🌿) — тело, здоровье, восстановление, спорт, сон, движение, питание, медитация, йога, тренировка, энергия, самочувствие
> - **SOCIAL** (🤝❤️👥🫂💕🌐👯‍♀️💃) — отношения, семья, люди, общение, дружба, любовь, партнер, родители, дети, коллеги, нетворкинг, влияние, сообщество

**RULES:**
- FSM Router: `state=18` → action=EXTRACT
- AI Extractor инструкция: "Map answer to exactly one of 3 values: CRAFT (...), VITALITY (...), SOCIAL (...). Always return the English name in UPPERCASE."
- Fallback regex парсинг:
  - CRAFT: `/craft|💪|💼|💰|📊|🎯|🚀|📈|🛠|💡|крафт|деньги|доход|работа|дело|навык|опыт|карьера|бизнес|проект|продажи|маркетинг|контент|обучение|творчество/i` → CRAFT
  - VITALITY: `/vitality|🫀|💚|🏃|🧘|🏋|🌿|тело|здоровье|восстановление|спорт|сон|движение|питание|медитация|йога|тренировка|энергия|самочувствие/i` → VITALITY
  - SOCIAL: `/social|🤝|❤|👥|🫂|💕|🌐|👯|💃|отношения|семья|люди|общение|дружба|любовь|партнер|родители|дети|коллеги|нетворкинг|влияние|сообщество/i` → SOCIAL
- Draft: `draft.habits[0].track = UPPERCASE_TRACK`
- Validation: если трек не распознан → `askSame("Выбери один из трёх: CRAFT, VITALITY или SOCIAL")`
- Advance to state 19

---

**State 19: Привычка 2 + награда** → analogue to state 17  
**State 20: Трек привычки 2 (только трек)** → analogue to state 18  
**State 21: Привычка 3 + награда** → analogue to state 17  
**State 22: Трек привычки 3 (только трек)** → analogue to state 18

| State | Вопрос | AI extracts |
|---|---|---|
| 17 | Привычка 1 + награда | `{habit_name, reward: number}` |
| 18 | Трек привычки 1 | `{track: CRAFT\|VITALITY\|SOCIAL}` |
| 19 | Привычка 2 + награда | `{habit_name, reward: number}` |
| 20 | Трек привычки 2 | `{track: CRAFT\|VITALITY\|SOCIAL}` |
| 21 | Привычка 3 + награда | `{habit_name, reward: number}` |
| 22 | Трек привычки 3 | `{track: CRAFT\|VITALITY\|SOCIAL}` |

---

### States 23–25: KPI

| State | Вопрос | AI extracts |
|---|---|---|
| 23 | KPI 1 + награда | `{kpi_name, reward: number}` |
| 24 | KPI 2 + награда | — |
| 25 | KPI 3 + награда | — |

---

### States 26–28: Цель уровня

| State | Вопрос | AI extracts |
|---|---|---|
| 26 | Главная цель уровня | `{goal_text: string}` |
| 27 | Treasure (желаемый результат) | `{treasure_text: string}` |
| 28 | Горизонт (кол-во дней) | `{goal_horizon_days: number}` |

Fallback state 28: чистое число из user_input

---

### State 29: Переход к боссам

FSM Router: `state=29` → просто advance to state 30 (никаких экстракций)

**Юзер получает:**
> Теперь боссы. Это не "плохие черты характера", а то, что чаще всего сбивает тебя с пути. Я зафиксирую их как реальные препятствия игры.

**Юзер получает (state 30):**
> Назови первого босса и его стоимость.  
> Формат: босс — стоимость

---

### States 30–32: Боссы

| State | Вопрос | AI extracts |
|---|---|---|
| 30 | Босс 1 + стоимость | `{boss_name, boss_cost: number}` |
| 31 | Босс 2 + стоимость | — |
| 32 | Триггеры боссов | `{boss_triggers_description: string}` |

Fallback 30/31: `parseDashFormat()`

---

### States 33–38: Игровые правила

| State | Вопрос | AI extracts |
|---|---|---|
| 33 | Stuck-порог ("Y дней, X прогресса") | `{stuck_days: number, stuck_money_threshold: number}` |
| 34 | Штраф за fail ("100", "все") | `{fail_penalty_xp_gold: number\|'ALL'}` |
| 35 | Recovery-правила (HP<60/HP<30) | `{rules_enabled: bool, recovery_active_rule, recovery_passive_rule}` |
| 36 | Магазин наград ("отдых — 50") | `{shop_items: [{name, cost}]}` |
| 37 | Названия 4 уровней | `{level_names: [string×4]}` |
| 38 | Money Net threshold (число THB/мес) | `{money_net_monthly_threshold: number}` |

Fallback state 33: парсинг числа stuck days  
Fallback state 34: число из user_input  
Fallback state 38: число из user_input

---

### State 39: Контракт

FSM Router: `state=39` → `buildContract(draft)` — генерирует персонализированный контракт из всего draft

**Юзер получает:**
> Контракт с подтверждением (AI-составленный текст на основе всех ответов)  
> "Подтверждаешь?" — ждёт ответ

---

**Если юзер говорит "нет" / отказывается:**
- FSM Router: `contractYes(text)=false` → снова возвращает контракт state=39 или askSame

**Если юзер подтверждает ("да", "всё верно", "подтверждаю"):**
- action=ACTIVATE → Build Activation Writes

**Build Activation Writes** → Append Activation GAME_RULES + Append Activation GAME_LOG

**В GAME_RULES пишется** (`GAME_RULES!B:X`, ~22 строки):
- BASE_RULE: resource_model, bio_layer, sprint_rule, fail_rule, recovery_active, recovery_passive, stuck_rule, level_up_rule, attempt_rule, stuck_time_rule...
- Задачи (Craft/Social/Vitality tasks)
- Привычки (habits 1-3)
- KPI (kpi 1-3)
- Боссы (boss 1-2)
- Shop items
- Level names
- Goal / treasure / horizon

**В GAME_LOG пишется** (~4 строки через Append Activation GAME_LOG, `GAME_LOG!B:S`):
- `ONBOARDING_COMPLETED` — онбординг завершён
- `ATTEMPT_STARTED` — первая попытка стартует
- `SPRINT_STARTED` — первый спринт открывается
- `RULE_CREATED` (summary) — правила активированы

**Return Response → MAIN Format Final Response → Telegram:**

**Юзер получает:**
> Ответ от AI Extractor (reply_to_user из Onboarding FSM Router для ACTIVATION) — что-то вроде финального подтверждения что всё настроено

---

## СЦЕНАРИЙ 3 — Логирование привычки

**Условие:** onboardingCompleted=true, hasRules=true

**Юзер пишет:** "медитация сделана" / "пропустила тренировку" / "выполнила habit_name"

**MAIN:**
1. AI Router (Sekretutka) классифицирует: `intent='log_habit'`
2. Route Decision: ищет в `habitAliases` → матч по имени/треку/id привычки
3. Если привычка не найдена → прямой ответ: "Уточни, пожалуйста, какую именно привычку нужно отметить."
4. Если найдена → `routeTarget='quest'`, `quest_mode='HABIT_LOG'`, передаёт: `{item_id, status:'DONE'|'SKIPPED', value:1, date:today}`

**QUEST (режим HABIT_LOG):**
1. Trigger → If Habit Mode? → YES
2. Read GAME_RULES (Quest) → Calc Habit Logic: ищет правило с `type=HABIT`, `id_name=item_id`
3. Has Habit Rule? YES → Append GAME_LOG (Habit), NO → Return "привычка не найдена в правилах"

**В GAME_LOG пишется** (`GAME_LOG!B:S`):
```
TS | HABIT_DONE или HABIT_SKIPPED | QUEST_ARCH | HABIT | {item_id} | {habit_name} | {track} | {status} | {value} | XP_GOLD или HP | {xp_value} | {period} | {today} | {stage} | {level} | {attempt_id} | {context_json} | {comment}
```

**Return Habit Result → MAIN Format Final Response → Telegram**

**Юзер получает:**
> ✅ Привычка [habit_name] засчитана. +[reward] [XP_Gold/HP].

---

## СЦЕНАРИЙ 4 — Добавление задачи

**Юзер пишет:** "добавь задачу: написать отчёт к пятнице"

**MAIN:**
1. AI Router: `intent='add_task'`
2. Route Decision → `routeTarget='quest'`, `quest_mode='TASK_CAPTURE'`

**QUEST (режим TASK_CAPTURE):**
1. Trigger → If Habit Mode? NO → If Task Status Mode? NO → Read PROFILE_GAME → Build AI Context → Quest Architect Agent (GPT-4o)
2. Agent анализирует сообщение, определяет: task name, XP, recurring, due_date, category, priority, track
3. Parse Response → Has Tasks? YES → Read Task List Occupancy (`Task List!B7:Q1000`)
4. Build Task List Writes: создаёт/обновляет задачи, генерирует TASK_ID
5. Has Task Upserts? YES → параллельно: Expand Task List Upserts → Update Task List Rows + Append GAME_LOG (Task Create)

**В Task List пишется** (`Task List`, через native GS node):
- Новая строка: Task, XP|HP, Recurring?, Frequency, Start Date, End Date, Due Date, (пусто col I), Status, Category, Priority, Track, Important?, Urgent?, Notes, TASK_ID

**В GAME_LOG пишется** (`GAME_LOG!B:S`):
```
TS | TASK_CREATED | QUEST_ARCH | TASK | {task_id} | {task_name} | {track} | {status} | {xp} | XP_GOLD | ... | {today} | ... | {context_json} | ...
```

6. Resolve Plan Date → Read Task List For Day → Read Daily Planner Grid → Build Daily Planner Writes
7. Если Should Fill Planner? YES → Update Daily Planner Grid + Append GAME_LOG (Planner)

**В Daily Planner пишется** (если текущий день): добавление в сетку планировщика

**Return Response → MAIN → Telegram**

**Юзер получает:**
> ✅ Задача "написать отчёт" добавлена. [+XP] за выполнение. [Дедлайн: пятница]

---

## СЦЕНАРИЙ 5 — Запись финансов

**Юзер пишет:** "потратила 500 бат на еду в кафе"

**MAIN:**
1. AI Router: `intent='log_finance'`, entities: `{amount:500, currency:'THB', category:'Dining out'}`
2. Route Decision:
   - `detectAmount()` = 500
   - `detectCurrency()` = THB
   - `detectFinanceType()` = "Variable Expenses" (ключевых слов дохода нет)
   - `detectCategory()` = "Dining out" (совпадение по "кафе")
   - `detectAccount()` = "BKK-bank"
3. `routeTarget='finance'`, передаёт: `{amount:500, currency:'THB', finance_type:'Variable Expenses', category:'Dining out', account:'BKK-bank', details:'потратила 500 бат на еду в кафе', date:today}`

**FIN:**
1. Get row(s) in sheet: читает WALLET (все заполненные строки)
2. Calc Row Number: считает заполненные строки, определяет target_row, генерирует ENTRY_ID
3. Append or update row in sheet → пишет строку в WALLET (native GS appendOrUpdate)

**В WALLET_Monthly Budget пишется:**
- Строка на target_row: TYPE=Variable Expenses, DATE=today, AMOUNT=500, CURRENCY=THB, CATEGORY=Dining out, DETAILS=..., ACCOUNT=BKK-bank, ENTRY_ID=generated_hash

4. Determine Event Type: `typeStr="variable expenses"` → не в gainTypes → `eventType='GOLD_LOST'`
5. Append GAME_LOG Event

**В GAME_LOG пишется** (`GAME_LOG!B:S`):
```
TS | GOLD_LOST | FINANCE | WALLET | {entry_id} | Variable Expenses | ... | Dining out | 500 | XP_GOLD | -500 | ... | {today} | ... | {context_json} | ...
```

**Return → MAIN → Telegram**

**Юзер получает:**
> ✅ Записано: -500 THB (Dining out, BKK-bank).

---

## СЦЕНАРИЙ 6 — Планирование дня вручную

**Юзер пишет:** "распланируй день" / "что у меня на сегодня?"

**MAIN:**
1. AI Router: `intent='plan_day'`
2. `routeTarget='quest'`, `quest_mode='PLANNER'`

**QUEST (режим PLANNER):**
1. Read PROFILE_GAME → Build AI Context → Quest Architect Agent
2. Agent читает текущие задачи, HP, stage, приоритеты → строит план дня
3. Parse Response → Should Fill Planner? YES → Resolve Plan Date
4. Read Task List For Day → Read Daily Planner Grid
5. Build Daily Planner Writes → Has Planner Updates? YES → Update Daily Planner Grid
6. Append GAME_LOG (Planner)

**В Daily Planner пишется:** блоки на сегодня + задачи по времени

**В GAME_LOG пишется:**
```
TS | DAY_PLAN_UPDATED | QUEST_ARCH | PLAN | ... | ... | {today} | ...
```

**Юзер получает:**
> (AI-ответ с планом дня: задачи, приоритеты, блоки времени)

---

## СЦЕНАРИЙ 7 — Обновление статуса задачи

**Юзер пишет:** "сделала отчёт" / "задача [название] выполнена"

**MAIN:**
1. AI Router: `intent='add_task'` или `intent='log_habit'`... но реально это task status update
2. Если AI распознал как task status → `quest_mode='TASK_STATUS'`

**QUEST (режим TASK_STATUS):**
1. Trigger → If Habit Mode? NO → If Task Status Mode? YES
2. Resolve Task Status Input → Read Daily Planner Status Grid
3. Build Task Status Write → Read Task List Status Rows (`Task List!B7:Q1000`)
4. Build Task List Status Update: ищет задачу по имени (normalized match)
5. Has Task List Status Update? YES → Update Task Status In Task List (native GS) + Append GAME_LOG (Task Status)
6. Has Task Status Update? YES → Update Task Status In Planner

**В Task List пишется:** Status = "✅ Completed" (или другой финальный статус)

**В GAME_LOG пишется:**
```
TS | TASK_STATUS_CHANGED | QUEST_ARCH | TASK | {task_id} | {task_name} | ... | COMPLETED | ... | {today} | ...
```

**Юзер получает:**
> ✅ Статус обновлён в Task List: [task_name]

---

## СЦЕНАРИЙ 8 — Разговор с Терапевтом

**Юзер пишет:** "хочу поговорить" / "тяжело" / "всё идёт не так"

**MAIN:**
1. AI Router: `intent='therapist'`
2. `routeTarget='therapist'`

**THER:**
1. Normalize Input → Read PROFILE_GAME Snapshot → Read Recent BELIEFS → Read GAME_RULES
2. Build Compact Context: собирает HP, stage, XP, active debuff, boss, последние beliefs, правила
3. AI Therapist Core (GPT-4o) → выдаёт JSON со структурой
4. Parse + Validate JSON: проверяет `status` ∈ {discussion, final, level_up_interview, five_failures_analysis, base_rule_reframe}

**Варианты статуса:**
- `discussion` — обычный диалог, записывает THERAPY_DISCUSSION в GAME_LOG
- `final` — завершение терапии, записывает THERAPY_SESSION_COMPLETED, HP_SPENT/HP_GAIN опционально
- `five_failures_analysis` — анализ 5 провалов, BELIEF_RECORDED в BELIEFS, FIVE_FAILURES_ANALYZED в GAME_LOG
- `base_rule_reframe` — переосмысление базового правила, RULE_UPDATED в GAME_RULES
- `level_up_interview` — интервью перед переходом уровня

5. Build Writes → Has Rule Deactivations? → Has Rule Appends? → Has BELIEF Writes? → Has GAME_LOG Writes?

**В BELIEFS пишется** (при новом инсайте):
- Новая строка: timestamp, trigger, category, method, old_belief, new_insight, story_text

**В GAME_RULES пишется** (при base_rule_reframe):
- Обновление существующего правила (deactivate old → append new)

**В GAME_LOG пишется** (набор событий из build_writes):
```
TS | THERAPY_DISCUSSION / BELIEF_RECORDED / HP_SPENT / BOSS_DEFEATED / ... | THERAPIST | ...
```

**Return Result → MAIN → Telegram**

**Юзер получает:**
> (AI-ответ Терапевта: поддержка, вопросы, рефрейм)

---

## СЦЕНАРИЙ 9 — Просмотр личного кабинета /lk или /status

**Юзер пишет:** `/lk` или `/status`

**MAIN:**
1. If LK Route: `message.text === '/lk'` OR `message.text === '/status'` → TRUE
2. Call '1.7. Sekretutka_LK_Render' → LK самостоятельно шлёт Telegram

**LK:**
1. Read_PROFILE: читает PROFILE_GAME (native GS, все поля по имени)
2. Read_Task_List: читает Task List (только активные задачи)
3. Code in JavaScript: строит HTML-rich сообщение

**Юзер получает (parse_mode=Markdown):**
```
📊 **ЛИЧНЫЙ КАБИНЕТ: EARLY**

❤️ **HP:** 78/100 [████████░░]
✨ **Прогресс уровня:** 34% [███░░░░░░░]

⚔️ **ТЕКУЩИЙ БОСС:** Прокрастинация
💀 **АКТИВНЫЙ ДЕБАФФ:** [если есть — имя + время до снятия]
──────────────────
📜 **АКТИВНЫЕ КВЕСТЫ:**
🔥 Сдать отчёт (50 XP)
📍 Позвонить маме (30 XP)
──────────────────

🌟 **ПЕРЕХОД ДОСТУПЕН!** [если LEVEL_READY=TRUE]
Используй /level_up
```

---

## СЦЕНАРИЙ 10 — Статус игры текстом

**Юзер пишет:** "как у меня дела" / "статус"

**MAIN:**
1. AI Router: `intent='read_status'`
2. Route Decision: прямой вывод (без вызова воркфлоу)

**Юзер получает** (если onboardingCompleted):
> Сейчас у тебя HP [X]/100, stage [stage], попытка [id]. Активных задач на сегодня: [N], просроченных: [N]. [Переход на следующий уровень доступен. — если level_ready]

**Юзер получает** (если !onboardingCompleted):
> Игра ещё не началась: можно заполнять таблицу вручную, но игровой статус появится только после завершения онбординга.

---

## СЦЕНАРИЙ 11 — Редактирование правил

**Юзер пишет:** "измени длину спринта на 14 дней" / "поменяй правило привычки"

**MAIN:**
1. AI Router: `intent='rules_edit'`
2. `routeTarget='rules'`, `mode='RULES_EDIT'`, `edit_target=rule_topic`

**RULES (режим RULES_EDIT):**
1. Mode Switch → ветка RULES_EDIT
2. Read Active GAME_RULES (`GAME_RULES!B2:X1000`)
3. Rules Edit Router: ищет релевантное правило по тексту запроса
4. Rules Edit Action Switch: ASK (уточнить) или EXTRACT (есть что редактировать)
5. EXTRACT → AI Extractor → Extractor JSON Parser → Extract Flow Switch → Rules Edit Build Patch
6. Rules Edit Build Patch: формирует патч правила
7. Is Guarded Base Rule? — базовые правила (BASE_RULE) нельзя редактировать напрямую → защита
8. Has Deactivate Updates? YES → Deactivate Old Tactical Rules (batch update, ставит ACTIVE=FALSE на старое правило)
9. Append Updated Tactical Rules → Append RULE_UPDATED GAME_LOG

**В GAME_RULES пишется** (`GAME_RULES!B:X`):
- Старое правило: ACTIVE ставится в FALSE
- Новое правило: новая строка с обновлёнными параметрами

**В GAME_LOG пишется:**
```
TS | RULE_UPDATED | GAME_RULES_WORKFLOW | RULE | {rule_id} | {rule_name} | ... | UPDATED | ... | {today} | ...
```

**Юзер получает:**
> Правило [название] обновлено.

---

## СЦЕНАРИЙ 12 — Голосовое сообщение

**Юзер отправляет голосовое**

**MAIN:**
1. Telegram Trigger → If LK Route → NO → If (voice?) → YES (`message.voice` существует)
2. TG: Get File Info → TG: Download Voice → OpenAI Whisper (транскрибирует аудио)
3. Set: Voice Text → далее тот же путь что и текстовое сообщение
4. `is_voice=true` → в routerInput добавляется флаг

Дальше полностью идентично текстовому сценарию в зависимости от распознанного intent.

---

## СЦЕНАРИЙ 13 — Прерванный онбординг (возврат)

**Условие:** юзер начал онбординг, остановился на state=15, через 2 дня пишет "давай продолжим"

**MAIN:**
1. Route Decision: `!onboardingCompleted=true` → `routeTarget='rules'`, `current_state=0`

**RULES:**
1. Hydrate Onboarding Session: находит последний ONBOARDING_CHECKPOINT в GAME_LOG → `checkpoint.current_state=15`, `checkpoint.draft_json={...всё собранное до state 15...}`
2. Восстанавливает state=15, draft
3. FSM Router: state=15 → показывает prompts[15] (задача Vitality)

**Юзер получает:**
> Теперь одна задача для трека Vitality. Напиши задачу и количество HP, которые ты ей присваиваешь. Формат: задача — награда/цена

*(Продолжает с места остановки)*

---

## СЦЕНАРИЙ 14 — CRON: MIDNIGHT (00:10 BKK)

**Триггер:** Schedule cron `10 0 * * *` → Set Scheduled Context

`run_mode='MIDNIGHT'` (час < 6), `send_telegram=false`

**ENGINE:**
1. Run Mode Switch → ветка MIDNIGHT → Read PROFILE_GAME Snapshot → ... → Compute Engine State
2. Compute Engine State: пересчитывает все показатели (XP, stage, attempt, stuck_time, fails)
3. Has GAME_LOG Writes? Если есть → Append GAME_LOG Events

**Что может записаться в GAME_LOG:**
- `STUCK_TIME_DETECTED` — если attempt активен И молчание > 25 часов И уже не логировалось сегодня
- `ATTEMPT_FAILED` + `GOLD_LOST` — если silentDays >= failThresholdDays И уже не логировалось сегодня
- `STAGE_CHANGED` — если stage изменился
- `CONSISTENCY_CHECKED` — регулярная проверка
- `SPRINT_ENDED` + `SPRINT_STARTED` — если sprint закончился по длине

4. Quest Needed? questMode='' → НЕТ → Build Final Output
5. Should Send Telegram? `send_telegram=false` → НЕТ

**Юзер не получает ничего**, но GAME_LOG обновляется.

---

## СЦЕНАРИЙ 15 — CRON: MORNING (09:00 BKK)

**Триггер:** Schedule cron `0 9 * * *` → Set Scheduled Context

`run_mode='MORNING'`, `send_telegram=true`

**ENGINE:**
1. Полный пересчёт (как MIDNIGHT)
2. Если !needsTherapist → `questMode='PLANNER'`
3. Если needsTherapist → `questMode=''`, `recommendedNextWorkflow='THERAPIST'`
4. Quest Needed? questMode='PLANNER' → YES → Execute Quest Architect (PLANNER mode)
5. QUEST строит план дня по текущим задачам + пишет в Daily Planner + GAME_LOG

**В GAME_LOG пишется:**
- `DAY_PLAN_UPDATED` от Quest Architect
- + любые ENGINE events (если случились)

6. Build Final Output: `engine.telegram_text + '\n\n' + quest.response_text`

**Юзер получает (Telegram):**
> Доброе утро.  
> Попытка: ACTIVE #1. Stage: EARLY.  
> XP_Gold: 340. XP_Total: 340.  
> TOTAL_PROGRESS: 0.34.  
> Commit: ON_TRACK.  
> Задачи: сегодня 3, overdue 1.  
>
> (план дня от Quest Architect)

---

## СЦЕНАРИЙ 16 — CRON: EVENING (17:00 BKK)

**Триггер:** Schedule cron `0 17 * * *` → Set Scheduled Context

`run_mode='EVENING'`, `send_telegram=true`

**ENGINE:**
1. Полный пересчёт
2. `questMode='EVENING_REVIEW'` (если !needsTherapist)
3. Execute Quest Architect (EVENING_REVIEW mode): агент смотрит что было сделано за день, итог

**Юзер получает (Telegram):**
> Вечерний чек.  
> [ENGINE stats]  
>
> (итог дня от Quest Architect: что выполнено, что нет, следующий шаг)

---

## СЦЕНАРИЙ 17 — CRON: SATURDAY (10:00 BKK)

**Триггер:** Schedule cron `0 10 * * 6` → Set Scheduled Context

`run_mode='SATURDAY'`, `send_telegram=true`

**ENGINE:**
1. Run Mode Switch → ветка SATURDAY (отдельная ветка!)
2. Generate Week Sheets → Duplicate Sheet × 8 (7 daily + 1 weekly)
3. Build Sheet Setup Writes → Write Sheet Setup Writes: настраивает заголовки листов
4. Saturday Report: строит сообщение о созданных листах

**В Google Sheets:** создаются 7 новых daily-листов и 1 weekly-лист для следующей недели

**Юзер получает (Telegram):**
> 📅 *Листы на неделю созданы!*  
> 📆 [дата воскресенье] – [следующая суббота]  
>
> ✅ Настроено: 8 листов  
>
> Daily и Weekly planners готовы.

---

## СЦЕНАРИЙ 18 — CRON: NOTIFIER — ежечасные напоминания

**Триггер:** `Every Hour` (каждый час, cron not specified explicitly — hourly schedule)

**NOTIF:**
1. Read Task List: читает Task List (native GS)
2. Find Due Soon: ищет задачи, у которых START_DATE попадает в окно [now+30мин, now+90мин]
3. Has Reminders? НЕТ → No Reminders — Skip. ДА →
4. Send Reminder: шлёт Telegram

**Юзер получает (Telegram):**
> ⏰ *Напоминание — через ~1 час:*
>
> 📌 [task_name]  
> 🕐 Начало в [start_date] (BKK)

5. Append GAME_LOG (`GAME_LOG!B:S`):
```
TS | TASK_REMINDER_SENT | NOTIFIER | TASK | {task_id or row_number} | {task_name} | ... | {today} | ...
```
6. Mark as Reminded: ставит в Task List отметку что напоминание отправлено (чтобы не дублировать)

---

## СЦЕНАРИЙ 19 — Attempt FAIL (детектируется ENGINE)

**Условие:** 5+ дней без любой игровой активности в GAME_LOG (по EVENT_TYPE), attempt ACTIVE

**ENGINE MIDNIGHT или MORNING:**
1. Compute Engine State: `silentDays >= failThresholdDays` И `!hasEventToday('ATTEMPT_FAILED')`
2. `failHappenedNow=true`

**В GAME_LOG пишется:**
- `ATTEMPT_FAILED` — попытка провалена
- `GOLD_LOST` с `RESOURCE_TYPE=XP_GOLD`, `DELTA=-penalty` — штраф XP_Gold по правилу fail_penalty

3. messageParts: "Провал attempt: [N] дней тишины. Сгорает XP_Gold по правилу."
4. После fail — следующая активность автоматически создаёт новый `ATTEMPT_STARTED`

**Юзер получает (MORNING Telegram):**
> ...  
> Провал attempt: 5 дней тишины. Сгорает XP_Gold по правилу.  
> ...

---

## СЦЕНАРИЙ 20 — STUCK_TIME (детектируется ENGINE)

**Условие:** attempt ACTIVE, молчание 25+ часов (но < failThreshold дней)

**ENGINE:**
1. `silentHoursSinceActivity >= 25` И `!hasEventToday('STUCK_TIME_DETECTED')`

**В GAME_LOG пишется:**
- `STUCK_TIME_DETECTED` — зафиксировано залипание
- `THERAPIST_REQUIRED` — если установлено правилом

messageParts: "STUCK по времени: [N]+ часов молчания. Нужна помощь."

**Юзер получает (если MORNING/EVENING):**
> ...  
> STUCK по времени: 26+ часов молчания. Нужна помощь.  
> ...

*(Это сигнал для пользователя зайти и поговорить с Терапевтом)*

---

## СЦЕНАРИЙ 21 — 5 провалов → обязательная терапия

**Условие:** ENGINE обнаружил 5+ ATTEMPT_FAILED в GAME_LOG

**ENGINE:**
1. `needsTherapist=true` → `recommendedNextWorkflow='THERAPIST'`
2. Вместо PLANNER/EVENING_REVIEW → ENGINE не запускает Quest Architect
3. В messageParts: "Нужен therapist / reframe после 5 fail."

Следующее сообщение юзера (или текущее MORNING) → при ответе юзера MAIN роутит в THER

**THER (режим five_failures_analysis):**
- Глубокий анализ паттерна провалов
- Записывает BELIEF_RECORDED в BELIEFS
- Записывает FIVE_FAILURES_ANALYZED в GAME_LOG

---

## СЦЕНАРИЙ 22 — Уровень готов

**Условие:** TOTAL_PROGRESS >= 1.0 И BALANCE_THB >= 1 000 000

**ENGINE:**
1. `levelReady=true`
2. messageParts: "Уровень готов к подтверждению."
3. В GAME_LOG пишется: `LEVEL_READY_REACHED`

**Юзер получает утром/вечером:**
> ... Уровень готов к подтверждению. ...

Юзер инициирует level up → THER запускает `level_up_interview` → после подтверждения пишет `LEVEL_UP_APPROVED` в GAME_LOG

---

## СЦЕНАРИЙ 23 — Неизвестный запрос

**Юзер пишет:** "кто такой Наполеон?" (ничего общего с игрой)

**MAIN:**
1. AI Router: `intent='unknown'`, `confidence<0.5`
2. Route Decision: ни одна ветка не сработала → `output='Не до конца поняла запрос. Это про привычку, задачу, деньги или нужен разбор состояния?'`

**Юзер получает:**
> Не до конца поняла запрос. Это про привычку, задачу, деньги или нужен разбор состояния?

---

## Сводная таблица: что куда пишется

| Действие | Воркфлоу | Таблица | Событие в GAME_LOG |
|---|---|---|---|
| /start | MAIN | — | — |
| Онбординг checkpoint | RULES | GAME_LOG!B:S | ONBOARDING_CHECKPOINT |
| Онбординг завершён | RULES | GAME_RULES!B:X + GAME_LOG!B:S | ONBOARDING_COMPLETED, ATTEMPT_STARTED, SPRINT_STARTED, RULE_CREATED |
| Привычка | QUEST | GAME_LOG!B:S | HABIT_DONE / HABIT_SKIPPED |
| Новая задача | QUEST | Task List + GAME_LOG!B:S | TASK_CREATED |
| Обновление статуса задачи | QUEST | Task List + Planner + GAME_LOG!B:S | TASK_STATUS_CHANGED |
| Планировщик дня | QUEST | Daily Planner + GAME_LOG!B:S | DAY_PLAN_UPDATED |
| Финансы (расход) | FIN | WALLET + GAME_LOG!B:S | GOLD_LOST |
| Финансы (доход) | FIN | WALLET + GAME_LOG!B:S | GOLD_GAINED |
| Терапия | THER | BELIEFS + GAME_RULES (опц) + GAME_LOG!B:S | THERAPY_DISCUSSION, BELIEF_RECORDED |
| Редактирование правил | RULES | GAME_RULES!B:X + GAME_LOG!B:S | RULE_UPDATED |
| ENGINE MIDNIGHT | ENGINE | GAME_LOG!B:S | CONSISTENCY_CHECKED, STAGE_CHANGED... |
| ENGINE MORNING | ENGINE + QUEST | GAME_LOG!B:S + Planner | DAY_PLAN_UPDATED + ENGINE events |
| ENGINE EVENING | ENGINE + QUEST | GAME_LOG!B:S + Planner | EVENING_REVIEW |
| ENGINE SATURDAY | ENGINE | Новые листы в GS | — |
| Напоминание о задаче | NOTIF | GAME_LOG!B:S + Task List | SERVICE_EFFECT_RECORDED / TASK_REMINDER_SENT |
| Attempt fail | ENGINE | GAME_LOG!B:S | ATTEMPT_FAILED + GOLD_LOST |
| Stuck time | ENGINE | GAME_LOG!B:S | STUCK_TIME_DETECTED |

---

## Найденные и исправленные баги

### ✅ Исправлено в этой сессии

1. **LK Render MarkdownV2 → Markdown** — сообщение использовало `**жирный**` (Markdown V1), но parse_mode был `MarkdownV2`. Исправлено: `parse_mode: "Markdown"` в `1.7. Sekretutka_LK_Render.json`

2. **shouldForceOnboarding не учитывал !hasRules** — если GAME_LOG содержал старый ONBOARDING_COMPLETED, но GAME_RULES был пустой, "давай" не запускал онбординг. Исправлено: `onboardingConfirmation && (!onboardingCompleted || !hasRules)` в `1. Sekretutka_main.json`

3. **Format Final Response брал $json (Sheets ответ)** — когда Call Game Rules возвращал 2 элемента (Sheets + reply), первым шёл Sheets без `reply_to_user` и format не находил текст. Исправлено: `_allInputItems.find(i => i.json.reply_to_user || i.json.output)` в `1. Sekretutka_main.json`

4. **/start вызывал Game Rules** — state в GAME_LOG продвигался, потом "давай" попадал на state=2 и не получал ответа. Исправлено: при `/start && !onboardingCompleted` routeTarget='none', текст отдаётся напрямую

5. **AI Extractor пустой системный промпт** — GPT-4o-mini не знал что делать. Исправлено: явный prompt в `1.0 Sekretutka_Game Rules.json`

6. **AI Extractor JSON fallbacks** — если GPT не смог распознать число/дату/текст, система зависала. Исправлены прямые fallbacks для states 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 13, 15, 17, 19, 21, 23, 24, 25, 26, 27, 28, 30, 31, 32, 33, 34, 35, 36, 37, 38

### ⚠️ Известные ограничения (не баги, фиче)

- **Notifier** — legacy, работает каждый час. Не интегрирован с ENGINE. По плану переносить логику в ENGINE
- **ENGINE не пишет TASK_REMINDER_SENT** — Notifier пишет сам в GAME_LOG, но EVENT_TYPE не унифицирован с реестром
- **LK Render chatId захардкожен** (`145771888`) — работает только для одного пользователя, что соответствует текущей архитектуре single-user
