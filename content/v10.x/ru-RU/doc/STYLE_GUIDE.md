# Руководство по стилю

* Документация, написанная в markdown файлах с названиями, отформатированными как `строчные буквы-с-тире.md`. 
  * Underscores in filenames are allowed only when they are present in the topic the document will describe (e.g. `child_process`).
  * Некоторые файлы, такие как markdown файлы высокого уровня, являются исключением.
* Переход на новую строку в документах ограничена 80 символами.
* Форматирование, описанное в `.editorconfig`, является предпочтительным. 
  * [plugin](http://editorconfig.org/#download) доступен для некоторых редакторов, чтобы автоматически применять эти правила.
* Changes to documentation should be checked with `make lint-md`.
* Предпочтительна американская орфография английского языка. "Capitalize" против "Capitalise", "color" против "colour", и т. д.
* Use [serial commas](https://en.wikipedia.org/wiki/Serial_comma).
* Избегайте употребления личных местоимений в справочной документации («я», «вы», «мы»). 
  * Personal pronouns are acceptable in colloquial documentation such as guides.
  * Use gender-neutral pronouns and gender-neutral plural nouns. 
    * OK: "they", "their", "them", "folks", "people", "developers"
    * НЕПРАВИЛЬНО: "его", "её", "ему", "ей", "ребята", "чуваки"
* При сочетании парных знаков препинания (скобки и кавычки), завершающий знак препинания должен располагаться: 
  * Внутри парных знаков препинания, если заключенные внутри элементы составляют полное предложение - подлежащее, сказуемое и определение.
  * Вне парных знаков препинания, если внутри заключена только часть предложения.
* Знак, завершающий предложение, помещайте внутри парных знаков препинания - точка ставится внутри скобок и кавычек, не после.
* Документы должны начинаться с заголовка первого уровня.
* Предпочтительно прикреплять ссылки, а не встраивать - предпочтение `[a link][]` к `[a link](http://example.com)`.
* При документировании API, обратите внимание, когда была предоставлена версия API в конце раздела. Если API устарел, обратите также внимание на первую версию, в которой устаревший API появился.
* При использовании тире, используйте [Em dashes](https://en.wikipedia.org/wiki/Dash#Em_dash) ("—" или `Option+Shift+"-"` на macOS), окруженный пробелами, согласно с [The New York Times Manual of Style and Usage](https://en.wikipedia.org/wiki/The_New_York_Times_Manual_of_Style_and_Usage).
* Включая активы: 
  * If you wish to add an illustration or full program, add it to the appropriate sub-directory in the `assets/` dir.
  * Link to it like so: `[Asset](/assets/{subdir}/{filename})` for file-based assets, and `![Asset](/assets/{subdir}/{filename})` for image-based assets.
  * Для иллюстраций желательно использовать формат SVG. When SVG is not feasible, please keep a close eye on the filesize of the asset you're introducing.
* For code blocks: 
  * Use language aware fences. ("```js")
  * Code need not be complete — treat code blocks as an illustration or aid to your point, not as complete running programs. If a complete running program is necessary, include it as an asset in `assets/code-examples` and link to it.
* When using underscores, asterisks, and backticks, please use proper escaping (`\_`, `\*` and `` \` `` instead of `_`, `*` and `` ` ``).
* References to constructor functions should use PascalCase.
* References to constructor instances should use camelCase.
* References to methods should be used with parentheses: for example, `socket.end()` instead of `socket.end`.
* To draw special attention to a note, adhere to the following guidelines: 
  * Make the "Note:" label italic, i.e. `*Note*:`.
  * Use a capital letter after the "Note:" label.
  * Preferably, make the note a new paragraph for better visual distinction.
* Function arguments or object properties should use the following format: 
  * `* \<code>name` {type|type2} Optional description. **Default:** `defaultValue`.</code>
  * Например: `* <code>byteOffset` {integer} Index of first byte to expose. **Default:** `0`.</code>
  * The `type` should refer to a Node.js type or a [JavaScript type](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Grammar_and_types#Data_structures_and_types).
* Function returns should use the following format: 
  * `* Returns: {type|type2} Optional description.`
  * Например: `* Returns: {AsyncHook} A reference to <code>asyncHook`.</code>
* Use official styling for capitalization in products and projects. 
  * OK: JavaScript, Google's V8
  * NOT OK: Javascript, Google's v8

See also API documentation structure overview in [doctools README](../tools/doc/README.md).