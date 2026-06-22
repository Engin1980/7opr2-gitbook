# TODO Comparator

#### Nenativní porovnání instancí tříd - rozhraní Comparator

Při vytváření předchozího příkladu si lze povšimnout, že můžeme definovat, zda se třída _Person_ bude porovnávat podle věku, nebo podle jména, ale **nelze definovat obojí**, protože funkce _compareTo()_ může být v třídě _Person_ pouze jedna.

Pokud budeme tedy mít dva seznamy stejných osob a oba setřídíme pomocí volání metody _sort()_, bude pořadí jejich prvků shodné. Co ale, pokud chceme jeden seznam řadit podle věku a druhý podle příjmení?

Pokud tedy chceme instance určité třídy porovnávat více způsoby, musíme použít jiný přístup, který je založen na implementaci rozhraní s podobným názvem, ale odlišnou funkcionalitou - _java.lang.Comparator_. Nezaměňujte názvy těchto rozhraní (_Comparable_ vs. _Comparator_)!

Pro každý způsob řazení, který chceme realizovat (samozřejmě vyjma již případně realizovaného nativního řazení pomocí rozhraní _Comparable_), nejdříve vytvoříme samostatnou třídu, která bude implementovat rozhraní _Comparator_.

Uvažujme opět příklad s třídou _Person_, kdy jsme realizovali nativní řazení pomocí porovnávání hodnot jména (poslední výpis).

Pokud budeme také chtít řadit osoby podle věku, musíme vytvořit novou, samostatnou třídu, která bude implementovat požadované rozhraní _Comparator_.

class PersonByAgeComparator implements java.util.Comparator\<Person>{

@Override

public int compare(Person o1, Person o2) {

// implementace porovnání

}

}

Toto rozhraní po nás vyžaduje implementaci metody (opět s podobným názvem) _compare()_, která ale přijímá dva parametry - oba objekty, které se mají porovnávat mezi sebou[\[30\]](https://word2md.com/#footnote-30). Funkce opět vrací celočíselnou hodnotu, jejíž význam je stejný jako u metody _compareTo()_:

* Hodnotu menší než 0 - pokud je první objekt menší než druhý objekt;
* Hodnotu rovnu 0 - pokud jsou oba objekty shodné;
* Hodnotu větší než 0 - pokud je první objekt větší druhý objekt.

Implementaci již známe z předchozí kapitoly, pouze ji upravíme o správné objekty.

class PersonByAgeComparator implements java.util.Comparator\<Person> {

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

Povšimněte si, že protože již nepíšeme kód do třídy _Person_, ale do nové třídy, nedostaneme se na soukromý člen třídy _age_ a musíme do třídy _Person_ doplnit a následně využít volání getteru _getAge()_.

Zbývá ukázka, jak takovéto porovnání použít.

java.util.List\<Person> byName = new java.util.ArrayList();

java.util.List\<Person> byAge = new java.util.ArrayList();

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

**Collections.sort(byName);**

// řazení pomocí vlastního komparátoru

**Collections.sort(byAge, new PersonByAgeComparator());**

// výpis řazený podle jména

System.out.println("By name:");

for(Person it : byName)

System.out.println(it.getName());

// výpis řazený podle věku

System.out.println("By age:");

for(Person it : byAge)

System.out.println(it.getName());

}

Opět jsme doplnili do třídy _Person_ getter pro hodnotu jména _getName()_. Tučné řádky ukazují způsob seřazení kolekce - v prvním případě s využitím nativního řazení a metody _compareTo()_ přes rozhraní _Comparable_, druhý případ ukazuje řazení s využitím instance vlastního komparátoru.

run:

**By name:**

Brad

Iva

Petra

**By age:**

Iva

Petra

Brad

BUILD SUCCESSFUL (total time: 1 second)

Je zřejmé, že takovýchto tříd - komparátorů - si můžeme do projektu doplnit libovolný počet.
