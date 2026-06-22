# TODO Generické datové typy

## Generické datové typy

V této kapitole se dozvíte:

* Co jsou generika a jak se používají?

Po jejím prostudování byste měli být schopni:

* Vytvořit vlastní generický typ

Klíčová slova této kapitoly:

Generika, šablony

Doba potřebná ke studiu: 5 hodin

**Průvodce studiem**

Následující kapitola se věnuje problematice generických typů, které umožňují vytvářet třídy pro práci s obecnými typy, které se specifikují až při vytváření instancí. Tento postup značně zvyšuje produktivitu práce u úloh, které se prováděj pro podobné typy velmi obdobně.

Pro studium kapitoly a samostatné ověření příkladů vlastními programy si vyhraďte 5 hodin.

Generické datové typy, tzv. generika, je jednou z důležitých vlastností moderních objektově orientovaných programovacích jazyků. V jazyce Java byla generika zavedena v roce 2004 jakou součást verze Java 1.5. Oč tedy jde?

Určité obecné algoritmy nebo řešení lze zapouzdřit do metod tříd. Tyto algoritmy se provádějí principielně vždy stejně, pouze typicky nad různými třídami.

Typickým příkladem, který již byl ve studijní opoře zmíněn, je seznam. Názorně bude ukázán příklad mírně odlišný - zásobník. Zásobník je objekt, do kterého lze objekty vkládat (metoda _push()_) a následně vybírat (metoda _pop()_), přičemž platí, že poslední vložený objekt se vybírá jako objekt první. Jednoduchá třída (bez implementace) nabízející rozhraní pro tuto problematiku by mohla vypadat nějak takto:

class Stack {

// nějaké třídní proměnné

public void push(Object item){

// uložení objektu do zásobníku

}

public Object pop (){

// získání posledního vloženého objektu do zásobníku

}

}

Třída opravdu nabízí požadované metody pro vkládání a vybírání prvků. Následující ukázka ukazuje rychlé použití takového zásobníku.

Stack stack = new Stack();

stack.push(1);

stack.push(2);

**stack.push("tři");**

Integer value = null;

value = **(Integer)** stack.pop();

Z výše uvedeného výpisu si lze povšimnout, že toto, byť obecné řešení, sebou přináší dva základní problémy:

* Pomocí metody _push()_ může do zásobníku programátor vložit libovolný objekt;
* Pomocí metody _pop()_ získá ze zásobníku obecný objekt a neví, o instanci kterého konkrétního typu jde. Získávaný objekt může při vracení ze zásobníku přetypovat na předpokládaný typ, ale pokud v zásobníku bude nepřetypovatelný objekt, program spadne s chybou za běhu.

Úpravu tohoto problému již známe z problematiky kolekcí - chtěli bychom vytvořit zásobník tzv. typově, tedy explicitně říci, pro které typ bude, pomocí zápisu:

Stack\<Integer> stack = new Stack();

Tato definice ale použít nelze, protože lomené závorky lze přidat pouze ke generickým typům a tím výše zmíněná třída _Stack_ ještě není.

Z „běžného" typu vytvoříme generický typ velmi jednoduše - do deklarace třídy dopíšeme stejnou syntax, kterou používáme při vytvoření proměnné, místo konkrétního typu však použijeme nějaký obecný identifikátor typu, typicky _T_.

class Stack\<T> {

// nějaké třídní proměnné

public void push(T item){

// uložení objektu do zásobníku

}

public T pop (){

// získání posledního vloženého objektu do zásobníku

}

}

Takto definováno platí, že třída _Stack_ je **generická třída** s jedním **generickým parametrem**.

Je důležité si povšimnout, že tento typ _T_ (tedy generický parametr), uvedený v deklaraci názvu třídy, můžeme použít kdekoliv uvnitř těla třídy, tak jak je to ukázáno také na metodách _push()_ a _pop()_. Najednou z definovaných metod zcela zmizí použití typu _Object_. Kompilátor bude předpokládat, že programátor při použití typu _Stack_ vždy explicitně u deklarace proměnné do lomených závorek uvede, pro jaký typ se zásobník vytváří - a kompilátor tento typ vezme a v definici třídy (výše uvedeném kódu) tento typ automaticky dosadí za typ _T_. Můžeme udělat tedy několik různých zásobníků pro různé typy a každý z nich bude typově bezpečný a nepůjde do něj vložit jiná instance než je instance definovaného typu.

Stack\<Integer> intStack;

Stack\<String> stringStack;

Stack\<java.util.Calendar> calendarStack;

Bez generik bychom pro dosažení tohoto chování museli realizovat pro každý použitý typ implementaci zvlášť.

Pro doplnění, generická mohou být samozřejmě i rozhraní, s použitím stejné syntaxe.

Definice generických typů lze samozřejmě zanořovat do sebe a vytvořit tak například zásobník zásobníků zásobníků pro typ int.

Stack\<Stack\<Stack\<Integer>>> reStack;

V praxi se však užívají zanoření maximálně do hloubky 2, protože to velmi ztěžuje čitelnost kódu.

Třída v deklaraci může mít více generických parametrů a při použití musí programátor definovat odpovídající počet typů zastupujících generické parametry - typickým příkladem jsou právě mapy.

public interface Map\<K,V> {…}

### Omezení pro generické parametry

V určitých případech si při definování generického typu nevystačíme pouze se znalostí faktu, že bude obsahovat jeden nebo více generických parametrů, ale potřebuje zajistit určité schopnosti (tedy metody) těchno generických parametrů.

Vše bude prezentováno na jednoduchém příkladě seznamu, který udržuje všechny prvky vždy seřazené podle nativního řazení. Kvůli stručnosti a přehlednosti bude opravdu implementováno pouze nezbytné minimum funkci.

Základní kostra takového seznamu by mohla vypadat takto.

class SortedList\<T>{

public void add (T item){}

public T get(int index){}

}

Vidíme, že máme generickou třídu _SortedList_ s jedním neznámým, generickým parametrem. Třída bude obsahovat dvě metody - jednu pro přidání prvku do seznamu, druhou pro získání prvku ze seznamu podle indexu.

Pro udržení seznamu přidaných prvků uvnitř instance třídy využijeme vnitřní proměnnou jako klasický seznam a náš kód se bude pouze starat o správné seřazení v případě potřeby. Nejdříve tedy doplníme vnitřní seznam, ať máme kam prvky přidávat a odkud je vracet.

class SortedList\<T>{

private List\<T> inner = new ArrayList();

public void add (T item){

inner.add (item);

}

public T get(int index){

return inner.get(index);

}

}

Je důležité si opět povšimnout použití generického parametru i v kódu uvnitř třídy - vnitřní proměnná seznamu je typový seznam pro náš generický typ.

Nyní je třeba zařídit správné řazení prvků. Myšlenka bude jednoduchá - v rámci třídy bude příznak _isSorted_, který bude říkat, zda je vnitřní seznam _inner_ aktuálně seřazený nebo ne. Kdykoliv někdo do kolekce nový prvek přidá, kolekce se tak může stát neseřazenou (protože nově přidaný prvek může být menší než poslední prvek) a tedy příznak _isSorted_ nastavíme na hodnotu _false_. Kdykoliv chce někdo z kolekce vyjmout prvek pomocí metody _get()_, před vrácením prvku podle indexu zkontrolujeme, zda je vnitřní seznam setřízen, a pokud není, před vrácením provedeme seřazení (nové boky kódu označeny tučně).

class SortedList\<T>{

**private boolean isSorted = false;**

private List\<T> inner = new ArrayList();

public void add (T item){

inner.add (item);

**isSorted = false;**

}

public T get(int index){

**if (isSorted == false)**

**reSort();**

return inner.get(index);

}

private void reSort(){

…

}

}

Přibyla metoda _ReSort()_, která provádní přetřízení kolekce na požádání. Posledním krokem bude doplnění jejího kódu.

class SortedList\<T>{

private boolean isSorted = false;

private List\<T> inner = new ArrayList();

public void add (T item){

inner.add (item);

isSorted = false;

}

public T get(int index){

if (isSorted == false)

reSort();

return inner.get(index);

}

private void reSort(){

**Collections.sort(inner);**

**isSorted = true;**

}

}

Tento kód ale nepůjde přeložit - objeví se poměrně komplikované chybové oznámení, že:

Error

…

method java.util.Collections.\<T>sort(java.util.List\<T>) is not applicable (inferred type does not conform to declared bound(s) inferred: T bound(s): java.lang.Comparable\<? super T>) SortedListExamples.java

Tato komplikovaná hláška upozorňuje na jednu důležitou věc - snažíme se střídit kolekci voláním _Collections.sort(inner);_ a jak bylo ukázáno v předchozím textu (kapitola 7.2.4), pro porovnávání prvků se používá nativní řazení. Aby ale nativní řazení fungovalo, musí porovnávaný prvek implementovat rozhraní _Comparable_. My jsme sice použili generický parametr _T_, aby třída _SortedList_ pracovala s konkrétním typem, ale už nikde není zaručeno, že typ nahrazující generický parametr _T_, který použije programátor, bude implementovat i toto rozhraní. Kompilátor je toto schopen zjistit a nedovolí překlad programu, pokud toto nezajistíme - nevynutíme tedy, aby generický parametr _T_ navíc neimplementoval i rozhraní _Comparable_.

Pro zajištění dodatečných požadavků na generický parametr použijeme tzv. omezení nad generickým parametrem. Bližší objasní doplnění zápisu do třídy.

class SortedList<**T extends Comparable\<T>**>{

…

}

Omezení se tedy zapisuje jako klasická dědičnost/implementace rozhraní do lomených závorek k omezovanému typu. Výše uvedený zápis tedy znamená, že při vytváření instance může programátor generický parametr nahradit libovolným typem, ale pouze takovým, který implementuje generické rozhraní _Comparable_ pro daný typ.

Od této chvíle programátor, který bude chtít definovat proměnnou typu _SortedList_, musí do lomených závorek uvést typ, který implementuje toto rozhraní; v opačném případě program nepůjde přeložit[\[31\]](https://word2md.com/#footnote-31).

Před vysvětlením, jak je schopen kompilátor vlastně takové chování vyžádat, bude představena ještě jedna důležitá problematika - tzv. _widlcards_.

### Generika a dědičnost

Generické typy lze klasickým způsobem začlenit a používat v hierarchii dědičnosti. Lze tedy jak:

* odvodit nový generický typ z negenerického předka,
* odvodit nový generický typ z generického předka,
* odvodit nový negenerický typ z generického předka.

Všechny tyto vazby platí jak v dědičnosti tříd, tak v případech, kdy třída implementuje (potenciálně generické) rozhraní a stejně tak v případě, kdy rozhraní dědí jiné rozhraní. Ve všech případech platí jednoduché pravidlo - pokud používáme jako předka generický typ, musíme buď konkrétně specifikovat jeho generické argumenty, nebo použít generické vytvářeného potomka. Samozřejmě, přístupy lze kombinovat. Jasněji asi vše ukáže následující jednoduchý příklad.

interface NonGenericMapForIntAndString extends Map<**Integer, String**> {}

interface GenericMapForIntKey<**T**> extends Map<**Integer, T**> {}

interface GenericMap<**K, T**> extends Map<**K, T**> {}

Vytvořený potomek dědí z předka, který je generický a má dva generické parametry. Pro oba parametry musíme tedy určit buď konkrétní typ, nebo udělat generickou i naši třídu a předat jako generický parametr předkovi generický parametr potomka. Potud je tedy problematika celkem jasně čitelná a srozumitelná.

V určitých případech zejména začínající programátoři narazí na problém kombinace dědičnosti a generických typů, zejména v případě kolekcí. Uvažujme následující příklad[\[32\]](https://word2md.com/#footnote-32):

Zoo potřebuje realizovat klece pro zvířata. Uvažujme lvy a motýly. Lze si představit obecného předka - _zvíře_ s odpovídající třídou, stejně jako samostatné třídy reprezentující lvy a motýly. Lev i motýl je samozřejmě druhem zvířete a proto i třídy jsou potomky odpovídajícího předka.

abstract class Animal {}

class Lion extends Animal {}

class Butterfly extends Animal {}

Uvažujme obecnou klec pro nějaký objekt - tedy něco jako kolekci, ale navíc se schopností zadržet dovnitř uložené prvky a nepustit je ven (to je základní požadavek na klec). Definice klece tedy bude využívat obecné generické rozhraní pro kolekce. Objekt v kleci také stanovíme genericky - zatím neomezujeme, pro co je klec určena.

class Cage\<E> implements java.util.Collection\<E> {…}

Dalším krokem lze ukázat jednoduché vytvoření klece pro lvy i pro motýly. Klec pro lvy jistě bude mít silné mříže, protože potřebujeme, aby lev neutekl. A zjevně, do klece pro lvy tedy lva dát můžeme.

Cage \<Lion> lionCage = …;

Lion king = new Lion();

lionCage.add(king);

Obdobně, klec pro motýly bude muset být specifická naopak velmi malou vzdáleností mříží, aby nám motýli mezerou mezi nimi neulétli.

Cage \<Butterfly> butterflyCage = …;

Butterfly butterfly = new Butterfly();

butterflyCage.add(butterfly);

Zjevně, do klece pro lvy motýla dát nemůžeme a naopak, motýlí klec by lva také neudržela. To vše v našem případě zajistí generika a typovost klecí.

Uvažujme ale obecnou klec pro zvířata - která by uměla zadržet libovolné zvíře. Jaký bude vztah klece pro lvy a této klece? Proč by ne? Vždyť pokud máme skupinu lvů, tak skupina obecných zvířat by měla být více generickým pojmem a neměl by tedy být problém přiřadit skupinu lvů do skupiny obecných zvířat. Tuto myšlenku a z ní vyplývající běžnou programátorskou chybu ukazuje následující kód.

Cage\<Animal> animalCage = …;

Animal someAnimal = king;

Animal otherAnimal = butterfly;

**animalCage = lionCage;**

**animalCage = butterflyCage;**

Červeně označené příkazy **však nebudou fungovat**.

Klec pro zvířata však nebude víc obecná než klec pro lvy a motýly, ale naopak více specifická - musí mít mříže nejen silné (aby udržela lvy), ale také dostatečně husté (aby zadržela motýly). Proto tyto příkazy provést nelze - mezi _animalCage_ a _lionCage_ opravdu není žádný vztah dědičnosti, ten je pouze u jejich generických parametrů.

### Wildcards

Z výše uvedeného vyplývá, že nemůžeme jednoduše převádět kolekce obsahující potomky určité třídy do proměnné deklarované na kolekci obsahující předka určité třídy. Často se ale stane, že potřebujeme automaticky procházet přes kolekce a nad všemi prvky provést určitou operaci, kterou podporuje jejich předek. Jednoduchým příkladem může být krmení zvířat ve výše zmíněných klecích. Protože krmení je častá věc a dít se může opakovaně, chtěli bychom vytvořit funkci, která nakrmí všechna zvířata v dané kleci a tuto funkci používat opakovaně. Jak ale předat funkci jako parametr klec, která se má krmit:

* Můžeme udělat množství funkcí, pro každý typ klece jednu. Funkcí tedy bude tolik, kolik bude typů zvířat, což velmi zvýší nepřehledost, časovou náročnost a údržbu kódu.
* Můžeme udělat jenom jednu funkci a specifikujeme, že jako parametr do ní bude vstupovat klec pro nějaké konkrétní (ale zatím nevíme jaké) zvíře.

Druhá varianta je samozřejmě programátorsky mnohem výhodnější - bude existovat pouze jedna funkce, bude fungovat univerzálně a případná nová zvířata (tj. nové třídy) nebudou generovat nové funkce.

Pro zápis takového parametru použijeme již známé lomené závorky z generik a navíc klíčové slovo _extends_ z dědičnosti.

private void feedAnimalsInCage (Cage<**? extends Animal**> cage){

for (Animal a : cage){

// feed animal "a"

}

}

Zvláštností je použití znaku „?", který je právě chápán jako wildcard (žolík, divoký znak) reprezentující nějaký typ, který kompilátor nezná. Ví o něm ale, že tento typ bude potomkem (_extends_) třídy „Animal". Kompilátor nyní bude schopen zkontrolovat, že při volání funkce _feedAnimalsInCage()_ programátor předá jako proměnnou takový typ, který bude obsahovat instanci _Cage\<T>_, kde pro _T_ platí, že je potomkem třídy _Animal_.

Důvod použití je opět možnost práce s běžnými metodami třídy _Animal_ uvnitř metody _feedAnimalsInCage()_. Díky tomu, že jsme specifikovali, že v kleci bude „něco, co je zvíře", můžeme udělat obecný for-each cyklus nad prvky klece a specifikovat, že jeden prvek klece je zvíře (Animal) a volat nad proměnnou _a_ odpovídající metody.

**Poznámka.** Toto je způsob, jakým kompilátor zjistil, že ve výše uvedeném příkladu v kapitole 8.1 nemůže zavolat metodu sort() a zahlásil podivnou chybu. Metoda sort() totiž vyžaduje, aby byla volána pouze s parametrem takového typu, který implementuje nativní řazení.

**Poznámka.** Někdy se může hodit situace opačná - kdy potřebujeme specifikovat, že jako parametr do funkce vstupuje generický typ, jehož generický parametr je **předkem** svého obsahu - typicky v případech, kdy do funkce předáváme jako parametr kolekci, do které chceme prvky přidávat. Tehdy se použije klíčové slovíčko opačné - tedy „super"[\[33\]](https://word2md.com/#footnote-33).

### Generické metody

Poslední představenou oblastí z problematiky generik budou generické metody.

Je zjevné, že pokud vytvoříme generickou třídu, generický parametr této třídy můžeme kdekoliv uvnitř třídy používat jako běžný jiný typ.

Uvažujme, že potřebujeme ze tří parametrů (obecně libovolného typu) vybrat ten, který je zmíněn jako první a není _null_. Zároveň potřebujeme zajistit, aby všechny parametry byly (sice libovolného), ale stejného typu.

Pro takové chování lze vytvořit jednoduchou třídu _NotNullAnalyser_ a v ní jednoduchou metodu _getFirstNotNull()_. Třída bude generická a bude tedy fungovat univerzálně pro ten typ, který programátor později specifikuje.

class NotNullAnalyser\<T>{

public T getFirstNotNull (T a, T b, T c){

T ret = null;

if (a != null)

ret = a;

else if (b != null)

ret = b;

else if (c != null)

ret = c;

return ret;

}

}

V metodě _getFirstNotNull()_ generický parametr _T_ opravdu používáme jako zcela normální, jinou třídu. Následuje příklad použití.

NotNullAnalyser\<Integer> nna = new NotNullAnalyser();

Integer a = null;

Integer b = null;

Integer c = 15;

Integer result;

result = nna.getFirstNotNull(a, b, c);

System.out.println(result);

Otázka zní - je třeba opravdu pro takový případ, kdy potřebujeme, aby nám genericky fungovala metoda, vytvářet celou třídu? Odpověď je zjevná již z názvu kapitoly - ne.

Jazyk Java totiž umožňuje vytvářet i samotné generické metody, a to **i v třídách, které samy o sobě generické nejsou**.

Generická metoda se vytváří jako klasická metoda - navíc těsně před deklaraci návratového typu ale přibyde již známá syntaxe definující generické parametry, které lze v rámci metody použít.

public static **\<T>** T getFirstNotNull (T a, T b, T c){

T ret = null;

if (a != null)

ret = a;

else if (b != null)

ret = b;

else if (c != null)

ret = c;

return ret;

}

}

Opět, takto generický parametr lze použít kdekoliv v těle (i parametrech, návratovém typu) funkce. I zde můžeme používat klíčové slovo _extends_ na omezení použitelných typů, pro které bude funkce volána a typů může být obecně libovolný počet. Znovu zdůrazňujeme že:

* tato metoda může být generována i v „běžném, negenerickém" typu,
* tato metoda může využívat i generické parametry třídy, pokud je definována v generické třídě; tehdy může tedy používat jak generické parametry vlastní, tak generické parametry třídy, ve které je uvedena.

Klasickým příkladem generické metody je již zmiňovaná metoda _sort()_ ze třídy _Collections._ Tato třída generická není, má však v sobě definovánu tuto generickou metodu. Pro zajímavost, signatura této metody vypadá následovně:

public static \<T extends Comparable\<? super T>> void sort(\
List\<T> list)

Výraz generického parametru lze přečíst jako „typ T, který implementuje rozhraní _Comparable_ pro nějaký jiný typ takový, že náš typ _T_ je jeho předkem."

**Kontrolní otázky:**

* Co je generický typ?
* Co je generický parametr?
* Co vše může sloužit jako generický parametr?
* Kdy je vhodné mít generickou metodu a kdy je vhodné mít celou generickou třídu?

Úkoly k zamyšlení:

Na jaký typ úlohy byste ještě zkusili generika použít?

Korespondenční úkol:

Výsledek z korespondenčního úkolu z předchozí kapitoly přepište do generického tvaru.

Shrnutí obsahu kapitoly

Kapitola představila problematiku generických typů a jejich použití na úlohy, kdy je třeba provádět obdobnou činnost pro různé datové typy stejným způsobem. Zmíněny byl také vyhody a nevýhody těchto přístupů, použití v dědičnosti, použití _wildcards_ a použití generických metod.
