---
icon: square-3
---

# JSA - Práce se streamy

V předchozích kapitolách jsme si ukázali, jak ze zdrojové kolekce vytvořit stream a jak nad ním provádět základní operace. Teď se zaměříme na jádro práce se Stream API: filtrování dat a jejich transformaci. Právě kombinace těchto operací dává streamům jejich sílu, protože umožňuje zapisovat datové transformace deklarativně, čitelně a bez explicitních cyklů.

{% hint style="info" %}
V následujících ukázkách kódu budeme používat jako terminální operaci `foreach`, která vezme daný prvek a udělá nad ním požadovanou činnost. Nejčastěji budeme používat:

* `foreach(System.out::println)`, která daný prvek vytiskne na konzoli, případně
* `foreach(q -> System.out.println(q.something())`, která vytiskne výsledek dané operace na konzoli.
{% endhint %}

## Filtrace

Operace `filter` slouží k výběru prvků ze streamu podle podmínky. Tato podmínka je reprezentována funkcionálním rozhraním `Predicate<T>`, tedy funkcí, která pro daný vstup vrací `true`, nebo `false`. Prvky, pro které predikát vrací `true`, projdou dál pipeline, ostatní jsou odfiltrovány.

Operace `filter` je v praxi velmi frekventovaná, protože téměř každá reálná úloha začíná výběrem relevantních dat.

Jednoduchý příklad: máme seznam čísel a chceme vybrat pouze sudá. Této definici odpovídá jednoduchá lambda kontrolující dělitelnost dvěma: `n -> n % 2 == 0`:

```java
import java.util.List;

public class Main {
    public static void main(String[] args) {
        List<Integer> cisla = List.of(1, 2, 3, 4, 5, 6);

        cisla.stream()
              .filter(n -> n % 2 == 0)
              .forEach(System.out::println);
    }
}
```

Další příklad. Máme seznam slov a chceme vybrat pouze ta, která mají délku větší než tři znaky:

```java
import java.util.List;

public class Main {
    public static void main(String[] args) {
        List<Integer> cisla = List.of(1, 2, 3, 4, 5, 6);

        cisla.stream()
              .filter(n -> n % 2 == 0)
              .forEach(System.out::println);
    }
}
```

Podmínka ve `filter` může být samozřejmě složitější. Můžeme kombinovat více kritérií, například vybrat slova, která začínají na určité písmeno a zároveň mají minimální délku:

```java
import java.util.List;

public class Main {
    public static void main(String[] args) {
        List<String> slova = List.of("auto", "autobus", "vlak", "letadlo", "alfa");

        slova.stream()
             .filter(s -> s.startsWith("a") && s.length() >= 5)
             .forEach(System.out::println);
    }
}
```

Zajímavější situace nastává u objektů. Typicky filtrujeme podle nějaké vlastnosti. Například seznam produktů a chceme pouze ty, které jsou skladem:

```java
import java.util.List;

class Produkt {
    String nazev;
    int pocetNaSklade;
}

public class Main {
    public static void main(String[] args) {
        List<Produkt> produkty = List.of(
            new Produkt("Notebook", 5),
            new Produkt("Myš", 0),
            new Produkt("Klávesnice", 12)
        );

        List<Produkt> skladem = produkty.stream()
                .filter(p -> p.pocetNaSklade > 0);
                .forEach(p -> System.out.println(p.nazev));
    }
}
```

Podmínky ve filtru mohou být i vícestupňové a lze je rozdělit do více filtrovacích kroků. To často zlepší čitelnost. Následující příklad je funkčně stejný jako předchozí, ale ukazuje rozdělení logiky:

```java
import java.util.List;

public class Main {
    public static void main(String[] args) {
        List<String> slova = List.of("auto", "autobus", "vlak", "letadlo", "alfa");

        slova.stream()
             .filter(s -> s.startsWith("a"))
             .filter(s -> s.length() >= 5)
             .forEach(System.out::println);
    }
}
```

Další praktický scénář je práce s `null` hodnotami. Ve starším kódu se s nimi setkáváme často a Stream API umožňuje jejich snadné odfiltrování:

```java
import java.util.List;

public class Main {
    public static void main(String[] args) {
        List<String> hodnoty = List.of("A", null, "B", null, "C");

        hodnoty.stream()
                .filter(s -> s != null)
                .forEach(System.out::println);
    }
}
```

Velmi častá je i kombinace `filter` s vlastním pomocným predikátem. Podmínku můžeme vyčlenit do samostatné metody, což zvyšuje přehlednost a znovupoužitelnost:

```java
import java.util.List;

class Uzivatel {
    String jmeno;
    boolean aktivni;
}

public class Main {

    static boolean jeAktivni(Uzivatel u) { // <-- vlastní pomocná metoda
        return u.aktivni;
    }

    public static void main(String[] args) {
        List<Uzivatel> uzivatele = List.of(
            new Uzivatel("Jan", true),
            new Uzivatel("Petr", false),
            new Uzivatel("Eva", true)
        );

        uzivatele.stream()
                 .filter(Main::jeAktivni) // <-- použití vlastní pomocné metody
                 .forEach(u -> System.out.println(u.jmeno));
    }
}
```

## Řazení

Řazení dat je další velmi častá operace při práci s kolekcemi. Ve Stream API se provádí pomocí metody `sorted()`, která vrací nový stream s prvky uspořádanými podle zvoleného kritéria. Jde o mezilehlou operaci, takže ji obvykle zařazujeme doprostřed pipeline a výsledek získáme až pomocí ukončovací operace, například `toList()` nebo `forEach`.

### Nativní řazení

Nejjednodušší varianta je řazení přirozeným způsobem. To funguje pro typy, které implementují rozhraní `Comparable`, například `Integer` nebo `String`.

```java
import java.util.List;

public class Main {
    public static void main(String[] args) {
        List<Integer> cisla = List.of(5, 2, 9, 1);

        List<Integer> serazeno = cisla.stream()
            .sorted()
            .toList();

        System.out.println(serazeno);
    }
}
```

Výsledkem je seznam `[1, 2, 5, 9]`. U řetězců se použije lexikografické pořadí:

```java
List<String> slova = List.of("banana", "apple", "pear");

List<String> serazeno = slova.stream()
    .sorted()
    .toList();
```

Obecně, základní metoda `sorted()` využívá řazení definovaného nad řazeným typem. Daný typ tedy musí implementovat rozhraní `Comparable<T>`, jehož implementace definuje, jak se řazení provádí.&#x20;

### Řazení vlastním porovnáním

Pokud chceme řazení upravit, použijeme variantu `sorted(Comparator)`. Komparátor určuje, podle čeho a jak se mají prvky porovnávat.

V nejjednodušší formě zadáváme způsob, jak se porovnávají dva prvky (např. `a` a `b`). Příklad - řazení čísel sestupně:

```java
List<Integer> serazeno = cisla.stream()
    .sorted((a, b) -> b - a)
    .toList();
```

### Řazení přes Comparator

Pro další budeme používat vestavěné metody třídy `Comparator`.

Při práci s objekty je **řazení podle&#x20;**_**něčeho**_ velmi běžný scénář. Typicky chceme prvky seřadit podle nějaké vlastnosti. K tomu slouží metoda `Comparator.comparing`.

```java
import java.util.Comparator;
import java.util.List;

class Produkt {
    String nazev;
    double cena;
}

public class Main {
    public static void main(String[] args) {
        List<Produkt> produkty = List.of(
            new Produkt("Notebook", 25000),
            new Produkt("Myš", 500),
            new Produkt("Monitor", 7000)
        );

        List<Produkt> serazeno = produkty.stream()
            .sorted(Comparator.comparing(p -> p.cena))
            .toList();

        serazeno.forEach(p -> System.out.println(p.nazev));
    }
}
```

Tento zápis říká: seřaď produkty podle ceny vzestupně.

Pokud chceme sestupné řazení, použijeme metodu `reversed()`:

```java
List<Produkt> serazeno = produkty.stream()
    .sorted(Comparator.comparing((Produkt p) -> p.cena).reversed())
    .toList();
```

Velmi užitečné je i vícestupňové řazení. Například chceme seřadit osoby nejprve podle věku a při shodě podle jména:

```java
import java.util.Comparator;

List<Osoba> serazeno = osoby.stream()
    .sorted(
        Comparator.comparing((Osoba o) -> o.vek)
                  .thenComparing(o -> o.jmeno)
    )
    .toList();
```

Takový zápis odpovídá řazení podle více sloupců, jak ho známe z databází.

{% hint style="warning" %}
Je dobré si uvědomit, že operace `sorted()` musí zpracovat celý stream, protože pro správné seřazení potřebuje znát všechny prvky. To znamená, že na rozdíl od některých jiných operací není „líná“ ve smyslu postupného vyhodnocování a může být náročnější na výkon u velkých dat.
{% endhint %}

Pro ukázku je v následující tabulce shrnutí běžně používaných metod.

<table data-header-hidden><thead><tr><th width="174.75"></th><th width="254.949951171875"></th><th></th></tr></thead><tbody><tr><td><strong>Metoda</strong></td><td><strong>Popis</strong></td><td><strong>Příklad použití ve Streamu</strong></td></tr><tr><td><code>naturalOrder()</code></td><td>Řadí prvky v jejich přirozeném pořadí (např. čísla vzestupně, text abecedně). Prvky musí implementovat <code>Comparable</code>.</td><td><p><code>.sorted(</code></p><p>  <code>Comparator.naturalOrder())</code></p></td></tr><tr><td><code>reverseOrder()</code></td><td>Řadí prvky v opačném než přirozeném pořadí (sestupně).</td><td><p><code>.sorted(</code></p><p>  <code>Comparator.reverseOrder())</code></p></td></tr><tr><td><code>comparing()</code></td><td>Nejčastější metoda. Extrahne klíč (vlastnost objektu) a seřadí objekty podle něj.</td><td><p><code>.sorted(</code></p><p>  <code>Comparator.comparing(</code></p><p>    <code>User::getName))</code></p></td></tr><tr><td><p><code>comparingInt()</code></p><p><br></p><p><code>comparingLong()</code></p><p><br></p><p><code>comparingDouble()</code></p></td><td>Speciální varianty <code>comparing()</code> pro primitivní datové typy. Brání zbytečnému autoboxingu.</td><td><p><code>.sorted(</code></p><p>  <code>Comparator.comparingInt(</code></p><p>    <code>User::getAge))</code></p></td></tr><tr><td><code>thenComparing()</code></td><td>Slouží k řetězení řazení. Pokud jsou si prvky v prvním kritériu rovny, rozhodne toto další.</td><td><p><code>.sorted(</code></p><p>  <code>Comparator.comparing(</code></p><p>    <code>User::getLastName)</code></p><p>  <code>.thenComparing(</code></p><p>    <code>User::getFirstName))</code></p></td></tr><tr><td><code>reversed()</code></td><td>Obrátí pořadí již existujícího komparátoru.</td><td><p><code>.sorted(</code></p><p>  <code>Comparator.comparing(</code></p><p>    <code>User::getAge)</code></p><p>  <code>.reversed())</code></p></td></tr><tr><td><code>nullsFirst()</code></td><td>Zajistí, že prvky s hodnotou <code>null</code> budou na začátku streamu (předchází <code>NullPointerException</code>).</td><td><p><code>.sorted(</code></p><p>  <code>Comparator.nullsFirst(</code></p><p>    <code>Comparator.naturalOrder()))</code></p></td></tr><tr><td><code>nullsLast()</code></td><td>Stejné jako <code>nullsFirst</code>, ale prvky s hodnotou <code>null</code> odsunie až na úplný konec streamu.</td><td><p><code>.sorted(</code></p><p>  <code>Comparator.nullsLast(</code></p><p>    <code>Comparator.naturalOrder()))</code></p></td></tr></tbody></table>

## Transformace

Operace `map` je druhý základní stavební kámen práce se Stream API. Zatímco `filter` rozhoduje, které prvky dál pustí a které zahodí, `map` se stará o to, **jak se jednotlivé prvky změní**. Jinými slovy: `map` transformuje každý prvek streamu na jiný prvek.

Princip je jednoduchý, ale velmi silný. Každý vstupní prvek projde transformační funkcí a výsledkem je nový stream obsahující už transformované hodnoty. Tato transformační funkce odpovídá funkcionálnímu rozhraní `Function<T, R>`, takže vstupní a výstupní typ se mohou lišit.

Začněme úplně jednoduchým příkladem. Máme seznam čísel a chceme každé číslo přepočítat, například vynásobit dvěma (lambda `n -> n * 2`):

```java
import java.util.List;

public class Main {
    public static void main(String[] args) {
        List<Integer> cisla = List.of(1, 2, 3, 4);

        cisla.stream()
              .map(n -> n * 2)
              .forEach(System.out::println);
    }
}
```

Každý prvek `n` projde funkcí `n -> n * 2` a výsledkem je nový stream hodnot `2, 4, 6, 8`.

Důležité je si uvědomit, že `map` **nemění původní kolekci**, ale vytváří nový stream. Stream API je deklarativní a nemění stav dat, ale vytváří transformační pipeline.

Velmi časté je použití `map` při převodu objektů na jejich atributy. Například máme seznam osob a chceme získat pouze jejich jména:

```java
import java.util.List;

class Osoba {
    String jmeno;
    int vek;
}

public class Main {
    public static void main(String[] args) {
        List<Osoba> osoby = List.of(
            new Osoba("Jan", 20),
            new Osoba("Petra", 25),
            new Osoba("Karel", 30)
        );

        osoby.stream()
              .map(o -> o.jmeno)
              .forEach(System.out::println);
    }
}
```

Tady je dobře vidět změna typu: ze `Stream<Osoba>` se stává `Stream<String>`. To je typický vzor – z komplexních objektů vytahujeme hodnoty, které nás zajímají.

Transformace ale nemusí být jen „vytažení hodnoty“. Může jít o libovolnou logiku. Například převod jmen na velká písmena:

```java
osoby.stream()
     .map(o -> o.jmeno.toUpperCase())
     .forEach(System.out::println);
```

Nebo složenější transformace, kde vytváříme úplně nový objekt:

```java
class UserDTO {
    String displayName;

    UserDTO(String displayName) {
        this.displayName = displayName;
    }
}

osoby.stream()
     .map(o -> new UserDTO(o.jmeno + " (" + o.vek + ")"))
     .forEach(dto -> System.out.println(dto.displayName));
```

V tomto případě `map` převádí doménový objekt na jinou reprezentaci, což je velmi časté například ve webových aplikacích (entity → DTO).

`map` se také často používá v kombinaci s dalšími operacemi. Typický vzor je: nejprve data vyfiltrujeme a až poté transformujeme. Například chceme jména osob starších 25 let:

```java
osoby.stream()
     .filter(o -> o.vek > 25)
     .map(o -> o.jmeno)
     .forEach(System.out::println);
```

Pořadí operací hraje roli. Nejprve se provede `filter`, čímž se zmenší množství dat, a teprve pak se na zbývající prvky aplikuje `map`. To je nejen čitelnější, ale i efektivnější.

Další praktická oblast je práce s čísly a výpočty. Například chceme ze seznamu cen vytvořit seznam cen s DPH:

```java
import java.util.List;

public class Main {
    public static void main(String[] args) {
        List<Double> ceny = List.of(100.0, 200.0, 300.0);

        ceny.stream()
            .map(c -> c * 1.21)
            .forEach(System.out::println);
    }
}
```

Tento styl je velmi čitelný, protože přesně odpovídá tomu, co chceme říct: „vezmi každou cenu a přepočítej ji“.

Ještě jedna důležitá vlastnost: **`map` vždy zachovává počet prvků ve streamu. Každý vstupní prvek odpovídá právě jednomu výstupnímu.**

Operace `map` je tak univerzální nástroj pro transformaci dat. V kombinaci s `filter` tvoří základ většiny streamových pipeline a umožňuje zapisovat logiku způsobem, který je přehledný, modulární a snadno rozšiřitelný.

## Transformace flatMap

Operace `flatMap` patří mezi pokročilejší transformační operace ve Stream API. Na první pohled se může zdát podobná jako `map`, ale její význam je zásadně odlišný. Zatímco `map` převádí jeden prvek na jeden jiný prvek, `flatMap` umožňuje převést jeden prvek na **více prvků** a ty následně „spojit“ (zploštit) do jednoho výsledného streamu.

To znamená, že `flatMap` řeší situace, kdy transformační funkce vrací kolekci nebo jiný stream. Pokud bychom použili obyčejné `map`, dostali bychom vnořenou strukturu, například `Stream<List<T>>`. `flatMap` tuto strukturu rozbalí na „plochý“ `Stream<T>`.

{% hint style="info" %}
**Motivace:** Uvažujme, že osobá má seznam telefonních čísel. Pokud mám list osob a provedl bych operaci `map` pro získání telefonních čísel, vznikl by mi list listů telefonních čísel, protože pro každou osobu mám vlastní list - `map` zachovává počet prvků ve streamu.

Já ale chci jeden velký list, abych na tato čísla mohl poslat SMS. Proto použiji `flatMap`, která ten vyšší list rozbalí a všechny položky dá do jednoho seznamu.

Například, pro:

```json
[
  {
    "name": "John",
    "phoneNumbers": [111, 222]
  },
  {
    "name": "Jane",
    "phoneNumbers": [555]
  },
  {
    "name": "Victor",
    "phoneNumbers": [777, 888, 999]
  }
]
```

... byste jednoduchým `map` dostali list listů položek (všimněte si závorek `[]`):

```json
[
  [111, 222], [555], [777, 888, 999]
]
```

... ale pomocí `flatMap` získám nezanořený (flat) list:

```json
[ 111, 222, 555, 777, 888, 999]
```
{% endhint %}

Začněme jednoduchým příkladem, na kterém je dobře vidět rozdíl mezi `map` a `flatMap`. Máme seznam vět a chceme získat všechna slova.

Pokud použijeme `map`, každá věta se převede na pole slov:

```java
import java.util.List;
import java.util.Arrays;

public class Main {
    public static void main(String[] args) {
        List<String> vety = List.of(
            "Java je super",
            "Stream API je mocne"
        );

        vety.stream()
            .map(v -> v.split(" "))
            .forEach(arr -> System.out.println(Arrays.toString(arr)));
    }
}
```

Výsledkem není jeden proud slov, ale proud polí. Každý prvek streamu je pole (`String[]`). To je často nepoužitelné pro další zpracování.

Teď použijeme `flatMap`:

```java
import java.util.List;
import java.util.Arrays;

public class Main {
    public static void main(String[] args) {
        List<String> vety = List.of(
            "Java je super",
            "Stream API je mocne"
        );

        vety.stream()
            .flatMap(v -> Arrays.stream(v.split(" ")))
            .forEach(System.out::println);
    }
}
```

Každá věta se sice opět převede na pole slov, ale `flatMap` následně všechny tyto „vnitřní streamy“ spojí do jednoho. Výsledkem je lineární stream všech slov.

To je základní myšlenka: `flatMap` = transformace + zploštění.

Velmi časté použití `flatMap` je při práci s kolekcemi uvnitř kolekcí. Můžeme si to představit jako relaci „jeden prvek obsahuje více prvků“. Například máme seznam objednávek a každá objednávka obsahuje seznam položek:

```java
import java.util.List;

class Polozka {
    String nazev;
}

class Objednavka {
    List<Polozka> polozky;
}

public class Main {
    public static void main(String[] args) {
        List<Objednavka> objednavky = List.of(
            new Objednavka(List.of(new Polozka("Kniha"), new Polozka("Pero"))),
            new Objednavka(List.of(new Polozka("Notebook")))
        );

        objednavky.stream()
                  .flatMap(o -> o.polozky.stream())
                  .forEach(p -> System.out.println(p.nazev));
    }
}
```

Každá objednávka obsahuje seznam položek. Pomocí `flatMap` tyto seznamy rozbalíme a získáme jeden společný stream všech položek.

Bez `flatMap` bychom skončili se strukturou `Stream<List<Polozka>>`, což by znamenalo nutnost vnořeného zpracování.

Důležitá vlastnost `flatMap` je, že jeden vstupní prvek může odpovídat:

* žádnému výstupnímu prvku (prázdný stream),
* jednomu prvku,
* nebo více prvkům.

To znamená, že `flatMap` může současně fungovat i jako filtr. Například pokud v transformační funkci vrátíme prázdný stream, daný prvek se „ztratí“:

```java
import java.util.List;
import java.util.stream.Stream;

public class Main {
    public static void main(String[] args) {
        List<Integer> cisla = List.of(1, 2, 3, 4, 5);

        cisla.stream()
              .flatMap(n -> n % 2 == 0 ? Stream.of(n * 10) : Stream.empty())
              .forEach(System.out::println);
    }
}
```

Tento příklad:

* sudá čísla transformuje (vynásobí 10),
* lichá odstraní.

Z hlediska návrhu pipeline je to kombinace filtrace a transformace v jedné operaci. Přesto je v praxi většinou přehlednější použít zvlášť `filter` a `map`. `flatMap` se vyplatí tam, kde skutečně pracujeme s více prvky na vstupu.

`flatMap` se také často používá v kombinaci s dalšími operacemi v pipeline. Například chceme:

1. vzít seznam vět,
2. rozdělit je na slova,
3. ponechat jen delší slova,
4. převést je na velká písmena.

```java
import java.util.List;
import java.util.Arrays;

public class Main {
    public static void main(String[] args) {
        List<String> vety = List.of(
            "Java Stream API",
            "je velmi silný nástroj"
        );

        vety.stream()
            .flatMap(v -> Arrays.stream(v.split(" ")))
            .filter(s -> s.length() > 3)
            .map(String::toUpperCase)
            .forEach(System.out::println);
    }
}
```

Pipeline je zde velmi přirozená: nejprve rozbalení dat (`flatMap`), poté výběr (`filter`) a nakonec transformace (`map`).

Je důležité si uvědomit, že `flatMap` je mezilehlá operace, stejně jako `map` nebo `filter`. Nevytváří výsledek okamžitě, ale pouze popisuje transformaci. K jejímu provedení dojde až při terminační operaci, například `forEach` nebo `collect`.

Z hlediska čitelnosti kódu platí jedno praktické pravidlo: pokud každému vstupnímu prvku odpovídá právě jeden výstupní prvek, použijeme `map`. Pokud jeden vstupní prvek produkuje více hodnot (nebo žádnou), použijeme `flatMap`.

Operace `flatMap` je tedy klíčová ve chvílích, kdy pracujeme s hierarchickými nebo vnořenými daty. Umožňuje tato data převést do lineární podoby a dále s nimi pracovat stejně jednoduše jako s běžným streamem. V kombinaci s `filter` a `map` tvoří základ pro řešení i složitějších transformačních úloh nad kolekcemi.
