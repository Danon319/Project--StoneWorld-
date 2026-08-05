# Свёрнутый список каналов связи в GlobalHeader — план сборки

> **Для агентов:** ОБЯЗАТЕЛЬНАЯ ПОД-СКИЛЛ: `superpowers:subagent-driven-development`
> (рекомендуется) или `superpowers:executing-plans`. Шаги отмечаются чекбоксами.

**Цель:** заменить пилюлю из четырёх плашек каналов связи одной плашкой-триггером
с выпадающим списком и перенести границу состава шапки с 1024/1025 на 768/769.

**Архитектура:** правится одна уже собранная секция — GlobalHeader (SEC-2) в
`/index.html`. Стили и разметка лежат в ЗОНЕ B (строки 456–1271), поведение — в
ЗОНЕ C (строки 9210–9366). Новый компонент изолирован в секции, общего состояния
с другими секциями не заводит, третьим контрактом не становится.

**Стек:** vanilla HTML/CSS/JS. GSAP в новом компоненте не используется.

**Спецификация:** `docs/superpowers/specs/2026-08-05-header-channels-disclosure-design.md`

## Глобальные ограничения

Взяты из `CLAUDE.md` разделы 3 и 4, действуют в каждой задаче:

- Никаких глобальных CSS-селекторов, сбросов, своего `@font-face`, префиксов `t-`/`tn-`.
- БЭМ с префиксом секции `sw-header__*`. Стили не выходят за `.sw-header`.
- `var()` всегда со вторым аргументом. В `@media` — литеральные px, не `var()`.
- Весь JS — внутри существующей IIFE секции. Глобальных переменных не заводить.
- Все ассеты — по абсолютным URL файлохранилища Тильды.
- Границы перестроения: только 768/769, 1024/1025, 1200, 425. Новых не заводить.
- Комментарии в коде на русском, в стиле секции: объяснять почему, а не что.
- jQuery, сборщиков, `import` нет.

**Токены и их фолбэки — дословно, менять нельзя:**

```
--sw-color-bg-page      #F4F5F5      --sw-color-cta          #242424
--sw-color-bg-section   #FFFFFF      --sw-color-text-on-cta  #FFFFFF
--sw-radius-pill        999px        --sw-dur-fast           150ms
--sw-ease-standard      cubic-bezier(0.4, 0, 0.2, 1)
```

**Ассеты, уже на CDN:**

```
phone_dark    https://static.tildacdn.com/tild3864-6431-4566-b364-366462666633/phone_dark.svg
phone_white   https://static.tildacdn.com/tild3164-6564-4935-b336-656139303866/phone_white.svg
arrow_545454  https://static.tildacdn.com/tild3530-6530-4465-a333-373633376136/Arrow_545454.svg
arrow_white   https://static.tildacdn.com/tild3166-6664-4430-b933-633063343766/Arrow_FFFFFF.svg
```

---

## Как в этом проекте выглядит «тест»

Автотестов в проекте нет и заводить их не нужно: продакшеном vanilla-код не
является. Цикл проверки — **замер в браузере**, и он обязателен на каждой задаче.

Поднять сервер от корня проекта (иначе замеры врут — путь к шрифту ломается):

```bash
node "$SCRATCH/serve.js" "d:/Programming/Проекты/Worker Project/Project 'StoneWorld'" 8777
```

Замерная функция, вызывается через Playwright `browser_evaluate` после
`browser_resize` на нужную ширину. **Возвращает те величины, по которым каждая
задача сверяет ожидание:**

```js
async () => {
  await document.fonts.ready;
  const q = s => document.querySelector(s);
  const w = el => el ? +el.getBoundingClientRect().width.toFixed(2) : null;
  const h = el => el ? +el.getBoundingClientRect().height.toFixed(2) : null;
  const bar = q('.sw-header__bar');
  const cs = getComputedStyle(bar);
  return {
    vw: innerWidth,
    barInner: +(bar.getBoundingClientRect().width
      - parseFloat(cs.paddingLeft) - parseFloat(cs.paddingRight) - 2).toFixed(2),
    barHeight: cs.height,
    barPadding: cs.paddingLeft,
    gridCols: cs.gridTemplateColumns,
    logoW: w(q('.sw-header__logo')),
    logoImgH: getComputedStyle(q('.sw-header__logo-img')).height,
    logoNatural: (() => { const i = q('.sw-header__logo-img');
      return i ? +(i.getBoundingClientRect().height * 738 / 358).toFixed(2) : null; })(),
    navW: w(q('.sw-header__nav')),
    navFontSize: getComputedStyle(q('.sw-header__nav-link')).fontSize,
    navGap: getComputedStyle(q('.sw-header__nav-list')).columnGap,
    toggleW: w(q('.sw-header__channels-toggle')),
    listW: w(q('.sw-header__channels')),
    listH: h(q('.sw-header__channels')),
    platesVisible: [...document.querySelectorAll('.sw-header__channels .sw-header__plate')]
      .filter(p => getComputedStyle(p).display !== 'none').length,
    docScrollX: document.documentElement.scrollWidth > document.documentElement.clientWidth
  };
}
```

`docScrollX` обязан быть `false` на каждой проверяемой ширине — это защита от
горизонтального скролла всей страницы Тильды.

**Замеры снимались в Chrome с полосой прокрутки 15px.** Числа ниже приведены под
неё. Если у исполнителя полоса другой ширины, `barInner` сдвинется — сверять надо
не абсолютное совпадение, а то, что содержимое помещается и `docScrollX === false`.

---

## Файлы

| Файл | Что делается |
|---|---|
| `index.html` 751–777 | `.sw-header__pill` заменяется на компонент триггера и списка |
| `index.html` 882–915 | фокус, hover и `:active` — добавляются правила триггера |
| `index.html` 938–1078 | блок `min-width: 1025px` делится надвое: композиция → `769px`, метрика страницы остаётся на `1025px`. Формулы не трогаются |
| `index.html` 1092–1112 | блок `min-width: 1200px` — снимается правило плашки телефона |
| `index.html` 1114–1127 | `prefers-reduced-motion` — добавляются новые переходы |
| `index.html` 1210–1256 | разметка пилюли → разметка триггера и списка |
| `index.html` 9292–9309 | граница 1025 в `gsap.matchMedia()` для text-roll → 769 |
| `index.html` ~9364 | новый блок 5 поведения секции, перед закрытием IIFE |
| `CLAUDE.md` | реестр границ, состав шапки |
| `Documentation/PRD.md` | диапазон Mobile\_menu (SEC-14) |

---

## Task 1: Триггер и выпадающий список — разметка и стили

Граница состава на этой задаче **не трогается**, остаётся 1024/1025. Проверка
идёт на ≥1025, где компонент уже виден.

**Файлы:**
- Modify: `index.html:751-777` (блок «пилюля и плашки каналов связи»)
- Modify: `index.html:882-915` (фокус, hover, `:active`)
- Modify: `index.html:1114-1127` (`prefers-reduced-motion`)
- Modify: `index.html:1210-1256` (разметка пилюли)

**Интерфейсы:**
- Отдаёт задачам 2–4: классы `.sw-header__channels-wrap`,
  `.sw-header__channels-toggle`, `.sw-header__channels`, модификатор состояния
  `.sw-header__channels--open`, атрибут `aria-expanded` на триггере,
  `id="sw-header-channels"` на списке.

- [ ] **Шаг 1: снять замер «до» на 1025 и 1440**

Поднять сервер, `browser_navigate` на `http://localhost:8777/index.html`,
`browser_resize` 1025×900 и 1440×900, на каждой — замерная функция.

Ожидание до правки: `toggleW: null`, `listW: null` — компонента ещё нет,
`.sw-header__pill` на 1025 даёт 215,88 шириной.

Записать `barInner` и `logoW` — они не должны измениться после правки.

- [ ] **Шаг 2: заменить блок стилей пилюли**

В `index.html` заменить блок целиком — от комментария
`/* ─── пилюля и плашки каналов связи ─── */` до закрывающей скобки
`.sw-header__plate { … }` включительно **не трогая** `.sw-header__plate` и всё,
что ниже него. Меняется только `.sw-header__pill` и добавляется новое:

```css
/* ─── триггер каналов связи и выпадающий список ─── */
/* Пилюля из четырёх плашек заменена одной плашкой-триггером: на 769 та требовала
   201px при бюджете 108,7 (спецификация, раздел 1). Все четыре канала уехали
   в список, раскрывающийся вниз под панель.
   Габарит триггера числом не задан и выводится из содержимого, как и габарит
   прежней пилюли: padding 5 + плашка + gap 5 + каретка + padding 10.
   Проверка: 769 → 78,0; 1025 → 82,86; 1200 → 86,2; 2560 → 112,0.
   Обёртка несёт position: relative — от неё позиционируется список. */
.sw-header__channels-wrap {
  position: relative;
  display: flex;
  flex: 0 0 auto;
  align-items: center;
}

/* Кнопка, а не ссылка: триггер никуда не ведёт.
   В покое светлая, как прежняя пилюля. В hover, focus-visible и раскрытом
   состоянии — целиком тёмная, с белым глифом и белой кареткой. Это повторяет
   обращение бургера: на ≤768 в правом углу тёмная кнопка, на ≥769 — светлая
   пилюля, темнеющая при обращении. */
.sw-header__channels-toggle {
  display: flex;
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

/* Каретка. Своя рампа, анкеры 769 и 2560: проверка 769 → 14,0; 2560 → 22,0.
   Поворот на 180° — единственное указание на раскрытое состояние, кроме цвета. */
.sw-header__channels-arrow {
  position: relative;
  display: block;
  flex: 0 0 auto;
  width: clamp(14px, 0.4462vw + 10.57px, 22px);
  height: clamp(14px, 0.4462vw + 10.57px, 22px);
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

/* Раскрытое состояние. Селектор по aria-expanded, а не по классу: атрибут
   всё равно обязан быть верным для скринридера, и второй источник правды
   в виде класса разошёлся бы с ним молча. */
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

/* Список. Вертикальная пилюля, оформление один в один с прежней горизонтальной.
   Габарит выводится из содержимого: на 1025 это 57,72 × 215,88.
   Панель не имеет overflow: hidden, список свободно выходит за её нижнюю кромку.
   pointer-events наследуется от панели (auto) — корневое none секции не мешает.
   Скрытие через visibility, а не через атрибут hidden: hidden даёт display: none,
   и переход появления при снятии атрибута не проигрывается без лишнего кадра.
   visibility так же убирает содержимое из последовательности фокуса, но
   анимируется. Задержка на visibility при закрытии держит элемент видимым
   ровно на время перехода. */
.sw-header__channels {
  position: absolute;
  top: calc(100% + 8px);
  right: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 5px;
  padding: 5px;
  background: var(--sw-color-bg-page, #F4F5F5);
  border-radius: var(--sw-radius-pill, 999px);
  visibility: hidden;
  opacity: 0;
  transform: translateY(-8px);
  transition:
    opacity var(--sw-dur-fast, 150ms) var(--sw-ease-standard, cubic-bezier(0.4, 0, 0.2, 1)),
    transform var(--sw-dur-fast, 150ms) var(--sw-ease-standard, cubic-bezier(0.4, 0, 0.2, 1)),
    visibility 0s linear var(--sw-dur-fast, 150ms);
}

.sw-header__channels--open {
  visibility: visible;
  opacity: 1;
  transform: translateY(0);
  transition:
    opacity var(--sw-dur-fast, 150ms) var(--sw-ease-standard, cubic-bezier(0.4, 0, 0.2, 1)),
    transform var(--sw-dur-fast, 150ms) var(--sw-ease-standard, cubic-bezier(0.4, 0, 0.2, 1)),
    visibility 0s linear 0s;
}
```

- [ ] **Шаг 3: добавить триггер в правила фокуса и hover**

В списке `:focus-visible` (`index.html:885-890`) добавить строку перед
`.sw-header__burger:focus-visible`:

```css
.sw-header__channels-toggle:focus-visible,
```

В блоке `@media (hover: hover)` (`index.html:896-906`) добавить:

```css
  .sw-header__channels-toggle:hover { background: var(--sw-color-cta, #242424); }
  .sw-header__channels-toggle:hover .sw-header__channels-glyph { background: var(--sw-color-cta, #242424); }
  .sw-header__channels-toggle:hover .sw-header__channels-icon,
  .sw-header__channels-toggle:hover .sw-header__channels-arrow-img { opacity: 0; }
  .sw-header__channels-toggle:hover .sw-header__channels-icon--active,
  .sw-header__channels-toggle:hover .sw-header__channels-arrow-img--active { opacity: 1; }
```

`:active` триггеру не нужен: смена состояния на нём и так видна по
`aria-expanded`, а на телефоне компонента нет вовсе — ниже 769 его не существует.

- [ ] **Шаг 4: добавить новые переходы в prefers-reduced-motion**

В блок `@media (prefers-reduced-motion: reduce)` (`index.html:1121-1126`) в
существующий список селекторов добавить:

```css
  .sw-header__channels-toggle,
  .sw-header__channels-glyph,
  .sw-header__channels-icon,
  .sw-header__channels-arrow,
  .sw-header__channels-arrow-img,
  .sw-header__channels,
```

Состояние остаётся различимым: цвет триггера меняется мгновенно, поворот каретки
снимается вместе с переходом.

- [ ] **Шаг 5: заменить разметку пилюли**

Заменить блок от комментария `<!-- Пилюля. В панели с 1025 … -->` до закрывающего
`</div>` пилюли включительно (`index.html:1210-1256`). Четыре плашки переносятся
**дословно** — классы, `href`, `aria-label`, оба глифа, маркеры ANL сохраняются:

```html
      <!-- Триггер каналов связи. Кнопка, а не ссылка: никуда не ведёт, только
           раскрывает список. Поведение — блок 5 в ЗОНЕ C.
           Порядок каналов в списке задан пользователем: телефон, MAX, VK, почта.
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

        <div class="sw-header__channels" id="sw-header-channels">
          <!-- ANL: имя цели не назначено -->
          <a class="sw-header__plate sw-header__plate--phone" href="tel:+79649387392"
             aria-label="Позвонить по номеру +7 964 938 73 92">
            <img class="sw-header__plate-icon"
                 src="https://static.tildacdn.com/tild3864-6431-4566-b364-366462666633/phone_dark.svg" alt="">
            <img class="sw-header__plate-icon sw-header__plate-icon--active"
                 src="https://static.tildacdn.com/tild3164-6564-4935-b336-656139303866/phone_white.svg" alt="">
          </a>

          <!-- ANL: имя цели не назначено -->
          <a class="sw-header__plate sw-header__plate--max"
             href="https://max.ru/join/Dssb1Agm0wct4o-rTod9jFze2If3DDvlooDwi95e550"
             target="_blank" rel="noopener noreferrer" aria-label="Написать в MAX">
            <img class="sw-header__plate-icon"
                 src="https://static.tildacdn.com/tild3132-3362-4738-b264-316533313838/max_logo_dark.svg" alt="">
            <img class="sw-header__plate-icon sw-header__plate-icon--active"
                 src="https://static.tildacdn.com/tild6465-6263-4965-b232-373639656339/max_logo_white.svg" alt="">
          </a>

          <!-- ANL: имя цели не назначено -->
          <a class="sw-header__plate sw-header__plate--vk" href="https://vk.ru/id1118207836"
             target="_blank" rel="noopener noreferrer" aria-label="Открыть страницу VK">
            <img class="sw-header__plate-icon"
                 src="https://static.tildacdn.com/tild3565-6435-4861-b362-343736333331/vk_logo_dark.svg" alt="">
            <img class="sw-header__plate-icon sw-header__plate-icon--active"
                 src="https://static.tildacdn.com/tild3631-3161-4166-a136-633330356331/vk_logo_white.svg" alt="">
          </a>

          <!-- Почта. target и rel не ставятся: mailto: новую вкладку не открывает,
               схему обрабатывает почтовый клиент. Тот же случай, что у tel:
               в плашке телефона. -->
          <!-- ANL: имя цели не назначено -->
          <a class="sw-header__plate sw-header__plate--mail" href="mailto:Thestoneworld@yandex.ru"
             aria-label="Написать на почту Thestoneworld@yandex.ru">
            <img class="sw-header__plate-icon"
                 src="https://static.tildacdn.com/tild3265-3331-4433-a434-343438356363/Yandex-mail_dark.svg" alt="">
            <img class="sw-header__plate-icon sw-header__plate-icon--active"
                 src="https://static.tildacdn.com/tild3539-3136-4834-b538-336536333761/Yandex-mail_white.svg" alt="">
          </a>
        </div>
      </div>
```

- [ ] **Шаг 6: замерить и сверить**

`browser_navigate` (перезагрузить), затем на каждой ширине — замерная функция.

| Ширина | `toggleW` | `listW` | `listH` | `platesVisible` | `docScrollX` |
|---|---|---|---|---|---|
| 1025 | 82,86 | 57,72 | 215,88 | 4 | false |
| 1200 | 86,20 | 60,26 | 170,79 | 3 | false |
| 1440 | 90,75 | 63,75 | 181,25 | 3 | false |
| 2560 | 112,00 | 80,00 | 230,00 | 3 | false |

На 1200 и выше плашек три — правило `.sw-header__plate--phone { display: none }`
ещё действует, снимается в Task 3. Это ожидаемо, не дефект.

`barInner` и `logoW` обязаны совпасть со снятыми на шаге 1: раскладка панели
на этой задаче не менялась.

Допуск ±0,5px — округление подпиксельной раскладки.

- [ ] **Шаг 7: глазами**

`browser_take_screenshot` на 1440. Проверить: триггер стоит на месте прежней
пилюли, светлый, с тёмным глифом телефона и серой кареткой вниз. Список не виден.

- [ ] **Шаг 8: коммит**

```bash
git add index.html
git commit -m "feat(header): триггер каналов связи вместо пилюли из четырёх плашек

Пилюля заменена одной плашкой-триггером с выпадающим списком. Габарит
триггера 82,86 на 1025 против 215,88 у пилюли. Граница состава пока
не тронута, поведение раскрытия — следующей задачей.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>"
```

---

## Task 2: Поведение раскрытия

**Файлы:**
- Modify: `index.html:~9364` — новый блок 5 внутри IIFE секции, перед `})();`

**Интерфейсы:**
- Потребляет из Task 1: `.sw-header__channels-wrap`, `.sw-header__channels-toggle`,
  `.sw-header__channels`, `.sw-header__channels--open`, `aria-expanded`.
- Отдаёт Task 4: константу границы `769` в `matchMedia` — она обязана совпасть
  с границей появления компонента в стилях.

- [ ] **Шаг 1: убедиться, что раскрытия ещё нет**

`browser_resize` 1440×900, `browser_click` по триггеру, затем:

```js
() => {
  const t = document.querySelector('.sw-header__channels-toggle');
  const l = document.querySelector('.sw-header__channels');
  return { expanded: t.getAttribute('aria-expanded'),
           open: l.classList.contains('sw-header__channels--open'),
           visibility: getComputedStyle(l).visibility };
}
```

Ожидание: `{ expanded: "false", open: false, visibility: "hidden" }` — клик
пока ничего не делает.

- [ ] **Шаг 2: вписать блок 5**

Вставить перед строкой `})();`, закрывающей IIFE GlobalHeader
(`index.html:9365`), после блока 4 «Пункты меню: text-roll»:

```js
  /* ── 5. Каналы связи: раскрытие списка ────────────────────────────────────
     Своё состояние секции, не контракт: Mobile_menu (SEC-14) о списке не знает,
     список о нём не знает. Общего состояния между секциями не появляется.
     Библиотек блок не использует — GSAP здесь не нужен и не проверяется.
     Единственный источник правды о состоянии — атрибут aria-expanded на кнопке:
     по нему же рисует CSS, второго источника в виде отдельной переменной нет. */
  var channelsWrap = header.querySelector('.sw-header__channels-wrap');
  var channelsToggle = channelsWrap && channelsWrap.querySelector('.sw-header__channels-toggle');
  var channelsList = channelsWrap && channelsWrap.querySelector('.sw-header__channels');

  if (channelsWrap && channelsToggle && channelsList) {

    /* Граница обязана совпадать с границей появления компонента в стилях
       секции — при её пересмотре правятся обе точки. Та же связка, что
       у text-roll в блоке 4. */
    var channelsMQ = window.matchMedia('(min-width: 769px)');

    var isChannelsOpen = function () {
      return channelsToggle.getAttribute('aria-expanded') === 'true';
    };

    var setChannelsOpen = function (next) {
      if (next === isChannelsOpen()) return;
      channelsToggle.setAttribute('aria-expanded', next ? 'true' : 'false');
      if (next) channelsList.classList.add('sw-header__channels--open');
      else channelsList.classList.remove('sw-header__channels--open');
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

    /* pointerdown, а не click: закрытие обязано срабатывать до того, как
       клик дойдёт до элемента под списком. Клик по самой кнопке сюда
       не попадает — она внутри обёртки. */
    document.addEventListener('pointerdown', function (event) {
      if (!isChannelsOpen()) return;
      if (channelsWrap.contains(event.target)) return;
      setChannelsOpen(false);
    });

    /* Уход фокуса за пределы обёртки — закрытие. Перехвата фокуса нет и
       не нужно: список стоит сразу за кнопкой в DOM, Tab идёт по нему сам. */
    channelsWrap.addEventListener('focusout', function (event) {
      if (!isChannelsOpen()) return;
      if (event.relatedTarget && channelsWrap.contains(event.relatedTarget)) return;
      setChannelsOpen(false);
    });

    /* Ниже 769 компонента на странице нет — открытым его оставлять нельзя.
       addListener — фолбэк для Safari до 14, addEventListener там отсутствует. */
    var onChannelsMQ = function () {
      if (!channelsMQ.matches) setChannelsOpen(false);
    };

    if (channelsMQ.addEventListener) channelsMQ.addEventListener('change', onChannelsMQ);
    else if (channelsMQ.addListener) channelsMQ.addListener(onChannelsMQ);
  }
```

- [ ] **Шаг 3: проверить пять способов закрытия**

Перезагрузить, `browser_resize` 1440×900. Для каждого сценария снимать
состояние функцией из шага 1.

| Сценарий | Действия | Ожидание |
|---|---|---|
| Открытие | клик по триггеру | `expanded: "true"`, `open: true`, `visibility: "visible"` |
| Повторный клик | клик по триггеру ещё раз | `expanded: "false"`, `visibility: "hidden"` |
| Esc | открыть, `browser_press_key Escape` | закрыт; активный элемент — триггер |
| Клик вне | открыть, клик по логотипу | закрыт |
| Уход фокуса | открыть, Tab до выхода за список | закрыт |
| Ресайз | открыть, `browser_resize` 500×900 | закрыт |

Проверка возврата фокуса после Esc:

```js
() => document.activeElement.className
```

Ожидание: строка содержит `sw-header__channels-toggle`.

- [ ] **Шаг 4: проверить клавиатуру и консоль**

`browser_console_messages` — новых ошибок нет.

Пройти Tab от триггера: фокус обязан пройти по всем четырём плашкам списка при
открытом состоянии и **перепрыгнуть их** при закрытом (`visibility: hidden`
убирает из последовательности).

- [ ] **Шаг 5: коммит**

```bash
git add index.html
git commit -m "feat(header): раскрытие списка каналов связи

Пять способов закрытия: повторный клик, Esc с возвратом фокуса, клик вне,
уход фокуса, переход ниже 769. Источник правды о состоянии один -
атрибут aria-expanded, по нему же рисует CSS.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>"
```

---

## Task 3: Плашка телефона в списке на ≥1200

Реализует раздел 5 спецификации «Принятые допущения». Отдельная задача именно
потому, что решение спорное и может быть отклонено без ущерба соседним.

**Файлы:**
- Modify: `index.html:1110-1111` (блок `min-width: 1200px`)

- [ ] **Шаг 1: замер «до» на 1200**

`browser_resize` 1200×900, замерная функция.
Ожидание: `platesVisible: 3`, `listH: 170.79`.

- [ ] **Шаг 2: снять правило**

Удалить из блока `@media (min-width: 1200px)`:

```css
  /* Телефон при полном составе — текстом в правой зоне, плашка из пилюли уходит. */
  .sw-header__plate--phone { display: none; }
```

И в комментарии-шапке того же блока (`index.html:1080-1091`) заменить строку
«приходят текстовый номер и вторичная ссылка, плашка телефона уходит из пилюли»
на:

```
   На этой границе в правую зону приходят текстовый номер и вторичная ссылка.
   Плашка телефона из списка каналов больше не убирается: правило стояло, пока
   плашка и номер стояли в одном визуальном ряду. За кликом ряда больше нет,
   а список из четырёх каналов, теряющий один элемент по ширине, непоследователен.
   Дублирование телефона остаётся, но разнесено по слоям.
```

- [ ] **Шаг 3: замерить**

| Ширина | `platesVisible` | `listH` |
|---|---|---|
| 1200 | 4 | 226,06 |
| 1440 | 4 | 240,00 |
| 2560 | 4 | 305,00 |

- [ ] **Шаг 4: коммит**

```bash
git add index.html
git commit -m "feat(header): плашка телефона остаётся в списке каналов на 1200 и выше

Правило display: none стояло, пока плашка и текстовый номер были в одном
визуальном ряду. За кликом ряда нет, а список, теряющий один канал
по ширине, непоследователен.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>"
```

---

## Task 4: Перенос границы состава

**Ни одна формула не меняется.** Медиаблок делится надвое: композиция переезжает
на 769, метрика страницы остаётся на 1025. Обоснование и замеры — раздел 4
спецификации.

Ошибка здесь тихая: если утащить метрику страницы вместе с композицией, вертикальные
края шапки разойдутся с остальными секциями на всей полосе 769–1024, а в консоли
не будет ничего.

**Файлы:**
- Modify: `index.html:938-1078` (блок `min-width: 1025px` — делится надвое)
- Modify: `index.html:843-852` (комментарий бургера)
- Modify: `index.html:615, 684` (комментарии «в панели с 1025»)
- Modify: `index.html:9292-9309` (граница в `gsap.matchMedia()` для text-roll)

**Интерфейсы:**
- Потребляет из Task 2: границу `769` в `matchMedia` — обе точки обязаны совпасть.

- [ ] **Шаг 1: замер «до» на границе**

`browser_resize` 768×900 и 769×900. Замерная функция на обеих.

Ожидание до правки: на 769 состав компактный — `navW: null` (меню `display: none`),
`toggleW: null`. Записать `barHeight`, `barPadding`, `logoImgH` и padding самого
`.sw-header`:

```js
() => getComputedStyle(document.querySelector('.sw-header')).padding
```

На 768 и 769 он обязан быть `0px 9px 4.8px` и **остаться таким после правки**.
Это контрольная величина всей задачи.

- [ ] **Шаг 2: вынести метрику страницы в отдельный блок**

Первым правилом блока `@media (min-width: 1025px)` (`index.html:939-941`) стоит
не композиция, а метрика страницы. **Вырезать это правило из блока** и положить
отдельным блоком сразу после закрывающей скобки композиционного блока:

```css
/* ─── 1025px и выше: метрика страницы ─── */
/* Боковой отступ страницы и нижний зазор до следующей секции. По
   design_system 7 они уменьшены на 40% в диапазоне <1024 и восстанавливаются
   блоком min-width: 1025px в КАЖДОЙ секции — вертикальные края всех секций
   обязаны совпадать.
   ЭТА ГРАНИЦА НЕ ПЕРЕЕХАЛА НА 769 ВМЕСТЕ С СОСТАВОМ ПАНЕЛИ и переехать не может:
   шапка получила бы полную метрику там, где остальные секции ещё на уменьшенной,
   и края разошлись бы на всей полосе 769–1024. Замер при попытке переноса:
   padding стал 0 15px 8px вместо 0 9px 4.8px, внутренняя ширина панели на 769
   упала с 674,0 до 662,0, логотип раздавило до 15,3.
   Композиция секции переключается на 768/769 — это другой блок, выше. */
@media (min-width: 1025px) {
  .sw-header {
    padding: 0 clamp(15px, 1.6287vw - 1.69px, 40px) clamp(8px, 2.0847vw - 13.37px, 40px);
  }
}
```

- [ ] **Шаг 3: сменить границу композиционного блока**

В оставшемся блоке заменить `@media (min-width: 1025px)` на
`@media (min-width: 769px)`.

**Ни одну формулу внутри не трогать.** Все рампы на 769 уже стоят на своих полах —
проверено замером: высота панели 72, padding панели 30, зазор «логотип → меню» 28,
зазор пунктов меню 12, высота логотипа 58, плашка 44, глиф 22. Компактные клампы
по другую сторону границы дают на 768 ровно те же величины, разрыва нет.

Заголовок блока переписать:

```
/* ─── 769px и выше: в панель приходят меню и правая зона ─── */
/* ГРАНИЦА СОСТАВА ВЕРНУЛАСЬ С 1024/1025 НА 768/769 вместе со свёртыванием
   каналов связи в один триггер: пилюля из четырёх плашек на 769 требовала 201px
   при бюджете 108,7, триггер требует 78,0.
   ФОРМУЛЫ ЭТОГО БЛОКА НЕ ПЕРЕАНКЕРОВАНЫ И НЕ ТРЕБУЮТ ЭТОГО. Каждая прямая уходит
   ниже своего пола раньше 1025, поэтому на 769 все они стоят на полах: 72 / 30 /
   28 / 12 / 58 / 44 / 22. Компактные клампы дают на 768 те же значения — разрыва
   на границе нет. На 769–1024 величины стоят плоско на полу и начинают расти
   с 1025: это потолок клампа, а не заморозка.
   Метрика страницы (боковой отступ, нижний зазор) в этом блоке больше не живёт —
   она осталась на 1025 отдельным блоком ниже, вместе со всеми секциями. */
```

- [ ] **Шаг 4: поправить границу в JS**

В блоке 4 `gsap.matchMedia()` (`index.html:9309`) заменить:

```js
        '(hover: hover) and (min-width: 1025px) and (prefers-reduced-motion: no-preference)',
```

на:

```js
        '(hover: hover) and (min-width: 769px) and (prefers-reduced-motion: no-preference)',
```

И в комментарии выше (`index.html:9296-9300`) заменить «Граница 1025» на
«Граница 769» с пояснением, что она переехала вместе с составом шапки.

- [ ] **Шаг 5: поправить комментарии о составе**

Три места в стилях говорят «в панели с 1025» / «до 1024 включительно»:
`index.html:615` (меню), `684` (правая зона), `843-852` (бургер). Комментарий
пилюли (`754`) уже переписан в Task 1. Привести к 769/768. В комментарии бургера
переписать следствие для SEC-14:

```text
   БУРГЕР ЖИВЁТ ДО 768 ВКЛЮЧИТЕЛЬНО. Граница вернулась с 1024 на 768 вместе
   со свёртыванием каналов связи в один триггер: пилюля из четырёх плашек
   на 769 требовала 201px при бюджете 108,7, триггер требует 78,0.
   Следствие для SEC-14: мобильное меню покрывает диапазон до 768 включительно
   и обязано нести все четыре канала связи — ниже 769 их в панели нет.
```

- [ ] **Шаг 6: проверить непрерывность на границе**

`browser_resize` 768×900 и 769×900, замерная функция на обеих.

| Величина | 768 | 769 |
|---|---|---|
| padding `.sw-header` | `0px 9px 4.8px` | `0px 9px 4.8px` |
| `barHeight` | 72px | 72px |
| `barPadding` | 30px | 30px |
| `logoImgH` | 58px | 58px |
| `logoW` | 119,6 | 119,6 |

**Первая строка — главная проверка задачи.** Если padding на 769 стал
`0px 15px 8px`, метрика страницы уехала вместе с композицией: блок разделён
неверно, шапка разошлась краями со всеми секциями ниже.

**Разрыв допустим только в составе:** на 768 `navW: null` и виден бургер, на 769
`navW: 385,7` и виден триггер. Разрыв в пяти величинах таблицы — дефект.

- [ ] **Шаг 7: проверить помещаемость по всем ширинам**

| Ширина | Ожидание |
|---|---|
| 769 | `navW: 385,7`; `navGap: 12px`; `toggleW: 78,00`; `barInner: 674,0`; `docScrollX: false` |
| 1024 | padding `.sw-header` всё ещё `0px 9px 4.8px`; `docScrollX: false` |
| 1025 | padding `.sw-header` стал `0px 15px 8px` — метрика страницы включилась здесь |
| 1199 | `docScrollX: false` |
| 1200 | `logoW === logoNatural`; `docScrollX: false` |
| 1440 | `logoW === logoNatural`; `docScrollX: false` |
| 1920 | `logoW === logoNatural`; `docScrollX: false` |
| 2560 | `barHeight: 150px`; `barPadding: 90px`; `logoImgH: 104px`; `navGap: 80px` |

Строки 1024 и 1025 проверяют, что **метрика страницы осталась на своей границе** —
она обязана переключиться именно здесь, а не на 769.

**`logoW === logoNatural` на 1200, 1440, 1920 — это закрытие открытого дефекта
ужатия логотипа.** До правки на 1200 было 88,06 против натуральных 130,44.

- [ ] **Шаг 8: проверить text-roll и мобильное состояние**

На 900×900 навести на пункт меню — text-roll обязан работать (раньше на этой
ширине меню не было вовсе).

На 500×900 — виден бургер, меню и триггера нет, `docScrollX: false`.

`browser_console_messages` — новых ошибок нет.

- [ ] **Шаг 9: коммит**

```bash
git add index.html
git commit -m "feat(header): граница состава с 1024/1025 на 768/769

Медиаблок разделён надвое: композиция переехала на 769, метрика страницы
(боковой отступ, нижний зазор) осталась на 1025 вместе со всеми секциями.
Утащить её на 769 нельзя - края шапки разошлись бы с секциями ниже.

Формулы не переанкерованы и не требуют этого: каждая рампа уходит ниже
своего пола раньше 1025, поэтому на 769 все стоят на полах, а компактные
клампы дают на 768 те же значения. Разрыва на границе нет.

Граница в gsap.matchMedia() для text-roll приведена к тому же значению.

Логотип на 1200-1900 больше не ужимается: на 1200 было 88,06 против
натуральных 130,44, стало 130,44.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>"
```

---

## Task 5: Документация

Правки в коде без этой задачи разойдутся с документацией молча.

Правки в коде без этой задачи разойдутся с документацией молча. **Объём правки
меньше, чем кажется** — проверено чтением документов, а не по памяти.

**Файлы:**
- Modify: `CLAUDE.md` раздел 1 (состав шапки) и раздел 4 (реестр границ)
- Modify: `Design_system/design_system.md` 2.7, строка 269
- Проверить, править не требуется: `Documentation/PRD.md`

- [ ] **Шаг 1: CLAUDE.md — состав шапки и реестр границ**

В разделе 4, абзац «Границ в проекте четыре»: строка про 1024 гласит
«1024 (композиция SEC-3, зазоры SEC-3 и SEC-4)». **Она остаётся как есть** —
шапка в ней не упомянута, а 1024 продолжает работать в SEC-2 на метрике страницы.

Строка про 1200 гласит «1200 (состав правой зоны шапки)» — **тоже остаётся**:
1200 по-прежнему переключает приход блока номера.

Добавить в раздел 1, к описанию собранных секций, состав шапки после правки:

```text
**Состав GlobalHeader по ширинам:** ≤768 логотип и бургер; 769–1199 логотип,
меню и триггер каналов связи; ≥1200 добавляется блок номера. Каналы связи
свёрнуты в список за триггером на всех ширинах ≥769. Композиция секции
переключается на 768/769 и 1200; 1024/1025 в секции осталась только за
метрикой страницы, общей у всех секций.
```

- [ ] **Шаг 2: design\_system.md 2.7, строка 269**

Строка сейчас:

```text
| `1024 / 1025` | композицию SEC-3 в две колонки; нижний зазор SEC-3; `row-gap` SEC-4; композицию шапки SEC-5; число колонок каталога | код секций | SEC-3, SEC-4, SEC-5 |
```

Формулировка «композицию шапки SEC-5» относится к шапке каталога, не к SEC-2 —
GlobalHeader в реестре 1024 не числился никогда. **Менять её не нужно.**

Что нужно: в строке границы `768 / 769` добавить SEC-2 в список потребителей —
на неё переехал состав панели. Найти строку:

```bash
grep -n '768 / 769' Design_system/design_system.md
```

И в разделе 12 «долги документации» снять запись о том, что состав шапки
переключается на 1024, если она там есть:

```bash
grep -n 'шапк' Design_system/design_system.md | grep -n '1024'
```

- [ ] **Шаг 3: PRD — проверить, что правка не нужна**

```bash
grep -n 'SEC-14' Documentation/PRD.md
```

Строка 123 уже гласит «Mobile\_menu — полноэкранное меню **на ≤768**, открывается
кнопкой бургера из GlobalHeader». PRD никогда не переписывался под границу 1024 —
после этой работы он снова верен. **Править нечего, только убедиться.**

Если найдётся другое место с ≤1024 для SEC-14 — привести к ≤768, требование нести
все четыре канала связи оставить: оно по-прежнему верно.

- [ ] **Шаг 4: коммит**

```bash
git add CLAUDE.md Design_system/design_system.md
git commit -m "docs: состав шапки после переноса границы на 768/769

Граница 1024 из реестра не уходит: в SEC-2 она осталась за метрикой страницы,
в SEC-3 и SEC-4 - за композицией. С состава панели она снята, состав
переключается на 768/769.

PRD правки не потребовал: он всё это время описывал SEC-14 как меню на 768
и после этой работы снова верен.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>"
```

---

## Финальная приёмка

Прогнать критерии из раздела 8 спецификации целиком, на живом цехе:

- [ ] 769, 1024, 1025, 1199, 1200, 1440, 1920, 2560 — панель не переполняется,
      `docScrollX: false` на каждой
- [ ] 1200, 1440, 1920 — `logoW === logoNatural`
- [ ] 768 и 769 — высота панели, высота логотипа, padding панели **и padding
      `.sw-header`** совпадают; последний равен `0px 9px 4.8px` на обеих
- [ ] 1024 → 1025 — padding `.sw-header` переключается `0px 9px 4.8px` → `0px 15px 8px`:
      метрика страницы осталась на своей границе
- [ ] формулы рамп не менялись — `git diff` по блоку не содержит правок `clamp()`
- [ ] список проходит все шесть сценариев из Task 2 шаг 3 (открытие и пять закрытий)
- [ ] `aria-expanded` соответствует фактическому состоянию
- [ ] переход ниже 769 закрывает открытый список
- [ ] все четыре канала достижимы с клавиатуры, обводка фокуса видна
- [ ] `prefers-reduced-motion: reduce` — перехода нет, состояние различимо
- [ ] стоп-лист `CLAUDE.md` раздел 3 не нарушен: `grep -n "^body\|^\*\s*{\|@font-face\|jQuery\|\bimport\b" index.html`
      не даёт новых совпадений в диапазоне секции

---

## Отступления от спецификации

**1. Скрытие списка.** Раздел 3.4 говорит скрывать список атрибутом `hidden`.
В плане он скрывается через `visibility: hidden`. Причина: `hidden` даёт
`display: none`, и переход появления при снятии атрибута не проигрывается без
принудительного лишнего кадра. `visibility` так же убирает содержимое из
последовательности фокуса, но анимируется. Функционально требование раздела 3.4
выполняется целиком.

**2. Переанкеровка рамп.** Ранняя редакция раздела 4 спецификации требовала
переанкеровать пять рамп с 1025 на 769. Это было неверно, раздел переписан по
результату замера. Все рампы на 769 уже стоят на своих полах, разрыва на границе
нет, формулы не трогаются. Вместо переанкеровки медиаблок делится надвое, чтобы
метрика страницы осталась на 1024/1025 вместе со всеми остальными секциями.
План соответствует переписанной редакции.
