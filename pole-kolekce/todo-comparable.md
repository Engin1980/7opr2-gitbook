# TODO Comparable

Výše uvedený výpis uvádí chybu, která se zmiňuje o faktu, že instance třídy _Person_ nelze přetypovat na rozhraní _java.lang.Comparable_. Kompilátoru totiž u pro něj neznámých tříd musíme říct, na základě čeho má porovnávat mezi sebou instance naší třídy.&#x20;

{% hint style="info" %}
Pokud uděláme vlastní třídu `Pračka`, kompilátor neví, podle čeho má říci, že pračka `a` je v pořadí výše/níže než pračka `b`.
{% endhint %}

Musíme tedy definovat porovnávací kritérium, které řekne, kdy je určitá instance výše/níže při porovnávání než druhá.

## Rozhraní Comparable

Základním rozhraním pro definici porovnávání je rozhraní `Comparable<E>`\` . K rozhraní se přidává generický parametr E, který nahrazujeme konkrétní třídou, kterou má naše implementace umět porovnávat.&#x20;

Typicky tedy zápis vypadá:

```java
public class Mouse implements Comparable<Mouse> {...}

public class Car implements Comparable<Car> {...}
```

{% hint style="info" %}
Toto rozhraní lze zapsat standardně bez generického parametru  `<>` (tedy `Comparable`) a tehdy umožňuje porovnávání instanci libovolných tříd mezi sebou. Často ale není žádoucí porovnávat cokoliv s čímkoliv, ale pouze shodné typy (tedy neporovnáváme jablka s hruškami, ale pouze jablka s jablky a hrušky s hruškami).&#x20;

Běžně použijeme již známý zápis využívající lomených závorek `Comparable<E>` se zapsáním porovnávaného typu. Ve studijní opoře budeme dále využívat pouze druhou představenou variantu.
{% endhint %}

Toto rozhraní vyžaduje u tříd, které jej implementují, implementaci metody:

```java
int compareTo (E other)
```

Funkce přijímá druhý objekt jako parametr a provede porovnání mezi oběma objekty způsobem, který specifikuje programátor. Funkce následně vrací následující hodnoty:

* Hodnotu menší než 0 - pokud je aktuální objekt menší než objekt předaný jako parametr;
* Hodnotu rovnu 0 - pokud jsou oba objekty shodné;
* Hodnotu větší než 0 - pokud je aktuální objekt větší než objekt předaný jako parametr.

{% hint style="info" %}
Prvním objektem je instance, nad kterou se daná metoda volá, v rámci metody se na první objekt tedy odkazujeme pomocí klíčového slova _this_.

Obecné volání pro porovnání objektů `a` a `b` by tedy bylo `a.compareTo(b)`. V reálu je většinou objektem `a` volající objekt, tedy `this`.
{% endhint %}

Pokud výše uvedené třídě _Person_ doplníme požadavek na implementaci rozhraní _Comparable\<Person>_, nabídne nám vývojové prostředí negenerování požadované metody automaticky.

```java
class Person implements Comparable<Person> {
    private String name;
    private int age;
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    @Override
    public int compareTo(Person o) {
        // způsob porovnání
    }
}
```

Nyní zbývá doplnit funkcionalitu porovnání. Jak bylo řečeno, ta záleží zcela na vůli programátora. V našem případě nejdříve realizujeme porovnání podle věku, které by mohlo vypadat například takto:

```java
class Person implements Comparable<Person> {
    private String name;
    private int age;
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    @Override
    public int compareTo(Person o) {
        if (this.age < o.age)
            return -1;
        else if (this.age == o.age)
            return 0;
        else
            return 1;
    }
}
```

Lze vidět, že požadavky na chování funkce (uvedené výše) jsme zachovali. Chování můžeme velmi rychle ověřit kdekoliv v kódu (například v metodě _main()_) při pokusu srovnat dva prvky. Změnou hodnot věku lze rychle zjistit, že se daná funkce chová korektně.

```java
Person a = new Person("Petra", 20);
Person b = new Person("Iva", 18);
int compareResult = a.compareTo(b);
System.out.println(compareResult);
```

Teoreticky složitější příklad nastane, pokud bychom chtěli, aby třída porovnávala osoby podle jména. Jméno je totiž datového typu _String_ a při porovnání řetězců nelze použít operátory <, > nebo ==. Z kapitoly 5.4.1.2 však již víme, že datový typ _String_ má metodu _equals()_, která ale vrací pouze rovnost/nerovnost instancí. Třída _String_ však obsahuje také metodu _compareTo()_. Pozorní čtenáři si jistě povšimnou, že takovou funkci máme v naší třídě _Person_ také - a důvod je zřejmý: i třída _String_ implementuje rozhraní _Comparable_, aby jej bylo možno porovnávat. Pro porovnání řetězců tedy pouze jednoduše použijeme tuto funkcionalitu a výsledný kód třídy _Person_, resp. její metody _compareTo()_ tak bude ještě jednodušší, než u porovnávání věku:

```java
class Person implements Comparable<Person> {
    private String name;
    private int age;
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    @Override
    public int compareTo(Person o) {
        return this.name.compareTo(o.name);
    }
}
```

Takto můžeme upravit libovolnou vlastní třídu tak, aby bylo možno ji použít v souvislosti s třídou _TreeSet_. Toto chování ale můžeme využít také u seznamů - seznamy (_java.util.List_, kapitola 7.2.5) mají metodu _sort()_, která umožňuje seřazení jejich prvků, a toto seřazení také používá výše uvedený mechanismus.

Tomuto způsobu řazení, kdy ve třídě definujeme pomocí rozhraní _Comparable_ metodu _compareTo()_, ve které definujeme způsob řazení instancí třídy, se říká přirozené, nebo také **nativní řazení** (natural ordering). Alternativou je řazení pomocí rozhraní `Comparator` vysvětlené dále.
