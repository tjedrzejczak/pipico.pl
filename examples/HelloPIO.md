---
layout: page
title: "HelloPIO"
permalink: /examples/HelloPIO
topimage: top-keyb1.jpg
---

PIO czyli (od ang. Programmable I/O) to wysoce wyspecjalizowany układ do precyzyjnego sterownia portami GPIO. Sterowanie nimi odbywa się za pomocą rozkazów w specjalizowanym języku pioasm. Każda z instrukcji pioasm wykonuje się w dokładnie jednym takcie zegara, co pozwala na generowanie przebiegów o ściśle określonych przebiegach (bit-banging).

Przykładowy program [HelloPIO](https://github.com/tjedrzejczak/pipico.pl.examples/tree/main/HelloPIO) to trochę bardziej złożony przykład, niż zazwyczaj spotykane programy "Hello …".

Zadaniem programu jest wyświetlanie za pomocą wbudowanej w Pi Pico diody LED (PIN 25) bit po bicie danych przekazanych w postaci słowa 32 bitowego. Każdy bit wyświetlany jest przez 50 ms, czyli przez 1/20 sekundy. Takie proporcje czasowe uzyskano przez złożenie dwóch faktów. Cały program w PioAsm zajmuje dokładnie 100 taktów zegara SM, a preskaler został dobrany tak, aby SM pracowała z częstotliwością 2kHz.


Na początek kod w PioAsm, zawarty w pliku hello.pio:
```text
.program hello

.wrap_target
    out pins, 1 [19]    ; 20 cycles
    nop         [19]    ; 20 cycles
    nop         [19]    ; 20 cycles
    nop         [19]    ; 20 cycles
    nop         [19]    ; 20 cycles
.wrap
```

Program składa się z dwóch rozkazów.
Rozkaz NOP to operacja nie przynosząca żadnych skutków, poza wykorzystaniem jednego cyklu zegara. De facto jest to pseudo-rozkaz ponieważ jest on tłumaczony na MOV Y,Y. W&nbsp;powyższym kodzie użyto rozszerzonej składni:
```text
    nop         [19]
```
Wartość [19] oznacza, że po wykonaniu rozkazu, które zajmuje jeden cykl zegara, zaczekaj jeszcze 19 cykli. W&nbsp;rezultacie cały rozkaz wykonuje się przez 20 cykli zegara.

Rozkaz:
```text
    out pins, 1 [19] 
```
pobiera określoną liczbę bitów (tu 1) z OSR (Output Shift Register) i wpisuje je we wskazane miejsce. W tym wypadku do PINS czyli jako wartość określonego pinu/pinów GPIO. Po tym rozkazie także mamy 19 cykli opóźnienia.

Cały przebieg pobierający jeden bit z OSR i przekazujący go do GPIO  zajmuje zatem w tym przypadku dokładnie 100 cykli zegara SM.

W pliku hello.pio znalazła się także funkcja hello_program_init.
```c
void hello_program_init(PIO pio, uint sm, uint offset, uint pin, float freq)
{
    pio_sm_config conf = hello_program_get_default_config(offset);

    // set pin
    pio_gpio_init(pio, pin);
    sm_config_set_out_pins(&conf, pin, 1);

    // pin direction is output
    pio_sm_set_consecutive_pindirs(pio, sm, pin, 1, true);

    // set fifo
    sm_config_set_out_shift(&conf, true, true, 32);
    sm_config_set_fifo_join(&conf, PIO_FIFO_JOIN_TX);

    // set frequency
    float div = (float)clock_get_hz(clk_sys) / freq;
    sm_config_set_clkdiv(&conf, div);

    // load & start
    pio_sm_init(pio, sm, offset, &conf);
    pio_sm_set_enabled(pio, sm, true);
}
```

Jej zadaniem jest właściwa konfiguracja SM dla danego programu PioAsm.
Funkcja hello_program_get_default_config jest generowana automatycznie do pliku hello.pio.h.

Wywołanie funkcji hello_program_init znajduje się bezpośrednio w funkcji main.

```c
int main()
{
    const uint led1_pin = 25;
    const float freq = 2000; // Hz

    stdio_init_all();

    PIO pio = pio0;
    uint sm = pio_claim_unused_sm(pio, true);
    uint offset = pio_add_program(pio, &hello_program);

    hello_program_init(pio, sm, offset, led1_pin, freq);

    while (true)
    {
        uint32_t data = 0x55555555; // - 0101
        pio_sm_put_blocking(pio, sm, data);
        pio_sm_put_blocking(pio, sm, data);

        data = 0x33333333; // - 0011
        pio_sm_put_blocking(pio, sm, data);
        pio_sm_put_blocking(pio, sm, data);

        data = 0xffff00f0;
        pio_sm_put_blocking(pio, sm, data);
        pio_sm_put_blocking(pio, sm, data);
    }
}
```

Część inicjująca kończy się na wywołaniu hello_program_init. Dalej, w&nbsp;pętli while, odbywa się właściwe przekazywanie danych do FIFO danej maszyny stanowej. Użyto funkcji [pio_sm_put_blocking](https://www.raspberrypi.com/documentation/pico-sdk/hardware.html#group_hardware_pio_1gaee8bfc3409cb8d93cccdeda3961bc377) w wersji blokującej, czyli zatrzymującej działanie programu gdy w FIFO nie ma jeszcze miejsca na nowe dane.