## Парадигма

- Именование классов по БЭМ, разметка в [pug](https://pugjs.org/) и стилизация [Sass](http://sass-lang.com/). См. [Как работать с CSS-препроцессорами и БЭМ](http://nicothin.github.io/idiomatic-pre-CSS/)
- Каждый БЭМ-блок в своей папке внутри `./src/blocks/` (.scss, и .pug файлы обязательны).
- Список использованных в проекте БЭМ-блоков и доп. файлов указан в `./projectConfig.json`. Это главный конфигурационный файл проекта.
- Есть глобальные файлы: стилевые (стили печати), js (по умолчанию пуст), шрифты, картинки.
- Диспетчер подключения стилей `./src/scss/main.scss` генерируется автоматически при старте любой gulp-задачи (на основе данных из `./projectConfig.json`).
- Список pug-примесей `./src/pug/main.pug` генерируется автоматически при старте любой gulp-задачи (на основе данных из `./projectConfig.json`).
- Перед созданием коммита запускается проверка стилевых файлов, входящих в коммит и всех pug-файлов. При наличии ошибок коммит не происходит (ошибки будут выведены в терминал).
- Есть механизм быстрого создания нового блока: `node createBlock.js new-block` (создаёт файлы, папки, прописывает блок в `./projectConfig.json`).



## Разметка

Используется [pug](https://pugjs.org/api/getting-started.html). HTML никак не обрабатывается.

По умолчанию используются [наследование шаблонов](https://pugjs.org/language/inheritance.html) — все страницы (см. `./src/html/pages/index.pug`) являются расширениями шаблонов, в страницах описывается только содержимое «шапки», «подвала» и контентной области посредством [блоков](https://pugjs.org/language/inheritance.html#block-append-prepend).

Каждый блок (в `./src/blocks/`) может содержать pug-файл с одноименной примесью, который будет взят includ-ом в файле `./src/pug/mixins.pug` (этот файл формируется автоматически на основании списка используемых на проекте блоков).



## Стили

Файл-диспетчер подключений (`./src/scss/main.scss`) формируется автоматически на основании указанных в `./projectConfig.json` блоков и доп. файлов. Писать в `./src/scss/main.scss` что-либо руками бессмысленно: при старте автоматизации файл будет перезаписан.

Используемый постпроцессинг:

1. [autoprefixer](https://github.com/postcss/autoprefixer)
2. [css-mqpacker](https://github.com/hail2u/node-css-mqpacker)
3. [postcss-import](https://github.com/postcss/postcss-import)
4. [postcss-inline-svg](https://github.com/TrySound/postcss-inline-svg)
5. [gulp-cleancss](https://github.com/mgcrea/gulp-cleancss) (только в режиме сборки без карт кода)
6. [postcss-object-fit-images](https://github.com/ronik-design/postcss-object-fit-images) (в паре с [полифилом](https://github.com/bfred-it/object-fit-images))
7. [postcss-image-inliner](https://www.npmjs.com/package/postcss-image-inliner)

Для [postcss-image-inliner](https://www.npmjs.com/package/postcss-image-inliner) указано ограничение на размер файла в 5 Кб, файлы ищутся в `src/blocks/**/bg-img/`. Чтобы избежать конфликтов имен, добавляйте к именам изображений префикс (имя блока), например: `src/blocks/mega-block/bg-img/mega-block__avatar.png`

## Блоки

Каждый блок лежит в `./src/blocks/` в своей папке. Каждый блок — как минимум, папка и одноимённые scss- и pug-файл.

Возможное содержимое блока:

```bash
demo-block/               # Папка блока
  img/                    # Изображения, используемые блоком и обрабатываемые автоматикой сборки
  bg-img/                 # Изображения для использования в стилях (не обрабатываются автоматикой сборки)
  demo-block.pug          # **Обязательный**. Разметка (pug-примесь, отдающая разметку блока, описание API примеси)
  demo-block.scss         # **Обязательный**. Стилевой файл блока
  demo-block.js           # js-файл блока
  demo-block--mod.scss    # Отдельный стилевой файл БЭМ-модификатора блока
  demo-block--mod.js      # js-файл для отдельного БЭМ-модификатора блока
  readme.md               # Описание для документации, подсказки
```



## Подключение блоков

Список используемых блоков и доп. файлов указан в `./projectConfig.json`. Список файлов и папок, взятых в обработку можно увидеть в терминале, если раскомментировать строку `console.log(lists);` в `gulpfile.js`.

**ВНИМАНИЕ!** `./projectConfig.json` — это JSON. Это строгий синтаксис, у последнего элемента в любом контексте не должно быть запятой в конце строки.

### `blocks`

Объект с блоками, используемыми на проекте. Каждый блок — отдельная папка с файлами, по умолчанию лежат в `./src/blocks/`.

Каждое подключение блока — массив, который можно оставить пустым или указать файлы элементов или модификаторов, если они написаны в виде отдельных файлов. В обоих случаях в обработку будут взяты одноименные стилевые файлы, pug-файл, js-файлы и картинки из папки `img/` блока.

Пример, подключающий 3 блока:

```
"blocks": {
  "page": [],
  "page-header": [],
  "page-footer": []
}
```

### `addJsBefore`

Массив js-файлов, которые будут взяты в обработку (конкатенация/сжатие) ПЕРЕД js-файлами блоков.

Пример, добавляющий в список обрабатываемых js-файлов несколько зависимостей:

```
"addJsBefore": [
  "./node_modules/jquery/dist/jquery.min.js",
  "./node_modules/jquery-migrate/dist/jquery-migrate.min.js",
],
```

### `addJsAfter`

Массив js-файлов, которые будут взяты в обработку (конкатенация/сжатие) ПОСЛЕ js-файлов блоков.

Пример, добавляющий в конец списка обрабатываемых js-файлов глобальный скрипт.

```
"addJsAfter": [
  "./src/js/global-script.js"
],
```

### `addSass`

Массив с дополнительными стилевыми файлами, которые будут взяты в компиляцию ПЕРЕД стилевыми файлами блоков.


```
"addSass": [
    "src/sass/helpers.scss",
    "src/sass/functions.scss",
    "src/sass/mixins.scss",
    "src/sass/base.scss",
    "src/sass/elements.scss",
    "src/sass/plugins.scss",
    "src/sass/pages.scss"
  ],
```

### Пример `./projectConfig.json`

```json
{
  "blocks": {
    "page-header": [],
    "page-footer": [
      "__extra-element",
      "--extra-modifier"
    ]
  },
  "addSass": [
    "src/sass/helpers.scss",
    "src/sass/functions.scss",
    "src/sass/mixins.scss",
    "src/sass/base.scss",
    "src/sass/elements.scss",
    "src/sass/plugins.scss",
    "src/sass/pages.scss"
  ],
  "dirs": {
    "srcPath": "./src/",
    "buildPath": "./build/",
    "blocksDirName": "blocks"
  }
}
```

## Удобное создание нового блока

Предусмотрена команда для быстрого создания файловой структуры нового блока. По умолчанию создаются: scss- и pug-файл, `readme.md` блока и его подпапки `img` и `bg-img`

```bash
# формат: node createBlock.js ИМЯБЛОКА [доп. расширения через пробел]
node createBlock.js block-test # создаст папку блока, block-test.pug, block-test.scss, подпапки img/ и bg-img/ для этого блока
```

Если блок уже существует, файлы не будут затёрты, но создадутся те файлы, которые ещё не существуют.



## Назначение папок

```bash
dist/          # Папка сборки, здесь работает сервер автообновлений.
src/            # Исходные файлы
  blocks/       # - блоки проекта
  fonts/        # - шрифты проекта (будут автоматически скопированы в папку сборки)
  img/          # - добавочные или общие для нескольких блоков картинки
  js/           # - добавочные js-файлы
  hmtl/         # - примеси, шаблоны pug
  scss/         # - стили (файл main.scss скомпилируется, прочие нужно подключить в addSass, иначе они будут проигнорированы)
```
