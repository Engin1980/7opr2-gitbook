---
icon: square-5
---

# JSA - Množinové a pokročilé operace

V předchozích částech jsme viděli základní použití `collect(...)`, typicky pro převod streamu na `List`, `Set` nebo `Map`. Collectory ale umí mnohem víc než jen „uložit prvky do kolekce“. Jejich skutečná síla se projeví při složitějším zpracování dat, například při seskupování nebo výpočtu agregací nad skupinami.

Základní princip zůstává stejný: stream popisuje, jak data transformujeme, a collector určuje, jak bude vypadat výsledná struktura. U pokročilého použití ale collector často nejen ukládá prvky, ale zároveň nad nimi provádí další operace.

## Množinové operace

Při práci s kolekcemi často potřebujeme kombinovat data z více zdrojů nebo z nich odstranit duplicity. Stream API sice nemá přímo metody pojmenované `union` nebo `intersection`, ale tyto operace lze snadno vyjádřit pomocí existujících nástrojů, jako jsou `concat`, `filter` a `distinct`.

### Sjednocení

Začněme operací **union**. Ta odpovídá sjednocení dvou množin, tedy spojení všech prvků z obou kolekcí do jedné. Pokud nechceme řešit duplicity, stačí použít spojení streamů.

```java
import java.util.List;
import java.util.stream.Stream;

public class Main {
    public static void main(String[] args) {
        List<Integer> a = List.of(1, 2, 3);
        List<Integer> b = List.of(3, 4, 5);

        var vysledek = Stream.concat(a.stream(), b.stream())
                             .toList();

        System.out.println(vysledek);
    }
}
```

Tento zápis vytvoří jeden stream ze dvou a následně jej převede na seznam. Výsledek bude obsahovat všechny prvky, včetně duplicit.

Pokud chceme skutečné sjednocení ve smyslu množinové operace (tedy bez duplicit), přidáme operaci `distinct`:

```java
var vysledek = Stream.concat(a.stream(), b.stream())
                     .distinct()
                     .toList();
```

### Průnik

Operace **intersection** představuje průnik dvou kolekcí, tedy prvky, které se nacházejí v obou. Nejjednodušší způsob, jak ji vyjádřit, je filtrovat jednu kolekci podle toho, zda její prvky obsahuje ta druhá.

```java
import java.util.List;

public class Main {
    public static void main(String[] args) {
        List<Integer> a = List.of(1, 2, 3);
        List<Integer> b = List.of(2, 3, 4);

        var vysledek = a.stream()
            .filter(b::contains)
            .toList();

        System.out.println(vysledek);
    }
}
```

Result obsahuje pouze hodnoty, které jsou v obou kolekcích. V případě potřeby lze opět přidat `distinct`, pokud by vstupy obsahovaly duplicity.

Je třeba si uvědomit, že operace `contains` může být u některých typů kolekcí pomalejší. V takovém případě se často používá `Set` kvůli rychlejšímu vyhledávání, ale princip zůstává stejný.

### Rozdíl

Další běžnou operací nad kolekcemi je množinový rozdíl. Ten odpovídá situaci, kdy chceme vzít prvky z jedné kolekce a odstranit z nich všechny prvky, které se nacházejí v jiné kolekci. Výsledkem jsou tedy prvky, které existují v první kolekci, ale ne ve druhé.

Stream API pro tuto operaci opět nemá speciální metodu, ale lze ji velmi přirozeně vyjádřit pomocí kombinace `filter` a `contains`.

Základní varianta vypadá takto:

```java
import java.util.List;

public class Main {
    public static void main(String[] args) {
        List<Integer> a = List.of(1, 2, 3, 4);
        List<Integer> b = List.of(3, 4, 5);

        List<Integer> rozdil = a.stream()
            .filter(n -> !b.contains(n))
            .toList();

        System.out.println(rozdil);
    }
}
```

Zápis se čte velmi přímočaře: vezmi prvky z kolekce `a` a ponech jen ty, které nejsou obsažené v kolekci `b`. Výsledkem bude seznam `[1, 2]`.

Tento přístup je vhodný pro menší kolekce, ale je dobré si uvědomit, že metoda `contains` má u seznamu lineární složitost. To znamená, že pro větší data může být řešení neefektivní. V takových případech se vyplatí druhou kolekci převést na množinu (`Set`), kde je vyhledávání výrazně rychlejší.

```java
import java.util.*;

public class Main {
    public static void main(String[] args) {
        List<Integer> a = List.of(1, 2, 3, 4);
        List<Integer> b = List.of(3, 4, 5);

        Set<Integer> set = new HashSet<>(b);

        List<Integer> rozdil = a.stream()
            .filter(n -> !set.contains(n))
            .toList();

        System.out.println(rozdil);
    }
}
```

Zobrazit více řádků

Princip zůstává stejný, ale vyhledávání v `Set` má konstantní složitost, takže se řešení výrazně zrychlí.

Množinový rozdíl lze použít nejen pro jednoduché typy, ale i pro objekty. Typicky chceme odebrat prvky na základě nějaké vlastnosti. V takovém případě se často kombinuje s transformací na klíče:

```java
Set<String> zakazanaJmena = uzivateleB.stream()
    .map(u -> u.jmeno)
    .collect(Collectors.toSet());

List<Uzivatel> vysledek = uzivateleA.stream()
    .filter(u -> !zakazanaJmena.contains(u.jmeno))
    .toList();
```

Tento příklad říká: vezmi uživatele z první kolekce a odstraň ty, jejichž jméno se vyskytuje ve druhé kolekci.

Z pohledu návrhu jde o velmi jednoduchý, ale důležitý vzor. Kombinace `filter` a negované podmínky (`!contains`) je základní stavební kámen pro vyjádření rozdílu mezi kolekcemi.

### Jedinečnost

#### Distinct

Operace **distinct** slouží k odstranění duplicitních prvků ze streamu. Zachová první výskyt každé hodnoty a ostatní zahodí. Používá se velmi často jako samostatná operace nebo v kombinaci s jinými kroky.

```java
import java.util.List;

public class Main {
    public static void main(String[] args) {
        List<String> slova = List.of("java", "stream", "java", "api");

        var vysledek = slova.stream()
            .distinct()
            .toList();

        System.out.println(vysledek);
    }
}
```

Výsledkem je seznam bez duplicit.

`distinct` je založený na metodách `equals` a `hashCode`. To znamená, že u vlastních objektů je potřeba mít tyto metody správně implementované, jinak nemusí odstranění duplicit fungovat podle očekávání.

#### DistinctBy

Metoda `distinct()` odstraňuje duplicity na základě celé rovnosti objektu (`equals`). V reálných úlohách ale často potřebujeme odstranit duplicity jen podle určité vlastnosti, například podle ID nebo jména. Této operaci se běžně říká `distinctBy`.

Stream API ji přímo neobsahuje, ale lze ji velmi dobře vyjádřit pomocí collectoru `toMap`. Myšlenka je taková, že vytvoříme mapu, kde klíčem je hodnota, podle které určujeme unikátnost, a hodnotou samotný objekt. Pokud narazíme na duplicitní klíč, rozhodneme, který prvek chceme zachovat.

Ukažme si příklad. Máme seznam osob a chceme odstranit duplicity podle jména:

```java
import java.util.*;
import java.util.stream.Collectors;

class Osoba {
    String jmeno;
    int vek;
}

public class Main {
    public static void main(String[] args) {
        List<Osoba> osoby = List.of(
            new Osoba("Jan", 20),
            new Osoba("Petra", 25),
            new Osoba("Jan", 30)
        );

        List<Osoba> vysledek = new ArrayList<>(
            osoby.stream()
                .collect(Collectors.toMap(
                    o -> o.jmeno,
                    o -> o,
                    (puvodni, novy) -> puvodni
                ))
                .values()
        );

        vysledek.forEach(o -> System.out.println(o.jmeno + " " + o.vek));
    }
}
```

Collector `toMap` zde vytváří mapu, kde klíčem je jméno. Pokud se objeví dvě osoby se stejným jménem, použije se merge funkce `(puvodni, novy) -> puvodni`, která říká, že se má zachovat první výskyt.

Výsledkem je kolekce unikátních prvků podle zvoleného klíče.

Tento přístup má několik výhod. Vyhýbá se side efektům, je bezpečný i pro paralelní streamy a zároveň velmi dobře zapadá do deklarativního stylu Stream API. Stačí si zapamatovat jednoduchý vzor: převést data do mapy podle klíče a pak vzít její hodnoty.

Použití `distinctBy` tímto způsobem je velmi časté, protože umožňuje přesně říct, co znamená „duplicitní“ prvek, aniž bychom museli upravovat metodu `equals` v doménových třídách.

{% hint style="danger" %}
V návodech někdy můžete najít řešení postavené nad `Set`, do které se ukládají už viděné klíče. Filtr pak propustí pouze ty prvky, jejichž klíč se v množině objeví poprvé.

```java
Set<String> seen = new HashSet<>();

List<Osoba> vysledek = osoby.stream()
    .filter(o -> seen.add(o.jmeno))
    .toList();
```

Metoda `Set.add()` vrací `true`, pouze pokud se prvek v množině objeví poprvé. Toho využíváme přímo ve filtru. Výsledkem je seznam, kde se každé jméno objeví jen jednou.

Je ale důležité si uvědomit, že **toto řešení používá side effect** (změnu obsahu množiny). To znamená, že není vhodné pro paralelní streamy, kde by mohlo dojít k problémům s konzistencí. Pokud bychom chtěli paralelní variantu, bylo by potřeba použít thread-safe strukturu (`ConcurrentHashMap` apod.).
{% endhint %}

## Seskupování

{% hint style="info" %}
Tato kapitola pokrývá pokročilejší techniky práce s Java Stream API a její znalost není povinná.
{% endhint %}

### GroupingBy

Jedním z nejčastějších případů je seskupování pomocí `groupingBy`. Tato operace vytváří mapu, kde klíčem je hodnota odvozená z prvku a hodnotou je kolekce prvků, které do skupiny patří.

Typický příklad: máme seznam osob a chceme je seskupit podle věku.

```java
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

class Osoba {
    String jmeno;
    int vek;
}

public class Main {
    public static void main(String[] args) {
        List<Osoba> osoby = List.of(
            new Osoba("Jan", 20),
            new Osoba("Petra", 25),
            new Osoba("Karel", 20)
        );

        Map<Integer, List<Osoba>> vysledek = osoby.stream()
            .collect(Collectors.groupingBy(o -> o.vek));

        System.out.println(vysledek);
    }
}
```

Výsledkem je mapa, kde klíčem je věk a hodnotou seznam osob s tímto věkem. Tento styl odpovídá klasickému „group by“ z databází.

### PartitioningBy

Podobná operace je `partitioningBy`, která rozděluje prvky jen do dvou skupin podle podmínky. Klíčem je zde typu `boolean`.

Například chceme rozdělit čísla na sudá a lichá:

```java
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

public class Main {
    public static void main(String[] args) {
        List<Integer> cisla = List.of(1, 2, 3, 4, 5);

        Map<Boolean, List<Integer>> vysledek = cisla.stream()
            .collect(Collectors.partitioningBy(n -> n % 2 == 0));

        System.out.println(vysledek);
    }
}
```

Klíč `true` obsahuje sudá čísla, klíč `false` lichá. Tento zápis je velmi přehledný pro případy, kdy máme jednoduchou podmínku typu ano/ne.

### Merge funkce

Operace `toMap`, kterou jsme už částečně viděli, lze také použít pokročileji. Kromě definice klíče a hodnoty můžeme řešit i kolize klíčů pomocí tzv. merge funkce.

Například chceme mapu jméno → věk, ale máme duplicity:

```java
Map<String, Integer> mapa = osoby.stream()
    .collect(Collectors.toMap(
        o -> o.jmeno,
        o -> o.vek,
        (stary, novy) -> novy
    ));
```

Třetí parametr říká, co se má stát, když existují dva prvky se stejným klíčem. V tomto případě se použije nová hodnota.

### Downstream operace

Velmi silnou vlastností collectorů je možnost tzv. downstream operací. To znamená, že výsledek seskupení můžeme ještě dále zpracovat, aniž bychom museli psát další pipeline.

Například místo toho, abychom u `groupingBy` dostali seznamy, můžeme rovnou spočítat jejich velikost:

```java
Map<Integer, Long> pocetVeSkupine = osoby.stream()
    .collect(Collectors.groupingBy(
        o -> o.vek,
        Collectors.counting()
    ));
```

Výsledkem není mapa věk → seznam osob, ale věk → počet osob.

Podobně můžeme použít `mapping`, který umožňuje transformovat prvky uvnitř skupiny. Například chceme místo objektů uložit jen jména:

```java
Map<Integer, List<String>> jmenaPodleVeku = osoby.stream()
    .collect(Collectors.groupingBy(
        o -> o.vek,
        Collectors.mapping(o -> o.jmeno, Collectors.toList())
    ));
```

Zde se nejprve provede seskupení podle věku a následně se každý prvek uvnitř skupiny transformuje na jméno.

Downstream operace výrazně rozšiřují možnosti collectorů. Umožňují kombinovat více kroků do jednoho zápisu a vyhnout se zbytečnému vytváření mezivýsledků.

Typická komplexnější pipeline pak může vypadat takto: data se převedou na stream, aplikují se základní transformace a nakonec se pomocí `collect` nejen uloží, ale rovnou uspořádají a agregují.

Z praktického pohledu tak collectory nejsou jen nástroj pro „uložení výsledku“, ale plnohodnotný mechanismus pro datové transformace. Umožňují vyjádřit i složitější operace, jako je seskupování, dělení dat nebo výpočty nad skupinami, a to vše deklarativně a přehledně.
