# Práce se souborovým systémem

Práce se soubory a souborovým systémem je jedna z nejdůležitějších dovedností v praktickém programování. Potřeba ukládat někam data do příštího spuštění programu je u všech složitějších aplikací nezbytná.

V javě se pro práci se soubory využívají balíčky _java.io_ (input-output) a _java.nio_ (new input-output). Už podle názvu je balíček _java.nio_ novější, proto se zaměříme převážně na něj. Přesto, přinejmenším základy práce s _java.io_ budou představeny při demonstraci přístupu k souboru. Této problematice bude však věnována až následující kapitola. V této kapitole budou demonstrovány operace se složkami a soubory, tedy jejich procházení, prohledávání, kopírování, přesouvání a mazání.

Je důležité poznamenat, že práce se souborovým systémem je již z principu nespolehlivá záležitost z důvodů možností vzniků různých chyb počínaje špatně zadanou cestou a konče náhle nedostupným zařízením. Proto je třeba u většiny metod hlídat a ošetřovat možnost vzniku výjimek. Třídy z balíčku _java.nio.file_ a jejich metody typicky přímo výjimky nenutí ošetřovat (vyvolávají běhové výjimky, viz kapitola 9), je však vhodné počítat s možností jejich výskytu a program tomu přizpůsobit.

## Cesta k souboru - třída Path

Při práci se složkami a soubory musíme vždy znát cestu ke zdroji (tj. souboru nebo složce), se kterým chceme pracovat. Různé operační systémy identifikují zdroje různým způsobem, vždy však vycházejí ze stromové reprezentace a vždy však musí existovat **jednoznačný popis daného zdroje** v daném operačním systému.

Jednotlivé uzly stromu jsou od sebe odděleny pomocí tzv. oddělovače (angl. _delimiter_), který může být opět na různých operačních systémech odlišný (typicky „\\" na systémech Windows, „/" na Solaris/Unix).

Cesta ke zdroji může být dána několika způsoby:

* Absolutně - tedy plná cesta, která obsahuje kořen a všechny postupně vnořené složky až k danému zdroji. Cesta je tedy samopopisná a plně identifikuje daný zdroj. Například „C:\Windows\System32\nt.dll".
* Relativně - cesta obsahující pouze koncovou část cesty. Relativní cesta **nepopisuje jednoznačně zdroj** a k jednoznačnému popisu potřebuje doplnit jinou cestu jako svůj začátek. Příkladem relativní cesty může být „System32\nt.dll". Tato cesta nepopisuje zdroj jednoznačně, po doplnění různým začátkem může reprezentovat různé soubory.
* Pomocí symbolických odkazů - podporují je pouze některé operační systémy a lze si je jednoduše představit jako odkazy na jiné zdroje. Studijní opora se jimi nebude zabývat, případné zájemce odkážeme na podrobnější literaturu.

Výše zmíněná představená cesta je v jazyce Java reprezentovatelná pomocí třídy _java.nio.**file**.Path_. Instance této třídy umí zachytit absolutní nebo relativní cestu ke zdroji. Tato cesta **nemusí fyzicky existovat**, instance třídy tedy může reprezentovat i cestu, která bude například teprve v budoucnu vytvořena. Třída sama řeší problematiku multiplatformnosti a umí se přizpůsobit podle aktuálního operačního systému.

Pro práci s třídou _Path_ slouží třída obsahující odpovídající **statické** metody nazvaná _Paths_. Její metoda _get()_ umí vytvořit novou instanci třídy _Path_.

```java
java.nio.file.Path first = java.nio.file.Paths.get("D:\\temp");
Path second = Paths.get("myApp\\tempFile.tmp");
```

{% hint style="info" %}
Při zadávání cesty na OS z rodiny Windows je třeba zadávat zpětná lomítka dvojitě - jedná se o klasické escape-sekvence u řetězců.
{% endhint %}

O instanci třídy _Path_ lze získávat další informace. Samozřejmostí je překrytí metody _toString()_. Následuje ukázka použití a tabulka s odpovídajícím výstupem a popisem volaných metod.

```java
java.nio.file.Path path = java.nio.file.Paths.get(
  "D:\\temp\\myApp\\tempFile.tmp");
System.out.println(path.toString());
System.out.println(path.getFileName());
System.out.println(path.getName(0));
System.out.println(path.getNameCount());
System.out.println(path.subpath(0, 2));
System.out.println(path.getParent());
System.out.println(path.getRoot());
```

<table><thead><tr><th width="167">Metoda</th><th width="173">Výstup</th><th>Popis</th></tr></thead><tbody><tr><td><code>toString()</code></td><td>D:\temp\myApp\tempFile.tmp</td><td>Vrací plnou cestu.</td></tr><tr><td><code>getFileName()</code></td><td>tempFile.tmp</td><td>Vrací jméno posledního uzlu (<strong>nemusí být nutně soubor</strong>) v dané cestě.</td></tr><tr><td><code>getName(0)</code></td><td>temp</td><td>Vrací název uzlu odpovídající danému indexu.</td></tr><tr><td><code>getNameCount()</code></td><td>3</td><td>Vrací počet uzlů v cestě.</td></tr><tr><td><code>subpath(0,2)</code></td><td>temp\myApp</td><td>Vrací „podcestu" jako část původní cesty. První parametr je počáteční index „od", druhý počet uzlů.</td></tr><tr><td><code>getParent()</code></td><td>D:\temp\myApp</td><td>Vrací cestu bez posledního uzlu.</td></tr><tr><td><code>getRoot()</code></td><td>D:\</td><td>Vrací kořenový adresář dané cesty.</td></tr></tbody></table>

Pro převod instance třídy _Path_ na řetězec využijeme přímo metody _toString()_.

#### Operace s instancí třídy Path

Klasickým případem reprezentace cesty je využití zástupných symbolů pro aktuální („.") nebo nadřazený adresář („.."). Třída _Path_ umí tyto zástupné znaky sloučit a odstranit do nové instance třídy _Path_ pomocí metody _normalize()_. Opět - nezkoumá se, zda výsledná cesta existuje nebo dává v daném operačním systému smysl.

```java
Path path = Paths.get(
"D:\\temp\\.\\myApp\\..\\..\\otherTemp\\.\\file.txt");
Path normalizedPath = path.normalize();
System.out.println(path);
System.out.println(normalizedPath);
```

Následuje ukázka výstupu:

```
D:\temp\myApp\..\..\otherTemp\file.txt
D:\otherTemp\file.txt
```

Další klasickou operací je **skládání**. Dvě cesty lze spojit k sobě s využitím metody _resolve()_. Pozor - druhá (případně další) cesta musí být relativní, jinak se použije poslední zadaná absolutní cesta.&#x20;

{% hint style="info" %}
## **Absolutní vs. relativní cesta**

Absolutní cesta určuje přesné umístění souboru nebo složky od kořenového adresáře systému. Obsahuje celou cestu, takže nezáleží na tom, odkud je aplikace spuštěna. Příklad: `C:\Projects\App\data\file.txt` nebo `/home/user/app/data/file.txt`.

Relativní cesta určuje umístění souboru vzhledem k aktuálnímu pracovnímu adresáři nebo umístění projektu. Je kratší a přenosnější, ale závisí na tom, odkud se program spouští. Příklad: `data/file.txt`.

V praxi se v aplikacích častěji používají relativní cesty nebo konfigurovatelné cesty, protože umožňují snadnější přesun projektu mezi různými počítači a prostředími.
{% endhint %}

Opět následuje příklad a výstup:

```java
Path first = Paths.get("D:\\temp");
Path second = Paths.get("myApp");
Path third = Paths.get("tempFile.tmp");
Path result = first.resolve(second).resolve(third);
System.out.println(result);
```

```
D:\temp\myApp\tempFile.tmp
```

Opačnou operací je snaha získat **rozdíl** dvou cest. Máme dvě cesty, a chceme získat relativní cestu, která nás přesune z první cesty k cestě druhé. K tomuto účelu slouží metoda _relativize()_.

```java
Path first = Paths.get("D:\\temp");
Path second = Paths.get("D:\\temp\\myApp\\tempFile.tmp");
Path resultA = first.relativize(second);
Path resultB = second.relativize(first);
System.out.println(resultA);
System.out.println(resultB);
```

```
myApp\tempFile.tmp
..\..
```

Je důležité si uvědomit, že zde záleží na pořadí parametrů, jak ukazuje výše uvedený výstup.

Poslední běžnou operací je **porovnávání** **cest**. Lze porovnávat:

* Shodnost - tedy zda dvě cesty jsou si zcela shodné, pomocí metody _equals()_;
* Shodnost začátku - tedy zda první cesta začíná stejnými uzly, jaké obsahuje celá druhá cesta, pomocí metody _startsWith()_;
* Shodnost konce - tedy zda první cesta končí stejnými uzly, jaké obsahuje celá druhá cesta, pomocí metody _endsWith()_.

Použití je obdobné jako u třídy _String_ a jejich metod a proto nebude demonstrováno.

## Práce se složkami a soubory

Pro reprezentaci složky nebo souboru byla představena třída _Path_. Opět je třeba zdůraznit, že tato třída **nikdy nekontroluje, zda cesta**, kterou reprezentuje, **existuje fyzicky**. Navíc, instance třídy _Path_ díky tomu nikdy přímo neví, zda reprezentuje složku, či soubor.

Pro zjištění, zda daná cesta existuje a zda je složka či soubor, využijeme jednu z následujících metod třídy _java.nio.file.Files_:

* _isDirectory(path)_ - vrací _true_, pokud je zadaný parametr složka;
* _isRegularFile(path)_ - vrací _true_, pokud zadaný parametr je soubor;
* _isReadable(path)_ - vrací _true_, pokud je daný zdroj přístupný pro čtení (je fyzicky k dispozici a má odpovídající přístupová práva pro aktuálního uživatele);
* _isWriteable(path)_ - vrací _true_, pokud je daný zdroj přístupný pro zápis (stejné jako výše);
* _isExecutable(path)_ - vrací _true_, pokud je daný zdroj přístupný ke spuštění (stejné jako výše).

{% hint style="info" %}
Je třeba si uvědomit, že volání funkcí `isReadable()`/`isWriteable()` automaticky nezajistí, že  zdroj bude pro následné čtení/zápis dostupný, protože jeho přístupnost se může mezi kontrolou a samotným čtením/zápisem změnit. Více viz zkratka TOCTTOU.
{% endhint %}

Následuje krátký příklad.

```java
Path first = Paths.get("D:\\temp");
Path second = Paths.get("D:\\temp\\tempFile.tmp");
boolean isFirstFile =
    Files.isRegularFile(first) && Files.isReadable(first);
boolean isSecondFile =
    Files.isReadable(second) && Files.isReadable(second);
boolean isFirstFolder = Files.isDirectory(first);
System.out.println(isFirstFile);
System.out.println(isSecondFile);
System.out.println(isFirstFolder);
```

```
false
false
true
```

Základní operace při práci se soubory jsou kopírování, přesouvání a mazání.&#x20;

{% hint style="info" %}
Přejmenovávání je speciální případ přesouvání – jedná se vlastně o přesunutí souboru z jednoho názvu do názvu druhého. Protože se však fyzicky jedná pouze o přepis názvu v souborovém systému, operace je velmi rychlá. Žádná metoda přímo na přejmenování neexistuje.
{% endhint %}

Následující statické metody (a další) podporující tyto základní operace patří opět třídě _System.nio.file.Files_. Následuje jejich rychlý přehled:

* _remove(path)_ - smaže daný cíl (pokud neexistuje zadaná cesta, vyvolá chybu);
* _removeIfExists(path)_ - smaže daný cíl, pokud existuje;

{% hint style="info" %}
## Varargs - proměnný počet argumentů

Před představením dalších metod je třeba na chvilku odbočit a věnovat se problematice tzv. **proměnného počtu argumentů**. Tato problematika nebude představena blíže, princip je však třeba představit, aby byl jasný význam volání některých funkcí. Tento princip v jazce Java umožňuje, aby funkce měla libovolný počet argumentů **stejného typu**. Programátor potom zapisuje další argumenty jako u běžného volání funkce, oddělené čárkou. Rozdíl oproti přetěžování je, že v době kompilace není znám počet argumentů a u přetěžování by tvůrce musel vytvořit obecně „n" metod pokrývající všechna možná přetížení. Programátor-uživatel takové funkce potom může (ale také nemusí) uvést libovolný počet takových argumentů. Hodnoty argumentů typicky bývají dané konstanty.

Zájemci o bližší studium nechť si vyhledají klíčové slovo `varargs` v kontextu jazyka Java.
{% endhint %}

Kopírování lze provést jednoduchou funkcí _copy_ se signaturou:

* copy (source, target), nebo
* _copy (source, target, java.nio.file.StandardCopyOption.REPLACE\_EXISTING)_

, kde konstanta _REPLACE\_EXISTING_ reprezentuje právě volitelný argument funkce, který může, ale nemusí být uveden. Tato konstanta je definována v datovém typu _java.nio.file.StandardCopyOption_. Pokud uvedena není a při kopírování by bylo třeba nějaká cílová data přepsat, operace se nepovede a skončí výjimkou.

Obdobně, přesouvání lze provést pomocí funkce _move_ se stejnou signaturou jako u funkce _copy()_.

```java
Path source = Paths.get("D:\\temp\\tempFile.tmp");
Path target = Paths.get("D:\\temp\\otherTempFile.tmp");
try {
  Files.copy(source, target);
  Files.move(source, target,
  java.nio.file.StandardCopyOption.REPLACE_EXISTING);
  Files.delete(target);
} catch (IOException ex) {
  // log error...
}
```

U všech těchto operací se již musíme postarat o korektní zachycení výjimek.

Následuje pouze rychlý přehled dalších, potenciálně užitečných metod třídy _Files_:

* _size(path) -_ vrací velikost v bytech;
* _isHidden(path) -_ zda je daná položka skrytá v operačním systému;
* _getLastModifiedTime(path) -_ datum a čas poslední změny souboru (existuje odpovídající _set_… metoda);
* _getOwner(path) -_ vrací majitele položky (existuje odpovídající _set…_ metoda);
* metody pro práci s atributy položky

## Procházení obsahu složky

Jednou z typických úloh je procházení obsahu složky. Představen bude velmi stručně původní řešení pomocí třídy _java.io.File_ a nové řešení, pomocí balíčku _java.nio.files_.

### Třída java.io.File

Třída _java.io.File_ je původní třídou řešící problematiku práce se soubory. Základem je vytvoření instance od zkoumaného objektu pomocí konstruktoru, kterému se předá cesta. Stejně jako u třídy _java.nio.files.Path_, cesta nemusí fyzicky existovat.

Vytvořená instance obsahuje několik zajímavých metod umožňujících zjišťovat informace o objektu na dané cestě, jako jsou atributy, zda se jedná o soubor/složku, velikost a další. Pro procházení je důležitá metoda _listFiles()_, která umí projít celý obsah dané složky a vrátit jako kolekci instancí třídy _java.io.File_ všechny vnořené soubory a složky.

Třídě nebudeme věnovat přílišnou pozornost a ihned zmíníme krátký příklad, který ukazuje vypsání všech souborů v zadané složce.

```java
String folderName = "C:\\Windows";
java.io.File folder = new java.io.File(folderName);
for(File file : folder.listFiles()){
    if (file.isFile())
        System.out.println(file);
}
```

Nejdříve se vytvoří instance třídy _file_ pro zadanou cestu. Následně se v cyklu for-each prochází přes všechny instance _File_ získané voláním metody _listFiles()_. Podmínka zkontroluje, zda je daný objekt soubor (metoda vrací složky i soubory dohromady, v nedeterministickém pořadí) a název souboru vypíše na konzoli uživateli.

### Třída java.nio.file.SimpleFileVisitor

Nové řešení v balíčku _java.nio.file_ přináší trošku složitější, ale univerzálnější řešení. Základem jednoduchého řešení je třída _java.nio.file.SimpleFileVisitor_, které umožňuje při procházení všech souborů a složek pro každou složku a každý soubor provést určitou operaci. Třída umožňuje překrýt potomkovi čtyři základní operace.

<table><thead><tr><th width="202">Název metody</th><th>Popis</th></tr></thead><tbody><tr><td><code>preVisitDirectory()</code></td><td>Volána předtím, než se budou procházet postupně všechny položky dané složky.</td></tr><tr><td><code>postVisitDirectory()</code></td><td>Volána poté, co byly projity všechny položky dané složky.</td></tr><tr><td><code>visitFile()</code></td><td>Volána při zpracovávání konkrétního souboru složky.</td></tr><tr><td><code>visitFileFailed()</code></td><td>Volána v případě vzniku nějaké chyby. Pokud tato metoda není překryta, chyba způsobí vyvolání obecné výjimky IOError.</td></tr></tbody></table>

Programátor vytvoří vlastního potomka od třídy _SimpleFileVisitor_ a dané metody překryje tak, aby implementoval požadovanou funkcionalitu. Třída _SimpleFileVisitor_ je generická, jako generický parametr se však při běžném použití používá právě třída _Path_.

Demonstrační příklad ukáže řešení, kdy chceme vypsat všechny soubor dané složky (a jejich podsložek), jejichž velikost je menší než 10000 bytů.

Základním krokem je vytvoření potomka třídy _SimpleFileVisitor\<Path>_, který bude překrývat metodu _visitFile()_. V této metodě se budou hledat odpovídající soubory menší než daná velikost a v případě splnění podmínky se soubor vypíše na konzoli.

```java
class MyFileVisitor extends java.nio.file.SimpleFileVisitor<Path>{
    private static final long MAX_RETURN_SIZE = 10000;

    @Override
    public FileVisitResult visitFile (
            Path path,
            java.nio.file.attribute.BasicFileAttributes attributes)
            throws IOException{
        if (java.nio.file.Files.size(path) < MAX_RETURN_SIZE)
            System.out.println(path);
        return FileVisitResult.CONTINUE;
    }
}
```

Samotný kód je poměrně jednoduchý. Ve třídě se definuje konstanta udávající maximální velikost vypisovaného souboru. Dalším blokem je překrytí metody _visitFile()_. Uvnitř této metody, která bude automaticky volána pro každý soubor, se bude kontrolovat velikost souboru, a pokud je podmínka splněna, název souboru (resp. jeho cesta) se vypíše na konzoli.

Kód filtrující soubory a provádějící s nimi operace lze samozřejmě libovolně přizpůsobit.

Vytvořenou třídu aplikujeme pomocí statické metody _walkFileTree()_ třídy _java.nio.file.Files_. Té předáme jako parametry výchozí cestu a právě **instanci** třídy, která je potomkem _SimpleFileVisitor_.

```java
String folderName = "C:\\Windows";
java.nio.file.Path folderPath = java.nio.file.Paths.get(folderName);
try {
    java.nio.file.Files.walkFileTree(folderPath, new MyFileVisitor());
} catch (IOException ex) {
    System.out.println("Operation failed. " + ex.getMessage());
}
```
