
Napište aplikaci, která bude základem e-shopu internetového knihkupectví Nežárka.NET - vaše aplikace bude představovat implementaci logiky na straně webového serveru. Po spuštění obdrží aplikace na standardním vstupu textový popis dat e-shopu (tedy vlastně reprezentaci dat z relační databáze) v následujícím formátu (každý záznam je obsažen na samostatném řádku, jednotlivé položky každého záznamu jsou oddělené 1 středníkem; první položka na řádku označuje typ záznamu, resp. začátek/konec seznamu datových záznamů):

```
DATA-BEGIN
BOOK;_BookId_;_BookTitle_;_BookAuthor_;_BookPrice_
CUSTOMER;_CustId_;_CustName_;_CustSurname_
CART-ITEM;_CustId_;_BookId_;_BookCount_
DATA-END

```

Veškerá data čtená aplikací ze standardního vstupu jsou case sensitive, tedy velikost písmen je rozhodující!

Na vstupu jsou nejprve všechny záznamy typu BOOK, pak následují všechny záznamy typu CUSTOMER, pak následují všechny záznamy typu CART-ITEM. Každého typu záznamu může být na vstupu libovolné množství (i nula).

Záznam BOOK popisuje informaci o jedné knize, kterou obchod Nežárka.NET nabízí. Záznam CUSTOMER popisuje informace o jednom zaregistrovaném zákazníkovi. Každý zákazník má k sobě přiřazen právě jeden virtuální nákupní košík. Záznam CART-ITEM popisuje informace o jedné položce v košíku nějakého zákazníka (pro nějakou dvojici CustId a BookId je na vstupu maximálně jeden záznam CART-ITEM). Pokud je zákazníkův košík prázdný, tak se na vstupu nevyskytuje žádný záznam CART-ITEM s CustId rovné Id tohoto zákazníka. BookCount reprezentuje počet kusů dané knihy, které má zákazník ve svém košíku. Můžete počítat, že k dispozici máte O(n) paměti, když n je celkový počet všech záznamů na vstupu (resp. celkový počet všech záznamů v každém okamžiku reprezentovaný v programu) - tedy se očekává, že veškerá data o knihách, zákaznících a nákupních košících budete mít neustále v paměti.

-   BookId, CustId, BookCount, BookPrice jsou libovolná cela nezáporná čísla (vejdou se do typu int)
-   BookTitle, BookAuthor, CustName, CustSurname jsou libovolné textové řetězce (včetně mezer), které nebudou obsahovat žádný výskyt znaku středník nebo nový řádek.
-   BookId, ani CustId nemusí být souvislá posloupnost hodnot.

Pokud je ve vstupních datech libovolná chyba (text na místě čísla, neznámé klíčové slovo, apod.), tak program na standardní výstup vypíše:

```
Data error.

```

a skončí.

Základní datový model, který reprezentuje výše popsaná data v C# programu, najdete zde:  [NezarkaModel.cs](https://recodex.mff.cuni.cz:4000/v1/uploaded-files/9dc5bee4-aba7-11e7-a937-00505601122b/download)  
Tento kód použijte ve své aplikaci. Kód můžete i dále upravovat a rozšiřovat (např. o části business logiky, která bude popsána dále).

```
----

```

Po načtení vstupních dat dostává aplikace na standardním vstupu příkazy od klientů systému (jejich webových prohlížečů), kde na každém řádku je vždy jeden příkaz. Po ukončení standardního vstupu dojde k ukončení celé aplikace. Příkazy se zpracovávají jeden po druhém, výsledek zpracování každého příkazu se vypíše ve formě HTML na standardní výstup. Výstup každého jednotlivého vstupního příkazu je na standardním výstupu ukončen samostatným řádkem, který obsahuje pouze 4 znaky rovnítko (====).

Implementace obchodu Nežárka.NET musí podporovat následujících 5 typy příkazů:

-   `GET _CustId_ http://www.nezarka.net/Books`
-   `GET _CustId_ http://www.nezarka.net/Books/Detail/_BookId_`
-   `GET _CustId_ http://www.nezarka.net/ShoppingCart`
-   `GET _CustId_ http://www.nezarka.net/ShoppingCart/Add/_BookId_`
-   `GET _CustId_ http://www.nezarka.net/ShoppingCart/Remove/_BookId_`

Jako výsledek těchto příkazů se generuje jedna z následujících stránek (více viz šablony níže):

-   SeznamKnih ([02-Books.html](https://recodex.mff.cuni.cz:4000/v1/uploaded-files/9dc5a9c0-aba7-11e7-a937-00505601122b/download)) - obsahuje společné záhlaví; informace o knihách se zobrazují v HTML tabulce vždy 3 knihy na řádek (pouze u posledního řádku je zleva jen tolik knih, kolik jich zbývá); informace o knihách se zobrazují od první do poslední vždy v pořadí, jak jsou uložené v datovém modelu (pořadí v jakém je vrací vzorová implementace metody  `GetBooks()`), postupně zleva doprava a shora dolů; pokud je seznam knih prázdný, ve výstupním HTML dokumentu bude na místě dle šablony prázdný HTML element  `<table>`, tedy nebude v něm ani jedna ze vzorových sekcí  `<tr>`  a  `<td>`:

```
...  
<table>  
</table>  
...

```

-   InformaceOKnize (šablona/příklad:  [03-BooksDetail.html](https://recodex.mff.cuni.cz:4000/v1/uploaded-files/9dc5a418-aba7-11e7-a937-00505601122b/download)) - obsahuje společné záhlaví
-   ObsahKošíku (šablona/příklad:  [04-ShoppingCart.html](https://recodex.mff.cuni.cz:4000/v1/uploaded-files/9dc5b4ad-aba7-11e7-a937-00505601122b/download)) - obsahuje společné záhlaví (vždy zobrazuje stav nákupního košíku po provedení příkazu); pokud košík neobsahuje žádné položky, zobrazí se místo tabulky položek zvláštní informace (šablona/příklad:  [05-ShoppingCart-Empty.html](https://recodex.mff.cuni.cz:4000/v1/uploaded-files/9dc59ea3-aba7-11e7-a937-00505601122b/download)).
-   ChybnýPříkaz (šablona/příklad:  [09-InvalidRequest.html](https://recodex.mff.cuni.cz:4000/v1/uploaded-files/9dc5b9e4-aba7-11e7-a937-00505601122b/download)) - _NE_obsahuje společné záhlaví (ale je validní HTML soubor)
-   Společné záhlaví obsahuje křestní jméno aktuálního zákazníka (podle CustId), a menu s odkazy na příkazy /Books a /ShoppingCart (u tohoto příkazu se zobrazuje aktuální počet položek v košíku aktuálního zákazníka - dle CustId).

Příkazy mají následující význam:

-   `/Books`  - beze změny dat, zobrazí SeznamKnih
-   `/Books/Detail/_BookId_`  - beze změny dat, zobrazí InformaceOKnize pro knihu BookId
-   `/ShoppingCart`  - beze změny dat, zobrazí ObsahKošíku pro aktuálního zákazníka (dle CustId)
-   `/ShoppingCart/Add/_BookId_`  - přídá 1 položku (1 kus) BookId do nákupního košíku aktuálního zákazníka (dle CustId); pokud kniha BookId již v košíku je, tak se jen zvětší počet kusů o 1; zobrazí ObsahKošíku
-   `/ShoppingCart/Remove/_BookId_`  - odebere 1 položku (1 kus) BookId z nákupního košíku (tj. zmenší počet kusů o 1); pokud je kniha BookId v košíku jen jednou, tak z košíku odebere celý záznam pro tuto knihu; zobrazí ObsahKošíku

Pokud je libovolná část příkazu neplatná (např. jiný typ příkazu než GET, špatný formát příkazu, neplatné číslo zákazníka, neplatné číslo knihy, odebírání knihy která není v košíku zákazníka, apod.), tak se zobrazí ChybnýPříkaz.

**POZOR:**  Formát generovaných HTML souborů musí přesně odpovídat výše uvedeným šablonám.

V tomto archivu  [Example.zip](https://recodex.mff.cuni.cz:4000/v1/uploaded-files/9dc5c4a9-aba7-11e7-a937-00505601122b/download)  najdete příklad vstupu (soubor NezarkaTest.in) a k němu odpovídajícího výstupu (soubor NezarkaTest.out). Pro přehlednost a možnost vyzkoušení jsou v archivu přibaleny i soubory 01.html až 11.html, které obsahují výstup NezarkaTest.out "rozsekaný" na výstupy jednotlivých příkazů (zakončovací ==== jsou zde odebrány). Upozornění: vaším úkolem není takové soubory generovat, vaše aplikace vypisuje vše jen na standardní výstup ve formátu odpovídajícímu NezarkaTest.out.

**HINT:**  Při objektovém návrhu vaší aplikace se zkuste zamyslet nad možností použití varianty návrhového vzoru  _Model-View-Controller_  (MVC).

```
----

```

**OTÁZKA:**  Jak v programu reprezentovat a vypisovat HTML kód?

**ODPOVĚĎ:**  Asi nejjednodušší a přehledná varianta je mít pro každý řádek výstupního souboru jedno volání  `.WriteLine()`  na vhodném "writeru". Pokud bychom např. měli naprogramovat aplikaci, která vždy na standardní výstup vrátí HTML dokument s aktuálním datem a časem dle následující šablony/příkladu ([TimeServiceExample.out.html](https://recodex.mff.cuni.cz:4000/v1/uploaded-files/9dc59668-aba7-11e7-a937-00505601122b/download)):

```
<!DOCTYPE html>
<html lang="en" xmlns="http://www.w3.org/1999/xhtml">
<head>
    <meta charset="utf-8" />
    <title>Nezarka.NET: Also a Time Service!</title>
</head>
<body>
    <style type="text/css">
        pre {
            line-height: 70%;
        }
    </style>
    <h1><pre>  v,<br />Nezarka.NET: Also a Time Service!</pre></h1>
    It is 21. 10. 2014 16:20:22 on our server.<br />
    Enjoy!
</body>
</html>

```

tak například následující zdrojový soubor ([TimeServiceProgram.cs](https://recodex.mff.cuni.cz:4000/v1/uploaded-files/9dc5af79-aba7-11e7-a937-00505601122b/download)) by mohl být rozumnou (pro tuto úlohu přijatelnou) implementací:

```
using System;
using System.Collections.Generic;

namespace NezarkaTimeService {
	class Program {
		static void Main(string[] args) {
			Console.WriteLine("<!DOCTYPE html>");
			Console.WriteLine("<html lang=\"en\" xmlns=\"http://www.w3.org/1999/xhtml\">");
			Console.WriteLine("<head>");
			Console.WriteLine("    <meta charset=\"utf-8\" />");
			Console.WriteLine("    <title>Nezarka.NET: Also a Time Service!</title>");
			Console.WriteLine("</head>");
			Console.WriteLine("<body>");
			Console.WriteLine("    <style type=\"text/css\">");
			Console.WriteLine("        pre {");
			Console.WriteLine("            line-height: 70%;");
			Console.WriteLine("        }");
			Console.WriteLine("    </style>");
			Console.WriteLine("    <h1><pre>  v,<br />Nezarka.NET: Also a Time Service!</pre></h1>");
			Console.WriteLine("    It is " + DateTime.Now + " on our server.<br />");
			Console.WriteLine("    Enjoy!");
			Console.WriteLine("</body>");
			Console.WriteLine("</html>");
		}
	}
}
```
