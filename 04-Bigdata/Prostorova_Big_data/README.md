# Prostorová Big Data (Geospatial Analytics)

**Autoři:** čet. Patrik Panský, čet. Ondřej Hanák

## Úvod do prostorových Big Dat
* Prostorová data nejsou tvořena pouze čísly a texty, ale jde o záznamy nesoucí kontext místa a času.
* Příkladem takového záznamu jsou GPS souřadnice, tedy zeměpisná šířka (např. 49.195), zeměpisná délka (např. 16.606) a časový údaj (např. 07:35:00).
* V moderním světě vznikají každý den miliony nových záznamů.
* Zdrojem těchto lokačních dat jsou běžně používaná zařízení a služby, jako je každý chytrý telefon, auta s navigací, systémy hromadné dopravy, sdílené koloběžky nebo i bankovní transakce.
* Cílem této prezentace a ukázky je představit, jak je možné se na tyto obrovské sady dat napojit a jak je zpracovat rovnou v prohlížeči.
* Na příkladu dopravních nehod v ČR si lze ukázat, jak z jinak nepřehledné databáze vytvořit interaktivní 3D model.

---

## Proč selhávají běžné mapy?
* Běžné mapové podklady, mezi které patří například Google Maps API, vykreslují každý jednotlivý bod jako samostatný objekt (DOM element) v paměti webového prohlížeče.
* Při pokusu o vykreslení více než 10 000 bodů se paměť prohlížeče zahltí, což vede k jeho pádu.
* I v případě, že by prohlížeč dokázal body zpracovat, naskládání milionu teček přes sebe vytvoří vizuální chaos plný nečitelných barevných fleků.
* Analýza Big Dat proto vyžaduje změnu přístupu, protože cílem není hledat jeden konkrétní taxík nebo jednu specifickou nehodu.
* K efektivní analýze potřebujeme nástroj, který dokáže data na pozadí matematicky agregovat (sečíst) a vizualizovat širší trendy.

---

## Technologie Kepler.gl a WebGL
* Systém Kepler.gl je open-source nástroj, u jehož zrodu stáli inženýři ze společnosti Uber.
* Vznikl z potřeby vizualizovat v reálném čase celosvětový pohyb statisíců aut.
* Nástroj využívá trik s technologií WebGL, díky kterému nefunguje jako běžná webová stránka, ale obchází hlavní procesor (CPU) počítače.
* Miliony zpracovávaných souřadnic posílá přímo ke zpracování do grafické karty (GPU).
* Grafické karty jsou z počítačových her hardwarově uzpůsobené pro paralelní výpočty a dokáží plynule vykreslovat miliony pixelů za vteřinu.
* Tento technologický přístup umožňuje v Kepler.gl načítat obrovské datasety a plynule jimi rotovat ve 3D zobrazení i na běžném obyčejném notebooku.

---

## Architektura systému – Jak se data dostanou na mapu?

| Fáze | Nástroj | Popis |
| :--- | :--- | :--- |
| **1. Sběr v reálném čase** | **Apache Kafka** | Systém primárně zachytává obrovský proud dat, což mohou být například signály GPS vysílané automobily. Apache Kafka zde funguje jako extrémně robustní roura schopná přijímat statisíce zpráv každou vteřinu bez jakýchkoliv výpadků. |
| **2. Zpracování a čištění** | **Apache Spark** | Vstupní data jsou syrová a běžně obsahují chyby. Apache Spark provádí na clusteru serverů distribuované výpočty k úpravám dat – například filtruje chybné souřadnice hlásící nehodu v oceánu, opravuje formáty času a logicky spojuje tabulky. |
| **3. Masivní úložiště** | **Hadoop HDFS / NoSQL** | Po vyčištění jsou záznamy přesunuty do distribuovaných databází. Tyto databáze, jako je například Cassandra, jsou architektonicky stavěné na velmi rychlé čtení miliard záznamů. |
| **4. Zobrazení dat** | **Kepler.gl** | Během posledního kroku přistupuje datový analytik prostřednictvím systému Kepler.gl přímo do napojené databáze, aby si zpracovaná data vizuálně prohlédl. |

---

## Živá ukázka
Interaktivní demo prostředí nástroje naleznete na následujícím odkazu:
**[https://kepler.gl/demo](https://kepler.gl/demo)**

---

## Závěr
* I když dokážeme do databází, jako je HDFS, úspěšně uložit miliony záznamů, lidský mozek nedokáže z pouhého pohledu do tabulek pochopit širší souvislosti.
* Z toho důvodu je nezbytná Geospatial vizualizace, která efektivně propojuje svět Big Dat s lidským pochopením problému.
* Právě vizualizace dává velkým datům potřebný kontext času a konkrétního místa v prostoru.
