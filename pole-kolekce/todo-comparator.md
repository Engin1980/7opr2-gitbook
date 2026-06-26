# Comparator

Při vytváření předchozího příkladu si lze povšimnout, že můžeme definovat, zda se třída `Person` bude porovnávat podle věku, nebo podle jména, ale **nelze definovat obojí**, protože funkce `compareTo()` může být v třídě `Person` pouze jedna.

Pokud budeme tedy mít dva seznamy stejných osob a oba setřídíme pomocí volání metody `sort()`, bude pořadí jejich prvků shodné. Co ale, pokud chceme jeden seznam řadit podle věku a druhý podle příjmení?

Pokud tedy chceme instance určité třídy porovnávat více způsoby, musíme použít jiný přístup, který je založen na implementaci rozhraní s podobným názvem, ale odlišnou funkcionalitou - `java.lang.Comparator`.&#x20;

{% hint style="warning" %}
Nezaměňujte názvy těchto rozhraní (`Comparable` vs. `Comparator`)!
{% endhint %}

## Rozhraní Comparator

Pro každý způsob řazení, který chceme realizovat (samozřejmě vyjma již případně realizovaného nativního řazení pomocí rozhraní `Comparable`), nejdříve vytvoříme samostatnou třídu, která bude implementovat rozhraní `Comparator`.

Uvažujme opět příklad s třídou `Person`, kdy jsme realizovali nativní řazení pomocí porovnávání hodnot jména (poslední výpis).

Pokud budeme také chtít řadit osoby podle věku, musíme vytvořit novou, samostatnou třídu, která bude implementovat požadované rozhraní `Comparator`.

```java
class PersonByAgeComparator implements java.util.Comparator<Person>{
    @Override
    public int compare(Person o1, Person o2) {
        // implementace porovnání
    }
}
```

Toto rozhraní po nás vyžaduje implementaci metody (opět s podobným názvem) `compare()`, která ale přijímá dva parametry - oba objekty, které se mají porovnávat mezi sebou[\[30\]](https://word2md.com/#footnote-30). Funkce opět vrací celočíselnou hodnotu, jejíž význam je stejný jako u metody `compareTo()`:

* Hodnotu menší než 0 - pokud je první objekt menší než druhý objekt;
* Hodnotu rovnu 0 - pokud jsou oba objekty shodné;
* Hodnotu větší než 0 - pokud je první objekt větší druhý objekt.

Implementaci již známe z předchozí kapitoly, pouze ji upravíme o správné objekty.

```java
class PersonByAgeComparator implements java.util.Comparator<Person> {
    @Override
    public int compare(Person o1, Person o2) {
        if (o1.getAge() < o2.getAge()) {
            return -1;
        } else if (o1.getAge() == o2.getAge()) {
            return 0;
        } else {
            return 1;
        }
    }
}
```

Povšimněte si, že protože již nepíšeme kód do třídy `Person`, ale do nové třídy, nedostaneme se na soukromý člen třídy `age` a musíme do třídy `Person` doplnit a následně využít volání getteru `getAge()`.

Zbývá ukázka, jak takovéto porovnání použít.

```java
java.util.List<Person> byName = new java.util.ArrayList();
java.util.List<Person> byAge = new java.util.ArrayList();
Person p;
// naplnění obou kolekcí stejnými objekty
p = new Person("Petra", 20);
byName.add(p);
byAge.add(p);
p = new Person("Iva", 18);
byName.add(p);
byAge.add(p);
p = new Person("Brad", 45);
byName.add(p);
byAge.add(p);
// nativní řazení
Collections.sort(byName);
// řazení pomocí vlastního komparátoru
Collections.sort(byAge, new PersonByAgeComparator());
// výpis řazený podle jména
System.out.println("By name:");
for(Person it : byName)
    System.out.println(it.getName());
// výpis řazený podle věku
System.out.println("By age:");
for(Person it : byAge)
    System.out.println(it.getName());
}
```

Opět jsme doplnili do třídy `Person` getter pro hodnotu jména `getName()`. Tučné řádky ukazují způsob seřazení kolekce - v prvním případě s využitím nativního řazení a metody `compareTo()` přes rozhraní `Comparable`, druhý případ ukazuje řazení s využitím instance vlastního komparátoru.

```
run:
By name:
Brad
Iva
Petra
By age:
Iva
Petra
Brad
BUILD SUCCESSFUL (total time: 1 second)
```

Je zřejmé, že takovýchto tříd - komparátorů - si můžeme do projektu doplnit libovolný počet.

## Více komparátorů třídy

V dnešní době lambda výrazů a Java Stream API se již vlastní komparátory moc nepoužívají, ale pokud byste je z nějakého důvodu potřebovali, zjistíte, že v projektu najednou děláte spoustu nových tříd. \
Proto je vhodné jim dát nějaký systém. Za mne je dávám jako vnitřní třídy původní třídy, takže pokud pak hledám nějaký konkrétní komparátor, začnu vždy názvem třídy + tečka + `by...` podle hledané vlastnosti. Nástřel třídy vypadá takto:

```java
import java.util.Comparator;

public class Person {
    private final int id;
    private final String name;
    private final String surname;
    private final int age;

    public Person(int id, String name, String surname, int age) {
        this.id = id;
        this.name = name;
        this.surname = surname;
        this.age = age;
    }

    public int getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public String getSurname() {
        return surname;
    }

    public int getAge() {
        return age;
    }

    public static class ByIdComparer implements Comparator<Person> {
        @Override
        public int compare(Person p1, Person p2) {
            return Integer.compare(p1.getId(), p2.getId());
        }
    }

    public static class ByNameComparer implements Comparator<Person> {
        @Override
        public int compare(Person p1, Person p2) {
            return p1.getName().compareTo(p2.getName());
        }
    }

    public static class BySurnameComparer implements Comparator<Person> {
        @Override
        public int compare(Person p1, Person p2) {
            return p1.getSurname().compareTo(p2.getSurname());
        }
    }

    public static class ByAgeComparer implements Comparator<Person> {
        @Override
        public int compare(Person p1, Person p2) {
            return Integer.compare(p1.getAge(), p2.getAge());
        }
    }
}
```

A vlastní volání pak takto:

```java
List<Person> people = new ArrayList<>();
people.add(new Person(3, "Jan", "Novak", 25));
people.add(new Person(1, "Eva", "Cerna", 30));
people.add(new Person(4, "Adam", "Bily", 20));
people.add(new Person(2, "Marie", "Dvorakova", 35));

// Použití ByIdComparer
people.sort(new Person.ByIdComparer());  // <--- zde
System.out.println("Seřazeno podle ID:");
printList(people);

// Použití ByNameComparer
people.sort(new Person.ByNameComparer());  // <--- zde
System.out.println("\nSeřazeno podle jména:");
printList(people);

// Použití BySurnameComparer
people.sort(new Person.BySurnameComparer());  // <--- zde
System.out.println("\nSeřazeno podle příjmení:");
printList(people);

// Použití ByAgeComparer
people.sort(new Person.ByAgeComparer());  // <--- zde
System.out.println("\nSeřazeno podle věku:");
printList(people);
```

## Směr řazení ve vlastním komparátoru

Pro volbu směru řazení stačí v konstruktoru komparátoru přidat parametr udávající chování (například `enum`). Dle něj lze pak následně řídit chování:

```java
import java.util.Comparator;

public static class BySurnameComparer implements Comparator<Person> {
    public enum Direction {
        ASC, DESC
    }

    private final Direction direction;

    public BySurnameComparer(Direction direction) {
        this.direction = direction;
    }

    @Override
    public int compare(Person p1, Person p2) {
        int result = p1.getSurname().compareTo(p2.getSurname());
        return direction == Direction.DESC ? -result : result;
    }
}
```

Následné volání:

```java
people.sort(new Person.BySurnameComparer(
    Person.BySurnameComparer.Direction.DESC));
System.out.println("Sestupně podle příjmení:");
printList(people);

// Řazení vzestupně (A-Z)
people.sort(new Person.BySurnameComparer(
    Person.BySurnameComparer.Direction.ASC));
System.out.println("\nVzestupně podle příjmení:");
printList(people);
```
