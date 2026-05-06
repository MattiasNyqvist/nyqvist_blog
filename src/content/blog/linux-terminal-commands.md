---
title: "Linux i terminalen – kommandon du faktiskt använder"
description: "En bred referens över de kommandon du stöter på i vardagen – organiserad efter situation, inte kommandonamn."
pubDate: 2026-05-05
heroImage: "../../assets/images/blog/linux-terminal-commands/hero-linux-terminal-commands.jpg"
tags: ["linux", "cli", "terminal", "tutorial"]
---

Ett grafiskt gränssnitt är smidigt för det vanliga. Terminalen är bättre när du vill automatisera, kombinera kommandon, felsöka på riktigt eller arbeta mot en server som inte har något GUI alls. Du skriver exakt vad du vill göra och får exakt det tillbaka – ingen meny som döljer alternativen.

Den här guiden är organiserad efter situation, inte efter kommandonamn. Kommandona är testade på [Linux Mint](https://linuxmint.com/) 22.3, men fungerar i stort sett likadant på Ubuntu och andra Debian-baserade system.

Kör det här för att se vilken distribution och version du använder:

```bash
neofetch         # förinstallerat på Linux Mint
lsb_release -a  # kortare alternativ om neofetch saknas
```

Neofetch skriver ut systeminformation tillsammans med en ASCII-logotyp för din distribution – kernel, minne, CPU och mer på en gång. `lsb_release -a` ger samma grundinformation men i ett rakare textformat, och finns på de flesta Debian-baserade system.

**Linux är skiftlägeskänsligt.** Det innebär att `Dokument`, `dokument` och `DOKUMENT` är tre helt olika mappar. Samma gäller kommandon och filnamn. Det är en av de vanligaste källorna till förvirring i början – ett filnamn som ser rätt ut kan ändå ge ett felmeddelande om stora och små bokstäver inte stämmer exakt.

## Var är jag, och vad finns här?

Det första man lär sig är orientering. `pwd` berättar var i filsystemet du befinner dig – full sökväg, ingen tvekan. `ls` listar vad som finns i mappen. Lägger du till `-la` får du även dolda filer och detaljer som rättigheter och storlek.

```bash
pwd         # var är jag just nu?
ls -la      # vad finns här, inklusive det dolda?
```

`cd` tar dig dit du vill. `cd ~` eller bara `cd` skickar dig hem till din hemkatalog, `cd ..` tar dig ett steg uppåt. Vill du hoppa till föregående mapp fungerar `cd -` som ett minne – praktiskt när du byter fram och tillbaka mellan två ställen.

```bash
cd ~            # gå till hemkatalogen
cd Dokument     # gå in i en mapp (kom ihåg: skiftlägeskänsligt)
cd ..           # ett steg uppåt
cd -            # hoppa tillbaka till föregående plats
```

`tree` ger en grafisk vy av mappstrukturen, bra för att snabbt förstå hur ett projekt är organiserat. Det är inte förinstallerat – och det är ett bra tillfälle att nämna hur man installerar saker.

## Pakethantering

På Linux Mint och andra Debian-baserade system installerar du program med `apt`. Det är också det du använder för att hålla systemet uppdaterat.

```bash
sudo apt update                     # hämta information om tillgängliga uppdateringar
sudo apt upgrade                    # installera uppdateringar
sudo apt install tree               # installera ett program
sudo apt remove tree                # avinstallera
sudo apt search något               # sök efter paket
```

Kommandon i den här guiden som inte är förinstallerade markeras med `# kräver installation` i kodexemplet. Tillbaka till `tree`:

```bash
sudo apt install tree   # kräver installation
tree -L 2               # visa två nivåer djupt
```

`du -sh` summerar hur mycket plats en mapp tar. Bra för att förstå vart diskutrymmet tar vägen:

```bash
du -sh Bilder       # hur stor är mappen Bilder?
du -sh *            # storlek på allt i nuvarande mapp
```

## Hantera filer och mappar

`touch` skapar en tom fil. `mkdir` skapar en mapp, och med `-p` skapar den hela sökvägen på en gång:

```bash
touch notes.txt
mkdir -p projekt/src/components
```

`cp` kopierar, `mv` flyttar eller döper om, `rm` tar bort. Den sista är den man bör respektera mest – det finns ingen papperskorg.

```bash
cp notes.txt notes_backup.txt
cp -r src/ src_backup/          # kopiera en hel mapp
mv notes.txt Dokument/notes.txt
rm notes_backup.txt
rm -rf gamla-projektet/         # tar bort mapp och allt innehåll, utan att fråga
```

`ln -s` skapar en symbolisk länk – en genväg till en fil eller mapp som ser ut som originalet:

```bash
ln -s /etc/nginx/sites-available/min-sajt /etc/nginx/sites-enabled/
```

## Arkivering och komprimering

`tar` är standardverktyget för att packa ihop och packa upp filer. Flaggorna kan kännas kryptiska i början, men det är i princip alltid samma kombinationer:

```bash
tar -czf arkiv.tar.gz mapp/     # packa ihop och komprimera
tar -xzf arkiv.tar.gz           # packa upp
tar -tzf arkiv.tar.gz           # lista innehållet utan att packa upp
```

`zip` och `unzip` för att hantera zip-filer:

```bash
zip -r arkiv.zip mapp/
unzip arkiv.zip
```

## Titta på innehållet i en fil

`cat` skriver ut hela filen direkt i terminalen. Funkar fint för korta filer. För längre är `less` bättre – det låter dig scrolla och söka med `/`.

`head` och `tail` visar de första respektive sista raderna. Standardvärdet är tio rader, men du kan ange hur många du vill:

```bash
head -20 /var/log/syslog
tail -50 /var/log/syslog
```

`tail -f` är det kommando man använder mest när något krånglar. Det följer en loggfil i realtid medan den växer:

```bash
tail -f /var/log/syslog
```

`wc` räknar rader, ord och tecken i en fil – smidigt för att få en snabb känsla för storleken:

```bash
wc -l /var/log/syslog    # hur många rader?
```

## Rättigheter och användare

Många filer i Linux är skyddade. `sudo` låter dig köra ett kommando med administratörsbehörighet. Det är lätt att köra det lite för reflexmässigt – om något kräver `sudo` är det värt att förstå varför.

`chmod` ändrar vem som får läsa, skriva eller köra en fil. `chown` byter ägare:

```bash
chmod 755 script.sh         # ägaren kan allt, övriga kan läsa och köra
chmod +x script.sh          # gör en fil körbar
chown mattias:mattias fil.txt
```

`whoami` visar vilket konto du är inloggad som. `id` ger mer detaljer, inklusive vilka grupper du tillhör:

```bash
whoami
id
```

## Redigera filer direkt i terminalen

Förr eller senare behöver du ändra en konfigurationsfil utan tillgång till något grafiskt gränssnitt. Nano är det enklaste alternativet och finns förinstallerat på Linux Mint. Gränssnittet är enkelt, och de viktigaste tangentkombinationerna visas längst ned på skärmen hela tiden.

```bash
nano ~/.bashrc
```

Du öppnar filen, ändrar det du vill och sparar med `Ctrl + O` följt av Enter. Sedan stänger du med `Ctrl + X`.

Om du föredrar något modernare finns **Micro** – en terminaleditor med stöd för musklick, syntax-highlighting och kortkommandon som liknar vanliga texteditorer:

```bash
sudo apt install micro   # kräver installation
micro ~/.bashrc
```

I Micro sparar du med `Ctrl + S` och stänger med `Ctrl + Q`. Känns mer bekant om man är van vid vanliga editorer.

## Vad körs på systemet?

`htop` är förstaval för att få en snabb bild av processer, CPU och minne – en mer läsbar variant av `top`. Det behöver installeras:

```bash
sudo apt install htop   # kräver installation
htop
```

Om du vill ha en statisk ögonblicksbild går `ps aux` bra, ofta kombinerat med `grep` för att hitta en specifik process:

```bash
ps aux | grep firefox
```

`kill` avslutar en process med dess ID. Vägrar den stängas finns `kill -9` – det är den hårda versionen. `pkill` låter dig avsluta processer på namn istället för ID:

```bash
kill 1234
kill -9 1234
pkill firefox
```

`systemctl` hanterar tjänster som körs i bakgrunden:

```bash
systemctl status bluetooth
systemctl restart bluetooth
systemctl enable bluetooth      # starta automatiskt vid omstart
```

`df -h` visar ledigt diskutrymme på alla monterade enheter. `free -m` visar RAM-användning. Bra att ha till hands när något känns trögare än vanligt:

```bash
df -h
free -m
```

`uptime` visar hur länge systemet har kört och den genomsnittliga belastningen.

## Loggar med journalctl

På moderna Linux-system samlar `journalctl` ihop loggar från hela systemet på ett ställe. Det är ett bra komplement till `tail -f` när du vill filtrera eller söka bakåt i tid.

```bash
journalctl -n 50                          # visa de senaste 50 raderna
journalctl -f                             # följ loggar i realtid
journalctl -u bluetooth                   # visa bara loggar för en specifik tjänst
journalctl --since "1 hour ago"           # vad har hänt den senaste timmen?
journalctl --since yesterday --until now  # bredare tidsfönster
```

## Nätverk och fjärråtkomst

`ip addr` visar dina nätverksgränssnitt och IP-adresser. Utdatan kan se ut som mycket men det du letar efter är raden som börjar med `inet` under ditt aktiva gränssnitt – där står din lokala IP-adress, t.ex. `192.168.1.42`. Gränssnittet heter oftast `eth0` för kablet nätverk eller `wlan0` för wifi, men namnet kan variera beroende på systemet:

```bash
ip addr
```

`ping` kontrollerar om en server svarar och ger dig svarstider:

```bash
ping google.com
ping -c 4 google.com    # skicka exakt 4 paket, sen avsluta
```

`traceroute` visar hela vägen paketen tar från din dator till destinationen – bra för att se var i kedjan något hänger sig:

```bash
sudo apt install traceroute   # kräver installation
traceroute google.com
```

`nslookup` och `dig` används för att felsöka DNS – du kan fråga vilket IP-nummer ett domännamn pekar på:

```bash
nslookup google.com
dig google.com
```

`ss -tulpn` visar vilka portar som lyssnar på din maskin och vilket program som håller i dem:

```bash
ss -tulpn
```

`ssh` loggar in dig på en annan maskin säkert. `scp` kopierar filer mellan maskiner via SSH:

```bash
ssh user@server.example.com
scp fil.txt user@server.example.com:/home/user/
```

`curl` hämtar innehåll från en URL direkt i terminalen – användbart för att testa att en tjänst svarar:

```bash
curl https://google.com
curl -I https://google.com                   # visa bara HTTP-headers
curl -o fil.zip https://example.com/fil.zip  # ladda ner och spara
```

`wget` är ett alternativ till `curl` som är bättre på att ladda ner filer, särskilt stora:

```bash
wget https://example.com/stor-fil.iso
```

## Jag letar efter något

Det här är terminalen på sitt bästa. `find` söker efter filer baserat på namn, storlek eller tid. Punkten `.` betyder "sök härifrån och nedåt":

```bash
find . -name "*.pdf"            # alla PDF-filer härifrån och nedåt
find . -mtime -7                # filer ändrade senaste 7 dagarna
find . -size +100M              # filer större än 100 MB
```

`grep` söker inuti filer efter text. Ovärderligt när du letar efter en inställning i en konfigurationsfil och inte vet exakt var:

```bash
grep -r "WantedBy" /lib/systemd/system/   # sök rekursivt i en mapp
grep -n "error" /var/log/syslog           # visa radnummer i resultatet
```

`which` svarar på frågan "vilket program är det egentligen som körs?" – praktiskt när du undrar vilken version av ett verktyg som är aktiv:

```bash
which python3
which bash
```

`whereis` ger lite mer – visar binär, källkod och manualsida för ett program:

```bash
whereis nano
```

## Omdirigering och piping

Det är här terminalen verkligen börjar visa sin styrka – att koppla ihop kommandon med varandra.

`|` (pipe) skickar utdata från ett kommando vidare till nästa:

```bash
ps aux | grep firefox                       # filtrera processlistan
journalctl -n 1000 | grep "error" | wc -l  # räkna fel i loggen
```

`>` skriver utdata till en fil, `>>` lägger till utan att skriva över:

```bash
echo "hello" > fil.txt          # skriver över
echo "world" >> fil.txt         # lägger till
ls -la > fillistning.txt
```

`2>` fångar felmeddelanden separat – användbart i skript när du vill skilja på vanlig utdata och fel:

```bash
kommando 2> fel.log             # spara fel till fil
kommando > utdata.log 2>&1      # spara både utdata och fel
```

## Sådant som sparar tid varje dag

Pil upp och pil ned bläddrar bland tidigare kommandon – ett av de första man lär sig och ett av de man använder mest.

Tab-komplettering är nästa steg. Börja skriva ett filnamn och tryck Tab – terminalen fyller i resten. Finns det flera alternativ visas de.

`history` listar tidigare kommandon. `Ctrl + R` låter dig söka bakåt i historiken – börja skriva och den matchar mot det du kört tidigare.

`alias` skapar kortkommandon för kommandon du skriver om och om igen. Skriver du bara `alias` i terminalen gäller det bara för den sessionen – stänger du fönstret är det borta. För att spara dem permanent lägger du till dem i `~/.bashrc`:

```bash
nano ~/.bashrc
```

Lägg till dina alias längst ned:

```bash
alias ll='ls -la'
alias gs='git status'
```

Ladda sedan om filen så att ändringarna gäller direkt:

```bash
source ~/.bashrc
```

`clear` rensar terminalfönstret. `Ctrl + C` avbryter ett kommando som körs. `Ctrl + Z` pausar det och skickar det till bakgrunden.

`man` plus ett kommandonamn öppnar manualen för det verktyget. Det är det faktiska facit när man undrar vad en flagga gör:

```bash
man find
man curl
```

Det tar lite tid att lära sig terminalen ordentligt, men det är en av de färdigheter som faktiskt stannar kvar. Varje kommando du förstår på riktigt – inte bara kopierar – gör nästa problem lite enklare att lösa. Och efter ett tag börjar man märka att det inte längre är ett verktyg man tvingas använda, utan ett man väljer.