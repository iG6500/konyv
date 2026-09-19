# Automata díjbekérő — beüzemelési útmutató

Ez a két fájl (`dijbekero-dedikalt.html`, `dijbekero-normal.html`) nem a
weboldal része, semmi nem hivatkozik rájuk élesben. Kizárólag arra valók,
hogy a teljes forráskódjukat bemásold a MailerLite Automation → Email
lépés → **"Code your own" / Custom HTML** szerkesztőjébe.

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

Az automatizálás triggerébe/szűrőjébe vedd fel:

```
mod = elorendeles
ÉS
valtozat = dedikalt   (a dijbekero-dedikalt.html-hez)
  vagy
valtozat = normal     (a dijbekero-normal.html-hez)
ÉS
sorszam_varolista ≠ igen
```

Az utolsó feltétel azért kritikus, mert ha valaki dedikáltat kért, amikor
épp nem volt szabad szám, ő csak **sorba állt**, nem kapott tényleges
példányt. Ha erre is automatikusan kimenne a díjbekérő, pénzt kérnél egy
könyvért, ami lehet, hogy nincs is neki.

Ha a sorszámot te magad rendeled hozzá utólag (a jelentkezés beérkezése
után választod ki, melyik konkrét kötet legyen az övé), előbb frissítsd a
subscriber "Példány számok" (`peldany_szamok`) mezőjét a MailerLite-ban,
csak utána engedd elmenni a levelet — így a `{$peldany_szamok}` a helyes,
végleges számot fogja mutatni.

## 4. Ha a tartalom megint csonkán jelenik meg beillesztés után

Ha az újraírt, egyszerűsített verzió is csonkán jelenne meg:

1. Próbáld **Ctrl+Shift+V** (beillesztés formázás nélkül) a sima Ctrl+V
   helyett — ha a MailerLite mezője gazdag szövegdoboz (nem sima
   szövegmező), a normál beillesztés néha megpróbálja "értelmezni" (azaz
   megjeleníteni) a HTML-t beillesztéskor ahelyett, hogy nyers szövegként
   kezelné, és csak a látható rész marad meg belőle.
2. A MailerLite saját dokumentációja szerint a Custom HTML importáláshoz
   nem csak beillesztés létezik: **ZIP-fájl behúzása** vagy **URL-ről
   importálás** is működik — ha a beillesztés továbbra sem megbízható,
   ezekkel érdemes próbálkozni.
3. Írd meg pontosan, hol szakad meg a tartalom (melyik sor/rész az
   utolsó, ami még megjelenik) — abból tovább lehet szűkíteni az okot.
