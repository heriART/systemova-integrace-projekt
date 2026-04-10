# Handoff dokument – na co navázat v kapitolách 7–10

## Kontext projektu

Navrhujeme integraci **CRM systému Salesforce** do fiktivní hotelové sítě **Aurora Hotels a.s.** (7 hotelů v ČR, 280 zaměstnanců, obrat 320 mil. Kč). Systémový integrátor je **Seyfor a.s.** (reálná firma, Salesforce partner).

---

## Co je hotové (kapitoly 1–6)

### Kapitoly 1–3 (Sváťa)
- Úvod, charakteristika podniku, profil Seyforu
- Definovány ekonomické ukazatele obou firem

### Kapitola 4 – Globální strategie (Petr)
- SWOT analýza Aurora Hotels
- 6 globálních cílů (centralizace dat, opakované návštěvy, marketing, zákaznická zkušenost, firemní klientela, mimosezónní obsazenost)
- Portfolio služeb: ubytování, konference, gastronomie, wellness, doplňkové služby

### Kapitola 5 – Informační strategie (Petr)
- **Stávající systémy v podniku** (na toto musíte navázat!):
  - **PMS Horesplus** – správa rezervací, check-in/out, pokoje, vyúčtování. Každý hotel má **samostatnou instanci** s lokální DB. Data mezi hotely se nesdílí.
  - **Účetní systém Money S5** – centrální, data z PMS se přenáší manuálním exportem 1× denně.
  - **Channel Manager SiteMinder** – správa online rezervací z Booking.com, Expedia. Komunikuje s PMS, ale nepředává data do centrálního systému.
  - **Microsoft 365** – e-mail, Office. Kampaně rozesílány manuálně přes Outlook.
  - **Webové stránky** – prezentační web s online rezervací přes Channel Manager. Bez zákaznické zóny.
  - **Excel tabulky** – nekoordinované seznamy VIP hostů, firemních klientů v jednotlivých hotelech.
- **Kontextový diagram stávající situace** – draw.io soubor v `data/diagrams/`
- **6 cílů informační strategie:**
  1. Zavedení centrálního CRM Salesforce
  2. Integrace CRM ↔ PMS Horesplus
  3. Integrace CRM ↔ Channel Manager SiteMinder
  4. Nasazení Salesforce Marketing Cloud
  5. Vytvoření centrální databáze hostů
  6. Zavedení reportingového nástroje (Salesforce Reports & Dashboards)
- **Tabulka dimenzí** – každý cíl je namapován na dimenze EA (procesní, datová, aplikační, technologická), viz níže
- **CSF tabulka** – 10 kritických faktorů, nejkritičtější: migrace dat, integrace s PMS, GDPR

### Kapitola 6 – Business architektura (Ondra/Petr)
- **BPMN diagram** – proces „Správa rezervace a check-in hosta" s 5 účastníky (Host, Recepce, PMS, CRM Salesforce, Channel Manager). BPMN soubor v `data/diagrams/`.
- **Katalog požadavků:**
  - 14 funkčních (F01–F14): zákaznický profil, historie pobytů, preference, firemní kontakty, obchodní příležitosti, segmentace, kampaně, synchronizace CRM–PMS, synchronizace CRM–ChM, reporty, věrnostní program, GDPR, komunikace
  - 10 nefunkčních (N01–N10): dostupnost 99,9%, odezva <3s, GDPR šifrování, MFA, webový přístup, mobilní přístup, 100 souběžných uživatelů / 500k záznamů, REST API integrace, zálohování, vícejazyčnost

---

## Co je potřeba udělat – návaznosti po kapitolách

### Kapitola 7 – Marek – Datová architektura

**Co se od tebe čeká:**
1. **Nový kontextový diagram (cílový stav)** – stejné entity jako stávající diagram, ale uprostřed je teď Salesforce CRM jako centrální systém. Přibydou nové toky (CRM ↔ PMS, CRM ↔ Channel Manager, CRM → Marketing Cloud → Host).
2. **DFD první úrovně** – rozlož centrální systém na procesy. Navrhované procesy:
   - P1: Správa zákaznických profilů
   - P2: Zpracování rezervací
   - P3: Správa obchodních příležitostí (firemní klienti)
   - P4: Marketingové kampaně
   - P5: Reporting a dashboardy
   - P6: Synchronizace dat (PMS ↔ CRM)
3. **ERD** – návrh databázové struktury CRM. Klíčové entity:
   - **Host** (kontakt) – jméno, e-mail, telefon, GDPR souhlas, segment, věrnostní úroveň
   - **Firma** – název, IČO, kontaktní osoba, rámcová smlouva
   - **Rezervace** – host, hotel, pokoje, datum od/do, kanál (přímý/OTA/web), stav
   - **Pobyt** – realizovaný pobyt, využité služby, vyúčtování
   - **Preference** – typ pokoje, patro, stravování, speciální požadavky
   - **Obchodní příležitost** – firma, typ akce, fáze, hodnota, zodpovědný obchodník
   - **Kampaň** – název, segment, kanál, datum, výsledky
   - **Komunikace** – host, typ (e-mail/telefon/poznámka), datum, obsah
   - **Hotel** – název, město, kapacita

**Na co navázat:**
- Stávající systémy z kap. 5.1 (PMS, Money S5, SiteMinder)
- Funkční požadavky F01–F14 z kap. 6.2 – každý požadavek implikuje datové entity
- Informační toky z kontextového diagramu

---

### Kapitola 8 – Alex – Aplikační architektura

**Co se od tebe čeká:**
1. **UML diagram tříd** – objektový model CRM modulu/funkcionality
2. **Use Case diagram** – případy užití

**Diagram tříd – navrhované třídy:**
- `Host` (atributy: id, jmeno, prijmeni, email, telefon, segment, vernostniUroven, gdprSouhlas)
- `Firma` (atributy: id, nazev, ico, adresa)
- `KontaktniOsoba` (vazba na Firma, atributy: jmeno, pozice, email, telefon)
- `Rezervace` (vazba na Host, Hotel; atributy: datumOd, datumDo, kanal, stav, typPokoje)
- `Pobyt` (vazba na Rezervace; atributy: skutecnyDatumOd, skutecnyDatumDo, celkovaCena)
- `Preference` (vazba na Host; atributy: typPokoje, patro, stravovani, poznamky)
- `ObchodniPrilezitost` (vazba na Firma; atributy: typAkce, faze, hodnota, zodpovednyObchodnik)
- `Kampan` (atributy: nazev, segment, kanal, datumOd, datumDo)
- `Komunikace` (vazba na Host; atributy: typ, datum, obsah, autor)
- `Hotel` (atributy: id, nazev, mesto, kapacita)

**Use Case – navrhovaní aktéři:**
- **Recepční** – vyhledání hosta, zobrazení profilu, vytvoření profilu, check-in s preferencemi, záznam komunikace
- **Obchodník** – správa firemních kontaktů, obchodní příležitosti, cenové nabídky
- **Marketing** – segmentace zákazníků, vytvoření kampaně, vyhodnocení kampaně, správa věrnostního programu
- **Manažer/Vedení** – zobrazení reportů a dashboardů
- **IT správce** – konfigurace synchronizace CRM–PMS, správa uživatelů, správa GDPR souhlasů
- **Systém (PMS)** – automatická synchronizace dat (aktor = systém, ne člověk)

**Na co navázat:**
- Požadavky F01–F14 z kap. 6.2 – každý funkční požadavek = potenciální use case
- Účastníci procesu z BPMN diagramu (kap. 6.1) = aktéři v use case
- ERD entity od Marka (kap. 7) = třídy v diagramu tříd. **Domluvte se s Markem, aby entity byly konzistentní!**

---

### Kapitola 9 – Sergiu – Technologická architektura

**Co se od tebe čeká (formou tabulek):**

**9.1 Požadavky na externí informační zdroje:**
- Salesforce platforma (cloudová, external)
- OTA kanály (Booking.com API, Expedia API) – přes SiteMinder
- Platební brány (pro online rezervace)
- Registr ekonomických subjektů ARES (ověření firemních klientů)
- Google Analytics (webová analytika)

**9.2 Požadavky na technické vybavení:**
- Salesforce je cloud → **nejsou potřeba vlastní servery pro CRM**
- Ale jsou potřeba: klientské stanice (PC na recepcích), síťová infrastruktura, případně tablety pro mobilní přístup
- Integrace PMS–CRM vyžaduje middleware server nebo integracni platformu (MuleSoft / Salesforce Connect)

**9.3 Požadavky na SW:**
- Salesforce Sales Cloud (licence per user)
- Salesforce Marketing Cloud (licence)
- Salesforce Reports & Dashboards (součást licence)
- Middleware pro integraci (MuleSoft / Informatica / custom REST API)
- Stávající systémy zůstávají: PMS Horesplus, Money S5, SiteMinder, M365

**9.4 Bezpečnost:**
- GDPR compliance (šifrování AES-256, TLS 1.2+) – viz nefunkční požadavek N03
- MFA pro všechny uživatele – viz N04
- Salesforce Shield (monitoring, audit trail)
- Role-based access control (RBAC) v Salesforce
- Zálohování 1× denně, RTO 4h – viz N09

**9.5 Požadavky na specializovaný personál:**
- Salesforce administrátor (1 FTE nebo external)
- IT správce pro middleware/integrace
- Data steward pro kvalitu zákaznických dat

**9.6 Požadavky na školení:**
- Recepční (cca 50 lidí) – základní ovládání CRM, vyhledání hosta, profil
- Obchodníci (cca 10 lidí) – firemní kontakty, obchodní příležitosti
- Marketing (cca 5 lidí) – segmentace, kampaně, Marketing Cloud
- Vedení/manažeři (cca 10 lidí) – reporty, dashboardy
- IT tým (cca 5 lidí) – administrace, integrace, troubleshooting

**Na co navázat:**
- Stávající systémy z kap. 5.1
- Nefunkční požadavky N01–N10 z kap. 6.2 – přímo definují technologické parametry
- CSF z kap. 5.4 – zejména CSF 8 (GDPR), CSF 3 (migrace dat), CSF 7 (školení)

---

### Kapitola 10 – Sváťa – Integrační plán

**Co se od tebe čeká:**
- Ganttův diagram / harmonogram projektu
- Finanční náklady

**Navrhované fáze projektu:**
1. **Analýza a návrh** (měsíc 1–2) – upřesnění požadavků, detailní návrh architektury
2. **Příprava prostředí** (měsíc 2–3) – konfigurace Salesforce, nastavení sandboxu
3. **Vývoj integrací** (měsíc 3–5) – integrace CRM–PMS, CRM–Channel Manager, middleware
4. **Migrace dat** (měsíc 4–5) – export dat z PMS všech 7 hotelů, čištění, import do CRM
5. **Testování** (měsíc 5–6) – UAT, integrační testy, bezpečnostní audit
6. **Školení uživatelů** (měsíc 6) – viz kap. 9.6
7. **Pilotní provoz** (měsíc 7) – nasazení v 1–2 hotelech
8. **Plný provoz** (měsíc 8) – rollout na zbývající hotely
9. **Podpora a údržba** (ongoing) – roční poplatek

**Odhad nákladů (orientační):**
- Salesforce licence: cca 1 500–3 000 Kč/uživatel/měsíc × počet uživatelů
- Implementace (Seyfor): závisí na rozsahu, orientačně 2–5 mil. Kč
- Middleware/integrace: 500k–1,5 mil. Kč
- Školení: 300–500k Kč
- Migrace dat: 200–400k Kč
- Roční provozní náklady: licence + podpora

**Na co navázat:**
- CSF z kap. 5.4 – dodržení rozpočtu a termínu jsou kritické faktory
- Technologické požadavky od Sergia (kap. 9) – ceny HW/SW
- Rozsah školení od Sergia (kap. 9.6) – kolik lidí, jak dlouho

---

## Tabulka dimenzí – jak spolu kapitoly souvisí

Tato tabulka z kap. 5.3 ukazuje, které kapitoly (dimenze) se dotýkají kterého cíle. Pokud u tvé dimenze je ✓, měl bys daný cíl ve své kapitole nějak reflektovat.

| Cíl informační strategie | Procesní (kap. 6) | **Datová (kap. 7)** | **Aplikační (kap. 8)** | **Technologická (kap. 9)** |
| :--- | :---: | :---: | :---: | :---: |
| 1. Zavedení CRM Salesforce | ✓ | ✓ | ✓ | ✓ |
| 2. Integrace CRM–PMS | ✓ | ✓ | ✓ |  |
| 3. Integrace CRM–Channel Manager |  | ✓ | ✓ |  |
| 4. Salesforce Marketing Cloud | ✓ | ✓ | ✓ |  |
| 5. Centrální databáze hostů |  | ✓ |  | ✓ |
| 6. Reportingový nástroj | ✓ | ✓ | ✓ |  |

---

## Soubory v repozitáři

- `data/projekt.docx.md` – hlavní dokument (editujte přímo)
- `data/diagrams/kontextovy-diagram-stavajici-v2.drawio` – kontextový diagram stávající situace (draw.io)
- `data/diagrams/kontextovy-diagram-stavajici-v2.mmd` – totéž v Mermaid formátu
- `data/diagrams/bpmn-rezervace-checkin-v2.bpmn` – BPMN diagram (otevřít na demo.bpmn.io)
- `data/diagrams/bpmn-rezervace-checkin.mmd` – BPMN v Mermaid formátu
- `data/diagrams/csf-data.csv` – data CSF tabulky pro graf

## Důležité: konzistence!

- Používejte **stejné názvy systémů**: PMS Horesplus, Money S5, SiteMinder, Salesforce CRM, Salesforce Marketing Cloud
- Používejte **stejné názvy entit**: Host, Firma, Rezervace, Pobyt, Preference, Obchodní příležitost, Kampaň, Komunikace, Hotel
- Požadavky F01–F14 a N01–N10 jsou referenční – odkazujte na ně ve svých kapitolách
- Číslování tabulek pokračuje od **Tabulka 8** (aktuálně máme 1–7)
- Číslování obrázků pokračuje od **Obrázek 4** (aktuálně máme 1–3)

---

## Shrnutí – kdo co využívá

**Kapitola 7 – Marek (Datová architektura):** Vychází ze stávajících systémů definovaných v kap. 5.1 (PMS Horesplus, SiteMinder, Money S5) a z kontextového diagramu stávající situace (kap. 5.2). Nový kontextový diagram musí ukázat cílový stav se Salesforce CRM uprostřed. DFD rozkládá procesy navržené v BPMN (kap. 6.1) na datové toky. ERD vychází přímo z funkčních požadavků F01–F14 (kap. 6.2) – každý požadavek implikuje datové entity, které je potřeba namodelovat.

**Kapitola 8 – Alex (Aplikační architektura):** Diagram tříd musí být konzistentní s ERD od Marka (kap. 7) – stejné entity, jen v objektové notaci s metodami. Use Case diagram vychází z funkčních požadavků F01–F14 (kap. 6.2), kde každý požadavek odpovídá jednomu nebo více případům užití. Aktéři v Use Case odpovídají účastníkům BPMN procesu z kap. 6.1 (Recepční, Obchodník, Marketing, Manažer, IT správce).

**Kapitola 9 – Sergiu (Technologická architektura):** Tabulky HW/SW vybavení vychází z nefunkčních požadavků N01–N10 (kap. 6.2), které definují konkrétní technické parametry (dostupnost, odezva, šifrování, MFA, kapacita, API). Bezpečnostní požadavky navazují na CSF 8 – GDPR (kap. 5.4) a požadavek F13/N03. Školení dimenzuje podle aktérů a jejich počtů definovaných v katalogu požadavků.

**Kapitola 10 – Sváťa (Integrační plán):** Harmonogram musí pokrýt všechny fáze od analýzy po plný provoz a reflektovat kritické faktory CSF 1 (včasné dodání) a CSF 2 (rozpočet) z kap. 5.4. Finanční náklady se skládají z licencí Salesforce a nákladů na HW/SW z kap. 9, implementačních prací Seyforu a školení z kap. 9.6. Ganttův diagram by měl zohlednit závislosti – např. migrace dat (CSF 3) nemůže začít před dokončením integrací.
