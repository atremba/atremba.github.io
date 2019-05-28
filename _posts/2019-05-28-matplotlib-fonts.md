---
title: Шрифты в matplotlib.pyplot
---

**Задача:** в формулах на рисунках, сделанных в `matplotlib`, хочется иметь шрифты, максимально похожие на шрифты основного текста (в частности - шрифты с засечками).

**Подзадача:** разобраться с настройкой шрифтов.

**Дано:** Windows 10, Jupyter notebook в Anaconda, Python 3.6, `matplotlib` 3.0.3.

> TL;DR  
> `matplotlib.rcParams['mathtext.fontset'] = 'dejavuserif'`

## Описание проблемы

По умолчанию шрифты на рисунках без засечек (семейство `sans-serif`), а в основном тексте документа (Latex) - с засечками (`serif`).

Попытка установить шрифт стандартным способом (`matplotlib.rcParams[...]`, см. ниже) не действует на формулы, создавая впечатление, как будто шрифты "не подгружаются".

Похожие проблемы в интернете:

- [нельзя установить `serif`](https://stackoverflow.com/questions/52320969/matplotlib-reports-font-family-serif-not-found) - в ответах предложили переустановить другую версию matplotlib,
- [аналогичная проблема с базовыми шрифтами](https://stackoverflow.com/questions/42097053/matplotlib-cannot-find-basic-fonts/42936670#42936670) - даже сделан [баг-репорт](https://github.com/matplotlib/matplotlib/issues/8876).
Решение - установить `msttcorefonts` (через `apt-get`, т.е. не для Windows).

На самом деле всё решается намного проще: RTFM.
Само решение [в конце этой заметки](#решение).

## Отступление: шрифты в `matplotlib`

Список доступных шрифтов выявляет [модуль `font_manager`](https://matplotlib.org/api/font_manager_api.html).

```python
import matplotlib.font_manager
# 'ttf' option to show files with TrueType Fonts only
matplotlib.font_manager.findSystemFonts(fontpaths=None, fontext='ttf')
```

Можно попытаться найти (отфильтровать) искомый шрифт командой
`prop` - строка (в [не совсем очевидном формате](https://www.freedesktop.org/software/fontconfig/fontconfig-user.html)) или объект [FontProperties](https://matplotlib.org/api/font_manager_api.html#matplotlib.font_manager.FontProperties).
Можно работать с текстовой строкой с помощью модуля [`matplotlib.fontconfig_pattern`](https://matplotlib.org/api/fontconfig_pattern_api.html?highlight=font#module-matplotlib.fontconfig_pattern).

```python
prop = matplotlib.font_manager.FontProperties(family='serif')
matplotlib.font_manager.findfont(prop, fontext='ttf')
```

В результате был выдан один шрифт: `DejaVuSerif.ttf`, то есть вроде наличие шрифта с засечками есть.

Имена шрифтов (не имена файлов!) можно выяснить опосредованно, зная путь к файлу шрифта

```python
flist = matplotlib.font_manager.findSystemFonts()
names = [matplotlib.font_manager.FontProperties(fname=fname).get_name() for fname in flist]
```

Список найденных шрифтов хранится в файле `~/.matplotlib/fontlist-v300.json`,
Точный путь к папке кеширования:

```python
matplotlib.font_manager.get_cachedir()
```

Можно встретить совет удалить папку `.matplotlib`, но это не помогает решить проблему.

### Основы шрифтовых настроек `matplotlib`

Хранятся или изменяются либо через объект `matplotlib.rcParams` (напр. `rcParams['lines.linewidth'] = 2`),
либо командой `rc('lines', linewidth=2, color='r')`. Последняя эквивалентна двум командам формата `rcParams[...] = ...`.

В [описании `matplotlib.rc`](https://matplotlib.org/api/matplotlib_configuration_api.html#matplotlib.rc) как раз есть пример с выбором шрифта.

[Сброс настроек matplotlib](https://matplotlib.org/users/customizing.html): `matplotlib.rcdefaults()`.

{% comment %}

### Текущий шрифт (по умолчанию)
Есть некоторая путаница между семейством и конкретным шрифтом, см. [подробности](https://stackoverflow.com/questions/27817912/find-out-which-font-matplotlib-uses).

И все эти списки - **рекомендации** для поиска наиболее подходящего шрифта.

- обновить Анаконду:

  ```cmd
  conda update conda
  conda update --all
  ```

{% endcomment %}

## Решение

Возвращаемся к задаче выбора шрифта в формуле.

Обычно тип шрифта, используемый в командах `matplotlib` [настраивается](https://matplotlib.org/gallery/api/font_family_rc_sgskip.html) установкой `rcParams['font.family']` и
`rcParams['font.<font_family>'] = <список шрифтов>` (список нужен для того, чтобы сразу перечислить "запасные" шрифты).
Это не сработает: 

```python
import matplotlib as mpl
import matplotlib.pyplot as plt

plt.figure(figsize=(8,1.2))

mpl.rcdefaults() # reset font properties
plt.text(0.05, 0.7, 'a) Default fonts for all: Text with $For = m^{ul}(a) \\rightarrow \\max_a$', fontsize=14)

mpl.rcParams['font.family'] = 'serif'
plt.text(0.05, 0.05, 'b) Incomplete attempt to set a serif font everywhere:\n '
    r'Text with $For = m^{ul}(a) \rightarrow \max_a$', fontsize=14)
```

![rcParams['font.family'] fails to set font in math mode](/assets/2019-05-28-matplotlib-fonts\a-b.png)

**Шрифты внутри математических формул** вида `'Text $x^2$'` определяются другим параметром в `rcParams`, а именно **`'mathtext'`** .

Соответствующая секция присутствует в [описании-примере конфигурационного файла](https://matplotlib.org/users/customizing#a-sample-matplotlibrc-file),
также эта особенность [нашлась в объяснении "бага" Matplotlib](https://github.com/matplotlib/matplotlib/issues/8702) - там даже предлагалось изменить документацию `matplotlib`.

Это же решение нашлось явно в самой документации, в разделе [математического режима Matplotlib](https://matplotlib.org/tutorials/text/mathtext.html).
Согласно ей (и [примеру](https://matplotlib.org/users/customizing#a-sample-matplotlibrc-file)) ключевой параметр `rcParams[mathtext.fontset]` может принимать ограниченный ряд значений `'dejavusans'`, `'dejavuserif'`, `'cm'` (Computer Modern), `'stix'`, `'stixsans'` or `'custom'`.

Я сначала использовал шрифт `cm` - Сomputer Modern, основной в Latex.

```python
import matplotlib
matplotlib.rcParams['mathtext.fontset'] = 'cm'
```

Однако его размер не совпал с размером шрифта в тексте (пример `с)`), пришлось поменять на `'dejavuserif'` (пример `d)`), т.к. `matplotlib` сам не обнаружил шрифт `Computer Modern`, а искать его вручную я не стал.

```python
plt.figure(figsize=(8,0.7))
mpl.rcdefaults() # reset font properties
# and set text and math font to serif ones
mpl.rcParams['font.family'] = 'serif'
mpl.rcParams['mathtext.fontset'] = 'cm'
plt.text(0.05, 0.4, 'с) Serif fonts everywhere: Text with $For = m^{ul}(a) \\rightarrow \\max_a$', fontsize=14)
```
![mpl.rcParams['mathtext.fontset'] = 'cm' size mismatch](/assets/2019-05-28-matplotlib-fonts\c.png)

```python
plt.figure(figsize=(8,0.7))
# Make text font match math font
mpl.rcParams['font.family'] = 'serif'
mpl.rcParams['font.serif'] = 'DejaVu Serif'
mpl.rcParams['mathtext.fontset'] = 'dejavuserif'
plt.text(0.05, 0.4, 'd) Dejavu Serif for all: Text with $For = m^{ul}(a) \\rightarrow \max_a$', fontsize=14)
```
![mpl.rcParams['mathtext.fontset'] = 'dejavuserif' matches 'DejaVu Serif' font](/assets/2019-05-28-matplotlib-fonts\d.png)


### Нюансы

- должен быть отключён внешний Latex-интерпретатор: `rcParams['text.usetex'] = false`. Настройка `'mathtext.fontset'` влияет на _собственный_ интерпретатор `matplotlib`.
- набор доступных шрифтов для формул очень ограничен, и им может не найтись соответствия среди **установленных** шрифтов (у меня совпал только вариант `'dejavuserif'` для формул и `'DejaVu Serif'` для текста).
- эту настройку можно сделать только с помощью `rcParams[...]` или `rc(...)`, и нельзя передать как аргумент (наподобие `fontdict`) в функцию отрисовки (`plot`, `text` ...).
По этому поводу [в `matplotlib` открыт баг](https://github.com/matplotlib/matplotlib/issues/7107).
Возможно, по этой причине возникают следующие особенности: 
  - нельзя настроить разные стили (шрифты) математического текста на одной картинке,
  - в jupyter notebook, к примеру, можно установить математический шрифт _после_ вывода (`text(...)`), но до отрисовки `plt.show()`. Примеры `e), f), g)`.
- размеры шрифтов в обычном тексте и формулах могут различаться (пример `c)` выше).


```python
import matplotlib as mpl
import matplotlib.pyplot as plt

plt.figure(figsize=(8,2.5))

mpl.rcdefaults() # reset font properties
# try to set a sans (non-serif) font
mpl.rcParams['mathtext.fontset'] = 'dejavusans'
plt.text(0.05, 0.8, 'e) This shall be a sans font: Text with $For = m^{ul}(a) \\rightarrow \max_a$', fontsize=14)

mpl.rcParams['font.family'] = 'serif'
plt.text(0.05, 0.4, 'f) Here the font shall be as in b) case:\n '
    r'Text with $For = m^{ul}(a) \rightarrow \max_a$', fontsize=14)

# let's set math font to a serif one
mpl.rcParams['mathtext.fontset'] = 'cm'
plt.text(0.05, 0.2, 'g) Change math formulae to serif only now. WTF?', fontsize=14)
plt.text(0.05, 0.04, 'Now all formulae appears to be serif... ($x^2 = 42$)', fontsize=14)

plt.show()
```
![setting math font after printout but before show() works perfectly](/assets/2019-05-28-matplotlib-fonts\e-f-g.png)

## Выводы

1. Подберите нужный шрифт для формул (один из 5), соответствующий одному из установленных шрифтов (напр. `cm` и `Computer Modern` или `dejavusans` и `Dejavu Sans` и т.п.)
2. Используйте `mpl.rcParams['mathtext.fontset'] = ...` для настройки шрифта в формулах.


## Ссылки

- <https://matplotlib.org/users/customizing#a-sample-matplotlibrc-file> - пример файла настроек `matplotlib` с комментариями и объяснениями
- <https://matplotlib.org/tutorials/text/mathtext.html> - описание обработки в `matplotlib` формул в Latex-стиле
- <http://jonathansoma.com/lede/data-studio/matplotlib/changing-fonts-in-matplotlib/> - шрифты можно изменять в любом текстовом объекте независимо
- <https://jenyay.net/Matplotlib/RcParams> - о параметрах `matplotlib` в цикле статей на русском языке
{% comment %}- сслыка на matplotlib-fonts.ipynb {% endcomment %}
