# Часто задаваемые вопросы

## Я нашел ужасный баг: TRegExpr вызывает исключение нарушения доступа!

**Ответ**

Вы должны создать объект перед его использованием. 
Поэтому, после того как вы объявили что-то вроде:

``` pascal
r : TRegExpr
```

не забудьте создать экземпляр объекта:

``` pascal
r := TRegExpr.Create. 
```

## Поддерживает ли он Unicode?

**Ответ**

[Как использовать Unicode](tregexpr.md#unicode)

## Почему TRegExpr возвращает больше одной строки?

Например, рег. выражение `<font .\*>` возвращает первый `<font`, а затем остальную часть 
файла, включая последний `</html>`.

**Ответ**

Для обратной совместимости [модификатор /s](regular_expressions.md#s) по умолчанию включен.

Отключите его, и `.` будет соответствовать любому символу, кроме 
[разделителей строк](regular_expressions.md#lineseparators) - именно так, как вы хотите.

Кстати, я предлагаю `<font ([^\n>]*)>`, в `Match[1]` будет URL.

## Почему TRegExpr возвращает больше, чем я ожидал?

Например, рег. выражение `<p>(.+)</p>` примененное к строке `<p>a</p><p>b</p>` 
возвращает `a</p><p>b`, а не `a`, как я ожидал.

**Ответ**

По умолчанию все операторы работают в `жадном` режиме, поэтому они соответствуют как 
можно большему количеству.

Если вы хотите `нежадный` режим, вы можете использовать `нежадные` операторы, такие 
как `+?` и так далее, или переключить все операторы в `нежадный` режим с помощью модификатора `g` (используйте соответствующие свойства TRegExpr или оператор `?(-g)` в рег. выражении).

## Как анализировать HTML с помощью TRegExpr?

**Ответ**

Извините, но это практически невозможно!

Конечно, вы можете легко использовать TRegExpr для извлечения некоторой информации из 
HTML, как показано в моих примерах, но если вы хотите точного анализа, вам нужен 
настоящий парсер, а не рег. выражения.

Вы можете прочитать полное объяснение, например, в книге Тома Кристиансена и Натана 
Торкингтона `Perl Cookbook`.

Короче говоря - многие структуры, которые могут быть легко проанализированы настоящим 
парсером, вообще не могут быть проанализированы рег. выражениями, и настоящий парсер 
гораздо быстрее справляется с анализом, потому что рег. выражения не просто сканируют 
входной поток, они выполняют поиск с оптимизацией, который может занять много времени.

## Есть ли способ получить несколько совпадений шаблона в TRegExpr?

**Ответ**

Вы можете итерировать совпадения с помощью метода ExecNext.

Если вам нужен какой-то пример, пожалуйста, посмотрите реализацию метода 
`TRegExpr.Replace` или примеры для [HyperLinksDecorator](demos.md)

## Я проверяю пользовательский ввод. Почему TRegExpr возвращает `True` для неправильных входных строк?

**Ответ**

Во многих случаях пользователи TRegExpr забывают, что регулярное выражение предназначено 
для **поиска** во входной строке.

Так, например, если вы используете выражение `\d{4,4}`, вы получите успех для неправильных 
пользовательских вводов, таких как `12345` или `любые буквы 1234`.

Вы должны проверять от начала строки до конца, чтобы убедиться, что вокруг нет ничего 
лишнего: `^\d{4,4}$`.

## Почему иногда нежадные итераторы работают как в жадном режиме?

Например, рег. выражение `a+?,b+?` примененное к строке `aaa,bbb` соответствует 
`aaa,b`, но не должно ли оно соответствовать `a,b` из-за нежадности первого итератора?

**Ответ**

Это из-за способа работы TRegExpr. Фактически, многие другие движки рег. выражений 
работают точно так же: они выполняют только `простую` оптимизацию поиска и не пытаются 
найти лучшее решение.

В некоторых случаях это плохо, но в общем это скорее преимущество, чем недостаток, по 
причинам производительности и предсказуемости.

Основное правило - рег. выражение в первую очередь пытается найти совпадение с текущего 
места и только если это совершенно невозможно, перемещается вперед на один символ и 
пытается снова с следующей позиции в тексте.

Так что, если вы используете `a,b+?`, это будет соответствовать `a,b`. В случае `a+?,b+?` 
это теперь не рекомендуется (мы добавили модификатор нежадности), но все еще возможно 
соответствие более чем одной `a`, поэтому TRegExpr будет это делать.

TRegExpr, как и Perl или Unix r.e., не пытается переместиться вперед и проверить, будет 
ли это "лучше" соответствие. Прежде всего, потому что нет способа сказать, что 
совпадение лучше или хуже.

## Как я могу использовать TRegExpr с Borland C++ Builder?

У меня проблема, так как нет доступного файла заголовка (`.h` или `.hpp`).

**Ответ**

- Добавьте `RegExpr.pas` в проект `bcb`.
- Скомпилируйте проект. Это генерирует файл заголовка `RegExpr.hpp`.
- Теперь вы можете писать код, который использует единицу `RegExpr`.
- Не забудьте добавить `#include “RegExpr.hpp”`, где это необходимо.
- Не забудьте заменить все `\` в регулярных выражениях на `\\` или переопределить константу [EscChar](tregexpr.md#escchar).

## Почему многие рег. выражения (включая рег. выражения из справки и демо TRegExpr) работают неправильно в Borland C++ Builder?

**Ответ**

Подсказка в предыдущем вопросе ;) Символ `\` имеет специальное значение в `C++`, поэтому 
вы должны `экранировать` его (как описано в предыдущем ответе). 

Но если вам не нравятся рег. выражения вроде `\\w+\\w+\\.\\w+`, вы можете переопределить 
константу `EscChar` (в `RegExpr.pas`). Например, `EscChar = "/"`. 

Тогда вы можете писать `/w+/w+/./w+`, выглядит необычно, но более читаемо.

