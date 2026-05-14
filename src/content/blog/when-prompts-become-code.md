---
title: "Vem äger agenterna?"
description: "Vem äger egentligen logiken när instruktionen är skriven på naturligt språk? Och är det ens ett bra mål att beställaren ska kunna bygga sina egna lösningar?"
pubDate: 2026-05-14
heroImage: "../../assets/images/blog/when-prompts-become-code/hero-when-prompts-become-code.jpg"
tags: ["ai", "reflektion", "utveckling"]
---

Nyligen deltog jag i en workshop där man byggde AI-agenter utan att skriva en enda rad traditionell kod. Det är en mäktig känsla att se en agent självständigt arbeta för att lösa en uppgift. Det går snabbt, det känns kraftfullt och det sänker tröskeln för vem som kan bygga.

Jag har förstås gjort det här tidigare, men nu sätter jag ord på mina funderingar. Vem äger egentligen logiken när instruktionen är skriven på naturligt språk? Det känns som att beställaren kan börja bygga sina egna lösningar – är det en bra riktning? Det finns något lockande i tanken, kortare väg från idé till verklighet, mindre beroende av utvecklare, lösningar byggda av de som bäst känner till sitt eget behov. Men det är också precis där ansvaret riskerar att bli diffust.

Som utvecklare med många rader kod bakom sig är det många frågor som snabbt väcks till liv.

## Från det exakta till det sannolika

Den stora omställningen handlar inte om verktyg, utan om tänkesätt. Vi går från ett system där resultatet alltid är helt förutbestämt av indata (deterministisk) till en verklighet där vi arbetar med sannolikheter och tolkningar.

I traditionell kod kan vi bryta ner ett problem, följa flödet och hitta exakt var något går fel. Det finns ett facit, även när det är komplext.

Med AI-agenter ser det annorlunda ut, och det är ibland helt avsiktligt. En AI-tjänst som skapar musik vore ganska tråkig om den producerade exakt samma låt varje gång – där är variation en feature, inte en bugg. Men i affärssystem, där vi förväntar oss förutsägbarhet, förändrar samma egenskap karaktären på felen. De blir mindre binära och mer situationsberoende.

Det kräver en annan typ av disciplin. Vi kan inte längre enbart fråga "fungerar det?" utan måste börja fråga "under vilka förutsättningar fungerar det – och när slutar det göra det?" Den första frågan leder till leverans. Den andra leder till förståelse.

## Risken med den dolda logiken

Vi pratar ofta varmt om eldsjälar, personer som driver innovation framåt. Men i en miljö där AI-agenter växer fram organiskt uppstår ett nytt fenomen, en slags agent-entropi, en gradvis ökad oordning i systemet.

Ju fler individuella lösningar som byggs utan gemensam struktur, desto svårare blir det att förstå helheten. Vad är det som säger att inte grannteamet bygger samma sak som vi, eller än värre en egen variant av det vi redan har? När affärslogik flyttar från kodbaser och dokumentation in i personliga prompter förlorar vi spårbarhet. Beslut som tidigare var explicita blir implicita. Resonemang som tidigare gick att granska blir istället inbäddade i formuleringar.

Det skapar en ny typ av beroende, inte till system utan till individer. Om en nyckelperson bygger en agent som blir central för en process och sedan lämnar organisationen, står vi kvar med något som fungerar men som vi inte riktigt förstår. En svart låda, inte för att tekniken är obegriplig, utan för att intentionen bakom den aldrig skrevs ner – eller skrevs ner olika av olika personer med olika preferenser.

Jag har hört invändningen att struktur lätt blir överadministration. Det förstår jag, men jag ser risken på andra hållet som större. Odokumenterade system, implicita beslut och saknad historik är svårare att städa upp än ett dokument för mycket, och i ett stort företag där alla bjuds in att utveckla egna agenter – hur håller man ens en gemensam röd linje? Jag tror inte att man gör det, inte utan någon form av gemensam struktur som är förstådd och förankrad.

Utmaningen är att hitta en rimlig brytpunkt mellan administration och arbetsinsats, tillräcklig ordning utan att det blir en börda i sig. Den frågan är inte ny, men jag tror den blir svårare när konfiguration ersätter kod och fler personer bjuds in att forma systemen. Den röda linjen som är svår att hålla idag kommer inte bli lättare att hålla när agenter används på bred front.

Det är teknisk skuld i en ny form.

## Att förvalta en digital medarbetare

Lösningen är inte att bromsa utvecklingen eller begränsa kreativiteten, men vi behöver börja se vad vi faktiskt bygger.

En AI-agent är inte en funktion. Det är ett beteende, och beteenden kräver förvaltning.

Att sätta en agent i produktion liknar mer att introducera en ny medarbetare än att driftsätta en tjänst. Den behöver tydliga mål, feedback, uppföljning och ibland korrigering. Den behöver också ett sammanhang, en förståelse för när den ska agera och när den inte ska det.

Det innebär att våra roller förändras. Utvecklarens roll blir mindre fokuserad på att skriva all logik och mer på att skapa stabila ramar med tydliga integrationer, säker datahantering och mekanismer för uppföljning. Det är infrastrukturen som möjliggör tillit.

Samtidigt kliver verksamheten närmare tekniken. Beställaren blir inte bara kravställare, utan också tränare och tolk – någon som iterativt formar agentens beteende och säkerställer att det speglar verkliga behov.

Det är i det samspelet mellan teknisk struktur och domänkunskap som hållbara lösningar växer fram. Åtminstone är det vad jag tror just nu.

## En livscykel för prompter

Gartner förutspår att en tredjedel av all företagsprogramvara kommer att innehålla agentisk AI redan 2028, upp från under en procent 2024. Det är en snabb resa, och frågan om hur vi förvaltar det vi bygger hinner lätt hamna i skymundan. WEF:s Future of Jobs Report 2025 pekar i samma riktning och listar mjukvaruutvecklare som fjärde snabbast växande yrke globalt – det är inte rollen som försvinner, det är en viss typ av arbete inom den som förändras.

En sak som hjälper är att börja behandla prompter som kod. Inte metaforiskt, utan bokstavligt. Versionering, granskning, testning, staging.

Det handlar om att skapa en struktur för hur prompter utvecklas och förvaltas, från experimentmiljö till produktion. Parametriserade mallar istället för ad hoc-text. Automatiserade tester för edge cases. Centraliserad lagring med metadata om syfte och förändringar.

Det låter tungt, men behöver inte vara det. Mycket liknar det vi redan gör med kod, och det kräver mest att vi börjar se prompter som artefakter värda att förvalta, inte som lösa anteckningar.

## Från byggande till ansvar

Att bygga en agent är idag relativt enkelt. Verktygen är tillgängliga, trösklarna är låga och resultaten kommer snabbt. Det verkliga skiftet ligger inte i hur lätt det är att bygga, utan i vad vi tar ansvar för när vi gör det.

När logiken inte längre är strikt definierad i kod, utan formulerad i språk, kan vi inte outsourca ansvaret till verktygen. Det måste bäras av organisationen.

Det är nog där den svåraste utmaningen finns. Inte att få agenter att fungera. Utan att bygga en organisation som förstår, förvaltar och litar på hur de fungerar – även när svaret inte är exakt detsamma varje gång.