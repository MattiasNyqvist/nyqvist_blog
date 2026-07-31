---
title: "Google Analytics setup"
description: "This is my post about ..."
pubDate: 2026-06-28
heroImage: "../../assets/images/blog/google-analytics/hero-google-analytics.jpg"
tags: ["Google", "HowTo"]
---
# Så integrerar du Google Analytics 4 (GA4) på din webbplats

Innan vi dyker ner i själva tekniken kan det vara bra att svara på den grundläggande frågan: **Varför ska man egentligen ha Google Analytics?**

## Varför behöver man Google Analytics och vad kostar det?

När du driver en webbplats – oavsett om det är en personlig blogg, en portföljsajt eller en företagsplattform – är det svårt att förbättra upplevelsen utan att veta hur besökarna interagerar med den.

* **Det täcker behovet av insikt:** Du får svar på frågor som hur många som hittar till din sajt, vilka sidor som är mest populära, hur länge stannar besökarna kvar och varifrån trafiken kommer (till exempel via sökmotorer, direktlänkar eller sociala medier).
* **Datadriven utveckling:** Istället för att gissa vad som fungerar kan du se vilka sidor som engagerar och vilka som tappar besökare, vilket gör det enklare att prioritera vad som behöver förbättras.
* **Vad kostar det?** Standardversionen av Google Analytics (GA4) är helt kostnadsfri för de flesta webbplatser. Det krävs inga abonnemangskostnader för att komma igång och samla in standarddata.

## 1. Skapa egendom och hämta ditt Mätnings-ID

Allt börjar på Google Analytics-kontot:

* Logga in på ditt Google Analytics-konto och skapa en ny **Egendom** (Property) för din webbplats.
* Skapa därefter ett **Webbdataflöde** för din domän.
* När flödet är skapat får du ett unikt **Mätnings-ID** som alltid börjar med bokstäverna `G-` (till exempel `G-D4HEFBCNSV`). Det här ID:t är nyckeln som kopplar ihop din webbplats med statistiken.

## 2. Integrera spårningskoden i webbplatsens huvud

Google Analytics fungerar genom att ladda ett litet JavaScript-skript på varje sida som besöks. För att detta ska ske automatiskt på hela webbplatsen placerar du koden inuti `<head>`-taggen på din webbplats gemensamma sidhuvud eller mall.

Klistra in följande kodblock och byt ut `DIN-MATNINGS-ID` mot ditt faktiska `G-`-ID på de två ställen där det förekommer:

```html
<!-- Google tag (gtag.js) -->
<script async src="https://www.googletagmanager.com/gtag/js?id=DIN-MATNINGS-ID"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());

  gtag('config', 'DIN-MATNINGS-ID');
</script>
```

*Tips:* Det enklaste och mest robusta tillvägagångssättet är att skriva in ditt faktiska `G-`-ID direkt inom citattecken i koden, snarare än att krångla med externa miljövariabler som kan skilja sig åt beroende på miljö.

## 3. Verifiera att mätningen fungerar i realtid

När koden är sparad och publicerad vill du såklart säkerställa att trafiken registreras korrekt:

* Öppna din webbplats i ett nytt webbläsarfönster (gärna ett inkognitofönster för att slippa störningar från lokala adblockers eller tillägg).
* Klicka dig runt på några olika sidor för att generera aktivitet.
* Logga in på ditt Google Analytics-konto, navigera till **Rapporter** och öppna vyn **Översikt i realtid**.
* Inom någon minut bör du se att en aktiv användare dyker upp på kartan och att dina sidvisningar registreras i listorna.

## 4. Klart för lansering

När verifieringen visar grön lyskraft är mätningen fullt aktiv. Datan samlas nu in automatiskt i bakgrunden och du kan börja analysera besöksbeteenden, populära sidor och trafikkällor direkt i din analytiska instrumentpanel.
