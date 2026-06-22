---
icon: square-6
---

# JSA - Best Practices

Při práci se Stream API je snadné začít psát velmi elegantní a krátký kód, ale stejně snadné je zajít do extrému, kdy se pipeline stane nepřehlednou a obtížně udržovatelnou. Proto je důležité dodržovat několik základních zásad, které pomáhají udržet kód čitelný, správný a efektivní.

Jedním z nejdůležitějších pravidel je vyhýbat se tzv. side effects. Streamy jsou navrženy jako deklarativní nástroj pro transformaci dat, nikoliv pro měnění stavu. Pokud uvnitř operací jako `map`, `filter` nebo `forEach` začneme zapisovat do externích proměnných nebo kolekcí, ztrácíme výhody Stream API a kód se chová méně předvídatelně.

Nevhodný přístup může vypadat například takto:

```java
List<Integer> vysledek = new ArrayList<>();

cisla.stream()
     .filter(n -> n % 2 == 0)
     .forEach(vysledek::add);
```

I když tento kód funguje, není ideální. Využívá side effect (`add`) a obchází deklarativní styl. Správně bychom měli použít `collect`:

```java
List<Integer> vysledek = cisla.stream()
    .filter(n -> n % 2 == 0)
    .toList();
```

Díky tomu je kód čistší a bezpečnější.

Dalším důležitým aspektem je rovnováha mezi čitelností a složitostí. Streamy umožňují řetězit operace za sebe, ale příliš dlouhá pipeline může být hůře pochopitelná než klasický cyklus. Pokud se v jedné pipeline kombinuje mnoho operací, složité podmínky nebo vnořená logika, je vhodné ji rozdělit nebo část logiky extrahovat do samostatných metod.

Například místo tohoto zápisu:

```java
cisla.stream()
     .filter(n -> n > 10 && n % 2 == 0 && n < 100)
     .map(n -> n * 2)
     .forEach(System.out::println);
```

může být čitelnější rozdělení s využitím pomocné metody, jejíž název nám v zápisu pomůže rozpoznat, o co se vlastně příkaz snaží:

```java
boolean jeValidni(int n) {
    return n > 10 && n % 2 == 0 && n < 100;
}

cisla.stream()
     .filter(Main::jeValidni)
     .map(n -> n * 2)
     .forEach(System.out::println);
```

Další důležitá otázka zní, kdy streamy vůbec nepoužívat. I když jsou velmi užitečné, nejsou vždy nejlepší volbou. Například v případě jednoduché logiky nebo složitého větvení může být klasický `for` cyklus přehlednější. Stejně tak pokud potřebujeme jemnou kontrolu nad průběhem výpočtu nebo pracujeme s komplikovaným stavem, imperativní styl může být vhodnější.

Jednoduchý cyklus může být v některých případech srozumitelnější:

```java
for (int n : cisla) {
    if (n % 2 == 0) {
        System.out.println(n);
    }
}
```

## Ladění pomocí \`peek\`

Specifickou technikou při práci se streamy je ladění pomocí operace `peek`. Ta umožňuje nahlédnout do průběhu pipeline a provést nad každým prvkem nějakou operaci - například jej vypsat na konzoli. Používá se především při debugování.

```java
cisla.stream()
     .filter(n -> n > 2)
     .peek(n -> System.out.println("Po filter: " + n))
     .map(n -> n * 2)
     .peek(n -> System.out.println("Po map: " + n))
     .forEach(System.out::println);
```

Operace `peek` by se neměla používat pro běžnou logiku, protože jde o vedlejší efekt. Je určena primárně pro diagnostiku a ladění, podobně jako výpisy v klasickém cyklu.

Na závěr je dobré si uvědomit, že cílem Stream API není jen zkrácení kódu, ale především jeho zpřehlednění. Pokud stream vede k méně čitelnému řešení, je lepší zvolit jiný přístup. Správně napsaná pipeline by měla působit jako přirozený popis toho, co se s daty děje.
