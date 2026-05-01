---
title: "Guide: Så skapar du en modern blogg helt gratis"
description: "En steg-för-steg guide till hur jag satte upp min dev-blogg lokalt och publicerade den via GitHub och Azure Static Web Apps – inklusive editor, bilder och dagligt arbetsflöde."
pubDate: "Apr 30 2026"
heroImage: "/images/blog/astro/astro_hero.jpg"
---
# Från noll till live

Som utvecklare vill man ha en plats att dela sina tankar – utan krångel och höga kostnader. Jag ville bygga något snabbt, versionshanterat och helt gratis att driva. Valet föll på ramverket **Astro** i kombination med **Azure Static Web Apps**.

Det här är guiden jag önskat att jag haft från dag ett.

---

## Förutsättningar

Innan du börjar behöver du ha följande på plats:

- **Node.js** installerat lokalt på din dator
- Ett konto på **GitHub**
- Ett konto på **Microsoft Azure** (det kostnadsfria kontot räcker)

---

## Steg 1: Skapa projektet lokalt

Istället för färdiga webblösningar startade jag projektet direkt i terminalen – det ger full kontroll över koden från dag ett.

Öppna terminalen och kör:

```bash
# Initialize a new Astro project
npm create astro@latest
```

Välj den inbyggda **Blog**-mallen och låt guiden installera alla beroenden. När det är klart, starta den lokala servern:

```bash
# Start local development server
npm run dev
```

Öppna `http://localhost:4321` i webbläsaren. Det du ser där är exakt vad som kommer att publiceras.

---

## Steg 2: Versionshantering med GitHub

När du är nöjd med den första versionen är det dags att flytta koden till molnet.

1. Skapa ett nytt tomt arkiv (repository) på ditt GitHub-konto
2. Koppla din lokala mapp till GitHub och skicka upp koden:

```bash
git remote add origin https://github.com/<ditt_konto>/<blog_repository>
git add .
git commit -m "Initial commit - my new blog"
git push -u origin main
```

---

## Steg 3: Publicera bloggen

Astro bygger till vanliga statiska filer – HTML, CSS och JavaScript. Det innebär att du kan publicera bloggen på i princip vilken plattform som helst. Här är de två vanligaste vägarna.

### Alternativ A: Azure Static Web Apps (rekommenderas)

Det här är den väg jag valde. Du får en automatisk pipeline där varje push till GitHub triggar ett nytt bygge – utan att du behöver göra något manuellt.

1. Logga in i **Azure Portal** och sök efter *Static Web Apps*
2. Välj **Skapa** och anslut till ditt GitHub-konto
3. Leta upp ditt arkiv och välj grenen `main`
4. Under **Build Presets**, välj **Astro** – Azure skapar automatiskt en GitHub Actions-fil i din kod
5. Klicka **Skapa**. Inom ett par minuter bygger Azure din sida och ger dig en publik URL

> **CI/CD – push och glöm:** Från och med nu räcker det att pusha till GitHub. Azure märker varje push och bygger om sidan automatiskt. Inga manuella uppladdningar behövs.

**Eget domännamn:** Azure ger dig en URL som ser ut ungefär som `salmon-rock-123.azurestaticapps.net`. Om du vill använda ett eget domännamn gör du det under **Custom domains** i Azure-portalen – det är kostnadsfritt att koppla en befintlig domän.

### Alternativ B: Vanligt webbhotell

Har du redan ett webbhotell (Loopia, Hostinger, One.com eller liknande) kan du publicera bloggen dit istället. Kör bygget lokalt:

```bash
npm run build
```

Astro skapar en `dist/`-mapp med alla färdiga filer. Ladda upp innehållet i den mappen till ditt webbhotells `public_html/` (eller motsvarande) via FTP eller filhanteraren i kontrollpanelen.

> **Nackdelen** med det här alternativet är att du förlorar automatiken – varje gång du skriver ett nytt inlägg behöver du bygga och ladda upp manuellt. GitHub + Azure ger dig den processen helt gratis och automatisk.

---

## Steg 4: Skriva och publicera inlägg

Det dagliga arbetsflödet är enkelt: du skriver en Markdown-fil lokalt och commitar den till GitHub. Azure gör resten automatiskt.

### Rekommenderad editor: VS Code

[Visual Studio Code](https://code.visualstudio.com/) är det naturliga valet. Med några tillägg blir upplevelsen riktigt bra:


| Tillägg                | Vad det gör                                      |
| ------------------------- | --------------------------------------------------- |
| **Markdown All in One** | Förhandsvisning, genvägar och table of contents |
| **markdownlint**        | Flaggar formateringsfel i realtid                 |
| **Astro**               | Syntax-highlighting för`.astro`-filer            |

Öppna Markdown-förhandsvisningen med `Ctrl + Shift + V` (eller `Cmd + Shift + V` på Mac) för att se hur inlägget ser ut medan du skriver.

### Skapa ett nytt inlägg

Varje inlägg är en `.md`-fil i mappen `src/content/blog/`:

```
src/
└── content/
    └── blog/
        └── mitt-nya-inlagg.md
```

Börja alltid filen med **frontmatter** – metadata som Astro läser och använder för titel, beskrivning och hero-bild:

```markdown
---
title: "Rubriken på mitt inlägg"
description: "En kort beskrivning som syns i sökmotorer."
pubDate: "May 15 2026"
heroImage: "/images/blog/mitt-nya-inlagg/hero.jpg"
---

Skriv ditt innehåll här...
```

> ⚠️ **Framtids-filtret:** Om `pubDate` ligger framåt i tiden döljer Astro automatiskt inlägget i listan. Kontrollera alltid att datumet har passerat innan du förväntar dig att inlägget ska synas.

### Tekniska kodblock

Det är enkelt att skriva tekniska inlägg – ange bara språket direkt efter de tre backticksen:

````markdown
```csharp
// Din kod här
```
````

Astro renderar syntaxmarkering automatiskt för de flesta vanliga språk.

---

## Steg 5: Hantera bilder

Astro har **två platser** för bilder och de fungerar på helt olika sätt. Förstår du den skillnaden spar du dig mycket förvirring.

### Snabbreferens – välj rätt mapp direkt


| What                     | Folder                                                  | Markdown Reference                                             |
| -------------------------- | --------------------------------------------------------- | ---------------------------------------------------------------- |
| Hero image (frontmatter) | `public/images/blog/fran-noll-till-live/hero.jpg`       | `heroImage: "/images/blog/fran-noll-till-live/hero.jpg"`       |
| Image in post body       | `public/images/blog/fran-noll-till-live/screenshot.jpg` | `![alt text](/images/blog/fran-noll-till-live/screenshot.jpg)` |
| Logo, favicon            | `public/logo.svg`                                       | `src="/logo.svg"`                                              |
| Auto-optimized image     | `src/assets/images/about-me.jpg`                        | `import` + `<Image />` in `.astro`/`.mdx`                      |

### Rekommenderade bildformat och storlekar


| Image type              | Size            | Format           | Max file size |
| ------------------------- | ----------------- | ------------------ | --------------- |
| Hero                    | 1200 × 630 px  | `.webp` / `.jpg` | 200 KB        |
| Screenshot in post body | max 800 px wide | `.webp` / `.png` | 100 KB        |
| Logo                    | any             | `.svg`           | —            |

Komprimera bilderna med [Squoosh](https://squoosh.app/) – gratis, direkt i webbläsaren – innan du lägger dem i projektet.

För en blogg med vanliga `.md`-filer är svaret enkelt: **lägg alla bilder i `public/`**. Det är det du behöver i 95 % av fallen.

### De två mapparna förklarade

**`public/` – statiska filer som serveras direkt**

Allt i `public/` kopieras oförändrat till den färdiga webbplatsen. Astro rör inte filerna – ingen automatisk optimering, men enkel och förutsägbar hantering. Referera alltid med en sökväg som börjar med `/`.

Använd `public/` för:

- Logotyp och favicon
- Alla bilder i vanliga Markdown-filer (`.md`)
- Filer som måste ha en fast, förutsägbar URL

**`src/assets/` – bilder som Astro optimerar**

Bilder här behandlas av Astros bildpipeline: automatisk komprimering, konvertering till modernt format och rätt dimensioner. Nackdelen är att de bara kan användas via `import` i `.astro`- och `.mdx`-filer, inte i vanlig Markdown. Det är ett avancerat spår som du sällan behöver för en blogg.

### Rekommenderad mappstruktur

Skapa en undermapp per inlägg – det håller ordning när antalet inlägg växer.

```
project-root/
├── public/
│   ├── favicon.svg
│   ├── logo.svg
│   └── images/
│       └── blog/
│           ├── fran-noll-till-live/
│           │   ├── hero.jpg
│           │   └── screenshot.jpg
│           └── mitt-andra-inlagg/
│               └── hero.jpg
│
└── src/
    └── assets/
        └── images/
            └── about-me.jpg    ← används i .astro-komponenter
```

### Logotyp och favicon

```astro
<!-- src/components/Header.astro -->
<img src="/logo.svg" alt="Min blogg logotyp" width="120" height="40" />
```

```html
<!-- src/layouts/BaseLayout.astro – i <head> -->
<link rel="icon" type="image/svg+xml" href="/favicon.svg" />
```

### Optimerade bilder i `.astro`- och `.mdx`-filer

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

---

## Steg 6: Det dagliga arbetsflödet – från idé till live

Här är det kompletta flödet för att lägga upp ett nytt inlägg med bilder:

1. **Förbered och placera bilderna**

```
public/images/blog/ditt-inlagg/hero.jpg
public/images/blog/ditt-inlagg/screenshot.jpg
```

2. **Skapa inläggsfilen** i `src/content/blog/ditt-inlagg.md`

```markdown
---
title: "Mitt nya inlägg"
description: "Kort beskrivning."
pubDate: "May 15 2026"
heroImage: "/images/blog/ditt-inlagg/hero.jpg"
---

Introduktion...

![Beskrivande alt-text](/images/blog/ditt-inlagg/screenshot.jpg)

Mer text...
```

3. **Förhandsgranska lokalt**

```bash
npm run dev
```

Öppna `http://localhost:4321` och kontrollera att layout, bilder och att inlägget syns i listan.

4. **Commita och pusha**

```bash
git add src/content/blog/ditt-inlagg.md
git add public/images/blog/ditt-inlagg/
git commit -m "Add: inlägg om X"
git push
```

Azure Static Web Apps märker pushen och publicerar automatiskt. Inom ett par minuter är inlägget live.

> **Byggfel?** Gå till ditt GitHub-repo, klicka på fliken **Actions** och leta upp det senaste flödet. Där ser du exakt vad som gick fel och var i byggprocessen det hände.

---

## Steg 7: Anpassa utseendet

Mallen ger dig en funktionell blogg direkt, men du vill troligtvis sätta din egen prägel. Det finns två ställen att börja på:

**Globala stilar** – `src/styles/global.css`

Här styr du typsnitt, färger och grundlayout. Det är en vanlig CSS-fil – inga ramverksspecifika konstigheter.

**Komponenter** – mappen `src/components/`

Här hittar du `Header.astro`, `Footer.astro` och liknande byggblock. Redigera dem precis som du skulle redigera vanlig HTML, fast med `.astro`-syntax.

Ett bra första steg är att byta ut logotyp, namn och färger i `Header.astro` – det räcker långt för att sidan ska kännas personlig.

---

## Sammanfattning

Genom att börja lokalt och bygga en pipeline via GitHub till Azure har du en plattform som är professionell, versionshanterad och kostnadsfri att driva.

**Det dagliga arbetsflödet i korthet:**

1. Komprimera och placera bilder i `public/images/blog/inlagg-namn/`
2. Skriv ett nytt `.md`-inlägg i `src/content/blog/` i VS Code
3. Förhandsgranska lokalt med `npm run dev`
4. Commita och pusha till GitHub
5. Azure bygger och publicerar automatiskt ✓

**Vanliga gotchas att hålla koll på:**

- `pubDate` framåt i tid → inlägget syns inte i listan
- Bilder i Markdown → lägg dem alltid i `public/`, inte i `src/assets/`
- Byggfel → kolla fliken **Actions** i ditt GitHub-repo
