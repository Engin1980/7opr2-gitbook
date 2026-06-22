# TODO Object

## Object vs Dědičnost

V kapitole 3 byla zopakována, či upřesněna problematika dědičnosti tříd. Dědičnost mezi třídami nevytváří pouze programátor na základě svého rozhodnutí, ale obecně, všechny třídy jsou řazeny do hierarchie třídy provázané právě dědičností.

Nejvyšším předkem, tj. nejvyšší třídou v hierarchii dědičnosti, je třída _System.Object_. Tato třída funguje jako nejvyšší předek - do instance této třídy je tedy na základě vztahu generalizace-specializace (kapitola 3.2) možno vložit instanci libovolné možné třídy. Je tomu tak proto, že **každá třída dědí ze třídy&#x20;**_**Object**_**, i pokud nemá žádnou dědičnost specifikovánu programátorem.** Proto, pokud programátor u vlastní třídy neuvede explicitně dědění z jiné třídy pomocí klíčového slova _extends_, automaticky se doplní _extends Object_ a třída tak bude v hierarchii dědičnosti zahrnuta jako přímý potomek třídy _Object_.

Díky dědičnosti získávají všechny třídy implementaci některých metod, které mohou dále přetížit. Některé metody se věnují vícevláknovému programování (mimo rozsah této opory). Některé z nich lze však velmi často využít i při implementaci vlastních tříd a proto musí být představeny.

## Metody třídy Object

### Metoda toString()

Metoda _toString()_ je využívána při potřebě reprezentace instance třídy jako řetězce. Kdykoliv je třeba reprezentovat instanci jako řetězec, vykoná se tato metoda a vrátí se její výsledek.

Základní implementace ve třídě _Object_ vrací typ, od kterého je vytvořena instance, doplněný o číselný kód získaný z funkce _hashCode()_ (tato funkce bude představena později; zatím si pod ní představte nějaké náhodné číslo). Tato informace samozřejmě programátorovi typicky moc informací o instanci neřekne. Navíc - výsledek volání této metody využívá nejen programátorské prostředí, ale i některé nástroje realizující zobrazení dat uživateli a tehdy bychom uživateli zobrazili nějaká podivná data.

```java
Car c = new Car();
Object o = c;
String v = o.toString();
System.out.println(v);
```

Výše uvedený příklad předpokládá existenci třídy _Car_ v balíčku _somePackageName_ a vrátí následující výstup.

```
somePackageName.Car@79404a
```

Je zřejmé, že tento výstup programátorovi ani uživateli moc informací o dané instanci neposkytne.

Díky technice překrývání metod (kapitola 3.4.2) je však možno v potomkovi danou metodu překrýt a definovat tak její nové chování, do kterého můžeme vypsat určité informace o dané instanci. Takový výstup potom jistě ocení programátor i uživatel.

```java
public class Car {
    private String spz;
    public String getSpz() {
        return spz;
    }
    public void setSpz(String spz) {
        this.spz = spz;
    }
    @Override
    public String toString() {
        return "Car{" + spz + '}';
    }
}
```

Ještě je třeba poupravit kód spuštěné metody _main_().

```java
Car c = new Car();
c.setSpz("4T5 4687");
Object o = c;
String v = o.toString();
System.out.println(v);
```

Výše uvedená třída má jeden atribut (SPZ) a ten vrací jako informaci v překryté metodě _toString()_. Výstup stejného kódu jako v předchozím příkladu potom bude vypadat takto:

```
Car{4T5 4687}
```

V případě metody _toString()_ je ještě poměrně časté odkazování se na výsledek volání metody předka, který nám již nějaké informace poskytuje a nemusíme tak celý vypisovaný řetězec skládat dohromady ručně.

```java
class OwnedCar extends Car{
    private String ownerName;
    public String getOwnerName() {
        return ownerName;
    }
    public void setOwnerName(String ownerName) {
        this.ownerName = ownerName;
    }
    @Override
    public String toString() {
        String ret =
            super.toString() + "{owner: " + ownerName + "}";
        return ret;
    }
}
```

Třída _OwnedCar_ reprezentuje potomka třídy _Car_, který navíc definuje název/jméno vlastníka. Je důležité si povšimnout, jakým způsobem - voláním metody _super.toString()_ - využívá implementace získání výsledku již dostupné implementace metody _toString()_ v předkovi. Vracená hodnota metody _toString()_ třídy _OwnedCar_ tedy vrací původní řetězec, který vytvořila třída _Car_, doplněný o další informaci týkající se vlastníka. Nemusíme tedy vytvářet celý výsledný řetězec sami ručně.

Vykonávaný kód se téměř nezmění.

```java
OwnedCar c = new OwnedCar();
c.setSpz("4T5 4687");
c.setOwnerName("Darling, s.r.o.");
Object o = c;
String v = o.toString();
System.out.println(v);
```

Výstupem bude doplněné volání:

```
Car{4T5 4687}{owner: Darling, s.r.o.}
```

{% hint style="info" %}
Při implementaci metody toString() nemusíme zdrojový kód psát sami, ale v prostředí IDE můžeme využít možnosti nechat si odpovídající kód vygenerovat a vygenerovaný kód případně poupravit podle našich představ.&#x20;
{% endhint %}

### Metody equals() a hashCode()

Další dvojicí metod jsou metody _equals()_ a _hashCode()_. Motivaci k existenci obou metod je nutnost porovnávat jednotlivé instance na shodnost - tedy, zda jsou instance stejné. Ne vždy je však pro programátora výhodný předpoklad, že odlišné instance reprezentují odlišný objekt. Někdy chceme definovat, že určitý objekt je shodný s jiným objektem (tedy že určitá instance nějaké třídy je shodná s jinou instanci třídy), pokud se shoduje jejich obsah. Typickým příkladem je třída _String_. Přestože můžeme vytvořit v paměti dvě odlišné instance, ale každá bude obsahovat řetězec „ahoj". Těžko bychom tehdy při porovnání chtěli, aby nám systém říkal, že se jedná o odlišné objekty, když reprezentují totéž.

Nejjednodušší porovnání je pomocí porovnávacího operátoru „==", ten však vždy realizuje porovnání, zda jsou shodné instance - tedy zda proměnné ukazují na stejné místo v paměti. Jeho chování je v jazyce Java pevně dané a neměné.

#### Equals

Základní metodou pro porovnání, kterou lze přizpůsobit, je metoda _equals()_ definovaná ve třídě _Object_. Metoda vrací příznak true/false popisující, zda jsou objekty chápány jako shodné či nikoliv. Ve třídě _Object_ se realizuje klasické porovnání v paměti - tedy pokud dvě proměnné odkazují na stejné místo v paměti, reprezentují stejný objekt, jinak nikoliv.

Toto chování však programátor může opět překrýt. Při překrytí může říci, jak se bude chápat rovnost dvou objektů - v čem se musí objekty rovnat, aby si byly ekvivalentní.

```java
Car a = new Car();
Car b = new Car();
a.setSpz("4T5 4687");
b.setSpz("4T5 4687");
System.out.println(a == b);
System.out.println(a.equals(b));
```

Výše uvedený kód ukazuje, jakým způsobem lze jednoduše porovnat dva objekty. První výpis ukazuje porovnání pomocí operátoru „==", druhý výpis porovnání pomocí metody _equals()_. Předpokládáme-li nezměněnou třídu _Car_ uvedenou v předchozích příkladech, získáme výstup:

```
false
false
```

Jak bylo však řečeno, chování metody _equals()_ můžeme jednoduše změnit. Upravíme třídu _Car_.

```java
public class Car {
    private String spz;
    public String getSpz() {
        return spz;
    }
    public void setSpz(String spz) {
        this.spz = spz;
    }
    @Override
    public boolean equals(Object other){
        // pokud je hodnota proměnné other instancí třídy "Car"
        if (other instanceof Car){
            // přetypuji hodnotu do nové proměnné typu "Car"
            Car otherCar = (Car) other;
            // a vrátím porovnání jejich SPZtek
            // všimněte si využití metody .equals() třídy "String"
            return
                this.spz.equals(otherCar.spz);
        }
        else
            // pokud je hodnota proměnné Other odlišný typ než "Car",
            // tak jistě nejsou hodnoty shodné.
            return false;
    }
}
```

Pokud spustíme nyní výše uvedený kód pro porovnání, dostaneme odlišný výstup:

```
false
true
```

Je vidět, že nyní, i když proměnné odkazují na odlišné instance, jsou při volání metody _equals()_ proměnné chápány jako shodné.

Důležité je povšimnout si, že i **pro porovnání shodnosti řetězců** **nesmíme použít operátor ==, ale použijeme metodu&#x20;**_**equals()**_**.**

Ve vývojovém prostředí je nyní oznámeno varování, že jsme překryli metodu _equals()_, ale nepřekryli jsme metodu _hashCode()_. K čemu tedy slouží metoda _hashCode()_?

#### Hashcode

Jistě jste si povšimli, že v metodě _equals()_ můžete definovat libovolný vlastní zdrojový kód, který v některých případech může být poměrně komplikovaný. Samozřejmě u komplikovaného kód je typicky časově náročné i jeho vykonání a v některých případech, například při prohledávání pole o velkém počtu položek, by opakované volání takovéto metody mělo výrazný negativní výkonnostní dopad. Metoda _hashCode()_ umožňuje tento dopad částečně eliminovat.

Metoda _hashCode()_ vrací číselný kód (tzv. hash), pro který platí následující pravidla:

* Pokud je kód pro dva objekty odlišný, objekty **nutně musí** být odlišné (při porovnání metodou _equals()_) - tedy dva objekty s odlišným kódem nesmí při volání metody _equals()_ vrátit hodnotu _true_.
* Pokud je kód pro dva objekty shodný, objekty **mohou být (ale nemusí!)** shodné (při porovnání metodou _equals()_) - tedy dva objekty se shodným kódem ještě nutně nemusí při volání metody _equals()_ vrátit hodnotu _true_.

Pointa celého principu je, že při vyhledávání se pomocí metody _hashCode()_ rychle vyřadí ty objekty, které jistě nejsou s prohledávaným prvkem shodné a teprve nad zbývající skupinou „potenciálně shodných objektů" se testuje shodnost pomocí metody _equals()_. Toto řešení poskytuje značný výkonnostní zisk.

Kód metody _hashCode()_ lze napsat několika různými způsoby - od nejjednodušších variant až po různě složité, optimalizované varianty. Typickými jednoduchými variantami jsou řešení spočívající ve sčítání jednotlivých položek nebo použití funkce _XOR_. Složitější varianty umožňují vygenerovat vlastní zdrojový kód, který optimalizuje výsledky metody _hashCode()_ pro různé instance.

```java
public class Car {
    private String spz;
    public String getSpz() {
        return spz;
    }
    public void setSpz(String spz) {
        this.spz = spz;
    }
    @Override
    public boolean equals(Object other){
        // stejný kód jako výše
    }
    @Override
    public int hashCode(){
        return this.spz.hashCode();
    }
}
```

Výše uvedený příklad ukazuje nejjednodušší překrytí metody _hashCode()_ pro výše třídu _Car_. Pokud víme, že chceme porovnávat hodnoty pouze podle jednoho atributu, prostě vrátíme hashCode tohoto atributu.

{% hint style="info" %}
Stejně jako u metody toString(), i u metod hashCode() a equals() můžeme využít možnosti nechat si kód vygenerovat. Průvodce v IDE nám většinou dá na výběr, které třídní proměnné chceme zahrnout do výpočtu `hashcode()`a `equasl()` a odpovídající kód vytvoří.

Následující kód ukazuje automaticky generovaný kód těchto metod pro třídu Car.
{% endhint %}

```java
public boolean equals(Object obj) {
    if (obj == null) {
        return false;
    }
    
    if (getClass() != obj.getClass()) {
        return false;
    }
    
    final Car other = (Car) obj;
    if (!Objects.equals(this.spz, other.spz)) {
        return false;
    }
    return true;
}

public int hashCode() {
    int hash = 5;
    hash = 67 * hash + Objects.hashCode(this.spz);
    return hash;
}
```
