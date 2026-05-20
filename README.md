# Лабораторная работа №7

[![CMake CI](https://github.com/VincentTheBrainfuqqer/lab07/actions/workflows/ci.yml/badge.svg)](https://github.com/VincentTheBrainfuqqer/lab07/actions/workflows/ci.yml)

## Цель работы

Изучить систему управления пакетами Hunter и научиться подключать внешние зависимости через пакетный менеджер.

## Задание

В ходе лабораторной работы необходимо:

- создать публичный репозиторий `lab07`;
- взять за основу проект из предыдущей лабораторной работы;
- подключить Hunter;
- заменить локальное подключение GTest на подключение через Hunter;
- добавить локальную конфигурацию Hunter;
- добавить демонстрационное приложение `demo`;
- подключить Polly;
- проверить сборку проекта и запуск тестов.

Домашняя часть в данной работе не выполнялась.

## Выполнение работы

Сначала был создан репозиторий `lab07`, после чего в него была перенесена структура проекта из лабораторной работы №6.

~~~bash
git clone --recurse-submodules https://github.com/VincentTheBrainfuqqer/lab06 projects/lab07
cd projects/lab07
git remote remove origin
git remote add origin https://github.com/VincentTheBrainfuqqer/lab07.git
git branch -M main
~~~

После этого были удалены временные файлы сборки.

~~~bash
rm -rf _build _builds artifacts
~~~

Для подключения Hunter был создан каталог `cmake`, после чего был загружен файл `HunterGate.cmake`.

~~~bash
mkdir -p cmake
wget https://raw.githubusercontent.com/cpp-pm/gate/master/cmake/HunterGate.cmake -O cmake/HunterGate.cmake
~~~

В файл `CMakeLists.txt` было добавлено подключение Hunter.

~~~cmake
include("cmake/HunterGate.cmake")

HunterGate(
    URL "https://github.com/cpp-pm/hunter/archive/v0.23.251.tar.gz"
    SHA1 "5659b15dc0884d4b03dbd95710e6a1fa0fc3258d"
    LOCAL
)
~~~

После этого локальная копия GTest была удалена из проекта.

~~~bash
git rm -rf third-party/gtest
~~~

Вместо локального подключения GTest было добавлено подключение через Hunter.

~~~cmake
hunter_add_package(GTest)
find_package(GTest CONFIG REQUIRED)
~~~

Также была изменена линковка тестов.

~~~cmake
target_link_libraries(check print GTest::main)
~~~

Теперь библиотека GTest не хранится внутри проекта, а загружается автоматически через Hunter во время конфигурации проекта.

Для настройки версии пакета был создан файл `cmake/Hunter/config.cmake`.

~~~cmake
hunter_config(GTest VERSION 1.7.0-hunter-9)
~~~

Далее было создано демонстрационное приложение `demo`.

~~~bash
mkdir demo
~~~

Файл `demo/main.cpp`:

~~~cpp
#include <print.hpp>

#include <cstdlib>
#include <fstream>
#include <iostream>
#include <string>

int main()
{
    const char* log_path = std::getenv("LOG_PATH");

    if (log_path == nullptr)
    {
        std::cerr << "undefined environment variable: LOG_PATH" << std::endl;
        return 1;
    }

    std::string text;

    while (std::cin >> text)
    {
        std::ofstream out{log_path, std::ios_base::app};
        print(text, out);
        out << std::endl;
    }

    return 0;
}
~~~

Приложение считывает слова из стандартного ввода и записывает их в файл. Путь к файлу задается через переменную окружения `LOG_PATH`.

В `CMakeLists.txt` были добавлены правила сборки и установки приложения `demo`.

~~~cmake
add_executable(demo ${CMAKE_CURRENT_SOURCE_DIR}/demo/main.cpp)
target_link_libraries(demo print)

install(TARGETS demo
    RUNTIME DESTINATION bin
)
~~~

После этого был подключен Polly.

~~~bash
mkdir -p tools
git submodule add https://github.com/ruslo/polly tools/polly
~~~

Polly был добавлен как git submodule, поэтому информация о нем появилась в файле `.gitmodules`.

~~~ini
[submodule "tools/polly"]
	path = tools/polly
	url = https://github.com/ruslo/polly
~~~

Для проверки Polly были выполнены команды:

~~~bash
python3 tools/polly/bin/polly.py --test
python3 tools/polly/bin/polly.py --install
python3 tools/polly/bin/polly.py --toolchain clang-cxx14
~~~

## Сборка проекта

Для сборки проекта использовались команды:

~~~bash
cmake -H. -B_builds -DBUILD_TESTS=ON
cmake --build _builds
~~~

Для запуска тестов использовалась команда:

~~~bash
cmake --build _builds --target test
~~~

Во время конфигурации проекта Hunter автоматически загружает нужную версию GTest и подключает ее к проекту.

Также была проверена директория Hunter.

~~~bash
ls -la $HOME/.hunter
~~~

## Проверка demo

Для проверки демонстрационного приложения была задана переменная окружения `LOG_PATH`.

~~~bash
export LOG_PATH=log.txt
~~~

После этого был выполнен запуск приложения.

~~~bash
echo "hello world" | ./_builds/demo
cat log.txt
~~~

В результате в файл `log.txt` были записаны слова, переданные во входной поток.

~~~text
hello
world
~~~

## GitHub Actions

Для автоматической проверки проекта используется GitHub Actions.

Файл workflow расположен по пути:

~~~text
.github/workflows/ci.yml
~~~

При каждом push в ветку `main` выполняются следующие действия:

- клонирование репозитория;
- загрузка submodule;
- конфигурация проекта через CMake;
- сборка проекта;
- запуск тестов;
- установка проекта;
- сборка пакета.

Это позволяет автоматически проверять, что проект собирается и тесты проходят успешно.

## Структура проекта

~~~text
lab07/
├── .github/
│   └── workflows/
│       └── ci.yml
├── cmake/
│   ├── Hunter/
│   │   └── config.cmake
│   └── HunterGate.cmake
├── demo/
│   └── main.cpp
├── examples/
│   ├── example1.cpp
│   └── example2.cpp
├── include/
│   └── print.hpp
├── sources/
│   └── print.cpp
├── tests/
│   └── test1.cpp
├── tools/
│   └── polly/
├── .gitignore
├── .gitmodules
├── CMakeLists.txt
├── CPackConfig.cmake
├── ChangeLog.md
├── DESCRIPTION
├── LICENSE
└── README.md
~~~

## Результат

В результате выполнения лабораторной работы:

- был создан репозиторий `lab07`;
- был подключен Hunter;
- локальная зависимость `third-party/gtest` была удалена;
- GTest был подключен через Hunter;
- была добавлена локальная конфигурация Hunter;
- было создано демонстрационное приложение `demo`;
- был подключен Polly;
- проект успешно собирается;
- тесты успешно проходят;
- проект проверяется через GitHub Actions.

## Вывод

В ходе лабораторной работы была изучена система управления пакетами Hunter.

Было показано, как подключать внешние библиотеки без хранения их исходного кода внутри проекта. Локальное подключение GTest было заменено на подключение через Hunter. Также была добавлена локальная конфигурация Hunter, позволяющая указать нужную версию пакета.

Дополнительно был подключен Polly, который позволяет использовать готовые toolchain для сборки проекта. В результате проект стал чище, так как внешние зависимости теперь управляются через пакетный менеджер, а не хранятся напрямую в репозитории.
