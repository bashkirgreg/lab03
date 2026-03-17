# lab03

Отчёт по третьей лабораторной работе.

## Laboratory work III

Данная лабораторная работа посвещена изучению систем автоматизации сборки проекта на примере **CMake**.

## Tutorial

Экспортируем переменную окружения:
```sh
$ export GITHUB_USERNAME=<имя_пользователя>
```

Переходим в рабочую директорию профиля на GitHub, создавая точку возврата (`pushd`) для последующего возвращения обратно. Команда `pushd .` сохраняет текущее положение директории, позволяя вернуться туда позже командой `popd`:
```sh
$ cd ${GITHUB_USERNAME}/workspace
$ pushd .
$ source scripts/activate
```

Клонируем предыдущий репозиторий (`lab02`), создаем новую папку `projects/lab03`. Затем переустанавливаем удаленное хранилище (`origin`) на вновь созданный репозиторий `lab03`:
```sh
$ git clone https://github.com/${GITHUB_USERNAME}/lab02.git projects/lab03
$ cd projects/lab03
$ git remote remove origin
$ git remote add origin https://github.com/${GITHUB_USERNAME}/lab03.git
```
После команды `git clone https://github.com/${GITHUB_USERNAME}/lab02.git projects/lab03` терминал выдаёт:
```sh
Cloning into 'projects/lab03'...
remote: Enumerating objects: 3036, done.
remote: Counting objects: 100% (3036/3036), done.
remote: Compressing objects: 100% (2334/2334), done.
remote: Total 3036 (delta 542), reused 2994 (delta 522), pack-reused 0 (from 0)
Receiving objects: 100% (3036/3036), 13.56 MiB | 5.54 MiB/s, done.
Resolving deltas: 100% (542/542), done.
```

Компилируем исходный код `sources/print.cpp` в объектный файл (`print.o`), подключая заголовочные файлы из каталога `./include`: 
```sh
$ g++ -std=c++11 -I./include -c sources/print.cpp
```
Проверяем наличие объектного файла с помощью команды `$ ls print.o`:
```sh
print.o
```
Получаем информации о символах в объектном файле с помощью команды `$ nm print.o | grep print`:
```
0000000000000000 T _Z5printRKNSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEERSo
000000000000002a T _Z5printRKNSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEERSt14basic_ofstreamIcS2_E
```
Создаём статическую библиотеку с помощью команды `$ ar rvs print.a print.o`:
```
ar: creating print.a
a - print.o
```
Подтверждаем, что библиотека создана верно и имеет нужный формат через команду `$ file print.a`:
```
print.a: current ar archive
```
Аналогично первой команде, компилируем файл `examples/example1.cpp` в объектный файл `example1.o`:
```
$ g++ -std=c++11 -I./include -c examples/example1.cpp
```
Проверяем существование объектного файла через команду `ls example1.o`:
```
example1.o
```
Соединяем объектный файл `example1.o` и статическую библиотеку `print.a`, формируя исполняемый файл `example1`:
```
$ g++ example1.o print.a -o example1
```
Запускаем итоговую программу с помощью команды `./example1 && echo` и получаем результат:
```
hello
```

Компилируем исходный файл `examples/example2.cpp` в объектный файл (`example2.o`), подключая заголовочные файлы из директории `./include`.
```sh
$ g++ -std=c++11 -I./include -c examples/example2.cpp
```
Получаем информации о символах в объектном файле с помощью команды `$ nm example2.o`:
```
0000000000000000 V DW.ref.__gxx_personality_v0
                 U __gxx_personality_v0
0000000000000000 T main
                 U __stack_chk_fail
                 U _Unwind_Resume
                 U _Z5printRKNSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEERSt14basic_ofstreamIcS2_E
                 U _ZNSt14basic_ofstreamIcSt11char_traitsIcEEC1EPKcSt13_Ios_Openmode
                 U _ZNSt14basic_ofstreamIcSt11char_traitsIcEED1Ev
0000000000000000 W _ZNSt15__new_allocatorIcED1Ev
0000000000000000 W _ZNSt15__new_allocatorIcED2Ev
0000000000000000 n _ZNSt15__new_allocatorIcED5Ev
                 U _ZNSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEC1EPKcRKS3_
                 U _ZNSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEED1Ev
                 U _ZSt21ios_base_library_initv
0000000000000000 r _ZStL19piecewise_construct
```
Соединяем объектный файл `example2.o` и статическую библиотеку `print.a`, формируя исполняемый файл `example2`:
```
$ g++ example2.o print.a -o example2
```
Запускаем программу через команду `$ ./example2` и получаем результат через чтение файла посредством команды `$ cat log.txt && echo`:
```
hello
```

Удаляем ранее созданные нами объектный и исполняемый файлы, а также статическую библиотеку и файл журнала действий:
```sh
$ rm -rf example1.o example2.o print.o
$ rm -rf print.a
$ rm -rf example1 example2
$ rm -rf log.txt
```

Создаём и наполняем файл `CMakeLists.txt`:
```sh
$ cat > CMakeLists.txt <<EOF
cmake_minimum_required(VERSION 3.4)
project(print)
EOF
```

Дополняем файл `CMakeLists.txt` новой информацией:
```sh
$ cat >> CMakeLists.txt <<EOF
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
EOF
```

Продолжаем дополнять файл `CMakeLists.txt`:
```sh
$ cat >> CMakeLists.txt <<EOF
add_library(print STATIC \${CMAKE_CURRENT_SOURCE_DIR}/sources/print.cpp)
EOF
```

И ещё один раз:
```sh
$ cat >> CMakeLists.txt <<EOF
include_directories(\${CMAKE_CURRENT_SOURCE_DIR}/include)
EOF
```

Создаём файлы сборки в директории `_build` с помощью команды `$ cmake -H. -B_build`:
```sh
CMake Deprecation Warning at CMakeLists.txt:1 (cmake_minimum_required):
  Compatibility with CMake < 3.5 will be removed from a future version of
  CMake.

  Update the VERSION argument <min> value or use a ...<max> suffix to tell
  CMake that the project does not need compatibility with older versions.


-- The C compiler identification is GNU 13.3.0
-- The CXX compiler identification is GNU 13.3.0
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working C compiler: /usr/bin/cc - skipped
-- Detecting C compile features
-- Detecting C compile features - done
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Check for working CXX compiler: /usr/bin/c++ - skipped
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Configuring done (0.7s)
-- Generating done (0.0s)
-- Build files have been written to: /home/user1/bashkirgreg/workspace/projects/lab03/_build
```
А затем производим сборку проекта на основании этих файлов через команду `$ cmake --build _build`:
```
[ 50%] Building CXX object CMakeFiles/print.dir/sources/print.cpp.o
[100%] Linking CXX static library libprint.a
[100%] Built target print
```

Вновь дополняем `CMakeLists.txt` новыми данными:
```sh
$ cat >> CMakeLists.txt <<EOF
add_executable(example1 \${CMAKE_CURRENT_SOURCE_DIR}/examples/example1.cpp)
add_executable(example2 \${CMAKE_CURRENT_SOURCE_DIR}/examples/example2.cpp)
EOF
```

И ещё один раз:
```sh
$ cat >> CMakeLists.txt <<EOF
target_link_libraries(example1 print)
target_link_libraries(example2 print)
EOF
```

Запускаем общую сборку проекта через команду `$ cmake --build _build`:
```sh
CMake Deprecation Warning at CMakeLists.txt:1 (cmake_minimum_required):
  Compatibility with CMake < 3.5 will be removed from a future version of
  CMake.

  Update the VERSION argument <min> value or use a ...<max> suffix to tell
  CMake that the project does not need compatibility with older versions.


-- Configuring done (0.0s)
-- Generating done (0.0s)
-- Build files have been written to: /home/user1/bashkirgreg/workspace/projects/lab03/_build
[ 33%] Built target print
[ 50%] Building CXX object CMakeFiles/example1.dir/examples/example1.cpp.o
[ 66%] Linking CXX executable example1
[ 66%] Built target example1
[ 83%] Building CXX object CMakeFiles/example2.dir/examples/example2.cpp.o
[100%] Linking CXX executable example2
[100%] Built target example2
```
Теперь запускаем сборку библиотеки `print` с помощью команды `cmake --build _build --target print`:
```
[100%] Built target print
```
Потом запускаем сборку исполняемого файла `example1` с помощью команды `$ cmake --build _build --target example1`:
```
[ 50%] Built target print
[100%] Built target example1
```
И наконец запускаем сборку исполняемого файла `example2` с помощью команды `$ cmake --build _build --target example2`:
```
[ 50%] Built target print
[100%] Built target example2
```

Смотрим характеристики файла библиотеки с помощью команды `ls -la _build/libprint.a`:
```sh
-rw-rw-r-- 1 user1 user1 2246 Mar 17 13:12 _build/libprint.a
```
Запускаем первый исполняемый файл с подтверждением выполнения через вывод `-hello` с помощью команды `_build/example1 && echo -hello`:
```
hello-hello
```
Запускаем второй исполняемый файл (`_build/example2`) с подтверждением выполнения через вывод `-hello` с помощью команды `cat log.txt && echo -hello`:
```
hello-hello
```
Удаляем появившийся файл журнала через команду `$ rm -rf log.txt`.

[PS]: Я поменял `hello` на `-hello`, чтобы не было путаницы в терминале, поскольку сами файлы итак выдают `hello` в конце.


Заменяем текущий файл `CMakeLists.txt` на свежий экземпляр из удалённого репозитория и чистим временное хранилище:
```sh
$ git clone https://github.com/tp-labs/lab03 tmp
$ mv -f tmp/CMakeLists.txt .
$ rm -rf tmp
```

Выводим содержимое файла `CMakeLists.txt` с помощью команды `cat CMakeLists.txt`:
```sh
cmake_minimum_required(VERSION 3.4)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

option(BUILD_EXAMPLES "Build examples" OFF)

project(print)

add_library(print STATIC ${CMAKE_CURRENT_SOURCE_DIR}/sources/print.cpp)

target_include_directories(print PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
)

if(BUILD_EXAMPLES)
  file(GLOB EXAMPLE_SOURCES "${CMAKE_CURRENT_SOURCE_DIR}/examples/*.cpp")
  foreach(EXAMPLE_SOURCE ${EXAMPLE_SOURCES})
    get_filename_component(EXAMPLE_NAME ${EXAMPLE_SOURCE} NAME_WE)
    add_executable(${EXAMPLE_NAME} ${EXAMPLE_SOURCE})
    target_link_libraries(${EXAMPLE_NAME} print)
    install(TARGETS ${EXAMPLE_NAME}
      RUNTIME DESTINATION bin
    )
  endforeach(EXAMPLE_SOURCE ${EXAMPLE_SOURCES})
endif()

install(TARGETS print
    EXPORT print-config
    ARCHIVE DESTINATION lib
    LIBRARY DESTINATION lib
)

install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/ DESTINATION include)
install(EXPORT print-config DESTINATION cmake)
```
Затем настраиваем проект для сборки, сохраняя сгенерированные файлы в директорию `_build`, и устанавливаем путь установки в директорию `_install` через команду `cmake -H. -B_build -DCMAKE_INSTALL_PREFIX=_install`:
```
CMake Deprecation Warning at CMakeLists.txt:1 (cmake_minimum_required):
  Compatibility with CMake < 3.5 will be removed from a future version of
  CMake.

  Update the VERSION argument <min> value or use a ...<max> suffix to tell
  CMake that the project does not need compatibility with older versions.


-- Configuring done (0.0s)
-- Generating done (0.0s)
-- Build files have been written to: /home/user1/bashkirgreg/workspace/projects/lab03/_build
```
Запускаем сборку проекта и его установку с помощью команды `cmake --build _build --target install`:
```
[100%] Built target print
Install the project...
-- Install configuration: ""
-- Installing: /home/user1/bashkirgreg/workspace/projects/lab03/_install/lib/libprint.a
-- Installing: /home/user1/bashkirgreg/workspace/projects/lab03/_install/include
-- Installing: /home/user1/bashkirgreg/workspace/projects/lab03/_install/include/print.hpp
-- Installing: /home/user1/bashkirgreg/workspace/projects/lab03/_install/cmake/print-config.cmake
-- Installing: /home/user1/bashkirgreg/workspace/projects/lab03/_install/cmake/print-config-noconfig.cmake
```
И в конце, для удобства, выводим древовидную структуру файлов и папок с помощью команды `tree _install`:
```
install
├── cmake
│   ├── print-config.cmake
│   └── print-config-noconfig.cmake
├── include
│   └── print.hpp
└── lib
    └── libprint.a

4 directories, 4 files
```

Начинаем отслеживать файл `CMakeLists.txt`, создаем коммит с изменениями и публикуем его на удаленном репозитории:
```sh
$ git add CMakeLists.txt
$ git commit -m"added CMakeLists.txt"
$ git push origin master
```
Вывод терминала после второй команды:
```
[master 5eb8029] added CMakeLists.txt
 1 file changed, 36 insertions(+)
 create mode 100644 CMakeLists.txt
```
И после третьей команды:
```
Enumerating objects: 3008, done.
Counting objects: 100% (3008/3008), done.
Compressing objects: 100% (2304/2304), done.
Writing objects: 100% (3008/3008), 13.55 MiB | 9.39 MiB/s, done.
Total 3008 (delta 528), reused 3000 (delta 525), pack-reused 0
remote: Resolving deltas: 100% (528/528), done.
remote: 
remote: Create a pull request for 'master' on GitHub by visiting:
remote:      https://github.com/bashkirgreg/lab03/pull/new/master
remote: 
To https://github.com/bashkirgreg/lab03.git
 * [new branch]      master -> master
```


## Report

Возвращаемся в начальную директорию, задаём номер лабораторной, клонируем материалы, создаём директорию отчёта, копируем инструкцию, редактируем отчёт и публикуем его на Gist:
```sh
$ popd
$ export LAB_NUMBER=03
$ git clone https://github.com/tp-labs/lab${LAB_NUMBER} tasks/lab${LAB_NUMBER}
$ mkdir reports/lab${LAB_NUMBER}
$ cp tasks/lab${LAB_NUMBER}/README.md reports/lab${LAB_NUMBER}/REPORT.md
$ cd reports/lab${LAB_NUMBER}
$ edit REPORT.md
$ gist REPORT.md
```


## Homework

Представьте, что вы стажер в компании "Formatter Inc.".
### Задание 1
Вам поручили перейти на систему автоматизированной сборки **CMake**.
Исходные файлы находятся в директории [formatter_lib](formatter_lib).
В этой директории находятся файлы для статической библиотеки *formatter*.
Создайте `CMakeList.txt` в директории [formatter_lib](formatter_lib),
с помощью которого можно будет собирать статическую библиотеку *formatter*.

### Задание 2
У компании "Formatter Inc." есть перспективная библиотека,
которая является расширением предыдущей библиотеки. Т.к. вы уже овладели
навыком созданием `CMakeList.txt` для статической библиотеки *formatter*, ваш 
руководитель поручает заняться созданием `CMakeList.txt` для библиотеки 
*formatter_ex*, которая в свою очередь использует библиотеку *formatter*.

### Задание 3
Конечно же ваша компания предоставляет примеры использования своих библиотек.
Чтобы продемонстрировать как работать с библиотекой *formatter_ex*,
вам необходимо создать два `CMakeList.txt` для двух простых приложений:
* *hello_world*, которое использует библиотеку *formatter_ex*;
* *solver*, приложение которое испольует статические библиотеки *formatter_ex* и *solver_lib*.

**Удачной стажировки!**
