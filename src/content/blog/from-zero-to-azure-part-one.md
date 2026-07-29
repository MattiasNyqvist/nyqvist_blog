---
title: "From Zero To Azure Part One"
description: "Del 1 av tre: installera verktygen, skapa kontona och sätt upp en Azure SQL-databas — grunden för att bygga och driftsätta en ASP.NET Core-app."
pubDate: 2026-05-24
heroImage: "../../assets/images/blog/from-zero-to-azure-part-one/from-zero-to-azure-part-one.jpg"
tags: ["Azure"]
---
# Del 1 — Sätt upp miljön

> **Serien From Zero to Azure**
> **Del 1 — Sätt upp miljön** (du är här) · [Del 2 — Bygg appen](/blog/from-zero-to-azure-part-two) · [Del 3 — Ut i molnet](/blog/from-zero-to-azure-part-three)

Det här är första delen av tre. När du är klar har du verktygen på plats, kontona skapade och en Azure SQL-databas redo att användas.

* Del 1 gör du en gång.
* Del 2 och Del 3 återanvänder du sedan, varje gång du vill bygga något nytt.

## Först: det kommer inte kosta dig något

Rädslan för en oväntad faktura är det vanligaste skälet till att folk aldrig provar Azure. Så låt oss ta det direkt: allt vi bygger i den här serien ryms inom gratisnivåerna. Databasen, webbappen och byggpipelinen kostar noll kronor.

Skräckhistorierna finns, men de handlar om resurser man glömt bort eller om tjänster i en helt annan storleksordning. Botemedlet är att veta vad som kostar och att sätta upp en spärr som säger till.

Sist i den här delen gör vi just det: du får en budget med larm till din e-post, och ett anomalilarm som reagerar om förbrukningen plötsligt ändrar mönster. Tio minuters arbete som gör att du vågar experimentera resten av serien.

## Förbered miljön

Allt i den här serien är skrivet och testat för Windows. Mycket fungerar även på macOS eller Linux med VS Code, men utifrån följande specifikation får du mest ut av guiden.

| Komponent | Version |
| --- | --- |
| Windows | 11 |
| Visual Studio Community | 2026 |
| .NET SDK | 10 (LTS) |
| Git | Senaste stabila |
| Azure CLI | Senaste stabila |

Versionerna var de senaste när det här skrevs.

## Sätt verktygen till engelska

Guiden är på svenska, men kod, kommentarer, commit-meddelanden och resursnamn skrivs på engelska. Verktygen ska också stå på engelska, dels för att klickvägarna i guiden ska stämma, dels för att all dokumentation och alla felmeddelanden du kommer söka på är på engelska.

Har du redan verktygen installerade på svenska byter du så här:

* Azure-portalen: kugghjulet uppe till höger → **Language + region** → **Language: English** → **Apply**.
* Visual Studio: **Tools** → **Options** → **Environment** → **International Settings** → **Language: English**.
* Visual Studio kräver språkpaketet, som installeras via **Visual Studio Installer** → **Language packs**.

## Installera verktygen

Här följer länkar för installation av de verktyg du behöver för den här guideserien. Har du dem redan, hoppa vidare till nästa avsnitt och kontrollera versionerna i stället.

* Visual Studio Community 2026: [visualstudio.microsoft.com/downloads](https://visualstudio.microsoft.com/downloads). Välj workloadsen **ASP.NET and web development**, **Azure and AI development** och **Data storage and processing**. Den sista ger dig databasvyn vi använder i stället för ett separat databasverktyg. Har du redan installerat Visual Studio Community kan du lägga till workloads i efterhand via Visual Studio Installer.

  Visual Studio Code går i princip lika bra att använda men guiden kommer endast utgå ifrån Visual Studio Community 2026.
* .NET 10 SDK: följer normalt med via workloaden ovan, eller via [dotnet.microsoft.com/download/dotnet/10.0](https://dotnet.microsoft.com/download/dotnet/10.0). SDK:en följer inte med Visual Studio Code utan måste laddas ner via den bifogade länken.
* Git: [git-scm.com/downloads](https://git-scm.com/downloads), eller via Visual Studio.
* Azure CLI: installera via Microsofts guide för Windows och 64-bitars versionen, [Install Azure CLI on Windows](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-windows?view=azure-cli-latest). CLI står för command line interface och betyder att du styr tjänsten med kommandon i en terminal i stället för att klicka i portalen.
* EF Core-verktyget:

```bash
dotnet tool install --global dotnet-ef
```

Öppna sedan en ny terminal. Annars kan PATH-uppdateringarna saknas.

## Kontrollera installationen

Kör:

```bash
dotnet --list-sdks
git --version
dotnet ef --version
az version
```

Det här är vad du ska förvänta dig:

* `dotnet --list-sdks` ska visa minst en installerad .NET SDK-version som börjar med `10.`
* Det är normalt om flera SDK-versioner visas samtidigt.
* `git --version` ska visa en giltig Git-version, till exempel `git version 2.x.x`.
* `dotnet ef --version` ska visa en 10.x-version.
* `az version` ska visa att Azure CLI är installerat och ge versionsinformation.

Om något saknas eller visar en gammal version, installera eller uppdatera det innan du går vidare.

Om EF-verktyget redan fanns sedan tidigare och visar fel version, uppdatera det:

```bash
dotnet tool update --global dotnet-ef
```

## Konfigurera Git

Använd samma e-postadress som på ditt GitHub-konto, annars kopplas inte commits till din profil.

```bash
git config --global user.name "Ditt Namn"
git config --global user.email "din.epost@example.com"
git config --global init.defaultBranch main
```

## Skapa konton

Skapa tre konton, alla med gratisnivåer som räcker för den här serien:

* GitHub: [github.com/signup](https://github.com/signup)
* Azure: [azure.microsoft.com/free](https://azure.microsoft.com/free)
* Azure DevOps: [dev.azure.com](https://dev.azure.com)

Använd samma inloggning till Azure och Azure DevOps. Det gör kopplingen mellan dem enklare i Del 3.

## Logga in på Azure

Logga in med Azure CLI:

```bash
az login
```

Ett webbläsarfönster öppnas. Efter inloggning visas normalt dina prenumerationer, och om du bara har en kan du ofta bara trycka Enter för att välja den.

Verifiera att du verkligen är inloggad:

```bash
az group list --output table
```

En tom lista är okej om du inte har skapat några resursgrupper än. Ett autentiseringsfel är det inte.

## Hitta subscription och tenant

Två värden behöver du leta upp, resten skapar du själv längre fram. Öppna ett anteckningsdokument och håll det framme — sist i den här delen finns en tabell med allt du ska ha samlat.

```bash
az account show --query "{Name:name, SubscriptionId:id, TenantId:tenantId}" --output table
```

I Azure-portalen på [portal.azure.com](https://portal.azure.com) hittar du subscription-namn och ID under **Subscriptions**, och tenant-ID under **Microsoft Entra ID** → **Overview**.

## Välj namnstandard

Bestäm en enkel standard och håll dig till den.

| Resurstyp          | Exempel           |
| ------------------ | ----------------- |
| Resource group     | `rg-dev`          |
| SQL logical server | `sql-dev-mn`      |
| SQL database       | `sqldb-todo-one`  |
| App Service plan   | `asp-todo-one`    |
| App Service        | `app-todo-one-mn` |

SQL-serverns namn måste vara globalt unikt i hela Azure, eftersom det blir DNS-namnet `<namn>.database.windows.net`. Ett generiskt namn som `sql-dev` är med största sannolikhet redan taget av någon annan.

Suffixet `mn` i exemplen är initialer. Byt det mot dina egna, mot några siffror, eller mot vad du vill — det enda som spelar roll är att du håller det konsekvent. Portalens `Name`-fält visar en grön bock när namnet är ledigt.

Välj också en region och använd samma för allt. Det minskar onödig latens mellan app och databas.

## Skapa resursgruppen

En resursgrupp är en behållare för relaterade resurser. Vi lägger SQL-servern i en grupp som kan leva vidare mellan projekt.

I portalen:

1. Sök efter **Resource groups**.
2. Klicka **Create**.
3. Välj din subscription.
4. Sätt **Resource group** till `rg-dev`.
5. Välj din region.
6. Klicka **Review + create** och sedan **Create**.

## Skapa SQL-servern och databasen

En logical server är inte en egen maskin. Den är den logiska behållaren för adress, inloggning och brandväggsregler. Själva databaserna ligger under servern.

I portalen:

1. Sök efter **SQL database**, i singular. Söker du på "SQL databases" dyker inte tjänsten upp bland förslagen.
2. Klicka på pilen bredvid **Create** och välj **SQL database (Free offer)**.

Free offer ger 100 000 vCore-sekunder, 32 GB data och 32 GB backup gratis per månad så länge prenumerationen finns. Når du gränsen pausas databasen i stället för att börja kosta pengar. Det räcker med god marginal för den här serien.

Formuläret har bara två flikar, **Basics** och **Review + create**. Det vanliga alternativet **SQL database** har fem flikar och fler val, men behövs inte här.

3. På fliken **Basics**:
   * Subscription: har du bara en är den redan vald. Namnet noterade du i avsnittet om subscription och tenant.
   * Resource Group: `rg-dev`.
   * Database name: `sqldb-todo-one`.
   * Server: klicka **Create new**.

4. I panelen **Create SQL Database Server**:
   * Server name: `sql-dev-mn`. Adressen `.database.windows.net` visas under fältet.
   * Location: din region.
   * Authentication method: **Use both SQL and Microsoft Entra authentication**.
   * Set Microsoft Entra admin: ditt eget konto fylls normalt i automatiskt. Är fältet tomt, klicka **Set admin** och välj dig själv.
   * Server admin login: välj ett eget användarnamn. Inte `admin`, `sa` eller `root`.
   * Password och Confirm password: välj ett starkt lösenord och spara det direkt. Det går inte att läsa ut senare, bara återställa.
   * Klicka **OK**.

   SQL-inloggningen är den vi använder genom hela serien, och den fungerar likadant oavsett vad du väljer under Authentication method. Entra läggs bredvid och kostar ingenting nu, men gör att du senare kan byta till Managed Identity utan att bygga om servern.

5. Tillbaka på **Basics** ska Server nu visa `(new) sql-dev-mn`. Klicka **Review + create**.

6. Granska sammanfattningen. Compute + storage och Backup storage redundancy är redan ifyllda — Free offer sätter dem åt dig, och formuläret har inga fält för elastic pool, workload environment eller backup redundancy. Klicka **Create**.

Azure visar en deployment-sida i stället för resursen. Först står det **Deployment is in progress** med bara servern listad, sedan **Your deployment is complete**. Det tar några minuter.

Servern och databasen är nu skapade. Sidan är en kvittens och behöver inget mer — gå vidare till nästa avsnitt.

## Öppna brandväggen

Free offer-formuläret har ingen **Networking**-flik, så servern skapas utan brandväggsregler. Det betyder att ingen kommer åt databasen än — varken du från din egen maskin eller App Service i Del 3. Det fixar vi nu.

I portalen:

1. Sök efter **Azure SQL**, klicka på tjänsten och välj `All resources` i vänstermenyn. Klicka på `sql-dev-mn`, raden med `Resource type: Microsoft.Sql/servers`. Observera att det är servern du ska in på, inte databasen.
2. I vänstermenyn: **Security** → **Networking**.
3. På fliken **Public access**, välj **Selected networks**.
4. Under **Firewall rules**, klicka **Add your client IPv4 address**. Din publika IP står i knapptexten.
5. Scrolla ner till rubriken **Exceptions**, längst ner på sidan under brandväggsreglerna. Kryssa i **Allow Azure services and resources to access this server**. Utan den kommer App Service inte åt databasen i Del 3.
6. Klicka **Save**.

De två reglerna beter sig olika över tid. **Allow Azure services** sätter du en gång och sedan gäller den. IP-regeln gäller bara så länge du har kvar samma publika adress, och den kan ändras när som helst — vid en routeromstart, när operatören delar ut en ny adress, när du slår på VPN eller när du sitter på ett annat nätverk. Slutar du plötsligt komma åt databasen är det här första stället att titta. Se avsnittet nedan.

## Spara anslutningssträngen

Anslutningssträngen är den rad som talar om för din app var databasen finns och hur den ska logga in. Du behöver den i Del 2.

Sök **Azure SQL** → **All resources** → klicka på databasen, alltså raden med `Resource type: Microsoft.Sql/servers/databases`. På `Overview` finns länken **Show database connection strings** under rubriken `Connection strings`. Samma sida nås via **Settings** → **Connection strings** i vänstermenyn.

Sidan listar flera varianter. Ta den som heter **ADO.NET (SQL authentication)** — det är den inloggning vi använder i serien. Varianterna för Microsoft Entra är till för lösenordsfri inloggning, som vi återkommer till i Del 3.

Strängen ser ut ungefär så här:

```text
Server=tcp:sql-dev-mn.database.windows.net,1433;Initial Catalog=sqldb-todo-one;Persist Security Info=False;User ID=<ditt-användarnamn>;Password={your_password};MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;
```

Ditt användarnamn står redan ifyllt. Lösenordet gör det inte — där står `{your_password}`, klammerparenteser och allt, eftersom lösenordet inte lagras i klartext någonstans. Ersätt hela `{your_password}` med lösenordet du valde när servern skapades, och spara strängen lokalt.

Lägg den aldrig i repot. En anslutningssträng med lösenord i klartext hör inte hemma i versionshanterad kod. I Del 2 lägger vi den i User Secrets, som ligger utanför projektmappen.

## Testa anslutningen

Det här steget är värt att göra. Fungerar anslutningen härifrån fungerar normalt också connection stringen i Del 2.

Du behöver inget extra program — Visual Studio har en inbyggd databasvy.

1. Öppna Visual Studio och välj **View** → **SQL Server Object Explorer**.
2. Högerklicka på **SQL Server** i trädet och välj **Add SQL Server**.
3. Server name: `sql-dev-mn.database.windows.net`.
4. Authentication: **SQL Server Authentication**.
5. Login och Password: uppgifterna du valde när servern skapades.
6. Under **Database Name**, skriv `sqldb-todo-one`. Lämnar du fältet tomt ansluter du mot `master` och ser inte dina tabeller.
7. Klicka **Connect**.

Bläddra sedan ner till `Databases` → `sqldb-todo-one` → `Tables`. Den är tom än så länge, och det är väntat — vi skapar tabellen i Del 2.

Saknas menyvalet **SQL Server Object Explorer** behöver du lägga till workloaden `Data storage and processing` i Visual Studio Installer.

Det finns också ett dedikerat verktyg för databasarbete, SQL Server Management Studio. Det är utmärkt, men vi håller oss till Visual Studio i den här serien — färre program att installera och hålla uppdaterade betyder färre saker som kan gå sönder på vägen.

## När anslutningen slutar fungera

Det här är det vanligaste felet i hela serien, och det dyker upp långt efter att du satt upp allt.

Azure SQL släpper bara in de IP-adresser som står i brandväggen. Din publika adress är i regel inte fast, utan kan bytas vid en routeromstart, av operatören, eller när du byter nätverk. VPN ger också en annan adress.

Symptomen är att databasvyn inte längre ansluter, eller att appen kastar `Cannot open server ... Client with IP address ... is not allowed`. Felmeddelandet innehåller din nya adress ordagrant.

Åtgärden tar tjugo sekunder:

1. Portalen → sök **Azure SQL** → `All resources` → klicka `sql-dev-mn`.
2. **Security** → **Networking**.
3. Klicka **Add your client IPv4 address**.
4. Klicka **Save**.

Gamla regler kan ligga kvar utan att göra skada, men det är prydligt att rensa bort dem med jämna mellanrum.

## Kolla kostnaden

Databasen är på plats och du har inte betalat något. Nu sätter vi upp spärren som gör att det förblir så.

Så här ser kostnadsbilden ut konkret:

| Resurs | Kostnad i den här serien |
| --- | --- |
| Resursgrupp | Gratis, alltid. Det är bara en behållare. |
| SQL-databas via Free offer | Gratis. Pausas automatiskt vid gränsen. |
| App Service `Free F1` (Del 3) | Gratis. |
| Azure DevOps | Gratis upp till fem användare. |

Det syns i sammanfattningen du klickade förbi när databasen skapades: `Estimated total: Free` och `Overage billing: Disabled`. Det sista betyder att databasen pausas när gratisgränsen nås, i stället för att börja debitera.

Ändå ska du sätta upp bevakning. Inte för att den här serien blir dyr, utan för att vanan är värd att ha när du väl provar något som kostar — och för att det är just den vissheten som gör att man vågar experimentera.

### Tre verktyg under Monitoring

I portalen, sök efter **Cost Management** och titta i vänstermenyn under rubriken `Monitoring`. Där finns tre saker som låter lika men gör olika:

| Verktyg | Vad det gör | Värt att sätta upp nu? |
| --- | --- | --- |
| **Budgets** | Du sätter ett belopp och får mejl vid valda procentgränser. | Ja. |
| **Cost alerts** | Inkorgen för larm som redan utlösts. Här skapar du också anomalilarm. | Ja, se nedan. |
| **Alerts** | Larm på faktureringsnivå: krediter, fakturor, reservationsutnyttjande. | Nej. |

Den sista är lätt att fastna i. Öppnar du `Alerts` från en **Billing profile** får du en dialog där `Alert type` erbjuder saker som `Reservation utilization` — larm om att reservationer du köpt inte används. Har du inga reservationer, vilket du inte har, finns inget att bevaka. Stäng och gå vidare.

### Skapa en budget

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

### Sätt larmen

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

### Anomalilarm under `Cost alerts`

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

### Om något ändå kostar

Gå till **Cost Management** → `Cost analysis` och gruppera på `Service name`. Då ser du exakt vilken tjänst det är.

Den kraftigaste åtgärden är att ta bort hela resursgruppen: **Resource groups** → `rg-dev` → **Delete resource group** → skriv namnet för att bekräfta. Allt inuti försvinner, och kostnaden upphör. Du kan alltid köra Del 1 igen — det tar en kvart.

## Anteckna innan du går vidare

Följande uppgifter behöver du i Del 2 och 3. Skriv ner dem i ett eget dokument nu, så slipper du leta senare.

Skulle du ändå tappa bort något står det i tabellen var du hittar det igen. Sökrutan högst upp i portalen är att föredra framför menyvägar, eftersom menyerna flyttar på sig mellan portalversioner medan tjänstnamnen ligger still. Sök på `Azure SQL` och inte på `SQL server` — det senare ger mest träffar på Marketplace-produkter och SQL Server på virtuella maskiner.

| Uppgift | Var du hittar den |
| --- | --- |
| Subscription name | Sök `Subscriptions` |
| Subscription ID | Sök `Subscriptions` |
| Tenant ID | Sök `Microsoft Entra ID` → `Overview` |
| Azure-region, t.ex. `Sweden Central` | Sök `Resource groups`, kolumnen `Location` |
| Resource group | Sök `Resource groups` |
| SQL-serverns adress | Sök `Azure SQL` → `All resources` → servern → `Overview` |
| Databasnamn | Sök `Azure SQL` → `All resources`, raden `Microsoft.Sql/servers/databases`. Namnet är delen före parentesen |
| SQL admin-användare | Sök `Azure SQL` → `All resources` → servern → `Overview` |
| SQL admin-lösenord | Går inte att hämta. Sätt ett nytt via servern → `Overview` → `Reset password` |
| Anslutningssträng | Sök `Azure SQL` → `All resources` → databasen → `Overview` → `Show database connection strings` → `ADO.NET (SQL authentication)` |
| Entra-admin för SQL-servern | Sök `Azure SQL` → `All resources` → servern → `Settings` → `Microsoft Entra ID` |
| GitHub-användarnamn | Under din profilbild uppe till höger på `github.com` |
| Azure DevOps-organisation | Skapas i Del 3. Namnet står sedan i adressfältet: `dev.azure.com/<organisation>` |

## Felsökning

### `az login` slutar med `No subscriptions found`

Inloggningen har två delar: autentisering och uppräkning av tenants och prenumerationer. Autentiseringen kan lyckas även om uppräkningen misslyckas, så läs felraderna ovanför slutmeddelandet noga.

### `AADSTS700082`

Tokenen har gått ut efter inaktivitet. Logga in igen mot rätt tenant:

```bash
az login --tenant tenant-id>
```

Om det inte räcker:

```bash
az logout
az account clear
az login --tenant tenant-id>
```

### `AADSTS50076`

Testa att begära ARM-scopet direkt:

```bash
az login --scope https://management.core.windows.net//.default
```

Det dubbla snedstrecket före `.default` är korrekt.

### `Cannot open server ... Client with IP address ... is not allowed`

Det är brandväggen. Lägg till din klient-IP i Azure SQL och prova igen.

### `Login failed for user`

Det är oftast fel användarnamn eller lösenord. Om du fastnar kan du återställa lösenordet i portalen.

### Anslutningen fungerar men inga tabeller syns

Då är du sannolikt ansluten mot `master`. Ange `sqldb-todo-one` under **Database Name** när du lägger till servern.

## Klar?

* Verktygen står på engelska.
* `dotnet --list-sdks` visar en 10.x-rad.
* `dotnet ef --version` visar 10.x.
* `git --version` fungerar och Git är konfigurerat med namn och e-post.
* `az group list --output table` ger svar.
* Resursgrupp, SQL-server och databas finns i portalen.
* Brandväggsreglerna är sparade: din IP och Allow Azure services.
* En budget är satt med minst ett larm kopplat till din e-post.
* Du når databasen från **SQL Server Object Explorer** i Visual Studio.
* Uppgifterna under *Anteckna innan du går vidare* är nedskrivna.

## Vidare till Del 2

Bockade du av allt? Då är den tråkiga delen avklarad, och du behöver inte göra om den. Det du har byggt är en grund: verktygen på plats, en databas i molnet, en spärr som säger till om något börjar kosta. Nästa gång du får en idé kan du hoppa direkt till Del 2 och vara igång på en kvart.

Just nu står databasen tom. Ingen tabell, ingen rad, ingenting. Det är i Del 2 vi ändrar på det.

Där bygger vi appen från en tom mapp: skelettet från terminalen, koden i Visual Studio, och en enkel todo-lista med allt du behöver kunna göra — lägga till, markera som klar, ta bort. Både som webbsida och som API.

Och när du kör `SELECT * FROM Todos` på slutet ser du dina egna rader ligga i molnet. Det är då det brukar klicka.

Vi ses i Del 2.