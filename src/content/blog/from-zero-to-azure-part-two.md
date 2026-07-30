---
title: "From Zero To Azure Part Two"
description: "Del 2 av fem: sätt ett kostnadstak i Azure, få larm i tid och slipp oroa dig för oväntade fakturor."
pubDate: 2026-05-31
heroImage: "../../assets/images/blog/from-zero-to-azure-part-two/from-zero-to-azure-part-two.jpg"
tags: ["Azure"]
---

# Del 2 — Håll koll på kostnaderna

> **Serien From Zero to Azure**
> [Del 1 — Sätt upp miljön](/blog/from-zero-to-azure-part-one) · **Del 2 — Håll koll på kostnaderna** (du är här) · [Del 3 — Bygg appen](/blog/from-zero-to-azure-part-three) · [Del 4 — Sätt upp tjänsterna](/blog/from-zero-to-azure-part-four) · [Del 5 — Bygg pipelinen](/blog/from-zero-to-azure-part-five)

Det här är andra delen av fem, och den kortaste. Vi skriver ingen kod. Vi sätter upp spärren som gör att du vågar experimentera i resten av serien.

Rädslan för en oväntad faktura är det vanligaste skälet till att folk aldrig provar Azure. Skräckhistorierna finns, men de handlar nästan alltid om resurser man glömt bort, eller om tjänster i en helt annan storleksordning än den här serien.

**Förutsättning:** Del 1 är klar. Du har en resursgrupp och en databas i Azure.

## Om siffrorna i det här inlägget

Priser, gratisnivåer och villkor i Azure ändras löpande, och de skiljer sig mellan regioner, prenumerationstyper och valutor. Det du läser här är hur jag har förstått att det fungerar när inlägget skrevs, och det är avsiktligt hållet på en nivå där det ska stämma en tid.

Men lita inte på min uppgift när det gäller pengar. Innan du skapar något som kan kosta, kontrollera aktuellt pris i [Azure Pricing Calculator](https://azure.microsoft.com/pricing/calculator/) och läs villkoren för din egen prenumeration under **Cost Management**. Det är din faktura som gäller, inte ett blogginlägg.

Är du osäker på om något debiteras, fråga Microsofts support via portalen. Det kostar ingenting och tar mindre tid än en obehaglig överraskning.

Just därför är resten av det här inlägget inriktat på bevakning snarare än på priser. En budget med larm skyddar dig oavsett vad tjänsterna kostar just nu.

Databasen är på plats och du har inte betalat något. Nu sätter vi upp spärren som gör att det förblir så.

Så här ser kostnadsbilden ut konkret:

| Resurs | Kostnad i den här serien |
| --- | --- |
| Resursgrupp | Gratis, alltid. Det är bara en behållare. |
| SQL-databas via Free offer | Gratis. Pausas automatiskt vid gränsen. |
| App Service `Free F1` (Del 4) | Gratis. |
| Azure DevOps | Gratis upp till fem användare. |

Det syns i sammanfattningen du klickade förbi när databasen skapades: `Estimated total: Free` och `Overage billing: Disabled`. Det sista betyder att databasen pausas när gratisgränsen nås, i stället för att börja debitera.

Ändå ska du sätta upp bevakning. Inte för att den här serien blir dyr, utan för att vanan är värd att ha när du väl provar något som kostar — och för att det är just den vissheten som gör att man vågar experimentera.

## Tre verktyg under Monitoring

I portalen, sök efter **Cost Management** och titta i vänstermenyn under rubriken `Monitoring`. Där finns tre saker som låter lika men gör olika:

| Verktyg | Vad det gör | Värt att sätta upp nu? |
| --- | --- | --- |
| **Budgets** | Du sätter ett belopp och får mejl vid valda procentgränser. | Ja. |
| **Cost alerts** | Inkorgen för larm som redan utlösts. Här skapar du också anomalilarm. | Ja, se nedan. |
| **Alerts** | Larm på faktureringsnivå: krediter, fakturor, reservationsutnyttjande. | Nej. |

Den sista är lätt att fastna i. Öppnar du `Alerts` från en **Billing profile** får du en dialog där `Alert type` erbjuder saker som `Reservation utilization` — larm om att reservationer du köpt inte används. Har du inga reservationer, vilket du inte har, finns inget att bevaka. Stäng och gå vidare.

## Skapa en budget

Gå in via prenumerationen, så hamnar budgeten på rätt nivå direkt. Öppnar du `Cost Management` från portalens startsida landar du ofta på faktureringskontot i stället, och då bevakar budgeten hela kontot. Det fungerar, men är trubbigare — du ser inte vad just den här prenumerationen kostar.

1. Sök **Subscriptions** i portalen och klicka på prenumerationen i listan.
2. I vänstermenyn: **Cost Management** → `Budgets` → klicka **+ Add**.
3. `Budget scoping`:
   * Scope: ska redan visa prenumerationens namn.
   * Filters: hoppa över. Vill du senare bevaka bara en resursgrupp lägger du till ett filter här.
4. `Budget Details`:
   * Name: `budget-monthly`. Namnet måste vara unikt inom scopet.
   * Reset period: **Monthly**. Beloppet nollställs den första varje månad.
   * Creation date: innevarande månad, dagen fylls i automatiskt.
   * Expiration date: låt förslaget stå. Budgeten upphör då att gälla, så sätt en påminnelse om du vill förlänga.
5. `Budget Amount`:
   * Amount: **150**. Azure föreslår en siffra baserad på din prognos, ofta noll eller några kronor på ett nytt konto. Sätt ändå ett rimligt tak — 150 kr i månaden är lågt nog att larma tidigt utan att pipa för varje krona.

   Har du redan andra resurser i prenumerationen behöver du välja belopp med omsorg. Ligger din nuvarande förbrukning på 130 kr i månaden kommer ett tak på 150 larma varje månad utan att något faktiskt hänt, och larm som alltid tjuter slutar man läsa. Antingen höjer du taket ordentligt över baslinjen, eller så filtrerar du budgeten till bara `rg-dev` under `Budget scoping` → `Add filter` → `Resource group`. Det andra alternativet är oftast bättre — då mäter du den här serien för sig, och dina befintliga resurser stör inte.
6. Klicka **Next** för att komma till `Set alerts`.

Har du redan skapat en budget och är osäker på var den hamnade: öppna `Budgets` via menyn ovan. Syns den i listan är den kopplad till prenumerationen. Är listan tom ligger den på faktureringskontot, och du kan skapa om den härifrån.

Står det i stället `Failed to retrieve billing information` har sidan inte lyckats hämta data — det säger ingenting om huruvida budgeten finns. Felet är övergående. Klicka `Refresh`, eller kom tillbaka senare. Inget i den här serien är beroende av budgeten, så du kan gå vidare under tiden.

## Sätt larmen

Här väljer du vid vilka nivåer du vill bli störd. Lägg till tre rader:

| Type | % of budget | Vad det betyder med 150 kr |
| --- | --- | --- |
| Actual | 50 | Mejl vid 75 kr förbrukat |
| Actual | 80 | Mejl vid 120 kr |
| Forecasted | 100 | Mejl när Azure *förutspår* att du landar över 150 kr |

Den sista är den mest användbara. `Forecasted` larmar innan pengarna är spenderade, baserat på hur förbrukningen ser ut hittills. `Actual` larmar när det redan hänt.

Fyll i din e-postadress under `Alert recipients`, och klicka **Create**.

Vill du larma flera personer skriver du en adress per rad — tryck Enter emellan. Semikolon eller komma på samma rad underkänns, eftersom hela strängen då tolkas som en enda adress. Alla mottagare får samma mejl vid samma tröskel.

**En viktig sak att veta:** en budget stoppar ingenting. Den skickar mejl. Azure fortsätter köra dina resurser även när taket passeras. Vill du att något ska stängas av krävs en automatisering, vilket ligger utanför den här serien. Budgeten är en rökdetektor, inte ett sprinklersystem.

## Anomalilarm under `Cost alerts`

En budget larmar när summan blir för hög. Ett anomalilarm larmar när mönstret ändras — om din dagliga förbrukning plötsligt dubblas, även om totalen fortfarande är låg. Det fångar den klassiska tabben: en resurs som råkat skapas i fel prisklass, eller en tjänst som inte stängdes av.

Anomalilarm går bara att skapa på prenumerationsnivå, och det gäller alla konton. Öppnar du `Cost Management` direkt från portalens startsida landar du ofta på faktureringskontot i stället, och då är valet gråmarkerat med texten *Not available for billing profiles. Please change scope to a subscription.* Gå därför in via prenumerationen från början:

1. Sök **Subscriptions** i portalen och klicka på prenumerationen i listan.
2. I vänstermenyn: **Cost Management** → `Cost alerts`.
3. Klicka **+ Add**.
4. `Alert type`: **Anomaly**. Övriga fält i `Condition` fylls i automatiskt och kan stå som de är.
5. `Subject`, `Message` och `Alert name` är förifyllda med rimliga värden. Ändra bara om du vill.
6. `Recipients`: din e-postadress. Flera går bra, en per rad.
7. `Language`: styr språket i larmmejlet. Sätt engelska om du vill hålla samma linje som resten av serien.
8. Klicka **Create**. Knappen tänds först när fältet du senast skrev i tappar fokus — tryck Tab om den ser gråmarkerad ut trots att allt är ifyllt.

Notera datumet efter `until` i `Condition`. Regeln slutar gälla då, normalt ett år fram. Vill du ha bevakning längre än så får du förlänga eller skapa om den.

Är du redan inne i `Cost Management` och ser `Billing profile` i rubriken kan du i stället klicka på scope-namnet högst upp på sidan och byta till prenumerationen.

Två begränsningar värda att känna till: Azure lär sig ditt mönster från de senaste 60 dagarna, så på ett nytt konto tar det ett tag innan larmet är meningsfullt, och detektionen har ett par dygns fördröjning. Det ersätter alltså inte budgeten — det kompletterar den.

## Om något ändå kostar

Gå till **Cost Management** → `Cost analysis` och gruppera på `Service name`. Då ser du exakt vilken tjänst det är.

Den kraftigaste åtgärden är att ta bort hela resursgruppen: **Resource groups** → `rg-dev` → **Delete resource group** → skriv namnet för att bekräfta. Allt inuti försvinner, och kostnaden upphör. Du kan alltid köra Del 1 igen — det tar en kvart.

## Vidare till Del 3

Nu har du ett tak, larm som når dig i tid, och ett sätt att hitta boven om något börjar kosta. Det är tio minuters arbete som gör resten av serien betydligt mer avspänd.

I Del 3 börjar vi koda. Databasen du skapade i Del 1 står tom — ingen tabell, ingen rad, ingenting. Det ändrar vi på nu.

Vill du gräva djupare i vad Azure-tjänster faktiskt kostar finns ett separat inlägg om App Service: [Azure App Service — vad det kostar, och vad som ändras på Linux](/blog/azure-app-service-costs-and-linux). Det förklarar bland annat varför du betalar för planen och inte för apparna.
