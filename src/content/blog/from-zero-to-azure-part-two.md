---
title: "From Zero To Azure Part Two"
description: "Del 2 av tre: bygg en ASP.NET Core-app med CRUD mot Azure SQL, från tomt skelett till körande app på din egen maskin."
pubDate: 2026-05-31
heroImage: "../../assets/images/blog/from-zero-to-azure-part-two/from-zero-to-azure-part-two.jpg"
tags: ["Azure"]
---
# Del 2 — Bygg appen

> **Serien From Zero to Azure**
> [Del 1 — Sätt upp miljön](/blog/from-zero-to-azure-part-one) · **Del 2 — Bygg appen** (du är här) · [Del 3 — Ut i molnet](/blog/from-zero-to-azure-part-three)

Det här är andra delen av tre. Här går du från tom mapp till en fungerande app med databas, tester och en lokal körning.

Del 1 är gjord en gång. Del 2 och Del 3 är upplagda för att gå att återanvända nästa gång du vill bygga en ny app. Byt bara namnen och kör igen.

Förutsättning: Del 1 är klar. Du har .NET 10 SDK, Git, Visual Studio 2026 och en Azure SQL-databas med anslutningsuppgifter.

## Så jobbar vi

Del 2 har två faser.

- **Fas A** bygger skelettet från terminalen. Det gör stegen tydliga, repeterbara och oberoende av hur Visual Studio råkar se ut i just din version.
- **Fas B** fyller på innehållet i Visual Studio. Där är IntelliSense, debuggern och refaktorisering mer värda än kommandoraden.

Guiden förutsätter att du vet vad `mkdir`, `cd` och `dir` gör. Den förutsätter inte att du kan EF Core eller Razor Pages.

## Namnen för den här omgången

| Vad | Exempel |
|---|---|
| Mapp och repo | `todo-one` |
| Solution, projekt, namespace | `TodoOne` |
| Testprojekt | `TodoOne.Tests` |
| Databas från Del 1 | `sqldb-todo-one` |

Utsidan — mappar, repon och molnresurser — skrivs med gemener och bindestreck. .NET-sidan skrivs i PascalCase, eftersom projektnamnet automatiskt blir både assembly-namn och root namespace.

## Fas A — Skelettet

### Steg 1 — Skapa mappen och initiera Git

Öppna **PowerShell**: tryck Windows-tangenten, skriv `powershell` och tryck Enter. Du kan också högerklicka på Start-knappen och välja **Terminal**.

PowerShell är standard i Windows 11, hanterar långa sökvägar bra och ger dig autokomplettering med Tab.

Alla kommandon i guiden är skrivna för PowerShell. Flera av dem, som `New-Item` och `Get-ChildItem`, finns inte i den gamla kommandotolken `cmd` — kör du dem där får du bara ett felmeddelande om att kommandot inte hittas.

Vi lägger alla repon under `C:\git`. Har du en annan struktur du trivs med kan du använda den. Det viktiga är att du vet var repo-roten ligger.

```powershell
mkdir C:\git\todo-one
cd C:\git\todo-one
git init
dotnet new gitignore
```

`dotnet new gitignore` lägger en `.gitignore` anpassad för .NET i mappen. Utan den riskerar du att committa `bin`, `obj` och användarspecifika filer direkt.

### Steg 2 — Skapa solution och projekt

Kommandona nedan skapar filer relativt den mapp du står i, så kontrollera att du står i repo-roten. Prompten ska sluta på `todo-one`:

```powershell
PS C:\git\todo-one>
```

Gör den inte det, navigera dit — eller till den mapp du valde i steg 1:

```powershell
cd C:\git\todo-one
```

Samma sak gäller alla kommandon i fas A. Står du fel hamnar projektet på fel plats, och sökvägarna i pipelinen i Del 3 slutar stämma.

```powershell
dotnet new sln --name TodoOne
dotnet new webapp --name TodoOne --output src/TodoOne --framework net10.0
dotnet new xunit --name TodoOne.Tests --output src/TodoOne.Tests --framework net10.0

dotnet sln add src/TodoOne/TodoOne.csproj
dotnet sln add src/TodoOne.Tests/TodoOne.Tests.csproj
dotnet add src/TodoOne.Tests/TodoOne.Tests.csproj reference src/TodoOne/TodoOne.csproj
```

Tre kommandon skapar solutionen och projekten. Tre kommandon kopplar ihop dem.

Du kan klistra in hela blocket på en gång — PowerShell kör raderna i tur och ordning. Varnar Windows Terminal för att du klistrar in flera rader är det bara en bekräftelsefråga.

Kom bara ihåg att körningen inte stannar vid fel. Misslyckas ett kommando körs resten ändå, och följdfelen kan bli svårlästa. Läs utskriften innan du går vidare, eller kör ett kommando i taget om du vill ha full kontroll. Det gäller alla flerradiga block i guiden.

I .NET 10 skapar `dotnet new sln` som standard en `*.slnx`-fil i det nya XML-baserade formatet i stället för en klassisk `*.sln`-fil. Visual Studio 2026 öppnar den utan problem.

### Steg 3 — Lägg till paketen

```powershell
dotnet add src/TodoOne/TodoOne.csproj package Microsoft.EntityFrameworkCore.SqlServer
dotnet add src/TodoOne/TodoOne.csproj package Microsoft.EntityFrameworkCore.Tools
dotnet add src/TodoOne.Tests/TodoOne.Tests.csproj package Microsoft.EntityFrameworkCore.InMemory
```

`SqlServer` är providern mot Azure SQL. `Tools` behövs för migrationskommandon i Package Manager Console. `InMemory` gör att testerna kan köra utan att röra Azure.

### Steg 4 — Skapa tomma kodfiler

Vi använder `dotnet new` även för enskilda filer. Då hamnar de direkt på rätt plats med rätt namespace.

```powershell
dotnet new class --name Todo --output src/TodoOne/Models
dotnet new class --name AppDbContext --output src/TodoOne/Data
dotnet new apicontroller --name TodoController --output src/TodoOne/Controllers
dotnet new page --name Todos --output src/TodoOne/Pages
dotnet new class --name TodoTests --output src/TodoOne.Tests
```

`page`-mallen skapar två filer: `Todos.cshtml` och `Todos.cshtml.cs`.

Ta sedan bort den testfil som xUnit-mallen skapade åt dig:

```powershell
Remove-Item src\TodoOne.Tests\UnitTest1.cs
```

### Steg 5 — Kontrollera resultatet

```powershell
dotnet build
```

Det här är vad du ska förvänta dig:

- Bygget ska gå igenom utan fel.
- Några varningar kan förekomma, men inga röda fel.
- Om det faller här beror det nästan alltid på att något kommando i steg 2 eller 3 inte kördes som det skulle.

Så här ska strukturen se ut i stora drag:

```text
C:\git\todo-one\
├─ .git\
├─ .gitignore
├─ TodoOne.slnx
└─ src\
   ├─ TodoOne\
   │  ├─ TodoOne.csproj
   │  ├─ Program.cs
   │  ├─ appsettings.json
   │  ├─ Controllers\
   │  │  └─ TodoController.cs
   │  ├─ Data\
   │  │  └─ AppDbContext.cs
   │  ├─ Models\
   │  │  └─ Todo.cs
   │  ├─ Pages\
   │  │  ├─ Todos.cshtml
   │  │  ├─ Todos.cshtml.cs
   │  │  ├─ Index.cshtml
   │  │  ├─ Privacy.cshtml
   │  │  └─ Shared\
   │  │     └─ _Layout.cshtml
   │  ├─ Properties\
   │  └─ wwwroot\
   └─ TodoOne.Tests\
      ├─ TodoOne.Tests.csproj
      └─ TodoTests.cs
```

På disken ligger fler filer än så. Webapp-mallen skapar bland annat `Error.cshtml`, `_ViewImports.cshtml` och `_ViewStart.cshtml`, och bygget har lagt till `bin` och `obj` under båda projekten. Inget av det behöver du bry dig om.

Det som spelar roll är att de fem filerna du skapade i steg 4 finns på rätt plats. Kontrollera dem så här:

```powershell
Get-ChildItem -Recurse -Include Todo.cs,AppDbContext.cs,TodoController.cs,Todos.cshtml,TodoTests.cs |
    Select-Object -ExpandProperty FullName
```

Fem rader ska komma tillbaka:

```text
C:\git\todo-one\src\TodoOne\Controllers\TodoController.cs
C:\git\todo-one\src\TodoOne\Data\AppDbContext.cs
C:\git\todo-one\src\TodoOne\Models\Todo.cs
C:\git\todo-one\src\TodoOne\Pages\Todos.cshtml
C:\git\todo-one\src\TodoOne.Tests\TodoTests.cs
```

Saknas någon rad kördes motsvarande kommando i steg 4 inte som det skulle.

Lägg särskilt sökvägen `src/TodoOne/TodoOne.csproj` på minnet. Den behövs igen i Del 3.

### Steg 6 — Checka in skelettet

Stå i repo-roten. Punkten i `git add .` betyder den här mappen och allt under den, så står du i en undermapp får du bara med en del av projektet.

```powershell
git add .
git commit -m "Add project skeleton"
```

Du får sannolikt en lång rad varningar som ser ut så här:

```text
warning: in the working copy of 'src/TodoOne/wwwroot/lib/jquery/dist/jquery.slim.min.js',
LF will be replaced by CRLF the next time Git touches it
```

Det är inget fel. Git för Windows lagrar radbrytningar som LF i repot men skriver ut dem som CRLF på disk, och talar om att den gör det. De flesta varningarna kommer från de färdiga JavaScript-biblioteken under `wwwroot/lib` som webapp-mallen la dit. Filerna är oförändrade, och commiten går igenom som den ska.

Det här är en bra första commit: projektet bygger, strukturen är klar och du har en ren punkt att gå tillbaka till.

## Fas B — Innehållet

Öppna solutionen:

```powershell
Invoke-Item TodoOne.slnx
```

`Invoke-Item` gör samma sak som ett dubbelklick i Utforskaren: öppnar filen i det program Windows kopplat till filtypen. Eftersom `.slnx` hör till Visual Studio startas det med solutionen laddad.

Från och med nu sker resten av Del 2 i Visual Studio.

Behöver du köra ett kommando härifrån använder du Visual Studios inbyggda terminal: **Tools** → **Command Line** → **Terminal**. Den kör Developer PowerShell och öppnas i projektets mapp, så du slipper både växla fönster och navigera dit.

### Steg 7 — Modellen

Öppna `Models/Todo.cs` och ersätt innehållet med:

```csharp
using System.ComponentModel.DataAnnotations;

namespace TodoOne.Models;

public class Todo
{
    public int Id { get; set; }

    [Required]
    [StringLength(200)]
    public string Text { get; set; } = string.Empty;

    public bool Done { get; set; }
}
```

Attributen gör dubbelt jobb. EF Core använder dem när databasen byggs, och ASP.NET Core använder dem för modellvalidering.

### Steg 8 — DbContext

Ersätt innehållet i `Data/AppDbContext.cs`:

```csharp
using Microsoft.EntityFrameworkCore;
using TodoOne.Models;

namespace TodoOne.Data;

public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options)
        : base(options)
    {
    }

    public DbSet<Todo> Todos => Set<Todo>();
}
```

`DbContext` är appens session mot databasen. `DbSet<Todo>` motsvarar tabellen `Todos`.

### Steg 9 — Program.cs

Ersätt innehållet i `Program.cs`:

```csharp
using Microsoft.EntityFrameworkCore;
using TodoOne.Data;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddRazorPages();
builder.Services.AddControllers();

builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

var app = builder.Build();

if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();

app.MapRazorPages();
app.MapControllers();

using (var scope = app.Services.CreateScope())
{
    var db = scope.ServiceProvider.GetRequiredService<AppDbContext>();
    db.Database.Migrate();
}

app.Run();
```

Filen gör tre saker: registrerar tjänster, kopplar upp middleware och routing, och startar appen.

`db.Database.Migrate()` applicerar väntande migrationer när appen startar. Det är praktiskt i en demo och i lokal utveckling, men i skarp drift finns det invändningar. Det återkommer Del 3 till.

### Steg 10 — Connection string i User Secrets

Lägg först in en tom `ConnectionStrings`-sektion i `appsettings.json`:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": ""
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*"
}
```

Högerklicka sedan på projektet **TodoOne** i Solution Explorer och välj **Manage User Secrets**. Visual Studio öppnar då `secrets.json` för projektet.

Lägg in connection stringen du sparade i Del 1.

Har du den inte till hands hämtar du den i portalen: sök **Azure SQL** → **All resources** → klicka på databasen, alltså raden med `Resource type: Microsoft.Sql/servers/databases`. På `Overview` finns länken **Show database connection strings**.

Sidan listar fyra varianter. Ta den som heter **ADO.NET (SQL authentication)** — de andra är till för Microsoft Entra-inloggning, som vi inte använder här.

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=tcp:sql-dev-mn.database.windows.net,1433;Initial Catalog=sqldb-todo-one;Persist Security Info=False;User ID=<ditt-användarnamn>;Password=<ditt-lösenord>;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;"
  }
}
```

Ditt användarnamn står redan ifyllt i strängen från portalen. Lösenordet gör det inte — där står `{your_password}`, klammerparenteser och allt. Ersätt hela den delen med lösenordet du valde när servern skapades.

`secrets.json` ligger utanför projektmappen och är gjord för känsliga värden i utvecklingsmiljön. Det är därför rätt ställe för lösenord lokalt.

Committa innan du går vidare. Nästa steg är första gången något lämnar din maskin, och då är brandvägg, inloggning och nätverk med i bilden. Har du en commit här vet du att koden var hel när felsökningen började.

Från terminalen:

```powershell
git add .
git commit -m "Add Todo model, DbContext and connection string setup"
```

Eller i Visual Studio: **Git** → **Commit or Stash**, skriv meddelandet i rutan högst upp och klicka **Commit All**.

Menyvalet heter så för att det döljer två olika saker. En **commit** sparar en punkt i historiken som du alltid kan gå tillbaka till. En **stash** lägger dina ändringar åt sidan tillfälligt, så att du kan byta gren eller prova något annat, och plocka fram dem igen senare. Stash är en ficka, commit är ett bokmärke. Här vill du ha en commit.

`secrets.json` följer inte med — den ligger utanför repot. Kontrollera gärna att `appsettings.json` fortfarande har en tom `DefaultConnection`.

Härifrån committar du när du själv vill säkra en punkt i arbetet — efter varje avslutat steg är en bra vana, och migrationen i nästa steg är ett bra tillfälle. Guiden nämner det bara en gång till, i sista steget.

### Steg 11 — Skapa databastabellen

En migration är en beskrivning av hur databasen ska ändras för att matcha modellen.

Innan du kör något: kontrollera att `TodoOne` är **startup project**. Har solutionen flera projekt väljer Visual Studio ett åt dig, och det blir ofta testprojektet.

Högerklicka `TodoOne` i Solution Explorer och välj **Set as Startup Project**. Projektnamnet visas i fetstil när det är rätt satt.

Öppna sedan **Tools → NuGet Package Manager → Package Manager Console**. Högst upp i konsolfönstret sitter en verktygsrad med två listor, `Package source` och `Default project`. Kontrollera att `Default project` står på `src\TodoOne` eller `TodoOne`, beroende på hur Visual Studio visar projektet.

De två inställningarna gör olika saker och båda måste stämma. `Default project` bestämmer var migrationsfilerna hamnar. Startup project är det som EF Core Tools bygger och läser konfigurationen ur — alltså det som behöver ha `DbContext` och din anslutningssträng.

Kör följande i Package Manager Console, inte i terminalen:

```text
Add-Migration InitialCreate
```

Kommandot hämtar sina två projekt från inställningarna ovan: migrationsfilerna hamnar i det som står i `Default project`, och bygget läses från startup project. Klagar EF Core på fel projekt är det någon av dem som inte stämmer.

Det här är vad du ska förvänta dig:

- En mapp `Migrations` skapas i projektet.
- Du får en migrationsfil och en model snapshot.
- Kommandot ska avslutas utan fel om paketen i steg 3 är rätt installerade.

Applicera sedan migrationen, också i Package Manager Console:

```text
Update-Database
```

Det här är vad du ska förvänta dig:

- Tabellen `Todos` skapas i databasen.
- Tabellen `__EFMigrationsHistory` får en rad för `InitialCreate`.
- Inga fel om anslutningssträngen i User Secrets är korrekt.

Verifiera i **SQL Server Object Explorer**, som du kopplade upp i Del 1. Högerklicka `sqldb-todo-one` och välj **New Query**:

```sql
SELECT * FROM Todos;
SELECT * FROM __EFMigrationsHistory;
```

`Todos` ska finnas men vara tom. `__EFMigrationsHistory` ska innehålla en rad.

### Steg 12 — API:et

Ersätt innehållet i `Controllers/TodoController.cs`:

```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using TodoOne.Data;
using TodoOne.Models;

namespace TodoOne.Controllers;

[ApiController]
[Route("api/[controller]")]
public class TodoController : ControllerBase
{
    private readonly AppDbContext _db;

    public TodoController(AppDbContext db)
    {
        _db = db;
    }

    [HttpGet]
    public async Task<ActionResult<IEnumerable<Todo>>> GetAll()
    {
        return await _db.Todos.AsNoTracking().ToListAsync();
    }

    [HttpGet("{id:int}")]
    public async Task<ActionResult<Todo>> Get(int id)
    {
        var todo = await _db.Todos.FindAsync(id);

        if (todo is null)
        {
            return NotFound();
        }

        return todo;
    }

    [HttpPost]
    public async Task<ActionResult<Todo>> Create(Todo todo)
    {
        _db.Todos.Add(todo);
        await _db.SaveChangesAsync();

        return CreatedAtAction(nameof(Get), new { id = todo.Id }, todo);
    }

    [HttpPut("{id:int}")]
    public async Task<IActionResult> Update(int id, Todo input)
    {
        var todo = await _db.Todos.FindAsync(id);

        if (todo is null)
        {
            return NotFound();
        }

        todo.Text = input.Text;
        todo.Done = input.Done;
        await _db.SaveChangesAsync();

        return NoContent();
    }

    [HttpDelete("{id:int}")]
    public async Task<IActionResult> Delete(int id)
    {
        var todo = await _db.Todos.FindAsync(id);

        if (todo is null)
        {
            return NotFound();
        }

        _db.Todos.Remove(todo);
        await _db.SaveChangesAsync();

        return NoContent();
    }
}
```

Här får du fem endpoints: läsa alla, läsa en, skapa, uppdatera och ta bort. `AsNoTracking()` i läsningen är en enkel best practice när du bara ska läsa data och inte ändra den i samma kontext.

### Steg 13 — Webbsidan

Ersätt innehållet i `Pages/Todos.cshtml.cs`.

I Solution Explorer ligger den filen dold under `Todos.cshtml`. Klicka på pilen till vänster om `Todos.cshtml` för att fälla ut den. Visual Studio grupperar filer som hör ihop på det sättet, och samma sak gäller `Index.cshtml` och `Privacy.cshtml`.

```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;
using Microsoft.EntityFrameworkCore;
using TodoOne.Data;
using TodoOne.Models;

namespace TodoOne.Pages;

public class TodosModel : PageModel
{
    private readonly AppDbContext _db;

    public TodosModel(AppDbContext db)
    {
        _db = db;
    }

    public IReadOnlyList<Todo> Items { get; private set; } = [];

    [BindProperty]
    public string? NewText { get; set; }

    public async Task OnGetAsync()
    {
        Items = await _db.Todos.AsNoTracking().OrderBy(t => t.Id).ToListAsync();
    }

    public async Task<IActionResult> OnPostAddAsync()
    {
        if (!string.IsNullOrWhiteSpace(NewText))
        {
            _db.Todos.Add(new Todo { Text = NewText.Trim() });
            await _db.SaveChangesAsync();
        }

        return RedirectToPage();
    }

    public async Task<IActionResult> OnPostToggleAsync(int id)
    {
        var todo = await _db.Todos.FindAsync(id);

        if (todo is not null)
        {
            todo.Done = !todo.Done;
            await _db.SaveChangesAsync();
        }

        return RedirectToPage();
    }

    public async Task<IActionResult> OnPostDeleteAsync(int id)
    {
        var todo = await _db.Todos.FindAsync(id);

        if (todo is not null)
        {
            _db.Todos.Remove(todo);
            await _db.SaveChangesAsync();
        }

        return RedirectToPage();
    }
}
```

Ersätt innehållet i `Pages/Todos.cshtml`:

```cshtml
@page
@model TodoOne.Pages.TodosModel
@{
    ViewData["Title"] = "Todo list";
}

<h1>Todo list</h1>

<form method="post" asp-page-handler="Add" class="mb-4">
    <div class="input-group">
        <input asp-for="NewText" class="form-control" placeholder="What needs doing?" />
        <button type="submit" class="btn btn-primary">Add</button>
    </div>
</form>

@if (Model.Items.Count == 0)
{
    <p class="text-muted">Nothing here yet.</p>
}

<ul class="list-unstyled">
    @foreach (var item in Model.Items)
    {
        <li class="d-flex justify-content-between align-items-center border-bottom py-2">
            <span class="@(item.Done ? "text-decoration-line-through text-muted" : "")">
                @item.Text
            </span>

            <div class="d-flex gap-2">
                <form method="post" asp-page-handler="Toggle" asp-route-id="@item.Id">
                    <button type="submit" class="btn btn-sm btn-outline-secondary">
                        @(item.Done ? "Undo" : "Done")
                    </button>
                </form>

                <form method="post" asp-page-handler="Delete" asp-route-id="@item.Id">
                    <button type="submit" class="btn btn-sm btn-outline-danger">Delete</button>
                </form>
            </div>
        </li>
    }
</ul>
```

Razor Pages kopplar formulär till metoder via namn. `asp-page-handler="Add"` letar upp `OnPostAddAsync` och `asp-route-id` skickar med id till rätt metod.

Layouten kommer från Bootstrap, som redan ingår i mallen. `justify-content-between` på `<li>` trycker isär innehållet så att texten hamnar till vänster och knapparna till höger, och `<div>`-elementet runt de två formulären håller ihop dem som en grupp.

Lägg också till en menylänk i `Pages/Shared/_Layout.cshtml`. Leta upp den befintliga `Privacy`-posten och lägg den nya direkt efter, så att det ser ut så här:

```cshtml
<li class="nav-item">
    <a class="nav-link text-dark" asp-area="" asp-page="/Privacy">Privacy</a>
</li>
<li class="nav-item">
    <a class="nav-link text-dark" asp-area="" asp-page="/Todos">Todos</a>
</li>
```

`Privacy`-posten finns redan i filen — det är bara de fyra raderna under som är nya.

### Steg 14 — Testet

Ersätt innehållet i `TodoOne.Tests/TodoTests.cs`:

```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using TodoOne.Controllers;
using TodoOne.Data;
using TodoOne.Models;
using Xunit;

namespace TodoOne.Tests;

public class TodoTests
{
    private static AppDbContext CreateContext()
    {
        var options = new DbContextOptionsBuilder<AppDbContext>()
            .UseInMemoryDatabase(Guid.NewGuid().ToString())
            .Options;

        return new AppDbContext(options);
    }

    [Fact]
    public async Task Create_ThenGetAll_ReturnsTheItem()
    {
        await using var db = CreateContext();
        var controller = new TodoController(db);

        await controller.Create(new Todo { Text = "Buy coffee" });

        var result = await controller.GetAll();
        var items = Assert.IsAssignableFrom<IEnumerable<Todo>>(result.Value);

        Assert.Single(items);
        Assert.Equal("Buy coffee", items.First().Text);
    }

    [Fact]
    public async Task Delete_RemovesTheItem()
    {
        await using var db = CreateContext();
        var controller = new TodoController(db);
        db.Todos.Add(new Todo { Id = 1, Text = "Write CV" });
        await db.SaveChangesAsync();

        var result = await controller.Delete(1);

        Assert.IsType<NoContentResult>(result);
        Assert.Empty(db.Todos);
    }
}
```

Testerna kör mot en databas i minnet och rör aldrig Azure. `Guid.NewGuid()` gör att varje test får en egen isolerad databasinstans.

Kör testerna via **Test → Run All Tests** eller `Ctrl+R, A`.

Det här är vad du ska förvänta dig:

- Två gröna tester i Test Explorer.
- Inga beroenden till Azure för att testerna ska lyckas.
- Snabb körning, eftersom allt sker i minnet.

### Steg 15 — Kör appen

Tryck `F5`. Visual Studio bygger, startar appen och öppnar webbläsaren.

Testa:

- `https://localhost:<port>/todos` — lägg till några poster, markera en som klar och ta bort en. Du kan också klicka `Todos` i menyn, som är länken du la in i förra steget.
- `https://localhost:<port>/api/todo` — samma data som JSON.

Porten ser du i adressfältet och i `Properties/launchSettings.json`.

Adresserna är värda att skriva in för hand en gång. `/todos` kommer från filnamnet `Todos.cshtml` och `/api/todo` från klassnamnet `TodoController` minus suffixet — routing i ASP.NET Core bygger till stor del på den sortens konventioner, och de blir tydligare när man ser dem i adressfältet.

Vill du testa API:ets skrivoperationer går det bra med `curl`:

```powershell
curl -k -X POST https://localhost:<port>/api/todo -H "Content-Type: application/json" -d "{\"text\":\"From the API\",\"done\":false}"
```

Ladda om `/todos`. Posten du skapade via API:et ska nu synas även på sidan.

Kontrollera också i **SQL Server Object Explorer** att raderna verkligen ligger i Azure:

```sql
SELECT * FROM Todos;
```

### Steg 16 — Sista committen

Gör en sista commit, till exempel med meddelandet:

```text
Add CRUD for todos with API, page and tests
```

Öppna sedan `appsettings.json` en sista gång och kontrollera att `DefaultConnection` fortfarande är tom. Det är en enkel men viktig best practice.

## Klar?

- [ ] `dotnet build` går igenom utan fel.
- [ ] Test Explorer visar två gröna tester.
- [ ] `/todos` går att lägga till, markera klar och ta bort i.
- [ ] `/api/todo` returnerar samma data som JSON.
- [ ] Raderna syns i databasen.
- [ ] `appsettings.json` innehåller inget lösenord.
- [ ] Allt är committat.

Då är du redo för Del 3.

## Felsökning

### `Your startup project 'TodoOne.Tests' doesn't reference Microsoft.EntityFrameworkCore.Design`

Visual Studio har valt testprojektet som startup project. EF Core Tools bygger och läser konfiguration därifrån, och testprojektet har varken `DbContext` eller anslutningssträng.

Högerklicka `TodoOne` i Solution Explorer och välj **Set as Startup Project**, och kontrollera att `Default project` i konsolen står på `TodoOne`.

Vill du kringgå inställningarna helt kan du peka ut projekten i kommandot:

```text
Add-Migration InitialCreate -Project TodoOne -StartupProject TodoOne
```

### `File '...\src\TodoOne\bin\Debug\net10.0\TodoOne.Tests.dll' not found`

Samma orsak, men i ett halvfärdigt tillstånd: konsolen letar efter testprojektets assembly i app-projektets utdatamapp. Sätt `TodoOne` som startup project, kör **Build → Clean Solution** följt av **Build → Rebuild Solution**, stäng Package Manager Console och öppna den igen. Går det fortfarande inte, använd `-Project` och `-StartupProject` som ovan.

### `Add-Migration` hittas inte i Package Manager Console

`Microsoft.EntityFrameworkCore.Tools` saknas i projektet, eller så behöver Visual Studio startas om efter att paketet installerats.

### `Cannot open server ... Client with IP address ... is not allowed`

Det är Azure SQL-brandväggen. Din publika IP har ändrats, eller så använder du VPN. Lägg till aktuell klient-IP i portalen.

### `Login failed for user`

Det brukar bero på fel lösenord i `secrets.json`, eller på att JSON-strängen blev trasig vid inklistring. Öppna `Manage User Secrets` igen och kontrollera värdet.

### `A network-related or instance-specific error ... The server was not found`

Servernamnet är fel. Det ska vara `<server>.database.windows.net`.

### Första anropet tar 30–60 sekunder och timeoutar

Databasen är sannolikt serverless och har gått i vila. Vänta en stund, eller väck den med en `SELECT` i **SQL Server Object Explorer** först.

### `/api/todo` ger 404

`AddControllers()` saknas i `Program.cs`, eller så mappar du inte controllers med `app.MapControllers()`.

### Sidan visar poster men de försvinner vid omstart

Appen pekar mot en annan databas än du tror. Kontrollera att `secrets.json` verkligen pekar på databasen från Del 1 och inte på någon lokal instans.

### `InvalidOperationException: Unable to resolve service for type 'AppDbContext'`

`AddDbContext<AppDbContext>` saknas i `Program.cs`, eller ligger efter `builder.Build()`. Då hinner tjänsten aldrig registreras.