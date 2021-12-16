
Cílem této úlohy je implementovat jednoduchý formulář pro vyplňování výkazů práce. Formulář je navržen jako jednostránková aplikace (SPA) a všechna datová komunikace probíhá pomocí AJAX dotazů cílených na REST-like API.

Stáhněte si  [startovací balíček](https://recodex.mff.cuni.cz/api/v1/uploaded-files/91230d51-f746-11e8-b0fd-00505601122b/download), který již implementuje uživatelské rozhraní. Vašim úkolem je implementovat vrstvu, která má na starosti datovou komunikaci (datový model). Tato vrstva je realizována třídou  `DataModel`  v souboru  `solution.js`. Jiné části aplikace neupravujte a do ReCodExu  **odevzdávejte pouze výsledný soubor  `solution.js`**.

### Rozhraní datového modelu

Objekt třídy  `DataModel`  reprezentuje jeden datový endpoint (reprezentovaný jedním URL), se kterým aplikace komunikuje. URL endpointu (přesněji URL prefix, kterému chybí část  _query_) je předáno jako jediný parametr konstruktoru. Vedle funkcí pro práci s daty v sobě objekt datového modelu ukrývá také transparentní cache.

Metoda  `getData()`  má za úkol načtení dat. Jako jediný argument dostane  `callback`  funkci, kterou zavolá, jakmile jsou data k dispozici, nebo pokud dojde k chybě. V případě kladného výsledku je  `callback`  zavolán s jedním argumentem -- platnými daty. V opačném případě dostane argumenty dva -- první (na místě dat) je  `null`, druhý je řetězec s chybovou hláškou.

Data jsou vrácena jako pole objektů, kde každý objekt má položky  `id`,  `caption`  a  `hours`, tedy unikátní identifikátor záznamu, textový popisek kategorie a počet vyplněných hodin. REST API bylo ovšem původně navrženo tak, že seznam kategorií a hodiny pro jednotlivé kategorie vrací zvlášť. Budete tedy muset provést více volání a výsledky korektně složit dohromady.

Po úspěšném stažení dat z REST API musí datový model uložit tato data do svojí cache. Při následných voláních metody  `getData`  jsou pak data vrácena z cache, a to okamžitým synchronním zavoláním  `callback`  funkce.

Metoda  `invalidate()`  provede smazání interní cache. První následující volání  `getData()`  po zavolání  `invalidate()`  musí provést asynchronní HTTP požadavky na REST API a znovu načíst data.

Poslední metoda  `setHours()`  dostane  `id`  záznamu a novou hodnotu vyplněných hodin (`hours`) a pomocí asynchronního volání REST API provede požadovanou změnu. Pokud je změna úspěšná, upraví interní záznamy v cache a zavolá  `callback`, který může být volitelně předán jako třetí argument. V případě chyby je opět zavolán  `callback`  s jediným argumentem -- řetězcem s chybovou hláškou.

Při volání obou asynchronních metod (s callbackem) je garantováno, že na objektu datového modelu nebudou volány žádné metody, dříve, než je zavolán samotný callback. Volání metod datového modelu uvnitř callbacku však možné je.

### REST API

REST API je reprezentováno jedním URL, přičemž volání různých funkcí jsou rozlišena pomocí URL (query) parametrů a metody HTTP požadavku. API vrací vždy JSON kolekci (objekt), který vždy obsahuje položku  `ok`  typu  `boolean`, určující, zda volání proběhlo v pořádku, nebo nastala chyba. Pokud nastala chyba, může kolekce volitelně ještě obsahovat položku  `error`  s řetězcem chybové hlášky. V případě kladného výsledku jsou vrácená data v položce  `payload`.

API podporuje celkem 3 funkce:

-   `GET`  požadavek bez dalších parametrů: Vrátí všechny kategorie jako pole objektů, kde každý objekt má položky  `id`  (unikátní identifikátor), a  `caption`  (textový popisek).
-   `GET`  požadavek s query parametry  `action=hours`  a  `id=<id>`, kde  `<id>`  je identifikátor příslušného záznamu (např.  `?action=hours&id=42`). Vrátí objekt, obsahující položky  `id`,  `caption`  a  `hours`, první dvě jsou identické s tím, co vrací první funkce API, položka  `hours`  obsahuje vyplněné hodiny pro danou kategorii.
-   `POST`  požadavek s query parametry  `action=hours`,  `id=<id>`  a  `hours=<hours>`, kde  `<hours>`  je nový počet hodin pro záznam s identifikátorem  `<id>`. Přestože se jedná o POST dotaz, žádná data nejsou přenášená v těle požadavku. volání této funkce navíc nevrací žádný  `payload`  (ve výsledném JSONu je pouze  `ok`  a v případě chyby  `error`).

**Upozornění:**  Chybové hlášky (řetězce), které vrací REST API v položce  `error`  je třeba předávat do  `callback`  funkce v rozhraní datového modelu nezměněné, aby bylo možné je korektně testovat v ReCodExu.

### Testování a ladění

Vaše řešení bude v ReCodExu testování  [CLI skriptem](https://recodex.mff.cuni.cz/api/v1/uploaded-files/91231466-f746-11e8-b0fd-00505601122b/download), který používá mock funkce  [`fetch()`](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)  a práci se standardními  [`Promise`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)  objekty. Ve vašem řešení tedy nepoužívejte starší  `XMLHTTPRequest`  ani jiná AJAX API (tedy ani jQuery, přestože je ve startovacím balíčku použito). Zmíněný mock neimplementuje všechny funkce (např. nepodporuje  `AbortController`) a je třeba jej používat pokud možno přímočaře (nesnažte se řešit detaily, které zadání explicitně nevyžaduje).

Pro účely ladění v prohlížeči (s použitím startovacího kódu) je potřeba webový server s podporou PHP (7.x). Pravděpodobně nejsnazší řešení je použít váš účet na  [webíku](https://webik.ms.mff.cuni.cz/)  a kód ladit přímo tam. Nezapomeňte nastavit práva souboru  `data.json`  v  `restapi`, aby do něho mohl webserver zapisovat. Alternativně můžete zkusit nainstalovat vlastní lokální server s PHP (např. pomocí WAMP/LAMP balíčků). Třetí možnost je ladit pouze s použitím CLI skriptu a  [Node.js](https://nodejs.org/en/).
