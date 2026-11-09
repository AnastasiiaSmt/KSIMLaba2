## Комп'ютерні системи імітаційного моделювання
## СПм-25-2, **Смотрова Анастасія Олександрівна**
### Лабораторна робота №**2**. Редагування імітаційних моделей у середовищі NetLogo

<br>

### Варіант 12, модель у середовищі NetLogo:
[Muscle Development] http://www.netlogoweb.org/launch#http://www.netlogoweb.org/assets/modelslib/Sample%20Models/Biology/Muscle%20Development.nlogo)

<br>

### Внесені зміни у вихідну логіку моделі, за варіантом:

**Частина 1** Додати у моделі вплив харчування (умовний регулюємий параметр, наприклад, "якість харчування") на можливість зростання м'язової маси.  
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

**Частина 2 ** Додати регулюємі користувачем параметри, що визначають вірогідність та міру тимчасового відхилення від вказаних початкових значень інтенсивності тренувань, кількості годин сну та днів між тренуваннями.

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

<pre>
;; set speed-limit 1
set speed-limit get-random-float 1.1 0.9
</pre>
Використовується кольорова диференціація машин залежно від їхньої швидкісного ліміту:
<pre>
  ;; set label speed-limit
  if(speed-limit > 1.2) [
      set color green
  ]
   if(speed-limit > 1.5) [
     set color red
  ]

  ;; ask sample-car [ set color red ]
</pre>
Сині найповільніші, зелені швидше, червоні найшвидші.  
Забарвлення обраної для відстеження машини скасовано.

**Зміна логіки гальмування та набору швидкості** залежно від наявності перешкоди перед машиною:
<pre>
  let car-ahead-1 one-of turtles-on patch-ahead 1
    let car-ahead-2 one-of turtles-on patch-ahead 2
    ifelse car-ahead-1 = nobody and car-ahead-2 = nobody
      [ speed-up-car ] ;; otherwise, speed up
      [ slow-down-car car-ahead-1 car-ahead-2 ]
    ;; don't slow down below speed minimum or speed up beyond speed limit
    if speed < speed-min [ set speed speed-min ]
    if speed > speed-limit [ set speed speed-limit ]
    fd speed
</pre>
Для цього також були внесені зміни до процедури slow-down-car, яка спрацьовує при гальмуванні:
<pre>
to slow-down-car [ car-ahead-1 car-ahead-2 ] ;; turtle procedure
  if (car-ahead-1 != nobody) 
  [
      let speed-car-ahead-1 [ speed ] of car-ahead-1
      ;; slow down so you are driving more slowly than the car ahead of you
      set speed speed-car-ahead-1 - deceleration
      stop
  ]
  ]  
  if (car-ahead-2 != nobody) [
    let speed-car-ahead-2 [ speed ] of car-ahead-2
    let speed-difference-ahead-2 abs speed - speed-car-ahead-2
    set speed speed - speed-difference-ahead-2 / 2
  ]
end
</pre>

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
