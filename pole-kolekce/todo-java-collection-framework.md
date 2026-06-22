# TODO Java Collection Framework

Jak bylo zmíněno, jazyk Java představuje několik datových typů pro práci se sadou prvků, které svou funkcionalitou překračují klasické pole. Tyto prvky představuje pod knihovnou _Collection framework_. Všechny typy jsou obsaženy v balíčku _java.lang_.

Obecně lze všechny třídy v tomto frameworku rozdělit do dvou skupin:

* Kolekce - jsou typy určené pro udržení skupiny hodnot v tzv. kolekci. Do kolekce lze typicky prvky přidávat, odebírat a postupně je procházet.
* Mapy - jsou typy určené pro reprezentaci dvojic hodnot. Neobsahují tedy samotné hodnoty, ale vždy dvojice hodnot - klíč a jemu odpovídající hodnotu.

Ještě před představení obou skupin je ale třeba zmínit pojem _typových kolekc&#xED;**.**_

#### Typové kolekce

Představme si jednoduchou třídu reprezentující seznam, která nabízí dvě základní metody:

* _void add(Object o)_ - metoda, která přidá do seznamu prvek předaný jako parametr,
* _int size() -_ metoda, která vrací počet prvků v seznamu, a
* _Object get(int index) -_ metoda, která vrátí prvek ze seznamu na zadaném _indexu_.

Pomocí zmíněných metod můžeme do seznamu prvky přidávat a následně je vybírat podle indexu[\[22\]](https://word2md.com/#footnote-22), jak ukazuje následující rychlý příklad.

seznam.add(1);\
Object prvek = seznam.get(0);

Problém je, jak si lze všimnout, v typech, se kterými obě operace pracují. Metoda _add()_ má jako parametr typ _Object_, do seznamu tedy lze vložit cokoliv. Naopak, metoda _get()_ vrací také datový typ _Object_, prvek vybraný z kolekce tedy musíme explicitně přetypovat na požadovaný typ. Přitom se můžeme dostat do nepříjemné situace.

seznam.add("první");\
seznam.add("druhý");\
seznam.add("třetí");\
seznam.add(new java.util.Date());\
for (int i = 0; i < seznam.size(); i++){\
\*\*String prvek = (String) seznam.get(i);\
\*\*System.out.println(prvek);\
}

Problematickou situaci způsobí přidání čtvrtého prvku, který není řetězcem, ale jedná se o instanci třídy _Date_. Tím, že kolekce pracuje s jakoukoliv třídou (protože všechny třídy jsou potomci třídy _Object_), můžeme do ní vložit libovolný objekt, což velmi často není žádoucí a navíc nás na tuto chybu ani kompilátor nijak neupozorní. Za běhu programu se potom při procházení všech položek pokoušíme při vybírání prvků všechny přetypovat na řetězec (tučně vyznačený řádek). U čtvrtého prvku však operace selže, protože _Date_ nelze přetypovat na _String_ a program způsobí chybu za běhu. Tyto chyby se velmi špatně hledají, protože se nemusí projevit vždy, ale pouze v určitých případech, kdy někdo (typicky finální uživatel aplikace) přidá do seznamu nějakou nečekanou hodnotu. Tomuto chování, kdy třídy v _Java Collection Frameworku_ pracují s obecnými objekty, se říká _**netypové kolekce**._

Java však od verze 1.5 (Java 5) umožňuje tvorbu kolekcí **typových**. Typová kolekce má definici obecnou - například výše uvedený seznam metod bychom mohli přepsat jako:

* _void add(E e)_ - metoda, která přidá do seznamu prvek předaný jako parametr,
* _int size() -_ metoda, která vrací počet prvků v seznamu, a
* _E get(int index) -_ metoda, která vrátí prvek ze seznamu na zadaném _indexu_.

Lze si povšimnout, že stejný seznam najednou pracuje s nějakou neznámou třídou, označenou jako _E_. Seznam „neví", jakého konkrétního typu prvky v něm uloženy budou, ale ví, že budou určitého, stejného typu, nazvaného (prozatím) _E_. Důležité je, že metoda _add()_ i metoda _get()_ pracují se stejným typem, takže instance toho typu, který se do kolekce vloží, bude také z kolekce vyjmut.

Programátor pak při použití takového seznamu explicitně řekne u deklarace proměnné, co vlastně oním typem _E_ bude - a provede to tak, že požadovaný typ zapíše do lomených závorek tak, jak ukazuje následující příklad.

List\*\*\<String>\*\* seznam = new ArrayList();\
seznam.add("první");\
seznam.add("druhý");\
seznam.add("třetí");\
\*\*// nelze provést, Date není String\
// seznam.add(new java.util.Date());\
\*\*\
for (int i = 0; i < seznam.size(); i++){\
\*\*String prvek = seznam.get(i);\
\*\*System.out.println(prvek);\
}

Co se změnilo? Při deklaraci se do lomených závorek uvedlo, že seznam tentokrát bude fungovat speciálně pro typ _String_. Kompilátor tedy všechny výskyty typu _E_ uvedené v předchozích bodech nahradí za typ _String_. Program se díky tomu bude chovat jinak:

* Čtvrtý prvek již vůbec nelze do kolekce přidat - **kompilátor již při překladu** zjistí, že typ _Date_ nelze převést na typ _String_ a proto na daném řádku zahlásí kompilační chybu a program vůbec nepůjde přeložit.
* Při získávání prvku ze seznamu metodou _get()_ již nemusíme provádět explicitní přetypování pomocí „_(String)_", protože kompilátor již ví, že seznam bude obsahovat jenom instance třídy _String_.

Toto řešení umožňuje odstranit velké množství chyb již při návrhu aplikace. Chyby, které by se dříve projevily až za běhu aplikace, lze nyní detekovat již při překladu a bez jejich odstranění program vůbec nepůjde přeložit.

Drtivá většina datových typů (tříd, ale i rozhraní) z _Java Collection Framework_ umožňuje při deklaraci definovat a pracovat s takovýmto neznámým typem _E_, který se konkrétně specifikuje až při deklaraci proměnné. Beztypové kolekce se již téměř nepoužívají. Proto i v této studijní opoře budeme pracovat výhradně s typovými kolekcemi, i když explicitně postfix \<E>, kterým bychom takovouto třídu označovali, uvádět nebudeme. Velmi důležitá poznámka na konec: **Jako definici typu do lomených závorek lze zapsat pouze potomky třídy Object!** Nelze tedy použít primitivní typy, ale **musíme využít jejich wrapovacích typů** (viz kapitola 5.2). Při použití primitivního typu překladač zahlásí chybu.

#### Rozdělení kolekcí

Kolekce jsou první skupinou. Reprezentují je vlastně všechny třídy, které dědí z rozhraní _java.util.Collection_. Potomky - rozhraní i jejich implementace ukazuje následný graf[\[23\]](https://word2md.com/#footnote-23).

Collection

Set

TreeSet

HashSet

List

ArrayList

Modrá barva reprezentuje rozhraní, fialová barva reprezentuje implementace - tedy konkrétní třídy. Lze si povšimnout, že kolekce lze rozdělit do dvou základních skupin podle nadřazeného rozhraní:

* Potomci/implementace třídy **Set** - reprezentují _množiny_;
* Potomci/implementace třídy **List** - reprezentují _seznamy_.

Následně budou obě skupiny podrobně vysvětleny.

#### Množiny

Dalším příkladem tříd z _Java Collection Frameworku_ jsou seznamy. Lze do nich také přidávat prvky, odebírat prvky a procházet je pomocí cyklu _for-each_, na rozdíl od seznamu ale má odlišné chování, na které je třeba pamatovat:

* Nelze vložit hodnotu _null_.
* Každá hodnota může být v množině pouze jednou.
* Prvky nemusí být v pořadí, ve kterém byly do množiny vloženy[\[24\]](https://word2md.com/#footnote-24).
* Na jednotlivé prvky se nelze dostat pomocí indexu a nelze tedy ani použít cyklus _for_.

Základní rozhraní množiny _Set_ implementuje několik metod určených pro práci s prvky.

| Signatura funkce           | Popis                                                                         |
| -------------------------- | ----------------------------------------------------------------------------- |
| boolean add(E e)           | Vloží do množiny prvek. Pokud již prvek v množině existuje, nic se neprovede. |
| void clear()               | Odstraní všechny prvky z množiny.                                             |
| boolean contains(Object o) | Vrací true, pokud prvek předaný jako parametr v množině existuje.             |
| boolean isEmpty()          | Vrací true, pokud je množina prázdná.                                         |
| boolean remove(Object o)   | Odstraní prvek z množiny.                                                     |
| int size()                 | Vrací počet prvků v množině.                                                  |

Lze si povšimnout, že opravdu chybí jakákoliv metoda pro získání prvku z množiny. Prvky množiny lze procházet pomocí cyklu _for-each_, který postupně projde všechny prvky, ale žádná metoda, která by vrátila prvek pomocí indexu, v množině neexistuje.

Před představením implementací je ještě třeba zmínit, jak množina pozná, že už stejný prvek v množině existuje. Nepoužívá se porovnání pomocí operátoru „==" - tehdy by se například v množině mohly vyskytovat dva stejné řetězce, umístěné na různých místech v paměti. Množiny namísto toho využívají již dříve představené metody _hashCode()_ a _equals()_. Obě metody byly představeny v kapitole 5.3.2 a programátor může pomocí přetížení těchto metod u každé třídy specifikovat, jak bude porovnání probíhat.

Existují dvě základní implementace pro množiny:

* HashSet - základní množina udržující prvky v nějakém nedeterministickém pořadí;
* TreeSet - množina udržující prvky seřazené podle velikosti.

Typy množin se liší tedy podle toho, jakým způsobem si do sebe ukládají jednotlivé prvky.

Obě třídy budou představeny blíže. Opět je vhodné se v ukázkových zdrojových kódech povšimnout, že konkrétní typ množiny (_HashSet_ nebo _TreeSet_) určujeme pouze při vytváření nové instance (tedy za klíčovým slovem _new_), ale proměnnou typujeme na obecného předka _Set_ - programujeme tedy opět proti rozhraní.

**HashSet**

Už podle názvu, třída _HashSet_ pracuje s hash kódem a má tedy souvislost s metodou _hashCode()_ (viz kapitola 5.3.2), kterou má každý objekt. Jak bylo zmíněno, do instance takové množiny lze vložit instance libovolné třídy, vyjma hodnoty null. Instance se navíc nesmí opakovat.

Výhoda typu _HashSet_ je v rychlosti přidávání prvků (konkrétně v testování, zda již existující prvek v množině není). Pokud chceme rychle vytvořit skupinu prvků a zároveň zajistit, že každý prvek se vyskytuje v množině pouze jednou, použijeme právě _HashSet_. Opakované přidání existujícího prvku ale nezpůsobí žádnou chybu; pokud přidávaný prvek již existuje, prostě se jeho opětovné přidání neprovede.

&#x20;public static void example1() {

java.util.Set\<String> hs = new java.util.HashSet();

hs.add("c");

hs.add("b");

hs.add("a");

hs.add("a");

hs.add("b");

hs.add("c");

for(String item : hs)

System.out.println(item);

System.out.println("Celkem: " + hs.size());

}

Jak bylo zmíněno, použitá kolekce je typově bezpečná (pro typ _String_), lze ji procházet pomocí cyklu _for-each_ a ve výsledku výše uvedený kód vrátí následující výsledek:

run:

b

c

a

Celkem: 3

BUILD SUCCESSFUL (total time: 0 seconds)

Přestože jsme tedy do množiny přidali 6 prvků, duplicitní vložení se neprovedlo a množina tedy obsahuje pouze tři prvky. Tyto prvky jsou navíc **v nespecifikovatelném pořadí** odlišném od pořadí, ve kterém byly do množiny vkládány!

**TreeSet**

Druhým (běžným) typem množiny je stromová množina - _TreeSet_. V této množině platí, že jednotlivé vložené prvky jsou vždy seřazeny[\[25\]](https://word2md.com/#footnote-25) vzestupně. Tento typ množiny tedy vždy zachovává pořadí prvků podle velikosti, jak stručně představuje jen velmi mírně upravený výpis.

&#x20;public static void example2() {

java.util.Set\<String> hs = new java.util.**TreeSet**();

hs.add("c");

hs.add("b");

hs.add("a");

hs.add("a");

hs.add("b");

hs.add("c");

for (String item : hs) {

System.out.println(item);

}

System.out.println("Celkem: " + hs.size());

}

Upravená část kódu je zvýrazněna tučně a je zde vidět výhoda programování proti rozhraní - programátor se může kdykoliv rozmyslet a nahradit původní množinu jinou implementací bez dopadu na okolní zdrojový kód. Následuje výsledek výše uvedeného výpisu.

run:

a

b

c

Celkem: 3

BUILD SUCCESSFUL (total time: 0 seconds)

Je vidět, že prvky jsou opravdu seřazeny.

Při práci však můžeme chtít do množiny vložit instanci libovolné třídy. Uvažujme například velmi jednoduchou třídu _Person_.

class Person {

private String name;

private int age;

public Person(String name, int age) {

this.name = name;

this.age = age;

}

}

Výše uvedený příklad upravíme pro vložení instancí třídy _Person_.

java.util.Set\<Person> hs = new java.util.TreeSet();

hs.add(new Person("Petra", 20));

hs.add(new Person("Iva", 18);

System.out.println("Celkem: " + hs.size());

Po spuštění však kód nebude fungovat a vrátí chybu (zkráceno).

run:

Exception (…): **eng.demos.collectionFramework.Person cannot be cast to java.lang.Comparable**

(…)

BUILD SUCCESSFUL (total time: 2 seconds)

Řekli jsme totiž, že _TreeSet_ seřazuje do něj vložené objekty podle velikosti, ale prostředí jazyka Java netuší, jakým způsobem má porovnávat instance naší třídy _Person_. Pro další práci s typem _TreeSet_ je tedy třeba se seznámit s mechanismem, který definuje, jakým způsobem lze mezi sebou porovnávat instance vlastní třídy.
