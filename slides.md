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

# Три категории фич с разной ценой

Процент на caniuse показывает, у скольких людей браузер уже знает метод. В проекте смотрим на другое: какой файл сборка отдаст посетителю. Она смотрит список браузеров в `.browserslistrc` и год JavaScript в поле `target`.

- **Синтаксис — новые знаки.** Пишем `name ?? 'гость'`: если имени нет, подставить слово «гость». Старый браузер знаков `??` не понимает, и сборка ставит на их место обычную проверку в файле приложения. Отдельный файл из‑за этого не появляется, но файл с кодом становится чуть длиннее.
- **Метод — новый вызов.** `items.toSorted()` остаётся в коде как написали. Если браузер из списка метода не знает, сборка кладёт готовую функцию в отдельный файл — полифил. Этот файл скачивают все посетители: сборка заранее не знает, кто откроет сайт.
- **Возможность браузера.** `new AbortController()` отменяет запрос. Для сборки это обычный вызов: она его не переписывает и готовую функцию не подставляет. Если у браузера из списка такой возможности нет, строка падает.

<style>
.slidev-layout.thesis-slide p,
.slidev-layout.thesis-slide li {
  font-size: 16px;
  line-height: 1.4;
}
.slidev-layout.thesis-slide ul {
  margin-top: 0.6rem;
}
</style>

---
class: thesis-slide build-slide
---

# Пример сборки и размеров

| Что открываем | Что там стоит |
| --- | --- |
| `.browserslistrc` | `defaults` |
| Поле `target` в `tsconfig.app.json` | `es2023` |

<img src="/build-output.png" alt="Вывод сборки: файл полифилов 11,94 КБ gzip, код приложения 68,78 КБ gzip" style="max-height: 96px; margin: 0.15rem 0" />

`defaults` — для каких браузеров собираем сайт. `es2023` — какого года JavaScript попадёт в готовый файл. На снимке `toSorted` ещё нет. Если его добавить, файл полифилов вырастет с 11,94 КБ до 12,09 КБ, на 0,15 КБ. Код приложения останется 68,78 КБ.

Дальше следуют четыре ответа, все для этого списка. **Используем** — браузеры уже умеют, пишем как есть. **С полифилом** — берём в код, готовая функция ляжет в общий файл, его скачают все. **Не рекомендуется** — браузер умеет, но в обычном коде так лучше не писать. **Ждём** — в списке метода ещё нет, и когда браузеры догонят, ответ станет «используем».

<style>
.slidev-layout.build-slide p {
  font-size: 14px;
  line-height: 1.32;
  margin: 0.22rem 0;
  text-align: left;
}
.slidev-layout.build-slide table {
  font-size: 15px;
  margin: 0.2rem 0;
}
</style>

---
class: thesis-slide year-slide
---

# ES2016–2019 — уже все используют

| Фича | Статус | В браузерах |
| --- | --- | --- |
| `**`, `Array.includes` | используем | с 2016–2017 |
| `Object.values` / `entries`, `padStart` | используем | с 2017 |
| `async` / `await`, rest/spread в объектах | используем | с 2017–2018 |
| `Promise.finally`, `flat` / `flatMap` | используем | с 2018–2019 |
| `Object.fromEntries` | используем | с 2019 |

Эти возможности уже давно есть во всех обычных браузерах. При нашем списке `defaults` их можно писать спокойно. `Object.fromEntries` делает обратное `Object.entries`: из списка пар «ключ — значение» снова собирает объект.

<style>
.slidev-layout.year-slide table {
  font-size: 14px;
  margin-top: 0.6rem;
}
.slidev-layout.year-slide p {
  text-align: left;
  font-size: 16px;
  line-height: 1.4;
}
</style>

---
class: thesis-slide year-slide span-years
---

# ES2020–2022 — уже можно использовать

| Фича | Статус | В браузерах |
| --- | --- | --- |
| `?.`, `??`, `1_000`, `&&=`, `??=` | используем | с 2020–2021 |
| `allSettled`, `any`, `import()`, `globalThis`, `BigInt` | используем | с 2018–2021 |
| `replaceAll`, `at`, `hasOwn`, поля класса, `#private`, cause у `Error` | используем | с 2020–2022 |
| `WeakRef`, `FinalizationRegistry` | не рекомендуется | с 2021 |
| top-level `await` | не рекомендуется | Safari с сент. 2026 |

Почти всё из таблицы браузеры из списка уже умеют. Если в `target` указан старый год, сборка заменяет `?.` и `??` обычной проверкой в файле приложения, отдельный полифил не появляется. В числе `1_000` подчёркивание только для глаза: для JavaScript это обычная тысяча.

`WeakRef` держит объект так, что сборщик мусора может забрать его между двумя строками. `FinalizationRegistry` зовёт функцию уже после удаления. В обычном коде так не пишем.

Обычный `await` пишут внутри функции. Top-level `await` — тот же `await` прямо в файле, рядом с `import`: `const data = await fetch('/config')`. Файл не отдаст то, что из него импортируют, пока запрос не закончится. Один медленный запрос задерживает старт приложения, а если два файла ждут друг друга, загрузка может зависнуть. Так пишем только в точке входа или в скриптах на Node. Во всех основных браузерах это с сентября 2026, в нашем списке Safari ещё старее.

<style>
.slidev-layout.span-years table {
  font-size: 13px;
  margin-top: 0.1rem;
}
.slidev-layout.span-years th,
.slidev-layout.span-years td {
  padding-top: 0.18rem !important;
  padding-bottom: 0.18rem !important;
}
.slidev-layout.span-years p {
  text-align: left;
  font-size: 13px;
  line-height: 1.3;
  margin: 0.22rem 0;
}
</style>

---
class: thesis-slide year-slide
---

# ES2023

| Фича | Статус | В браузерах |
| --- | --- | --- |
| `toSorted` / `toReversed` / `toSpliced` / `with` | с полифилом | с июля 2023 |
| `findLast` / `findLastIndex` | используем | с 2022 |

`toSorted` и три соседних метода возвращают новый массив и старый не мутируют. В браузерах они с июля 2023, но в нашем `defaults` ещё есть Chrome 109, а методы появились в Chrome 110. Без готовой функции вызов там упадёт, поэтому берём с полифилом.

`findLast` и `findLastIndex` полифил не просят: в том же Chrome 109 они уже есть. В Safari оба пришли в версии 15.4, вместе с `at`.

<style>
.slidev-layout.year-slide table {
  font-size: 14px;
  margin-top: 0.6rem;
}
.slidev-layout.year-slide p {
  text-align: left;
  font-size: 16px;
  line-height: 1.4;
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

| Фича | Статус | В браузерах |
| --- | --- | --- |
| `Object.groupBy` / `Map.groupBy` | с полифилом | с марта 2024 |
| `Promise.withResolvers` | с полифилом | с марта 2024 |

В браузерах с марта 2024. При `defaults` всё ещё нужен полифил: в списке остаются браузеры, которые этих методов не знают. Сейчас — как `groupBy` убирает ручную группировку, а `withResolvers` — сохранение `resolve` снаружи для кнопок.

<style>
.slidev-layout.year-slide table {
  font-size: 14px;
  margin-top: 0.6rem;
}
.slidev-layout.year-slide p {
  text-align: left;
  font-size: 16px;
  line-height: 1.4;
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

| Фича | Статус | В браузерах |
| --- | --- | --- |
| `Set`: `intersection` / `difference` / `union` | с полифилом | с июня 2024 |
| `Promise.try` | с полифилом | с янв. 2025 |
| методы итераторов (`take`, `map`, `toArray`) | ждём | с марта 2025 |

Методы `Set` в стандарт попали в 2025-м, а в браузерах есть с июня 2024. Берём с полифилом. `Promise.try` тоже: функция может бросить ошибку сразу, ещё до промиса, и `try` всё равно передаст её в `catch`.

Методы итераторов пока ждём. `take` берёт первые несколько значений, `map` преобразует каждое, `toArray` собирает список. В нашем списке их ещё нет у свежего Safari и у Firefox. Когда появятся, статус станет «используем».

JSON-файл подключаем так: `import data from './file.json' with { type: 'json' }`. Это запись 2025 года. Сборщик и так умеет её собрать, отдельно решать не нужно.

<style>
.slidev-layout.year-slide table {
  font-size: 14px;
  margin-top: 0.4rem;
}
.slidev-layout.year-slide p {
  text-align: left;
  font-size: 15px;
  line-height: 1.35;
  margin: 0.4rem 0;
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

# ES2026 — стандарт вышел в июне

| Фича | Статус | В браузерах |
| --- | --- | --- |
| `Map.getOrInsert` | с полифилом | с февр. 2026 |
| `Uint8Array.toBase64` / `fromBase64` | с полифилом | с сент. 2025 |
| `Array.fromAsync` | с полифилом | с янв. 2024 |
| `Error.isError` | ждём | Safari умеет частично |

`map.getOrInsert(key, [])` сам создаёт массив, если ключа ещё нет. Та же проверка «нет ключа — создай массив», что у `groupBy`: её стоит разобрать так же. `toBase64` и `fromBase64` переводят байты в base64 напрямую. `btoa` ждёт текст, а не байты, и часть данных портит.

`Array.fromAsync` собирает массив из значений, которые приходят по очереди. В браузерах он с января 2024, в стандарт попал в июне 2026. Полифил около 1,36 КБ — заметно больше, чем у `toSorted`. `Error.isError` узнаёт ошибку даже из другого окна, где `instanceof Error` говорит «нет». В Safari метод не узнаёт часть ошибок страницы, поэтому ждём.

`Math.sumPrecise` складывает числа без хвоста у дроби, `Iterator.concat` соединяет несколько перечислений. Оба ждём: в нашем списке браузеров их ещё нет.

<style>
.slidev-layout.year-slide table {
  font-size: 13px;
  margin-top: 0.15rem;
}
.slidev-layout.year-slide p {
  text-align: left;
  font-size: 13.5px;
  line-height: 1.32;
  margin: 0.28rem 0;
}
</style>

---
class: thesis-slide year-slide
---

# ES2027 — что ждём

В стандарт эти три вещи ещё не вошли, это следующий выпуск. В обычный код их пока не кладём: браузеры из нашего списка ещё не умеют использовать такой код. Когда догонят, статус сможет стать «используем».

- **[`Temporal`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Temporal) — даты, время и часовые пояса.** Старый `Date` для этого неудобен: месяц начинается с нуля, пояса переводят руками. В первом докладе уже разбирали, здесь хватит ссылки.
- **`using` сам закрывает ресурс.** Файл, соединение или подписку не нужно закрывать вручную: когда блок кода закончился, закрытие произойдёт само, даже если внутри вылетела ошибка.
- **`Iterator.zip` идёт сразу по нескольким спискам** и собирает пары: первый элемент с первым, второй со вторым. Цикл с общим индексом для этого писать не нужно.

<style>
.slidev-layout.year-slide p,
.slidev-layout.year-slide li {
  text-align: left;
  font-size: 16px;
  line-height: 1.4;
}
.slidev-layout.year-slide ul {
  margin-top: 0.5rem;
}
</style>

---
class: thesis-slide year-slide boundary-slide
---

# Без полифила до ES2022, с полифилом до ES2025

До ES2022 при нашем `defaults` браузеры уже умеют всё из таблиц, пишем как есть. С ES2023 по ES2025 берём то, что упрощает код или убирает баг — `toSorted`, `groupBy`, методы `Set`. За это платим полифилом, обычно долями килобайта.

Год в названии стандарта не говорит, когда методом уже можно пользоваться. `Array.fromAsync` в браузерах есть с января 2024, а в стандарт попал только в 2026-м. Смотрим, есть ли метод у наших браузеров и сколько стоит полифил.

Что новее ES2025, решаем по одной фиче. Где-то хватит полифила, где-то ждём, пока браузеры догонят.

Регулярные выражения — именованные группы, новые флаги — в этом докладе не разбираем. Для них будет отдельный.

<style>
.slidev-layout.boundary-slide p {
  text-align: left;
  font-size: 16px;
  line-height: 1.4;
  margin: 0.5rem 0;
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
| `Error.isError` | 2026 | Safari частично | +0,14 КБ |
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
