---
layout: post
title: Локальный запуск сайта на Jekyll
---

Итак, [блог на Github успешно стартовал]({% post_url 2019-04-11-start-blog-on-github %}) и захотелось

- делать новые записи,
- изменить внешний вид сайта,
- добавить отдельные страницы типа [`/about`](/about),
- настроить теги и т.п.

Любое из этих действий меняет содержимое файлов.
Поскольку фактически Jekyll работает как компилятор исходного кода настроек и страниц (на `YAML/markdown`) в статические HTML-страницы[^1], то он может обнаружить ошибку и проигнорировать обновление сайта (блога), в зависимости от серьёзности ошибки.

На Github Pages запуск Jekyll для обновления сайта происходит каждый раз, когда в репозитории в ветке `master` обновляются файлы (неважно, каким образом - через веб-интерфейс или git; единственное ограничение касается [обновления репозитория](https://help.github.com/en/articles/generic-jekyll-build-failures#deploy-key-used-for-push) программным способом, с помощью ключа развёртывания).

В любом случае, Jekyll будет присылать предупреждения или сообщения об ошибках на почту, указанную при регистрации на Github, а также предупреждать в настройках репозитория блога (раздел Github Pages)[^2]:
![image from Github help](/assets/2019-04-24-local-jekyll/repo-actions-settings.png)
![cut image from Github help](/assets/2019-04-24-local-jekyll/pages-repository-settings-box-with-page-build-error-cut.png)

Подробнее об ошибках см. заметку на Github [Viewing Jekyll build error messages](https://help.github.com/en/articles/viewing-jekyll-build-error-messages).

"Пожар легче предупредить, чем потушить", а программу надо сначала отладить, а потом запускать[^3].
В контексте сайта на Github Pages (точнее, на Jekyll) все новые страницы, настройки блога и изменения шаблонов необходимо проверить на работоспособность заранее, до размещения на Github Pages. Лучший способ - **установить Jekyll локально.**

Тогда алгоритм ведения блога будет следующим:

1. Обновление записей/модификация настроек или внешнего вида,
2. Тестирование - блог будет доступен по локальному адресу <http://127.0.0.1:4000>, с возвратом к п.1 при обнаружении ошибок,
3. Сохранение изменений в локальный git-репозиторий,
4. Загрузка изменений из локального репозитория в Github,
5. Контроль работоспособности блога в интернете.

Пункты 3-4 требуют хотя бы примитивного [знания Git](/notes/links#git), но для начала можно обойтись и без него, просто копируя изменённые файлы через web-интерфейс Github (как это [делалось при запуске блога]({% post_url 2019-04-11-start-blog-on-github %})).

Локальная установка и использование Jekyll хорошо описана на том же Github: [Setting up your GitHub Pages site locally with Jekyll](https://help.github.com/en/articles/setting-up-your-github-pages-site-locally-with-jekyll). Но есть нюанс - для этого надо **заиметь Jekyll на компьютере**[^4].

> Мотивация установки локальной версии Jekyll:
>
> - удобно вычитывать текст, проверять оформление записей блога и контролировать отсутствие "битых" картинок,
> - можно убедиться в работоспособности сайта, и в частности, динамически собираемых страниц,
> - можно отладить получающийся статичный HTML-сайт (хранится в папке `_site`),
> - можно перестать беспокоиться об [ограничении Github Pages на 10 "сборок" сайта в час](https://help.github.com/en/articles/what-is-github-pages#usage-limits).

Это несложно организовать в Mac OS и Linux ([см. документацию Jekyll](https://jekyllrb.com/docs/installation/)), но установка Jekyll на Windows поддерживается только [неофициально](https://jekyllrb.com/docs/installation/windows/). Далее описаны нескоторые подводные камни и их обход.

## Установка Jekyll на Windows

> Теория: Jekyll написан на языке программирования [Ruby](https://www.ruby-lang.org/ru/), поэтому требует его интерпретатор. Заодно получим и встроенный веб-сервер (WEBrick).

Шаги:

1. Установить Ruby (если с инсталлятором, то надо выбирать `Ruby+Devkit`). А можно установить вручную:
    - скачать архив 7zip c [официального сайта Ruby](https://rubyinstaller.org/downloads/),
    - распаковать его (напр. в `./ruby/`),
    - добавить в переменною окружения PATH путь к `ruby/bin`
2. Установить сопутствующую инфраструктуру (MSYS2 + MinGW) для установки/компиляции `gem`-ов - пакетов Ruby:
    - выполнить `ridk install` в папке `./ruby/ridk_use` **и выбрать либо вариант 3 (`3 - MSYS2 and MINGW development toolchain`), либо просто нажать Enter**. MSYS2+MINGW это окружение (toolchain) для сборки и компиляции программ под Windows.
    - (опционально) обновить MSYS2/MinGW до последних версий[^5].
3. Установить **Jekyll** и **bundler** (пакет для удобного запуска Jekyll - с учётом правильной версии), командой `gem` из Ruby):

    ```cmd
    gem install jekyll bundler
    ```

4. Запустить Jekyll.

Альтернатива установке - [Ruby на DockerHub](https://hub.docker.com/_/ruby?tab=description) или сразу [Jekyll на DockerHub](https://hub.docker.com/search?q=jekyll&type=image), включая [официальный Docker Jekyll](https://hub.docker.com/r/jekyll/jekyll) (не пользовался, поэтому конкретный вариант не подскажу).

## Запуск Jekyll

Есть два варианта запуска Jekyll (в любом случае надо находиться в папке с файлом `_config.yml`). Можно либо просто запустить Jekyll для однократной сборки сайта (`jekyll build` или просто `jekyll`), либо сразу запустить локальный сервер. В последнем случае происходят три вещи:

1. сборка статических файлов,
2. запуск локального веб-сервера с созданным сайтом (по умолчанию на адресе <http://127.0.0.1:4000>),
3. периодическая проверка изменений исходных `markdown`-файлов и автоматическая пересборка страниц.

Для этого надо перейти в каталог блога (тот самый, в котором находится файл `_config.yml`, это важно[^6]) и выполнить:

- `jekyll serve` или
- **`bundle exec jekyll serve`**

Правильно использовать второй вариант, т.к. при этом `bundle` читает специальный файл `Gemfile` из каталога блога, и запускает `jekyll serve` **для нужной версию Jekyll**, вместе с необходимыми модулями (gems). Если этого файла нет, его [рекомендуется создать](https://help.github.com/en/articles/setting-up-your-github-pages-site-locally-with-jekyll#step-2-install-jekyll-using-bundler), а если он существует, то необходимо проверить его на соответствие версии Jekyll хостера.
Подробности в записи о [базовой настройке Jekyll]({% post_url 2019-04-28-jekyll-main-configuration %}).

Я использую эту команду с двумя параметрами:

```cmd
bundle exec jekyll serve --baseurl "" --future
```

Первый параметр `--baseurl ""` говорит о переопределении поля `baseurl` в `_config.yml`. Если её не добавить, сайт скорее всего выдаст страницу с ошибкой:

> 404: Page not found
>
> Sorry, we've misplaced that URL or it's pointing to something that doesn't exist. Head back home to try finding it again.

Второй параметр-флаг `--future` обрабатывает записи с "будущей" датой. Она полезна для отладки черновиков. Отмечу, что если файл с "будущей" записью выложить на Github, то хотя он не будет обрабатываться Jekyll (и соответственно, не будет доступен по адресу `<имя-сайта>.github.io/...`), его содержимое будет доступно как часть публичного репозитория[^7].

Некоторые опции команды `jekyll serve` позволяют настроить имя сервера, порт и SSL ключи. Полный список см. в [документации по конфигурации](#ссылки) Jekyll. Есть возможность [задавать разные окружения](https://jekyllrb.com/docs/configuration/environments/).

В результате всех манипуляций должен запуститься веб-сервер, а браузер - автоматически открыться на странице `http://127.0.0.1:4000`, отображая последнюю запись блога.

Далее: [первичная настройка Jekyll]({% post_url 2019-04-28-jekyll-main-configuration %}).

---

## Ссылки

- [Troubleshooting GitHub Pages builds](https://help.github.com/en/articles/troubleshooting-github-pages-builds) - подробная документация по ошибкам Jekyll на Github.

- <https://jekyllrb.com/docs/installation/> - варианты установки Jekyll на локальные системы (MacOS, Linux, Windows).

- <https://help.github.com/en/articles/setting-up-your-github-pages-site-locally-with-jekyll> - локальная установка Jekyll в документации Github Pages.

- <https://guides.hexlet.io/jekyll/#Установка-Jekyll-на-локальную-машину> - описание установки на русском языке.

- <https://hub.docker.com/r/jekyll/jekyll> - официальный образ Jekyll на DockerHub.

- <https://jekyllrb.com/docs/usage/> - варианты запуска Jekyll из командной строки.

- <https://jekyllrb.com/docs/configuration/options/> - параметры Jekyll при запуске из командной строки (flags, отмечены синим).

---

[^1]: Результат хранится в папке `\_site`, которую нельзя увидеть через web-интерфейс Github.

[^2]: Подробно об ошибками Jekyll написано на самом Github в разделе помощи [Troubleshooting GitHub Pages builds](https://help.github.com/en/articles/troubleshooting-github-pages-builds). В первом же разделе [Viewing Jekyll build error messages](https://help.github.com/en/articles/viewing-jekyll-build-error-messages) предлагается установить Jekyll локально. Там же есть начальные рекомендации по настройке автоматического тестирования (Continuous Intergration, CI).

[^3]: [Программирование по стечению обстоятельств](https://pragprog.com/the-pragmatic-programmer/extracts/coincidence) - плохая идея.

[^4]: Для знакомых с Docker: см. [статью на hexlet.io](https://guides.hexlet.io/jekyll/#установка-и-запуск-jekyll-через-docker) или [ссылку](#ссылки) на Docker-образы.

[^5]: Запустить `msys2`, и в появившемся окне запустить `pacman -Syuu`. После выполнения обновления закрыть окно `msys2`, открыть второе и повтороить `pacman -Syuu` (аналогично перезагрузке Windows - не все действия можно сделать за раз).

[^6]: Если запустить Jekyll в любом другом каталоге (напр. `jekyll serve`), он  сработает как простой файл-сервер, т.е. будет просто просматривать и выдавать файлы через браузер, и даже обрабатывать Liquid-шаблоны. Но это может привести к ошибкам из-за несоответствия файловой структуры каталога типовой структуре сайта на Jekyll.

[^7]: Для платного Github аккаунта такой проблемы нет: для приватного репозитория виден только сгенерированный из него сайт.
