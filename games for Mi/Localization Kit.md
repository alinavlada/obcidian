"String\_ID","RU\_Text","EN\_Text","Context / Character","Notes for Translators"

# **\==========================================**

# **1\. UI & NAVIGATION (Интерфейс и Кнопки)**

# **\==========================================**

"ui\_btn\_play","ИГРАТЬ","PLAY","Main Menu","Large green button"

"ui\_btn\_next","ДАЛЕЕ","NEXT","Level Complete","Button to proceed"

"ui\_btn\_undo","ПРОВЕРИТЬ НАЗАД","UNDO / CHECK BACK","Gameplay","The most important button in the game. Shows reversibility."

"ui\_btn\_hint","ПОДСКАЗКА","HINT","Gameplay","Optional button if stuck"

"ui\_level","Уровень {0}","Level {0}","Top Bar","Variables: {0} \= level number"

"ui\_score","Очки: {0}","Score: {0}","Top Bar","Variables: {0} \= points amount"

"ui\_loading","Загрузка лаборатории...","Loading the lab...","Loading Screen",""

# **\==========================================**

# **2\. CORE PHASES (Двуязычные термины фаз)**

# **\==========================================**

"phase\_prediction","ПРЕДСКАЗАНИЕ / PREDICTION","PREDICTION / ПРЕДСКАЗАНИЕ","Top Banner","Displayed at the start of the level. Keep both languages."

"phase\_counting\_on","ДОСТРОЙКА / COUNTING ON","COUNTING ON / ДОСТРОЙКА","Top Banner","When adding elements to find the difference."

"phase\_inversibility","ОБРАТИМОСТЬ / INVERSIBILITY","INVERSIBILITY / ОБРАТИМОСТЬ","Top Banner","When checking the math backwards."

"phase\_comparison","СРАВНЕНИЕ / COMPARISON","COMPARISON / СРАВНЕНИЕ","Top Banner","When comparing two similar equations."

"phase\_pattern","ПАТТЕРН / PATTERN","PATTERN / ПАТТЕРН","Top Banner","When a rule or similarity is found."

"phase\_decomposing","РАЗЛОЖЕНИЕ / DECOMPOSING","DECOMPOSING / РАЗЛОЖЕНИЕ","Top Banner","Breaking numbers apart across tens."

# **\==========================================**

# **3\. FEEDBACK MESSAGES (Система обратной связи)**

# **\==========================================**

"feedback\_match","Совпало\! Баланс восстановлен.","It matches\! Balance restored.","System/Zero","Success message. Do not use 'Correct'."

"feedback\_no\_match","Не совпало. Давай найдем, где разошлось.","Didn't match. Let's find where it diverged.","System/Tens","Error message. STRICT RULE: Do not use 'Wrong', 'Error', or 'Incorrect'."

"feedback\_glitch\_caught","Вот это внимательность\! Ты управляешь числами.","What great attention\! You control the numbers.","System","Huge praise for catching the System's intentional mistake."

"feedback\_prediction\_ok","Гипотеза принята. Давай проверим\!","Hypothesis accepted. Let's test it\!","System","After the player guesses an answer."

# **\==========================================**

# **4\. CHARACTER DIALOGUES: ONES (Дроны-Единички)**

# **\==========================================**

"char\_ones\_intro","Привет\! Мы рабочие дроны. Поможешь нам построить мостик?","Hi\! We are worker drones. Will you help us build a bridge?","Ones (Tutorial)","Energetic, slightly nervous tone."

"char\_ones\_add","Давай добавим по одному\!","Let's add them one by one\!","Ones",""

"char\_ones\_limit","Ой\! Нам не хватает места\!","Oh\! We don't have enough space\!","Ones","When approaching 10 slots."

"char\_ones\_merge","Стыкуемся\!","Docking sequence\!","Ones","Right before turning into a Ten (Mech)."

"char\_ones\_pattern","Смотри, хвостик тот же самый\! Узор не изменился\!","Look, the tail is exactly the same\! The pattern hasn't changed\!","Ones","When noticing ![][image1] is like ![][image2]."

# **\==========================================**

# **5\. CHARACTER DIALOGUES: TENS (Меха-Десятки)**

# **\==========================================**

"char\_tens\_intro","Мы — Мехи. Нам нужна энергия для работы.","We are Mechs. We need energy to operate.","Tens (Tutorial)","Heavy, calm, robotic tone."

"char\_tens\_partner","Мне нужен напарник, чтобы замкнуть цепь.","I need a co-pilot to close the circuit.","Tens","Explaining why they hold hands with 0 or 1."

"char\_tens\_loss","Энергия падает... Отсоединяю броню.","Energy dropping... Detaching armor.","Tens","When a One is taken away from them (![][image3]) and they break apart."

"char\_tens\_compare","Десятки растут, но Единички ведут себя одинаково. Это закон структуры.","The Tens grow, but the Ones behave the same. It's the law of structure.","Tens","Explaining the pattern."

"char\_tens\_brackets","Коробка меняет всё. Сначала считаем то, что внутри силового поля.","The box changes everything. Calculate what's inside the force field first.","Tens","Explaining brackets () priority."

# **\==========================================**

# **6\. CHARACTER DIALOGUES: ZERO (Нолик)**

# **\==========================================**

"char\_zero\_empty","Я просто храню место. Меня тут как бы нет.","I'm just keeping the spot. I'm not really here.","Zero","Philosophical, calm tone."

"char\_zero\_power","Контакт\! Мы вместе — это мощь\!","Contact\! Together, we are power\!","Zero","When holding hands with a Mech (forming 10)."

"char\_zero\_check","Я храню баланс. Давай проверим назад\!","I keep the balance. Let's check it back\!","Zero","Prompting the user to press UNDO."

"char\_zero\_shock","Равенства нет\! Так не бывает\!","There is no equality\! That's impossible\!","Zero","When an equation is unbalanced."

"char\_zero\_glitch","Система хитрит\! Проверь её\!","The System is being tricky\! Check it\!","Zero","Warning the player about the System Boss."

# **\==========================================**

# **7\. CHARACTER DIALOGUES: SYSTEM (Трикстер / ИИ)**

# **\==========================================**

"char\_sys\_predict\_ask","Как думаешь, понадобится много деталей или мало?","Do you think we'll need a lot of parts, or just a few?","System","Neutral, guiding tone."

"char\_sys\_glitch\_1","Я нашла новый паттерн\! 33 \- 7 \= 36\. Идем дальше?","I found a new pattern\! 33 \- 7 \= 36\. Shall we move on?","System (Boss)","Smug, overly confident. (Intentional mistake)."

"char\_sys\_glitch\_2","Я оптимизировала процессы. 20 \- (10 \- 5\) это тоже 5\. Завершаем уровень?","I've optimized the processes. 20 \- (10 \- 5\) is also 5\. Finish the level?","System (Boss)",""

"char\_sys\_defeat","Твоя взяла. Баланс важнее моих теорий.","You win. Balance is more important than my theories.","System (Boss)","Conceding defeat after the player uses UNDO."

"char\_sys\_var\_intro","Новая директива: Красный кристалл теперь равен 5.","New directive: The Red Crystal is now equal to 5.","System","Introducing Variables (Block 5)."

# **\==========================================**

# **8\. STORY CONTEXTS (Сюжеты из реального мира)**

# **\==========================================**

"story\_music\_1","В музыкальном такте не хватает ударов. Помоги достроить ритм.","The musical measure is missing beats. Help rebuild the rhythm.","Story Banner",""

"story\_sport\_1","Спортсмены проверяют результат после тренировки. Сделал шаг — проверь дистанцию.","Athletes check their results after training. Take a step — check the distance.","Story Banner",""

"story\_cook\_1","Сначала смешиваем муку в одной миске, а сахар — в другой.","First, we mix the flour in one bowl, and the sugar in another.","Story Banner","Used for Brackets ()."

"story\_backpack\_1","Сначала собери вещи в рюкзак, а потом клади рюкзак в машину.","First, pack your things in the backpack, then put the backpack in the car.","Story Banner","Used for Brackets ()."

[image1]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADUAAAAXCAYAAACrggdNAAABuElEQVR4Xu2UPUvDUBSGHeqmm0WpTT+HugnGycnBwUFwVBEHXZSioJOD/8DVH+Cm4uYHuLi6CDrp0FVELQgKKmhL0efKTbg9adoUTBfzwEtuznlzPuilXV0RERGhkk6nL9C3jDuQK6EqqqRSqbtkMjkoPWFCT1vNl8lkLnmeoVPOx+hIyTUWCoVezGsYbtUHfksRX0X7tm13x+PxHs6f6Fn6woTB55wZG8k1qqUSiUSfOnuSBjpXNt7HVIxGy6YvTOi3h7bpucGzyHNFifNjPp+3pP+XFkupa1cy3ue1f8n0hQm93mUsm80OE9+RcZdmS0nwXQf1hgkzvMlYHW0upbznMt5JuHqT/B9syngdrZaiwBT5F1RDJzLfSVhovdmsLq2WMqHoovJyp/tlzgTfQFDJb5uhZ/2ScQ/tLKUI4mfY8aCS3/phWdao6svNOZA5D35D0jCji0yYcT9/2DDOoe5dlDkPfkMS29JL7Yp4Q3/Y0PNez7Mgcx78hszlciniNY4xJ8b7SODCf4wzJ7/YtMw5xDC8oif0oFVGH6aJAjO62BW60UVnTU+noHcFVek/JHMRERER/5MfLZSl7qwYuvYAAAAASUVORK5CYII=>

[image2]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADUAAAAXCAYAAACrggdNAAAB+0lEQVR4Xu2WzStEURjGR1GUpUFjvqfEShk2SllYSAorJAs2NFmwslA+tlbKysoOycZH2fgLFCuUraakxIIFM038zp1zde7bmJnNnVm4Tz3dc573Oed9z8e9Mz6fBw8eXEE0Gh2LRCLfNulvS48CsQeYhZlwOPwYDAbbpMdNkDOp67vieQHPaZ/CE0WHWS9mHKbgve53Cs8iPEgmk3V+v7+R9id8MT1ug8KndG0F+Wukkw6FQr3GWKVlHKa8pgY+G/0+pZFo3vS5CfLtwy1yLvNM8VxQpP2USCRCptG5Sp+1I5tKi8fj7YZPXbsHoz+tx87Zmtsg14fUYrFYF/qOQwwEAk2cVL+pcXf3VMEsrt7UTRC/kZtRDVDDu9QKAuNdqYL1KV1KvZJg04c4gBWpF0KNPqVdGWCCEWJvMAfPZLySoL6lUhtvQb1DGF/hsYxJMOmsmpQ73SJjJvC1lks5thj0TfmSugP85jToE9qQsb+gJy66W8w3UC7l2L/AN6BH5eXmHMqYA5gyoj/MZ7JZtUkY1ZMMCk/JRbkByjnSuVMy9guKXVeLENqa3Sa2qhe1Z3qqtShypnU9MzJmQb/8VnGStod3LUw/R7PW1uh3F53YRdj1cWKjMmZBLsSk6WOCCa1fw1vVRps0PZVCJP+PJ0v+Dhnz4MGDh/+JH6tOtUcsMIwEAAAAAElFTkSuQmCC>

[image3]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADUAAAAXCAYAAACrggdNAAAA8klEQVR4Xu2UOwrCQBRFU9jaWUiKfKrsQUVEW7di6z5clIULcAt+EMHGTtA7jwyMg06uYBLQd+AxSd5JuLdJFCmKUitpmq4xd/+5C/bT0hv4uyYJZi2KopskyQLC1kjvRDjzLMtW1mmjFJtVxDiOe+Y6JOZ5PjYnPrppsxST9QlGbLOUC5NVYEQtVSNMVoERbSmcQ3/XJExWgRGdUiN/9wr8Mfvs+O+GYLIKjGhL2b9hFQg7Ycd/NwSTVWBEW+rTEN+GySow4k+Xwjnzd01SlbWD5QVzwOzKOWKuroT7G+aE2ZeOOc8ot3S9mqGyKoqi/B8P4jSSW4aSQ6UAAAAASUVORK5CYII=>