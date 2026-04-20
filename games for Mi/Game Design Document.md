# **GAME DESIGN DOCUMENT (GDD)**

## **Проект: «Trikster - The World of Place Values**

**Жанр:** Educational Logic Puzzle / Образовательная головоломка

**Платформы:** iOS, Android, Web (Планшеты и смартфоны)

**Целевая аудитория:** Дети 5–10 лет

**Движок:** Flutter (Flame) / Unity 2D

## **1\. PRODUCT VISION (Краткая концепция)**

Игра переосмысляет обучение математике для детей. Вместо заучивания абстрактных правил и страха перед красным крестиком («ОШИБКА»), игра предлагает физическую песочницу. Цифры — это осязаемые объекты со своими физическими законами. Ребенок формулирует гипотезы, строит числовые «мостики», проверяет баланс обратным действием и выявляет паттерны. Финальная цель — подготовить мозг ребенка к алгебре (уравнения, переменные, скобки) через наглядные игровые механики.

## **2\. ПЕРСОНАЖИ И ЛОР (Characters & Lore)**

Действие происходит в «Мире разрядов», который тесно переплетается с реальными бытовыми историями (музыка, готовка, спорт). Математические символы здесь — живые существа.

### **2.1. Единички (Ones)**

* **Визуал:** Маленькие, подвижные, юркие существа-дроны (Глубокий черный цвет / Dark Charcoal). У них светящиеся глазки и тонкие ручки-манипуляторы.
* **Роль:** Чернорабочие игры (дроны-штурманы). Они перетаскиваются пальцем, со звонком ("Цок!") заполняют ячейки и стыкуются друг с другом с помощью магнитных пазов.  
* **Закон физики:** На их «уровне» есть только 10 слотов (ячеек). Если приходит 11-я Единичка, происходит обязательный физический процесс схлопывания — 10 Единичек сливаются в одну Десятку и улетают на уровень выше.

### **2.2. Десятки (Tens)**

* **Визуал:** Крупные, массивные бронированные роботы (Мехи-экзоскелеты) глубокого синего цвета. Двигаются тяжело и основательно.
* **Роль:** Демонстрация мощной структуры.
* **Правило Замкнутой цепи:** Десяток обладает слишком мощным реактором для самостоятельной работы. Ему **всегда** нужен напарник для замыкания цепи. Мех должен держать за руку либо Единичку-дрона (если число 11-19), либо Нолик-стабилизатор (если выведено ровно 10, 20 и т.д.). При разрыве этой цепи (например, при вычитании единички) броня теряет энергию (блекнет) и десяток может рассыпаться обратно на 10 маленьких дронов.

### **2.3. Нолик (Zero)**

* **Визуал:** Полупрозрачный круг или контур. Светящийся.  
* **Роль:** Хранитель баланса с «биполярным» поведением.  
  * *Состояние 1 (Пустота):* Стоит в пустом разряде. Реплика: «Я просто держу место, меня тут нет».  
  * *Состояние 2 (Сила):* Когда соединяется с Десяткой. Реплика: «Мы вместе — это мощь\!».  
  * *Механика:* Появляется в интерфейсе, когда нужно проверить равенство или нажать кнопку **Undo (Обратимость)**. Радуется, когда после проверки пример возвращается в начальную точку.

### **2.4. Система / Трикстер (System)**

* **Визуал:** Невидимый голос или абстрактный интерфейсный элемент (Око/Радар).  
* **Роль:** Ведет игрока по уроку, подсвечивает паттерны. Но в конце уровня (фаза Boss Fight) намеренно выдает логическую ошибку, выдавая её за правило, чтобы игрок её опроверг.

## **3\. CORE GAMEPLAY LOOP (Основной цикл игры)**

Каждое задание (или серия заданий внутри уровня) проходит через 6 строгих фаз. Интерфейс всегда дублирует термины на двух языках (RU/EN).

1. **Prediction (Предсказание):** Игра задает контекст (например, перекладываем камни). Игрок должен выбрать гипотезу («Понадобится много или мало?», «Влезет в 10 или нет?»).  
2. **Counting On (Достройка / Вычисление):** Игрок физически решает задачу. Например, перетаскивает Единички, чтобы заполнить пустые слоты от 7 до 13\.  
3. **Inversibility (Обратимость):** Появляется Нолик. Игрок нажимает кнопку Undo. Игра проигрывает действие в обратном порядке (![][image1]), доказывая равенство.  
4. **Comparison (Сравнение):** На экране появляется похожий пример (был ![][image2], стал ![][image3]). Игрок решает его.  
5. **Pattern (Паттерн):** Система подсвечивает одинаковые элементы («хвостики») в двух примерах. Десятки объясняют закономерность.  
6. **System Glitch (Каверза):** (Только в конце уровня). Система предлагает ложный паттерн. Игрок должен нажать Undo, чтобы доказать Системе её неправоту, и только после этого открывается переход на следующий уровень.

## **4\. КЛЮЧЕВЫЕ ИГРОВЫЕ МЕХАНИКИ (Core Mechanics)**

### **4.1. Механика Абакуса (The Grid)**

* **Основа:** Экран разделен на зоны (Разряды). Зона Единиц содержит ровно 10 слотов-лунок.  
* **Взаимодействие:** Игрок может перетаскивать (Drag & Drop) Единички в лунки или свайпать, чтобы добавить сразу несколько.  
* **Переполнение (Bridging 10):** Если слотов не хватает, включается анимация: 10 заполненных лунок вспыхивают, сливаются в один Синий Блок (Десятку) и перемещаются в левую зону (зону Десяток). Оставшиеся Единички падают в пустые лунки.

### **4.2. Механика «Мостика» (Counting On)**

* **Описание:** Основной метод вычитания и поиска неизвестного (![][image4]).  
* **Как работает:** Вместо того чтобы «отнимать» объекты, игра показывает начальную точку (например, заполнено 7 лунок) и призрачный контур целевой точки (отметка на 13-й лунке). Игрок должен пальцем добавлять Единички, пока не дойдет до контура. Счетчик на экране считает *добавленные* Единички (это и есть ответ).

### **4.3. Механика Скобок (Grouping / Brackets)**

* **Визуал:** Эластичный прозрачный мешок (или рюкзак), внутри которого сидят числа/Единички.  
* **Как работает:** \* Объекты внутри мешка подсвечены.  
  * Объекты снаружи мешка затемнены и неактивны (не реагируют на тапы).  
  * Игрок должен тапнуть на мешок или перетащить Единички внутри него друг на друга. Когда пример внутри решен, мешок лопается с приятным звуком, оставляя одну цифру. Только после этого разблокируются остальные элементы примера.  
  * *Защита от ошибки:* Попытка взаимодействовать с внешними числами до решения скобок вызывает покачивание мешка (напоминание).

### **4.4. Механика Присвоения (Assignment Board / Variables)**

* **Визуал:** В верхнем углу экрана появляется отдельная физическая табличка — «Доска Договоренностей». На ней написано: \[Иконка Звезды\] \= 5\.  
* **Как работает:** \* В главном примере появляется Звезда (![][image5]).  
  * Игрок не может сложить Звезду и 3\. Он должен пальцем перетащить цифру 5 с Доски Договоренностей на Звезду. Звезда превращается в 5\. Затем пример решается как обычно.  
  * На более высоких уровнях Доска меняется прямо в процессе уровня, заставляя игрока пересчитывать пример с новой переменной.

## **5\. UI & UX (Интерфейс и Опыт Игрока)**

### **5.1. Правила обратной связи (Feedback Rules)**

* **Запрещено:** Использовать красный цвет для ошибок, иконки крестиков, слова «Error», «Wrong», «Try Again».  
* **Разрешено:** \* *При ошибке в подсчетах:* Элементы просто не стыкуются (как магниты с одинаковым полюсом). Появляется надпись: **Didn't match / Не совпало**.  
  * *При успехе:* Мягкое свечение, звук колокольчика, надпись: **It matches\! / Совпало\!**.

### **5.2. Расположение элементов (HUD)**

* **Верхняя панель:** Текстовая подсказка/реплика Системы. Справа — Доска Договоренностей (если активна).  
* **Центр экрана:** Игровое поле. Полоса Абакуса или числовая прямая. Здесь происходит весь Drag & Drop.  
* **Низ экрана (Контролы):** \* Крупная кнопка **Undo / Обратимость** (с иконкой зацикленной стрелки и Нолика).  
  * Тумблеры выбора при предсказании (Prediction).

### **5.3. Двуязычность**

Все ключевые термины в UI отображаются в формате EN / RU (например, на кнопке написано **Prediction / Предсказание**). Реплики персонажей озвучиваются на выбранном языке, но в текстовых бабблах (bubbles) могут иметь перевод по тапу.

## **6\. ПРОГРЕССИЯ И ДИНАМИКА УРОВНЕЙ (Level Design Overview)**

### **6.1. Динамический Гринд и Полоса Прогресса**
Уровни типа "Стандарт" не статичны. При запуске уровня игра генерирует серию примеров на выбранное правило.
* Игрок заполняет «Полосу энергии» (progress bar).
* Решение примера без ошибок с первой попытки даёт больше энергии (+30%), а решение с ошибками — меньше (+10%). 
* Это защищает от бессмысленного "прокликивания" и заставляет систему давать больше примеров, если паттерн не усвоен.

### **6.2. Режим «Квиз» / Распечатка (Printable Worksheets)**
Для родителей и учителей доступна кнопка **«Распечатать рабочую тетрадь» (Print Worksheet)**. 
* **Как работает:** По выбранной теме игра генерирует лист А4 в формате PDF со случайными примерами. Ребёнок может решить их на бумаге для тренировки моторики.
* **Проверка:** После решения на бумаге, ответы можно вбить в игру через специальный режим "Проверка тетради". Система наглядно (через Абакус и роботов) покажет, правильный ответ или нет.

### **6.3. Глобальная структура миров**
Игра разбита на 9 больших блоков (Worlds). В каждом блоке около 10 уровней.

1. **Блок 1: Основы (Number Sense).** Знакомство с Единичками и Абакусом. Сложение и вычитание уже известных чисел от 1 до 10.
2. **Блок 2: Переходы (Place Value).** Появление Десяток. Работа с числами от 10 до 1000 через сложение и вычитание. Изучение визуальных паттернов при переходе через разряд.
3. **Блок 3: Пропавшая часть (Энергетические воронки).** Мягкое введение в алгебру на отрезке от 10 до 1000. Ввод переменной `X` вместо чисел.
4. **Блок 4: Коробки-скобки (Силовые щиты).** Визуализация приоритета операций. Защитные пузыри, которые нужно "лопать", прежде чем складывать внешние цифры.
5. **Блок 5: Лаборатория Хаоса (Умножение и деление до 100).** Изучение связи сложения и умножения через клонирование/деление элементов.
6. **Блок 6: Большой Хаос (Умножение и деление до 1000).** Усложнение расчетов. Тренировка понимания больших паттернов и быстрой разбивки больших скоплений.
7. **Блок 7: Дроби (Fractions).** Введение концепции частей целого на базе деления. От простых 1/2 и 1/3 к составным 2/3, 4/8 и 4/9.
8. **Блок 8: Договоренности (Variables).** Работа с Доской Договоренностей. Подстановка значений вместо иконок/символов.  
9. **Блок 9: Формальная запись (Formal Notation).** Настоящая магия алгебры. Вывод абстрактных уравнений `X + Y = Z` через физическую механику сборки.

## **7\. АУДИО И МУЗЫКА (Audio Design)**

* **Музыка:** Lo-Fi, спокойные эмбиенты. Никакой раздражающей, быстрой "цирковой" музыки. Музыка должна помогать фокусироваться (состояние потока).  
* **SFX (Звуки):** \* Перетаскивание Единичек: глухой, приятный деревянный/пластиковый стук (как косточки настольной игры или кубики Lego).  
  * Схлопывание десятков: приятный звук всасывания (вакуум) и звонкий "поп\!".  
  * Кнопка Undo: Звук перемотки пленки (очень короткий и мягкий).  
  * Ошибки Системы: звук "заедающей" пластинки или легкий цифровой глитч.

**Конец документа.**

[image1]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAM4AAAAXCAYAAABDLnAjAAAFJklEQVR4Xu2aS4gcVRSGM45K8IEbJyPjTJ+entGB8YE6IrhKBBF8koWKEJQsNE8foMuAgqI7NyK4caMoCG58RFHRoPgEUTERDUgWkqCGYDToGI2O8f+7bzGXv+txq6erCuP94NBd554699Q559are9WqSCQSiUQikUgkEjmBMbP3IcdVn4CxvZC/aQPZ3+l0WmpTNW7uPa1W6w18vtput1+BvExR27pAHO+6uP5kXnS8ScbGxs5ATL+4+Fi759WmSjDfEZeXXZDXXM269UINH1T7ukAcVzEnqk+w5V7v1rSv1+fm5s7EwN2QPXSU5Qz6bTjQFxYWFk7BQd+G7aOQw2pXJZj3nCTGNFH7OkBO7sXci/zE5ii+fzE5OXme2jUF4vkSsb05PT19DeI6F9tLalMlWiNf2E9qXyWD9Pr4+PjpltbrPCNNTEyc7XbIc8axQ8n21NTU5e7g7/PtqgRz3YU5n4Pcz4PDQtpCwfd3MHaz2lcNm1Hz5XLylK9rCsTyAeTrZNud8I4j7nHfLgTs953qQnB9swM5uQfzb/VqltpnVSK9/mFWDC7m8F53O2Q5+wvybbINJ7c4+22+XZVgrp2qQyJOQyF2q74OXDIf8nWI5RJ/exhgnp9UF8BJjI/N4iuhu9LfDsUGXzgvpui+QZ5Wq75OLH/hlOt1N5jqTIHdJ6G2VYHkr28qhtnZ2THOjbP3FRR8fwDxrFO7YcArG4rXUX0eiOeJJDcsPOSxsj58bMCFo7BmkI2qrxvLWThKYa+HLhwc+GbaoRDv6VidIIZDkF2qrwPMez1zgMv4jPUeIt+GLCEnN6rtEBgte/uHWD5lfKjVWnwegOzkNq/QahuCDWHhzMzMrGHNVN8EVm7h5Pc6DfKcYew6yM/O7nUdrxM07IWIYS++juiY0ul0zoft79ZrcBVelo9B/rDeQyDtFllk9eMDmx3Wy8OvoqfuMl83DLhwTB9Qc3DHxFgeT3Rt93LFtwvFhrBwXDysWQh80aK18mvGt11+zX5TB3lYwcKx5V5fsqJep6M8Zz6w20BbvqnRMR8+iLJgIaL75oG5j6GZrlZ9XZhbOIh7veiZw6IijuqxhwhOFhMsJE8E6lBhftJq6eJ7RPUJ8H2WzkvBPgdUR9H9s8CJaIpzN1kzHytYOD5W1OsuqUHOSID9CJK7FrIuRHTnHEYK5q0cNPFNjIGN5usDctJ9LarHHiqu+W5QnwrsPk+Lw8W3qPoEnOgu1jndvAdVR5mfnz9VfaSB/Q/bYC85KsFKLBySW9esQSTTqOdDqq/Psq8aNM6mMvPyxyvYf2a99/dB0veDVz/dt1ZtOetSVya2UHDMd8DvUdVnAdvtaXG4+Papvghb4a0a58UxvKT6HHir1leXPFEHeVjGwhmo17MGbfl+/lnRp9pXDZr1mTLz8oEY9g9DHg0V7DaqfhTGgMv3RaqzCn5kpE8c92bVZ+F+7OzLkYvvadUXYUNYOCb9U4RfjxDR/fOwjIVjg/R61qC7P/0HX09OdGwYZ3+nZ1oL1nsw7IuzbtDIbyGOr3wd48KZ9QJft1L4TwT4vVT1RVjvQXpDsg0/s4PmzYazcJ5UfVNYxsIp0+u8JPK/TD9Cvndy0OQ+GM1wq9u5e+9Mge5236YuMPdHnF/1TWC9X+fZoB+7vFyrNisFPo+oLhA+C7K2+5O66Q+iodgQFg5ONFtVXzN+r/9gAb2OmHfze1O9HvmPs9KFE4n8L2nJX4sikUgkEolEIpFIJHJi8C+6dkMao/TNdgAAAABJRU5ErkJggg==>

[image2]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADUAAAAXCAYAAACrggdNAAABuElEQVR4Xu2UPUvDUBSGHeqmm0WpTT+HugnGycnBwUFwVBEHXZSioJOD/8DVH+Cm4uYHuLi6CDrp0FVELQgKKmhL0efKTbg9adoUTBfzwEtuznlzPuilXV0RERGhkk6nL9C3jDuQK6EqqqRSqbtkMjkoPWFCT1vNl8lkLnmeoVPOx+hIyTUWCoVezGsYbtUHfksRX0X7tm13x+PxHs6f6Fn6woTB55wZG8k1qqUSiUSfOnuSBjpXNt7HVIxGy6YvTOi3h7bpucGzyHNFifNjPp+3pP+XFkupa1cy3ue1f8n0hQm93mUsm80OE9+RcZdmS0nwXQf1hgkzvMlYHW0upbznMt5JuHqT/B9syngdrZaiwBT5F1RDJzLfSVhovdmsLq2WMqHoovJyp/tlzgTfQFDJb5uhZ/2ScQ/tLKUI4mfY8aCS3/phWdao6svNOZA5D35D0jCji0yYcT9/2DDOoe5dlDkPfkMS29JL7Yp4Q3/Y0PNez7Mgcx78hszlciniNY4xJ8b7SODCf4wzJ7/YtMw5xDC8oif0oFVGH6aJAjO62BW60UVnTU+noHcFVek/JHMRERER/5MfLZSl7qwYuvYAAAAASUVORK5CYII=>

[image3]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADUAAAAXCAYAAACrggdNAAAB+0lEQVR4Xu2WzStEURjGR1GUpUFjvqfEShk2SllYSAorJAs2NFmwslA+tlbKysoOycZH2fgLFCuUraakxIIFM038zp1zde7bmJnNnVm4Tz3dc573Oed9z8e9Mz6fBw8eXEE0Gh2LRCLfNulvS48CsQeYhZlwOPwYDAbbpMdNkDOp67vieQHPaZ/CE0WHWS9mHKbgve53Cs8iPEgmk3V+v7+R9id8MT1ug8KndG0F+Wukkw6FQr3GWKVlHKa8pgY+G/0+pZFo3vS5CfLtwy1yLvNM8VxQpP2USCRCptG5Sp+1I5tKi8fj7YZPXbsHoz+tx87Zmtsg14fUYrFYF/qOQwwEAk2cVL+pcXf3VMEsrt7UTRC/kZtRDVDDu9QKAuNdqYL1KV1KvZJg04c4gBWpF0KNPqVdGWCCEWJvMAfPZLySoL6lUhtvQb1DGF/hsYxJMOmsmpQ73SJjJvC1lks5thj0TfmSugP85jToE9qQsb+gJy66W8w3UC7l2L/AN6BH5eXmHMqYA5gyoj/MZ7JZtUkY1ZMMCk/JRbkByjnSuVMy9guKXVeLENqa3Sa2qhe1Z3qqtShypnU9MzJmQb/8VnGStod3LUw/R7PW1uh3F53YRdj1cWKjMmZBLsSk6WOCCa1fw1vVRps0PZVCJP+PJ0v+Dhnz4MGDh/+JH6tOtUcsMIwEAAAAAElFTkSuQmCC>

[image4]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABIAAAAYCAYAAAD3Va0xAAABFklEQVR4XmNgGDlASUlJTkpKSkROTk4QRCsoKEgYGxuzguSUlZXF1NXVecXFxbllZGQ4QXx0/WAA0gQ0IFdeXn4pEP8H4r9AsQKg4fxQ+UyoOAhfAaotRzcDA0AVf8Yhfh1dHCcAKn4C0oQmdh/kZWQxggDoDQWQQUCN00F8IHsPkK2Nro4oAAsPoJmngHQgujzRAKi5F2YYuhxJAOiVWIoNAqUnoAEnYAYBDQ1BV0MQABMiF1DzMxAbSPdADfuHro4gAGr6icY/APUeC7I4XgD0QjswliKQxYCGRIMMAooHIItjBcC8owL1wnN0ORAgxntMQAVfgPg9EH8CsYE2J8Ikgfy3QPwViD9C1XwG8ZENGAUjHgAAxqxQY20iKikAAAAASUVORK5CYII=>

[image5]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAHQAAAAXCAYAAADa+mvTAAAEcUlEQVR4Xu1YXYhVVRQeR0ksjcEcJ++dmX1n7o2BSSMyAgVBbOghjfBJHCTIJyGQfEkMUQoSpKYY9EH8R1BSEAwkg7Sg/BmU6SUi7DHLP0QcmtG0mvy+c9e+s866Z+4cufc6IPuDxdnrW2vvtc5e5+y9z2loCAgICAgICKiAKc65fZBv29vb1xcKhWdjVhjuQ25DLkMGIMOQ/+HcGXMMiAHz08N5gjyQ+frS+tQauVzubcT6WFGNjN3R0fFaiZFkvvF6JpOZI4leLjkFlAHzMwp5pbm5eSauvTJny6xfLYGCXkSMvZqTuGc0MaTsnltLR8s/ScBDvNFyaYG5+ZDz093d/ZTiOLF1nTOMf8XGkLhjNcSNva7sEfAkvCsdp1nbkwIWxXJp0dbWtkTmp9Fzj6OgFqjTy4yJfPLWFgMr7pPr7OxsR/u4T1jkX8hP+gkVcMO+Azknfje8AQ/OGujXcf0dsk3sa3Vn6P+pGFpOQpbTB/vFG2j/jZvpx/VXyHU9Rloghy2WqwbME2Pusnw9wZgo5quWj8E/fc48weS4hre2ts6GLIB+D/pvxucUbAW2aZM+66hLQU/7Uxna30FGdX95eAa8jnYfx9A+0O/7QwDGmk47Yr6gfdKglgVFDu8xj5aWlmesrV7I5/NtiHnT8iXA+A+TkiKsSrCPsKBexyRm9WRLgENeJ6D/6H1YUG2T4pUtUeBOqvZn1idJh+zUXBrUoqCIe9XJqsLlz9qT4IqrW5Jw/nli5lfHPchdyLDt7wHbL6jBDMuXwT/1HFTzHFwXFPoFp94wtI/D/gOue6x4Hw3wtyBHEvjdql1WUI9sNvsc4r1PO4pz0No94NMEed4K+n1qOQre/hY7RhowD/eYvgwQZ95485II3Ngmmagez7ni9+lfkEHINVfcx1YrO22xI/V4gN9hjhf7fhqzRXultMsKygkHN8qlX3wqFhT+LyHPpVbQ54DlvNgx0gB5nGYu2LLmW1utwS3GzktF8OTLDri5jzzHAuTUGwpMpQ9ks9i/HidI6SSo4VcCxHrRc9AXaR9b0K6urlk2xkQFHQ/VLLmuuDzG9n/oOySX9Zq3gM/PjyK2/0RI/CxxsskjubcUZwsaTSbke2l/Tr3BFDCXsB97SIzSBDizFzpTUPiu0Lr4MIfY3p0GVRaUMW0eXHXIv6l5C9g/eRSx/RWmWiICOt1K4JiYTTg65XqdJzrholOs9xFue674m4qn1B1i6/N+BOxN9NXHbnDHtA/sO3UeKMI7EqODusSgfh7tDWM9J0aVBb2LeJsMxzxib229ILEovdYWAckdVU6UQWXL4eZPGDtPYpfAL9TjNBT/LUb/NkWiYnpgqK9c8fTGf8Z8Oz8g7+TPVAU5C1kpvtHfEvT9IpPJPI3rLugjOk4aVFNQwhUPdbwXLo3M8Q/rUy8g1n7In6lOuZMBFLq/0gcykh9CAbZavhpUW9CACmBBLaeBgu6tdUEx5mLLBdQImNx5ltPgiTifz8+1fEBAQEBAwCThIb9LrraWC8bkAAAAAElFTkSuQmCC>