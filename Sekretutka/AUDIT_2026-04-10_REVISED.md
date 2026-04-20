# СЕКРЕТУТКА — АУДИТ АРХИТЕКТУРЫ (ПЕРЕСМОТРЕННЫЙ)

**Дата:** 2026-04-10  
**Пересмотр:** После 19 исправлений от пользователя + Phase 1 Critical Fixes  
**Статус:** Система функциональна. ✅ 3 из 7 критических несостыковок ИСПРАВЛЕНЫ. Текущая работа — синхронизация документации + 4 оставшихся критических задачи.

---

## 📊 СВОДКА

| Уровень | Кол-во | Статус |
|---|---|---|
| 🔴 **КРИТИЧНЫЕ** | **7** | Блокируют реальный онбординг с новым игроком |
| 🟠 **ВЫСОКИЕ** | **8** | Влияют на корректность расчётов |
| 🟡 **СРЕДНИЕ** | **5** | Оптимизация и уточнение |
| 🔵 **НИЗКИЕ** | **2** | Техдолг и документация |

**Всего:** 22 проблемы

---

## ✅ ЧТО УЖЕ РАБОТАЕТ (и было ошибочно классифицировано)

### 1. **FSM онбординга — ✅ СУЩЕСТВУЕТ**
- Локация: `1.0 Sekretutka_Game Rules.json` (lines 101–108, Onboarding FSM Router code node)
- Структура: 39 состояний, state-based routing (не agent prompt)
- Правильно: Промежуточные состояния (PROMO → ... → CONTRACT_CONFIRM) живут в коде как switch-case

**Статус:** Рабочий, но нужна документация FSM-шагов

---

### 2. **STAGE_RULES лист — ❌ НЕ СУЩЕСТВУЕТ**
- Как оказалось, это legacy
- Все правила хранятся в GAME_RULES с TYPE column

**Действие:** Удалить все ссылки на STAGE_RULES из документов и кода

---

### 3. **RESOURCE_LOG & BOSS_LOG — ❌ НЕ СУЩЕСТВУЮТ**
- Это старые логи, которые удалены
- Новый источник: GAME_LOG

**Действие:** Удалить все ссылки из документов

---

### 4. **Quest Architect режимы — ✅ СУЩЕСТВУЮТ**
- Локация: `1.3 Quest_Architect_Daily_Planner.json`
- Режимы: TASK_CAPTURE, PLANNER, GOAL_DECOMPOSER, HABIT_LOG, EVENING_REVIEW

**Статус:** Нужна проверка, что они правильно вызываются из main

---

## 🔴 КРИТИЧЕСКИЕ НЕСОСТЫКОВКИ (РЕАЛЬНЫЕ)

### 1. **ATTEMPT LIFECYCLE: FAILED vs STUCK — ✅ ИСПРАВЛЕНО**

**Было ошибочно:** "5 дней тишины = FAILED"

**Правильно:**
- **FAILED**: После **5 дней подряд** нет успешно записанной активности
- **STUCK_TIME**: После **25 часов** молчания (alert, требует помощи, не меняет статус)
- **STUCK_TRIGGERED**: Денежный блок (X THB за Y дней без прогресса)

**✅ РЕШЕНО (2026-04-10):**
1. ✅ ENGINE обновлена с логикой STUCK_TIME (25-часовой порог)
2. ✅ Добавлена функция `hoursBetween()` для точного расчёта времени
3. ✅ GAME_LOG логирует оба события: `STUCK_TIME_DETECTED` и `STUCK_TRIGGERED`
4. ✅ README и PRD обновлены с информацией о STUCK_TIME

**Файлы:**
- `1.1 Sekretutka_ENGINE_updated.json` — STUCK_TIME detection (lines с использованием hoursBetween)
- `README.md` — добавлено объяснение STUCK_TIME правила
- `EVENT_TYPE_Registry.json` — задокументированы оба события STUCK

---

### 2. **THERAPIST_WORKFLOW_ID БАГ — ✅ ИСПРАВЛЕНО**

**Проблема:** В `1. Sekretutka_main.json` есть node "Call Therapist", который использует `$env.THERAPIST_WORKFLOW_ID`, но не ясно, установлена ли переменная окружения.

**✅ РЕШЕНО (2026-04-10):**
1. ✅ Проверено: THERAPIST_WORKFLOW_ID правильно установлен в main.json (line 580-584)
   - Workflow ID: `fxeZXplAm5MfdSGDdi338`
2. ✅ Quest_Architect workflow ID также верифицирован (line 789)
   - Workflow ID: `bVNR2xmUUo_m1_PB7382n`
3. ✅ Route `therapist` правильно маршрутизируется в main

**Файлы:**
- `1. Sekretutka_main.json` — THERAPIST_WORKFLOW_ID verified at lines 580-584

---

### 3. **EVENT_TYPE СЛОВАРЬ — ✅ ИСПРАВЛЕНО**

**Проблема:** PRD обещает полный канонический словарь EVENT_TYPE, но его нет ни в документах, ни в GAME_RULES.

**✅ РЕШЕНО (2026-04-10):**
1. ✅ Создан `EVENT_TYPE_Registry.json` с полным реестром
2. ✅ Реестр синхронизирован с фактическими событиями active workflows:
   - Rules-Layer: RULE_CREATED, RULE_UPDATED, COMMIT_LOG, ONBOARDING_COMPLETED
   - Engine-Layer: ATTEMPT_STARTED, ATTEMPT_FAILED, STUCK_TIME_DETECTED, STUCK_TRIGGERED, CONSISTENCY_CHECKED, LEVEL_READY_REACHED, STAGE_CHANGED, THERAPIST_REQUIRED
   - Quest Architect Layer: HABIT_DONE, HABIT_SKIPPED
   - Therapist-Layer: BELIEF_RECORDED, HP_SPENT, THERAPY_DISCUSSION, THERAPY_SESSION_COMPLETED, LEVEL_UP_INTERVIEW_STARTED, LEVEL_UP_APPROVED, LEVEL_UP_HELD, FIVE_FAILURES_ANALYZED, TARGET_REVIEWED, THERAPY_REFRAME_STARTED, THERAPY_REFRAME_COMPLETED
3. ✅ EVENT_VALIDATION_RULES обновлены под актуальный active-set
4. ✅ LAYER_MAPPING документирует текущую структуру по слоям

**Файлы:**
- `EVENT_TYPE_Registry.json` — Реестр, синхронизированный с active workflows (v2.0, status: active)

---

### 4. **XP_TOTAL РАСЧЁТ — ТРЕБУЕТ ПРОВЕРКИ**

**Формула из PRD:**
```
XP_TOTAL = cumulative(0.5 × XP_Gold gains + 0.5 × HP-positive gains)
```

**Проблема:** Неясно, считается ли это правильно в ENGINE. Low-Token Redesign говорит ENGINE может быть запутан с XP.

**Последствие:** Stage будет неправильно вычисляться (stage зависит от STAGE_XP)

**Действие:**
1. Проверить ENGINE: где и как считается XP_TOTAL?
2. Убедиться, что он NOT просто сумма XP_Gold
3. Добавить логирование всех XP-событий в GAME_LOG

---

### 5. **GOAL DECOMPOSITION: ПРОВЕРКА ЗАПИСИ В ЯЧЕЙКИ**

**Определение из PRD:**
- Action steps: колонки `G, L, Q, V, AA, AF, AK, AP, AU, AZ` (строки 17-35)
- Deadline: строка 37
- Reward: строка 38

**Проблема:** Неясно, пишет ли туда Quest Architect режим GOAL_DECOMPOSER?

**Действие:**
1. Проверить `1.3 Quest_Architect_Daily_Planner.json`
2. Убедиться, что GOAL_DECOMPOSER режим пишет именно в эти ячейки
3. Добавить валидацию на макс 10 шагов
4. Протестировать в sheets

---

### 6. **CHECKPOINT ОНБОРДИНГА: ГДЕ ХРАНИТСЯ?**

**Требование из PRD (раздел 6.2):**
- Сохранять checkpoint текущего state (например, HABIT_1)
- Сохранять промежуточные данные (draft_json)
- Не сбрасывать в начало после каждого сообщения
- Передавать session_id между сообщениями

**Проблема:** Неясно, как это реализовано:
- Является ли checkpoint отдельной таблицей?
- Или это EVENT_TYPE в GAME_LOG?
- Как session_id (Telegram chat_id) передается?

**Действие:**
1. Уточнить архитектуру: ONBOARDING_CHECKPOINTS лист или GAME_LOG?
2. Задокументировать структуру checkpoint JSON
3. Убедиться, что main передает session_id

---

### 7. **ATTEMPT FAIL LOGIC: РЕАЛИЗАЦИЯ**

**Требование:** Attempt FAILED если 5 дней подряд нет успешной активности

**Проблема:**
- Кто проверяет? (ENGINE?)
- Откуда берется информация? (GAME_LOG?)
- Как считаются "дни подряд"? (от последнего события?)

**Действие:**
1. Проверить ENGINE: есть ли логика?
2. Если нет, реализовать:
   - Последнее успешное событие из GAME_LOG
   - Дни = TODAY - DATE_OF_LAST_EVENT
   - Если дни >= 5: ATTEMPT_STATUS = FAILED
   - Сбросить XP_GOLD, сохранить XP_TOTAL
   - Логировать в GAME_LOG

---

## 🟠 ВЫСОКИЕ НЕСОСТЫКОВКИ

### 8. **MAIN.JSON ВЕРИФИКАЦИЯ**

**Статус:** Пользователь говорит "should be correct", но нужна полная проверка

**Что проверить:**
1. Все read nodes (PROFILE_NODE, RULES_NODE, SCHEDULE_NODE, VOICE_TEXT_NODE) читают правильные данные?
2. Normalization функции (parseBoolean, parseNumber) работают правильно?
3. Intent routing — все 11 intents маршрутизируются правильно?
4. State override logic (THERAPIST/PLANNER lock) реализована?

**Действие:** Провести детальный code review main.json

---

### 9. **STAGE CALCULATION: BASELINE**

**Требование из PRD:**
```
STAGE_XP = XP_TOTAL - XP_AT_ATTEMPT_START
```

**Проблема:**
- Где хранится XP_AT_ATTEMPT_START?
- Когда он пересчитывается?
- ENGINE это читает правильно?

**Действие:**
1. Убедиться, что при старте attempt снимается снимок:
   - `XP_AT_ATTEMPT_START = XP_TOTAL`
   - `LEVEL_START_TS = NOW`
2. Эти значения в PROFILE_GAME snapshot keys?
3. ENGINE считает: `STAGE_XP = XP_TOTAL - XP_AT_ATTEMPT_START`

---

### 10. **LOW HP RULES: ПОЛНАЯ ЛОГИКА**

**Требование:** Три режима:
- NORMAL (HP >= 100)
- RECOVERY_ACTIVE (HP < 60)
- RECOVERY_PASSIVE (HP < 30)

**Проблема:** Какие действия блокируются/разблокируются в каждом режиме?

**Действие:** Задокументировать таблицу action restrictions по режимам

---

### 11. **"2 ИЗ 3" ЭНЕРГЕТИЧЕСКОЕ ПРАВИЛО**

**Требование из PRD (раздел 4.12):**
> "2 из 3 (сон / еда / вода) = день засчитан"

**Проблема:**
- Кто это проверяет?
- Как система узнает, спал ли игрок / ел ли / пил ли?
- Это привычки, которые логируются?
- Это входит в онбординг как настраиваемое правило?

**Действие:**
1. Уточнить: это 3 привычки или 3 категории поведения?
2. Где эта логика должна быть: в ENGINE, в therapist, или в quest architect?
3. Как система валидирует, что игрок действительно спал/ел/пил?

---

### 12. **STUCK DEFINITION: ПАРАМЕТРЫ**

**Требование:** STUCK возникает при отсутствии денежного прогресса X за Y дней

**Проблема:**
- Какие X и Y?
- Где они хранятся? (GAME_RULES?)
- Кто это проверяет? (ENGINE?)

**Действие:**
1. В онбординге (раздел STUCK_RULE) собрать параметры:
   - `STUCK_MONEY_THRESHOLD = X` (например, 5000 THB)
   - `STUCK_DAYS_WINDOW = Y` (например, 14 дней)
2. Сохранить в GAME_RULES
3. ENGINE проверяет при CRON

---

### 13. **LEVEL UP: ТРЕХМЕСЯЧНАЯ ФИНАНСОВАЯ ПРОВЕРКА**

**Требование:** Level up требует:
- `Money Net >= X` в месяц (минимум 3 месяца подряд)

**Проблема:**
- Где это проверяется?
- Что такое "Money Net"? (Доход - Расход?)
- Как система отслеживает "3 месяца подряд"?

**Действие:**
1. Четко задокументировать формулу Money Net
2. Где эта логика: ручная или автоматическая в workflow?
3. Если автоматическая, то в каком workflow?

---

### 14. **LEVEL NAMES: ФИКСИРОВАННО 4 ИЛИ "КАК МИНИМУМ 4"?**

**Требование:** "в онбординге игрок задает имена как минимум для 4 уровней"

**Проблема:** Система работает с фиксированно 4 уровнями или более?

**Действие:** Уточнить в онбординге: жесткое ограничение 4 уровня или гибкое?

---

### 15. **PROFILE vs PROFILE_GAME: ПОЛНАЯ АУДИТ СТАРЫХ ССЫЛОК**

**Требование:** Все старые ссылки удалены

**Проблема:** Нужна полная grep-аудит всех json-файлов:
- "PROFILE" (должно быть PROFILE_GAME)
- "GAME_STAGE" (должно быть PLAYER_STAGE)
- "LEVEL_ID" (должны быть LEVEL_NAME + LEVEL_NUMBER)

**Действие:** Провести полную аудит и обновить все старые ссылки

---

## 🟡 СРЕДНИЕ НЕСОСТЫКОВКИ

### 16. **DOCUMENTATION SYNC: README vs ACTUAL CODE**

**Проблема:** README упоминает ENGINE_v4.json, но в папке ENGINE_updated.json

**Действие:** Обновить README с правильными названиями файлов

---

### 17. **MULTI-CURRENCY HANDLING**

**Проблема:** README говорит о BALANCE_THB, BALANCE_RUB, BALANCE_USD, но как level readiness считается в multi-currency?

**Действие:** Уточнить в PRD или finance workflow

---

### 18. **BELIEF → XP MAPPING**

**Формула TOTAL_PROGRESS:**
> "+ 0.1×(Beliefs_From_Level_Start/3)"

**Проблема:** Как beliefs переводятся в прогресс? Это просто счетчик beliefs?

**Действие:** Уточнить расчёт в ENGINE

---

### 19. **RECOVERY MODES: HP_MAX НАСТРАИВАЕТСЯ ЛИ?**

**Проблема:** PRD говорит HP_MAX может быть переопределено, но это не задокументировано в онбординге

**Действие:** Добавить раздел "HP and Recovery Settings" в онбординг FSM

---

## 🔵 НИЗКИЕ НЕСОСТЫКОВКИ

### 20. **"АРИАДНА" УПОМИНАНИЯ**

**Статус:** Удалена, но может быть ещё ссылки в документах

**Действие:** Grep на "Ариадна" и удалить

---

### 21. **TIMEZONE & CREDENTIALS CHECKLIST**

**Проблема:** README не содержит инструкцию: как новый разработчик настроит n8n с нуля?

**Действие:** Создать SETUP.md с пошаговыми инструкциями

---

## 📋 МАТРИЦА ЗАВИСИМОСТЕЙ (ИСПРАВЛЕННАЯ)

```
[Telegram message from user]
  ↓
[main router]
  ├─ Read snapshot (6-8 fields from PROFILE_GAME)
  ├─ AI Intent Classifier (JSON only)
  ├─ Code Router
  └─ Target workflow based on intent
       ├─ log_habit → Quest Architect (HABIT_LOG)
       ├─ add_task → Quest Architect (TASK_CAPTURE)
       ├─ plan_day → Quest Architect (PLANNER)
       ├─ decompose_goal → Quest Architect (GOAL_DECOMPOSER)
       ├─ therapist → therapist workflow
       ├─ rules_onboarding / rules_edit → game_rules (FSM)
       ├─ log_finance → finance workflow
       └─ read_status → temporary direct response in main
       └─ read_status → lk_render

[ENGINE - CRON every hour]
  ├─ Check 5-day silence → ATTEMPT_FAILED
  ├─ Check 25-hour silence + context → STUCK_TIME_DETECTED
  ├─ Calculate STAGE_XP = XP_TOTAL - XP_AT_ATTEMPT_START
  ├─ Check LEVEL_READY conditions
  └─ Write events to GAME_LOG

[All writes to sheets]
  ├─ PROFILE_GAME (main snapshot)
  ├─ GAME_LOG (all events)
  ├─ GAME_RULES (rules and parameters)
  ├─ Task List (tasks)
  ├─ BELIEFS (beliefs from therapist)
  └─ FINANCE (income/expense tracking)
```

---

## ✅ ПЛАН ИСПРАВЛЕНИЙ (ПЕРЕСМОТРЕННЫЙ)

### Фаза 1: КРИТИЧЕСКИЕ (блокируют онбординг)
1. **Проверить ATTEMPT FAIL логику:** 5 дней vs 25 часов STUCK
2. **Исправить THERAPIST_WORKFLOW_ID баг**
3. **Создать EVENT_TYPE реестр** (13+ типов событий)
4. **Проверить XP_TOTAL расчёт** в ENGINE
5. **Убедиться Goal Decomposition пишет в правильные ячейки**
6. **Уточнить checkpoint архитектуру** (отдельная таблица или GAME_LOG?)
7. **Реализовать ATTEMPT_FAILED логику** (если её нет)

### Фаза 2: ВЫСОКИЕ (проверка корректности)
8. **Code review main.json** на полноту
9. **Проверить STAGE calculation** baseline
10. **Документировать HP recovery modes** и restrictions
11. **Уточнить "2 из 3" энергетическое правило**
12. **Задокументировать STUCK параметры** (X, Y)
13. **Уточнить level up финансовую проверку**
14. **Уточнить level names** (4 или больше?)
15. **Grep audit PROFILE → PROFILE_GAME** и старые ссылки

### Фаза 3: СРЕДНИЕ (оптимизация)
16. **Обновить README** (ENGINE_updated вместо v4)
17. **Уточнить multi-currency** logic
18. **Уточнить belief → XP** mapping
19. **Задокументировать HP_MAX** переопределение в онбординге

### Фаза 4: НИЗКИЕ (техдолг)
20. **Удалить "Ариадну"** из документов
21. **Создать SETUP.md** для новых разработчиков

---

## 🎯 КРИТЕРИЙ ГОТОВНОСТИ К РЕАЛЬНОМУ ОНБОРДИНГУ

Система готова, когда:
1. ✅ Все 7 критических исправлены и протестированы
2. ✅ EVENT_TYPE реестр создан и валидируется
3. ✅ ENGINE pass e2e тест (5-day fail, 25-hour stuck, stage calculation)
4. ✅ Онбординг FSM протестирован (все 39 шагов)
5. ✅ Goal decomposition протестирована (правильные ячейки, макс 10 шагов)
6. ✅ Checkpoint логика протестирована (continuation работает)
7. ✅ main.json pass полный code review
8. ✅ SETUP.md создана для новых разработчиков

---

## 📝 СЛЕДУЮЩИЙ НЕМЕДЛЕННЫЙ ШАГ

**Рекомендация:** Начать с Фазы 1 по приоритету:

1. **Проверить ENGINE логику** (5 дней fail, 25 часов stuck)
2. **Исправить Therapist ID bug**
3. **Создать EVENT_TYPE реестр**

Каждое исправление должно быть:
- Реализовано в коде (n8n)
- Отражено в документации
- Протестировано
