# TODO Práce se soubory

Po představení operací pro procházení a práci se složkami a soubory následuje část vysvětlující problematiku práce s daty uvnitř souboru. Obecně lze v Javě rozdělit přístupy na tři základní:

* Nejjednodušší práce pomocí zapouzdřených funkcí, typicky vhodná pro jednoduché textové soubory s využitím balíčku `java.nio.files`.
* Pokročilejší techniky práce s proudy umožňující plné přizpůsobení techniky čtení a zápisu dat s využitím balíčku `java.io`.
* Nejnovější techniky práce s kanály umožňující plné přizpůsobení techniky čtení dat, navíc rozšiřitelné o možnosti zamykání, souběžného přístupu, mapování do paměti a dalších, s využitím balíčku `java.nio`.

Poslední přístup vysvětlen nebude, jeho použití je vhodné už u pokročilejších aplikací a v případě potřeby je třeba se s ním seznámit individuálně.

## Práce se soubory s třídou Files

Nejjednodušší práce se soubory je vhodná na problematiku malých (typicky textových) souborů a spočívá v načtení celého textového do nějaké proměnné, či naopak, v uložení dat do cílového souboru. Obě operace podporuje svými metodami třída `java.nio.file.Files`. Všechny metody očekávají zadání názvu souboru nikoliv jako řetězec, ale jako instanci od výše představené třídy `java.nio.file.Path`. Většina operací samozřejmě také typicky vyvolává výjimku `java.io.IOException`, kterou je třeba zachytit a ošetřit.

<table><thead><tr><th width="159">Metoda</th><th width="421">Popis</th></tr></thead><tbody><tr><td><code>readAllBytes()</code></td><td>Načte obsah souboru jako pole bytů.</td></tr><tr><td><code>write(...)</code></td><td>Uloží pole bytů do souboru.</td></tr><tr><td><code>readAllLines()</code></td><td>Načte ze souboru všechny řádky jako <strong>kolekci řetězců</strong>. Při načítání se použije kódová stránka zadaná jako druhý parametr volání funkce.</td></tr><tr><td><code>write(...)</code></td><td>Druhé přetížení stejné metody umožňuje zapsat do souboru řetězec (či pole řetězců) se zadanou kódovu stránkou.</td></tr></tbody></table>

Práce s polem bytů je jednoduchá, a proto nebude představena. Složitější je problematika řetězců, které při načítání potřebují vědět, v jaké kódové stránce se mají zpracovat. Programátor tedy musí vědět, v jaké kódové stránce byl soubor uložen, jinak buď soubor nepůjde vůbec načíst (a celá operace skončí vyhozenou výjimkou), nebo načtené znaky nebudou odpovídat znakům, které programátor očekával.

Kódová stránka je reprezentována instancí třídy `java.nio.charsets.Charset` a nová instance se nejjednodušším způsobem získá voláním statické metody `forName` této třídy, které se dá jako parametr název kódové stránky (například UTF8, ASCII atd.).

Následuje jednoduchá ukázka načtení obsahu souboru do proměnné typu kolekce řetězců.

```java
String fileName = "D:\\demo.txt";
java.nio.file.Path filePath = java.nio.file.Paths.get(fileName);
java.nio.charset.Charset charset =
    java.nio.charset.Charset.forName("UTF8");
java.util.List<String> content = null;
try {
    content = java.nio.file.Files.readAllLines(filePath, charset);
} catch (IOException ex) {
    System.out.println(
        "Unable to read content of file " + fileName);
}
```

V programu se připravil název souboru, převedl se na cestu, připravila se odpovídající kódová stránka a následně se načte celý obsah souboru do proměnné `content`. Pozor, že obsah souboru se nenačítá jako jeden řetězec, ale jako kolekce řetězců reprezentujících jednotlivé řádky. Pokud chce programátor získat prostý řetězec, musí kolekci spojit programově v jeden objekt.

Uložení textu do souboru je obdobně jednoduché. Důležité je však, že i zde se používají funkce využívající proměnného počtu argumentů. Posledním argumentem je volitelně jedna, nebo více následujících hodnot výčtového typu `StandardOpenOptions`, který nabývá mj. hodnoty uvedené v tabulce.

<table><thead><tr><th width="206">Hodnota</th><th width="380">Význam</th></tr></thead><tbody><tr><td>WRITE</td><td>Otevře soubor pro zápis.</td></tr><tr><td>APPEND</td><td>Při zápisu se bude přidávat na konec souboru. Používá se v kombinaci s WRITE nebo CREATE.</td></tr><tr><td>TRUNCATE_EXISTING</td><td>Smaže obsah otevíranému souboru (soubor se tedy otevře, smaže se jeho obsah a případný zápis se provádí od začátku souboru). Používá se v kombinaci s WRITE.</td></tr><tr><td>CREATE_NEW</td><td>Otevře nový soubor. Pokud již soubor existuje, vyhodí se výjimka.</td></tr><tr><td>CREATE</td><td>Otevře soubor. Pokud soubor neexistuje, vytvoří se nový.</td></tr><tr><td>DELETE_ON_CLOSE</td><td>Smaže soubor po ukončení práce. Používá se pro dočasné soubory, které poté programátor nemusí mazat ručně.</td></tr></tbody></table>

Pokud chceme tyto konstanty používat v programu přímo (tak jak ukazuje následující příklad), musíme na začátek souboru do sekcí importů přidat statický import.

```java
import static java.nio.file.StandardOpenOption.*;
```

Níže uvedený příkaz tedy ukáže, jak připsat (tedy přidat na konec) řetězec do souboru, který již existuje.

```java
String fileName = "D:\\demo.txt";
java.nio.file.Path filePath = java.nio.file.Paths.get(fileName);
java.nio.charset.Charset charset =
    java.nio.charset.Charset.forName("UTF8");

java.util.List<String> lines = new java.util.ArrayList ();
lines.add("První řádek.");
lines.add("Druhý řádek.");

try {
    java.nio.file.Files.write(filePath, lines, charset,
        WRITE, APPEND);
} catch (IOException ex) {
    System.out.println("Unable to write content to file "
        + fileName + " :" + ex.toString());
}
```

A obdobný příklad volání, ale tentokrát pokud potřebujeme soubor přepsat (původní obsah tedy nechceme zachovat).

```java
java.nio.file.Files.write(filePath, lines, charset,
    WRITE, TRUNCATE_EXISTING);
```

## Práce s proudy

Původní technika práce se soubory vycházel z pojmu tzv. _proudů_ - _stream_.

Proud je obecný, abstraktní producent, nebo konzument dat. Obsahuje tedy jednoduchou metodu `read()` nebo `write()`, pomocí které proud data dodává, případně přijímá. Podle toho se proudy dělí na vstupní (ty, které dodávají data) a výstupní (ty, které přijímají data).

Proudem může být obecně cokoliv, co má tuto schopnost vůči vytvářenému programu. Typicky soubory (soubor pro zápis je schopen přijímat data, soubor pro čtení je schopen je dodávat), připojení k síti (zapisující proud posílá data z počítače pryč, čtecí proud přijímá data ze sítě), ale také například klávesnice (vstupní proud dodávající stisknuté klávesy) nebo tiskárna (výstupní proud posílající data tiskáren). Z běžně používaných tříd se programátor setkává nejčastěji s proudy výstupu a čtení z konzole (`System.in` a `System.out`)

V Javě povinně je proud vždy **buď vstupní, nebo výstupní.**

Druhé dělení proudů je podle typu dat, se kterými proud pracuje. Opět, dělí se na dvě skupiny:

* **Textové proudy**, zapisující nebo čtoucí text (tj. znaky);
* **Bytové proudy**, zapisující nebo čtoucí byty.

Každá z vytvořených variant má svůj typický název, který se používá jako postfix u názvů tříd a podle něj tedy lze u názvu třídy zjistit, o jaký typ proudu se jedná.

<table><thead><tr><th width="169">Směr</th><th width="175">Typ dat</th><th width="208">Název</th></tr></thead><tbody><tr><td>Vstupní</td><td>Byty</td><td>InputStream</td></tr><tr><td>Výstupní</td><td>Byty</td><td>OutputStream</td></tr><tr><td>Vstupní</td><td>Znaky</td><td>Reader</td></tr><tr><td>Výstupní</td><td>Znaky</td><td>Writer</td></tr></tbody></table>

Celá problematika proudů staví na abstrakci, dědičnosti a rozhraních. Základy, jak bylo zmíněno, jsou pro všechny proudy shodné. Budeme uvažovat příklad typu `InputStream`. Pokud to bude proud dat z klávesnice, bude tento proud obsahovat byty reprezentující jednotlivé klávesy, které může programátor postupně po jednom načítat metodou `read()`. Pokud to bude proud dat ze síťové karty, bude tento proud obsahovat byty, které přišly do počítače ze sítě a které může programátor postupně po jednom načítat metodou `read()`. Pokud to bude vstupn z externí karty, odkud chodí nějaká data (například GPS modul), může tato data programátor načítat postupně po jednom metodou `read()`. Jak je zjevné, způsob práce se pro programátora nemění (vždy jej zajímá načítání pomocí metody `read()`, které mu vrací jednotlivé byty), odlišná je pouze implementace. Od této implementace je však programátor odstíněn, protože programuje proti rozhraní nebo abstraktní třídě. Programátor tedy **vůbec nemusí vědět, na jakém principu daný proud funguje**, pouze jej zajímá způsob získání (případně vložení) dat.

Proto pro každou ze 4 skupin uvedených výše existuje abstraktní třída - předek všech proudů tohoto typu, který implementuje právě požadovanou metodu `read()/write()` a nějaké pomocné metody navíc. Tvůrce proudu tuto třídu podědí a požadovanou logiku naimplementuje dle potřeby. Tito předci se jmenují stejně jako název typu proudu a jejich název se dává jako postfix (přípona) názvů proudu od nich odvozených.

Dále budeme představovat tuto problematiku na souborech, ale obecné principy se vztahují na všechny proudy.

Při práci se zdrojem pomocí proudu musí programátor již sám ošetřovat některé záležitosti, a proto základ práce spočívá vždy nejméně ve třech krocích:

* Otevření zdroje (proudu)
* Práce s daty proudu (čtení, nebo zápis, podle typu proudu)
* **Uzavření zdroje**

Otevření spočívá ve vytvoření instance proudu - vytvořená instance automaticky otevírá zdroj, ze kterého bude čerpat (nebo kam bude zapisovat). Typicky se otevírání provádí pomocí přetíženého konstruktoru, kde jeho parametry udávají způsob a cíl otevření. Na tento bod typicky programátoři nezapomínají, protože bez nich program nebude fungovat.

Neméně důležité je však uzavření zdroje po konci používání. Tím se zdroj uvolní pro ostatní aplikace, které s otevřeným zdrojem nemohou pracovat. Typickou ukázkou nezavřeného zdroje jsou chybové hlášky typu „soubor nelze smazat, neboť je dosud používán" apod. Pro uzavření svého zdroje mají všechny proudy metodu `close()`. Opakované volání této metody nezpůsobí žádnou chybu.

### Čtení ze souboru pomocí proudů

Představen bude výše uvedený příklad čtení dat ze souboru. Kvůli trochu větší náročnosti tvorby bude vytvořen po částech a postupně do něj budou doplňovány složitější myšlenky a konstrukce.

Základem je otevření a uzavření zdroje.

```java
String fileName = "D:\\demo.txt";
java.io.FileReader fileReader = null;

fileReader = new java.io.FileReader(fileName); // <-- otevření zdroje
// ...
fileReader.close(); // <-- uzavření zdroje
```

První zvýrazněný řádek provádí otevření souboru. Druhý zvýrazněný řádek ukazuje na způsob uzavření souboru.

Ani jeden ze zvýrazněných příkazů však nepůjde přeložit - první příkaz totiž vyhazuje výjimku v případě, že soubor není nalezen. Proto ji ošetříme příkazem _try-catch-finally_.

```java
String fileName = "D:\\demo.txt";

java.io.FileReader fileReader = null;
try {
    fileReader = new java.io.FileReader(fileName); 
    // ...
} catch (FileNotFoundException ex) {
    System.out.println("File with name "
        + fileName + " was not found: " + ex.toString());
} finally {
    fileReader.close(); // <-- uzavření
}
```

Povšimněte si způsobu ošetření v bloku finally. Proud `fileReader` se zavíráme v bloku `finally` vždy, ale pouze pokud se jej podařilo dříve otevřít (tedy není `null`). V bloku `try` již zavírání provádět nebudeme, protože blok `finally` se provede vždy.

I tentokrát však kód nepůjde přeložit, protože i zavření souboru může způsobit chybu a vyvolat výjimku. I zde bychom museli doplnit bloky try-catch. O způsobu, jak korektně ošetřit případ, že se soubor nepodařilo zavřít, lze dlouho polemizovat - v našem případě pouze vypíšeme chybu uživateli na konzoli.

Tento způsob (uzavírání přes `finally`) se však dneska kvůli složitosti nepoužívá a místo toho kód přepíšeme s využitím _try-with-resources_.

```java
String fileName = "D:\\demo.txt";

try (java.io.FileReader fileReader = new java.io.FileReader(fileName)) {
    // práce se souborem
    ...
} catch (FileNotFoundException ex) {
    System.out.println("File with name "
            + fileName + " was not found: " + ex);
} catch (IOException ex) {
    System.out.println("I/O error while working with file "
            + fileName + ": " + ex);
}
```

Nyní můžeme pokračovat. Při čtení ze souboru se používá funkce `read()`, která v případě, že již nejsou žádná další data k dispozici, vrací hodnotu -1.

```java
String fileName = "D:\\demo.txt";

try (java.io.FileReader fileReader = new java.io.FileReader(fileName)) {
    int i = fileReader.read();

    while (i != -1) {
        char c = (char) i;
        System.out.print(c);
        i = fileReader.read();
    }

} catch (FileNotFoundException ex) {
    System.out.println("File with name "
            + fileName + " was not found: " + ex);
} catch (IOException ex) {
    System.out.println("Error while reading file "
            + fileName + ": " + ex);
}
```

Funkce `read()` však také vrací výjimku v případě chyby - proto musíme k blokům `catch` doplnit další reflektující tuto chybu. Funkce také vrací `int` a nikoliv `char` (aby bylo možno vrátit hodnotu -1) a proto hodnotu před výpisem musíme explicitně přetypovat na `char`, jinak by se na konzoli ukazovaly číselné kódy vypisovaných znaků.

Zápis do souboru bude předveden až po další technice, která umožňuje pracovat s proudy, a to je vnořování proudů.

### Vnořování proudů

Jak bylo řečeno, proud je hodně obecné řešení, které programátorovi umožňuje uvnitř implementované třídy realizovat libovolnou operaci, ke které potřebuje pouze dodat nebo vrátit nějaká data. Tento typ operací je poměrně běžný - nejlépe názornou ukázkou je například komprimace: jedná se o operaci, kdy někdo vloží nějaká data, s těmi se provede operace (komprimace) a výsledek se předává dále. Dekomprimace naopak na požádání odněkud vezme data, dekomprimuje je a vrátí výsledek tazateli.

V principu se u komprimace tedy jedná o proudové řešení, které někdo vloží nějaká data, ta se zkomprimují a dále se vloží do dalšího proudu, který s nimi udělá další operaci (například je uloží na disk). Do komprimačního proudu se tedy **vloží** další proud, kterému komprimační proud předává data po komprimaci. Proudům, které dělají nějakou operaci a výsledek předávají do v nich vnořeného proudu, se říká _filtrační proudy_. Jejich konstruktor typicky obsahuje přetížení, do něhož se předává jako parametr proud, který fungovat bude vnořený.

Jako praktický příklad může sloužit demonstrace čtení obsahu souboru - již bylo představeno čtení po znacích, ale typickou úlohou je čtení celých řádků. Pro tento účel slouží proud nazvaný `BufferedReader` (podle postfixu je to tedy proud pro čtení textových dat). Ten je filtrační a potřebuje pod sebe podkladový proud, ze kterého bude tato data číst.

```java
String fileName = "D:\\demo.txt";

try (java.io.BufferedReader bufferedReader =
         new java.io.BufferedReader(new java.io.FileReader(fileName))) {

    String line = bufferedReader.readLine();

    while (line != null) {
        System.out.println(line);
        line = bufferedReader.readLine();
    }

} catch (FileNotFoundException ex) {
    System.out.println("File with name "
            + fileName + " was not found: " + ex);
} catch (IOException ex) {
    System.out.println("Error while reading file "
            + fileName + ": " + ex);
}
```

{% hint style="info" %}
U `try` si můžeme všimnout vícenásobné inicializace více zdrojů - `FileReader` i `BufferedReader`. Oba se uzavřou automaticky po opuštění `try-catch`.

Řesení lze zapsat i s vytvořením dvou proměnných (je-li třeba) jako:

```java
...
try (
    java.io.FileReader fileReader = new java.io.FileReader(fileName);
    java.io.BufferedReader bufferedReader = new java.io.BufferedReader(fileReader)
) {
...
```
{% endhint %}

### Zápis do souboru pomocí proudů

Opačný směr zápisu bude předveden na upraveném posledním, výše uvedeném výpisu kódu určeného ke čtení ze souboru.

Princip je zcela shodný se čtením, jenom tentokráte provádíme zápis, hledáme tedy ne třídu `Reader`, ale `Writer`. Prefixy názvů třídy typicky zůstávají stejné, mění se pouze koncovky, proto nám u použitých tříd proudů stačí nahradit `BufferedReader` za `BufferedWriter` a `FileReader` za `FileWriter`.

Dalším krokem je samozřejmě nahrazení algoritmu čtení nějakým zápisem. Ukázaný příklad zapíše do souboru dva řádky textu.

```java
String fileName = "D:\\demo.txt";

try (java.io.BufferedWriter bufferedWriter =
    new java.io.BufferedWriter(
        new java.io.FileWriter(fileName))) {
    bufferedWriter.write("Toto je první řádek.\r\n");
    bufferedWriter.write("Toto je druhý řádek.\r\n");
} catch (IOException ex) {
    System.out.println("Error while reading file "
        + fileName + ": " + ex.toString());
}
```

Ohledně zápisu do výstupních proudů je třeba zmínit ještě jednu problematiku - zápis na výstupní zařízení je typicky velmi pomalá operace (zvláště u souborů zapisovaných na klasický pevný disk) a zapisování dat postupně (například po jednom bytu) velmi výrazně sníží rychlost běhu aplikace. Z toho důvodu ty výstupní proudy, u kterých by tento problém mohl nastat, používájí tzv. `bufferování` dat - tzn. při požadavku na zápis nezapíší data ihned, ale počkají, až se jich nashromáždí větší množství a data zapíší najednou. U tohoto přístupu je však nebezpečí, že pokud aplikace spadne neřízeně dříve, než se připravená data opravdu fyzicky zapíší do cíle proudu, k žádnému zápisu nedojde. Pokud chceme tedy v určitou chvíli explicitně vynutit, aby se data zapsala do cílového zdroje bez ohledu na jejich množství, zavoláme nad výstupním proudem metodu `flush()`. Tato metoda se volá také samozřejmě automaticky, kdykoliv je výstupní proud uzavírání.

### Další typy proudů

Jak již bylo zmíněno, typy proudů se liší podle zdroje, se kterým pracují - princip práce ale pro programátora zůstává stejný. Typicky také platí, že pokud existuje určitý typ proudu pro zapisování, existuje obdobný typ proudu pro čtení. Pro představu o základních možnostech proudů jazyka Java představuje následující tabulka další běžně používané typy proudů.

| Název třídy                                                                       | Použití                                                                                                                                                                                                                                        |
| --------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| <p><code>ByteArrayInputStream</code></p><p><code>ByteArrayOutputStream</code></p> | Proudy pro čtení/zápis do pole bytů. Programátor tak nemusí k poli bytů přistupovat pomocí indexů, ale vytvoří si polem proud a jednotlivé byty čte nebo zapisuje postupně.                                                                    |
| <p><code>PushbackInputStream</code></p><p><code>PushbackReader</code></p>         | Přidává funkcionalitu čtení dalšího znaku bez jeho výběru. Nad vnořeným streamem tedy umí provést operaci, kdy se „podívá", co je další znak.                                                                                                  |
| `SequenceInputStream`                                                             | Umožňuje zařadit čtení několika proudů za sebou. Jakmile jsou vyčtena všechna data z proudu prvního, pokračuje čtení proudem dalším. Vstupní proudy se zadávají v konstruktoru této třídy.                                                     |
| <p><code>PrintStream</code></p><p><code>PrintWriter</code></p>                    | Alternativa k proudu OutputStream. Příkladem může být _System.out_. Umí lépe (smysluplněji) vypisovat metodou _write()_ jiné datové typy; hlavně však u těchto operaci nevyhazuje výjimky, takže se výpisy nemusí balit do sekvence try-catch. |
| <p><code>CharArrayReader</code></p><p><code>CharArrayWriter</code></p>            | Obdoba typu _ByteArrayInputStream_, podkladovými daty je však pole znaků (nikoliv bytů).                                                                                                                                                       |

## Serializace

S proudy souvisí ještě jedna programátorsky poměrně zajímavá technika, a to je _**serializace**_.

Programátor při tvorbě programu typicky vytváří třídy, které reprezentují data, se kterými program pracuje. Dříve či později se dostane do situace, kdy tato data potřebuje uložit - zajistit, aby byla **perzistentní** vůči aplikaci, tedy aby po opětovném spuštění aplikace byla tato data k dispozici. Jednou z možností je podpora ukládání například do databází, u jednoduchých aplikací a určitých typů dat (zejména malého množství) je vhodné provést uložení do souboru na disk.

Pro následující příklady bude uvažována jednoduchá třída. Uvažujeme také metody _get/set_ k jednotlivým třídním proměnným, kvůli jejich trivialitě a nedostatku místa však nejsou ve výpisu uvedeny.

```java
class Person {
    private String name;
    private int age;
    private List<String> phones = new ArrayList();
}
```

Tato jednoduchá třída definuje osobu, která má jméno, věk a seznam telefonních čísel. Dále uvažujeme seznam osob, reprezentovaných jako kolekci instancí výše uvedené třídy.

```java
class PersonCollection extends ArrayList<Person> { }
```

Uživatel (programátor) nějakým způsobem naplní kolekci osob a nyní potřebuje zajistit, aby se tato kolekce uložila do souboru, případně dokázala ze souboru zase úspěšně načíst.

### Vlastní serializace

U první varianty musí programátor vše udělat ručně. Problematiku lze rozložit na dvě části:

* Jak zajistit uložení sady prvků v kolekci.
* Jak zajistit uložení jednoho konkrétního prvku.

Uložení prvku v kolekci zajistí třída `PersonCollection` metodou `save()`, kde parametrem bude název cílového souboru. Kolekce musí zajistit uložení dvou částí: a) uložení počtu prvků v souboru (aby při načítání bylo hned jasné, kolik prvků se bude načítat; b) opakovaného postupného uložení jednotlivých prvků.

Před samotným kódem ještě jedna problematika - chceme-li mít řešení univerzální, platí vlastně, že kolekce se umí uložit do čehokoliv, „do čeho se dá psát" (tedy libovolný výstupní stream) a naopak načítat z čehokoliv, „z čeho se dá číst" (tedy libovolný vstupní stream).

Problematiku zápisu do souboru tedy rozdělíme ještě na dvě části, obě implementované stejně nazvanou metodou s různým přetížením:

* Řešení práce se souborem
* Samotný zápis dat do proudu

Práci se souborem zajistí jednoduchá metoda `save()`, která přijme jako parametr název souboru. Tato metoda řeší otevření a zavření souboru a předání chyby, ale **neřeší vůbec samotné uložení dat**. Pro tuto část využije přetížení, které přijímá jako parametr zapisovací proud. Tato metoda ve třídě `PersonCollection` bude vypadat například následovně.

```java
public void save (String fileName) throws IOException {
    try(FileWriter writer = new FileWriter(fileName)){
        this.save(writer);
    } catch (IOException ex){
        throw ex;
    }
}
```

V prvním kroku se pomocí syntaxe _try-with-resources_ otevře zapisovací proud do souboru zadaného jména. V druhém kroku se zavolá přetížená metoda (bude uvedena dále), která provede samotný zápis dat. Konec bloku `try` zajistí korektní uzavření souboru po dokončení operace.

Druhá metoda, realizující samotný zápis, bude přijímat jako parametr obecný `Writer`, tedy jakýkoliv textový výstupní proud - nejsme při ukládání vůbec omezeni na cíl, do kterého se budou data ukládat.

```java
public void save (Writer writer) throws IOException {
    BufferedWriter bufferedWriter = new BufferedWriter(writer);
    bufferedWriter.write(Integer.toString(this.size())); // zapiseme pocet prvku
    bufferedWriter.newLine(); // odradkujeme
    bufferedWriter.flush();
    for (Person p : this){
        p.save(writer); // ukladame postupne osoby
    }
}
```

V prvním kroku k danému zapisovacímu proudu vytvoříme `BufferedWriter`, abychom mohli zapisovat řetězce a znaky nového řádku (což tato třída umí sama).

Další trojpříkaz zapíše do proudu počet položek v kolekci (`this.size()`) a následně připíše znak konce řádku.

{% hint style="info" %}
**Pozor na zápis čísla do proudu!**&#x20;

Metoda `write()` má přetížení přijímající jako parametr `int`, ale zapisuje jej jako char (tj. podle ASCII tabulky nebo kódování, místo 60 tedy napíše písmeno „A" apod.). Cílem je však zapsat řetězec reprezentující dané číslo, proto musíme hodnotu `int` převést na řetězec a teprve tento řetězec zapsat do souboru. O převod čísla na řetězec se nám postará metoda `toString()` třídy `Integer` - celý příkaz tedy je `Integer.toString(\<hodnota\_k\_prevedeni>)`.&#x20;
{% endhint %}

Poslední příkaz zajistí propsání zadaných dat do cílového zdroje pomocí metody `flush()`.

Dále se v cyklu přes všechny osoby v kolekci zavolá metoda `save()` třídy `Person` (tato metoda bude vytvořena v zápětí) a jako parametr se jí předá cílový proud, do kterého se zapisuje.

Tímto kolekce umí uložit veškeré informace, které potřebuje zpět ke svému načtení.

Dalším krokem je implementace metody `save()` u třídy `Person.` Princip zůstává shodný s výše představeným, pouze kód bude trošku delší kvůli většímu množství dat, které tato třída ukládá.

```java
public void save (Writer writer) throws IOException {
    BufferedWriter bufferedWriter = new BufferedWriter (writer);
    bufferedWriter.write(name);
    bufferedWriter.newLine();
    bufferedWriter.write(Integer.toString(age));
    bufferedWriter.newLine();
    bufferedWriter.write(Integer.toString(phones.size()));
    bufferedWriter.newLine();
    for(String phoneNumber : phones){
        bufferedWriter.write(phoneNumber);
        bufferedWriter.newLine();
    }
    bufferedWriter.flush();
}
```

Nejdříve se opět vytvoří `BufferedWriter` pro snazší zápis.

Následuje zápis jména a odřádkování, ihned na další řádek zápis věku a odřádkování.

Princip zápisu telefonních čísel je stejný jako u zápisu počtu prvků kolekce. Nejdříve se zapíše počet telefonních čísel (a odřádkuje se), následně se na každý řádek zapíše jedno telefonní číslo.

Posledním příkazem se zajistí propsání výsledku do zdroje.

Všimněte si, že ani v této metodě se proud `bufferedWriter` **nezavírá!** Zavření proudu by způsobilo zavření i vnořeného proudu a to není cílem, protože by se případné další osoby neměly kam zapsat. Proto je vhodné držet pravidlo, že zavíráme pouze ty fyzické proudy, které jsme sami otevřeli. Filtrační proudy tedy fyzické proudy zavírat nemají.

Následuje velmi krátká a jednoduchá ukázka použití.

```java
PersonCollection persons = new PersonCollection();

Person person = null;
person = new Person();
person.setName("Michal");
person.setAge(31);
person.getPhones().add("605503032");
person.getPhones().add("774839384");
persons.add(person);

person = new Person();
person.setName("Jana");
person.setAge(28);
person.getPhones().add("747873821");
person.getPhones().add("777382948");
person.getPhones().add("698392832");
persons.add(person);

try {
    persons.save("D:\\lide.txt");
} catch (IOException ex) {
    System.out.println(ex.getMessage());
}
```

Tučně označený blok realizuje samotné uložení dat. Nejdříve se vytvoří dvě demonstrační osoby a následně se uloží do souboru D:\lide.txt. Následuje ukázka tohoto souboru - středník a následující znaky na řádku nejsou součástí souboru a slouží pouze jako vysvětlivka uložených dat.

```
2 ; počet záznamů
Michal ; jméno prvního záznamu
31 ; věk prvního záznamu
2 ; počet telefonních čísel prvního záznamu
605503032 ; první telefonní číslo
774839384 ; druhé telefonní číslo
Jana ; jméno druhého záznamu
28 ; věk druhého záznamu
3 ; počet telefonních čísel druhého záznamu
747873821 ; první telefonní číslo
777382948 ; druhé telefonní číslo
698392832 ; třetí telefonní číslo
```

Lze tedy vidět, že uložení dat do souboru vyžaduje trochu programátorské práce a zručnosti. Výhodou je, že výsledný soubor je textový a programátor, který zná jeho obsah, jej může sám upravit a přizpůsobit tak případně data dle potřeby.

Druhou částí je načtení dat z tohoto souboru. Opět budeme pracovat stejným způsobem, jenom funkce nazveme `load()`. Důležitou odchylkou je ale vazba funkce ke své instanci. Zatímco funkce `save()` ví, kterou instanci ukládá (tu instanci, nad kterou byla zavolána), funkce `load()` v době svého vykonání ještě žádnou instanci nemá, teprve se ji snaží načíst ze souboru. Z toho důvodu musí být funkce `load()` statické. Vytvořenou instanci budou vracet jako svou návratovou hodnotu.

Začneme opět třídou `PersonCollection` a prací se souborem.

```java
PersonCollection ret = null;

try (FileReader reader = new FileReader(fileName)){
    ret = PersonCollection.load(reader);
} catch (IOException ex){
    throw ex;
}
return ret;
```

Opět, cílem je pouze otevřít soubor do instance třídy `FileReader` pomocí bloku _try-with-resources_. O načtení dat se postará přetížená metoda `load()`, které předáme jako parametr vstupní textový proud.

Trošku složitější je problematika načítání z vstupního textového proudu. Pro jednoduchou práci si pomůžeme vytvořením filtrovacíhou proudu `BufferedReader` - ten však již podle názvu data _bufferuje_, tedy si je pro svou potřebu přednačte do svých vnitřních struktur. Díky tomu, že data již byla načtena, nemůžeme dál předávat původní proud `reader`, protože v něm již žádná data být nemusí. Proto budeme dál třídě `Person` předávat přímo instanci `bufferedReader` a nikoliv `reader` jako u ukládání.

```java
public static PersonCollection load (Reader reader) throws IOException {
    String tmp;
    int count;
    PersonCollection ret = new PersonCollection();
    BufferedReader bufferedReader = new BufferedReader(reader);
    tmp = bufferedReader.readLine();
    count = Integer.parseInt(tmp);
    for (int i = 0; i < count; i++) {
        Person person = Person.load(bufferedReader);
        ret.add(person);
    }
    return ret;
}
```

V prvním kroku vytvoříme pomocnou proměnnou `tmp` a proměnnou `count`, do které později uložíme počet načítaných osob. Dále vytvoříme instanci objektu `PersonCollection`, který budeme vracet. Nakonec vytvoříme instanci `bufferedReaderu`.

Na začátku načtení nejdříve načteme ze vstupu řádek (`bufferedReader.readLine()`) a ten následně převedem z řetězce na typ `int`. Víme, že první hodnotou v souboru je počet záznamů, který jsme nyní zjistili.

Následně v cyklu načítáme postupně jednotlivé osoby a načtené instance přiřazujeme do cílové proměnné `ret`.

Po ukončení načítání všech osob výslednou proměnnou vrátíme jako výsledek funkce.

Dalším krokem je vytvoření funkce `load()` ve třídě `Person`. I zde budeme postupovat analogicky jako v předchozím případě.

```java
public static Person load (BufferedReader bufferedReader) 
    throws IOException {

    String tmp;
    int age;
    int phonesCount;
    
    Person ret = new Person();
    tmp = bufferedReader.readLine();
    ret.setName(tmp);
    
    tmp = bufferedReader.readLine();
    age = Integer.parseInt(tmp);
    ret.setAge(age);
    
    tmp = bufferedReader.readLine();
    phonesCount = Integer.parseInt(tmp);
    for (int i = 0; i < phonesCount; i++) {
        tmp = bufferedReader.readLine();
        ret.getPhones().add(tmp);
    }
    return ret;
}
```

Nejdříve připravíme pomocné proměnné pro řetězec, věk a počet telefonních čísel. Také vytvoříme budoucí výsledný vrácený objekt `ret`.

První načtený řádek z dat nastavíme jako jméno navraceného objektu. Druhý načtený řádek z dat převedeme z řetězce na číslo a nastavíme jako věk vraceného objektu. Třetí načtený řádek z dat převedeme z řetězce na číslo a uložíme si jej jako počet telefonních čísel, které daná osoba bude mít. Následně v cyklu načteme ze souboru ještě stejný počet řádků - telefonních čísel, které uložíme do kolekce telefonních čísel. Výsledný objekt vrátíme.

Celá problematika není příliš složitá (opakují se neustále stejné vzorce chování), ale je časově náročná na vytvoření.

### Automatická serializace

Řešením je využití automatické `serializace`, kterou Java podporuje. V Javě existují třídy `ObjectOutputStream` a `ObjectInputStream` podporující právě takovýto automatizovaný přístup. Tyto třídy obsahují metody pro automatické uložení primitivních datových typů a také instance obecné třídy. Uložení obecné třídy probíhá (velmi zjednodušeně) tak, že objekt zjistí, jaké třídní proměnné jsou ve třídě definovány, jakých jsou datových typů a následně je všechny uloží. Načtení probíhá obdobně.

Z pohledu programátora je obrovskou výhodou, že tento způsob se velmi snadno programuje. Je třeba zajistit dvě věci:

* Všechny třídy, které se mají ukládat, musí implementovat rozhraní `Serializable`. Toto rozhraní neobsahuje žádné členy ani metody a slouží pouze jako příznak, že danou třídu lze serializovat. Primitivní typy lze serializovat automaticky. Většina typů definovaných v rámci knihoven Javy toto rozhraní implementuje také.
* Realizovat programátorsky uložení a načtení dat.

Prvním krokem je zajištění implementace rozhraní. V představeném příkladě to znamená, že je třeba doplnit implementaci tohoto rozhraní ke třídě `Person`. Třída `PersonCollection` dědí ze třídy `ArrayList`, která již toto rozhraní (nepřímo) implementuje.

```java
class Person implements Serializable {
  // původní kód třídy
}
```

Nyní již stačí do třídy PersonCollection dopsat metodu na uložení dat serialize() s využitím třídy ObjectOutputStream.

```java
public void serialize (String fileName) throws IOException {
    try(FileOutputStream fileStream = new FileOutputStream(fileName)){
        ObjectOutputStream oos = new ObjectOutputStream(fileStream);
        oos.writeObject(this);
    } catch (IOException ex){
        throw ex;
    }
}
```

Lze si povšimnout, že kód je velmi jednoduchý. Začátek spočívá opět v otevření výstupního proudu. Protože serializace pracuje s **byty**, potřebujeme nyní pracovat s `…OutputWriter` proudy a nikoliv s `…Writer` proudy. Třídu `FileWriter` jsme tedy nahradili třídou `FileOutputStream`.

Dalším krokem je použití filtrovacího proudu `ObjectOutputStream` jako filtrovacího nad proudem do souboru. Samotné uložení realizuje jediný příkaz - `writeObject(this)`. Ten zajistí automatické uložení dat aktuálního objektu (`this`) do souboru.

Načtení je obdobně jednoduché.

```java
public static PersonCollection deserialize (String fileName)
    throws IOException {

    PersonCollection ret = null;
    try (FileInputStream fileStream = new FileInputStream (fileName)) {
        ObjectInputStream ois = new ObjectInputStream(fileStream);
        ret = (PersonCollection) ois.readObject();
    } catch (IOException ex) {
        throw ex;
    } catch (ClassNotFoundException ex){
        throw new IOException ("Failed to load required class.", ex);
    }
    return ret;
}
```

Opět je třeba nejdříve otevřít vstupní bytový souborový proud (`FileInputStream`) a nad ním vytvořit `ObjectInputStream`.

Dále již stačí pouze zavolat metodu `readObject()`, která načte objekt ze souboru - protože tato metoda vrací jako návratový typ `Object`, musíme ještě výsledek explicitně přetypovat na datový typ, který chceme vracet jako výsledek z funkce (`PersonCollection`).

Drobnou starostí navíc je ošetření výjimky `ClassNotFoundException`, která nástává v případě, kdy se deserializace snaží načíst ze souboru datový typ, který virtuální stroj Javy nezná.

Následuje ukázka použití.

```java
try {
    persons.serialize("D:\\lide.dat");
} catch (IOException ex) {
    System.out.println(ex.getMessage());
}

try {
    persons = PersonCollection.deserialize("D:\\lide.dat");
} catch (IOException ex) {
    System.out.println(ex.getMessage());
}
```

Výsledný serializovaný soubor (velmi zhruba) ukazuje následující výpis.

```
¬í sr eng.demos.ionio.PersonCollection'"«úÔ'A xr java.util.ArrayListxŇ™Çať I sizexp w
sr eng.demos.ionio.Personź_âŽ.x]ú I ageL namet Ljava/lang/String;L phonest Ljava/util/List;xp ­t Michalsq ~ w
t 605503032t 774839384xsq ~ t Janasq ~ w
t 747873821t 777382948t 698392832xx
```

Lze si povšimnout, že se jedná o bytový, datový soubor, který již nelze ručně editovat a případná změna v něm s velkou pravděpodobností způsobí, že soubor již nepůjde načíst.
