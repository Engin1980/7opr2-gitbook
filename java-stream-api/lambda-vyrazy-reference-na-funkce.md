---
icon: square-0
---

# Lambda výrazy, Reference na funkce

## Lambda výraz

Lambda výrazy představují jeden z nejdůležitějších kroků, který Java udělala směrem k modernějšímu stylu programování. Poprvé se objevily ve verzi Java 8 a jejich hlavním cílem bylo zjednodušit zápis kódu, ve kterém předáváme nějaké chování jako parametr – tedy typicky funkci.

Dříve bylo v Javě běžné, že když jsme chtěli předat nějakou logiku, museli jsme vytvořit samostatnou třídu nebo alespoň anonymní implementaci rozhraní. Takový kód byl často zbytečně dlouhý a složitý, i když samotná logika byla triviální. Lambda výrazy tohle výrazně zjednodušují — umožňují zapsat funkci přímo na místě, kde ji potřebujeme.

Z pohledu čtení kódu je klíčové pochopit, že lambda výraz je prostě krátký zápis funkce. Má vstup (parametry) a výstup (výraz nebo blok kódu), a zapisuje se pomocí šipky `->`. Typický tvar vypadá například takto:

```java
n -> n * 2
```

Tento zápis říká: vezmi hodnotu `n` a vrať její dvojnásobek. Neřešíme žádné typy, žádnou třídu ani metodu — všechno je odvozeno z kontextu.

Takové výrazy se vyskytují v různých variantách. Některé lambda výrazy nemají žádný vstup, typicky pokud chceme něco pouze vytvořit:

```java
() -> Math.random()
```

Jiné berou jeden parametr:

```java
name -> name.toUpperCase()
```

Jiné mohou brát více parametrů:

```java
// Součet čísel
(a, b) -> a + b

// Složení jména a příjmení dohromady do výsledného řetězce
(name, surname) -> name + " " + surname

// Výpočet přepony pravoúhlého trojúhelníku
(a, b) -> Math.sqrt(a * a + b * b)

// Výpočet diskriminantu kvadratické rovnice
(a, b, c) -> b * b - 4 * a * c
```

Tento zápis říká: vezmi vstupní hodnoty a transformuj je do **jedné** výstupní hodnoty, kterou vrať.

Většina lambda výrazů je krátká a skládá se jen z jednoho výrazu. Pokud ale potřebujeme více kroků, můžeme použít blok:

```java
n -> {
    int result = n * 2;
    return result;
}
```

Tady už se to začíná podobat klasické metodě, jen bez její „obálky“.

Složitější blok tedy může vypadat takto:

```java
(a, b, c) -> {
    double d = b * b - 4 * a * c;
    double x1 = (-b + Math.sqrt(d)) / (2 * a);
    double x2 = (-b - Math.sqrt(d)) / (2 * a);
    return List.of(x1, x2);
}
```

Velmi důležité je, že v žádném z těchto příkladů nevidíme typy proměnných. Přesto Java přesně ví, co děláme. Důvodem je, že typ lambda výrazu se vždy odvozuje z kontextu, ve kterém je použitý. Jinými slovy, lambda výraz sám o sobě nemá pevně daný typ — ten získává až podle toho, kam ho předáme. Kompilátor Java sám z kontextu téměř vždy pozná, jaké typy má očekávat a zda má daný výraz smysl.

Přesto, pokud chceme (nebo musíme), lze typ zapsat ručně:

```java
(User u) -> u.getEmail()
```

ale v praxi se to téměř nepoužívá, protože to zbytečně prodlužuje kód a nepřináší novou informaci. Jednou z hlavních výhod lambda výrazů je právě to, že typy většinou psát nemusíme.

Pro praktické použití je nejdůležitější způsob uvažování: lambda výraz bereme jako krátký zápis chování. Neřešíme jeho typ ani implementaci, ale pouze to, co má udělat. Díky tomu můžeme zapisovat operace nad daty velmi přirozeně — například ve streamech přímo říkáme, jak má každý prvek projít filtrem nebo jak se má transformovat, aniž bychom museli vytvářet jakékoliv pomocné struktury.

## Funkcionální rozhraní

Funkcionální rozhraní je speciální typ rozhraní v Javě, který má **právě jednu abstraktní metodu**. To je jeho klíčová vlastnost – díky tomu může být takové rozhraní použito jako „typ funkce“, tedy něco, co můžeme implementovat pomocí lambda výrazu.

Nejde tedy o nic nového nebo zvláštního – je to obyčejné rozhraní, jen s jedním pravidlem: smí obsahovat právě jednu metodu, kterou musí implementovat třída (nebo lambda). Může mít ale libovolný počet výchozích (`default`) nebo statických metod.

```java
interface MathOperation {
    int apply(int a, int b);
}
```

Toto rozhraní má jednu metodu `apply`, která bere dvě čísla a vrací výsledek. Právě proto je to funkcionální rozhraní.

Dříve bychom ho použili takto:

```java
MathOperation addition = new MathOperation() {
    @Override
    public int apply(int a, int b) {
        return a + b;
    }
};

int result = addition.apply(3, 5);
```

Tento zápis je funkční, ale poměrně dlouhý vzhledem k tomu, jak jednoduchou operaci řešíme.

Díky tomu, že `MathOperation` má jen jednu metodu, můžeme ho implementovat lambda výrazem:

```java
MathOperation addition = (a, b) -> a + b;

int result = addition.apply(3, 5);
```

Lambda `(a, b) -> a + b` zde představuje implementaci metody `apply`.

Funkcionální rozhraní nemusí být jen pro matematiku:

```java
interface StringFormatter {
    String format(String input);
}
```

Na základě tohoto rozhraní můžeme velmi jednoduše implementovat několik různých způsobů formátování textu:

```java
StringFormatter upper = s -> s.toUpperCase();
StringFormatter prefix = s -> ">> " + s;

System.out.println(upper.format("hello"));
System.out.println(prefix.format("hello"));
```

První varianta, `upper`, převádí text na velká písmena. Druhá varianta, `prefix`, každému textu dá prefix `>>`.

## Vztah lambda vs funkcionální rozhraní

Důležité je pochopit, že lambda výraz **nemůže existovat samostatně**. Vždy musí být přiřazen k nějakému typu – a tím typem je právě funkcionální rozhraní.

Například výraz `(a, b) -> a + b` sám o sobně nic neznamená. Teprve, když se přiřadí do kontextu:

```java
MathOperation op = (a, b) -> a + b;
```

Java ví, že:

* `a` a `b` jsou `int`
* návratová hodnota je `int`

Typ se tedy vždy odvozuje z kontextu (z rozhraní, do kterého lambdu přiřazujeme).

{% hint style="info" %}
Každý lambda výraz se **vždy** přiřazuje do nějakého funkcionálního rozhraní, aby kompilátor Java byl schopen zkontrolovat správnost. Děje se tak i tehdy, když je lambda použta přímo (například jako parametr funkce); v tu chvíli se odpovídající rozhraní rozpoznává automaticky z kontextu.
{% endhint %}

### Anotace @FunctionalInterface

V praxi se často používá anotace:

```java
@FunctionalInterface
interface MathOperation {
    int apply(int a, int b);
}
```

Tato anotace není povinná, ale má výhodu:

* kompilátor zkontroluje, že rozhraní má opravdu jen jednu abstraktní metodu
* chrání nás před chybou (např. nechtěným přidáním další metody)

### Předdefinovaná funkcionální rozhraní

V praxi velmi často vůbec nemusíme definovat vlastní rozhraní, protože Java už nabízí sadu univerzálních typů, které pokrývají nejběžnější situace. Díky tomu se výrazně zjednodušuje práce — místo vytváření nových rozhraní jen vybíráme vhodný typ podle toho, jakou „funkci“ chceme reprezentovat.

Nejčastěji se využívají 4 základní rozhraní:

* `Function<T, R>`: vstup → výstup
* `Predicate<T>`: vstup → boolean
* `Supplier<T>`: bez vstupu → výstup
* `Consumer<T>`: vstup → bez návratové hodnoty

Tato čtyři rozhraní pokrývají nejběžnější situace, kdy chceme v Javě pracovat s „funkcí“, tedy s nějakým chováním, které přijímá vstup a případně vrací výstup. Každé z nich reprezentuje trochu jiný typ operace a je dobré si je představit nejen jako názvy, ale jako konkrétní způsoby práce s daty.

Když se na tyto typy podíváme společně, začnou dávat smysl jako základní „stavebnice“:

* někdy chceme hodnotu přeměnit na něco jiného (`Function`),
* někdy jen ověřit podmínku (`Predicate`),
* někdy hodnotu vytvořit (`Supplier`),
* někdy s hodnotou něco udělat bez návratu (`Consumer`).

Právě tyto čtyři případy pokrývají většinu situací, se kterými se při práci s moderní Javou setkáme. Jakmile si na ně zvykneme, začneme přirozeně rozpoznávat, jaký typ funkce je v konkrétním místě potřeba, a lambda výrazy se pro nás stanou běžným a velmi čitelným nástrojem.

#### Funkce X -> Y

Rozhraní `Function<T, R>` reprezentuje klasickou transformaci: máme vstup jedné hodnoty a z něj chceme vytvořit hodnotu jiného typu. Typ `T` označuje vstupní typ, `R` návratový. Je to nejuniverzálnější případ — například převod objektu na jeho vlastnost, výpočet nějaké hodnoty nebo změna formátu dat. Když napíšeme:

```java
Function<String, Integer> length = s -> s.length();
```

říkáme tím: vstupem je `String` a výsledkem je `Integer`.&#x20;

Používá se všude tam, kde „převádíme“ data z jedné podoby na jinou. Typická použití jsou tedy například získání jména zákazníka z jeho objektu, získání čísel faktur, získání studijních čísel studentů, získání e-mailů osob.

#### Predikát X -> bool

Rozhraní `Predicate<T>` je jednodušší a specializované na situace, kdy nás zajímá pouze **podmínka**. Vstupem je nějaká hodnota a výstupem je vždy `boolean` — tedy odpověď na otázku „platí / neplatí“. Typicky ho používáme pro filtrování nebo validaci. Například:

```java
Predicate<Integer> isPositive = x -> x > 0;
```

Tato funkce neprodukuje žádnou „novou“ hodnotu, jen rozhoduje, jestli daný prvek splňuje určité kritérium.

Používá se typicky v podmínkách a filtrech - vyber objekty, které splňují nebo nesplňují určitou podmínku, např. osoby z určité lokace, minimální váhy, faktury, které nemají žádnou položku, studenty bez známek atp.

#### Dodavatel () -> X

Rozhraní `Supplier<T>` reprezentuje specifický případ `Function`. Nemá žádný vstup, ale vrací hodnotu. Můžeme si ho představit jako „zdroj dat“ nebo „generátor“. Funkce se zavolá a vrátí nějaký výsledek, aniž by k tomu potřebovala parametr. Typickým příkladem je generování náhodné hodnoty nebo vytvoření nového objektu:

```java
Supplier<Double> random = () -> Math.random();
```

Každé zavolání této funkce vrátí novou hodnotu, ale nemáme možnost ji přímo ovlivnit vstupem.&#x20;

Používá se, pokud potřebujeme generovat nějaké hodnoty (náhodná čísla, nové objekty atp.)

#### Konzument X -> ()

Naopak rozhraní `Consumer<T>` představuje situaci, kdy máme vstup, ale **nezajímá nás návratová hodnota**. Funkce něco provede (například zapíše výstup, upraví data, odešle zprávu), ale nic nevrací. Typickým příkladem je operace, která má vedlejší efekt:

```java
Consumer<String> printer = s -> System.out.println(s);
```

Zde vstupem je `String`, ale výstup žádný není — důležitá je samotná akce.

Používá se typicky, když data nepotřebujeme předávat dál, ale chceme je nějak zpracovat - například vypsat na konzoli, uložit do souboru, poslat e-mailem atp.

## Reference na metody

Method reference můžeme chápat jako další zjednodušení práce s lambda výrazy. Zatímco lambda nám umožňuje zapsat malou funkci přímo na místě, method reference jde ještě o krok dál — umožňuje nám vyjádřit situaci, kdy žádnou novou logiku vlastně nevytváříme, ale pouze **odkazujeme na už existující metodu**.

Typická situace vypadá tak, že lambda výraz jen „obaluje“ volání metody. Například zápis

```java
s -> s.toUpperCase()
```

neobsahuje žádnou vlastní logiku, pouze říká: vezmi vstup a zavolej na něm metodu `toUpperCase`. V takovém případě je možné použít kratší zápis

```java
String::toUpperCase
```

který vyjadřuje přesně totéž. Rozdíl tedy není ve funkčnosti, ale čistě ve formě zápisu.

{% hint style="info" %}
Zápis `String::toUpperCase` vlastně říká "aplikuj na vstup funkci \`toUpperCase\` z typu `String`".  Všimněte si, že bez kontextu lambda funkce nemusí mít význam smysl.
{% endhint %}

Důležité je pochopit, že method reference vždy stojí na stejném principu jako lambda — musí odpovídat nějakému funkcionálnímu rozhraní. Java z kontextu pozná, jaké parametry metoda má a jaký typ vrací, a podle toho ji „přizpůsobí“. V tomhle směru se method reference chová úplně stejně jako lambda výraz, jen je kompaktnější.

Poměrně rychle si lze všimnout, že tento přístup výrazně zpřehledňuje kód ve chvíli, kdy pracujeme s jednoduchými transformacemi. Například místo zápisu `s -> s.length()` můžeme psát `String::length`, místo `n -> Math.sqrt(n)` pak `Math::sqrt` a místo `x -> System.out.println(x)` jednoduše `System.out::println`. V každém z těchto případů platí, že lambda nepřidává žádnou logiku navíc, pouze deleguje volání na jinou metodu, takže její zkrácení dává smysl.

Další zajímavou variantou je použití u konstruktorů. Pokud máme lambda výraz, který jen vytváří nový objekt, například `() -> new ArrayList<>()`, můžeme ho nahradit zápisem `ArrayList::new`. I zde jde pouze o jinou syntaxi, která ale často působí přehledněji a elegantněji.

Je ale dobré zdůraznit, že method reference není vždy lepší volba. Pokud lambda obsahuje i drobnou logiku navíc, například `s -> s.trim().toUpperCase()`, už ji nelze jednoduše nahradit, protože nejde o jedno jediné volání metody. V takových případech je klasická lambda výrazně srozumitelnější. Obecně tedy platí jednoduché pravidlo: method reference použijeme tehdy, když lambda pouze volá existující metodu a nic dalšího nepřidává.

Syntaxe metod reference je trošku složitěji čitelnější bez zvyku. Je však třeba se ji naučit, protože v Javě se běžně používá. Ve výsledku nejde o novou funkci jazyka v pravém smyslu, ale spíše o syntaktickou zkratku, která pomáhá psát kód stručněji a přehledněji, zejména v kombinaci s lambda výrazy a funkcionálními rozhraními.
