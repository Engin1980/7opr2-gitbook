---
icon: square-4
---

# JSA - Ukončovací operace streamů

Doposud jsme pracovali s operacemi jako `filter`, `map` nebo `flatMap`. Tyto operace nazýváme mezilehlé (intermediate), protože pouze popisují transformaci dat. Samy o sobě ještě nespustí žádné zpracování – definují jen jak má pipeline vypadat.

Teprve ukončovací operace (terminal operations) způsobí, že se celý stream opravdu vyhodnotí a provede.

Je důležité si uvědomit, že stream je ze své podstaty „líný“ (lazy). To znamená, že **dokud není zavolána ukončovací operace, nic se nepočítá**. Celá pipeline existuje jen jako definice kroků. Ve chvíli, kdy zavoláme například `forEach`, `collect` nebo `reduce`, dojde k průchodu dat a jednotlivé operace se aplikují.

Právě ukončovací operace tedy představují bod, kdy:

* získáváme konkrétní výsledek (např. číslo, kolekci, objekt),
* nebo provádíme vedlejší efekt (např. výpis na konzoli).

Po provedení ukončovací operace už stream nelze znovu použít. Každý stream lze projít pouze jednou. Pokud bychom se pokusili použít ho znovu, dostaneme chybu typu `IllegalStateException`. To vychází z toho, že stream není datová struktura, ale jednorázový tok dat.

Ukončovací operace také určují, jaký bude výsledný typ celé pipeline. Například:

* `forEach` nic nevrací, pouze provede akci nad každým prvkem,
* `toList()` nebo `collect(...)` vrací kolekci,
* `min`, `max` vrací jeden prvek (zabalený v `Optional`),
* `reduce` umožňuje vytvořit agregaci (např. součet).

Z pohledu návrhu aplikace mají ukončovací operace zásadní význam. Rozhodují o tom, zda pipeline:

* skončí jako datová transformace do nové kolekce,
* nebo jako výpočet jedné hodnoty,
* případně jako čistě procedurální operace (např. logování).

Typickým cílem práce se Stream API je navrhnout pipeline tak, aby byla:

* čitelná (každý krok má jasný význam),
* efektivní (zpracovávají se jen potřebná data),
* a zakončená vhodnou ukončovací operací, která odpovídá požadovanému výsledku.

V následujících částech se podrobně podíváme na konkrétní ukončovací operace, jako jsou `forEach`, `toList`, `collect`, `min`, `max` nebo obecný mechanismus `reduce`, které tvoří základ pro agregace i převod výsledků do finální podoby.

## Kolektory

Operace `collect(...)` představuje obecný mechanismus, jak převést stream na konkrétní výsledek. Zatímco jednodušší ukončovací operace vrací jeden prvek nebo provedou nějakou akci, `collect` umožňuje „nasbírat“ všechny prvky do datové struktury. Právě proto patří mezi nejčastěji používané operace ve Stream API.

Základní myšlenka je taková, že stream popisuje tok dat a jejich transformaci, zatímco collector určuje, jak se mají výsledky ukládat. Tato logika je zapouzdřená v třídě `Collectors`, která obsahuje hotové implementace pro běžné situace. Programátor tak nemusí psát vlastní cykly ani ručně naplňovat kolekce.

### Zabalení do List

Nejjednodušší variantou je převod streamu na seznam. I když moderní Java nabízí metodu `toList()`, použití `collect(Collectors.toList())` je obecnější a dobře ukazuje princip collectorů.

```java
import java.util.List;
import java.util.stream.Collectors;

public class Main {
    public static void main(String[] args) {
        List<String> slova = List.of("java", "stream", "api");

        List<String> vysledek = slova.stream()
                                     .map(String::toUpperCase)
                                     .collect(Collectors.toList());

        System.out.println(vysledek);
    }
}
```

Každý prvek streamu je nejprve transformován metodou `map` a následně uložen do nového seznamu. Původní kolekce zůstává nezměněná, protože stream vytváří nová data.

Jak bylo nicméně již zmíněno, protože získání dat do `List` je jednou z nejčastějších operací, byla pro ni vytvořena a lze využít zkrácenou variantu `toList()`.

### Zabalení do množiny

Velmi podobně lze vytvořit množinu (`Set`). Rozdíl spočívá v tom, že množina neobsahuje duplicity. To znamená, že pokud se ve streamu opakuje více stejných hodnot, ve výsledku budou jen jednou.

```java
import java.util.List;
import java.util.Set;
import java.util.stream.Collectors;

public class Main {
    public static void main(String[] args) {
        List<String> slova = List.of("java", "stream", "java", "api");

        Set<String> vysledek = slova.stream()
                                    .collect(Collectors.toSet());

        System.out.println(vysledek);
    }
}
```

Takový zápis je velmi užitečný například při odstraňování duplicitních hodnot. Není potřeba psát žádnou speciální logiku, stačí změnit collector.

### Zabalení do mapy

Zajímavější situace nastává při práci s mapou (`Map`), tedy se strukturou klíč–hodnota. V tomto případě musíme definovat, co bude klíčem a co hodnotou. To se provádí pomocí metody `Collectors.toMap`, která přijímá dvě funkce.

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
            new Osoba("Karel", 30)
        );

        Map<String, Integer> mapa = osoby.stream()
            .collect(Collectors.toMap(
                o -> o.jmeno,
                o -> o.vek
            ));

        System.out.println(mapa);
    }
}
```

Každý objekt `Osoba` se zde převede na jeden záznam v mapě. Klíčem je jméno a hodnotou věk. Výsledkem je přehledná struktura, která umožňuje rychlé vyhledávání podle klíče.

Je dobré si uvědomit, že klíče musí být unikátní. Pokud by se ve streamu objevily dva prvky se stejným klíčem, došlo by k výjimce. V praxi se tento problém řeší doplněním tzv. merge funkce, která určuje, jak se mají hodnoty sloučit. Pro základní použití ale stačí vědět, že unikátnost klíčů je nutná podmínka.

#### Modifikovatelnost výsledku

Je dobré zmínit ještě jednu důležitou vlastnost při práci s kolekcemi, které vznikají ze streamu. Metoda `toList()`, která byla přidána v novějších verzích Javy (Java 16+), vrací **neměnný (unmodifiable) seznam**. To znamená, že do něj nelze přidávat, odebírat ani měnit prvky – pokus o takovou operaci skončí výjimkou `UnsupportedOperationException`.

```java
var list = List.of(1, 2, 3).stream().toList();
list.add(4); // chyba za běhu
```

Tato vlastnost je záměrná. Neměnné kolekce zvyšují bezpečnost programu, protože zaručují, že se data po vytvoření nebudou měnit, což je výhodné například v paralelním zpracování nebo při předávání dat mezi částmi systému.

U collectorů je situace trochu odlišná. Například `Collectors.toList()` historicky vrací běžný modifikovatelný `List`, ale specifikace to výslovně negarantuje – říká jen, že výsledek je nějaký seznam. V praxi se sice většinou jedná o `ArrayList`, ale neměli bychom na tom záviset.

Pokud chceme explicitně získat neměnný seznam pomocí collectoru, existuje na to speciální varianta:

```java
import java.util.List;
import java.util.stream.Collectors;
List<Integer> list = List.of(1, 2, 3).stream()
    .collect(Collectors.toUnmodifiableList());
```

Podobně existují i neměnné varianty pro další kolekce, například `toUnmodifiableSet()` nebo `toUnmodifiableMap()`.

Shrnutí je tedy následující: moderní metoda `toList()` vždy vrací neměnnou kolekci, zatímco klasické collectory (`toList`, `toSet`, `toMap`) obvykle vrací kolekce modifikovatelné, ale bez pevné garance. Pokud je pro návrh aplikace důležitá neměnnost, je vhodné ji použít explicitně.

### Shrnutí

Použití `collect(...)` se téměř vždy objevuje na konci streamové pipeline. Nejprve data filtrujeme, potom transformujeme a nakonec uložíme do výsledné struktury. Tento styl odpovídá přirozenému způsobu uvažování o datech.

```java
import java.util.List;
import java.util.stream.Collectors;

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

        List<String> vysledek = produkty.stream()
            .filter(p -> p.cena > 1000)
            .map(p -> p.nazev)
            .collect(Collectors.toList());

        System.out.println(vysledek);
    }
}
```

Celá pipeline je čitelná jako posloupnost kroků: vyber relevantní data, uprav je a ulož. Není potřeba psát cykly, mezivýsledky ani ručně spravovat kolekce.

Collectory tak představují důležitou součást Stream API, protože propojují deklarativní zpracování dat s konkrétním výsledkem. Umožňují vytvářet různé datové struktury bez zbytečné boilerplate logiky a zároveň zachovat kód přehledný a snadno rozšiřitelný.

## ForEach

Operace `forEach` je nejjednodušší ukončovací operací ve Stream API. Jejím účelem není vytvořit nový výsledek (například kolekci nebo číslo), ale **provést nějakou akci nad každým prvkem streamu**. Typicky jde o vedlejší efekt, jako je výpis na konzoli, logování nebo ukládání do externí struktury.

Na rozdíl od `collect` nebo `reduce` tedy `forEach` nic nevrací. Pouze projde všechny prvky streamu a pro každý z nich zavolá zadanou funkci.

Nejčastější použití je prostý výpis dat:

```java
import java.util.List;

public class Main {
    public static void main(String[] args) {
        List<String> slova = List.of("Java", "Stream", "API");

        slova.stream()
             .forEach(System.out::println);
    }
}
```

Každý prvek streamu se předá metodě `println`. Tento zápis je velmi stručný a dobře čitelný.

`forEach` se často používá na konci pipeline, kde už jsme data předtím vyfiltrovali nebo transformovali. Například chceme vypsat jen delší slova a ještě je převést na velká písmena:

```java
slova.stream()
     .filter(s -> s.length() > 4)
     .map(String::toUpperCase)
     .forEach(System.out::println);
```

Pipeline zde jasně ukazuje postup: výběr dat, transformace a nakonec jejich „spotřebování“ pomocí `forEach`.

Je důležité si uvědomit, že `forEach` se hodí primárně pro operace se **side efektem**. Není určený pro tvorbu výsledných kolekcí nebo agregací. I když bychom technicky mohli pomocí `forEach` naplňovat například seznam, není to doporučený způsob, protože to odporuje stylu Stream API.

{% hint style="info" %}
Side effect funkce je taková funkce, která kromě návratové hodnoty ještě mění stav systému nebo pracuje s vnějším prostředím (například zapisuje do proměnné, souboru nebo na konzoli). Na rozdíl od čistých funkcí tedy její výsledek nezávisí jen na vstupu a **může mít dopady mimo vlastní výpočet**.
{% endhint %}

Takový zápis sice funguje:

```java
import java.util.ArrayList;
import java.util.List;

public class Main {
    public static void main(String[] args) {
        List<Integer> cisla = List.of(1, 2, 3, 4);

        List<Integer> vysledek = new ArrayList<>();

        cisla.stream()
             .map(n -> n * 2)
             .forEach(vysledek::add);

        System.out.println(vysledek);
    }
}
```

ale není považován za ideální, protože místo deklarativního přístupu opět pracujeme s modifikovatelnou strukturou a vedlejšími efekty. Správný způsob by byl použít `collect`.

Na druhou stranu existují situace, kdy je `forEach` přirozenou volbou, například při komunikaci s externím světem. Můžeme například zapisovat do logu nebo volat nějakou službu:

```java
cisla.stream()
     .filter(n -> n % 2 == 0)
     .forEach(n -> System.out.println("Zpracovávám: " + n));
```

Zde už nedává smysl žádný návratový výsledek – cílem je samotná akce.

Je také dobré vědět, že existuje varianta `forEachOrdered`, která se používá zejména u paralelních streamů. Ta zaručuje zachování pořadí prvků, i když se stream zpracovává paralelně. U běžného sekvenčního streamu není rozdíl viditelný.

Operace `forEach` tak představuje typický koncový bod pipeline ve chvíli, kdy nechceme data dál zpracovávat, ale pouze nad nimi provést nějakou akci. V kombinaci s `filter` a `map` umožňuje velmi přehledně zapsat logiku od výběru dat až po jejich finální použití.

## Reduktory

Vedle operací, které převádějí stream na kolekce (`collect`), existuje další důležitá skupina ukončovacích operací, jejichž cílem je získat **jednu výslednou hodnotu**. Tyto operace se označují jako redukční, protože postupně „redukují“ celý stream na jeden výsledek.

### Min/Max

Nejjednoduššími zástupci jsou metody `min` a `max`. Jejich účelem je najít nejmenší, resp. největší prvek podle zadaného porovnání. Stream sám o sobě neví, jak prvky porovnávat, proto musíme předat tzv. komparátor.

Příklad se seznamem čísel:

```java
import java.util.List;
import java.util.Comparator;

public class Main {
    public static void main(String[] args) {
        List<Integer> cisla = List.of(5, 2, 9, 1, 7);

        var min = cisla.stream().min(Comparator.naturalOrder());
        var max = cisla.stream().max(Comparator.naturalOrder());

        min.ifPresent(System.out::println);
        max.ifPresent(System.out::println);
    }
}
```

Výsledek je typu `Optional`, protože stream může být prázdný. V takovém případě neexistuje žádný minimální ani maximální prvek. Tento návrh nutí programátora ošetřit situaci, kdy data chybí.

Při práci s objekty si typicky určujeme vlastní kritérium. Například chceme najít nejdražší produkt:

```java
import java.util.List;
import java.util.Comparator;

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

        var nejdrazsi = produkty.stream()
            .max(Comparator.comparing(p -> p.cena));

        nejdrazsi.ifPresent(p -> System.out.println(p.nazev));
    }
}
```

Metody `min` a `max` jsou ve skutečnosti speciálním případem obecnější operace, kterou představuje `reduce`.

### Reduce

Operace `reduce` umožňuje definovat vlastní způsob, jak postupně kombinovat prvky streamu do jedné hodnoty. Funguje na principu akumulátoru: vezme dva prvky, spojí je do jednoho výsledku a ten pak použije v dalším kroku.

Nejjednodušší příklad je výpočet součtu:

```java
import java.util.List;

public class Main {
    public static void main(String[] args) {
        List<Integer> cisla = List.of(1, 2, 3, 4, 5);

        int soucet = cisla.stream()
            .reduce(0, (a, b) -> a + b);

        System.out.println(soucet);
    }
}
```

První parametr (`0`) je tzv. identita, výchozí hodnota. Druhý parametr je funkce, která popisuje, jak se mají dvě hodnoty zkombinovat. V tomto případě jde o sčítání.

Operace `reduce` je velmi flexibilní. Můžeme ji použít nejen pro čísla, ale i pro řetězce nebo složitější objekty. Například spojení všech slov do jedné věty:

```java
import java.util.List;

public class Main {
    public static void main(String[] args) {
        List<String> slova = List.of("Java", "Stream", "API");

        String veta = slova.stream()
            .reduce("", (a, b) -> a + " " + b);

        System.out.println(veta);
    }
}
```

V tomto případě postupně spojujeme jednotlivé řetězce.

Existuje i varianta `reduce` bez identity, která vrací `Optional`. Ta se používá tehdy, když nemáme přirozenou výchozí hodnotu:

```java
var vysledek = cisla.stream()
    .reduce((a, b) -> a + b);

vysledek.ifPresent(System.out::println);
```

Z hlediska návrhu pipeline mají redukční operace jasný význam: používáme je ve chvíli, kdy nás nezajímá kolekce výsledků, ale jedna agregovaná hodnota. Typickými příklady jsou součet, minimum, maximum, průměr nebo spojení textu.

Je také dobré zmínit, že některé běžné agregace mají své specializované metody nebo collectory, které jsou čitelnější než obecný `reduce`. Například `count`, `summaryStatistics` nebo `Collectors.summingInt`. Přesto je `reduce` důležitý, protože poskytuje obecný mechanismus, na kterém jsou tyto operace založeny.

### Průměr & funkce mapTo...()&#x20;

Při práci s agregacemi bychom teoreticky mohli všechno zapisovat pomocí operace `reduce`. Ta je velmi obecná a umožňuje vyjádřit libovolný výpočet. V praxi ale bývá pro některé úlohy zbytečně složitá a hůře čitelná, zejména pokud pracujeme s čísly.

Typickým příkladem je výpočet průměru. Pomocí `reduce` bychom museli současně počítat součet i počet prvků, což vede ke komplikovanějšímu kódu, který špatně vyjadřuje samotný záměr.

Proto Stream API nabízí speciální operace `mapToInt`, `mapToDouble` a `mapToLong`. Ty převádějí běžný stream na tzv. numerický stream (`IntStream`, `DoubleStream`, `LongStream`), nad kterým už můžeme velmi jednoduše používat hotové agregační metody.

Typický příklad je právě průměr:

```java
import java.util.List;

public class Main {
    public static void main(String[] args) {
        List<Integer> cisla = List.of(10, 20, 30, 40);

        double prumer = cisla.stream()
            .mapToInt(Integer::intValue)
            .average()
            .orElse(0);

        System.out.println(prumer);
    }
}
```

Zápis přímo říká, co děláme: převedeme hodnoty na čísla a spočítáme jejich průměr. Není potřeba řešit žádnou vlastní logiku sčítání a počítání prvků.

Stejný princip funguje i pro objekty. Pokud máme například seznam produktů, můžeme jednoduše spočítat průměrnou cenu:

```java
double prumer = produkty.stream()
    .mapToDouble(p -> p.cena)
    .average()
    .orElse(0);
```

Numerické streamy kromě `average()` nabízejí také další přehledné operace, například `sum()`, `min()` nebo `max()`. Díky tomu lze většinu běžných výpočtů zapsat jednoduše a čitelně, bez použití obecného `reduce`.

```java
import java.util.List;

public class Main {
    public static void main(String[] args) {
        List<Integer> cisla = List.of(1, 2, 3, 4, 5);

        int soucet = cisla.stream()
            .mapToInt(Integer::intValue)
            .sum();

        System.out.println(soucet);
    }
}
```

Hlavní výhoda tedy spočívá v tom, že místo univerzální, ale složitější operace máme specializované nástroje, které přesně odpovídají konkrétnímu problému. Kód je díky tomu kratší, přehlednější a lépe vystihuje záměr programu.

## Příznakové operace (any/all/none)

Stream API neumožňuje jen transformovat data nebo je převádět do kolekcí, ale také nad nimi jednoduše pokládat tzv. boolean dotazy. Tyto operace odpovídají situacím, kdy nás zajímá, zda kolekce splňuje určitou podmínku. Výsledkem je vždy logická hodnota (`true` nebo `false`).

Základ tvoří tři metody: `anyMatch`, `allMatch` a `noneMatch`. Všechny pracují s predikátem, tedy funkcí, která pro každý prvek vrací pravdu nebo nepravdu, podobně jako u `filter`.

Metoda `anyMatch` zjišťuje, zda existuje alespoň jeden prvek, který splňuje podmínku. Jakmile takový prvek najde, stream se dál neprochází a operace okamžitě končí. To je důležité z hlediska výkonu.

```java
import java.util.List;

public class Main {
    public static void main(String[] args) {
        List<Integer> cisla = List.of(1, 3, 5, 8);

        boolean existujeSude = cisla.stream()
            .anyMatch(n -> n % 2 == 0);

        System.out.println(existujeSude);
    }
}
```

Výsledek bude `true`, protože ve streamu existuje číslo 8, které splňuje podmínku.

Metoda `allMatch` naopak ověřuje, zda podmínku splňují všechny prvky. Pokud narazí na prvek, který ji nesplňuje, okamžitě vrátí `false`.

```java
boolean vsechnaKladna = cisla.stream()
    .allMatch(n -> n > 0);

System.out.println(vsechnaKladna);
```

Tento dotaz ověřuje, zda jsou všechna čísla kladná. Jakmile by se objevil první záporný prvek, výsledek by byl `false`.

Metoda `noneMatch` je negací `anyMatch`. Ověřuje, že žádný prvek nesplňuje danou podmínku.

```java
boolean zadneZaporne = cisla.stream()
    .noneMatch(n -> n < 0);

System.out.println(zadneZaporne);
```

Z pohledu čitelnosti je `noneMatch` často lepší než psát negaci nad `anyMatch`, protože přímo vyjadřuje záměr: „neexistuje žádný takový prvek“.

Tyto operace se velmi často používají při validaci dat. Můžeme například ověřit, že seznam obsahuje pouze platné hodnoty, že žádný prvek není `null` nebo že alespoň jeden prvek splňuje důležitou podmínku.

Příklad validace vstupních dat:

```java
import java.util.List;

public class Main {
    public static void main(String[] args) {
        List<String> jmena = List.of("Jan", "Petra", "Karel");

        boolean bezNull = jmena.stream()
            .noneMatch(s -> s == null);

        boolean vseDlouhe = jmena.stream()
            .allMatch(s -> s.length() >= 3);

        System.out.println(bezNull);
        System.out.println(vseDlouhe);
    }
}
```

Význam těchto operací je v tom, že umožňují formulovat podmínky nad kolekcemi deklarativně a velmi čitelně. Místo ručního procházení kolekce a práce s proměnnými typu „nalezeno / nenalezeno“ můžeme přímo vyjádřit logický dotaz.

Pro praktické použití si lze představit typický scénář: máme kolekci dat a chceme ověřit nějakou vlastnost. Například zda všechny objednávky mají kladnou cenu:

```java
boolean validni = objednavky.stream()
    .allMatch(o -> o.cena > 0);
```

Takový zápis je krátký, ale velmi přesně vystihuje záměr. Právě proto patří `anyMatch`, `allMatch` a `noneMatch` mezi důležité nástroje pro kontrolu podmínek nad kolekcemi v rámci Stream API.

## Vyhledávací operace (find...)

Při práci se streamy často narazíme na situaci, kdy nechceme zpracovávat všechny prvky, ale potřebujeme najít jeden konkrétní. Typicky chceme odpovědět na otázku „existuje nějaký prvek, který splňuje podmínku, a pokud ano, který to je“. Právě pro tyto případy slouží operace `findFirst` a `findAny`.

Obě metody jsou ukončovací operace a jejich výsledkem není přímo prvek, ale `Optional<T>`. To znamená, že výsledek může existovat i nemusí, a je nutné s tím počítat.

Nejčastější scénář je kombinace s filtrem. Například máme seznam uživatelů a chceme najít prvního, který je administrátor:

```java
var uzivatel = uzivatele.stream()
    .filter(u -> u.admin)
    .findFirst();
```

Pipeline se čte velmi přirozeně: vezmi stream, vyfiltruj administrátory a najdi prvního z nich.

Metoda `findFirst` vrací první prvek podle pořadí ve streamu. U běžného (sekvenčního) streamu to znamená první prvek v kolekci. Je vhodná tehdy, když má pořadí význam, například když pracujeme se seřazenými daty nebo chceme deterministický výsledek.

```java
import java.util.List;

public class Main {
    public static void main(String[] args) {
        List<Integer> cisla = List.of(10, 20, 30, 40);

        var prvek = cisla.stream()
            .filter(n -> n > 15)
            .findFirst();

        prvek.ifPresent(System.out::println);
    }
}
```

V tomto případě se vrátí číslo 20, protože je první, které splňuje podmínku.

Metoda `findAny` má velmi podobné použití, ale liší se v tom, že negarantuje pořadí. Může vrátit libovolný prvek ze streamu, který splňuje podmínku. U sekvenčního streamu se často chová stejně jako `findFirst`, ale její význam se projeví hlavně u paralelního zpracování, kde může být rychlejší.

```java
var prvek = cisla.stream()
    .filter(n -> n > 15)
    .findAny();

prvek.ifPresent(System.out::println);
```

Z praktického hlediska se tedy rozhodujeme takto: pokud potřebujeme přesně první prvek, použijeme `findFirst`. Pokud nám stačí „nějaký“ prvek splňující podmínku, použijeme `findAny`.

Důležitou součástí práce s těmito metodami je zpracování výsledku typu `Optional`. Nejjednodušší přístup je použít `orElse` a definovat náhradní hodnotu:

```java
int vysledek = cisla.stream()
    .filter(n -> n > 100)
    .findFirst()
    .orElse(-1);
```

Pokud žádný prvek neexistuje, vrátí se hodnota -1.

V situacích, kdy je absence výsledku chyba, můžeme použít `orElseThrow`:

```java
int vysledek = cisla.stream()
    .filter(n -> n > 100)
    .findFirst()
    .orElseThrow(() -> new RuntimeException("Prvek nenalezen"));
```

Tento zápis jasně říká, že očekáváme existující výsledek a jiná situace je nepřípustná.

Z hlediska návrhu programu jsou `findFirst` a `findAny` velmi užitečné, protože umožňují ukončit zpracování streamu dříve. Jakmile je nalezen vhodný prvek, další prvky se už neprochází. To může mít významný vliv na výkon, zejména u větších datových struktur.

Typická praktická úloha vypadá například takto: máme kolekci objednávek a chceme najít první nezaplacenou:

```java
var nezaplacena = objednavky.stream()
    .filter(o -> !o.zaplaceno)
    .findFirst();
```

Tento zápis je krátký, čitelný a přesně vystihuje záměr. Právě v tom spočívá hlavní výhoda – místo ručního cyklu a kontrolních proměnných můžeme přímo vyjádřit, co chceme v datech najít.
