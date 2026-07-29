---
title: "From Zero To Azure Part Three"
description: "Del 3 av tre: GitHub, Azure DevOps och en pipeline som bygger, testar och driftsätter appen till Azure App Service."
pubDate: 2026-07-07
heroImage: "../../assets/images/blog/from-zero-to-azure-part-three/from-zero-to-azure-part-three.jpg"
tags: ["Azure"]
---

# Del 3 — Ut i molnet

> **Serien From Zero to Azure**
> [Del 1 — Sätt upp miljön](/blog/from-zero-to-azure-part-one) · [Del 2 — Bygg appen](/blog/from-zero-to-azure-part-two) · **Del 3 — Ut i molnet** (du är här)

Det här är tredje och sista delen. Här tar du appen från din egen maskin till en publik adress, med ett bygge som körs automatiskt varje gång du pushar.

Del 1 sätter upp miljön. Del 2 bygger appen lokalt. Del 3 kopplar ihop GitHub, Azure DevOps och Azure App Service.

Förutsättning: Del 1 och 2 är klara. Du har en app som kör lokalt, ett lokalt Git-repo med allt committat och en Azure SQL-databas redo att användas.

Om du hoppar över Del 2 kan du utgå från ett färdigt repo när du har ett sådant. Poängen är att du måste äga ditt eget GitHub-repo innan du kan pusha och bygga vidare.

Har du hoppat över Del 2 och vill börja här: klona referensprojektet på [github.com/<användarnamn>/todo-one](https://github.com/), skapa ett eget tomt repo enligt steg 1 och peka om `origin` till det. Du kan inte pusha till någon annans repo, och pipelinen behöver ett repo du själv äger.

## Vad vi bygger

```text
Din maskin  →  GitHub  →  Azure DevOps  →  Azure App Service
   push        källkod      bygger,           kör appen
                            testar,
                            deployar
```

Det mesta som går fel i den här delen handlar inte om koden, utan om kopplingarna mellan tjänsterna. Därför är felsökningen längst ner viktig att spara.

## Namnen för den här omgången

| Vad | Exempel |
|---|---|
| GitHub-repo | `todo-one` |
| App Service plan | `asp-todo-one` |
| App Service | `app-todo-one-mn` |
| Publik URL | Genereras av Azure, se steg 4 |
| Azure DevOps-organisation | `dev.azure.com/<organisation>` |
| Azure DevOps-projekt | `todo-one` |
| Service connection | `azure-todo-one` |
| Resource group från Del 1 | `rg-dev` |
| Databas från Del 1 | `sqldb-todo-one` |

Suffixet `mn` i exemplen är initialer. Byt det mot vad du vill — namnet behöver bara vara unikt inom den region du väljer. Håll det konsekvent, eftersom det dyker upp igen som `webAppName` i pipelinen längre fram.

Den publika adressen kan du inte gissa dig till. Azure lägger till en slumpad hash och en regionsidentifierare, så att adressen blir `<namn>-<hash>.<region>-01.azurewebsites.net`. Det görs för att ingen annan ska kunna ta över adressen om du raderar appen. Du hittar den färdiga adressen på appens `Overview` när den är skapad.

## Fas A — GitHub

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

## Fas B — Azure App Service

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

Linux går också att välja, men kräver justeringar i pipelinen och ligger utanför den här guiden.

Hittar du inte den publika adressen efteråt står den på appens `Overview` under `Default domain`.

### Steg 5 — Lägg in connection stringen

App Service kan inte läsa dina lokala User Secrets. Samma connection string måste därför läggas in i Azure.

Sök efter **App Services** i portalen och klicka på `app-todo-one-mn`. I vänstermenyn: fäll ut **Settings** och välj **Environment variables**.

Sidan har två flikar. `App settings` är förvald — klicka på **Connection strings** i stället. Klicka sedan **+ Add**.

Fyll i:

- **Name:** `DefaultConnection`. Exakt så, med versal `D` på båda orden. Det är nyckeln koden slår upp med `GetConnectionString("DefaultConnection")`, så stavningen måste stämma tecken för tecken.
- **Value:** hela connection stringen från Del 1, med riktigt lösenord.
- **Type:** `SQLAzure`.
- **Deployment slot setting:** lämna urkryssad.

Klicka **Apply** i panelen, och därefter **Apply** längst ner på sidan. Det andra klicket är lätt att glömma, och utan det sparas ingenting. Bekräfta omstarten av appen.

I App Service lagras Azure SQL-anslutningar som miljövariabler med prefixet `SQLAZURECONNSTR_`, och .NET kan läsa dem via `GetConnectionString("DefaultConnection")` utan att du ändrar koden.

### Steg 6 — Kontrollera brandväggen

Sök efter **Azure SQL** i portalen och öppna **All resources**. Klicka på servern, alltså raden med `Resource type: Microsoft.Sql/servers`. I vänstermenyn: fäll ut **Security** och välj **Networking**.

Scrolla ner till rubriken `Exceptions`, längst ner på sidan under brandväggsreglerna. Där sitter kryssrutan **Allow Azure services and resources to access this server** — kontrollera att den är ikryssad.

Utan den kommer App Service inte åt databasen, och felet dyker upp först när du deployat och undrar varför appen fungerade lokalt.

## Fas C — Azure DevOps

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

- **Identity type:** `App registration (automatic)`. Den är märkt `Recommended` och skapar registreringen åt dig.
- **Credential:** `Workload identity federation`. Också `Recommended`. Den använder federerad inloggning i stället för en hemlighet som måste förnyas — alternativet `Secret` fungerar men går ut efter ett par år.
- **Scope level:** `Subscription`.
- **Subscription:** din prenumeration, oftast redan vald.
- **Resource group:** `rg-dev`. Här står ofta något annat förvalt, till exempel en `DefaultResourceGroup-` som Azure skapat automatiskt. Byt till `rg-dev` — det begränsar vad pipelinen får röra.
- **Service Connection Name:** `azure-todo-one`. Skriv exakt så. Strängen ska in i YAML-filen i nästa steg och matchningen är bokstavlig.
- **Service Management Reference:** lämna tom. Används i organisationer som kräver spårbarhet mot ett internt register.
- **Description:** valfri, men värd en rad, till exempel `Deploys the todo-one demo app to App Service`.
- **Grant access permission to all pipelines:** kryssa i. Utan den måste du godkänna kopplingen manuellt vid första körningen.

Klicka **Save**.

Går det inte igenom beror det ofta på att kontot inte får skapa app-registreringar eller federerade kopplingar i tenanten.

## Fas D — Pipelinen

### Steg 10 — Lägg till YAML-filen

Filen skapar du lokalt på din dator, i repo-roten — alltså samma mapp som `TodoOne.slnx` ligger i, till exempel `C:\git\todo-one`. Inte i `src`, och inte via GitHubs webbgränssnitt. Sedan committar och pushar du den som vanligt, så hamnar den på GitHub.

Skapa `azure-pipelines.yml` med det här innehållet:

```yaml
trigger:
  branches:
    include:
      - main

pr:
  branches:
    include:
      - main

variables:
  buildConfiguration: 'Release'
  dotnetSdkVersion: '10.0.x'
  projectPath: 'src/TodoOne/TodoOne.csproj'
  testProjectPath: 'src/TodoOne.Tests/TodoOne.Tests.csproj'
  azureServiceConnection: 'azure-todo-one'
  webAppName: 'app-todo-one-mn'

stages:
  - stage: Build
    displayName: 'Build and test'
    jobs:
      - job: BuildJob
        pool:
          vmImage: 'windows-latest'
        steps:
          - task: UseDotNet@2
            displayName: 'Install .NET SDK'
            inputs:
              packageType: 'sdk'
              version: $(dotnetSdkVersion)

          - task: DotNetCoreCLI@2
            displayName: 'Restore'
            inputs:
              command: 'restore'
              projects: |
                $(projectPath)
                $(testProjectPath)

          - task: DotNetCoreCLI@2
            displayName: 'Build'
            inputs:
              command: 'build'
              projects: |
                $(projectPath)
                $(testProjectPath)
              arguments: '--configuration $(buildConfiguration) --no-restore'

          - task: DotNetCoreCLI@2
            displayName: 'Test'
            inputs:
              command: 'test'
              projects: '$(testProjectPath)'
              arguments: '--configuration $(buildConfiguration) --no-build --logger trx'
              publishTestResults: true

          - task: DotNetCoreCLI@2
            displayName: 'Publish'
            inputs:
              command: 'publish'
              publishWebProjects: false
              projects: '$(projectPath)'
              arguments: '--configuration $(buildConfiguration) --no-build --output $(Build.ArtifactStagingDirectory)'
              zipAfterPublish: true

          - task: PublishBuildArtifacts@1
            displayName: 'Publish artifact'
            inputs:
              PathtoPublish: '$(Build.ArtifactStagingDirectory)'
              ArtifactName: 'drop'

  - stage: Deploy
    displayName: 'Deploy to App Service'
    dependsOn: Build
    condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
    jobs:
      - deployment: DeployJob
        pool:
          vmImage: 'windows-latest'
        environment: 'production'
        strategy:
          runOnce:
            deploy:
              steps:
                - download: current
                  artifact: 'drop'

                - task: AzureWebApp@1
                  displayName: 'Deploy Azure Web App'
                  inputs:
                    azureSubscription: '$(azureServiceConnection)'
                    appType: 'webApp'
                    appName: '$(webAppName)'
                    package: '$(Pipeline.Workspace)/drop/**/*.zip'
                    deploymentMethod: 'auto'
```

Den här filen gör fyra saker:

- Bygger appen.
- Kör testerna.
- Publicerar en deploybar artefakt.
- Deployar till App Service när ändringen kommer från `main`.

Att sätta `UseDotNet@2` till `10.0.x` före restore och build är viktigt när projektet targetar .NET 10. Det minskar risken att agenten använder en för gammal SDK.

Test- och publish-stegen använder `--no-build`, eftersom build redan har körts. Det sparar tid och undviker att koden byggs om i onödan — men det förutsätter att `Build`-steget faktiskt byggt båda projekten. Därför står både `projectPath` och `testProjectPath` under `projects` i både restore och build.

Committa och pusha filen:

```powershell
git add azure-pipelines.yml
git commit -m "Add Azure DevOps pipeline"
git push
```

### Steg 11 — Koppla pipelinen till GitHub

I Azure DevOps-projektet:

1. Gå till **Pipelines → Pipelines**.
2. Klicka **Create Pipeline**.
3. Välj **GitHub**. Du skickas till GitHub för inloggning.
4. Har du inte kopplat Azure Pipelines till GitHub tidigare visas sidan **Approve & Install Azure Pipelines**. Låt `Only select repositories` vara valt med `todo-one` i listan — då får appen bara se det repot. Klicka **Approve & Install**.

   Behörigheterna som listas ser omfattande ut men behövs. Läs- och skrivrättighet på `checks`, `commit statuses` och `pull requests` är vad som gör att bygget kan rapportera grönt eller rött i dina pull requests i steg 16.

5. Välj repot `todo-one`.
6. Välj **Existing Azure Pipelines YAML file**.
7. Ange branch `main` och path `/azure-pipelines.yml`.
8. Klicka **Run**.

Syns inte `todo-one` i listan i punkt 5 beror det på att appen installerades med `Only select repositories` vid ett tidigare tillfälle, och det nya repot inte fanns då. Lägg till det i GitHub under ditt konto → **Settings** → **Applications** → **Azure Pipelines** → **Configure** → `Repository access`.

Det är bättre att välja din egen YAML-fil än att låta Azure DevOps generera en mall åt dig. Då behåller du kontrollen över SDK-version, paths och deploylogik.

### Steg 12 — Godkänn behörigheter första gången

Första körningen kan stanna med ett meddelande om att pipelinen behöver åtkomst till en resurs.

Det här är vad du ska förvänta dig:

- Meddelandet är normalt första gången.
- Du klickar **View** och sedan **Permit**.
- Det kan behöva göras mer än en gång, till exempel för både service connection och environment.

### Steg 13 — Följ första körningen

Öppna körningen och följ stegen.

Det här är vad du ska förvänta dig:

- **Build** ska bli grön.
- **Deploy** ska bli grön.
- När deployen är klar ska appen gå att öppna på din publika adress, med `/todos` på slutet.

Första anropet kan vara långsamt om appen precis startats eller om databasen varit vilande.

## Fas E — PR-flödet

### Steg 14 — Gör en ändring på egen gren

```powershell
git checkout -b feature/add-count
```

Ändra rubriken i `src/TodoOne/Pages/Todos.cshtml` till:

```cshtml
<h1>Todo list (@Model.Items.Count)</h1>
```

Commit och pusha:

```powershell
git add .
git commit -m "Show item count in heading"
git push --set-upstream origin feature/add-count
```

### Steg 15 — Skapa pull request

Öppna repot på GitHub och skapa en pull request från `feature/add-count` till `main`.

Det här är vad du ska förvänta dig:

- En pipeline startar automatiskt för PR:n.
- Build och test ska köras.
- Deploy ska inte köras, eftersom YAML-filen uttryckligen blockerar deploy från PR-flödet.

Det är precis så PR-validering ska fungera.

### Steg 16 — Kräv grönt bygge före merge

I GitHub kan du skydda `main` så att bara gröna byggen får mergas.

Gå till:

**Settings → Rules → Rulesets**

Skapa ett ruleset för default branch och slå på:

- **Require a pull request before merging**
- **Require status checks to pass**

Välj din pipeline som obligatorisk check när den väl har kört minst en gång mot en PR.

### Steg 17 — Merge och deploy

När pull requesten är grön:

1. Klicka **Merge pull request**.
2. Bekräfta.
3. Ta bort grenen.

Efter merge triggas en ny körning på `main`. Nu ska både Build och Deploy gå igenom.

Städa lokalt efteråt:

```powershell
git checkout main
git pull
git branch -d feature/add-count
```

## Fas F — Verifiera och städa

### Steg 18 — Verifiera hela kedjan

| Vad | Var |
|---|---|
| Webbsidan | Din publika adress + `/todos` |
| API:et | Din publika adress + `/api/todo` |
| Data i databasen | **SQL Server Object Explorer** → `SELECT * FROM Todos;` |

Lägg till en post via webbsidan i Azure och kontrollera att den dyker upp i databasen. Då vet du att App Service, deploy, connection string och databas faktiskt hänger ihop.

Om något inte fungerar är **Log stream** i App Service ofta första rätta stället att titta. Där syns startup-fel och databasproblem direkt.

### Steg 19 — Kostnad och städning

App Service Free F1 kostar inget, men Azure SQL kan kosta beroende på vilken nivå du valde i Del 1.

Vanliga val efter demo:

- Ta bort bara appen och App Service-planen.
- Låt databasen ligga kvar.
- Eller ta bort hela resource groupen om allt i gruppen får försvinna.

Budgetlarm sätter du under **Subscriptions → Cost Management → Budgets**.

## Klar?

- [ ] Koden ligger på GitHub utan hemligheter i historiken.
- [ ] App Service finns och har `DefaultConnection` under **Connection strings**.
- [ ] Pipelinen bygger, testar och deployar vid push till `main`.
- [ ] En pull request bygger och testar men deployar inte.
- [ ] `main` går inte att merga till utan grönt bygge.
- [ ] Webbsidan och API:et svarar på den publika adressen.
- [ ] En post skapad i molnet syns i databasen.

## Vad du faktiskt har byggt

Om du bockat av allt ovan, stanna upp en sekund. Du skriver en rad kod på din egen dator, gör en commit, och några minuter senare kör den raden i ett datacenter, mot en databas i molnet, på en adress vem som helst i världen kan öppna. Ingen manuell publicering, inget FTP, ingen som behöver klicka. Du har byggt kedjan som gör det.

Det tog många moment, och det är värt att säga varför. Det handlar inte om att du gjorde det svårt för dig. Du har kopplat samman fyra separata system — din maskin, GitHub, Azure DevOps och Azure — och varje koppling behöver en identitet, en behörighet och ett namn som stämmer på båda sidor. Lägg till att ingenting får läcka ett lösenord på vägen, och komplexiteten är inte överdriven. Den är kostnaden för att göra det på ett sätt som håller.

Den goda nyheten är att du betalade den kostnaden en gång. Del 1 gör du aldrig igen. Del 2 och 3 tar en halvtimme nästa gång, för nu vet du var fällorna sitter: sökvägen till projektfilen, namnet på service connection, appnamnet i YAML-filen, brandväggen som måste släppa in Azure-tjänster.

Och du har något att visa. Inte en todo-lista — dem finns det tusentals av — utan en fungerande CI/CD-kedja med tester, PR-validering och en app i drift. Det är den skillnaden en arbetsgivare ser.

Nästa gång du får en idé är avståndet från tanke till körande app i molnet en kvart. Det är hela poängen.

## Felsökning

### F1. `Project file does not exist` eller `MSB1003`

`projectPath` matchar inte repots verkliga struktur. Lägg till ett diagnostiskt steg i Build för att lista alla projektfiler och använd den faktiska sökvägen därifrån.

```yaml
- script: dir /s /b *.csproj
  displayName: 'List project files'
```

### F2. `The current .NET SDK does not support targeting .NET 10.0`

Agenten använder fel SDK. Kontrollera att `UseDotNet@2` ligger före restore och build och att versionen är `10.0.x`.

### F3. `This pipeline needs permission to access a resource`

Normalt vid första körningen. Öppna körningen, klicka **View**, sedan **Permit**.

### F4. `No hosted parallelism has been purchased or granted`

Organisationen saknar tillgång till Microsofts byggagenter. Se steg 8 och koppla organisationen till din Azure-prenumeration under **Organization settings** → **Billing**, vilket normalt aktiverar gratisnivån direkt.

Är faktureringen redan uppsatt och felet kvarstår, ansök om grantet på [aka.ms/azpipelines-parallelism-request](https://aka.ms/azpipelines-parallelism-request) och räkna med några arbetsdagars handläggning.

Kontrollera aktuellt läge under **Organization settings** → **Pipelines** → **Parallel jobs**. Att byta till ett publikt projekt är inte längre en utväg — publika projekt är avvecklade och går inte att skapa.

Reglerna kring gratisnivån har ändrats flera gånger. Stämmer inte det du ser med beskrivningen här, utgå från [Microsofts dokumentation om parallel jobs](https://learn.microsoft.com/en-us/azure/devops/pipelines/licensing/concurrent-jobs).

### `The process 'dotnet.exe' failed with exit code 1` i Test-steget

Bygget gick igenom men testerna föll. Klicka på det röda steget och läs loggen — den säger vilket av två saker som hänt.

Står det att en assembly inte hittas, eller att projektet inte är byggt, har `Build`-steget inte byggt testprojektet. Kontrollera att både `$(projectPath)` och `$(testProjectPath)` står under `projects` i build-steget. `dotnet test --no-build` kräver att projektet redan är byggt.

Rapporterar loggen i stället ett misslyckat test är det koden som klagar, och då står det vilket test och varför.

### F5. `Could not find any file matching the pattern` i deploy-steget

Artefakten hittades inte. Kontrollera att `ArtifactName: 'drop'` i build matchar `artifact: 'drop'` i deploy och att `zipAfterPublish: true` är kvar.

### F6. `Failed to create an app in Azure Active Directory`

Kontot får inte skapa app-registreringar eller federerade identiteter i tenanten. Be en admin om hjälp, eller skapa kopplingen manuellt.

### F7. Pipelinen triggas inte vid push

Kontrollera att `trigger` pekar på `main`, att grenen faktiskt heter `main` och att GitHub-auktoriseringen fortfarande gäller.

### F8. .NET 10 saknas i Runtime stack-listan

Nya stackar rullas ut stegvis mellan regioner. Testa en annan region om det behövs. Som reservspår kan appen publiceras self-contained, men det bör ses som en fallback snarare än förstahandsval.

### F9. `HTTP Error 500.30 - ASP.NET Core app failed to start`

Appen kraschar under uppstart. Titta först i **Log stream**. Vanliga orsaker är fel connection string eller problem när `db.Database.Migrate()` körs.

### F10. `Cannot open server ... Client with IP address ... is not allowed`

Det är brandväggen. Kommer felet från App Service saknas ofta tillåtelsen för Azure-tjänster på SQL-servern.

### F11. `Login failed for user`

Lösenordet i App Service är fel, eller connection stringen blev trasig vid inklistring. Visa värdet i **Connection strings** och kontrollera det noga.

### F12. Webbsidan fungerar men API:et ger 404 i Azure

Kontrollera att senaste deploy faktiskt gick ut och att rätt artefakt användes. Kudu kan hjälpa dig att jämföra vad som ligger i `site/wwwroot`.

### F13. Första anropet efter en stund timeoutar

App Service på Free F1 kan sova, och Azure SQL serverless kan också vara pausad. Vänta en stund och prova igen.

## Bygg vidare

När grundflödet fungerar finns flera naturliga nästa steg:

- **Managed Identity i stället för SQL-lösenord.** Kräver att SQL-servern har en Entra-admin, vilket den har om du valde **Use both SQL and Microsoft Entra authentication** i Del 1.

  1. App Service → **Settings** → **Identity** → **System assigned** → **Status: On** → **Save**.
  2. Anslut till databasen som Entra-admin, via **Query editor** i portalen.
  3. Kör:

     ```sql
     CREATE USER [app-todo-one-mn] FROM EXTERNAL PROVIDER;
     ALTER ROLE db_datareader ADD MEMBER [app-todo-one-mn];
     ALTER ROLE db_datawriter ADD MEMBER [app-todo-one-mn];
     ALTER ROLE db_ddladmin ADD MEMBER [app-todo-one-mn];
     ```

  4. Byt connection string i App Service till `Server=tcp:sql-dev-mn.database.windows.net,1433;Initial Catalog=sqldb-todo-one;Authentication=Active Directory Default;Encrypt=True;` — utan `User ID` och `Password`.
  5. Starta om appen.

  Kommandot i punkt 3 måste köras av Entra-admin. SQL-admin har inte rätt att skapa externa användare. `db_ddladmin` behövs bara så länge appen kör `db.Database.Migrate()` vid start.
- **Staging slot** för säkrare deployer.
- **Migrationer som eget pipeline-steg** i stället för att köra dem vid startup.
- **Integrationstester** med `WebApplicationFactory`.
- En riktig `README.md` i repot som förklarar arkitekturen.