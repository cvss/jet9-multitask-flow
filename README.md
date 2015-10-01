

Утилита для выполнения init-скриптов в пользовательском контейнере.

### Выполнение

Используются переменные окружения INIT_DIR и LOCK_DIR, указывающие на каталог с выполняемыми скриптами и на каталог для временных файлов во время исполнения.

В аргументах указывается команда (action), например, `start` или `stop`. Эта команда будет передана для каждого выполняемого скрипта. После команды может быть указан один или несколько скриптов, для которых должна быть передана эта команда. Если скрипты в аргументах не указаны, то берутся все исполняемые файлы из каталога INIT_DIR.

Названия скриптов должны иметь только буквы, цифры и подчеркивание. Дефисы и точки не допускаются.


    Usage: init [-a] [-n] [-v] [-d] action [tasks ...]
    
    Arguments:
        -a       - add all WAIT-dependent tasks to run list
        -n       - dry run
        -v       - verbose
        -d       - output dependencies for action, can be used for 'tsort'
                   to check for problems
        action   - start, stop or another action
        tasks    - task scripts from INIT_DIR;
                   all scripts from INIT_DIR used if tasks are omitted
    
    Environment:
        INIT_DIR=/own/etc/init
        LOCK_DIR=/run/init


#### Зависимости

Скрипты в INIT_DIR или в аргументах init выполняются параллельно. Скрипты могут выполняться с учетом зависимостей. Если для выполнения одного скрипта требуется, чтобы перед ним уже был выполнен другой скрипт, это указывается через зависимость. Параллельное выполнение учитывает зависимости и задерживает выполнение зависимых скриптов до тех пор, пока не будут выполнены все требуемые зависимости. Порядок с зависимостями соблюдается как для выполнения всех скриптов из INIT_DIR, так и при перечислении скриптов в аргументах программы.

Скрипты могут сообщать о своих зависимостях, отвечая на команду `wait` или `wait _action_`, где в качестве _action_ указывается команда, для которой существует эта зависимость. Например, при старте apache может быть зависимость на mysql, а при остановке apache может быть зависимость на nginx.

Название скрипта является именем зависимости. Если есть скрипт с названием mysql, на него можно устанавливать зависимость.


##### Виртуальные зависимости

Могут также быть "виртуальные" зависимости, которые не имеют скриптов. Например, для старта `apache` может быть указана виртуальная зависимость database. Эта зависимость не является скриптом, но другие скрипты могут отмечать себя в качестве этой зависимости. Таких скриптов может быть несколько. Например, скрипты `mysql` и `postrgresql` могут являться не только зависимостями mysql и postgresql, но и реализовывать виртуальную зависимость database. Тогда apache будет ожидать выполнения обоих скриптов mysql и postgresql, для выполнения ими зависимости database. Скрипты могут сообщать от виртуальных зависимостях, которые они поставляют, отвечая на команду `mark` или `mark _action_`.

##### Автоматическое добавление зависимых задач

При выполнении перечисленных в аргументах задач можно с помощью ключа `-a` автоматически выполнять другие задачи (и реальные скрипты, и зависимости MARK), от которых зависят задачи в аргументах. Например, если указать `start apache` с ключем `-a`, то будет также выполнена зависимость `database`, которая в свою очередь потребует выполнение `mysql` и `postgresql`. С помощью ключа `-a` можно также указывать команду для виртуальных MARK-задач, например, `-a start database`.

#### Прерывание процесса

Init-процесс можно остановить SIGINT и SIGTERM, при этом те скрипты, которые ожидали выполнения своих зависимостей, будут прерваны.

#### Защита от повторного выполнения

LOCK_DIR создается в момент выполнения скриптов и удаляется после завершения. Если при запуске init каталог уже существует, это означает, чтобы ранее был запущен и еще не завершен другой init. Init, обнаруживший существование LOCK_DIR, завершается сразу же с ошибкой.

#### Холостой прогон

Ключ -n выполняет прогон init-процесса с соблюдением зависимостей для указанной команды, но не передавая команду в сами init-скрипты.

#### Циклы и несуществующие зависимости

Программа не проверяет полученные зависимости на наличие циклов или несуществующих зависимостей. Несуществующие зависимости на выполнение При несуществующей зависимости выполнение будет продолжено. При наличии цикла программа заблокируется (dead lock). С помощью ключа -d можно получить граф зависимостей в виде списка зависимых пар, и передать его на вход программы топологической сортировки `tsort`, которая может показывать петли и несуществующие зависимости.

#### Ошибки в выполнении скриптов

Код выполнения скриптов не проверяется, при ошибке работа будет продолжена.


#### Включение shell-команд из INIT_DIR

После парсинга аргументов программа проверяет наличие файла .init.sh в каталоге INIT_DIR. Если он присутствует, то файл включается в текущий скрипт. Это можно использовать для корректировки используемых флагов или для переопределения функций.


##### Переопределение функций

Через включаемый скрипт можно переопределить фнукцию task_dependencies(), которая извлекает для скрипта его wait и mark зависимости. Фукнция принимает 4 аргумента: отношение (wait или mark), название задачи, команда, полный путь к скрипту. После выполнения функции, она должна сохранить имена зависимостей в переменную task_dependencies_result.

В программе функция определения зависимостей выполняет скрипт, передавая ему имя отношения и команду, и читает результат из его стандартного вывода. Переопределив эту функцию можно получать зависимости с помощью grep или awk из заголовков скриптов, либо любым другим образом.




## LICENSE:

Copyright © 2014-2015 Cyril Vechera http://jet9.net
All rights reserved.

BSD-2

Redistribution and use in source and binary forms, with or without modification, are permitted provided that the following conditions are met:

1. Redistributions of source code must retain the above copyright notice, this list of conditions and the following disclaimer.

2. Redistributions in binary form must reproduce the above copyright notice, this list of conditions and the following disclaimer in the documentation and/or other materials provided with the distribution.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.