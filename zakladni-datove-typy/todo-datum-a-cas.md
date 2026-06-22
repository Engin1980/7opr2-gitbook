# TODO Datum a čas

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
String formatPattern = "yyyy-MM-dd HH:mm:ss";\
SimpleDateFormat sdf = new SimpleDateFormat(formatPattern);\
String dateAsString = sdf.format(c.getTime());\
System.out.println(dateAsString);\
// nějaká opravdu složitá šablona\
formatPattern =\
"EEEE, 'datum' d. MMMM yyyy, G, 'čas' hh:mm:ss, a; z";\
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

&#x20;// příprava proměnných\
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

&#x20;// nastavím dvě lokality, default (českou) a americkou (US)\
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

&#x20;Calendar c = Calendar.getInstance();\
DateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");\
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
