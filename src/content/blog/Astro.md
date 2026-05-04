---
title: "Guide: Så skapar du en modern blogg helt gratis"
description: "En steg-för-steg guide till hur jag satte upp min dev-blogg lokalt och publicerade den via GitHub och Azure Static Web Apps – inklusive editor, bilder och dagligt arbetsflöde."
pubDate: 2026-04-30
heroImage: "../../assets/images/blog/astro/astro_hero.jpg"
tags: ["astro", "azure", "webbutveckling"]
---
Jag önskade ett eget hörn på nätet. Inte nödvändigtvis för att nå en stor publik, utan för att ha någonstans att skriva ner det man lär sig – och kanske slippa googla samma sak två gånger. Tänker även att det handlar om att dela med sig till andra.

Jag ville ha något snabbt, versionshanterat och gratis att driva. Valet föll på **Astro** i kombination med **Azure Static Web Apps** med en automatisk pipeline via **GitHub** som sköter publiceringen

## Förutsättningar

Innan du kör igång behöver du ha följande på plats:

- **Node.js** installerat lokalt
- Ett konto på **GitHub**
- Ett konto på **Microsoft Azure** – det kostnadsfria kontot räcker gott

---

## Steg 1: Skapa projektet lokalt

Öppna terminalen och kör:

```bash
# Initialize a new Astro project
npm create astro@latest
```

Välj den inbyggda **Blog**-mallen och låt guiden installera beroendena. Sen startar du den lokala servern:

```bash
# Start local development server
npm run dev
```

Öppna `http://localhost:4321` i webbläsaren. Det du ser där är exakt det som kommer att publiceras – inga överraskningar.

---

## Steg 2: Versionshantering med GitHub

När du är nöjd med startläget är det dags att flytta upp koden till GitHub.

1. Skapa ett nytt tomt arkiv (repository) på ditt GitHub-konto
2. Koppla din lokala mapp och pusha:

```bash
git remote add origin https://github.com/<ditt_konto>/<blog_repository>
git add .
git commit -m "Initial commit - my new blog"
git push -u origin main
```

---

## Steg 3: Publicera bloggen

Astro bygger till vanliga statiska filer – HTML, CSS och JavaScript. Det innebär att du kan publicera var du vill. Här är de två vanligaste vägarna.

### Alternativ A: Azure Static Web Apps (rekommenderas)

Det här är vägen jag valde. Varje push till GitHub triggar ett automatiskt bygge – du behöver aldrig tänka på driftsättning igen.

1. Logga in i **Azure Portal** och sök efter *Static Web Apps*
2. Välj **Skapa** och anslut till ditt GitHub-konto
3. Välj ditt repo och grenen `main`
4. Under **Build Presets**, välj **Astro** – Azure skapar en GitHub Actions-fil i din kod automatiskt
5. Klicka **Skapa**. Om ett par minuter är sidan live med en publik URL

> **Push och glöm:** Från och med nu räcker det att pusha till GitHub. Azure märker varje push och bygger om sidan automatiskt.

**Eget domännamn:** Du får en URL i stil med `salmon-rock-123.azurestaticapps.net`. Vill du använda ett eget domännamn hittar du inställningen under **Custom domains** i Azure-portalen – det kostar ingenting extra att koppla en befintlig domän.

### Alternativ B: Vanligt webbhotell

Har du redan ett webbhotell – Loopia, Hostinger, One.com eller något annat – fungerar det lika bra. Bygg sidan lokalt:

```bash
npm run build
```

Astro skapar en `dist/`-mapp med allt som behövs. Ladda upp innehållet till din `public_html/` via FTP eller filhanteraren i kontrollpanelen.

> **Nackdelen** är att du förlorar automatiken. Varje nytt inlägg kräver ett manuellt bygge och uppladdning. Med GitHub + Azure sköts det åt dig, gratis.

---

## Steg 4: Skriva och publicera inlägg

Varje inlägg är en Markdown-fil. Du skriver lokalt, commitar till GitHub och Azure gör resten.

### Rekommenderad editor: VS Code

[Visual Studio Code](https://code.visualstudio.com/) är det självklara valet. Några tillägg gör skillnad:


| Tillägg                | Vad det gör                                      |
| ------------------------- | --------------------------------------------------- |
| **Markdown All in One** | Förhandsvisning, genvägar och table of contents |
| **markdownlint**        | Flaggar formateringsfel i realtid                 |
| **Astro**               | Syntax-highlighting för`.astro`-filer            |

Förhandsgranska Markdown-filen med `Ctrl + Shift + V` (eller `Cmd + Shift + V` på Mac).

### Skapa ett nytt inlägg

Varje inlägg är en `.md`-fil i `src/content/blog/`:

```
src/
└── content/
    └── blog/
        └── mitt-nya-inlagg.md
```

Börja alltid med **frontmatter** – det är metadata som Astro läser för att bygga sidan:

```markdown
---
title: "Rubriken på mitt inlägg"
description: "En kort beskrivning som syns i sökmotorer."
pubDate: "May 15 2026"
heroImage: "../../assets/images/blog/mitt-nya-inlagg/hero.jpg"
---

Skriv ditt innehåll här...
```

> ⚠️ **Framtids-filtret:** Om `pubDate` ligger framåt i tiden döljer Astro automatiskt inlägget i listan. Kontrollera att datumet har passerat om inlägget inte syns.

### Tekniska kodblock

Ange språket direkt efter de tre backticksen – Astro sköter syntaxmarkeringen automatiskt:

````markdown
```csharp
// Din kod här
```
````

---

## Steg 5: Hantera bilder

Det här är den del som brukar ställa till det. Astro har **två platser** för bilder och de fungerar på fundamentalt olika sätt. Väljer du fel mapp får du ett felmeddelande som inte är helt uppenbart att tolka.

### Snabbreferens


| What                     | Folder                                          | Reference                                                    |
| -------------------------- | ------------------------------------------------- | -------------------------------------------------------------- |
| Hero image (frontmatter) | `src/assets/images/blog/mitt-inlagg/hero.jpg`   | `heroImage: "../../assets/images/blog/mitt-inlagg/hero.jpg"` |
| Image in post body       | `public/images/blog/mitt-inlagg/screenshot.jpg` | `![alt text](/images/blog/mitt-inlagg/screenshot.jpg)`       |
| Logo, favicon            | `public/logo.svg`                               | `src="/logo.svg"`                                            |

Hero måste ligga i src/assets/ på grund av <Image />-komponenten. Brödtextbilder i public/ är ett aktivt val för enkelhetens skull – inte ett krav.

### Rekommenderade format och storlekar


| Image type              | Size            | Format           | Max file size |
| ------------------------- | ----------------- | ------------------ | --------------- |
| Hero                    | 1200 × 630 px  | `.webp` / `.jpg` | 200 KB        |
| Screenshot in post body | max 800 px wide | `.webp` / `.png` | 100 KB        |
| Logo                    | any             | `.svg`           | —            |

Komprimera bilderna med [Squoosh](https://squoosh.app/) innan du lägger dem i projektet – det är gratis och fungerar direkt i webbläsaren.

### De två mapparna – och varför de skiljer sig

**`src/assets/` – för hero-bilder**

Hero-bilden hanteras av Astros `<Image />`-komponent som kräver att bilden importeras av Astro vid bygget. Därför måste den ligga i `src/assets/`. Sökvägen i frontmatter är relativ från `.md`-filens plats:

```markdown
heroImage: "../../assets/images/blog/mitt-inlagg/hero.jpg"
```

**`public/` – för bilder i brödtexten**

Bilder du refererar direkt i Markdown-texten med `![alt](sökväg)` ska ligga i `public/`. Allt i `public/` kopieras oförändrat till den färdiga webbplatsen och refereras alltid med en sökväg som börjar med `/`:

```markdown
![En beskrivande alt-text](/images/blog/mitt-inlagg/screenshot.jpg)
```

### Rekommenderad mappstruktur

```
project-root/
├── public/
│   ├── favicon.svg
│   ├── logo.svg
│   └── images/
│       └── blog/
│           └── mitt-inlagg/
│               └── screenshot.jpg
│
└── src/
    └── assets/
        └── images/
            └── blog/
                └── mitt-inlagg/
                    └── hero.jpg
```

### Logotyp och favicon

```astro
<!-- src/components/Header.astro -->
<img src="/logo.svg" alt="Min blogg" width="120" height="40" />
```

```html
<!-- src/layouts/BaseLayout.astro – i <head> -->
<link rel="icon" type="image/svg+xml" href="/favicon.svg" />
```

---

## Steg 6: Det dagliga arbetsflödet – från idé till live

Så här ser ett normalt flöde ut:

1. **Förbered bilderna** – komprimera med Squoosh och placera dem rätt:

```
src/assets/images/blog/ditt-inlagg/hero.jpg
public/images/blog/ditt-inlagg/screenshot.jpg
```

2. **Skapa inläggsfilen** `src/content/blog/ditt-inlagg.md`:

```markdown
---
title: "Mitt nya inlägg"
description: "Kort beskrivning."
pubDate: "May 15 2026"
heroImage: "../../assets/images/blog/ditt-inlagg/hero.jpg"
---

Introduktion...

![Beskrivande alt-text](/images/blog/ditt-inlagg/screenshot.jpg)

Mer text...
```

3. **Förhandsgranska lokalt**

```bash
npm run dev
```

Öppna `http://localhost:4321` och kontrollera att hero-bilden visas, att bilder i brödtexten stämmer och att inlägget syns i listan.

4. **Commita och pusha**

```bash
git add src/content/blog/ditt-inlagg.md
git add src/assets/images/blog/ditt-inlagg/
git add public/images/blog/ditt-inlagg/
git commit -m "Add: inlägg om X"
git push
```

Azure märker pushen och publicerar automatiskt. Inom ett par minuter är inlägget live.

> **Byggfel?** Gå till ditt GitHub-repo och klicka på fliken **Actions**. Där ser du exakt vad som gick fel och var i processen det hände.

---

## Steg 7: Anpassa utseendet

Mallen ger dig en fungerande blogg direkt, men du vill säkert sätta din egen prägel. Det finns två naturliga ställen att börja:

**`src/styles/global.css`** – typsnitt, färger och grundlayout. Vanlig CSS, ingenting konstigt.

**`src/components/`** – här hittar du `Header.astro`, `Footer.astro` och övriga byggblock. Redigera dem som vanlig HTML, fast med `.astro`-syntax.

Ett bra första steg är att byta namn, logotyp och accentfärg i `Header.astro`. Det räcker långt för att sidan ska kännas som din.

---

## Sammanfattning

Det här upplägget tar lite tid att förstå från start, men när det väl sitter är det en riktigt smidig arbetsyta. Du skriver Markdown, pushar till GitHub och Azure sköter resten.

**Arbetsflödet i korthet:**

1. Hero-bilder → `src/assets/images/blog/inlagg-namn/`
2. Bilder i brödtext → `public/images/blog/inlagg-namn/`
3. Skriv inlägget i `src/content/blog/` i VS Code
4. Förhandsgranska med `npm run dev`
5. Pusha till GitHub – Azure publicerar automatiskt ✓

**Vanliga misstag att undvika:**

- Hero-bilden i `public/` istället för `src/assets/` → `ImageNotFound`-fel
- `pubDate` framåt i tid → inlägget syns inte i listan
- Byggfel → kolla fliken **Actions** i ditt GitHub-repo
