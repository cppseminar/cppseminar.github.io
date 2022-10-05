# Parsovanie reťazcov

Extrahovanie dát zo stringov je jedna z najčastejších programátorských úloh. Či už ide o dáta od užívateľa, alebo zo súborového systému. C a C++ ponúkajú rôzne nástroje na túto úlohu. 

## sscanf

TODO

## strtok

TODO

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

## std::regex

TODO

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