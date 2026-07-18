# TODO Testování aplikací

## Testování aplikací

Při vývoji aplikaci musí programátor neustále kód, který vytvoří, kontrolovat na výskyt chyb. V praxi to znamená, že vytvoří nejen implementaci, která provádí požadovanou funkcionalitu, ale také vytvoří zdrojový kód, který tuto funkcionalitu otestuje. V ideálním případě by programátor měl testovat nejen to, že kód pro správné vstupy vykoná požadovaný výsledek, ale také to, jak se bude daný program chovat pro vstupy nesprávné - tedy například zadání data ve špatném formátu, zadání prázdné hodnoty místo očekávané hodnoty textu a podobně. Programátor **nikdy nesmí předpokládat**, že jeho zdrojový kód bude použit pouze správně, a vždy se musí věnovat i případu ošetření chyb. V programovacím jazyce Java je situace o to důležitější, že jazyk Java podporuje kontrolovaný výjimky (viz kapitola Výjimky a jejich zpracování).

Samozřejmě je možné veškeré funkcionality testovat tak, že po jejich napsání upravíme například metodu _main()_ tak, aby nově vytvořenou funkcionalitu spustila a vyzkoušeli tak požadované chování. Tím samozřejmě ověříme, zda všechno funguje tak, jak má, nicméně při vytvoření další části kódu se typicky původní ověřovací kód původní implementace smaže a nahradí se blokem novým, testujícím nový kód.

Vždy tak je aktivní pouze blok, který testuje nově vytvořenou část aplikace, a původní testovací bloky jsou smazány. Tím však nejen že programátor přichází o testovací kód, který dříve vytvořil, přichází také o mechanismus, který je schopen zkontrolovat, že **původně vytvořený kód funguje v pořádku** **i poté, co bylo přidáno či změněno něco jiného**. To je velmi důležité, protože dřívější testy jsou v dalším vývoji schopny zkontrolovat, zda se při změně či rozšíření kódu nezpůsobila omylem chybovost některého z původních bloků kódu.

Je proto vhodné mít možnost testy realizovat systematicky, ponechávat je a moci je automatizovaně spouštět hromadným způsobem, aby se podařilo zjistit, zda po změnách i všechny původní bloky fungují v pořádku. Takový mechanismus přináší knihovna _JUnit_ a princip testy řízeného vývoje, tzv. _TDD_.

### Obecný princip TDD

TDD je obecně technika vývoje softwaru, která spočívá na základě, že nejdříve se naprogramuje test, který ověřuje danou funkcionalitu, teprve potom se provede požadovaná implementace (typicky co nejjednodušším způsobem, aby to „nějak fungovalo" a teprve po ověření se výsledný kód opravuje. Zjednodušeně to lze popsat pomocí bodů:

* Vytvořit test, který ověřuje zatím nefungující funkcionalitu.
  * Tato část je důležitá, protože umožňuje, aby si programátor rozmyslel, **jak** bude daný kód používat, namísto toho **co** bude daný kód dělat. Typickou (špatnou) vlastností programátora je vymýšlení, co to bude dělat - a po implementaci programátor zjistí, že daná metoda/třída funguje správně, leč neposkytuje přesně to, co od ní potřebuje, a musí výsledný kód opravit.
* Spuštěný testu, který **musí selhat**, protože daná funkcionalita ještě vůbec nebude vytvořena.
* Vytvoření nějaké, co nejjednodušší implementace, který by měla způsobit úspěšné spuštění testu.
  * Cílem není napsat dokonalý kód, ale kód, který bude fungovat. Jedná se vlastně o nějaké prototypové ověření, že požadovaná funkcionalita jde vytvořit, že ji lze napsat.
* Spuštění testu, který by měl projít. Pokud test neprojde, vracíme se k bodu 3.
* Refaktoring - to je metoda, kdy se programátor dívá na hotový kód (v daném případě na naprogramovanou implementaci) a vymýšlí, jak jej vyčistit a zlepšit, aby byl čitelnější. Typickými úlohamy je přejmenování proměnných na smysluplnější, dekompozice funkcí na menší, odstranění zakomentovaných bloků kódu a další.
* Spuštění testu, který by měl projít i na refaktorovaném řešení.
* Dokud není programátor spokojený s vytvořenou funkcionalitou, vrací se k bodu 5. V opačném případě je daná funkcionalita implementovaná. **Test se neodstraňuje**, zůstává v balíku testů, které se budou nadále pouštět vždy s přidanou novou funkcionalitou. Pokud nějaký test v budoucnosti neprojde, znamená to, že se podařilo změnit i něco, co bylo naprogramováno dříve a některá část aplikace nyní nemusí fungovat korektně.

Tento princip je obecný, slouží jako motivace k řešení projektů s využitím TDD vývoje.

## Realizace v IntelliJ Idea

### Složka pro testy

Aby mohl testovací framework JUnit i samotné vývojové prostředí správně fungovat, nemůžeme testovací třídy ukládat na libovolné místo v projektu. **Čistá architektura softwaru vyžaduje striktní oddělení produkčního kódu od kódu testovacího.** V praxi to znamená, že testy nikdy nebalíme do výsledného programu (např. do souboru JAR) určeného pro koncové uživatele. Testy potřebujeme pouze během vývoje a v nástrojích pro kontinuální integraci. Z toho důvodu vzniká v projektu paralelní adresářová struktura.

Většina moderních javových projektů postavených na nástrojích Maven nebo Gradle používá standardizované uspořádání. Zatímco produkční zdrojové kódy se nacházejí v adresáři `src/main/java`, pro testy se zakládá sesterská složka `src/test/java`. Pokud student vytváří projekt od nuly ručně nebo pracuje na specifickém zadání, může se stát, že tuto složku (např. s jednoduchým názvem `test` nebo `tests`) musí vytvořit sám. Samotná existence složky na disku však vývojovému prostředí nestačí. IntelliJ IDEA totiž ke každému adresáři v projektu přistupuje na základě jeho specifické role.

{% hint style="info" %}
Pokud jste projekt zákládali jako klasiký Java projekt v IntelliJ Idea, budete mít vytvouřenou pouze složku na zdrojové kódy - `src`.
{% endhint %}

![Přehled projektu bez složky pro testy](.gitbook/assets/projectview.png)

V takovém případě do projektu musíme vytvořit novou složku (ideálně na úrovni složky `src`), kterou můžeme nazvat libovolně, ale běžně se používají názv jako `tst` , `test` nebo `tests`.

Pokud nově vytvořené složce pro testy neexplicitně nedefinujeme její význam, IntelliJ IDEA ji považuje za běžný adresář s textovými soubory. IDE v této složce nebude správně doplňovat kód Javy, nebude nabízet automatické importy pro knihovnu JUnit a především u testovacích tříd vůbec nezobrazí zelené spouštěcí šipky.&#x20;

Proto musíme složku označit jako takzvaný _Test Sources Root_. V IntelliJ IDEA toho docílíme kliknutím pravým tlačítkem myši na danou složku v levém panelu projektu, navigováním na položku _Mark Directory as_ a zvolením možnosti _Test Sources Root_. Jakmile tento krok provedeme, ikona složky v prostředí IntelliJ IDEA změní barvu ze standardní žluté na specifickou zelenou.

![Menu "Mark Directory"](.gitbook/assets/test-markdirectory.png)

Tímto vizuálním tahem dáme vývojovému prostředí najevo, že soubory uvnitř této složky jsou plnohodnotné javové třídy, které mají přístup ke všem produkčním třídám v `src/main/java`, ale zároveň jsou izolovány pro účely testování. IntelliJ IDEA okamžitě přepne daný adresář do režimu plné podpory Javy – začne hlídat syntaktické chyby, správnost balíčků, umožní refaktorizaci a hlavně aktivuje spouštěč testů JUnit. Při automatickém generování testů pomocí klávesové zkratky pak IDE bude přesně vědět, kam má nově vznikající testovací třídy ukládat, a automaticky v zelené složce replikuje balíčkovou strukturu z produkční části projektu.

![Test directory created](.gitbook/assets/test-folder.png)



### Vytvoření vlastního testu

Test můžeme vytvořit dvěma způsoby:

* necháme si ho vygenerovat přes prostředí IntelliJ, které samo vytvoří potřebný soubor a případně nageneruje další kód,
* vytvoříme si ho ručně a kód budeme implmentovat sami.

#### Vytvoření testu ručně

Pro vytvoření testu ručně stačí do složky **s testy** vložit novou třídu (jako klasickou třídu do projektu) a do ní doplnit požadovaný kód. V základu se může jednat o úplně jednoduchou třídu.&#x20;

{% hint style="info" %}
Testovací třídy mají typicky postfix `Test`.
{% endhint %}

```java
public class CarTest {
}
```

Do takové třídy budeme dále psát kód dle potřeby. Je důležité si povšimnout, že třída se opravdu musí vytvořit do testové složky.

#### Vytvoření testu automaticky

Automatické vytvoření testu může být výhodnější - jednak proto, že nemusíme psát potřebný kód, ale také proto, že u nového projektu většinou chybí předinstalovaná knihovna pro testování a automatický proces nám pomůže knihovnu do projektu připojit.

V IntelliJ Idea umístíme kurzur nad název třídy, pro kterou chceme dělat test (v našem případě do názvu `Car`) a otevřeme kontextové menu (Alt+Enter). Z nabídky vybereme položku "Create test".

{% hint style="info" %}
Pozor, i v tomto případě již musíme mít v projektu vytvořenou a nastavenou testovací složku. V opačném případě nám bude protředí testovací třídy dávat mezi běžné třídy projektu, což je nechtěné chování.
{% endhint %}

![Menu "Create Test" pro automatické vytvoření testu](.gitbook/assets/test-create.png)

Po potvrzení volby se otevře dialog "Create Test" pro vytvoření testu. V tomto dialogu:

* Můžeme vybrat testovací knihovnu. Pro javu je k dispozici několik testovacích frameworků, my si budeme ukazovat framework "JUnit"; vybereme odpovídající verzi (aktálně verzi 5 nebo 6).
* Můžeme doplnit chybějící knihovnu - symbol 💡 říká, že v projektu chybí odpovídající knihovna. Stiskem tlačítka "Fix" ji do projektu můžeme nechat automaticky přidat (pouze potvrdíme následně otevřený dialog).
* Class Name - udává, jak se bude naše testovací třída jmenovat. Typicky se jmenuje jako původní třída s přidaným postfixem `Test`.
* Superclass - chceme-li, můžeme definovat předka pro naši testovací třídu, který už může obsahovat nějakou předdefinovanou funkcionalitu.
* Destination package - můžeme definovat balíček, do kterého se třída bude vytvářet.

![Obrazovka "Create Test"](.gitbook/assets/test-createscreen.png)

* Varianty `setUp`/`@Before` a `tearDown`/`@After` slouží k vytvoření metod, které se volají před a po testech. Bližší vysvětlení bude uvedeno dále.
* Poslední část okna nabízí zašktávací pole pro automatické generování **koster** testů pro vybrané metody. Můžeme je zvolit a nechat si testy vygenerovat; v našem případě si však testy napíšeme sami.

Po potvrzení dostaneme vytvořenou testovací třídu se základním obsahem a případně vygenerovaným kódem:

```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

class CarTest {
}
```

### Spuštění testu v Idea

Poslední lehce komplikovanou oblastí pro neznalé je spuštění testu.





![Ukázka výsledku testu](.gitbook/assets/test-result.png)

configsel

![Zvolení konfigurace](.gitbook/assets/configurationselection.png)

config

![Run/Debug Configurations window](.gitbook/assets/configurationwindow.png)

### Realizace TDD v NetBeans

V jazyce Java je testy řízený vývoj (nebo obecně testování[\[10\]](https://word2md.com/#footnote-10)) lze provádět s využitím knihoven - nejčastěji se používá knihovna nazvaná JUnit.

Knihovna JUnit obsahuje třídy a metody, které slouží pro vyvolávání testů a jejich vyhodnocení a publikování výsledků programátorovi.

Celé řešení bude ukázáno na příkladu třídy _SMS_, která slouží pro práci se SMS zprávami. V této třídě bude metoda _compressAsSms()_, která na vstupu bere text (řetězec) a vrací výstup (také řetězec), který komprimuje takovým způsobem, že odstraní všechny mezery a první znaky v začátcích slov nahradí velkými písmeny, vyjma mezer za čárkou nebo na konci věty.

V kódu použijeme příkaz

throw new NotImplementedException();

, který způsobí, že se běh aplikace zastaví chybou. Tento příkaz můžeme (a je to vhodné) používat kdykoliv, kdy se vytváří blok, jehož implementace se doplní později. Běhové prostředí Javy v tomto případě při volání tohoto bloku upozorní, že není implementován a že před použitím je třeba kód doplnit. Naopak, je krajně nevhodné takové bloky kódu nechávat projít bez chyby, protože se na ně typicky zapomene a následně se v kódu stanou nedohledatelnými.

Zdrojový kód třídy _SMS_ může na počátku vypadat například takto:

package eng.demos.tdd;\
import sun.reflect.generics.reflectiveObjects.NotImplementedException;\
public class SMS {\
\*\*public static String compressAsSms(String message){\
\*\*throw new NotImplementedException();\
\*\*}\
\*\*}

Funkce _compressAsSMS()_ je připravena, nic však nerealizuje.

#### Vytvoření testu

Prvním krokem tedy bude vytvoření testu, který ji bude ověřovat. V prostředí _NetBeans_ nad souborem _SMS.java_ s požadovanou třídou k testování zvolíme v kontextovém menu volby _Tools -> Create Tests_.

Obrázek - Vygenerování testu v NetBeans

V otevřeném dialogovém okně _Create Tests_ zvolíme název třídy, do které se mají testy generovat - testy se negenerují do třídy, která se testuje, ale do druhé třídy, která má typicky stejný název jako testovaná třída s dodatkem _Test_ - v tomto případě bude mít testovací třída název například _SMSTest_. Další možností je zvolit, které všechny členy tříd v dané třídě je vhodné testovat - pro každého člena třídy bude následně vytvořena metoda, která ověří jeho chování.

Obrázek - Volba názvu a obsahu testovací třídy

Pokud se v projektu vytváří první test, projekt potřebuje připojit odkaz na knihovnu, která bude testování provádět. Knihovna JUnit je v prostředí _NetBeans_ používána ve dvou verzích - 3.x a 4.x. V současném řešení je samozřejmě vhodné vždy používat knihovnu nejnovější, tedy 4.x. Po zvolení dialogu může být programátor volitelně požádán prostředím _NetBeans_ o povolení automatického stažení balíčku s touto knihovnou.

Obrázek - Volba verze JUnit

V prostředí _NetBeans_ se po ukončení vkládání a generování testovací třídy projeví dvě změny. První změnou je ve stromu _Projects_, větvi _Test Libraries_, kde přibude odkaz na knihovnu _JUnit 4.10-juni4-10.jar_. Druhou změnou je ve stromu ve větvi _Test Packages_, kde se objeví balíček i zdrojový kód třídy _SMSTest_.

Obrázek - Připojení knihovny JUnit do testovacích knihoven

Zdrojový kód třídy _SMSTest_ je ukázán níže.

package eng.demos.tdd;\
import org.junit.\*;\
import static org.junit.Assert.\*;\
public class SMSTest {\
public SMSTest() {\
}\
@BeforeClass\
public static void setUpClass() {\
}\
@AfterClass\
public static void tearDownClass() {\
}\
@Before\
public void setUp() {\
}\
@After\
public void tearDown() {\
}\
\*\*public void testCompressAsSms() {\
\*\*System.out.println("compressAsSms");\
String message = "";\
String expResult = "";\
String result = SMS.compressAsSms(message);\
assertEquals(expResult, result);\
// TODO review the generated test code and remove

// the default call to fail.\
fail("The test case is a prototype.");\
\*\*}\
\*\*}

Ve výpisu si lze povšimnout čtyř zajímavých metod, které blíže nebudou osvětleny, ale jejích význam je snad zřejmý z následující tabulky:

| Název metody    | Význam                                                                                                                                                                                                       |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| setUpClass()    | Volá se **před** spuštěním **první** ze všech testovacích metod, které se budou v této třídě vykonávat při testování. Slouží typicky k nastavení proměnných a hodnot, které jsou společné pro všechny testy. |
| tearDownClass() | Volá se **po** dokončení **poslední** ze všech testovacích metod, které se v této třídě vykonaly. Typicky slouží k úklidu po provedení testů.                                                                |
| setUp()         | Volá se **před** spuštěním **každé** jednotlivé testovací metody dané třídy.                                                                                                                                 |
| tearDown()      | Volá se **po** dokončení **každé** testovací metody dané třídy.                                                                                                                                              |

Tyto metody tedy slouží pro nastavení třídy před spuštěním testů a také pro úklid po provedení všech testů.

Důležitou testovací metodou je metoda _testCompressAsSMS()_, která je právě testovací metodou pro vytvářenou metodu _compressAsSms()_ třídy _SMS._ Tato metoda již obsahuje nějaký zdrojový kód, jehož jednotlivé řádky lze popsat.

@Test\
public void testCompressAsSms() {\
// vytiskně info na konzoli, jaký test se vlastně provádí\
System.out.println("compressAsSms");\
// "message" je vstupní parametr testovací metody, na tomto řádku může programátor\
// nastavit, co bude do dané metody vstupovat pro testovací ověření\
String message = "";\
// "expResult" je proměnná, do které programátor v testu nastaví očekávaný výsledek\
// po volání testované metody (tedy "expected result")\
String expResult = "";\
// Toto je samotné vyvolání testovací metody. Metoda "compressAsSMS()" je statická,\
// nepotřebuje tedy instanci. V opačném případě by bylo třeba ještě vytvořit instanci,\
// nad kterou by se daná metoda vyvolala\
String result = SMS.compressAsSms(message);\
// Testovací metoda "assert", která ověřuje, zda jsou výsledky testování shodné.\
// Tato metoda poskytuje výsledek prostředí, zda se test povedl, či nikoliv.\
// Pozor, po vykonání této metody funkce pokračuje dál! Nekončí se tedy tímto příkazem\
// ani v případě, že test úspěšně projde!\
assertEquals(expResult, result);\
// Následující řádek je nagenerován, aby každý test po vytvoření generováním\
// automaticky selhal. Až programátor nahoře doplní požadované vstupy a výstupy\
// testu, následující řádek smaže.\
fail("The test case is a prototype.");\
}

#### Příprava a spuštění testu

Dalším krokem je příprava a spuštění testu. V přípravě nastavíme proměnné, které vstupují jako testovací do testu, na hodnoty, které chceme testovat. V ukázaném příkladu nastavíme hodnotu proměnné _message_ například na řetězec „Ahoj, Karle. Jdeme na squash?". Odpovídajícím způsobem musíme nastavit také hodnotu proměnné, která reprezentuje úspěšný očekávaný výsledek po volání této funkce, tedy v tomto případě: „Ahoj,Karle.JdemeNaSquash?"

@Test\
public void testCompressAsSms() {\
System.out.println("compressAsSms");\
\*\*String message = "Ahoj, Karle. Jdeme na squash?";\
String expResult = "Ahoj,Karle.JdemeNaSquash?";\
\*\*\
String result = SMS.compressAsSms(message);\
assertEquals(expResult, result);\
\~\~fail("The test case is a prototype.");\
\~\~\
}

Protože byl test připraven, odstraníme ještě poslední řádek s příkazem _fail_, který má selhat, když programátor vstupní a výstupní hodnoty testu ještě nenastavil.

Nyní lze test spustit. Test se spustí klávesovou zkratkou _Alt+F6_, nebo z menu _Run -> Test Project_.

Test po spuštění zobrazí výsledek pomocí barevné čáry.

Obrázek - Ukázka spuštění neúspěšného testu

Test neprošel. To je samozřejmé, protože implementace ještě nebyla vytvořena. Proto bude proveden nástřel implementace (význam metod v tomto případě není důležitý, ale je vhodné se k tomuto testu vrátit po prostudování problematiky práce s řetězecem). Funkcionalita tedy nebude představena podrobně, jde pouze o ilustrační příklad. Funkcionalita bude tedy vytvořena například takto.

public static String compressAsSms(String message){\
boolean f = false;\
StringBuilder sb = new StringBuilder();\
for (int i = 0; i < message.length(); i++) {\
char c = message.charAt(i);\
if (c == ' ')\
f = true;\
else{\
if (f){\
sb.append(Character.toUpperCase(c));\
f = false;\
}else{\
sb.append(c);\
}\
}\
}\
return sb.toString();\
}

Po spuštění tohoto kódu dostaneme úspěšný výsledek testu.

Přesto však kód není dokonalý. Je vhodné jej trochu upravit, hlavně pojmenovat proměnné, protože takto je velmi nečitelný.

Pro přejmenování proměnných použijeme tzv. _refactoring_. Po umístění kurzoru do textu proměnné lze stisknout klávesu _Ctrl+R_ a přepíšeme název proměnné na nový text. Následně potvrdíme klávesou _Enter_. Upravený zdrojový kód může vypadat například takto.

public static String compressAsSms(String message){\
boolean wasLastCharSpace = false;\
StringBuilder result = new StringBuilder();\
for (int i = 0; i < message.length(); i++) {\
char currentChar = message.charAt(i);\
if (currentChar == ' ')\
// aktualni je mezera, nepridava se\
wasLastCharSpace = true;\
else{\
// pokud byla predchozi mezera...\
if (wasLastCharSpace){\
// ... prevedeme znak na velka\
result.append(Character.toUpperCase(currentChar));\
wasLastCharSpace = false;\
}else{\
// .. jinak pouze pridame aktualni znak\
result.append(currentChar);\
}\
}\
}\
return result.toString();\
}

Po přejmenování proměnných a po doplnění komentářů bude text pro programátora mnohem čitelnější.

Opět spustíme test, aby se prokázalo, že změnou při redaktorování se neprovedla chyba a kód je stále v pořádku.

#### Vlastní tvorba testu

Při tvorbě testu není třeba se omezit pouze na nagenerované testy, ale je možno vytvářet i testy vlastní. Klasickým případem navazujícím na přechozí příklad je otestování nestandartních vstupů. Nechť je do třídy _SMSTest_ vytvořen nový test, který ověřuje, co se stane, když do testované metody _compressAsSms_() vstoupí jako parametr hodnota _null_.

Nejdříve se tedy vytvoří prázdný test. Vytvoří se nová metoda (ve třídě _SMSTest_), a před ni se doplní tzv. anotace, tedy řetězec @Test. Podle něj prostředí _NetBeans_ pozná, že se daná metoda má spouštět jako testovací.

\*\*@Test\
\*\*public void testCompressAsSmsWithNull() {\
}

Dalším krokem je vytvoření očekávaných vstupních a výstupních hodnot. Jako vstup bude sloužit hodnota _null_, co s výstupem? To musí vědět ten, co program vytváří - jak se má metoda s takovým vstupem chovat? Má vyvolat chybu? Nebo má vrátit také hodnotu _null_? To musí vědět zadavatel - tvůrce zdrojového kódu. Řekněme, že v tomto příkladu bude očekáván výsledek _null_. Tedy:

@Test\
public void testCompressAsSmsWithNull() {\
\*\*String message = null;\
String expResult = null;\
\*\*}

Nyní bude doplněno samotné volání funkce.

@Test\
public void testCompressAsSmsWithNull() {\
String message = null;\
String expResult = null;\
\*\*String result = SMS.compressAsSms(message);\
\*\*}

A konečně, pomocí funkce _assertEquals()_ bude ověřeno, zda je opravdu navrácený výsledek shodný s očekávaným výsledkem. Do funkce je vhodné ještě doplnit úvodní výpis na konzoli, aby na konzoli bylo prokazatelně vidět vyvolání testu.

@Test\
public void testCompressAsSmsWithNull() {\
\*\*System.out.println("testCompressAsSmsWithNull");\
\*\*\
String message = null;\
String expResult = null;\
String result = SMS.compressAsSms(message);\
\*\*assertEquals(expResult, result);\
\*\*}

Následně lze všechny testy spustit a počkat na výsledek.

Obrázek - Spuštění ručně přidaného testu

Ve výsledku jde vidět, že už se vykonávají testy dva. Úspěšně však prošla pouze polovina, nově vytvořený test na test hodnoty _null_ neprošel. V okně si lze také zkontrolovat důvod neúspěchu testu. Je vidět, že test vyvolal výjimku _java.lang.NullPointerException_, tedy, že někde se pracuje s hodnotou null a není to možné.

Postupnou kontrolou by bylo zjištěno, že k chybě dojde v metodě _compressAsSMS(null)_ na příkazu

for (int i = 0; i < message.length(); i++) {

, protože metodu _length()_ nelze vyvolat nad objektem _null_, protože _null.length()_ nedává smysl. Původní funkci tedy stačí lehce poupravit.

public static String compressAsSms(String message){\
\*\*if (message == null) return null;\
\*\*\
...\
}

Po doplnění jednoho řádku již oba testy projdou úspěšně.

#### Poznámky k testování

V uvedeném příkladu bylo testování provedeno nad statickou metodou. Pokud je metoda instanční, potřebujeme vytvořit instanci. Například pro testování kalkulačky:

public class Calculator {\
public double multiply (double a, double b){\
return a \* b;\
}\
}

… bude test navíc obsahovat vytvoření instance pomocí klíčového slova _new_:

@Test\
public void testMultiply() {\
System.out.println("multiply");\
double a = 5.5;\
double b = 2.0;\
Calculator instance = new Calculator();\
double expResult = 11.0;\
double result = instance.multiply(a, b);\
assertEquals(expResult, result, 0.0);\
}

Další úpravou je volání funkce _assertEquals_.

**Pro neceločíselné typy:**

Počítač automaticky zaokrouhluje výsledky s přesností tak, aby se mu hodnoty vešly do paměti přidělené pro daný datový typ. Proto pokud sečteme i s použitím typu _float_ například 1 + 0,000000000000000000000000000001, výsledkem bude opět hodnota 1. Počítač desetinnou část, která je nepatrná a do paměti se mu nevešla, opomine. V jednoduchém testu umocníme hodnotu PI na druhou a následně ji opět odmocníme - a vše vyjde v pořádku, test bude úspěšný.

@Test\
public void testMultiply2() {\
double temp = Math.pow(Math.PI, 2.0);\
double result = Math.pow( temp, 1/2.0);\
double expResult = Math.PI;\
assertEquals(expResult, result, 0.0);\
}

Pokud však v kódu již využijeme přetypování na rozsahově menší typ _float_, výsledek se bude drobně odlišovat. I drobná odchylka již způsobí, že test neprojde.

@Test\
public void testMultiply2() {\
float temp = (float) Math.pow(Math.PI, 2.0);\
double result = Math.pow( temp, 1/2.0);\
double expResult = Math.PI;\
assertEquals(expResult, result, 0.0);\
}

Tento test se vrátí s chybou:

Failed: expected: <3.141592653589793>, but was: <3.1415926073757197>

Proto při práci s desetinnýmí čísly se v metodě _assertEquals_ používá i třetí parametr, tzv. _delta_, která říká, o kolik se mohou dvě desetinná čísla lišit, aby byla ještě považována za ekvivalentní. Upravený test s přidanou hodnotou _delta_ již projde v pořádku.

@Test\
public void testMultiply2() {\
float temp = (float) Math.pow(Math.PI, 2.0);\
double result = Math.pow( temp, 1/2.0);\
double expResult = Math.PI;\
assertEquals(expResult, result, **0.000001**);\
}

**Testování polí:**

Pole nelze porovnávat jako _assertEquals_, protože dvě pole se mohou v paměti nacházet na odlišných místech a funkce _assertEquals()_ porovnává adresy jejich uložení. Pokud tedy budou dvě pole, obě obsahovat stejné hodnoty, ale budou se nacházet na různých místech v paměti, metoda selže a bude tvrdit, že pole se liší.

@Test\
public void testArray() {\
int \[] a = new int \[]{ 1, 2, 3};\
int \[] b = new int \[] { 1, 2, 3};\
assertEquals(a,b);\
}

Takový test se vrátí s chybou, že očekávaná hodnota byla \[I&53431, ale nalezená hodnota \[I@8674 - tedy, že pole nejsou shodná. Přitom však lze vidět, že pole obsahují stejné hodnoty. Pokud je třeba porovnávat pole (a kolekce) na rovnost jejich jednotlivých prvků, je třeba využít funkce _assertArrayEquals()_.

@Test\
public void testArray() {\
int \[] a = new int \[]{ 1, 2, 3};\
int \[] b = new int \[] { 1, 2, 3};\
assertArrayEquals(a, b);\
}

**Vlastní chybová zpráva:**

Všechny příkazy _assert….()_ mohou mít jako první, nepovinný, textový parametr, který říká, jaká chybová hláška se má vypsat, pokud test selže.

@Test\
public void testOwnMessage() {\
assertEquals("Toto je vlastní zpráva.", 5, 7);\
}

Tento test selže (5 != 7) a vypíše se vlastní chybová zpráva.

Obrázek - Vlastní chybová zpráva v testu

**Nucené selhání testu:**

Je-li třeba v určitou chvíli nechat test selhat, lze využít volání příkazu _fail()_. Ten může brát jako parametr informaci o selhání testu. Test je chápán jako úspěšný, **pokud neselhal**, a test může selhat buď při volání funkce _assert…()_, nebo příkazem _fail()_. Pokud tedy vytvoříme prázdnou metodu, ve které se žádné vyhodnocení neprovede, bude tato metoda chápána jako úspěšné projití testu.

Bude-li jednoduchý příklad třídy, která vrací konstantní hodnotu:

public class GetNumber {\
public int getZero(){\
return 0;\
}\
}

Jednoduchý test, který toto ověřuje, může mít klasický výraz _assertEquals(result, 0)_, ale také jej lze zapsat jako:

@Test\
public void testGetZero() {\
System.out.println("getZero");\
GetNumber instance = new GetNumber();\
int result = instance.getZero();\
if (result != 0)\
fail();\
}

Příkaz _fail()_ způsobí selhání testu. Jiné vyhodnocení testu se nevyskytuje, takže v opačném případě bude test chápán jako úspěšný. Tato technika se používá ve složitějších případech, kdy je složité, nebo nemožné napsat ohodnocení pomocí metody _assert…()_.

**Kontrolní otázky:**

* Jaké jsou hlavní výhody automatizovaného testování?
* K čemu slouží metody „assert…()"?
* Lze nechat libovolný test selhat? Jak?

Úkoly k zamyšlení:

Jak realizovat testování, pokud nemáte k dispozici JUnit?

Korespondenční úkol:

Realizujte kalkulačku s operacemi sčítání, odčítání, násobení a dělení jako třídu s metodami. Otestujte všechny tyto metody pomocí testů. Nezapomeňte na chybové stavy při dělení.

Shrnutí obsahu kapitoly

Kapitola představila problematiku testů a testování malých bloků aplikace při jejím výhody. Ukázala, proč automatizované testování pomáhá udržet kvalitu aplikace a jak může přispět k tomu, že v aplikaci bude v budoucnosti možno detekovat chyby způsobené změnou existujícího kódu.
