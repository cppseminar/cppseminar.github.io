# Vector

Vector je kontainer, ktorý je súčasťou štandardnej knižnice C++ a poskytuje intedrface dynamického pola. To znamená, že `std::vector` nám umožňuje ukladať a spravovať dáta vo forme poľa, pričom veľkosť tohto poľa môžete meniť počas behu programu. Vector patrí k základným a najpoužívanejším dátový štruktúram vôbec. 

`std::vector` sídli v hlavičkovom súbore `#include <vector>`.

*Funkcie, ktoré sa ďalej budú spomínať majú veľa overloadov, všetky neuvádzame, dajú sa ľahko nájsť v dokumentácií.*

Niektoré dôležité vlastnosti a operácie, ktoré sú spojené s `std::vector`:

1. **Dynamická veľkosť**: Hlavnou výhodou `std::vector` je schopnosť meniť svoju veľkosť počas behu programu. Môžeme pridávať a odoberať prvky podľa potreby, čo je veľmi užitočné, ak nevieme presne, koľko prvkov budeme potrebovať vopred, ako to potrebuje pri statických poliach (e.g. `int[]`).

2. **Rýchly prístup** Prístup k prvkom `std::vector` je rýchly, pretože vnútorné dáta sú usporiadané v pamäti ako kontinuálny blok, čo umožňuje použiť smerníkovú aritmetiku aby sme sa rýchlo dostali k prvku, ktorý nás zaujíma. Pre prístup k prvku môžeme použiť operátor `[]` rovnako ako pri bežných poľových dátových štruktúrach. 

3. **Automatická správa pamäte**: `std::vector` spravuje svoju pamäť automaticky. To znamená, že sa stará o alokáciu a dealokáciu pamäte, čo nás zbavuje starostí s ručnou správou pamäte. 

4. **Iterátory**: Môžeme použiť interface iterátorov na prechádzanie a manipuláciu s prvkami v `std::vector`. 

## Vytváranie

### Vytvorenie prázdneho vektora

Ak chcete vytvoriť prázdny vektor a neskôr doň pridávať prvky, jednoducho vytvorte vektor bez uvedenia počiatočných hodnôt.

```cpp
std::vector<int> numbers;
```

### Inicializácia vektora s určenou veľkostou

Môžeme inicializovať vektor s určitým počtom prvkov, ktoré majú počiatočnú hodnotu. Nato použijeme konštruktor s dvoma argumentami - počtom prvkov a ich počiatočnou hodnotou. *Pozor toto nie je iba predalokovanie miesta.*

```cpp
std::vector<int> numbers(5, 1); // Vektor s 5 jednotkami
```

### Inicializácia vektora z existujúceho poľa pomocou iterátorov

Môžeme inicializovať vektor na základe existujúceho poľa alebo iného vektora. Na to môžete použiť konštruktor `std::vector` s dvoma iterátormi ukazujúcimi na začiatok a koniec rozsahu, ktorý chceme skopírovať.

```cpp
int array[] = {1, 2, 3, 4, 5};
std::vector<int> numbers(array + 1, array + 5); // 2, 3, 4, 5

std::vector<int> numbers2(numbers.begin(), numbers.end() - 1); // 2, 3, 4
```

### Inicializácia vektora pomocou zoznamu inicializácie

Od C++11 môžeme vytvoriť a inicializovať vektor pomocou zoznamu inicializácie *(angl. initializer list)*.

```cpp
std::vector<int> numbers = {1, 2, 3, 4, 5};
```

## Prechádzanie cez vektor

Na iteráciu vektora používame funkcie `begin()` a `end()`. Tieto funkcie poskytujú iterátory na začiatok a koniec kontajnera, čo umožňuje prechádzať jeho prvky pomocou klasických `for`-cyklusov. Alebo môžeme využiť automatické rozsahé for-cyklusov (*range-based for loops*). Oba spôsoby sú OK, akurát druhý je modernejší. 

### Pomocou `begin()` a `end()` a klasického `for`-cyklu

```cpp
std::vector<int> numbers = {1, 2, 3, 4, 5};

for (std::vector<int>::iterator it = numbers.begin(); it != numbers.end(); ++it) {
    std::cout << *it << " "; // výpis: 1 2 3 4 5
}
```

### Pomocou rozsahového for-cyklu

Rozsahový for-cyklus umožňuje jednoduchšiu iteráciu vektora bez priameho použitia iterátorov. Aj keď pod kapotou sa používajú iterátory a teda oba spôsoby iterácie sú viacmenej ekvivalentné. 

```cpp
std::vector<int> numbers = {1, 2, 3, 4, 5};

for (int num : numbers) {
    std::cout << num << " "; // výpis: 1 2 3 4 5
}
```

## Pridávanie prvkov

### Na koniec

Pridávanie na koniec `std::vector` sa robí pomocou funkcie `push_back`. Operácia je veľmi rýchla O(1) v priemernom prípadne. Ak sa však veľkosť vektora približuje svojmu maximálnemu kapacitnému limitu, vektor môže vyžadovať alokáciu nového bloku pamäte a kopírovanie existujúcich prvkov do nového bloku. Toto môže mať lineárnu časovú zložitosť O(n), kde n je aktuálna veľkosť vektora.

Vektor si teda zvyčajne drží väčší blok pamäte ako reálne potrebuje. Preto väčšinou má miesto na ešte jeden prvok. Ak ale zrovna nemá, tak je pridanie drahé. Stratégie ako to vektor robí sa líša od vendora štandardnej knižnice. Zvyčajne sa veľkosť vektora zdvojnásobuje. 

Povedzme, že máme vektor s 8 prvkami. Pridanie ďalšieho na koniec. Urobí realokáciu a 8 rvkov sa musí posununúť. Lenže teraz má vektor dvojnásobnú kapacitu, teda 16 prvkov, preto ďalších 7 pridaní na koniec nebude vyžadovať realokáciu. 

### Na začiatok a inde do vektora

Funkcia `insert` má dva parametre. Prvý je pozícia, kam chceme vložiť, druhý je čo chceme vložiť. 

```cpp
std::vector<int> numbers = {1, 2, 3, 4, 5};
numbers.insert(numbers.begin() + 2, 99); // 1, 2, 99, 3, 4, 5
```

Ako vidíme, pozícia je určená iterátorom a nie indexom. To ale nie je problém, lebo stačí urobiť `vec.begin() + idx` a konvertujeme index `idx` na iterátor do vektora `vec`. 

## Veľkosť a kapacita

V kontexte štandardného vektora v jazyku C++ je dôležité rozlišovať medzi veľkosťou a kapacitou vektora. Obe tieto vlastnosti sú dôležité pre správu dát vektora. Ich poznaním sa vieme vyhnúť nedefinovanému správaniu. 

### Veľkosť (size)

Veľkosť vektora označuje počet prvkov, ktoré momentálne obsahuje. Na zistenie veľkosti vektora používate funkciu `size()`.

```cpp
std::vector<int> numbers = {1, 2, 3, 4, 5};
size_t size = numbers.size(); // size obsahuje hodnotu 5
```

Veľkosť vektora sa automaticky mení, keď pridávate alebo odoberáte prvky. Ak ju chceme zmeniť explicitne, tak môžeme pomocou `resize(new_size)`, ak je nová veľkošt väčšia, tak sa nové prvky default inicializujú. 

### Kapacita (capacity):

Kapacita vektora označuje maximálny počet prvkov, ktoré môže vektor obsahovať, bez potreby reálnej alokácie ďalšej pamäti. Kapacitu môžete zistiť pomocou funkcie `capacity()`. Kapacita sa môže zvýšiť počas operácie `push_back()` alebo iných operácií, ktoré pridávajú prvky do vektora. Štandardne sa kapacita nezmenšuje pri odoberaní prvkov. 

Keď je veľkosť vektora rovnaká ako kapacita, znamená to, že vektor je plný a ďalšie pridávanie prvkov môže viesť k reálnej alokácií pamäte, čo môže byť náročné na výkon. Preto v kritickych miestach je vhodné poznať kapacitu a podľa potreby ju meniť, aby sa minimalizovala realokácia pamäte.

Ak chcete explicitne nastaviť kapacitu vektora, môžete použiť metódu `reserve()`. Znova vieme iba zväčšovať. Ak chceme kapacitu zmenšiť máme funkciu `shrink_to_fit()`, ktorá **by mala** presunuť vektor do bloku pamäte, ktorý presne zodpovedá jeho aktuálnym potrebám. 

```cpp
std::vector<int> numbers;

// explicitné rezervovanie kapacity pre 10 prvkov
numbers.reserve(10);

// teraz môžeme pridávať prvky
for (int i = 1; i <= 10; i++) {
    numbers.push_back(i); // nikdy neurobi realokaciu
}
```

## Najčastejšie chyby

Vektor is síce extrémne rýchly, ale táto rýchlost je dosiahnutá velkým množstvom nedefinovaného správania. 

### Operátor []

Operátor prístupu k prvkom nekontroluje či pristupujeme správne, takže sa nám môže stať, že čitame neinicializovanú pamäť, alebo ešte horšie spôsobime porušenie ochrany pamäte. 

```cpp
std::vector<int> a = { 1, 2, 3, 4, 5};

std::cout << a[5]; // garbage

a[10000] = 0; // access violation or segmentation fault
```

### Neplatné iterátory

Iterátory sa zneplatia, keď prvok na ktorý ukazujú sa z vektora odstráni. Rovnako sú zneplatnené aj iterátory za prvkom, ktorý mažeme. To je ešte celkom pochopiteľné, navyše ale iterátory prestávajú byť platné aj keď vektor urobí realokáciu. 

```cpp
std::vector<int> a = { 1, 2, 3, 4, 5};

auto it = a.begin() + 1; // points to 2

a.erase(a.end() - 1);
// it still valid

a.erase(a.begin());
// it invalid
```

Pri `erase` ešte pozor `vec.erase(vec.end())` je nedefinované správanie.

```cpp
std::vector<int> a = { 1, 2, 3, 4, 5};

auto it = a.begin() + 1; // points to 2

a.push_back(0);
// it may be invalid
```

V predchádzajúcom príklade nie je úplne jasné, či je iterátor platný, alebo nie. Môžeme to zistiť, tak že skontrolujeme kapacitu. 

```cpp
std::vector<int> a = { 1, 2, 3, 4, 5};

auto it = a.begin() + 1; // points to 2

auto space_available = a.capacity() > a.size();
a.push_back(0);

if (space_available) {
    // it is valid
} else {
    // it is invalid
}
```

### Modifikácia vektora počas iterácie

```cpp
std::vector<int> a = { 1, 2, 3, 4, 5};

for (auto i : a) {
    if (i % 2 == 1) {
        a.push_back(0);
    }
}
```

Mohli by sme sa domievať, že kód vyššie na konci nechá vektor naplnený hodnotami `1, 2, 3, 4, 5, 0, 0, 0` (jedna nula za každé nepárne číslo). Nie je tomu ale tak, lebo range based for cyklus používa iterátory a `push_back` ich môže zneplatniť. Preto kód vyššie môźe viesť k nedefinovanému správaniu. 

Vo všeobecnosti nie je dobrý nápad iterovať vektor a meniť počet prvkov. Niektoré použitia sú ale OK. Napríklad `erase` vo for cykle. Stačí si zapamätať, čo zneplatnuje iterátory a mali by ste byť OK. 