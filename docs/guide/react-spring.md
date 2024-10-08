---
sidebar_position: 12
title: Руководство по React Spring
description: Руководство по React Spring
keywords: ['javascript', 'js', 'react.js', 'reactjs', 'react', 'react spring', 'react-spring', 'spring', 'animation', 'guide', 'руководство', 'переход', 'анимация', 'пружина']
tags: ['javascript', 'js', 'react.js', 'reactjs', 'react', 'react spring', 'react-spring', 'spring', 'animation', 'guide', 'руководство', 'переход', 'анимация', 'пружина']
---

# React Spring

> [React Spring](https://react-spring.io/) - это основанная на физике упругости (spring - пружина, рессора) библиотека для реализации анимации в `React-приложениях`. Разработчики данной библиотеки утверждают, что она способна удовлетворить большую часть потребностей, возникающих при разработке пользовательских интерфейсов.

[Примеры использования](https://www.react-spring.io/docs/hooks/examples).

## Установка

```bash
yarn add react-spring
# или
npm i react-spring
```

## Хуки

### Основы

В настоящее время `react-spring` предоставляет 5 хуков (примитивов для анимации):

- `useSpring` - простая пружина, перемещающая данные из точки "а" в точку "б"
- `useSprings` - несколько пружин, для списков, где каждая пружина перемещает данные...
- `useTrail` - несколько пружин с общим набором данных, одна пружина следует за другой
- `useTransition` - для реализации переходов при монтировании/размонтировании компонентов (например, для элементов списка, которые добавляются/удаляются/обновляются). *Обратите внимание*: частью разрабатываемого в настоящее время командой `React` конкуретного режима является хук `useTransition`, предназначенный для обработки переходов между состояниями компонента. Как будет решена эта коллизия, покажет время
- `useChain` - для постановки в очередь или выстраивания в цепочку нескольких анимаций

Самым простым является хук `useSpring`

```js
import { useSpring, animated } from 'react-spring'

function App() {
  const props = useSpring({ opacity: 0, from: { opacity: 1 } })
  return <animated.div style={props}>Помогите! Я исчезаю</animated.div>
}
```

`animated` - это специальная фабрика, библиотека для выполнения анимации за пределами `React` (по соображениям производительности). Что делает `animated`? Она расширяет нативные элементы, чтобы они могли принимать анимированные значения (как `styled` в стилизованных компонентах).

Пружина анимирует значение от одного состояния до другого. Обновления суммируются, пружина запоминает переданные значения. Полученные пропы не являются статическими, они автоматически обновляются.

`animated` способна анимировать не только HTML-элементы, как в приведенном выше примере, но также React-компоненты, нативные компоненты, стилизованные компоненты и т.д.

```js
// компонент React
const MyAnimatedComponent = animated(MyComponent)

// React Native
const AnimatedView = animated(View)

// styled-components, emotion и т.д.
const AnimatedListItem = styled(animated.li)`
  ...
`
```

#### Передаваемые значения - это не совсем стили

Их можно использовать так:

```js
const props = useSpring({
  opacity: 1,
  from: { opacity: 0 }
})
return <animated.h1 style={props}>Привет!</animated.h1>
```

Можно анимировать атрибуты:

```js
const props = useSpring({
  x: 100,
  from: { x: 0 }
})
return (
  <animated.svg strokeDashoffset={props.x}>
    <path d="...">
  </animated.svg>
)
```

Можно анимировать текстовое содержимое:

```js
const props = useSpring({
  number: 1,
  from: {
    x: 0
  }
})
return <animated.span>{props.number}</animated.span>
```

Можно анимировать прокрутку страницы:

```js
const props = useSpring({
  scroll: 100,
  from: {
    scroll: 0
  }
})
return <animated.div scrollTop={props.scroll} />
```

Или генерировать пропы компонента:

```js
const AnimatedCircle = animate(Circle)

const props = useSpring({
  value: 100,
  from: {
    value: 0
  }
})
return <AnimatedCircle percent={props.value} />
```

#### Передняя интерполяция (up-front interpolation)

*Обратите внимание*: интерполяция (здесь) - это переход значения из одного состояния в другое через одно или более промежуточных состояний.

Пружины умеют работать не только с числами, но и с:

- цветами (названия, rgb, rgba, hsl, hsla, градиенты)
- абсолютными значениями (см, мм, дюймы, пиксели, пункты, пики)
- относительными значениями (em, ex, ch, rem, vw, vh, vmin, vmax, %)
- углами (deg, rad, grad, turn)
- единицами flex и grid (fr и т.д.)
- всеми HTML-атрибутами
- путями SVG (до тех пор, пока совпадает количество точек, в противном случае, следует использовать кастомную интерполяцию)
- массивами
- строковыми паттернами (transform, border, boxShadow и т.п.)
- не анимируемыми строковыми значениями (visibility, pointerEvents и др.)
- scrollTop/scrollLeft

```js
const props = useSpring({
  vector: [0, 10, 30],
  display: 'block',
  padding: 20,
  background: 'linear-gradient(to right, #009fff, #ec2f4b)',
  transform: 'translate3d(0px, 0, 0) scale(1) rotateX(0deg)',
  boxShadow: '0px 10px 20px 0px rgba(0, 0, 0, 0.4)',
  borderBottom: '10px solid #2d3747',
  shape: 'M20,20 L20,380 L380,380 L380,20 L20,20 Z',
  textShadow: '0px 5px 15px rgba(255, 255, 255, 0.5)'
})
```

#### Интерполяция представления (view interpolation)

В случае, когда нам требуется зафиксировать или экстраполировать значения, можно воспользоваться функцией `interpolate`, которую содержит каждое анимированное значение. Данная функция принимает функцию или объект с диапазоном значений. Из интерполяций можно формировать цепочки, позволяющие подключать одни вычисления к другим или повторно использовать произведенные ранее вычисления.

Причина, по которой интерполяцию лучше выполнять в представлении, а не в пружине, заключается в том, что интерполяция представления работает немного быстрее и занимает меньше места

```js
import { useSpring, animated, interpolate } from 'react-spring'

const { o, xyz, color } = useSpring({
  from: {
    o: 0,
    xyz: [0, 0, 0],
    color: 'red'
  }
  o: 1,
  xyz: [10, 20, 5],
  color: 'green'
})

return (
  <animated.div
    style={{
      // По-возможности, всегда используйте обычные анимированные значения,
      // когда они просто передаются
      color,
      // Пока не потребуется их интерполировать
      background: o.interpolate(o => `rgba(210, 57, 77, ${o})`),
      // Что также работает с массивами
      transform: xyz.interpolate((x, y, z) => `translate3d(${x}px, ${y}px, ${z}px)`),
      // Для комбинации нескольких значений используется утилита `interpolate`
      border: interpolate([o, color], (o, c) => `${o * 10}px solid ${c}`),
      // Можно создавать диапазоны и даже цепочки из нескольких интерполяций
      padding: o.interpolate({ range: [0, 0.5, 1], output: [0, 0, 10] }).interpolate(o => `${o}%`),
      // Разрешена интерполяция строк
      borderColor: o.interpolate({ range: [0, 1], output: ['red', '#ffaabb'] }),
      // Существует сокращенный синтаксис для диапазонов без настроек
      opacity: o.interpolate([0.1, 0.2, 0.6, 1], [1, 0.1, 0.5, 1])
    }}
  >
    {o.interpolate(n => n.toFixed(2))} /* интерполяция текста */
  </animated.div>
)
```

#### Заметки

##### Анимирование значения `auto`

Одним из отличий хуков от пропов для рендеринга является то, что хуки "не знают" о представлении. Поэтому хуки не умеют работать со значением `auto`. Это может показаться недостатком, однако, вычисление размера элемента с помощью таких хуков как <a href="https://github.com/FezVrasta/react-resize-aware">`useResizeAware`</a> или <a href="https://github.com/pmndrs/react-use-measure">`useMeasure`</a> никогда не было таким простым, как сейчас.

```js
const [ref, {height}] = useMeasure()
const props = useSpring({ height })
return (
  <animated.div style={{ overflow: 'hidden', ...props }}>
    <div ref={ref}>Контент</div>
  </animated.div>
)
```

##### Имитация ключевых кадров

```js
/*
  0% { transform: scale(1); }
  25% { transform: scale(.97); }
  35% { transform: scale(.9); }
  45% { transform: scale(1.1); }
  55% { transform: scale(.9); }
  65% { transform: scale(1.1); }
  75% { transform: scale(1.03); }
  100% { transform: scale(1); }
*/

const props = useSpring({ x: state ? 1 : 0 })
return (
  <animated.div
    style={{
      transfrom: props.x
        .interpolate({
          range: [0, 0.25, 0.35, 0.45, 0.55, 0.65, 0.75, 1],
          output: [1, 0.97, 0.9, 1.1, 0.9, 1.1, 1.03, 1]
        })
        .interpolate(x => `scale(${x})`)
    }}
  />
)
```

### Общий интерфейс

#### Настройки

Для настройки анимации используется объект `config`, передаваемый `useSpring` и другим хукам

```js
useSpring({ config: { duration: 250 }, ... })
```

| Свойство | Значение по умолчанию | Описание |
| --- | --- | --- |
| mass (масса) | 1 | вес пружины |
| tension (натяжение) | 170 | энергичность растяжения/сжатия пружины |
| friction (трение)  | 26 | сопротивление пружины |
| clamp (ограничение)  | false | если `true`, тогда пружина останавливается при достижении границы |
| precision (точность)  | 0.01 | точность |
| velocity (скорость)  | 0 | начальная скорость |
| duration (продолжительность)  | undefined | установление значения больше 0 приводит к переключению анимации с основанной на упругости на основанную на продолжительности |
| easing (характер движения)  | t => t | по умолчанию является линейным |

#### Пресеты

Существует набор общих пресетов, охватывающих некоторые распространенные случаи

```js
import { ..., config } from 'react-spring'

useSpring({ ..., config: config.default })
```

| Свойство | Значение |
| --- | --- |
| config.defaut | `{ mass: 1, tension: 170, friction: 26 }` |
| config.gentle |	`{ mass: 1, tension: 120, friction: 14 }` |
| config.wobbly |	`{ mass: 1, tension: 180, friction: 12 }` |
| config.stiff | `{ mass: 1, tension: 210, friction: 20 }` |
| config.slow	| `{ mass: 1, tension: 280, friction: 60 }` |
| config.molasses | 	`{ mass: 1, tension: 280, friction: 120 }` |

#### Свойства

```js
useSpring({ from: {...}, to: {...}, delay: 100, onRest: () => ... })
```

Все примитивы наследуют следующие свойства (некоторые могут предоставлять дополнительный функционал):

| Свойство | Тип | Описание |
| --- | --- | --- |
| from | obj | Базовые значение, опционально |
| to | obj/fn/arr(obj) | Конечные значения |
| delay | num/fn | Задержка в мс перед началом анимации. Может быть функцией для отдельных ключей: key => delay |
| immediate | bool/fn | Если `true`, анимация отключается. Может быть функцией для отдельных ключей |
| config | obj/fn | Настройки пружины (масса, натяжение, трение и т.д.). Может быть функцией для отдельных ключей |
| reset | bool | Если `true`, анимация начинается с самого начала |
| reverse | bool | Если `true`, `from` и `to` меняются местами. Это имеет смысл только при совместном использовании с `reset` |
| onStart | fn | Колбек, вызываемый в начала анимации |
| onRest | fn | Колбек, вызываемый после остановки всех анимаций |
| onFrame | fn | Покадровый (frame by frame) колбек, первый передаваемый аргумент является значением анимации |

#### Интерполяция

`value.interpolate` принимает объект следующей формы:

| Свойство | Значение по умолчанию | Описание |
| --- | --- | --- |
| extrapolateLeft | extend | Левая экстраполяция. Возможные значения: identity/clamp/extend |
| extrapolateRight | extend | Правая экстраполяция. Возможные значения: identity/clamp/extend |
| extrapolate | extend | Сокращенный синтаксис для установки левой и правой экстраполяций |
| range | [0, 1] | Массив входных значений |
| output | undefined | Массив соответствующих (mapped) выходных значений |
| map | undefined | Фильтр, применяемый к входному значению |

Сокращение для диапазона: `value.interpolate([...inputRange], [...outputRange])`. Может быть функцией: `value.interpolate(v => ...)`

### useSpring

```js
import { useSpring, animated } from 'react-spring'
```

Преобразует обычные значения в анимируемые.

Повторный рендеринг компонента с изменившимися пропами приводит к обновлению анимации

```js
const props = useSpring({ opacity: toggle ? 1 : 0 })
```

`useSpring` возвращает пропы (props) и две функции: для обновления анимации (set) и ее остановки (stop)

```js
const [props, set, stop] = useSpring(() => ({ opacity: 1 }))

// Обновляем пружину новыми пропами
set({ opacity: toggle ? 1 : 0 })
// Останавливаем анимацию
stop()
```

Для запуска анимации `props` передается в атрибут `style` компонента

```js
return <animated.div style={props}>Я появляюсь и исчезаю</animated.div>
```

#### Заметки

##### Сокращение для пропа `to`

Любое неопознанное свойство становится свойством `to`, например, `opacity: 1` становится `to: { opacity: 1 }`

```js
// Это
const props = useSpring({ opacity: 1, color: 'red' })
// является сокращением для этого
const props = useSpring({ to: { opacity: 1, color: 'red' } })
```

##### Асинхронные цепочки/скрипты

Проп `to` также позволяет превращать анимацию в скрипт и объединять анимации. Поскольку такие анимации будут асинхронными, передача свойства `from` с начальными значениями является обязательной, иначе, пропы будут пустыми.

Вот как можно создать скрипт

```js
const props = useSpring({
  to: async (next, cancel) => {
    await next({ opacity: 1, color: '#ffaaee' })
    await next({ opacity: 0, color: 'rgb(14, 26, 19)' })
  },
  from: { opacity: 0, color: 'red' }
})
// ...
return <animated.div style={props}>Я исчезаю и появляюсь</animated.div>
```

А так можно создать цепочку из анимаций

```js
const props = useSpring({
  to: [
    {
      opacity: 1,
      color: '#ffaaee'
    },
    {
      opacity: 0,
      color: 'rgb(14, 26, 19)'
    }
  ],
  from: {
    opacity: 0,
    color: 'red'
  }
})
// ...
return <animated.div style={props}>Туда и обратно</animated.div>
```

### useSprings

```js
import { useSprings, animated } from 'react-spring'
```

Создает несколько пружин. Каждая пружина имеет собственные настройки. Используется для статических списков и т.д.

Повторный рендеринг компонента с изменившимися пропами приводит к обновлению анимации

```js
const springs = useSprings(number, items.map(i => ({ opacity: i.opacity })))
```

`useSprings` возвращает пружины (springs) в виде массива и две функции: для обновления анимации (set) и ее остановки (stop)

```js
const [springs, set, stop] = useSprings(number, index => ({ opacity: 1 }))

// Обновляем пружины новыми пропами
set(index => ({ opacity: 0 }))
// Останавливаем все пружины
stop()
```

`springs` - это массив, содержащий анимированные пропы

```js
return springs.map(props => <animated.div style={props} />)
```

### useTrail

```js
import { useTrail, animated } from 'react-spring'
```

Создает несколько пружин с общими настройками, каждая пружина следует за предыдущей. Используется для последовательной (запланированной) анимации

Повторный рендеринг компонента с изменившимися пропами приводит к обновлению анимации

```js
const trail = useTrail(number, { opacity: 1 })
```

`useTrail` возвращает анимации (trail) в виде массива и две функции: для обновления анимации (set) и ее остановки (stop)

```js
const [trail, set, stop] = useTrail(number, () => ({ opacity: 1 }))

// Обновляем анимации
set({ opacity: 0 })
// Останавливаем их
stop()
```

`trail` - массив, содержащий анимированные пропы

```js
return trail.map(props => <animated.div style={props} />)
```

### useTransition

```js
import { useTransition, animated } from 'react-spring'
```

Анимированная `TransitionGroup`. `useTransition` принимает элементы, ключи (которые могут иметь значение `null`, если элементы являются атомарными) и жизненные циклы. Данный хук предназначен для анимирования добавления/удаления элементов.

Можно анимировать переходы массивов

```js
const [items, set] = useState([...])
const transitions = useTransition(items, item => item.key, {
  from: { transform: 'translate3d(0, -40px, 0)' },
  enter: { transform: 'translate3d(0, 0px, 0)' },
  leave: { transform: 'translate3d(0, -40px, 0)' }
})
return transitions.map(({ item, props, key }) => <animated.div key={key} style={props}>{item.text}</animated.div>)
```

Переключение компонентов

```js
const [toggle, set] = useState([...])
const transitions = useTransition(toggle, null, {
  from: { position: 'absolute', opacity: 0 },
  enter: { opacity: 1 },
  leave: { opacity: 0 }
})
return transitions.map(({ item, key, props }) =>
  item
    ? <animated.div style={props}>👍</animated.div>
    : <animated.div style={props}>👎</animated.div>
)
```

Отображение монтирования/размонтирования одного компонента

```js
const [show, set] = useState(false)
const transitions = useTransition(show, null, {
  from: { position: 'absolute', opacity: 0 },
  enter: { opacity: 1 },
  leave: { opacity: 0 }
})
return transitions.map(({ item, key, props }) =>
  item && <animated.div key={key} style={props}>✌️</animated.div>
)
```

#### Свойства

Кроме общих, `useTransition` принимает объект со следующими настройками

| Свойство | Тип | Описание |
| --- | --- | --- |
| initial | obj/fn | Начальные значения (значения первого запуска анимации), опционально (может иметь значение `null`) |
| from | obj/fn | Начальные значения, опционально |
| enter | obj/fn/arr(obj) | Стили, применяемые при монтировании элементов |
| update | obj/fn/arr(obj) | Стили, применяемые при обновлении элементов (можно обновлять сам хук) |
| leave | obj/fn/arr(obj) | Стили, применяемые при размонтировании элементов |
| trail | num | Задержка в мс перед началом анимации, применяется при каждом добавлении/обновлении/удалении |
| unique | bool/fn | Если `true`, тогда ключи элементов будут сохраняться |
| reset | bool/fn | Используется совместно с `reset`, "вхождение" элементов запускается с самого начала |
| onDestroyed | fn | Колбек, вызываемый после исчезновения объектов |

#### Заметки

##### Переходы через несколько состояний

Жизненные циклы initial/from/enter/update/leave могут быть объектами, массивами или функциями. Функция позволяет возвращать объект, массив или функцию для осуществления перехода через несколько состояний. При передаче массива можно реализовать базовый мультипереход без доступа к элементу

```js
useTransition(items, item => item.id, {
  enter: item => [
    {
      opacity: item.opacity,
      height: item.height
    },
    {
      life: '100%'
    }
  ],
  leave: item => async (next, cancel) => {
    await next({ life: '0%' })
    await next({ opacity: 0 })
    await next({ height: 0 })
  },
  from: {
    life: '0%',
    opacity: 0,
    height: 0
  }
})
```

##### Переход между маршрутами (routes)

```js
const location = useLocation()
const transitions = useTransition(location, location => location.pathname, {...})
return transitions.map(({ item, props, key }) => (
  <animated.div key={key} style={props}>
    <Switch location={item}>
      <Route path="/a" component={A}>
      <Route path="/b" component={B}>
      <Route path="/c" component={C}>
    </Switch>
  </animated.div>
))
```

### useChain

```js
import { useChain, animated } from 'react-spring'
```

Устанавливает порядок выполнения определенных ранее хуков анимации. `useChain` принимает массив со ссылками (refs) на анимации, что блокирует самостоятельный запуск анимаций. Порядок элементов в массиве определяет порядок выполнения анимаций

```js
// Создаем пружину и добавляем к ней ссылку
const springRef = useRef()
const props = useSpring({..., ref: springRef})
// Создаем переход и добавляем к нему ссылку
const transitionRef = useRef()
const transitions = useTransition({..., ref: transitionRef})
// Сначала запускает пружину, затем переход
useChain([springRef, transitionRef])
// Используем анимированные пропы как обычно
return (
  <animated.div style={props}>
    {transitions.map(({ item, key, props }) => (
      <animated.div key={key} style={props} />
    ))}
  </animated.div>
)
```

В качестве второго аргумента `useChain` принимает опциональный объект с настройками: `timeSteps` и `timeFrame` (по умолчанию имеет значение `1000` мс). `timeSteps` - массив с отступами от 0 до 1, определяющими начало и конец `timeFrame`

```js
// Пружина будет запущена сразу: 0.0 * 1000 мс = 0 мс
// Переход начнется через: 0.5 * 1000 мс = 500 мс
useChain([springRef, transitionRef], [0, 0.5] /*1000*/)
```

## Пропы для рендеринга

### Spring

Аналог `useSpring`

```js
import { Spring } from 'react-spring/renderprops'

<Spring
  from={{ opacity: 1 }}
  to={{ opacity: 0 }}>
  {props => <div style={props}>Помогите! Я исчезаю</div>}
</Spring>
```

#### Интерполяция

```js
<Spring
  from={{
    width: 100,
    padding: 0,
    background:
      'linear-gradient(to right, #30e8bf, #ff8235)',
    transform:
      'translate3d(400px,0,0) scale(2) rotateX(90deg)',
    boxShadow: '0px 100px 150px -10px #2D3747',
    borderBottom: '0px solid white',
    shape: 'M20,380 L380,380 L380,380 L200,20 L20,380 Z',
    textShadow: '0px 5px 0px white'
  }}
  to={{
    width: 'auto',
    padding: 20,
    background:
      'linear-gradient(to right, #009fff, #ec2f4b)',
    transform:
      'translate3d(0px,0,0) scale(1) rotateX(0deg)',
    boxShadow: '0px 10px 20px 0px rgba(0,0,0,0.4)',
    borderBottom: '10px solid #2D3747',
    shape: 'M20,20 L20,380 L380,380 L380,20 L20,20 Z',
    textShadow: '0px 5px 15px rgba(255,255,255,0.5)'
  }}>
  {props => ...}
</Spring>
```

#### Настройки

```js
import { Spring, config } from 'react-spring/renderprops'

// пресет
return <Spring config={config.default} />
// кастомизация
<Spring config={{tension: 0, friction: 2, precision: 0.1}} />
// кастомизация по ключу
<Spring to={{width: 10, color: 'red'}} config={key => (key === 'width' ? config.slow : config.wobbly)} />
```

### Trail

Аналог `useTrail`

```js
import { Trail } from 'react-spring/renderprops'

<Trail
  items={items}
  keys={item => item.key}
  from={{transform: 'translate3d(0, -40px, 0)'}}
  to={{transform: 'translate3d(0, 0px, 0)'}}
>
  {item => props => <span style={props}>{item.text}</span>}
</Trail>
```

### Transition

Аналог `useTransition`

```js
import { Transition } from 'react-spring/renderprops'

const items = [...]

<Transition
  items={items} keys={item => item.key}
  from={{ transform: 'translate3d(0, -40px, 0)' }}
  enter={{ transform: 'translate3d(0, 0px, 0)' }}
  leave={{ transform: 'translate3d(0, -40px, 0)' }}
>
  {item => props => <div style={props}>{item.text}</div>}
</Transition>
```

### Keyframes

Позволяет формировать цепочки, композиции из анимаций.

`Keyframes` - это фабрика для расширения `spring` и `trail`. Сначала определяются именованные слоты с анимируемыми свойствами. Все неизвестные свойства считаются свойствами `to`. Возвращается компонент со специальным свойством `state` с названием одного из слотов.

Слот принимает:

- объект со свойствами
- массив объектов, выстраиваемых в цепочку
- функцию, позволяющую создавать анимации в виде скриптов или формировать циклы

```js
// Допускается создание ключевых кадров для `Spring` и `Trail`
const Container = Keyframes.Spring({
  // Единичное значение
  show: { opacity: 1 },
  // Цепочка из анимаций (массив)
  showAndHide: [ { opacity: 1 }, { opacity: 0 } ],
  // Функция с побочными эффектами с доступом к пропам компонента
  wiggle: async (next, cancel, ownProps) => {
    await next({ x: 100, config: config.wobbly })
    await delay(1000)
    await next({ x: 0, config: config.gentle })
  }
})

return <Container state="showAndHide">{styles => <div style={styles}>Hello</div>}</Container>
```

### Parallax

```js
import { Parallax, ParallaxLayer } from 'react-spring/renderprops-addons'
```

`Parallax` создает "прокручиваемый" (scroll) контейнер. Ему передается любое количество `ParallaxLayer`, которые перемещаются согласно их отступам (offsets) и скоростям (speeds)

```js
<Parallax pages={3} scrolling={false} horizontal ref={ref => (this.parallax = ref)}>
  <ParallaxLayer offset={0} speed={0.5}>
    <span onClick={() => this.parallax.scrollTo(1)}>Layers can contain anything</span>
  </ParallaxLayer>
</Parallax>
```

#### Настройки

##### Parallax

| Свойство | Тип | Required | Default | Описание |
| --- | --- | --- | --- | --- |
| config | obj | false | config.slow | Настройки пружины |
| scrolling | bool | false | true | Определяет "прокручиваемость" контента |
| horizontal | bool | false | false | Определяет направление прокрутки |
| pages | num | true | - | Определяет общее пространство входящего контента, где каждая страница занимает 100% видимого контейнера |

##### ParallaxLayer

| Свойство | Тип | Required | Default | Описание |
| --- | --- | --- | --- | --- |
| factor | num | false | 1 | Размер страницы (1 = 100%, 1.5 = 150% и т.д.) |
| offset | num | false | 0 | Определяет, где будет слой после начала прокрутки (0 = начало, 1 = первая страница и т.д.) |
| speed | num | false | 0 | Определяет скорость сдвига слоя в соответствии с его отступом, значения могут быть как положительными, так и отрицательными |
