---
layout: post
title: "Kompilacja OpenOCD w środowisku Windows i MSYS2"
date: 2024-03-18
permalink: /2024/03/openocd
topimage: top-code1.jpg
---

#### MinGW
MinGW (w tym [mingw-w64](https://www.mingw-w64.org)) (ang. Minimalist GNU for Windows) to kompilator GCC języka C/C++ i zestaw narzędzi towarzyszących przeznaczony dla Windows.

#### MSYS2
Z kolei [MSYS2](https://www.msys2.org) to środowisko dostarczające różnych narzędzi dla systemu Windows, głównie związanych z tworzeniem aplikacji, obsługiwanych w powłoce typu Unixowego. Wśród nich pacman, git czy wspomniany mingw-w64.

Instalację MSYS2 przeprowadza się za pomocą instalatora. Link do aktualnej wersji instalatora znajduje się na głównej stronie [MSYS2](https://www.msys2.org).

![installer](/img/p202403/MSYS201.png){: .mx-auto .d-block }


Po zainstalowaniu MSYS2 należy uruchomić konsolę MSYS2 MINGW64 i wykonać...
```console
pacman -S mingw-w64-x86_64-toolchain git make libtool pkg-config autoconf automake texinfo mingw-w64-x86_64-libusb
```
...instalując wszystko co jest domyślnie sugerowane.


#### Kompilacja OpenOCD dla Pico
Ponieważ od pewnego czasu [picoprobe](https://github.com/raspberrypi/debugprobe) wspiera CMSIS_DAP v2, a [OpenOCD](https://openocd.org) zawiera obsługę RP2040 w głównej gałęzi repozytorium, należy pobrać aktualną wersję OpenOCD do wybranego folderu...

```console
git clone https://github.com/openocd-org/openocd.git
```
… a następnie ją skompilować:
```console
cd openocd
./bootstrap
./configure --enable-cmsis-dap.v2
make
```

Po zakończeniu procesu (wynikiem kompilacji jest plik openocd.exe) należy w jednym folderze zgromadzić:
- plik openocd.exe (z folderu openocd/src)
- folder tcl (z openocd)
- biblioteka libusb-1.0.dll (z folderu mingw64\bin\ w miejscu instalacji MSYS2)

Tak przygotowany OpenOCD można wykorzystać do [debugowania](../../2023/04/wsl-dbg) z użyciem sondy probe:

```console
openocd.exe -f interface/cmsis-dap.cfg -c "adapter speed 5000" -f target/rp2040.cfg -s tcl -c "bindto 0.0.0.0"
```