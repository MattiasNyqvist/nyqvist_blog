---
title: "Projects-Skills and RAGs in Claude"
description: "Några begrepp dyker upp så fort man börjar använda Claude på allvar. Här är vad projekt, skills och RAG betyder, hur de skiljer sig och var de hör hemma."
pubDate: 2026-05-17
heroImage: "../../assets/images/blog/Projects-Skills-and-RAGs-in-Claude/hero-projects-skills-and-rags-in-claude.jpg"
tags: ["ai","claude","utveckling"]
---

# Projekt #

Ett projekt är en egen arbetsyta med egen historik och en egen kunskapsbas. Du laddar upp dokument, kod eller annan kontext och kan skriva projektinstruktioner som styr tonen eller låter Claude svara utifrån en viss roll. Allt du lägger in finns kvar mellan chattarna, så du slipper förklara samma sak om och om igen.

På ett gratiskonto kan du ha upp till fem projekt, medan betalplanerna i praktiken är obegränsade. Slutar du betala och hamnar på gratisvarianten försvinner inte innehållet automatiskt, men du landar på taket fem. Exakt vad som händer med projekten du har utöver det skriver Anthropic inte ut tydligt, så har du många kan det vara värt att kontrollera läget innan du säger upp abonnemanget.

I kunskapsbasen får varje fil vara upp till 30 MB. Du kan lägga in många filer i format som PDF, Word, CSV och vanlig text. Du kan också klistra in text direkt, vilket är vägen om du vill lägga in innehållet från en hemsida. Någon synkad länk till en webbsida eller en YouTube-video finns inte, utan då får du kopiera in texten eller transkriptet själv.

Ett projekt kan dessutom efterlikna hur du skriver. Lägger du in en stilguide eller några av dina egna texter, plus en instruktion om tonen, börjar svaren låta som dig. Samma sak med kod, beskriv dina konventioner så följer Claude dem. Det blir en approximation, inte en perfekt kopia, men ofta nära nog.

# RAG (Retrieval Augmented Generation) #

När kunskapsbasen växer sig stor händer något i bakgrunden. I stället för att läsa hela innehållet på en gång slår projektet om till RAG. Förenklat innebär det att Claude söker fram just de delar av din kunskapsbas som är relevanta för frågan och använder dem när svaret skapas. Det aktiveras automatiskt och utökar kapaciteten upp till tio gånger.

Anthropic säger att kvaliteten hålls konstant. I regel stämmer det. Det finns ändå en praktisk baksida att känna till. Eftersom Claude bara plockar fram det som söksteget bedömer som relevant kan något missas, särskilt om en uppgift är utspridd över flera filer. Det märks som att ett svar saknar en detalj du vet finns i underlaget. Motmedlet är enkelt. Ge filerna tydliga namn, samla ihop sådant som hör samman och hänvisa till ett dokument vid namn när du frågar om något specifikt.

# Skill #

En skill är inte en plats utan ett arbetssätt. Det är en samling instruktioner som Claude laddar in just när de behövs för en viss uppgift och lägger undan igen efteråt. En del är inbyggda, till exempel de som skapar Word-, Excel- och PDF-filer, som aktiveras automatiskt. Andra bygger du själv. Då räcker det ofta med en textfil i Markdown som beskriver hur uppgiften ska lösas.

markdown

```markdown
# Commit-meddelande #
Skriv i imperativ och håll första raden kort.
Förklara varför ändringen gjordes, inte hur.
En rad räcker oftast.
```

När en sådan skill väl finns på plats räcker det att be om ett commit-meddelande för att rätt arbetssätt ska slå igång.

Det är här tipsen från sociala medier kommer in. Du har säkert sett inlägg som säger åt dig att börja prompten med ett snedstreck följt av ett namn, typ /humen, för att få Claude att bete sig på ett visst sätt. Det är just en skill som triggas igång. I Claude Code och i verktyg som följer den öppna standarden kan du starta en skill direkt på det viset. I den vanliga chatten på claude.ai slår du i stället på skillen under Anpassa och låter Claude använda den när den passar. Samma sak under huven, olika sätt att starta den.

Du behöver inte börja från noll. Under Anpassa finns en katalog med färdiga skills, bland annat från Notion, Figma och Atlassian. Dina egna dyker upp i samma lista. Eftersom de bygger på en öppen standard kan du dela och återanvända dem, så det blir mer än en textsnutt sparad åt dig själv.

# Hur de hänger ihop #

Det enklaste sättet att hålla isär projekt och skills är att tänka så här. Ett projekt handlar om vad Claude vet, en skill om hur Claude gör.

Ett par begrepp till hör till samma familj. Anpassade instruktioner gäller brett över alla dina chattar oavsett projekt och passar för tonläge och generella önskemål. MCP-kopplingar är något annat. De ger Claude tillgång till externa tjänster och verktyg, medan en skill beskriver hur verktygen ska användas. De krockar inte, utan löser olika delar av samma problem.

Det mesta av det här kräver ingen kod. Det räcker att beskriva vad man vill ha och hur det ska göras, i vanlig text. Var gränsen mellan att instruera och att programmera går blir nog mindre och mindre tydlig framöver.
