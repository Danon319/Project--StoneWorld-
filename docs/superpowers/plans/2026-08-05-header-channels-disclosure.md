# Свёрнутый список каналов связи на 769–1024 — план сборки

> **Для агентов:** ОБЯЗАТЕЛЬНАЯ ПОД-СКИЛЛ: `superpowers:subagent-driven-development`
> (рекомендуется) или `superpowers:executing-plans`. Шаги отмечаются чекбоксами.

**Цель:** занять пустующую полосу 769–1024 — вывести туда меню и каналы связи,
свернув четыре плашки в один триггер с выпадающим списком. **Ширины от 1025
не меняются ни в чём.**

**Архитектура:** правится одна собранная секция — GlobalHeader (SEC-2) в
`/index.html`. Стили и разметка в ЗОНЕ B (строки 456–1271), поведение в ЗОНЕ C
(строки 9210–9366). Компонент изолирован в секции, общего состояния с другими
секциями не заводит, третьим контрактом не становится.

**Стек:** vanilla HTML/CSS/JS. GSAP в новом компоненте не используется.

**Спецификация:** `docs/superpowers/specs/2026-08-05-header-channels-disclosure-design.md`
(редакция 2)

## Главное ограничение этой работы

**Геометрия и поведение секции на ширинах ≥1025 обязаны остаться неизменными
до сотых пикселя.** Это прямое требование пользователя, а не вывод: на 1025 запас
панели 106px, менять там нечего.

Проверяется не глазами, а сверкой замеров с эталоном, снятым до первой правки
(Task 1, шаг 1). Любое расхождение на 1025, 1200, 1440, 1920, 2560 — дефект,
даже если выглядит безобидно.

## Глобальные ограничения

Из `CLAUDE.md` разделы 3 и 4, действуют в каждой задаче:

- Никаких глобальных CSS-селекторов, сбросов, своего `@font-face`, префиксов `t-`/`tn-`.
- БЭМ с префиксом секции `sw-header__*`. Стили не выходят за `.sw-header`.
- `var()` всегда со вторым аргументом. В `@media` — литеральные px, не `var()`.
- Весь JS — внутри существующей IIFE секции. Глобальных переменных не заводить.
- Все ассеты — по абсолютным URL файлохранилища Тильды.
- Границы: только 768/769, 1024/1025, 1200, 425. Новых не заводить.
- Комментарии на русском, в стиле секции: объяснять почему, а не что.
- jQuery, сборщиков, `import` нет.

**Токены и фолбэки — дословно:**

```text
--sw-color-bg-page      #F4F5F5      --sw-color-cta          #242424
--sw-color-bg-section   #FFFFFF      --sw-color-text-on-cta  #FFFFFF
--sw-radius-pill        999px        --sw-dur-fast           150ms
--sw-ease-standard      cubic-bezier(0.4, 0, 0.2, 1)
```

**Ассеты, уже на CDN:**

```text
phone_dark    https://static.tildacdn.com/tild3864-6431-4566-b364-366462666633/phone_dark.svg
phone_white   https://static.tildacdn.com/tild3164-6564-4935-b336-656139303866/phone_white.svg
arrow_545454  https://static.tildacdn.com/tild3530-6530-4465-a333-373633376136/Arrow_545454.svg
arrow_white   https://static.tildacdn.com/tild3166-6664-4430-b933-633063343766/Arrow_FFFFFF.svg
```

---

## Как в этом проекте выглядит «тест»

Автотестов в проекте нет и заводить их не нужно: vanilla-код продакшеном не
является. Цикл проверки — **замер в браузере**, обязателен на каждой задаче.

Сервер поднимается **от корня проекта**, иначе замеры врут: через `file://` и через
сервер не от корня Chrome не загрузит шрифт и посчитает ширины на фолбэк-стеке.

```bash
node "$SCRATCH/serve.js" "d:/Programming/Проекты/Worker Project/Project 'StoneWorld'" 8777
```

Замерная функция — вызывается через Playwright `browser_evaluate` после
`browser_resize`:

```js
async () => {
  await document.fonts.ready;
  const q = s => document.querySelector(s);
  const w = el => el ? +el.getBoundingClientRect().width.toFixed(2) : null;
  const h = el => el ? +el.getBoundingClientRect().height.toFixed(2) : null;
  const disp = s => { const el = q(s); return el ? getComputedStyle(el).display : null; };
  const bar = q('.sw-header__bar');
  const cs = getComputedStyle(bar);
  return {
    vw: innerWidth,
    headerPadding: getComputedStyle(q('.sw-header')).padding,
    barInner: +(bar.getBoundingClientRect().width
      - parseFloat(cs.paddingLeft) - parseFloat(cs.paddingRight) - 2).toFixed(2),
    barHeight: cs.height,
    barPadding: cs.paddingLeft,
    gridCols: cs.gridTemplateColumns,
    logoW: w(q('.sw-header__logo')),
    logoImgH: getComputedStyle(q('.sw-header__logo-img')).height,
    logoNatural: (() => { const i = q('.sw-header__logo-img');
      return i ? +(i.getBoundingClientRect().height * 738 / 358).toFixed(2) : null; })(),
    navDisplay: disp('.sw-header__nav'),
    navW: w(q('.sw-header__nav')),
    navFontSize: getComputedStyle(q('.sw-header__nav-link')).fontSize,
    navGap: getComputedStyle(q('.sw-header__nav-list')).columnGap,
    burgerDisplay: disp('.sw-header__burger'),
    contactsW: w(q('.sw-header__contacts')),
    contactBlockW: w(q('.sw-header__contact-block')),
    toggleDisplay: disp('.sw-header__channels-toggle'),
    toggleW: w(q('.sw-header__channels-toggle')),
    pillPosition: q('.sw-header__pill') ? getComputedStyle(q('.sw-header__pill')).position : null,
    pillW: w(q('.sw-header__pill')),
    pillH: h(q('.sw-header__pill')),
    platesVisible: [...document.querySelectorAll('.sw-header__plate')]
      .filter(p => getComputedStyle(p).display !== 'none').length,
    docScrollX: document.documentElement.scrollWidth > document.documentElement.clientWidth
  };
}
```

`docScrollX` обязан быть `false` на каждой проверяемой ширине — защита от
горизонтального скролла всей страницы Тильды.

**Замеры снимались в Chrome с полосой прокрутки 15px.** Если у исполнителя она
другой ширины, `barInner` сдвинется — тогда сверять не абсолютные числа, а
эталон-против-текущего (Task 1 шаг 1) и `docScrollX === false`.

---

## Файлы

| Файл | Что делается |
|---|---|
| `index.html` 751–777 | базовые правила триггера, обёртки, каретки. `.sw-header__pill` не трогается |
| `index.html` 882–915 | фокус и hover — добавляются правила триггера |
| `index.html` 1114–1127 | `prefers-reduced-motion` — добавляются новые переходы |
| `index.html` 1210–1216 | разметка: обёртка и кнопка перед пилюлей |
| `index.html` 938–941 | `.sw-header { padding }` выносится в свой блок на 1025 |
| `index.html` 917, 1078 | композиционный блок `min-width: 1025px` → `769px` |
| новый блок после 1078 | `(min-width: 769px) and (max-width: 1024px)` — режим списка |
| `index.html` 615, 684, 843–852 | комментарии «в панели с 1025» → 769 |
| `index.html` 9309 | граница `gsap.matchMedia()` для text-roll → 769 |
| `index.html` ~9364 | блок 5 поведения секции, перед закрытием IIFE |
| `index.html` 1092–1112 | **не трогается** — блок `min-width: 1200px` |
| `CLAUDE.md`, `design_system.md` | состав шапки, реестр границ |

---

## Task 1: Компонент триггера, невидимый на всех ширинах

Первая задача **ничего не меняет визуально**. Она добавляет обёртку, кнопку и её
стили, но кнопка скрыта `display: none` по умолчанию и включится только в Task 2.

Смысл именно такого разбиения: главная проверка работы — «≥1025 не изменилось» —
делается здесь, на изолированной правке, где расхождение не с чем перепутать.

**Файлы:**
- Modify: `index.html:751-777` (блок пилюли — добавляются правила рядом)
- Modify: `index.html:882-915` (фокус, hover)
- Modify: `index.html:1114-1127` (`prefers-reduced-motion`)
- Modify: `index.html:1210-1216` (разметка перед пилюлей)

**Интерфейсы:**
- Отдаёт Task 2 и 3: `.sw-header__channels-wrap`, `.sw-header__channels-toggle`,
  `.sw-header__pill--open`, `id="sw-header-channels"` на пилюле, `aria-expanded`.

- [ ] **Шаг 1: снять эталон и сохранить его в файл**

Поднять сервер, `browser_navigate` на `http://localhost:8777/index.html`.
Снять замерную функцию на **768, 1025, 1200, 1440, 1920, 2560** и записать
результат в `$SCRATCH/header-baseline.json`.

Это эталон всей работы. К нему сверяются Task 1, 2 и финальная приёмка.

- [ ] **Шаг 2: базовые стили триггера**

Вставить **перед** комментарием `/* ─── пилюля и плашки каналов связи ─── */`
(`index.html:751`). Блок `.sw-header__pill` ниже **не трогать вовсе**:

```css
/* ─── триггер каналов связи ─── */
/* Живёт только на 769–1024: там пилюля из четырёх плашек не помещается —
   требует 201px при бюджете 108,7 (спецификация 1). На ≥1025 запас панели 106px,
   пилюля стоит развёрнутой, и триггера нет вовсе.
   Отсюда display: none в базовом правиле: включает триггер только блок
   769–1024 ниже. На ≥1025 секция получает ровно тот же каскад, что до правки, —
   это и есть требование «выше 1024 не меняется ничего».
   Обёртка несёт position: relative — от неё позиционируется выпавший список.
   Габарит триггера числом не задан и выводится из содержимого, как у пилюли:
   padding 5 + плашка + gap 5 + каретка + padding 10. Проверка: 769 → 78,00;
   1024 → 82,85. */
.sw-header__channels-wrap {
  position: relative;
  display: flex;
  flex: 0 0 auto;
  align-items: center;
}

/* Кнопка, а не ссылка: триггер никуда не ведёт.
   В покое светлая, как пилюля. В hover, focus-visible и раскрытом состоянии —
   целиком тёмная, с белым глифом и белой кареткой. Это повторяет обращение
   бургера: на ≤768 в правом углу тёмная кнопка, на 769–1024 — светлая пилюля,
   темнеющая при обращении. */
.sw-header__channels-toggle {
  display: none;
  flex: 0 0 auto;
  align-items: center;
  gap: 5px;
  margin: 0;
  padding: 5px 10px 5px 5px;
  border: 0;
  border-radius: var(--sw-radius-pill, 999px);
  background: var(--sw-color-bg-page, #F4F5F5);
  cursor: pointer;
  -webkit-appearance: none;
  appearance: none;
  -webkit-tap-highlight-color: rgba(0, 0, 0, 0);
  transition: background-color var(--sw-dur-fast, 150ms) var(--sw-ease-standard, cubic-bezier(0.4, 0, 0.2, 1));
}

/* Круг с глифом телефона. Габарит взят у плашки канала связи той же рампой —
   триггер обязан читаться как та же деталь, что и содержимое списка.
   Свой класс, а не .sw-header__plate: правила hover и :active плашки к триггеру
   не применяются, у него своё состояние по aria-expanded. */
.sw-header__channels-glyph {
  position: relative;
  display: flex;
  flex: 0 0 auto;
  align-items: center;
  justify-content: center;
  width: clamp(44px, 1.452vw + 32.84px, 70px);
  height: clamp(44px, 1.452vw + 32.84px, 70px);
  overflow: hidden;
  background: var(--sw-color-bg-section, #FFFFFF);
  border-radius: 50%;
  transition: background-color var(--sw-dur-fast, 150ms) var(--sw-ease-standard, cubic-bezier(0.4, 0, 0.2, 1));
}

/* Оба глифа лежат в разметке и переключаются прозрачностью: по событию
   не подгружаются, иначе первое обращение даст пустой кадр. Тот же приём,
   что у .sw-header__plate-icon. */
.sw-header__channels-icon {
  position: absolute;
  top: 50%;
  left: 50%;
  width: clamp(22px, 0.726vw + 16.42px, 35px);
  height: clamp(22px, 0.726vw + 16.42px, 35px);
  transform: translate(-50%, -50%);
  opacity: 1;
  transition: opacity var(--sw-dur-fast, 150ms) var(--sw-ease-standard, cubic-bezier(0.4, 0, 0.2, 1));
}

.sw-header__channels-icon--active { opacity: 0; }

/* Каретка. Анкеры 769 и 1024 — выше 1024 триггера нет. Проверка: 769 → 14,00;
   1024 → 15,14. Потолок 16 в полосе недостижим и стоит предохранителем.
   Поворот на 180° — указание на раскрытое состояние, помимо цвета. */
.sw-header__channels-arrow {
  position: relative;
  display: block;
  flex: 0 0 auto;
  width: clamp(14px, 0.4471vw + 10.56px, 16px);
  height: clamp(14px, 0.4471vw + 10.56px, 16px);
  transition: transform var(--sw-dur-fast, 150ms) var(--sw-ease-standard, cubic-bezier(0.4, 0, 0.2, 1));
}

.sw-header__channels-arrow-img {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  opacity: 1;
  transition: opacity var(--sw-dur-fast, 150ms) var(--sw-ease-standard, cubic-bezier(0.4, 0, 0.2, 1));
}

.sw-header__channels-arrow-img--active { opacity: 0; }

/* Раскрытое состояние. Селектор по aria-expanded, а не по классу: атрибут всё
   равно обязан быть верным для скринридера, и второй источник правды в виде
   класса разошёлся бы с ним молча. */
.sw-header__channels-toggle[aria-expanded="true"] {
  background: var(--sw-color-cta, #242424);
}

.sw-header__channels-toggle[aria-expanded="true"] .sw-header__channels-glyph {
  background: var(--sw-color-cta, #242424);
}

.sw-header__channels-toggle[aria-expanded="true"] .sw-header__channels-icon,
.sw-header__channels-toggle[aria-expanded="true"] .sw-header__channels-arrow-img {
  opacity: 0;
}

.sw-header__channels-toggle[aria-expanded="true"] .sw-header__channels-icon--active,
.sw-header__channels-toggle[aria-expanded="true"] .sw-header__channels-arrow-img--active {
  opacity: 1;
}

.sw-header__channels-toggle[aria-expanded="true"] .sw-header__channels-arrow {
  transform: rotate(180deg);
}
```

- [ ] **Шаг 3: фокус и hover**

В список `:focus-visible` (`index.html:885-890`) добавить строку перед
`.sw-header__burger:focus-visible`:

```css
.sw-header__channels-toggle:focus-visible,
```

В блок `@media (hover: hover)` (`index.html:896-906`) добавить:

```css
  .sw-header__channels-toggle:hover { background: var(--sw-color-cta, #242424); }
  .sw-header__channels-toggle:hover .sw-header__channels-glyph { background: var(--sw-color-cta, #242424); }
  .sw-header__channels-toggle:hover .sw-header__channels-icon,
  .sw-header__channels-toggle:hover .sw-header__channels-arrow-img { opacity: 0; }
  .sw-header__channels-toggle:hover .sw-header__channels-icon--active,
  .sw-header__channels-toggle:hover .sw-header__channels-arrow-img--active { opacity: 1; }
```

`:active` триггеру не нужен: состояние видно по `aria-expanded`, а на телефоне
компонента нет вовсе — ниже 769 его не существует.

- [ ] **Шаг 4: prefers-reduced-motion**

В существующий список селекторов блока `@media (prefers-reduced-motion: reduce)`
(`index.html:1121-1126`) добавить:

```css
  .sw-header__channels-toggle,
  .sw-header__channels-glyph,
  .sw-header__channels-icon,
  .sw-header__channels-arrow,
  .sw-header__channels-arrow-img,
  .sw-header__pill,
```

- [ ] **Шаг 5: разметка**

Пилюля и все четыре плашки **не трогаются**. Обернуть пилюлю и поставить перед
ней кнопку. Заменить открывающий тег `<div class="sw-header__pill">`
(`index.html:1216`) и закрывающий её `</div>` так:

```html
      <!-- Триггер каналов связи. Кнопка, а не ссылка: никуда не ведёт, только
           раскрывает список. Виден только на 769–1024 — выше пилюля стоит
           развёрнутой. Поведение — блок 5 в ЗОНЕ C.
           Глифы декоративные (alt пустой), доступное имя даёт aria-label кнопки.
           loading="lazy" не ставится — первый экран (NFR-4). -->
      <div class="sw-header__channels-wrap">
        <button type="button" class="sw-header__channels-toggle"
                aria-expanded="false" aria-controls="sw-header-channels"
                aria-label="Каналы связи">
          <span class="sw-header__channels-glyph">
            <img class="sw-header__channels-icon"
                 src="https://static.tildacdn.com/tild3864-6431-4566-b364-366462666633/phone_dark.svg" alt="">
            <img class="sw-header__channels-icon sw-header__channels-icon--active"
                 src="https://static.tildacdn.com/tild3164-6564-4935-b336-656139303866/phone_white.svg" alt="">
          </span>
          <span class="sw-header__channels-arrow">
            <img class="sw-header__channels-arrow-img"
                 src="https://static.tildacdn.com/tild3530-6530-4465-a333-373633376136/Arrow_545454.svg" alt="">
            <img class="sw-header__channels-arrow-img sw-header__channels-arrow-img--active"
                 src="https://static.tildacdn.com/tild3166-6664-4430-b933-633063343766/Arrow_FFFFFF.svg" alt="">
          </span>
        </button>

        <div class="sw-header__pill" id="sw-header-channels">
          <!-- четыре плашки — без изменений -->
        </div>
      </div>
```

Комментарий над пилюлей (`index.html:1210-1215`) поправить: строку «В панели
с 1025, ниже уходит в Mobile_menu (SEC-14) вместе с меню» заменить на «В панели
с 769. На 769–1024 свёрнута в список за триггером, с 1025 стоит развёрнутой.
Ниже 769 уходит в Mobile_menu (SEC-14) вместе с меню».

- [ ] **Шаг 6: сверить с эталоном — главная проверка задачи**

Перезагрузить. Снять замеры на **768, 1025, 1200, 1440, 1920, 2560** и сравнить
с `header-baseline.json` **по всем полям, кроме `toggleDisplay` и `toggleW`**.

Ожидание: **полное совпадение**. Новых полей два:

| Поле | Ожидание на всех шести ширинах |
|---|---|
| `toggleDisplay` | `"none"` |
| `toggleW` | `0` |

Расхождение в `contactsW`, `pillW`, `pillH` или `gridCols` означает, что обёртка
исказила раскладку — это дефект, а не «мелочь на 0,2px». Обёртка обязана быть
геометрически прозрачной.

- [ ] **Шаг 7: коммит**

```bash
git add index.html
git commit -m "feat(header): стили и разметка триггера каналов связи

Обёртка и кнопка добавлены, кнопка скрыта display: none - включит её
следующая задача в блоке 769-1024. Пилюля и четыре плашки не тронуты.

Сверено с эталоном на 768, 1025, 1200, 1440, 1920, 2560: геометрия
не изменилась ни в одном поле.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>"
```

---

## Task 2: Полоса 769–1024 — меню, правая зона, режим списка

Задача выводит содержимое в пустующую полосу. Ошибка здесь тихая: если утащить
метрику страницы вместе с композицией, вертикальные края шапки разойдутся со всеми
секциями ниже на всей полосе, а в консоли не будет ничего.

**Файлы:**
- Modify: `index.html:938-941` (`.sw-header { padding }` выносится)
- Modify: `index.html:917, 1078` (граница композиционного блока)
- Create: новый блок `(min-width: 769px) and (max-width: 1024px)` после 1078
- Modify: `index.html:615, 684, 843-852` (комментарии о составе)
- Modify: `index.html:9296-9309` (граница text-roll)

- [ ] **Шаг 1: вынести метрику страницы в свой блок**

Первым правилом блока `@media (min-width: 1025px)` (`index.html:939-941`) стоит
не композиция, а метрика страницы. **Вырезать это правило** и положить отдельным
блоком сразу после закрывающей скобки композиционного блока:

```css
/* ─── 1025px и выше: метрика страницы ─── */
/* Боковой отступ страницы и нижний зазор до следующей секции. По design_system 7
   они уменьшены на 40% в диапазоне <1024 и восстанавливаются блоком
   min-width: 1025px в КАЖДОЙ секции — вертикальные края всех секций обязаны
   совпадать.
   ЭТА ГРАНИЦА НЕ ПЕРЕЕХАЛА НА 769 ВМЕСТЕ С СОСТАВОМ ПАНЕЛИ и переехать не может:
   шапка получила бы полную метрику там, где остальные секции ещё на уменьшенной.
   Замер при попытке переноса: padding стал 0 15px 8px вместо 0 9px 4.8px,
   внутренняя ширина панели на 769 упала с 674,0 до 662,0, логотип раздавило
   до 15,3.
   Композиция секции переключается на 768/769 — это другой блок, выше. */
@media (min-width: 1025px) {
  .sw-header {
    padding: 0 clamp(15px, 1.6287vw - 1.69px, 40px) clamp(8px, 2.0847vw - 13.37px, 40px);
  }
}
```

- [ ] **Шаг 2: сменить границу композиционного блока**

Заменить `@media (min-width: 1025px)` на `@media (min-width: 769px)`
(`index.html:938`, после выноса метрики).

**Ни одну формулу внутри не трогать.** Все рампы на 769 уже стоят на своих полах —
проверено замером: высота панели 72, padding панели 30, зазор «логотип → меню» 28,
зазор пунктов меню 12, высота логотипа 58, плашка 44, глиф 22. Компактные клампы
по другую сторону границы дают на 768 ровно те же величины, разрыва нет.

Заголовок блока переписать:

```text
/* ─── 769px и выше: в панель приходят меню и правая зона ─── */
/* ГРАНИЦА СОСТАВА ВЕРНУЛАСЬ С 1024/1025 НА 768/769: полоса 769–1024 стояла
   почти пустой, а пилюля из четырёх плашек туда не помещалась — требовала 201px
   при бюджете 108,7. Помещается триггер: 78,0 на 769.
   ФОРМУЛЫ ЭТОГО БЛОКА НЕ ПЕРЕАНКЕРОВАНЫ И НЕ ТРЕБУЮТ ЭТОГО. Каждая прямая уходит
   ниже своего пола раньше 1025, поэтому на 769 все стоят на полах: 72 / 30 / 28 /
   12 / 58 / 44 / 22. Компактные клампы дают на 768 те же значения — разрыва
   на границе нет. На 769–1024 величины стоят плоско на полу и растут с 1025:
   это потолок клампа, а не заморозка.
   Метрика страницы (боковой отступ, нижний зазор) в этом блоке больше не живёт —
   она осталась на 1025 отдельным блоком ниже, вместе со всеми секциями. */
```

- [ ] **Шаг 3: блок режима списка**

Добавить новый блок сразу после блока метрики страницы:

```css
/* ─── 769–1024: каналы связи свёрнуты в список ─── */
/* ЕДИНСТВЕННОЕ МЕСТО, ГДЕ ПИЛЮЛЯ ВЕДЁТ СЕБЯ ИНАЧЕ. Выше 1024 её базовые правила
   не переопределяются ничем, и она стоит развёрнутой строкой в потоке — ровно
   как до этой работы.
   Верхняя граница ОБЯЗАНА совпадать с условием matchMedia в блоке 5 ЗОНЫ C.
   Разойдутся — на 1025 останется открытый список поверх развёрнутой пилюли.
   Габарит списка выводится из содержимого: на 769 это 54 × 201.
   Скрытие через visibility, а не через display или атрибут hidden: display
   не анимируется, а атрибут hidden потребовал бы лишнего кадра при снятии.
   visibility так же убирает содержимое из последовательности фокуса.
   Задержка на visibility при закрытии держит список видимым ровно на время
   перехода. */
@media (min-width: 769px) and (max-width: 1024px) {
  .sw-header__channels-toggle { display: flex; }

  .sw-header__pill {
    position: absolute;
    top: calc(100% + 8px);
    right: 0;
    flex-direction: column;
    visibility: hidden;
    opacity: 0;
    transform: translateY(-8px);
    transition:
      opacity var(--sw-dur-fast, 150ms) var(--sw-ease-standard, cubic-bezier(0.4, 0, 0.2, 1)),
      transform var(--sw-dur-fast, 150ms) var(--sw-ease-standard, cubic-bezier(0.4, 0, 0.2, 1)),
      visibility 0s linear var(--sw-dur-fast, 150ms);
  }

  .sw-header__pill--open {
    visibility: visible;
    opacity: 1;
    transform: translateY(0);
    transition:
      opacity var(--sw-dur-fast, 150ms) var(--sw-ease-standard, cubic-bezier(0.4, 0, 0.2, 1)),
      transform var(--sw-dur-fast, 150ms) var(--sw-ease-standard, cubic-bezier(0.4, 0, 0.2, 1)),
      visibility 0s linear 0s;
  }
}
```

- [ ] **Шаг 4: граница text-roll в JS**

В блоке 4 `gsap.matchMedia()` (`index.html:9309`) заменить:

```js
        '(hover: hover) and (min-width: 1025px) and (prefers-reduced-motion: no-preference)',
```

на:

```js
        '(hover: hover) and (min-width: 769px) and (prefers-reduced-motion: no-preference)',
```

В комментарии выше (`index.html:9296-9300`) заменить «Граница 1025» на «Граница 769»
с пояснением, что она переехала вместе с составом панели.

- [ ] **Шаг 5: комментарии о составе**

Три места в стилях говорят «в панели с 1025» / «до 1024 включительно»:
`index.html:615` (меню), `684` (правая зона), `843-852` (бургер). Привести к 769/768.
В комментарии бургера переписать следствие для SEC-14:

```text
   БУРГЕР ЖИВЁТ ДО 768 ВКЛЮЧИТЕЛЬНО. Граница вернулась с 1024 на 768: на 769–1024
   в панель приходят меню и каналы связи, свёрнутые в один триггер.
   Следствие для SEC-14: мобильное меню покрывает диапазон до 768 включительно
   и обязано нести все четыре канала связи — ниже 769 их в панели нет.
```

- [ ] **Шаг 6: проверить, что ≥1025 не шелохнулось**

Снять замеры на **1025, 1200, 1440, 1920, 2560** и сравнить с
`header-baseline.json` по всем полям. **Ожидание: полное совпадение, включая
`toggleDisplay: "none"` и `pillPosition: "static"`.**

Это повторная проверка главного ограничения: границу композиции двигали, и надо
убедиться, что верхняя сторона не задета.

- [ ] **Шаг 7: проверить границу 768/769**

| Величина | 768 | 769 |
|---|---|---|
| `headerPadding` | `0px 9px 4.8px` | `0px 9px 4.8px` |
| `barHeight` | `72px` | `72px` |
| `barPadding` | `30px` | `30px` |
| `logoImgH` | `58px` | `58px` |
| `logoW` | 119,6 | 119,6 |
| `navDisplay` | `none` | `block` |
| `burgerDisplay` | `flex` | `none` |
| `toggleDisplay` | `none` | `flex` |

**Первая строка — главная проверка задачи.** Если `headerPadding` на 769 стал
`0px 15px 8px`, метрика страницы уехала вместе с композицией: блок разделён неверно.

Разрыв допустим только в трёх последних строках — это смена состава.

- [ ] **Шаг 8: проверить полосу целиком**

| Ширина | Ожидание |
|---|---|
| 769 | `navW: 385,7`; `navGap: 12px`; `toggleW: 78,00`; `barInner: 674,0`; `pillPosition: absolute`; `docScrollX: false` |
| 900 | `docScrollX: false`; `toggleDisplay: flex` |
| 1024 | `headerPadding: 0px 9px 4.8px`; `toggleW: 82,85`; `pillPosition: absolute`; `docScrollX: false` |
| 1025 | `headerPadding: 0px 15px 8px`; `toggleDisplay: none`; `pillPosition: static`; `pillW: 215,88` |

Строки 1024 и 1025 проверяют обе границы сразу: метрика страницы переключается
на 1025, режим пилюли — там же.

- [ ] **Шаг 9: глазами и text-roll**

`browser_take_screenshot` на 900. Проверить: логотип слева, меню, справа светлый
триггер с тёмным глифом телефона и серой кареткой вниз. Бургера нет. Список не виден.

Навести на пункт меню — text-roll обязан работать (раньше на этой ширине меню
не было вовсе). `browser_console_messages` — новых ошибок нет.

- [ ] **Шаг 10: коммит**

```bash
git add index.html
git commit -m "feat(header): меню и каналы связи на 769-1024

Композиционный блок переехал с 1025 на 769. Метрика страницы (боковой
отступ, нижний зазор) выделена в свой блок и осталась на 1025 вместе
со всеми секциями - утащить её на 769 нельзя, края шапки разошлись бы
с секциями ниже.

На 769-1024 пилюля становится выпадающим списком за триггером. Выше 1024
её базовые правила ничем не переопределяются: развёрнутая строка в потоке,
как до правки. Сверено с эталоном - геометрия >=1025 не изменилась.

Формулы не переанкерованы: каждая рампа уходит ниже своего пола раньше
1025, на 769 все стоят на полах, компактные клампы дают на 768 те же
значения. Разрыва на границе нет.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>"
```

---

## Task 3: Поведение раскрытия

**Файлы:**
- Modify: `index.html:~9364` — блок 5 внутри IIFE секции, перед `})();`

**Интерфейсы:**
- Потребляет из Task 1: `.sw-header__channels-wrap`, `.sw-header__channels-toggle`,
  `aria-expanded`. Из Task 2: `.sw-header__pill--open` и медиаусловие
  `(min-width: 769px) and (max-width: 1024px)` — оно обязано совпасть посимвольно.

- [ ] **Шаг 1: убедиться, что раскрытия ещё нет**

`browser_resize` 900×900, `browser_click` по триггеру, затем:

```js
() => {
  const t = document.querySelector('.sw-header__channels-toggle');
  const l = document.querySelector('.sw-header__pill');
  return { expanded: t.getAttribute('aria-expanded'),
           open: l.classList.contains('sw-header__pill--open'),
           visibility: getComputedStyle(l).visibility };
}
```

Ожидание: `{ expanded: "false", open: false, visibility: "hidden" }` — клик пока
ничего не делает.

- [ ] **Шаг 2: вписать блок 5**

Вставить перед строкой `})();`, закрывающей IIFE GlobalHeader, после блока 4:

```js
  /* ── 5. Каналы связи: раскрытие списка на 769–1024 ────────────────────────
     Своё состояние секции, не контракт: Mobile_menu (SEC-14) о списке не знает,
     список о нём не знает. Общего состояния между секциями не появляется.
     Библиотек блок не использует — GSAP здесь не нужен и не проверяется.
     Единственный источник правды о состоянии — атрибут aria-expanded на кнопке:
     по нему же рисует CSS, второго источника в виде переменной нет. */
  var channelsWrap = header.querySelector('.sw-header__channels-wrap');
  var channelsToggle = channelsWrap && channelsWrap.querySelector('.sw-header__channels-toggle');
  var channelsList = channelsWrap && channelsWrap.querySelector('.sw-header__pill');

  if (channelsWrap && channelsToggle && channelsList) {

    /* Условие ОБЯЗАНО совпадать с медиазапросом режима списка в стилях секции
       посимвольно. Разойдутся — на 1025 останется открытый список поверх
       развёрнутой пилюли. При пересмотре границ правятся обе точки. */
    var channelsMQ = window.matchMedia('(min-width: 769px) and (max-width: 1024px)');

    var isChannelsOpen = function () {
      return channelsToggle.getAttribute('aria-expanded') === 'true';
    };

    var setChannelsOpen = function (next) {
      if (next === isChannelsOpen()) return;
      channelsToggle.setAttribute('aria-expanded', next ? 'true' : 'false');
      if (next) channelsList.classList.add('sw-header__pill--open');
      else channelsList.classList.remove('sw-header__pill--open');
    };

    channelsToggle.addEventListener('click', function () {
      setChannelsOpen(!isChannelsOpen());
    });

    /* Esc возвращает фокус на кнопку: иначе он остался бы на плашке, которая
       только что выпала из последовательности по visibility. */
    document.addEventListener('keydown', function (event) {
      if (event.key !== 'Escape' && event.keyCode !== 27) return;
      if (!isChannelsOpen()) return;
      setChannelsOpen(false);
      channelsToggle.focus();
    });

    /* pointerdown, а не click: закрытие обязано срабатывать до того, как клик
       дойдёт до элемента под списком. Клик по самой кнопке сюда не попадает —
       она внутри обёртки. */
    document.addEventListener('pointerdown', function (event) {
      if (!isChannelsOpen()) return;
      if (channelsWrap.contains(event.target)) return;
      setChannelsOpen(false);
    });

    /* Уход фокуса за пределы обёртки — закрытие. Перехвата фокуса нет и не нужно:
       список стоит сразу за кнопкой в DOM, Tab идёт по нему сам. */
    channelsWrap.addEventListener('focusout', function (event) {
      if (!isChannelsOpen()) return;
      if (event.relatedTarget && channelsWrap.contains(event.relatedTarget)) return;
      setChannelsOpen(false);
    });

    /* Выход из полосы 769–1024 в ЛЮБУЮ сторону закрывает список: ниже 769
       компонента нет вовсе, с 1025 пилюля обязана стоять развёрнутой, а
       aria-expanded у скрытого триггера — в false.
       addListener — фолбэк для Safari до 14, addEventListener там отсутствует. */
    var onChannelsMQ = function () {
      if (!channelsMQ.matches) setChannelsOpen(false);
    };

    if (channelsMQ.addEventListener) channelsMQ.addEventListener('change', onChannelsMQ);
    else if (channelsMQ.addListener) channelsMQ.addListener(onChannelsMQ);
  }
```

- [ ] **Шаг 3: проверить открытие и пять закрытий**

Перезагрузить, `browser_resize` 900×900. Состояние снимать функцией из шага 1.

| Сценарий | Действия | Ожидание |
|---|---|---|
| Открытие | клик по триггеру | `expanded: "true"`, `open: true`, `visibility: "visible"` |
| Повторный клик | клик ещё раз | `expanded: "false"`, `visibility: "hidden"` |
| Esc | открыть, `browser_press_key Escape` | закрыт; активный элемент — триггер |
| Клик вне | открыть, клик по логотипу | закрыт |
| Уход фокуса | открыть, Tab до выхода за список | закрыт |
| Вверх из полосы | открыть, `browser_resize` 1200×900 | закрыт |
| Вниз из полосы | открыть на 900, `browser_resize` 500×900 | закрыт |

Проверка возврата фокуса после Esc:

```js
() => document.activeElement.className
```

Ожидание: строка содержит `sw-header__channels-toggle`.

- [ ] **Шаг 4: проверить, что на 1025 список не всплывает**

Открыть список на 1024, растянуть до 1200, снять:

```js
() => {
  const t = document.querySelector('.sw-header__channels-toggle');
  const l = document.querySelector('.sw-header__pill');
  const cs = getComputedStyle(l);
  return { expanded: t.getAttribute('aria-expanded'),
           hasOpenClass: l.classList.contains('sw-header__pill--open'),
           position: cs.position, visibility: cs.visibility,
           toggleDisplay: getComputedStyle(t).display };
}
```

Ожидание: `expanded: "false"`, `hasOpenClass: false`, `position: "static"`,
`visibility: "visible"`, `toggleDisplay: "none"` — пилюля развёрнута, триггера нет.

- [ ] **Шаг 5: клавиатура и консоль**

`browser_console_messages` — новых ошибок нет.

На 900: Tab от триггера обязан пройти по всем четырём плашкам при открытом
состоянии и **перепрыгнуть их** при закрытом (`visibility: hidden` убирает
из последовательности).

- [ ] **Шаг 6: коммит**

```bash
git add index.html
git commit -m "feat(header): раскрытие списка каналов связи на 769-1024

Открытие кликом, пять способов закрытия: повторный клик, Esc с возвратом
фокуса, клик вне, уход фокуса, выход из полосы 769-1024 в любую сторону.
Источник правды о состоянии один - атрибут aria-expanded, по нему же
рисует CSS.

Условие matchMedia повторяет медиазапрос режима списка посимвольно:
разойдутся - на 1025 останется открытый список поверх развёрнутой пилюли.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>"
```

---

## Task 4: Документация

Объём правки меньше, чем кажется — проверено чтением документов, а не по памяти.

**Файлы:**
- Modify: `CLAUDE.md` раздел 1
- Modify: `Design_system/design_system.md` 2.7
- Проверить, править не требуется: `Documentation/PRD.md`

- [ ] **Шаг 1: CLAUDE.md — состав шапки**

В разделе 4 реестр границ **не меняется**: 1024 остаётся («композиция SEC-3,
зазоры SEC-3 и SEC-4»), 1200 остаётся («состав правой зоны шапки»). В SEC-2
граница 1024/1025 продолжает работать — на метрике страницы и на режиме пилюли.

Добавить в раздел 1, к описанию собранных секций:

```text
**Состав GlobalHeader по ширинам:** ≤768 логотип и бургер; 769–1024 логотип,
меню и триггер каналов связи, свёрнутых в выпадающий список; 1025–1199 логотип,
меню и пилюля из четырёх плашек; ≥1200 добавляется блок номера, плашек три.
Композиция секции переключается на 768/769, 1024/1025 и 1200.
```

- [ ] **Шаг 2: design\_system.md 2.7**

Найти строку границы `768 / 769` и добавить SEC-2 в список потребителей — на неё
переехал состав панели:

```bash
grep -n '768 / 769' Design_system/design_system.md
```

Строку `1024 / 1025` (строка 269) **не менять**: формулировка «композицию шапки
SEC-5» относится к шапке каталога, GlobalHeader в ней никогда не числился.

Проверить раздел 12 «долги документации» на записи о составе шапки на 1024:

```bash
grep -n 'шапк' Design_system/design_system.md
```

- [ ] **Шаг 3: PRD — проверить, что правка не нужна**

```bash
grep -n 'SEC-14' Documentation/PRD.md
```

Строка 123 уже гласит «Mobile\_menu — полноэкранное меню **на ≤768**». PRD никогда
не переписывался под границу 1024 — после этой работы он снова верен. **Править
нечего, только убедиться.** Если найдётся другое место с ≤1024 для SEC-14 —
привести к ≤768, требование нести все четыре канала оставить.

- [ ] **Шаг 4: коммит**

```bash
git add CLAUDE.md Design_system/design_system.md
git commit -m "docs: состав шапки после вывода содержимого на 769-1024

Реестр границ не меняется: 1024 остаётся, в SEC-2 она продолжает работать
на метрике страницы и на режиме пилюли. К границе 768/769 добавлен SEC-2.

PRD правки не потребовал: он всё это время описывал SEC-14 как меню на 768
и после этой работы снова верен.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>"
```

---

## Финальная приёмка

Критерии раздела 7 спецификации, на живом цехе:

- [ ] 769, 900, 1024 — панель не переполняется, `docScrollX: false`
- [ ] **1025, 1200, 1440, 1920, 2560 — все поля совпадают с `header-baseline.json`
      до сотых.** Главный критерий работы
- [ ] 768 и 769 — `headerPadding`, `barHeight`, `barPadding`, `logoImgH`, `logoW`
      совпадают; `headerPadding` равен `0px 9px 4.8px` на обеих
- [ ] 1024 → 1025 — `headerPadding` переключается `0px 9px 4.8px` → `0px 15px 8px`,
      `pillPosition` — `absolute` → `static`
- [ ] `git diff` по секции не содержит ни одной правки внутри `clamp()`
- [ ] список проходит все семь сценариев Task 3 шаг 3
- [ ] на ≥1025 `aria-expanded: "false"`, триггер `display: none`, пилюля развёрнута
- [ ] все четыре канала достижимы с клавиатуры, обводка фокуса видна
- [ ] `prefers-reduced-motion: reduce` — перехода нет, состояние различимо
- [ ] `text-roll` работает на 769–1024
- [ ] блок `@media (min-width: 1200px)` в диффе отсутствует целиком
- [ ] стоп-лист: `grep -n "^body\|^\*\s*{\|@font-face\|jQuery\|\bimport\b" index.html`
      не даёт новых совпадений в диапазоне секции

---

## Отступление от спецификации

Спецификация 3.3 не называет способ скрытия списка формально. План использует
`visibility: hidden` в паре с `opacity` и `transform`. Причина: `display: none`
не анимируется, а атрибут `hidden` потребовал бы принудительного лишнего кадра
при снятии. `visibility` так же убирает содержимое из последовательности фокуса,
но анимируется.
