# Hapje/Drankje – context (25 okt 2026)
- Lokale HTML-app om restaurantrekeningen (hapje/drankje) bij te houden op iPhone en iPad in Chrome.
- Oorspronkelijke code was afgekapt en opende niet goed in Safari op iPad. Op Mac opent het in Chrome.
- index.html is de complete, self-contained, mobiel-geoptimaliseerde basisversie (v1).
- Tabs: Overzicht, Toevoegen, Foto's, Geschiedenis. Data in localStorage (hapjeDrankjeVisits, hapjeDrankjePhotos).
- Opgemerkt, nog niet aangepast: actief bezoek wordt niet in localStorage bewaard (weg bij herladen);
  showTab() gebruikt globale `event`, dus bij automatisch tabwisselen licht het verkeerde tabje op;
  foto's als base64 in localStorage kunnen snel de opslaglimiet (~5 MB) vullen.

## v2 (25 okt 2026, index-v2.html = huidige index.html)
- Actief bezoek wordt na elke wijziging opgeslagen (localStorage 'hapjeDrankjeCurrent') en komt terug na herladen.
- Tabjes hebben data-tab; showTab() markeert het juiste tabje, ook bij automatisch wisselen.
- Foto's staan nu in IndexedDB ('hapjeDrankjeDB', store 'photos'), verkleind naar max 1600px JPEG 80%; oude localStorage-foto's worden automatisch overgezet.
- Namen worden ge-escaped bij weergave (geen kapotte opmaak door < of >).
- Alleen syntax gecontroleerd; nog niet in een echte browser getest.

## v3 (25 okt 2026, index-v3.html = huidige index.html; index-v2.html ongewijzigd)
- 'Nieuw restaurantbezoek starten' opent nu eerst de stap 'Menukaart' (geen prompt/confirm meer):
  grote knop om een of meer foto's/screenshots te kiezen (input accept=image/*, multiple, zonder capture),
  'Overslaan', en een lijst 'Eerder bezocht' om een bewaarde menukaart te hergebruiken (met ✕ om te verwijderen).
- Tekstherkenning in de browser met Tesseract.js 5.1.1 van jsDelivr (nld+eng), pas geladen als je een foto kiest.
  Voortgangsbalk + Nederlandse status. Foto's worden na elkaar gelezen; kleine afbeeldingen (<1100 px breed) worden
  vergroot tot ~1600 px, grote (>2000 px) verkleind tot 2000 px, max 12 MP. Eerst PSM 3; als het resultaat zwak is
  (veel namen zonder prijs) nog een keer met PSM 6 en het beste resultaat gekozen. Woorden met lage zekerheid
  (stippellijntjes die als 'eee'/'vive' gelezen worden) worden weggefilterd (ocrDataToText).
- Parser = pure functie parseMenuText(text) -> { name, items:[{name,price,category}], unpriced }, in het script tussen
  '// ==PARSER-START==' en '// ==PARSER-END=='. Kent 12,50 / 12.50 / € 12,50 / €12 / 12,- / EUR / 7,5, stippellijnen,
  prijs op de volgende regel, omschrijvingsregels, kopjes (Dranken, Wijnen, Voorgerechten...), meerdere prijzen op
  één regel (glas/fles, twee kolommen), menunummers, ruis (adres, telefoon, openingstijden, www).
  Categorie eten/drinken via kopje of trefwoorden. Geen lookbehind-regex (iOS Safari 15).
- Controlescherm: naam restaurant, keuze 'Inschatting vooraf' / 'Echte rekening' (standaard: echte rekening),
  bewerkbare gerechtenlijst (naam, prijs, categorie, ✕), '+ Gerecht toevoegen', 'Nog een foto scannen', 'Bezoek starten'.
- Opslag: menukaarten in localStorage 'hapjeDrankjeMenus' = [{id, name, items, photoIds, updated}];
  bezoek krijgt extra velden menu, restaurantId, menuPhotoIds (oude data blijft werken).
  Menufoto's in IndexedDB-store 'photos' met kind:'menu', visitId, restaurant.
- Toevoegen-tab: bovenaan de menukaart als grote tikrijen (naam + prijs, teller 'n×'), zoekveld, toast
  '✓ ... toegevoegd (2× besteld)', link 'Menukaart aanpassen'. Daaronder de oude kaarten 'Item toevoegen' en 'Snel toevoegen'.
  Prijsvelden zijn nu tekstvelden met decimaal toetsenbord (komma werkt).
- Overzicht: per regel − / + knoppen; bij aantal 1 wordt − een ✕ (verwijderen).
- 'Alles wissen' wist bezoeken en foto's, maar laat bewaarde menukaarten staan.
- Getest: node-unit-tests parser (60/60), end-to-end Tesseract.js in node + headless Chrome (puppeteer-core) met
  gegenereerde menukaarten (screenshot licht/donker 14/14, wazige scheve foto 14/14 prijzen, 13/14 namen exact).
  Nog niet op een echte iPhone/iPad getest.

## Doel en specificatie (letterlijk van de gebruiker, 25 okt 2026)
Hapje/Drankje

Achterliggend idee: in de hand houden van de kosten en eenvoudige controle over de eindafrekening.

Ik wil een toepassing(Hapje/Drankje) op mijn iPhone waarmee ik de rekening van restaurants kan controleren. Aan de hand van een screenshot van de menu -en drankaart en het verbruik laat de toepassing zien hoeveel kosten je hebt gemaakt tijdens het eten en drinken. Handmatige invoer voor ‘buiten de kaart om’ moet mogelijk zijn. Het laat dus het kostenverloop van begin tot eind voortdurend zien. Wanneer de rekening komt, dan kun je in één oogopslag zien of het klopt. De toepassing kan ook worden gebruikt om van tevoren een inschatting te maken hoeveel een restaurantbezoek ongeveer gaat kosten. Vraag: kun je zo’n toepassing voor mij maken? Het moet iets worden wat zeer eenvoudig is te bedienen met goed leesbare letters en de toepassing moet gemakkelijk te installeren zijn op smartphones, te beginnen met de iPhone.

Eerste test opmerkingen (op v3):
Het veld waarin staat of iets eten of drinken is vind ik wel erg groot.
Het aangeven wat je bestelt gaat gemakkelijk, het lopende totaal bovenaan is prima.
Er ontbreekt een pro forma rekening, dus een voorlopig rekening overzicht waarop het gebruikte item met prijs aantal gebruikt en totaal per regel.

## v4 (25 okt 2026, index-v4.html = huidige index.html; index-v3.html ongewijzigd)
- Categorie (eten/drinken/extra) is nu een compacte knoppenrij 🍽 / 🥤 / ➕ (knoppen 40x40, gekozen knop blauw omrand)
  in plaats van een volle-breedte keuzeveld: in het controlescherm naast prijs en ✕, en bij 'Item toevoegen'
  (daar met het label van de gekozen soort). Bij 'Item toevoegen' staan prijs en aantal naast elkaar.
- Nieuw tabje 'Rekening' (🧾) en knop '🧾 Voorlopige rekening bekijken' op Overzicht: pro forma rekening met
  kolommen ✓ | Item | Prijs | Aantal | Totaal, gegroepeerd Eten / Drinken / Extra met subtotalen, groot totaal,
  restaurantnaam, datum en 'Inschatting'-label. Lange namen lopen door op de volgende regel; geen horizontaal scrollen op 375 px.
  Afvinken per regel (item.checked) en veld 'Bedrag op de rekening van het restaurant' (visit.billAmount) met
  'Klopt ✓' (groen) of 'Verschil: € x,xx' + hoger/lager (rood). Beide worden bij het bezoek bewaard.
  Rekensom in centen (geen afrondingsfouten).
- Geschiedenis: tabje heet nu 'Eerder' (past op iPhone-breedte); elk bezoek is aan te tikken en opent zijn rekening,
  met 'Klopt ✓' / 'Verschil' in de lijst als er een bedrag is ingevuld. Ook oude v2/v3-bezoeken werken.
- Leesbaarheid: tabjes met pictogram + vet label, kleine tekst 15 px (was 14), donkerder grijs, rekening 17 px.
  Kop + tabjes zitten samen in één vaste balk (voorheen schoof de blauwe kop over de tabjes heen bij scrollen).
- Getest in headless Chrome op 375 px: totalen incl. aantallen en extra's (€ 53,65 klopt met handmatige som),
  verschilberekening (+2,00 / 0 / −3,65), vinkjes en bedrag blijven na herladen, geschiedenisweergave, geen
  horizontale overloop op alle schermen, geen JS-fouten. node --check OK, parsertests 60/60.

## v5 (25-10-2026) – grote letters, rekening als blokken, plaatje op beginscherm
Aanleiding (gebruiker): "voor mijn oude ogen veel te klein … Stuur aub ook altijd de link mee … Uiteindelijk zou ik toepassing als een plaatje op mijn iphone en ipad willen hebben."
Live link: https://peterhanselman-star.github.io/hapje-drankje/

- **Lettergrootte**: alle CSS in rem. `<html>` = 22px (Groot, standaard), class `ts-extra` = 25px, `ts-enorm` = 28px.
  - Knoppen A−/A+ in de kop.
  - Keuze opgeslagen in localStorage `hapjeDrankjeTextSize` ('groot'|'extra'|'enorm'). Een klein script in `<head>` zet de klasse al vóór het tekenen.
  - Body 22px, namen ±24px vet, lopend totaal 2rem (44px), eindtotaal 2.3rem. Alle knoppen en velden ≥ 52px hoog. Tekst zwart/#111/#333 op wit.
- **Tabbalk onderaan** (vast, met `env(safe-area-inset-bottom)`), 4 tabs: Overzicht, Bestellen (= tab-toevoegen), Rekening, Eerder.
  - Labelgrootte `min(0.8rem, 4.6vw)`, dus nooit afgebroken.
  - Foto's zit niet meer in de balk: knop '📷 Foto's' onder Overzicht, met een ←-terugknop. showTab('fotos') licht Overzicht op.
- **Overzicht-regels**: naam op een eigen regel; daaronder 'q × € p' en het regeltotaal vet, met de −/+ knoppen rechts.
- **Rekening**: de tabel is vervangen door blokken `.bill-line`.
  - Elk blok heeft een groot vinkvak `[data-tick=idx]`, de naam (`.bl-name`) en `.bl-calc` met '2 × € 7,50' links en '€ 15,00' rechts.
  - Daarnaast groepkoppen `.bill-group`, subtotalen `.bill-sub`, `.grand .amt`, het veld `#billAmount` (class bill-amount) en `#billCheck` (✓ Klopt / Verschil).
- **Beginscherm**:
  - apple-touch-icon.png (180, vol vierkant; iOS rondt zelf af), icon-192.png en icon-512.png (blauw #1a73e8 met wit bord + €, vork en mes), gemaakt met PIL (/tmp/build/icon.py).
  - manifest.json: standalone, start_url ./, scope ./. In de head staan de links naar het manifest en de iconen.
  - De installatietip (`#installHint`) staat alleen in beeld als de app niet standalone draait (`navigator.standalone` of `display-mode: standalone`). Wegtikken wordt bewaard in `hapjeDrankjeInstallHint`='weg'.
  - De tip meldt dat iOS de opslag van Safari en van het beginscherm-plaatje apart houdt.
- `scroll-padding` boven en onder, zodat velden niet achter de kop of de tabbalk vallen.
- Data: geen wijzigingen; v2-, v3- en v4-gegevens werken ongewijzigd.
- Tests (/tmp/ocrtest/e2e5.js, 73 controles):
  - totaal € 53,65, subtotalen, verschil bij 55,65 / 50 en Klopt bij 53,65;
  - vinkjes en bedrag blijven bewaard, net als de lettergrootte;
  - geschiedenis en het oude v2-bezoek;
  - geen horizontale overloop op 375 en 820px in alle 3 de maten, op 7 schermen;
  - geen JS-fouten; parser 60/60; node --check OK.
- Bestanden: index.html = index-v5.html; index-v1 t/m v4 ongewijzigd.

## v6 (2026-10-25) – menukaart = bestelkaart
Feedback: menukaart en bestelkaart apart was verwarrend; aantallen corrigeren (6 oesters → 3); rood bij hogere rekening, groen bij lagere.
- Aparte controle-stap (`#tab-controle`) verwijderd. Na scannen / bewaarde kaart / overslaan → `createVisit()` → direct tab "Bestellen" (`#tab-toevoegen`).
- Bestellen: bovenaan restaurantnaam (`#orderName`) + compacte Inschatting/Echte rekening (`#segEstimate/#segReal`). Elk gerecht = `.order-row` met naam, prijs en − [aantal] + (≥52px); tik op naam/prijs = +1; besteld = groen gemarkeerd (`.ordered`). "✏️ Aanpassen" schakelt kaart-aanpasstand (naam/prijs/soort, ✕ verwijderen met bevestiging); wijzigingen gaan ook door naar al bestelde regels (koppeling naam+prijs). Zoeken (>6 gerechten), "📷 Nog een foto scannen", "Buiten de kaart" en "Fooi of extraatje" blijven.
- Aantallen: `qtyCtlHtml()` overal (Bestellen, Overzicht, Rekening). Tik op getal → `#qtyDialog` met cijfertoetsenbord. 0 = regel weg; 1→0 met − geeft `#undoBar` "Ongedaan maken" (5 s). Afgeronde bezoeken: alleen kijken, "✏️ Aantallen aanpassen" (`billEdit`).
- Controle: hoger = rood (`.check-result.diff`, "Rekening is € x hoger"), lager = groen (`.lower`, "Rekening is € x lager"), gelijk = groen "✓ Klopt". Ook in Eerder (`.h-check diff-text|lower-text|ok-text`).
- Updates: `APP_VERSION='6'`; `checkForUpdate()` haalt index.html met no-store op en toont "Er is een nieuwe versie – Nu vernieuwen". Geen service worker.
- Gegevens ongewijzigd compatibel (zelfde keys en visit-structuur). Parser identiek aan v4/v5.
- Tests: /workspace/hd-tools (ptest.js 38/38, e2e6.js 96/96, node --check OK). Screenshot v6_bestellen.png.

## v7 (2026-10-25) – aantal personen en bedrag per persoon
Verzoek: na het totaal een veld met aantal personen en het bedrag per persoon.
- Rekening: direct na "Totaal" een blok `#personsBox` "Aantal personen" met − [n] + (≥52px, `data-p`), tik op getal → `#qtyDialog` (uitleg `#qtyHint` "Van 1 tot 50 personen"). Standaard 1, min 1, max 50 (`MAX_PERSONS`).
- `#perPerson`: groot vet "Per persoon: € x,xx" (onze telling). Is het bedrag van het restaurant ingevuld en anders: geel blok "Per persoon (volgens rekening restaurant)" ("Dit is wat ieder echt betaalt") + blauw blok "Per persoon (volgens onze telling)". Gelijk → één blok + "Dit klopt met de rekening van het restaurant".
- Rekenen in centen: `splitCents(c, n)` = ceil; rest → "afgerond; samen € 0,0x meer dan het totaal".
- Opslag: `visit.persons` (alleen als >1; ontbreekt = 1). Afgeronde bezoeken: alleen kijken; knop heet nu "✏️ Aanpassen (aantallen, personen)" (`#billEditBtn`). Eerder-lijst: "4 personen · € 15,00 per persoon" (`.h-persons`, op rekening restaurant als ingevuld).
- `openQtyDialog(name, q, cb, hint)` kreeg optionele uitleg. `APP_VERSION='7'` → v6-app toont "Nu vernieuwen".
- Tests: e2e7.js 41/41, e2e6 (regressie) ALLES OK, node --check OK. Screenshot v7_personen.png.

## v8 (2026-10-25) – verborgen totaal, Letters-uitleg, drinkregels, fooi
Feedback: totaal bovenaan alleen bij aantikken (gasten); wat betekent A−/A+; zes in te vullen drinkregels; fooi direct meetellen.
- Kop: `#totalToggle` "👁 Totaal tonen" (bedrag `#totalDisplay` verborgen). Tik = "Lopend totaal € x · tik om te verbergen"; na 10 s vanzelf weer dicht; elke start verborgen (alleen in geheugen, `totalShown`). Rekening toont totalen gewoon.
- A−/A+ met label "Letters" (`.ts-label`); kopregel mag afbreken.
- Bestellen: na Eten een sectie `#drinkSection`: herkende drankjes van de kaart als gewone regels, daaronder "Drinken zelf invullen" met 6 regels (`visit.drinkRows` [{id,name,price,priceText,qty}]) + "+ Nog een regel". Rekeningregel heeft `drinkRow: id` (categorie drinken). `syncDrink()` houdt naam/prijs gelijk; 0 = weg (met Ongedaan maken). Aantal komt van de rekeningregel (dus Overzicht/Rekening-wijzigingen volgen). `sameDish` negeert drinkRow-regels. Ontbrekende drinkRows → 6 lege.
- Fooi op Rekening (`#tipBox`, na subtotalen, vóór totaal): Geen / 5% / 10% / 15% / Eigen bedrag (`#tipAmount`). `visit.tip` = {pct} of {amount}; geen = afwezig. `tipCents()`: % van de rekening van het restaurant als ingevuld, anders van onze telling (afronden op centen). Totaal heet dan "Te betalen incl. fooi" met "€ x + fooi € y". Vergelijking Klopt/hoger/lager blijft ZONDER fooi (label "(zonder fooi)"). Per persoon = (telling of restaurant) + fooi, gelabeld "incl. fooi". Eerder: "Fooi 10%: € 6,00 · te betalen € 66,00". Bestellen heeft kaart "💶 Fooi kiezen" (`goToTip()`); oude kaart heet nu "Iets extra's" (placeholder zonder fooi).
- `APP_VERSION='8'`. Data compatibel (nieuwe velden optioneel).
- Tests: e2e8.js 74/74, e2e6 ALLES OK, e2e7 40/41 (enige 'fout' = verwachte versie 7), ptest 38/38, node --check OK. Screenshots v8_bestellen_drinken.png, v8_rekening_fooi.png.

## v9 (2026-10-25) – 🔎 Kaart zoeken
Verzoek: knop die met de ingetypte restaurantnaam op Google '<naam> menukaart' zoekt, zodat je een schermafbeelding van de kaart kunt maken en inladen.
- Nieuw bezoek (`#tab-menukaart`): bovenaan nu een veld **Restaurant** (`#newName`), daaronder grote blauwe knop `#kaartZoekenBtn` "🔎 Kaart zoeken" (big-button huge), uitleg `#kaartZoekenHint` "Maak een schermafbeelding van de kaart die je vindt en laad die hier in." (1.1rem vet) en tip `#kaartZoekenTip` "💡 Tip: snijd de schermafbeelding bij, zodat alleen de kaart erop staat." Daarna de bestaande foto-kiezen-knop.
  - De getypte naam gaat mee naar het bezoek (scannen: getypte naam gaat vóór de herkende naam; overslaan: naam ingevuld, en dan geen "Vul de naam in" in de melding). `openMenuStep()` maakt het veld leeg. Bewaarde kaart kiezen blijft de bewaarde naam gebruiken.
- Bestellen: in de kop van de Menukaart-kaart (`.menu-head .head-btns`) naast "✏️ Aanpassen" een knop `#kaartZoekenBtn2` "🔎 Kaart zoeken" (stijl edit-toggle, ≥52px). Ook zichtbaar als er nog geen kaart is; niet in aanpas-stand. Aanpassen blijft de eerste `.edit-toggle` (tests).
- Werking: echte `<a target="_blank" rel="noopener">` (opent op iOS ook vanuit de beginscherm-app in Safari; app zelf navigeert niet weg). `kaartZoekUrl(naam)` = `https://www.google.com/search?q=` + encodeURIComponent('<naam> menukaart') (spaties → %20). `kaartZoeken(ev, inputId, msgId, link)` zet bij tikken de href opnieuw; lege naam → preventDefault, rode melding (`#newNameMsg` / `#orderNameMsg`, class `.name-msg`) "Vul eerst de naam van het restaurant in." + focus op het veld; melding verdwijnt bij typen.
- `APP_VERSION='9'`. Data ongewijzigd.
- Tests: e2e9.js 35/35 (knop op beide plekken, URL 'Verlan Amsterdam' → q=Verlan%20Amsterdam%20menukaart in nieuw tabblad, codering é/&, lege naam melding+focus zonder tabblad, geen overloop 375/820 in alle 3 lettermaten, versie 9). Regressie: e2e6 ALLES OK, e2e7 40/41 en e2e8 73/74 (enige 'fout' = verwachte oude versie), ptest 38/38, node --check OK. Screenshot v9_kaart_zoeken.png (375 px).

## v10 (2026-10-25) – Kaarten klaargezet (online menu's) + link ?kaart=
Verzoek: 🔎 Kaart zoeken weg; in plaats daarvan kaarten die wij als JSON in de repo zetten, met één tik te laden, ook via een link.
- **Kaart zoeken verwijderd** (nieuw bezoek én Bestellen): knoppen `#kaartZoekenBtn`/`#kaartZoekenBtn2`, uitleg, `#newNameMsg`/`#orderNameMsg` (lege-naam-melding), CSS `.search-btn/.search-hint/.name-msg` en `kaartZoekUrl()/kaartZoeken()` zijn weg. Het veld **Restaurant** (`#newName`) blijft (naam voor overslaan/scannen). De bijsnij-tip staat nu als `#cropTip` ("💡 Tip: snijd de foto of schermafbeelding bij…") direct boven de fotoknop.
- **Nieuw bezoek**: bovenaan een kaart `#webMenus` "Kaarten klaargezet" met per kaart een grote knop `.web-btn[data-web=id]` "Verlan Amsterdam — kaart van 25 okt" (+ plaats; "al bewaard – tik om bij te werken" als hij al lokaal staat). `openMenuStep()` roept `loadWebMenuIndex()` aan: `fetch('menus/index.json?t=…', {cache:'no-store'})` (relatief, dus werkt op GitHub Pages in de repo-root). Mislukt, geen JSON, 404 of lege lijst → sectie stil verborgen (geen melding). Alleen op http(s).
- **Tikken** → `startWebMenu(id)`: haalt `entry.file` (no-store), zet items om (`convertWebItems`: cat 'drinken'/'drink' → drinken, 'extra' → extra, anders eten; prijs getal of tekst "6,50"; regels zonder naam/geldige prijs vallen weg), bewaart/overschrijft in `hapjeDrankjeMenus` als `{ id: 'web-<id>', name, items:[{name,price,category}], photoIds:[], updated, online:<id>, retrieved, source }` en start `createVisit(name, items, [], 'web-<id>', notice, { webRetrieved })` → Bestellen, net als na een bevestigde scan. Lukt het ophalen niet maar staat de kaart al lokaal → die wordt gebruikt (melding). Anders rustige melding `#webMenuMsg` "Deze kaart kon nu niet worden opgehaald…".
- Bezoek krijgt `visit.webRetrieved` (optioneel). Bestellen toont dan blijvend `#webNote` (blauw): "🌐 Kaart van internet, prijzen van 25 okt — controleer ze even." Ook bij kiezen via "Eerder bezocht" (bewaarde kaart heeft `retrieved`). `createVisit()` kreeg een optioneel 6e argument `extra` (wordt in het bezoek gezet).
- **Lange kaart** (> 20 items, bv. Verlan met 134 wijnen): onder het zoekvak `#jumpBar` met snelknoppen (≥52px) "🍽 Eten" (`#grp-eten`), "🥤 Drinken" (`#drinkSection`), "✍️ Zelf invullen" (`#grp-zelf`, de zelf-in-te-vullen drinkregels). Snelknop wist een lopende zoekopdracht.
- **Link** `?kaart=<id>` of `#kaart=<id>` (`handleKaartLink()` aan het eind van het script): parameter wordt meteen met `history.replaceState` uit de URL gehaald (andere parameters blijven), dan index laden; loopt er een bezoek → grote eigen vraag `#kaartConfirm` "Nieuw bezoek starten met de kaart van X?" (Ja, nieuw bezoek starten / Nee, huidig bezoek houden; `askBig()` geeft een Promise). Onbekende id → toast "Kaart niet gevonden", verder niets.
  - Let op: een link in een chat opent in **Safari**, niet in de beginscherm-app (aparte opslag). De link vult dus Safari; in de app zelf kies je de kaart onder "Kaarten klaargezet".
- `APP_VERSION='10'`. Gegevens compatibel.

### JSON-formaten
`menus/index.json`:
```json
{"menus": [{"id": "verlan-amsterdam", "name": "Verlan Amsterdam", "place": "Amsterdam", "retrieved": "2026-10-25", "file": "menus/verlan-amsterdam.json"}]}
```
`menus/<id>.json`:
```json
{"name": "Verlan Amsterdam", "source": "https://… (meerdere: gescheiden door ' ; ')", "retrieved": "2026-10-25",
 "items": [{"name": "Oeuf mayonnaise - smoked eel", "price": 7.0, "cat": "voor"}, {"name": "Steak frites - peppercorn sauce", "price": 29.0, "cat": "hoofd"},
           {"name": "…", "price": 6.0, "cat": "bij"}, {"name": "…", "price": 12.0, "cat": "na"}, {"name": "…", "price": 45, "cat": "drinken"}]}
```
- `cat` (v11): `voor` | `hoofd` | `na` | `bij` | `eten` (= Overig eten; oude v10-bestanden met alleen `eten` blijven werken) | `drinken` | `extra`. Ook geaccepteerd: starter(s)/appetizer(s)/voorgerecht → voor, main(s)/hoofdgerecht → hoofd, dessert(s)/nagerecht → na, side(s)/bijgerecht → bij, drink(s) → drinken; onbekend → eten.
- id: kleine letters, cijfers, `-`/`_` (andere tekens worden weggefilterd). `file` is relatief t.o.v. index.html; ontbreekt het → `menus/<id>.json`. `retrieved` = JJJJ-MM-DD (knop/notitie tonen "25 okt").

### Een kaart toevoegen
1. Maak `menus/<id>.json` in het formaat hierboven.
2. Voeg een regel toe aan `menus/index.json` (`id`, `name`, `place`, `retrieved`, `file`).
3. Upload **beide** bestanden naar de repo (map `menus/` in de root, naast index.html). index.html hoeft niet opnieuw. Link om te delen: `https://peterhanselman-star.github.io/hapje-drankje/?kaart=<id>`.
4. Kaart bijwerken = bestand vervangen + `retrieved` aanpassen; in de app opnieuw op de knop tikken overschrijft de bewaarde kaart.

- Tests (/workspace/hd-tools/e2e10.js, 73/73, tegen `python3 -m http.server 8765 --directory /workspace/hapje-drankje`, netwerk deels nagebootst met request interception; voorbeeldbestanden in hd-tools/v10fixture/): Kaart zoeken weg; lijst uit index.json; echte Verlan-kaart laden (163 items, 29 eten/134 drinken, Steak frites € 29,00, Oyster € 4,50/5,50; opgeslagen in app-formaat); notitie; bestellen → € 29,00; zoeken "champagne" + snelknoppen; opnieuw tikken overschrijft (1 kaart, nieuwe prijs/datum); reserve bij mislukt bestand; ongeldige regels; ?kaart= zonder bezoek + URL schoon + herladen herhaalt niet; #kaart= met bezoek → grote vraag (Nee/Ja); onbekende id; abort/404/lege lijst/kapotte JSON → sectie verborgen; geen overloop 375/820 × 3 lettermaten (nieuw bezoek, Bestellen met Verlan, vraag); versie 10. Regressie: e2e6 ALLES OK, e2e7 40/41 en e2e8 73/74 (enige 'fout' = verwachte oude versie), ptest 38/38, node --check OK. e2e9 vervalt (alleen Kaart-zoeken; het overslaan-met-naam-deel zit nu in e2e10). Screenshot v10_kaarten.png (375 px, 3 voorbeeldkaarten).
- Bestanden: index.html = index-v10.html; nieuw menus/index.json (+ menus/verlan-amsterdam.json).

## v11 (2026-10-26) – Voor/Hoofd/Na, alles aanpasbaar, laatste kaart geldt
Verzoek (gebruiker na testen): "De Menukaart Eten onderverdelen naar voor, hoofd en nagerechten. Menukaart items dienen allemaal aangepast te kunnen worden, zowel qua omschrijving als qua prijs. Bij de voorbereiding wordt een kaart vanaf Internet ingevoerd. In restaurant kan de kaart anders zijn. Op dat moment een foto of pdf die vanaf dat moment geldig is. Dus aanpassen dat laatste invoer geldig is."
- **Soorten** (`category` in app, `cat` in JSON): `voor`, `hoofd`, `bij`, `na`, `eten` (= Overig eten, ook alle oude kaarten), `drinken`, `extra`. `normCat()` zet alles om (aliassen, onbekend → eten). `FOOD_GROUPS` = volgorde op Bestellen: Voorgerechten, Hoofdgerechten, Bijgerechten, Nagerechten, Overig eten; daarna Drinken (met de zelf-in-te-vullen drinkregels, ongewijzigd) en Extra. Lege groepen worden niet getoond. Heeft een kaart alleen `eten` (oude kaart), dan heet de groep gewoon "Eten". Kopjes hebben id `grp-<cat>`.
- **Rekening/Overzicht** blijven Eten / Drinken / Extra (`billCat()`: voor/hoofd/bij/na/eten → Eten).
- **Snelknoppen** (> 20 items): alleen de aanwezige groepen (Voor, Hoofd, Bij, Na, Overig) + 🥤 Drinken + ✍️ Zelf invullen. Ook in de aanpas-stand (dan zonder Zelf invullen).
- **✏️ Aanpassen**: álle items (ook van internetkaarten) in groepen, met zoekvak; per item naam, prijs, ✕ weghalen en grote soortknoppen `.grp-pills [data-grp]` Voor/Hoofd/Na/Bij/Overig/Drinken (≥52px, gekozen = blauw met ✓). "+ Gerecht of drankje toevoegen" boven én onder; nieuw item komt bovenaan onder "Nieuw op de kaart" (standaard Hoofd) tot Klaar. Elke wijziging (`commitMenuEdits`, weghalen) wordt meteen bewaard in het bezoek én in de bewaarde kaart van het restaurant (`syncRestaurant`). Een aangepaste internetkaart krijgt de notitie "✏️ Eigen kaart, aangepast 26 okt" i.p.v. de internet-notitie.
- **Al bestelde regels veranderen niet** als de kaart wordt aangepast of vervangen: ze houden hun eigen naam en prijs (aanpassen kan nog op de rekening). Komt een bestelde regel niet meer overeen met de kaart, dan staat hij op Bestellen onder "Besteld, niet (meer) op de kaart" (was "Buiten de kaart besteld"), met − / +. Alleen de soort gaat mee als naam en prijs gelijk blijven.
- **📷 Nieuwe kaart (foto of PDF)** (`#newCardBtn` → `#newCardInput`, `handleNewCardFiles` → `replaceVisitMenu`): grote knop bovenaan de menukaart op Bestellen (ook als het bezoek nog geen kaart heeft). Gevonden gerechten **vervangen** de kaart van dit bezoek én de bewaarde kaart van het restaurant (laatste invoer geldt); notitie "📷 Kaart van foto, 26 okt" / "📄 Kaart van PDF, 26 okt" / "📷 Kaart van foto en PDF, …" (`visit.cardFrom = {kind, date}`, `#cardNote`, groen) i.p.v. de internet-notitie. Niets gevonden of geen internet → oude kaart blijft, melding. "➕ Nog een foto of PDF bij deze kaart" (`#moreMenuBtn`) voegt nog steeds toe i.p.v. te vervangen.
- **PDF**: overal waar je een kaart kiest (nieuw bezoek, nieuwe kaart, erbij) mag ook een PDF. pdf.js 3.11.174 (legacy UMD-build, jsDelivr) wordt pas geladen als je een PDF kiest (`loadPdfJs`). Per pagina (max 30) de tekstlaag (`getTextContent`); twee kolommen worden herkend en apart gelezen (`pdfItemsToText`); doorlopende regels worden samengevoegd (`MenuParser.joinWrapped`, alleen voor PDF-tekst: afgebroken omschrijvingen, prijs op eigen regel, "Per 100 gram", "2,5 p.p", "CHEFS MENU / 3 courses 55" → "Chefs menu (3 courses)"). Pagina met < 40 letters (scan) → tekenen op canvas (max ~2000 px breed, ≤12 MP) en lezen met Tesseract. PDF's worden niet als foto bewaard.
- **Scanner/parser** (tussen `// ==PARSER-START==` en `// ==PARSER-END==`): kopjes in NL/EN/FR (`FOOD_HEADS`, `HEAD_PHRASES`, `headerCategory`): Voorgerechten/Starters/Appetizers/Entrées/Amuse/Hapjes → voor; Hoofdgerechten/Mains/Main courses/Plats/To share → hoofd; Bijgerechten/Sides → bij; Nagerechten/Desserts/Kaas/Fromages → na; Dranken/Wijnen/Bier/… → drinken; andere eten-kopjes (Lunch, Chefs menu, …) → eten. Gerechten erna krijgen die soort; drankwoorden onder een eten-kopje blijven drinken (bv. Irish coffee onder Nagerechten), behalve lange gerechtomschrijvingen. "Kaas" met prijs op de volgende regel is een gerecht. Namen tot 90 tekens (op een woordgrens). Geen lookbehind (iOS 15).
- **Laatste invoer geldt / internetkaart**: bewaarde kaart heeft nu `updated` (laatste wijziging van de gerechten), `origin` ('web' | 'foto' | 'pdf' | 'mix' | 'hand'), `used` (laatst gekozen; "Eerder bezocht" sorteert hierop). Is de eigen kaart (origin ≠ web) van dezelfde dag of nieuwer dan `retrieved` van de internetkaart, dan zegt de knop onder Kaarten klaargezet "je eigen kaart van 26 okt is nieuwer – tik om toch de internetkaart te laden" (`.wb-newer`), en tikken vraagt groot: "Je eigen kaart van 26 okt is nieuwer dan de internetkaart (25 okt). Toch de internetkaart laden?" [Ja, internetkaart laden] [Nee, mijn eigen kaart gebruiken]. Ja → internetkaart overschrijft de bewaarde kaart; Nee → bezoek met de eigen kaart. Internetkaart en eigen kaart van hetzelfde restaurant worden gekoppeld via id `web-<id>` of dezelfde naam. `askBig(text, {yes, no, sub})`.
- Bewaarde kaart in localStorage `hapjeDrankjeMenus`: `{ id, name, items:[{name, price, category}], photoIds, updated, used, origin, online?, retrieved?, source?, webLoaded? }`. Bezoek: `cardFrom?: {kind, date}` naast `webRetrieved?`.
- `menus/verlan-amsterdam.json`: de 29 gerechten ingedeeld volgens de PDF-kopjes: APPETIZERS (13) en STARTER (3) → voor; CHEFS MENU (1, keuze: hoofd), MAIN (3) en TO SHARE (3) → hoofd; SIDES (3) → bij; DESSERTS (3, incl. kaas) → na. Namen, prijzen, volgorde en de 134 drankjes ongewijzigd.
- `APP_VERSION='11'`. Gegevens compatibel (oude `eten`-kaarten en -bezoeken werken; ze tonen "Eten" tot je ze indeelt).
- Tests (/workspace/hd-tools): e2e11.js 105/105 (Verlan in groepen + volgorde + snelknoppen; aanpassen naam/prijs/soort/weghalen/toevoegen, bewaard na herladen, bestelde regels ongewijzigd; internet vs. nieuwere eigen kaart (Nee/Ja); nieuwe kaart met foto (OCR), tekst-PDF en gescande PDF; mislukte foto houdt oude kaart; nieuw bezoek met PDF; echte Verlan-PDF 29/29 gerechten met juiste prijs; oude eten-kaart; geen zijwaarts scrollen 375/820 × 3 lettermaten; versie 11). ptest11.js 178/178 (NL/EN/FR-kopjes, joinWrapped). Regressie: e2e6v11 ALLES OK (e2e6 met v11-verwachtingen), e2e7/e2e8 alleen de oude-versiecheck, e2e10 68/73 (rest = verwachte v10-dingen: 29 eten, 3 snelknoppen, versie 10), ptest 32/38 (6 = voorgerecht/hoofdgerecht heet nu voor/hoofd i.p.v. eten). Fixtures in hd-tools/v11fixture/ (mkfix11.js).
- Bestanden: index.html = index-v11.html; menus/verlan-amsterdam.json bijgewerkt; CONTEXT.md.


## v12 (2026-10-26) – Naam en prijs direct op de kaart veranderen
Verzoek (gebruiker): "Ziet er goed uit, alleen het veranderen van omschrijving en prijs zou ik graag direct op de menukaart doen."
- **Bestellen**: tik op de **naam** of de **prijs** van een gerecht → die regel wordt een veranderblok (`#inlineEdit`, geel met oranje rand): groot veld **Naam** (`#ieName`), groot veld **Prijs in €** (`#iePrice`, `inputmode="decimal"`, komma of punt), knoppen **✓ Klaar** (`#ieSave`) en **Annuleer** (`#ieCancel`) (≥60px). Tik op naam → focus naamveld; tik op prijs → focus prijsveld (alles geselecteerd). Enter/Return = bewaren, Escape = annuleren. Maar één regel tegelijk: tik je op een andere regel, dan gaat de vorige dicht zonder te bewaren. "Prijs ontbreekt – tik hier" opent nu ook dit blok (prijsveld) i.p.v. Aanpassen.
- **Let op – gedrag veranderd**: tikken op naam/prijs bestelt niet meer (was: +1). Bestellen gaat met **+**, het aantal typen met tik op het getal; − / + / getal openen nooit het veranderblok.
- **Vindbaar**: één regel boven de kaart `#menuTip` "Tik op een naam of prijs om die te veranderen." en een klein grijs ✎ (`.or-pen`) achter elke prijs. Niet in de aanpas-stand.
- **Bewaren** (`saveInlineEdit`) = zelfde logica als Aanpassen: kaart van het bezoek + bewaarde kaart van het restaurant (`syncRestaurant` → `updated` nieuw, `origin` 'hand' bij een internetkaart; foto/pdf blijft foto/pdf), `markHandEdited()` → internet-notitie wordt "✏️ Eigen kaart, aangepast <datum>". Lege naam → oude naam blijft (melding). Ongeldige/lege prijs → rode melding in het blok ("Dat is geen geldige prijs. Typ bijvoorbeeld 12,50." / "Vul een prijs in, bijvoorbeeld 12,50."), blok blijft open; verdwijnt bij typen. Een gerecht zonder prijs mag zonder prijs blijven.
- **Al besteld** (koppeling bestelregel ↔ kaart gaat via naam + prijs, `sameDish`): 
  - Alleen de naam veranderd → de bestelde regels krijgen de nieuwe naam mee (prijs gelijk, totaal gelijk), zodat het aantal bij − / + blijft staan. Geen vraag.
  - Prijs veranderd en er is iets van besteld → grote vraag (`askBig`) "Ook de prijs van wat je al besteld hebt aanpassen?" met uitleg "Je hebt 2× Steak besteld voor € 24,00. Nieuwe prijs op de kaart: € 26,00." **Ja** → bestelde regels krijgen nieuwe prijs (en naam), aantal blijft bij de kaartregel. **Nee** → bestelde regels houden de oude prijs (v11-regel; naam gaat wel mee) en staan onder "Besteld, niet (meer) op de kaart"; nieuwe bestellingen gaan tegen de nieuwe prijs.
- **✏️ Aanpassen** ongewijzigd (soort, ✕ weghalen, toevoegen, ook naam/prijs). Aanpassen openen of van tabblad wisselen sluit een open veranderblok.
- `APP_VERSION='12'`. Gegevens ongewijzigd/compatibel.
- Tests (/workspace/hd-tools, tegen `python3 -m http.server 8765 --directory /workspace/hapje-drankje`): **e2e12.js 79/79** (hint + ✎; − / + / getal openen geen veranderstand; naam via Enter; prijs 12,50 en 5.25; Annuleer; één tegelijk; lege naam; foute/lege prijs; besteld: Nee en Ja, Rekening € 51,20 → € 51,60 → € 58,60; naam+prijs Ja; alleen naam zonder vraag; herladen + bewaarde kaart + Aanpassen; Verlan: notitie "✏️ Eigen kaart, aangepast", bewaarde web-kaart origin hand, herladen; geen zijwaarts scrollen 375/820 × 3 lettermaten (blok, melding, vraag); versie 12, v11-app ziet "Nu vernieuwen"). Regressie: e2e11v12.js 103/105 (alleen de 2 versiechecks; = e2e11 met bestellen via + en ✎ weggefilterd), e2e6v12.js ALLES OK (= e2e6v11 met + i.p.v. tik op naam), e2e10v12 68/73 (zelfde 5 verwachte v10-dingen als bij v11), e2e7/e2e8 alleen oude-versiecheck, ptest11 178/178. Screenshot v12_inline.png (375 px).
- Bestanden: index.html = index-v12.html; CONTEXT.md. (menus/ ongewijzigd.)

## v13 (2026-10-26) – Teksten "Tik op +", Iets extra's als invulregels
Verzoek (gebruiker na testen, met foto van de gele melding): "tekst klopt niet meer, tik op een gerecht … Het veld snel toevoegen wekt verwarring bij mij, graag weglaten en vervangen door een + bij de extra regel."
- **Teksten** (sinds v12 bestelt tikken op een gerecht niet meer, maar opent het veranderblok):
  - Internetkaart: "163 gerechten en drankjes geladen. Tik op + om te bestellen." (was "Tik op een gerecht of op + om te bestellen.")
  - Scan bij nieuw bezoek: "N gerechten herkend. Tik op + om te bestellen. Klopt een naam of prijs niet? Tik op de naam of prijs om die te veranderen."
  - Nieuwe kaart (foto/PDF): "Nieuwe kaart geladen: N gerechten en drankjes. Deze kaart geldt nu voor dit restaurant. Tik op + om te bestellen. Wat je al besteld had, … oude prijs. Klopt een naam of prijs niet? Tik op de naam of prijs om die te veranderen."
  - Hint `#menuTip` "Tik op een naam of prijs om die te veranderen." ongewijzigd. Meldingen zonder gerechten ("Tik op ✏️ Aanpassen om ze zelf in te vullen") blijven. Nergens meer "Tik op een gerecht".
- **Iets extra's** (kaart in `#offMenuCards`, titel `#extraTitle`): velden quickName/quickPrice en knop "Snel toevoegen" (`addQuickItem`) zijn weg. Nu invulregels precies zoals Drinken zelf invullen: uitleg "Zoals extra saus of brood. Typ naam en prijs, en tik op + . Dan staat het meteen op de rekening.", per regel naam (placeholder "Bijv. extra saus, brood"), prijs en − [n] +, start met **2** lege regels, knop `#addExtraRowBtn` "+ Nog een regel".
  - Opslag: `visit.extraRows = [{id:'x…', name, price, priceText, qty}]` (net als `drinkRows`); bestelregel op de rekening `{name, price, category:'extra', qty, extraRow:<id>}` → Rekening onder **Extra**. Functies `ensureExtraRows/extraQty/syncExtra/extraRowHtml/renderExtraRows/addExtraRow/findExtraRow/setExtraQty`; DOM `#extraRows .dr-row.xr-row[data-xr]`, velden `[data-xrf=name|price]`, aantal `data-qx`. `renderMenuList()` tekent ook de extra-regels.
  - Gedrag = drinkregels: + zonder naam/prijs → rode regel "Nog niet op de rekening: vul naam en prijs in"; naam/prijs wijzigen past de rekeningregel aan; − naar 0 haalt de regel van de rekening (met Ongedaan maken), de invulregel blijft staan; tik op het getal = aantal typen. Bewaard per bezoek (herladen ok).
  - `sameDish()` en "Besteld, niet (meer) op de kaart" negeren `extraRow`-regels (net als `drinkRow`).
  - **Oude gegevens**: extra's van vóór v13 (category 'extra' zonder extraRow) blijven gewoon op de Rekening onder Extra, in Eerder, en op Bestellen onder "Besteld, niet (meer) op de kaart" met − / +. Een lopend bezoek krijgt er 2 lege extra-regels bij.
- `#offMenuCards { padding-bottom: 4.5rem }` zodat "+ Nog een regel" (onderaan de pagina) niet onder de zwarte "Ongedaan maken"-balk valt.
- Het Fooi-kaartje op Bestellen noemde Snel toevoegen niet; geen andere verwijzingen gevonden.
- `APP_VERSION='13'`. Gegevens compatibel.
- Tests (/workspace/hd-tools, tegen `python3 -m http.server 8765 --directory /workspace/hapje-drankje`): **e2e13.js 59/59** (geen "Tik op een gerecht" in bestand/scherm, gele melding bij echte Verlan-kaart; Snel toevoegen weg; 2 regels; + leeg → niets + waarschuwing; Extra saus 2,50 → 1× en 2× onder Extra; Brood 4.75, totaal € 9,75; prijs wijzigen; herladen; − naar 0 weg + Ongedaan maken; + Nog een regel met focus; aantal typen; drinkregels apart; oud afgerond bezoek en lopend v12-bezoek met extra's; geen zijwaarts scrollen 375/820 × 3 lettermaten (Bestellen + Rekening); versie 13, v12-app ziet "Nu vernieuwen"). Regressie: e2e12 78/79, e2e11v12 103/105 (alleen versiechecks), e2e6v12 ALLES OK, e2e10v12 68/73 (zelfde 5 verwachte v10-dingen), ptest11 178/178, node --check OK. Screenshot v13_extra.png (375 px; shot13.js).
- Bestanden: index.html = index-v13.html; CONTEXT.md. (menus/ ongewijzigd.)

## v14 (2026-10-26) – Eén bezoek apart wissen op Eerder
Verzoek (gebruiker): "Ja graag ook een bezoek apart wissen".
- **Eerder** (`#tab-geschiedenis`, `#historyList`): elk bezoek is nu een rij `.history-row` = de bestaande knop `.history-item[data-visit]` (tik = rekening openen, ongewijzigd) + rechts een ✕-knop `.qty-btn.remove[data-delvisit]` (zelfde ronde rode ✕ als bij "Eerder bezocht", aria-label "Bezoek <naam>, <datum> wissen"). Uitleg: "Tik op een bezoek om de rekening te bekijken. Tik op ✕ om een bezoek te wissen." Let op voor tests: `.history-item:nth-child(n)` → `.history-row:nth-child(n) .history-item`.
- **Wissen met Ongedaan maken** (geen vraag; de app had al de zwarte balk): ✕ haalt het bezoek meteen uit `hapjeDrankjeVisits` (bewaard) en toont 8 s de balk 'Bezoek "<naam>, <datum>" gewist' [↩ Ongedaan maken]. Ongedaan maken zet het bezoek op dezelfde plek terug. Tikken op ✕ opent het bezoek niet.
- **Foto's** pas definitief gewist (IndexedDB `deletePhoto`) als de balk weg is (na 8 s, bij een volgende ✕ of een ander weghalen met Ongedaan maken, of bij `pagehide`). Het gaat om foto's met `visitId` = bezoek en de `menuPhotoIds` van het bezoek. Veiligheidsnet: de foto-id's staan zolang in localStorage `hapjeDrankjePendingPhotoDelete`; bij opstarten worden die alsnog gewist (`purgePhotos`). Ongedaan maken haalt ze uit die lijst.
- **Bewaarde menukaarten blijven** (naam, gerechten, alles); alleen verwijzingen naar gewiste foto's gaan uit `photoIds` (zoals bij Alles wissen).
- **Lopend bezoek** (`hapjeDrankjeCurrent` met hetzelfde id, kan normaal niet maar wordt afgevangen): wordt ook gewist, app gaat naar Overzicht; Ongedaan maken zet het terug. Staat de Rekening op het gewiste bezoek (`billVisitId`), dan wordt die losgelaten.
- **Alles wissen** ongewijzigd (met vraag); sluit ook een openstaande Ongedaan-balk en wachtlijst. `#tab-geschiedenis { padding-bottom: 4.5rem }` zodat "Alles wissen" niet onder de balk valt.
- Functies: `visitLabel, visitPhotoIds, deleteVisit, undoVisitDelete, finalizeVisitDelete, purgePhotos, read/writePendingPhotoIds`; `undoState.kind === 'visit'`; `hideUndo()` maakt een wachtend bezoek-wissen definitief, `showUndo()` roept eerst `hideUndo()` aan.
- `APP_VERSION='14'`. Gegevens compatibel.
- Tests (/workspace/hd-tools, tegen `python3 -m http.server 8765 --directory /workspace/hapje-drankje`): **e2e14.js 60/60** (✕ per bezoek ≥40px, zelfde stijl, uitleg; ✕ opent niet, geen vraag; alleen dat bezoek weg + na herladen; Ongedaan maken op dezelfde plek, foto's blijven; definitief: foto's van dat bezoek weg, andere blijven, kaarten ongewijzigd; herladen tijdens balk + opstart-opruiming; tweede ✕ maakt eerste definitief; rekening van gewist bezoek; lopend bezoek → Overzicht + Ongedaan; Alles wissen (niet bedekt door balk); geen zijwaarts scrollen 375/820 × 3 lettermaten, met en zonder balk; versie 14, v13-app ziet "Nu vernieuwen"). Regressie: e2e13 58/59, e2e12 78/79 (alleen versiecheck), e2e11v12 103/105 (versiechecks), e2e6v14.js ALLES OK (= e2e6v12 met nieuwe rij-selector), ptest11 178/178, node --check OK. Screenshot v14_eerder.png (375 px).
- Bestanden: index.html = index-v14.html; CONTEXT.md. (menus/ ongewijzigd.)

## v15 (2026-10-26) – Alleen A+, 3 lettermaten erbij
Verzoek (gebruiker): "De standaard letter grootte is prima. Laat de knop om letters kleiner te maken weg en vergroot de mogelijkheid om letters groter te maken met 3 stappen extra."
- **A− weg** (`#tsMinus` bestaat niet meer). Kop: "Letters" + één knop `#tsPlus`.
- **Maten** (`TEXT_SIZES`, class op `<html>`, localStorage `hapjeDrankjeTextSize`): `groot` 22px (= standaard = kleinste, toast "Normaal"), `extra` 25px, `enorm` 28px, **nieuw** `enorm2` 31px, `enorm3` 34px, `enorm4` 37px. Was 2 stappen boven standaard (+3px per stap), nu 5 (zelfde +3px). Max 37px = 1,68× standaard (was 28px).
- **A+**: elke tik één stap groter (toast "Letters: Groter (n van 5)"). Op het grootst heet de knop **"Normaal"** (wit met blauwe letters, `.ts-reset`, aria-label "Letters terug naar normaal"; toast "Letters: Grootst (5 van 5). Tik op Normaal om terug te gaan."). Tik op Normaal → terug naar 22px, knop weer "A+", toast "Letters: Normaal"; de opgeslagen waarde wordt dan verwijderd.
- **Oude/onbekende waarde** (bv. iets kleiners uit een oude versie) → normaal, en wordt bij opstarten uit localStorage gehaald. Oude `groot/extra/enorm` blijven werken. Het vroege `<head>`-script kent de nieuwe maten (geen flits).
- **Opmaak bij grote maten** (zodat niets zijwaarts uitsteekt op 375/390 px): `.header-row` mag omslaan (Letters-knop rechts op een tweede regel); kop-onderdelen begrensd met `min(…, vw)` (h1 6vw, Letters 5vw, knop max 60px hoog / 6vw letter, Totaal 6vw/11vw, header-padding 3.5vw); `.qty-btn` max 17vw, `.qty-num` max 19vw breed, gat max 2.5vw (op 375 px: 64 px ronde knoppen; bij normale maat en op iPad ongewijzigd); `.menu-group`/`.bill-group` max 7vw; vanaf `enorm2` `hyphens: auto` + `overflow-wrap: break-word` op body (lange woorden zoals "restaurantbezoek" breken af).
- `APP_VERSION='15'`. Gegevens compatibel.
- Tests (/workspace/hd-tools, tegen `python3 -m http.server 8765 --directory /workspace/hapje-drankje`): **e2e15.js 93/93** (A− weg, label Letters, standaard 22px; 5× A+ → 25/28/31/34/37, op het grootst knop "Normaal"; herladen houdt 37px; Normaal → 22px + A+; tussenstap bewaard; "klein"/"normaal"/"small"/"xx" → 22px en opgeruimd; oude groot/extra/enorm ok, van enorm verder naar 31; geen zijwaarts scrollen Overzicht/Bestellen/veranderblok/Rekening/Eerder op 390 × alle 6 maten en 375/820 op de grootste; tabbalk past; versie 15, v14-app ziet "Nu vernieuwen"). Screenshots v15_*.png (390 px, grootste maat), v15_bestellen_375.png (shot15.js). Regressie: e2e14 59/60, e2e13 58/59, e2e12 78/79, e2e11v12 103/105 (alleen de vaste versiechecks), e2e6v14 ALLES OK (knoppen ≥ 52px), ptest11 178/178, node --check OK.
- Bestanden: index.html = index-v15.html; CONTEXT.md.

## v16 (2026-10-26) – A− en A+ terug: 2× kleiner, normaal, 2× groter
Verzoek (gebruiker na v15): "Toch niet fijn. Herintroduceer kleiner en groter dan normaal 2x".
- **Kop** weer "Letters A− A+" (`#tsMinus`, `#tsPlus`, zoals v14). Het v15-gedrag (knop wordt "Normaal", rondje terug) is weg.
- **5 maten** (`TEXT_SIZES`, `TS_NORMAL = 2`, class op `<html>`): `klein2` 16px, `klein` 19px, `groot` 22px (normaal, standaard; niets opgeslagen), `extra` 25px, `enorm` 28px. Stap 3px. Toasts: "Letters: Kleiner (2 van 2)", "Kleiner (1 van 2)", "Normaal", "Groter (1 van 2)", "Groter (2 van 2)".
- A− staat uit en is grijs (opacity 0.4, `.ts-btn:disabled`) op 16px; A+ op 28px. Geen rondje.
- **Oude waarden**: v15 `enorm2/enorm3/enorm4` (31/34/37px) → `enorm` 28px (ook in localStorage herschreven; het `<head>`-script doet hetzelfde, geen flits). Onbekend → normaal (opgeruimd). `groot` → opgeruimd (= normaal).
- **v15-opmaakbeveiliging blijft** (kop mag omslaan, `min(…, vw)`-grenzen voor kop, − / aantal / +, groepskopjes); afbreken van lange woorden (`hyphens`) nu bij `enorm` (28px, de grootste). Nieuw: bij `klein`/`klein2` blijven alle knoppen ≥ 44px (− / aantal / +, ✓-vakjes, ✕, A−/A+).
- `APP_VERSION='16'`. Gegevens compatibel.
- Tests (/workspace/hd-tools): **e2e16.js 129/129** (kop "Letters A− A+"; A+ 25 → 28, A+ uit/grijs, doet niets meer; A− terug naar Normaal; A− 19 → 16, A− uit/grijs; herladen houdt 16/19/28; v15 enorm2/3/4 → 28; xx/small/normaal/groot → 22; knoppen ≥ 44px bij 16/19px; geen zijwaarts scrollen Overzicht/Bestellen/veranderblok/Rekening/Eerder op 375 en 390 × 5 maten en 820 × 16/28; tabbalk en A−/A+ in beeld; geen JS-fouten; v15-app ziet "Nu vernieuwen"). Screenshots v16_*.png (390, 28px), v16_bestellen_375_klein2.png / _enorm.png (shot16.js / shot16b.js). Regressie: e2e14 59/60, e2e13 58/59, e2e12 78/79, e2e11v12 103/105 (alleen vaste versiechecks), e2e6v14 ALLES OK, ptest11 178/178.
- Bestanden: index.html = index-v16.html; CONTEXT.md.

## v17 (2026-10-26) – Snelknoppen op Bestellen blijven in beeld
Verzoek (gebruiker): "De navigatie levert wat problemen. Als ik tik op bijv. hoofdgerechten hoe kan ik dan terug naar voorgerecht en naar nagerecht etc."
- **Plakkende balk** (`#jumpBar.jump-bar`, nog steeds alleen bij > 20 items, alleen knoppen voor groepen die er zijn): `position: sticky; top: var(--topbar-h)`. `--topbar-h` = hoogte van de blauwe kop (`.topbar`, kan omslaan bij grote letters), bijgehouden door `syncTopbarHeight()` (ResizeObserver op `.topbar`, resize, na elke `renderMenuCard`). Witte achtergrond, lijn + schaduw onder; balk loopt over de hele kaartbreedte (marge −0.75rem). Gerechten schuiven eronder door. Plakt binnen `#menuCard` (dus niet meer bij "Iets extra's").
- **Eén rij die zelf zijwaarts schuift** (`overflow-x: auto`, geen schuifbalk zichtbaar, `flex-wrap: nowrap`); knoppen ≥ 52px hoog (`max(52px, 2rem)`, zoals v11 vroeg; letter `min(1rem, 6vw)`), balk ca. 66–76px hoog. De pagina zelf schuift nooit zijwaarts.
- **↑ Boven** (`data-jump="menuTop"`, `.jb-top`) als eerste knop: begin van de menukaart (kop "Menukaart", zoekvak) net onder de blauwe kop.
- **Springen**: `window.scrollTo` met berekende afstand (kop + balk + 6px), zodat het kopje (Voorgerechten, …, Drinken, Drinken zelf invullen) net onder de balk staat. Zoeken wordt gewist (zoals voorheen).
- **Waar ben ik** (`updateJumpActive()`, scroll-listener met requestAnimationFrame): de knop van het stuk waarvan het kopje boven of net onder de balk staat is blauw gevuld (`.on`, `aria-current`); bovenaan (nog geen kopje gepasseerd) is ↑ Boven gemarkeerd. Onderaan de pagina (laatste stuk kan niet helemaal omhoog) blijft de aangetikte knop gemarkeerd (`jumpLast`). De gemarkeerde knop wordt binnen de balk in beeld geschoven (`scrollLeft`, niet de pagina).
- Aanpas-stand: zelfde balk, zonder Zelf invullen, Drinken → `grp-drinken`.
- `APP_VERSION='17'`. Gegevens ongewijzigd.
- Let op voor oudere tests: hun zijwaarts-scrollen-check telt ook knoppen binnen een schuifbalk; e2e17 slaat elementen binnen een `overflow-x: auto`-ouder over (en checkt dat de balk zelf binnen het scherm valt).
- Tests (/workspace/hd-tools): **e2e17.js 233/233** (alleen aanwezige groepen + ↑ Boven; met/zonder Bij, Overig; per 390 normaal, 375 16px, 375 28px, 390 25px, 390 19px: één rij, knoppen ≥ 52px, balk ≤ 80px, sticky/wit/eigen schuifbalk, plakt direct onder de kop na scrollen, ligt boven de gerechten; elke knop (Voor, Hoofd, Bij, Na, Drinken, Zelf invullen) landt met het kopje net onder de balk en markeert die knop (zichtbaar in de balk); van onderaan terug naar Voor; zelf scrollen werkt de markering bij; ↑ Boven; lettergrootte wisselen op Bestellen (kop wordt hoger/lager) → balk blijft direct onder de kop; zoeken wordt gewist; aanpas-stand; echte Verlan-kaart; geen zijwaarts scrollen 375/390 × 5 maten; geen JS-fouten; versie 17, v16-app ziet "Nu vernieuwen"). Regressie: e2e16 128/129, e2e14 59/60, e2e13 58/59 (alleen versiecheck), e2e6v14 ALLES OK, ptest11 178/178, node --check OK. e2e12v17 78/79 en e2e11v17 102/105 = e2e12 / e2e11v12 met de schuifbalk-bewuste zijwaarts-check (zonder die aanpassing melden ze de knoppen in de schuivende balk als "zijwaarts"); e2e11v17 mist verder alleen de versiechecks en de oude knoppenlijst zonder ↑ Boven. Screenshots v17_snelknoppen_normaal.png / _enorm.png (390 px, na tik op Hoofd).
- Bestanden: index.html = index-v17.html; CONTEXT.md.

## v18 (2026-11-09) – Rekening delen via WhatsApp
Verzoek (gebruiker na geslaagde restauranttest): "wanneer de rekening is gecontroleerd en akkoord bevonden een knop … waarmee ik de rekening in haar geheel naar derden via whatsapp kan versturen."
- **Wanneer**: op Rekening, onder het restaurantbedrag, zodra `#billAmount` een getal is (`billShareReady`: items aanwezig én `typeof v.billAmount === 'number'`). Ook bij verschil ("hoger"/"lager"), niet alleen bij ✓ Klopt. Verdwijnt weer als het bedrag leeggemaakt wordt. Zelfde kaart op een afgerond bezoek uit Eerder.
- **Knoppen** (`#billShare.share-box`): grote groene **📤 Stuur via WhatsApp** (`#shareWaBtn`, `.share-wa`) + **Andere app / kopiëren** (`#shareOtherBtn`) + inklapbaar **Bekijk tekst** (`#sharePreview` → `#shareText`).
- **Bericht** (`billShareText`, plain text, WhatsApp `*vet*`): restaurantnaam + datum; per groep 🍽 Eten / 🥤 Drinken / ➕ Extra de regels `n × naam — € x,xx` en subtotaal; `*Totaal*`; fooi + `*Te betalen incl. fooi*` indien fooi; rekening restaurant (+ ✓ Klopt / hoger / lager); bij >1 persoon het bedrag per persoon (naar boven, zelfde als in de app); eindigt op `Gemaakt met Hapje/Drankje`. Bedragen via `formatEuro` (komma).
- **WhatsApp** (`shareBillWhatsApp`): iPhone/iPad → `whatsapp://send?text=` en als de app niet opent binnen 1,2 s `https://wa.me/?text=`; overige → `window.open(https://wa.me/?text=)`. Tekst met `encodeURIComponent` (€ → `%E2%82%AC`, regeleinden → `%0A`; `*` mag ongecodeerd blijven).
- **Andere app** (`shareBillOther`): `navigator.share({text})` als die er is, anders klembord (`navigator.clipboard` / `execCommand('copy')`) + toast.
- `APP_VERSION='18'`. Gegevens ongewijzigd. Alleen `index.html` wordt gepubliceerd (lokale `menus/` niet relevant).
- Tests (/workspace/hd-tools): **e2e18.js 59/59** (knop pas na bedrag, verdwijnt bij leeg, blijft bij verschil; bericht met eten/drinken/extra/fooi%/eigen fooi/personen/Klopt/hoger; encoding wa.me; Android-open stub; kopiëren + navigator.share; Bekijk tekst; geen zijwaarts scrollen 375/390 × 5 lettermaten met preview open; afgerond bezoek; geen JS-fouten; versie 18, v17-app ziet "Nu vernieuwen"). Screenshot v18_delen.png (390 px). Regressie: e2e17 232/233, e2e16 128/129 (alleen versiecheck).
- Bestanden: index.html = index-v18.html; CONTEXT.md.

## v19 (2026-11-09) – Zelf invullen: één plek, keuze Eten/Drinken
Verzoek (gebruiker na gebruik): veel buiten-de-kaart-regels, eten en drinken door elkaar via drinkregels + Iets extra's. Keuze: één invulplek met per regel 🍽 Eten | 🥤 Drinken.
- **Sectie** op Bestellen: `Zelf invullen (buiten de kaart)` (`#grp-zelf`, `#selfFill`), ~6 regels + **+ Nog een regel** (`addSelfRow`). Drink-"zelf invullen" en de kaart **Iets extra's** (`#extraTitle` / `#extraRows`) zijn weg. Herkende drankjes van de kaart blijven onder Drinken; daaronder komt deze sectie.
- **Per regel** (`.sr-row`, opslag `visit.selfRows`): naam, prijs, − / aantal / +, en grote soortknoppen `.cat-tog` **🍽 Eten** | **🥤 Drinken** (≥44px, blauw als gekozen, mogen onder naam/prijs staan). Standaard nieuwe lege regel: Eten. Tik op de andere soort → `setSelfCategory` zet `row.category` en past meteen de rekeningregel aan (`category` eten/drinken).
- **Rekening / WhatsApp**: zelf-ingevulde regels onder Eten of Drinken (`billCat`). Groep **Extra** blijft alleen voor oude regels met `category: 'extra'` (zonder of met legacy `extraRow`).
- **Snelknop** ✍️ Zelf invullen → `#grp-zelf` (ongewijzigd doel, nieuwe titel).
- **Migratie** (`ensureSelfRows`): bestaande `drinkRows` → selfRows met category `drinken` (zelfde id's, koppeling via `drinkRow`/`selfRow`); gevulde `extraRows` → selfRows met category `extra` (blijven onder Extra tot je Eten/Drinken kiest); lege extra-regels weg; aanvullen tot 6× Eten. Items houden hun bedragen/aantallen.
- Functies: `newSelfRow/ensureSelfRows/selfRowHtml/selfFillHtml/syncSelf/setSelfQty/setSelfCategory`; `drinkSectionHtml` toont kaart-drinken + selfFill. `sameDish` negeert ook `selfRow`.
- `APP_VERSION='19'`. Tip-gedrag ongewijzigd. Alleen index.html publiceren.
- Tests (/workspace/hd-tools): **e2e19.js 54/54** (Iets extra's weg; 6× Eten; bestellen eten+drinken; wisselen op de rekening; WA-tekst zonder Extra; + regel; migratie drink+extra+legacy Extra; knoppen ≥44 en geen zijwaarts scrollen 375/390 × 5 maten; versie 19, v18 ziet "Nu vernieuwen"). Screenshot v19_zelf.png. Regressie: e2e18 58/59, e2e17 232/233 (alleen versiecheck).
- Bestanden: index.html = index-v19.html; CONTEXT.md.

## v20 (2026-11-09) – App-link delen via WhatsApp
Verzoek (gebruiker): "Nog een puntje, kan je nog een knop toevoegen waarmee ik de link via whatsapp naar derden kan sturen."
- **Overzicht** (`#tab-overzicht`, altijd zichtbaar, ook met lopend bezoek): kaart `#appShare` **App delen** met grote knop **📲 Deel de app via WhatsApp** (`#shareAppWaBtn`) en **Andere app / kopiëren** (`#shareAppOtherBtn`).
- **Bericht** (`appShareText`, vaste URL `APP_SHARE_URL`, nooit `location.href`):
  `*Hapje/Drankje* – handige app om je restaurantrekening te controleren.`
  `Open deze link in Safari of Chrome:`
  `https://peterhanselman-star.github.io/hapje-drankje/`
  `Tip: zet hem op je beginscherm via de deelknop → "Zet op beginscherm".`
- **Versturen**: hergebruikt v18-mechanisme via gedeelde helpers `shareViaWhatsApp(text)` / `shareTextOther(text)` (iOS: whatsapp:// → wa.me; anders wa.me; andere app: navigator.share of klembord). Rekening-delen (`shareBillWhatsApp` / `shareBillOther`) gebruikt dezelfde helpers.
- Knoppen ≥ 44px; past op 375/390 bij alle 5 lettermaten.
- `APP_VERSION='20'`. Gegevens ongewijzigd.
- Tests (/workspace/hd-tools): **e2e20.js 37/37** (knoppen op Overzicht; exacte tekst + vaste URL; encoding/wa.me; kopiëren + share; bij lopend bezoek; geen zijwaarts scrollen 375/390 × 5 maten; rekening-delen intact; versie 20, v19 ziet "Nu vernieuwen"). Screenshot v20_delen.png. Regressie: e2e19 53/54 (alleen versiecheck).
- Bestanden: index.html = index-v20.html; CONTEXT.md.

## v21 (2026-11-09) – Beveiliging, verschiltekst, compact Bestellen, labels, back-up, slim verschil
Ronde 1 (goedkeuring gebruiker na evaluatie zoon): zeven punten.
1. **pdf.js** 3.11.174 → **4.10.38** legacy `.mjs` (jsDelivr), geladen met `import()`; `getDocument({…, isEvalSupported: false})` (CVE-2024-4367).
2. **"De rekening is gecontroleerd"** alleen bij ✓ Klopt. Bij verschil: "Er is een verschil van € x,xx" + uitleg (rood hoger / groen lager). Zelfde lijn op Eerder ("Verschil € …") en in de deel-intro op Rekening. WhatsApp-bericht houdt ✓ Klopt / hoger / lager (geen "gecontroleerd" bij verschil).
3. **"Zelf invullen (buiten de kaart)"** → overal **Zelf invullen**. Oude kaart "Buiten de kaart" verwijderd (vervangen door Zelf invullen).
4. **Compacter Bestellen**: inklapbare bezoek-instellingen, zoeken achter "🔍 Zoeken", tip in de kop weg, Nieuwe-kaart-knop onder de lijst, dichtere rijen (naam + prijs/−/+). Layout 390×844 @22px: eerste gerecht ~38% (was ~85%), ≥5 zichtbaar zonder snelknoppen; met snelknoppen ~46%/≥3. Knoppen −/+ ≥44px.
5. **Labels**: totaal **"👁 Tot nu toe – tik hier"** / open **"Tot nu toe"**; modus **"🏠 Vooraf schatten"** / **"🍽 Echt bestellen"**; badge **"Vooraf geschat"**.
6. **Back-up** op Eerder: 💾 Bewaar / 📂 Zet terug. JSON met visits, current, menus, settings; **geen foto's** (staat in de notitie). Export via `navigator.share({files})` of download. Import: vraag Samenvoegen (op id) vs Alles vervangen.
7. **Slim verschil**: als |diff| = 1× of 2× prijs van een bestelde regel of kaartitem → tip "Het verschil is precies de prijs van … Misschien extra gerekend?" / "… vergeten op de rekening?" (max 3; alleen in de app, niet in WhatsApp).
- `APP_VERSION='21'`.
- Tests: **e2e21.js 46/46**. Screenshots v21_bestellen_22.png / _28.png. Regressie: e2e20 36/37, e2e19 53/54, e2e18 58/59 (alleen versiecheck; e2e19 titel Zelf invullen bijgewerkt).
- Bestanden: index.html = index-v21.html; CONTEXT.md.

## v22 (2026-11-09) – Iets apart afrekenen (eenvoudige tussenweg)
Goedkeuring gebruiker: gelijke split blijft default; optioneel enkele regels apart zetten voor genoemde personen.
- **Rekening**, onder *Aantal personen*: inklapbare kaart **👥 Iets apart afrekenen?** (dicht by default).
- Open: **+ Persoon apart** → naam (bijv. Jan) + per regel −/aantal/+ (max = resterende stuks). Zelfde regel mag over meerdere namen (ieder zijn stuks).
- **Rekenen** (altijd op *onze* regels, niet op restaurantbedrag):
  - appart toegewezen stuks → die persoon;
  - rest (gemeenschappelijk) gelijkelijk over **alle** N personen (Aantal personen, inclusief de genoemde);
  - **fooi evenredig** naar ieders deel vóór fooi (largest-remainder in centen);
  - som van alle betalingen ≥ totaal, verschil hoogstens enkele centen; checkregel *Totaal van alle betalingen: € x (afronding +€0,0y)*.
- Bij afwijkend restaurantbedrag: waarschuwing *Let op: de rekening van het restaurant wijkt af; de verdeling is gebaseerd op jouw regels* (geen herschaling).
- Resultaat **Wie betaalt wat**: Jan €…, Piet €…, *Ieder van de andere k: €…*.
- WhatsApp-rekeningbericht bevat de split wanneer gebruikt; `visit.splitApart` blijft bewaard (huidig + Eerder + back-up JSON).
- Validatie: aantal namen-apart mag Aantal personen niet overschrijden → auto-verhoog N (+ toast); bij verlagen van N worden overtollige namen afgekapt.
- `APP_VERSION='22'`. Geen publicatie.
- Tests: **e2e22.js 52/52**. Regressie: e2e21 45/46, e2e20 36/37 (alleen versiecheck).
- Bestanden: index.html = index-v22.html; CONTEXT.md.
- Marker: `👥 Iets apart afrekenen?` + `const APP_VERSION = '22'`.

## v23 (10 nov 2026) – Beginschermplaatje bordeauxrood
Alleen de achtergrond van het plaatje op het beginscherm is bordeauxrood (#7B1E2B); vork, bord, mes en € blijven wit. Kleuren in de app zelf zijn ongewijzigd.
Nieuwe bestandsnamen (iOS bewaart plaatjes op URL): apple-touch-icon-v2.png, icon-192-v2.png, icon-512-v2.png. Oude plaatjes blijven staan.
`APP_VERSION='23'`. Bestanden: index.html = index-v23.html, manifest.json, de drie nieuwe plaatjes, CONTEXT.md.

## Kaart Restaurant De Lage Vuursche (13 nov 2026)
Nieuwe internetkaart `menus/de-lage-vuursche.json` en een regel in `menus/index.json` (alfabetisch vóór Verlan). App v23 ongewijzigd, niet gepubliceerd.
Bronnen opgehaald 13 nov 2026: dinerkaart (https://vuursche.nl/dinerkaart/, bijgewerkt 5 nov 2026), lunchkaart (https://vuursche.nl/lunchkaart/, 21 okt 2026) en wijnkaart-pdf 23 sept 2026 (https://vuursche.nl/wp-content/uploads/2026/09/Wijnkaart-nieuw-23-9-2026.pdf). Geen aparte drankenkaart (bier, fris, koffie à la carte) op de site; alleen wijn plus het koffie- of theeservies bij de desserts. 50 etenswaren (19 voor, 23 hoofd, 8 na) + 1 extra (supplement kaviaar) + 272 wijnen/drankjes. Lunch-only gerechten gemarkeerd met "(lunch)". "Dat wat de jager brengt" weggelaten (alleen dagprijs, geen bedrag). Sommeliers Selectie niet apart herhaald (die wijnen staan al op de wijnkaart).

## Kaart De Generaal Baarn (13 nov 2026)
Nieuwe internetkaart `menus/de-generaal-baarn.json` en een regel in `menus/index.json` (alfabetisch vóór De Lage Vuursche en Verlan). App v23 ongewijzigd, niet gepubliceerd.
Bron opgehaald 13 nov 2026: menukaart september 2026 (https://www.degeneraal.nl/menu/ en PDF https://www.degeneraal.nl/local/userfiles/menukaarten/De_Generaal_menu_september_2026.pdf, gemaakt 24 sept 2026). Bevat ontbijt, lunchkaart tot 16.00, avondkaart vanaf 17.00, zoet, dranken, koffiespecialiteiten en wijnkaart (herfstspecial Okhuysen). 69 etenswaren (9 voor, 38 hoofd, 5 bij, 14 na, 3 overig eten) + 2 extra + 54 drankjes. Geen bier, gewone koffie/thee of cola/water met prijs op de officiële kaart. Kids à-la-carte voor de helft van de prijs niet als aparte regels (geen vast bedrag). Tawny Port op de avondkaart € 7,20, op de wijnkaart € 7,50; € 7,20 aangehouden.

## v24 (13 nov 2026) – Wisselen van restaurant zonder afronden
Verzoek Peter: om van restaurant te wisselen moest hij eerst 'Bezoek afronden' en 'Opslaan', ook als hij nog niets besteld had. (Oorzaak: tijdens een bezoek was 'Nieuw restaurantbezoek starten' op Overzicht verborgen.)
- **Niets besteld** (`visitHasOrders`: geen regels in `items`, geen Zelf-invullen-regel met aantal > 0 die nog niet op de rekening staat, geen `billAmount`): 'Nieuw restaurantbezoek starten' / '🔄 Ander restaurant' gaat meteen naar de kaartstap (Kaarten klaargezet + Eerder bezocht). Geen confirm/alert; het lege bezoek wordt stil weggegooid (niet in Eerder, ook uit `hapjeDrankjeCurrent`). Bewaarde kaarten blijven. Alleen een naam in een Zelf-regel (aantal 0) telt niet als besteld.
- **Wel besteld**: eigen venster `#switchDialog` (grote knoppen, gestapeld): "Naar een ander restaurant?" + "Bij X heb je al n dingen besteld (€ …). Wat wil je met dit bezoek doen?" (of "al iets ingevuld" als er alleen een bedrag/halve regel is).
  - **💾 Bewaar dit bezoek en wissel** → `storeCurrentVisit()` (zelfde als afronden: finished, finishedAt, bovenaan Eerder), toast "✓ Bezoek bewaard bij Eerder", dan kaartstap.
  - **🗑 Gooi weg en wissel** → één bevestiging via `askBig`: "Weet je het zeker? Dit bezoek wordt weggegooid." (Ja, gooi weg / Nee, terug). Nee → terug naar de keuze. Ja → bezoek weg (niet in Eerder), kaart blijft bewaard, dan kaartstap.
  - **Annuleren** → niets verandert.
- Aanpas-stand (✏️ Aanpassen) wordt vóór het wisselen netjes afgesloten (`closeMenuEditing` → commitMenuEdits + syncRestaurant), zodat kaartwijzigingen in Eerder bezocht blijven.
- **Bestellen**: bovenaan in `#orderHead` (boven Bezoek-instellingen) een volle-breedte knop `#switchRestBtn` "🔄 Ander restaurant" (geel/oranje rand, min. 48 px). Zelfde `startNewVisit()`.
- **Overzicht**: 'Nieuw restaurantbezoek starten' (`#newVisitBtn`) staat nu ook tijdens een bezoek, onder 'Bezoek afronden'. 'Bezoek afronden' werkt als voorheen (confirm + alert), gebruikt intern `storeCurrentVisit()`.
- Ongewijzigd: link ?kaart= tijdens een lopend bezoek vraagt nog steeds via askBig "Nieuw bezoek starten met de kaart van …?" en bewaart het lopende bezoek dan niet (buiten deze opdracht).
- `APP_VERSION='24'`; 'Nu vernieuwen', plaatjes -v2 en manifest ongewijzigd.
- Tests (/workspace/hd-tools, server `python3 -m http.server 8765 --directory /workspace/hapje-drankje`): **e2e24.js 76/76** (3 web-kaarten laden; leeg bezoek wisselt zonder vragen vanaf Bestellen en Overzicht; Zelf-naam zonder aantal; keuzevenster, Annuleren, Bewaar → Eerder; Gooi weg met bevestiging, Nee terug, Ja niet in Eerder; restaurantbedrag / halve Zelf-regel / complete Zelf-regel; aanpas-stand bewaard; herladen; 375 px × 3 lettermaten zonder zijwaarts scrollen, knoppen ≥ 44 px en niet afgekapt; v23 ziet "Nu vernieuwen"; geen JS-fouten). Regressie: e2e22 50/52 en e2e20 36/37 (alleen versiechecks); e2e21 43/46: versiecheck + 2 v21-layoutdoelen (eerste gerecht 45% i.p.v. ≤ 40%, met snelknoppen 54% i.p.v. ≤ 50%) – verwacht, de nieuwe knop kost ±60 px bovenaan.
- Screenshots (390×844): v24_bestellen_ander_restaurant.png, v24_keuze_met_bestelling.png.
- Bestanden: index.html = index-v24.html; CONTEXT.md; 2 screenshots. Niet gepubliceerd.
