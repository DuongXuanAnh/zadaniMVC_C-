
V této úloze si vyzkoušíme práci s šablonovým systémem. Vašim úkolem je napsat kompilátor, který přeloží HTML šablonu z níže definovaného formátu do PHP-interleaved formátu (tj. do PHP souboru, který může být přímo použit jako šablona). Cílem tohoto systému je nabídnout lépe čitelný formát (než interleaved PHP), zajistit kontrolu uzávorkování řídicích struktur a automatickou sanitizaci dat při vkládání do výstupu. Naopak nechceme přijít o výhody, které nám interleaved PHP dává (efektivní zpracování, použití výrazů při vkládání dat, ...).

### Formát šablon

Šablona je textový (typicky HTML) soubor, který může obsahovat následující značky:

-   `{= expr}`  - vloží výsledek výrazu bezpečně do HTML (tedy je ekvivalentní s PHP konstrukcí  `<?= htmlspecialchars(expr) ?>`)
-   `{if cond}`  a  `{/if}`  - obsah mezi těmito dvěma značkami je vypsán, pouze pokud je splněna podmínka  `cond`  (je ekvivalentní s  `<?php if (cond) { ?>`  a  `<?php } ?>`)
-   `{for expr}`  a  `{/for}`  - obsah mezi těmito dvěma značkami je iterován dle výrazu for cyklu (je ekvivalentní s  `<?php for (expr) { ?>`  a  `<?php } ?>`)
-   `{foreach expr}`  a  `{/foreach}`  - obsah mezi těmito dvěma značkami je iterován dle výrazu foreach cyklu (je ekvivalentní s  `<?php foreach (expr) { ?>`  a  `<?php } ?>`)

Výrazy uvnitř značek (zastoupené symboly  `expr`  a  `cond`) šablonový systém nijak neinterpretuje (přímo je zkopíruje do odpovídajících PHP struktur). Jediná podmínka je, že uvnitř těchto výrazů se nesmí nacházet  `{`  a  `}`  a že jsou neprázdné. Párové značky (`if`,  `for`  a  `foreach`) je možné libovolně vnořovat, ale počáteční-koncové značky musí mít správné uzávorkování.

Pokud se kdekoli mimo značky objeví znaky  `{`  nebo  `}`, musí být korektně zkopírovány do výstupu. Tedy např. text  `{four}`  se zkopíruje bez změny, protože se nejedná o platnou značku. Naopak  `{for }`  je platná značka, ale s prázdným výrazem, takže šablonový systém nahlásí chybu při kompilaci.

### Rozhraní

Stáhněte si  [startovací balíček](https://recodex.mff.cuni.cz/api/v1/uploaded-files/5d87a379-27a1-11eb-8e81-005056ad4f31/download), ve kterém (kromě ukázky šablony a jejího použití) naleznete soubor  `templator.php`. Soubor obsahuje třídu  `Templator`  s již předpřipravenými veřejnými metodami, které máte dodržet. Tento soubor je také  **jediný, který odevzdáváte do ReCodExu**. Metoda  `loadTemplate()`  načte vstupní šablonu ze souboru, ale nijak ji neinterpretuje. Metoda  `compileAndSave()`  provede kompilaci šablony a výsledek uloží do daného souboru. Volání  `compileAndSave()`  bez načtení šablony je považováno za chybu. Všechny chyby reportujte výhradně vyhozením výjimky. Výjimka musí být třídy  `Exception`  nebo odvozené, konkrétní chybové hlášky nejsou testovány ReCodExem.

Při řešení můžete deklarovat libovolné další třídy nebo funkce, ale nepište žádný výkonný kód (kód který je přímo spuštěn při inkluzi souboru  `templator.php`) ani nemodifikujte globální proměnné uvnitř metod. Rovněž nesmíte cokoli vypisovat na standardní výstup (ani chybové hlášky). Pro ladění můžete použít skript  `compile.php`  (volaný jako CLI), který provede kompilaci pomocí vaší třídy  `Templator`  (obdobně bude testováno v ReCodExu).
