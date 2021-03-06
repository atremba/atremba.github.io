﻿---
layout: post
title: Использование шаблонизатора Liquid в Jekyll
some_var: Текст в переменной some_var.
example_list: [1, 2, 3, 4, 5, 6, 7]
---

Продолжаем разбираться с Jekyll. Основным компонентом для создания полезного функционала сайта является некий Liquid. Его же надо знать для модификации сайта и добавления "рюшечек" и "свистелок".

**Liquid** - это язык генерации страниц из шаблонов, используемый в Jekyll.
В нём есть переменные, циклы и некоторые базовые операции, а потому формально он может считаться языком программирования (впрочем, очень ограниченным).

У Liquid есть [собственная документация](https://shopify.github.io/liquid/).
Она подходит для справки, но в контексте сайта/блога её не хватает, т.к. Jekyll [использует расширенную вариацию Liquid](https://jekyllrb.com/docs/liquid/),
с тремя [дополнительными тегами](https://jekyllrb.com/docs/liquid/tags/), а также [множеством фильтров](https://jekyllrb.com/docs/liquid/filters/) и некоторыми Jekyll-специфическими переменными. В заметке рассмотрен именно этот вариант Liquid.

Компанией Cloudcannon составлен [детальный справочник](https://learn.cloudcannon.com/jekyll-cheat-sheet/) по переменным, тегам, фильтрам и прочим элементам Jekyll/Liquid.

> Терминология: в Liquid есть **теги** и **фильтры**. "Теги" Liquid никак не связаны с тегами (метками) в обычном смысле (как в "облаке тегов"), а являются аналогами "тегов HTML", т.е. выполняют роль **разметки**. Фильтры используются внутри тегов и выполняют преобразования над переменными: от замены текста до сортировки.

Обработка Liquid тегов и фильтров происходит в **первую очередь, до** преобразования Markdown-разметки в HTML.
Можно посмотреть [исходный текст этой записи](https://github.com/atremba/atremba.github.io/blob/master/_posts/2019-05-12-jekyll-and-liquid.md) как источник примеров.

Для [отладки ошибок Liquid](#отладка-liquid) см. раздел ниже.

Далее по порядку разобраны компоненты Liquid:

- [переменные](#переменные-liquid),
- [теги](#теги-liquid),
- [фильтры](#фильтры-liquid).

Ещё раз подчеркну, что некоторые переменные и теги существуют только в "Liquid для Jekyll", а не "чистого Liquid".

## Переменные Liquid

Переменные - ядро Liquid. В большинстве случаев их можно только читать, использовать, но не изменять. В документации Liquid они отделены от "объектов", но для простоты удобно считать "объекты" просто переменными для чтения.

Переменные[^1] "появляются" несколькими способами:

- берутся из YAML-разметки, а именно:
  - из заголовков страниц/записей блога (Front Matter),
  - из конфигурационного файла Jekyll `_config.yml`.
  
  Они включают пользовательские переменные, [внутренние переменные Jekyll](https://jekyllrb.com/docs/variables/), также переменные со значениями по умолчанию[^2].
- из [файлов данных Jekyll](/TODO#jekyll-data),
- создаются тегами `assign` и `capture`, см. ниже.

Переменными просто пользоваться, но некоторые вещи банально не документированы.
Они возникают на стыке YAML (данные), Liquid (обработка данных) и Ruby (как языка, на котором написан Jekyll и обработчик Liquid).

Некоторые факты:

- Liquid лояльно относится[^3] к несуществующим переменным, полям, индексам массивов и возвращает специальное пустое значение Nil (когда результат Liquid-кода "пуст"). При выводе это значение соответствует пустой строке:
  > {% raw %} "{{ not_a_var }}", "{{ some_var.no_such_attribute }}" и "{{ list[-3] }}" {% endraw %} выведут
  > "{{ not_a_var }}", "{{ some_var.no_such_attribute }}" и "{{ list[-3] }}".
- Кавычки (и двойные, и одинарные) определяют **строку**, а не переменную. В примере текст специально взят в квадратные скобки. Кавычки не входят в состав строки (переменная `some_var` не определена):
  > {% raw %} [{{ some_var }}], [{{ "some_var" }}] и [{{ 'some_var' }}] {% endraw %} выведут
  > [{{ some_var }}], [{{ "some_var" }}] и [{{ 'some_var' }}].
- Есть ключевые слова, например, `true`, `false` (чувствительны к регистру)
  > {% raw %} "{{ true }}", "{{ false }}" и "{{ False }}" {% endraw %} выведут
  > "{{ true }}", "{{ false }}" и "{{ False }}"
- Есть [типы](https://shopify.github.io/liquid/basics/types/), сравнение между которыми выдаст ошибку (например, строка и число). С другой стороны, может происходить автоматическое преобразование типов, например, в фильтрах.
- При выводе тегом {% raw %}`{{ ... }}`{% endraw %} содержимое преображается в строку.

В YAML можно задать **скаляры**, **массивы** и **словари** (см. [заметку по YAML]({% post_url 2019-04-20-yaml %})), а Liquid формально может использовать только [**скаляры и массивы**](https://shopify.github.io/liquid/basics/types/).

Это не мешает использовать **доступ к полям** словаря с помощью синтаксиса `dict_name.field_name` или равноправного ``dict_name[field_name]``, см. [примеры YAML]({% post_url 2019-04-20-yaml %}). Эти значения можно использовать, **но не изменять**.

> {% raw %} "{{ site.url }}", "{{ site["url"] }}" и "{{ page.title }}" {% endraw %} выведут
> "{{ site.url }}", "{{ site["url"] }}" и "{{ page.title }}"

Здесь использованы предопределённые переменные, связанные с сайтом и конкретно с этой страницей.

Доступ к элементам массива осуществляется по числовому индексу (начинается с 0):
`array_name[0], array_name[1], ...`

Наконец, в Jekyll [определены два фильтра](https://jekyllrb.com/docs/liquid/filters/): `sort` и `where`, имеющие доступ к полям (properties).
А именно, они могут **сортировать** и **фильтровать** массивы словарей, исходя из значений конкретных полей.

### Создание переменных в  Liquid

Иметь дело с переменными в Liquid не так просто, в первую очередь из-за их постоянства. Как уже говорилось, изменять некоторые сложные переменные динамически нельзя (в частности, словари и их поля).

Зато можно создавать новые переменные.
Можно определить и создать переменную прямо внутри тега `assign`:

```markdown
{%- raw -%}
{% assign my_variable = ... %}
{% endraw -%}
```

А парные теги {% raw %} `{% capture my_variable %} ... {% endcapture %}` {% endraw %} "захватывают" в переменную `my_variable` всё содержимое между ними.

Списки (массивы) можно породить либо тегом `split` (из строк, см. [пример в разделе про фильтры](#фильтры-liquid)), либо применением фильтров к **существующим** массивам/словарям.

### Модификация переменных

В Liquid предусмотрены особые теги итерации {% raw %}`{% increment var_name %}` и `{% decrement var_name %}`{% endraw %}, создающие и модифицирующие целочисленные переменные (начиная с нуля). Некоторый юмор в том, что созданная таким образом переменная `var_name` отличается от переменной `var_name` с тем же именем, но созданной с помощью {% raw %} `{% assign var_name = ... %}` {% endraw %}
Как с этим работать, я так и не понял. Но для себя сделал вывод, что Liquid **немного странный**.

## Теги Liquid

**Теги** бывают двух видов[^4]. Они могут:

1. Подставить текст переменной. Они состоят из двойных открывающих и закрывающих фигурных скобок, например {% raw %} `{{ page.some_var }}` {% endraw %}
выведет переменную `some_var`, заданную (без префикса![^5]) в заголовке этой страницы: `{{ page.some_var }}`.
Перед выводом переменная может быть преобразована **фильтром**, например, заменяющим часть строки `"Текст в"` на `"Data in"`: `{{ page.some_var | replace: "Текст в", "Data in"}}`.

2. Выполнить команду в формате {% raw %} `{% command ... %}` {% endraw %}. Эти теги, в свою очередь, делятся на

   - одиночные.  
   Например {% raw %} `{% post_url 2019-04-20-yaml %}` {% endraw %} формирует [ссылку на запись про YAML]({% post_url 2019-04-20-yaml %}), которая находится в файле `_posts/2019-04-20-yaml.md`,
   - парные [**управляющие конструкции**](https://shopify.github.io/liquid/tags/control-flow/), типа `for`, `if`, switch-конструкции (называются `case/when`), [циклы](https://shopify.github.io/liquid/tags/iteration/) с `break/continue` и пр.

    {% raw %}
    > {% for counter in page.example_list %}{{ counter }}{% endfor %}
{% endraw %}

     Выведет:

     > {% for counter in page.example_list %}{{ counter }}{% endfor %}

   У тегов есть **нетривиальные** [особенности работы с окружающими пробелами](https://shopify.github.io/liquid/basics/whitespace/), особенно с вложенными многокомпонентными конструкциями (типа `if - elseif - else - endif`), поэтому удобнее всего их использовать в HTML-шаблонах, а не markdown-разметке[^4].

  Комментарии, ограниченные тегами {%- raw -%} `{% comment %} ... {% endcomment %}` {% endraw %}, будет исключены из текста.
  
  Чтобы отменить интерпретацию тегов {% raw %} (`{{ ... }}` и `{% ... %}` {% endraw %}), их надо обрамлять  `{% raw %}{%{% endraw %}` `raw` `{% raw %}%}{% endraw %}` и `{% raw %}{%{% endraw %}` `endraw` `{% raw %}%}{% endraw %}`.

### Специфические теги Jekyll

Помимо [стандартных Liquid-тегов](https://shopify.github.io/liquid/tags/control-flow/), в Jekyll добавлено [ещё три](https://jekyllrb.com/docs/liquid/tags/):

- для подсветки синтаксиса (средствами HTML!). Мне кажется, что этот функционал дублирует блочные записи c ```` ```lang ... ``` ```` в Markdown.
- для ссылок на произвольные страницы (`link`) и записи блога (`post_url`). Они весьма полезны, т.к. выполняют **валидацию ссылок**, и если соответствующей страницы/записи нет, сборка Jekyll прервётся, а в режиме запущенного веб-сервера (`jekyll serve`) будет выдано сообщение  об ошибке, как в примерах выше.

К сожалению, если сайт располагается не в корне, то надо добавлять префикс `baseurl` (задаваемый в `_config.yml`):
> `{% raw %}[Name of Link]({{ site.baseurl }}{% post_url 2010-07-21-name-of-post %}) {% endraw %}`

## Фильтры Liquid

Фильтры применяются для форматирования вывода и преобразования выражений.
Для вывода используется тег {% raw %} `{{ ... }}`{% endraw %}, а преобразования полезны для формирования переменных, например, это способ создать массив:
> {% raw %}  
{% assign my_array = "ants, bugs, bees, bugs, ants" \| split: ", " %}  
{% endraw %}
{% comment %} Экранирование "\|" нужно только отображения в markdown-разметке. В результате ниже его нет. {% endcomment %}
создаст список `my_array`, который будет выведен как
{% assign my_array = "ants, bugs, bees, bugs, ants" | split: ", " %}
> {{ my_array }}

- Фильтры бывают без параметров, например,

  {% raw %} `{{ 123.456 | ceil }}` {% endraw %} выведет
  > `{{ 123.456 | ceil }}`

- Фильтры с параметрами:
  {% raw %} `{{ 7 | divided_by: 2.0 }}` {% endraw %} выведет
  > {{ 7 | divided_by: 2.0 }}

- Фильтры можно применять последовательно:  
  {% raw %} `{{ my_array | uniq | join: ", " }}` {% endraw %} выведет
  > {{ my_array | uniq | join: ", " }}

Кроме [полного списка фильтров Jekyll и Liquid](https://jekyllrb.com/docs/liquid/filters/), удобно
использовать [справочник](https://learn.cloudcannon.com/jekyll-cheat-sheet/) с группировкой по типам объектам (целые числа, массивы, строке),
к которым применяются фильтры. Заодно в этом справочнике приведены встроенные переменные и список управляющих тегов (условия, циклы и пр.)

{% comment %}## Особенности Liquid в Jekyll

Включает дополнительные _теги_ и _фильтры_ (ссылка на документацию по тегам и фильтрам - отдельно ). {% endcomment %}

{% comment %}Во-вторых, словари, по-видимому, воспринимаются тегами Liquid как массивы (YAML допускает это).
{% comment %}
   {% raw %}

   ```md
   {% if now | date: "%d" <= 10 }} %}
     Первая треть месяца: {{ now | date: "%d" }}.
   {% elsif now_data = 13 %}
     Тринадцатое! День недели - {{ now | date: "%d.%m.%Y" }}.
   {% else %}
     Обычный день: {{ now | date: "%d.%m.%Y" }}.
   {% endif %}
   ```

   {% endraw %} {% endcomment %}
{% if counter >= 3 %}
   > Элемент списка: {{ counter -}}
{% elsif counter == 1 %}
   > Первый элемент!
{% else %}
   > Не ">=3" и не "==1": {{ counter -}}
{%- endif -%}
{% endcomment %}

## Отладка Liquid

Удобнее всего экспериментировать с Liquid, [запустив Jekyll локально]({% post_url 2019-04-24-local-jekyll %}), а потом редактировать файл, периодически его сохраняя. Исключение - главный конфигурационный файл `_config.yml`; для учёта изменений в нём надо перезапустить сервер.

Интересно, что **запуск** Jekyll в режиме сервера (`jekyll serve` и `bundle exec jekyll serve`) прервётся, если будет обнаружена ошибка в Liquid. Однако если запустить его на "хорошем" коде, а потом внести ошибку в код, то работа сервера **не прервётся**.

При обнаружении ошибок или предупреждений Jekyll/Liquid выдаёт в консоль, откуда был запущен, содержательную информацию, включающую номер строки.
Правда, иногда он не соответствует строкам markdown-кода из-за заголовка[^6].
Примеры:

> Liquid Warning: Liquid syntax error (line 47): Expected end_of_string but found pipe in ""now" \| date: "%d" <= 10" in <path_to_blog>/_posts/2019-05-08-jekyll-and-liquid.md

> Liquid Exception: Could not find post "2019-04-29-local-jekyll" in tag 'post_url'. Make sure the post exists and the name is correct. in <path_to_blog>/_posts/2019-05-08-jekyll-and-liquid.md

> Error: Could not find post "2019-04-28-local-jekyll" in tag 'post_url'. Make sure the post exists and the name is correct.

После исправления ошибки выдача станет нормальной:

> Regenerating: 1 file(s) changed at 2019-05-12 10:53:49  
_posts/2019-05-08-jekyll-and-liquid.md  
...done in 3.0911087 seconds.

Для подробного вывода можно запустить `jekyll build --trace`, [точнее]({% post_url 2019-04-24-local-jekyll %}),

```cmd
bundle exec jekyll build --trace
```

## Заключение

Язык Liquid чаще всего используется для создания шаблонов или тем сайтов/блогов.

Однако с его помощью можно и формировать страницы динамически, и даже организовывать [подобие базы
данных](/TODO#liquid-data) с помощью файлов данных (в режиме чтение).

В обычных записях можно использовать ссылки на другие записи.
Например, некоторые записи этого блога начинаются со ссылок вида
{% raw %} `[текст]({% post_url 2019-04-09-blog-on-github-pages.md %})`{% endraw %}.

---

## Ссылки

- <https://shopify.github.io/liquid/basics/introduction/> - оригинальная документация и введение в Liquid. [Исходный код Liquid](https://github.com/Shopify/liquid) лежит в Github, а [исходники документации на Liquid+Markdown](https://github.com/Shopify/liquid/tree/gh-pages) доступна в ветке `gh-pages`.

- <https://jekyllrb.com/docs/liquid/> - документация Liquid и его модификации на сайте Jekyll.

- <https://learn.cloudcannon.com/jekyll-cheat-sheet/> - отличный компактный **справочник/cheatsheet** по переменным, тегам и фильтрам Liquid (включая таковые для Jekyll).

- <https://help.shopify.com/en/themes/liquid> - альтернативная, немного устаревшая документация на сайте Shopify, создателя Liquid. Здесь
  - фильтры сгруппированы по темам,
  - ещё один [**справочник/cheatsheet**](https://ru.shopify.com/partners/shopify-cheat-sheet), подробный и удобный, но для варианта Liquid-для-Shopify.
  - есть [примеры кода](https://shopify.github.io/liquid-code-examples),
  - объяснения, [как работают различные теги](https://www.siteleaf.com/blog/tags/liquid/) - `sort`, `where` и `group_by`.

{% comment %}
- <https://habr.com/ru/post/336266/> - введение в Liquid на Хабре, c примерами
{% endcomment %}

---

[^1]: Формально переменные являются "объектами", и их значения выводятся с помощью конструкции {% raw %} `{{ ... }}` {% endraw %}. Для простоты я называю эти двойные фигурные скобки тоже "тегами".

[^2]: Переменные по умолчанию есть и [в заголовках](https://jekyllrb.com/docs/front-matter/), и в [конфигурационном файле](https://jekyllrb.com/docs/configuration/default/). Github Pages добавляет к ним [метаданные репозитория](https://help.github.com/en/articles/repository-metadata-on-github-pages). Кроме того, в Jekyll можно [добавить свои переменные по умолчанию](https://jekyllrb.com/docs/configuration/front-matter-defaults/) для нужных страниц или записей, сделав раздел `defaults:` в `_config.yml`.

[^3]: [Согласно документации](https://github.com/Shopify/liquid), технически это поведение (обработку неопределённых переменных и фильтров) можно изменить при запуске обработчика Liquid-кода. Но как включить этот режим из Jekyll - не знаю.

[^4]: В спецификации Liquid [есть вариации](https://shopify.github.io/liquid/basics/whitespace/) тегов, включающие дефисы: {% raw %} `{{- ... -}}`,  и `{%- ... -%}` {% endraw %}. Они "отключают" пробелы вокруг результата выполнения тега, что важно в контексте Markdown-разметки. Похоже, что они удаляют пробельные символы до/после тега, "приклеивая" его содержимое к предыдущему/последующему тексту. Эти дефисы-модификаторы могут быть не парными. Так, {% raw %} `{{ ... -}}` {% endraw %} отключит пробелы **после** вывода. По моему мнению, поведение этих тегов в сложных и вложенных конструкциях типа `for ... if-then-else-endif ... endfor` документировано недостаточно.

{% comment %}
их использование не требуется (или в Jekyll-варианте Liquid пробельные символы удаляются автоматически, проверить не удалось).
{% endcomment %}

[^5]: Все заданные в заголовке (Front Matter) переменные добавляются как поля в словаре `page`. Аналогично, все переменные, заданные в `_config.yml`, являются полями словаря `site`, и доступны в тексте любой страницы.

[^6]: YAML-заголовок (Front Matter) считывается и убирается из текста **до обработчика** Liquid. Это логично, ведь собственно из него и берётся часть переменных. Поэтому номер строки ошибки, предупреждения или исключения Liquid надо увеличивать на число строк Front Matter - включая начальные и закрывающие строки с "`---`".