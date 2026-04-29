---
title: "Guide: Så skapar du en snabbt en blogg helt gratis"
description: "En steg-för-steg guide till hur jag satte upp min dev-blogg lokalt och publicerade den via GitHub och Azure Static Web Apps – inklusive editor, bilder och dagligt arbetsflöde."
pubDate: "Apr 30 2026"
heroImage: "/images/blog/fran-noll-till-live/hero.jpg"
---

# Från noll till live: Min resa till en modern dev-blogg

Många utvecklare vill ha en plats att dela sina tankar utan krångel eller höga kostnader. Jag ville bygga något som var snabbt, versionshanterat och helt gratis att driva. Valet föll på ramverket **Astro** i kombination med molnkraft från **Azure**.

Här är den exakta processen jag använde för att gå från en tom mapp på datorn till en publik webbplats – inklusive hur jag skriver nya inlägg, hanterar bilder och publicerar med ett enda kommando.

---

## Förutsättningar (Prerequisites)

Innan du börjar behöver du ha följande på plats:

1. **Node.js** installerat lokalt på din dator.
2. Ett konto på **GitHub**.
3. Ett konto på **Microsoft Azure** (det kostnadsfria kontot räcker gott).

---

## Steg 1: Skapa projektet lokalt

Istället för att använda färdiga webblösningar startade jag projektet direkt i terminalen. Detta ger full kontroll över koden från dag ett.

Öppna terminalen och kör:

```bash
# Initialize a new Astro project
npm create astro@latest
```

Jag valde den inbyggda **Blog**-mallen och lät guiden installera alla beroenden (dependencies).

För att se resultatet medan jag jobbade startade jag den lokala servern:

```bash
# Start local development server
npm run dev
```

Öppna sedan `http://localhost:4321` i webbläsaren – det du ser där är exakt vad som kommer att publiceras.

---

## Steg 2: Versionshantering med GitHub

När jag var nöjd med den första versionen lokalt behövde koden flyttas till molnet för att kunna publiceras.

1. Skapa ett nytt tomt arkiv (repository) på ditt GitHub-konto.
2. Koppla din lokala mapp till GitHub och skicka upp koden:

```bash
git remote add origin https://github.com/DITT-ANVÄNDARNAMN/nyqvist_blog.git
git add .
git commit -m "Initial commit - my new blog"
git push -u origin main
```

---

## Steg 3: Driftsättning med Azure Static Web Apps

Nu kommer det magiska steget där koden blir en riktig hemsida.

1. Logga in i **Azure Portal** och sök efter *Static Web Apps*.
2. Välj **Skapa** och anslut till ditt GitHub-konto.
3. Leta upp ditt arkiv och välj din gren (`main`).
4. Under **Build Presets**, välj **Astro**. Azure skapar automatiskt en GitHub Action (en YAML-fil i din kod).
5. Klicka på **Skapa**. Inom ett par minuter bygger Azure din sida och ger dig en publik URL.

> **CI/CD – push och glöm:** Från och med nu räcker det att pusha till GitHub. Azure märker varje push och bygger om sidan automatiskt. Inga manuella uppladdningar eller FTP-klienter behövs.

---

## Steg 4: Skriva nya inlägg lokalt

Det dagliga arbetsflödet är enkelt – du skriver en Markdown-fil lokalt och commitar den till GitHub. Azure gör resten automatiskt.

### Rekommenderad editor: VS Code

[Visual Studio Code](https://code.visualstudio.com/) är det naturliga valet för att skriva Markdown lokalt. Med några tillägg blir upplevelsen riktigt bra:

| Tillägg | Vad det gör |
|---|---|
| **Markdown All in One** | Förhandsvisning, genvägar och table of contents |
| **markdownlint** | Flaggar formateringsfel i realtid |
| **Astro** | Syntax-highlighting för `.astro`-filer |

Öppna Markdown-förhandsvisningen med `Ctrl + Shift + V` (eller `Cmd + Shift + V` på Mac) för att se hur inlägget ser ut medan du skriver.

### Så här skapar du ett nytt inlägg

Skapa en ny `.md`-fil i mappen `src/content/blog/`:

```
src/
└── content/
    └── blog/
        └── mitt-nya-inlagg.md
```

Börja alltid filen med **frontmatter** – metadata som Astro läser:

```markdown
---
title: "Rubriken på mitt inlägg"
description: "En kort beskrivning som syns i sökmotorer."
pubDate: "May 15 2026"
heroImage: "/images/blog/mitt-nya-inlagg/hero.jpg"
---

Skriv ditt innehåll här...
```

> **Framtids-filtret:** Om `pubDate` ligger framåt i tiden döljer Astro automatiskt inlägget i listan. Se till att datumet har passerat för att det ska synas.

---

## Steg 5: Hantera bilder – ordning och reda

Bilder i Astro följer en tydlig logik. Det viktigaste att förstå är att Astro har **två platser** där bilder kan ligga – och de fungerar på helt olika sätt.

### De två mapparna

**`public/` – statiska filer som serveras direkt**

Allt som läggs i `public/` kopieras oförändrat till den färdiga webbplatsen. Astro rör inte filerna – ingen automatisk optimering.

Använd `public/` för:
- Logotyp och favicon
- Bilder som refereras i vanliga Markdown-filer (`.md`)
- Filer som måste ha en **förutsägbar, fast URL**

**`src/assets/` – bilder som Astro optimerar**

Bilder under `src/assets/` behandlas av Astros bildpipeline. Astro komprimerar automatiskt, konverterar till modernt format och genererar rätt dimensioner. Men de kan bara användas via `import` i `.astro`- och `.mdx`-filer – inte i vanlig Markdown.

### Rekommenderad mappstruktur

```
project-root/
├── public/
│   ├── favicon.svg
│   ├── logo.svg
│   └── images/
│       └── blog/
│           ├── fran-noll-till-live/
│           │   ├── hero.jpg
│           │   └── azure-portal.png
│           └── mitt-andra-inlagg/
│               └── hero.jpg
│
└── src/
    └── assets/
        └── images/
            └── about-me.jpg        ← används i .astro-komponenter
```

### Logotyp och favicon

Logotypen används i din layout-komponent och behöver en stabil URL – `public/` är rätt plats.

```
public/logo.svg
public/favicon.svg
```

Referera i din `.astro`-layout:

```astro
<!-- src/components/Header.astro -->
<img src="/logo.svg" alt="Mattias Dev Blog logotyp" width="120" height="40" />
```

Favicon i `<head>`:

```html
<link rel="icon" type="image/svg+xml" href="/favicon.svg" />
```

### Bilder i blogginlägg (vanlig Markdown `.md`)

Skapa en undermapp per inlägg under `public/images/blog/` – det håller ordning när antalet inlägg växer.

```
public/images/blog/fran-noll-till-live/azure-portal.png
```

Referera i Markdown (sökvägen börjar alltid med `/`):

```markdown
![Azure Portal – skapa Static Web App](/images/blog/fran-noll-till-live/azure-portal.png)
```

Hero-bilden sätts i frontmatter och läses av din layout:

```markdown
---
heroImage: "/images/blog/fran-noll-till-live/hero.jpg"
---
```

### Bilder med automatisk optimering (`.astro` / `.mdx`)

Vill du ha Astros inbyggda optimering använder du komponenten `<Image />`:

```astro
---
import { Image } from 'astro:assets';
import aboutMe from '../assets/images/about-me.jpg';
---

<Image
  src={aboutMe}
  alt="Mattias Nyqvist – fullstack .NET-utvecklare"
  width={800}
  height={600}
/>
```

### Snabbreferens – bilder

| Vad | Placering | Referens |
|---|---|---|
| Logotyp, favicon | `public/logo.svg` | `src="/logo.svg"` |
| Bild i Markdown | `public/images/blog/inlagg/bild.jpg` | `![alt](/images/blog/inlagg/bild.jpg)` |
| Hero-bild (frontmatter) | `public/images/blog/inlagg/hero.jpg` | `heroImage: "/images/blog/inlagg/hero.jpg"` |
| Bild med optimering | `src/assets/images/bild.jpg` | `import` + `<Image />` i `.astro`/`.mdx` |

---

## Steg 6: Det dagliga arbetsflödet – från idé till live

Så här ser ett normalt flöde ut när du lägger till ett nytt inlägg med bilder:

**1. Förbered bilderna**

Komprimera med [Squoosh](https://squoosh.app/) (gratis, direkt i webbläsaren) och spara som `.webp` eller `.jpg`. Håll filstorleken under 200 KB för hero-bilder och under 100 KB för bilder i brödtexten.

**2. Placera bilderna**

```
public/images/blog/ditt-inlagg/hero.jpg
public/images/blog/ditt-inlagg/screenshot.png
```

**3. Skapa inläggsfilen**

```
src/content/blog/ditt-inlagg.md
```

**4. Skriv inlägget i VS Code**

```markdown
---
title: "Mitt nya inlägg"
description: "Kort beskrivning."
pubDate: "May 15 2026"
heroImage: "/images/blog/ditt-inlagg/hero.jpg"
---

Introduktion...

![Beskrivande alt-text](/images/blog/ditt-inlagg/screenshot.png)

Mer text...
```

**5. Förhandsgranska lokalt**

```bash
npm run dev
```

Öppna `http://localhost:4321` och kontrollera att allt ser bra ut.

**6. Commita och pusha**

```bash
git add src/content/blog/ditt-inlagg.md
git add public/images/blog/ditt-inlagg/
git commit -m "Add: inlägg om X"
git push
```

Azure Static Web Apps märker pushen och publicerar automatiskt. Inom ett par minuter är inlägget live.

---

## Markdown och kod

Det bästa med det här upplägget är hur enkelt det är att skriva tekniska inlägg. Du anger språket direkt efter de tre backticksen:

````markdown
```csharp
// Din kod här
```
````

Så här ser ett C#-exempel ut i praktiken:

```csharp
public class BlogEngine
{
    public string Name { get; set; } = "Mattias Dev Blog";
    public bool IsLive { get; set; } = true;

    public void Status()
    {
        // Enkel loggning till konsolen
        Console.WriteLine($"{Name} is now up and running!");
    }
}
```

---

## Sammanfattning

Genom att börja lokalt och bygga en brygga via GitHub till Azure har jag skapat en plattform som är både professionell och kostnadseffektiv. Det ger en enorm frihet att kunna styra varje pixel och rad kod direkt från min egen editor.

**Det dagliga arbetsflödet i korthet:**

1. Skriv ett nytt `.md`-inlägg i VS Code
2. Komprimera och placera bilder i `public/images/blog/inlagg-namn/`
3. Förhandsgranska lokalt med `npm run dev`
4. Commita och pusha till GitHub
5. Azure bygger och publicerar automatiskt ✓
