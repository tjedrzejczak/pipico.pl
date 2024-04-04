---
layout: page
title: "Przykłady użycia biblioteki PicoLib"
permalink: /picolib
topimage: top-keyb1.jpg
---

#### Biblioteka i przykłady

Bibliteka PicoLib znajduje się w repozytorium [PicoLib](https://github.com/tjedrzejczak/PicoLib)
Przykłady jej użycia znajdują się w innym repozytorium [pipico.pl.examples](https://github.com/tjedrzejczak/pipico.pl.examples) i odwołują się do biblioteki za pomocą submodułu git.

Wszystkie projekty przykładowe używają wspólnego folderu biblioteki, dzięki czemu możliwe jest pobranie z repozytorium kodu wszystkich przykładów:

```console
git clone https://github.com/tjedrzejczak/pipico.pl.examples
cd pipico.pl.examples
git submodule init
git submodule update
```

#### Kompilowanie przykładów

Kompilacji poszczególnych przykładów należy dokonywać z ich folderów.

```console
cd HelloWorld
```

Przed pierwszą kompilacją należy w folderze, w którym znajduje się `main.c` lub `main.cpp`, wykonać:

```console
mkdir build && cd build
cmake ..
make -j$(nproc)
```

Następne kompilacje wymagają ponownego wykonania w folderze `build` polecenia:

```console
make -j$(nproc)
```

#### Przykłady

[HelloWorld](../examples/HelloWorld) - najprostszy program

[HelloPIO](../examples/HelloPIO) - prosta demonstracja pioasm
