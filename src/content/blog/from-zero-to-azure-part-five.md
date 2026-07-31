---
title: "From Zero To Azure Part Five"
description: "Del 5 av fem: bygg en Azure DevOps-pipeline som bygger, testar och driftsätter appen automatiskt vid varje push."
pubDate: 2026-06-21
heroImage: "../../assets/images/blog/from-zero-to-azure-part-five/hero-from-zero-to-azure-part-five.jpg"
tags: ["Azure"]
---

# Del 5 — Bygg pipelinen

> **Serien From Zero to Azure**
> [Del 1 — Sätt upp miljön](/blog/from-zero-to-azure-part-one) · [Del 2 — Håll koll på kostnaderna](/blog/from-zero-to-azure-part-two) · [Del 3 — Bygg appen](/blog/from-zero-to-azure-part-three) · [Del 4 — Sätt upp tjänsterna](/blog/from-zero-to-azure-part-four) · **Del 5 — Bygg pipelinen** (du är här)

Sista delen. Tjänsterna står på plats efter Del 4 — nu binder vi ihop dem.

När du är klar händer det här varje gång du pushar till `main`: koden hämtas, byggs, testas, paketeras och läggs ut på din App Service. Utan att du klickar på något.

**Förutsättning:** Del 4 är klar. Du har ett GitHub-repo, en App Service med connection string, ett Azure DevOps-projekt med byggagenter, och en service connection.

Tre namn från Del 4 återkommer i YAML-filen och måste stämma exakt:

| Vad | Exempel |
| --- | --- |
| Service connection | `azure-todo-one` |
| App Service | `app-todo-one-mn` |
| Projektfilens sökväg | `src/TodoOne/TodoOne.csproj` |

## Fas 1 — Pipelinen

### Steg 1 — Lägg till YAML-filen

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

### Steg 2 — Koppla pipelinen till GitHub

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

### Steg 3 — Godkänn behörigheter första gången

Första körningen kan stanna med ett meddelande om att pipelinen behöver åtkomst till en resurs.

Det här är vad du ska förvänta dig:

- Meddelandet är normalt första gången.
- Du klickar **View** och sedan **Permit**.
- Det kan behöva göras mer än en gång, till exempel för både service connection och environment.

### Steg 4 — Följ första körningen

Öppna körningen och följ stegen.

Det här är vad du ska förvänta dig:

- **Build** ska bli grön.
- **Deploy** ska bli grön.
- När deployen är klar ska appen gå att öppna på din publika adress, med `/todos` på slutet.

Första anropet kan vara långsamt om appen precis startats eller om databasen varit vilande.

## Fas 2 — PR-flödet

### Steg 5 — Gör en ändring på egen gren

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

### Steg 6 — Skapa pull request

Öppna repot på GitHub och skapa en pull request från `feature/add-count` till `main`.

Det här är vad du ska förvänta dig:

- En pipeline startar automatiskt för PR:n.
- Build och test ska köras.
- Deploy ska inte köras, eftersom YAML-filen uttryckligen blockerar deploy från PR-flödet.

Det är precis så PR-validering ska fungera.

### Steg 7 — Kräv grönt bygge före merge

I GitHub kan du skydda `main` så att bara gröna byggen får mergas.

Gå till:

**Settings → Rules → Rulesets**

Skapa ett ruleset för default branch och slå på:

- **Require a pull request before merging**
- **Require status checks to pass**

Välj din pipeline som obligatorisk check när den väl har kört minst en gång mot en PR.

### Steg 8 — Merge och deploy

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

## Fas 3 — Verifiera och städa

### Steg 9 — Verifiera hela kedjan

| Vad | Var |
|---|---|
| Webbsidan | Din publika adress + `/todos` |
| API:et | Din publika adress + `/api/todo` |
| Data i databasen | **SQL Server Object Explorer** → `SELECT * FROM Todos;` |

Lägg till en post via webbsidan i Azure och kontrollera att den dyker upp i databasen. Då vet du att App Service, deploy, connection string och databas faktiskt hänger ihop.

Om något inte fungerar är **Log stream** i App Service ofta första rätta stället att titta. Där syns startup-fel och databasproblem direkt.

### Steg 10 — Kostnad och städning

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

Den goda nyheten är att du betalade den kostnaden en gång. Del 1 och 2 gör du aldrig igen. Del 3 till 5 tar en halvtimme nästa gång, för nu vet du var fällorna sitter: sökvägen till projektfilen, namnet på service connection, appnamnet i YAML-filen, brandväggen som måste släppa in Azure-tjänster.

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

Organisationen saknar tillgång till Microsofts byggagenter. Se steg 8 i Del 4 och koppla organisationen till din Azure-prenumeration under **Organization settings** → **Billing**, vilket normalt aktiverar gratisnivån direkt.

Är faktureringen redan uppsatt och felet kvarstår, ansök om grantet på [aka.ms/azpipelines-parallelism-request](https://aka.ms/azpipelines-parallelism-request) och räkna med några arbetsdagars handläggning.

Kontrollera aktuellt läge under **Organization settings** → **Pipelines** → **Parallel jobs**. Att byta till ett publikt projekt är inte längre en utväg — publika projekt är avvecklade och går inte att skapa.

Reglerna kring gratisnivån har ändrats flera gånger. Stämmer inte det du ser med beskrivningen här, utgå från [Microsofts dokumentation om parallel jobs](https://learn.microsoft.com/en-us/azure/devops/pipelines/licensing/concurrent-jobs).

### F5. `The process 'dotnet.exe' failed with exit code 1` i Test-steget

Bygget gick igenom men testerna föll. Klicka på det röda steget och läs loggen — den säger vilket av två saker som hänt.

Står det att en assembly inte hittas, eller att projektet inte är byggt, har `Build`-steget inte byggt testprojektet. Kontrollera att både `$(projectPath)` och `$(testProjectPath)` står under `projects` i build-steget. `dotnet test --no-build` kräver att projektet redan är byggt.

Rapporterar loggen i stället ett misslyckat test är det koden som klagar, och då står det vilket test och varför.

### F6. `Could not find any file matching the pattern` i deploy-steget

Artefakten hittades inte. Kontrollera att `ArtifactName: 'drop'` i build matchar `artifact: 'drop'` i deploy och att `zipAfterPublish: true` är kvar.

### F7. Pipelinen triggas inte vid push

Kontrollera att `trigger` pekar på `main`, att grenen faktiskt heter `main` och att GitHub-auktoriseringen fortfarande gäller.

### F8. `HTTP Error 500.30 - ASP.NET Core app failed to start`

Appen kraschar under uppstart. Titta först i **Log stream**. Vanliga orsaker är fel connection string eller problem när `db.Database.Migrate()` körs.

### F9. `Cannot open server ... Client with IP address ... is not allowed`

Det är brandväggen. Kommer felet från App Service saknas ofta tillåtelsen för Azure-tjänster på SQL-servern.

### F10. `Login failed for user`

Lösenordet i App Service är fel, eller connection stringen blev trasig vid inklistring. Visa värdet i **Connection strings** och kontrollera det noga.

### F11. Webbsidan fungerar men API:et ger 404 i Azure

Kontrollera att senaste deploy faktiskt gick ut och att rätt artefakt användes. Kudu kan hjälpa dig att jämföra vad som ligger i `site/wwwroot`.

### F12. Första anropet efter en stund timeoutar

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
