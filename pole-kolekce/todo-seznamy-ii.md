# TODO Seznamy II

#### Seznamy

Seznam je typ kolekce, který obsahuje sadu do něj vložených prvků. Prvky lze postupně po jednom nebo hromadně přidávat, lze je odebírat, a hlavně - lze na ně přistupovat pomocí indexu. Prvky se v seznamu mohou vyskytovat vícekrát, seznam může i obsahovat hodnotu _null_.

Rozhraní _List_ nabízí tedy sadu metod pro práci s jednotlivými prvky. Některé z operací lze také provádět hromadně[\[27\]](https://word2md.com/#footnote-27). List lze také procházet pomocí cyklu for-each.

| Signatura funkce            | Popis                                                                                           |
| --------------------------- | ----------------------------------------------------------------------------------------------- |
| boolean add(E e)            | Přidá prvek na konec seznamu.                                                                   |
| void add(int index, E e)    | Přidá prvek na pozici v seznamu definovanou parametrem _index_.                                 |
| void clear()                | Odstraní všechny prvky ze seznamu.                                                              |
| boolean isEmpty()           | Vrací true, pokud je seznam prázdný.                                                            |
| boolean contains(Object o)  | Vrací true, pokud seznam obsahuje prvek předaný jako parametr.                                  |
| E get(int index)            | Vrací prvek na pozici parametru _index_.                                                        |
| int indexOf(Object o)       | Vrací index prvního výskytu prvku předaného jako parametr.                                      |
| int lastIndexOf(Object o)   | Vrací index posledního výskytu prvku předaného jako parametr.                                   |
| E remove(int index)         | Odstraní prvek ze zadaného indexu a vrací odstraněný prvek jako výsledek volání.                |
| boolean remove(Object o)    | Odstraní prvek předaný jako parametr.                                                           |
| E set(int index, E element) | Nahradí prvek na indexu novou hodnotou. Nahrazovanou hodnotu vrací jako výsledek volání funkce. |
| int size()                  | Vrací počet prvků v seznamu.                                                                    |

Nejčastěji používanou implementací je třída _ArrayList_, která již podle názvu reprezentuje „chytré" pole - objekt, který se chová jako pole, u kterého se ale programátor nemusí starat o kontrolu délky při operaci s prvky, zejména při přidávání nového prvku.

**Poznámka.** U kolekcí se v Javě asi nejvíce používá programování proti rozhraní - vytvoří se instance od konkrétního typu, ale přiřadí se do proměnné svého předka/rozhraní a další operace se volají nad tímto předkem/rozhraním.

Rychlý příklad vytvoření a použití listu lze vidět v kapitole 7.2.1, která demonstruje použití generického listu.

**Další operace se seznamy - třída Collections**

Třída _java.util.List_ - tedy seznam - neobsahuje téměř žádné operace, které by byly běžné a bylo je možné s daným seznamem provádět, jako například řazení, kopírování, otáčení apod. Všechny tyto operace lze nalézt v další třídě z Java Collection Frameworku, nazvané _Collections_[_\[28\]_](https://word2md.com/#footnote-28). Tato třída obsahuje statické metody umožňující provádět operace s kolekcemi, zejména[\[29\]](https://word2md.com/#footnote-29):

* **binarySearch()** - pro vyhledání indexu určitého prvku v seznamu prvků. Pozor, jedná se (už podle názvu) o vyhledávání binární a proto musí být seznam nejdříve setřízen (pomocí metody **sort()**);
* **copy()** - zkopíruje všechny prvky z jednoho seznamu do seznamu druhého;
* **max()** - vrací největší prvek seznamu podle nativního řazení (kapitola 7.2.4);
* **min()** - vrací nejmenší prvek seznamu podle nativního řazení;
* **replaceAll()** - nahradí v seznamu všechny výskyty určitého prvku prvkem jiným;
* **reverse()** - otočí pořadí prvků v seznamu;
* **shuffle()** - náhodně zpřehazuje prvky v seznamu;
* **sort()** - setřídí seznam podle parametrů (ukázka použití viz kapitola 7.2.6);
* **swap()** - přehodí prvky na dvou indexech.

Jak bylo zmíněno, všechny metody jsou statické a vyžadují tak typicky jako první parametr seznam (instanci libovolného potomka třídy _java.util.List_), nad kterým se daná operace bude provádět.

Následuje velmi krátký příklad demonstrující seřazení seznamu čísel od největšího k nejmenšímu s použitím třídy _Collections_.

List\<Integer> lst = new ArrayList();

lst.add(5);

lst.add(8);

lst.add(3);

lst.add(1);

// seřazení od nejmenšího k největšímu

// pomocí nativního řazení a metody "sort()"

Collections.sort(lst);

// otočení pořadí prvků, čímž bude list

// seřazen od největšího k nejmenšímu

Collections.reverse(lst);

for(Integer value : lst)

System.out.println(value);

A očekávaný výstup:

run:

8

5

3

1

BUILD SUCCESSFUL (total time: 24 seconds)
