---
title: "Git via CLI – en praktisk guide från setup till rollback"
description: "En genomgång av hur du arbetar med Git i terminalen. Från att sätta upp din identitet till att hantera branches, pusha till GitHub och rulla tillbaka misstag."
pubDate: 2026-05-03
heroImage: "../../assets/images/blog/git/hero_git.jpg"
tags: ["git", "cli", "version-control", "tutorial"]
---

Git är ett av de viktigaste verktygen i en utvecklares vardag, men det kan kännas överväldigande i början. Den här guiden tar dig igenom allt du behöver för att arbeta med Git via terminalen – från allra första steget till ett väloljat arbetsflöde med branches och GitHub.

Vi utgår från att du redan har Git installerat. Kör `git --version` för att bekräfta det.

---

## Sätt upp din identitet

Innan du gör din första commit behöver Git veta vem du är. Det här konfigureras globalt, vilket innebär att det gäller för alla dina repos på datorn och lagras i filen `~/.gitconfig`.

```bash
git config --global user.name "Mattias Nyqvist"
git config --global user.email "din@email.com"
```

Det är viktigt att e-postadressen matchar den du använder på GitHub – annars kopplas inte dina commits till ditt konto korrekt.

Det finns också ett par inställningar som är bra att göra direkt:

```bash
# Sätt VS Code som standardeditor (valfritt)
git config --global core.editor code

# Namnge default-branch "main" istället för det äldre "master"
git config --global init.defaultBranch main
```

### Byta e-postadress

Har du bytt jobb-e-post, eller vill du använda olika adresser för olika projekt? Inga problem.

Uppdatera globalt (gäller alla nya commits framöver):

```bash
git config --global user.email "ny@email.com"
```

Vill du använda en annan adress för ett specifikt repo – till exempel separera privata projekt från jobbet – kan du åsidosätta den globala inställningen lokalt. Kör detta inuti repo-mappen:

```bash
git config --local user.email "jobb@foretaget.se"
```

Den lokala inställningen lagras i `.git/config` i just det repot och slår alltid över den globala.

Kontrollera vad som faktiskt är aktivt med:

```bash
git config --list
git config user.email   # kolla bara e-posten
```

---

## Skapa ett nytt repo

Navigera till din projektmapp i terminalen och kör:

```bash
git init
```

Det skapar en dold `.git/`-mapp som är hjärtat i ditt repo – all historik och konfiguration lagras där. Rör den inte manuellt.

Vill du skapa en ny mapp och initiera repot i ett enda steg:

```bash
git init mitt-projekt
```

---

## .gitignore – berätta vad Git ska ignorera

Inte allt ska versionshanteras. Build-output, hemligheter och IDE-specifika filer bör aldrig commitas. Det löser du med en `.gitignore`-fil i roten av ditt repo.

```bash
# .gitignore
bin/
obj/
*.user
.env
.DS_Store
node_modules/
```

Varje rad är ett mönster. Mappar avslutas med `/`, wildcard-mönster använder `*`. Filen committas och följer med repot, så alla i teamet ignorerar samma saker.

**Viktigt:** `.env`-filer innehåller ofta API-nycklar och lösenord. Lägg alltid till `.env` i `.gitignore` redan från start – det är svårt att städa upp om du råkar pusha det till GitHub.

> **Tips:** GitHub kan generera en `.gitignore` åt dig när du skapar ett nytt repo via webben. Välj template för ditt språk, t.ex. *VisualStudio* för .NET-projekt.

---

## LICENSE – välj en licens för ditt projekt

Om du publicerar kod på GitHub bör du inkludera en licensfil i roten. Utan licens råder upphovsrätt som standard, vilket innebär att andra tekniskt sett inte får använda din kod.

MIT-licensen är det vanligaste valet för open source-projekt – den tillåter fri användning med minimal restriktion. GitHub låter dig lägga till en licensfil direkt när du skapar repot via webben, vilket är det enklaste sättet.

---

## Staga och committa

Det här är kärnan i Git-arbetsflödet, och det begrepp som kan vara förvirrande i början: **staging-zonen** (kallas också "index").

Tänk så här: dina filer har tre tillstånd.

1. **Unstaged** – du har gjort ändringar men inte berättat för Git om dem
2. **Staged** – du har valt ut exakt vilka ändringar som ska med i nästa commit
3. **Committed** – ändringarna är sparade i historiken

Staging-zonen är med andra ord ett mellansteg som ger dig kontroll. Du kan t.ex. ha ändrat tre filer men bara staga och committa en av dem.

### Kolla vad som är ändrat

```bash
git status
```

Det här är det kommando du kommer köra mest. Det visar vilka filer som är ändrade, stagade eller ej trackade av Git.

```bash
git diff            # ändringar som inte stagats än
git diff --staged   # vad som är stagat och redo att committas
```

### Staga filer

```bash
git add Program.cs          # staga en specifik fil
git add src/                # staga hela en mapp
git add .                   # staga allt i nuvarande mapp och undermappar
```

Vill du ha finare kontroll – t.ex. bara staga vissa rader i en fil – använd det interaktiva läget:

```bash
git add -p
```

Git visar dig varje "hunk" (förändring) och frågar om du vill inkludera den. Praktiskt när du vill dela upp arbete i logiska commits.

Ångrar du att du stagade något?

```bash
git reset HEAD Program.cs   # ta bort från staging, ändringarna finns kvar
```

### Committa

En commit är en sparad ögonblicksbild av dina stagade ändringar. Varje commit får ett unikt hash-ID och ett meddelande du skriver.

```bash
git commit -m "Add login endpoint"
```

Håll commit-meddelanden korta och beskrivande. Skriv vad du gjort, inte hur. "Add login endpoint" är bra. "Fixed stuff" är inte bra.

Har du glömt att staga och vill göra allt i ett steg (fungerar bara på filer som redan trackas av Git):

```bash
git commit -am "Fix null check in UserService"
```

Råkade du skriva fel i commit-meddelandet och har inte pushad än?

```bash
git commit --amend -m "Rätt meddelande här"
```

---

## Koppla till GitHub

Ditt lokala repo och GitHub är separata saker. Du kopplar ihop dem med en **remote** – en pekare till ett repo på en server.

Skapa först ett tomt repo på GitHub (utan README, .gitignore eller licens om du redan har dem lokalt), och kör sedan:

```bash
git remote add origin https://github.com/MattiasNyqvist/repo-namn.git
```

`origin` är det konventionella standardnamnet för din remote. Kolla att det är rätt:

```bash
git remote -v
```

Behöver du byta URL – t.ex. om du bytt repo-namn:

```bash
git remote set-url origin https://github.com/MattiasNyqvist/nytt-namn.git
```

### Klona ett befintligt repo

Om repot redan finns på GitHub och du vill hämta hem det:

```bash
git clone https://github.com/MattiasNyqvist/repo-namn.git
```

Remote sätts automatiskt till `origin`. Vill du ha ett annat mappnamn lokalt:

```bash
git clone https://github.com/MattiasNyqvist/repo-namn.git mitt-mappnamn
```

### Autentisering

GitHub stödjer inte längre lösenord via HTTPS. Det enklaste sättet att autentisera är via **GitHub CLI**:

```bash
gh auth login
```

Följ guiden i terminalen. Alternativt skapar du ett Personal Access Token på GitHub och använder det som lösenord.

---

## Pusha och hämta ändringar

### Pusha till GitHub

Första gången du pushar en branch behöver du ange vart:

```bash
git push -u origin main
```

Flaggan `-u` sätter en "upstream" – en koppling mellan din lokala branch och remote-branchen. Därefter räcker det med:

```bash
git push
```

### Hämta hem ändringar

Om någon annan (eller du själv från en annan dator) har pushad ändringar:

```bash
git pull
```

Det är egentligen en kombination av två kommandon: `git fetch` (hämtar ändringar) + `git merge` (slår ihop dem med din lokala branch).

Vill du bara kolla vad som finns på GitHub utan att merga in det:

```bash
git fetch origin
```

---

## Branches – jobba parallellt utan kaos

En branch är en separat linje av commits. Du skapar en branch för en feature eller buggfix, jobbar där utan att påverka `main`, och mergar tillbaka när du är klar.

**Tumregel:** committa aldrig direkt på `main`. Skapa alltid en branch för ditt arbete.

### Skapa och byta branch

```bash
git checkout -b feature/user-login
```

Det skapar en ny branch och byter till den direkt. Det modernare sättet (Git 2.23+):

```bash
git switch -c feature/user-login
```

Kolla var du befinner dig:

```bash
git branch        # listar lokala branches (* = aktiv)
git branch -a     # listar lokala + remote branches
```

### Byt branch och gå tillbaka till main

```bash
git switch main
```

Om du har oavslutade ändringar som inte är committade kommer Git protestera. Antingen committar du dem, eller så parkerafr du dem temporärt med `git stash`:

```bash
git stash         # lägg undan ändringarna
git switch main   # byt branch
git stash pop     # återställ ändringarna när du är tillbaka
```

Hoppa tillbaka till föregående branch (som `cd -` i terminalen):

```bash
git checkout -
```

### Merga en branch

Stå på `main` och dra in din feature-branch:

```bash
git switch main
git merge feature/user-login
```

Vill du ha en tydlig merge-commit i historiken (rekommenderas):

```bash
git merge --no-ff feature/user-login
```

### Städa upp branches

När en branch är mergad är det bra att ta bort den:

```bash
git branch -d feature/user-login          # radera lokalt
git push origin --delete feature/user-login  # radera på GitHub
```

---

## Ångra och rulla tillbaka

Det händer. Du committar fel sak, pushar för tidigt, eller inser att det du byggde var en återvändsgränd. Git har verktyg för allt det här.

### Ångra icke-committade ändringar

Återställ en fils ändringar till hur den såg ut vid senaste commit:

```bash
git restore Program.cs   # varning: ändringarna försvinner permanent
git restore .            # återställ allt
```

Ta bort en fil från staging utan att tappa ändringarna:

```bash
git restore --staged Program.cs
```

### Ångra commits lokalt – reset

`git reset` flyttar HEAD (pekaren som säger var du är i historiken) bakåt. Det finns tre varianter beroende på vad du vill göra med ändringarna:

```bash
git reset --soft HEAD~1    # ångra commit, ändringarna finns kvar stagade
git reset --mixed HEAD~1   # ångra commit, ändringarna finns kvar unstaged
git reset --hard HEAD~1    # ångra commit + radera ändringarna (farlig!)
```

`HEAD~1` betyder "ett steg bakåt". Vill du gå till en specifik commit använder du dess hash:

```bash
git reset --hard abc1234
```

Hitta hashen med `git log --oneline`.

> **Viktigt:** använd aldrig `git reset` på commits som redan pushats till GitHub. Det skriver om historiken och skapar problem för alla som har samma repo.

### Ångra pushad kod – revert

Om du behöver ångra något som redan pushats är `git revert` rätt verktyg. Det skapar en **ny commit** som vänder på ändringarna – historiken bevaras intakt.

```bash
git revert HEAD         # ångra senaste commit
git revert abc1234      # ångra en specifik commit
```

---

## Inspektera historiken

```bash
git log                              # fullständig historik
git log --oneline                    # en rad per commit
git log --oneline --graph --all      # visuell trädvy av alla branches
git log --author="Mattias"           # filtrera på författare
```

Visa innehållet i en specifik commit:

```bash
git show abc1234
```

Vem ändrade en viss rad i en fil?

```bash
git blame Program.cs
```

---

## Ett typiskt arbetsflöde

Så här kan en vanlig arbetsdag se ut:

```bash
# 1. Hämta senaste från GitHub
git pull

# 2. Skapa en branch för det du ska göra
git switch -c feature/rapport-export

# 3. Jobba, jobba, jobba...
# 4. Kolla vad du ändrat
git status
git diff

# 5. Staga och committa
git add .
git commit -m "Add PDF export for monthly reports"

# 6. Pusha branchen till GitHub
git push -u origin feature/rapport-export

# 7. Skapa en Pull Request på GitHub, få code review, merga

# 8. Tillbaka till main och städa upp
git switch main
git pull
git branch -d feature/rapport-export
```

---

Det tar ett tag att bygga muskelminne för Git-kommandon, men arbetsflödet ovan täcker 90% av det du stöter på i vardagen. Resten hittar du när du behöver det.