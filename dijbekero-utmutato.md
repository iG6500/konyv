# Automata díjbekérő — beüzemelési útmutató

Ez a három fájl (`dijbekero-dedikalt.html`, `dijbekero-normal.html`,
`koszonto-varolista.html`) nem a weboldal része, semmi nem hivatkozik rájuk
élesben. Kizárólag arra valók, hogy a teljes forráskódjukat bemásold a
MailerLite Automation → Email lépés → **"Code your own" / Custom HTML**
szerkesztőjébe.

*(Az előző verzió a fájl tetején, egy nagy HTML-kommentben tartalmazta ezt
az útmutatót — kivettem onnan, mert az egyik gyanús ok arra, hogy a
MailerLite beillesztéskor levágta a tartalom nagy részét, pont az volt,
hogy a fájl nem `<!DOCTYPE html>`-lel kezdődött. Most tisztán azzal
kezdődik, semmi más nincs előtte.)*

## 1. Banki adatok

Mindkét fájlban keresd meg és írd át:
- `[SZÁMLATULAJDONOS NEVE]`
- `[BANKSZÁMLASZÁM]`

## 2. Személyre szabó címkék ellenőrzése

A `{$name}` és `{$email}` biztosan jó — ezek MailerLite beépített mezők,
dokumentált szintaxis. Az egyéni mezőknél (`{$osszesen}`,
`{$peldany_szamok}`, `{$dedikalas}`, `{$dedikalas_uzenet}`, `{$cim}`,
`{$peldanyszam}`) ugyanezt a `{$mezőkulcs}` mintát követtem, MailerLite
dokumentációja alapján — de élő fiók nélkül nem tudtam 100%-ig
leellenőrizni. Mielőtt élesíted:

1. Nyisd meg a MailerLite szerkesztőjében a **"Fields and variables"**
   panelt.
2. Minden egyéni mezőnél **másold ki onnan** a pontos címkét.
3. Ha valamelyik eltér attól, amit a HTML-ben találsz, írd felül azzal.

## 3. Automatizálás feltétele — EZ FONTOS

A bevált, leellenőrzött megoldás: **Subscribers → Segments** alatt hozz
létre egy szegmenst mindkét változathoz, majd a szegmens lapján a
**"Create automation"** gombbal indítsd az automatizálást — ez azt a
triggert állítja be, hogy "amikor egy feliratkozó belép ebbe a
szegmensbe". A szegmens feltételei (mindhárom "És"-sel összekötve):

```
Fields → Mód          Equals        elorendeles
Fields → Változat     Equals        dedikalt   (a dijbekero-dedikalt.html-hez)
                        vagy         normal     (a dijbekero-normal.html-hez)
Fields → Sorszám várólista   Does not equal   igen
```

Az utolsó feltétel azért kritikus, mert ha valaki dedikáltat kért, amikor
épp nem volt szabad szám, ő csak **sorba állt**, nem kapott tényleges
példányt. Ha erre is automatikusan kimenne a díjbekérő, pénzt kérnél egy
könyvért, ami lehet, hogy nincs is neki.

### Tégy egy késleltetést az e-mail lépés elé

Ha valaki dedikáltat választott, de **nem** kattintott konkrét sorszámra a
jegyzékből, a `peldany_szamok` mező üresen érkezik be — neked kell utólag
kiválasztanod és beírnod a végleges számot a feliratkozó adatlapján, a
MailerLite-ban. Ha az automatizálás **azonnal**, a szegmensbe kerüléskor
(vagyis a beküldés pillanatában) elküldi a levelet, a díjbekérő üres
"Sorszám" sorral megy ki, mielőtt esélyed lenne kézzel kitölteni.

**Tedd be a Delay/Wait lépést az e-mail lépés elé** (akár csak 1-2 óra is
elég) — így van időd ellenőrizni az új feliratkozót, és ha üres a
sorszám, kézzel kitölteni, mielőtt a levél ténylegesen kimegy. Azoknál,
akik konkrét számot választottak maguknak a jegyzékből, ez a mező eleve
ki van töltve a beküldéskor — nekik a késleltetés csak biztonsági
ráhagyás.

## 4. A "csonka levél" hiba — MEGOLDVA, de tartsd szem előtt

Korábban a ténylegesen kiküldött levél (nem csak a szerkesztő élő
előnézete) félúton megszakadt, majd minden nyitott HTML-címke egyszerre
lezárult. Több teszt-levél nyers forrásának lemérésével kiderült: **a
MailerLite automatizálás-motorjának van egy nem dokumentált, kb. 6,5–7 KB
körüli kemény korlátja** a kiküldött levél méretén — ez nem a HTML
szerkezetében volt hiba, hanem méret kérdése.

A jelenlegi sablonok (`dijbekero-dedikalt.html` ~6,0 KB,
`dijbekero-normal.html` ~5,4 KB) már biztonságosan a korlát alatt vannak,
és élesben, teljes egészében leellenőrzött állapotban vannak (a nyers
e-mail forrás `</html>`-ig ért).

**Ha a jövőben bővíted a szöveget** (pl. új mező, hosszabb magyarázat), és
a levél megint csonkán érkezne:

1. Mérd le a fájl méretét (`wc -c dijbekero-*.html`) — ha 6 KB fölé megy,
   valószínűleg megint elakad.
2. Rövidíts a bekezdéseken, vagy vedd ki a nem létfontosságú sorokat.
3. Ellenőrzés: kérj egy "Send test email"-t, majd a Gmailben az
   "Eredeti üzenet letöltése" / "Show original" nézetből másold ki a
   `Content-Type: text/html` rész teljes tartalmát, és nézd meg, eléri-e
   a `</html>` záró címkét.

(Mellékesen: próbáld **Ctrl+Shift+V**-vel beilleszteni a sima Ctrl+V
helyett, ha a beillesztés magában is gyanúsan viselkedne — egyes
gazdag szövegdobozok megpróbálják "értelmezni" a HTML-t beillesztéskor.
A MailerLite ZIP-behúzást és URL-importálást is támogat alternatívaként.)

## 5. Köszöntő e-mail a várólistának

A `koszonto-varolista.html` ugyanúgy Custom HTML-ként megy be, saját
automatizálásba:

1. **Subscribers → Segments**: hozz létre egy szegmenst `Fields → Mód
   Equals varolista` feltétellel (ez mindenkit befog, aki most iratkozik
   fel, hiszen a `mod` mező `"varolista"` értékkel érkezik — lásd
   `index.html` `MAILERLITE_FIELDS.mod`).
2. A szegmens lapján **"Create automation"**, e-mail lépésben illeszd be
   a `koszonto-varolista.html` teljes forráskódját.
3. **Nincs szükség Delay/Wait lépésre** ennél — a köszöntő nem függ olyan
   mezőtől, amit utólag neked kellene kitöltened (ellentétben a
   díjbekérővel, ahol a `peldany_szamok` mező üres lehet). Mehet
   azonnal, a szegmensbe kerüléskor.
4. Csak `{$name}` személyre szabó címkét használ — nincs egyéni mező,
   amit előbb ellenőrizni kellene.

Ez a levél **nem** a díjbekérő — nem kér fizetést, nem tartalmaz banki
adatot. Csak megerősíti a feliratkozást, és felkészíti az embert arra,
hogy október 22-én kap linket.

## 6. Korai fizetési lehetőség a listásoknak

Döntés: ez **csak a listásoknak** szól, nem mindenkinek. A nyilvánosság
felé az oldal továbbra is "várólista" marad október 26-ig (hero, meta
leírás, FAQ — semmi nem változott ezekben). A listásoknak viszont nem
kell megvárniuk október 22-ét, ha fizetnének — ez most már az `index.html`
kódjában is megvalósul, nem csak e-mail-küldéssel.

### 6a. Azonnali "Fizetnék most" — ez az elsődleges út

Amikor valaki beküldi a várólista-űrlapot, a visszaigazoló panelen
("Felvéve a listára") megjelenik egy **"Fizetnék most →"** gomb
(`#paynow-btn`, `index.html`). Erre kattintva a JS helyben átkapcsolja az
oldalt előrendelés-módba (a `preorderOpen` változót igazra állítja,
`applyMode()` + a kapcsolódó render-függvények újrafutnak) — a már
kitöltött adatok (név, e-mail, választott változat, kiválasztott
sorszám) megmaradnak, csak a szállítási cím mező bukkan elő és válik
kötelezővé. A második beküldésnél a `mod` mező már `"elorendeles"`
értékkel megy ki, tehát **ugyanaz** a 3. pontban leírt automatizálás
(`Mód Equals elorendeles` szegmens → díjbekérő) kapja el — nem kell
hozzá semmilyen új MailerLite-beállítás.

Technikai megjegyzés, ha később hozzányúlsz a kódhoz: az `applyMode()`
korábban `removeChild`-del **véglegesen kivette** a DOM-ból a
`data-preorder-only` mezőket (pl. a szállítási címet), amíg nem nyitott
az előrendelés. Ez összeférhetetlen lett volna a "Fizetnék most"
funkcióval (a mező soha nem tudott volna visszakerülni), ezért ez most
`style.display`-jal reverzibilis elrejtésre lett átírva.

### 6b. Emlékeztető kampány október 22-én — kiegészítő, nem kötelező

Aki nem kattintott a "Fizetnék most" gombra, annak érdemes egy
**időzített, egyszeri kampányt** küldeni (MailerLite: Campaigns →
Regular campaign, NEM Automation) a `CONFIG.PREORDER_OPEN` dátumára
(jelenleg október 22. csütörtök 9:00 — ha ez a dátum változik, az
ütemezést is told el vele együtt):

1. Címzett: a várólista-szegmens (`Mód Equals varolista`).
2. Tartalom: rövid emlékeztető + a live oldal linkje — ők ekkor már
   automatikusan előrendelés-módban látják az oldalt, hiszen a
   `CONFIG.PREORDER_OPEN` időpontja elérkezett.
3. Ne hirdesd sehol máshol (poszt, hirdetés) október 26. előtt — a
   nyilvánosság felé az oldal csak akkor vált látszólag is nyilvánosan
   megnyitottá.

## 7. Új mezők (szállítás és elállás miatt) — hozd létre a MailerLite-ban

**Subscribers → Fields**, mindhárom **Text** típus, pontosan ezekkel a
kulcsokkal:

| Kulcs | Mire való |
|---|---|
| `foxpost` | a választott Foxpost automata (a rendelés űrlapjáról) |
| `elallas_targy` | melyik rendelésről áll el a vevő (az elállási oldalról) |
| `elallas_idopont` | az elállás beérkezésének ideje, pl. `2026. 11. 30. 14:05` |

Enélkül a MailerLite csendben eldobja ezeket az értékeket.

A díjbekérőkben (`dijbekero-*.html`) mostantól a végösszeg a **szállítással
együtt** szerepel (az oldal számolja ki: ár × darab + 1 800 Ft), és a cím
helyén a **Foxpost automata + számlázási cím** áll — ezért **mindkét
díjbekérőt újra be kell illeszteni** (Ctrl+Shift+V, utána teszt-levél).

## 8. Elállási automatizálás — kötelező (2023/2673 irányelv)

A `elallas.html` oldal kétlépéses űrlapja ugyanarra a MailerLite-űrlapra küld,
`mod = elallas` értékkel. A visszaigazolást a te automatizálásod küldi ki:

1. **Segments**: új szegmens `Fields → Mód  Equals  elallas`.
2. **Create automation** a szegmens lapján, e-mail lépésben az
   `elallas-visszaigazolas.html` teljes forráskódja. **Delay nélkül** —
   a visszaigazolásnak késedelem nélkül ki kell mennie.
3. Teszteld: töltsd ki az elállási oldalt egy saját címmel, és nézd meg,
   megérkezik-e a levél az időponttal és a rendelés megnevezésével.

**Figyeld ezt a szegmenst** (vagy kapcsolj rá értesítést): minden belépés egy
elállás, amire 14 napon belül vissza kell utalni. Ha a vevő korábban
leiratkozott a listáról, előfordulhat, hogy a MailerLite nem veszi fel újra —
ezért az oldal mindig felkínálja az e-mailes küldést is, tehát a
nemjopasztor@gmail.com postafiókot is érdemes figyelni.
