# Best practices en valkuilen

Aanbevelingen op basis van de analyse van eerder gemigreerde modules binnen ArcelorMittal en van de algemene literatuur over legacy-modernisering (Orosz 2022, Patel 2024).

## Paradigmaverschuiving aanvaarden

CA Gen is modelgedreven; PL/I is procedureel. Wat in CA Gen impliciet door de generator wordt afgehandeld (declaraties, view-koppelingen, SQL-generatie), moet in PL/I expliciet geschreven worden. Pogingen om één-op-één te vertalen zonder rekening te houden met dit paradigmaverschil leiden tot onnodig complexe of inefficiënte code.

## Conventies van de bestaande codebase volgen

De PLAPO-codebase bevat reeds tientallen gemigreerde modules. Nieuwe modules moeten dezelfde conventies volgen op vlak van:

- Naming van procedures, parameters en variabelen.
- Structuur van DCL-blokken en commentaarheaders.
- Foutafhandeling en exitstate-codes.
- Aanroep-volgorde tussen generieke en specifieke modules.

Afwijkingen leiden tot onleesbare code en bemoeilijken het onderhoud.

## View Matching expliciet maken

Een veelvoorkomende fout is het impliciet meenemen van velden die in CA Gen wel maar in PL/I niet automatisch gekoppeld worden. Best practice: bij elke `CALL` alle parameters expliciet meegeven en elke koppeling tussen aanroepende en aangeroepen view formeel documenteren in het analysedossier.

## Embedded SQL i.p.v. verborgen entity actions

In CA Gen zijn `READ`-statements compact omdat de generator de SQL afleidt uit het model. In PL/I moet de SQL volledig uitgeschreven worden. Aanbevolen: de SQL formuleren op basis van de datamatrix uit Stap 1.5 en niet op basis van een visuele interpretatie van de CA Gen-code, om subtiele JOIN- of filtervoorwaarden niet te missen.

## Group View-grenzen bewaken

Group Views hebben een vaste maximumkardinaliteit. Bij vertaling naar PL/I-arrays moet deze grens zowel aan de declaratiekant (`DCL ... (n)`) als aan de luskant gerespecteerd worden. Een fetch-lus die verder leest dan de array-grens veroorzaakt geheugencorruptie zonder duidelijke foutmelding.

## Exitstate-semantiek behouden

De aanroepende module verwacht specifieke exitstate-waarden om beslissingen te nemen. Een onvolledige mapping naar return-codes leidt tot stille fouten in upstream-modules. Aanbeveling: maak een mapping-tabel exitstate → return-code en review deze samen met de eigenaar van de aanroepende module.