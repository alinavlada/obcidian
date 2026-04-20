# КРИТИЧЕСКИЕ ПРОБЛЕМЫ — ИСПРАВИТЬ НЕМЕДЛЕННО

**Дата:** 2026-04-10  
**Статус:** 3 критических бага найдены и требуют немедленного исправления

---

## 🔴 НАЙДЕННЫЕ БАГИ

### 1. **THERAPIST_WORKFLOW_ID БАГ — ПОДТВЕРЖДЕНО**

**Локация:** `1. Sekretutka_main.json`, строка 672-680  
**Нода:** "Call '1.5 Sekretutka_Therapist'"

**Проблема:**
```json
{
  "name": "Call '1.5 Sekretutka_Therapist'",
  "parameters": {
    "workflowId": {
      "value": "xFmyWUFSK1CwQxCC54lRF",
      "cachedResultName": "1.4. Sekretutka_finance"  ← WRONG! Это FINANCE, не THERAPIST!
    }
  }
}
```

**Последствие:** Когда main маршрутизирует intent="therapist", вместо запуска терапевта запускается финансовый workflow.

**Риск:** 🔴 БЛОКИРУЕТ therapist route полностью

**Решение:**
1. Найти реальный workflowId терапевта в n8n
2. Заменить "xFmyWUFSK1CwQxCC54lRF" на реальный ID
3. Обновить cachedResultName на "1.5 Sekretutka_Therapist"
4. Пересохранить main.json

---

### 2. **STUCK ЛОГИКА — НЕПОЛНАЯ РЕАЛИЗАЦИЯ**

**Локация:** `1.1 Sekretutka_ENGINE_updated.json`, строки 597-620

**Проблема:**
Текущая реализация проверяет:
```javascript
stuckDetected = goalActionsCount > 0 && moneyNetWindow < stuckX;
```

Это проверяет: "есть ли действия по цели И денежный прогресс < X за Y дней"

Но документация (PRD) говорит о двух типах STUCK:
1. **STUCK по деньгам:** X дней без денежного прогресса (текущее)
2. **STUCK по времени:** 25 часов тишины (со стандартной целью / денежной целью) — **НЕ РЕАЛИЗОВАНО**

**Последствие:** Система не может обнаружить STUCK за 25 часов тишины

**Риск:** 🔴 БЛОКИРУЕТ часть STUCK механики

**Решение:**
1. Добавить параметр `STUCK_HOURS_THRESHOLD` в GAME_RULES (default 25)
2. В ENGINE добавить логику:
   ```javascript
   const silentHours = (daysBetween(lastActivityDate, today) * 24) + currentHour;
   if (silentHours >= STUCK_HOURS_THRESHOLD) {
     stuckDetected = true;
     // логировать STUCK_TRIGGERED
   }
   ```
3. Задокументировать оба типа STUCK в PRD

---

### 3. **EVENT_TYPE СЛОВАРЬ ОТСУТСТВУЕТ**

**Проблема:** ENGINE логирует события в GAME_LOG без явного словаря EVENT_TYPE

**Текущие события в ENGINE:**
- ATTEMPT_FAILED
- THERAPIST_REQUIRED
- ATTEMPT_STARTED
- STAGE_CHANGED
- STUCK_TRIGGERED
- LEVEL_READY_REACHED
- CONSISTENCY_CHECKED

**Минимальный требуемый словарь:**
- `ATTEMPT_STARTED`, `ATTEMPT_FAILED`, `STUCK_TIME_DETECTED`, `STUCK_TRIGGERED`
- `RULE_CREATED`, `RULE_UPDATED`, `COMMIT_LOG`, `ONBOARDING_COMPLETED`
- `HABIT_DONE`, `HABIT_SKIPPED`
- `BELIEF_RECORDED`, `HP_SPENT`
- `THERAPY_DISCUSSION`, `THERAPY_SESSION_COMPLETED`
- `LEVEL_UP_INTERVIEW_STARTED`, `LEVEL_UP_APPROVED`, `LEVEL_UP_HELD`
- `FIVE_FAILURES_ANALYZED`, `TARGET_REVIEWED`
- `STAGE_CHANGED`, `LEVEL_READY_REACHED`
- `THERAPIST_REQUIRED`, `THERAPY_REFRAME_STARTED`, `THERAPY_REFRAME_COMPLETED`
- `CONSISTENCY_CHECKED`

**Риск:** 🔴 Невозможно валидировать события, нарушается целостность логирования

**Решение:**
1. Создать лист `EVENT_TYPES` в GAME_RULES sheet (или отдельный реестр)
2. Структура:
   ```
   EVENT_TYPE | CATEGORY | DESCRIPTION | REQUIRED_FIELDS | OPTIONAL_FIELDS
   ATTEMPT_FAILED | ATTEMPT | Attempt failed after N silent days | attempt_id | xp_gold_penalty
   ...
   ```
3. Добавить валидацию в ENGINE и каждый workflow перед логированием

---

## 📋 ИТОГОВЫЙ ПОРЯДОК ИСПРАВЛЕНИЙ

### Немедленно (блокирует онбординг):
1. **Исправить THERAPIST_WORKFLOW_ID** в main.json
   - Оценка: 5 минут
   - Проверка: Вызвать therapist intent и убедиться, что маршрутизируется правильно

2. **Создать EVENT_TYPE реестр**
   - Оценка: 30 минут
   - Проверка: ENGINE логирует события, соответствующие реестру

3. **Добавить STUCK по времени (25 часов)** в ENGINE
   - Оценка: 1 час
   - Проверка: Имитировать 25 часов тишины, убедиться, что STUCK_TRIGGERED логируется

### Следующий этап (перед тестированием):
4. Проверить XP_TOTAL расчёт в ENGINE
5. Проверить goal decomposition запись в PROFILE_GAME
6. Code review main.json на полноту
7. Checkpoint архитектура онбординга

---

## ✅ ПРОВЕРОЧНЫЙ СПИСОК ДЛЯ КАЖДОГО ИСПРАВЛЕНИЯ

### Для THERAPIST_WORKFLOW_ID:
- [ ] Найден реальный workflowId терапевта
- [ ] Обновлен main.json
- [ ] Пересохранен в n8n
- [ ] Экспортирован в папку (для версиона контроля)
- [ ] Протестирован: therapist intent → therapist workflow

### Для EVENT_TYPE:
- [ ] Создан реестр (sheet или document)
- [ ] 13+ типов событий внесены
- [ ] Каждый тип имеет description и required fields
- [ ] ENGINE обновлен для валидации при логировании
- [ ] GAME_LOG имеет образцы всех типов

### Для STUCK по времени:
- [ ] Параметр STUCK_HOURS_THRESHOLD добавлен в GAME_RULES (default 25)
- [ ] ENGINE содержит логику проверки часов тишины
- [ ] STUCK_TRIGGERED логируется правильно
- [ ] Документация (PRD) обновлена с обоими типами STUCK
- [ ] Протестирована имитация 25 часов тишины
