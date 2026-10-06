---
theme: seriph
colorSchema: dark
background: '#32353C'
title: Доклад 2. Что из современного JS мы уже можем использовать
class: text-center
drawings:
  persist: false
transition: slide-left
comark: true
duration: 20min
---

# Что из современного JS мы уже можем использовать

---
class: thesis-slide
---

# Можно ли это использовать?

- **На caniuse.com у метода свой процент.** Это доля людей, чей браузер уже понимает этот метод. У `toSorted` одна цифра, у `Object.groupBy` — другая.
- **На вопрос слайда этот процент не отвечает.** Пользователь скачивает не точно тот файл, который мы написали. Сборка отдаёт другой JavaScript. Как именно он будет выглядеть, решают файл `.browserslistrc` (правило, какие версии браузеров поддерживать) и поле `target` в `tsconfig`.

---
class: thesis-slide
---

# Три категории фич с разной ценой

- **Синтаксис.** Сборка заменяет новую запись на проверку, которую браузер уже понимает.

```js
// что пишем мы
user?.name
// что примерно получит браузер
user == null ? undefined : user.name
```

- **Метод.** Вызов остаётся в файле как есть. Если браузер из списка его не знает, рядом кладут готовую функцию полифил.

```js
items.toSorted()
```

- **Возможность браузера.** Сборка эту строку не меняет. Она либо есть в браузере, либо нет.

```js
new AbortController()
```

<style>
.slidev-layout.thesis-slide .slidev-code {
  font-size: 15px !important;
  line-height: 22px !important;
  margin: 0.35rem 0 0.8rem !important;
}
</style>

---
class: thesis-slide
---

# Синтаксис заменяется проверкой

```js
// что пишем мы
name ?? 'гость'
// что примерно получит браузер
name != null ? name : 'гость'
```

- **Замену решают две настройки.** Список в `.browserslistrc`: есть ли там браузер, который не понимает `??`. Поле `target` в `tsconfig` — это год JavaScript в готовом файле. Если в том году `??` ещё не было, сборка ставит проверку из примера.
- **Сборка заменяет `??` проверкой в файле с кодом приложения.** Браузер её уже понимает, поэтому в файл полифилов `??` не попадает. Проверка длиннее, чем `name ?? 'гость'`, и файл приложения из-за этого чуть больше.

<style>
.slidev-layout.thesis-slide .slidev-code {
  font-size: 15px !important;
  line-height: 22px !important;
  margin: 0.35rem 0 0.8rem !important;
}
</style>

---
class: thesis-slide
---

# Полифил скачивают все

- **Сборка читает `.browserslistrc` до открытия сайта.** Если метода нет у браузера из списка, готовая реализация попадает в отдельный файл.
- **Этот файл скачивают все.** В том числе те, чей браузер метод уже знает. Сам вызов сборка не переписывает, он остаётся в коде приложения.

---
class: thesis-slide
---

# Возможность браузера остаётся как есть

```js
new AbortController()
```

- **Сборка оставляет вызов как есть.** `AbortController` — возможность браузера отменить запрос. Это не проблема распознавания нового синтаксиса: строку `new AbortController()` браузер уже умеет читать, поэтому сборка не меняет её на другую. В файл полифилов функция тоже не попадает.
- **Если у браузера из `.browserslistrc` этой возможности нет, вызов падает.** Сборка готовую функцию не подставит. Число и дату в нужном формате через `Intl` браузер либо умеет показать сам, либо нет.

---
class: thesis-slide
---

# Пример сборки и бандла

| Что открываем | Что там стоит |
| --- | --- |
| `.browserslistrc` | `defaults` |
| Поле `target` в `tsconfig.app.json` | `es2023` |

<img src="/build-output.png" alt="Вывод сборки: файл полифилов 11,94 КБ gzip, код приложения 68,78 КБ gzip" style="max-height: 168px; margin-top: 0.4rem" />

`defaults` — правило, какие версии браузеров поддерживать. `es2023` — год JavaScript в файле с кодом приложения. На снимке `toSorted` ещё нет. С ним файл полифилов 12,09 КБ, на 0,15 КБ больше.

---
class: thesis-slide problem-slide
---

# `sort` мутирует массив

```js
const [items, setItems] = useState(['груша', 'арбуз', 'слива'])

const onSort = () => {
  setItems(items.sort())
}
```

- **После нажатия «Сортировать» порядок должен измениться.**
- **`sort` сортирует этот же массив и возвращает его же.** `setItems` получает тот же массив, React не рисует список заново, и на экране остается старый порядок.

---
class: thesis-slide answer-slide
---

# `toSorted` возвращает новый массив

```js
setItems(items.toSorted())
```

- **Сборка вызов не переписывает.** Если метода нет у браузера из списка, готовая функция попадает в файл полифилов.
- **В бандл это добавляет 0,15 КБ.** Файл полифилов был 11,94 КБ, с вызовом `toSorted` стал 12,09 КБ. Код приложения остаётся 68,78 КБ.
- **Место в коде есть.** В списке «Сортировать» вместо `items.sort()` пишем `items.toSorted()`. Метод возвращает новый массив, React рисует список заново: арбуз, груша, слива.
- **Ещё три метода возвращают новый массив.** `toReversed` для reverse, `toSpliced` для splice и `with` для записи `items[0] = 'яблоко'` в тот же массив.

---
class: thesis-slide problem-slide
---

# Проблема создания категорий вручную

```js
const goods = [
  { name: 'Яблоки', category: 'fruits' }, { name: 'Бананы', category: 'fruits' },
  { name: 'Огурцы', category: 'vegetables' },
]

const grouped = goods.reduce((acc, item) => {
  const key = item.category
  if (!acc[key]) acc[key] = []
  acc[key].push(item)
  return acc
}, {})
```

- **Для новой категории пустой массив создаём сами.** Иначе `acc[key]` пустой, и строка с `push` падает.
- **Вокруг группировки много лишнего кода.** Надо прочитать ключ, проверку и `push` в массив.

<style>
.slidev-layout.thesis-slide .slidev-code {
  font-size: 15px !important;
  line-height: 22px !important;
  margin: 0.35rem 0 0.8rem !important;
}
</style>

---
class: thesis-slide answer-slide
---

# `Object.groupBy` собирает группы сам

```js
const grouped = Object.groupBy(goods, (item) => item.category)

grouped.fruits
```

- **Пустой массив создавать не нужно.** Метод сам кладёт товары одной категории в массив. Вся группировка — одна строка.
- **Ключ — строка.** Функция возвращает `item.category`: для яблок и бананов это `'fruits'`. Если вернуть число, ключом всё равно будет строка.
- **Сборка не переписывает вызов.** Если у браузера из списка нет такого метода, готовая функция попадает в файл полифилов. Файл был 11,94 КБ, стал 12,10 КБ. Место — список товаров в приложении.

---
class: thesis-slide answer-slide
---

# Ключом `Map.groupBy` может быть объект

```js
const fruits = { title: 'Фрукты' }
const vegetables = { title: 'Овощи' }

const grouped = Map.groupBy(goods, (item) => {
  return item.category === 'fruits' ? fruits : vegetables
})

grouped.get(fruits)
```

- **`Object.groupBy` так не умеет.** Его ключ только строка. Яблоки и бананы здесь читаются вызовом `grouped.get(fruits)` по объекту категории.
- **Сборка этот вызов тоже не переписывает.** Если у браузера из списка нет такого метода, готовая функция попадает в файл полифилов. Отдельно это те же 0,16 КБ. Место — те же товары, ключом служит объект.

---
class: thesis-slide problem-slide
---

# Сравнение списков через `filter`

```js
const tagsA = ['react', 'js', 'css']
const tagsB = ['js', 'html', 'css']

tagsA.filter((tag) => tagsB.includes(tag))
tagsA.filter((tag) => !tagsB.includes(tag))
```

- **Первая строка ищет теги из обоих списков.** На каждом теге из `tagsA` заново проходят список `tagsB` целиком.
- **Вторая строка — теги только из первого списка.** Без отрицания она станет копией первой.

---
class: thesis-slide answer-slide
---

# `intersection` и `difference`

```js
const setA = new Set(['react', 'js', 'css'])
const setB = new Set(['js', 'html', 'css'])

setA.intersection(setB)
setA.difference(setB)
```

- **`intersection` оставляет теги из обоих наборов.** Теги `js` и `css` есть в обоих. `difference` оставляет тег `react` из первого набора.
- **Сборка сам вызов не переписывает.** В списке вызовов `intersection` нет. На этой строке есть `new Set`, и вместе с ним метод попадает в файл полифилов. Место — списки тегов в приложении.

---
class: thesis-slide answer-slide
---

# `union` убирает повторы

```js
const listA = [1, 2, 3]
const listB = [3, 4, 5]

// раньше: склеить оба списка, убрать повтор, снова сделать список
const oldUnion = [...new Set([...listA, ...listB])]

new Set(listA).union(new Set(listB))
```

- **Нужен один список, где 3 встречается один раз.** Раньше списки сначала склеивали в 1, 2, 3, 3, 4, 5, потом убирали повтор. Если `Set` в середине забыть, ошибки нет, но 3 остаётся дважды.
- **`union` сразу собирает набор, где 3 встречается один раз.** Сам вызов сборка не узнаёт. Рядом есть `new Set`, поэтому готовая функция всё же попадает в файл полифилов.

---
class: thesis-slide
---

# Методы приезжают вместе с `new Set`

```js
// в коде приложения
new Set(tagsA).intersection(new Set(tagsB))

// если new Set в коде нет — в настройке сборки
additionalModernPolyfills: [
  'core-js/modules/es.set.intersection.v2.js',
  'core-js/modules/es.set.difference.v2.js',
  'core-js/modules/es.set.union.v2.js',
]
```

- **Вызов `intersection` сборка не узнаёт.** Без `new Set` в коде готовой функции нет, и на браузере из списка без метода вызов падает.
- **`new Set` сборка узнаёт.** С ним в файл полифилов попадают эти три метода. В приложении такой вызов уже есть. Если его нет, те же файлы дописывают отдельным списком.

---
class: thesis-slide problem-slide
---

# `resolve` вытаскивают из конструктора

```js
let resolve

const answer = new Promise((res) => {
  resolve = res
})

function onYes() {
  resolve('да')
}

function onNo() {
  resolve('нет')
}
```

- **`resolve('да')` заканчивает ожидание и передаёт строку «да».** Эту строку получает код, который ждёт переменную `answer` и продолжает работу. По кнопке «Нет» туда же уходит строка «нет».
- **Кнопки написаны отдельно и внутрь конструктора не попадают.** Поэтому `resolve` сохраняют снаружи. Если присваивание забыть, ожидание так и висит.

---
class: thesis-slide answer-slide
---

# `withResolvers` отдаёт `resolve` сразу

```js
const { promise: answer, resolve } = Promise.withResolvers()

function onYes() {
  resolve('да')
}

function onNo() {
  resolve('нет')
}
```

- **Кнопка передаёт в ожидание ответ: «да» или «нет».** Внешняя переменная и присваивание внутри конструктора не нужны.
- **Сборка вызов не переписывает.** Если у браузера из списка нет такого метода, готовая функция попадает в файл полифилов. Это +0,14 КБ. Место — кнопки «Да» и «Нет» в приложении.

---
class: thesis-slide problem-slide
---

# Ошибка вылетает мимо `catch`

```js
function start(task) {
  return Promise.resolve(task())
}

start(() => {
  throw new Error('сбой')
}).catch(() => {
  console.log('поймали')
})
```

- **В консоль «поймали» не попадает.** `task` бросает ошибку сразу, `start` падает на этом вызове, промис не создаётся.
- **Обработчик ждёт промис, а его нет.** До `return` функция не дошла.

---
class: thesis-slide answer-slide
---

# `Promise.try` отдаёт ошибку в `catch`

```js
function start(task) {
  return Promise.try(task)
}

start(() => {
  throw new Error('сбой')
}).catch(() => {
  console.log('поймали')
})
```

- **В консоль попадает «поймали».** Ошибка становится отказом промиса. Если `task` вернёт промис, `start` дождётся и его.
- **Сборка вызов не переписывает.** Если у браузера из списка нет такого метода, готовая функция попадает в файл полифилов. Это +0,29 КБ. Место — запуск с ошибкой «сбой» в приложении.

