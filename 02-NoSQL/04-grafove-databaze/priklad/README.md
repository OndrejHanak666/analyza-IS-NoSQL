# Forenzní Analýza Kybernetického Incidentu a Trasování Kryptoměn v Neo4j

---

## 1. Úvod do vyšetřování (Incident Context)

Jako vedoucí týmu pro reakci na incidenty (**DFIR - Digital Forensics and Incident Response**) jste byli povoláni k závažnému bezpečnostnímu incidentu ve finanční instituci. V ranních hodinách došlo k ransomwarovému zašifrování kritického databázového serveru `SRV-FIN-DB`. Útočník zanechal na serveru vyděračský dopis (ransom note) s požadavkem na zaplacení výkupného v hodnotě **5 BTC** na specifickou bitcoinovou peněženku.

Tradiční relační logy a textové soubory SIEMu neumožňují efektivní vizualizaci a trasování pokročilých perzistentních hrozeb (**APT**) ani komplexních finančních transakcí. Bezpečnostní analytici proto agregovali veškerá dostupná data – síťové toky, přístupová práva, e-mailovou komunikaci, logy přihlášení a blockchainové transakce – a importovali je do grafové databáze **Neo4j**.

Vaším úkolem je využít sílu dotazovacího jazyka **Cypher** k rekonstrukci celého útoku, zmapování cesty útočníka v síti (**Lateral Movement**), vysledování praní špinavých peněz (**Peel Chain**) a odhalení případného vnitřního pachatele (**Rogue Insider**).

---

## 2. Datový Model (Database Schema)

Při psaní dotazů vycházejte z této definované struktury uzlů a vztahů. Atributy v závorkách představují klíčové vlastnosti, které můžete filtrovat.

### Uzly (Nodes):
*   `User` (Zaměstnanecké účty: `{username, name, email, department, role}`)
*   `Machine` (Pracovní stanice a servery: `{hostname, ip, os, status, description}`)
*   `IP_Address` (Interní i externí síťové adresy: `{ip, type, country, threat_score, description}`)
*   `Wallet` (Kryptoměnové peněženky: `{address, type, description}`)
*   `Email` (E-mailové zprávy: `{id, subject, sender, timestamp, has_malicious_link, body_preview}`)

### Vztahy (Relationships):
*   `(User)-[:HAS_ACCESS_TO]->(Machine)` – Přístupová oprávnění uživatelů k zařízením.
*   `(Machine)-[:COMMUNICATES_WITH {protocol, port, direction}]->(Machine)` – Detekovaný síťový provoz mezi stroji.
*   `(Email)-[:DELIVERED_TO]->(User)` – E-maily doručené zaměstnancům.
*   `(Machine)-[:DROPPED_NOTE_WITH_ADDRESS]->(Wallet)` – Škodlivý kód na serveru uložil ransom note odkazující na konkrétní peněženku.
*   `(Wallet)-[:TRANSFERRED_TO {amount, tx_id, timestamp}]->(Wallet)` – Blockchainové transakce mezi peněženkami (v BTC).
*   `(User)-[:OWNS]->(Wallet)` – Registrovaní majitelé peněženek (odhaleno např. z OSINT analýzy nebo interního šetření).
*   `(User)-[:LOGGED_IN_FROM {timestamp, successful}]->(IP_Address)` – Historie přihlášení uživatelů z různých IP adres.

---

## 3. Vyšetřovací Úkoly (Milestones)

Pro úspěšné uzavření případu a podání zprávy vedení společnosti musíte vyřešit následující tři komplexní úkoly:

### Úkol 1: Rekonstrukce laterálního pohybu a vektor průniku
Epicentrem útoku je server s hostname `SRV-FIN-DB` (má nastaven vlastnost `status: 'ENCRYPTED'`). 
1.  Najděte kompletní řetězec komunikace, který vedl k tomuto serveru z původně kompromitovaného interního počítače. Aby se zabránilo falešným poplachům, zaměřte se výhradně na administrativní protokoly typické pro laterální pohyb (RDP na portu 3389 nebo SSH na portu 22).
2.  Identifikujte konkrétního zaměstnance, jehož počítač byl zneužit jako odrazový můstek.
3.  Najděte podvodný phishingový e-mail (s atributem `has_malicious_link: true`), který stál na samém počátku incidentu.

### Úkol 2: Analýza finančních toků (Trasování peel chainu)
Útočníci zřídka převádějí výkupné přímo na burzu. Využívají tzv. **Peel Chain** – techniku, kdy se z hlavní peněženky postupně odštěpují menší částky (např. jako poplatky nebo falešné platby), zatímco zbytek peněz se přeposílá na novou a novou adresu, dokud se neocitne v bezpečné konsolidační peněžence.
1.  Extrahujte adresu počáteční peněženky zanechané na serveru `SRV-FIN-DB`.
2.  Sledujte cestu transakcí (`TRANSFERRED_TO`) a zjistěte adresu **koncové konsolidační peněženky**, kde se praní finančních prostředků zastavilo (tato peněženka již nemá žádné odchozí transakce).
    *Poznámka: Blokový graf obsahuje transakční šum a poplatky. Sledujte pouze hlavní větev peel chainu (transakce, kde se převádí částka `amount > 1.0` BTC).*

### Úkol 3: Atribuce incidentu a odhalení vnitřního pachatele
Existuje podezření, že útočníci měli pomoc zevnitř organizace. Korelujte kybernetické stopy s těmi finančními:
1.  Zjistěte externí IP adresu útočníka, ze které proběhlo anomální přihlášení na kompromitovaný účet oběti z Úkolu 1.
2.  Zkontrolujte, zda se z této identické externí IP adresy nepřihlásil i jiný zaměstnanec organizace do svého vlastního účtu.
3.  Ověřte, zda tento podezřelý zaměstnanec není přímo propojen s praním peněz (např. zda nevlastní koncovou konsolidační peněženku z Úkolu 2).
4.  Uveďte jméno a uživatelské jméno tohoto rogue insidera.

---

## 4. Jak Začít (Rychlý Start)

1.  **Spuštění Neo4j**: Přejděte do této složky (`priklad/`), která obsahuje `docker-compose.yml`, a spusťte předpřipravený kontejner:
    ```bash
    docker compose up -d
    ```
    *Poznámka: Tento kontejner obsahuje již plně předpřipravenou a naseedovanou databázi s incidentovým grafem. Nemusíte importovat žádná data ručně!*
2.  **Otevření konzole**: V prohlížeči navštivte `http://localhost:7474`. Přihlaste se výchozími údaji:
    *   **Uživatel:** `neo4j`
    *   **Heslo:** `password`
3.  **Vyšetřování**: Začněte psát Cypher dotazy do konzole nad připraveným grafem.
4.  **Odevzdání výsledků**: Zaznamenejte si své výsledné Cypher dotazy a zjištěné odpovědi (jména podezřelých, adresy peněženek, hostname zařízení a síťové IP adresy) a prezentujte je přímo vyučujícímu k vyhodnocení.

---

## 4.5. Využití Vizuální Analýzy (Neo4j Browser Visual Tracing)

Grafové databáze vynikají možností vizuálního zkoumání vztahů. Pro úspěšné vyřešení incidentu a rozklíčování komplexních stop doporučujeme aktivně využívat grafické rozhraní Neo4j Browseru:
1.  **Barevné kódování uzlů**: Kliknutím na jakýkoliv štítek uzlu (např. `:User`, `:Machine`, `:Wallet`) v horním panelu výsledků můžete změnit jeho barvu, velikost a zobrazovaný atribut (např. nastavit, aby se u `:User` zobrazovalo `username` nebo `name` místo interního `id`). To vám pomůže vizuálně zmapovat incident.
2.  **Expanze sousedů**: Dvojklikem na libovolný kulatý uzel v zobrazeném grafu automaticky rozbalíte všechny jeho přímé sousedy. Můžete tak ručně trasovat, jaké stroje komunikují s daným serverem nebo komu patří bitcoinové peněženky, aniž byste museli psát složité dotazy.
3.  **Vizuální trasování tloušťky hran (transakcí)**: V blockchainové analýze reprezentují vztahy `TRANSFERRED_TO` tok financí. Ve vizualizaci můžete sledovat vlastnosti (properties) jednotlivých hran tak, že na ně najedete myší – uvidíte tak atribut `amount`. Můžete tak vizuálně oddělit drobný transakční šum (např. poplatky < 0.5 BTC) od hlavních finančních toků.
4.  **Odhalování dekoyů (falešných stop)**: V tomto úkolu na vás čeká několik větvení. K doménovému kontroléru `SRV-AD-01` uvidíte přistupovat více počítačů přes RDP a v peel chainu uvidíte větvení do více směrů. Budete muset zkombinovat vizuální double-click trasování s Cypher filtry, abyste odlišili legitimní nebo falešné toky od těch škodlivých.

---

## 5. Pokročilé Nápovědy a Skryté Tipy

Pokud jste se při psaní Cypher dotazů ztratili, rozklikněte si níže uvedené nápovědy. Jsou navrženy tak, aby vás vedly krok za krokem a pomohly vám složit řešení z menších dílčích dotazů.

<details>
<summary>💡 Nápovědy k Úkolu 1: Rekonstrukce laterálního pohybu</summary>

Pro vyřešení tohoto úkolu doporučujeme postupovat po menších logických krůčcích:

<details>
<summary>🔍 Krok 1.1: Kdo komunikoval přímo se zašifrovaným serverem?</summary>

Začněte od serveru `SRV-FIN-DB` a vyhledejte příchozí síťové spoje. Uvidíte více příchozích šipek (větvení v síťovém grafu). Hledáme administrátorské protokoly (SSH na portu 22).

**💡 Koncepční nápověda:**
Budete potřebovat klauzuli `MATCH` pro vyhledání vzoru uzlů a vztahů, filtraci na konkrétní `hostname` cílové stanice a port komunikace, a klauzuli `RETURN`.

*Užitečné Cypher konstrukce a příkazy:* `MATCH`, `WHERE`, `RETURN`, šipkový vztah `<-[r:COMMUNICATES_WITH]-`

<details>
<summary>🛠️ Ukázat Cypher dotaz pro vyřešení</summary>

```cypher
MATCH (target:Machine {hostname: 'SRV-FIN-DB'})<-[r:COMMUNICATES_WITH]-(m:Machine)
RETURN m.hostname, r.protocol, r.port
```
*Tento dotaz vám odhalí Active Directory kontrolér, přes který útočník přišel.*
</details>
</details>

<details>
<summary>🔍 Krok 1.2: Odkud pokračoval útočník na doménový kontrolér?</summary>

Vyjděte ze zjištěného Active Directory kontroléru `SRV-AD-01` a zjistěte, která zařízení se k němu připojovala přes vzdálenou správu Windows RDP (port 3389). Všimněte si, že se zobrazí 4 paralelní spojení. Budete muset zjistit, které z nich je zapojeno do dalších fází!

**💡 Koncepční nápověda:**
Použijte `MATCH` pro vyhledání komunikace směřující do AD kontroléru. Omezte port vztahu přesně na RDP port `3389` a vypište hostname zdrojových stanic.

*Užitečné Cypher konstrukce a příkazy:* `MATCH`, `WHERE`, `=`, `RETURN`

<details>
<summary>🛠️ Ukázat Cypher dotaz pro vyřešení</summary>

```cypher
MATCH (ad:Machine {hostname: 'SRV-AD-01'})<-[r:COMMUNICATES_WITH]-(m:Machine)
WHERE r.port = 3389
RETURN m.hostname, r.protocol
```
*Tento dotaz vám odhalí seznam 4 stanic s RDP logem – včetně naší počáteční stanice.*
</details>
</details>

<details>
<summary>🔍 Krok 1.3: Kdo vlastní a má přístup ke kompromitované stanici?</summary>

Najděte uživatele, kteří mají oprávnění k přístupu (`HAS_ACCESS_TO`) k prvotnímu kompromitovanému stroji `PC-HORVATH-04`. Uvidíte, že k tomuto stroji mají přístup dva zaměstnanci (Marian Horváth a Jana Černá). Musíte korigovat, kdo z nich obdržel phishing!

**💡 Koncepční nápověda:**
Formulujte `MATCH` dotaz propojující uzel `User` se vztahem `HAS_ACCESS_TO` směřujícím k uzlu `Machine` se specifickým hostname.

*Užitečné Cypher konstrukce a příkazy:* `MATCH`, `-[:HAS_ACCESS_TO]->`, `RETURN`

<details>
<summary>🛠️ Ukázat Cypher dotaz pro vyřešení</summary>

```cypher
MATCH (u:User)-[:HAS_ACCESS_TO]->(m:Machine {hostname: 'PC-HORVATH-04'})
RETURN u.name, u.username, u.department
```
*Tento dotaz odhalí účet prvotní oběti.*
</details>
</details>

<details>
<summary>🔍 Krok 1.4: Který phishingový e-mail stál na samém počátku?</summary>

Hledejte e-maily doručené (`DELIVERED_TO`) podezřelým uživatelům, které mají vlastnost `has_malicious_link` nastavenou na `true`. V datasetu je více phished uživatelů, ale pouze e-mail doručený uživateli `horvath.m` se pojí s kompromitovanou stanicí.

**💡 Koncepční nápověda:**
Spojte e-mail a uživatele Marian Horváth přes doručovací vztah a vyfiltrujte pouze e-maily s pravdivým příznakem škodlivého odkazu.

*Užitečné Cypher konstrukce a příkazy:* `MATCH`, `WHERE`, `AND`, `RETURN`

<details>
<summary>🛠️ Ukázat Cypher dotaz pro vyřešení</summary>

```cypher
MATCH (e:Email)-[:DELIVERED_TO]->(u:User {username: 'horvath.m'})
WHERE e.has_malicious_link = true
RETURN e.subject, e.sender, e.body_preview
```
</details>
</details>

<details>
<summary>🔍 Krok 1.5: Elegantní spojení cesty do jednoho dotazu (Pokročilé)</summary>

Zkombinujte phishingový mail, uživatele, přístup na stanici a síťovou cestu s délkou od 1 do 5 skoků do jednoho příkazu. Abychom odfiltrovali běžný síťový provoz a šum, ujistěte se, že všechny přechodné síťové vztahy využívají výhradně administrátorské porty `22` (SSH) nebo `3389` (RDP).

**💡 Koncepční nápověda:**
Definujte cestu s proměnným počtem skoků `*1..5` pro komunikační vztahy. Využijte predikát `ALL` k vyhodnocení, zda každá hrana cesty, která je typu `COMMUNICATES_WITH`, splňuje podmínku portu (22 nebo 3389). Ostatní vztahy (doručení, přístup) musí být ignorovány pomocí prověření typu vztahu.

*Užitečné Cypher konstrukce a příkazy:* `MATCH path = ...`, `relationships(path)`, `type(r)`, `IN`, `ALL()`, `RETURN`

<details>
<summary>🛠️ Ukázat Cypher dotaz pro vyřešení</summary>

```cypher
MATCH path = (e:Email)-[:DELIVERED_TO]->(u:User)-[:HAS_ACCESS_TO]->(m1:Machine)-[:COMMUNICATES_WITH*1..5]->(m2:Machine {hostname: 'SRV-FIN-DB'})
WHERE e.has_malicious_link = true
  AND m1.status = 'COMPROMISED'
  AND ALL(r IN relationships(path) WHERE type(r) <> 'COMMUNICATES_WITH' OR r.port IN [22, 3389])
RETURN path
```
</details>
</details>
</details>

<details>
<summary>💡 Nápovědy k Úkolu 2: Trasování kryptoměn (Peel Chain)</summary>

Sledování finančních transakcí v blockchainu vyžaduje postupné trasování toku financí:

<details>
<summary>🔍 Krok 2.1: Jaká je počáteční ransom peněženka?</summary>

Nejprve zjistěte adresu bitcoinové peněženky, kterou ransomware zanechal v dopise na serveru `SRV-FIN-DB`.

**💡 Koncepční nápověda:**
Vyhledejte vztah zanechání dopisu (`DROPPED_NOTE_WITH_ADDRESS`) ze serveru do cílové peněženky (`Wallet`).

*Užitečné Cypher konstrukce a příkazy:* `MATCH`, `-[:DROPPED_NOTE_WITH_ADDRESS]->`, `RETURN`

<details>
<summary>🛠️ Ukázat Cypher dotaz pro vyřešení</summary>

```cypher
MATCH (m:Machine {hostname: 'SRV-FIN-DB'})-[:DROPPED_NOTE_WITH_ADDRESS]->(w:Wallet)
RETURN w.address, w.description
```
</details>
</details>

<details>
<summary>🔍 Krok 2.2: Kam odešly první prostředky?</summary>

Sledujte odchozí transakce `TRANSFERRED_TO` z počáteční peněženky. Uvidíte rozvětvení (3 transakce) – drobný poplatek, střední částka (falešná odbočka) a velká celá částka (hlavní peel chain). Klikněte na hrany ve vizualizaci a porovnejte jejich atributy `amount`!

**💡 Koncepční nápověda:**
Sestavte `MATCH` dotaz s počáteční peněženkou a jednokrokovým vztahem `TRANSFERRED_TO`. Vypište adresy cílových peněženek a transakční částky.

*Užitečné Cypher konstrukce a příkazy:* `MATCH`, `-[t:TRANSFERRED_TO]->`, `RETURN`, `t.amount`

<details>
<summary>🛠️ Ukázat Cypher dotaz pro vyřešení</summary>

```cypher
MATCH (w_start:Wallet {address: '1Lbcfr7sA1prDdPMXaNHGWZ654321'})-[t:TRANSFERRED_TO]->(w_next:Wallet)
RETURN w_next.address, w_next.description, t.amount, t.tx_id
```
*Všimněte si, že 5.0 BTC putovalo do hlavní peel-chain adresy.*
</details>
</details>

<details>
<summary>🔍 Krok 2.3: Automatické trasování peel chainu až do jímky</summary>

Sledujte řetězec převodů s libovolnou délkou a vyhledejte koncovou peněženku, která již nemá žádné odchozí transakce. Abychom odfiltrovali paralelní decoy větve, které mají částky pod 1.0 BTC, použijte predikát `ALL` a omezte každý skok na `amount > 1.0`.

**💡 Koncepční nápověda:**
Hledáme cestu o proměnlivé délce `*1..10` transakcí z počáteční peněženky. Pomocí predikátu `ALL` vynutíme, že každá hrana cesty má `r.amount > 1.0`. Negací `NOT` zajistíme, že z cílové peněženky již neodchází žádný vztah převodu.

*Užitečné Cypher konstrukce a příkazy:* `MATCH path = (w_start:...)-[:TRANSFERRED_TO*1..10]->(w_final:Wallet)`, `NOT (w_final)-[:TRANSFERRED_TO]->()`, `ALL()`, `relationships(path)`, `length(path)`

<details>
<summary>🛠️ Ukázat Cypher dotaz pro vyřešení</summary>

```cypher
MATCH path = (w_start:Wallet {address: '1Lbcfr7sA1prDdPMXaNHGWZ654321'})-[:TRANSFERRED_TO*1..10]->(w_final:Wallet)
WHERE NOT (w_final)-[:TRANSFERRED_TO]->()
  AND ALL(r IN relationships(path) WHERE r.amount > 1.0)
RETURN w_final.address, length(path) as skoky
```
</details>
</details>
</details>

<details>
<summary>💡 Nápovědy k Úkolu 3: Korelace a atribuce vnitřního pachatele</summary>

Pro odhalení rogue insidera musíme propojit síťové logy s finanční stopou:

<details>
<summary>🔍 Krok 3.1: Odkud se přihlásil externí útočník?</summary>

Zjistěte historii přihlášení (`LOGGED_IN_FROM`) pro Mariana Horvátha. Hledejte externí IP adresu s vysokým rizikovým skóre (`threat_score > 80`).

**💡 Koncepční nápověda:**
Vyhledejte login logy Mariana Horvátha. Filtrujte typ IP adresy na hodnotu `External` a threat score na hodnotu větší než 80.

*Užitečné Cypher konstrukce a příkazy:* `MATCH`, `-[:LOGGED_IN_FROM]->`, `WHERE`, `AND`, `RETURN`

<details>
<summary>🛠️ Ukázat Cypher dotaz pro vyřešení</summary>

```cypher
MATCH (u:User {username: 'horvath.m'})-[l:LOGGED_IN_FROM]->(ip:IP_Address)
WHERE ip.type = 'External' AND ip.threat_score > 80
RETURN ip.ip, ip.country, ip.threat_score, l.timestamp
```
</details>
</details>

<details>
<summary>🔍 Krok 3.2: Přihlásil se z této stejné IP adresy ještě někdo jiný?</summary>

Vyhledejte všechny uživatele, kteří se přihlásili ze stejné anomální IP adresy `198.51.100.42`. Uvidíte více výsledků (dva podezřelé administrátory!).

**💡 Koncepční nápověda:**
Vyhledejte uživatele mající vztah přihlášení směřující k uzlu IP adresy s touto konkrétní IP hodnotou.

*Užitečné Cypher konstrukce a příkazy:* `MATCH`, `<-[:LOGGED_IN_FROM]-`, `RETURN`

<details>
<summary>🛠️ Ukázat Cypher dotaz pro vyřešení</summary>

```cypher
MATCH (ip:IP_Address {ip: '198.51.100.42'})<-[:LOGGED_IN_FROM]-(u:User)
RETURN u.name, u.username, u.role, u.department
```
*Tímto odhalíte jména podezřelých kolegů.*
</details>
</details>

<details>
<summary>🔍 Krok 3.3: Je podezřelý propojen s vypranými penězi?</summary>

Zkontrolujte, který z obou podezřelých z předchozího kroku vlastní (`OWNS`) finální kryptoměnovou peněženku z konce finančního toku (`1Consolidate999`). Uvidíte, že druhý podezřelý `dvorak.a` sice také vlastní peněženku, ale jde o burzovní decoy jímku `1DecoyExchangeSink` z falešné větve!

**💡 Koncepční nápověda:**
Propojte uživatele s cílovou adresou z Úkolu 2 pomocí vlastnického vztahu `OWNS`.

*Užitečné Cypher konstrukce a příkazy:* `MATCH`, `-[:OWNS]->`, `RETURN`

<details>
<summary>🛠️ Ukázat Cypher dotaz pro vyřešení</summary>

```cypher
MATCH (u:User {username: 'kovar.v'})-[:OWNS]->(w:Wallet {address: '1Consolidate999'})
RETURN u.name, w.address, w.description
```
*Toto představuje nezvratný důkaz vnitřního komplice.*
</details>
</details>

<details>
<summary>🔍 Krok 3.4: Spojení všech důkazů do jedné korelační zprávy</summary>

Propojte anomální přihlášení oběti, společnou IP, rogue insidera a vlastnictví finální peněženky do jednoho elegantního dotazu. Ujistěte se, že vyloučíte shodu oběti a pachatele.

**💡 Koncepční nápověda:**
Formulujte vzor propojující uzly oběti a pachatele přes společný uzel IP adresy, a přidejte k pachateli jeho vlastněnou peněženku. V klauzuli `WHERE` zkontrolujte nerovnost uživatelských jmen oběti a pachatele.

*Užitečné Cypher konstrukce a příkazy:* `MATCH`, `WHERE`, `<>`, `RETURN`

<details>
<summary>🛠️ Ukázat Cypher dotaz pro vyřešení</summary>

```cypher
MATCH (victim:User {username: 'horvath.m'})-[:LOGGED_IN_FROM]->(ip:IP_Address)<-[:LOGGED_IN_FROM]-(suspect:User)-[:OWNS]->(w_final:Wallet {address: '1Consolidate999'})
WHERE suspect <> victim
RETURN suspect.name, suspect.username, suspect.role, ip.ip, w_final.address
```
</details>
</details>
</details>