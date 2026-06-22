---
icon: square-2
---

# JSA - Vytvoření streamu

Při práci se Stream API je potřeba nejprve pochopit, odkud se stream vlastně bere. Stream totiž není samostatná datová struktura, ale vždy vzniká z nějakého zdroje dat, který chceme zpracovávat. V praxi existuje několik typických situací, se kterými se budeme setkávat.

**Stream z List/Set**

Nejčastěji budeme vycházet z běžných kolekcí, jako jsou seznamy (`List`) nebo množiny (`Set`). V takovém případě je vytvoření streamu velmi jednoduché — kolekce poskytuje metodu `stream()`, která nad jejími prvky vytvoří pohled určený pro další zpracování. Jakmile máme stream, můžeme nad ním začít skládat operace:

```java
List<String> names = List.of("Alice", "Bob", "Charlie");

List<String> result = names.stream()
    .filter(n -> n.startsWith("A"))
    .toList();
```

Tento způsob je zdaleka nejběžnější a v reálných aplikacích se s ním budeme setkávat prakticky neustále. Kolekce zde funguje jako zdroj dat a stream jako nástroj, jak nad nimi provést dotaz.

**Stream z Array**

Pokud pracujeme s polem, situace je velmi podobná, jen nemůžeme použít metodu `stream()` přímo. Pole tuto metodu nemá, a proto využijeme pomocnou třídu `Arrays`, která nám umožní pole převést na stream:

```java
String[] namesArray = {"Alice", "Bob", "Charlie"};

List<String> result = Arrays.stream(namesArray)
    .filter(n -> n.length() > 3)
    .toList();
```

Princip je stejný jako u kolekcí — pouze se liší způsob, jak stream vytvoříme.

**Stream z hodnot**

Další možností je vytvořit stream přímo z konkrétních hodnot. To se hodí například v testech, ukázkách nebo situacích, kdy pracujeme jen s malým množstvím dat. K tomu slouží metoda `Stream.of(...)`, která vezme libovolný počet prvků a vytvoří z nich stream:

```java
List<String> result = Stream.of("Alice", "Bob", "Charlie")
    .filter(n -> n.contains("o"))
    .toList();
```

Tento přístup je velmi jednoduchý a často se používá pro rychlé experimenty nebo demonstrace.

**Stream z generovaných dat**

Zajímavou možností je také vytváření streamu jako generátoru dat. V takovém případě stream nevychází z existující kolekce, ale hodnoty se vytvářejí postupně podle nějakého pravidla. Typickým příkladem je metoda `Stream.iterate(...)`, která umožňuje definovat počáteční hodnotu a způsob, jak se generují další:

```java
List<Integer> result = Stream.iterate(1, n -> n + 1)
    .limit(5)
    .toList();
```

{% hint style="info" %}
Prvním parametrem je počáteční hodnota, druhým parametrem je způsob výpočtu další hodnoty v pořadí)
{% endhint %}

Výsledkem bude sekvence hodnot `[1, 2, 3, 4, 5]`. Vidíme zde zároveň, že u takto generovaných streamů je nutné použít omezení (`limit`), jinak by šlo o nekonečný proud dat.

Podobně funguje i metoda `Stream.generate(...)`, kde se nepopisuje návaznost mezi hodnotami, ale pouze způsob, jak vytvořit každou další hodnotu:

```java
List<Double> result = Stream.generate(() -> Math.random())
    .limit(3)
    .toList();
```

Zde dostaneme několik náhodných čísel. Opět platí, že bez omezení by stream generoval hodnoty neustále.

Ve všech uvedených případech je důležité si uvědomit společný princip: stream vždy vzniká z nějakého zdroje, nad kterým následně definujeme operace. Ať už jde o kolekci, pole nebo generátor, způsob dalšího zpracování zůstává stejný. Díky tomu si vystačíme s několika základními způsoby vytvoření streamu, které pokryjí drtivou většinu praktických situací.

## Bonus - parallení proudy

{% hint style="info" %}
Tato část je zde zmíněna pouze pro úplnost pro zvídavé studenty.
{% endhint %}

Jako zajímavost stojí za zmínku, že kolekce v Javě kromě metody `stream()` nabízí také metodu `parallelStream()`. Ta vytváří stream, který se snaží zpracovávat data **paralelně ve více vláknech**.

Z pohledu zápisu kódu je použití velmi jednoduché — stačí změnit způsob vytvoření streamu:

```java
List<String> result = users.parallelStream()
    .filter(u -> u.isActive())
    .map(u -> u.getEmail())
    .toList();
```

Rozdíl oproti běžnému streamu není v tom, **co kód dělá**, ale **jakým způsobem to probíhá na pozadí**. Místo sekvenčního zpracování se operace rozdělí na více částí, které mohou běžet současně.

Je však důležité si uvědomit, že paralelní streamy nejsou univerzální řešení:

* jejich přínos závisí na typu operace a velikosti dat,
* mohou být méně předvídatelné z hlediska výkonu,
* je nutné se vyhnout práci se sdíleným stavem (side effects).

Proto se v běžné praxi používají opatrně a až ve chvíli, kdy máme důvod řešit výkon. Pro pochopení a běžné používání Stream API je vhodné nejprve dobře zvládnout sekvenční streamy, které se chovají jednodušeji a předvídatelněji.
