# Координати

Для того, щоб переміщувати елементи на екрані, ми повинні познайомитися з системи координат.

Більшість відповідних методів JavaScript працюють з однією з двох систем координат:

1. **Відносно вікна браузера** - схоже на `position:fixed`, обчислюється від верхнього лівого кута вікна браузера.
   - позначимо ці координати як `clientX/clientY`, аргументація такої назви стане зрозуміла пізніше, коли ми вивчимо властивості події.
2. **Відносно документа** - схоже на `position:absolute` у корені документа, обчислюється від верхнього лівого кута документа.
   - позначимо їх як `pageX/pageY`.

Коли сторінка прокручена до самого початку, то верхній лівий кут вікна точно збігається з верхнім лівим кутом документа, тому їх системи координат також збігаються. Але якщо документ прокрутити, то координати елементів відносно вікна змінюються, а координати, відносно документа, залишаються сталими.

На цьому малюнку ми беремо точку в документі та демонструємо її координати перед прокруткою (ліворуч) і після неї (праворуч):

![](document-and-window-coordinates-scrolled.svg)

Після прокрутки документа:
- `pageY` - координата відносно документа залишилася незмінною, оскільки вона відраховується від верхнього краю документа (який зараз прокручений вгору).
- `clientY` - координата відносно вікна змінилася (стрілка стала коротшою), оскільки та сама точка стала ближче до вершини вікна.

## Координати елемента відносно вікна: getBoundingClientRect

Метод `elem.getBoundingClientRect()` повертає координати у контексті вікна для мінімального за розмірами прямокутника, який вміщує `elem` у вигляді об’єкта вбудованого класу [DOMRect](https://www.w3.org/TR/geometry-1/#domrect).

Основні властивості `DOMRect`:

- `x/y` -- координати X/Y початку прямокутника відносно вікна,
- `width/height` -- ширина/висота прямокутника (можуть бути від’ємними).

Крім того, в об’єкті містяться такі похідні властивості:

- `top/bottom` -- Y-координата для верхнього/нижнього краю прямокутника,
- `left/right` -- X-координата для лівого/правого краю прямокутника.

```online
Наприклад, натисніть на цю кнопку, щоб побачити ЇЇ координати відносно вікна:

<p><input id="brTest" type="button" value="Отримати координати цієї кнопки за допомогою button.getBoundingClientRect()" onclick='showRect(this)'/></p>

<script>
function showRect(elem) {
  let r = elem.getBoundingClientRect();
  alert(`x:${r.x}
y:${r.y}
width:${r.width}
height:${r.height}
top:${r.top}
bottom:${r.bottom}
left:${r.left}
right:${r.right}
`);
}
</script>

Якщо ви прокрутите сторінку та спробуєте знову, то помітите, що по мірі зміни положення кнопки, змінюються і ЇЇ координати відносно вікна (`y/top/bottom`, при вертикальній прокрутці).
```

Ось зображення з результатом виклику `elem.getBoundingClientRect()`:

![](coordinates.svg)

Як бачите, `x/y` та `width/height` повністю описують прямокутник. З них можна легко обчислити похідні властивості:

- `left = x`
- `top = y`
- `right = x + width`
- `bottom = y + height`

Зверніть увагу:

- Координати можуть бути десятковими дробами, наприклад `10.5`. Це нормально, тому, що внутрішньо браузер використовує дроби у своїх обчисленнях. Нам не потрібно їх округлювати, коли встановлюємо значення `style.left/top`.
– Координати можуть бути від’ємними. Наприклад, якщо сторінка прокручена таким чином, що `elem` знаходиться над видимою частиною вікна, то `elem.getBoundingClientRect().top` буде від’ємними.

```smart header="Навіщо потрібні похідні властивості? Чому існує `top/left`, якщо є `x/y`?"
Математично прямокутник однозначно визначається його початковою точкою `(x,y)` і вектором напрямку `(width,height)`. Тому додаткові похідні властивості призначені для зручності.

Технічно можливо, щоб `width/height` були від’ємними, це дозволяє використовувати «спрямований» прямокутник, наприклад для представлення виділення мишею з правильно позначеними початком і кінцем.

Від’ємні значення `width/height` означають, що прямокутник починається з нижнього правого кута, а потім "зростає" ліворуч вгору.

Ось прямокутник із від’ємними значеннями `width` і `height` (наприклад, `width=-200`, `height=-100`):

![](coordinates-negative.svg)

Як бачите, у такому випадку, `left/top` не дорівнює `x/y`.

Однак на практиці `elem.getBoundingClientRect()` завжди повертає позитивні значення ширини/висоти, тут ми згадуємо про негативні значення `width/height` лише для того, щоб ви зрозуміли, чому ці, здавалося б, повторювані властивості насправді не є повторюваними.
```

```warn header="Internet Explorer: немає підтримки `x/y`"
Internet Explorer не підтримує властивості `x/y` з історичних причин.

Тож ми можемо або створити поліфіл (додати гетери в `DomRect.prototype`), або просто використовувати `top/left`, оскільки вони завжди дорівнюють `x/y` для додатніх `width/height`, зокрема в результаті виклику `elem.getBoundingClientRect()`.
```

```warn header="Координати right/bottom відрізняються від однойменних властивостей CSS"
Існує очевидна подібність між координатами відносними до вікна, та CSS `position:fixed`.

Але в CSS властивість `right` означає відстань від правого краю, а властивість `bottom` означає відстань від нижнього краю.

Якщо подивимося на малюнок вище, то побачимо, що в JavaScript це не так. Усі координати вікна відраховуються від верхнього лівого кута, включаючи `right` та `bottom`.
```

## elementFromPoint(x, y) [#elementFromPoint]

Виклик `document.elementFromPoint(x, y)` повертає найбільш вкладений елемент вікна з координатами `(x, y)`.

Синтаксис такий:

```js
let elem = document.elementFromPoint(x, y);
```

Наприклад, наведений нижче код виділяє та виводить тег елемента, який зараз знаходиться в середині вікна:

```js run
let centerX = document.documentElement.clientWidth / 2;
let centerY = document.documentElement.clientHeight / 2;

let elem = document.elementFromPoint(centerX, centerY);

elem.style.background = "red";
alert(elem.tagName);
```

Оскільки код використовує координати відносно вікна, то елемент може відрізнятися залежно від поточної позиції прокручування.

````warn header="Для координат, які знаходяться поза вікном `elementFromPoint` повертає `null`"
Метод `document.elementFromPoint(x,y)` працює, лише якщо координати `(x,y)` знаходяться у видимій області вікна.

Якщо будь-яка з координат від’ємна, або більша ніж ширина/висота вікна, то повертається `null`.

Ось типова помилка, яка може виникнути, якщо не додати перевірку:

```js
let elem = document.elementFromPoint(x, y);
// якщо координати виходять за межі вікна, то elem = null
*!*
elem.style.background = ''; // Помилка!
*/!*
```
````

## Використання разом з "position:fixed"

Найчастіше нам потрібні координати, щоб щось розташувати.

Щоб показати щось поблизу елемента, ми можемо використати `getBoundingClientRect` для отримання його координат, а потім CSS `position` разом із `left/top` (або `right/bottom`).

Наприклад, наведена нижче функція `createMessageUnder(elem, html)` виводить повідомлення під елементом `elem`:

```js
let elem = document.getElementById("coords-show-mark");

function createMessageUnder(elem, html) {
  // створюємо елемент повідомлення
  let message = document.createElement('div');
  // тут краще було б використати CSS клас
  message.style.cssText = "position:fixed; color: red";

*!*
  // призначаємо координати, не забуваємо про "px"!
  let coords = elem.getBoundingClientRect();

  message.style.left = coords.left + "px";
  message.style.top = coords.bottom + "px";
*/!*

  message.innerHTML = html;

  return message;
}

// Використання:
// додаємо повідомлення у документ на 5 секунд 
let message = createMessageUnder(elem, 'Привіт, світ!');
document.body.append(message);
setTimeout(() => message.remove(), 5000);
```

```online
Натисніть кнопку, щоб запустити приклад:

<button id="coords-show-mark">Кнопка з id="coords-show-mark", під нею з’явиться повідомлення</button>
```

Код можна змінити, щоб показати повідомлення ліворуч, праворуч, знизу, застосувати CSS анімацію, і так далі. Це легко, оскільки у нас є всі координати та розміри елемента.

Але зверніть увагу на важливу деталь: коли сторінка прокручується, повідомлення відпливає від кнопки.

Причина очевидна: елемент повідомлення позиціонується за допомогою `position:fixed`, тому він залишається на тому ж місці у вікні, під час прокрутки сторінки.

Щоб це змінити, нам потрібно використовувати систему координат відносно документа та `position:absolute`.

## Координати відносно документа [#getCoords]

Координати, відносні до документа, починаються з верхнього лівого кута документа, а не вікна.

У CSS координати відносно вікна відповідають `position:fixed`, тоді як координати відносно документа подібні до `position:absolute` на верхньому рівні вкладеності.

Ми можемо використовувати `position:absolute` і `top/left`, щоб розмістити щось у певному місці документа таким чином, щоб воно залишалося там навіть під час прокручування сторінки. Але спочатку нам потрібно отримати правильні координати.

Не існує стандартного методу для отримання координат елемента відносно до документа. Але написати його легко.

Дві системи координат з’єднуються за формулою:
- `pageY` = `clientY` + висота прокрученої вертикальної частини документа.
- `pageX` = `clientX` + ширина прокрученої горизонтальної частини документа.

Функція `getCoords(elem)` візьме координати вікна з `elem.getBoundingClientRect()` і додасть до них значення поточної прокрутки:

```js
// отримуємо координати елемента відносно документа
function getCoords(elem) {
  let box = elem.getBoundingClientRect();

  return {
    top: box.top + window.pageYOffset,
    right: box.right + window.pageXOffset,
    bottom: box.bottom + window.pageYOffset,
    left: box.left + window.pageXOffset
  };
}
```

Якщо б у наведеному вище прикладі ми використовували його разом з `position:absolute`, то повідомлення залишалося б біля елемента під час прокручування.

Модифікована функція `createMessageUnder`:

```js
function createMessageUnder(elem, html) {
  let message = document.createElement('div');
  message.style.cssText = "*!*position:absolute*/!*; color: red";

  let coords = *!*getCoords(elem);*/!*

  message.style.left = coords.left + "px";
  message.style.top = coords.bottom + "px";

  message.innerHTML = html;

  return message;
}
```

## Підсумки

Будь-яка точка на сторінці має координати:

1. Відносно вікна -- `elem.getBoundingClientRect()`.
2. Відносно документа -- `elem.getBoundingClientRect()` плюс значення поточної прокрутки.

Координати відносно вікна зручно використовувати з `position:fixed`, а координати відносно документа з `position:absolute`.

Обидві системи координат мають свої плюси і мінуси. Іноді нам потрібна та чи інша, так само я і в CSS ми обираємо CSS `position` між `absolute` та `fixed`.
