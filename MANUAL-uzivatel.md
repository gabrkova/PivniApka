# Pivní počítadlo — uživatelský manuál

Společné počítadlo piv pro partu. Každý na svém mobilu čárkuje svoje piva, žebříček vidí všichni.

**Odkaz:** `https://gabrkova.github.io/PivniApka/`

---

## Pro pijáky

### Začátek
1. Otevři odkaz v mobilu (nic se neinstaluje, nikam se nepřihlašuješ).
2. Napiš svoje jméno a dej **Jdu pít**.
   - Aplikace si tvoje jméno na tomhle telefonu **zapamatuje** — příště tě rovnou pustí dál.
   - Piš stejné jméno pokaždé. Stejné jméno = stejný účet, jiné jméno = nový účet.

### Počítání
- Za každé vypité pivo ťukni na velké tlačítko **+1 PIVO**.
- Spletl ses? Dej **− 1 (překlep)**.

### Co na obrazovce uvidíš
- Nahoře **název akce** (např. „Tatanice 2026").
- **Tvoje číslo** — kolik piv máš na tácku.
- **Žebříček** — pořadí celé party, medaile pro první tři.
- **Parta celkem** — kolik piv i litrů padlo dohromady.
- **Kdo, kdy, co** — historie: kdo si kdy přičetl nebo ubral pivo.

Vše je společné a průběžně se samo obnovuje (zhruba každých 15 sekund), takže vidíš i piva ostatních.

### Užitečné volby (dole)
- **Přejmenovat se** — změní tvoje jméno a **počet piv ti zůstane**.
- **Přihlásit se jako někdo jiný** — přepne telefon na jiného člověka (třeba když půjčíš mobil).
- **Export žebříčku (CSV)** — stáhne tabulku „kdo kolik" (otevře se v Excelu, s diakritikou).

### Když je akce uzamčená 🔒
Někdy admin akci **uzamkne** (např. po skončení, kvůli vyúčtování). Pak vidíš banner „Akce je uzamčená — jen ke čtení", žebříček a historie zůstávají, ale **přičítat piva nejde**. To je v pořádku, ne chyba.

### Když se ukáže „Zatím žádná aktivní akce"
Znamená to, že admin ještě nezaložil (nebo nepřepnul) akci. Počkej chvíli a dej **Zkusit znovu**.

---

## Pro admina

Admin rozhraní: `https://gabrkova.github.io/PivniApka/admin.html` — přihlášení přes Google účet (musíš mít oprávnění).

### Přehled akcí
Po přihlášení vidíš seznam všech akcí. U každé je datum, počet lidí a piv, případně odznak **AKTUÁLNÍ** nebo **🔒 ZAMČENO**.

### Co můžeš dělat
- **Nová akce** — zadáš název, akce se založí a rovnou se nastaví jako aktuální (dosavadní zůstanou v seznamu). Nová akce je odemčená.
- **Nastavit jako aktuální** — přepne aplikaci všem na vybranou akci (i na dřívější — návrat k předchozí akci).
- **Zamknout / Odemknout** — u zamčené akce mohou lidé jen číst, nemohou zapisovat.
- **Přejmenovat** — změní název akce.
- **Zobrazit** — žebříček akce jen ke čtení.
- **Export** — stáhne CSV žebříčku akce.

### Typický průběh
1. Před akcí: **Nová akce** → název (např. „Chalupa 2026"). Tím se stane aktuální a odemčenou.
2. Během akce: lidé čárkují piva.
3. Po akci: **Zamknout** (zafixuje počty pro vyúčtování). Data zůstávají a dají se kdykoli zobrazit nebo exportovat.
4. Příště: založíš další novou akci. Ke starým se kdykoli vrátíš přes **Nastavit jako aktuální** nebo je jen prohlížíš.

---

## Časté dotazy

**Musím si něco instalovat nebo se registrovat?** Ne. Stačí otevřít odkaz v prohlížeči.

**Vidí ostatní, kolik mám piv?** Ano, žebříček i historie jsou společné pro všechny s odkazem.

**Ťukl jsem omylem víc piv.** Použij **− 1 (překlep)**.

**Změnil jsem si jméno — přijdu o piva?** Ne, přes **Přejmenovat se** ti počet zůstane. (Pozor: „Přihlásit se jako někdo jiný" je něco jiného — to tě přepne na jiného člověka.)

**Appka ukazuje starou verzi.** Obnov stránku (na mobilu potáhni dolů).

Pijte s rozumem. 🍻
