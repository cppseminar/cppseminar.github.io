# Práca s binárnymi súbormi

Doteraz sme využívali hlavne textové súbory. Ich výhodou je, že za nás riešia niektoré platformovo špecifické vlastnosti. Ako napríklad konce riadkov. Tie sú na Windowse `CR LF` a na Linuxe a Macu iba `LF`. Binárne súbory nám umožňujú pristúpiť priamo k bajtom uloženým v súbore. 

## `std::ios::binary`

Ak pri otváraní súboru špecifikujeme binárny mód, potom všetky dodatočné kontroly sú vypnuté. Teda mali byť sme maž zaručené, že to čo zapíšeme je potom aj to čo prečítame (pre textový režim môžu byť iné pravidlá, POSIX sa ale správa k binárnym aj textovým súborom úplne rovnako). Operátory `>>` a `<<` pracujú rovnako ako v textovom režime, teda hodnoty formátujú do textu. 

```cpp
std::ofstream ofs("output.bin", std::ios::binary);
ofs << 1237;
```

Obsah súboru bude `"1237"`. 

## Čítanie bajtov

Často sa nám stane, že potrebujeme prečítať zo súboru pár bajtov. To sa najlahšie robí pomocou funkcie `read`. 

```cpp
std::ifstream ifs("input.bin", std::ios::binary);

char buffer[128];
ifs.read(buffer, std::size(buffer));

for (size_t i = 0; i < ifs.gcount(); ++i)
{
    std::cout << "0x" << std::hex << std:: << static_cast<unsigned char>(buffer[i]) << ' ';
}
```

Výstup bude `0x1 0x2 0x3 0x7`, čo nie je čo sme chceli. Problém je, že `<<` pre streamy je preťažený pre `unsigned char` a jeho výstup je priamo znak, nie jeho číselná hodnota. Toto môžeme opraviť, explicitným pretypovaním na číselný typ. 

```cpp
std::cout << "0x" << std::hex << std::setw(2) << static_cast<unsigned>(buffer[i]) << ' ';
```

Teraz už je výstup `0x31 0x32 0x33 0x37`. 

## Zápis binárnych dát

Videli sme, že operátor `<<` zapisuje čísla ako text. Čo ak chceme ale zapísať čísla ako ich reprezentáciu v pamäti, teda binárne? Na to musíme použiť metódu `write`. 

```cpp
int main()
{
    std::ofstream ofs("output.bin", std::ios::binary);

    int64_t data = -24;
    if (!ofs.write(reinterpret_cast<const char*>(&data), sizeof(data)))
        return EXIT_FAILURE;
}
```

V závislosti od platformy, potom v súbore môže byť 8 bajtov s hodnotou `E8 FF FF FF FF FF FF FF` ak sa používa dvojkový doplnok na reprezentáciu hodnôt (skoro všetky moderné platformy) a [little endian](https://en.wikipedia.org/wiki/Endianness) na usporiadanie bajtov. Čítanie sa potom udeje analogicky pomocou `read`. Jedený problém je, že garantovanie rovnakú hodnotu dostaneme iba na rovnakej platforme ako sme robili zápis (dnes to teoreticky nie je až taký problém, ale treba na to občas myslieť). 

```cpp
int main()
{
    std::ifstream ifs("output.bin", std::ios::binary);

    int64_t data = 0;
    if (!ifs.read(reinterpret_cast<char*>(&data), sizeof(data)))
        return EXIT_FAILURE;

    if (ifs.gcount() != sizeof(data))
        return EXIT_FAILURE;

    std::cout << data << std::endl; // -24
}
```