# Лабораторная работа №8

## Цель работы

Изучить основы контейнеризации приложений с помощью Docker и научиться запускать собранное C++ приложение внутри Docker-контейнера.

## Задание

В ходе лабораторной работы необходимо:

- создать публичный репозиторий `lab08`;
- взять за основу проект из предыдущей лабораторной работы;
- создать `Dockerfile`;
- собрать Docker-образ с именем `logger`;
- запустить контейнер на основе созданного образа;
- подключить директорию с логами через volume;
- проверить запись данных в лог-файл;
- настроить автоматическую проверку проекта через GitHub Actions.

Домашняя часть в данной работе не выполнялась.

## Выполнение работы

Сначала был создан репозиторий `lab08`, после чего в него был перенесен проект из лабораторной работы №7.

    cd ~/VincentTheBrainfuqqer/workspace/projects
    git clone --recurse-submodules https://github.com/VincentTheBrainfuqqer/lab07 lab08
    cd lab08

После этого был изменен удаленный репозиторий.

    git remote remove origin
    git remote add origin https://github.com/VincentTheBrainfuqqer/lab08.git
    git branch -M main

Далее были удалены временные файлы сборки и служебные каталоги, которые не должны попадать в репозиторий.

    rm -rf _build _builds _install _logs artifacts logs

Также был обновлен файл `.gitignore`.

    _build/
    _builds/
    _install/
    _logs/
    artifacts/
    logs/

    *.tar.gz
    *.zip
    *.deb
    *.rpm
    *.dmg
    *.msi

    log.txt
    file.txt

## Создание Dockerfile

Для сборки и запуска проекта внутри контейнера был создан файл `Dockerfile`.

    FROM ubuntu:18.04

    RUN apt-get update && apt-get install -y \
        build-essential \
        cmake \
        git \
        ca-certificates \
        && rm -rf /var/lib/apt/lists/*

    COPY . /print
    WORKDIR /print

    RUN cmake -H. -B_build -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=_install
    RUN cmake --build _build
    RUN cmake --build _build --target install

    ENV LOG_PATH=/home/logs/log.txt

    VOLUME /home/logs

    WORKDIR /print/_install/bin

    ENTRYPOINT ["./demo"]

В данном файле используется базовый образ `ubuntu:18.04`.

В контейнер устанавливаются необходимые инструменты для сборки проекта:

- `build-essential`;
- `cmake`;
- `git`;
- `ca-certificates`.

После этого исходные файлы проекта копируются в каталог `/print`.

    COPY . /print
    WORKDIR /print

Затем выполняется конфигурация проекта через CMake.

    RUN cmake -H. -B_build -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=_install

После конфигурации выполняется сборка проекта.

    RUN cmake --build _build

Затем выполняется установка проекта во внутреннюю директорию `_install`.

    RUN cmake --build _build --target install

Для демонстрационного приложения задается переменная окружения `LOG_PATH`.

    ENV LOG_PATH=/home/logs/log.txt

Она указывает путь к файлу, в который приложение будет записывать данные.

Также создается volume.

    VOLUME /home/logs

Это нужно для того, чтобы файл логов сохранялся не только внутри контейнера, но и был доступен на основной системе.

В конце указывается рабочая директория и команда запуска.

    WORKDIR /print/_install/bin

    ENTRYPOINT ["./demo"]

При запуске контейнера автоматически запускается приложение `demo`.

## Сборка Docker-образа

Для сборки Docker-образа была использована команда:

    docker build -t logger .

Флаг `-t logger` задает имя образа.

В результате был создан Docker-образ `logger`, внутри которого находится собранный проект.

## Запуск контейнера

Для запуска контейнера была создана директория `logs`.

    mkdir -p logs

После этого контейнер был запущен командой:

    docker run -it -v "$(pwd)/logs:/home/logs" logger

Опция `-v` подключает локальный каталог `logs` к каталогу `/home/logs` внутри контейнера.

То есть приложение внутри контейнера записывает данные в файл:

    /home/logs/log.txt

А на основной системе этот файл находится по пути:

    logs/log.txt

## Проверка работы приложения

После запуска контейнера были введены строки:

    text1
    text2
    text3

После завершения ввода была выполнена проверка содержимого файла логов.

    cat logs/log.txt

В результате в файле появились строки, которые были переданы во входной поток приложения.

    text1
    text2
    text3

Это означает, что контейнер был запущен корректно, volume был подключен правильно, а приложение успешно записало данные в лог-файл.

## GitHub Actions

Для автоматической проверки проекта был добавлен workflow GitHub Actions.

Файл workflow расположен по пути:

    .github/workflows/ci.yml

Содержимое файла:

    name: Docker CI

    on:
      push:
        branches: [ main ]
      pull_request:
        branches: [ main ]

    jobs:
      docker:
        runs-on: ubuntu-latest

        steps:
          - name: Checkout repository
            uses: actions/checkout@v4
            with:
              submodules: recursive

          - name: Build Docker image
            run: docker build -t logger .

          - name: Run Docker container
            run: |
              mkdir -p logs
              printf "text1\ntext2\ntext3\n" | docker run -i -v "$(pwd)/logs:/home/logs" logger

          - name: Check log file
            run: cat logs/log.txt

При каждом push в ветку `main` выполняются следующие действия:

- клонирование репозитория;
- загрузка submodule;
- сборка Docker-образа;
- запуск Docker-контейнера;
- передача тестовых строк во входной поток приложения;
- проверка содержимого файла логов.

Это позволяет автоматически проверить, что Docker-образ собирается, контейнер запускается, а приложение корректно записывает данные в лог-файл.

## Структура проекта

    lab08/
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
    ├── Dockerfile
    ├── LICENSE
    └── README.md

## Результат

В результате выполнения лабораторной работы:

- был создан репозиторий `lab08`;
- за основу был взят проект из лабораторной работы №7;
- был создан `Dockerfile`;
- был собран Docker-образ `logger`;
- был запущен Docker-контейнер;
- был подключен volume для хранения логов;
- была проверена запись данных в файл `logs/log.txt`;
- была настроена автоматическая проверка через GitHub Actions.

## Вывод

В ходе лабораторной работы были изучены базовые принципы контейнеризации приложений с помощью Docker.

Было показано, как описать окружение для сборки проекта в файле `Dockerfile`, как собрать Docker-образ и как запустить приложение внутри контейнера. Также был использован volume, позволяющий сохранять данные, созданные внутри контейнера, на основной системе.

Дополнительно была настроена автоматическая проверка через GitHub Actions. При каждом push проект автоматически собирается в Docker-образ, запускается контейнер и проверяется создание лог-файла. Это позволяет убедиться, что приложение работает не только локально, но и в чистом окружении контейнера.
