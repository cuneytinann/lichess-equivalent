**[English](#lichess-equivalent)** · **[Türkçe](#turkce)**

# lichess-equivalent

A chess arbiter that plays by **lichess's rules** rather than FIDE's, in a single HTML file of **3,044 bytes** — and the same arbiter stripped of its board, in **1,228**. Two players, one screen. No libraries, no build step, no server. Download a file, double-click, play.

The two rulebooks are close, but they are not the same book, and every place they part ways is written down below.

Part of the [Golfstack](https://www.fidelite.art/) project.

## Play

| file | interface | bytes | GitHub Pages | project site |
| --- | --- | --- | --- | --- |
| `index.html` | clickable board, clock, lichess colours | 3,044 | [lichess-equivalent](https://cuneytinann.github.io/lichess-equivalent/) | [Lichess-equivalent.html](https://www.fidelite.art/special/Lichess-equivalent.html) |
| `numerical_packed.html` | square numbers typed into a `prompt()` box, no board | 1,228 | [numerical_packed.html](https://cuneytinann.github.io/lichess-equivalent/numerical_packed.html) | [Lichess-equivalent_numerical.html](https://www.fidelite.art/special/Lichess-equivalent_numerical.html) |

On the project site both builds live under `special/`, off to the side of the `L1`–`L3` ladder. They are not another rung on it; they follow a different rulebook.

Anything from late 2020 onwards will run them: Chrome 85+, Firefox 79+, Safari 14+. Three things set that floor — BigInt, which the starting position is written with; the `safe` keyword in `place-content`, which keeps the board reachable on a narrow screen; and the `||=` operator, which the packed build uses.

**On the board.** Click a piece, then click where it should go. Legal squares pick up a dot, a piece you can take picks up a ring, the square you came from and the square you landed on stay tinted, and a king in check glows red. After every move the board turns around to face whoever is to play. When a pawn reaches the last rank the file it landed on turns into the picker: the four promotion choices sit stacked on the board itself, each on a grey disc, and you click the one you want. `½` offers a draw, or claims one when you are entitled to it, and it lights up on its own when the offer is yours to answer or the position has come round for the third time; `⚐` resigns. The two controls sit side by side and look alike on purpose — same height, white on colour, green for the draw and red for the resignation. The clock starts at ten minutes and hands back five seconds a move.

**Without the board.** `numerical_packed.html` draws nothing at all. Squares are numbered 1 to 64, a1 through h8, and a move is the two numbers written end to end: e2–e4 is `1329`. To promote, add a fifth digit — `0` bishop, `1` rook, `2` knight, anything else queen. A draw is offered the same way, by hanging a non-digit on the end of the move: `1329x` plays e2–e4 with an offer attached. Type the non-digit on its own and the offer still stands, but you owe a move afterwards. The opponent accepts by answering in kind; a plain move declines it and wipes it off. If the position has already come up three times, that same non-digit claims the threefold outright. To resign, leave the box empty or press Cancel.

That is the one place the two builds part company, and it is a difference of gesture rather than of rule. On the board a draw offer is a standing flag you raise with `½` and lower the same way; in the dialog it rides along with a move. Either way the arbiter reads the same two bits, and either way the opponent's plain move turns the offer down. The dialog title carries White's clock, Black's clock and the last move you got past the arbiter, so when an illegal move is quietly refused you will see that the last one never changed. Both clocks start at 900,000 milliseconds — fifteen minutes, no increment — and they keep running while the dialog is open, so thinking costs you what it would over the board. Keeping track of the position is your job.

## Two files, one arbiter

These are not a full version and a cut-down one. Same game, same rules, same core — and **1,228 bytes is what that core really costs.** Everything on top of it exists for you, not for chess.

The 1,815 bytes `index.html` spends beyond the packed file buy a board you can see, pieces you can click, a clock that ticks, a promotion picker drawn onto the board, colours that tell you which squares are open. Strip all of it away and not one rule moves. Mate is still mate, en passant is still en passant, the fifty-move counter still fills on exactly the same ply.

So the two files answer two different questions. *What does it cost to referee a game of chess?* Twelve hundred bytes. *What does it cost to let a human enjoy one?* Another eighteen hundred.

That is a claim worth measuring rather than asserting, so both builds were run side by side. First their rule layers, move for move: 21,410 plies, with the legal-move list of every piece belonging to the side to play compared square by square, and the board, the side to move, the en passant square, the castling rights, the halfmove clock and the material verdicts compared after each one. 142,253 comparisons, not a single disagreement. Then fifty complete games, played by clicking in Chrome and replayed digit by digit through the `prompt()` box — 16,630 plies, the same result code every time.

## Why lichess and not FIDE

Chess has a rulebook, and then chess servers have rulebooks of their own. They agree on almost everything and disagree in a handful of interesting places. This arbiter follows lichess deliberately, and the decision was made by reading lichess's own source rather than by guessing at its behaviour.

| ending | FIDE | lichess | here |
| --- | --- | --- | --- |
| checkmate, stalemate | — | — | ✓ |
| insufficient material | automatic | automatic | automatic |
| fivefold repetition | automatic | automatic | automatic |
| threefold repetition | **claim** | **claim** | **claim** |
| fifty moves | **claim** | **automatic** | **automatic** |
| seventy-five moves | automatic | *never reached* | *never reached* |
| draw by agreement | — | — | ✓ |
| flag fall | loss | loss | loss |
| flag fall, opponent cannot mate | **draw** | **draw** | **draw** |
| resignation | loss | loss | loss |
| resignation, opponent cannot mate | **draw** | **draw** | **draw** |

The bold rows are where the interesting part lives.

**Fifty moves.** FIDE lets a player *claim* the draw at fifty and has the arbiter declare it unasked at seventy-five. Lichess does not wait that long: `Variant.autoDraw` checks `halfMoveClock >= 100` and ends the game right there. Which means the seventy-five-move rule never gets a chance to fire on lichess, and it has no reason to exist here either. The game simply ends as `50`.

**Resigning against a bare king.** Article 5.1.2 says a resignation loses the game — *unless* the opponent could not deliver mate by any series of legal moves, in which case it is a draw. Most servers skip this. Lichess does not: in `RoundAsyncActor`, a resignation from a player who `cannotLose` is recorded as `InsufficientMaterialClaim`, a draw. This file does the same and calls it `RM`.

**Running out of time against a bare king.** Article 6.9, the same idea one article along. Lichess's `Finisher.outOfTime` hands over the win only `ifFalse(position.opponentHasInsufficientMaterial)`. Here that ending is `TM`.

**Blocked positions.** A position can be dead with pieces to spare: the pawns interlock, and mate becomes impossible even if both players set out to arrange one. Article 5.2.2 calls that a draw on the spot. Lichess does not look for it, and neither does this file — both play on until repetition or the fifty-move counter closes the game instead. Taking that detector out is most of what separates this build from its FIDE sibling on the project site.

**En passant, and the mistake nearly everyone makes.** For repetition, two positions count as the same only if a legal en passant capture is available in both or in neither. Plenty of engines write the en passant square after every double pawn push, whether or not anything can actually take. Nothing looks broken, because the capture gets refused anyway — but the repetition key comes out different, and a threefold that should have triggered arrives late or never arrives at all. Lichess gets this right: its position hash uses `enPassantSquare`, which has already been filtered through `isLegalEnPassant`, not the raw `potentialEpSquare`. So does this file, and it is stricter about it — the square goes in only after the legal move generator has confirmed that a neighbouring pawn really can take.

*Read from [scalachess](https://github.com/lichess-org/scalachess) (`Variant.scala`, `Position.scala`, `Hash.scala`) and [lila](https://github.com/lichess-org/lila) (`Finisher.scala`, `RoundGame.scala`) in September 2026. Lichess can change its mind later; this file cannot.*

## Result codes

Fourteen ways a game can end: six decisive, eight drawn.

| | | | |
| --- | --- | --- | --- |
| `W#` White mates | `B#` Black mates | `SM` stalemate | `IM` insufficient material |
| `WT` White wins on time | `BT` Black wins on time | `TM` flag fall, mate impossible | `50` fifty-move rule |
| `WR` White wins by resignation | `BR` Black wins by resignation | `RM` resignation, mate impossible | `5R` fivefold repetition |
| `3R` threefold claim | `DA` draw by agreement | | |

A code beginning with `W` is a win for White and one beginning with `B` a win for Black; everything ending in `M`, beginning with `D`, or carrying a digit is a draw. `numerical_packed.html` reports the same fourteen endings as numbers, `1` through `15`, with `4` left unused — that slot belonged to the seventy-five-move rule.

The FIDE build on the project site has fifteen codes and calls insufficient material `DP`, dead position, because it detects blocked ones as well. Here the code is `IM`, which names what the file actually tests.

## What's in it

- **Every piece's movement**, worked out with arithmetic. No direction tables, no offset arrays.
- **Full legality.** A move that would leave your own king in check is never let through.
- **Castling** on both wings, with all of it checked: the right still standing, the rook's path clear, the king not in check, not crossing an attacked square, not landing on one.
- **En passant**, including the repetition-key subtlety above.
- **Promotion** to queen, rook, bishop or knight — a picker on the board, a fifth digit without it.
- **A running clock.** Ten minutes plus five seconds a move on the board, fifteen flat minutes in the dialog.
- **Draw offers and claims**, resignation, flag fall, and the two impossible-mate exceptions that turn a loss into a draw.
- **All fourteen endings**, told apart from one another.

## What's not in it

- No blocked-position detection, as above. Lichess has none either.
- No engine, no takebacks, no FEN in or out, no PGN.
- No coordinates around the board. It turns around every ply, and the status line keeps out of the way.

For the full FIDE arbiter — fifteen codes, dead positions, eight front ends — see [fidelite.art](https://www.fidelite.art/).

## Read it in the browser

`index.html` is not packed and not minified. **Right-click → View Page Source** (`Ctrl` `U`, or `⌥` `⌘` `U` on macOS) puts the whole program on screen, and that is what this repository is for: open the page, play a few moves, then go and read the source that just refereed them.

`numerical_packed.html` is the exception. It unpacks itself, and there is a section further down on how to open it up.

---

## Anatomy

Every byte of `index.html`, by part.

| part | bytes | |
| --- | --- | --- |
| markup and CSS | 629 | board, panel, controls, colours, layout |
| `<script>` tags | 17 | |
| state and aliases | 181 | board, clock, castling rights, repetition table |
| `G` | 264 | can this piece reach that square |
| `V` | 51 | is this square attacked |
| `L` | 101 | is this move legal — play it, ask, take it back |
| `C` | 28 | which castling right a square forfeits |
| `M` | 223 | counter, promotion, en passant victim, rook hop, en passant square |
| `I`, `H` | 161 | material, and whether mate is possible at all |
| `Z`, `D`, `F` | 151 | the verdict, the draw, the flag |
| `j` | 62 | the clock |
| setup | 71 | the 64 cells, generated at load |
| `d` | 782 | draw the board, and the promotion picker on it |
| `A`, `Bt`, `S` | 292 | play the move, the buttons, the click |
| first draw and interval | 32 | |
| **total** | **3,044** | |

Sliced the other way: the rules come to **1,138** bytes and the page that shows them to **1,906**. The referee is cheap and the stage is expensive, which is exactly the argument the packed file makes.

## Verification

Both builds were checked by running them rather than by reading them. `index.html` was driven through real Chrome over the DevTools protocol and again under jsdom; `numerical_packed.html` ran in Node against a stubbed `prompt()`.

- **perft**, on both builds, over the five standard positions: 4,865,609 from the starting position at depth 5; 4,085,603 from Kiwipete at depth 4; 674,624 from CPW position 3 at depth 5; 422,333 from position 4 at depth 4; 2,103,487 from position 5 at depth 4.
- **All fourteen endings**, each one reached and checked — `TM` and `RM` among them, and the automatic `50` firing on ply 100 exactly.
- **The two builds against each other**, twice over: 21,410 plies of step-by-step comparison on the rule layer, taking in 142,253 legal-move lists and the full state after every ply; then 50 complete games clicked out in Chrome and replayed digit by digit in the numerical build, 16,630 plies. Neither run turned up a difference.
- **Markup**: the W3C Nu Html Checker returns zero errors and zero warnings on `index.html`. Standards mode, UTF-8, no BOM, not a single line break in the file.
- **Rendering**, cell by cell in Chrome: a 568×568 board of 71×71 squares, the last move and the selection tinted correctly on light and dark squares alike, the check glow on the king's square and nowhere else, and the promotion discs landing on the right four squares of the right file with all four pieces in the mover's colour, whatever stands underneath them.

## Unpacking

`numerical_packed.html` unpacks itself. Its script is a RegPack decompression loop that ends in `eval(_)`, so to get the plain source back you replace that one call:

```js
eval(_)   →   console.log(_)
```

Nothing in the loop touches the game, so it is safe to do this in Node. What falls out is 1,474 bytes of source.

**Download the file rather than copying it out of the browser.** The dictionary keys are control characters from the `\x01`–`\x1f` range, and the clipboard — or any editor that tidies up line endings — will quietly destroy them.

## Packing

[RegPack 5.0.1](https://github.com/Siorki/RegPack). These settings rebuild `numerical_packed.html` from that source **byte for byte**:

| option | value |
| --- | --- |
| `reassignVars` | `false` |
| `crushGainFactor` | `0` |
| `crushLengthFactor` | `0` |
| `crushCopiesFactor` | `0` |
| `crushTiebreakerFactor` | `0` |
| `useES6` | `true` |

Stage 2 wins, the regexp character class: `[\x01-\x1f@Aj_ZX]`, 37 tokens, 35 substitution rounds. The bytes land like this:

```
   8 B  <script>
1211 B  packed payload
   9 B  </script>
----
1228 B
```

Turning `reassignVars` on saves three bytes and brings the file down to 1,225. It stays off: the renamer spends `R` through `W` as dictionary tokens, so the source that comes back out has had its variables shuffled and no longer reads as the program anyone wrote. Three bytes do not buy that back.

### The packed file was not built from the shortest source

The source above runs to 1,474 bytes. The same game, written to be as small as possible *in plain form*, comes to 1,401 — seventy-three bytes lighter — and packing that one produces a **bigger** file.

Aliases are the reason. `a=Math.abs`, `N='indexOf'`, a single shared `Z`/`A`/`D`/`F` instead of the same bodies inlined at every call site: each of these wins in plain form and loses under the packer, because RegPack is paid in repeated substrings and an alias is precisely the thing that takes repetition away. Spelled out and inlined, the source runs 73 bytes longer on its own and 40 bytes shorter once packed.

So there are two lineages of one program, and the one that got packed is not the short one.

---

## Related

- [FideLite](https://github.com/cuneytinann/FideLite) · [fidelite.art](https://www.fidelite.art/) — the full FIDE arbiter, fifteen result codes, eight front ends
- [Chess LUX](https://github.com/cuneytinann/Chess_LUX) — the other direction entirely: dead positions carried to 99.97%
- [chessarbiter2kb](https://github.com/cuneytinann/chessarbiter2kb) — the same rules a level down, letters and numbers side by side
- [chess1023byte](https://github.com/cuneytinann/chess1023byte) — the bare rules, packed, in 1,023 bytes

## License

MIT

---
---

<a id="turkce"></a>

# lichess-equivalent (Türkçe)

FIDE'nin değil, **lichess'in kurallarıyla** hükmeden bir satranç hakemi; tek bir HTML dosyasında **3.044 bayt** — ve aynı hakemin tahtasından soyulmuş hâli, **1.228 baytta**. İki oyuncu, tek ekran. Kütüphane yok, derleme adımı yok, sunucu yok. Dosyayı indirin, çift tıklayın, oynayın.

İki kural kitabı birbirine yakındır ama aynı kitap değildir; yolların ayrıldığı her nokta aşağıda tek tek yazılı.

[Golfstack](https://www.fidelite.art/) projesinin bir parçasıdır.

## Oyna

| dosya | arayüz | bayt | GitHub Pages | proje sitesi |
| --- | --- | --- | --- | --- |
| `index.html` | tıklanabilir tahta, saat, lichess renkleri | 3.044 | [lichess-equivalent](https://cuneytinann.github.io/lichess-equivalent/) | [Lichess-equivalent.html](https://www.fidelite.art/special/Lichess-equivalent.html) |
| `numerical_packed.html` | `prompt()` kutusuna yazılan kare numaraları, tahta yok | 1.228 | [numerical_packed.html](https://cuneytinann.github.io/lichess-equivalent/numerical_packed.html) | [Lichess-equivalent_numerical.html](https://www.fidelite.art/special/Lichess-equivalent_numerical.html) |

Proje sitesinde iki sürüm de `special/` altında, `L1`–`L3` merdiveninin bir kenarında durur. O merdivenin bir basamağı değildirler; başka bir kural kitabını izlerler.

2020 sonu ve sonrasının her tarayıcısı çalıştırır: Chrome 85+, Firefox 79+, Safari 14+. Bu tabanı üç şey belirliyor — başlangıç dizilişinin yazıldığı BigInt; dar ekranda tahtanın erişilebilir kalmasını sağlayan `place-content`'teki `safe` anahtar sözcüğü; ve paketli sürümün kullandığı `||=` işleci.

**Tahtayla.** Bir taşa tıklayın, sonra gitmesini istediğiniz kareye. Yasal kareler birer nokta alır, alabileceğiniz taş bir halka; çıktığınız ve indiğiniz kare boyalı kalır; şah altındaki şahın karesi kırmızı parlar. Her hamleden sonra tahta dönüp sırası gelene bakar. Bir piyon son yatayı bulduğunda indiği sütun seçiciye dönüşür: dört terfi seçeneği tahtanın üstünde, her biri gri bir diskin içinde alt alta durur, istediğinize tıklarsınız. `½` beraberlik teklif eder, hakkınız varsa talep eder — ve cevap sırası sizdeyken ya da konum üçüncü kez geldiğinde kendiliğinden yanar; `⚐` terk eder. İki denetim yan yana ve bilerek birbirine benzer: aynı boy, renk üstüne beyaz, beraberlikte yeşil, terkte kırmızı. Saat on dakikadan başlar ve her hamlede beş saniye geri verir.

**Tahtasız.** `numerical_packed.html` hiçbir şey çizmez. Kareler a1'den h8'e doğru 1'den 64'e numaralıdır; hamle, iki numaranın uç uca yazılmış hâlidir: e2–e4 `1329` olur. Terfi için beşinci bir basamak eklersiniz — `0` fil, `1` kale, `2` at, başka bir şey vezir. Beraberlik de aynı yoldan teklif edilir: hamlenin sonuna rakam olmayan bir karakter iliştirirsiniz, `1329x` yazmak e2–e4'ü teklifle birlikte oynar. O karakteri tek başına yazarsanız teklif yine geçerlidir ama sıra sizde kalır, arkasından bir hamle borcunuz olur. Rakip aynı şekilde yanıt verirse beraberlik olur; düz bir hamle oynarsa teklifi reddetmiş ve silmiş olur. Konum daha önce üç kez oluştuysa aynı karakter üçlü tekrarı doğrudan talep eder. Terk etmek için kutuyu boş bırakın ya da İptal'e basın.

İki sürümün ayrıştığı tek yer burası ve bu bir kural farkı değil, bir jest farkı. Tahtada beraberlik teklifi `½` ile kaldırıp yine `½` ile indirdiğiniz, havada duran bir bayraktır; iletişim kutusunda ise hamleye binerek gider. İki durumda da hakem aynı iki biti okur ve iki durumda da rakibin düz hamlesi teklifi reddeder. İletişim kutusunun başlığında Beyaz'ın saati, Siyah'ın saati ve hakemden geçirebildiğiniz son hamle durur; yasadışı bir hamle sessizce reddedildiğinde sonuncunun hiç değişmediğini oradan anlarsınız. İki saat de 900.000 milisaniyeden başlar — artırımsız on beş dakika — ve kutu açıkken işlemeyi sürdürür, yani düşünmek size tahta başındaki kadara mal olur. Konumu takip etmek ise tümüyle sizin işiniz.

## İki dosya, tek hakem

Bunlar tam sürüm ile kırpılmış sürüm değil. Aynı oyun, aynı kurallar, aynı öz — ve **o özün gerçek bedeli 1.228 bayt.** Üstüne binen her şey satranç için değil, sizin için var.

`index.html`'in paketli dosyanın ötesinde harcadığı 1.815 bayt şunu satın alıyor: gördüğünüz bir tahta, tıkladığınız taşlar, işleyen bir saat, tahtanın üstüne çizilen bir terfi seçicisi, hangi karenin açık olduğunu söyleyen renkler. Hepsini soyun, tek bir kural yerinden oynamaz. Mat yine mattır, geçerken alma yine geçerken almadır, elli hamle sayacı yine tam aynı yarım hamlede dolar.

Yani iki dosya iki ayrı soruya yanıt veriyor. *Bir satranç oyununu yönetmenin bedeli nedir?* Bin iki yüz bayt. *Bir insanın o oyundan keyif almasının bedeli nedir?* Bin sekiz yüz bayt daha.

Bu, iddia edilmektense ölçülmeyi hak eden bir cümle; o yüzden iki sürüm yan yana koşturuldu. Önce kural katmanları, hamle hamle: 21.410 yarım hamle boyunca sırası gelen tarafın her taşının yasal hamle listesi kare kare karşılaştırıldı, her hamleden sonra da tahta, sıra, geçerken alma karesi, rok hakları, yarım hamle sayacı ve materyal hükümleri. 142.253 karşılaştırma, tek bir ayrılık yok. Sonra elli tam oyun: Chrome'da tıklanarak oynandı ve `prompt()` kutusuna basamak basamak yeniden girildi — 16.630 yarım hamle, her seferinde aynı sonuç kodu.

## Neden lichess, neden FIDE değil

Satrancın bir kural kitabı vardır, satranç sunucularının da kendi kural kitapları. Neredeyse her şeyde anlaşır, birkaç ilginç noktada ayrılırlar. Bu hakem bilerek lichess'i izliyor ve bu karar, davranışını tahmin ederek değil, lichess'in kendi kaynağı okunarak verildi.

| bitiş | FIDE | lichess | burada |
| --- | --- | --- | --- |
| mat, pat | — | — | ✓ |
| yetersiz materyal | otomatik | otomatik | otomatik |
| beş kez tekrar | otomatik | otomatik | otomatik |
| üç kez tekrar | **talep** | **talep** | **talep** |
| elli hamle | **talep** | **otomatik** | **otomatik** |
| yetmiş beş hamle | otomatik | *hiç ulaşılmaz* | *hiç ulaşılmaz* |
| anlaşmalı beraberlik | — | — | ✓ |
| süre bitimi | yenilgi | yenilgi | yenilgi |
| süre bitimi, rakip mat edemiyor | **beraberlik** | **beraberlik** | **beraberlik** |
| terk | yenilgi | yenilgi | yenilgi |
| terk, rakip mat edemiyor | **beraberlik** | **beraberlik** | **beraberlik** |

İşin ilginç yanı kalın satırlarda.

**Elli hamle.** FIDE ellinci hamlede oyuncuya *talep* hakkı tanır, yetmiş beşinci hamlede ise hakem sormadan ilan eder. Lichess o kadar beklemez: `Variant.autoDraw` doğrudan `halfMoveClock >= 100` koşuluna bakar ve oyunu orada bitirir. Dolayısıyla yetmiş beş hamle kuralı lichess'te tetiklenme fırsatı bile bulamaz; burada da bulunmasının bir anlamı yok. Oyun yalnızca `50` ile kapanır.

**Çıplak şaha karşı terk.** 5.1.2. madde terkin oyunu kaybettirdiğini söyler — *ancak* rakip hiçbir yasal hamle dizisiyle mat edemiyorsa beraberliktir. Çoğu sunucu bu ayrıntıyı atlar. Lichess atlamaz: `RoundAsyncActor`'da `cannotLose` durumundaki bir oyuncunun terki `InsufficientMaterialClaim` olarak, yani beraberlik olarak kaydedilir. Bu dosya da aynısını yapıyor ve buna `RM` diyor.

**Çıplak şaha karşı süre bitimi.** 6.9. madde, bir madde ötede aynı fikir. Lichess'in `Finisher.outOfTime` işlevi galibiyeti yalnızca `ifFalse(position.opponentHasInsufficientMaterial)` koşuluyla teslim eder. Buradaki karşılığı `TM`.

**Kilitli pozisyonlar.** Bir konum, taş fazlasıyla yeterliyken bile ölü olabilir: piyonlar birbirine kenetlenir ve iki oyuncu mat kurmak için anlaşsa bile mat kurulamaz. 5.2.2. madde bunu olduğu anda beraberlik sayar. Lichess bunu aramaz, bu dosya da aramaz — ikisi de tekrar ya da elli hamle sayacı oyunu kapatana kadar oynamayı sürdürür. Bu sürümü proje sitesindeki FIDE kardeşinden ayıran şeyin büyük bölümü, o dedektörün sökülmüş olması.

**Geçerken alma ve neredeyse herkesin düştüğü hata.** Tekrar açısından iki konum, ancak yasal bir geçerken alma imkânı ikisinde de varsa ya da ikisinde de yoksa aynı sayılır. Pek çok motor, gerçekten alabilen bir piyon olsun olmasın, her çift adımdan sonra geçerken alma karesini yazar. Dışarıdan bozuk bir şey görünmez, çünkü alma zaten reddedilir — ama tekrar anahtarı farklı çıkar ve tetiklenmesi gereken üçlü tekrar ya geç gelir ya hiç gelmez. Lichess bu ayrımı doğru yapıyor: konum karması, ham `potentialEpSquare`'i değil, `isLegalEnPassant` süzgecinden geçmiş `enPassantSquare`'i kullanıyor. Bu dosya da öyle, hatta biraz daha katı davranıyor — kare, ancak yasal hamle üreticisi komşu bir piyonun gerçekten alabildiğini onayladıktan sonra yazılıyor.

*Eylül 2026'da [scalachess](https://github.com/lichess-org/scalachess) (`Variant.scala`, `Position.scala`, `Hash.scala`) ve [lila](https://github.com/lichess-org/lila) (`Finisher.scala`, `RoundGame.scala`) dosyalarından okundu. Lichess ileride fikrini değiştirebilir; bu dosya değiştiremez.*

## Sonuç kodları

Bir oyunun bitebileceği on dört yol: altısı kesin sonuç, sekizi beraberlik.

| | | | |
| --- | --- | --- | --- |
| `W#` Beyaz mat eder | `B#` Siyah mat eder | `SM` pat | `IM` yetersiz materyal |
| `WT` Beyaz süreyle kazanır | `BT` Siyah süreyle kazanır | `TM` süre bitti, mat imkânsız | `50` elli hamle kuralı |
| `WR` Beyaz terkle kazanır | `BR` Siyah terkle kazanır | `RM` terk, mat imkânsız | `5R` beş kez tekrar |
| `3R` üç kez tekrar talebi | `DA` anlaşmalı beraberlik | | |

`W` ile başlayan kod Beyaz'ın, `B` ile başlayan Siyah'ın galibiyetidir; `M` ile biten, `D` ile başlayan ya da rakam taşıyan her şey beraberliktir. `numerical_packed.html` aynı on dört bitişi sayı olarak bildirir, `1`'den `15`'e; `4` boş kalır — o yuva yetmiş beş hamle kuralınındı.

Proje sitesindeki FIDE sürümünde on beş kod vardır ve yetersiz materyale `DP`, yani ölü pozisyon der; çünkü kilitli olanları da tespit eder. Burada kod `IM` — dosyanın gerçekten sınadığı şeyin adı.

## İçinde neler var

- **Bütün taş hareketleri**, aritmetikle çıkarılmış. Yön tablosu yok, ofset dizisi yok.
- **Tam yasallık.** Kendi şahınızı açıkta bırakacak bir hamle hiçbir zaman geçmez.
- **İki kanatta rok**, hepsi denetlenerek: hak hâlâ duruyor mu, kalenin yolu boş mu, şah tehdit altında mı, geçtiği kare tehdit altında mı, indiği kare tehdit altında mı.
- **Geçerken alma**, yukarıdaki tekrar anahtarı inceliğiyle birlikte.
- **Terfi**: vezir, kale, fil, at — tahtada bir seçiciyle, tahtasız sürümde beşinci basamakla.
- **İşleyen bir saat.** Tahtada on dakika artı hamle başına beş saniye, iletişim kutusunda artırımsız on beş dakika.
- **Beraberlik teklifi ve talebi**, terk, süre bitimi ve yenilgiyi beraberliğe çeviren iki mat-imkânsızlığı istisnası.
- **On dört bitişin hepsi**, birbirinden ayrılmış hâlde.

## İçinde neler yok

- Kilitli pozisyon tespiti yok, yukarıda anlattığımız gibi. Lichess'te de yok.
- Motor yok, geri alma yok, FEN alışverişi yok, PGN yok.
- Tahtanın çevresinde koordinat yok. Tahta her yarım hamlede dönüyor, durum satırı da yoldan çekiliyor.

Tam FIDE hakemi için — on beş kod, ölü pozisyonlar, sekiz arayüz — [fidelite.art](https://www.fidelite.art/) adresine bakın.

## Tarayıcıda okuyun

`index.html` paketlenmiş de küçültülmüş de değil. **Sağ tık → Sayfa Kaynağını Görüntüle** (`Ctrl` `U`, macOS'ta `⌥` `⌘` `U`) programın tamamını ekrana getirir; bu depo zaten bunun için var: sayfayı açın, birkaç hamle oynayın, sonra o hamleleri az önce yöneten kaynağı okuyun.

`numerical_packed.html` bunun istisnası. Kendi kendini açar; nasıl açılacağı aşağıda anlatılıyor.

---

## Anatomi

`index.html`'in her baytı, parça parça.

| parça | bayt | |
| --- | --- | --- |
| markup ve CSS | 629 | tahta, panel, denetimler, renkler, yerleşim |
| `<script>` etiketleri | 17 | |
| durum ve takma adlar | 181 | tahta, saat, rok hakları, tekrar tablosu |
| `G` | 264 | bu taş o kareye ulaşabilir mi |
| `V` | 51 | bu kare tehdit altında mı |
| `L` | 101 | bu hamle yasal mı — oyna, sor, geri al |
| `C` | 28 | bir karenin düşürdüğü rok hakkı |
| `M` | 223 | sayaç, terfi, geçerken alınan piyon, kale sıçraması, geçerken alma karesi |
| `I`, `H` | 161 | materyal ve matın mümkün olup olmadığı |
| `Z`, `D`, `F` | 151 | hüküm, beraberlik, bayrak |
| `j` | 62 | saat |
| kurulum | 71 | 64 hücre, açılışta üretiliyor |
| `d` | 782 | tahtayı ve üstündeki terfi seçicisini çiz |
| `A`, `Bt`, `S` | 292 | hamleyi oyna, düğmeler, tıklama |
| ilk çizim ve interval | 32 | |
| **toplam** | **3.044** | |

Başka türlü bölersek: kurallar **1.138** bayt, onları gösteren sayfa **1.906**. Hakem ucuz, sahne pahalı — paketli dosyanın öne sürdüğü şey tam olarak bu.

## Doğrulama

İki sürüm de okunarak değil, koşturularak sınandı. `index.html` hem DevTools protokolü üzerinden gerçek Chrome'da hem de jsdom altında sürüldü; `numerical_packed.html` Node'da, taklit bir `prompt()` karşısında çalıştı.

- **perft**, iki sürümde de, beş standart pozisyon üzerinde: başlangıç dizilişinden 5. derinlikte 4.865.609; Kiwipete'ten 4. derinlikte 4.085.603; CPW 3. pozisyondan 5. derinlikte 674.624; 4. pozisyondan 4. derinlikte 422.333; 5. pozisyondan 4. derinlikte 2.103.487.
- **On dört bitişin hepsi**, tek tek üretilip sınandı — aralarında `TM` ile `RM` ve tam 100. yarım hamlede tetiklenen otomatik `50` de var.
- **İki sürüm birbirine karşı**, iki ayrı biçimde: kural katmanında 21.410 yarım hamlelik adım adım karşılaştırma, 142.253 yasal hamle listesi ve her hamleden sonra tam durum; ardından Chrome'da tıklanarak oynanıp sayısal sürümde basamak basamak tekrarlanan 50 tam oyun, 16.630 yarım hamle. İki koşuda da fark çıkmadı.
- **Markup**: W3C Nu Html Checker `index.html` için sıfır hata, sıfır uyarı veriyor. Standart mod, UTF-8, BOM yok, dosyada tek bir satır sonu bile yok.
- **Çizim**, Chrome'da hücre hücre: 71×71 karelerden oluşan 568×568 tahta; son hamle ve seçim açık ve koyu karelerde ayrı ayrı doğru tonda; şah parlaması yalnız şahın karesinde; terfi diskleri doğru sütunun doğru dört karesine oturuyor ve dört taş da altlarında ne olursa olsun hamleyi yapanın renginde çıkıyor.

## Paketi açma

`numerical_packed.html` kendi kendini açar. Betiği `eval(_)` ile biten bir RegPack açma döngüsüdür; düz kaynağı geri almak için o tek çağrıyı değiştirmeniz yeter:

```js
eval(_)   →   console.log(_)
```

Döngünün içinde oyuna dokunan hiçbir şey yok, bunu Node'da yapmak güvenli. Dökülen şey 1.474 baytlık kaynak.

**Dosyayı tarayıcıdan kopyalamak yerine indirin.** Sözlük anahtarları `\x01`–`\x1f` aralığındaki denetim karakterleridir; pano ya da satır sonlarını derleyip toplayan herhangi bir editör onları sessizce yok eder.

## Paketleme

[RegPack 5.0.1](https://github.com/Siorki/RegPack). Şu ayarlar `numerical_packed.html`'i o kaynaktan **bayt bayt** yeniden kurar:

| ayar | değer |
| --- | --- |
| `reassignVars` | `false` |
| `crushGainFactor` | `0` |
| `crushLengthFactor` | `0` |
| `crushCopiesFactor` | `0` |
| `crushTiebreakerFactor` | `0` |
| `useES6` | `true` |

Kazanan aşama 2, yani düzenli ifade karakter sınıfı: `[\x01-\x1f@Aj_ZX]`, 37 token, 35 değiştirme turu. Baytlar şöyle yerleşiyor:

```
   8 B  <script>
1211 B  paketli yük
   9 B  </script>
----
1228 B
```

`reassignVars`'ı açmak üç bayt kazandırıp dosyayı 1.225'e indiriyor. Yine de kapalı: yeniden adlandırıcı `R`–`W` harflerini sözlük tokeni olarak harcıyor, geri çıkan kaynağın değişkenleri karışmış oluyor ve metin artık kimsenin yazdığı program gibi okunmuyor. Üç bayt bunu geri satın almaya yetmez.

### Paketli dosya en kısa kaynaktan kurulmadı

Yukarıdaki kaynak 1.474 bayt tutuyor. Aynı oyunun *düz hâlde* olabildiğince küçük olacak şekilde yazılmışı 1.401 — yetmiş üç bayt hafif — ve onu paketlemek **daha büyük** bir dosya veriyor.

Sebep takma adlar. `a=Math.abs`, `N='indexOf'`, aynı gövdeleri her çağrı yerine gömmek yerine tek bir ortak `Z`/`A`/`D`/`F`: bunların her biri düz hâlde kazandırıp paketleyicide kaybettiriyor, çünkü RegPack tekrar eden dizgelerle beslenir ve takma ad tam da tekrarı ortadan kaldıran şeydir. Açık yazılıp gömüldüğünde kaynak kendi başına 73 bayt uzuyor, paketlendikten sonra 40 bayt kısalıyor.

Yani tek bir programın iki soyu var ve paketlenen, kısa olan değil.

---

## İlgili

- [FideLite](https://github.com/cuneytinann/FideLite) · [fidelite.art](https://www.fidelite.art/) — tam FIDE hakemi, on beş sonuç kodu, sekiz arayüz
- [Chess LUX](https://github.com/cuneytinann/Chess_LUX) — tam ters yön: ölü pozisyonlar %99,97'ye taşınmış
- [chessarbiter2kb](https://github.com/cuneytinann/chessarbiter2kb) — aynı kurallar bir seviye aşağıda, harfler ve sayılar yan yana
- [chess1023byte](https://github.com/cuneytinann/chess1023byte) — yalnız çıplak kurallar, paketlenmiş, 1.023 baytta

## Lisans

MIT
