# Prostorová Big Data (Geospatial Analytics)

**Autoři:** čet. Patrik Panský: 2, čet. Ondřej Hanák : 3

## Úvod do prostorových Big Dat : 4
* Prostorová data nejsou tvořena pouze čísly a texty, ale jde o záznamy nesoucí kontext místa a času. : 5
* Příkladem takového záznamu jsou GPS souřadnice, tedy zeměpisná šířka (např. 49.195), zeměpisná délka (např. 16.606) a časový údaj (např. 07:35:00). : 6
* V moderním světě vznikají každý den miliony nových záznamů. : 8
* Zdrojem těchto lokačních dat jsou běžně používaná zařízení a služby, jako je každý chytrý telefon, auta s navigací, systémy hromadné dopravy, sdílené koloběžky nebo i bankovní transakce. : 7
* Cílem tohoto projektu a ukázky je představit, jak je možné se na tyto obrovské sady dat napojit a jak je zpracovat rovnou v prohlížeči. : 9
* Na příkladu dopravních nehod v ČR si lze ukázat, jak z jinak nepřehledné databáze vytvořit interaktivní 3D model. : 9

---

## Proč selhávají běžné mapy? : 10
* Běžné mapové podklady, mezi které patří například Google Maps API, vykreslují každý jednotlivý bod jako samostatný objekt (DOM element) v paměti webového prohlížeče. : 11
* Při pokusu o vykreslení více než 10 000 bodů se paměť prohlížeče zahltí, což vede k jeho pádu. : 12
* I v případě, že by prohlížeč dokázal body zpracovat, naskládání milionu teček přes sebe vytvoří vizuální chaos plný nečitelných barevných fleků. : 13
* Analýza Big Dat proto vyžaduje změnu přístupu, protože cílem není hledat jeden konkrétní taxík nebo jednu specifickou nehodu. : 14
* K efektivní analýze potřebujeme nástroj, který dokáže data na pozadí matematicky agregovat (sečíst) a vizualizovat širší trendy. : 15
* Lze tak například snadno odhalit, které křižovatky jsou z plošného hlediska ty nejnebezpečnější. : 15

---

## Technologie Kepler.gl a WebGL : 16
* Systém Kepler.gl je open-source nástroj, u jehož zrodu stáli inženýři ze společnosti Uber. : 17
* Vznikl z potřeby vizualizovat v reálném čase celosvětový pohyb statisíců aut. : 17
* Nástroj využívá trik s technologií WebGL, díky kterému nefunguje jako běžná webová stránka, ale obchází hlavní procesor (CPU) počítače. : 18
* Miliony zpracovávaných souřadnic posílá přímo ke zpracování do grafické karty (GPU). : 18
* Grafické karty jsou z počítačových her hardwarově uzpůsobené pro paralelní výpočty a dokáží plynule vykreslovat miliony pixelů za vteřinu. : 19
* Tento technologický přístup umožňuje v Kepler.gl načítat obrovské datasety a plynule jimi rotovat ve 3D zobrazení i na běžném obyčejném notebooku. : 20

---

## Architektura systému – Jak se data dostanou na mapu? : 21

| Fáze | Nástroj | Popis |
| :--- | :--- | :--- |
| **1. Sběr v reálném čase** | **Apache Kafka** | Systém primárně zachytává obrovský proud dat, což mohou být například signály GPS vysílané automobily. : 22 Apache Kafka zde funguje jako extrémně robustní roura schopná přijímat statisíce zpráv každou vteřinu bez jakýchkoliv výpadků. : 23 |
| **2. Zpracování a čištění** | **Apache Spark** | Vstupní data jsou syrová a běžně obsahují chyby. : 24 Apache Spark provádí na clusteru serverů distribuované výpočty k úpravám dat – například filtruje chybné souřadnice hlásící nehodu v oceánu, opravuje formáty času a logicky spojuje tabulky. : 25 |
| **3. Masivní úložiště** | **Hadoop HDFS / NoSQL** | Po vyčištění jsou záznamy přesunuty do distribuovaných databází. : 26 Tyto databáze, jako je například Cassandra, jsou architektonicky stavěné na velmi rychlé čtení miliard záznamů. : 26 |
| **4. Zobrazení dat** | **Kepler.gl** | Během posledního kroku přistupuje datový analytik prostřednictvím systému Kepler.gl přímo do napojené databáze, aby si zpracovaná data vizuálně prohlédl. : 27 |

---

## Živá ukázka : 28
Interaktivní demo prostředí nástroje naleznete na následujícím odkazu:
**[https://kepler.gl/demo(https://kepler.gl/demo)** : 29

---

## Závěr : 30
* I když dokážeme do databází, jako je HDFS, úspěšně uložit miliony záznamů, lidský mozek nedokáže z pouhého pohledu do tabulek pochopit širší souvislosti. : 31
* Z toho důvodu je nezbytná Geospatial vizualizace, která efektivně propojuje svět Big Dat s lidským pochopením problému. : 32
* Právě vizualizace dává velkým datům potřebný kontext času a konkrétního místa v prostoru. : 32