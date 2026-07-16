# TODO Datum a čas

## Původní Java Date API

Když v roce 1995 spatřila světlo světa první verze Javy, obsahovala pro práci s datem a časem pouze třídu `java.util.Date`. Ta se však velmi rychle ukázala jako nekoncepční a plná chyb. Vývojáři se pokusili situaci zachránit v Javě 1.1 představením třídy `java.util.Calendar`, ale ani to nepřineslo kýžené řešení. Výsledkem byl nepřehledný, matoucí a pro moderní programování vysoce rizikový kód. S příchodem Javy 8 v roce 2014 byl proto představen zcela nový balíček `java.time` (tzv. Java Time API), který staré třídy definitivně odsunul do propadliště dějin. Proč byste se měli starému API obloukem vyhnout, si ukážeme na třech zásadních problémech.

**1. Mutabilita a bezpečnost ve vícevláknovém prostředí**

Největším hříchem staré třídy `java.util.Date` je její mutabilita. To znamená, že jakmile objekt vytvoříte, můžete jeho vnitřní stav (hodnotu) kdykoliv změnit. Pokud takový objekt sdílíte mezi více částmi aplikace nebo v něm pracuje více programových vláken najednou, velmi snadno dojde k nechtěnému přepsání dat a těžko dohledatelným chybám.

Podívejte se na následující ukázku, jak snadné je nechtěně změnit datum, které mělo zůstat konstantní:

```java
import java.util.Date;

public class StareApiProblem {
    public static void main(String[] args) {
        // Vytvoříme datum splatnosti
        Date datumSplatnosti = new Date(); // Aktuální čas
        
        // Předáme datum nějaké jiné metodě v aplikaci
        zpracujAUprav(datumSplatnosti);
        
        // Původní datum splatnosti se bez našeho vědomí změnilo!
        System.out.println("Původní datum po úpravě: " + datumSplatnosti);
    }

    private static void zpracujAUprav(Date datum) {
        // Metoda provede nějakou operaci a nechtěně posune čas o 10 dní dál
        long desetDniVMilisekundach = 10L * 24 * 60 * 60 * 1000;
        datum.setTime(datum.getTime() + desetDniVMilisekundach);
    }
}
```

Tento způsob (měnitelné objekty) byl v pořádku na přelomu tisíciletí v době vzniku jazyka Java, dnes je však již překonaný.

Moderní třídy z balíčku `java.time` jsou naopak striktně neměnné (immutable). Jakákoliv operace, jako je přičtení dnů, nevytváří změnu v původním objektu, ale vrací objekt zcela nový. Díky tomu jsou tyto třídy přirozeně bezpečné pro použití ve vícevláknových aplikacích bez nutnosti složité synchronizace.

**2. Nelogický design a matoucí indexování**

Práce se starým API často připomínala detektivní hru. Vývojáři museli neustále myslet na podivná pravidla, která odporují lidské intuici. Například měsíce v třídě `Calendar` byly indexovány od nuly. Leden byl reprezentován číslem `0`, zatímco prosinec číslem `11`. Aby toho nebylo málo, roky se v některých konstruktorech třídy `Date` počítaly jako počet let od roku 1900. Pokud jste chtěli nastavit datum na 16. července 2026, museli jste napsat kód, který na první pohled nedává žádný smysl:

Java

```
import java.util.Date;

// Nastavení data na 16. červenec 2026 pomocí starého API
// Rok: 2026 - 1900 = 126
// Měsíc: červenec je 7. měsíc, tedy index 6
Date stareDatum = new Date(126, 6, 16);
System.out.println("Staré datum: " + stareDatum);
```

Takový kód je extrémně náchylný k chybám. Moderní API tyto nesmyslné historické relikvie odstraňuje. Pokud v novém API vytváříte datum, píšete přesně to, co vidíte v kalendáři:

Java

```
import java.time.LocalDate;

// Moderní, čisté a srozumitelné řešení
LocalDate noveDatum = LocalDate.of(2026, 7, 16);
System.out.println("Nové datum: " + noveDatum);
```

**3. Špatné oddělení zodpovědností**

Třída `java.util.Date` se jmenuje „Date“ (datum), ale ve skutečnosti reprezentuje jak datum, tak přesný čas včetně milisekund. To je matoucí. V reálném světě často potřebujeme pracovat pouze s datem (např. datum narození, kdy nás čas nezajímá) nebo pouze s časem (např. čas otevření obchodu). Staré API vás nutilo používat pro všechno jeden univerzální objekt, což vedlo k plýtvání pamětí a nejasnému významu proměnných. Navíc tento typ vůbec nepracuje s časovými pásmy, což situaci komplikuje při tvorbě moderních mezinárodních aplikací. Moderní Java Time API tento problém řeší elegantně tím, že nabízí specializované třídy pro každou situaci, které si detailně představíme v další části.

## Moderní Java Time API: Základní stavební kameny

Srdcem moderní práce s časem v Javě je balíček `java.time`. Jeho návrh stojí na principech objektově orientovaného programování a řeší všechny bolesti, které jsme si ukázali u starého API. Než se podíváme na jednotlivé třídy, je naprosto klíčové pochopit jeden základní koncept, který se prolíná celým tímto API – **imutabilitu** (neměnnost).

### Immutable princip

Stejně jako u známé třídy `String`, ani objekty reprezentující datum a čas v novém API nelze po jejich vytvoření změnit. Pokud zavoláte metodu, která má s datem či časem manipulovat, původní instance zůstane netknutá a metoda vám vrátí instanci zcela novou s upravenými hodnotami. Pokud na toto pravidlo zapomenete, vaše úpravy se v kódu nijak neprojeví, protože zahodíte nově vytvořený objekt.

{% hint style="info" %}
Stejným způsobem se chová například typ `String`. Jakmile jej jednou nějak vytvoříte, už jej nemůžete změnit. Všimněte si, že všechny metody typu "nahraď znak za jiný" atp. vždy vrací nový `String`, nikdy nemění existující.
{% endhint %}

### Třídy v Java.Time API

Nové API již nepoužívá jednu univerzální třídu pro všechno. Místo toho nám dává do ruky sadu úzce specializovaných nástrojů podle toho, jaká data zrovna potřebujeme reprezentovat. Každá z těchto tříd slouží k trochu jinému účelu a vyjadřuje jinou úroveň detailu.

{% hint style="info" %}
Není důležité znát všechny třídy, jsou uvedeny pouze pro přehled, abyste věděli, co od této knihovny očekávat.
{% endhint %}

Zde je přehledová tabulka všech základních tříd moderního Java Time API:

| **Třída**        | **Co reprezentuje?**                                | **Příklad hodnoty**                     | **Hlavní využití**                                           |
| ---------------- | --------------------------------------------------- | --------------------------------------- | ------------------------------------------------------------ |
| `LocalDate`      | Pouze datum (bez času a časového pásma)             | `2026-07-16`                            | Datum narození, splatnost faktury, státní svátky.            |
| `LocalTime`      | Pouze čas (bez data a časového pásma)               | `14:30:00`                              | Otevírací doba, čas pravidelného budíku.                     |
| `LocalDateTime`  | Datum i čas (bez časového pásma)                    | `2026-07-16T14:30:00`                   | Zápis schůzky v osobním kalendáři bez ohledu na lokaci.      |
| `Instant`        | Jedinečný okamžik na časové ose v UTC               | `2026-07-16T19:35:17Z`                  | Logování událostí, časová razítka v databázi, měření výkonu. |
| `ZonedDateTime`  | Kompletní datum a čas včetně časového pásma         | `2026-07-16T14:30+02:00[Europe/Prague]` | Plánování mezinárodních letů, schůzky napříč kontinenty.     |
| `OffsetDateTime` | Datum a čas s pevným posunem vůči UTC               | `2026-07-16T14:30:00+02:00`             | Komunikace se síťovými protokoly, databázemi a XML/JSON API. |
| `Period`         | Časový úsek založený na datu (roky, měsíce, dny)    | `2 roky, 3 měsíce a 5 dní`              | Výpočet věku člověka, platnosti smlouvy.                     |
| `Duration`       | Časový úsek založený na čase (sekundy, nanosekundy) | `45 minut` (nebo `2700 sekund`)         | Měření doby běhu algoritmu, stopky.                          |

#### Rychlé vysvětlení klíčových rozdílů

* Skupina `Local...` (Místní čas): Tyto třídy představují čas tak, jak ho vidíte na svých nástěnných hodinách nebo v kalendáři na stole. Nemají žádný kontext časového pásma. Když řeknete `LocalTime.of(12, 0)`, je to prostě poledne – ale jestli v Praze nebo v Tokiu, to už objekt neví.
* Strojový vs. lidský čas: Zatímco `Instant` je jedno velké číslo (počet sekund od roku 1970) určené pro počítače a databáze, třídy jako `LocalDate` a `LocalDateTime` jsou navrženy tak, aby se s nimi snadno pracovalo lidem (chápou koncepty jako "měsíc" nebo "den v týdnu").
* Rozdíl mezi `Period` a `Duration`: Obě třídy měří časový úsek, ale každá v jiných jednotkách. `Period` používá kalendářní jednotky (dny, měsíce, roky) a bere v potaz, že např. únor má méně dní než březen. `Duration` měří přesný fyzický čas na úrovni sekund a nanosekund.

Nejdůležitější a nejběžnější třídy budou vysvětleny dále.

### **LocalDate: Práce s čistým datem**

Třída `LocalDate` představuje datum v kalendáři bez jakékoliv zmínky o čase nebo časovém pásmu. Je to ideální volba pro situace, kdy vás zajímá pouze konkrétní den – například datum narození, datum splatnosti faktury nebo státní svátek.

```java
import java.time.LocalDate;

public class UkazkaLocalDate {
    public static void main(String[] args) {
        // Získání aktuálního data v systému
        LocalDate dnes = LocalDate.now();
        System.out.println("Dnes je: " + dnes);

        // Vytvoření konkrétního data (16. červenec 2026)
        LocalDate specifickeDatum = LocalDate.of(2026, 7, 16);
        System.out.println("Specifické datum: " + specifickeDatum);
    }
}
```

**LocalTime: Práce s čistým časem**

Pokud naopak potřebujete reprezentovat čas na hodinách a datum vás vůbec nezajímá, sáhnete po třídě `LocalTime`. Opět platí, že tato třída neobsahuje žádné informace o časovém pásmu. Použijete ji například pro nastavení ranního budíku, otevírací doby obchodu nebo pro určení začátku přednášky.

```java
import java.time.LocalTime;

public class UkazkaLocalTime {
    public static void main(String[] args) {
        // Získání aktuálního systémového času
        LocalTime nyni = LocalTime.now();
        System.out.println("Aktuální čas: " + nyni);

        // Vytvoření konkrétního času (14 hodin, 30 minut)
        LocalTime zacatekPrednasky = LocalTime.of(14, 30);
        System.out.println("Přednáška začíná v: " + zacatekPrednasky);
    }
}
```

**LocalDateTime: Spojení data a času**

Třída `LocalDateTime` v sobě kombinuje vlastnosti obou předchozích tříd. Reprezentuje konkrétní datum a čas, stále však bez vazby na konkrétní časové pásmo. Typickým příkladem použití je zápis schůzky do vašeho osobního kalendáře. Víte, že schůzka se koná 16. července 2026 ve 14:30, ale z pohledu tohoto objektu neřešíte, zda je to v Praze, nebo v Londýně.

```java
import java.time.LocalDateTime;
import java.time.LocalDate;
import java.time.LocalTime;

public class UkazkaLocalDateTime {
    public static void main(String[] args) {
        // Vytvoření spojením konkrétního data a času
        LocalDateTime schuzka = LocalDateTime.of(2026, 7, 16, 14, 30);
        System.out.println("Schůzka je naplánována na: " + schuzka);

        // Můžeme také spojit již existující objekty LocalDate a LocalTime
        LocalDate datum = LocalDate.of(2026, 7, 16);
        LocalTime cas = LocalTime.of(14, 30);
        LocalDateTime spojenaSchuzka = LocalDateTime.of(datum, cas);
    }
}
```

**Instant: Okamžik na časové ose (Strojový čas)**

Na rozdíl od předchozích tříd, které jsou navrženy pro lidské chápání času, třída `Instant` reprezentuje jeden specifický bod na časové ose v UTC (koordinovaném světovém čase). Tento bod se měří jako počet sekund a nanosekund od 1. ledna 1970 (tzv. Unix epoch). `Instant` je ideální pro ukládání do databáze, měření výkonu aplikace nebo logování systémových událostí, protože je naprosto jednoznačný po celém světě.

```java
import java.time.Instant;

public class UkazkaInstant {
    public static void main(String[] args) {
        // Získání aktuálního globálního okamžiku v UTC
        Instant nyniVUtc = Instant.now();
        System.out.println("Aktuální globální okamžik (UTC): " + nyniVUtc);
    }
}
```

###

###

###

###

###

###

###

### Převod kalendáře na řetězec a získání kalendáře z řetězce

Jak bylo ukázáno, funkce _toString()_ implementovaná ve třídě _Calendar_ vypisuje příliš mnoho informací a je ukázkou, jak takové volání _toString()_ nemá vypadat, protože jej programátor nemůže jednoduše použít. Pro získání výpisu smysluplného data pro uživatele lze použít několik přístupů.

Ten nejjednodušší již byl představen v předchozích příkladech. U třídy _Date_ jsme si ukázali, že metoda _toString()_ vrací přehledně zobrazené datum. Toho samozřejmě můžeme využít a vypsat datum jednoduše pomocí získání instance třídy _Date_ z kalendáře (metoda _getTime()_) a následním vypsáním přes metodu _toString()_.

Calendar a = Calendar.getInstance();\
Date d = a.getTime();\
System.out.println(d.toString());\
// volání "toString()" lze vynechat, a pak se provede automaticky\
System.out.println(d);\
// nebo totéž zkráceně, bez nutnosti proměnné d\
System.out.println(a.getTime());

Můžeme však požadovat složitější formátování, závislé například buď na lokalitě, kde se uživatel (cílový počítač) nachází (například v USA jsou jiné formáty data než u nás), nebo přímo programátor může vyžadovat určitý formát.

Pro obecné formátování data slouží **abstraktní** třída _DateFormat_ a její potomek _SimpleDateFormat_[_\[16\]_](https://word2md.com/#footnote-16). Pozor - obě třídy se nacházejí v balíčku _java.text_ - pro použití tedy musíme nahoru do zdrojového kódu třídy, která bude tyto třídy využívat, přidat odpovídající import. Tyto třídy definují základní používané metody:

* _format() -_ která převede datum (instanci třídy _Date_) na řetězec;
* _parse() -_ která převede řetězec na datum (na instanci třídy _Date_).

Třídu _DateFormat_ si zmíníme pouze z toho důvodu, že má u sebe definovány konstanty, které definují obecně délku vypisovaného data - konstanty _FULL, LONG, MEDIUM a SHORT_. Tento typ definujeme při volání metody _getDateInstance()_ jako parametr, jak ukazuje následující příklad.

Calendar c = Calendar.getInstance();\
DateFormat df;\
String dateAsString;\
df = DateFormat.getDateInstance(DateFormat.FULL);\
dateAsString = df.format(c.getTime());\
System.out.println("FULL: " + dateAsString);\
df = DateFormat.getDateInstance(DateFormat.LONG);\
dateAsString = df.format(c.getTime());\
System.out.println("LONG: " + dateAsString);\
df = DateFormat.getDateInstance(DateFormat.MEDIUM);\
dateAsString = df.format(c.getTime());\
System.out.println("MEDIUM: " + dateAsString);\
df = DateFormat.getDateInstance(DateFormat.SHORT);\
dateAsString = df.format(c.getTime());\
System.out.println("SHORT: " + dateAsString);

Pokud chce programátor použít vlastní formát, jak vypsat konkrétní datum, může použít třídu _SimpleDateFormat_. V konstruktoru třídy _SimpleDateFormat_ lze definovat, s jakým formátem data má třída pracovat - formát se ale definuje skládání pomocí specifických znaků uvedených v následující tabulce[\[17\]](https://word2md.com/#footnote-17):

| Symbol | Význam                        | Typ             | Příklad                                                                 |
| ------ | ----------------------------- | --------------- | ----------------------------------------------------------------------- |
| G      | Era                           | Text            | "GG" -> "AD"                                                            |
| y      | Rok                           | Číslo           | "yy" -> "03″ "yyyy" -> "2003″                                           |
| M      | Měsíc                         | Text nebo číslo | "M" -> "7″ "M" -> "12″ "MM" -> "07″ "MMM" -> "Jul" "MMMM" -> "December" |
| d      | Den v měsíci                  | Číslo           | "d" -> "3″ "dd" -> "03″                                                 |
| h      | Hodina (1-12, AM/PM)          | Číslo           | "h" -> "3″ "hh" -> "03″                                                 |
| H      | Hodina (0-23)                 | Číslo           | "H" -> "15″ "HH" -> "15″                                                |
| k      | Hodina (1-24)                 | Číslo           | "k" -> "3″ "kk" -> "03″                                                 |
| K      | Hodina (0-11 AM/PM)           | Číslo           | "K" -> "15″ "KK" -> "15″                                                |
| m      | Minuta                        | Číslo           | "m" -> "7″ "m" -> "15″ "mm" -> "15″                                     |
| s      | Sekunda                       | Číslo           | "s" -> "15″ "ss" -> "15″                                                |
| S      | Milisekunda (0-999)           | Číslo           | "SSS" -> "007″                                                          |
| E      | Den v týdnu                   | Text            | "EEE" -> "Tue" "EEEE" -> "Tuesday"                                      |
| D      | Den v roce (1-365 nebo 1-364) | Číslo           | "D" -> "65″ "DDD" -> "065″                                              |
| F      | Den týdne v měsíci (1-5)      | Číslo           | "F" -> "1″                                                              |
| w      | Týden v roce (1-53)           | Číslo           | "w" -> "7″                                                              |
| W      | Týden v měsíci (1-5)          | Číslo           | "W" -> "3″                                                              |
| a      | AM/PM                         | Text            | "a" -> "AM" "aa" -> "AM"                                                |
| z      | Časová zóna                   | Text            | "z" -> "EST" "zzz" -> "EST" "zzzz" -> "Eastern Standard Time"           |
| '      | Escape znak                   | Oddělovač       | "'Hodina' h" -> "Hodina 9″                                              |
| "      | Apostrof                      | Znak            | "ss"SSS" -> "45′876″                                                    |

Všimněte si, že v tabulce jsou uvedeny měsíce nebo názvy dnů anglicky. V praxi bude záležet, na jakém počítači k vyhodnocení dojde a názvy měsíců/dnů se získají z aktuálního nastavení operačního systému.

Důležité je si uvědomit, že:

* záleží na velikosti písmen - např M a m odlišují měsíce od minut;
* záleží na počtu písmen - čím více písmen, tím „rozvitější" zápis bude; viz například u zápisu měsíce.

Calendar c = Calendar.getInstance();\
// klasická šablona "1999-12-31 12:34:56"\
String formatPattern = "yyyy-MM-dd HH:flag\_mm:ss";\
SimpleDateFormat sdf = new SimpleDateFormat(formatPattern);\
String dateAsString = sdf.format(c.getTime());\
System.out.println(dateAsString);\
// nějaká opravdu složitá šablona\
formatPattern =\
"EEEE, 'datum' d. MMMM yyyy, G, 'čas' hh:flag\_mm:ss, a; z";\
sdf = new SimpleDateFormat(formatPattern);\
dateAsString = sdf.format(c.getTime());\
System.out.println(dateAsString);

Výpis:

run:

2012-03-09 10:45:58

Pátek, datum 9. březen 2012, po Kr., čas 10:45:58, dop.; CET

BUILD SUCCESSFUL (total time: 1 second)

Načtení data z řetězce je typickou úlohou v případech, kdy uživatel zadává datum a program jej potřebuje zpracovat. Načtení je jednoduché. Důležitou úlohou je však korektně uživateli specifikovat, jaký formát data se od něj vyžaduje. V opačném případě uživatel může zadat datum v libovolném, pro něj „normálním" formátu, ale kvůli rozmanitosti různých formátů si s tím program v Javě nebude schopen poradit. **Vždy je tedy důležité uživateli říci, jaký formát data požadujete.**

Pro definovaný formát data stačí vytvořit instanci třídy _SimpleDateFormat_, stejným způsobem a podle stejných pravidel jaká jsou uvedena výše. Pro převedení řetězce na datum pak stačí pouze zavolat metodu _parse()_; této metodě předáte řetězec získaný od uživatele a metoda vrací instanci třídy _Date_. Tuto instanci potom již známou technikou převedete na instanci _Calendar_, je-li třeba.

// příprava proměnných\
Date dateFromUser;\
Calendar calendarFromUser;\
// nastavení formátu data\
String formatPattern = "yyyy-MM-dd HH:mm";\
SimpleDateFormat sdf = new SimpleDateFormat(formatPattern);\
String dataFromUser = null;\
// instanci třídy scanner použijeme k jednoduchému načtení\
// dat od uživatele\
Scanner scanner = new Scanner(System.in);\
// vypíšeme uživateli požadavek\
System.out.println("Zadejte datum ve formátu " + formatPattern +\
", například " + sdf.format(new Date()) + ":");\
// načteme od uživatele řádek textu\
dataFromUser = scanner.nextLine();\
// převedem text na instanci Date\
dateFromUser = sdf.parse(dataFromUser);\
// vytvoříme instanci Calendar a vložíme do něj\
// hodnotu z instance Date\
calendarFromUser = new GregorianCalendar();\
calendarFromUser.setTime(dateFromUser);\
// výpis výsledku\
System.out.println("Datum od uživatele je " +\
sdf.format(calendarFromUser.getTime()));

Výstup tohoto volání (tučný řádek je ten, který uživatel zadal do programu):

run:

Zadejte datum ve formátu yyyy-MM-dd HH:mm, například 2012-03-09 10:59:

**1980-12-08 8:35**

Datum od uživatele je 1980-12-08 08:35

BUILD SUCCESSFUL (total time: 9 seconds)

**Poznámka.** U tohoto kódu zjistíte, že při operaci parse() může snadno dojít k chybě - uživatel datum zadá špatně, ať už úmyslně (uživatelé už jsou takoví) nebo překlepem. V obou případech ale funkce neumí rozpoznat zadané datum a chování by skončilo chybou. Správa chyba a technika zachycení výjimek zatím představena ale nebyla. V současném kódu tedy pouze přidáme za deklaraci funkce, ve které volání probíhá, předání výjimky dále - řetězec throws ParseException[\[18\]](https://word2md.com/#footnote-18).

### Třídy Locale a TimeZone

Poslední dvě třídy pravděpodobně nebudete používat ani jako programátoři běžně, ale přesto se u určitých operací mohou hodit. Třída _Locale_ (balíček _java.util_) je svázána s národním nastavením a umí tedy přizpůsobit chování aplikace oblasti, ve které je aplikace používána. Není nutně svázána s prací s datumem - definuje chování i čísel (oddělovače desetinných míst, oddělovače tisíců), informace o měně (desetinná místa, zkratka měny před/za číslem) a další.

Instanci třídy _Locale_ pro aktuální umístění můžeme získat pomocí volání statické metody nad touto třídou _getDefault()_. Pro odlišná umístění můžeme využít statické konstanty třídy, které obsahují již vytvořené a připravené instance třídy _Locale_.

Instance třídy _Locale_ potom můžeme nastavovat instancím třídy _DateFormat_[_\[19\]_](https://word2md.com/#footnote-19) a měnit tak chování jejich metod _format()_ a _parse()_. Abychom ukázali, že toto chování je obecné, do příkladu jsme zahrnuli i použití _Locale_ vzhledem k formátování čísel - použije se zde zatím nepředstavená třída _NumberFormat_; není však složitá - poskytuje programátorovi komfort při práci s čísly, stejně jako třída _DateFormat_ tento komfort poskytuje pro práci s datem.

// nastavím dvě lokality, default (českou) a americkou (US)\
Locale czLocale = Locale.getDefault();\
Locale usLocale = Locale.US;\
// nejdříve příklad pro datumy\
System.out.println("Example of locale on dates:");\
// získám kalendář a dvě třídy na formátování - každá\
// formátuje podle odlišné "Locale"\
Calendar c = Calendar.getInstance();\
DateFormat czdf =\
DateFormat.getDateInstance(DateFormat.FULL, czLocale);\
DateFormat usdf =\
DateFormat.getDateInstance(DateFormat.FULL, usLocale);\
System.out.println("CZ: " + czdf.format(c.getTime()));\
System.out.println("US: " + usdf.format(c.getTime()));\
// a stejný příklad pro normální čísla\
System.out.println("Example of locale on numbers:");\
// definuji nějaké číslo a dvě odlišná formátování - každá\
// formátuje podle odlišné "Locale"\
Double d = 1234567.8912345;\
NumberFormat cznf = NumberFormat.getInstance(czLocale);\
NumberFormat usnf = NumberFormat.getInstance(usLocale);\
System.out.println("CZ: " + cznf.format(d));\
System.out.println("US: " + usnf.format(d));

Výpis tohoto volání bude:

run:

Example of locale on dates:

CZ: Pátek, 9. březen 2012

US: Friday, March 9, 2012

Example of locale on numbers:

CZ: 1 234 567,89

US:1,234,567.891

BUILD SUCCESSFUL (total time: 0 seconds)

Další zajímavou třídou je třída _TimeZone_ (opět balíček _java.util_). Ta definuje chování různých časových pásem. Využijete ji při převádění data a času na odlišné časové pásmo.

Nejdříve tedy vytvoření instance. Opět - třída _TimeZone_ je abstraktní a pro vytvoření instance musíme využít některou ze statických metod:

* _getInstance()_ - vytvoří instanci podle aktuálního časového pásma počítače;
* _getTimeZone(String zooid)_ - vytvoří instanci časového pásma podle zadaného řetězce. Řetězcem se rozumí určité standardizované řetězce časových pásem[\[20\]](https://word2md.com/#footnote-20) (například „Europe/Prague", nebo „America/Los\_Angeles"), nebo relativní definice vůči času GMT ve formátu (například „GMT-8:00" nebo „GMT+2:30").

**Poznámka.** Pozor! Metoda getTimeZone() v případě, že dostane jako parametr neznámý nebo nerozlišitelný řetězec časové zóny **nevrací chybu**, ale časovou zónu pro pásmo _GMT_. Proto je maximálně vhodné po nastavení nějak zkontrolovat, že získání časové zóny proběhlo korektně.

Získanou instanci časové zóny potom můžeme nastavit (a poté i získat) pomocí get/set metod _setTimeZone()_ (případně _getTimeZone()_) a to jak u třídy _Calendar_, tak u formátovacích tříd _DateFormat_. U kalendáře se budou po změně zóny korektně přepočítávat hodiny/minuty, přestože vnitřní reprezentace pomocí milisekund zůstane nezměněna. U instance třídy _DateFormat_ se opět definuje, jakým způsobem se bude přepočítávat výsledek formátovaný výsledek (nebo načítat „parsovaný" řetězec). Použití ukazuje následující příklad.

Calendar c = Calendar.getInstance();\
DateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:flag\_mm:ss");\
TimeZone czTimeZone = TimeZone.getDefault();\
TimeZone usaTimeZone = TimeZone.getTimeZone("America/New\_York");\
System.out.println("Local: " + czTimeZone.getID());\
System.out.println("USA: " + usaTimeZone.getID());\
c.setTimeZone(czTimeZone);\
df.setTimeZone(czTimeZone);\
System.out.println(c.get(Calendar.HOUR\_OF\_DAY));\
System.out.println(df.format(c.getTime()));\
c.setTimeZone(usaTimeZone);\
df.setTimeZone(czTimeZone);\
System.out.println(c.get(Calendar.HOUR\_OF\_DAY));\
System.out.println(df.format(c.getTime()));\
c.setTimeZone(czTimeZone);\
df.setTimeZone(usaTimeZone);\
System.out.println(c.get(Calendar.HOUR\_OF\_DAY));\
System.out.println(df.format(c.getTime()));\
c.setTimeZone(usaTimeZone);\
df.setTimeZone(usaTimeZone);\
System.out.println(c.get(Calendar.HOUR\_OF\_DAY));\
System.out.println(df.format(c.getTime()));

Příklad definuje dvě časové zóny, aktuální čas _c_ a formátovací objekt _df_. Potom postupně vyzkouší všechny varianty nastavení české a americká mezi oba objekty _c_ a _df_ a vypíše informace. Program vrátí následující výstup:

run:

Local: Europe/Prague

USA: America/New\_York

13

2012-03-09 13:18:01

7

2012-03-09 13:18:01

13

2012-03-09 07:18:01

7

2012-03-09 07:18:01

BUILD SUCCESSFUL (total time: 0 seconds)

Nejdříve se vypíší nastavené časové zóny - zde korektně, první je česká a druhá je americká.

Následuje dvojice hodnot - číslo a datum. Číslo reprezentuje „hodinu" získanou z kalendáře, dlouhý řetězec data reprezentuje naformátované datum přes formátovací objekt.

První varianta zobrazuje kalendář v českém časovém pásmu a české časové pásmo u formátování, výpis je tedy stejný a zobrazuje aktuální čas spuštění programu.

Druhá varianta reprezentuje americké pásmo u kalendáře a české pásmo formátování. Americký kalendář je vůči českému posunutý o několik hodin zpět, metoda _get_() volaná nad instancí kalendáře vrací tedy americký čas; naopak výpis data se zobrazí korektně, ekvivalentně českému času, protože časové pásmo formátování je české.

Třetí varianta reprezentuje naopak české časové pásmo u kalendáře ale anglické časové pásmo u formátování. Kalendář vypisuje korektně český počet hodin (tedy 13), formátování ale kvůli svému časovému pásmu převede výsledek na americký čas.

Poslední výpis reprezentuje americké pásmo nastavené u kalendáře i formátovací jednotky a výpis je tedy v obou případech shodný (kalendář má 7 hodin, v datumu je také 7 hodin).

**Kontrolní otázky:**

* Čím se liší třídy _Date_ a _Calendar_?
* Jak zjistit aktuální čas?

Úkoly k zamyšlení:

Jak složité je vytvořit vlastní třídu, která by reprezentovala datum a čas?

Korespondenční úkol:

Vypracujte program, který po zadání hodnot od uživatele zjistí, kolik dní je na světě.

Shrnutí obsahu kapitoly

Kapitola představila problematiku práce s datumem a časem v jazyce Java. Byly představeny třídy _Date_, _Calendar_, způsob nastavení jejich hodnot a převod mezi těmito hodnotami. Byla také představena problematika vypsání hodnoty datumu a času jako řetězce a převodu zpět.
