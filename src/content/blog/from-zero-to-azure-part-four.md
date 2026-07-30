---
title: "From Zero To Azure Part Four"
description: "Del 4 av fem: lägg upp koden på GitHub, skapa en Azure App Service och koppla Azure DevOps till din prenumeration."
pubDate: 2026-06-14
heroImage: "../../assets/images/blog/from-zero-to-azure-part-four/from-zero-to-azure-part-four.jpg"
tags: ["Azure"]
---

# Del 4 — Sätt upp tjänsterna

> **Serien From Zero to Azure**
> [Del 1 — Sätt upp miljön](/blog/from-zero-to-azure-part-one) · [Del 2 — Håll koll på kostnaderna](/blog/from-zero-to-azure-part-two) · [Del 3 — Bygg appen](/blog/from-zero-to-azure-part-three) · **Del 4 — Sätt upp tjänsterna** (du är här) · [Del 5 — Bygg pipelinen](/blog/from-zero-to-azure-part-five)

Det här är fjärde delen av fem. Appen fungerar på din maskin. Nu ska den ut i molnet, och till det behövs tre tjänster som pratar med varandra.

```
Din maskin  →  GitHub  →  Azure DevOps  →  Azure App Service
   push        källkod      bygger,           kör appen
                            testar,
                            deployar
```

I den här delen sätter du upp tjänsterna och kopplingarna mellan dem. I Del 5 bygger du pipelinen som får kedjan att rulla av sig själv.

Det mesta som går fel härifrån handlar om kopplingarna, inte om koden. Namn som ska stämma på två ställen, behörigheter som saknas, brandväggar som blockerar. Därför är felsökningsavsnitten i Del 4 och 5 ovanligt utförliga — läs dem när något stannar, inte i förväg.

**Förutsättning:** Del 1 till 3 är klara. Du har en app som kör lokalt, ett lokalt Git-repo med allt committat, en Azure SQL-databas och en kostnadsbudget.

Har du hoppat över Del 3 och vill börja här: klona referensprojektet, skapa ett eget tomt repo enligt steg 1 och peka om `origin` till det. Du kan inte pusha till någon annans repo, och pipelinen behöver ett repo du själv äger.

## Namnen för den här omgången

| Vad | Exempel |
| --- | --- |
| GitHub-repo | `todo-one` |
| App Service plan | `asp-todo-one` |
| App Service | `app-todo-one-mn` |
| Publik URL | Genereras av Azure, se steg 4 |
| Azure DevOps-organisation | `dev.azure.com/<organisation>` |
| Azure DevOps-projekt | `todo-one` |
| Service connection | `azure-todo-one` |
| Resource group från Del 1 | `rg-dev` |
| Databas från Del 1 | `sqldb-todo-one` |

Suffixet `mn` i exemplen är initialer. Byt det mot vad du vill — namnet behöver bara vara unikt inom den region du väljer. Håll det konsekvent, eftersom det dyker upp igen som `webAppName` i pipelinen i Del 5.

## Fas 1 — GitHub

### Steg 1 — Skapa repot

Du har redan ett lokalt repo från Del 2. GitHub-repot ska därför vara tomt när du skapar det.

Gå till [github.com/new](https://github.com/new) och fyll i:

- **Repository name:** `todo-one`
- **Description:** `Demo app for the blog series From Zero to Azure — ASP.NET Core, Azure SQL and Azure DevOps pipelines`
- **Visibility:** `Public` om repot ska kunna visas upp, annars `Private`
- **Add a README file:** av
- **Add .gitignore:** `None`
- **Choose a license:** `None`

De tre sista måste vara av. Kryssar du i någon av dem skapar GitHub en första commit på sin sida, och när du sedan pushar din lokala historik avvisas den med `refusing to merge unrelated histories`.

Vill du ha en README och en licens i repot lägger du till dem som vanliga filer efteråt och pushar dem. En README är värd besväret om repot är publikt — det är oftast det första någon öppnar.

Klicka sedan **Create repository** och kopiera HTTPS-adressen.

### Steg 2 — Pusha första gången

Stå i repo-roten, till exempel `C:\git\todo-one`:

```powershell
git remote add origin https://github.com/<användarnamn>/todo-one.git
git branch -M main
git push -u origin main
```

Det här gör kommandona:

- `git remote add origin` kopplar det lokala repot till GitHub.
- `git branch -M main` säkerställer att huvudgrenen heter `main`.
- `git push -u origin main` skickar upp allt och sparar kopplingen för framtida pushar.

Gick pushen igenom är du klar med steget. Gå vidare till steg 3.

#### Bara om du fick `error: remote origin already exists`

Då pekar ditt lokala repo redan på ett annat GitHub-repo, till exempel från ett tidigare försök. Kontrollera vart:

```powershell
git remote -v
```

Ta bort den gamla kopplingen och kör om kommandona ovan:

```powershell
git remote remove origin
```

### Steg 3 — Kontrollera att inget läckt

Öppna repot i GitHub, alltså `https://github.com/<användarnamn>/todo-one`, och klicka dig fram till `src` → `TodoOne` → `appsettings.json`.

Det här är vad du ska förvänta dig:

- `DefaultConnection` ska vara tom.
- `bin` och `obj` ska inte ligga i repot.
- Inget lösenord ska finnas i historiken.

Om ett lösenord har råkat hamna i repot ska det bytas direkt i Azure. Att bara ta bort det i ett senare commit räcker inte.

## Fas 2 — Azure App Service

App Service är tjänsten som kör din app i Azure. Vi skapar den innan pipelinen, eftersom pipelinen behöver ett mål att deploya till.

### Steg 4 — Skapa Web App

I Azure-portalen på [portal.azure.com](https://portal.azure.com), sök efter **App Services** och klicka **Create** → **Web App**.

Formuläret består av flera sidor. Längst ner sitter en knapp som visar vilken sida som kommer härnäst, till exempel `Next : Database >`. Det finns också en flikrad med samma namn högre upp, men följ knappen så kommer sidorna i rätt ordning.

#### Sidan Basics

- **Subscription:** har du bara en är den redan vald.
- **Resource Group:** `rg-dev`.
- **Name:** `app-todo-one-mn`. Under fältet visar portalen den färdiga adressen, med hash och region. Anteckna den — du behöver den i steg 18.
- **Publish:** `Code`.
- **Runtime stack:** `.NET 10 (LTS)`.
- **Operating System:** `Windows`.
- **Region:** samma som databasen, alltså den du valde i Del 1. Portalen föreslår ofta en annan, så kontrollera fältet.
- **Windows Plan:** klicka `Create new` och döp den till `asp-todo-one`. Azure föreslår annars ett namn med slumpat suffix.
- **Pricing plan:** `Free F1`. Portalen föreslår en betald nivå, så det här fältet måste du ändra.
- **Zone redundancy:** `Disabled`.
- **Managed Instance (Preview):** avstängd.

Klicka **Next : Database >**.

#### Sidan Database

Ändra ingenting. Portalen erbjuder sig att skapa en ny databas, men du har redan din från Del 1.

Klicka **Next : Deployment >**.

#### Sidan Deployment

Sätt **Continuous deployment** till `Disable`. Det är den enda inställningen på den här sidan som spelar roll, och den är viktig: Azure DevOps sköter deployen, och två flöden skriver över varandra.

Klicka **Next : Networking >**.

#### Sidan Networking

Ändra ingenting.

Klicka **Next : Monitor + secure >**.

#### Sidan Monitor + secure

- **Enable Application Insights:** `Yes`. Det är den som gör fel felsökbara när appen ligger ute i molnet.
- **Application Insights:** ta den föreslagna, `(New) app-todo-one`. Har du befintliga resurser från andra projekt dyker de upp i listan, men återanvänder du en blandas telemetrin från flera appar och felsökningen blir svårare.
- **Enable Defender for App Service:** lämna urkryssad. Det är en betaltjänst för hotdetektering som debiteras per app och månad. Bra i produktion, onödig kostnad för ett demo.

Klicka **Next : Tags >**.

#### Sidan Tags

Ändra ingenting. Taggar är etiketter för att gruppera resurser i stora miljöer, och behövs inte här.

Klicka **Next : Review + create >**.

#### Sidan Review + create

Granska sammanfattningen och klicka **Create**.

Knappen **Review + create** finns tillgänglig på alla sidor, men klickar du den direkt från `Basics` hoppar du över `Deployment` och får continuous deployment påslaget.

Linux går också att välja och är betydligt billigare på betalda nivåer, men kräver ett par justeringar. Det tar jag upp separat i [Azure App Service — vad det kostar, och vad som ändras på Linux](/blog/azure-app-service-costs-and-linux).

Hittar du inte den publika adressen efteråt står den på appens `Overview` under `Default domain`.

### Steg 5 — Lägg in connection stringen

App Service kan inte läsa dina lokala User Secrets. Samma connection string måste därför läggas in i Azure.

Sök efter **App Services** i portalen och klicka på `app-todo-one-mn`. I vänstermenyn: fäll ut **Settings** och välj **Environment variables**.

Sidan har två flikar. `App settings` är förvald — klicka på **Connection strings** i stället. Klicka sedan **+ Add**.

Fyll i:

| Fält | Värde |
| --- | --- |
| Name | `DefaultConnection` |
| Value | Hela connection stringen från Del 1, med riktigt lösenord |
| Type | `SQLAzure` |
| Deployment slot setting | Urkryssad |

`Name` måste vara exakt `DefaultConnection`, med versal `D` på båda orden. Det är nyckeln koden slår upp med `GetConnectionString("DefaultConnection")`, så stavningen måste stämma tecken för tecken.

Klicka **Apply** i panelen, och därefter **Apply** längst ner på sidan. Det andra klicket är lätt att glömma, och utan det sparas ingenting. Bekräfta omstarten av appen.

I App Service lagras Azure SQL-anslutningar som miljövariabler med prefixet `SQLAZURECONNSTR_`, och .NET kan läsa dem via `GetConnectionString("DefaultConnection")` utan att du ändrar koden.

### Steg 6 — Kontrollera brandväggen

Sök efter **Azure SQL** i portalen och öppna **All resources**. Klicka på servern, alltså raden med `Resource type: Microsoft.Sql/servers`. I vänstermenyn: fäll ut **Security** och välj **Networking**.

Scrolla ner till rubriken `Exceptions`, längst ner på sidan under brandväggsreglerna. Där sitter kryssrutan **Allow Azure services and resources to access this server** — kontrollera att den är ikryssad.

Utan den kommer App Service inte åt databasen, och felet dyker upp först när du deployat och undrar varför appen fungerade lokalt.

## Fas 3 — Azure DevOps

### Steg 7 — Skapa organisation och projekt

Gå till [dev.azure.com](https://dev.azure.com) och logga in med samma konto som i Azure.

Har du redan en organisation, använd den. En organisation är den yttersta behållaren, med fakturering, användare och regionval, och den rymmer hur många projekt du vill. Att skapa en ny per projekt ger dig bara mer att hålla ordning på — och eftersom både fakturering och byggagenter sitter på organisationsnivå slipper du göra om det du redan ordnat.

Saknar du en organisation skapar du den så här:

1. Klicka **New organization**.
2. Godkänn villkoren.
3. Namnge organisationen, till exempel efter dig själv eller ditt alias.
4. Välj region, förslagsvis `West Europe` eller `North Europe`.

Skapa sedan projektet. Det är här arbetet ligger, och ett projekt per app är en rimlig indelning.

Klicka **+ New project** och fyll i:

- **Project name:** `todo-one`
- **Description:** valfri, men värd att fylla i. Projektet innehåller bara pipelinen — koden ligger på GitHub — så skriv något som påminner dig om det, till exempel `Build and deployment pipeline for the todo-one demo app. Source code lives on GitHub.`
- **Visibility:** `Private` är enda alternativet. Publika Azure DevOps-projekt är avvecklade och går inte längre att skapa.

Fäll ut **Advanced** för att se de två sista fälten:

- **Version control:** `Git`
- **Work item process:** `Basic`

Klicka **Create**.

### Steg 8 — Se till att du har byggagenter

En byggagent är den maskin som kör din pipeline. Utan tillgång till någon blir din första körning stående i kö med meddelandet `No hosted parallelism has been purchased or granted`.

Har du kört pipelines i den här organisationen tidigare är det redan ordnat. Gå vidare till steg 9.

Nya organisationer får inte automatiskt tillgång. Snabbaste vägen förbi det är att koppla organisationen till din Azure-prenumeration:

1. Klicka på organisationsnamnet uppe till vänster för att lämna projektet. Står du kvar i projektet heter länken nere till vänster `Project settings`, och det är en annan sak — fakturering och byggagenter ligger på organisationsnivå.
2. Klicka **Organization settings** nere till vänster.
3. I vänstermenyn, under rubriken **General**, klicka **Billing**.
4. Klicka **Set up billing** och välj prenumerationen i listan.

Namnet på knappen är olyckligt. Att koppla en prenumeration kostar ingenting — det är bara en referens mellan organisationen och prenumerationen. Kostnader uppstår först om du fyller i ett tal under `Paid parallel jobs`, eller köper användarlicenser utöver de fem som ingår.

Står det **Change billing** i stället för **Set up billing** är kopplingen redan gjord, och du är klar med steget.

På samma sida ser du om gratisnivån är aktiv. Under `Pipelines for private projects` ska det stå `1800 minutes` för `MS Hosted CI/CD`. Det betyder ett jobb åt gången och 1 800 byggminuter i månaden, vilket räcker med god marginal för den här serien.

Fungerar det inte får du ansöka om grantet på [aka.ms/azpipelines-parallelism-request](https://aka.ms/azpipelines-parallelism-request). Räkna med några arbetsdagar, längre under perioder med hög belastning. Skicka in ansökan direkt så löper väntetiden medan du gör resten av guiden.

Vill du inte vänta alls kan du köra bygget på din egen dator med en så kallad self-hosted agent. Den nivån ges automatiskt och har inga tidsgränser, men kräver en del uppsättning och ligger utanför den här guiden.

### Steg 9 — Skapa service connection till Azure

En service connection är identiteten som pipelinen använder när den pratar med Azure.

Den här inställningen ligger i projektet, inte i organisationen. Lämnade du projektet i steg 8 behöver du gå tillbaka in: klicka **Azure DevOps** uppe till vänster och välj projektet `todo-one`.

1. Klicka **Project settings** nere till vänster.
2. I vänstermenyn, under rubriken **Pipelines**, klicka **Service connections**.
3. Klicka **Create service connection**.
4. Markera **Azure Resource Manager** i listan och klicka **Next** nere till höger. Valet markerar bara raden — det är `Next` som tar dig vidare.
5. Ett inloggningsfönster mot Azure kan dyka upp. Logga in, så visas formuläret.

Fyll i formuläret uppifrån och ner:

| Fält | Värde |
| --- | --- |
| Identity type | `App registration (automatic)` |
| Credential | `Workload identity federation` |
| Scope level | `Subscription` |
| Subscription | Din prenumeration, oftast redan vald |
| Resource group | `rg-dev` |
| Service Connection Name | `azure-todo-one` |
| Service Management Reference | Lämna tom |
| Description | Valfri, t.ex. `Deploys the todo-one demo app to App Service` |
| Grant access permission to all pipelines | Ikryssad |

Fyra av raderna tål en förklaring.

`Credential` bör vara `Workload identity federation`, som är märkt `Recommended`. Den använder federerad inloggning i stället för en hemlighet som måste förnyas — alternativet `Secret` fungerar men går ut efter ett par år.

`Resource group` har ofta något annat förvalt, till exempel en `DefaultResourceGroup-` som Azure skapat automatiskt. Byt till `rg-dev`, så begränsar du vad pipelinen får röra.

`Service Connection Name` ska skrivas exakt. Strängen ska in i YAML-filen i Del 5 och matchningen är bokstavlig.

`Grant access permission to all pipelines` sparar dig ett manuellt godkännande vid första körningen.

Klicka **Save**.

Går det inte igenom beror det ofta på att kontot inte får skapa app-registreringar eller federerade kopplingar i tenanten.

## Klar?

- [ ] Koden ligger på GitHub utan hemligheter i historiken.
- [ ] App Service finns och har `DefaultConnection` under **Connection strings**.
- [ ] `Allow Azure services and resources to access this server` är ikryssad på SQL-servern.
- [ ] Azure DevOps-projektet finns.
- [ ] Organisationen har byggagenter, alltså `1800 minutes` under `MS Hosted CI/CD`.
- [ ] En service connection vid namn `azure-todo-one` finns i projektet.

## Vidare till Del 5

Tjänsterna står på plats men de gör ingenting än. Ingen bygger din kod, ingen deployar den, och App Service är tom.

I Del 5 skriver du YAML-filen som binder ihop alltihop, ser första körningen gå grönt, och öppnar din app på en publik adress. Det är där det klickar.

## Felsökning

### `Failed to create an app in Azure Active Directory`

Kontot får inte skapa app-registreringar eller federerade identiteter i tenanten. Be en admin om hjälp, eller skapa kopplingen manuellt.

### .NET 10 saknas i Runtime stack-listan

Nya stackar rullas ut stegvis mellan regioner. Testa en annan region om det behövs. Som reservspår kan appen publiceras self-contained, men det bör ses som en fallback snarare än förstahandsval.