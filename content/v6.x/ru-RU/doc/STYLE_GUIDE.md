# Руководство по стилю

* Документация, написанная в markdown файлах с названиями, отформатированными как `строчные буквы-с-тире.md`. 
  * Подчеркивания в названии файла допускается только, когда они присутствуют в теме, о которой пойдет речь в документе (например, `child_process`).
  * Некоторые файлы, такие как markdown файлы высокого уровня, являются исключением.
* Переход на новую строку в документах ограничена 80 символами.
* Форматирование, описанное в `.editorconfig`, является предпочтительным. 
  * [plugin](http://editorconfig.org/#download) доступен для некоторых редакторов, чтобы автоматически применять эти правила.
* Механические проблемы, такие как орфография и грамматика, должны определяться инструментами (tools), насколько это возможно. Если не обнаружены с помощью инструмента, они должны быть отмечены рецензентами.
* Предпочтительна американская орфография английского языка. "Capitalize" против "Capitalise", "color" против "colour", и т. д.
* Хотя спорная, [Oxford comma](https://en.wikipedia.org/wiki/Serial_comma) предпочтительна для ясности.
* Избегайте личных местоимений в справочной документации ("я", "вы", "мы"). 
  * Местоимения допустимы в более нелитературной документации, такой как руководства.
  * Используйте гендерно-нейтральные местоимения и имена нарицательные. Некоторые примеры: 
    * ПРАВИЛЬНО: "они", "их", "им", "народ", "люди", "разработчики", "кошки"
    * НЕПРАВИЛЬНО: "его", "её", "ему", "ей", "ребята", "чуваки"
* При сочетании парных знаков препинания (скобки и кавычки), завершающий знак препинания должен располагаться: 
  * Внутри парных знаков препинания, если заключенные внутри элементы составляют полное предложение - подлежащее, сказуемое и определение.
  * Вне парных знаков препинания, если внутри заключена только часть предложения.
* Знак, завершающий предложение, помещайте внутри парных знаков препинания - точка ставится внутри скобок и кавычек, не после.
* Документы должны начинаться с заголовка первого уровня. Пример документа будет прикреплен здесь в конечном итоге.
* Предпочтительно прикреплять ссылки, а не встраивать - предпочтение `[a link][]` к `[a link](http://example.com)`.
* При документировании API, обратите внимание, когда была предоставлена версия API в конце раздела. Если API устарел, обратите также внимание на первую версию, в которой устаревший API появился.
* При использовании тире, используйте [Em dashes](https://en.wikipedia.org/wiki/Dash#Em_dash) ("—" или `Option+Shift+"-"` на macOS), окруженный пробелами, согласно с [The New York Times Manual of Style and Usage](https://en.wikipedia.org/wiki/The_New_York_Times_Manual_of_Style_and_Usage).
* Включая активы: 
  * Если вы хотите добавить иллюстрацию или целую программу, добавьте ее к соответствующему подкаталогу в `assets/` dir.
  * Ссылка на это так: `[Asset](/assets/{subdir}/{filename})` для файловых активов, и `![Asset](/assets/{subdir}/{filename})` для активов образов.
  * Для иллюстраций предпочтительней SVG другим активам. When SVG is not feasible, please keep a close eye on the filesize of the asset you're introducing.
* For code blocks: 
  * Use language aware fences. ("```js")
  * Code need not be complete — treat code blocks as an illustration or aid to your point, not as complete running programs. If a complete running program is necessary, include it as an asset in `assets/code-examples` and link to it.
* When using underscores, asterisks, and backticks, please use proper escaping (`\_`, `\*` and `` \` `` instead of `_`, `*` and `` ` ``).
* References to constructor functions should use PascalCase.
* References to constructor instances should use camelCase.
* References to methods should be used with parentheses: for example, `socket.end()` instead of `socket.end`.
* Function arguments or object properties should use the following format: 
  * `* \<code>name` {type|type2} Optional description. **Default:** `defaultValue`</code>
  * E.g. `* <code>byteOffset` {integer} Index of first byte to expose. **Default:** `0`</code>
  * The `type` should refer to a Node.js type or a [JavaScript type](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Grammar_and_types#Data_structures_and_types)
* Function returns should use the following format: 
  * `* Returns: {type|type2} Optional description.`
  * E.g. `* Returns: {AsyncHook} A reference to <code>asyncHook`.</code>