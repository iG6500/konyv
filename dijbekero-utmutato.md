# Automata díjbekérő — beüzemelési útmutató

Ez a három fájl (`dijbekero.html`, `koszonto-varolista.html`,
`elallas-visszaigazolas.html`) nem a weboldal része, semmi nem hivatkozik rájuk
élesben. Kizárólag arra valók, hogy a teljes forráskódjukat bemásold a
MailerLite Automation → Email lépés → **"Code your own" / Custom HTML**
szerkesztőjébe.

*(Az előző verzió a fájl tetején, egy nagy HTML-kommentben tartalmazta ezt
az útmutatót — kivettem onnan, mert az egyik gyanús ok arra, hogy a
MailerLite beillesztéskor levágta a tartalom nagy részét, pont az volt,
hogy a fájl nem `<!DOCTYPE html>`-lel kezdődött. Most tisztán azzal
kezdődik, semmi más nincs előtte.)*

## 1. Banki adatok

A `dijbekero.html`-ben keresd meg és írd át:
- `[SZÁMLATULAJDONOS NEVE]`
- `[BANKSZÁMLASZÁM]`

## 2. Személyre szabó címkék ellenőrzése

A `{$name}` és `{$email}` biztosan jó — ezek MailerLite beépített mezők,
dokumentált szintaxis. Az egyéni mezőknél (`{$osszesen}`,
`{$tetel}`, `{$atvetel}`, `{$foxpost}`, `{$cim}`) ugyanezt a `{$mezőkulcs}` mintát követtem, MailerLite
dokumentációja alapján — de élő fiók nélkül nem tudtam 100%-ig
leellenőrizni. Mielőtt élesíted:

1. Nyisd meg a MailerLite szerkesztőjében a **"Fields and variables"**
   panelt.
2. Minden egyéni mezőnél **másold ki onnan** a pontos címkét.
3. Ha valamelyik eltér attól, amit a HTML-ben találsz, írd felül azzal.

## 3. A két csatorna és az automatizálások

Az oldalon két külön csatorna van:

- **F1 — „Csak feliratkozom”:** név + e-mail, semmi kötelezettség. A `mod` mező értéke `varolista`.
- **F2 — „Megrendelem”:** azonnali vásárlás, normál vagy dedikált példány. A `mod` mező értéke `elorendeles`.

**Ajánlott felépítés: két MailerLite-űrlap, két csoport.** Forms →
Embedded form → mindkettőhöz saját csoport; az űrlapok címét az
`index.html` tetején a `FORM_ENDPOINT_F1` és `FORM_ENDPOINT_F2` kapja.
Amíg a második űrlapot nem hozod létre, mindkét cím ugyanaz, és a két
csatornát a `mod` mező választja szét (szegmensekkel) — ez is működik.

Az ingyenes csomag **3 automatizálása** pont elég:

1. **F1-sorozat** (trigger: belép az F1 csoportba / a `Mód = varolista`
   szegmensbe): azonnal a köszöntő (`koszonto-varolista.html`), 72 óra
   múlva a részlet a könyvből, 168 óra múlva az önismereti felmérő és a
   vásárlásra hívó levél — egyetlen automatizálásban, Delay lépésekkel.
2. **F2 — díjbekérő** (trigger: belép az F2 csoportba / a `Mód =
   elorendeles` szegmensbe): azonnal a `dijbekero.html`. Késleltetés nem
   kell, és nincs szükség további feltételre sem: sorszám már nincs.
3. **Elállás** (lásd 8. pont).

A kifizetés utáni visszaigazolást (számlával) kézzel küldöd a Gmailből, a
novemberi „postázunk” levelet pedig egy kézzel vezetett „Fizetett”
csoportnak (a MailerLite nem tud csatolmányt küldeni, csak linket).

Az F2-vásárlók az F1-leveleket is megkapják: ehhez a díjbekérő-automatizálás
végére add hozzá őket az F1-sorozat csoportjához (vagy a sorozat
triggerét állítsd úgy, hogy az F2-re is induljon).

## 4. A "csonka levél" hiba — MEGOLDVA, de tartsd szem előtt

Korábban a ténylegesen kiküldött levél (nem csak a szerkesztő élő
előnézete) félúton megszakadt, majd minden nyitott HTML-címke egyszerre
lezárult. Több teszt-levél nyers forrásának lemérésével kiderült: **a
MailerLite automatizálás-motorjának van egy nem dokumentált, kb. 6,5–7 KB
körüli kemény korlátja** a kiküldött levél méretén — ez nem a HTML
szerkezetében volt hiba, hanem méret kérdése.

A jelenlegi díjbekérő (`dijbekero.html`, ~5,0 KB) biztonságosan a korlát alatt van,
és élesben, teljes egészében leellenőrzött állapotban vannak (a nyers
e-mail forrás `</html>`-ig ért).

**Ha a jövőben bővíted a szöveget** (pl. új mező, hosszabb magyarázat), és
a levél megint csonkán érkezne:

1. Mérd le a fájl méretét (`wc -c dijbekero.html`) — ha 6 KB fölé megy,
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

## 5. Köszöntő e-mail (F1 első levele)

Fájl: `koszonto-varolista.html`. Ez az F1-sorozat első levele (lásd 3. pont).

- Csak `{$name}` címkét használ — nincs egyéni mező.
- **Két helykitöltő van benne, ezeket élesítés előtt ki kell cserélned:**
  `[IDE KERÜL A KULISSZATITKOK SZÖVEGE]` és a kimutatás `[xx]%` értékei
  (a sorok nevei átírhatók).
- **Méretkorlát:** a fájl most ~5,6 KB, a MailerLite automatizálás-motorja
  ~6,5 KB körül csonkolja a levelet (lásd 4. pont). A helykitöltők helyére
  kb. 800 karakternyi szöveg fér még. Ha hosszabb a kulisszatitok-szöveg,
  tedd külön oldalra az oldalon, és a levélben csak egy rövid ízelítő és
  egy gomb legyen — szólj, és megcsinálom.

A 72 órás (részlet) és a 168 órás (önismereti felmérő + vásárlás) levelek
sablonját is megcsinálom, ha megvan a szövegük.

## 7. Új mezők (szállítás és elállás miatt) — hozd létre a MailerLite-ban

**Subscribers → Fields**, mind **Text** típus, pontosan ezekkel a
kulcsokkal:

| Kulcs | Mire való |
|---|---|
| `osszesen` | a fizetendő végösszeg szállítással együtt, pl. `7 200 Ft` — **enélkül üres a díjbekérő „Fizetendő összesen” sávja és az „Összeg” sor** |
| `tetel` | a díjbekérő „Tétel” sora kész szövegként (változat, darabszám, dedikálás) — **enélkül üres a Tétel sor** |
| `foxpost` | a választott Foxpost automata (a rendelés űrlapjáról) |
| `atvetel` | az átvétel módja: „Foxpost csomagautomata” vagy „Személyes átvétel — Budapest / Szeged / Baja, a szerzőnél” |
| `elallas_targy` | melyik rendelésről áll el a vevő (az elállási oldalról) |
| `elallas_idopont` | az elállás beérkezésének ideje, pl. `2026. 11. 30. 14:05` |

Enélkül a MailerLite csendben eldobja ezeket az értékeket.

**Ellenőrzés:** a mezők létrehozása után **új e-mail-címmel** teszteld
(pl. `valami+teszt2@gmail.com`) — a korábbi tesztcím már bent van a
szegmensben, arra az automatizálás nem indul újra. Ha a mezők létrehozása
után is üresen jönnek, a MailerLite űrlapszerkesztőjében add hozzá őket
az űrlaphoz (rejtett mezőként is elég).

A díjbekérőben (`dijbekero.html`) a végösszeg a **szállítással
együtt** szerepel (az oldal számolja ki: ár × darab + 1 800 Ft), és a cím
helyén a **Foxpost automata + számlázási cím** áll.

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

## 9. Az e-mail sablonok arculata (2026. szeptember)

Mindhárom sablon (`koszonto-varolista`, `dijbekero`, `elallas-visszaigazolas`) az oldal mostani arculatát
követi: borítókép (az elállás-visszaigazolás kivételével), „Szőke Tamás”,
kapitális cím arany „bőrbe”-vel, alcím, lent a kiadó adatai és link az
oldalra. A borítókép a `https://www.nemjopasztor.hu/borito-email.jpg`
címről töltődik be — ha a fájlt átnevezed vagy törlöd, a levelekből eltűnik.

Mindegyik a méretkorlát alatt van (a díjbekérő a legnagyobb, ~5,0 KB). Beillesztés továbbra is **Ctrl+Shift+V**-vel, utána teszt-levél.
