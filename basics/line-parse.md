# Parsovanie reťazcov

Extrahovanie dát zo stringov je jedna z najčastejších programátorských úloh. Či už ide o dáta od užívateľa, alebo zo súborového systému. C a C++ ponúkajú rôzne nástroje na túto úlohu. 

## sscanf

Funkciu `scanf` z C máme dostupnú aj v C++. Na extrakciu dát z reťazca sa dá použiť jej varianta `sscanf`. Prvý parameter je smerník na string, druhý smerník na formátovací reťazec a nakoniec nasledujú premenné kam sa extrahované dáta uložia (s pravidla sú to smerníky, keďže ukazujú kam sa majú dáta uložiť). Napríklad ak chceme zo stringu dosťať dve čísla, môžeme použiť

```cpp
std::string s = "12 34";
int a = 0, b = 0;
sscanf(s.c_str(), "%d %d", &a, &b);

std::cout << a << " " << b << '\n';
```

Vo formátovacom reťazci zadávame, čo vlastne chceme. Najznámejšie formáty sú `%d` a `%s`, prvý znamená číslo a druhý reťazec. Je ich ale [oveľa viacej](https://en.cppreference.com/w/cpp/io/c/fscanf). Dá sa špecifikovať aj veľkosť (teda či je to `int`, `long`, `long long`...), pri stringoch sa dá definovať napríklad maximálna veľkosť `%128s`. 

> `%d` je formatovací špecifikátor, ktorá číta v podstate typ `int`. Existuje aj `%i`, ktorý robí viacmenej to isté, ale čislo nemusí byť v desiatkovej sústave. Ak bude na vstupe `0x10`, tak `%d` zoberie len `0`, ale `%i` prečíta číslo `16`;

Kód vyššie je viacmenej OK, lebo vieme aký string do funkcie dáme, ak ale toto nevieme zaručiť (je to vstup od užívateľa), tak máme problém. Ak by na vstupe bol `"12 ab"`, tak sa konverzia nepodari. Nato musíme skontrolovať návratovú hodnotu `sscanf`. Tá nám vráti počet hodnôt, ktoré sa podarilo sparsovať, teda v našom prípade by to bolo `0-2`, `0` ak sa nepodarí sparsovať nič (napr. string `"abc"`), `1` ak sa podari len prvé číslo (napr. string `"12 ab"`), `2` ak sa podari všetko.

### Rozdelenie stringu pomocou znaku

S trochou skúseností, môžeme zo `sscanf` urobiť skoro všetko. Povedzme, že chceme rozdeliť string podľa znaku `'.'`. V prvom rade musíme vedieť prečitať všetky znaky až po bodku do buffra. Ak by sme použili len `%s`, tak skončíme pri whitespace a nie pri bodke. 

```cpp
std::string s = "Split me. based on. full stop.";

char buffer[128];
if (sscanf(s.c_str(), "%127s", &buffer) == 1)
	std::cout << buffer << " " << '\n';
```

> Neriešime tu úplne veľkosť buffra, ktorú by sme potrebovali, preto sme zvolili 128 a do formátu dali 127, aby sme mali miesto na null terminator. 

Výsledok je string `Split`. So samotným `%s` si nevystačíme keďže ten vždy berie whitespace ako koniec stringu. Namiesto neho ale môžeme použiť syntax `%[set]`, kde `set` je množina znakov, ktoré sú string, všetky ostatné znaky sú delimiter. Dokonca to môžeme aj otočiť pomocou znaku `^`, teda `%[^.]` matchne všetko až po prvú bodku. 

```cpp
std::string s = "Split me. based on. full stop.";

char buffer[128];
if (sscanf(s.c_str(), "%127[^.]", &buffer) == 1)
	std::cout << buffer << " " << '\n';
```

Výsledok je `Split me`. Teraz musíme iba toto zopakovať aby sme vyextrahovali všetko. Keď sa zamyslíme zistíme, že je to trochu problém, lebo vo formátovacom reťazci nevieme nijak "urobiť cyklus" a ak dáme `sscanf` do cyklu, tak ideme vždy od začiatku. Našťastie existuje `%n`, ktorý nám vráti koľko znakov sme už skonzumovali až po `%n`, samotná operácia nič nekonzumuje ani neposúva v podstate ako by tam nebola. 

```cpp
std::string s = "Split me. based on. full stop.";

char buffer[128];
int start = 0;
int size = 0;
while (sscanf(s.c_str() + start, "%127[^.]%n", &buffer, &size) == 1) {
	std::cout << buffer << " " << '\n';

    start += size;
}
```

> Všimnime si, že síce máme vo formátovacom stringu dve premenné, ale testujeme výstup na `1` je to preto, lebo `%n` sa neráta do tohoto výsledku. 

Program znovu vypiše len `Split me`, je to preto lebo, v `size` nie je zahrnutá bodka a preto ďalšia iterácia začne na bodke a teda `%127[^.]` by mal matchnúť prázdny string, čo sa nepovažuje za úspech. Fixneme to tak, že vždy skúsime tú bodku matchnúť, formátovácie značky, ktoré začínajú `*` sa síce matchnú, ale nepriradia sa do žiadnej premennej.

```cpp
std::string s = "Split me. based on. full stop.";

char buffer[128];
int start = 0;
int size = 0;
while (sscanf(s.c_str() + start, "%127[^.]%*1[.]%n", &buffer, &size) == 1) {
	std::cout << buffer << " " << '\n';

	start += size;
}
```

Konečne funguje. 

> Pozornemu čitateľovi asi neuniklo, že samotný string je ukončený bodkou. Ak by nebol, tak nám to nefunguje a urobí to nedefinované správanie. Opraviť sa to dá tak, že sa vždy presvedčíme či tam bodka je a ak nie, tak doplníme, prípadne `%*1[.]` budeme naozaj priradovať do premennej a potom poďla návratovej hodnoty vieme ukončiť čítanie. 

### Problémy `sscanf`

   * Pri extrahoaní stringov nám môže pretiecť buffer, existuje na to funkcia `sscanf_s`, ale ta nie je súčasťou C++, ale iba C
   * Musíme zaručiť aby počet premenných bol rovnaký ako je počet formátovacích značiek vo formátovacom reťazci. Toto už mnohé kompilátory kontrolujú, ale samozrejme to nevedia vždy, keďžue formátovací reťazec môže byť premenná, nie iba priamo literál. 
   * Nesmieme zabudnúť skontrolovať návratovú hodnotu. 
   * Nie je úplne jednoduché dostať z funkciu ľubovoľne veľký string buffer, keďže sa len nedozvieme aký veľký buffer potrebujeme. 

## strtok

**`strtok` nie je thread safe, v C existuje funkcia `strtok_s`, ale tá nie je súčasťou C++**

`strtok` slúži na rozdelenie stringu podľa delimitera(ov). Funkcia používa priamo vstupný string, takže ten sa nám mení pod rukami. Napríklad rozdelenie stringu podľa bodiek. 

```cpp
char *token = strtok(s.data(), ".");
while (token) {
    std::cout << token << '\n';
    token = std::strtok(nullptr, ".");
}
```

Prvé volanie je zo stringom, ktorý chceme tokenizovať. Nemôže to byť `const char*`, ale `char*`. Ďalšie volania `strtok` potom robime s `nullptr`, tým hovoríme, že chceme pokračovať v tokenizáciu už rozbehnutého stringu. 

## std::stringstream

*Na použitie musíme includnuť `#include <sstream>`*

String stream je vlastne iostream, ktorý má pevne daný string nad ktorým pracuje. Môžeme teda využiť všetky formátovacie operácie, ktoré existujú napríklad nad `ifstream`, alebo `cout`. Napriklad chceme rozbiť string poďla whitespaceov. 

```cpp
std::vector<std::string> parts;

std::string s = "Too much  whitespaces   around	 here.   ";
std::stringstream ss(s);

std::string buf;
while (ss >> buf) {
    parts.push_back(buf);
}
```

V konštruktore stringstreamu dáme ako parameter reťazec, z ktorého sa stream vykonštruuje. V cykle potom čítame stringy, keďže operátor `>>` pre stringy preskakuje whitespace, tak nakoniec ostaneme len zo slovami. Dokonca nevadia ani whitespace na konci. Tie sa totiž ignorujú až sa príde na úplny koniec, to je v podstate `eof` pre string stream. Keďže sa nám ale žiaden znak nepodarilo načítať, tak string stream sa dostane do `fail` stavu a preto cyklus rovno skončí. 

Rozdelenie stringu podľa konkrétneho znaku môžeme vykonať pomocou funkcie `std::getline`, tá nám štandardne vracia string až po nový riadok, môžeme to ale zmeniť parametrom. 

```cpp
std::string s = "Split me. based on. full stop.";
std::stringstream ss(s);

std::string buf;
while (std::getline(ss, buf, '.')) {
    std::cout << buf << '\n';
}
```

`std::getline` môže ísť do podmienky, lebo vlastne vracia svoj prvý parameter, teda stream. Takýto kód si na rozdiel od príkladu s `sscanf` poradí aj so stringami, ktoré nekončia bodkou.

## std::regex

C++ podporuje aj regulárne výrazy, nich sa dajú použiť takzvané *capture groups*, ktoré umožnia zo stringu vysekať substringy podľa daných pravidieľ. O tom ale niekedy nabudúce. 

## Kód

Ak je štruktúra problému jednoduchá môžeme sa vykašlať na všetky nástroje zo štandardnej knižnice a urobíme si to sami. Tento krok veľmi neodporúčam, ale pri obzvlášť jednoduchých úlohách to môže byť ono. 

Napríklad chceme string rozdeliť podľa bodiek. 

```cpp
std::string s = "Split me. based on. full stop.";
size_t start = 0;
while (true) {
    size_t pos = s.find_first_of('.', start);
    if (pos == std::string::npos) {
        parts.push_back(s.substr(start));
        break;
    } else {
        parts.push_back(s.substr(start, pos - start + 1));
        start = pos + 1;
    }
}
```

Problém je, že aj takýto jednoduchý kód nás môže celkom pomýliť. Musíme si dávať pozor na indexy, +1, -1. Akonáhle by sme chceli niečo čo i len trochu komplikovanejšieho, tak kód sa stane veľmi nečitateľným. 