# Stämpelklockan

En enkel mobilapp för att stämpla in och ut, ta lunch- och korta raster och exportera arbetstiderna månadsvis som CSV.

- **Appen** (`index.html`) ligger på GitHub Pages.
- **Datan** sparas i ett Google-kalkylark, i fliken *Pass* (ett pass per rad).
- **Inloggning** sker med Google-konto. Bara e-postadresser som står i listan `ALLOWED_EMAILS` i `Code.gs` släpps in. Alla andra får ett felmeddelande, även om de hittar adressen till appen.

Installationen tar ungefär 20 minuter och görs en gång.

---

## 1. Skapa kalkylarket och backend

1. Skapa ett nytt Google-kalkylark, t.ex. *Arbetstid*. Gör det med det konto som ska äga datan.
2. Välj **Tillägg → Apps Script**.
3. Radera innehållet i `Kod.gs` och klistra in hela `Code.gs` från det här projektet.
4. Ändra `ALLOWED_EMAILS` till din frus Gmail-adress. Fler adresser går bra, och varje person ser bara sina egna pass.
5. Spara. Välj funktionen **setup** i listan och klicka **Kör**. Godkänn behörigheterna. Flikarna *Pass* och *Status* skapas i kalkylarket.

`CLIENT_ID` fyller du i efter steg 2.

## 2. Skapa ett OAuth-klient-ID (för Google-inloggningen)

1. Gå till <https://console.cloud.google.com/> och skapa ett nytt projekt, t.ex. *Stämpelklockan*.
2. **API:er och tjänster → OAuth-samtyckesskärm**: välj *Extern*, fyll i appnamn och din e-post. Lägg till din frus adress under *Testanvändare*. Du kan också publicera appen; den använder bara grundläggande profilinformation.
3. **API:er och tjänster → Inloggningsuppgifter → Skapa inloggningsuppgifter → OAuth-klient-ID**
   - Programtyp: **Webbapplikation**
   - *Auktoriserade JavaScript-ursprung*: `https://DITT-GITHUB-NAMN.github.io`
4. Kopiera klient-ID:t (slutar på `.apps.googleusercontent.com`).
5. Klistra in det i `CLIENT_ID` i Apps Script-koden och spara.

## 3. Publicera backend som webbapp

1. Klicka **Distribuera → Ny distribution** i Apps Script och välj typen **Webbapp**.
   - *Kör som*: **Jag**
   - *Vem har åtkomst*: **Alla**

   Inställningen *Alla* behövs för att appen på GitHub ska kunna nå skriptet. Skriptet godkänner ändå bara inloggade Google-konton som finns i `ALLOWED_EMAILS`.
2. Kopiera webbappens URL (slutar på `/exec`).

> När du ändrar i `Code.gs` senare: välj **Distribuera → Hantera distributioner → redigera → Version: Ny version**. Då behåller du samma URL.

## 4. Lägg upp appen på GitHub Pages

1. Öppna `index.html` och fyll i de två raderna högst upp:
   ```js
   API_URL: "https://script.google.com/macros/s/…/exec",
   CLIENT_ID: "123…apps.googleusercontent.com"
   ```
2. Skapa ett repo på GitHub, t.ex. `stampelklockan`, och ladda upp `index.html`.
3. Gå till **Settings → Pages**, välj *Deploy from a branch*, `main` och `/ (root)`, och spara.
4. Efter någon minut finns appen på `https://DITT-GITHUB-NAMN.github.io/stampelklockan/`.

Ursprunget i steg 2.3 ska vara `https://DITT-GITHUB-NAMN.github.io`, utan `/stampelklockan`.

## 5. På telefonen

Öppna adressen och logga in med Google.

Lägg sedan till appen på hemskärmen, så öppnas den som en vanlig app:
- **iPhone (Safari):** Dela-knappen → *Lägg till på hemskärmen*
- **Android (Chrome):** menyn ⋮ → *Lägg till på startskärmen*

---

## Så används appen

| Knapp | Vad händer |
|---|---|
| **Kom** | Stämplar in med serverns klocka |
| **Lunchrast / Kort rast** | Startar en rast. Rasttiden dras av från arbetstiden |
| **Avsluta rast** | Tillbaka i arbete |
| **Gå** | Stämplar ut och sparar passet som en rad i kalkylarket. En pågående rast avslutas automatiskt |
| **+ Lägg till pass** | Lägger till ett pass i efterhand, t.ex. om hon glömde stämpla |
| Tryck på ett pass | Ändrar eller raderar passet |
| **Exportera CSV** | Laddar ned den visade månaden som `arbetstid-ÅÅÅÅ-MM.csv` |

CSV-filen är semikolonseparerad och använder decimalkomma, så den öppnas rätt i svensk Excel. Den har en summarad längst ner.

Kalkylarket går också att läsa direkt. Kolumnerna *Start (ms)*, *Slut (ms)* och *Rastdata* är dolda; dem använder appen internt.

## Säkerhet i korthet

- Inloggningen görs av Google. Appen hanterar inga lösenord.
- Varje anrop till kalkylarket innehåller en signerad inloggningsbiljett från Google. Skriptet kontrollerar biljetten hos Google och jämför e-postadressen med `ALLOWED_EMAILS`.
- Biljetten gäller i en timme och förnyas automatiskt när hon är inloggad i Google på telefonen.
