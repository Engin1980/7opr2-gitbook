# TODO Výjimky

Výjimky jsou základním mechanismem v Jazyce java pro zpracování chybových stavů. Protože při programování běžných aplikací se jejich použití nikdy nelze vyhnout, je třeba důkladně a dokonale porozumět principu jak vznikají a jak se zpracovávají.

Dle definice je výjimka „_událost, která nastane při zpracování programu a která přeruší běžný tok instrukcí programu_".

Každý program běží jako vykonávaná metoda (typicky _main()_), která následně volá další a další metody ostatních objektů. Tyto metody po provedení svého kódu předají vykonávání zpět nadřazené metodě. Seznamu aktuálně do sebe zanořených metod se říká _zásobník volání_ (call stack).

Pokud v určité metodě dojde k chybě, která způsobí vyvolání výjimky, běhové prostředí v metodě vytvoří objekt popisující tuto chybu a místo vzniku. Tento objekt se poté předává zpět nahoru volanými metodami až do nalezení prvního bloku (tzv. _handleru_), který umí danou výjimku ošetřit.

Program.main()

Manager.orderPizza()

pizzeria.preparePizza()

chiefCook.cookPizza()

Volání metod

Hledání ošetření výjimky

Místo vzniku chyby

Pokud žádná metoda ošetření neprovede, program spadne za běhu a uživateli se objeví kritická chybová hláška o činnosti programu.

Protože Java je objektově orientovaný programovací jazyk, výjimka jako objekt je reprezentována jako instance třídy _Throwable_, nebo některého z jejich potomků.

Obecně se v Javě vyskytují tři typy výjimek:

* Kontrolované výjimky (checker exceptions) - jedná se o chybové stavy, které by měla dobře napsaná aplikace být schopna ošetřit a korektně se z nich zotavit. Příkladem může být pokus o smazání neexistujícího souboru, reakce na chybové vstupy uživatele, ošetření náhle nedostupných zařízení nebo výpadků v síťovém připojení. Program na tyto chyby umí zareagovat a korektně uživatele informovat o problému. Při správném zachycení program pokračuje korektně (s případnými omezeními) dále. Kontrolované výjimky jsou všechny výjimky vyjma těch uvedených v dalších dvou bodech.
* Chyby (errors) - jedná se o fatální stavy typicky způsobené prostředím vně aplikace. Příkladem může být odepření přístupu k paměti, chyba ve virtuálním stroji, neplatná instrukce, selhání hardware a podobně. Z těchto chyb se aplikace typicky nedokáže smysluplně zotavit a po pádu kvůli této výjimce její běh končí. Tyto výjimky se typicky neošetřují, protože programátor je nemůže předpokládat, ani smysluplně ošetřovat. Chyby reprezentují všechny třídy dědící z předka _Error_.
* Běhové výjimky (runtime exceptions) - jedná se o chyby v rámci aplikace, které typicky vznikají kvůli programátorským chybám. Patří zde aritmetické chyby (dělení nulou), špatné použití indexů (mimo rozsah pole), použití _null_ objektu k vyvolání metod a další. Aplikace se typicky z této chyby nedokáže zotavit. Pokud je však taková chyba zachycena, programátor může opravit logiku aplikace, aby její výskyt nemohl příště nastat (opravou kódu, omezením vstupu od uživatele apod.). Běhové výjimky reprezentují potomci třídy _RuntimeException_.

Výjimky lze tedy rozdělit do skupin dle následujícího obrázku.

Souvislost se skupinami vychází i dělení výjimek podle hierarchie dědičnosti jednotlivých tříd, ze kterých dané výjimky vycházejí. Pozor, že tato **hierarchie tříd nekoresponduje s hierarchií typů výjimek**.

Každá výjimka je tedy reprezentována **instancí třídy**, která je přímým nebo nepřímým potomkem třídy _Throwable_. Pokud se jedná o potomka třídy _Error_, jedná se o chybu. Pokud se jedná o potomka třídy _RuntimeException_, jedná se o běhovou výjimku. V ostatních případech se jedná o kontrolovanou výjimku.

## Zachytávání výjimek

Zachytávat lze všechny typy výjimek, povinně se však zachytávají pouze výjimky kontrolované. Pro zachytávání výjimek se používá v Javě zvláštní pravidlo, které se nazývá _Catch Or Specify Requirement_. Toto pravidlo říká, že **pro každou kontrolovanou výjimku musí platit jedna z následujících variant:**

* **Catch -** místo vyvolání výjimky je uzavřeno v bloku _try_ {…}. K tomuto bloku musí být bezprostředně připojen blok _catch {…}_, který umí ošetřit a zotavit program při vzniku této výjimky.
* **Specify -** metoda, který vyvolává tuto výjimku, musí explicitně ve své deklaraci (úvodním řádku) definovat, že tuto výjimku vyvolává, pomocí klíčového slova _throws_[_\[34\]_](https://word2md.com/#footnote-34) následovaným seznamem vyvolávaných výjimek oddělených čárkou.

### Catch requirement

Výjimka se zachytává do sekvence bloků _try{…}_ _catch{…} finally{…}_, kde platí, že:

* Blok _try_ musí být právě jeden;
* Bloků _catch_ může být více, ale každý musí platit pro odlišný typ výjimky (bude vysvětleno dále);
* Blok _finály_ je nepovinný.

Blok _try_ kontroluje kód na vznik výjimky. Pokud je nějaký výjimka vyvolána, řízení se předá do bloku _catch_ odpovídajícího typu výjimky. Pokud existuje i blok _finally_, vykoná se vždy buď po úspěšném ukončení bloku _try_ nebo po ukončení odpovídajícího bloku _catch_, a to i v případech, že blok _catch_ obsahuje odskok pryč z funkce (například klíčovým slovem _return_).

```java
try{
    // prikazy, ktere mohou vyvolat vyjimku
}
catch (Exception ex){
    // zachyceni vyjimky a zpracovani
    // informace o chybe jsou v promenne "ex"
}
finally {
    // nepovinny blok, ktery se provede vzdy po bloku try nebo catch
}
```

Blok _try_ může pokrývat několik příkazů (volání funkcí), není tedy třeba pro každou funkci, o které víme, že bude vyvolávat kontrolovanou výjimku, vytvářet vlastní sekvenci try-catch(-finally).

Blok _catch_ je oproti ostatním blokům konstrukčně složitější a specifičtější. Za klíčové slovo _catch_ se do závorek (obdobně jako parametr funkce) musí zapsat datový typ (třída) zachytávané výjimky a proměnná, do níž se výjimka uloží. Typicky se uvádí zápis:

```java
catch (Exception ex)
```

, kde _catch_ je klíčové slovo, _Exception_ je typ zachytávané výjimky, a _ex_ je název proměnné, do které bude uložen objekt reprezentující danou výjimku. Typ výjimky musí být vždy buď třída _Throwable_ nebo nějaký její přímý či nepřímý potomek. Jeden blok může zachytávat více typů vyjímek[\[35\]](https://word2md.com/#footnote-35) (tehdy se typy oddělují pomocí znaku „svislítka" - |, nebo bloků _catch_ může být uvedeno více (zapisují se pod sebou) a každý bude mít definován odlišný typ výjimky, který může zachytit[\[36\]](https://word2md.com/#footnote-36).

```java
try {
    int a = 0;
    int b = 0;
    int c = a / b; // dělení nulou
} catch (IllegalArgumentException ex) {
    System.out.println("-IllegalArgumentException-");
} catch (RuntimeException ex) {
    System.out.println("-RuntimeException-");
} catch (Exception ex) {
    System.out.println("-CheckedException-");
} catch (Throwable ex) {
    System.out.println("-Throwable-");
}
```

Zde:

* První blok _catch_ zachycuje pouze výjimku třídy _IllegalArgumentException_ - což je výjimka, kterou vyvolávají některé funkce v případech, kdy se jim předá nesprávný parametr.
* Druhý blok _catch_ zachycuje všechny běhové výjimky (v našem případě zde dojde k zachycení dělení nulou, které se provádí v bloku _try_).
* Třetí blok zachycuje obecně všechny kontrolované výjimky.
* Čtvrtý blok zachycuje obecně všechny ostatní výjimky, jaké mohou v programu nastat.

Lze si povšimnout, že i když je doporučeno zachytávat pouze kontrolované výjimky (jiné by v programu nastat neměly, nebo se z nich nelze zotavit), tak zachytit lze úplně všechny výjimky (s využitím třídy _Throwable_).

### Specify requirement

Sekvence try-catch(-finally) předpokládá, že programátor v bloku _catch_ umí odstranit problémy způsobené se vznikem výjimky a uvést program do stavu, ve kterém bude schopen fungovat korektně dále.

Například - uživatel zadá soubor ke kopírování, cílový soubor však existuje. Vyhodí se výjimka, která říká, že cílový soubor nelze přepsat. Programátor zobrazí uživateli dialog, že soubor nelze přepsat a program sám pokračuje korektně dále, byť se daná operace neprovede. (Mimochodem, kdyby tento případ ošetřen nebyl, celá aplikace by spadla na kritickou chybu a ukončila by se, což by běžného uživatele jistě nepotěšilo.)

Ne vždy je však programátor schopen (na daném místě) provést korektní zotavení z chyby a uvést program do konzistentního stavu. Například při práci s databází může selhat operace vložení hodnoty do tabulky, protože uživatel zadal číslo, které neodpovídá požadovanému rozsahu. Ve funkci, která vložení provádí, se však programátor nemůže zeptat uživatele, co má s danou chybou dělat (zda například nechce zadat jinou hodnotu), protože taková operace může typicky běžet na úplně jiném počítači, než na kterém požadavek na uložení dat vznikl. Programátor tedy v tuto chvíli nemá k dispozici prostředek, jak korektně provést uložení dat a výjimku smysluplně ošetřit. (Opět, je důležité si uvědomit chování programu. Programátor nemůže uložit do sloupce například nejbližší hodnotu - opravdu uživatel, který omylem místo 2011 zadá 1911, chce, aby program sám automaticky uložil například hodnotu 2000 (protože je nejblíže povolené hodnotě) a uživateli oznámil, že uložení proběhlo úspěšně?)

**Pokud programátor nemá způsob, jak smysluplně ošetřit danou výjimku v metodě, ve které se nachází, může ji předat výše** (nadřazené metodě v zásobníku volání), a ta může zkusit zpracování provést (nebo výjimku předat ještě výše).

```java
/**
 * Vrací velikost souboru v bytech.
 * @param fileName Název souboru
 * @return Velikost souboru v bytech.
 * @throws java.io.IOException Vyvolá chybu, pokud soubor neexistuje.
 */
public static long getFileSize (String fileName)
        throws java.io.IOException{
    // vytvoreni pomocneho objektu pro praci se souborem
    java.nio.file.Path filePath = java.nio.file.Paths.get(fileName);
    // zjisteni velikosti souboru
    // tato operace vyvola chybu, pokud soubor neexistuje !
    long ret = java.nio.file.Files.size(filePath);
    return ret;
}
```

Výše uvedený příklad ukazuje funkci, která vrací velikost souboru (tato problematika bude vysvětlena dále ve studijní opoře - zaměřeno bude tedy pouze na problematiku zpracování chyby). Pro zjištění velikosti využije funkci _java.nio.file.Files.size_ - tato funkce však v případě, kdy soubor neexistuje, spadne na výjimce _java.io.IOException_. Jak víme, můžeme buď výjimku zachytit pomocí bloku try-catch(-finally), ale jak? Programátor na této úrovni nemá způsob, jak smysluplně výjimku zachytit. Proto ji zpracovávat nebude, ale do záhlaví funkce připíše _throws java.io.IOException_ (viz tučný text). Tato výjimka se bude tedy z této metody posílat výše do nadřazené/ých metod, dokud nedojde k jejímu zpracování, nebo k pádu programu (pokud nebude ošetřena nikde).

### Běhové vs. kontrolované výjimky

V úvodu kapitoly bylo zmíněno dělení na běhové a kontrolované výjimky. Jejich odlišnost je jednak z pohledu hierarchie tříd: běhové výjimky jsou potomky třídy _RuntimeException_, zatímco kontrolované výjimky „jen" třídy _Exception_.

Druhou odlišností je potřeba a způsob zachytávání:

* Pokud metoda vyvolává kontrolovanou výjimku, musí za deklarací obsahovat příkaz _throws_ \[ty&#x70;_&#x6B;ontrolované\_výjimky]. Pokud metoda vyvolává běhovou výjimku, není třeba ji deklarovat příkazem \_throws_. (Nicméně je to volitelné, a pokud chceme, můžeme blok _throws_ dopsat i pro běhovou výjimku.)
* Programátor může libovolně vyhazovat výjimky jak běhové, tak kontrolované, a to vždy příkazem _throw \[instance\_výjimky]_, tj. např. _throw new Exception()_.
* Programátor **musí** zachytávat všechny vyvolané, nebo propagované **kontrolované** výjimky. Vyvolaná kontrolovaný výjimka je taková, která se v kódu vyvolá pomocí příkazu _throw \[instance\_výjimky]_. Propragovanými kontrolovanými výjimkami myslíme volání metod, která za svou deklarací uvádějí …_throws \[typ\_výjimky]_[_\[37\]_](https://word2md.com/#footnote-37). Na druhou stranu programátor pouze **může** zachytávat běhové výjimky. Pokud je zachytávat bude, je řešení shodné jako u výjimek kontrolovaných. Pokud programátor nebude zachytávat běhovou výjimku a ta při běhu programu vznikne a nebude zachycena žádným blokem _catch_, způsobí pád programu.

## Zjištění informaci o výjimce

Jak bylo zmíněno na začátku kapitoly, instance třídy _Throwable_, která je zachycena v bloku _catch_, obsahuje informace o místu vzniku chyby, důvodu vzniku chyby a případné další informace.

Dvě základní informace, které je třeba u většiny výjimek získat, abychom byli schopni rozhodnout, jak s danou výjimkou naložit, jsou:

* Přesný datový typ výjimky - zjistit, jaký je přesný datový typ vyhozeného objektu, protože název výjimky může pomoci identifikovat typ chyby a navrhnout tak korektně její opravu.
* Vnitřní informace o výjimce, tzv. zprávu výjimky - každá výjimka v sobě může obsahovat zadaný textový řetězec, který popisuje bližší informace o chybě.

První informaci zjistíme pomocí příkazu (předpokládáme proměnnou _ex_) _ex.getClass().getName()_. Tento složitější příkaz nejdříve zjistí, jaká třída je vlastně reprezentována proměnnou _ex_ (příkaz _getClass()_) a následně vrátí její název (příkaz _getName()_).

Druhá informace je mnohem jednodušší - informaci o zprávě výjimky zjistíme jednoduchým voláním metody _getMessage()_.

Obě informace dohromady vrací jako výsledek metoda _toString()_.

Následující příklad nejdříve demonstračně vyvolá nějakou chybu. Následující řádky v bloku _catch_ ukazují na způsob získání daných informací o výjimce.

```java
try{
    throw new IllegalArgumentException("Hodnota proměnné je null.");
} catch (IllegalArgumentException ex){
    System.out.println("Typ výjimky je: " +
        ex.getClass().getName());
    System.out.println("Zpráva výjimky je: " + ex.getMessage());
    System.out.println("-toString()- výjimky je: " + ex.toString());
}
```

Následuje výstup výše uvedeného příkladu.

```
Typ výjimky je: java.lang.IllegalArgumentException
Zpráva výjimky je: Hodnota proměnné je null.
-toString()- výjimky je: java.lang.IllegalArgumentException: Hodnota proměnné je null.
```

Tyto získané informace lze většinou částečně použít i pro výpisy do grafického prostředí uživatelům - je však třeba dávat pozor u lokalizovaných programů, že chybové hlášky výjimek jsou typicky v anglickém jazyce.

## Vyvolání výjimek

Výjimky nemusí vyvolávat pouze hotové knihovny, ale může je vyvolávat samozřejmě sám programátor. Pro vyvolání potřebuje klíčové slovo _throw_ a **instance** třídy výjimky, kterou chceme vyvolat.

Typicky se vyvolávají výjimky dvěma způsoby:

* Původní výjimka, která se zachytí v bloku _catch_. V bloku _catch_ se provedou nějaké úpravy (například logování chyby) a původní výjimka se vyhodí do nadřazeného bloku;
* Nová výjimka, kterou programátor vytvoří.

První způsob je velmi jednoduchý. V bloku _catch_ zachytíme výjimku, typicky provedeme nějaké pomocné operace (jako uvedení třídy do korektního stavu, logování chyby, odeslání zprávy o chybách apod.) a následně **původní** výjimku beze změny vyhodíme do vyššího bloku, pomocí klíčového slova _throw_ následovaného objektem výjimky (viz tučný text).

```java
try{
    methodToRaiseException();
} catch (Exception ex){
    ...
    throw ex; // <-- zde vyhazujeme původní výjimku
}
```

Tento způsob je velmi rychlý a jednoduchý. Nevýhodou je, že nejsme schopni doplnit bližší informace o chybě do existující instance třídy výjimky.

U druhé, složitější varianty musí programátor sám vytvořit instanci třídy _Exception_ (resp. některého z jejich potomků) a zajistit naplnění tohoto objektu smysluplnými informacemi o původu chyby.

Začínající programátoři nejčastěji používají obecnou třídu _Exception_, je však vhodné u výjimek zkoumat, zda neexistuje určitá specifičtější, která by byla jako nositel informace o chybě vhodnější. Případně je možné dokonce vlastní výjimku (ve smyslu třídy) vytvořit (bude popsáno dále).

Ještě jednou je důležité zdůraznit, že se vyvolává **instance třídy** a nikoliv třída samotná. Proto ve většině případů za klíčovým slovem _throw_ ještě následuje slovo _new_ a teprve potom konstruktor dané třídy.

```
throw new Exception();
```

Naprostá většina výjimek má přetížený konstruktor, který jako jeden z argumentů přijímá řetězec, kam může programátor zadat text blížší popis chyby.

```
throw new Exception("Vyhození testovací výjimky");
```

Pro vyhozenou výjimku platí stejné pravidlo _Catch Or Specify Requirement_, tzn. výjimka musí být vyvolána buď v bloku try-catch(-finally), nebo metoda, v níž je vyvolána, obsahuje klíčové slovo _throws_ s doplněním typu výjimky. Protože typicky programátor nevyvolává výjimku, aby ji hned zachytával, používá se v drtivé většině případů druhý způsob. Následující příklad ukazuje pozměněnou metodu zjišťující velikost souboru, kdy programátor doplnil ošetření, zda parametr s názvem souboru není _null_.

```java
 /**
 * Vrací velikost souboru v bytech.
 * @param fileName Název souboru
 * @return Velikost souboru v bytech.
 * @throws java.io.IOException Vyvolá chybu, pokud soubor neexistuje.
 */
public static long getFileSize2 (String fileName)
        throws java.io.FileNotFoundException, java.io.IOException{
    if (fileName == null)
        throw new java.io.FileNotFoundException("Filename is null.");
    // vytvoreni pomocneho objektu pro praci se souborem
    java.nio.file.Path filePath = java.nio.file.Paths.get(fileName);
    // zjisteni velikosti souboru
    // tato operace vyvola chybu, pokud soubor neexistuje !
    long ret = java.nio.file.Files.size(filePath);
    return ret;
}
```

## Tvorba vlastní výjimky

Vytvoření vlastní výjimky je velmi jednoduché. Protože **programátorské výjimky by měl být vždy** až na závažné případy **kontrolované výjimky**, novou výjimku programátor vytvoří jednoduchým poděděním ze třídy _java.lang.Exception_.

```java
class ArgumentNullException extends Exception
```

V drtivé většině případů nezůstává tato třída prázdná, ale typicky se přetěžují nebo vytvářejí konstruktory reprezentující odpovídající informace.

Představen bude velmi jednoduchý příklad výjimky _ArgumentNullException_, která bude vyvolávána, pokud do nějaké funkce vstoupí argument, jehož hodnota nesmí být null. Budou vytvořeny vlastní konstruktory, které voláním konstruktoru nadřazeného předka[\[38\]](https://word2md.com/#footnote-38) zajistí, že výjimka bude vždy obsahovat smysluplnou zprávu popisující chybu.

```java
class ArgumentNullException extends Exception{
    public ArgumentNullException (){
        super ("Argument has -null- value.");
    }
    public ArgumentNullException (String argumentName){
        super ("Argument " + argumentName + " has -null- value.");
    }
}
```

Následuje velmi jednoduchá ukázka použití uvnitř funkce. Jedná se opět o upravenou funkci pro zjištění velikosti souboru (opakující se části byly nahrazeny výpustkou - …)

```java
public static long getFileSize3(String fileName)
        throws ArgumentNullException,
        java.io.FileNotFoundException,
        java.io.IOException {
    if (fileName == null) {
        throw new ArgumentNullException("fileName");
    }
    …
}
```

Poslední ukázka reprezentuje volání této funkce se špatným argumentem a zachycení všech výjimek a jejich výstup na konzoli uživateli.

```java
long fileSize = 0;

try {
    fileSize = getFileSize3(null);
} catch (ArgumentNullException ex) {
    System.out.println("Název souboru není platný -> "
        + ex.getMessage());
} catch (FileNotFoundException ex) {
    System.out.println("Soubor nebyl nalezen -> "
        + ex.getMessage());
} catch (IOException ex) {
    System.out.println("Chyba při práci se souborem -> "
        + ex.getMessage());
}
```

A nakonec výstup výše uvedeného volání.

```
Název souboru není platný -> Argument fileName has -null- value.
```

## Zřetězení výjimek - chained exceptions

Zřetězené výjimky (tzv. chained[\[39\]](https://word2md.com/#footnote-39) exceptions) je technika, kdy postupným zkoumáním výjimek programátor zjistí bližší informace o zadané chybě. Asi nejjednodušší bude rychlý příklad.

Programátor píše aplikaci pro kopírování souborů v pravidelných časových intervalech (zálohování). Protože chce aplikaci navrhnout přehledně, rozdělí aplikaci do několika vrstev, které se mezi sebou postupně volají k vykonání dané funkcionality - nemá tedy všechen kód v jedné funkci, ale využívá hierarchického volání funkcí.

Na nejvyšší úrovni je objektu _Timer_ vyvolána virtuálním strojem metoda _intervalElapsed()_. Timer ví, že při uplynutí intervalu má zahájit zálohování zavoláním metody _doBackup()_ objektu _BackupManager_. Tento správce zálohování provede nalezení souborů a složek k zálohování a pro cyklicky pro každou nalezenou složku požádá o její zkopírování objekt _CopyManager_ zavoláním jeho metody _copyFolder()_. CopyManager při kopírování složky nalezne všechny vnořené soubory a pro každý soubor cyklicky zavolá metodu _copyFile()_.

Programátor také ví, že někde v procesu může nastat chyba, a tak na nejvyšší úrovni umístí blok _try-catch_ a volání _BackupManager.doBackup()_ provede v bloku _try_.

Při použití však na nejnižší úrovni vznikne výjimka typu _IOException_, která je zachycena na úrovni nejvyšší a programátor se tam dozví popis chyby (například): _IOException - Invalid file name_. Žádné další informace nemá, neví tedy vůbec, kdy v procesu zálohování k chybě došlo, v jaké složce a u jakého souboru.

Chtěl by tedy provést zachycení výjimky v metodě _copyFile()_ a přidat k ní vlastní informace týkající se názvu souboru, aby věděl, který soubor způsobil chybu. Jak ale efektivně ponechat původní výjimku a přidat k ní nějaké další, vlastní informace? Odpovědí jsou právě zřetězené výjimky.

Konstruktor každé výjimky má přetížení, kdy posledním parametrem je objekt typu _Throwable_ nazvaný _cause_ - příčina. Při vytváření objektů výjimek pomocí konstruktoru tedy můžeme jako poslední parametr dávat výjimku jinou. Tento proces lze dělat cyklicky, jak ukazuje následující kód.

```java
Exception first = new Exception("D");
Exception second = new Exception ("C", first);
Exception third = new Exception("B", second);
Exception fourth = new Exception("A", third);
```

Vytvořili jsme první výjimku s nějakým popisem („D"), vytvořili jsme druhou, s dalším popisem („C") a příčinou („D") atd., až čtvrtá výjimka má popis („A"). Vytvořená čtvrtá výjimka je tedy „A", má v sobě výjimku „C", která má v sobě výjimku „B" a ta má v sobě výjimku „A". Výjimky tedy postupně vytvořily řetězec - proto zřetězené výjimky. Datové typy výjimek se mohou lišit - objekt _cause_ je typován na datový typ _Throwable_, takže je schopen držet instanci libovolného typu jakékoliv výjimky.

Na příčinu libovolné výjimky se lze dostat pomocí metody _getCause()_ - pokud žádná výjimka vnořená není, metoda vrací hodnotu _null_.

Programátor ve výše uvedeném příkladu tedy vždy na každé úrovni vytvoří vlastní výjimku, přidá do jejího popisu důležité informace a vnořenou výjimku z předchozí úrovně. Výsledný objekt pomocí klauzule _throw_ vyhodí opět o úroveň výše. Výsledné výjimky tedy mohou obsahovat v dané metodě:

* _copyFile -_ název souboru. Vnořená výjimka obsahuje důvod chyby kopírování souboru;
* _copyFolder_ - název složky. Vnořená výjimky obsahuje název souboru, v ní vnořená obsahuje důvod chyby kopírování souboru;
* _doBackup_ - název zálohovací úlohy. Vnořená výjimka obsahuje název složky, v ní vnořená obsahuje název souboru, v ní vnořená obsahuje důvod chyby kopírování souboru;
* _intervalElapsed -_ tady programátor výjimku zpracuje, uloží do seznamu chyb a vypíše informaci uživateli. Má k dispozici výjimku, která obsahuje název zálohovací úlohy, v ní vnořená výjimka obsahuje název složky, v ní vnořená obsahuje název souboru, v ní vnořená obsahuje důvod chyby kopírování souboru;

Při takovémto řetězení výjimek se typicky používají vlastní vytvořené výjimky (ve smyslu vlastních datových typů), které samy, již typicky podle názvu, reprezentují daný typ chyby.

Tento způsob se používá nejčastě ve spojení s blokem _catch_, kde v tomto bloku nejdříve zachytíme vyvolanou původní výjimku, vytvoříme vlastní objekt s doplňujícími informacemi a vnoříme do něj původní výjimku; a následně tento vytvořený objekt vyhodíme do nadřazeného bloku.

```java
try {
    copyFile(source, target);
} catch (Exception ex){
    throw new Exception ("Failed to copy file.", ex);
}
```

Jak bylo zmíněno, pro procházení zřetězených výjimek následně slouží metoda _getCause()_. Jednoduchou funkci, která projde zadanou výjimku jako parametr a vypíše ji i všechny vnořené výjimky oddělené trojznakem ||| by mohl ukázat následující kód.

```java
public static String reportAllChainedExceptions(Throwable exception){
    StringBuilder ret = new StringBuilder();
    Throwable t = exception;
    while (t != null){
        ret.append(t.toString());
        ret.append(" ||| "); // oddeleni mezi vyjimkami
        t = t.getCause();
    }
    return ret.toString();
}
```

Funkce si uloží výjimku do lokální proměnné _t_. Následně v cyklu, dokud v _t_ není _null,_ provede vložení popisu výjimky do objektu _ret_ a do objektu _t_ zkusí vložit vnořenou výjimku (pokud existuje). Jakmile další vnořená výjimka není nalezena, cyklus se ukončí.

Při volání této funkce na objekt _fourth_ z předchozího výpisu dostaneme následující výstup:

```
java.lang.Exception: A ||| java.lang.Exception: B ||| java.lang.Exception: C ||| java.lang.Exception: D |||
```

## Výjimky a práce se zdroji -- try-with-resources

V dřívějších verzích javy byl problém pracovat s určitým typem zdrojů, které bylo třeba otevírat a po ukončení práce zase zavírat. Typicky obě tyto operace mohou vyvolat chybový stav a programátor pak neví, jak takovou situaci ošetřit - například: pokud se soubor podaří otevřít, ale nepodaří se načíst, je třeba ho zavřít. Zavření se ale také nemusí povést a je třeba jej kontrolovat pomocí bloků try-catch. A jak zotavit program v případě, kdy se zavření nepovedlo?

Protože takové kontroly generovali velké množství kódu a zanořené bloky try-catch(-finally), od Javy 7 přichází upravený příkaz _try_, nazvaný _try-with-resources_.

Princip této problematiky ponecháme nevysvětlen[\[40\]](https://word2md.com/#footnote-40). Základem však je, že jisté zdroje (ve smyslu inicializované proměnné) se po použití umí na příkaz virtuálního stroje samy uzavřít. Na jejich uzavírání se tedy explicitně nemusí hlídat pomocí try-catch-finally bloků. Syntaxe příkazu je:

```java
try (datový_typ_zdroje proměnná = inicializace) { /* obsah bloku */ }
```

Následuje krátký příklad.

```java
try (BufferedReader br =
        new BufferedReader(new FileReader(path))) {
    return br.readLine();
}
```

Třídy jsou pro čtenáře zřejmě zatím neznámé, proto bude jenom zběžně vysvětlen princip. _BufferedReader_ je třída, která umí z nějakého objektu vyčíst řetězec dat. Zdrojem tohoto objektu je v našem případě nějaký soubor reprezentovaný cestou _path_. Soubor se samozřejmě po ukončení práce musí korektně uzavřít, jinak by s ním ostatní aplikace nemohly pracovat. Uzavření souboru se však nemusí provádět ve větvi finally, protože je použita **syntaxe&#x20;**_**try-with-resources**_, která **se postará, že daný zdroj** (v našem případě soubor) **bude automaticky uzavřen po opuštění bloku&#x20;**_**try**_.
