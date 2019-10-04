# Funkcia `main` a parametre

## Najkratší C++ program

Programy v C++ začínajú vždy vo funkcií `main()`, ktorá musí byť v globálnom namespace-i a nemôže byť preťaŽená (teda musí existovať práve jedna). Najkratší platný C++ program vyzerá takto. 

```cpp
int main() 
{
}
```

`main` je jedná funkcia, ktorá síce má navrátovú hodnotu (typu `int`), ale nemusí obsahuvať príkaz `return`. Štandard totiž garantuje, že v takomto prípade bude návratová hodnota `0`. 

> If control flows off the end of the *compound-statement* of `main`, the effect is equivalent to a `return` with operand 0.

Na väčšine platforiem je táto návratová hodnota vlastne exit kód procesu. Hodnota `0` znamená úspešný beh programu. Hodnota iná ako `0` značí chybu. C/C++ ešte v knižnici `#include <stdlib.h>` definuje navratové kódy `EXIT_SUCCESS` a `EXIT_FAILURE`, tie sú ale zvyčajne definované práve ako `0` respektíve `1` pre chybu. Nie je to nutné, ale je od aplikácie zdvorilé, aby naozaj pri ukončení chybou vrátila hodnotu inú ako `0`. 

## Parametre programu

Samotné parametre `main`-u nie sú v štandarde definované. Každá implementácia by mala vždy poskytovať aspoň. 

```cpp
int main(); // 1

int main(int arg, char** argv); // 2
```

Druhá varianta umožnuje aplikácií predať parametre z vonku. Zvyčajne ako parametre na prikázovom riadku. `argc` je počet parametrov a `argv` sú samotné parametre. Prvý je zvyčajne meno alebo cesta k spustiteľnému súboru, ktorý obsahuje náš program. Jednoduchý program, ktorý vypíše všetky svoje parametre by mohol vyzerať takto. 

```cpp
#include <iostream>

int main(int argc, char** argv)
{
    for (int i = 0; i < argc; ++i)
    {
        std::cout << argv[i] << '\n';
    }
}
```

Ak ho skompilujeme a nazveme napriklad `app.exe` potom po spustení príkazu 

```
C:\Tools\app.exe first 2 "third parameter"
```

bude výpis na štandardný vstup 

```
C:\Tools\app.exe
first
2
third parameter
```

Parametre sa do `main`-u dostanú ako reťazce. Medzera na príkazovom priadku je oddelovač, preto ak chceme parameter s medzerou musíme ho zabaliť do úvodzoviek. Znak `"` sa preto musi escapovať ak má byť súčasťou parametra. 

Mená `argc` a `argv` nie sú vyžadované, mená týchto parametrov môžu byť ľubovolné. Tieto dve sú také zaužívané, že nie je veľmi odporúčané ich meniť. 

Štandard ďalej ganrantuje, že `argv[argc]` je vždy `0`, teda nulový smerník. Preto by program vyššie mohol vyzerať aj takto. 

```cpp
#include <iostream>

int main(int argc, char** argv)
{
    for (int i = 0; argv[i] != 0; ++i)
    {
        std::cout << argv[i] << '\n';
    }
}
```

## Premenné prostredia

Všetky platformy, ktoré niečo znamenajú, ešte podporujú jednu signatúru `main` funkcie. 

```cpp
int main(int argc, char** argv, char** env);
```

Do tretieho parametra sa dostanú premenné prostredia *(angl. environment variables)*. Keďže ich počet nikde nie je, musíme sa spoľahnúť na fakt, že sú znovu ukončené nulovým smerníkom. Program 

```cpp
#include <iostream>

int main(int argc, char** argv, char** env)
{
    for (int i = 0; env[i] != 0; ++i)
    {
        std::cout << env[i] << '\n';
    }
}
```

potom vypiše niečo ako 

```
ALLUSERSPROFILE=C:\ProgramData
APPDATA=C:\Users\koscelansky\AppData\Roaming
CommonProgramFiles=C:\Program Files (x86)\Common Files
CommonProgramFiles(x86)=C:\Program Files (x86)\Common Files
...
```