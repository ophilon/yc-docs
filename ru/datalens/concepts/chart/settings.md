---
title: Настройки чарта
description: В этой статье вы узнаете о настройках чарта, а также узнаете, как отменить и восстановить изменения в чартах.
---

# Настройки чарта

Вы можете настраивать чарты. Например, добавить отображение легенды, настроить цветовую схему, указать собственный заголовок.

Доступность настроек зависит от типа настраиваемого чарта.

## Общие настройки {#common-settings}

Общие настройки влияют на общее отображение чартов.
Чтобы открыть общие настройки, нажмите значок ![image](../../../_assets/console-icons/gear.svg) в левой части экрана над чартом.

{% note warning %}

Указанные настройки доступны не для всех видов чартов.

{% endnote %}

#|
|| **Название** | **Описание** ||
|| Заголовок | Задает заголовок для чарта.<br/><br/>Доступные значения:

* **Скрыть** — скрыть заголовок.
* **Показать** — отображать заголовок. В текстовом поле справа вы можете ввести текст заголовка. ||
|| Легенда | Отображает легенду под чартом. Легенда содержит цвета чарта с текстовыми описаниями, которые соответствуют значениям полей в секции **Цвета**.<br/><br/>Доступные значения:

* **Вкл** — отображать легенду, если указано поле в [секции](#color-settings) **Цвета**.
* **Выкл** — скрыть легенду. ||
|| Тултип | Отображает всплывающие подсказки при наведении на элемент чарта.<br/><br/>Доступные значения:

* **Вкл** — отображать тултип.
* **Выкл** — скрыть тултип. ||
|| Сумма в тултипах | Отображает сумму в тултипе при наведении на отображение чарта.<br/><br/>Доступные значения:

* **Вкл** — отображать сумму в тултипе.
* **Выкл** — скрыть сумму в тултипе. ||
|| Навигатор | Отображает дополнительный элемент под чартом — навигатор. Он позволяет уменьшить выборку данных, которая отображается в чарте.<br/><br/>Доступные значения:

* **Вкл** — отображать навигатор.
* **Выкл** — скрыть навигатор.

Подробнее см. [инструкцию](../../operations/chart/config-chart-navigator.md). ||
|| Пагинация | Разбивает таблицу на страницы.<br/><br/>Доступные значения:

* **Вкл** — отображать таблицу по частям на страницах.
* **Выкл** — отображать таблицу целиком. ||
|| Лимит | Задает количество строк для экспорта. Также это количество строк будет отображаться на одной странице. Доступно, если включена настройка **Пагинация**. ||
|| Группировка | Отображает уникальные значения полей.<br/><br/>Доступные значения:

* **Вкл** — отображать уникальные значения.
* **Выкл** — отображать все значения. ||
|| Итоги | Отображает строку с итоговыми значениями для столбцов в таблице. Значения в строке рассчитываются по тем же формулам, что и [агрегация](../../dataset/data-model.md#aggregation) в показателе.<br/><br/>Доступные значения:

* **Вкл** — отображать строку **Итого**.
* **Выкл** — скрыть строку **Итого**. ||
|| Сохранять пробелы и переносы | Отображает пробелы и переносы строк в таблице, как в исходных данных.<br/><br/>Доступные значения:

* **Вкл** — отображать все пробелы и переносы.
* **Выкл** — сокращать множественные пробелы и переносы до одного пробела. ||

|| Накопление | Задает в [диаграмме с областями](../../visualization-ref/area-chart.md#stacking) отображение данных с накоплением или отдельно для каждой категории.<br/><br/>Доступные значения:

* **Вкл** — отображать с накоплением.
* **Выкл** — отображать без накопления. ||
|| Центр | Задает в [Картах](../../visualization-ref/map-chart.md) координаты центра по умолчанию.<br/><br/>Доступные значения:

* **Авто** — установить координаты автоматически.
* **Вручную** — указать координаты центра карты вручную, например `54.630761, 39.736882`. Для определения координат вы можете воспользоваться [Яндекс Картами](https://yandex.ru/maps). ||
|| Масштаб | Задает в [Картах](../../visualization-ref/map-chart.md) масштаб по умолчанию.<br/><br/>Доступные значения:

* **Авто** — использовать автоматический масштаб.
* **Вручную** — указать масштаб карты вручную. Значения соответствуют масштабу в Яндекс Картах:
  
  * `1` — минимальный масштаб;
  * `21` — максимальный масштаб. ||
|#

## Настройки полей {#field-settings}

Вы можете настраивать:

* измерения и показатели для [чартов на основе датасета](./dataset-based-charts.md);
* только показатели для [QL-чартов](./ql-charts.md).

Чтобы открыть настройки измерения или показателя, нажмите значок слева от его названия.

{% note info %}

Если применить к измерению агрегацию, оно станет показателем, и для него будут доступны соответствующие настройки.

{% endnote %}

### Измерения {#measure-settings}

#|
|| **Название** | **Описание** ||
|| Название | Задает имя измерения. ||
|| Тип | Задает тип данных измерения. ||
|| Группировка | Задает тип группировки или округление. Настройка доступна только для измерений типа `Дата` и `Дата и время`. ||
|| Формат | Задает формат отображения значения. ||
|| Режим даты | Задает режим отображения даты. Настройка доступна только для измерений типа `Дата` и `Дата и время`.<br/><br/>Доступные значения:

* **Непрерывный** — отображать все даты непрерывно.
* **Дискретный** — отображать даты, которые содержат значения. ||
|| Агрегация | Задает тип агрегации. Функции агрегации доступны в соответствии с таблицей [{#T}](../../dataset/data-model.md#aggregation). ||
|| Разметка | Значение поля будет отформатировано в соответствии с выбранным значением:

* `Нет` — без разметки.
* `HTML` — разметка HTML. Доступно только для полей с типом `Строка`.
* `Markdown` — разметка [{#T}](../../dashboard/markdown.md). Доступно только для полей с типом `Строка` в секциях в зависимости от типа визуализации.
  
  #|
  || **Типы визуализации** | **Секции** ||
  || [Точечная диаграмма](../../visualization-ref/scatter-chart.md) | X, Y, Точки, [Цвета](#color-settings), Формы ||
  || [Древовидная диаграмма](../../visualization-ref/tree-chart.md) | Измерения ||
  || [Карты](../../visualization-ref/map-chart.md) | [Тултип](#map-settings) ||
  || [Линейная](../../visualization-ref/line-chart.md), [Столбчатая](../../visualization-ref/column-chart.md), [Линейчатая](../../visualization-ref/bar-chart.md), [Круговая](../../visualization-ref/pie-chart.md), [Кольцевая](../../visualization-ref/ring-chart.md) диаграммы и [Диаграмма c областями](../../visualization-ref/area-chart.md) | [Подписи](#sign) ||
  |#

Настройка недоступна в визуализациях: [Индикатор](../../visualization-ref/indicator-chart.md), [Таблица](../../visualization-ref/table-chart.md) и [Сводная таблица](../../visualization-ref/pivot-table-chart.md). ||
|| Подытоги | Отображает столбцы и/или строки с подытогами. Настройка доступна только для чарта [Сводная таблица](../../visualization-ref/pivot-table-chart.md). ||
|#

### Показатели {#indicator-settings}

#|
|| **Название** | **Описание** ||
|| Название | Задает имя показателя. ||
|| Тип | Задает тип данных показателя. Недоступно для [QL-чартов](./ql-charts.md). ||
|| Агрегация | Задает тип агрегации. Функции агрегации доступны в соответствии с таблицей [{#T}](../../dataset/data-model.md#aggregation). Недоступно для [QL-чартов](./ql-charts.md). ||
|| Формат | Задает формат отображения значения. ||
|| Знаков после запятой | Задает количество знаков, отображаемых у значения после запятой. Недоступно для [QL-чартов](./ql-charts.md). ||
|| Отображать группы разрядов | Позволяет отображать разряды числовых значений.<br/><br/>Доступные значения:

* **С разделителем** — отображать пробелы между разрядами.
* **Слитно** — скрыть пробелы между разрядами. ||
|| Префикс | Задает текст, который отображается перед значением. ||
|| Постфикс | Задает текст, который отображается после значения. ||
|| Размерность | Задает масштаб округления значений. ||
|#

## Настройки секций {#section-settings}

Чтобы открыть настройки секции, нажмите значок ![image](../../../_assets/console-icons/gear.svg) в строке с названием секции.

### Оси {#axis-settings}

Настройки осей доступны только для тех чартов, у которых есть хотя бы одна ось X или Y:

* Линейная диаграмма.
* Диаграмма с областями (Накопительная и Нормированная).
* Столбчатая диаграмма (в том числе Нормированная).
* Линейчатая диаграмма (в том числе Нормированная).
* Точечная диаграмма.

Вы можете задавать настройки как для оси **X**, так и **Y**.

{% note warning %}

Не все указанные настройки доступны для всех осей.

{% endnote %}

#|
|| **Название** | **Описание** ||
|| Название оси | Задает подпись к оси.<br/><br/>Доступные значения:

* **Вкл** — использовать название поля. Если секция содержит несколько полей, то {{ datalens-short-name }} использует имя поля, которое указано первым в списке.
* **Выкл** — не отображать подписи к оси.
* **Вручную** — указать имя оси вручную в текстовом поле. ||
|| Тип оси | Определяет тип оси. Настройка доступна, если поля в секции оси имеют тип `Дробное число`.<br/><br/>Доступные значения:

* **Линейная** — использовать линейную ось.
* **Логарифмическая** — использовать логарифмическую ось. Подходит для чартов с большим разбросом значений. Логарифмическая ось позволяет привести быстрорастущий график к удобному для анализа виду, уменьшая значения на порядок. ||
|| Ось на графике | Настройка позволяет скрыть все, что относится к оси — линии, подписи, сетку.<br/><br/>Доступные значения:

* **Показать** — отображать все, что относится к оси.
* **Скрыть** — не отображать все, что относится к оси. ||
|| Режим отображения | Позволяет настроить непрерывность отображения графика.<br/><br/>Доступные значения:

* **Дискретный** — отображать график только для непустых значений.
* **Непрерывный** — отображать график для всех значений непрерывно. Непрерывный режим доступен только для типов данных `Целое число`, `Дробное число`, `Дата` и `Дата и время`. ||
|| Форматирование оси | Задает форматирование подписей на оси с числовыми значениями.<br/><br/>Доступные значения:

* **Авто** — форматирование по умолчанию.
* **По первому полю на оси X** — отображать подписи к оси X с форматированием, указанным в настройке **Формат** для первого поля в секции **X**. Доступно в настройках оси X.
* **По первому полю на оси Y** — отображать подписи к оси Y с форматированием, указанным в настройке **Формат** для первого поля в секции **Y**. Доступно в настройках оси Y. ||
|| Сетка | Включает/отключает отображение сетки в чарте.<br/><br/>Доступные значения:

* **Вкл** — отображать сетку.
* **Выкл** — скрыть сетку. ||
|| Шаги сетки, px | Задает шаг сетки в пикселях. Доступно, если включена настройка **Сетка**.<br/><br/>Доступные значения:

* **Авто** — вычислять размер сетки автоматически.
* **Вручную** — указать размер сетки в пикселях. ||
|| Подписи | Включает/отключает отображение подписей к оси, которые соответствуют значениям поля.<br/><br/>Доступные значения:

* **Вкл** — отображать подписи к осям.
* **Выкл** — скрыть подписи к осям. ||
|| Вид подписей | Определяет вид отображения подписей. Доступно, если включена настройка **Подписи**.<br/><br/>Доступные значения:

* **Авто** — отображать значения подписи.
* **Горизонтальные** — отображать значения подписей горизонтально.
* **Вертикальные** — отображать значения подписей вертикально.
* **Под углом** — отображать значения подписей под углом 45 градусов. ||
|| Пустые значения (null) | Позволяет выбрать способ обработки пустых значений.<br/><br/>Доступные значения:

* **Не отображать** — не отображать пустые значения в чарте.
* **Соединять** — соединять значения полей, между которыми находятся пустые значения.
* **Отображать как 0** — отображать пустые значения в чартах, как нулевые (0) значения поля.
* **Использовать предыдущее** — заменять пустые значения в чарте на значение предыдущей точки на оси. Доступно в настройках оси Y для [накопительной диаграммы](../../visualization-ref/area-chart.md). ||
|| Масштабирование | Задает масштаб осей чарта.<br/><br/>Доступные значения:

* **Авто** — использовать автоматический масштаб. Вы можете указать, каким образом {{ datalens-short-name }} задает масштаб: от 0 до максимального значения поля (**Автомасштаб от 0 до max**) или от минимального до максимального значений полей (**Автомасштаб от min до max**).
* **Вручную** — указать масштаб оси вручную. Вы можете задать максимальное и минимальное значения по оси — {{ datalens-short-name }} обрежет линии чарта по этому значению. ||
|#

### Цвета {#color-settings}

В общем случае вы можете задать определенный цвет для какого-либо значения на графике.

Для [древовидной диаграммы](../../visualization-ref/tree-chart.md), [таблицы](../../visualization-ref/table-chart.md) (в том числе [сводной](../../visualization-ref/pivot-table-chart.md)) и [карты](../../visualization-ref/map-chart.md) доступны следующие настройки:

#|
|| **Название** | **Описание** ||
|| Тип градиента | Задает количество цветов, используемых в градиенте.<br/><br/>Доступные значения:

* **Двухцветный** — задает два цвета для градиента.
* **Трехцветный** — задает три цвета для градиента. ||
|| Цвет | Задает цвет для значения. Доступные цвета зависят от типа градиента. ||
|| Задать пороговые значения | Позволяет задать пороговые значения, которые будут соответствовать каждому цвету. ||
|| Границы | Задает границы для геополигонов.<br/><br/>Доступные значения:

* **Показать** — отображать границы геополигонов.
* **Скрыть** — скрыть границы геополигонов. ||
|#

Если в [общих настройках](#common-settings) включена опция **Легенда**, то при помещении поля в секцию **Цвета**, под чартом отображается легенда. Легенда содержит цвета чарта с текстовыми описаниями, которые соответствуют значениям поля в секции **Цвета**

{% cut "Доступные настройки цвета" %}

Чтобы настроить цвета:

1. В правом верхнем углу секции **Цвета** нажмите значок ![image](../../../_assets/console-icons/gear.svg) (значок появляется при наведении указателя на секцию).
1. Укажите **Тип заливки**:

   {% note info %}

   Для измерения доступен тип заливки **Палитра**, для показателя — **Градиент**.

   {% endnote %}

   {% list tabs group=fill %}

   - Для измерения {#measure}

     1. Нажмите на поле выбора цветовой схемы и укажите цвет для каждого значения измерения.
     1. Нажмите кнопку **Применить**.

   - Для показателя {#indicator}

     1. Нажмите на поле выбора градиента и настройте:

        * **Тип градиента** — выберите двухцветный или трехцветный.

          * Цвет градиента — выберите цветовую гамму градиента из списка.
          * Направление градиента — измените направление градиента с помощью значка ![image](../../../_assets/console-icons/arrow-right-arrow-left.svg).

        * **Задать пороговые значения** — установите пороговые числовые значения, которые будут соответствовать каждому цвету.

     1. Нажмите кнопку **Применить**.

   {% endlist %}

{% endcut %}

Вы можете [создать палитры](../../operations/chart/create-palette.md) цветов и использовать их в своих чартах.

### Подписи {#sign}

Отображают значение показателя в чарте. Поддерживается использование [функций разметки](../../function-ref/markup-functions.md). Для полей с типом `Строка` можно настроить использование базового синтаксиса [{#T}](../../dashboard/markdown.md): нажмите на значок перед названием поля и включите опцию **Markdown**.


Подписи доступны для чартов типа:

* Линейная диаграмма.
* Диаграмма с областями (Накопительная и Нормированная).
* Столбчатая диаграмма (в том числе Нормированная).
* Линейчатая диаграмма (в том числе Нормированная).
* Круговая диаграмма.
* Карта.

### Сортировка {#sort}

Позволяет сортировать значения на чарте по показателю или измерению.

Сортировка доступна для чартов типа:

* Линейная диаграмма.
* Диаграмма с областями (Накопительная и Нормированная).
* Столбчатая диаграмма (в том числе Нормированная).
* Линейчатая диаграмма (в том числе Нормированная).
* Круговая диаграмма.
* Таблица (в том числе Сводная).

### Фильтры {#filter}

Позволяют делать выборку значений по измерениям или показателям.
Фильтры доступны для всех типов чартов.


## Настройки секций карт {#map-settings}

Вы можете настраивать слои, размер и цвет точек, тултипы и фильтры.
Чтобы открыть настройки секции, нажмите значок ![image](../../../_assets/console-icons/gear.svg) в строке с названием секции.

{% note warning %}

В зависимости от типа визуализации доступны разные настройки.

{% endnote %}

#|
|| **Название** | **Описание** ||
|| Размер | Задает размер точки в зависимости от значения показателя. ||
|| Цвета | Задают цвет для геоточек и геополигонов в зависимости от значения показателя. ||
|| Тултипы | Позволяют сделать подсказку, которая отобразится при наведении указателя на точку. Подсказка будет содержать значения измерений и показателей. Для полей с типом `Строка` можно настроить использование базового синтаксиса [{#T}](../../dashboard/markdown.md): нажмите на значок перед названием поля и включите опцию **Markdown**. ||
|| Фильтры слоя | Позволяют делать выборку значений для текущего слоя по измерениям или показателям. ||
|| Общие фильтры | Позволяют делать выборку значений для всего чарта по измерениям или показателям. ||
|#

### Слои {#map-layer}

На одну карту можно добавить не более 5 слоев с любыми типами визуализации:

* Точечная карта — тип измерения `Геоточки`.
* Фоновая карта — тип измерения `Геополигоны`.
* Тепловая карта — тип измерения `Геоточки (тепловая карта)`.

Слои можно удалять и переименовывать.

Можно изменить прозрачность текущего слоя с помощью ползунка. Также можно задать прозрачность в текстовом поле над ползунком.

![slider](../../../_assets/datalens/chart-settings/01-alpha-slider.png)

Прозрачность может принимать значение от 0 до 100.

### Цвета {#map-color}

Вы можете задать цвет для геоточек и геополигонов, который будет зависеть от значения показателя.

#|
|| **Название** | **Описание** ||
|| Тип градиента | Задает количество цветов, используемых в градиенте.<br/><br/>Доступные значения:

* **Двухцветный** — задает два цвета для градиента.
* **Трехцветный** — задает три цвета для градиента. ||
|| Границы | Задает границы для геополигонов.<br/><br/>Доступные значения:

* **Показать** — отображать границы геополигонов.
* **Скрыть** — скрыть границы геополигонов. ||
|| Цвет | Задает цвет для геоточек и геополигонов. Доступные цвета зависят от типа градиента. ||
|| Задать пороговые значения | Позволяет задать пороговые значения, которые будут соответствовать каждому цвету. ||
|#

Вы можете [создать палитры](../../operations/chart/create-palette.md) цветов и использовать их в своих чартах.


## Отмена и восстановление изменений в чартах {#undo-redo}

При редактировании чарта в визарде или [QL-чарта](./ql-charts.md) можно отменить или повторно выполнить внесенные изменения в пределах текущей версии:

* чтобы отменить изменения, нажмите значок ![image](../../../_assets/console-icons/arrow-uturn-ccw-left.svg) в верхней правой части экрана или сочетание клавиш **Ctrl** (**Cmd**) + **Z**;
* чтобы восстановить изменения, нажмите значок ![image](../../../_assets/console-icons/arrow-uturn-cw-right.svg) или сочетание клавиш **Ctrl** (**Cmd**) + **Shift** + **Z**.

Несохраненные изменения в текущей версии сбрасываются:


* при обновлении страницы;
* при сохранении чарта;
* при переключении на другую [версию](./versioning.md).


