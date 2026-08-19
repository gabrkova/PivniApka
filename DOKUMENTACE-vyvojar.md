# Pivní počítadlo — vývojářská dokumentace

Sdílené počítadlo piv pro partu. Každý na svém mobilu zapisuje piva, žebříček je společný. Aplikaci spravuje admin (zakládá akce, přepíná aktivní, zamyká). Bez instalace, bez registrace uživatelů.

---

## 1. Přehled architektury

- **Frontend:** dvě statické HTML stránky, čistý JavaScript bez frameworku a bez build kroku.
  - `index.html` — uživatelská aplikace.
  - `admin.html` — administrace (přihlášení Googlem).
- **Hosting:** GitHub Pages, repozitář `gabrkova/PivniApka`.
  - Aplikace: `https://gabrkova.github.io/PivniApka/`
  - Admin: `https://gabrkova.github.io/PivniApka/admin.html`
- **Backend:** Firebase Realtime Database, projekt `pivni-pocitacka`, region `europe-west1`.
  - Aplikace i admin čtou/zapisují přes **REST rozhraní** databáze (koncovky `.json`).
  - Admin se navíc přihlašuje přes **Firebase Authentication (Google)** a privilegované zápisy posílá s `?auth=<idToken>`.
- **Bezpečnost dat:** zajišťují **Firebase Security Rules** + přihlášení admina. Ne utajení klíče (Firebase `apiKey` je veřejný z principu).

Žádný server, žádná business logika mimo prohlížeč. Vše je klientská aplikace nad databází.

---

## 2. Datový model

```
/pivo
  /aktivni                 -> "<id akce>"           (řetězec; kterou akci aplikace používá)
  /akce
     /<id>
        nazev      : "Tatanice 2026"
        zalozeno   : 1712345678901                  (timestamp ms)
        odemcena   : true | false                   (chybí = bereme jako true)
        hraci
           /<slug> : { name, count, updatedAt }
        _log
           /<pushKey> : { name, delta, count, t }   (delta +1 / -1)
```

Starší uzly (po jednorázovém importu už aplikace nepoužívá, zůstávají jen ke čtení):

```
/pivo/nase-parta         -> původní jediná tabule (hráči + _log)
/pivo/archiv/<id>        -> původní snímky akcí { nazev, archivovanoKdy, data }
```

### slug

Identita hráče v rámci akce je odvozena z jména funkcí `slug()`: malá písmena, bez diakritiky, nealfanumerické znaky nahrazeny `_`. Stejné jméno = stejný záznam. Klíče začínající `_` (např. `_log`) nejsou hráči a v žebříčku se přeskakují.

---

## 3. `index.html` — uživatelská aplikace

### Konfigurace
- `DB_URL` — adresa Realtime Database (jediná konstanta k nastavení).
- `LOG_ZOBRAZIT` — kolik posledních událostí ukázat v „Kdo, kdy, co" (výchozí 25).

### Životní cyklus
1. `start()` načte jméno z `localStorage` a zavolá `nacti()`.
2. `nacti()`:
   - přečte `/pivo/aktivni`;
   - když je `null` → fáze **`zadnaAkce`** (hláška „Zatím žádná aktivní akce");
   - jinak načte `/pivo/akce/<id>`: nastaví `akceNazev`, `odemceno`, `radky` (= `hraci`), `logData` (= `_log`);
   - detekuje změnu aktivní akce přes `localStorage["pivo_akce"]` — při změně srovná lokální počet podle serveru (typicky na 0), aby se po přepnutí/založení akce telefony nespletly.
3. Samočinná obnova každých **15 s** (jen ve fázi `hraje` a když neběží přejmenování) a při návratu na záložku (`visibilitychange`).

### Fáze (`faze`)
`start` → `setup` (chybí DB_URL) / `zadnaAkce` / `jmeno` (zadání jména) / `hraje`.

### Zápis
- Přičtení/ubrání piva: `PUT /pivo/akce/<id>/hraci/<slug>` s `{ name, count, updatedAt }`.
- Zápis do dění: `POST /pivo/akce/<id>/_log` s `{ name, delta, count, t }`.
- Souběh: optimistické navýšení lokálně + fronta ukládání (`ukladam` / `ceka`); model je **last-write-wins** na úrovni záznamu hráče. Ojedinělé ztracené ťuknutí je přijatelné (je to zábavní appka).

### Klíčové funkce
- `pridej()` / `uber()` — ±1 pivo (v zamčené akci nedělají nic).
- `prihlas()` — přihlášení jménem; v zamčené akci jen čte, nezapisuje.
- `prihlasitJako()` — odhlásí telefon (smaže `pivo_jmeno`) a vrátí na zadání jména.
- `zacniPrejmenovat()` / `ulozNoveJmeno()` — přejmenování sebe: přepíše záznam pod novým slugem, **počet piv zůstává** (při kolizi jmen počty sečte), starý slug smaže.
- `exportCsv()` — stáhne žebříček aktivní akce jako CSV (UTF-8 s BOM, oddělovač `;`, sloupce `Jméno;Počet piv;Litry`).
- `seznam()` — sestaví a seřadí žebříček; `deni()` — posledních `LOG_ZOBRAZIT` událostí (nejnovější nahoře).

### Zamčený režim (`odemceno === false`)
Banner „🔒 Akce je uzamčená — jen ke čtení", pípa +1 i −1 neaktivní, skryté přejmenování. Žebříček, dění a export zůstávají. Ochrana je i v pravidlech databáze (viz níže), aplikace jen zrcadlí stav.

### localStorage
- `pivo_jmeno` — identita na tomto telefonu.
- `pivo_akce` — id naposledy viděné aktivní akce (detekce přepnutí akce).

---

## 4. `admin.html` — administrace

### Konfigurace
- `firebaseConfig` — veřejný konfigurační objekt z Firebase (apiKey, authDomain, databaseURL, projectId, appId, …).
- `ADMIN_EMAILS` — seznam adminů pro **UI bránu**. Ostrá brána je v pravidlech databáze (viz níže).

### Přihlášení
Firebase Auth (compat SDK, Google poskytovatel), `signInWithPopup`. Po přihlášení se kontroluje, zda je e-mail v `ADMIN_EMAILS`. ID token se získává `getIdToken()` a přikládá k privilegovaným zápisům jako `?auth=<token>`.

### Funkce
- **Nová akce** — `PUT /pivo/akce/<id>` `{ nazev, zalozeno, odemcena:true }` a nastaví `/pivo/aktivni = <id>`. Id = `slug(nazev)-<timestamp>`.
- **Nastavit jako aktuální** — `PUT /pivo/aktivni = <id>` (návrat k libovolné dřívější akci).
- **Zamknout / Odemknout** — `PUT /pivo/akce/<id>/odemcena = false|true`.
- **Přejmenovat** — `PUT /pivo/akce/<id>/nazev = "<název>"`.
- **Zobrazit** — jen ke čtení, žebříček akce.
- **Export** — CSV žebříčku akce.
- **Import dosavadních dat** — jednorázově zkopíruje `/pivo/nase-parta` a `/pivo/archiv/*` do `/pivo/akce/*` a nastaví aktivní akci. Nabízí se jen, když v novém modelu ještě žádná akce není. Stará data nechává beze změny.

---

## 5. Bezpečnostní pravidla (Realtime Database → Rules)

```json
{
  "rules": {
    "pivo": {
      "aktivni": {
        ".read": true,
        ".write": "auth != null && auth.token.email_verified == true && auth.token.email == 'gabr.kova@gmail.com'"
      },
      "akce": {
        ".read": true,
        "$id": {
          ".write": "auth != null && auth.token.email_verified == true && auth.token.email == 'gabr.kova@gmail.com'",
          "hraci": { ".write": "$id === root.child('pivo/aktivni').val() && root.child('pivo/akce/' + $id + '/odemcena').val() !== false" },
          "_log":  { ".write": "$id === root.child('pivo/aktivni').val() && root.child('pivo/akce/' + $id + '/odemcena').val() !== false" }
        }
      },
      "nase-parta": { ".read": true },
      "archiv": { ".read": true }
    }
  }
}
```

Co pravidla vynucují:
- **Čtení** akcí, ukazatele a starých uzlů je veřejné.
- **Zápis hráčů a dění** smí kdokoli, ale **jen do akce, která je aktivní a zároveň odemčená**.
- **Založení/přejmenování/zámek akce a přepnutí aktivní akce** smí jen admin (ověřený Google e-mail).
- Výchozí (chybějící) `odemcena` se bere jako odemčeno (`!== false`), takže starší akce se po nasazení nezamknou.

---

## 6. Přidání dalšího admina

Na **dvou místech**:
1. `admin.html` → přidat e-mail do `ADMIN_EMAILS` (UI brána).
2. Pravidla databáze → rozšířit podmínky `.write` u `aktivni` a `akce/$id` o další e-mail (ostrá brána). Např. `auth.token.email == 'a@x.cz' || auth.token.email == 'b@y.cz'`.

Jen UI úprava bez pravidel = admin uvidí tlačítka, ale zápisy mu databáze odmítne.

---

## 7. Nasazení

1. Commit `index.html` a/nebo `admin.html` do větve `main` repozitáře `gabrkova/PivniApka`.
2. GitHub Pages sestaví automaticky — v záložce **Actions** běh „pages build and deployment" (zelené ✓ = hotovo, typicky 1–3 min). Žádné ruční „deploy" není.
3. Pravidla databáze se publikují ve **Firebase konzoli** (Realtime Database → Rules → Publish).
4. Cache: kdyby se držela stará verze, `Ctrl+F5` (mobil: potáhnout dolů).

Pořadí: při změně, která závisí na pravidlech (zámek), publikovat pravidla; jinak pořadí nevadí.

---

## 8. Poznámky, omezení, provoz

- **Firebase `apiKey` je veřejný z principu** — není to tajemství. Data chrání pravidla + přihlášení, ne utajení klíče. Jediné, co do repa nikdy nepatří, je servisní účet / Admin SDK (soubor s `private_key`) — ten se nepoužívá.
- **Otevřenost aktivní akce:** kdokoli s odkazem může do aktivní odemčené akce zapisovat (u party záměr). Nic citlivého do databáze nepatří.
- **Růst logu:** `_log` roste s počtem ťuknutí; pro partu zanedbatelné, případně jde občas promazat v konzoli.
- **Volitelné zpřísnění:** omezit `apiKey` v Google Cloud konzoli na HTTP referrer `gabrkova.github.io/*` a jen Firebase API; případně zapnout App Check.
- **Přepnutí akce:** po `nastavAktualni`/založení nové akce se telefony srovnají při nejbližší obnově (detekce přes `pivo_akce`).

---

## 9. Struktura repozitáře

```
PivniApka/
  index.html     # uživatelská aplikace
  admin.html     # administrace
  README.md      # (volitelně)
```

Vše je ve dvou souborech — žádné závislosti, žádný build.
