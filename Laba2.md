## Комп'ютерні системи імітаційного моделювання
## СПм-25-2, **Смотрова Анастасія Олександрівна**
### Лабораторна робота №**2**. Редагування імітаційних моделей у середовищі NetLogo

<br>

### Варіант 12, модель у середовищі NetLogo:
[Muscle Development] http://www.netlogoweb.org/launch#http://www.netlogoweb.org/assets/modelslib/Sample%20Models/Biology/Muscle%20Development.nlogo)

<br>

### Внесені зміни у вихідну логіку моделі, за варіантом:

### Частина 1
Додати у моделі вплив харчування (умовний регулюємий параметр, наприклад, "якість харчування") на можливість зростання м'язової маси.  
Для початку додаємо новий керуючий елемент до інтерфейсу. Повзінок nutrition-quality відображатеме значення параметру якості харчування. Якість харчування буде мати 5 можливих станів:
<table>
<thead>
<tr><th>Значення nutrition-quality</th><th>Вплив на м'язову масу</th><th>Коефіцієнт</th></tr>
</thead>
<tbody>
<tr><td>1</td><td>Значне зниження м'язової маси</td><td>-0.015</td></tr>
<tr><td>2</td><td>Незначне зниження м'язової маси</td><td>-0.005</td></tr>
<tr><td>3</td><td>Підтримка поточно м'язової маси</td><td>0</td></tr>
<tr><td>4</td><td>Нормальний ріст м'язової маси</td><td>0.005</td></tr>
<tr><td>5</td><td>Прискорений ріст м'язової маси</td><td>0.015</td></tr>
</tbody>
</table>

Для цього у процедуру **grow** додамо код для додавання коефіцієнту відповідному якості харчування на зростання м'язової маси.
Додаємо рядок для збереження початкової кількісті м'язових волокон
<pre>
let original-size fiber-size
</pre>
Залишаємо оригінальний розрахунок зменшення та збільшення м'язової маси відповідно до гормонів
<pre>
set fiber-size (fiber-size - 0.20 * (log catabolic-hormone 10))
  set fiber-size (fiber-size + 0.20 * min (list (log anabolic-hormone 10)
                                                (1.05 * log catabolic-hormone 10)))
</pre>
Розраховуємо різницю між новою та початковою кількістю м'язових волокон
<pre>
let growth-change fiber-size - original-size
</pre>
Розраховуємо вплив коефіцієнтів відповідно до якості харчування на зростання або зменшення м'язових волокон (1-4) 
<pre>
if nutrition-quality = 1 [
    set fiber-size original-size - 0.015
  ]

  if nutrition-quality = 2 [
    set fiber-size original-size - 0.005
  ]

  if nutrition-quality = 3 [
    set fiber-size original-size
  ]

  if nutrition-quality = 4 [
   set fiber-size fiber-size + 0.005
  ]
</pre>

Розраховуємо вплив коефіцієнтів відповідно до якості харчування на зростання м'язових волокон (5) Також необхідно було збалансувати можливість від'ємного показника зростання додавши додаткове відновлення м’язової маси шляхом збільшення розміру м’язового волокна на 0,01.
<pre>
 if nutrition-quality = 5 [
    ifelse growth-change > 0 [
      set fiber-size fiber-size + 0.015
    ]
    [
  set fiber-size fiber-size + 0.01
    ]
  ]
</pre>

### Частина 2
Додати регулюємі користувачем параметри, що визначають вірогідність та міру тимчасового відхилення від вказаних початкових значень інтенсивності тренувань, кількості годин сну та днів між тренуваннями.

Додаємо 6 нових показників та елементів керування до інтерфейсу
<table>
<thead>
<tr><th>Назва параметру</th><th>Значення параметру</th></tr>
</thead>
<tbody>
<tr><td>intensity-deviation-chance</td><td>Імовірність відхилення інтенсивності тренувань</td></tr>
<tr><td>intensity-deviation-range</td><td>Максимальна величина відхилення інтенсивності</td></tr>
<tr><td>sleep-deviation-chance</td><td>Імовірність відхилення тривалості сну</td></tr>
<tr><td>sleep-deviation-range</td><td>Максимальна величина відхилення тривалості сну</td></tr>
<tr><td>workout-deviation-chance</td><td>Імовірність відхилення інтервалу між тренуваннями</td></tr>
<tr><td>workout-deviation-range</td><td>Максимальна величина відхилення інтервалу між тренуваннями</td></tr>
</tbody>
</table>
Для збереження фактичних значень параметрів до блоку globals додано чотири змінні. Вони використовуються для розрахунку тимчасових відхилень без зміни початкових налаштувань користувача, а також для визначення часу наступного тренування.
<pre>
current-intensity
current-sleep
current-workout-interval
next-workout-tick
</pre>
Внесені зміни до процедури setup
<pre>
set current-intensity intensity
set current-sleep hours-of-sleep
set current-workout-interval max (list 1 days-between-workouts)
set next-workout-tick 0
</pre>
Реалізація відхилень інтенсивності тренувань.

Додамо процедуру update-intensity, яка перед кожним тренуванням перевіряє ймовірність виникнення відхилення. Якщо відхилення відбувається, фактична інтенсивність змінюється на випадкову величину в межах установленого діапазону. Отримане значення обмежується інтервалом від 0 до 100%.
<pre>
to update-intensity
  set current-intensity intensity
  if random-float 100 < intensity-deviation-chance [
    let deviation (random-float (2 * intensity-deviation-range)) - intensity-deviation-range
    set current-intensity intensity + deviation
  ]
  set current-intensity max (list 0 (min (list 100 current-intensity)))
end
</pre>
Зміна у процедурі lift-weights
<pre>
current-intensity / 100 * current-intensity / 100
</pre>
Реалізація відхилень тривалості сну.

Додамо процедуру update-sleep, яка на кожному такті моделювання визначає фактичну тривалість сну. Залежно від заданої ймовірності вона може випадково збільшуватися або зменшуватися в межах установленого діапазону. Розраховане значення використовується для визначення впливу сну на гормональне середовище.
<pre>
to update-sleep
  set current-sleep hours-of-sleep
  if random-float 100 < sleep-deviation-chance [
    let deviation (random-float (2 * sleep-deviation-range)) - sleep-deviation-range
    set current-sleep hours-of-sleep + deviation
  ]
  set current-sleep max (list 0 (min (list 24 current-sleep)))
end
</pre>
Зміна у процедурі sleep.
<pre>
ask patches [
    set catabolic-hormone catabolic-hormone - 0.5 * (log catabolic-hormone 10) * current-sleep
    set anabolic-hormone anabolic-hormone - 0.48 * (log anabolic-hormone 10) * current-sleep
  ]
</pre>
Реалізація відхилень інтервалів між тренуваннями.

Для відображення нерегулярності тренувань додано процедуру update-workout-interval. Після кожного тренування вона визначає інтервал до наступного, враховуючи ймовірність і максимальну величину випадкового відхилення. Мінімальна тривалість інтервалу обмежена одним днем. Додаємо процедуру update-workout-interval.
<pre>
to update-workout-interval
  set current-workout-interval days-between-workouts
  if random-float 100 < workout-deviation-chance [
    let deviation (random (2 * workout-deviation-range + 1)) - workout-deviation-range
    set current-workout-interval days-between-workouts + deviation
  ]
  set current-workout-interval max (list 1 current-workout-interval)
end
</pre>
Змінено механізм планування тренувань. Трохи змінюємо процедуру go. Це дозволить змінювати інтервали між окремими тренуваннями.
<pre>
to go
  perform-daily-activity
  update-sleep
   if lift? and ticks >= next-workout-tick [
    update-intensity
    lift-weights
    update-workout-interval
    set next-workout-tick ticks + current-workout-interval
  ]
  sleep
  regulate-hormones
  develop-muscle
  set muscle-mass sum [fiber-size] of muscle-fibers
  tick
end
</pre>
Після цього додамо кілька моніторів для відстеження поточних станів.
- current-intensity Поточна інтенсивність
- current-sleep Поточна кількість сну
- current-workout-interval Поточний інтервал між тренуваннями
- next-workout-tick Кількість тактів, що залишилося до наступного тренування

![Скріншот моделі в процесі симуляції](photo1.png)

<br>








### Внесені зміни у вихідну логіку моделі, на власний розсуд:

**Додано ймовірність безвідповідальності водія, який, при нагоді, їздитиме узбіччям**.
Імовірність встановлюється користувачем через інтерфейс середовища моделювання (слайдер для bad-driver-probability) та використовується при додаванні машин на полі:
<pre>
globals [
  sample-car
  speed-min
  car-roadside-amount
  bad-driver-probability
]

;; виїзд на узбіччя здійснюється, якщо перед машиною є інша машина і ще одна, водій "поганий", знаходиться на дорозі (бо з узбіччя з'їжджати далі нікуди)
if (car-ahead-1 != nobody) [
    ;; перевіряємо, чи буде на даному тіку водій "недисциплінованим"
    let rand random 100
    ifelse(rand < bad-driver-probability) [
      ;; якщо недисциплінований, і при цьому є додаткова перешкода перед машинойї перед нами, то переміщуємось на узбіччя
      if(car-ahead-2 != nobody) [
        ;; перевірка, чи знаходимось ми зараз на дорозі і чи вільне узбіччя
        if (is-on-road) and (roadside-free-check) [
          set heading 180
          fd 1
          set heading 90
          ;; збільшуємо лічильник "узбічників"
          set car-roadside-amount car-roadside-amount + 1
        ]
      ]
    ]
  ]
</pre>
Лічильник "поганих водіїв" виводитиметься користувачеві.

Перед виїздом на узбіччя, воне проглядається водієм, чи вільно там, за допомогою функції is-roadside-free:
<pre>
to-report roadside-free-check
  set heading 180
  let car-roadside one-of turtles-on patch-ahead 1
  set heading 90
  ;ifelse(car-roadside = nobody) [
    report car-roadside = nobody
  ;;]
  ;;[
  ;;  report false
  ;;]
end
</pre>

Пеервірка, чи знаходимось на дорозі:
<pre>
to-report is-on-road
    report pycor = 0
end
</pre>

Додано узбіччя, дорога зведена до повноційної односмугової:
<pre>
to setup-road ;; patch procedure
  if pycor < 1 and pycor > -2 [ set pcolor yellow ]
  if pycor < 1 and pycor > -1 [ set pcolor white ]
end
</pre>

На кожному ході виконується перевірка, чи знаходиться машина на дорозі. Якщо машина їде узбіччям, то намагатиметься повернутися на дорогу:
<pre>
  if(not is-on-road) [
      if (road-main-free-check) [
        set heading 0
        fd 1
        set heading 90
        set car-roadside-amount car-roadside-amount - 1
      ]
    ]
</pre>
Функція перевірки, чи вільна дорога поблизу машини, щоб можна було повернутися з узбіччя:
<pre>
to-report road-main-free-check
  set heading 0
  let car-road one-of turtles-on patch-ahead 1
  set heading 90
  ;ifelse(car-roadside = nobody) [
    report car-road = nobody
  ;;]
  ;;[
  ;;  report false
  ;;]
end
</pre>

![Скріншот моделі в процесі симуляції](example-model.png)

Фінальний код моделі та її інтерфейс доступні за [посиланням](example-model.nlogo). *// якщо вносили зміни до інтерфейсу середовища моделювання - то експорт потрібен у форматі nlogo, як тут. Інакше, якщо змінювався лише код логіки моделі, достатньо викласти лише його, як [тут](example-model-code.html),якщо експортовано з десктопної версії NetLogo, або окремим текстовим файлом, шляхом копіпасту з веб-версії*.
<br>

## Обчислювальні експерименти
*// тут повинен бути наведений опис одного експерименту, за аналогією з першої л/р.* 
### 1. Вплив дисциплінованості водіів на середню швидкість переміщення
