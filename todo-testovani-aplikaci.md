# TODO Testování aplikací

Při vývoji aplikaci musí programátor neustále kód, který vytvoří, kontrolovat na výskyt chyb. V praxi to znamená, že vytvoří nejen implementaci, která provádí požadovanou funkcionalitu, ale také vytvoří zdrojový kód, který tuto funkcionalitu otestuje. V ideálním případě by programátor měl testovat nejen to, že kód pro správné vstupy vykoná požadovaný výsledek, ale také to, jak se bude daný program chovat pro vstupy nesprávné - tedy například zadání data ve špatném formátu, zadání prázdné hodnoty místo očekávané hodnoty textu a podobně. Programátor **nikdy nesmí předpokládat**, že jeho zdrojový kód bude použit pouze správně, a vždy se musí věnovat i případu ošetření chyb. V programovacím jazyce Java je situace o to důležitější, že jazyk Java podporuje kontrolovaný výjimky (viz kapitola Výjimky a jejich zpracování).

Samozřejmě je možné veškeré funkcionality testovat tak, že po jejich napsání upravíme například metodu _main()_ tak, aby nově vytvořenou funkcionalitu spustila a vyzkoušeli tak požadované chování. Tím samozřejmě ověříme, zda všechno funguje tak, jak má, nicméně při vytvoření další části kódu se typicky původní ověřovací kód původní implementace smaže a nahradí se blokem novým, testujícím nový kód.

Vždy tak je aktivní pouze blok, který testuje nově vytvořenou část aplikace, a původní testovací bloky jsou smazány. Tím však nejen že programátor přichází o testovací kód, který dříve vytvořil, přichází také o mechanismus, který je schopen zkontrolovat, že **původně vytvořený kód funguje v pořádku** **i poté, co bylo přidáno či změněno něco jiného**. To je velmi důležité, protože dřívější testy jsou v dalším vývoji schopny zkontrolovat, zda se při změně či rozšíření kódu nezpůsobila omylem chybovost některého z původních bloků kódu.

Je proto vhodné mít možnost testy realizovat systematicky, ponechávat je a moci je automatizovaně spouštět hromadným způsobem, aby se podařilo zjistit, zda po změnách i všechny původní bloky fungují v pořádku. Takový mechanismus přináší knihovna _JUnit_ a princip testy řízeného vývoje, tzv. _TDD_.

## Obecný princip TDD

TDD je obecně technika vývoje softwaru, která spočívá na základě, že nejdříve se naprogramuje test, který ověřuje danou funkcionalitu (který samořezjmě musí selhat, protože funkcionalita ještě neexistuje), teprve potom se provede požadovaná implementace (typicky co nejjednodušším způsobem, aby to „nějak fungovalo") a teprve po ověření se výsledný kód opravuje. Zjednodušeně to lze popsat pomocí bodů:

* 1\. Vytvořit test, který ověřuje zatím nefungující funkcionalitu.
  * Tato část je důležitá, protože umožňuje, aby si programátor rozmyslel, **jak** bude daný kód používat, namísto toho **co** bude daný kód dělat. Typickou (špatnou) vlastností programátora je vymýšlení, co to bude dělat - a po implementaci programátor zjistí, že daná metoda/třída funguje správně, leč neposkytuje přesně to, co od ní potřebuje, a musí výsledný kód opravit.
* 2\. Spuštěný testu, který **musí selhat**, protože daná funkcionalita ještě vůbec nebude vytvořena.
* 3\. Vytvoření nějaké, co nejjednodušší implementace, který by měla způsobit úspěšné spuštění testu.
  * Cílem není napsat dokonalý kód, ale kód, který bude fungovat. Jedná se vlastně o nějaké prototypové ověření, že požadovaná funkcionalita jde vytvořit, že ji lze napsat.
* 4\. Spuštění testu, který by měl projít. Pokud test neprojde, vracíme se k bodu 3.
* 5\. Refaktoring - to je metoda, kdy se programátor dívá na hotový kód (v daném případě na naprogramovanou implementaci) a vymýšlí, jak jej vyčistit a zlepšit, aby byl čitelnější. Typickými úlohamy je přejmenování proměnných na smysluplnější, dekompozice funkcí na menší, odstranění zakomentovaných bloků kódu a další.
* 6\. Spuštění testu, který by měl projít i na refaktorovaném řešení.
* Dokud není programátor spokojený s vytvořenou funkcionalitou, vrací se k bodu 5. V opačném případě je daná funkcionalita implementovaná. **Test se neodstraňuje**, zůstává v balíku testů, které se budou nadále pouštět vždy s přidanou novou funkcionalitou. Pokud nějaký test v budoucnosti neprojde, znamená to, že se podařilo změnit i něco, co bylo naprogramováno dříve a některá část aplikace nyní nemusí fungovat korektně.
* 7\. Nakonec by se měly spustit všechny (nebo relevantní sada testů), aby se ověřilo, že vzniklá funkcionalita nerozbila funkčnost ovlivňující některý z dřívějších testů.

Tento princip je obecný, slouží jako motivace k řešení projektů s využitím TDD vývoje. Je vhodné se jím řídit, nicméně i u obecného programování je vhodné používat testy pro ověření funkčnosti navržených metod.

Je důležité si ještě představit, jak se testy spouštějí. Testy nespouští přímo programátor, ten pouze "požádá" vývojové prostředí o spuštění testů. Vývojové prostředí samo spustí testy a provede jejich vyhodnocení. Důležitým efektem je, že **programátor sám** po spuštění testu **nevyhodnocuje, zda byl test úspěšný či nikoliv**, to řeší prostředí. Programátor pouze dostane výstupní informaci o tom, co se povedlo a co nikoliv.

## Obecný test v JUnit

### Základní vzhled testu

Obecný test v JUnit je reprezentován jednoduchou metodou. Pro tuto metodu platí:

* Nesmí být `private`, protože pak by ji testovací mechanismus neviděl a nemohl spustit.
* Nesmí přijímat žádné parametry, protože by testovací mechanismus nevěděl, jaké parametry má do metody přidat (viz poznámka níže).
* Má před sebou anotaci `@Test`, podle které testovací mechanismus rozpozná, které metody jsou testovací a které jsou pouze pomocné.

Oproti tomu název třídy, ve kterých jsou testovací metody, může být libovolný.

{% hint style="info" %}
**Parametry u testovací metody?**

Některé pokročilejší přístupy umožňují definovat, že testovací metoda má vstupní parametry. Zároveň ale také definuje mechanismus, pomocí kterého se hodnoty těchto parametrů budou plnit.&#x20;

Jedná se o pokročilejší oblast a studenty se zájmem odkazujeme například na studijní oporu k předmětu KIP/7TETL, či [https://docs.junit.org/6.1.2/writing-tests/parameterized-classes-and-tests.html](https://docs.junit.org/6.1.2/writing-tests/parameterized-classes-and-tests.html).
{% endhint %}

V těle metody se typicky vyskytuje:

* definice proměnné obsahující očekávaný výsledek
* definice proměnné obsahující reálný výsledek získaný výpočtem
* **povinně** příkaz provádějící automatické zhodnocení, zda se test povedl či nikoliv.

Před podrobnějším vysvětlením příklad:

```java
package cz.skripta.model;

import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

class CalculatorTest {

    @Test
    void testSumWithIntegerValues() {
        // Arrange - Příprava celočíselných vstupů a očekávaného výsledku
        int firstValue = 2;
        int secondValue = 3;
        int expectedResult = 5;

        // Act - Volání statické metody pro celá čísla
        int actualResult = Calculator.sum(firstValue, secondValue);

        // Assert - Přímé porovnání dvou celých čísel
        assertEquals(expectedResult, actualResult);
    }
}
```

V příkladu předpokládáme, že testujeme metodu `static int sum(int a, int b)` definovanou ve třídě `Calculator`. Metoda je statická a dělá triviální součet dvou čísel.

Všimněme si, že test je definovaný jako metoda ve třídě. Metoda má interní viditelnost, anotaci `Test` a nemá žádné parametry.

V metodě se definují vstupní data - proměnné `firstValue` a `secondValue`; dále se definuje `expectedResult` - to je hodnota "kolik by měl být správný výsledek" a hodnota `actualResult`  - kolik vrátil implementovaný výpočet: vidíme, že tuto hodnotu získáme reálným zavoláním metody `Calculator.sum(...)`.

Důležitý je poslední příkaz - funkce `assertEquals(...)` vyhodnotí, zda jsou hodnoty shodné. Pokud ne, test selže.

{% hint style="info" %}
Zajímavé může být, kde se vzala metoda `assertEquals(...)`. Tato metoda je statickou metodu třídy `org.junit.jupiter.api.Assertions`. V  záhlaví je uveden příkaz `import static`, který importuje všechny statické funkce třídy `Assertions` k přímému použití.
{% endhint %}

### Princip vyhodnocení testu

Důležitý je mechanismus, který vyhodnocuje, kdy je test úspěšný a kdy nikoliv.&#x20;

Obecně může výpočet testu skončit třemi stavy:

* Chyba při výpočtu - test nelze vyhodnotit, protože v průběhu testu vznikla chyba, která zabránila jeho dokončení. Takový test se zobrazí 🔴 červeně a je chápán jako neúspěšný. Tato varianta vznikne, pokud je implementace nekorektní a padá s chybou.
* Výpočet se provedl, ale výsledek nesedí - test lze vyhodnotit, ale vyhodnocení nebylo úspěšné - výsledek nesedí s očekávaným výsledkem. Takový test se zobrazí 🟠 oranžově/žlutě a je chápán jako neúspěšný. Tato varianta znamená, že implementace vrátí nějaký výsledek, ale ten nesedí s očekávanou hodnotou.
* Výpočet se provedl a nedošlo k žádné chybě - test lze vyhodnotit a při vyhodnocení očekávaných výsledků nevznikl žádný problém. Takový test se zobrazí 🟢  zeleně a je chápán jako úspěšný. Tato varianta znamená, že test se provedl a výsledek sedí s očekávanou hodnotou.

{% hint style="info" %}
Pokud znáte výjimky, tak:

* test je 🔴 neúspěšný, pokud je při jeho zpracování vyhozena výjimka (mimo další bod);
* test je 🟠 neúspěšný, pokud je při jeho zpracování vyhozena pouze výjimka/y typu `AssertionError`;
* test je 🟢 úspěšný, pokud neselže s žádnou chybou.
{% endhint %}

Případy 1 a 3 jsou asi jasné. Zajímavá oblast je, jak testovat, zda se získaná data shodují s daty očekávanými.

### Vyhodnocení výsledku testu

Základním kamenem ověřovací fáze každého unit testu je zjištění, zda se to, co program skutečně spočítal, shoduje s tím, co jsme od něj dopředu očekávali. K tomuto účelu slouží v knihovně JUnit 5 rodina přetížených metod `assertEquals`. Přestože v kódu používáme stále stejný název metody, na pozadí framework automaticky vybírá správnou variantu podle toho, jaké datové typy do ní předáváme.

Mechanismus fungování `assertEquals` má jedno striktní a neměnné pravidlo, které začínající programátoři často zaměňují: pořadí zadávaných argumentů. Metoda vždy jako svůj první parametr vyžaduje očekávanou hodnotu (`expected`) a jako druhý parametr hodnotu skutečnou (`actual`), kterou vrátila testovaná metoda. Toto pořadí je kriticky důležité pro vývojové prostředí IntelliJ IDEA. Pokud test selže, IDE porovná oba parametry a studentovi vypíše přehledný text ve stylu „Očekáváno: X, ale bylo: Y“. Pokud by student argumenty prohodil, test sice v případě úspěchu projde, ale při chybě bude výpis v IntelliJ IDEA tvrdit přesný opak reality, což zbytečně komplikuje hledání chyby.

Pro běžné testování si student vystačí se dvěma základními variantami této metody:

* První varianta je určena pro celočíselné datové typy (jako je `int` či `long`) a pro primitivní typ `boolean` nebo `char`. Zde funguje `assertEquals` na principu absolutní shody. Framework hodnoty interně porovná pomocí klasického javového operátoru rovnosti `==`. Pokud se hodnoty shodují, test pokračuje dál. Pokud se liší byť o jediný bit, testovací framework běh metody okamžitě přeruší a označí test za selhaný.
* Druhá základní varianta se aktivuje tehdy, když do metody předáme objekty – typickým příkladem je textový řetězec `String` nebo jakákoli vlastní vytvořená třída. U objektů JUnit nemůže použít operátor `==`, protože ten v Javě porovnává paměťové adresy, nikoli obsah. Metoda `assertEquals` proto na pozadí zavolá standardní javovou metodu `equals()`.&#x20;

{% hint style="info" %}
U druhé varianty je  zjevné, že využívá základního principu porovnávání shodnosti objektu založeném na funkci `equals()`, který byl vysvětlen v [todo-object.md](zakladni-datove-typy/todo-object.md "mention").
{% endhint %}

```java
@Test
void testPorovnaniCelychCisel() {
    // Arrange
    int expectedResult = 10;

    // Act
    int actualResult = 5 + 5;

    // Assert - Zde se interně porovnává pomocí operátoru ==
    assertEquals(expectedResult, actualResult);
}

@Test
void testPorovnaniTextovychRetezcu() {
    // Arrange
    String expectedResult = "Java";

    // Act
    String actualResult = "jaVa".substring(0, 2) + "va";

    // Assert - Zde se interně volá metoda equals(), porovnává se znak po znaku
    assertEquals(expectedResult, actualResult);
}
```

#### Prorovnávání desetinných čísel

Při testování metod, které pracují s desetinnými čísly typu `double` nebo `float`, narážíme v Javě na specifický problém, který vychází ze samotné podstaty toho, jak počítače s těmito čísly pracují. Počítače interně ukládají čísla ve dvojkové soustavě. Zatímco celá čísla dokáže systém reprezentovat naprosto přesně, u mnoha desetinných čísel (například u zdánlivě jednoduchého čísla 0.1 nebo 0.3) to není možné a dochází k periodickému opakování bitů. Výsledkem je, že při matematických operacích vznikají miniaturní zaokrouhlovací nepřesnosti.

Pokud v produkčním kódu sečteme `0.1 + 0.2`, lidské oko očekává výsledek přesně `0.3`. Java však kvůli těmto interním mikroskopickým chybám může vrátit hodnotu `0.30000000000000004`.

Pokud bychom se pokusili použít základní metodu `assertEquals(0.3, actualResult)`, test by okamžitě selhal. Pro JUnit framework jsou tato čísla odlišná, přestože z pohledu byznys logiky aplikace je výsledek správný. Z toho důvodu nemůžeme desetinná čísla nikdy porovnávat na absolutní shodu.

Řešením je speciální varianta metody `assertEquals`, která přijímá třetí číselný parametr, v programování označovaný jako delta nebo epsilon. Tento parametr definuje maximální povolenou odchylku (toleranci), o kterou se může očekávaný a skutečný výsledek lišit, aby test stále prošel.

```java
@Test
void testPiValueApproximation() {
    // Arrange
    double expectedResult = Math.PI; // 3.141592653589793...
    double epsilon = 0.001; // Zajímají nás pouze první 3 desetinná místa

    // Act
    double actualResult = 22.0 / 7.0; // Historický zlomkový odhad pí (3.142857...)

    // Assert - Test projde, protože rozdíl (cca 0.00126) se vejde do zaokrouhlení tolerance
    assertEquals(expectedResult, actualResult, epsilon);
}
```

Volba velikosti epsilonu závisí na konkrétní doméně. Pro běžné finanční nebo technické výpočty se nejčastěji volí hodnoty jako `0.001` nebo `0.00001`. Pokud by student na tento třetí parametr u typu `double` zapomněl (neni povinný), moderní verze JUnit 5/6 může upozornit pouze varováním, ale parametr není povinný.

#### Další porovnávací metody Equals

Kromě metody `assertEquals`, která porovnává shodu dvou hodnot, nabízí knihovna JUnit 5 celou řadu dalších asertivních metod. Tyto metody nám umožňují psát testy čitelněji, protože přesně vyjadřují logický záměr daného ověření. Pokud například testujeme, zda je nějaký objekt prázdný, je elegantnější použít metodu k tomu určenou než složitě porovnávat objekt s hodnotou `null` přes `assertEquals`.

Podívejme se na základní trojici pomocných metod, které se v testovací praxi vyskytují hned vedle kontroly rovnosti.

**Kontrola pravdivosti: assertTrue a assertFalse**

Tyto dvě metody používáme v situacích, kdy testovaná metoda vrací logickou hodnotu `boolean` (tedy `true` nebo `false`), nebo když ověřujeme stav nějakého objektu (například zda je seznam prázdný).

* `assertTrue(actual)` – test projde pouze tehdy, pokud je předaná hodnota nebo výraz roven `true`.
* `assertFalse(actual)` – test projde pouze tehdy, pokud je předaná hodnota nebo výraz roven `false`.

Typickým příkladem využití je testování validační logiky nebo vyhledávání:

```java
package cz.skripta.model;

import org.junit.jupiter.api.Test;
import java.util.List;
import java.util.ArrayList;
import static org.junit.jupiter.api.Assertions.*;

class ValidaceTest {

    @Test
    void testStavuSeznamu() {
        // Arrange
        List<String> uzivatele = new ArrayList<>();

        // Act & Assert
        // Ověřujeme, že nově vytvořený seznam je skutečně prázdný (isEmpty() vrací true)
        assertTrue(uzivatele.isEmpty());

        // Přidáme prvek a ověříme opačný stav
        uzivatele.add("Martin");
        assertFalse(uzivatele.isEmpty());
    }
}
```

**Kontrola existence objektů: assertNull a assertNotNull**

V Javě je běžné, že metody v určitých situacích vracejí hodnotu `null` – například když databáze nenajde uživatele podle zadaného ID. Pro tyto scénáře má JUnit připravené specializované metody, které hlídají přítomnost či absenci objektu v paměti.

* `assertNull(actual)` – očekává, že v proměnné není uložena žádná reference (hodnota je `null`). Pokud proměnná na nějaký objekt ukazuje, test selže.
* `assertNotNull(actual)` – přesný opak. Test vyžaduje, aby proměnná obsahovala žijící instanci objektu. Pokud je v ní `null`, test okamžitě končí chybou.

```java
package cz.skripta.model;

import org.junit.jupiter.api.Test;
import java.util.Map;
import java.util.HashMap;
import static org.junit.jupiter.api.Assertions.*;

class VyhledavaniTest {

    @Test
    void testHledaniUzivatele() {
        // Arrange
        Map<String, String> databaze = new HashMap<>();
        databaze.put("admin", "Martin");

        // Act
        String nalezeny = databaze.get("admin");
        String nenalezeny = databaze.get("hacker");

        // Assert
        // Pro existující klíč musíme dostat reálný objekt
        assertNotNull(nalezeny);
        
        // Pro neexistující klíč očekáváme striktně null
        assertNull(nenalezeny);
    }
}
```

#### Vynucené selhání - Fail

V některých situacích potřebujeme test ukončit jako neúspěšný, aniž bychom porovnávali konkrétní hodnoty pomocí standardních asercí. K tomuto účelu slouží metoda `fail`. Jejím jediným úkolem je okamžitě přerušit vykonávání testovací metody, označit test v IntelliJ IDEA červenou barvou a nahlásit selhání.

Metoda `fail` se v praxi používá jako pojistka v místech kódu, kam by se správně napsaný program při testování vůbec neměl dostat. Pokud tam tok programu přesto dorazí, znamená to, že v testované logice je chyba. Tato metoda se nejčastěji předává s textovým parametrem, který popisuje důvod selhání.

Tento přístup se používá například u primitivního testování výjimek uvedené dále.

{% hint style="info" %}
Obecně je předdefinovaných metod velké množství. Pro bližší seznam například viz dokumentace: [https://docs.junit.org/6.1.2/api/org.junit.jupiter.api/org/junit/jupiter/api/Assertions.html](https://docs.junit.org/6.1.2/api/org.junit.jupiter.api/org/junit/jupiter/api/Assertions.html).
{% endhint %}

{% hint style="info" %}
Samotné chování testů se dá ještě v mnohém rozšiřovat a měnit. Cílem zde je popsat jen naprosté základy. Pro další studium například viz dokumentace: [https://docs.junit.org/6.1.2/overview.html](https://docs.junit.org/6.1.2/overview.html).
{% endhint %}

#### Vlastní textové zprávy: Parametr message v asercích

Na závěr přehledu ověřovacích metod je důležité zmínit jeden společný prvek, který najdeme u všech doposud probraných funkcí, jako jsou `assertEquals`, `assertTrue`, `assertNull` či `fail`. Každá z těchto metod má totiž k dispozici ještě jednu schovanou variantu (přetížení), která jako svůj úplně poslední parametr přijímá textový řetězec `String message`.

Tento parametr slouží k zadání vlastního textového vysvětlení, které se zobrazí pouze a jen tehdy, když test selže. Pokud test dopadne dobře a projde, JUnit tento text kompletně ignoruje a nikde ho nevypisuje.

Význam tohoto parametru spočívá v lepší čitelnosti chybových výstupů, zejména ve chvíli, kdy v rámci jedné testovací metody provádíme více ověření za sebou, nebo když testujeme komplexní logiku. Standardní hlášení frameworku typu _„Expected true but was false“_ nám sice řekne, co se stalo technicky, ale neřekne nám, co to znamená logicky v kontextu aplikace.

Podívejme se na praktický rozdíl v zápisu:

```java
package cz.skripta.model;

import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

class EshopTest {

    @Test
    void testOdeslaniObjednavky() {
        Objednavka objednavka = new Objednavka();
        objednavka.pridejPolozku("Kniha o Javě");
        
        // Act
        objednavka.odeslat();

        // Assert s využitím parametru message na konci
        assertTrue(objednavka.jeZaplacena(), "Objednávka by měla být po odeslání automaticky zaplacena.");
        assertNotNull(objednavka.getDorucovaciAdresa(), "Doručovací adresa nesmí být po odeslání prázdná.");
    }
}
```

Pokud by v tomto případě první aserce selhala, IntelliJ IDEA nevypíše pouze strohé hlášení o nesplněné podmínce, ale přímo programátorovi v červeném chybovém panelu zobrazí text: `Visual Failure: Objednávka by měla být po odeslání automaticky zaplacena.`

Psaní těchto zpráv je výborným cvičením. Nutí totiž už při psaní testu přemýšlet nad tím, co přesně daný řádek kódu ověřuje a jaká situace nastane, když kód nebude fungovat správně. V reálné vývojářské praxi navíc tyto zprávy dramaticky zkracují čas strávený debugováním a hledáním příčin chyb v rozsáhlých projektech.

## Realizace v IntelliJ Idea

{% hint style="info" %}
Pozor, že přestože můžete v rámci různých vývojových prostředí (VS Code, Idea, NetBeans, Eclipse, ...) používat stejný framework "JUnit", způsob nastavení, spouštění a vyhodnocení testů (tj. postup operací) se může zásadně měnit.
{% endhint %}

### Složka pro testy

Aby mohl testovací framework JUnit i samotné vývojové prostředí správně fungovat, nemůžeme testovací třídy ukládat na libovolné místo v projektu. **Čistá architektura softwaru vyžaduje striktní oddělení produkčního kódu od kódu testovacího.** V praxi to znamená, že testy nikdy nebalíme do výsledného programu (např. do souboru JAR) určeného pro koncové uživatele. Testy potřebujeme pouze během vývoje a v nástrojích pro kontinuální integraci. Z toho důvodu vzniká v projektu paralelní adresářová struktura.

Většina moderních javových projektů postavených na nástrojích Maven nebo Gradle používá standardizované uspořádání. Zatímco produkční zdrojové kódy se nacházejí v adresáři `src/main/java`, pro testy se zakládá sesterská složka `src/test/java`. Pokud student vytváří projekt od nuly ručně nebo pracuje na specifickém zadání, může se stát, že tuto složku (např. s jednoduchým názvem `test` nebo `tests`) musí vytvořit sám. Samotná existence složky na disku však vývojovému prostředí nestačí. IntelliJ IDEA totiž ke každému adresáři v projektu přistupuje na základě jeho specifické role.

{% hint style="info" %}
Pokud jste projekt zákládali jako klasiký Java projekt v IntelliJ Idea, budete mít vytvouřenou pouze složku na zdrojové kódy - `src`.
{% endhint %}

![Přehled projektu bez složky pro testy](.gitbook/assets/projectview.png)

V takovém případě do projektu musíme vytvořit novou složku (ideálně na úrovni složky `src`), kterou můžeme nazvat libovolně, ale běžně se používají názv jako `tst` , `test` nebo `tests`.

Pokud nově vytvořené složce pro testy neexplicitně nedefinujeme její význam, IntelliJ IDEA ji považuje za běžný adresář s textovými soubory. IDE v této složce nebude správně doplňovat kód Javy, nebude nabízet automatické importy pro knihovnu JUnit a především u testovacích tříd vůbec nezobrazí zelené spouštěcí šipky.

Proto musíme složku označit jako takzvaný _Test Sources Root_. V IntelliJ IDEA toho docílíme kliknutím pravým tlačítkem myši na danou složku v levém panelu projektu, navigováním na položku _Mark Directory as_ a zvolením možnosti _Test Sources Root_. Jakmile tento krok provedeme, ikona složky v prostředí IntelliJ IDEA změní barvu ze standardní žluté na specifickou zelenou.

![Menu "Mark Directory"](.gitbook/assets/test-markdirectory.png)

Tímto vizuálním tahem dáme vývojovému prostředí najevo, že soubory uvnitř této složky jsou plnohodnotné javové třídy, které mají přístup ke všem produkčním třídám v `src/main/java`, ale zároveň jsou izolovány pro účely testování. IntelliJ IDEA okamžitě přepne daný adresář do režimu plné podpory Javy – začne hlídat syntaktické chyby, správnost balíčků, umožní refaktorizaci a hlavně aktivuje spouštěč testů JUnit. Při automatickém generování testů pomocí klávesové zkratky pak IDE bude přesně vědět, kam má nově vznikající testovací třídy ukládat, a automaticky v zelené složce replikuje balíčkovou strukturu z produkční části projektu.

![Test directory created](.gitbook/assets/test-folder.png)

### Vytvoření vlastního testu

Test můžeme vytvořit dvěma způsoby:

* necháme si ho vygenerovat přes prostředí IntelliJ, které samo vytvoří potřebný soubor a případně nageneruje další kód,
* vytvoříme si ho ručně a kód budeme implmentovat sami.

#### Vytvoření testu ručně

Pro vytvoření testu ručně stačí do složky **s testy** vložit novou třídu (jako klasickou třídu do projektu) a do ní doplnit požadovaný kód. V základu se může jednat o úplně jednoduchou třídu.

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

Poslední lehce komplikovanou oblastí pro neznalé je spuštění testu. Testy lze spustit:

* Přímo ze zdrojového kódu pomocí zelených šipek
* Vytvořením tzv. konfigurace a spouštěním konfigurace

#### Spuštěním ze zdrojového kódu

Test lze spustit přímo ze zdrojového kódu pomocí zelených šipek. Šipka u názvu třídy spouští všechny testy v rámci dané třídy, šipka u samotného testu spouští konkrétní test.

<figure><img src=".gitbook/assets/test-start.png" alt=""><figcaption></figcaption></figure>

#### Spuštění přes konfigurace

Když v IntelliJ IDEA klikneme na onu známou zelenou šipku vedle testovací třídy nebo metody, na pozadí se odehraje proces, který vývojové prostředí před uživatelem částečně skrývá. IDE nespouští testy „jen tak“. Pro každý takový klik vytvoří takzvanou spouštěcí konfiguraci (_Run/Debug Configuration_). Spouštěcí konfigurace je v podstatě předpis, recept nebo sada instrukcí, která říká, jakým způsobem, s jakými parametry a v jakém prostředí se má daný kód vykonat.

{% hint style="info" %}
Spouštěcí konfigurace vlastně říkají, co má Idea spustit v okamžiku, kdy uživatel obecně zvolí položku "Run" či "Debug".

U nových projektů se typicky spouští "Current File" — současný soubor — pokud obsahuje metodu `main()` (a tedy lze spustit).
{% endhint %}

Princip spouštěcích konfigurací vychází z toho, že spuštění testu v Javě je ve skutečnosti komplexní záležitost. Aby se test vykonal, musí IntelliJ IDEA na pozadí sestavit složitý příkaz pro příkazovou řádku operačního systému. Tento příkaz musí obsahovat přesnou cestu k Java Development Kitu (JDK), nastavení classpath (seznam adresářů a JAR souborů, kde se nachází produkční kód, testovací kód a knihovna JUnit) a samotné argumenty pro spouštěč JUnit. Ruční psaní takového příkazu by bylo pro programátora extrémně zdlouhavé a náchylné k chybám. Spouštěcí konfigurace tento proces plně automatizuje.

V IntelliJ IDEA najdeme správu těchto konfigurací v pravém horním rohu obrazovky, vedle hlavního tlačítka Play. Jakmile klikneme na šipku u konkrétního testu v editoru, IDE automaticky vytvoří _dočasnou_ spouštěcí konfiguraci. Ta dostane název podle testované třídy nebo metody a v seznamu konfigurací se objeví se zašedlou ikonkou. Dočasných konfigurací může mít IntelliJ IDEA uloženo jen omezené množství a ty nejstarší postupně maže, jakmile se spouští nové testy. Pokud však uživatel ví, že určitý test či celou sadu testů bude spouštět velmi často a chce si u nich upravit specifické parametry, může dočasnou konfiguraci jedním kliknutím uložit jako trvalou.

![Zvolení konfigurace](.gitbook/assets/configurationselection.png)

Trvalá konfigurace (a vůbec správa konfigurací) se provádí přes kontextové menu "Edit Configurations".  Otevře se dialogové okno, kde uživatel může pomocí šipky "+" přidat další konfiguraci, nebo v levém menu zvolit existující konfiguraci a upravit její parametry.

![Run/Debug Configurations window](.gitbook/assets/configurationwindow.png)

Výše uvedený obrázek ukazuje konfiguraci JUnit, která:

* se spouští na lokálním počítači přes javu 21
* má dodatečné parametry `-ea` (pokud znáte v Javě assertions, tak víte, co toto znamená)
* Budet testovat pouze jednu třídu "Class" a to tu uvedenou hned za tímto parametrem - `CarTest`.
* Dalšími parametry lze změnit chování testu.

U testů je běžné, že se nespouští jen jeden konkrétní test nebo jedna třída, ale typicky všechny testy v balíku nebo složce. Proto lze upravit rozbalovací seznam s položkou "Class" a vybrat tam "All in package" nebo "All in directory" a v pravém textovém poli zadat vybranou hodnotu. Potom se bude spouštět celá skupina testů najednou.

Pro uložení konfigurace jako permamentní lze vybrat symbol diskety 💾 napravo od symbolu "+" vlevo nahoře.

Po potvrzení a uložení dialogu pak stačí pouze v rozbalovacím seznamu konfigurací vybrat požadovanou konfiguraci a v IntelliJ Idea vybrat volbu Spustit/Run.

![Ukázka výsledku testu](.gitbook/assets/test-result.png)

## Tvorba vlastních JUnit testů

### Základy testování

asf



Všechny příkazy _assert….()_ mohou mít jako první, nepovinný, textový parametr, který říká, jaká chybová hláška se má vypsat, pokud test selže.

@Test\
public void testOwnMessage() {\
assertEquals("Toto je vlastní zpráva.", 5, 7);\
}

Tento test selže (5 != 7) a vypíše se vlastní chybová zpráva.

### Testování instancí

asfe

### Testování listů, polí a kolekcí

aslefj

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

### Nucené selhání testu

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
