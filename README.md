# Náplň cvičenia
- zoznámenie sa s registrami mikrokontrolera stm32-F3xx
- priami prístup k registrom mikrokontrolera
- zoznámenie sa s perifériou GPIO – ako funguje, vlastnosti
- konfigurácia GPIO portu a jednotlivých pinov

# GPIO port
<p align="center">
    <img src="https://bluetechs.files.wordpress.com/2013/12/115.png" width="600" title="GPIO pin block scheme">
</p>

### Nastavenie vstupného pinu
- typ vstupu: digitálny, analógový
- alternatívna funkcia
- pripojenie pull up, pull down rezistora

### Nastavenie výstupného pinu
- typ výstupu: push-pull, open-drain
- rýchlosť výstupu
- pripojenie pull up, pull down rezistora

### Registre GPIO
- Konfigurácia pinu: MODER, OTYPER, OSPEEDR, PUPDR, AFRL/H
- Čítanie vstupného pinu: IDR
- Čítanie/zápis na výstupný pin: ODR, BSRR, BRR

# Prístup k registrom

K registrom mikrokontrolera je možné pristupovať priamo pomocou adresy samotného registra.
```sh
*((volatile uint32_t *)((uint32_t)0x48000400)) = 0;
```
Hore uvedené je príklad ako pristúpiť (čítanie/zápis) k registru mikrokontrolera cez jeho adresu.
"0x48000400" je adresa v rámci adresovaného pamäťového priestoru mikrokontrolera. Adresa je 32 bitové číslo. Pre možnosť prístupu k hodnote nachádzajúcej sa na tejto adrese, je potrebné definovať adresu registra ako smerník na premennú typu uint32_t (32 bitový bezznamienkový integer).

### Maskovanie
V rámci registra majú jednotlivé bity alebo skupiny bitov svoju špeciálnu funkciu ako napr. nastavenie režimu GPIO pinu. To znamená, že ak chcem nastaviť konkrétne piny na hodnotu "0" alebo "1", nemôžem zvoliť spôsob zápisu do registra ako je to znázornené v hore uvedenom príklade, pretože všetky bity budú nastavené na hodnotu "0" (celý register je resetovaný).

Pre nastavenie konkrétnych bitov v registry sa využije "maska". Kedže sa ide nastavovať 32 bitový register, maska bude taktiež 32 bitové číslo.
- Nastavenie bitov na "1": maska sa vytvorí tak, že bity, ktoré chceme nastaviť v rámci registra budú mať hodnotu "1" a ostatné budú mať hodnotu "0". Následne sa vykoná operácia logický OR a výsledok operácie sa zapíše do registra. Takýmto spôsobom boli nastavené len tie bity, ktoré sme potrebovali nastaviť bez toho, aby sa zmenilo nastavenie ostatných bitov registra.

<pre>  0b10000000    <- register		
| 0b00100100    <- maska   
= 0b10100100    <- nové nastavenie registra </pre>

- Nastavenie bitov na "0": maska sa vytvorí tak, že bity, ktoré chceme resetovať v rámci registra budú mať hodnotu "0" a ostatné budú mať hodnotu "1". Následne sa vykoná operácia logický AND a výsledok operácie sa zapíše do registra. Takýmto spôsobom boli resetované len tie bity, ktoré sme potrebovali resetovať bez toho, aby sa zmenilo nastavenie ostatných bitov registra.

<pre>  0b11100101    <- register		
& 0b10011111    <- maska   
= 0b10000101    <- nové nastavenie registra </pre>

- Čítanie hodnoty z registra: ak chceme získať konkrétnu hodnotu bitu alebo skupiny bitov registra, taktiež použijeme masku. Maska bude mať nastavené bity na "1" na pozícii, z ktorej chcem čítať hodnotu z registra. Následne sa vykoná operácia "&" a jej výsledok sa porovná s maskou, aby sa zistilo, či sú dané bity registra nastavené na "1".

<pre>  0b10101101    <- register		
& 0b00011000    <- maska   
= 0b00001000    <- prečítaná hodnota z registra </pre>

# Zadanie
- modifikovať ukážkový program tak, aby sa ako vstup, na ktorom je pripojené tlačidlo, používal iný port a pin -> GPIOA, pin 3
- vytvoriť funkciu, detektor hrany, ktorá bude mat 2 vstupy (stav vstupného pinu, počet vzoriek) a výstup bude nadobúdať 3 hodnoty – "0" pre detekciu nábežnej hrany, "1" pre detekciu dobežnej hrany, "2" pre kľudový stav
