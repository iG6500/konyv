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

## 6. Korai fizetési lehetőség a listásoknak — ez NEM új automatizálás

Az oldal (`index.html`) `CONFIG.PREORDER_OPEN` értéke már október 22.
9:00-ra van állítva, a nyilvános `CONFIG.PUBLIC_OPEN` pedig október 26-ra.
Ez azt jelenti: **az oldal saját magától élesíti az előrendelést október
22-én** — bárki, aki aznap megnyitja a linket, tud rendelni és fizetni,
napokkal a nyilvános meghirdetés előtt.

A kódban erről ez áll (`index.html`, CONFIG blokk): *"az oldalon nincs
bejelentkezés, tehát nem tudja megkülönböztetni a listást a látogatótól
— a listás elsőbbség nem technikai zár, hanem az, hogy aznap csak ők
kapják meg a linket és a díjbekérőt e-mailben."*

Tehát a teendő nem egy automatizálás, hanem **egy időzített, egyszeri
kampány** (MailerLite: Campaigns → Regular campaign, NEM Automation):

1. Írj egy rövid levelet a várólista-szegmensnek (`Mód Equals varolista`)
   — "Nyitva az előrendelés, neked már most" + a live oldal linkje
   (`index.html` publikált URL-je).
2. Ütemezd **október 22. csütörtök 9:00**-ra (ugyanarra az időpontra,
   mint `CONFIG.PREORDER_OPEN` — ha ez a dátum változik, ezt az
   ütemezést is told el vele együtt).
3. Ne hirdesd sehol máshol (poszt, hirdetés) október 26. előtt — a
   "korai hozzáférés" kizárólag azon múlik, hogy addig csak a lista
   ismeri a linket.
4. Ha valaki a listáról dedikált példányra rendel, arra ugyanúgy
   vonatkozik a 2–3. pontban leírt automata díjbekérő-folyamat (Delay/Wait
   lépéssel), csak épp ő már csütörtökön be tud lépni a rendelésbe.
