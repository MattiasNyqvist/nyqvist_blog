---
title: "Min resa till Azure"
description: "Här skriver jag om hur jag satte upp min blogg och testar kodformatering."
pubDate: "Apr 28 2026"
---

# Hej världen!

Detta är mitt allra första inlägg. Jag har byggt denna blogg med Astro och Azure. För att se om allt fungerar som det ska, testar vi nu att visa lite C#-kod.

### Test av kodsyntax

Här är en enkel klass i C# för att hälsa på besökaren:

```csharp
public class WelcomeMessage
{
    public string Name { get; set; } = "Mattias";

    public void Greet()
    {
        // En enkel hälsning i konsolen
        Console.WriteLine($"Välkommen till {Name}s dev-blogg!");
    }
}
