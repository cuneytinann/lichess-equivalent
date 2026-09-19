**[English](#chesscom-equivalent)** · **[Türkçe](#turkce)**

# chesscom-equivalent

A chess arbiter that plays by **Chess.com's rules** rather than FIDE's, in a single HTML file of **2,766 bytes** — and the same arbiter stripped of its board, in **1,144**. Two players, one screen. No libraries, no build step, no server. Download a file, double-click, play.

The two rulebooks are close, but they are not the same book, and every place they part ways is written down below.

Part of the [Golfstack](https://www.fidelite.art/) project.

## Play

| file | interface | bytes |
| --- | --- | --- |
| `index.html` | clickable board, clock, Chess.com colours | 2,766 |
| `numerical_packed.html` | square numbers typed into a `prompt()` box, no board | 1,144 |

On the project site both builds live under `special/outofLevels`, off to the side of the `L1`–`L3` ladder. They are not another rung on it; they follow a different rulebook.

Anything from late 2020 onwards will run them: Chrome 85+, Firefox 79+, Safari 14+. Three things set that floor — BigInt, which the starting position is written with; the `safe` keyword in `place-content`, which keeps the board reachable on a narrow screen; and the `||=` operator, which the packed build uses.

**On the board.** Click a piece, then click where it should go. Legal squares pick up a dot, a piece you can take picks up a ring, the square you selected, the square you came from and the square you landed on all take the same tint, and a king in check glows red. After every move the board turns around to face whoever is to play. `½` offers a draw or accepts one; `⚐` resigns. The clock starts at ten minutes and hands back five seconds a move.

**Without the board.** `numerical_packed.html` draws nothing at all. Squares are numbered 1 to 64, a1 through h8, and a move is the two numbers written end to end: e2–e4 is `1329`. To promote, add a fifth digit — `0` bishop, `1` rook, `2` knight, anything else queen. A draw is offered the same way, by hanging a non-digit on the end of the move: `1329x` plays e2–e4 with an offer attached. Type the non-digit on its own and the offer still stands, but you owe a move afterwards. The opponent accepts by answering in kind; a plain move declines it and wipes it off. To resign, leave the box empty or press Cancel.

That is the one place the two builds part company, and it is a difference of gesture rather than of rule. On the board a draw offer is a standing flag you raise with `½` and lower the same way; in the dialog it rides along with a move. Either way the arbiter reads the same two bits, and either way the opponent's plain move turns the offer down. The dialog title carries White's clock, Black's clock and the last move you got past the arbiter, so when an illegal move is quietly refused you will see that the last one never changed. Both clocks start at 900,000 milliseconds — fifteen minutes, no increment — and they keep running while the dialog is open, so thinking costs you what it would over the board. Keeping track of the position is your job.

## Two files, one arbiter

These are not a full version and a cut-down one. Same game, same rules, same core — and **1,144 bytes is what that core really costs.** Everything on top of it exists for you, not for chess.

The 1,622 bytes `index.html` spends beyond the packed file buy a board you can see, pieces you can click, a clock that ticks, a promotion picker, colours that tell you which squares you may use, and a status line. The arbiter underneath is the same arbiter.

## Why Chess.com and not FIDE

Chess has a rulebook, and then chess servers have rulebooks of their own. They agree on almost everything and disagree in a handful of interesting places. This arbiter follows Chess.com deliberately, and the decision was made by measuring its behaviour rather than by guessing at it — Chess.com's source is closed, so the evidence here is its own help pages plus 30,198 finished games pulled from its Published-Data API.

| ending | FIDE | Chess.com | here |
| --- | --- | --- | --- |
| checkmate, stalemate | — | — | ✓ |
| insufficient material | automatic | automatic, **different test** | automatic |
| threefold repetition | **claim** | **automatic** | **automatic** |
| fivefold repetition | automatic | *never reached* | *never reached* |
| fifty moves | **claim** | **automatic** | **automatic** |
| seventy-five moves | automatic | *never reached* | *never reached* |
| draw by agreement | — | — | ✓ |
| flag fall | loss | loss | loss |
| flag fall, opponent cannot mate | **draw, helpmate test** | **draw, material test** | **draw, material test** |
| resignation | loss | loss | loss |
| resignation, opponent cannot mate | **draw** | **loss** | **loss** |

The bold rows are where the interesting part lives.

**Threefold and fifty moves.** FIDE hands both to the player as a claim and has the arbiter declare them unasked at fivefold and at seventy-five. Chess.com does not wait: the third repetition and the hundredth ply end the game on their own. Which means neither the fivefold rule nor the seventy-five-move rule ever gets a chance to fire, and neither has any reason to exist here. There is no claim channel in this file at all — the draw control offers and accepts, nothing else.

**Resigning against a bare king.** Article 5.1.2 says a resignation loses the game *unless* the opponent could not deliver mate by any series of legal moves, in which case it is a draw. That clause entered the handbook in 2023. Chess.com does not apply it: resignation is a loss, full stop. So the `RM` code that the FIDE and lichess builds carry does not exist here, and `F` is not a function — the resign button writes the result directly.

**Running out of time.** Article 6.9 asks whether the opponent could mate *by any series of legal moves*, the losing side helping. Chess.com asks a narrower question: strip the flagged player's army down to a bare king, and ask whether the surviving player's material could mate **that**. A knight against a rook mates under FIDE, because the rook can be walled in; against a bare king it cannot. So the same position is a win on one rulebook and a draw on the other, and the draw is what this file returns as `TM`. Two real games from the scan, both from 2026:

```
3k4/5R2/3PPn2/5K2/8/8/8/8 w - - 3 67        White R+2P flags, Black has K+N   -> draw
8/8/6K1/5Np1/5b2/1r2kq2/8/8 b - - 3 81      White K+N survives Q+R+B+P       -> draw
```

**The two tests are not the same test.** The material question gets asked twice, once on the board and once at flag fall, and Chess.com uses a different threshold each time. On the board it asks whether mate can be **forced**; at flag fall, whether mate is **possible at all**. King and two knights is the one material that answers differently: it cannot force mate, so against a bare king the game ends on the spot as `IM` — but it can construct one, so a player who flags against it loses. Chess.com's own help pages say both, in two separate articles, and the scan confirms both. Five games where king and two knights won on time, five where it drew immediately against a lone king.

**Multiple bishops and square colour.** Older reports, the most recent from December 2024, say Chess.com counted any two bishops as mating material and ignored which squares they stood on. That is no longer true. Tested directly in 2026: king and two dark-squared bishops against a bare king is declared drawn at once, while opposite-coloured bishops play on. The file does the same — the per-side test reads the bishops' square colours, and any number of bishops confined to one colour cannot force mate.

**Two knights against a piece.** The other half of the two-knight rule is easy to get wrong. King and two knights draws immediately against a **lone** king only; give that king anything at all to be walled in with and the game continues, because a forced mate exists. Here that falls out of a threshold rather than a branch: a side that cannot force mate is weighed — nought for a bare king, one for a single minor or for any number of same-coloured bishops, two for a pair of knights — and the game is drawn when the two weights together come to less than three.

**Blocked positions.** A position can be dead with pieces to spare: the pawns interlock, and mate becomes impossible even if both players set out to arrange one. Article 5.2.2 calls that a draw on the spot. Chess.com does not look for it, and neither does this file — both play on until repetition or the fifty-move counter closes the game instead. Taking that detector out is most of what separates this build from its FIDE sibling on the project site.

**En passant, and the mistake nearly everyone makes.** For repetition, FIDE counts two positions as the same only if a legal en passant capture is available in both or in neither. Plenty of engines write the en passant square after every double pawn push, whether or not anything can actually take. Nothing looks broken, because the capture gets refused anyway — but the repetition key comes out different, and a threefold that should have triggered arrives late or never arrives at all. Chess.com is one of those engines, and the evidence is in its own output: of 244 scanned positions carrying an en passant square, **234 had no enemy pawn anywhere near it.** The plainest of them is an opening:

```
rnbqkbnr/ppp1pppp/8/3p4/3P4/5N2/PPP1PPPP/RNBQKB1R b KQkq d3 0 2
```

White has just played d2–d4 and `d3` is recorded, though Black has nothing on c4 or e4 to take with. So this file writes the square unconditionally too, and the legality filter that the FIDE and lichess builds carry — a scan of the neighbouring squares, and a call into the legal move generator to confirm the capture — is gone. It was worth 62 bytes, and its removal takes one level of recursion out of `M` along with it.

*Measured in September 2026 against Chess.com's help centre and 30,198 of its finished games. Chess.com can change its mind later; this file cannot.*

## Result codes

Twelve ways a game can end: six decisive, six drawn.

| | | | |
| --- | --- | --- | --- |
| `W#` White mates | `B#` Black mates | `SM` stalemate | `IM` insufficient material |
| `WT` White wins on time | `BT` Black wins on time | `TM` flag fall, mate impossible | `50` fifty-move rule |
| `WR` White wins by resignation | `BR` Black wins by resignation | `3R` threefold repetition | `DA` draw by agreement |

A code beginning with `W` is a win for White and one beginning with `B` a win for Black; everything ending in `M`, beginning with `D`, or carrying a digit is a draw. `numerical_packed.html` reports the same twelve endings as numbers, with `4`, `5` and `15` left unused — those slots belonged to the seventy-five-move rule, fivefold repetition and resignation against a bare king.

The FIDE build on the project site has fifteen codes and calls insufficient material `DP`, dead position, because it detects blocked ones as well. Here the code is `IM`, which names what the file actually tests.

## What's in it

- **Every piece's movement**, worked out with arithmetic. No direction tables, no offset arrays.
- **Full legality.** A move that would leave your own king in check is never let through.
- **Castling** on both wings, with all of it checked: the right still standing, the rook's path clear, the king not in check, not crossing an attacked square, not landing on one.
- **En passant**, with Chess.com's unconditional repetition key.
- **Promotion** to queen, rook, bishop or knight — a picker on the board, a fifth digit without it.
- **A running clock.** Ten minutes plus five seconds a move on the board, fifteen flat minutes in the dialog.
- **Draw offers**, resignation, flag fall, and the material test that turns a flag fall into a draw.
- **All twelve endings**, told apart from one another.

## What's not in it

- No claim channel. Threefold and fifty moves fire on their own, so there is nothing to claim.
- No blocked-position detection, as above. Chess.com has none either.
- No helpmate search at flag fall, and no draw on resignation. Chess.com has neither.
- No engine, no takebacks, no FEN in or out, no PGN.
- No coordinates around the board. It turns around every ply, and the status line keeps out of the way.

For the full FIDE arbiter — fifteen codes, dead positions, eight front ends — see [fidelite.art](https://www.fidelite.art/). For the lichess rulebook, see [lichess-equivalent](https://github.com/cuneytinann/lichess-equivalent).

## Colours

Sampled from screenshots of a live game rather than taken from a stylesheet, then checked back against the pixels.

| | | |
| --- | --- | --- |
| light square | `#EBECD0` | |
| dark square | `#739552` | |
| page behind the board | `#302E2B` | |
| highlight | `#FFFF33` at 50% | `#F5F681` on light, `#B9CA42` on dark |
| legal move | black at 13% | a dot at 22% of the square, a ring from 49% to 61% for a capture |
| king in check | red, fading out | |

Two of these are shorter than what they replace. Chess.com paints the square you selected and the two squares of the last move in **the same** yellow, where lichess uses two different greens — one ternary instead of two. And the move dot is a flat black tint, where lichess tints it green with its own alpha. The ring costs more: lichess fills the corners of the square, Chess.com draws a true annulus, which needs two more colour stops.

The alpha is written as `#FF38`, four-digit hex, rather than `#FFFF3380`. That puts the alpha at 0.533 instead of 0.5 and every channel lands within five of the measured value — four bytes for a difference no eye will find.

## Read it in the browser

`index.html` is not packed and not minified. **Right-click → View Page Source** (`Ctrl` `U`, or `⌥` `⌘` `U` on macOS) puts the whole program on screen, and that is what this repository is for: open the page, play a few moves, then go and read the source that just refereed them.

`numerical_packed.html` is the exception. It unpacks itself, and there is a section further down on how to open it up.

---

## Anatomy

Every byte of `index.html`, by part.

| part | bytes | |
| --- | --- | --- |
| markup and CSS | 648 | board, panel, picker, colours, layout |
| `<script>` tags | 17 | |
| state and aliases | 160 | board, clock, castling rights, repetition table |
| `G` | 264 | can this piece reach that square |
| `V` | 51 | is this square attacked |
| `L` | 101 | is this move legal — play it, ask, take it back |
| `C` | 28 | which castling right a square forfeits |
| `M` | 173 | counter, promotion, en passant victim, rook hop, en passant square |
| `I` | 97 | material, and whether mate can be forced |
| `Z` | 96 | the verdict |
| `j` | 88 | the clock, and the flag-fall material test |
| setup | 162 | picker and the 64 cells, generated at load |
| `d` | 560 | draw the board |
| `A`, `Bt`, `S` | 290 | play the move, the buttons, the click |
| first draw and interval | 31 | |
| **total** | **2,766** | |

Sliced the other way: the rules come to **970** bytes and the page that shows them to **1,796**. The referee is cheap and the stage is expensive, which is exactly the argument the packed file makes.

Three functions that the FIDE and lichess builds need are missing here, and none of them was shortened — they were deleted. `H`, the flag-fall material test, collapsed into a second reading of `I`'s own counters. `F`, which turned a resignation or a flag fall into a result, had nothing left to decide once resignation became an unconditional loss. `D`, the draw offer, kept only its offer-and-accept half; the claim half went with the claims.

## Verification

Both builds were checked by running them, and the rule layer was checked against Chess.com itself.

- **Against real games.** 30,198 finished games were pulled from Chess.com's Published-Data API, spanning October 2025 to September 2026, and every one of them was classified by the material standing in its final position and the result Chess.com recorded. About 2,400 immediate insufficient-material rulings and a thousand flag-fall rulings, and the file's verdict matches on every single class. The material combinations that Chess.com ends on the spot are exactly the combinations this file calls drawn, and the ones it plays on through — bishop and knight, opposite-coloured bishops, three knights, two bishops and a knight — are exactly the ones this file calls sufficient.
- **Against Chess.com directly.** The same-coloured bishop case appears in none of those 30,198 games, so it was set up by hand on the site and played out: two dark-squared bishops against a bare king is drawn at once, opposite-coloured bishops are not, one bishop each is drawn whatever the colours.
- **The en passant convention**, read off 244 scanned positions that carried an en passant square.
- **The material test**, twenty-three cases on the board layer, covering both thresholds and every combination that separates them, including the ones no real game produced.
- **Scripted games on both builds**, in Node against a stubbed `prompt()`: checkmate, the automatic threefold, resignation, an offer carried on a move and accepted, and an en passant capture made after the square had been written unconditionally.

The move generator is untouched. The only change inside `M` is which square goes into the repetition key, and that square cannot add a move: the only piece that can reach it is the neighbouring pawn that holds the en passant right in the first place, and a pinned one is still thrown out by `L`. The perft figures from the FIDE build therefore carry over unchanged, but they have not been re-run on this file, and neither has the markup validation nor the cell-by-cell rendering check.

## Unpacking

`numerical_packed.html` unpacks itself. Its script is a RegPack decompression loop that ends in `eval(_)`, so to get the plain source back you replace that one call:

```js
eval(_)   →   console.log(_)
```

Nothing in the loop touches the game, so it is safe to do this in Node. What falls out is 1,293 bytes of source.

**Download the file rather than copying it out of the browser.** The dictionary keys are control characters from the `\x01`–`\x1f` range, and the clipboard — or any editor that tidies up line endings — will quietly destroy them.

## Packing

[RegPack 5.0.1](https://github.com/Siorki/RegPack). These settings rebuild `numerical_packed.html` from that source **byte for byte**:

| option | value |
| --- | --- |
| `reassignVars` | `false` |
| `crushGainFactor` | `0.5` |
| `crushLengthFactor` | `0.5` |
| `crushCopiesFactor` | `0` |
| `crushTiebreakerFactor` | `0` |
| `useES6` | `true` |

Stage 2 wins, the regexp character class: `[\x01-\x1f@-Bj_ZX]`, 38 tokens. The bytes land like this:

```
   8 B  <script>
1127 B  packed payload
   9 B  </script>
----
1144 B
```

Turning `reassignVars` on saves four bytes and brings the file down to 1,140. It stays off, for the same reason it stays off in the lichess build: the renamer spends `P` through `V` as dictionary tokens, so the source that comes back out has had its variables shuffled and no longer reads as the program anyone wrote. Four bytes do not buy that back.

The half-value crusher factors are worth three bytes over RegPack's own defaults and over the `1/0/0` that its README recommends, both of which land on 1,131. The token set barely moves between them; the difference is which of two near-equal candidates wins the last few substitution rounds.

### The packed file was not built from the shortest source

The source above runs to 1,293 bytes, and it is the lichess source with the rules swapped, not a fresh one. The same trade-offs apply: aliases and shared functions win in plain form and lose under the packer, because RegPack is paid in repeated substrings and an alias is precisely the thing that takes repetition away. The verdict expression is written out twice on purpose rather than being given a name, because the crusher turns the second copy into a single token and charges less for it than a function would cost.

One of those duplicates went away with the claims, though — the draw channel used to re-evaluate the verdict, and now it does not. That is a rule change, not a packing decision, and it is the reason the plain source dropped 181 bytes while the packed file dropped only 84.

---

## Related

- [FideLite](https://github.com/cuneytinann/FideLite) · [fidelite.art](https://www.fidelite.art/) — the full FIDE arbiter, fifteen result codes, eight front ends
- [lichess-equivalent](https://github.com/cuneytinann/lichess-equivalent) — the same exercise, one server over
- [Chess LUX](https://github.com/cuneytinann/Chess_LUX) — the other direction entirely: dead positions carried to 99.97%
- [chessarbiter2kb](https://github.com/cuneytinann/chessarbiter2kb) — the same rules a level down, letters and numbers side by side
- [chess1023byte](https://github.com/cuneytinann/chess1023byte) — the bare rules, packed, in 1,023 bytes

## License

MIT

---
---

<a id="turkce"></a>

# chesscom-equivalent (Türkçe)

FIDE'nin değil, **Chess.com'un kurallarıyla** oynayan bir satranç hakemi; tek bir HTML dosyasında **2.766 bayt** — ve aynı hakemin tahtasından soyulmuş hâli, **1.144** baytta. İki oyuncu, tek ekran. Kütüphane yok, derleme adımı yok, sunucu yok. Dosyayı indir, çift tıkla, oyna.

İki kural kitabı birbirine yakın ama aynı kitap değil, ve ayrıldıkları her yer aşağıda yazılı.

[Golfstack](https://www.fidelite.art/) projesinin bir parçası.

## Oyna

| dosya | arayüz | bayt |
| --- | --- | --- |
| `index.html` | tıklanabilir tahta, saat, Chess.com renkleri | 2.766 |
| `numerical_packed.html` | `prompt()` kutusuna yazılan kare numaraları, tahta yok | 1.144 |

Proje sitesinde iki yapı da `special/outofLevels` altında, `L1`–`L3` merdiveninin yanında duruyor. Merdivenin bir basamağı değiller; başka bir kural kitabını takip ediyorlar.

2020 sonundan itibaren her şey çalıştırır: Chrome 85+, Firefox 79+, Safari 14+. Bu tabanı üç şey belirliyor — başlangıç pozisyonunun yazıldığı BigInt; dar ekranda tahtayı erişilebilir tutan `place-content` içindeki `safe` anahtar sözcüğü; ve paketli yapının kullandığı `||=` operatörü.

**Tahtada.** Bir taşa tıkla, sonra gideceği kareye tıkla. Legal kareler nokta alır, alabileceğin taş halka alır, seçtiğin kare, geldiğin kare ve gittiğin kare aynı rengi alır, şahı tehdit altındaki taraf kırmızı parlar. Her hamleden sonra tahta sırası gelene dönüyor. `½` beraberlik teklif eder veya kabul eder; `⚐` terk eder. Saat on dakikadan başlar ve her hamlede beş saniye geri verir.

**Tahtasız.** `numerical_packed.html` hiçbir şey çizmiyor. Kareler a1'den h8'e 1–64 arası numaralı, hamle iki numaranın uç uca yazılmışı: e2–e4 `1329`. Terfi için beşinci bir rakam ekle — `0` fil, `1` kale, `2` at, başka her şey vezir. Beraberlik de aynı yoldan teklif edilir, hamlenin sonuna rakam olmayan bir karakter asarak: `1329x` e2–e4 oynar ve teklifi yanında taşır. Rakam olmayanı tek başına yazarsan teklif yine durur ama sonrasında bir hamle borçlusun. Rakip aynı şekilde cevap vererek kabul eder; düz bir hamle teklifi reddeder ve siler. Terk için kutuyu boş bırak ya da Cancel'a bas.

İki yapının ayrıldığı tek yer burası, ve bu bir kural farkı değil jest farkı. Tahtada beraberlik teklifi `½` ile kaldırıp aynı şekilde indirdiğin duran bir bayrak; diyalogda hamleyle birlikte yolculuk ediyor. Her iki hâlde de hakem aynı iki biti okuyor, her iki hâlde de rakibin düz hamlesi teklifi reddediyor. Diyalog başlığı Beyaz'ın saatini, Siyah'ın saatini ve hakemden geçirebildiğin son hamleyi taşıyor, yani illegal bir hamle sessizce reddedildiğinde sonuncunun hiç değişmediğini görürsün. İki saat de 900.000 milisaniyeden başlıyor — on beş dakika, artırım yok — ve diyalog açıkken işlemeye devam ediyorlar, yani düşünmek sana tahta başındaki kadara mal oluyor. Pozisyonu takip etmek senin işin.

## İki dosya, tek hakem

Bunlar tam sürüm ve kırpılmış sürüm değil. Aynı oyun, aynı kurallar, aynı çekirdek — ve **o çekirdeğin gerçek maliyeti 1.144 bayt.** Üstündeki her şey satranç için değil, senin için var.

`index.html`'in paketli dosyanın ötesinde harcadığı 1.622 bayt şunları satın alıyor: görebildiğin bir tahta, tıklayabildiğin taşlar, işleyen bir saat, terfi seçici, hangi kareleri kullanabileceğini söyleyen renkler ve bir durum satırı. Altındaki hakem aynı hakem.

## Neden Chess.com, neden FIDE değil

Satrancın bir kural kitabı var, bir de satranç sunucularının kendi kural kitapları. Neredeyse her şeyde anlaşıyor, bir avuç ilginç yerde ayrılıyorlar. Bu hakem bilerek Chess.com'u takip ediyor, ve karar davranışını tahmin ederek değil ölçerek verildi — Chess.com'un kaynağı kapalı, o yüzden buradaki kanıt kendi yardım sayfaları artı Published-Data API'sinden çekilmiş 30.198 bitmiş oyun.

| bitiş | FIDE | Chess.com | burada |
| --- | --- | --- | --- |
| mat, pat | — | — | ✓ |
| yetersiz malzeme | otomatik | otomatik, **farklı test** | otomatik |
| üçlü tekrar | **iddia** | **otomatik** | **otomatik** |
| beşli tekrar | otomatik | *hiç ulaşılmaz* | *hiç ulaşılmaz* |
| elli hamle | **iddia** | **otomatik** | **otomatik** |
| yetmiş beş hamle | otomatik | *hiç ulaşılmaz* | *hiç ulaşılmaz* |
| anlaşmayla beraberlik | — | — | ✓ |
| bayrak düşmesi | kayıp | kayıp | kayıp |
| bayrak düşmesi, rakip mat edemiyor | **beraberlik, yardımlı mat testi** | **beraberlik, malzeme testi** | **beraberlik, malzeme testi** |
| terk | kayıp | kayıp | kayıp |
| terk, rakip mat edemiyor | **beraberlik** | **kayıp** | **kayıp** |

Kalın satırlar işin ilginç kısmının yaşadığı yer.

**Üçlü tekrar ve elli hamle.** FIDE ikisini de oyuncuya iddia olarak veriyor ve beşli tekrarda, yetmiş beş hamlede hakemin sormadan ilan etmesini istiyor. Chess.com o kadar beklemiyor: üçüncü tekrar ve yüzüncü yarım hamle oyunu kendiliğinden bitiriyor. Bu da beşli tekrarın ve yetmiş beş hamle kuralının hiç ateşlenme fırsatı bulamaması demek, ve ikisinin de burada var olması için bir sebep kalmıyor. Bu dosyada iddia kanalı diye bir şey hiç yok — beraberlik kontrolü teklif ediyor ve kabul ediyor, başka bir şey yapmıyor.

**Çıplak şaha karşı terk etmek.** 5.1.2. madde, terk etmenin oyunu kaybettirdiğini söylüyor — *rakip legal hamleler dizisiyle mat edemiyorsa* beraberlik olması dışında. Bu cümle kitaba 2023'te girdi. Chess.com uygulamıyor: terk kayıptır, nokta. Dolayısıyla FIDE ve lichess yapılarının taşıdığı `RM` kodu burada yok, ve `F` bir fonksiyon değil — terk düğmesi sonucu doğrudan yazıyor.

**Sürenin bitmesi.** 6.9. madde rakibin *herhangi bir legal hamleler dizisiyle* mat edip edemeyeceğini soruyor, kaybeden taraf yardım ederek. Chess.com daha dar bir soru soruyor: bayrağı düşenin ordusunu çıplak şaha indir, ve hayatta kalan tarafın malzemesinin **onu** mat edip edemeyeceğini sor. Bir at, kaleye karşı FIDE'de mat eder, çünkü kale kendini kapatabilir; çıplak şaha karşı edemez. Yani aynı pozisyon bir kural kitabında kazanç, diğerinde beraberlik, ve bu dosyanın `TM` olarak döndürdüğü beraberlik. Taramadan iki gerçek oyun, ikisi de 2026'dan:

```
3k4/5R2/3PPn2/5K2/8/8/8/8 w - - 3 67        Beyaz K+2P bayrak düşürüyor, Siyah'ta Ş+A  -> beraberlik
8/8/6K1/5Np1/5b2/1r2kq2/8/8 b - - 3 81      Beyaz Ş+A, karşısında V+K+F+P              -> beraberlik
```

**İki test aynı test değil.** Malzeme sorusu iki kez soruluyor, bir kez tahtada bir kez bayrak düştüğünde, ve Chess.com her seferinde farklı bir eşik kullanıyor. Tahtada matın **zorlanabilir** olup olmadığını soruyor; bayrakta matın **hiç mümkün** olup olmadığını. Şah ve iki at, farklı cevap veren tek malzeme: mat zorlayamıyor, o yüzden çıplak şaha karşı oyun orada `IM` ile bitiyor — ama mat kurabiliyor, o yüzden ona karşı bayrak düşüren kaybediyor. Chess.com'un kendi yardım sayfaları ikisini de söylüyor, iki ayrı makalede, ve tarama ikisini de doğruluyor. Şah ve iki atın süreden kazandığı beş oyun, yalın şaha karşı anında beraberlik verdiği beş oyun.

**Birden fazla fil ve kare rengi.** Eski raporlar, en yenisi Aralık 2024'ten, Chess.com'un herhangi iki fili mat malzemesi saydığını ve hangi karelerde durduklarına bakmadığını söylüyor. Bu artık doğru değil. 2026'da doğrudan test edildi: şah ve iki koyu kare fili, çıplak şaha karşı anında beraberlik ilan ediliyor, zıt renk filler ise oynamaya devam ediyor. Dosya da aynısını yapıyor — taraf başına test fillerin kare renklerini okuyor, ve tek bir renge hapsolmuş kaç fil olursa olsun mat zorlayamıyor.

**İki at, bir taşa karşı.** İki at kuralının diğer yarısını yanlış yapmak kolay. Şah ve iki at yalnızca **yalın** şaha karşı anında beraberlik; o şaha kendini kapatacak herhangi bir şey ver, oyun devam ediyor, çünkü zorunlu bir mat var. Burada bu bir daldan değil bir eşikten çıkıyor: mat zorlayamayan taraf tartılıyor — çıplak şah sıfır, tek bir hafif taş veya aynı renkte kaç fil olursa olsun bir, at çifti iki — ve iki ağırlık birlikte üçten küçük kalınca oyun beraberlikle bitiyor.

**Bloke pozisyonlar.** Bir pozisyon, taşlar fazlasıyla yerindeyken de ölü olabilir: piyonlar kenetlenir, ve iki oyuncu da mat kurmaya çalışsa bile mat imkânsız hâle gelir. 5.2.2. madde buna anında beraberlik diyor. Chess.com bunu aramıyor, bu dosya da aramıyor — ikisi de oyunu tekrar veya elli hamle sayacı kapatana kadar sürdürüyor. O dedektörü çıkarmak, bu yapıyı proje sitesindeki FIDE kardeşinden ayıran şeyin büyük kısmı.

**En passant, ve neredeyse herkesin yaptığı hata.** Tekrar için FIDE iki pozisyonu ancak legal bir en passant alışı ikisinde de varsa veya ikisinde de yoksa aynı sayıyor. Birçok motor en passant karesini her çift piyon adımından sonra, gerçekten alabilecek bir şey olsun olmasın yazıyor. Hiçbir şey bozuk görünmüyor, çünkü alış zaten reddediliyor — ama tekrar anahtarı farklı çıkıyor, ve tetiklenmesi gereken bir üçlü tekrar geç geliyor ya da hiç gelmiyor. Chess.com o motorlardan biri, ve kanıt kendi çıktısında: en passant karesi taşıyan 244 taranmış pozisyonun **234'ünde o karenin yakınında hiç düşman piyonu yoktu.** En sadesi bir açılış:

```
rnbqkbnr/ppp1pppp/8/3p4/3P4/5N2/PPP1PPPP/RNBQKB1R b KQkq d3 0 2
```

Beyaz henüz d2–d4 oynadı ve `d3` kaydedilmiş, oysa Siyah'ın alacak hiçbir şeyi yok, ne c4'te ne e4'te. Yani bu dosya da kareyi koşulsuz yazıyor, ve FIDE ile lichess yapılarının taşıdığı legallik filtresi — komşu karelerin taranması, ve alışı doğrulamak için legal hamle üretecine yapılan çağrı — kalktı. 62 bayt ediyordu, ve kalkmasıyla `M`'den bir seviye özyineleme de gitti.

*Eylül 2026'da Chess.com'un yardım merkezi ve 30.198 bitmiş oyunu üzerinden ölçüldü. Chess.com sonradan fikrini değiştirebilir; bu dosya değiştiremez.*

## Sonuç kodları

Bir oyunun bitebileceği on iki yol: altısı sonuçlu, altısı beraberlik.

| | | | |
| --- | --- | --- | --- |
| `W#` Beyaz mat eder | `B#` Siyah mat eder | `SM` pat | `IM` yetersiz malzeme |
| `WT` Beyaz süreden kazanır | `BT` Siyah süreden kazanır | `TM` bayrak düştü, mat imkânsız | `50` elli hamle kuralı |
| `WR` Beyaz terkle kazanır | `BR` Siyah terkle kazanır | `3R` üçlü tekrar | `DA` anlaşmayla beraberlik |

`W` ile başlayan kod Beyaz'ın kazancı, `B` ile başlayan Siyah'ın; `M` ile biten, `D` ile başlayan veya içinde rakam taşıyan her şey beraberlik. `numerical_packed.html` aynı on iki bitişi sayı olarak bildiriyor; `4`, `5` ve `15` boş duruyor — o yuvalar yetmiş beş hamle kuralına, beşli tekrara ve çıplak şaha karşı terke aitti.

Proje sitesindeki FIDE yapısının on beş kodu var ve yetersiz malzemeye `DP`, ölü pozisyon diyor, çünkü bloke olanları da tespit ediyor. Burada kod `IM`, yani dosyanın gerçekten test ettiği şeyin adı.

## İçinde neler var

- **Her taşın hareketi**, aritmetikle çözülmüş. Yön tablosu yok, ofset dizisi yok.
- **Tam legallik.** Kendi şahını tehdit altında bırakacak bir hamle asla geçmiyor.
- **Rok**, iki kanatta da, hepsi kontrol edilerek: hakkın hâlâ ayakta olması, kalenin yolunun boş olması, şahın tehdit altında olmaması, tehdit edilen bir kareden geçmemesi, tehdit edilen bir kareye inmemesi.
- **En passant**, Chess.com'un koşulsuz tekrar anahtarıyla.
- **Terfi**: vezir, kale, fil veya at — tahtada bir seçici, tahtasız bir beşinci rakam.
- **İşleyen bir saat.** Tahtada on dakika artı hamle başına beş saniye, diyalogda düz on beş dakika.
- **Beraberlik teklifleri**, terk, bayrak düşmesi, ve bayrak düşmesini beraberliğe çeviren malzeme testi.
- **On iki bitişin hepsi**, birbirinden ayrılmış hâlde.

## İçinde neler yok

- İddia kanalı yok. Üçlü tekrar ve elli hamle kendiliğinden ateşleniyor, iddia edilecek bir şey kalmıyor.
- Bloke pozisyon tespiti yok, yukarıdaki gibi. Chess.com'da da yok.
- Bayrak düştüğünde yardımlı mat araması yok, terkte beraberlik yok. Chess.com'da ikisi de yok.
- Motor yok, geri alma yok, FEN girişi veya çıkışı yok, PGN yok.
- Tahtanın çevresinde koordinat yok. Her yarım hamlede dönüyor, ve durum satırı yoldan çekiliyor.

Tam FIDE hakemi için — on beş kod, ölü pozisyonlar, sekiz ön yüz — [fidelite.art](https://www.fidelite.art/) adresine bak. Lichess kural kitabı için [lichess-equivalent](https://github.com/cuneytinann/lichess-equivalent).

## Renkler

Bir stil dosyasından alınmadı, canlı bir oyunun ekran görüntülerinden örneklendi ve sonra piksellere karşı geri kontrol edildi.

| | | |
| --- | --- | --- |
| açık kare | `#EBECD0` | |
| koyu kare | `#739552` | |
| tahtanın arkasındaki sayfa | `#302E2B` | |
| vurgu | `#FFFF33`, %50 | açıkta `#F5F681`, koyuda `#B9CA42` |
| legal hamle | siyah, %13 | karenin %22'sinde nokta, alışta %49'dan %61'e halka |
| şahı tehdit altında | kırmızı, dışa doğru sönen | |

Bunların ikisi, yerini aldıkları şeyden kısa. Chess.com seçtiğin kareyi ve son hamlenin iki karesini **aynı** sarıyla boyuyor, oysa lichess iki farklı yeşil kullanıyor — iki ternary yerine bir tane. Hamle noktası da düz bir siyah tonlama, lichess onu kendi alfasıyla yeşile boyuyor. Halka ise pahalıya geliyor: lichess karenin köşelerini dolduruyor, Chess.com gerçek bir halka çiziyor, bu da iki renk durağı daha istiyor.

Alfa `#FFFF3380` yerine dört haneli `#FF38` olarak yazıldı. Bu alfayı 0,5 yerine 0,533 yapıyor ve her kanal ölçülen değerin beş birim içinde kalıyor — hiçbir gözün bulamayacağı bir fark için dört bayt.

## Tarayıcıda okuyun

`index.html` paketli değil ve küçültülmüş değil. **Sağ tık → Sayfa Kaynağını Görüntüle** (`Ctrl` `U`, macOS'ta `⌥` `⌘` `U`) bütün programı ekrana getiriyor, ve bu deponun varlık sebebi tam olarak bu: sayfayı aç, birkaç hamle oyna, sonra git az önce onları yöneten kaynağı oku.

`numerical_packed.html` istisna. Kendini açıyor, ve aşağıda nasıl açılacağına dair bir bölüm var.

---

## Anatomi

`index.html`'in her baytı, parça parça.

| parça | bayt | |
| --- | --- | --- |
| işaretleme ve CSS | 648 | tahta, panel, seçici, renkler, yerleşim |
| `<script>` etiketleri | 17 | |
| durum ve takma adlar | 160 | tahta, saat, rok hakları, tekrar tablosu |
| `G` | 264 | bu taş o kareye ulaşabilir mi |
| `V` | 51 | bu kare tehdit altında mı |
| `L` | 101 | bu hamle legal mi — oyna, sor, geri al |
| `C` | 28 | bir karenin hangi rok hakkını düşürdüğü |
| `M` | 173 | sayaç, terfi, en passant kurbanı, kale sıçraması, en passant karesi |
| `I` | 97 | malzeme, ve matın zorlanabilir olup olmadığı |
| `Z` | 96 | hüküm |
| `j` | 88 | saat, ve bayrak düşmesinin malzeme testi |
| kurulum | 162 | seçici ve 64 hücre, yüklenirken üretiliyor |
| `d` | 560 | tahtayı çiz |
| `A`, `Bt`, `S` | 290 | hamleyi oyna, düğmeler, tıklama |
| ilk çizim ve interval | 31 | |
| **toplam** | **2.766** | |

Diğer yönden dilimlenince: kurallar **970** bayta, onları gösteren sayfa **1.796** bayta çıkıyor. Hakem ucuz, sahne pahalı — paketli dosyanın öne sürdüğü argüman da tam olarak bu.

FIDE ve lichess yapılarının ihtiyaç duyduğu üç fonksiyon burada eksik, ve hiçbiri kısaltılmadı — silindiler. Bayrak düşmesinin malzeme testi `H`, `I`'nin kendi sayaçlarının ikinci kez okunmasına çöktü. Terk veya bayrak düşmesini sonuca çeviren `F`, terk koşulsuz kayıp hâline gelince karar verecek bir şey bulamadı. Beraberlik teklifi `D`, yalnızca teklif-ve-kabul yarısını korudu; iddia yarısı iddialarla birlikte gitti.

## Doğrulama

İki yapı da okunarak değil çalıştırılarak kontrol edildi, ve kural katmanı Chess.com'un kendisine karşı sınandı.

- **Gerçek oyunlara karşı.** Chess.com'un Published-Data API'sinden Ekim 2025 ile Eylül 2026 arasını kapsayan 30.198 bitmiş oyun çekildi, ve her biri son pozisyonundaki malzemeye ve Chess.com'un kaydettiği sonuca göre sınıflandırıldı. Yaklaşık 2.400 anında yetersiz malzeme hükmü ve bin kadar bayrak hükmü, ve dosyanın kararı her sınıfta tutuyor. Chess.com'un anında bitirdiği malzeme kombinasyonları tam olarak bu dosyanın beraberlik dediği kombinasyonlar, ve sürdürdükleri — fil ve at, zıt renk filler, üç at, iki fil ve bir at — tam olarak bu dosyanın yeterli dediği kombinasyonlar.
- **Doğrudan Chess.com'a karşı.** Aynı renk fil durumu o 30.198 oyunun hiçbirinde geçmiyor, o yüzden sitede elle kurulup oynatıldı: iki koyu kare fili çıplak şaha karşı anında beraberlik, zıt renk filler değil, her tarafta birer fil renkler ne olursa olsun beraberlik.
- **En passant kuralı**, en passant karesi taşıyan 244 taranmış pozisyondan okundu.
- **Malzeme testi**, tahta katmanında yirmi üç durum; iki eşiği de ve onları ayıran her kombinasyonu kapsıyor, hiçbir gerçek oyunun üretmediği olanlar dahil.
- **İki yapıda da senaryolu oyunlar**, Node'da sahte bir `prompt()` karşısında: mat, otomatik üçlü tekrar, terk, hamleyle taşınıp kabul edilen bir teklif, ve kare koşulsuz yazıldıktan sonra yapılan bir en passant alışı.

Hamle üreteci el değmemiş durumda. `M`'nin içindeki tek değişiklik tekrar anahtarına hangi karenin girdiği, ve o kare bir hamle ekleyemez: ona ulaşabilecek tek taş, en passant hakkını zaten elinde tutan komşu piyon, ve açmazdaki bir piyon yine `L` tarafından eleniyor. FIDE yapısının perft rakamları bu yüzden değişmeden geçerli, ama bu dosya üzerinde yeniden koşulmadılar; işaretleme doğrulaması ve hücre hücre görüntüleme kontrolü de koşulmadı.

## Paketi açma

`numerical_packed.html` kendini açıyor. Betiği `eval(_)` ile biten bir RegPack açma döngüsü, yani düz kaynağı geri almak için o tek çağrıyı değiştiriyorsun:

```js
eval(_)   →   console.log(_)
```

Döngüde oyuna dokunan hiçbir şey yok, o yüzden bunu Node'da yapmak güvenli. Dökülen şey 1.293 baytlık kaynak.

**Dosyayı tarayıcıdan kopyalamak yerine indir.** Sözlük anahtarları `\x01`–`\x1f` aralığından kontrol karakterleri, ve pano — ya da satır sonlarını düzelten herhangi bir editör — onları sessizce yok eder.

## Paketleme

[RegPack 5.0.1](https://github.com/Siorki/RegPack). Şu ayarlar `numerical_packed.html`'i o kaynaktan **bayt bayt** yeniden kuruyor:

| seçenek | değer |
| --- | --- |
| `reassignVars` | `false` |
| `crushGainFactor` | `0.5` |
| `crushLengthFactor` | `0.5` |
| `crushCopiesFactor` | `0` |
| `crushTiebreakerFactor` | `0` |
| `useES6` | `true` |

2. aşama kazanıyor, düzenli ifade karakter sınıfı: `[\x01-\x1f@-Bj_ZX]`, 38 token. Baytlar şöyle iniyor:

```
   8 B  <script>
1127 B  paketli yük
   9 B  </script>
----
1144 B
```

`reassignVars`'ı açmak dört bayt kazandırıyor ve dosyayı 1.140'a indiriyor. Kapalı kalıyor, lichess yapısında kapalı kaldığı sebeple aynı: yeniden adlandırıcı `P`'den `V`'ye kadarını sözlük token'ı olarak harcıyor, yani geri çıkan kaynağın değişkenleri karıştırılmış oluyor ve artık kimsenin yazdığı program gibi okunmuyor. Dört bayt bunu geri satın almıyor.

Yarım değerli crusher faktörleri, RegPack'in kendi varsayılanlarına ve README'sinin önerdiği `1/0/0`'a göre üç bayt ediyor; ikisi de 1.131'e iniyor. Token kümesi aralarında neredeyse hiç kıpırdamıyor; fark, son birkaç ikame turunu birbirine çok yakın iki adaydan hangisinin kazandığı.

### Paketli dosya en kısa kaynaktan kurulmadı

Yukarıdaki kaynak 1.293 bayt, ve o kaynak sıfırdan yazılmış değil, kuralları değiştirilmiş lichess kaynağı. Aynı takaslar geçerli: takma adlar ve paylaşılan fonksiyonlar düz hâlde kazanıyor, paketleyicinin altında kaybediyor, çünkü RegPack'in parası tekrar eden dizilerle ödeniyor ve bir takma ad tam olarak tekrarı ortadan kaldıran şey. Hüküm ifadesi bilerek iki kez yazılıyor, isim verilmek yerine, çünkü crusher ikinci kopyayı tek bir token'a çeviriyor ve bunun için bir fonksiyonun tutacağından daha az ücret alıyor.

Yine de o kopyalardan biri iddialarla birlikte gitti — beraberlik kanalı eskiden hükmü yeniden değerlendiriyordu, artık değerlendirmiyor. Bu bir paketleme kararı değil kural değişikliği, ve düz kaynağın 181 bayt düşerken paketli dosyanın yalnızca 84 bayt düşmesinin sebebi bu.

---

## İlgili

- [FideLite](https://github.com/cuneytinann/FideLite) · [fidelite.art](https://www.fidelite.art/) — tam FIDE hakemi, on beş sonuç kodu, sekiz ön yüz
- [lichess-equivalent](https://github.com/cuneytinann/lichess-equivalent) — aynı egzersiz, bir sunucu öteye
- [Chess LUX](https://github.com/cuneytinann/Chess_LUX) — tam ters yön: ölü pozisyonlar %99,97'ye taşınmış
- [chessarbiter2kb](https://github.com/cuneytinann/chessarbiter2kb) — aynı kurallar bir seviye aşağıda, harfler ve rakamlar yan yana
- [chess1023byte](https://github.com/cuneytinann/chess1023byte) — çıplak kurallar, paketli, 1.023 baytta

## Lisans

MIT
