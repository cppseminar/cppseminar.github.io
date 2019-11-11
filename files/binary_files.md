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