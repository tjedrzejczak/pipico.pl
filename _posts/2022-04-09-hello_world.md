---
layout: post
title: "Hello, World! w języku C"
date: 2022-04-09
permalink: /2022/04/hello
---

#### Instalacja zestawu narzędzi z wykorzystaniem WSL

Instalacja środowiska niezbędnego do tworzenia oprogramowania dla Raspberry Pi Pico w języku C w środowisku Windows jest możliwe, ale problematyczne. Proponuję inne rozwiązanie: zainstalować w systemie Windows podsystem WSL, a w nim zainstalować narzędzia niezbędne do programowania w języku C i C++ dla Raspberry Pi Pico.
O tym jak zainstalować WSL [pisałem już wcześniej](https://blog.ypro.tech/2021/04/wsl2-1) (środowiska graficznego nie trzeba instalować). Aby działać swobodniei bez obaw o interferencje z innym oprogramowaniem, proponuję działać w specjalnie do tego celu przeznaczonej [piaskownicy](https://blog.ypro.tech/2022/03/WSL2-piaskownice).

Po uruchomieniu instancji Linuxa przeznaczonej dla zestawu narzędzi do tworzenia programów dla Pico należy dokonać instalacji niezbędnych programów.
Zaczynamy od aktualizacji środowiska:
```console
sudo apt update
sudo apt upgrade
```
a następnie cały zestaw niezbędnych składników:
```console
sudo apt install cmake gcc-arm-none-eabi libnewlib-arm-none-eabi build-essential  libstdc++-arm-none-eabi-newlib
```

Należy pobrać także zestaw SDK. Proponuję umieścić go w folderze ``/opt/pico-sdk``:
```console
sudo git clone https://github.com/raspberrypi/pico-sdk.git /opt/pico-sdk
cd /opt/pico-sdk
sudo git submodule update --init
echo 'export PICO_SDK_PATH=/opt/pico-sdk' | sudo tee -a /etc/profile.d/pico-sdk.sh
```
Aby zmiana w profilu zadziałała należy wylogować się i zalogować się ponownie.

#### Pierwszy program

Proponuję kod programów umieszczać w strukturze folderów widocznej zarówno z systemu Windows jak też z Linuxa uruchomionego pod WSL. Jeśli takim folderem będzie np.: ``D:\Projects\`` to będzie on widoczny także jako ``/mnt/d/Projects/``.

W wybranym folderze należy umieścić dwa pliki:

``main.c``:
```c
#include <stdio.h>
#include "pico/stdlib.h"

void blinkLed(uint pin, uint32_t ms) {
    printf("Blinking led on pin %d - %d ms\r\n", pin, ms);

    gpio_put(pin, true);
    sleep_ms(ms);
    gpio_put(pin, false);
}

int main() {
    const uint led1_pin = 25; // onboard led
    const uint led2_pin = 22; // external led

    gpio_init(led1_pin);
    gpio_set_dir(led1_pin, GPIO_OUT);
    gpio_init(led2_pin);
    gpio_set_dir(led2_pin, GPIO_OUT);

    stdio_init_all();

    while (true) {
        printf("new loop\r\n");

        blinkLed(led1_pin, 500);

        for(uint i = 0; i < 3; i++) {
            blinkLed(led2_pin, 300);
            sleep_ms(200);
        }

        blinkLed(led1_pin, 500);
        blinkLed(led2_pin, 500);
    }
}
```

oraz ``CMakeLists.txt``:
```c
cmake_minimum_required(VERSION 3.13)

include($ENV{PICO_SDK_PATH}/external/pico_sdk_import.cmake)

project(HelloWorld C CXX ASM)

set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD 17)

pico_sdk_init()

add_executable(${PROJECT_NAME} main.c)

pico_add_extra_outputs(${PROJECT_NAME})

target_link_libraries(${PROJECT_NAME} pico_stdlib)

pico_enable_stdio_usb(${PROJECT_NAME} 1)
pico_enable_stdio_uart(${PROJECT_NAME} 0)
```

Kompilacja odbywa się w folderze ``build``. Przed pierwszą kompilacją należy będąc w folderze, w którym znajduje się ``main.c``, wykonać:
```console
mkdir build && cd build 
cmake ..
make -j$(nproc)
```
Następne kompilacje wymagają ponownego wykonania w folderze ``build`` polecenia:
```console
make -j$(nproc)
```

Nazwa projektu znajduje się w pliku ``CmakeLists.txt`` w deklaracji ``project()`` np.:
```c
project(HelloWorld C CXX ASM)
```
Zatem wynikiem kompilacji będzie m.in. plik ``HelloWorld.uf2``. Plik ten należy umieścić w pamięci Pico. Aby to było możliwe, należy połączyć Pico z komputerem przytrzymując wciśnięty przycisk BOOTSEL w chwili włączania zasilania Pico. Po tej czynności Pico będzie widoczne dla komputera jako pamięć masowa, do której należy wgrać wynikowy plik ``.uf2``. Wgranie pliku spowoduje restart Pico i uruchomienie wgranego programu.