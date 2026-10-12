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

# Можно ли использовать фичу?

- **На caniuse.com у метода свой процент.** Это доля людей, чей браузер уже понимает этот метод. У `toSorted` одна цифра, у `Object.groupBy` — другая.
- **Но на наш вопрос этот процент не отвечает.** Пользователь скачивает не точно тот js файл, который мы написали. Сборка отдаёт другой JavaScript. Как именно он будет выглядеть, решают две настройки. Это файл `.browserslistrc` со списком версий браузеров и поле `target` в tsconfig.
- **Цена в том, вырастет ли файл, который мы скачиваем.** У разных фич она разная. Далее рассмотрим три варианта, которые могут на это влиять.

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

- **Замену решают две настройки.** Список в `.browserslistrc`: есть ли там браузер, который не понимает `??`. Поле `target` в `tsconfig` — до какого года JavaScript сборке можно упростить наш код. Если в том году `??` ещё не было, сборка ставит проверку из примера.
- **Сборка заменяет `??` проверкой в файле с кодом приложения.** Браузер её уже понимает, поэтому в файл полифилов `??` не попадает. Проверка длиннее, чем `name ?? 'гость'`, и файл приложения из-за этого становится чуть больше.

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

# Файл полифилов скачивают все

- **Сборка читает `.browserslistrc` до открытия сайта.** Если метода нет у браузера из списка, готовая реализация попадает в отдельный файл.
- **Этот файл скачивают все.** В том числе те, чей браузер метод уже знает.
- **Сборка не переписывает вызов.** Он остаётся в коде приложения. Для любого нового метода правило одно и то же.

---
class: thesis-slide
---

# Возможность браузера остаётся как есть

```js
new AbortController()
```

- **Сборка оставляет вызов как есть.** `AbortController` — возможность браузера отменить запрос. Строку `new AbortController()` браузер уже умеет читать как обычный вызов, поэтому сборка не меняет её на другую запись. В файл полифилов функция тоже не попадает.
- **Если у браузера из `.browserslistrc` этой возможности нет, вызов падает.** Сборка готовую функцию не подставит.

---
class: thesis-slide
---

# Пример сборки и размеров

| Что открываем | Что там стоит |
| --- | --- |
| `.browserslistrc` | `defaults` |
| Поле `target` в `tsconfig.app.json` | `es2023` |

<img src="/build-output.png" alt="Вывод сборки: файл полифилов 11,94 КБ gzip, код приложения 68,78 КБ gzip" style="max-height: 168px; margin-top: 0.4rem" />

`defaults` — для каких браузеров собираем сайт. `es2023` — какого года JavaScript попадёт в готовый файл. На снимке `toSorted` ещё нет. Если его добавить, файл полифилов станет 12,09 КБ — на 0,15 КБ больше. Дальше «используем / с полифилом / не берём» считаем именно для такого списка браузеров.

---
class: thesis-slide year-slide
---

# ES2016–2019 — уже все используют

| Фича | Статус | Стандарт | В браузерах |
| --- | --- | --- | --- |
| `**`, `Array.includes` | используем | 2016 | с 2016–2017 |
| `Object.values` / `entries`, `padStart` | используем | 2017 | с 2017 |
| `async` / `await`, rest/spread в объектах | используем | 2018 | с 2017–2018 |
| `Promise.finally`, `flat` / `flatMap` | используем | 2019 | с 2018–2019 |

Эти возможности уже давно есть во всех обычных браузерах. При нашем списке `defaults` их можно писать спокойно — отдельно решать, брать или нет, не нужно.

<style>
.slidev-layout.year-slide table {
  font-size: 14px;
  margin-top: 0.6rem;
}
</style>

---
class: thesis-slide year-slide
---

# ES2020

| Фича | Статус | Стандарт | В браузерах |
| --- | --- | --- | --- |
| `?.`, `??` | используем | 2020 | с 2020 |
| `Promise.allSettled` | используем | 2020 | с 2020 |
| dynamic `import()` | используем | 2020 | с 2018–2020 |
| `globalThis` | используем | 2020 | с 2019–2020 |
| `BigInt` | используем | 2020 | с 2018–2020 |

В стандарт они попали в 2020-м, браузеры почти сразу подхватили — используем. `?.` и `??` — это новые знаки, не метод. Если в `target` указан год, когда их ещё не было, сборка заменит их на обычную проверку в файле приложения. Отдельный файл полифилов из‑за этого не появится — в отличие от `toSorted`.

<style>
.slidev-layout.year-slide table {
  font-size: 14px;
  margin-top: 0.6rem;
}
</style>

---
class: thesis-slide year-slide
---

# ES2021

| Фича | Статус | Стандарт | В браузерах |
| --- | --- | --- | --- |
| `String.replaceAll` | используем | 2021 | с 2020–2021 |
| `Promise.any` | используем | 2021 | с 2021 |
| `&&=`, `??=` и похожие | используем | 2021 | с 2021 |
| `WeakRef`, `FinalizationRegistry` | не берём | 2021 | с 2021 |

`WeakRef` нужен редко и с ним легко ошибиться. В обычном коде проекта его не пишем, даже если браузер уже умеет.

<style>
.slidev-layout.year-slide table {
  font-size: 14px;
  margin-top: 0.6rem;
}
</style>

---
class: thesis-slide year-slide
---

# ES2022

| Фича | Статус | Стандарт | В браузерах |
| --- | --- | --- | --- |
| `Array.at` / `String.at` | используем | 2022 | с 2022 |
| `Object.hasOwn` | используем | 2022 | с 2022 |
| `Error` cause | используем | 2022 | с 2021–2022 |
| top-level `await` | используем | 2022 | с 2021–2022 |
| поля класса, `#private` | используем | 2022 | с 2021–2022 |

И здесь стандарт и поддержка в браузерах идут рядом. При нашем `defaults` всё из таблицы можно брать без оглядки.

<style>
.slidev-layout.year-slide table {
  font-size: 14px;
  margin-top: 0.6rem;
}
</style>

---
class: thesis-slide year-slide
---

# ES2023

| Фича | Статус | Стандарт | В браузерах |
| --- | --- | --- | --- |
| `toSorted` / `toReversed` / `toSpliced` / `with` | только с полифилом | 2023 | с июля 2023 |
| `findLast` / `findLastIndex` | только с полифилом | 2023 | с 2022–2023 |

В браузерах эти методы есть с июля 2023. Но в нашем `defaults` ещё остаются более старые версии — без готовой подмены вызов у них упадёт. Поэтому берём с полифилом. Сейчас разберём `toSorted` и типичный баг с `sort` в React.

<style>
.slidev-layout.year-slide table {
  font-size: 14px;
  margin-top: 0.6rem;
}
</style>

---
class: thesis-slide problem-slide
---

# Метод `sort` мутирует массив

```js
const [items, setItems] = useState(['груша', 'арбуз', 'слива'])

const onSort = () => {
  setItems(items.sort())
}

<button onClick={onSort}>Сортировать</button>
```

- **После нажатия «Сортировать» порядок должен измениться.**
- **`sort` меняет массив на месте и возвращает его же.** `setItems` получает тот же массив, React не перерисовывает список, и на экране остается старый порядок.

---
class: thesis-slide answer-slide
---

# Метод `toSorted` возвращает новый массив

```js
setItems(items.toSorted())
```

- **В файл полифилов это добавляет 0,15 КБ.** Было 11,94 КБ, с вызовом `toSorted` стало 12,09 КБ. Код приложения остаётся 68,78 КБ.
- **По кнопке «Сортировать» список перерисовывается, потому что создаётся новый массив.** Вместо `items.sort()` используем `items.toSorted()` и сразу видим правильный порядок: арбуз, груша, слива.
- **Ещё три метода возвращают новый массив.** `toReversed` для reverse, `toSpliced` для splice и `with` для записи `items[0] = 'яблоко'` в тот же массив.

---
class: thesis-slide year-slide
---

# ES2024

| Фича | Статус | Стандарт | В браузерах |
| --- | --- | --- | --- |
| `Object.groupBy` / `Map.groupBy` | только с полифилом | 2024 | с марта 2024 |
| `Promise.withResolvers` | только с полифилом | 2024 | с марта 2024 |

В браузерах с марта 2024. При `defaults` всё ещё нужен полифил. Сейчас — как `groupBy` упрощает группировку, а `withResolvers` — работу с промисом из кнопок.

<style>
.slidev-layout.year-slide table {
  font-size: 14px;
  margin-top: 0.6rem;
}
</style>

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

- **Вокруг группировки много лишнего кода.** Надо прочитать ключ, проверку и `push` в массив.
- **Для новой категории пустой массив создаём сами.** Иначе `acc[key]` пустой, и строка с `push` падает.

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
- **Ключ — строка.** Функция возвращает `item.category`: для яблок и бананов это `'fruits'`. Если вернуть число, метод преобразует его в строку.
- **В файл полифилов это +0,16 КБ.** Было 11,94 КБ, стало 12,10 КБ.

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
- **Отдельно это те же +0,16 КБ** в файл полифилов. Ключом служит объект.

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
- **В файл полифилов это +0,14 КБ.**

---
class: thesis-slide year-slide
---

# ES2025

| Фича | Статус | Стандарт | В браузерах |
| --- | --- | --- | --- |
| `Set`: `intersection` / `difference` / `union` | только с полифилом | 2025 | с июня 2024 |
| `Promise.try` | только с полифилом | 2025 | с янв. 2025, ещё свежий |
| `Error.isError` | не берём | 2025 | не во всех браузерах |
| методы итераторов (`take`, `map`, `toArray`) | не берём | 2025 | с марта 2025, ещё свежий |

В стандарт методы `Set` попали в 2025-м, а в браузерах уже с июня 2024. `Promise.try` можно брать с полифилом. `Error.isError` и методы итераторов пока не берём: поддержка ещё неровная или задача слишком редкая. Ниже разберём только `Set`.

<style>
.slidev-layout.year-slide table {
  font-size: 13px;
  margin-top: 0.5rem;
}
</style>

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

- **Первая строка оставляет теги `js` и `css` из обоих списков.**
- **Вторая строка оставляет тег `react` из первого списка.**

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
- **`union` сразу собирает набор, где 3 встречается один раз.**

---
class: thesis-slide
---

# С `new Set` в файл попадают методы

```js
// вызов, который мы пишем
new Set(tagsA).intersection(new Set(tagsB))

// если new Set в коде нет — настройка сборки
additionalModernPolyfills: [
  'core-js/modules/es.set.intersection.v2.js',
  'core-js/modules/es.set.difference.v2.js',
  'core-js/modules/es.set.union.v2.js',
]
```

- **Слово `intersection` или `union` само по себе для сборки не повод класть полифил.** Без `new Set` кода метода в файле нет, и на браузере из списка без метода вызов падает.
- **`new Set` сборка видит** — и вместе с ним кладёт `intersection`, `difference` и `union`. Если `new Set` в коде нет, эти три файла дописывают в настройке сборки.

---
class: thesis-slide year-slide
---

# Граница «можно брать» ≈ ES2025

Всё до ES2022 при нашем `defaults` уже можно писать без отдельного решения. С ES2023 по ES2025 имеет смысл брать то, что упрощает код или убирает баг — как `toSorted`, `groupBy` или методы `Set`. За это почти всегда платим полифилом, обычно долями килобайта.

Узкие вещи из ES2025 вроде `Error.isError` и методов итераторов в общий код пока не кладём: поддержка ещё свежая или задача слишком редкая.

И отдельно: год в названии стандарта не говорит, когда методом уже можно пользоваться. `Array.fromAsync` в стандарте числится как ES2026, а в браузерах есть с января 2024. Полифил у него заметнее — около 1,36 КБ. Смотрим, есть ли метод у наших браузеров и сколько он стоит, а не цифру года на этикетке.

<style>
.slidev-layout.year-slide p {
  text-align: left;
  font-size: 15px;
  line-height: 1.45;
  margin: 0.55rem 0;
}
</style>

---
class: thesis-slide ref-slide
---

# Справка: таблица фич

| Фича | Стандарт | В браузерах | Полифил (gzip) |
| --- | --- | --- | --- |
| `toSorted` | 2023 | июл 2023 | +0,15 КБ |
| `Object.groupBy` / `Map.groupBy` | 2024 | мар 2024 | +0,16 КБ |
| `Promise.withResolvers` | 2024 | мар 2024 | +0,14 КБ |
| `Set` intersection / difference / union | 2025 | июн 2024 | с `new Set` |
| `Promise.try` | 2025 | янв 2025 | +0,29 КБ |
| `Error.isError` | 2025 | не везде | +0,14 КБ |
| `Array.fromAsync` | 2026 | янв 2024 | +1,36 КБ |
| iterator `take` / `map` / `toArray` | 2025 | мар 2025 | +0,32 КБ |

<style>
.slidev-layout.ref-slide table {
  font-size: 12px;
  margin-top: 0.4rem;
}
</style>

---
class: thesis-slide ref-slide
---

# Справка: как проверить самому

1. **MDN** — на странице метода смотрим, с какой даты он есть во всех основных браузерах.
2. **caniuse.com** — процент и конкретные версии. Процент сам по себе не отвечает, можно ли метод нам.
3. **`npx browserslist`** — какие браузеры реально входят в ваш `.browserslistrc` (у нас это `defaults`).

---
class: thesis-slide ref-slide
---

# Справка: закрепить в ESLint

- **`eslint-plugin-es-x`** — не даст написать синтаксис или API новее той версии ES, на которой договорились остановиться.
- **`eslint-plugin-compat`** — сверит вызов с вашим списком браузеров и подсветит то, чего у кого-то из списка ещё нет (если полифил не подключён).

