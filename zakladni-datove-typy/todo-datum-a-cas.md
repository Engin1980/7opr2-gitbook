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

{% hint style="info" %}
Pro úplný soupis tříd a jejich vysvětlení viz [https://docs.oracle.com/javase/tutorial/datetime/iso/overview.html](https://docs.oracle.com/javase/tutorial/datetime/iso/overview.html).
{% endhint %}

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

### **LocalTime: Práce s čistým časem**

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

### **LocalDateTime: Spojení data a času**

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

### **Instant: Okamžik na časové ose (Strojový čas)**

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

## Manipulace - Fluent API

Jakmile máme objekty pro datum a čas vytvořené, potřebujeme s nimi v aplikacích běžně manipulovat – posouvat je v čase dopředu a dozadu, měnit konkrétní hodnoty nebo je mezi sebou porovnávat. Díky objektovému návrhu a takzvanému _Fluent API_ (plynulému rozhraní) je psaní takového kódu v moderní Javě mimořádně elegantní. Metody na sebe můžete řetězit za sebou a výsledný kód se čte téměř jako běžná anglická věta.

Znovu si však připomeňme nejdůležitější pravidlo: všechny tyto operace vracejí novou instanci. Původní objekt zůstává nezměněn.

{% hint style="info" %}
## Fluent API

Fluent API (plynulé rozhraní) je styl navrhování softwarového rozhraní v objektově orientovaném programování, jehož hlavním cílem je vytvořit kód, který je snadno čitelný, intuitivní a píše se téměř jako běžná lidská věta. Tento přístup staví na principu řetězení metod (method chaining), kdy každá metoda na objektu provede svou práci a na konci vrátí buď upravený objekt (u mutovatelných objektů), nebo zcela novou instanci (u neměnných objektů). Vývojář tak nemusí neustále ukládat mezivýsledky do pomocných proměnných a volat na nich další metody na nových řádcích, ale může celou sekvenci příkazů zapsat do jediného elegantního bloku.

V praxi se s Fluent API setkáte všude tam, kde konfigurujete složité objekty nebo provádíte posloupnost kroků. Představte si například stavbu e-mailové zprávy. Bez Fluent API byste museli vytvořit objekt a na každém řádku volat jeho settery. S Fluent API však metody jako `od()`, `komu()` či `predmet()` vracejí samotný objekt, což vám umožní poskládat celý e-mail v jediném plynulém zápisu:

```java
// Ukázka Fluent API v praxi
Email email = new EmailBuilder()
    .od("jan.novak@email.cz")
    .komu("studenti@univerzita.cz")
    .predmet("Přednáška o Java Time API")
    .obsah("Ahoj všichni, zítra se podíváme na fluent rozhraní...")
    .vytvor();
```
{% endhint %}

### Slovník Fluent API: Jak se v metodách neztratit

Když poprvé otevřete dokumentaci k balíčku `java.time`, může vás překvapit obrovské množství metod, které jednotlivé třídy obsahují. Návrháři tohoto API však odvedli skvělou práci a zavedli přísnou a jednotnou jmennou konvenci. Jakmile pochopíte význam několika málo předpon, budete přesně vědět, co jaká metoda dělá, aniž byste museli nahlížet do dokumentace.

Tyto metody můžeme rozdělit do hlavních skupin: na ty, které nové instance vytvářejí, a na ty, které stávající instance transformují, na dotazovací metody a na konci si zvlášt vysvětlíme transformace z/na řetězec.

{% hint style="info" %}
Pro úplný popis Java Time Fluent API viz [https://docs.oracle.com/javase/tutorial/datetime/overview/naming.html](https://docs.oracle.com/javase/tutorial/datetime/overview/naming.html).
{% endhint %}

### Metody pro vytváření instancí (Tovární metody)

V moderním Java Time API nenajdete veřejné konstruktory. Zapomeňte tedy na klíčové slovo `new`. Místo toho se objekty vytvářejí pomocí takzvaných statických továrních metod (factory methods). Ty nejčastější začínají předponami `now`, `of` a `from`.

**Předpona `now` (Aktuální okamžik)**

Tuto předponu použijete vždy, když chcete získat aktuální stav podle systémových hodin počítače, na kterém kód běží.

```java
LocalDate dnes = LocalDate.now();
LocalTime nyni = LocalTime.now();
```

**Předpona `of` (Skládání z částí)**

Pokud přesně znáte jednotlivé hodnoty (rok, měsíc, den, hodinu...), použijete metodu `of`. Tato metoda má mnoho přetížených variant, takže jí můžete předat buď čísla, nebo specifické výčtové typy (např. třídu `Month`).

```java
// Vytvoření data předáním číselných hodnot (rok, měsíc, den)
LocalDate rande = LocalDate.of(2026, 12, 24);

// Čitelnější varianta s využitím výčtového typu Month
LocalDate randePrehledne = LocalDate.of(2026, Month.DECEMBER, 24);
```

**Předpona `from` (Konverze z jiného typu)**

Někdy už v ruce máte jeden objekt (například kompletní `LocalDateTime`) a potřebujete z něj vytvořit objekt jednodušší (například pouze `LocalDate`). K tomu slouží metoda `from`, která se pokusí z předaného objektu „vytáhnout“ ty informace, které sama potřebuje.

```java
LocalDateTime udalost = LocalDateTime.of(2026, 7, 16, 18, 0);

// Vytáhneme z kompletní události pouze samotné datum
LocalDate datumUdalosti = LocalDate.from(udalost);
```

### Metody pro změnu a posun (Transformační metody)

Jak už víme, tyto metody reálně nic nemění – pouze vezmou stávající data, provedou s nimi matematickou operaci a vrátí novou instanci. Všechny tyto metody voláme přímo na konkrétních objektech (jsou to instanční metody).

#### **Předpony `plus` a `minus` (Aditivní změny)**

Tyto metody slouží k přičítání a odečítání času. Jsou navrženy tak, aby se chovaly intuitivně a automaticky hlídaly kalendářní pravidla (například přestupné roky nebo různou délku měsíců).

Můžete využít specifické metody pro konkrétní jednotky:

```java
LocalDate dnes = LocalDate.of(2026, 7, 16);
LocalDate pristiTyden = dnes.plusWeeks(1);
LocalDate predTremiDny = dnes.minusDays(3);
```

Nebo můžete použít univerzální metody, které jako parametr přijímají množství a jednotku `ChronoUnit`:

```java
LocalDate zaDesetLet = dnes.plus(10, ChronoUnit.YEARS);
```

#### **Předpona `with` (Absolutní nastavení)**

Zatímco `plus` a `minus` dělají relativní změny (posouvají čas o nějakou hodnotu), předpona `with` provádí změnu absolutní. Říkáme jí: „Vezmi tento objekt, ale tuto konkrétní vlastnost nastav přesně na tuto hodnotu.“

```java
LocalDateTime start = LocalDateTime.of(2026, 7, 16, 12, 0);

// Změníme pouze hodinu na 15:00, zbytek zůstává zachován
LocalDateTime posunutyStart = start.withHour(15); 
// Výsledný čas bude 2026-07-16T15:00:00
```

#### **Předpona `at` (Slučování a doplňování)**

Tato předpona slouží k „propojování“ jednodušších objektů do komplexnějších. Představte si, že máte v ruce objekt `LocalDate` (čisté datum) a chcete z něj udělat `LocalDateTime` tím, že k němu přidáte konkrétní čas. K tomu slouží právě `atTime`.

```java
LocalDate datum = LocalDate.of(2026, 7, 16);
LocalTime cas = LocalTime.of(14, 30);

// Propojením vytvoříme LocalDateTime
LocalDateTime kompletniCas = datum.atTime(cas);

// Lze použít i přímé předání hodnot
LocalDateTime kompletniCasZkracene = datum.atTime(14, 30);
```

#### **Předpona `to` (Konverze a zjednodušení)**

Zatímco předpona `at` slouží k doplňování informací a vytváření komplexnějších objektů, předpona `to` funguje přesně opačně. Použijete ji v situacích, kdy chcete stávající objekt převést na jiný typ, často jednodušší, nebo z něj potřebujete získat konkrétní hodnotu v podobě primitivního datového typu (jako je `long`).

Tato předpona v podstatě říká: „Vezmi všechny informace, které máš, a transformuj je do jiného formátu nebo ořež ty, které teď nepotřebuji.“

S metodami začínajícími na `to` se nejčastěji setkáte ve dvou situacích:

**1. Převod komplexních objektů na jednodušší**

Pokud máte v aplikaci objekt, který nese hodně informací (například datum, čas i časové pásmo), a v určité metodě potřebujete pracovat pouze s jednou jeho částí, metody s předponou `to` vám umožní tyto informace snadno oddělit.

```java
import java.time.LocalDateTime;
import java.time.LocalDate;
import java.time.LocalTime;

public class UkazkaToKonverze {
    public static void main(String[] args) {
        LocalDateTime schuzka = LocalDateTime.of(2026, 7, 16, 14, 30);

        // Získání čistého data (zahodíme informaci o čase)
        LocalDate datumSchuzky = schuzka.toLocalDate();
        System.out.println("Datum schůzky: " + datumSchuzky); // 2026-07-16

        // Získání čistého času (zahodíme informaci o datu)
        LocalTime casSchuzky = schuzka.toLocalTime();
        System.out.println("Čas schůzky: " + casSchuzky); // 14:30
    }
}
```

**2. Převod časových úseků na konkrétní jednotky**

Třída `Duration` (která reprezentuje časový úsek v sekundách a nanosekundách) nabízí celou řadu metod s předponou `to`. Tyto metody slouží k tomu, aby se celý časový úsek přepočítal a vyjádřil v jedné konkrétní jednotce, jako jsou minuty, milisekundy nebo dny. Výsledkem pak není další objekt, ale běžné číslo typu `long`.

```java
import java.time.Duration;
import java.time.LocalTime;

public class UkazkaToJednotky {
    public static void main(String[] args) {
        LocalTime zacatek = LocalTime.of(9, 0);
        LocalTime konec = LocalTime.of(10, 15);
        Duration trvani = Duration.between(zacatek, konec);

        // Chceme zjistit, kolik je to celkem minut (vrátí long)
        long celkemMinut = trvani.toMinutes();
        System.out.println("Přednáška trvá celkem: " + celkemMinut + " minut"); // 75 minut

        // Převod na milisekundy pro potřeby starších knihoven
        long celkemMilisekund = trvani.toMillis();
        System.out.println("V milisekundách: " + celkemMilisekund); // 4500000
    }
}
```

### Získávání hodnot z objektů

#### **Předpona `get` (Čtení vnitřních hodnot)**

Pokud potřebujete z hotového objektu vytáhnout jednu konkrétní informaci – například zjistit, jaký je zrovna měsíc, nebo kolikátá minuta v hodině zrovna běží – použijete metody začínající na `get`. Tyto metody fungují jako klasické „gettery“, které znáte z objektově orientovaného programování.

Většina tříd nabízí jak specifické a velmi pohodlné metody, které vracejí přímočaře srozumitelné hodnoty (čísla nebo výčtové typy), tak i jednu univerzální metodu `get(TemporalField)`, do které předáváte konstantu z výčtu `ChronoField`.

```java
import java.time.LocalDate;
import java.time.Month;
import java.time.temporal.ChronoField;

public class UkazkaGet {
    public static void main(String[] args) {
        LocalDate datum = LocalDate.of(2026, 7, 16);

        // Čtení roku a dne v měsíci jako standardního celého čísla (int)
        int rok = datum.getYear(); // 2026
        int denVMesici = datum.getDayOfMonth(); // 16

        // Čtení měsíce jako výčtového typu Month a následně jako čísla
        Month mesicObjekt = datum.getMonth(); // JULY
        int mesicCislo = datum.getMonthValue(); // 7

        // Zjištění dne v týdnu (opět vrací praktický výčtový typ)
        System.out.println("Den v týdnu: " + datum.getDayOfWeek()); // THURSDAY

        // Univerzální přístup přes ChronoField (např. den v roce)
        int denVRoce = datum.get(ChronoField.DAY_OF_YEAR);
        System.out.println("Číslo dne v roce: " + denVRoce); // 197
    }
}
```

#### **Předpona `is` (Dotazování na stav a porovnání)**

Metody začínající předponou `is` slouží k ověřování různých vlastností a stavů. Vždy vracejí pravdivostní hodnotu typu `boolean` (tedy `true` nebo `false`). Můžete se jich ptát na vzájemný vztah dvou objektů na časové ose, na specifické vlastnosti kalendáře, nebo na to, zda daná třída vůbec podporuje operaci, kterou se na ní chystáte provést.

```java
import java.time.LocalDate;
import java.time.temporal.ChronoUnit;

public class UkazkaIs {
    public static void main(String[] args) {
        LocalDate dnes = LocalDate.of(2026, 7, 16);
        LocalDate terminSplatnosti = LocalDate.of(2026, 7, 20);

        // 1. Porovnávání polohy na časové ose
        boolean jePredTerminem = dnes.isBefore(terminSplatnosti); // true
        boolean jePoTerminu = dnes.isAfter(terminSplatnosti); // false

        // 2. Dotaz na specifické vlastnosti kalendáře
        boolean jePrestupnyRok = dnes.isLeapYear(); // false (rok 2026 není přestupný)

        // 3. Ověření podpory časových jednotek (obrana před výjimkou)
        // LocalDate reprezentuje pouze datum. Pokud bychom se pokusili přičíst hodiny,
        // aplikace by spadla s chybou. Pomocí 'isSupported' tomu můžeme předejít.
        boolean podporujeHodiny = dnes.isSupported(ChronoUnit.HOURS); // false
        boolean podporujeDny = dnes.isSupported(ChronoUnit.DAYS); // true
    }
}
```

### **Porovnávání dat a časů**

Při práci se starým API vývojáři často dělali chybu, že objekty porovnávali pomocí klasických operátorů `>` nebo `<`. V objektově orientovaném jazyce však objekty tímto způsobem porovnávat nelze. Moderní API nabízí pro srovnání jasně pojmenované metody `.isBefore()`, `.isAfter()` a `.isEqual()`. Tyto metody vrací pravdivostní hodnotu `boolean` a dělají kód okamžitě čitelným.

```java
import java.time.LocalDate;

public class PorovnavaniDat {
    public static void main(String[] args) {
        LocalDate terminSplatnosti = LocalDate.of(2026, 7, 20);
        LocalDate dnes = LocalDate.of(2026, 7, 16);

        // Je dnešní datum před termínem splatnosti?
        if (dnes.isBefore(terminSplatnosti)) {
            System.out.println("Faktura je stále v termínu splatnosti.");
        }

        // Je dnešní datum po termínu splatnosti?
        if (dnes.isAfter(terminSplatnosti)) {
            System.out.println("Faktura je po splatnosti!");
        }
        
        // Jsou si data rovna?
        if (dnes.isEqual(terminSplatnosti)) {
            System.out.println("Dnes je poslední den splatnosti!");
        }
    }
}
```

## Formátování a parsování (Vstup a výstup)

Aby mohl uživatel s daty pracovat, je třeba je často převést na text (tento proces se nazývá formátování). A naopak, když uživatel do formuláře nebo textového pole zadá datum jako text, musíte ho převést zpět na objekt, se kterým umí Java pracovat (tento proces se nazývá parsování).

{% hint style="info" %}
Pojmy "formátování" a "parsování" se používají obecně jako techniky převodu "na text" a "z textu" ve všech programovacích jazycích.
{% endhint %}

Jednou z největších předností je, že metody pro formátování a parsování jsou integrovány přímo do samotných objektů data a času jako součást jejich Fluent API.

### DateTimeFormatter

Třída `DateTimeFormatter` je hlavním mozkem pro překlad mezi světem objektů a světem textu. Řiká, jak vypadá formát na/z kterého se datum a čas převádí - například zda to je `8. 12. 1980` nebo `1980-12-08` či dokonce `80/08/12` - a obdobně pro čas.&#x20;

Způsob, jakým formátovač získáte a jak s ním pracujete, se odvíjí od toho, jak moc specifické požadavky na výstup máte. Máte k dispozici tři hlavní přístupy.

#### 1. Předpřipravené konstanty (Standardní formáty)

Pro běžné systémové výměny dat (např. v JSON API) nemusíte vymýšlet žádné vlastní vzory. Třída `DateTimeFormatter` v sobě obsahuje řadu statických konstant, které reprezentují celosvětově uznávané standardy (převážně normu ISO-8601).

* `DateTimeFormatter.ISO_LOCAL_DATE` – reprezentuje datum jako `yyyy-MM-dd` (např. `2026-07-17`).
* `DateTimeFormatter.ISO_LOCAL_TIME` – reprezentuje čas jako `HH:mm:ss` (např. `14:30:00`).
* `DateTimeFormatter.ISO_DATE_TIME` – reprezentuje kompletní datum i čas spojené znakem `T` (např. `2026-07-17T14:30:00`).

Tyto konstanty jsou extrémně rychlé a hlavně standardizované napříč jazyky a zaručují, že vaše aplikace bude bez problému komunikovat s jakýmkoliv jiným systémem na světě. Používají se tedy nejčastěji při přenosu informace o datumu a času napříč mezi různými systémy.

#### 2. Vlastní šablony pomocí `ofPattern`

Pokud potřebujete datum zobrazit běžným uživatelům, standard ISO-8601 pro ně nebude příliš přívětivý. Tehdy přichází na řadu metoda `DateTimeFormatter.ofPattern(String pattern)`, do které předepíšete přesnou šablonu pomocí specifických symbolů.

Zde jsou ty nejdůležitější symboly, které se používají:

<table data-header-hidden><thead><tr><th width="110"></th><th width="203"></th><th></th></tr></thead><tbody><tr><td><strong>Symbol</strong></td><td><strong>Význam</strong></td><td><strong>Příklad (pro 17. červenec 2026, 14:05)</strong></td></tr><tr><td><code>y</code> / <code>yyyy</code></td><td>Rok (Year)</td><td><code>26</code> (pro <code>yy</code>), <code>2026</code> (pro <code>yyyy</code>)</td></tr><tr><td><code>M</code></td><td>Měsíc v roce (Month)</td><td><code>7</code> (pro <code>M</code>), <code>07</code> (pro <code>MM</code>), <code>Čec</code> (pro <code>MMM</code>), <code>Červenec</code> (pro <code>MMMM</code>)</td></tr><tr><td><code>d</code></td><td>Den v měsíci (Day)</td><td><code>17</code> (pro <code>d</code> i <code>dd</code>)</td></tr><tr><td><code>E</code></td><td>Den v týdnu (Day of week)</td><td><code>Pá</code> (pro <code>E</code> / <code>EEE</code>), <code>Pátek</code> (pro <code>EEEE</code>)</td></tr><tr><td><code>H</code></td><td>Hodina v 24h formátu (Hour 0-23)</td><td><code>14</code> (pro <code>H</code> i <code>HH</code>)</td></tr><tr><td><code>h</code></td><td>Hodina v 12h formátu (Hour 1-12)</td><td><code>2</code> (pro <code>h</code>), <code>02</code> (pro <code>hh</code>) – často se doplňuje o symbol <code>a</code> (AM/PM)</td></tr><tr><td><code>m</code></td><td>Minuta v hodině (Minute)</td><td><code>5</code> (pro <code>m</code>), <code>05</code> (pro <code>mm</code>)</td></tr><tr><td><code>s</code></td><td>Sekunda v minutě (Second)</td><td><code>00</code> (pro <code>ss</code>)</td></tr></tbody></table>

{% hint style="info" %}
Všimněte si, že symboly jsou citlivé na malá a velká písmena - s tím se potkáte zejména při použití "měsíc" vs "minuta".
{% endhint %}

Seskládáním těchto symbolů za sebe definujete požadovaný formát data.

**Jak se liší počet opakování symbolu?**

Počet stejných písmen za sebou určuje styl výpisu. Dobře je to vidět na měsíci (`M`):

* `M` -> `7` (nejkratší číselný zápis bez úvodní nuly)
* `MM` -> `07` (vždy dvouciferné číslo s nulou na začátku)
* `MMM` -> `čvc` nebo `Jul` (zkratka názvu měsíce podle nastavené lokalizace)
* `MMMM` -> `červenec` nebo `July` (plný název měsíce)

#### 3. Lokalizace výstupu (Přizpůsobení jazyku a zvyklostem)

Pokud použijete textové symboly (například `EEEE` pro den v týdnu), Java musí vědět, v jakém jazyce má tento den vypsat. Ve výchozím nastavení použije jazyk operačního systému, na kterém běží. Chcete-li mít chování aplikace stoprocentně pod kontrolou, můžete formátovači vnutit konkrétní lokalizaci (tzv. `Locale`).

K tomu slouží plynulá metoda `.withLocale(Locale locale)`.

```java
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.Locale;

public class UkazkaLokalizace {
    public static void main(String[] args) {
        LocalDateTime nyni = LocalDateTime.of(2026, 7, 17, 14, 30);
        
        // Vytvoříme šablonu, která vypíše den v týdnu, den v měsíci a název měsíce
        DateTimeFormatter sablona = DateTimeFormatter.ofPattern("EEEE, d. MMMM yyyy");

        // 1. Formátování v češtině
        DateTimeFormatter ceskyFormatter = sablona.withLocale(new Locale("cs", "CZ"));
        System.out.println(nyni.format(ceskyFormatter)); 
        // Výstup: pátek, 17. červenec 2026

        // 2. Formátování v angličtině (USA)
        DateTimeFormatter anglickyFormatter = sablona.withLocale(Locale.US);
        System.out.println(nyni.format(anglickyFormatter)); 
        // Výstup: Friday, 17. July 2026
    }
}
```

Díky této vlastnosti je `DateTimeFormatter` extrémně silný nástroj pro tvorbu vícejazyčných aplikací. Můžete mít v kódu jednu šablonu, kterou pouze za běhu lokalizujete podle toho, ze které země se uživatel k vaší aplikaci připojuje.

Následuje několik příkladů. Pro názornost jsou v tabulce uvedeny jak české výstupy (s lokalizací `cs_CZ`), tak anglické (`en_US`), protože u textových polí (dny, měsíce) hraje lokalizace zásadní roli.

| **Šablona (Pattern)** | **Výstup (Česky)**        | **Výstup (Anglicky)**                    |
| --------------------- | ------------------------- | ---------------------------------------- |
| `dd.MM.yyyy`          | `17.07.2026`              | `17/07/2026` _(může se lišit oddělovač)_ |
| `d. M. yyyy`          | `17. 7. 2026`             | `17. 7. 2026`                            |
| `yyyy-MM-dd HH:mm:ss` | `2026-07-17 14:05:09`     | `2026-07-17 14:05:09`                    |
| `dd.MM.yy h:mm a`     | `17.07.26 2:05 odpoledne` | `17/07/26 2:05 PM`                       |
| `EEEE d. MMMM yyyy`   | `pátek 17. červenec 2026` | `Friday 17. July 2026`                   |
| `E, d. MMM yy`        | `pá, 17. čvc 26`          | `Fri, 17. Jul 26`                        |
| `HH'h' mm'm' ss's'`   | `14h 05m 09s`             | `14h 05m 09s`                            |

### Formátování: Převod objektu na text

Chcete-li objekt (např. `LocalDate` nebo `LocalDateTime`) převést na text, použijete jeho instanční metodu `.format()`. Této metodě předáte instanci `DateTimeFormatter`, která definuje, jak má výsledný text vypadat.

Můžete využít buď předpřipravené standardní formátovače, nebo si vytvořit svůj vlastní vzor (pattern) pomocí statické metody `DateTimeFormatter.ofPattern()`.

```java
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

public class UkazkaFormatovani {
    public static void main(String[] args) {
        LocalDateTime nyni = LocalDateTime.of(2026, 7, 16, 14, 30);

        // 1. Použití předpřipraveného standardního formátu (ISO-8601)
        String isoDatum = nyni.format(DateTimeFormatter.ISO_LOCAL_DATE);
        System.out.println("ISO formát: " + isoDatum); // 2026-07-16

        // 2. Vytvoření vlastního lidsky čitelného formátu
        // dd = den (01-31), MM = měsíc (01-12), yyyy = rok (čtyřciferný)
        // HH = hodina v 24h formátu (00-23), mm = minuta (00-59)
        DateTimeFormatter ceskyFormat = DateTimeFormatter.ofPattern("dd.MM.yyyy HH:mm");
        
        String naformatovanyCas = nyni.format(ceskyFormat);
        System.out.println("Český formát: " + naformatovanyCas); // 16.07.2026 14:30
    }
}
```

### Parsování: Převod textu na objekt

Pokud dostanete od uživatele nebo z externího systému (např. z textového souboru) datum jako textový řetězec, musíte z něj vytvořit objekt. K tomu slouží statická metoda `.parse()`, kterou najdete na všech hlavních třídách (`LocalDate`, `LocalTime`, `LocalDateTime`).

Metoda `.parse()` má dvě varianty:

* Jednoparametrová: `LocalDate.parse("2026-07-16")` – očekává text v přísném standardním ISO formátu. Pokud mu text neodpovídá, parsování selže.
* Dvouparametrová: `LocalDate.parse("16.07.2026", formatter)` – použije váš vlastní nadefinovaný vzor k tomu, aby text správně rozklíčovala.

```java
import java.time.LocalDate;
import java.time.format.DateTimeFormatter;
import java.time.format.DateTimeParseException;

public class UkazkaParsovani {
    public static void main(String[] args) {
        // 1. Parsování standardního ISO formátu (nevyžaduje formatter)
        LocalDate isoDatum = LocalDate.parse("2026-07-16");
        System.out.println("Napárované ISO datum: " + isoDatum);

        // 2. Parsování vlastního formátu (vyžaduje formatter)
        String vstupOdUzivatele = "16.07.2026";
        DateTimeFormatter ceskyFormat = DateTimeFormatter.ofPattern("dd.MM.yyyy");

        try {
            LocalDate ceskeDatum = LocalDate.parse(vstupOdUzivatele, ceskyFormat);
            System.out.println("Úspěšně napárováno: " + ceskeDatum);
            
        } catch (DateTimeParseException e) {
            // Vždy musíme počítat s tím, že uživatel zadá neplatný formát (např. "šestnáctého července")
            // V takovém případě Java vyhodí runtime výjimku DateTimeParseException
            System.err.println("Chyba: Zadaný text neodpovídá očekávanému formátu dd.MM.yyyy!");
        }
    }
}
```

## Intervaly: Period vs. Duration

Při práci s datem a časem velmi často narážíme na potřebu změřit vzdálenost mezi dvěma okamžiky. Chceme například vědět, kolik dní zbývá do konce platnosti smlouvy, kolik let a měsíců je našemu uživateli, nebo jak dlouho trval výpočet složitého algoritmu.

Java Time API pro tyto účely nabízí dvě specializované třídy: `Period` a `Duration`. Ačkoliv obě reprezentují časový úsek (interval), každá z nich se na čas dívá úplně jinou optikou. Pochopení rozdílu mezi nimi je klíčové pro správné fungování vašich výpočtů.

### Period: Kalendářní čas (pro lidi)

Třída `Period` měří časový úsek v jednotkách, které jsou přirozené pro lidi: v letech, měsících a dnech. Pracuje na úrovni kalendáře, což znamená, že počítá s tím, že měsíce mají různý počet dní a roky mohou být přestupné.

Tuto třídu používáme výhradně ve spojení s `LocalDate`. Pokud se pokusíte vypočítat `Period` mezi dvěma časy (např. `LocalTime`), kód nebude fungovat, protože na hodinách žádné dny ani měsíce neodměříte.

Pro zjištění rozdílu použijeme statickou metodu `Period.between()`. Získaný objekt pak nabízí metody jako `.getYears()`, `.getMonths()` a `.getDays()`, kterými se dotážeme na jednotlivé složky tohoto rozdílu.

```java
import java.time.LocalDate;
import java.time.Period;

public class UkazkaPeriod {
    public static void main(String[] args) {
        LocalDate datumNarozeni = LocalDate.of(1995, 5, 20);
        LocalDate dnes = LocalDate.of(2026, 7, 17);

        // Vypočítáme rozdíl mezi datem narození a dneškem
        Period vek = Period.between(datumNarozeni, dnes);

        // Získáme jednotlivé složky věku
        int let = vek.getYears();
        int mesicu = vek.getMonths();
        int dni = vek.getDays();

        System.out.println("Věk studenta: " + let + " let, " + mesicu + " měsíců a " + dni + " dní.");
        // Výstup: Věk studenta: 31 let, 1 měsíců a 27 dní.
    }
}
```

### Duration: Fyzický čas (pro stroje)

Třída `Duration` naopak reprezentuje přesný fyzický čas nezávislý na kalendáři. Měří časový úsek na úrovni sekund a nanosekund (případně hodin a minut). Nezajímá ji, jaký je zrovna měsíc nebo den v týdnu – měří čas čistě jako nepřetržitý proud fyzických jednotek.

Tuto třídu používáme ve spojení s `LocalTime`, `LocalDateTime` nebo s globálními okamžiky `Instant`. Je ideálním nástrojem například pro stopky, měření doby trvání operací nebo odpočítávání minut do konce události.

Pro výpočet rozdílu použijeme metodu `Duration.between()`. Pro získání výsledku v konkrétních jednotkách pak využíváme nám již známé metody s předponou `to` (např. `.toMinutes()` nebo `.toSeconds()`).

```java
import java.time.Instant;
import java.time.Duration;

public class UkazkaDuration {
    public static void main(String[] args) throws InterruptedException {
        // Spustíme stopky (zaznamenáme počáteční globální okamžik)
        Instant start = Instant.now();

        // Simulujeme vykonání složité operace (program na chvíli uspíme)
        Thread.sleep(1500); 

        // Zastavíme stopky (koncový okamžik)
        Instant konec = Instant.now();

        // Vypočítáme dobu trvání
        Duration trvani = Duration.between(start, konec);

        // Vyjádříme trvání v sekundách a milisekundách
        long sekundy = trvani.toSeconds();
        long milisekundy = trvani.toMillis();

        System.out.println("Metoda běžela: " + sekundy + " s (" + milisekundy + " ms)");
        // Výstup: Metoda běžela: 1 s (1500 ms)
    }
}
```

#### Shrnutí rozdílu v jedné větě

Zatímco `Period` vám odpoví na otázku: „_Kolik listů v kalendáři musím otočit mezi těmito dvěma dny?_“, `Duration` odpoví na otázku: „_Kolikrát musely tiknout atomové hodiny mezi těmito dvěma okamžiky?_“

## Lokalizace a mezinárodní nastavení: Třída Locale

Zatímco časová pásma řeší, _kolik hodin_ v daném místě na světě zrovna je, lokalizace řeší, _jakým způsobem_ tyto informace uživateli zobrazit. Vstupuje sem jazyk, národní zvyklosti, formátování čísel a specifické kalendářní zápisy. Pro tyto účely Java využívá třídu `java.util.Locale`.

Bez správného nastavení `Locale` bude vaše aplikace vypisovat dny v týdnu a měsíce v jazyce, který odpovídá nastavení serveru, na kterém běží. To může vést k tomu, že se českému uživateli zobrazí anglické názvy jako "Friday" místo "Pátek".

{% hint style="info" %}
Běžně se s tím můžete setkat, když aplikaci vyvíjíte u sebe lokálně a pak ji nasadíte například na americký server. Rázem budou všechna data vypadat jinak.
{% endhint %}

### Co je to Locale?

Objekt `Locale` v sobě nese informaci o konkrétním jazyce, zeměpisné oblasti a volitelně i o dalších specifických variantách. Standardně se skládá ze dvou hlavních částí:

1. Jazykový kód (podle normy ISO-639, např. `cs` pro češtinu, `en` pro angličtinu).
2. Kód země (podle normy ISO-3166, např. `CZ` pro Českou republiku, `US` pro Spojené státy).

Tyto dvě části se spojují do jednoho identifikátoru (např. `cs_CZ` nebo `en_US`). Rozlišování země je důležité, protože například angličtina v USA (`en_US`) a angličtina ve Velké Británii (`en_GB`) mají odlišné zvyklosti pro zápis data a času.

{% hint style="info" %}
Někdy se jako delimiter používá pomlčka, například `cs-CZ` či `en-US`.
{% endhint %}

#### Jak získat a vytvořit Locale

V Javě můžete `Locale` získat buď ze systému, použít předpřipravené konstanty, nebo si vytvořit vlastní pomocí konstruktoru či moderního builderu.

```java
import java.util.Locale;

public class UkazkaLocale {
    public static void main(String[] args) {
        // 1. Získání výchozího nastavení systému uživatele
        Locale systemoveLocale = Locale.getDefault();
        System.out.println("Výchozí jazyk systému: " + systemoveLocale.getDisplayName());

        // 2. Použití předpřipravených konstant (často obsahují pouze jazyk)
        Locale anglictina = Locale.ENGLISH;
        Locale usa = Locale.US; // Jazyk angličtina, země USA

        // 3. Ruční vytvoření pro specifickou zemi (např. čeština v ČR)
        Locale cesko = new Locale("cs", "CZ");
        
        // Modernější způsob vytváření pomocí Builderu (od Javy 7)
        Locale slovensko = new Locale.Builder()
                .setLanguage("sk")
                .setRegion("SK")
                .build();
    }
}
```

### Použití Locale v Java Time API

V moderním Java Time API se `Locale` nepředává přímo metodám jako `.format()` na objektech data a času. Místo toho se jím konfiguruje samotný `DateTimeFormatter` pomocí plynulé metody `.withLocale()`.

Díky tomu oddělujeme samotnou šablonu zápisu (např. "chceme plné názvy") od jazyka, ve kterém se tyto názvy nakonec vygenerují.

```java
import java.time.ZonedDateTime;
import java.time.format.DateTimeFormatter;
import java.util.Locale;

public class LokalizovaneFormatovani {
    public static void main(String[] args) {
        ZonedDateTime nyni = ZonedDateTime.now();

        // Definujeme obecnou šablonu (den v týdnu, den v měsíci, měsíc, rok a časové pásmo)
        DateTimeFormatter sablona = DateTimeFormatter.ofPattern("EEEE, d. MMMM yyyy (z)");

        // Formátování pro českého uživatele
        DateTimeFormatter ceskyFormat = sablona.withLocale(new Locale("cs", "CZ"));
        System.out.println("Česky:    " + nyni.format(ceskyFormat));
        // Výstup např.: pátek, 17. červenec 2026 (Středoevropský letní čas)

        // Formátování pro amerického uživatele
        DateTimeFormatter americkyFormat = sablona.withLocale(Locale.US);
        System.out.println("Anglicky: " + nyni.format(americkyFormat));
        // Výstup např.: Friday, 17. July 2026 (Central European Summer Time)

        // Formátování pro německého uživatele
        DateTimeFormatter nemeckyFormat = sablona.withLocale(Locale.GERMANY);
        System.out.println("Německy:  " + nyni.format(nemeckyFormat));
        // Výstup např.: Freitag, 17. Juli 2026 (Mitteleuropäische Sommerzeit)
    }
}
```

## Bonus 1: Most do minulosti: Převod na java.util.Date a zpět

I když při psaní nového kódu budete striktně používat moderní třídy z `java.time`, v praxi se nevyhnete práci se staršími knihovnami nebo starším databázovým kódem (legacy kód). V těchto situacích budete muset moderní objekty převést na starý `java.util.Date` (případně `java.sql.Date`) a naopak.

Návrháři Javy na to pamatovali a do starých tříd přidali nové metody, které slouží jako most mezi oběma světy. Klíčem k tomuto mostu je třída `Instant`, která reprezentuje absolutní bod v čase a je společným jmenovatelem pro starý i nový přístup.

### Jak převést starý `Date` na moderní `Instant` a `LocalDateTime`

Stará třída `java.util.Date` dostala novou metodu `.toInstant()`. Jakmile máte `Instant`, můžete ho snadno převést na jakýkoliv moderní typ, stačí mu dodat informaci o časovém pásmu (např. výchozím pásmu systému `ZoneId.systemDefault()`).

```java
import java.util.Date;
import java.time.Instant;
import java.time.LocalDateTime;
import java.time.ZoneId;

public class PrevodNaNove {
    public static void main(String[] args) {
        // Máme starý objekt Date (např. z nějaké starší knihovny)
        Date stareDatum = new Date();

        // 1. Krok: Převod starého Date na moderní Instant
        Instant instant = stareDatum.toInstant();

        // 2. Krok: Převod na LocalDateTime s využitím systémového časového pásma
        LocalDateTime noveDatum = LocalDateTime.ofInstant(instant, ZoneId.systemDefault());

        System.out.println("Původní Date: " + stareDatum);
        System.out.println("Nový LocalDateTime: " + noveDatum);
    }
}
```

### Jak převést moderní `Instant` nebo `ZonedDateTime` na starý `Date`

Pokud naopak vaše aplikace interně pracuje s moderním API, ale externí knihovna po vás vyžaduje starý `java.util.Date`, použijete statickou metodu `Date.from(Instant instant)`.

Protože třídy jako `LocalDate` nebo `LocalDateTime` v sobě nemají informaci o časovém pásmu, musíte je nejprve převést na `ZonedDateTime` (přiřazením pásma) a následně na `Instant`.

```java
import java.time.LocalDateTime;
import java.time.ZoneId;
import java.time.Instant;
import java.util.Date;

public class PrevodNaStare {
    public static void main(String[] args) {
        // Máme moderní LocalDateTime
        LocalDateTime nyni = LocalDateTime.now();

        // 1. Krok: Musíme určit časové pásmo, abychom získali jednoznačný okamžik
        Instant instant = nyni.atZone(ZoneId.systemDefault()).toInstant();

        // 2. Krok: Vytvoření starého Date pomocí statické metody Date.from()
        Date stareDatum = Date.from(instant);

        System.out.println("Moderní LocalDateTime: " + nyni);
        System.out.println("Převráceno na staré Date: " + stareDatum);
    }
}
```

## Bonus 2: Časová pásma

{% hint style="info" %}
Následující kapitola je nepovinná a je uvedena jen pro úplnost.
{% endhint %}

Při vývoji lokálních aplikací si často vystačíte s třídami typu `Local...`. Jakmile ale začnete programovat systémy, které komunikují napříč kontinenty, ukládají mezinárodní rezervace letenek nebo zpracovávají burzovní transakce, narazíte na nutnost řešit časová pásma.

Prosté tvrzení „schůzka je ve 14:30“ v globálním světě nestačí. Programátor musí vědět, zda jde o 14:30 v Praze, nebo v New Yorku. Pokud na to zapomenete, uživatelé na druhém konci světa vaše upozornění buď zmeškají, nebo jim přijde uprostřed noci. Moderní Java Time API řeší tento komplexní problém pomocí dvou hlavních tříd: `ZoneId` a `ZonedDateTime`.

### ZoneId: Identifikátor časového pásma

Třída `ZoneId` reprezentuje konkrétní geografické časové pásmo. Tato pásma nejsou definována pouze jednoduchým číselným posunem vůči nultému poledníku (UTC), ale celou historií politických a kalendářních rozhodnutí daného regionu. To zahrnuje pravidla pro přechod na letní čas (DST – _Daylight Saving Time_), která se mohou v průběhu let měnit.

Identifikátory časových pásem se nejčastěji zapisují ve formátu `Kontinent/Město` (např. `Europe/Prague` nebo `America/New_York`).

```java
import java.time.ZoneId;
import java.util.Set;

public class UkazkaZoneId {
    public static void main(String[] args) {
        // Získání výchozího časového pásma počítače, na kterém aplikace běží
        ZoneId mistniZone = ZoneId.systemDefault();
        System.out.println("Moje časové pásmo: " + mistniZone); // Např. Europe/Prague

        // Ruční vytvoření konkrétního časového pásma
        ZoneId newYorkZone = ZoneId.of("America/New_York");
        System.out.println("Časové pásmo New Yorku: " + newYorkZone);
    }
}
```

### ZonedDateTime: Kompletní kalendářní informace

Třída `ZonedDateTime` je nejkomplexnější třídou pro práci s časem v Javě. Spojuje v sobě datum (`LocalDate`), čas (`LocalTime`), konkrétní časové pásmo (`ZoneId`) a navíc přesný číselný posun vůči UTC v daném okamžiku (reprezentovaný třídou `ZoneOffset`).

Díky tomu je `ZonedDateTime` naprosto jednoznačná. S touto třídou můžete snadno převádět čas z jednoho konce světa na druhý, přičemž Java na pozadí automaticky spočítá správné posuny a zohlední případný letní nebo zimní čas v dané lokalitě.

```java
import java.time.LocalDateTime;
import java.time.ZoneId;
import java.time.ZonedDateTime;

public class PrevodCasovychPasem {
    public static void main(String[] args) {
        // 1. Vytvoříme si schůzku v Praze (17. července 2026 ve 14:30)
        LocalDateTime mistniCas = LocalDateTime.of(2026, 7, 17, 14, 30);
        ZoneId praha = ZoneId.of("Europe/Prague");
        
        ZonedDateTime schuzkaVPraze = ZonedDateTime.of(mistniCas, praha);
        System.out.println("Schůzka v Praze:      " + schuzkaVPraze);
        // Výstup: 2026-07-17T14:30+02:00[Europe/Prague]

        // 2. Chceme zjistit, kolik hodin bude v té samé chvíli našemu kolegovi v New Yorku
        ZoneId newYork = ZoneId.of("America/New_York");
        ZonedDateTime schuzkaVNewYorku = schuzkaVPraze.withZoneSameInstant(newYork);
        
        System.out.println("Stejný okamžik v NY:  " + schuzkaVNewYorku);
        // Výstup: 2026-07-17T08:30-04:00[America/New_York]
    }
}
```

Všimněte si, jak elegantně metoda `withZoneSameInstant()` vyřešila matematiku za nás: když je v Praze půl třetí odpoledne, v New Yorku je teprve půl deváté ráno. Java automaticky zjistila, že v červenci platí v Praze letní čas (posun `+02:00`) a v New Yorku rovněž letní čas (posun `-04:00`), a časový rozdíl 6 hodin správně odečetla.
