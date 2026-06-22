---
icon: square-1
---

# JSA - Motivace a představení

## Motivace

### Deklarativní vs Imperativní styl

Jedním z nejdůležitějších přínosů Stream API je přechod od imperativního ke deklarativnímu stylu programování. V imperativním přístupu popisujeme **každý jednotlivý krok algoritmu** — tedy jak přesně se má výpočet provést. Typicky to znamená psaní cyklů, podmínek, vytváření pomocných kolekcí a postupné skládání výsledku. Kód tak často obsahuje technický „šum“, který zakrývá samotný záměr.

Uvažujme jednoduchý příklad. Chceme zjistit e-maily českých zákazníků, kteří udělali 10 největších objednávek. V klasickém přístupu by řešení mohlo vypadat nějak takto:

```java
List<Order> filtered = new ArrayList<>();

for (Order o : orders) {
    if ("CZ".equals(o.getCustomer().getCountry())) {
        filtered.add(o);
    }
}

filtered.sort(new Comparator<Order>() {
    @Override
    public int compare(Order o1, Order o2) {
        return Double.compare(o2.getTotalPrice(), o1.getTotalPrice());
    }
});

List<String> emails = new ArrayList<>();
for (int i = 0; i < filtered.size() && i < 10; i++) {
    emails.add(filtered.get(i).getCustomer().getEmail());
}
```

Vidíme, že výše uvedený kód není nijak zvlášť přehledný a bez podrobnějšího studia není na první pohled zřejmé, co přesně dělá.

Deklarativní přístup naopak umožňuje vyjádřit pouze **výsledek, kterého chceme dosáhnout**, bez nutnosti popisovat jednotlivé kroky provedení. U Stream API to znamená, že místo ručního procházení kolekce pouze říkáme: _vyfiltruj data podle podmínky, seřaď je a vezmi prvních 10_. Jak přesně se tyto operace provedou, ponecháváme na implementaci knihovny.

Obecným zápisem by to mohlo vypadat nějak takto:

```
Vem všechny objednávky
-> vyber pouze ty s českými zákazníky
-> výsledek seřaď podle nejvyšší ceny objednávky
-> z výsledku vyber 10 prvních záznamů
-> z nich zjisti jen jména zákazníků
-> a ty vrať jako list.
```

Je vidět, že tento způsob zápisu by byl mnohem přehlednější. Přeskočíme teorii a rovnou prozradíme, že ve identická implementace v jazyce Java by vypadala cca takto:

```java
List<String> emails = orders.stream()
    .filter(o -> "CZ".equals(o.getCustomer().getCountry()))
    .sorted(Comparator.comparing(Order::getTotalPrice).reversed())
    .limit(10)
    .map(o -> o.getCustomer().getEmail())
    .toList();
```

Zatím, bez bližší znalosti _lambda výrazů_ a významu podivného symbolu `->` to může vypadat zmatečně, ale po seznámení uvidíte, že díky tomu je kód:

* kratší a přehlednější,
* lépe čitelný (rychleji pochopíme záměr),
* méně náchylný na chyby způsobené manuální manipulací se stavem.

Právě tahle změna stylu je klíčová — při práci se Stream API se učíme „myslet v dotazech nad daty“, nikoli v sekvencích instrukcí.

### Řetězení operací

Další klíčovou vlastností Stream API je možnost **řetězení operací do jedné plynulé pipeline**. Jednotlivé kroky, jako je filtrace, transformace, řazení nebo omezení počtu prvků, na sebe přirozeně navazují a výstup jednoho slouží jako vstup pro další. Díky tomu můžeme složit i poměrně komplexní dotaz jako lineární posloupnost operací, která se čte shora dolů.

Místo rozdělení logiky do několika oddělených bloků (filtr → pomocná kolekce → sort → další cyklus → transformace) máme všechno na jednom místě, v přesném pořadí, v jakém se má aplikovat. Každý krok je jasně viditelný a oddělený, což výrazně zlepšuje čitelnost i u složitějších operací.

Zároveň platí, že jednotlivé operace jsou malé a jednoúčelové — každá řeší jeden konkrétní problém (např. `filter` vybírá prvky, `map` je transformuje, `limit` je omezuje). Právě díky tomuto skládání jednoduchých kroků vzniká silný nástroj, který umožňuje psát kód, který je nejen kratší, ale hlavně přehlednější a lépe udržovatelný.

### Žádné mutace a pomocné kolekce

Další výraznou výhodou Stream API je, že **pracujeme bez mutací a bez explicitních pomocných kolekcí**. V klasickém imperativním kódu si často vytváříme nové seznamy, do kterých postupně přidáváme prvky, případně měníme stav existujících struktur. Takový kód je nejen delší, ale také náchylnější k chybám — například kvůli sdílenému stavu nebo špatnému pořadí operací.

Stream API naopak podporuje styl, kde jsou data zpracovávána jako tok a jednotlivé operace nad nimi jsou **bezstavové (stateless)**. Neříkáme „přidej prvek do seznamu“, ale „vezmi všechny prvky, které splňují podmínku, a převeď je na jinou podobu“. Výsledná kolekce vzniká až na konci pipeline, typicky pomocí `toList()` nebo `collect()`.

Díky tomu:

* odpadá potřeba mezikroků s dočasnými kolekcemi,
* nevznikají vedlejší efekty (nemění se sdílený stav),
* kód je bezpečnější a lépe se s ním pracuje.

Tento přístup dobře zapadá i do funkcionálního stylu programování, kde je snaha minimalizovat změny stavu a místo toho pracovat s transformacemi dat.

Praktickým benefitem je i to, že moderní IDE (například IntelliJ IDEA) umí při ladění stream pipeline **zobrazit průběžné výsledky jednotlivých kroků**. Při debugování tak vidíme, jak data „protékají“ přes `filter`, `map` nebo `limit`, což výrazně usnadňuje pochopení i hledání chyb. Tento komfort u klasických cyklů a ručních mezikolekcí často chybí nebo je výrazně hůře dosažitelný.

## Úvod do Java Stream API

Stream API bylo do Javy přidáno v rámci zásadního rozšíření jazyka ve verzi **Java 8 (rok 2014)**. Spolu s ním přišly i další klíčové prvky, jako lambda výrazy, funkční rozhraní nebo method reference. Cílem bylo umožnit vývojářům psát kód více deklarativním a funkcionálním stylem, který byl do té doby typický spíše pro jiné jazyky.

Před zavedením Stream API se většina práce s kolekcemi řešila pomocí klasických cyklů a postupných transformací, což vedlo k delším a hůře čitelným konstrukcím. Stream API tento přístup výrazně změnilo — umožnilo nad kolekcemi psát složitější operace jako jeden souvislý „dotaz“. Od té doby se Stream API stalo standardním nástrojem pro práci s kolekcemi v moderní Javě a je široce využíváno jak v běžném aplikačním kódu, tak i v knihovnách a frameworkech.

### Kolekce vs Stream

Než začneme používat Stream API v praxi, je důležité pochopit základní rozdíl mezi **kolekcí** a **streamem**, protože tyto dva pojmy se často zaměňují, ale mají zcela odlišnou roli.

* **Kolekce** (např. `List`, `Set`) slouží primárně k **uchovávání dat**. Je to datová struktura, která drží prvky v paměti, umožňuje k nim opakovaně přistupovat, měnit je, přidávat nebo odebírat. Kolekce tedy reprezentuje **stav** – obsahuje konkrétní hodnoty, se kterými můžeme libovolně pracovat.
* **Stream** naopak data **neukládá**, ale slouží pouze k jejich **zpracování**. Je to pohled na data z kolekce (nebo jiného zdroje), který nám umožňuje nad nimi provádět operace jako filtrování, transformaci nebo agregaci. Stream tedy reprezentuje spíše **výpočet** nebo „dotaz“, který se nad daty vykoná.

To má několik důsledků:

* stream nelze použít opakovaně (po provedení je „spotřebovaný“),
* stream nemění původní kolekci (pokud to explicitně neuděláme),
* operace nad streamem se provádějí až v momentě, kdy zavoláme ukončovací operaci (tzv. lazy execution).

Typický vztah mezi nimi vypadá takto:

```java
List<String> names = List.of("Alice", "Bob", "Charlie");

// kolekce → stream → výsledek
List<String> result = names.stream()
    .filter(name -> name.startsWith("A"))
    .toList();
```

Kolekce je zde zdroj dat, zatímco stream je mechanismus, jak nad těmi daty provést operace. Jinými slovy: kolekce říká „co máme“, zatímco stream říká „co s tím chceme udělat“.

### Stream Pipeline

Každý stream v Javě funguje jako **pipeline (zpracovatelský řetězec)**. Tato pipeline má vždy tři základní části: zdroj (source), mezikroky (intermediate operace) a ukončovací operaci (terminal operace).

Na začátku stojí **source**, tedy zdroj dat. Nejčastěji je to kolekce (`list.stream()`), ale může to být i pole nebo jiný generátor dat. Ze zdroje se vytvoří stream, který začíná „téct“.

{% hint style="info" %}
Typické zdroje dat pro stream jsou například běžné kolekce (seznamy, množiny), pole nebo přímo vytvořené sady hodnot — tedy obecně místa, odkud potřebujeme data dále zpracovávat.
{% endhint %}

Na tento zdroj pak navazují **intermediate operace**, jako například `filter`, `map`, `sorted` nebo `limit`. Tyto operace stream nijak „neuzavírají“, ale pouze ho dál transformují. Důležitá vlastnost je, že jsou **lazy** — samy o sobě se hned nevykonají, pouze skládají popis výpočtu.

{% hint style="info" %}
Typické mezikroky prováděné nad streamy zahrnují operace pro filtrování záznamů (`filter`), transformaci dat do jiné podoby (`map`), řazení prvků podle zvoleného kritéria (`sorted`) nebo omezení počtu výsledků (`limit`).
{% endhint %}

Teprve na konci pipeline přichází **terminal operace**, například `toList()`, `collect()`, `forEach()` nebo `findFirst()`. Až tato operace skutečně spustí celý výpočet a „protlačí“ data všemi předchozími kroky.

{% hint style="info" %}
Typické ukončovací operace jsou například operace pro získání výsledku jako kolekce (`toList`, `collect`), nalezení konkrétního prvku (`findFirst`) nebo ověření, zda data splňují určitou podmínku (`anyMatch`, `allMatch`).
{% endhint %}

Typická pipeline vypadá takto:

```java
List<String> result = users.stream()             // source
    .filter(u -> u.isActive())                   // intermediate
    .map(u -> u.getEmail())                      // intermediate
    .limit(10)                                   // intermediate
    .toList();                                   // terminal
```

S tím úzce souvisí důležitá vlastnost: **stream je jednorázový**. Jakmile se provede terminal operace, stream je „spotřebovaný“ a nelze ho použít znovu.

Například tento kód skončí chybou:

```java
Stream<User> stream = users.stream();

stream.filter(u -> u.isActive()).toList();
stream.map(u -> u.getEmail()).toList(); // ❌ IllegalStateException
```

Důvod je jednoduchý — stream nepředstavuje kolekci, ale jednorázový výpočet. Jakmile se jednou provede, už není co znovu zpracovávat.

Pokud potřebujeme stejná data zpracovat vícekrát, vždy si musíme vytvořit nový stream ze zdroje:

```java
users.stream().filter(u -> u.isActive()).toList();
users.stream().map(u -> u.getEmail()).toList();
```

Tento model odpovídá tomu, jak stream skutečně funguje — jako průchod dat „od začátku do konce“, nikoli jako struktura, kterou bychom mohli používat opakovaně.
