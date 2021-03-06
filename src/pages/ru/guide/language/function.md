---
title: Функция
order: 100
---

Вы можете поверить, что мы до сих пор не касались функций?

Объявление функции начинается с `fun` и возвращает выражение.

```reason
let greet = fun name => "Hello " ^ name;
```

Это создает функцию и привязывает ее к имени `greet`. Вызывается она так:

```reason
greet "world!"; /* "Hello world!" */
```

Аргументы разделяются пробелами:

```reason
let add = fun x y z => x + y + z;
add 1 2 3; /* 6 */
```

Тело функции оборачивается в фигурные скобки:

```reason
let greetMore = fun name => {
  let part1 = "Hello";
  part1 ^ " " ^ name
};
```

**Так как объявление функций встречается часто**, существует сокращение `let + fun`:

```reason
let add x y z => x + y + z;
/* тоде самое: let add = fun x y z => x + y + z; */
```

**Обращайте внимание на порядок вызова**! В некоторых ситуациях нужно оборачивать вызовы в
скобки:

```reason
let increment x => x + 1;
let double x => x + x;

let eleven = increment (double 5);
```

Если забудете обернуть `double 5` в скобки, то получите `increment double 5`, как будто
`increment` принимает два аргумента.

### Без аргументов

Функции в Reason всегда принимают аргумент. Но иногда вам не нужно ничего передавать. В других
языках вы просто не передаете аргумент. В Reason вы передаете значение `()`, называемое
"юнит (unit)".

```reason
/* функция, которая получает юнит аргумент */
let logSomething () => {
  print_endline "hello";
  print_endline "world";
};

/* вызов функции с аргументом юнитом */
logSomething ();
```

`()` это совершенно нормальное значение типа `unit`. Reason использует специальный синтаксис для
удобства.

### Именованные аргументы

При вызове функций с несколькими аргументами (особенно если они одного типа), легко запутаться.

```reason
let addCoordinates x y => {
  /* x и y используются здесь */
};
...
addCoordinates 5 6; /* какой из них x, а какой y? */
```

В OCaml/Reason, вы можете давать метки аргументам:

```reason
let addCoordinates x::x y::y => {
  /* x и y используются здесь */
};
...
addCoordinates x::5 y::6;
```

Аргументы можно передавать в любом порядке

```reason
addCoordinates y::6 x::5;
```

Запись `x::x` означает, что функция принимает аргумент `x` и внутри может ссылаться на
переменную `x` со значением этого аргумента. Это можно использовать для переименования аргумента
внутри функции

```reason
let drawCircle radius::r color::c => {
  setColor c;
  startAt r r;
  ...
};

drawCircle radius::10 color::"red";
```

Для частого случая `radius::radius` (когда метка и локальное имя совпадают), существует
специальный синтаксис `::x`:

```reason
let drawCircle ::radius ::color => {
  setColor color;
  startAt radius radius;
  ...
}
```

А вот так можно указать типы:

```reason
let drawCircle radius::(r: int) color::(c: string) => ...;
```

### Необязательные именованные аргументы

Именованные аргументы можно пометить как необязательные. Такие аргументы можно пропустить при
вызове.

```reason
/* радиус можно пропустить */
let drawCircle ::color ::radius=? () => {
  setColor color;
  switch radius {
  | None => startAt 1 1;
  | Some r_ => startAt r_ r_;
  }
};
```

В таком случае переменная `radius` **обернута** в тип `option`, со значением по умолчанию
равным `None`. Если аргумент будет предоставлен, то он будет обернут в `Some`. Потому тип
`radius` или `None` или `Some int`

**Важно**: `None | Some foo` это структура данных, описанная [выше](../../guide/language/variant).
Конкретно этот вариант предоставлен стандартной библиотекой и называется `option`.
Его определение: `type option 'a = None | Some 'a`.

**Важно** что юнит `()` находится в конце `drawCircle`. Так как `radius` и `color` являются
именованными, то из-за каррирования (про это далее) не ясно, что означает следующее:

```reason
let whatIsThis = drawCircle ::color;
```

Является ли `whatIsThis` каррированной версией `drawCircle` ожидающей аргумента `radius`?
Или вызов завершен и это результат? Для того, чтобы избежать путаницы нужно добавить
неименованный аргумент и OCaml будет считать, что именованный аргумент был пропущен.

```reason
let curriedFunction = drawCircle ::color;
let actualResultWithoutProvidingRadius = drawCircle ::color ();
```

#### Явно переданные необязательные аргументы

Иногда вам нужно передать значение я не зная является оно `None` или `Some a`. Наивный вариант
такой:

```reason
let result = switch payloadRadius {
| None => drawCircle ::color ()
| Some r => drawCircle ::color radius::r ()
};
```

Но это быстро надоедает. Потому существует короткая запись.

```reason
let result = drawCircle ::color radius::?payloadRadius ();
```

Это значит "Я понимаю, что `radius` необязателен и когда я передаю значение я не знаю, что
именно я передаю: `None` или `Some val`, потому передаю весь `option`"

#### Необязательный аргумент со значением по умолчанию

Можно указать значение по умолчанию для необязательного аргумента. Такие аргументы не
оборачиваются в `option`.

```reason
let drawCircle ::radius=1 ::color () => {
  setColor color;
  startAt r r;
};
```

#### Рекурсивные функции

По умолчанию значения не могут видеть имена к которым привязаны. Но можно обойти это, используя
ключевое слово `rec` в `let`. Это позволяет функциям видеть самих себя, давай всю мощь рекурсии.

```reason
let rec neverTerminate = fun () => neverTerminate ();
```

#### Взаимно рекурсивные функции

Такие функции объявляются как и обычные, с `rec`, но связываются словом `and`:

```reason
let rec callSecond = fun () => callFirst ()
and callFirst = fun () => callSecond ();
```

**Важно** что нет точки запятой после первой строки в `let`. Точка с запятой идут после второй
строки.

#### Каррирование

Reason позволяет **частично** вызывать функции:

```reason
let add = fun x y => x + y;
let addFive = add 5;
let eleven = addFive 6;
let twelve = addFive 7;
```

На самом деле, функция `add` всего лишь синтаксический сахар для слещующего кода:

```reason
let add = fun x => fun y => x + y;
```

OCaml по возможности устраняет не нужное выделение памяти (тут две функции)!
Таким образом мы получаем:

- Удобный синтаксис
- Каррирование бесплатно (на самом деле любая функция принимает только один аргумент!)
- Нет потери производительности
