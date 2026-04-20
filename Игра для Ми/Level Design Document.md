# **LEVEL DESIGN DOCUMENT (LDD)**

## **Проект: «Числа: Мир разрядов» (Numbers: The World of Place Values)**

**Версия документа:** 1.0

**Цель документа:** Описать архитектуру прогрессии, кривую сложности, структуру блоков (Worlds) и детальный сценарий прохождения уровней для геймдизайнеров и программистов.

## **1\. ФИЛОСОФИЯ ЛЕВЕЛ-ДИЗАЙНА (Level Design Philosophy)**

* **Pacing (Темп):** Каждый уровень начинается с медленного введения (Story/Tutorial), переходит к плотной практике (Grind/Drill) из 5–7 похожих примеров для закрепления паттерна, и завершается эмоциональным пиком (System Glitch), где игрок ловит игру на ошибке.  
* **Fail-Safe (Защита от провала):** Игрок не может «умереть» или провалить уровень. Если он долго не может решить пример (более 15 секунд бездействия), Десятки или Единички дают визуальную подсказку (покачивание нужного элемента).  
* **Связность:** Новая механика всегда вводится на уже знакомых, маленьких числах. Как только механика усвоена — числа увеличиваются (переход в двузначные).

## **2\. ГЛОБАЛЬНАЯ КАРТА ПРОГРЕССИИ (Global Progression Map)**

Игра разделена на 9 блоков (Worlds). В каждом блоке около 10 уровней (Levels).

* **World 1: Основы (Сборочный цех).** Знакомство с Единичками. Сложение и вычитание на базе уже известных чисел 1-10.
* **World 2: Переходы (Десятки).** Числа от 10 до 1000. Сложение и вычитание, паттерны переполнения разрядов. Появление тяжелых Мехов.
* **World 3: Пропавшая часть (Энергетические воронки).** Сложение и вычитание от 10 до 1000 через поиск скрытого значения X.
* **World 4: Коробки-скобки (Силовые щиты).** Группировка и приоритет операций. Нельзя складывать внешние цифры, пока не решено то, что внутри щита.
* **World 5: Умножение и деление до 100.** Изучение многократного сложения (клонирования) и разбивки на равные части.
* **World 6: Умножение и деление до 1000.** Быстрое считывание паттернов при работе с большими скоплениями Мехов.
* **World 7: Дроби (Fractions).** Введение долей от целого (1/2, 1/3, 1/4) и переход к составным дробям (2/3, 4/8).
* **World 8: Договоренности (Variables).** Работа с Доской Договоренностей. Подстановка значений вместо кристаллов/иконок.
* **World 9: Формальная запись.** Высший пилотаж. Переход к строгой абстрактной форме записи вроде X + 7 = 15.

## **3\. ДЕТАЛЬНЫЙ РАЗБОР БЛОКОВ (Deep Dive)**

Ниже представлен подробный дизайн уровней для самых важных этапов игры: **Блок 3 (Неизвестное)** и **Блок 4 (Скобки)**.

### **WORLD 3: ПРОПАВШАЯ ЧАСТЬ (Мягкий вход в уравнения)**

**Цель мира:** Научить ребенка решать уравнения вида x \+ a \= b и a \- x \= b методом достройки (Counting On) без использования сложных терминов.

#### **Level 3.1: Спрятанные Единички**

* **Сюжет (Story):** Единички играли в прятки. На поляне было 10 мест, 4 Единички стоят на виду, остальные спрятались под листом.  
* **Предсказание (Prediction):** ? \+ 4 \= 10\. На экране выбор: «Там спряталось больше 5 или меньше 5?».  
* **Действие (Counting On):** Игрок видит 4 заполненные лунки и контур на отметке 10\. Игрок тап-свайпом добавляет Единички от 4 до 10\. Счетчик показывает: добавлено 6\.  
* **Обратимость (Undo):** Нолик: «А ну-ка, вылезайте\!». Нажатие Undo убирает добавленные 6 Единичек, возвращая пример к изначальным 4\.  
* **Практика (Grind):** Серия из 5 примеров вида ? \+ 2 \= 8, ? \+ 5 \= 9\.

#### **Level 3.2: Восстановление Баланса (Вычитание X)**

* **Сюжет:** У нас было 12 камней, но птица унесла часть из них. Осталось только 8\.  
* **Предсказание:** 12 \- ? \= 8\. Нолик волнуется: «Сильно ли нарушился баланс?».  
* **Действие:** Игрок стартует с 8 (то, что осталось) и достраивает «мостик» до 12\. Ответ: не хватает 4\. Значит, птица унесла 4\.  
* **Сравнение (Comparison & Pattern):** На экране два примера: 12 \- ? \= 8 и 22 \- ? \= 18\.  
  * *Десятки:* «Смотри, нас стало на десяток больше, но птица унесла столько же камней\! Узор не изменился\!»  
* **System Glitch (Босс уровня):**  
  * *Система:* «Окей, 22 \- ? \= 18, значит 32 \- ? \= 18 тоже будет 4\! Я уверена\!».  
  * *Задача игрока:* Нажать Undo, чтобы доказать, что ![][image1], а не 32\. Система капитулирует: «Твоя взяла, баланс важнее моих теорий».

### **WORLD 4: ЭФФЕКТ КОРОБКИ (Группировка и Скобки)**

**Цель мира:** Визуализировать приоритет операций. Научить ребенка сначала решать то, что находится внутри скобок.

#### **Level 4.1: Рюкзак в дорогу**

* **Новая Механика:** Скобки визуализируются как прозрачный «Рюкзак» или «Пузырь», внутри которого заперты числа. Числа снаружи затемнены.  
* **Сюжет:** Единички собираются в поход. Сначала нужно собрать вещи в рюкзак, а уже потом класть рюкзак в машину.  
* **Пример:** (3 \+ 2\) \+ 4  
* **Действие:** Игрок пытается перетащить внешнюю «4» к числам в рюкзаке. Рюкзак отталкивает её (красный блик, звук натяжения резины). Игрок должен кликнуть на сам рюкзак. Внутри него ![][image2] схлопываются в 5\. Рюкзак лопается. Теперь можно сложить ![][image3].  
* **Практика:** (1+4) \+ 2, 5 \+ (2+2). Закрепление моторики.

#### **Level 4.2: Две коробки (Независимые группы)**

* **Сюжет (Кулинария):** Сначала смешиваем муку и сахар (одна миска), потом молоко и яйца (вторая миска).  
* **Пример:** (5 \+ 1\) \+ (4 \- 2\)  
* **Действие:** На экране два Пузыря. Игрок может лопнуть их в любом порядке.  
  * Пузырь 1: ![][image4]  
  * Пузырь 2: ![][image5]  
  * Финал: ![][image6].  
* **Обратимость (Undo):** По нажатию Undo финальная 8 снова распадается на два закрытых пузыря.

#### **Level 4.3: Великое Сравнение (Core Conflict)**

* **Сюжет:** Система утверждает, что порядок цифр — это главное. Если цифры одинаковые, то и ответ одинаковый.  
* **Предсказание:** На экране (10 \- 5\) \- 2 и 10 \- (5 \- 2).  
  * *Система:* «Как думаешь, результат будет одинаковым? (Prediction: Yes/No)».  
* **Действие (Часть 1):** Игрок решает (10 \- 5\) \- 2\. Схлопывает пузырь: 5\. Затем ![][image7].  
* **Действие (Часть 2):** Игрок решает 10 \- (5 \- 2). Схлопывает пузырь: 3\. Затем ![][image8].  
* **Pattern (Паттерн):** На экране горят ответы **3** и **7**.  
  * *Нолик в шоке:* «Равенства нет\!».  
  * *Десятки объясняют:* «Коробка меняет всё\! В первом случае мы убрали целых 5, а потом еще 2\. Во втором — мы убрали только 3\!».  
* **Практика (Grind):** Решение 5 пар подобных примеров для закрепления зрительного паттерна (где коробка в начале, ответ меньше).

#### **Level 4.4: Мастер коробок (System Boss)**

* **System Glitch:**  
  * *Система:* «Так, я поняла\! Если коробка в начале, ответ меньше\! Смотри: (20 \- 10\) \- 5 \= 5\. Значит, я могу просто переставить коробку в конец: 20 \- (10 \- 5), и ответ тоже будет 5, я просто систему оптимизировала\!».  
  * *Действие игрока:* Игрок должен не нажимать зеленую кнопку «Далее», а принудительно схлопнуть коробку Системы ![][image9], а затем ![][image10].  
  * *Нолик:* «Попалась\! 15 не равно 5\! Ты защитил баланс, исследователь\!».

## **4\. МЕТРИКИ УСПЕХА УРОВНЕЙ (KPIs & Metrics)**

Для каждого уровня геймдизайнер и аналитик отслеживают следующие параметры:

1. **Time to Complete (Время прохождения):** Идеальное время одного уровня (с учетом практики) — от 3 до 6 минут.  
2. **Undo Usage Rate:** Как часто игрок нажимает кнопку Обратимости *до* того, как это попросит сделать Система или Нолик (Цель: \>40% самостоятельных нажатий к 3-му Блоку — это индикатор формирования привычки самопроверки).  
3. **Glitch Catch Rate (Успех на Боссах):** Процент игроков, которые с первого раза замечают ловушку Системы в конце уровня (Цель: \>70%, иначе ловушка слишком сложная).  
4. **Drop-off Rate (Отток):** Если на этапе введения Скобок (Level 4.1) отток превышает 15%, необходимо упростить визуализацию "Рюкзака" или добавить дополнительный туториал.

**Конец документа.**

[image1]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAGIAAAAXCAYAAADwSpp8AAADS0lEQVR4Xu2XPWhTURTHW4pY/EBQa6X5uGkSzCKCBEUUREQnFUG3uji4KCo6CdVFqYOTDqIODrp1UEFB0M2Pza/F2aFarULRCqaV1q//ae59nPxzw3tpa7LcHxyS97//e+9557zcJB0dgUAgEAgkxBjzHPGXdQfG7so44jfiTyaTWc+eVpLNZndJPoVCIcNjrQL7j9qaSEwjlzXsyeVy+5VH4ip7Okql0nIMHEe8dUb2CFisG2OjfX19q1OpVBpFuIXr7+xrJdh/RPJtVyNQg8tSB9RmC+Kord8E+6x+wHpm69zf37+hxtTT07NMiqsmeBvh05HEMBY/zHocmPOAtWbB3ruRU6WdjcDeY/pa8pB8pEHK8wFF36x90H766hkR14je3t6lWsOG16EPai0JmPOQtWbBGh8Rb9rVCOy71dbrDuk1NeRrq50Vre5T4fBNcrgxFP+k1rQnKVjjEWvNgH1H7GvbGiHgPk6wxjXEMb4KBd+uPZh3Qzz5fH6F1iN4EQ30m24csRHHyxUJ9iUBiTxmrRmMPRLa3Qgftj6vWNe4vFmPcIVm3YGxIeeRkC8p9iRhPo3Avu/U+0SNwJO3Dr5JxC9PzCCmTfXcnrK+iu/XTxyoxzPJh3VGPPDeZj1CDI0WQvHOIJ7ibSc835y34TlnwYZrOTDvCWsS6XR6Jc/XYP9tmHvQXZuEjWgFyL1oC9zNYw7rGYfnPo/V0KgR5XJ5kU831aenTtdg0x0cmPOSNQn+daHB+CXM+4HXHG4oJWHsT0H8n9kk1zynVZiY/19CrvrzX+o7xGN1NGqETEbcY13w+eOYy9GEfQZwM6d1uHzx/rxc8xwHjqYsfK9NtXGJQubwOj7gvYY4RFpdTaDN0PVOPHhGaxGygG8Re+MvWBd8/jjm0ggfLt+4own/k5bAdwFxMWlgWhev4wPeKY82SdfncM/7tCZHPV46tRYhNyXBumD1momm+jQc01oSFroRWC/PY63AVL/oZ3PQgXyGlWcPj7vQawldECcQnxGfbHxBVLSpWCwulsnY5D1ev8p7fEpOaU9S5tsIebqw/zhizOYrudc8hf8b5LBXatAgBpzPMxaFXq8tzLcRgQUCT8MR1gKBQCAQCNTzD589XyAEodu4AAAAAElFTkSuQmCC>

[image2]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACsAAAAXCAYAAACS5bYWAAABqUlEQVR4Xu2VP0vDUBTFY1EHcXCJSEjzhyhCHASzS1H0K+igzuLg4CS6dXcTv4IufgVdXBTcCm4ugoM6iSgorZ5bXsrrqa2PWCtifnAJOfe+3PNu/llWTs4fxfO8Bd/33xGv6njINT2gH31fVH+JuyRJBrjIQqKGmLFtexjGV6Q4CIIS1/0k6HuOvmUc53HcUR4umoog7koijuNBTavvTq8zBQ2WWfsK13Un0O9E12B6VTzIXW+IYRjOKmOFVOu12XRgWLtOuvi40bUWVNE+6wb0ZTEroN/WJ5r4OGO9Aca+KUWO4wxxzoBCVrNMFEWjymyZc7KLW0RVCvBoTHPekK6ZhY9HRI31FtSOrljXgakxjmKx6Mhzx7oEr+8E6peMjAooPFUTnuRcCi5Y4sCaObltrEvw+nZg/T3iifU6SLzxLnB+oKa7oesGfOsxQL9LvDNTpFX0EzHV9JnCgiM12UVdNyCzWflboec169CO9ZNnNNjW8ukGqrpmSGaz6dA4MLg1LnwQ04iKKur8IW5PJrPot8cm08D1Rri+W2Qy+2vgPz/OWk7Of+QD3xKLJr8T5UAAAAAASUVORK5CYII=>

[image3]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACsAAAAXCAYAAACS5bYWAAABeElEQVR4Xu2Vu0rEQBiFs2trG7zE3EAQfAQr8RksBCs70cZLLYh2VhYWWgniI1jYiqCClj6CKNi4IGihEL+RIOMh0VyWgJgPzs7MmX8yJ8nsruO0tPxRwjBMoijaRQv019EFuta6JmH/S7Sn/mdYled5Y1rXJGmOzLB36DYIglPaDZ0vA29nTr2SDJDh6qewJ+pVpW5YsrxyjZVfw9KOuK47qPMl6NQJy/5rcRxPpP38sOg4LUg4DrNaU5BuzbAvVj83bKJjdGR7Bakclv2WeEiT1jg7rELRtt6AQqhhle/7o7SL6hvpeoX9bmScG7ZjDyiaN8WcnyHbtyHAtIo1M2hLfSNdb8OaHorNzyWYD8/sz5M+NH270NyBHoPl1OvafgEqHQPWrKrSXOem/1WYFZaCTfUKUilsFmmu78cAYwdNifdm/iBsryB9Dcu1DtR3CHbG5Dt6NkWM97WmILXDsv8TekT36AH1tKZf1A7bKHx7x9VrafmPfAANsH6okFtbtgAAAABJRU5ErkJggg==>

[image4]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAE4AAAAXCAYAAAClK3kiAAACP0lEQVR4Xu2W3SsEURjGl0iKy5WyuzPbbm2ut1xR/gJFpERS/gKXktxQbnChXCByR1wKVyT3LvgP5CNKIvn+eN6d2e09785qhjEW51dPu/uc98y888yZMxsKaTQazT/AMIw30zSnoD58H4D2oANZFwSxWKwb536V/k+APEbQyxX0TD1FIpFKpYCCk8IkUyn6ZnDONmjWPv+PB4eQ6qgX3MitVCpVjc8xaEYpQsExdIiBTWhYGfQI5i9Kzw3xeLyBPoskuBLqgxvZBcU9MjcU4wt8NrgsxRAczn8G3XCPVh0+SrmXCw5PZ204HK5SBj3yR4KjHtYTiUQNrqcD6pE1GSg4aMmeQPtbp6xxC06yID0vFFFwc9A1dARd0F4n6zKF4vcLwlvmnlt8Ck7dSz4AffYa1lvPSY/QA3QP3UG3qN+VxxBk9jfZg328Ne7lgYIhOVFCy5gebQctO3i1cn4hWNMlciwgssHtcxO/550yUZrEqmmnIlywyX1ONBptwnCzFL2ZpUeS8wvBglM34gCxe5jgHq5rUgmONcq9fvLS6XQ5993g46NaJsecwA1sQf2BB63LY0jo/PI/G7xpu6+c8aYYljcoPbf4FZzbm4baemjUg7rkMSSoeYVWuUfXZfBM8BiNw2xkNTSRNtRt7rnFr+CSyWSFHAsK2lqUkEK5VaheG8wdw3pr0OuXGp9TCjyQd3CXGNab7xw6sXUKXcqbGhSGtTopC/o78oQwV2SNr3w2uH8P7k6r9DQajUbze3gHLpfk2x9eKFUAAAAASUVORK5CYII=>

[image5]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAE4AAAAXCAYAAAClK3kiAAACKElEQVR4Xu2Wu0sDQRDGE1ALQXwhSC7JJiEaDGgTQcvgAyu1s7CxsxEE7bQVLURbe/8HxVIRxUJBxMZG0UYJBmwMaozGb8xG9obLC7k7wf3BR25n527mvr3Nncej0Wg0/xAhRJ7HHKAOdV+otlQ6Ho838CSnCIVC20ov+WAwOM5zTAQCgX43jEPNE2gFGoKeZMPnPM8J/H5/F2rfwLwBaBLHH2U9gatrSMiUTbIBNBpFzX01hl6mqlppG6C6sVisicegrBr7hrYFJnYikUiz08ah3rJsbI7FKZZWY04g65o8wPiNx4oT7/TrhnEEai5axOgGznjcbrA9k9gFbWrMykxPIpGoR3CPjt0yjoMt2iqbXedzbmBpnFD27l8wLhqNdgj5guBzHDwdM8jLlVBWFLbYqyi8sTPIP+TXqAQWcZR6wVPY+xPEhZbC4bAojmsxDud2ViPDMNr5ueVA/c9qe3AC6gXmjagxL4IXZBzcNEgY90h3DXya+NRkhhemJKvUID+5FKidQt0JHneDUOFTJIeHqds0Qf9tmFxQhcQtMo6O4fK86QSbQd1TLGIfi12rYw6ZjJzLGrTLr2GFKDxAx/ChRYkdqTkmYNawcGGbyBfULY9XulF5g6s1aJpfwwrk3fl8vkYWe1bHJuDwmHDBOKppJSzkLM+1G9Q94H0UxXMpeRN6hB6geygFZXieHaDOBm+wKCxkJ8+3G96Doiueq9FoNBqN5jd8Afbe3ejJVAFLAAAAAElFTkSuQmCC>

[image6]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAE4AAAAXCAYAAAClK3kiAAACnElEQVR4Xu1XO2gUURRdjRFSBBFUkv3N7EfWbDAI24mCpBNRsBGMhZ1NQILpYik2SeoUFtHCJqQTBAuJYmMRg4WdKIidEAz4SVQ2Wc/Nvt3cOZmYmcnOQPAdOMzMmTPv3jn7ZuZtKmVhYWHxnyCXy513HKeheJE9MeMQaq6p+svVavUwm5KC67rXTR9/ZIvjMfZsAifX8/n8bRhOYf8S+J09cQL1XoP3wGFwxTT9ln1JAbXryGMw1fxBR6Qf7HexqQHTtdYxwntvjKGBcR6xthuy2WwZ9V5oTfoxfV3WehJA3afYHCBtAPyohVG/kBBeH2tBECU41J8wM2yUdNGWtZYETN0bpPWDa1qom+AOYjsO3sUM6Nm6JByiBCdA3Ts+mtzAG9bjBibNE1N7AVmcFA37K9jPtE3GIJwHn4NL4GrbEBII7iFrUYBxjpq+Jvlc3EBAp1Uuq+BZ9PPZY1KGfqXNgHXtC4pOBFcul4875gPB5xiYHTel1x0oX8Tf4C+n+cX+Cf8rHsMPMrukvuKUx9A6obVSqZQTrVKp9GpdA54T8h704ZyPFup9idob3FOSKBaLeQlb9tG7q8KbbZv8gsOFR4y+7b3Tgqz7MOYFJmbcM9aEfP1OQM0vGPsK60mC8zDagkeXRtmYTqePmeA8X5Yg2MujinqLhUJhiLStJYAPJGR43oWgLDX+BVm3bQtO4NFx8ICNaN4RLZPJZLUeBFGDq9Vq3aj5ifXdbtRprq/uh+AIj8GQe2dNwLosQxr67w2OH4M/tCkoogYnPfgR491ib9xA3Q3UPac1vG7OQP+gNbnZmmlU/vbIo/vVYwiBKMGh3jQH1qIb8sPSKaD2N6e5FFkE18GX7OkoogRnkdr8pa6yZmFhYWGxf/AXpfT7CJ3Hje8AAAAASUVORK5CYII=>

[image7]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAE4AAAAXCAYAAAClK3kiAAACQklEQVR4Xu2WP0gjQRjFFfQaBRUJCeTPbkIaw2GTxuIKuVLQSjywEUHkwC6FxSE2YuFhIdheZXfdXSG5ThHBQkFQi2sEQQRFQRDixaDG9yUTmX1JcEV2LG5+8Ej2zZf93s7s7qSlxWKxWP4DHMcpu667Ak3iew7ahg65LmDa0POfZFG6ymQyH7jIFOi/qnLcQkXMzVeuqUwcC4Uu1wUJeu5AC9Bn6Frl2Oc6E+DSu9H7NBqN9qrj35KnbiFhnkFHiUTiDzTvGTRALBZLo/+G7iHHmITF57DumwB9C9KbPFnIS90TM+8xDIP+31SwGfLFu9I9E8gNhL6PuqeynOve88ThloyEQqFOz6AhkCHXwJOwe+ybBhmGJAsmNMUDeWhNBZX32xdPwTuAkD0qz3ceMw0y/IVK7FdWlo4fMHk/dc8k6XQ65KgNgscY5JxA3X0TlaA7qOhUd+wC6rf4HM1A/YVT3VVvwuFwB4/XgcI5x1/oiB/Vdie/oPejn/6mSKVSXZIHT8Esj7XqBygYlUJctKv7RCuGB31qgH/cDFnleDw+wv57I/PhWcw6o+pNiZfNZtt1P2jQczeZTPaTd6wfMzLJqDl8hdb5HIxcO7TdwHtx4ip/D3QvaGSR0POE/ZcuFON90OIrNM7n0FE5yiLdr/PwGC3h0fyk1UiRvFA9f0iDphaMhWzTXBs00hd38kf2MFcHuifmplPdhW5U4B+egoBBv2WesJoQNsL1QSObmep/iv5b8h0L+IvrLBaLxWKxvJUnaZnjwHZlmigAAAAASUVORK5CYII=>

[image8]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAFgAAAAXCAYAAACPm4iNAAACl0lEQVR4Xu2YPYtTQRSGd00h+AWCMWsgmXxhRMTOQgW1FVGwsrWUXSsttBYESxcb2R8gaCcuiOAWamFlo42roAgiayF+bRajQX1O7mSZnMxcg3jvWswDL7l5z5mZcw+zd252YiISiUQi/w3GmEfol/YH1Gq1m8R/WPWKxeImnZM1lUqlKTWiFfQRLeicvLB1fJEa0F10hx7dFnF9sp/Ubrc38+UsemYHeBtcLpe3SYzPDdJYrufQK52XNSZp7DEuC9zIC6mpWq1e0Hl5MOiXT9R2pp8kzZLmuQOGZrGIX6/X92oPHXW9LGGt/bImu3if430N1Zw1rNulkRdNskGnpakirj/r3D6hBrNDtvp8vJdoSftZUSqVNkodjUZjp3xvtVrFUM15wLozHm9Fe6uEisU77/Np/AOfnxfsliu25kUdWwuo5yo92a39VUINZuCsz8e77/PzgrU7sj717dKxtYBafmpviFCD8W4EfDk5R/ysYc236Dt6zo45ruMaeaSY5HDseSRvRDLXNyvJ6zSbze16njQYM4PmtD+ECTf4ms/n5u75fBcOxhI7bGoc6bF/gsNuj0le1ZZ1LE+o/ZT0gTNhvY4NkdLgaZ/PxA99vgs5h9GRcaTHjoO8otm6z+lYXoT6NkIoEW9HwH+DutrPiEnWWtR1sItP2LrnXd+FR0SV+BOTvOuPJRmj5wlh1x/pzwhpiT4fr2tSbuxfwk496KuPnX9aPD5nXd9FfhyRcwldHlcMK+h5Qvjq8pKWiP+Bm7yuPHnubHG9DJmkiU/RrYFhf1G+DtWcB6x9IK1vQoHgJ7SE3lm9Rx2diLeMFgbvvxxgh3RO1pjkT1hO/8f2xqTOdTovL+y/GqSOno5FIpFIJBKJ/B2/AS5NDd1Exp9dAAAAAElFTkSuQmCC>

[image9]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAGcAAAAXCAYAAAAWY1E4AAAD1ElEQVR4Xu2ZS0hUYRTHNSuwB2Fgho+5MwpNDUUPIwtaFAQtDCpqFwQVRAS9iIiiiKInLtq06LGKalER0aYHCUotoigkExMCixYVFEgPU7S033G+C9+cubdG8F51mh8cvjv/c+6c853vu4/RvLwcOXIMISUlJRO1li1Eo9EarY0KHMc5iPVqPZtgcdYwx71aH9FEIpEERT/kMF/7hFgsFsffr3UXzj+C/zP2TuKwmI4JGpO3ngU4zbgZO+dVM9pzbIHWw4T8rVgTto96NzHWYb913AA4WmjwUg99C/opxj6viRoKxIfVywfG7VinDgoaU0OKUft9HUczVuNr0HqYkP+trlU0HTeAOLUmMJGojPhP+MWgd9CEnVrj1DO2FjRmgg1YG3a7qqpqmo5xMbFPtB4W5H5Dzx7Qo2bZQIwzdcwATvLyv6t1m38sTr9+iUC7ZBowy9aDxK8+L2jI08HEDzXkbtOaJ6aJy7Vu47c4TLLaS2cnHBUd/3ntCwq7jrKysnLbp6Gu9RJfWVk5RfvCgNyv3WN6Nd32pSBF6p2v8VsctLU++iHRSXxD+4LC5KthrJdj7JaOceGWV2E2zyrtCwNyt5aXl5c5yZcoqfsZcoGOy+h24Lc4TG6rj77fNKhJ+4LC5Kt1P0eTb0Fptbng+44d1roNV9YMYn5ivzysF+vBuo1JXOffnnUuxL3EjrufKyoqqhyvWj1FheOzOGjrvHQac0B07LH2ubCARXJJZ2KEj9HnZ4LUIK/5Whf43mb817Q+XJh+NaSJKYIHfosTSf4+StPRjplkF7XPBd98GrQsE+MWUKjP17D7SrVmFue91gUn+TvjptbDwK/WtF6mCR74LY5gGlBkazT0suiMK209KMi/w+SbZ+tmwo225oL+FavTug23tQgxL7BXmZqco7/Hho02x0nWutvWRcN6bG1IFoedMFtp8lDuqK6uHmfrQcHi7JI6iouLJ9m6aPjO2ppBfjj3ybNRO2xKS0snOMm7gMw/I8vzerBbxGKxuaaulB/9ojn6GS0iq7g4RVSYxJ6Lw7mP8LVwONbVJJYFW2iFBU0+Odttgbr2+NWMXis+dvFU7QsDcnfbn6X/ptbUP58hXnF8XjvRv2FfsI/YB+yT0Qb+VGPFtcuXk+QCYxfjNtsfBuRdIjVgP5zkW1RXnp6sAV+jxGo9LOSNztQqzz3pZ388Hp+s4+QeWDichQ4HZiPd0fqI5H9cHO75K7Q+IqHYDdhVrWcjzPOeM8z/Mhg0XObX5f6n9WyCN6VFXDEntT4qSCQS4z0fTFkCC7NRazlyDIo/tENXSm3ESUkAAAAASUVORK5CYII=>

[image10]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAGIAAAAXCAYAAADwSpp8AAADH0lEQVR4Xu1XXYhNURi9KEnyf/3cv33/8nAfPEqRGRMaITUvnv288IiaRESmPBJK4oEHJS+UQtGEeOPFvI0UGpSREsaY0bW+ud8x+667753j4e552atW55z1rb33d/a3zzn7JBIBAQEBAQFTIJ/P7zTGVC1eYI8AvpuIjSnHk8nkPPa0G5rfFXAf2JfL5W7jOMo+n8D4G8GnmJ9OjgkkZ8TOgrtxflC84AD7opvrgXE/jq/kulAorLY9qVRqqeg4zpUC4Pwy+Mb2+IDmytzKPh/AItguExzl0aoQDhbY9B6Tvoa0X2ImraE42qHXScB44+ALsB833gFpBnt8AfOxQY7I5ZnMRYtCDIEDKNx98ATHJ6CTyZN+1J54NF7EHvUNgp9Ybycw3ghr040YhbjHWgPS6fSSqLIRMPGXpONisbhArnF+yFUI+B679HYiKgRueiHHpgtxC4H4iv/6rqLhS3uC0cE514RDe+jS2wkpBPI5KeNiIbzD8TB7fCNOIcDr4lHfLvY4oeZr1vUN14TrAA16O6G5zZHzSqUy29Rej4Pss4EnexU8P03t+8KUHeBvU/sujqjvR6lUWsb9NIOZuhD86v8D3rK1OmQymTIMw+jwjq1DO8+dCbAiH7h0G3jtLZdHMg65bRwgh3WSQ7lcns8xX4gKAXZxzAWj32DWJyCrTDs7zTFoB1wN0eaJS7cBTwfYGYfc1oW8Pg0RsHgWa957bN0nokJgUWzimKJuZwdvj/qLth4Fx+i6C6vZ6PlK14RDe2v8/UzNxFifOY9sNlsSrVUh8WrKmdqWV/6RYlHacD/NYCYLsdkRqwpJ2yuavFptXQLH0MkOW8N1b8KqJHem2ih4l/V2AWN95TxQgLWi5VvsouQnFJ5TYF9cotks7qcZjBYCC3eLI1YV2hpyPcKaGLdFZib5vsi2ljSv72bcQLexdkn6t/8aHLZ9vmEmP9bdHIN2BvO23tbgHZWtv639q5iLdcaa9zv4KPp/4P8PH8C4FzW/IT0+Z48vmNquS16XH8EPepQFe5x8/er9pjlfteMBAQEBAQEBAQFx8Bdink3f5fcWVgAAAABJRU5ErkJggg==>