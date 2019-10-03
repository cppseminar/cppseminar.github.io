# Hello world!

Každý správny začiatok programovania ukazuje spôsobakým vypísať pozdrav na konzolu. Cielom je hlavne uistiť sa, že prostredie funguje ako má a vieme spustiť primitívny program. Táto tradícia siaha k začiatkom jazyka C a jeho biblie [The C Programming Language](https://en.wikipedia.org/wiki/The_C_Programming_Language). 

```cpp
#include <iostream>

int main()
{
    std::cout << "Hello world!\n";
}
```

V starom dobrom C by ekvivalentný program vyzeral asi takto. 

```c
#include <stdio.h>

int main()
{
    printf("Hello world!\n");
}
```

Samozrejme spätná kompatibilita s C je jednou z dôležitých vlastností C++. Preto aj tento program sa skompiluje a bude fungovať aj v najnovšom C++17. My budeme preferovať prvý spôsob.  `std::cout` je _output stream_ a teda používa moderný interface, ktorý je bezpečnejší a ľahšie sa používa na formátovanie (nemusíme poznať formátovacie reťazce ako `%d` a `%s`). 