# TODO String, StringBuilder

#### Datový typ String

Základním datovým typem realizujícím práci s řetězcem je typ _String_. Jedná se o třídu, její název tedy začíná velkým písmenem a řetězec _String_ není klíčovým slovem.

Vytvoření proměnné obsahující řetězec je velmi jednoduché. Řetězce se v jazyce Java uzavírají do klasických (anglických) uvozovek.!**!**

String a = "první";\
String b = "b"; // uvozovky značí řetězec\
char c = 'b'; // apostrofy značí znak, nikoliv řetězec

Základní a velmi důležitou vlastností instance typu _String_ je, že _**String**_**&#x20;je neměnný!** To znamená, že vytvořenou instanci v paměti již nemůžete změnit a pokud například potřebujete v řetězci nahradit určitý znak jiným, nebo řetězec rozšířit/zkrátit, musíte vytvořit novou instanci v paměti požadovaného tvaru - existující instance se již nedá změnit.

Před představením operací s řetězcem je třeba ještě odlišit dva důležité stavy:

* Řetězec může nabývat hodnotu _null_ - tehdy v proměnné není uložena žádná hodnota, proměnná nikam neukazuje a **nad takovou proměnnou nelze volat žádné metody** (volání metod způsobí chybu). Nastavením takové hodnoty může být volání „String a = null;".
* Řetězec může nabývat hodnotu prázdného řetězce - tehdy je v proměnné uložena instance třídy _String_, jenom je prázdná - neobsahuje žádný text, její délka je rovna nule. Nad takovou proměnnou lze volat metody. Nastavením takové hodnoty může být volání „String a = "";".

**Práce s typem String**

Spojení dvou řetězců provedeme pomocí operátoru „+".

Třída _String_ dále obsahuje velké množství metod, které umožňují základní operace s řetězci. Nejběžnější funkce, které lze využít, ukazuje následující tabulka. Některé funkce pracují s indexy - _String_ je chápán jako pole znaků (pozor ale, že nelze jej přímo přetypovat na pole znaků) a indexování se používá stejně, jako u polí - tedy první index má hodnotu 0, poslední index má hodnotu o jedna menší, než je délka řetězce.

| Signatura funkce                                 | Popis                                                                                                                                             |
| ------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| int length()                                     | Vrací délku řetězece.                                                                                                                             |
| boolean isEmpty()                                | Vrací TRUE, pokud je řetězec prázdný (obsahuje hodnotu ""). Pozor - nelze volat u proměnných, které obsahují hodnotu null.                        |
| String trim()                                    | Ořeže řetězec z obou stran o „bílé" znaky - mezery, tabulátory apod.                                                                              |
| char charAt(int index)                           | Vrací znak na zadaném indexu.                                                                                                                     |
| boolean startsWith(String prefix)                | Vrací true, pokud řetězec začíná na zadaný podřetězec.                                                                                            |
| boolean endsWith(String sufix)                   | Vrací true, pokud řetězec končí na zadaný podřetězec.                                                                                             |
| int indexOf(char znak)                           | Vrací index, na kterém je nalezen daný znak. Vyhledávání postupuje zleva doprava. Obsahuje přetížení, mj. i pro vyhledání podřetězce v řetězci.   |
| Int lastIndexOf(char znak)                       | Vrací index, na kterém je nalezen daný znak. Vyhledávání postupuje zprava doleva. Obsahuje přetížení, mj. i pro vyhledávání podřetězce v řetězci. |
| String substring(  int beginIndex, int endIndex) | Vrací podřetězec z řetězce mezi zadanými indexy.                                                                                                  |
| String \[] split(String regex)                   | Rozdělí řetězec podle zadaného výrazu a vrací pole rozdělených řetězců.                                                                           |
| String replace(  char oldChar, char newChar)     | Nahradí znak původního řetězce novým znakem.                                                                                                      |

Povšimněte si, že u všech funkcí, které pracují se instancí řetězce, se vždy výsledek vrací jako nová hodnota funkce. Volání funkce proto nikdy nemění původní instanci (viz **string je neměnný** výše), ale vždy vytvoří novou, upravenou instanci. Proto nikdy nelze zavolat například:

a.replace('a', 'b');

, protože výsledná hodnota se v původní proměnné **nezmění**. Následující kód ukazuje příklady volání funkcí na proměnnou typu _String_.

String a = "Seven";\
boolean jeNull = a == null; // pozor, test na null provádíme takto\
boolean jePrazdny = a.isEmpty(); // prázdný řetězec nemusí být null!\
// výpis po znacích\
int delka = a.length();\
for (int i = 0; i < delka; i++){\
char c = a.charAt(i);\
System.out.println(c);\
}\
boolean zacinaNaS = a.startsWith("S");\
boolean konciNaN = a.endsWith("n");\
// ořez o krajní písmena\
String orezaneA = a.substring(1, delka-1);\
System.out.println("Ořezané: " + orezaneA);\
// nahrazení písmen 'v'\
// výsledek musím vložit do proměnné, byť stejné\
// jako je ta, nad kterou se funkce volá !\
a = a.replace('v', '7');\
System.out.println("Nahrazené: " + a);\
// výpis věty po slovech\
String veta = "Toto je věta.";\
String \[] slova = veta.split(" ");\
for(String s : slova)\
System.out.println(s);

Výše uvedený kód dá následující výstup:

S

e

v

e

n

Ořezané: eve

Nahrazené: Se7en

Toto

je

věta.

Povšimněte si volání řádku s funkcí _replace()_. Opravdu, i když chceme změnit přímo hodnotu v proměnné _a_, musíme si novou, upravenou hodnotu vrátit zpět do této proměnné. Pokud bychom přiřazení neprovedli (chybělo by v příkazu „a = "), operace by se sice provedla, ale výsledek by se nikam nevrátil a hodnota v proměnné _a_ by zůstala nezměněna.

Je vhodné si také uvědomit, že samotný řetězec je chápán jako instance třídy _String_, proto i přímo nad řetězcem můžeme volat metody, například:

int delkaVety = "Toto je poměrně dlouhá věta".length();

K funkcím třídy _String_ ještě jedna velmi důležitá poznámka. Je třeba se věnovat pojmenovávání parametrů funkcí třídy _String_ - některé funkce využívají parametry pojmenované jako _**regex**_ (například funkce _split()_). Tyto funkce využívají jako parametry tzv. regulární výrazy - což je (zjednodušeně řečeno) pseudojazyk určeny pro definici obecného vzhledu řetězce. V této kapitole regulární výrazy vysvětleny nebudou, je však mít na paměti minimum - že určité znaky v regulárních výrazech mají speciální význam a nebudou běžně fungovat. Následuje velmi krátký příklad.

String data =\
"Ano. Takto jsem to řekl. Řekl jsem to jasně. Tak už to víte.";\
// korektně rozdělí sadu vět na slova\
// parametr " " reprezentuje správně mezeru\
String \[] slova = data.split(" ");\
// má rozdělit sadu vět na věty\
\*\*// nebude ale fungovat\
\*\*// znak tečky "." má v regulárních výrazech zvláštní význam\
String \[]vety = data.split(".");

Bližší informace o regulárních výrazech lze najít například na [www.regularnivyrazy.info](http://www.regularnivyrazy.info).

**Porovnávání řetězců**

Důležitou operací je porovnávání řetězců. **Řetězce v Javě nikdy nelze porovnávat pomocí operátoru ==.** V Javě operátor „==" slouží pro porovnání referencí mezi objekty, zjišťuje tedy, zda dvě proměnné ukazují do paměti na stejné místo. U řetězců tento případ však velmi jednoduše nastat nemusí - pokud máme dvě proměnné, mohou ukazovat do paměti na různá místa, ale tyto místa mohou obsahovat stejný řetězec.

**String a**

**String b**

**Paměť**

"ABCD"

"ABCD"

Pokud chceme jednoduše porovnat hodnoty proměnných na shodnost řetězců, využijeme funkci _equals()_, která vrací příznak true/false, zda jsou hodnoty v obou řetězcích shodné. Můžeme využít také funkce _equalsIgnoreCase()_, která porovná řetězce bez ohledu na malá/velká písmena.

**Skládání řetězců vs. paměť**

Již byla zmíněna problematika neměnnosti řetězců. Její negativní dopad se velmi projeví při hromadném skládání řetězců. Kolik instancí proměnné _String_ vytvoří následující příklad na řádku, kde se vytváří hodnota pro proměnnou _dohromady_?

String a = "Toto";\
String b = "je";\
String c = "sestavená";\
String d = "věta";\
String dohromady =\
a + " " + b + " " + c + " " + d + ".";\
System.out.println(dohromady);

Celkem se vytvoří následující instance: „Toto „, „Toto je", „Toto je „, „Toto je sestavená", „Toto je sestavená „, „Toto je sestavená věta" a nakonec „Toto je sestavená věta." - a pouze tato poslední instance se předá do proměnné _dohromady_, ostatní proměnné (celkem 6) se předají garbage collectoru téměř ihned po vytvoření. Toto má samozřejmě značný negativní výkonnostní dopad.



#### Datový typ StringBuilder

Právě kvůli výše uvedené problematice existuje typ StringBuilder. Třída je, stejně jako _String_, v balíčku _java.lang_.

Oproti _Stringu_ nenabízí takový komfort jako instance klasického řetězce (například je nelze řetězit pomocí operátoru „+"), ale hlavně **umožňuje změnu již vytvořené hodnoty** řetězce, což klasický _String_, jak již bylo několikráte zdůrazněno, neumí.

Vytvoření nové prázdné instance, stejně jako převedení existujícího řetězce do proměnné _StringBuilder_ a převedení zpět, z instance _StringBuilder_ na řetězec, je, s využitím konstruktoru a metody _toString()_ velmi jednoduché.

StringBuilder a = new StringBuilder();\
// převod řetězce na StringBuilder\
StringBuilder b = new StringBuilder("první");\
// převod StringBuilderu na řetězec\
String s = b.toString();

Je třeba si však vždy uvědomit, že _StringBuilder_ a _String_ jsou vzájemně nekompatibilní typy, takže je nelze například řetězit dohromady pomocí operátoru „+".

**Poznámka**. V prostředí jazyka Java ještě existuje velmi podobná třída, nazvaná StringBuffer. Tato třída nabízí stejné metody jako StringBuilder, oproti němu je však bezpečná při použití vícevláknových operací, v důsledku je však o trochu pomalejší. My se problematice vícevláknových operací věnovat nebudeme, proto tato třída nebude dále vysvětlena.

**Operace typu StringBuilder**

Třída _StringBuilder_ také nabízí množství metod, většinu z nich orientovaných na editaci svého obsahu. Některé funkce mají stejné názvy, signatury i chování jako u typu _String_, například _charAt(), indexOf(), lastIndexOf()_. Následující tabulka ukazuje další funkce.

| Signatura funkce                                    | Popis                                                                                                                                                                                             |
| --------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| StringBuilder appends(String s)                     | Přidá k existujícímu řetězci na konec další řetězec zadaný jako parametr. Tato funkce má množství přetížení i pro různé další typy. Přidávaná hodnota se zařadí vždy na konec řetězce v proměnné. |
| StringBuilder insert(int offset, String s)          | Vloží na zadaný index _offset_ hodnotu parametru _s_. Tato funkce slouží pro vložení řetězce jinam, než na konec řetězce.                                                                         |
| StringBuilder delete(int start, int end)            | Smaže znaky od indexu _start_ do indexu _end_ (v rozsahu _start - (end-1)_.                                                                                                                       |
| StringBuilder reverse()                             | Otočí pořadí znaků v řetězci.                                                                                                                                                                     |
| StringBuilder replace(int start, int end, String s) | Vyjme řetězec mezi indexy _start_ (včetně) a _end_ (nevčetně) a vloží místo vyjmutého obsahu zadaný řetězec _s_.                                                                                  |

Pozor na funkci _replace()_ - má stejný název, jako funkce u typu _String_, ale provádí zcela odlišnou operaci!

Všimněte si také, že všechny funkce vrací instanci _StringBuilder_ - zde je to bohužel trochu matoucí. Všechny vrácené hodnoty jsou **instance stejné proměnné, která funkci vyvolala**. Hodnota se mění tedy přímo v proměnné, nad kterou se hodnota volá, a hodnot vrácených z těchto funkcí **si vůbec nemusíme všímat**.

Nejčastější metodou je metoda _append_, která slouží pro sestavení řetězce. Výše uvedený problémový případ použití řetězce _String_ lze přepsat pomocí třídy _StringBuilder_.

String a = "Toto";\
String b = "je";\
String c = "sestavená";\
String d = "věta";\
StringBuilder sb = new StringBuilder();\
sb.append(a);\
sb.append(" ");\
sb.append(b);\
sb.append(" ");\
sb.append(c);\
sb.append(" ");\
sb.append(d);\
sb.append(".");\
String dohromady = sb.toString();\
System.out.println(dohromady);

Přestože je zdrojový kód delší, toto volání bude mnohem rychlejší než volání nad typem _String_, protože se vytvoří pouze jedna instance (typu _StringBuilder_), v ní se odehrají všechny operace skládání a výsledný řetězec se vrátí jako _String_.

**Kontrolní otázky:**

* Jaký je rozdíl mezi třídami _String_ a _StringBuilder_? Co je pro ně naopak společné?
* K čemu slouží wrapovací typy?
* Seřaďte číselné typy podle velikosti čísla, které do nich lze zadat.
* K čemu slouží typ _boolean_?

Úkoly k zamyšlení:

Máme hodnoty rozsahu 0-255. Jaký datový typ pro jeho reprezentaci můžeme použít? Jaké jsou výhody a nevýhody vámi navrženého řešení?

Korespondenční úkol:

Vytvořte třídu, která realizuje komplexní číslo. Implementujte základní aritmetické metody, otestujte pomocí unitových testů a demonstrujte použití.

Vytvořte třídu, která umí rozdělit zadaný řetězec podle určitého znaku a opět jej složit dohromady, již bez tohoto znaku.

Shrnutí obsahu kapitoly

Kapitola představila problematiku práce se základními primitivními datovými typy v jazce Java, problematiku wrapování primitivních typů a problematiku konverze řetězců na čísla a zpět. Další část kapitoly se věnovala datovému typu _Object_ a poslední část kapitoly se věnovala typům _String_ a _StringBuilder_ a jejich použití.
