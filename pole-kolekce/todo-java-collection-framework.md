# Java Collection Framework

## Úvod do Java Collection Framework

Jak bylo zmíněno, jazyk Java představuje několik datových typů pro práci se sadou prvků, které svou funkcionalitou překračují klasické pole. Tyto prvky představuje pod knihovnou _Java Collection framework_. Všechny typy jsou obsaženy v balíčku `java.lang`.

Obecně lze všechny třídy v tomto frameworku rozdělit do dvou skupin:

* **Kolekce** - jsou typy určené pro udržení skupiny hodnot v tzv. kolekci. Do kolekce lze typicky prvky přidávat, odebírat a postupně je procházet.
* **Mapy** - jsou typy určené pro reprezentaci dvojic hodnot. Neobsahují tedy samotné hodnoty, ale vždy dvojice hodnot - klíč a jemu odpovídající hodnotu.

Ještě před představení obou skupin je ale třeba zmínit pojem _**typových** kolekc&#xED;**.**_

### Typové kolekce

Představme si jednoduchou třídu reprezentující seznam (například nákupní lístek), která nabízí tři základní metody:

* `void add(Object o)` - metoda, která přidá do seznamu prvek předaný jako parametr,
* `int size()` - metoda, která vrací počet prvků v seznamu, a
* `Object get(int index)` - metoda, která vrátí prvek ze seznamu na zadaném _indexu_.

Pomocí zmíněných metod můžeme do seznamu prvky přidávat a následně je vybírat podle indexu, jak ukazuje následující rychlý příklad.

```java
seznam.add(1);
Object prvek = seznam.get(0);
```

Problém je v typech, se kterými obě operace pracují. Metoda `add()` má jako parametr typ `Object`, do seznamu tedy lze vložit cokoliv (protože každá třída dědí z datového typu `Object`. Naopak, metoda `get()` vrací také datový typ `Object`, prvku vybranému z kolekce tedy musíme věřit, že je správného typu a navíc jej tedy musíme explicitně přetypovat na požadovaný typ. Přitom se můžeme dostat do nepříjemné situace.

```java
seznam.add("první");
seznam.add("druhý");
seznam.add("třetí");
seznam.add(new java.util.Date()); // <-- přidáváme typ Date, ne String
for (int i = 0; i < seznam.size(); i++){
  String prvek = (String) seznam.get(i); // <- selže u posledního prvku, protože není String ale Date
  System.out.println(prvek);
}
```

Problematickou situaci způsobí přidání čtvrtého prvku, který není řetězcem, ale jedná se o instanci třídy `Date`. Tím, že kolekce pracuje s jakoukoliv třídou (protože všechny třídy jsou potomci třídy `Object`), můžeme do ní vložit libovolný objekt, aniž by vznikla jakákoliv chyba. Toto velmi často není žádoucí - typicky dáváme do kolekce instance určitého konkrétního datového typu.

Za běhu programu se potom při procházení všech položek pokoušíme při vybírání prvků všechny přetypovat na řetězec (viz komentář v kódu). U čtvrtého prvku však operace selže, protože `Date` nelze přetypovat na `String` a program způsobí chybu neočekávanou za běhu. Tyto chyby se velmi špatně hledají, protože obecně se nemusí projevit vždy, ale pouze v určitých případech, kdy někdo (typicky finální uživatel aplikace) přidá do seznamu nějakou nečekanou hodnotu. Tomuto chování, kdy třídy v _Java Collection Frameworku_ pracují s obecnými objekty, se říká _**netypové kolekce**._

Java však od verze 1.5 (Java 5) umožňuje tvorbu kolekcí **typových**. Typová kolekce má definici obecnou - například výše uvedený seznam metod bychom mohli přepsat jako:

* `void add(E e)` - metoda, která přidá do seznamu prvek předaný jako parametr,
* `int size()` - metoda, která vrací počet prvků v seznamu, a
* `E get(int index)` - metoda, která vrátí prvek ze seznamu na zadaném _indexu_.

Lze si povšimnout, že stejný seznam najednou pracuje s nějakou neznámou třídou, označenou jako `E`. Seznam „neví", jakého konkrétního typu prvky v něm uloženy budou, ale ví, že budou určitého, stejného typu, nazvaného (prozatím) `E`. Důležité je, že metoda `add()` i metoda `get()` pracují se stejným typem, takže instance toho typu, který se do kolekce vloží, bude také z kolekce vyjmut.

Programátor pak při použití takového seznamu explicitně řekne u deklarace proměnné, co vlastně oním typem `E` bude - a provede to tak, že požadovaný typ zapíše do lomených závorek tak, jak ukazuje následující příklad.

```java
List<String> seznam = new ArrayList<String>(); // <-- list pro prvky String
seznam.add("první");
seznam.add("druhý");
seznam.add("třetí");
// seznam.add(new java.util.Date()); // <-- tento řádek způsobí chybu při kompilaci, Date není String
for (int i = 0; i < seznam.size(); i++){
  String prvek = seznam.get(i); // <-- zde není třeba přetypovat, kolekce přímo vrací String
  System.out.println(prvek);
}
```

Co se změnilo? Při deklaraci se do lomených závorek uvedlo, že seznam tentokrát bude fungovat speciálně pro typ `String`. Kompilátor tedy všechny výskyty typu `E` uvedené v předchozích bodech nahradí za typ `String`. Program se díky tomu bude chovat jinak:

* Čtvrtý prvek již vůbec nelze do kolekce přidat - **kompilátor již při překladu** zjistí, že typ `Date` nelze převést na typ `String` a proto na daném řádku zahlásí kompilační chybu a program vůbec nepůjde přeložit.
* Při získávání prvku ze seznamu metodou `get()` již nemusíme provádět explicitní přetypování pomocí `(String)`, protože kompilátor již ví, že seznam bude obsahovat jenom instance třídy `String`.

Toto řešení umožňuje odstranit velké množství chyb již při návrhu aplikace. Chyby, které by se dříve projevily až za běhu aplikace, lze nyní detekovat již při překladu a bez jejich odstranění program vůbec nepůjde přeložit.

{% hint style="info" %}
Při definici lze využít (a běžně se používá) thv. diamantový operátor `<>`, který datový typ instance odvodí z definice typu. Viz srovnání:

```java
List<String> seznam = new ArrayList<String>(); // nepoužívá se
List<String> seznam = new ArrayList<>(); // použití s diamantovým operátorem <>
```
{% endhint %}

Drtivá většina datových typů (tříd, ale i rozhraní) z _Java Collection Framework_ umožňuje při deklaraci definovat a pracovat s takovýmto neznámým typem `E`, který se konkrétně specifikuje až při deklaraci proměnné. Beztypové kolekce se již téměř nepoužívají. Proto i v této studijní opoře budeme pracovat výhradně s typovými kolekcemi, i když explicitně postfix \<E>, kterým bychom takovouto třídu označovali, uvádět nebudeme. Velmi důležitá poznámka na konec: **Jako definici typu do lomených závorek lze zapsat pouze potomky třídy Object!** Nelze tedy použít primitivní typy, ale **musíme využít jejich wrapovacích typů** (viz kapitola 5.2). Při použití primitivního typu překladač zahlásí chybu.

### Typové kolekce vs primitivní typy

Při práci s typovými kolekcemi v Javě narazíme na jedno zásadní omezení, které pramení z toho, jak Java s generickými typy pod kapotou pracuje. Do generických kolekcí totiž nelze ukládat primitivní datové typy jako `int`, `double`, `char` nebo `boolean`.

{% hint style="info" %}
Důvodem je mechanismus zvaný _Type Erasure_ (vymazání typů), které se provádí při kompilaci. Java zavedla generické typy kvůli zpětné kompatibilitě tak, že je po kompilaci v podstatě nahradí obecným typem `Object`. Protože primitivní typy v Javě nedědí od třídy `Object` a nejsou to objekty v pravém slova smyslu, nelze je nahradit za `Object` a generické kolekce s nimi neumí přímo pracovat.
{% endhint %}

Pokud se pokusíte vytvořit kolekci přímo s primitivním typem, kompilátor vás nepustí dál.

```java
// TENTO KÓD NEBUDE FUNGOVAT A ZPŮSOBÍ CHYBU KOMPILACE:
ArrayList<int> cisla = new ArrayList<int>(); 
```

Jak z toho ven? Využijeme prapovací typy pro primitivní typy představené v [todo-primitivni-datove-typy.md](../zakladni-datove-typy/todo-primitivni-datove-typy.md "mention"). Každý primitivní typ má svůj objektový protějšek, který je třídou (například `Integer` pro `int`, `Double` pro `double` nebo `Character` pro `char`). Kolekce pak definujeme právě pomocí těchto obalových tříd. Díky funkci _autoboxingu_ navíc Java umí primitivní hodnoty na objekty a zpět převádět automaticky, takže v běžném kódu rozdíl při vkládání prvků téměř nepoznáte.

```java
// SPRÁVNÉ ŘEŠENÍ s využitím obalové třídy Integer:
ArrayList<Integer> cisla = new ArrayList<>();

// Díky autoboxingu můžeme vkládat obyčejná čísla (int) naprosto přirozeně:
cisla.add(10);
cisla.add(20);
// cisla.add((Integer) 30); // <-- přetypování je zbytečné

// Java automaticky převede Integer zpět na primitivní int:
int prvniCislo = cisla.get(0); 
```

Nyní si dále představíme 3 základní skupiny typů, se kterými se v JCF nejčastěji pracuje: kolekce (množiny a listy) a mapy.
