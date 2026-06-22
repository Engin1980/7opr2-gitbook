# TODO Pole

Pole (anglicky _array_) je v jazyce Java základní mechanismus umožňující udržovat několik položek stejného datového typu pohromadě bez nutnosti vytvářet pro každou instanci novou proměnnou. Namísto složitého, nepraktického a pro hromadné operace nepoužitelného výpisu:

int age0 = 4;\
int age1 = 5;\
int age2 = 7;\
int age3 = 8;\
int age4 = 9;

… lze použít definici pomocí pole:

int \[] data = new int\[5];\
data\[0] = 4;\
data\[1] = 5;\
data\[2] = 7;\
data\[3] = 8;\
data\[4] = 9;

Práce s polem má oproti prvnímu způsobu má mnoho výhod - mezi nejdůležitější je částečná nezávislost na výsledném počtu prvků a snazší hromadné operace nad všemi prvky.

#### Vytvoření pole

Proměnnou pro pole deklarujeme přidáním hranatých závorek mezi datový typ a název proměnné - **u deklarace proměnné se neuvádí velikost pole**. Velikost pole specifikujeme až při vytvoření nové instance zadáním počtu prvků do lomených závorek.

int\[] data = null;\
data = new int\[5];

Při tomto způsobu se jednotlivým prvkům pole přiřadí výchozí hodnoty daného typu - tedy položky pole typu _boolean\[]_ se naplní hodnotami false, číselné typy se naplní nulami, pole nějaké třídy se naplní položkami s hodnotou _null_.

Alternativním zápisem lze vytvořit pole přímo pro zadaný počet hodnot - tehdy se jednotlivé hodnoty uvedou jako položky ve složených závorkách. Vytvořené pole se naplní hodnotami uvedenými ve složených závorkách.

int \[] otherData = new int\[] { 1, 10, 100, 1000, 10000 };

V jazyce Java lze vytvářet i vícerozměrná pole - tj. pole polí. Jedná se už podle názvu o pole, jehož jednotlivou položkou je další pole. Toto zřetězení lze teoreticky opakovat nekonečně, v praxi se však běžně používají dvourozměrná pole (typicky pro matice), případně třírozměrná pole. Více rozměrů se používá velmi zřídka.

U klasického dvourozměrného pole je ještě vhodné odlišovat, zda chceme pole pravoúhlé (klasická matice, každý řádek má stejný počet sloupců), nebo zda chceme, aby každý řádek měl odlišný počet sloupců (anglicky se jim také říká _jagged-arrays_). U pravoúhlého pole explicitně ihned při vytváření instance uvedeme, jaký chceme rozměr pole v obou dimenzích. V druhém případě specifikujeme pouze počet prvků první dimenze a „podpole" pro jednotlivé prvky první dimenze musíme vytvořit zvlášť.

int \[]\[] matrix = new int\[3]\[5];\
int \[]\[] ownSizeArray = new int \[3]\[];\
ownSizeArray\[0] = new int\[5];\
ownSizeArray\[1] = new int\[50];\
ownSizeArray\[2] = new int\[500];

Tento příklad vytvořil matici 3x5 prvků a druhé pole, které má 4 řádky; první řádek druhého pole má 5 prvků, druhý řádek 50 prvků a třetí řádek 500 prvků. Čtvrtému řádku jsme inicializaci neprovedli, proto jeho hodnotou bude hodnota _null_.

#### Přístup k prvkům pole

Na jednotlivé položky pole se přistupuje pomocí indexů - číselných hodnot od 0 do „počtu prvků pole - 1". Pokud tedy vytvoříme pětiprvkové pole (viz předchozí příklady), budou pro přístup k jednotlivým prvkům použity indexy 0, 1, 2, 3 a 4. Minimální index je tedy vždy nula, maximální index je o jedna menší než je počet prvků pole.

Pro získání hodnoty určitého prvku pole použijeme syntaxi „proměnná\[index]". Hodnotu můžeme získat i nastavit.

int \[] data = new int\[5];\
// nastavení hodnoty 2. prvku v poli\
data\[1] = 7;\
// získání hodnoty posledního prvku v poli\
int hodnota = data\[4];

Pro procházení jednotlivých prvků lze samozřejmě použít známých technik, zejména cyklů. Potřebujeme však znát počet prvků. Deklarace pole (např. _int\[]_) představuje samo o sobě opět třídu - proměnná tohoto typu tedy bude nabízet určité metody a instanční proměnné. Pro získání počtu prvků můžeme využít proměnnou _length_, kterou má každé pole. Nemusíme si tedy pamatovat počet prvků sami, instance pole nám vrátí informaci o počtu prvků na požádání.

int \[] data = new int\[5];\
int pocetPrvku = data.length;

Pro procházení přes všechny prvky pole lze použít typicky cyklus _for_.

int\[] data = new int\[5];\
int hodnota;\
for (int i = 0; i < data.length; i++) {\
hodnota = data\[i];\
System.out.println(hodnota);

Druhou běžnou alternativou je přístup, který je nazýván cyklus _for-each_ (nebo _foreach)_. Jeho syntax je trochu odlišná od klasického cyklu _for_, v cyklu chybí řídící proměnná (nemáme tedy povědomí o indexu aktuálně procházeného prvku) a uvnitř cyklus _foreach_ pracuje zcela odlišným způsobem, než cyklus _for_[_\[21\]_](https://word2md.com/#footnote-21). Výstup však bude u obou cyklů stejný. Syntax příkazu _for-each_ je

**for (datový\_typ\_jednoho\_prvku proměnná\_pro\_jeden\_prvek : zdrojové\_pole ){ /\* příkazy \*/ }**

Všimněte si způsobu, jakým se změnila práce s proměnnou _hodnota_ v následujícím příkladu oproti příkladu předchozímu.

int\[] data = new int\[5];\
for (int hodnota : data){\
System.out.println(hodnota);

Pokud chceme procházet vícerozměrná pole, musíme procházet každý rozměr zvlášť.

int \[]\[] data = new int \[3]\[5];\
int hodnota;\
// pomocí cyklu for\
for (int i = 0; i < data.length; i++) {\
for (int j = 0; j < data\[i].length; j++) {\
hodnota = data\[i]\[j];\
System.out.println(hodnota);\
}\
}\
// nebo pomocí for-each\
for(int \[] upper : data)\
for(int lower : upper)\
System.out.println(lower);

#### Operace s polem, třída _Arrays_

Výše byly představeny základní operace - vytvoření pole, procházení pole a přístup na jednotlivé prvky. S poli lze samozřejmě dělat velké množství operací a většinu z nich hromadně. Samotná proměnná typu pole však téměř žádnou funkcionalitu nenabízí - představena byla pouze proměnná obsahující počet prvků.

Valnou většinu funkcionality pro práci s poli naleznete ve statických metodách třídy _java.util.Array**s**_. Tato třída nabízí spoustu metod pro práci s polem, zejména:

* copyOf() - metoda pro kopírování položek jednoho pole do druhého;
* fill() - metoda pro vyplnění prvků pole určitou hodnotou;
* sort() - metoda pro seřazení prvků pole;
* equals() - důležitá metoda pro porovnávání polí. **Pole, stejně jako řetězce, nelze porovnávat pomocí operátoru „==", protože by se porovnaly pouze reference** - tedy zda proměnné odkazují na stejné místo paměti. Při porovnávání polí však typicky chceme porovnávat, zda jsou shodné prvky polí na odpovídajících indexech. Pro toto porovnání tedy musíme využít tuto metodu _equals()_.
* binarySearch() - metoda pro vyhledání prvku v poli. Metoda pracuje na principu půlení intervalů. Z toho důvodu vyžaduje, aby prohledávané pole bylo před voláním této metody seřazeno (pomocí metody _sort()_). Metoda vrací index prvku jako nezápornou hodnotu v případě úspěšného nalezení odpovídajícího prvku, zápornou hodnotu v opačném případě.
* toString() - metoda vypíše přehledně všechny prvky pole. Opět, samotná metoda _toString()_ od proměnné typované na pole nevrací smysluplný obsah. Pokud chceme přehledně vypsat všechny prvky pole, využijeme této metody.

Následující příklad představí výtah z výše představených operací. Algoritmus vytvoří dvě pole o stejném počtu prvků. První pole naplní náhodnými hodnotami (z intervalu 0-10), druhé konstantní hodnotou „k". Dále vytvoří třetí pole, do kterého dá pouze ty prvky prvního pole, které jsou větší nebo rovny prvku druhého pole na odpovídajícím indexu. Finálně zjistí hodnotu maximálního a minimálního prvku a zda je konstantní hodnota „k" obsažena ve výsledném poli.

// inicializace\
int initialCount = 30;\
int firstArrayMaximumValue = 10;\
int secondArrayValue = 5;\
Random rnd = new Random();\
// vytvoreni dvou vychozich poli\
int \[] first = new int\[initialCount];\
int \[] second = new int \[initialCount];\
// naplneni prvniho pole nahodnymi hodnotami\
for (int i = 0; i < first.length; i++)\
first\[i] = rnd.nextInt(firstArrayMaximumValue + 1);\
// naplneni druheho pole konstantni hodnotou\
Arrays.fill(second, secondArrayValue);\
// vytvoreni tretiho pole a citace poctu prvku ve tretim poli\
int \[] third = new int\[initialCount];\
int thirdLastIndex = -1;\
// vybrani prvku do tretiho pole\
for (int i = 0; i < first.length; i++) {\
if (first\[i] >= second\[i]){\
thirdLastIndex++;\
third\[thirdLastIndex] = first\[i];\
}\
}\
// kratkodobe pomocne pole (pouze pro prehlednost)\
int \[] pom = third;\
// odrezu z tretiho pole prvky, ktere na konci zustaly prazdne\
third = Arrays.copyOf(pom, thirdLastIndex+1);\
// udelam si kopii tretiho pole pro ziskani minima, maxima a hledani\
int \[] thirdSorted = Arrays.copyOf(third, third.length);\
// priprava promennych\
int min;\
int max;\
boolean isInArray = false;\
// setridim pole\
Arrays.sort(thirdSorted);\
// prvni prvek je nejmensi\
min = thirdSorted\[0];\
// posledni prvek je nejvetsi\
max = thirdSorted\[third.length-1];\
// zkusim nalezt index prvku s hodnotou druheho pole\
// pokud je hledani uspesne, vraci se index >= 0\
isInArray =\
Arrays.binarySearch(thirdSorted, secondArrayValue) >= 0;\
// vypis vysledku\
System.out.println("Pole s výslednými prvky: " + Arrays.toString(third));\
System.out.println("Maximum: " + max + ", minimum: " + min);\
System.out.println("Nalezena výchozí hodnota druhého pole? " + isInArray);

Práce s poli má však své nevýhody. Největším problémem je fixní velikost pole, takže programátor musí v případě operací, které pole rozšiřují nad jeho kapacitu, neustále vytvářet kopie zvětšené kopie původního pole. Bylo by proto vhodné mít pro práci se sadami prvků komfortnější třídu, která by umožnila snazší práci při přidávání, vybírání, řazení, probírání, editaci a mazání prvků, nehledě na implementaci určitých struktur běžných z klasických programovacích přístupů, jako jsou zejména fronta či zásobník.

Jazyk Java tyto nedostatky řeší pomocí své knihovny obecně nazývané jako _Collection Framework_.
