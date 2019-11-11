# Vlastné stream operátory `<<` a `>>`

V časti o [textových súboroch](./text_files.md) sme sa dozvedeli ako otvárať súbory, čítať z nich dáta a zapisovať. Takto sa dajú čítať a zapisovať dáta, ktoré majú v štandarde definovaný operátor `<<` a `>>`. Pre vlastné typy vieme tiež pridať túto podporu. 

## Vlastný typ TicTacToe

Najprv si musíme definovať typ, ktorý budeme čítať a zapisovať. Napríklad jednoduchú inštanciu [TicTacToe](https://en.wikipedia.org/wiki/Tic-tac-toe), u nás známou ako piškvorky 3x3. 

```cpp
enum class State
{
    Empty,
    Cross, // X
    Nought, // O
};

struct TicTacToe
{
    State board[3][3] = {};
};
```

Použili sme enum class na definovanie stavu políčka, buď je prázdne, alebo obsahuje krížik, prípadne krúžok. Samotný stav TicTacTou je potom jednoduchá štruktúra, ktorá obsahuje pole stavov políčok 3x3. `= {}` na konci spôsobí, že je toto pole inicializované na 0, čo je zhodou okolností `State::Empty`. 

## Vkladanie dát do `std::ostream`

Na podporu výstupu potrebujeme aby kompilátor vedel nájsť správny operátor `<<`. To urobíme tak, že mimo našu triedu deklarujeme nasledujúci operátor. 

```cpp
std::ostream& operator<<(std::ostream& os, const TicTacToe& rhs);
```

Ľavá strana operátora bude výstupný stream a pravá konštantná referencia na našu triedu. Výstupom potom bude ľavá strana, aby sa dal operátor reťaziť (ak by sme dali výstup ako `void`, tak by nefungovalo `os << t1 << t2`).

Serializovať budeme nasledovne. Deväť znakov za sebou, kde `X` bude krížik, `O` krúžok a `.` bude prázdne políčko. 

```cpp
std::ostream& operator<<(std::ostream& os, const TicTacToe& rhs)
{
    for (const auto& i : rhs.board)
    {
        for (const auto& j : i)
        {
            switch (j)
            {
            case State::Cross:
                os << 'X';
                break;
            case State::Nought:
                os << 'O';
                break;
            case State::Empty:
            default:
                os << '.';
            }
        }
    }
}
```

### Pristup k privátnym častiam

Niekedy v rámci operátora `<<` potrebujeme priamy prístup k členským premenným. To vieme zabezpečiť pomocou kľúčového slova `friend`. 

```cpp
class Person
{
    /* ... */
private:
    std::string m_name;
    uint32_t m_age;

    friend std::ostream& operator<<(std::ostream&, const Person&);
};

std::ostream& operator<<(std::ostream& os, const Person& rhs)
{
    return os << rhs.m_name << " " << rhs.m_age;
}
```

## Extrakcia dát zo `std::istream`

Analogicky pre čítanie vlastných typov zo streamu musíme preťažiť operátor `>>`. Ako výstup musí byť referencia stále na vstupný stream aby sa dal aj operátor `>>` reťaziť. Druhý parameter bude referencia na náš typ (nie konštantná, tú by sme moc nepomenili). 

```cpp
std::istream& operator>>(std::istream& is, TicTacToe& rhs);
```

Jediný problém pri získavaní dát je reportovanie chýb. Na to musíme v správnej chvíli nastaviť `failbit` aby sme použivateľovi funkcie dali vedieť, že je niečo zle. 

```cpp
std::istream& operator>>(std::istream& is, TicTacToe& rhs)
{
    for (size_t i = 0; i < 3; ++i)
    {
        for (size_t j = 0; j < 3; ++j)
        {
            if (char state = '\0'; is >> state)
            {
                switch (state)
                {
                case 'X':
                    rhs.board[i][j] = State::Cross;
                    break;
                case 'O':
                    rhs.board[i][j] = State::Nought;
                    break;
                case '.':
                    rhs.board[i][j] = State::Empty;
                    break;
                default:
                    is.setstate(std::ios::failbit);
                    return is;
                }
            }
            else
            {
                return is;
            }
        }
    }
    // everything OK
    return is;
}
```

Toto je jednoduchý príklad extrahovania dát, ak by nám nestačilo náš operátor vyskladať s už definovaných operátor `>>`, tak môžeme ísť o úroveň nižšie a implementovať niečo čomu sa hovorí [*FormattedInputFunction*](https://en.cppreference.com/w/cpp/named_req/FormattedInputFunction).