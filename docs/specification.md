# Roxy: технічне завдання на query DSL та редактор

Статус: специфікація для перевірки гіпотези; реалізації ще немає.

Дата: 26 жовтня 2026 року.

Робоча назва мови: **Roxy**. Репозиторій: **roxy-lang**. Назви npm-пакетів у цьому документі є запропонованими, а не підтвердженням їхньої доступності або публікації.

## 1. Мета

Перевірити, чи може одне текстове поле з контрольованою англійською мовою, контекстним автодоповненням і підсвіткою замінити типовий UI-конструктор вибірки даних.

Користувач описує бажаний результат реченням. Бібліотека перевіряє його за наданим каталогом, перетворює на типізований запит і передає хосту для виконання. Roxy має бути придатною одночасно для людини, редактора та програмного клієнта, включно з MCP-адаптером хоста.

Roxy — незалежна TypeScript-бібліотека. Вона не знає про конкретний продукт, його API, ORM, базу даних, IndexedDB, спосіб автентифікації чи бізнес-сутності.

## 2. Основний сценарій

Користувач набирає:

```text
Get first 20 articles from Articles
where Articles tags includes Ne
```

Редактор знає, що `Articles.tags` — масив ID записів іншого джерела. Він запитує в провайдера підказки для `Ne`, показує результати й дозволяє вибрати `News`.

Після вибору користувач бачить:

```text
Get first 20 articles from Articles
where Articles tags includes [News]
order by Articles publication_date desc.
```

Квадратні дужки тут лише позначають візуальний токен у документації. Це не додаткова текстова граматика. Редактор може показувати токен як звичайну назву з підсвіткою або компактний інтерактивний фрагмент.

У документі зберігаються стабільні посилання:

```text
Get first 20 articles from source("articles.records")
where field("tags") includes ref("tags.tags", "tag-news-001")
order by field("publication_date") desc.
```

`articles` після кількості — нова змінна результату. `source("articles.records")` — джерело. `field("publication_date")` — стабільний ключ поля поточного джерела. `News` — лише показана назва запису `tag-news-001`.

Це одна модель документа з різними представленнями, а не два незалежні запити, які можуть розійтися.

## 3. Межі першої версії

### У межах

- Query-діалект із явним іменем результату.
- Каталог джерел, полів, типів, операторів, лімітів та зв'язків.
- Розбір завершених і незавершених речень.
- Типізоване дерево синтаксису, семантична перевірка, структуровані діагностики.
- Канонічний текст зі стабільними посиланнями та версією мови.
- Контекстні підказки ключових слів, джерел, полів, enum-значень, змінних і записів довідників.
- Асинхронне завантаження довідників на вимогу.
- Текстовий редактор, підсвітка, стабільні reference-токени, клавіатурна навігація.
- Валідація та виконання завершеного запиту через адаптер хоста.
- Демонстраційний застосунок на синтетичних даних і автоматичні тести.

### Поза межами

- `watch`, SSE, автоматичні підписки на зміни даних.
- Workflow-діалект і виконання дій; `if/else`, цикли та тригери поки лише майбутні можливості.
- Генерація чи виконання SQL, довільний JavaScript, `eval`, AI-інтерпретація речень.
- Довільні cross-source joins, `group by`, агрегації, обчислювані колонки.
- Побудова розмітки сторінок або компонентів.
- Інтеграція з конкретним продуктом, продакшен-деплой, публікація npm-пакетів.
- Другий візуальний query builder паралельно з редактором.

Збереження імені змінної не означає створення серверної сутності або глобально доступної saved query. Область видимості змінної — поточний документ/сесія виконання, визначена хостом.

## 4. Незмінні принципи

1. Однаковий канонічний документ за однакового каталогу дає однаковий план запиту.
2. Назви є представленням; джерела, поля, записи й enum-значення мають стабільні машинні ключі.
3. Неоднозначність не вирішується прихованою евристикою.
4. Перейменування показаної назви не змінює сенс запиту.
5. Незавершена чи невалідна редакція не замінює останній валідний результат новим запитом.
6. Незалежність від транспорту забезпечується інтерфейсами хоста, а не умовами на конкретні бекенди.
7. Будь-які ліміти й відсутні можливості повідомляються явно, а не мовчазним урізанням або видаленням умови.
8. Майбутні діалекти можуть використовувати спільні типи, значення, посилання та мовний сервіс, але не впроваджуються завчасно.

## 5. Архітектура пакетів

Запропонована структура:

```text
packages/
  core/               @roxy-lang/core
  query/              @roxy-lang/query
  language-service/   @roxy-lang/language-service
  editor/             @roxy-lang/editor
examples/
  playground/
```

### Core

Спільні типи документа, діапазонів тексту, літералів, параметрів, стабільних посилань, областей видимості та діагностик. Версійований envelope документа й правила серіалізації. Без DOM, React, мережі та імпортів із query-діалекту.

### Query

Граматика query-діалекту, AST, семантична перевірка за каталогом, тип результату, нормалізація та побудова незалежного плану запиту. Форматування тексту. Без DOM і виконання HTTP/SQL-запитів.

### Language service

Позиційні підказки для незавершеного документа, semantic tokens, hover, діагностики, операції вибору/перейменування символів. Повертає редагування тексту і прив'язку до стабільних ID; не повертає React-елементи.

### Editor

Текстове поле з мовними можливостями та відображенням reference-токенів. Типові keyboard shortcuts, undo/redo, робота з IME, clipboard і доступність. Рекомендована відправна точка для оцінки — CodeMirror 6; якщо інший зрілий редактор краще забезпечує ці вимоги, вибір треба обґрунтувати в специфікації перед реалізацією інтеграції.

UI-фреймворк не повинен проникати в core/query/language-service. React-wrapper, якщо потрібен для playground, залишається тонким адаптером або окремим entrypoint. Не створювати власний contenteditable-редактор без доведеної потреби.

### Playground

Невеликий окремий застосунок із редактором, поточними діагностиками, результатом і технічною вкладкою з canonical text/AST/query plan. Технічні вкладки призначені для перевірки гіпотези й не є вимогою до інтерфейсу продукту.

## 6. Публічні контракти хоста

Точні TypeScript-сигнатури агент деталізує, зберігаючи такі відповідальності.

```ts
interface RoxyHost {
  catalog: CatalogProvider;
  references: ReferenceProvider;
  executor?: QueryExecutor;
}

interface ReferenceOption {
  id: string;
  label: string;
  description?: string;
  color?: string;
}

interface ReferenceSearchRequest {
  sourceId: string;
  text: string;
  cursor?: string;
  limit: number;
  signal: AbortSignal;
}

interface ReferenceSearchPage {
  items: readonly ReferenceOption[];
  nextCursor?: string;
}

interface ReferenceProvider {
  search(request: ReferenceSearchRequest): Promise<ReferenceSearchPage>;
  resolve(request: {
    sourceId: string;
    ids: readonly string[];
    signal: AbortSignal;
  }): Promise<readonly ResolvedReference[]>;
}
```

`ResolvedReference` повинен представляти успішне вирішення і стан «недоступне або не знайдене»; транспортна помилка має окремий шлях. Не вимагати від хоста розкривати, чи запис видалено, чи заборонено доступом.

Каталог може бути асинхронним: отримання доступних джерел і опис конкретного джерела — окремі операції. Бібліотека не завантажує всі поля всіх джерел наперед. Для completion поля потрібен опис тільки відповідного джерела.

Executor отримує перевірений незалежний query plan, параметри й `AbortSignal`. Він повертає bounded connection або один запис. Саме хост перетворює план на власний API та повторно перевіряє повноваження. Відсутність executor не заважає парсингу, підказкам і статичній перевірці.

## 7. Каталог та описи зв'язків

Опис джерела включає:

- стабільний `id` і revision/schema version;
- показану назву, опис та явно оголошені aliases;
- поля зі стабільними ключами, типами, optional/nullable і допустимими операторами;
- enum-значення зі стабільним значенням та показаною назвою;
- поля пошуку, унікальні lookup-ключі, можливості list/getOne/seek/count;
- ліміти вікна, умов, сортувань і підрахунку;
- metadata зв'язків із довідниками.

Приклад незалежного опису поля:

```ts
{
  key: 'tags',
  label: 'tags',
  kind: 'array',
  element: 'string',
  reference: {
    sourceId: 'tags.tags',
    identityField: 'id',
    labelField: 'name',
    colorField: 'color'
  }
}
```

`reference` означає, що рядки є ID, а не звичайним текстом. Під час `tags includes ...` пропонуються записи цього джерела. Для рядкового поля без reference пропонуються літерали або enum, а не довідник тегів.

Каталог є канонічним місцем опису зв'язку. Не вимагати дублювання того самого mapping у query-редакторі. Хост може адаптувати вже наявні metadata своєї системи до незалежних типів Roxy.

Зміна label не потребує міграції. Видалення або зміна стабільного ключа поля вимагає явної міграції чи діагностики несумісності. Невідома версія документа не виконується як відома.

## 8. Мовна форма першої версії

Це контрольована англійська з фіксованими конструкціями, а не довільний natural-language parser. Keywords нечутливі до регістру; ID і string literals регістр зберігають. Переноси рядків і зайві пробіли не змінюють сенс.

Базова форма списку:

```text
Get [the] first <limit> <variable> from <source>
[with only <fields>]
[where <predicate>]
[searching for <text>]
[order by <field> <asc|desc>, ...]
[after <cursor> | starting at <record-key>]
[including total count].
```

Квадратні дужки в описі граматики означають необов'язкові частини. Порядок clauses у v1 фіксований; не реалізовувати перестановку всіх слів. Крапка в кінці дозволена, але не обов'язкова; крапки всередині quoted IDs і field paths не є кінцем речення.

Для backward window:

```text
Get [the] last <limit> <variable> from <source>
[with only <fields>]
[where <predicate>]
[searching for <text>]
[order by <field> <asc|desc>, ...]
[before <cursor>]
[including total count].
```

`after` сумісний лише з first; `before` — лише з last; `starting at` — лише з first і без іншої межі. Результат backward window зберігає канонічний порядок, а не порядок обходу.

Форма одного запису:

```text
Get <variable> from <source> by <unique-field> <value>
[with only <fields>]
[where <predicate>].
```

Ідентифікатор змінної: ASCII letter/underscore на початку, потім letters/digits/underscore. Keywords зарезервовані. Значення параметрів позначаються `$name`, передаються окремо й ніколи не підставляються конкатенацією тексту.

V1 підтримує один query statement у документі. Спільна модель символів має дозволяти майбутню композицію, але не потрібні багатокомандне виконання й посилання на результати інших запитів зараз.

`latest`, `newest` без поля, `published` як неоголошений бізнес-макрос та приховане визначення дати не входять у v1. Канонічні напрями — `asc`/`desc`; `ascending`/`descending` дозволені як однозначні синоніми.

## 9. Підтримувані операції та приклади

Усі показані назви в прикладах є fixtures каталогу; вони не є вбудованими сутностями Roxy. Нижче використовується display notation, якщо прямо не зазначено інше.

### 9.1. Поля та кілька сортувань

```text
Get first 20 articles from Articles
with only id, title, publication_date
where Articles status is Published
order by Articles publication_date desc, Articles title asc.
```

Виконання: membership filter/search, order, window, output selection. Позиція `first` у реченні не означає, що вікно береться до сортування. Стабільний tie-breaker джерела додає виконавець відповідно до свого контракту.

### 9.2. Предикати

Підтримати явні відповідності:

| Roxy | Незалежний predicate operator |
| --- | --- |
| `is`, `is not` | `eq`, `ne` |
| `is one of`, `is not one of` | `in`, `notIn` |
| `<`, `<=`, `>`, `>=` | `lt`, `lte`, `gt`, `gte` |
| `contains`, `starts with`, `ends with` для рядка | `contains`, `startsWith`, `endsWith` |
| `includes` для scalar array | `some` з одним значенням |
| `includes any of`, `includes all of` | `some`, `all` |
| `is empty` для array/many | `isEmpty` |
| `is null`, `is not null` | `isNull`, `isNotNull` |
| `is absent`, `is present` | `isAbsent`, `isPresent` |

Підтримати `and`, `or`, `not` і дужки. Пріоритет: `not`, потім `and`, потім `or`. Не змішувати відсутнє поле з null або порожнім масивом.

```text
Get first 20 articles from Articles
where (Articles status is Published or Articles status is Scheduled)
and Articles publication_date >= $startDate
and Articles publication_date < $endDate.
```

```text
Get first 20 articles from Articles
where Articles tags includes any of [News, Education].
```

Показані `News`/`Education` — reference-токени. У canonical text вони є `ref(...)`, а масив — масивом цих reference literals. Enum display labels аналогічно відображають стабільне enum-значення; не порівнювати за перекладеним label.

Дозволені scalar literals: quoted strings з escaping, finite numbers, booleans, null там, де це дозволено оператором. Дати та datetime перевіряються за форматом поля; мовні `today`/`last week` у v1 не вводити. Reference ID є opaque string, а не обов'язково UUID.

### 9.3. Вкладені поля та owned children

Підтримати оголошені scalar paths усередині object/one. Для many — явні `any ... satisfy`, `all ... satisfy`, `is empty`:

```text
Get first 20 orders from Orders
where any Orders items satisfy
(item productId is $productId and item quantity > 2).
```

`item` — локальний зарезервований символ вкладеного предиката. Completion працює за схемою дочірнього запису. Це не довільний cross-source join. Порожні масиви й missing-значення мають семантику незалежного плану та хост-контракту; особливо перевірити `all` над порожнім набором.

### 9.4. Lookup

```text
Get article from Articles by id $articleId.
```

```text
Get article from Articles by slug "welcome".
```

`slug` доступний лише якщо каталог оголосив unique lookup. Lookup повертає один запис або null. `first 1` повертає список і не є еквівалентом lookup.

### 9.5. Пошук

```text
Get first 20 articles from Articles
searching for "освіта".
```

Пошук дозволений лише якщо джерело його рекламує. Хост явно описує режим; початковий сумісний режим — literal case-sensitive substring по оголошених полях. Не обіцяти stemming, ranking або case-insensitive full-text search.

### 9.6. Навігація та count

```text
Get first 20 articles from Articles after $cursor.
```

```text
Get last 20 articles from Articles before $cursor.
```

```text
Get first 20 articles from Articles starting at id $articleId.
```

```text
Get first 20 articles from Articles
where Articles status is Published
including total count.
```

Cursor — opaque value, значення якого Roxy не інтерпретує. Зміна membership/order може зробити cursor невалідним; це помилка хоста, яку редактор показує й пропонує явно скинути межу. Не переносити cursor мовчки на інший запит.

Count доступний лише за capability джерела; повертається `{value, bounded}`. Відображати `1000+`, якщо досягнуто оголошеної межі, а не називати bounded count точним. Значення 1000 є лише fixture; бібліотека бере межу з каталогу.

## 10. AST і незалежний query plan

Розділити syntax tree незавершеного тексту, resolved semantic model і валідний execution plan. Наявність parse tree сама по собі не означає, що запит можна виконувати.

План включає mode list/record, result variable, source ID, schema revision, selected fields, typed predicate, search, order, boundary, requested count. Значення плану — literals, parameters або stable references. Runtime bindings параметрів передаються окремо.

Не називати план SQL AST. Він має бути придатним для адаптера до HTTP Query API, локального complete dataset або іншого engine. Адаптер не має отримувати необроблений display label як query operand.

Хост задає порівняння й політики джерела. Для демонстраційного адаптера зафіксувати однакову семантику: порівняння з null/absent не є звичайним `ne`; text comparison детермінований; null і absent сортуються last для asc/first для desc. Непідтриманий семантичний режим не підміняється іншим.

## 11. Stable references і persistence

Canonical source повинен бути самодостатнім: він містить всі stable IDs, потрібні для відновлення сенсу. Не зберігати зв'язок «позиція слова → ID» лише у зовнішньому sidecar: звичайне редагування тексту й copy/paste руйнували б такий документ.

Запропоновані canonical literals:

```text
source("articles.records")
field("tags")
ref("tags.tags", "tag-news-001")
```

Остаточну граматику зафіксувати тестами до реалізації editor projection. Строки повинні мати формальні escaping rules і коректно обробляти лапки, Unicode та пунктуацію. Показані labels ніколи не інтерполюються як executable syntax.

Збережений документ містить `languageVersion`, `dialect: "query"`, canonical source та за потреби catalog revision. AST і кеш labels можна зберігати для прискорення, але їх можна відновити з canonical source і каталогу. Незавершений draft можна зберегти окремо від останнього валідного плану; його не виконують.

Reference-токен показує label, але cursor navigation, undo/redo, selection, deletion, paste та diagnostics мають узгоджене mapping між displayed positions і canonical positions. Сторонні API offsets не повинні випадково посилатися на іншу систему координат; контракт має явно вказувати позиції, наприклад UTF-16 offsets, і тестувати surrogate pairs.

### Copy/paste та редагування

- Plain-text export/copy за замовчуванням переносить canonical text зі stable references.
- Додатковий rich clipboard MIME може містити versioned document fragment для збереження вигляду.
- Окрема команда «Copy readable text» може копіювати labels, але має попереджати, що це не гарантує round-trip identity.
- Вставлення canonical fragment відновлює токени й завантажує labels через resolve.
- Вставлення чи ручне введення label створює unresolved draft reference. Для reference-поля навіть один збіг потребує явного вибору/підтвердження; не прив'язувати потай за текстом.
- Два теги з назвою News показуються окремо з додатковим контекстом; вибір зберігає конкретний ID.
- Токен можна замінити через completion. Редагування його назви як plain text скидає попередню прив'язку й переводить фрагмент у unresolved state.
- Після видалення/недоступності тегу показати зрозумілий unresolved reference зі збереженим ID. Не шукати «такий самий News» для автоматичної заміни.

Кеш попереднього label може бути лише підказкою для відображення з позначкою outdated/unresolved. Зміна principal/scope повинна очищувати кеш хоста або змінювати його namespace; не переносити labels між користувачами.

## 12. Автодоповнення

Визначати очікуваний grammar slot за позицією курсора навіть у невалідному документі:

```text
Get |                       -> first, last; record variable form
Get first 20 articles from | -> доступні джерела
... where Articles |        -> поля Articles
... where Articles status | -> сумісні оператори
... status is |             -> enum-значення
... tags includes Ne|       -> async пошук у tags.tags
... order by |              -> sortable fields поточного джерела
... publication_date |      -> asc, desc
```

Пропозиція включає label, kind, detail/type, replacement range, insert/edit operation та стабільну identity за потреби. Розділити синхронні grammar completions і асинхронні catalog/reference completions.

Показувати лише сумісні оператори й values. Назви з пробілами, keywords і однаковими labels мають мати явне token/quoted representation. Доступні aliases беруться з каталогу, а не вгадуються fuzzy matching.

Провайдер викликається тільки для релевантного reference slot. Початкова затримка пошуку — налаштовувані 200 ms. Кожен запит має cancellation, request identity та document version. Застаріла відповідь не змінює новіші підказки, токени чи результат. Додаткові сторінки завантажуються на вимогу.

Resolve групує невідомі ID за джерелом; не робити один виклик на кожен токен і не вантажити всі записи довідника. Кеш має бути обмеженим і керованим хостом або очищуваним; відмова пошуку не повинна назавжди блокувати повторну спробу.

## 13. Редактор та виконання

Основний UI — багаторядкове текстове поле. Потрібні:

- syntax/semantic highlighting keywords, variables, fields, enums, references, parameters;
- inline diagnostics із зрозумілим поясненням та exact range;
- completion на введення й Ctrl+Space, Enter/Tab для вибору, Escape для закриття;
- нормальна робота keyboard-only, screen reader labels, loading/error announcements;
- undo/redo для вибору й заміни token як цілісної дії;
- коректні multiline paste, IME composition і введення Unicode;
- повернення cursor/selection після token insertion без стрибків;
- tooltip/inspector для перегляду stable ID, типу поля та source;
- можливість відкрити canonical text для діагностики й програмного обміну.

Виконувати лише завершений, семантично валідний запит із усіма parameters/reference bindings. Для playground: автоматичний preview після паузи 300 ms та явний Run; затримку можна налаштувати. Незавершені слова, відкриті дужки, невибрані references й помилки не спричиняють запитів до executor.

При новому виконанні скасувати попереднє, якщо можливо, і в будь-якому разі ігнорувати застарілу відповідь. Попередній результат під час редагування позначати як результат попереднього валідного запиту, а не нового тексту. Empty, loading, validation error і execution error — різні стани.

Виконання preview — одноразове читання. Не додавати watch або polling. Хост повторно перевіряє авторизацію та валідність стабільних посилань; локальна перевірка редактора не є межею безпеки.

## 14. Діагностики й обмеження

Діагностика має code, severity, message, canonical range і, де доречно, semantic path та виправлення. Мінімальні категорії: syntax/incomplete, unknown source/field, unsupported operation, invalid value/type, ambiguous label, unresolved reference, missing parameter, incompatible schema version, exceeded bound, provider/execution failure.

Не дозволяти звернення до prototype properties. Ключі на зразок `__proto__`, `constructor`, `prototype` не обходять обмеження через quoted syntax. Доступ до declared paths перевіряється на кожному сегменті.

Задати налаштовувані захисні ліміти довжини документа, кількості токенів/умов, вкладеності та списків values. Надмірний ввід завершується контрольованою діагностикою, без stack overflow. Cursor position і document revisions мають бути валідовані.

Не виконувати непідтримані joins/aggregates як повне завантаження всіх даних у browser. Помилка провайдера не означає нуль знайдених записів. Будь-яке звуження можливостей джерела робить відповідну конструкцію невалідною.

## 15. Майбутні діалекти

Query — перший dialect. Core може мати discriminated document node та versioned entrypoint, через який потім підключаться workflow/conditions.

Повторно використовувані частини: literals, references, parameters, variable declarations, scopes, types, spans, diagnostics, catalog/provider interfaces і editor services.

Конструкції trigger/action/if/else/loop не парсити й не виконувати зараз. Не закладати правило «будь-яке валідне речення автоматично виконується» для всіх майбутніх dialects: одноразовий query preview та дії workflow матимуть різні execution policies.

Не створювати універсальну plugin platform або залежність core від query-specific AST заради гіпотетичних можливостей.

## 16. Демонстраційні дані та перевірка гіпотези

Playground має містити синтетичні Articles, Tags, People та Orders із:

- enum status, nullable/optional fields, scalar arrays, object/one/many;
- двома тегами News з різними ID;
- назвами з пробілами, лапками, Unicode і keywords;
- сервероподібною пагінацією довідника;
- затримками, помилками й відповідями не по порядку;
- кнопками або тестовими сценаріями перейменування/приховування тегу;
- одним джерелом без count і полем, непридатним для сортування.

Додати щонайменше два взаємозамінні reference provider: синхронні in-memory fixtures через Promise та імітацію віддаленого delayed provider. Вони не потребують реальної мережі. Завдання — довести незалежність, а не написати HTTP/IndexedDB SDK.

Результат query executor має реально відповідати предикату, selection, sort і window на синтетичному наборі. Не видавати статичні mocked rows за працюючу вибірку. Локальний evaluator може бути частиною demo/test adapter; загальна execution library для всіх хостів не є окремою вимогою.

## 17. Тести та критерії приймання

### Парсер і семантика

- Усі приклади розбираються в очікувану структуру.
- Parse/format/parse та serialize/reopen зберігають зміст.
- Case keywords, whitespace, optional final period, escaped strings і Unicode працюють передбачувано.
- AST містить declaration змінної, stable source/field IDs і reference IDs, а не labels.
- Фільтр і сортування застосовуються перед формуванням вікна; lookup і list із first 1 мають різні типи.
- Перевіряються boolean precedence, null vs absent, scalar array some/all та owned-child predicates.
- Невідомі поля, неправильні типи, неунікальні lookup keys, несумісні clauses та перевищення лімітів відхиляються з точною діагностикою.
- Query не може отримати права, source scope чи довільну функцію через текст.

### Identity і провайдери

- Після вибору News зберігається конкретний ID.
- Перейменування News -> Updates змінює показану назву й не змінює план або членство вибірки.
- Два News не зливаються; вибраний запис не підміняється першим знайденим.
- Після reopen resolve завантажує тільки потрібні ID, пакетами за джерелом.
- Після втрати доступу зберігається ID і показується unresolved state.
- Повільний пошук Ne не перекриває новіший пошук Ed; abort та помилки не ламають наступну спробу.
- Зміна host context не залишає labels з іншого principal/scope.
- Copy/paste canonical text зберігає ID без sidecar.

### Редактор і preview

- Keyboard-only створення основного запиту від порожнього документа до результату.
- Підказки залежать від grammar slot, типу поля, capabilities і reference metadata.
- Click/keyboard заміна token, undo/redo, видалення й plain-text edit не залишають прихованого старого ID.
- IME composition не запускає передчасні completion commits або execution.
- Невалідний draft не викликає executor; попередній результат правильно позначено.
- Застарілий execution result не перекриває результат новішого документа.
- Loading/error/empty/reference failure розрізняються.
- Count показує bounded status; backward window має канонічний порядок; invalid cursor показується без тихого скидання.

### Незалежність

- Core/query/language-service можна імпортувати й тестувати у Node без DOM/React/HTTP/IndexedDB.
- Заміна reference provider не потребує змін парсера чи editor logic.
- У репозиторії немає product-specific imports, production data, secrets або SQL execution.
- Є typecheck, unit tests, browser interaction tests і build demo. Команди задокументовані після появи реалізації.

## 18. Порядок реалізації для агента

1. Зафіксувати grammar, canonical reference literals, public interfaces і fixtures. Вибрати найменший зрілий parser/editor stack з урахуванням tolerant parsing та browser packaging.
2. Реалізувати вертикальний зріз: один list query -> tag completion -> stable token -> semantic plan -> реальний результат demo.
3. Довести identity round-trip, rename, ambiguous labels, undo/redo та clipboard. Це основний ризик гіпотези; не відкладати до косметичного полірування.
4. Додати всі оголошені оператори, selection, sort, parameters, lookup, pagination і bounded count.
5. Довести asynchronous cancellation/staleness, editor accessibility і діагностики.
6. Перевірити критерії, зібрати playground і оновити документацію реальними командами запуску та чесним списком реалізованих можливостей.

Якщо дослідження покаже, що обраний механізм відображення ID як labels не забезпечує коректний round-trip та звичне редагування, описати конкретний збій і варіанти зміни контракту. Не маскувати проблему hidden sidecar mappings або підміною стабільних ID назвами.

## 19. Очікуваний результат роботи

Працюючий незалежний TypeScript-прототип із пакетами, playground, синтетичними adapters, тестами та актуальною документацією. Користувач може набрати речення, вибрати News з підказки, отримати відфільтровані дані, зберегти та відкрити документ після перейменування тегу без зміни сенсу.

Не обов'язково мати всі можливості майбутньої мови або інтеграцію з існуючим продуктом. Обов'язково довести головну гіпотезу й чітко показати, які частини підтримуються. Публікація npm, ліцензія та production integration — окремі рішення власника після оцінки прототипу.
