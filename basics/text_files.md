# Práca s textovými súbormi

Práca s textom je jednou z prvých úloh, ktoré musí programátor zvladnúť. Preto sa ju snažia jazyky uľahčiť, C++ nie je výnimka. 

## stdio.h

V rámci C++ existujú aj funkcie z hlavičky `#include <stdio.h>`. Tie poskytujú základnú funkcionalitu na otvorenie súboru, čítanie, zápis a uzavretie súboru. 

```c
#include <stdio.h>

int main()
{
    if (FILE* f = fopen("hello.txt", "w"))
    {
        fprintf(f, "Hello world!\n");

        fclose(f);
    }
}
```

Problémové je volanie `fclose`, ktoré nesmieme zabudnúť, inak nám v programe miznú zdroje. Rovnako `fprintf` má všetky neduhy `printf`, formátovacie značky vo formátovacom reťazci musia súhlasiť s typmi parametrov, inak máme nedefinované správanie a potenciálnu bezpečnostnú dieru. 

## fstream

Odpoveď C++ na tieto problémy sa nachádzajú v hlavičkovom súbore `#include <fstream>`. V ňom sa nachádzajú triedy pre prácu so súbormi. Program vyššie by bol trochu kratší. 

```cpp
#include <fstream>

int main()
{
    std::ofstream f("hello.txt");

    f << "Hello world!\n";
}
```

Trieda `std::ofstream` slúži na zápis so súboru, obrovskou výhodou je prítomnosť zatvorenia súboru v deštruktore. Nemusíme teda explicitne volať `close`. Táto metóda síce existuje a slúži na zavretie súboru. Jej volanie má zmysel iba kvôli optimalizácií, aby sme skôr uvoľnili zdroje. Volať preto `f.close();` pred vrátením sa funkcie je nosenie dreva do lesa. Ničomu to nepomôže a ešte to bude pomalšie, lebo kompilátor tak či tak deštruktor zavolá.

Operátor ľavý posun (`<<`) je preťažený a preto umožňuje formátovaný výstup do streamu, dáta sa zoberú, naformátujú sa na reťazec a ten sa zapíše do súboru. Vracia referenciu na stream a preto sa dá reťaziť. 

```cpp
f << "String with an integer " << 14 << " and float " << 5.6 << '\n';
```

## Chybové stavy

V oboch variantách doteraz nám chýbalo spracovanie chýb. Ani v jednom nie je nedefinované správanie, ale ani nedávajú najavo, že sa niečo pokazilo. `std::ofstream` má pomerne rozsiahle možnosti hlásenia chýb. 

* Pomocou metódy `is_open` sa dá zistiť, či sa vôbec podarilo súbor otvoriť. 
* Pomocou volania `rdstate`, tá vracia flagy, ktoré indikujú čo sa stalo. Nezvyknú sa používať priamo. 
  * `goodbit` vśetko je v poriadku
  * `failbit` nastal logický problém, napríklad s formátovaním, je to chyba, z ktorej sa dá zotaviť, ďaľšia operácia nad súborom môže skončíť dobre
  * `badbit` nastal problém so streamom samotným, chyba je fatálna a ďalšie oprerácie pravdepodobne zlyhajú
  * `eofbit` nie je úplne chyba, ale indikuje, že nastal koniec súboru (**e**nd **o**f **f**ile)
* Pomocou funckcií 
  * `good` vráti `true` ak ani jeden z `eofbit`, `failbit` a `badbit` nie je nastavený
  * `fail` vráti `true` ak je nastavený buď `failbit`, alebo `badbit`
  * `bad` vráti `true` ak je nastavený `badbit`
  * `eof` vráti `true` ak je nastavený `eofbit`
* Pomocou volania `operator bool`, ktorý vráti `true` ak nie je nastavený ani `failbit` ani `badbit`. Teda je ekvivalentný volaniu `!fail()`

Vyzerá to komplikovane ale v podstate sa dá spoliehať na operátor konverzie na `bool` a volanie `eof()` na zistenie konca súboru. Môže byť užitočná aj funkcia `is_open`, ale nie je úplne nutná, keďže neskôr nad streamom zlyhajú ostatné operácie. 

Ak chceme doplniť hlásenie chýb do predchádzajúceho programu. 

```cpp
#include <fstream>
#include <iostream>

int main()
{
    std::ofstream f("hello.txt");

    if (!(f << "Hello world!\n"))
    {
        std::cerr << "Error while writing file.\n";
        return 1;
    }
}
```

Keďže operátor `<<` vracia refereciu na stream, tak v podmienke sa vlastne zavolá operátor `bool`. Toto zachytí aj prípady, keď sa nedá súbor otvoriť. Ako sme už spomínali potom všetky výstupné operácie zlyhajú. Možno by bolo ale krajšie dať užívateľovi najavo kde presne nastala chyba. 

```cpp
#include <fstream>
#include <iostream>

int main()
{
    std::ofstream f("hello.txt");
    if (!f.is_open())
    {
        std::cerr << "Cannot open file.\n";
        return 1;
    }

    if (!(f << "Hello world!\n"))
    {
        std::cerr << "Error while writing file.\n";
        return 1;
    }
}
```

## Vstupné a výstupné súbory

Doteraz sme používali `std::ofstream` "o" v jeho názve znamená "output" teda výstup. Existuje samozrejme aj alternatíva pre vstup, ktorá sa neprekvapivo volá `std::ifstream`. Existuje aj `std::fstream`, ten potom musíme otvoriť buď na čítanie, alebo zápis. Nebudeme sa ním teraz zaoberať, je lepšie keď náš sémanticky zámer rovno zakódujeme do typu. Potom nám vie kompilátor lepšie pomôcť pri odhalovaní chýb. 

Načítanie jedného reťazca je veľmi jednoduché. 

```cpp
#include <fstream>
#include <iostream>
#include <string>

int main()
{
    std::ifstream f("hello.txt");
    if (!f.is_open())
    {
        std::cerr << "Cannot open file.\n";
        return 1;
    }

    std::string greet;
    if (!(f >> greet))
    {
        std::cerr << "Error while writing file.\n";
        return 1;
    }

    std::cout << greet << '\n';
}
```

Preťažený operátor `>>` slúži na extrakciu dát zo streamu a naplnenie premennej. Podľa typu premennej zvládne aj dáta správne naformátovať (hlavne čísla).

Rovnako ako pri výstupných súboroch aj tu môžeme volať tie isté funkcie na kontrolu chýb s rovnakou sémantikou ako pri výstupných súboroch. Program by bol stále dobre aj bez volania `is_open()`, keďže formátováný vstup `>>` tiež vracia samotný stream a preto v podmienke vlastne voláme operátor pre konverziu na `bool`. 

Keď to spustíme nad súborom, ktorý sme pred chvílou vyrobili, tak zistíme, že namiesto `Hello world!` dostaneme na konzolu iba `Hello`. Dôvod je ten, že pre formátovací vstup `>>` je medzera oddeľovač reťazcov (všetky whitespace sú). 

## Čítanie po riadkoch

Na čítanie po riadkoch môžeme využiť funkciu `std::getline`, parametre sú nasledujúce. 
* Vstupný stream.
* Premenná typu `std::string`, do ktorej sa načíta riadok.
* Delimiter, ktorý určuje čo je koniec riadku (predvolene je to `'\n'``, ale dá sa použiť akýkoľvek znak).

Prečítať celý súbor po riadkoch a vypísať ich na konzolu sa dá urobiť takto. 

```cpp
#include <fstream>
#include <iostream>
#include <string>

int main()
{
    std::ifstream f("hello.txt");

    std::string line;
    while (std::getline(f, line))
    {
        if (!(std::cout << line))
        {
            std::cerr << "Cannot write to console.\n";
            return 1;
        }
    }
}
```

Takýto program má dva problémy. Po prvé kontrolovať či sa niečo podarilo zapísať na konzolu je trochu redundantné, to sa takmer vždy musí podariť. Ale keď to chceme mať nepriestrelné, tak je to potrebné. Druhý problém je trochu väčší, nikde nekontrolujeme, či sa podarilo prečítať celý súbor. To si musíme urobiť sami pomocou volanie `eof()`. 

```cpp
#include <fstream>
#include <iostream>
#include <string>

int main()
{
    std::ifstream f("hello.txt");

    std::string line;
    while (std::getline(f, line))
    {
        std::cout << line;
    }

    if (!f.eof())
    {
        std::cerr << "Cannot read whole input file.\n";
        return 1;
    }
}
```

Teraz ak sa náhodou nepodarí prečítať celý súbor, tak sa o tom dozvieme. 