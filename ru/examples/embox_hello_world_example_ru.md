# Пример “Hello World”
Чтобы написать пример команды "Hello, World!" -- нужно создать файл "hello.c" с исходным кодом на языке "C":
```c
#include <stdio.h>

int main(int argc, char **argv) {
	printf("Hello World!\n");
	return 0;
}
```
И затем добавить описание модуля.

## Добавление модуля
Все модули и интерфейсы системы описываются в "my"-файлах. **"My"-файлы** -- это файлы, имеющие расширение **".my"** или имя **"Mybuild"**.

**Структурно каждый "my"-файл содержит:**

 * объявление пакета, которому принадлежат все определяемые в файле сущности
 * список импортируемых имен из других пакетов (опционально)
 * определения самих модулей и интерфейсов

Например, чтобы добавить команду в командный интерпретатор, встроенный в ядро, нужно создать файл "Hello.my" с таким содержанием:
```java
package embox.cmd.tutorial

@AutoCmd
@Cmd(name="hello", help="Prints ‘Hello World’ string")
module hello {
	source "hello.c"
}
```
В этом примере мы описали простой модуль с единственным атрибутом "source" -- файлом "hello.c", который будем скомпилирован и связан с ядром, если модуль будет добавлен в сборку.

**Аннотация "@Cmd"** регистрирует модуль в системе как команду встроенного интерпретатора. Это позволяет запускать модуль по имени "hello".

Опциональный **параметр "help"** содержит строку, которая будет выведана при запуске команды “help hello” (если модуль "help" включен в сборку).

**Аннотация "@AutoCmd"** позволяет использовать привычную функцию "main()" в качестве точки входа в программу.

### Атрибуты модуля
**Модуль "hello"** достаточно примитивен и не определяет никаких внешних зависимостей или опций.
Единственный его атрибут - "source" - определяет набор исходных файлов для компиляции.

### Опции
Теперь давайте зададим строку приветствия в виде опции. Для этого мы воспользуемся атрибутом "option" и добавим модулю "hello" опцию "greeting":
```c
// ...
module hello {
	// ...
	option string greeting = "World"
}
```
И изменим файл "hello.c" таким образом, чтобы команда после слова "Hello" выводила строку, которая содержится в значении опции "greeting":
```c
#define GREETING OPTION_STRING_GET(greeting)

// ...
	printf("Hello %s!\n", GREETING);
// ...
```

## Включение в сборку
Для того, чтобы новый модуль оказался в результирующем образе ядра, его необходимо добавить в конфигурацию сборки, которая описывается в файле "conf/mods.conf":
```
package genconfig

configuration conf {
	// ...
	include embox.cmd.tutorial.hello
}
```
Чтобы изменить значение опции, следует указать новое значение в скобках:
```
// ...
	include embox.cmd.tutorial.hello(greeting="Everyone")
// ...
```
Как и многие атрибуты модуля, конфигурация поддерживает добавление аннотаций.
Чаще всего используется аннотация **"@Runlevel"**. Она позволяет загружать модули поэтапно и определять порядок их загрузки.

**Как правило, на ранних стадиях (runlevel=0) загружаются:**

 * драйверы устройств, необходимых для корректного функционирования системы (например, контроллер прерываний)
 * основные компоненты системы
Также на ранних стадиях исполняются низкоуровневые тесты и процедуры самодиагностики.

**Обратите внимание**: порядок загрузки модулей по зависимостям имеет самый высокий приоритет. Поэтому опция "@Runlevel" работает в тех случаях, когда она не противоречит порядку загрузки по зависимостям.

