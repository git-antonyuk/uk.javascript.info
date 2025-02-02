
# Fetch API

Станом на зараз ми вже дещо знаємо про `fetch`.

Розгляньмо решту API, щоб охопити всі його можливості.

```smart
Зверніть увагу: більшість з цих опцій використовуються рідко. Ви можете пропустити цей розділ і все одно без проблем користуватися `fetch`.

Тим не менш, корисно знати, що може робити `fetch`, тому якщо виникне така необхідність, завжди можна повернутись до цієї статті і розібратись детальніше.
```

Ось повний список всіх можливих опцій `fetch` з їх типовими значеннями (альтернативи в коментарях):

```js
let promise = fetch(url, {
  method: "GET", // POST, PUT, DELETE та інші
  headers: {
    // значення цього заголовку зазвичай встановлюється автоматично,
    // залежно від тіла запиту
    "Content-Type": "text/plain;charset=UTF-8"
  },
  body: undefined // string, FormData, Blob, BufferSource або URLSearchParams
  referrer: "about:client", // або "", щоб не посилати заголовок Referer,
  // або URL з поточного джерела
  referrerPolicy: "no-referrer-when-downgrade", // no-referrer, origin, same-origin...
  mode: "cors", // same-origin, no-cors
  credentials: "same-origin", // omit, include
  cache: "default", // no-store, reload, no-cache, force-cache або only-if-cached
  redirect: "follow", // manual, error
  integrity: "", // контрольна сума, наприклад "sha256-abcdef1234567890"
  keepalive: false, // true
  signal: undefined, // AbortController, щоб перервати запит
  window: window // null
});
```

Перелік вражає, чи не так?

Ми повністю висвітлили `method`, `headers` та `body` у главі <info:fetch>.

Опція `signal` висвітлена у главі <info:fetch-abort>.

Тепер розгляньмо решту можливостей.

## referrer, referrerPolicy

Ці опції вказують, як `fetch` встановлює HTTP-заголовок `Referer`.

Зазвичай цей заголовок встановлюється автоматично і містить адресу сторінки, з якої був зроблений запит. У більшості сценаріїв він взагалі не важливий, іноді, з метою безпеки, є сенс видалити або скоротити його.

**Опція `referrer` дозволяє задати будь-який `Referer` (в межах поточного джерела) або видалити його.**.

Щоб не надсилати `Referer`, задайте порожній рядок:
```js
fetch('/page', {
*!*
  referrer: "" // немає заголовку Referer
*/!*
});
```

Щоб встановити іншу URL-адресу в межах поточного джерела:

```js
fetch('/page', {
  // припускаючи, що ми на сторінці https://javascript.info
  // ми можемо задати будь-який заголовок Referer, але тільки в межах поточного джерела
*!*
  referrer: "https://javascript.info/anotherpage"
*/!*
});
```

**Опція `referrerPolicy` задає загальні правила для `Referer`.**.

Запити поділяються на 3 типи:

1. Запит на те ж джерело.
2. Запит на інше джерело.
3. Запит з HTTPS на HTTP (з безпечного на небезпечний протокол).

На відміну від опції `referrer`, яка дозволяє задати конкретне значення `Referer`, `referrerPolicy` вказує браузеру загальні правила для кожного типу запиту.

Можливі значення описані у [Referrer Policy specification](https://w3c.github.io/webappsec-referrer-policy/):

- **`"no-referrer-when-downgrade"`** -- типове значення: повний `Referer` надсилається завжди, хіба що ми не надсилаємо запит із HTTPS на HTTP (на менш безпечний протокол).
- **`"no-referrer"`** -- ніколи не надсилати `Referer`.
- **`"origin"`** -- надсилати лише джерело в `Referer`, а не повну URL-адресу сторінки, наприклад лише `http://site.com` замість `http://site.com/path`.
- **`"origin-when-cross-origin"`** -- надсилати повний `Referer` для запитів до однакового джерела, але лише частину джерела для запитів між різними джерелами (як показано вище).
- **`"same-origin"`** -- надсилати повний `Referer` для запитів до однакового джерела, але без `Referer` для запитів між різними джерелами.
- **`"strict-origin"`** -- надсилати лише джерело, а не `Referer` для HTTPS→HTTP запитів.
- **`"strict-origin-when-cross-origin"`** -- для запитів до однакового джерела надсилати повний `Referer`, для запитів між різними джерелами надсилати лише джерело, якщо це не HTTPS→HTTP запит, тоді нічого не надсилати.
- **`"unsafe-url"`** -- завжди надсилати повну URL-адресу в `Referer`, навіть для HTTPS→HTTP запитів.

Ось таблиця зі всіма комбінаціями:

| Значення                                                   | До того ж джерела | До іншого джерела | HTTPS→HTTP |
|------------------------------------------------------------|-------------------|-------------------|------------|
| `"no-referrer"`                                            | -                 | -                 | -          |
| `"no-referrer-when-downgrade"` або `""` (типове значення) | full              | full              | -          |
| `"origin"`                                                 | origin            | origin            | origin     |
| `"origin-when-cross-origin"`                               | full              | origin            | origin     |
| `"same-origin"`                                            | full              | -                 | -          |
| `"strict-origin"`                                          | origin            | origin            | -          |
| `"strict-origin-when-cross-origin"`                        | full              | origin            | -          |
| `"unsafe-url"`                                             | full              | full              | full       |

Припустімо, у нас є зона адміністрування зі структурою URL-адреси, яка не повинна бути відома за межами сайту.

Якщо ми надсилаємо `fetch`, то типово він завжди надсилає заголовок `Referer` із повною URL-адресою нашої сторінки (окрім випадків, коли ми надсилаємо запит із HTTPS на HTTP, тоді без `Referer`).

Наприклад, `Referer: https://javascript.info/admin/secret/paths`.

Якщо ми хочемо, щоб інші веб-сайти знали лише джерело, а не URL-шлях, ми можемо задати опцію:

```js
fetch('https://another.com/page', {
  // ...
  referrerPolicy: "origin-when-cross-origin" // Referer: https://javascript.info
});
```

Ми можемо додавати його до всіх викликів `fetch`, навіть інтегрувати в JavaScript бібліотеку нашого проекту, яка відповідає за запити та використовує `fetch` всередині.

Його єдина відмінність у порівнянні з типовою поведінкою полягає в тому, що для запитів до іншого джерела `fetch` надсилає лише вихідну частину URL-адреси (наприклад, `https://javascript.info`, без шляху). Для запитів до нашого джерела ми все ще отримуємо повний `Referer` (може бути корисним для задач з налагодження).

```smart header="Політика встановлення Referrer (Referrer Policy) стосується не тільки `fetch`"
Політика встановлення Referrer, що описана у [специфікації](https://w3c.github.io/webappsec-referrer-policy/), стосується не лише `fetch`, а є більш глобальною.

Зокрема, можна встановити типову політику для всієї сторінки за допомогою HTTP-заголовка `Referrer-Policy` або окремого посилання за допомогою `<a rel="noreferrer">`.
```

## mode

Опція `mode` є засобом безпеки, що запобігає випадковим запитам між різними джерелами:

- **`"cors"`** -- запити між різними джерелами типово дозволені, як описано в <info:fetch-crossorigin>,
- **`"same-origin"`** -- запити між різними джерелами заборонені,
- **`"no-cors"`** -- дозволені лише безпечні запити між різними джерелами.

Ця опція може бути корисною, якщо URL-адреса для `fetch` надходить від третьої сторони, і ми хочемо, щоб "глобальний перемикач" обмежував можливість запитів між різними джерелами.

## credentials

Опція `credentials` визначає, чи має `fetch` надсилати файли cookie та заголовки HTTP-Authorization разом із запитом.

- **`"same-origin"`** -- типове значення, не надсилати запити між різними джерелами,
- **`"include"`** -- завжди надсилати, вимагає `Access-Control-Allow-Credentials` від сервера іншого джерела, щоб JavaScript міг отримати доступ до відповіді, про що йдеться у главі <info:fetch-crossorigin>,
- **`"omit"`** -- ніколи не надсилати, навіть для запитів з однаковим джерелом.

## cache

Типово запити `fetch` використовують стандартне HTTP-кешування. Тобто він визнає заголовки `Expires` і `Cache-Control`, надсилає `If-Modified-Since` і т.д. Так само як роблять звичайні HTTP-запити.

Опції `cache` дозволяють ігнорувати HTTP-кеш або тонко налаштувати його використання:

- **`"default"`** -- `fetch` використовує стандартні правила HTTP-кешу та заголовки,
- **`"no-store"`** -- повністю ігнорувати HTTP-кеш, цей режим стане типовим, якщо ми встановимо заголовок `If-Modified-Since`, `If-None-Match`, `If-Unmodified -Since`, `If-Match` або `If-Range`,
- **`"reload"`** -- не брати результат із HTTP-кешу (якщо є), а заповнити кеш відповіддю (якщо заголовки відповіді дозволяють цю дію),
- **`"no-cache"`** -- створити умовний запит, якщо є кешована відповідь, і звичайний запит в іншому випадку. Заповнити HTTP-кеш відповіддю,
- **`"force-cache"`** -- використовувати відповідь з HTTP-кешу, навіть якщо він застарілий. Якщо в HTTP-кеші немає відповіді, зробити звичайний HTTP-запит, поводитися як зазвичай,
- **`"only-if-cached"`** -- використовувати відповідь з HTTP-кешу, навіть якщо він застарілий. Якщо в HTTP-кеші немає відповіді, то видати помилку. Працює лише тоді, коли `mode` має значення `"same-origin"`.

## redirect

Зазвичай `fetch` прозоро слідує HTTP-переадресаціям, таким як 301, 302 тощо.

Опція `redirect` дозволяє змінити це:

- **`"follow"`** -- типове значення, слідувати HTTP-переадресаціям,
- **`"error"`** -- помилка при HTTP-переадресації,
- **`"manual"`** -- дозволяє обробляти HTTP-переадресації вручну. У разі переадресації ми отримаємо спеціальний об’єкт відповіді з `response.type="opaqueredirect"` та нульовим/порожнім статусом і більшістю інших властивостей.

## integrity

Опція `integrity` дозволяє перевірити, чи відповідає відповідь контрольній сумі відомій наперед.

Як описано в [специфікації](https://w3c.github.io/webappsec-subresource-integrity/), хеш-функціями, що підтримуються, є SHA-256, SHA-384 і SHA-512, залежно від браузера.

Наприклад, ми завантажуємо файл і знаємо, що його контрольна сума SHA-256 дорівнює "abcdef" (справжня контрольна сума, звичайно, довша).

Ми можемо розмістити його в опції `integrity`, наприклад:

```js
fetch('http://site.com/file', {
  integrity: 'sha256-abcdef'
});
```

Далі `fetch` самостійно обчислить SHA-256 і порівняє його з нашим рядком. У разі розбіжності буде згенерована помилка.

## keepalive

Опція `keepalive` вказує на те, що запит має "пережити" веб-сторінку, яка його ініціювала.

Наприклад, ми збираємо статистику про те, як поточний відвідувач використовує нашу сторінку (кліки миші, фрагменти сторінки, які він переглядає), для аналізу і поліпшення користувацького досвіду.

Коли відвідувач залишає нашу сторінку -- ми хотіли б зберегти дані на нашому сервері.

Для цього ми можемо використовувати подію `window.onunload`:

```js run
window.onunload = function() {
  fetch('/analytics', {
    method: 'POST',
    body: "statistics",
*!*
    keepalive: true
*/!*
  });
};
```

Зазвичай, коли документ вивантажується, всі пов'язані з ним мережеві запити перериваються. Але опція `keepalive` говорить браузеру виконувати запит у фоновому режимі, навіть після того, як він залишить сторінку. Отже, ця опція необхідна для успішного виконання нашого запиту.

Вона має кілька обмежень:

- Ми не можемо відправляти мегабайти: ліміт тіла для запитів `keepalive` становить 64КБ.
    - Якщо нам потрібно зібрати багато статистики про відвідування, ми повинні надсилати її регулярно пакетами, щоб на останній запит `onunload` не залишилося багато.
    - Це обмеження стосується всіх `keepalive`-запитів разом. Іншими словами, ми можемо паралельно виконувати декілька запитів `keepalive`, але сума довжин їхніх тіл не повинна перевищувати 64КБ.
- Ми не зможемо обробити відповідь сервера, якщо документ буде вивантажений. Тому в нашому прикладі `fetch` пройде успішно завдяки `keepalive`, але наступні функції не працюватимуть.
    - У більшості випадків, наприклад, при розсилці статистики, це не є проблемою, оскільки сервер просто приймає дані та зазвичай на такі запити відправляє порожню відповідь.
