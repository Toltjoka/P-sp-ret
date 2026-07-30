# 🚂 PÅ SPÅRET — Draft Express

Ett eget "På Spåret"-spel byggt för fantasy football-draften, i LA Chargers-tema. Lag gissar sig fram till en NFL-stad ledtråd för ledtråd (10 → 8 → 6 → 4 → 2 → 1 poäng), med mellanfrågor mellan resorna, live poängtavla och en storbildsvy alla kan titta på.

**Live:** https://toltjoka.github.io/P-sp-ret/

Byggd som en enda statisk `index.html`-fil (vanilla JS, ingen byggprocess) som pratar direkt med [Supabase](https://supabase.com) för databas, inloggning och fillagring. Allt synkas i realtid mellan alla enheter som har sidan öppen.

---

## De tre lägena

Startskärmen låter var och en välja rätt läge för sig:

| Läge | Vem | Vad |
|---|---|---|
| 🎬 **Visningsläge** | Storskärmen/TV:n alla tittar på | Ren läsvy, ingen inloggning, uppdateras live |
| 🕹️ **Spelarläge** | Varje lag, på sin egen telefon | Går med via en 4-teckenskod, trycker "GISSA!", svarar på mellanfrågor |
| 🔐 **Adminläge** | Du (draftvärden) | Styr allt — resor, poäng, mellanfrågor, ljud, lag |

Man kan byta läge när som helst via "← Byt läge"-knappen. Spelarläget kommer ihåg vilket lag man var inloggad som om man laddar om sidan.

---

## Snabbstart en draftkväll

1. Öppna **Adminläge** → logga in.
2. **Lag & poäng**: sätt rätt antal lag ("Antal lag"-fältet), döp dem eller låt spelarna döpa sig själva i sitt läge, dela ut koderna (kopiera-knappen ⧉).
3. **Resor**: kolla igenom/redigera de tre färdiga resorna (Kansas City, Philadelphia, Baltimore) och deras ledtrådar, eller skapa fler.
4. **Mellanfrågor**: kolla igenom de 15 färdiga (5 per resa, parvis A/B-frågor).
5. Skicka länken + varje lags kod till spelarna, be dem öppna **Spelarläge**.
6. Sätt upp **Visningsläge** på en TV/projektor eller delad skärm.
7. I admin, tryck **▶ Starta resa** på en resa → nedräkning 3-2-1 → resan rullar igång automatiskt.
8. När ett lag trycker "GISSA!" fryser det — bocka av **✅ Rätt** eller **❌ Fel** i admin. Flera lag kan poängsätta samma resa, bara aldrig på samma nivå.
9. När resan är klar: **🏁 Avsluta resa → mellanfråga** kör igång dess mellanfrågor automatiskt. Bläddra med **Nästa fråga →**, dela ut poäng för fritt inskrivna svar.
10. Kör alla tre resor, avsluta till sist med **🏆 Avsluta spelet & visa vinnare** i Lag-fliken.

---

## Adminpanelen, flik för flik

### 🚂 Spela
- Bibliotek över alla resor, "▶ Starta resa" per kort.
- Under spel: stor bild/video + ledtrådstext, live nedräkningsstapel (bara admin ser den), lagens rätt/fel-knappar.
- Mellanfråge-kontroll dyker upp automatiskt efter avslutad resa.

### ✏️ Resor
- **Mediabibliotek** högst upp: ladda upp bilder/videor från valfri enhet (lagras i Supabase Storage, publikt), eller klistra in länkar.
- Resa-editor: 6 ledtrådar med egna poäng (default 10/8/6/4/2/1 — första bör vara en gåta, sista den självklara upplösningen), egen bild/video per ledtråd, sekunder-per-ledtråd (auto-nästa om ingen gissar), en **genomgående resevideo** som spolas fram i takt med resan, en **ankomstbild/-video** som visas vid sista ledtråden, och ett **ljud** från ljudbiblioteket som loopar i visningsläget.

### ❓ Mellanfrågor
- Kopplade till en specifik resa, körs automatiskt efter den.
- Två typer: **Fritext** (en fråga, ett svar) eller **Flerdelad** (A, B, C... — varje del egen text/facit/poäng, t.ex. "två frågor i ett").
- Ge varje fråga ett eget **namn** (visas istället för "Mellanfråga").
- Ni bedömer och skriver in exakt de poäng ni tycker passar per lag — inget facit-tvång.

### 🏈 Lag & poäng
- Antal lag, namn, koder, ljud per lag (från ljudbiblioteket, laddas upp från valfri enhet).
- Manuell poängjustering + **Ångra senaste poäng**.
- **🏆 Avsluta spelet & visa vinnare** / **↺ Fortsätt spela igen**.
- **🔄 Nollställ hela spelet** (poäng, resestatus, mellanfrågesvar).
- **Byt lösenord** för det inloggade admin-kontot.

---

## Tekniken bakom

- **Frontend**: en enda `index.html`, vanilla JS + [supabase-js](https://github.com/supabase/supabase-js) via CDN. Inget byggsteg — redigera filen och pusha.
- **Backend**: Supabase-projektet *"Toltjoka's Project"*.
  - `ps_journeys` — resor (ledtrådar, poäng, media, ljud)
  - `ps_teams` — lag (namn, poäng, join-kod, tilldelat ljud)
  - `ps_game_state` — singleton-rad med allt "live" (vilken resa som spelas, nedräkning, buzz-status, mellanfråga, vinnare)
  - `ps_bonus_questions` / `ps_bonus_answers` — mellanfrågor per resa + inskickade svar
  - `ps_sounds` / `ps_media` — bibliotek för uppladdade ljud/bilder/videor
  - Storage-bucket `ps-media` — publik, för uppladdade filer
- **Auth**: riktig Supabase Auth (e-post/lösenord) för admin. Två konton finns upplagda.
- **Hosting**: GitHub Pages, repo `Toltjoka/P-sp-ret`.
- **Realtid**: Supabase Realtime (Postgres changefeed) + en lätt 4-sekunders poll som backup, så alla öppna flikar uppdateras automatiskt.

## Kända begränsningar

- Spelare "loggar in" bara med en kod, ingen riktig autentisering — vem som helst med koden kan agera det laget. Funkar bra för en vänskaplig draftkväll, inte tänkt för skarpt läge.
- Svar/facit skickas tekniskt med i datan till alla anslutna enheter (bara dolt i gränssnittet) — en nyfiken deltagare skulle kunna se dem i webbläsarens nätverksflik.
- Auto-nedräkningen och video/ljud-uppspelningen i visningsläget kräver att den fliken hålls öppen; admin-timern kräver att admins flik hålls öppen.
- Ljud-/videofiler över ~10 MB kan vara klumpiga att hantera beroende på var de laddas upp ifrån.

## Att jobba vidare på

- Riktig per-lag-autentisering om det någonsin behöver vara mer skarpt.
- Fler resor / mellanfrågor efter behov — allt är byggt för att vara fritt redigerbart.
