# Günlük Kelimeler Uygulaması – Hazır Prompt

Aşağıdaki metni olduğu gibi bir yapay zekâ asistanına (Claude vb.) yapıştır. Asistan, uygulamayı tek bir HTML dosyası olarak oluşturur.

---

## PROMPT

Telefonumda açacağım, tek dosyalık (HTML + CSS + JavaScript) bir web uygulaması oluştur. Dışarıdan hiçbir kütüphane veya görsel yükleme. Aşağıdaki özellikleri ve kodu birebir kullan.

**Uygulamanın amacı:** Her gün sabah 09:00'da 3 yeni İngilizce kelime göstermek (biri A1, biri A2, biri B1 seviyesinde).

**Özellikler:**
- Her kelime için: İngilizce yazılışı, IPA telaffuzu, "Dinle" butonu (tarayıcının speechSynthesis özelliğiyle kelimeyi ve cümleyi sesli okur), Türkçe anlamı.
- Her kelime için, kelimeyi kullanan 1 İngilizce örnek cümle (kelime vurgulu) ve cümlenin Türkçe çevirisi.
- Kelimeler tarihe göre seçilir (sunucu gerekmez). Gün saat 09:00'da değişir; 09:00'dan önce önceki günün kelimeleri görünür. Her seviyede 30 kelime vardır (30 gün tekrar yok).
- Sonraki kelimelere kalan süre gösterilir ve sayfa her 30 saniyede yenilenir.
- Günlük takip akışı: Her kelime için 3 adım (Dinle → Sesli tekrar et → Kendi cümleni kur). Toplam 9 adım. Üstte günlük ilerleme çubuğu, art arda tamamlanan gün sayısı (seri) ve son 7 günün noktaları. Tüm adımlar bitince tebrik mesajı.
- İki sekme: **Bugün** (günlük kelimeler ve takip akışı) ve **Önceki Kelimeler** (geçmiş günlerin kelimeleri: İngilizce, IPA, Türkçe anlam, örnek cümle ve çevirisi, dinleme butonu). "Kendimi sına" modunda Türkçe anlamlar bulanıklaşır, dokununca görünür; "Son 30 günü göster" ile son 30 gün listelenir. Geçmiş, uygulamanın ilk kullanıldığı günden itibaren tutulur.
- **Aralıklı tekrar (Leitner kutuları):** "Önceki" sekmesinde her kelimenin altında üç buton: ✓ Biliyorum, ✗ Bilmiyorum ve 🔁 Tekrar edilecek. Bilince kelime bir üst kutuya çıkar (tekrar aralıkları 1, 3, 7, 14, 30 gün), bilmeyince 0. kutuya düşer. "Tekrar edilecek" kelimeyi elle işaretler (tekrar basınca kaldırır); "Biliyorum" işareti kaldırır. Veriler localStorage'da `dw-srs` ve `dw-flag` anahtarlarında.
- **Günlük tekrar ("🔁 Tekrar" sekmesi):** Her gün önceki günlerde verilmiş kelimelerden 3 kelime, günün tarihine göre karışık seçilir (en fazla 2'si işaretli/bilinmeyen kelimelerden, kalanı rastgele; önceki kelime yoksa bugünün kelimelerinden tamamlanır). Gün içinde aynı 3 kelime kalır (`dw-rev`). Üçü de kart olarak listelenir: "Anlamı göster" → Türkçe anlam ve cümle → Biliyorum/Bilmiyorum. Hepsi bitince "Günlük tekrar tamamlandı". İlerleme kartında "🔁 Tekrar: 2/3" görünür.
- **Quiz ("🎯 Quiz" sekmesi):** 10 soru; 3 tür dönüşümlü: çoktan seçmeli anlam, cümlede boşluk doldurma, sesi dinleyip kelimeyi seçme. Sorular görülen kelimelerden, şıklar tüm havuzdan gelir; doğru/yanlış tekrar sistemine işlenir.
- **Günlük hedef:** Sayfanın en üstünde (tarihin altında, sekmelerin üstünde) her sekmede görünen bir banner: "🎯 Günlük hedef: 10 soruluk quizi tamamla". 10 soruluk quiz bitince "✅ Günlük hedef tamamlandı!" olur ve yeşile döner. Günün en iyi quiz sonucu ilerleme kartında "🎯 Quiz: 8/10" olarak da görünür.
- Kalıcı hafıza ve telefona kurulum (PWA): tüm ilerleme, tekrar, quiz sonuçları ve yarım kalan quiz localStorage'da saklanır (anahtarlar: dw-v1, dw-start, dw-srs, dw-flag, dw-rev, dw-quiz, dw-qz); navigator.storage.persist() istenir. Sayfanın altında "💾 Yedekle / ♻️ Geri yükle" bulunur (yedek metni JSON olarak kopyalanır, geri yüklenir). Ayrıca manifest.webmanifest (standalone, ikonlar), sw.js (çevrimdışı çalışma, ağ-öncelikli önbellek) ve ikon dosyaları ayrı dosyalar olarak yayınlanır; `<head>` içinde manifest, theme-color ve apple-touch-icon etiketleri vardır, kodun sonunda service worker kaydı yapılır.
- **Kalıp sekmesi ("💬 Kalıp"):** En çok kullanılan 30 İngilizce cümle kalıbı (örn. "I’m looking forward to + V-ing", "Would you mind + V-ing?"), her gün 1 kalıp, tarihe göre sırayla gelir. Kalıp, Türkçe anlamı, örnek cümle ve çevirisi, 🔊 Dinle, "Öğrendim olarak işaretle" ve "🔁 Tekrar edilecek" butonları. Altında önceki kalıplar listelenir; her birinde Biliyorum / Bilmiyorum / Tekrar edilecek butonları (aralıklı tekrar sistemine bağlı).
- **Tekrar ve quiz entegrasyonu:** Günlük tekrar artık 3 kelime + 1 önceki kalıp (4 kart; işaretli/bilinmeyen kalıplar öncelikli). Quizde 10 sorunun 2'si kalıp sorusudur: "Bu kalıp ne anlama gelir?" ve "Bu Türkçe cümlede hangi kalıp kullanılır?". Kalıp verileri `dw-pat` anahtarında, yedeğe dahildir.
- İlerleme localStorage'da saklanır (her okuma/yazma try/catch içinde, depolama boşsa sayfa yine çalışır).
- Görünüm (lüks tema): açık modda fildişi/krem zemin ve koyu altın vurgu, koyu modda siyaha yakın zemin ve parlak altın vurgu; seviye renkleri mücevher tonlarında (A1 zümrüt, A2 safir, B1 ametist). Altın degrade butonlar ve ilerleme çubuğu, başlıkta altın degrade yazı, Google Fonts'tan Fraunces yazı tipi (yedek: Georgia), yumuşak gölgeler, kartlarda giriş animasyonu.
- Kelime görselleri: Her kelime için bir resim, kelime ve telaffuzun sağında büyük bir kutuda gösterilir (Bugün kartları, Önceki listesi ve Tekrar kartlarında; Tekrar kartlarında anlam açıldıktan sonra; "Kendimi sına" modunda bulanık; Quizde cevabı ele vermemek için gösterilmez). Resimler `IMG` nesnesinde kelime → data URI (webp) olarak gömülür; `IMG` boşsa veya kelime için resim yoksa `EM` dizisindeki emoji gösterilir. NOT: Bu prompt dosyasında `IMG` boş bırakıldı (resimler dosyayı çok büyütür); resimli tam sürüm ayrı `daily-words.html` dosyasındadır.
- Telefon uyumlu (viewport-fit=cover, safe-area boşlukları), açık/koyu tema desteği, arayüz Türkçe.

**Kod (aynen kullan):**

```html
<!DOCTYPE html>
<html lang="tr">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>Günlük Kelimeler</title>
<link rel="manifest" href="manifest.webmanifest">
<meta name="theme-color" content="#f4efe5" media="(prefers-color-scheme: light)">
<meta name="theme-color" content="#0d0f13" media="(prefers-color-scheme: dark)">
<meta name="mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-title" content="Kelimeler">
<link rel="apple-touch-icon" href="apple-touch-icon.png">
<link rel="icon" href="icon-192.png">
<link rel="preconnect" href="https://fonts.googleapis.com"><link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Fraunces:ital,opsz,wght@0,9..144,600;0,9..144,700;1,9..144,400&display=swap" rel="stylesheet">
<style>
:root{--bg:#faf7f2;--card:#fff;--ink:#1f2430;--mute:#6b7280;--line:#e8e2d8;--accent:#c2410c;--a1:#15803d;--a2:#1d4ed8;--b1:#a21caf;--hl:#fde68a;color-scheme:light;box-sizing:border-box;padding-top:env(safe-area-inset-top,0px);padding-bottom:env(safe-area-inset-bottom,0px)}
@media (prefers-color-scheme:dark){:root:not([data-theme="light"]){--bg:#14161c;--card:#1d2028;--ink:#f3f1ec;--mute:#9aa1ae;--line:#2c303b;--accent:#fb923c;--a1:#4ade80;--a2:#60a5fa;--b1:#e879f9;--hl:#78530f;color-scheme:dark}}
html{scroll-padding-top:env(safe-area-inset-top,0px)}
body{margin:0;background:var(--bg);color:var(--ink);font-family:Georgia,"Times New Roman",serif;line-height:1.5}
main{max-width:560px;margin:0 auto;padding:28px 18px 48px}
h1{font-size:30px;margin:0 0 4px;letter-spacing:-.5px}
.date{color:var(--mute);font-family:system-ui,sans-serif;font-size:14px}
.status{margin:16px 0 22px;padding:10px 14px;border:1px dashed var(--line);border-radius:10px;font:14px system-ui,sans-serif;color:var(--mute)}
.status b{color:var(--accent)}
.card{background:var(--card);border:1px solid var(--line);border-radius:16px;padding:18px 20px;margin-bottom:14px}
.top{display:flex;align-items:center;justify-content:space-between;gap:10px}
.lvl{font:700 12px system-ui,sans-serif;padding:3px 9px;border-radius:99px;border:1.5px solid currentColor}
.word{font-size:34px;font-weight:700;margin:8px 0 0}
.ipa{font:17px system-ui,"Segoe UI",sans-serif;color:var(--mute);margin-bottom:10px}
.say{font:14px system-ui,sans-serif;background:none;border:1px solid var(--line);color:var(--ink);padding:6px 12px;border-radius:99px;cursor:pointer}
.say:hover{border-color:var(--accent);color:var(--accent)}
.sent{font-size:18px;margin:0;padding-top:10px;border-top:1px solid var(--line)}
mark{background:var(--hl);color:inherit;padding:0 3px;border-radius:3px}
.prog{background:var(--card);border:1px solid var(--line);border-radius:16px;padding:14px 18px;margin-bottom:16px;font:14px system-ui,sans-serif}
.row{display:flex;justify-content:space-between;align-items:center;gap:8px;margin:4px 0}
.bar{height:8px;background:var(--line);border-radius:9px;overflow:hidden;margin:8px 0}
.bar i{display:block;height:100%;width:0;background:var(--accent);transition:width .3s}
.dots{display:flex;gap:6px}.dot{width:14px;height:14px;border-radius:50%;border:2px solid var(--line)}
.dot.on{background:var(--a1);border-color:var(--a1)}.dot.today{outline:2px solid var(--accent);outline-offset:2px}
.tr{font:600 20px system-ui,sans-serif;color:var(--accent);margin:0 0 10px}
.steps{display:flex;flex-wrap:wrap;gap:4px 14px;margin-top:12px;padding-top:8px;border-top:1px solid var(--line);font:15px system-ui,sans-serif}
.steps label{display:flex;align-items:center;gap:6px;min-height:38px;cursor:pointer}
.steps input{width:20px;height:20px;accent-color:var(--accent)}
.sentTr{font:15px system-ui,sans-serif;color:var(--mute);margin:6px 0 0}
.tabs{display:flex;gap:6px;margin:14px 0 4px;background:var(--line);padding:4px;border-radius:12px}
.tabs button{flex:1;border:0;background:none;color:var(--mute);font:600 14px system-ui,sans-serif;padding:10px 6px;border-radius:9px;cursor:pointer;min-height:40px}
.tabs button.on{background:var(--card);color:var(--ink);box-shadow:0 1px 3px rgba(0,0,0,.15)}
.hbar{display:flex;flex-wrap:wrap;gap:8px;margin:12px 0 4px}
.hbar .say.on{background:var(--accent);color:#fff;border-color:var(--accent)}
.day{font:700 13px system-ui,sans-serif;color:var(--mute);text-transform:uppercase;letter-spacing:.5px;margin:18px 0 8px}
.hw{background:var(--card);border:1px solid var(--line);border-radius:14px;padding:12px 16px;margin-bottom:8px}
.hl{display:flex;align-items:center;gap:8px;flex-wrap:wrap}
.hl b{font-size:22px}.ipa2{font:15px system-ui,sans-serif;color:var(--mute);flex:1}
.tr2{font:600 17px system-ui,sans-serif;color:var(--accent);margin:4px 0}
.hs{font:14px system-ui,sans-serif;color:var(--mute)}.hst{display:block}
.q .tr2,.q .hst{filter:blur(7px);cursor:pointer}.q .tr2.show,.q .hst.show{filter:none}
/* ---- görünüm katmanı ---- */
:root{--bg:#f7f1e8;--accent:#e2561b;--glow:rgba(226,86,27,.16);--shadow:0 8px 24px rgba(70,45,20,.09)}
@media (prefers-color-scheme:dark){:root:not([data-theme="light"]){--glow:rgba(251,146,60,.14);--shadow:0 8px 24px rgba(0,0,0,.4)}}
body{font-family:"Fraunces",Georgia,"Times New Roman",serif;background:radial-gradient(760px 380px at 95% -90px,var(--glow),transparent 70%),var(--bg)}
h1{font-family:"Fraunces",Georgia,serif;font-size:36px;font-weight:700;letter-spacing:-1px;margin:0}
.date{text-transform:uppercase;letter-spacing:1.2px;font-size:12px;margin-top:2px}
.tabs{border-radius:16px;background:var(--line);margin:18px 0 6px}
.tabs button{border-radius:12px;transition:background .2s,color .2s}
.status{border:1px solid var(--line);background:var(--card);border-radius:14px;box-shadow:var(--shadow)}
.prog{border-radius:20px;box-shadow:var(--shadow);padding:16px 20px}
.prog #pnum{font:700 17px system-ui,sans-serif}
#streak{background:var(--glow);color:var(--accent);padding:4px 10px;border-radius:99px;font-weight:600}
.bar{height:10px}.bar i{background:linear-gradient(90deg,var(--accent),#f5a524);border-radius:9px}
.dot{width:16px;height:16px}
.card{--lc:var(--accent);position:relative;overflow:hidden;border-radius:24px;padding:20px 22px 14px 26px;box-shadow:var(--shadow);animation:rise .5s ease both}
.card::before{content:"";position:absolute;left:0;top:0;bottom:0;width:6px;background:var(--lc)}
.card.a1{--lc:var(--a1)}.card.a2{--lc:var(--a2)}.card.b1{--lc:var(--b1)}
.card:nth-child(2){animation-delay:.08s}.card:nth-child(3){animation-delay:.16s}
@keyframes rise{from{opacity:0;transform:translateY(12px)}to{opacity:1;transform:none}}
@media (prefers-reduced-motion:reduce){.card{animation:none}}
.lvl{border:0;background:color-mix(in srgb,currentColor 14%,transparent);letter-spacing:.5px}
.word{font-size:42px;letter-spacing:-1.2px;line-height:1.1;margin:10px 0 6px}
.ipa{display:inline-block;background:var(--line);padding:2px 12px;border-radius:99px;font-size:15px;margin-bottom:12px}
.tr{font-size:22px;margin:2px 0 12px}
.top .say{background:var(--accent);color:#fff;border-color:var(--accent);font-weight:600;padding:8px 15px}
.sent{font-style:italic;font-size:19px;line-height:1.45;border-top:1px dashed var(--line)}
mark{background:linear-gradient(transparent 55%,var(--hl) 55%);font-style:normal;font-weight:600;padding:0}
.sentTr{font-size:15px;margin-top:8px}
.steps{gap:8px;border-top:0;padding-top:4px}
.steps label{border:1px solid var(--line);border-radius:99px;padding:4px 12px 4px 10px;min-height:38px;font-size:14px;transition:background .2s,border-color .2s}
.steps label:has(input:checked){background:color-mix(in srgb,var(--a1) 16%,transparent);border-color:var(--a1)}
.steps input{accent-color:var(--a1)}
.day{display:flex;align-items:center;gap:8px}.day::before{content:"";width:8px;height:8px;border-radius:50%;background:var(--accent)}
.hw{border-radius:18px;box-shadow:var(--shadow);padding:14px 18px}
.hl b{font-size:24px;letter-spacing:-.5px}
.tabs button{font-size:13px;padding:10px 2px}
.rate{display:flex;gap:8px;align-items:center;margin-top:10px;flex-wrap:wrap}
.bx{font:12px system-ui,sans-serif;color:var(--mute);flex:1;min-width:80px}
.rb{font:600 14px system-ui,sans-serif;border:1px solid var(--line);background:var(--card);color:var(--ink);padding:9px 14px;border-radius:99px;cursor:pointer;min-height:40px}
.rb.yes{background:color-mix(in srgb,var(--a1) 16%,transparent);border-color:var(--a1)}
.rb.no{background:color-mix(in srgb,#dc2626 12%,transparent);border-color:#dc2626}
.rb.show{background:var(--accent);color:#fff;border-color:var(--accent);width:100%;margin-top:8px}
.qh{font:600 13px system-ui,sans-serif;color:var(--mute);text-transform:uppercase;letter-spacing:1px;margin:6px 0 8px}
.qp{font-size:24px;line-height:1.35;margin:4px 0 12px}
.opt{display:block;width:100%;text-align:left;font:600 17px system-ui,sans-serif;background:var(--card);color:var(--ink);border:1.5px solid var(--line);border-radius:14px;padding:13px 16px;margin-bottom:8px;cursor:pointer;min-height:48px}
.opt.ok{border-color:var(--a1);background:color-mix(in srgb,var(--a1) 18%,transparent)}
.opt.bad{border-color:#dc2626;background:color-mix(in srgb,#dc2626 14%,transparent)}
#v3 .card,#v4 .card{animation:none}
.goal{margin:12px 0 0;padding:10px 14px;border-radius:14px;font:600 14px system-ui,sans-serif;border:1px dashed var(--line);color:var(--mute);background:var(--card)}
.goal.ok{border:1px solid var(--a1);color:var(--a1);background:color-mix(in srgb,var(--a1) 14%,transparent)}
.rb.flag.on{background:color-mix(in srgb,var(--accent) 18%,transparent);border-color:var(--accent);color:var(--accent)}
.rv .card{margin-bottom:12px}
/* ---- lüks tema ---- */
:root{--bg:#f4efe5;--card:#fffdf8;--ink:#1a1c22;--mute:#6d675c;--line:#e5dbc8;--accent:#96701f;--a1:#2b8a67;--a2:#3569cc;--b1:#8a58c4;--hl:#efdca4;--glow:rgba(169,130,47,.2);--shadow:0 10px 30px rgba(60,45,15,.1);--onacc:#1b1405}
@media (prefers-color-scheme:dark){:root:not([data-theme="light"]){--bg:#0d0f13;--card:#171a21;--ink:#f3eee3;--mute:#9b978d;--line:#2a2d36;--accent:#d8b36c;--a1:#4fc79a;--a2:#7aa2f7;--b1:#b79cf5;--hl:#6b5521;--glow:rgba(216,179,108,.13);--shadow:0 10px 30px rgba(0,0,0,.55)}}
h1{background:linear-gradient(135deg,var(--ink) 30%,var(--accent));-webkit-background-clip:text;background-clip:text;color:transparent;-webkit-text-fill-color:transparent}
.card,.hw,.prog,.status,.goal{border-color:color-mix(in srgb,var(--accent) 28%,var(--line))}
.top .say,.rb.show,.hbar .say.on{background:linear-gradient(135deg,#ecd495,#b8923f);border-color:transparent;color:var(--onacc)}
.bar i{background:linear-gradient(90deg,#b8923f,#f1d890)}
.tabs button.on{color:var(--accent)}
.wh{display:flex;align-items:center;gap:14px;margin:8px 0 6px}
.wh .word{margin:0 0 4px;font-size:38px}.wh .ipa{margin-bottom:0}
.pic{flex:none;width:68px;height:68px;border-radius:20px;display:grid;place-items:center;font-size:38px;background:color-mix(in srgb,var(--lc,var(--accent)) 13%,transparent);border:1px solid color-mix(in srgb,var(--lc,var(--accent)) 38%,transparent)}
.pe{font-size:24px}.q .pe{filter:blur(6px)}
.wh{justify-content:space-between;gap:10px}
.wh>div:first-child{flex:1;min-width:0}
.wh .word{font-size:clamp(28px,8.5vw,40px);overflow-wrap:anywhere}
.pic{width:clamp(96px,32vw,140px);height:clamp(92px,28vw,124px);padding:6px;box-sizing:border-box;background:radial-gradient(circle at 50% 45%,color-mix(in srgb,var(--lc,var(--accent)) 18%,transparent),transparent 75%);border:0;border-radius:24px;font-size:56px}
.pic img{max-width:100%;max-height:100%;object-fit:contain;filter:drop-shadow(0 6px 8px rgba(0,0,0,.28))}
.pe{width:46px;height:46px;display:inline-grid;place-items:center}.pe img{max-width:100%;max-height:100%}
.tabs{overflow-x:auto}.tabs button{font-size:12px;padding:10px 0;white-space:nowrap}
.card.p{--lc:var(--accent)}.lvl.p{color:var(--accent)}.card.p .word{font-size:clamp(22px,6.5vw,28px);line-height:1.25;margin:8px 0}
.foot{margin-top:28px;padding-top:14px;border-top:1px dashed var(--line);font:13px system-ui,sans-serif;color:var(--mute)}
.foot .say{margin:0 8px 8px 0}.foot textarea{width:100%;box-sizing:border-box;height:90px;margin-top:8px;border-radius:12px;border:1px solid var(--line);background:var(--card);color:var(--ink);font:12px monospace;padding:8px}
.a1{color:var(--a1)}.a2{color:var(--a2)}.b1{color:var(--b1)}
</style>
</head>
<body>
<main>
  <h1>Günlük Kelimeler</h1>
  <div class="date" id="date"></div>
  <div class="goal" id="goal"></div>
  <nav class="tabs"><button id="t1" class="on">Bugün</button><button id="t2">Önceki</button><button id="t3">🔁 Tekrar</button><button id="t4">🎯 Quiz</button><button id="t5">💬 Kalıp</button></nav>
  <div id="v1"><div class="status" id="status"></div>
  <section class="prog"><div class="row"><b id="pnum"></b><span id="streak"></span></div><div class="bar"><i id="pbar"></i></div><div class="row"><span class="dots" id="dots"></span><span id="msg"></span></div><div class="row"><span id="qres"></span><span id="due"></span></div></section>
  <div id="cards"></div></div>
  <div id="v2" hidden></div>
  <div id="v3" hidden></div>
  <div id="v4" hidden></div>
  <div id="v5" hidden></div>
  <footer class="foot"><button class="say" id="bk">💾 Yedekle</button><button class="say" id="rs">♻️ Geri yükle</button><div id="bkmsg">İlerlemen bu cihazda saklanır.</div><textarea id="bkta" hidden></textarea></footer>
</main>
<script>
var BANK={
A1:[
"apple|ˈæp.əl|I eat an apple every morning.","book|bʊk|She reads a book before bed.","chair|tʃer|Please sit on the chair.","door|dɔːr|Close the door, please.","happy|ˈhæp.i|The children are happy today.","water|ˈwɔː.t̬ɚ|I drink water after running.","friend|frend|My friend lives near the park.","house|haʊs|Their house has a red roof.","green|ɡriːn|The grass is green in spring.","big|bɪɡ|He has a big dog.","eat|iːt|We eat dinner at seven.","walk|wɑːk|I walk to work every day.","family|ˈfæm.əl.i|My family visits me on Sundays.","school|skuːl|The school opens at eight.","morning|ˈmɔːr.nɪŋ|The morning sun is warm.","window|ˈwɪn.doʊ|Open the window, it is hot.","table|ˈteɪ.bəl|The keys are on the table.","small|smɑːl|This room is small but clean.","music|ˈmjuː.zɪk|She listens to music on the bus.","cat|kæt|The cat is sleeping on the sofa.","bread|bred|I buy fresh bread every day.","red|red|He wore a red scarf.","sleep|sliːp|Babies sleep a lot.","shop|ʃɑːp|The shop closes at six.","rain|reɪn|I like the sound of rain.","night|naɪt|It is very quiet at night.","hand|hænd|Wash your hands with soap.","clock|klɑːk|The clock on the wall is slow.","phone|foʊn|My phone is on the desk.","smile|smaɪl|Her smile makes everyone happy."],
A2:[
"borrow|ˈbɑːr.oʊ|Can I borrow your pen?","ticket|ˈtɪk.ɪt|I bought a train ticket online.","weather|ˈweð.ɚ|The weather is lovely today.","invite|ɪnˈvaɪt|I want to invite you to my party.","journey|ˈdʒɝː.ni|The journey to Ankara takes five hours.","parcel|ˈpɑːr.səl|A parcel arrived for you this morning.","tired|ˈtaɪ.ɚd|He was tired after the long walk.","neighbour|ˈneɪ.bɚ|Our neighbour feeds the cat when we travel.","cheap|tʃiːp|These shoes were very cheap.","grow|ɡroʊ|Tomatoes grow well in summer.","improve|ɪmˈpruːv|I practise every day to improve my English.","kitchen|ˈkɪtʃ.ən|She is cooking soup in the kitchen.","holiday|ˈhɑː.lə.deɪ|We spent our holiday by the sea.","enjoy|ɪnˈdʒɔɪ|I enjoy walking by the water.","mistake|mɪˈsteɪk|It was a small mistake, so don’t worry.","order|ˈɔːr.dɚ|Let’s order pizza tonight.","advice|ədˈvaɪs|My teacher gave me good advice.","empty|ˈemp.ti|The fridge is empty again.","repair|rɪˈper|He can repair bikes.","tidy|ˈtaɪ.di|Please keep your room tidy.","popular|ˈpɑː.pjə.lɚ|This café is popular with students.","queue|kjuː|We waited in a long queue.","vegetable|ˈvedʒ.tə.bəl|Carrot is my favourite vegetable.","wallet|ˈwɑː.lɪt|I lost my wallet on the tram.","noisy|ˈnɔɪ.zi|The street is noisy at night.","arrive|əˈraɪv|The bus will arrive at nine.","lonely|ˈloʊn.li|He felt lonely in the new city.","lend|lend|Could you lend me ten lira?","dangerous|ˈdeɪn.dʒɚ.əs|The road is dangerous in winter.","useful|ˈjuːs.fəl|A map is useful when you travel."],
B1:[
"achieve|əˈtʃiːv|She worked hard to achieve her goal.","argue|ˈɑːr.ɡjuː|They argue about small things.","benefit|ˈben.ə.fɪt|Exercise is of great benefit to your health.","challenge|ˈtʃæl.ɪndʒ|Learning a language is a fun challenge.","consider|kənˈsɪd.ɚ|Please consider my offer carefully.","convince|kənˈvɪns|I couldn’t convince him to come.","damage|ˈdæm.ɪdʒ|The storm caused serious damage.","determine|dɪˈtɝː.mɪn|The test will determine your level.","efficient|ɪˈfɪʃ.ənt|The new system is fast and efficient.","environment|ɪnˈvaɪ.rən.mənt|We must protect the environment.","familiar|fəˈmɪl.jɚ|Her face looks familiar to me.","guarantee|ˌɡer.ənˈtiː|The shop gives a two-year guarantee.","influence|ˈɪn.flu.əns|Friends can influence your choices.","opportunity|ˌɑː.pɚˈtuː.nə.t̬i|This job is a great opportunity.","prefer|prɪˈfɝː|I prefer tea to coffee.","reliable|rɪˈlaɪ.ə.bəl|He is a reliable colleague.","refuse|rɪˈfjuːz|She had to refuse the invitation.","suggest|səɡˈdʒest|I suggest we leave early.","tend|tend|I tend to wake up early.","valuable|ˈvæl.jə.bəl|Time is more valuable than money.","willing|ˈwɪl.ɪŋ|Are you willing to help us?","afford|əˈfɔːrd|We can’t afford a new car.","average|ˈæv.rɪdʒ|The average age in the class is twenty.","behave|bɪˈheɪv|The children behave well at school.","complain|kəmˈpleɪn|Don’t complain about the food.","delay|dɪˈleɪ|The storm caused a delay at the airport.","flexible|ˈflek.sə.bəl|My boss is flexible about working hours.","generous|ˈdʒen.ɚ.əs|He is generous with his time.","ignore|ɪɡˈnɔːr|Try to ignore the noise.","nervous|ˈnɝː.vəs|I felt nervous before the interview."]
};
var TR={
A1:["elma","kitap","sandalye","kapı","mutlu","su","arkadaş","ev","yeşil","büyük","yemek yemek","yürümek","aile","okul","sabah","pencere","masa","küçük","müzik","kedi","ekmek","kırmızı","uyumak","dükkân, mağaza","yağmur","gece","el","saat (duvar saati)","telefon","gülümseme"],
A2:["ödünç almak","bilet","hava durumu","davet etmek","yolculuk","paket, koli","yorgun","komşu","ucuz","büyümek, yetişmek","geliştirmek","mutfak","tatil","keyif almak","hata","sipariş vermek","tavsiye","boş","tamir etmek","düzenli, derli toplu","popüler, sevilen","sıra (kuyruk)","sebze","cüzdan","gürültülü","varmak, ulaşmak","yalnız (yalnızlık hisseden)","ödünç vermek","tehlikeli","faydalı"],
B1:["başarmak","tartışmak","fayda, yarar","zorluk, meydan okuma","düşünmek, göz önünde bulundurmak","ikna etmek","hasar, zarar","belirlemek","verimli","çevre","tanıdık","garanti","etkilemek, etki","fırsat","tercih etmek","güvenilir","reddetmek","önermek","eğilimli olmak","değerli","istekli","karşılayabilmek (maddi olarak)","ortalama","davranmak","şikâyet etmek","gecikme","esnek","cömert","görmezden gelmek","gergin, endişeli"]};
var ST={
A1:["Her sabah bir elma yerim.","Yatmadan önce kitap okur.","Lütfen sandalyeye otur.","Lütfen kapıyı kapat.","Çocuklar bugün mutlu.","Koştuktan sonra su içerim.","Arkadaşım parkın yakınında yaşıyor.","Onların evinin kırmızı bir çatısı var.","İlkbaharda çimenler yeşildir.","Onun büyük bir köpeği var.","Akşam yemeğini saat yedide yeriz.","Her gün işe yürüyerek giderim.","Ailem pazar günleri beni ziyaret eder.","Okul saat sekizde açılır.","Sabah güneşi sıcaktır.","Pencereyi aç, hava sıcak.","Anahtarlar masanın üstünde.","Bu oda küçük ama temiz.","Otobüste müzik dinler.","Kedi koltukta uyuyor.","Her gün taze ekmek alırım.","Kırmızı bir atkı takmıştı.","Bebekler çok uyur.","Dükkân saat altıda kapanır.","Yağmur sesini severim.","Geceleri çok sessizdir.","Ellerini sabunla yıka.","Duvardaki saat geri kalıyor.","Telefonum masanın üstünde.","Gülümsemesi herkesi mutlu eder."],
A2:["Kalemini ödünç alabilir miyim?","İnternetten tren bileti aldım.","Bugün hava çok güzel.","Seni partime davet etmek istiyorum.","Ankara’ya yolculuk beş saat sürer.","Bu sabah sana bir paket geldi.","Uzun yürüyüşten sonra yorgundu.","Biz seyahat edince komşumuz kediyi besler.","Bu ayakkabılar çok ucuzdu.","Domates yazın iyi yetişir.","İngilizcemi geliştirmek için her gün pratik yapıyorum.","Mutfakta çorba pişiriyor.","Tatilimizi deniz kenarında geçirdik.","Su kenarında yürümekten keyif alırım.","Küçük bir hataydı, merak etme.","Bu akşam pizza sipariş edelim.","Öğretmenim bana iyi bir tavsiye verdi.","Buzdolabı yine boş.","Bisikletleri tamir edebilir.","Lütfen odanı düzenli tut.","Bu kafe öğrenciler arasında çok sevilir.","Uzun bir sırada bekledik.","Havuç en sevdiğim sebzedir.","Cüzdanımı tramvayda kaybettim.","Sokak geceleri gürültülüdür.","Otobüs dokuzda varacak.","Yeni şehirde kendini yalnız hissetti.","Bana on lira ödünç verir misin?","Yol kışın tehlikelidir.","Seyahat ederken harita faydalıdır."],
B1:["Hedefine ulaşmak için çok çalıştı.","Küçük şeyler yüzünden tartışırlar.","Egzersiz sağlığına büyük fayda sağlar.","Dil öğrenmek eğlenceli bir meydan okumadır.","Lütfen teklifimi dikkatlice düşün.","Onu gelmeye ikna edemedim.","Fırtına ciddi hasara yol açtı.","Test seviyeni belirleyecek.","Yeni sistem hızlı ve verimli.","Çevreyi korumalıyız.","Yüzü bana tanıdık geliyor.","Mağaza iki yıl garanti veriyor.","Arkadaşlar seçimlerini etkileyebilir.","Bu iş harika bir fırsat.","Çayı kahveye tercih ederim.","O güvenilir bir iş arkadaşı.","Daveti reddetmek zorunda kaldı.","Erken çıkmamızı öneririm.","Erken uyanmaya eğilimliyim.","Zaman paradan daha değerlidir.","Bize yardım etmeye istekli misin?","Yeni bir arabayı karşılayamayız.","Sınıftaki yaş ortalaması yirmi.","Çocuklar okulda iyi davranır.","Yemekten şikâyet etme.","Fırtına havalimanında gecikmeye neden oldu.","Patronum çalışma saatleri konusunda esnektir.","Zamanı konusunda cömerttir.","Gürültüyü görmezden gelmeye çalış.","Mülakattan önce gergin hissettim."]};
var HOUR=9;
function store(){var s=null;try{s=localStorage.getItem("dw-v1");}catch(e){}try{if(s)return JSON.parse(s);}catch(e){}return window.__m||{};}
function save(o){window.__m=o;try{localStorage.setItem("dw-v1",JSON.stringify(o));}catch(e){}}
function load(idx){return store()[idx]||[];}
function toggle(idx,j,v){var o=store(),a=o[idx]||[];a[j]=v;o[idx]=a;save(o);render();}
function done(a){var c=0;for(var j=0;j<9;j++)if(a&&a[j])c++;return c;}
function progress(idx){
  var o=store(),c=done(o[idx]);
  document.getElementById("pnum").textContent="Bugün: "+c+" / 9 adım";
  document.getElementById("pbar").style.width=(c/9*100)+"%";
  var s=0,d=(c===9)?idx:idx-1;while(done(o[d])===9){s++;d--;}
  document.getElementById("streak").textContent="🔥 "+s+" gün üst üste";
  var dots="";for(var k=6;k>=0;k--){dots+="<span class=\"dot"+(done(o[idx-k])===9?" on":"")+(k===0?" today":"")+"\"></span>";}
  document.getElementById("dots").innerHTML=dots;
  document.getElementById("msg").textContent=c===9?"Harika, bugün tamam! 🎉":"";
  var qo=gget('dw-quiz')[idx];
  document.getElementById("qres").textContent=qo?'🎯 Quiz: '+qo.c+'/'+qo.t:'🎯 Quiz: henüz yok';
  var rr=gget('dw-rev')[idx];document.getElementById("due").textContent='🔁 Tekrar: '+(rr?Object.keys(rr.d).length:0)+'/'+(rr?rr.k.length:4);
  var ok=qo&&qo.t>=10,g=document.getElementById("goal");g.className=ok?'goal ok':'goal';g.textContent=ok?'✅ Günlük hedef tamamlandı! ('+qo.c+'/'+qo.t+' doğru)':'🎯 Günlük hedef: 10 soruluk quizi tamamla';
}
var showAll=false,hideTr=false;
function startDay(idx){
  var s=null;try{s=localStorage.getItem("dw-start");}catch(e){}
  var v=s?parseInt(s,10):(window.__s||0),o=store(),m=idx;
  if(v&&v<m)m=v;
  Object.keys(o).forEach(function(x){x=Number(x);if(done(o[x])>0&&x<m)m=x;});
  if(!v||m<v){window.__s=m;try{localStorage.setItem("dw-start",String(m));}catch(e){}}
  return m;
}
function wordsFor(day){
  return Object.keys(BANK).map(function(lv){var l=BANK[lv],k=((day%l.length)+l.length)%l.length,p=l[k].split("|");return{lv:lv,w:p[0],ipa:p[1],s:p[2],tr:TR[lv][k],st:ST[lv][k]};});
}
function renderHistory(idx){
  var box=document.getElementById("v2");box.innerHTML="";box.className=hideTr?"q":"";
  var first=startDay(idx),lo=showAll?idx-30:Math.max(first,idx-30),n=0,html="";
  var bar=document.createElement("div");bar.className="hbar";
  var b1=document.createElement("button");b1.className="say"+(hideTr?" on":"");b1.textContent="🧠 Kendimi sına";
  b1.onclick=function(){hideTr=!hideTr;renderHistory(idx);};
  var b2=document.createElement("button");b2.className="say"+(showAll?" on":"");b2.textContent="Son 30 günü göster";
  b2.onclick=function(){showAll=!showAll;renderHistory(idx);};
  bar.appendChild(b1);bar.appendChild(b2);box.appendChild(bar);
  for(var d=idx-1;d>=lo;d--){
    n++;
    html+="<div class='day'>"+new Date(d*864e5).toLocaleDateString("tr-TR",{timeZone:"UTC",weekday:"long",day:"numeric",month:"long"})+"</div>";
    wordsFor(d).forEach(function(x){
      html+="<div class='hw'><div class='hl'><span class='lvl "+x.lv.toLowerCase()+"'>"+x.lv+"</span><span class='pe'>"+pimg(x)+"</span><b>"+x.w+"</b><span class='ipa2'>/"+x.ipa+"/</span><button class='say' data-w='"+x.w+"'>🔊</button></div><div class='tr2'>"+x.tr+"</div><div class='hs'>"+x.s+"<span class='hst'>🇹🇷 "+x.st+"</span></div><div class='rate'><span class='bx'>"+bx(x)+"</span><button class='rb no' data-lv='"+x.lv+"' data-w='"+x.w+"' data-v='0'>✗ Bilmiyorum</button><button class='rb yes' data-lv='"+x.lv+"' data-w='"+x.w+"' data-v='1'>✓ Biliyorum</button><button class='rb flag"+(isFlag(x)?" on":"")+"' data-lv='"+x.lv+"' data-w='"+x.w+"' data-v='2'>🔁 Tekrar edilecek</button></div></div>";
    });
  }
  if(!n)html="<div class='status'>Henüz önceki kelime yok. Yarın saat 09:00’dan sonra bugünün kelimeleri burada görünecek. Daha eski günleri görmek için “Son 30 günü göster”e dokun.</div>";
  var w=document.createElement("div");w.innerHTML=html;box.appendChild(w);
  Array.prototype.forEach.call(box.querySelectorAll(".say[data-w]"),function(b){b.onclick=function(){speak(b.getAttribute("data-w"));};});
  Array.prototype.forEach.call(box.querySelectorAll(".tr2,.hst"),function(e){e.onclick=function(){e.classList.toggle("show");};});
  Array.prototype.forEach.call(box.querySelectorAll('.rb[data-v]'),function(b){b.onclick=function(){var o2={lv:b.getAttribute('data-lv'),w:b.getAttribute('data-w')},v=b.getAttribute('data-v');if(v==='2')toggleFlag(o2);else rate(o2,v==='1',idx);renderHistory(idx);progress(idx);};});
}
var rvOpen={},qz=null,IV=[0,1,3,7,14,30];
function gid(i){return document.getElementById(i);}
function gget(k){var s=null;try{s=localStorage.getItem(k);}catch(e){}try{if(s)return JSON.parse(s);}catch(e){}return (window.__g&&window.__g[k])||{};}
function gset(k,o){window.__g=window.__g||{};window.__g[k]=o;try{localStorage.setItem(k,JSON.stringify(o));}catch(e){}}
function wkey(x){return x.lv+':'+x.w;}
function rate(x,known,idx){var o=gget('dw-srs'),r=o[wkey(x)]||{b:0},b=known?Math.min(5,(r.b||0)+1):0;o[wkey(x)]={b:b,due:idx+IV[b]};gset('dw-srs',o);if(known){var f=gget('dw-flag');delete f[wkey(x)];gset('dw-flag',f);}}
function isFlag(x){return !!gget('dw-flag')[wkey(x)];}
function toggleFlag(x){var f=gget('dw-flag'),k=wkey(x);if(f[k])delete f[k];else f[k]=1;gset('dw-flag',f);}
function bx(x){var r=gget('dw-srs')[wkey(x)];return (isFlag(x)?'🔁 ':'')+(r?`📦 Kutu ${r.b}/5`:'🆕 yeni');}
function seenWords(idx){var lo=Math.max(startDay(idx),idx-29),a=[];for(var d=idx;d>=lo;d--)a=a.concat(wordsFor(d));return a;}
function dueList(idx){var o=gget('dw-srs');return seenWords(idx).filter(function(x){var r=o[wkey(x)];return !r||r.due<=idx;});}
function allWords(){var a=[];Object.keys(BANK).forEach(function(lv){BANK[lv].forEach(function(row,k){var p=row.split('|');a.push({lv:lv,w:p[0],ipa:p[1],s:p[2],tr:TR[lv][k],st:ST[lv][k]});});});return a;}
function shuffle(a){for(var i=a.length-1;i>0;i--){var j=Math.floor(Math.random()*(i+1)),t=a[i];a[i]=a[j];a[j]=t;}return a;}
function opts(x,key,all){var o=[x[key]];shuffle(all.slice()).forEach(function(y){if(o.length<4&&o.indexOf(y[key])<0)o.push(y[key]);});return shuffle(o);}
function buildQuiz(idx){
  var pool=shuffle(seenWords(idx)),all=allWords(),q=[];
  for(var i=0;i<10;i++){
    var x=pool[i%pool.length],t=(i+Math.floor(i/pool.length))%3,o={x:x,t:t};
    o.a=t===0?x.tr:x.w;o.o=opts(x,t===0?'tr':'w',all);
    o.p=x.s.replace(new RegExp('\\b'+x.w+'\\w*','i'),'_____');
    q.push(o);
  }
  var ps=shuffle(seenPats(idx));
  if(ps.length){var pa=allPats();[3,4].forEach(function(t,j){var px=ps[j%ps.length],o={x:px,t:t};o.a=t===3?px.tr:px.w;o.o=opts(px,t===3?'tr':'w',pa);q[8+j]=o;});}
  return q;
}
function qsave(idx,c,t){var o=gget('dw-quiz'),p=o[idx];if(!p||c/t>p.c/p.t)o[idx]={c:c,t:t};gset('dw-quiz',o);}
function renderQuiz(idx){
  var box=gid('v4');
  if(!qz)qz={i:0,c:0,a:null,q:buildQuiz(idx)};
  var N=qz.q.length;gset('dw-qz',{d:idx,qz:qz});
  if(qz.i>=N){
    qsave(idx,qz.c,N);progress(idx);
    box.innerHTML=`<div class="card"><div class="qh">Quiz bitti</div><div class="qp">${qz.c} / ${N} doğru ${qz.c===N?'🎉':qz.c>=N/2?'👏':'💪'}</div><p class="sentTr">✅ Günlük hedef tamamlandı! Sonuç bugünün ilerlemesine eklendi. Yanlışların tekrar listesinde daha sık çıkacak.</p><button class="rb show" id="qagain">Yeniden başla</button></div>`;
    gid('qagain').onclick=function(){qz=null;renderQuiz(idx);};return;
  }
  var q=qz.q[qz.i],h=`<div class="qh">Soru ${qz.i+1} / ${N} · ✓ ${qz.c}</div><div class="card"><div class="qh">${['Anlam','Boşluk doldur','Dinle','Kalıp: anlam','Kalıp: cümle'][q.t]}</div>`;
  if(q.t===0)h+=`<div class="word">${q.x.w}</div><div class="ipa">/${q.x.ipa}/</div><div class="qp">Türkçe anlamı nedir?</div>`;
  if(q.t===1)h+=`<div class="qp">${q.p}</div><div class="sentTr">🇹🇷 ${q.x.st}</div>`;
  if(q.t===2)h+=`<button class="rb show" id="qsay">🔊 Tekrar dinle</button>`;
  if(q.t===3)h+=`<div class="qp">${q.x.w}</div><div class="sentTr">Bu kalıp ne anlama gelir?</div>`;
  if(q.t===4)h+=`<div class="qp">${q.x.st}</div><div class="sentTr">Bu cümlede hangi kalıp kullanılır?</div>`;
  h+='<div style="height:10px"></div>';
  q.o.forEach(function(op,j){var c='opt';if(qz.a!==null){if(op===q.a)c+=' ok';else if(op===qz.a)c+=' bad';}h+=`<button class="${c}" data-j="${j}">${op}</button>`;});
  if(qz.a!==null)h+=`<button class="rb show" id="qnext">${qz.i+1>=N?'Sonucu gör':'Sonraki'}</button>`;
  box.innerHTML=h+'</div>';
  Array.prototype.forEach.call(box.querySelectorAll('.opt'),function(b){b.onclick=function(){if(qz.a!==null)return;var op=q.o[+b.getAttribute('data-j')];qz.a=op;var ok=op===q.a;if(ok)qz.c++;rate(q.x,ok,idx);renderQuiz(idx);};});
  if(q.t===2){gid('qsay').onclick=function(){speak(q.x.w);};if(qz.a===null)setTimeout(function(){speak(q.x.w);},250);}
  if(qz.a!==null)gid('qnext').onclick=function(){qz.i++;qz.a=null;renderQuiz(idx);};
}
function seededRand(seed){var a=seed>>>0;return function(){a=(a+0x6D2B79F5)>>>0;var t=a;t=Math.imul(t^t>>>15,t|1);t^=t+Math.imul(t^t>>>7,t|61);return((t^t>>>14)>>>0)/4294967296;};}
function revKeys(idx){
  var all=gget('dw-rev');
  if(all[idx]&&all[idx].k&&all[idx].k.length){var q0=all[idx];if(!q0.k.some(function(k){return k.indexOf('P:')===0;})){q0.k.push(patKey(idx));all[idx]=q0;gset('dw-rev',all);}return q0;}
  var lo=Math.max(startDay(idx),idx-29),pool=[];
  for(var d=idx-1;d>=lo;d--)pool=pool.concat(wordsFor(d));
  var fl=gget('dw-flag'),srs=gget('dw-srs'),rnd=seededRand(idx*7919+13);
  function sh(a){a=a.slice();for(var i=a.length-1;i>0;i--){var j=Math.floor(rnd()*(i+1)),t=a[i];a[i]=a[j];a[j]=t;}return a;}
  function weak(x){var s=srs[wkey(x)];return fl[wkey(x)]||(s&&s.b===0);}
  var prio=sh(pool.filter(weak)),oth=sh(pool.filter(function(x){return !weak(x);}));
  var pick=prio.slice(0,2).concat(oth,prio.slice(2));
  if(pick.length<3)pick=pick.concat(wordsFor(idx));
  all[idx]={k:pick.slice(0,3).map(wkey).concat(patKey(idx)),d:{}};
  gset('dw-rev',all);
  return all[idx];
}
function renderReview(idx){
  var box=gid('v3'),r=revKeys(idx),all=allWords().concat(allPats());
  var items=r.k.map(function(k){return all.filter(function(x){return wkey(x)===k;})[0];}).filter(Boolean);
  var n=Object.keys(r.d).length;
  var h=`<div class="status">${n>=items.length?'🎉 Günlük tekrar tamamlandı!':'Bugünün tekrar kartları: '+n+' / '+items.length}</div><div class="rv">`;
  items.forEach(function(x,i){
    var l=x.lv.toLowerCase(),k=wkey(x),st=r.d[k],open=rvOpen[k]||st;
    h+=`<div class="card ${l}"><div class="top"><span class="lvl ${l}">${x.lv==='P'?'KALIP':x.lv}</span><button class="say" data-w="${x.lv==='P'?x.s:x.w}">🔊 Dinle</button></div><div class="wh"><div><div class="word">${x.w}</div>${x.ipa?`<div class="ipa">/${x.ipa}/</div>`:''}</div>${open?`<div class="pic">${pimg(x)}</div>`:''}</div>`;
    if(open)h+=`<div class="tr">🇹🇷 ${x.tr}</div><p class="sent">${x.s}</p><p class="sentTr">🇹🇷 ${x.st}</p>`;
    if(st)h+=`<div class="bx">${st==='1'?'✓ Biliyorum olarak işaretlendi':'✗ Bilmiyorum olarak işaretlendi'}</div>`;
    else if(open)h+=`<div class="rate"><button class="rb no" data-i="${i}" data-v="0">✗ Bilmiyorum</button><button class="rb yes" data-i="${i}" data-v="1">✓ Biliyorum</button></div>`;
    else h+=`<button class="rb show" data-i="${i}" data-v="s">Anlamı göster</button>`;
    h+='</div>';
  });
  box.innerHTML=h+'</div>';
  Array.prototype.forEach.call(box.querySelectorAll('.say[data-w]'),function(b){b.onclick=function(){speak(b.getAttribute('data-w'));};});
  Array.prototype.forEach.call(box.querySelectorAll('.rb[data-i]'),function(b){b.onclick=function(){
    var x=items[+b.getAttribute('data-i')],v=b.getAttribute('data-v');
    if(v==='s'){rvOpen[wkey(x)]=1;}
    else{rate(x,v==='1',idx);r.d[wkey(x)]=v;var a=gget('dw-rev');a[idx]=r;gset('dw-rev',a);progress(idx);}
    renderReview(idx);
  };});
}
var PAT=[
"How about + V-ing?|…ne dersin? / …sak nasıl olur?|How about going for a walk?|Yürüyüşe çıksak nasıl olur?",
"I’d like to + V|…mak istiyorum (kibar)|I’d like to book a table for two.|İki kişilik masa ayırtmak istiyorum.",
"Would you mind + V-ing?|…manın sakıncası var mı?|Would you mind opening the window?|Pencereyi açmanın sakıncası var mı?",
"I’m looking forward to + V-ing|…mayı sabırsızlıkla bekliyorum|I’m looking forward to seeing you.|Seni görmeyi sabırsızlıkla bekliyorum.",
"Could you please + V?|…ebilir misiniz?|Could you please repeat that?|Bunu tekrar eder misiniz?",
"I’m not sure if …|…olup olmadığından emin değilim|I’m not sure if he is at home.|Evde olup olmadığından emin değilim.",
"It depends on …|…a bağlı|It depends on the weather.|Havaya bağlı.",
"I used to + V|eskiden …erdim|I used to play football every day.|Eskiden her gün futbol oynardım.",
"Let me + V|…ayım / izin ver…eyim|Let me help you with that.|Sana bununla yardım edeyim.",
"What do you think about …?|…hakkında ne düşünüyorsun?|What do you think about this idea?|Bu fikir hakkında ne düşünüyorsun?",
"I’m supposed to + V|…mem gerekiyor / …mam bekleniyor|I’m supposed to meet him at six.|Onunla altıda buluşmam gerekiyor.",
"As soon as …|…r …maz|Call me as soon as you arrive.|Varır varmaz beni ara.",
"I can’t wait to + V|…mak için sabırsızlanıyorum|I can’t wait to see the results.|Sonuçları görmek için sabırsızlanıyorum.",
"It’s worth + V-ing|…maya değer|This museum is worth visiting.|Bu müze görülmeye değer.",
"I’m afraid …|Korkarım ki… / Maalesef…|I’m afraid I can’t come tonight.|Korkarım bu akşam gelemem.",
"Do you mind if I + V?|…sam sakıncası var mı?|Do you mind if I sit here?|Buraya oturmamın sakıncası var mı?",
"I’ve been + V-ing for …|…dır …yorum|I’ve been learning English for a year.|Bir yıldır İngilizce öğreniyorum.",
"It’s time to + V|…ma zamanı|It’s time to go home.|Eve gitme zamanı.",
"I’d rather + V|…mayı tercih ederim|I’d rather stay at home today.|Bugün evde kalmayı tercih ederim.",
"There’s no point in + V-ing|…manın anlamı yok|There’s no point in waiting any longer.|Daha fazla beklemenin anlamı yok.",
"What’s the matter with …?|…in nesi var?|What’s the matter with your phone?|Telefonunun nesi var?",
"I’m used to + V-ing|…a alışkınım|I’m used to waking up early.|Erken kalkmaya alışkınım.",
"If I were you, I would + V|Yerinde olsam …erdim|If I were you, I would take a break.|Yerinde olsam mola verirdim.",
"Thanks for + V-ing|…diğin için teşekkürler|Thanks for helping me.|Bana yardım ettiğin için teşekkürler.",
"I don’t feel like + V-ing|canım …mak istemiyor|I don’t feel like cooking tonight.|Bu akşam yemek yapmak istemiyorum.",
"By the way, …|Bu arada, …|By the way, did you call Ali?|Bu arada, Ali’yi aradın mı?",
"As far as I know, …|Bildiğim kadarıyla …|As far as I know, the shop is open.|Bildiğim kadarıyla dükkân açık.",
"I’m in the middle of + V-ing|şu an …mın tam ortasındayım|I’m in the middle of a meeting.|Şu an bir toplantının ortasındayım.",
"It’s up to you.|Sana kalmış.|Where we eat is up to you.|Nerede yiyeceğimiz sana kalmış.",
"Make sure (that) …|…diğinden emin ol|Make sure you lock the door.|Kapıyı kilitlediğinden emin ol."];
function patIdx(d){return ((d%PAT.length)+PAT.length)%PAT.length;}
function patObj(i){var p=PAT[i].split('|');return{lv:'P',w:p[0],ipa:'',tr:p[1],s:p[2],st:p[3]};}
function allPats(){return PAT.map(function(_,i){return patObj(i);});}
function seenPats(idx){var lo=Math.max(startDay(idx),idx-29),a=[];for(var d=idx;d>=lo;d--)a.push(patObj(patIdx(d)));return a;}
function patKey(idx){
  var lo=Math.max(startDay(idx),idx-29),pool=[];
  for(var d=idx-1;d>=lo;d--)pool.push(patObj(patIdx(d)));
  if(!pool.length)pool=[patObj(patIdx(idx))];
  var fl=gget('dw-flag'),srs=gget('dw-srs'),rnd=seededRand(idx*104729+7);
  var weak=pool.filter(function(x){var r=srs[wkey(x)];return fl[wkey(x)]||(r&&r.b===0);}),list=weak.length?weak:pool;
  return wkey(list[Math.floor(rnd()*list.length)]);
}
function renderPat(idx){
  var box=gid('v5'),t=patObj(patIdx(idx)),dn=gget('dw-pat')[idx];
  var h=`<div class="status">Her gün 1 yeni kalıp · yenisi 09:00’da gelir</div><div class="card p"><div class="top"><span class="lvl p">KALIP</span><button class="say" id="psay">🔊 Dinle</button></div><div class="word">${t.w}</div><div class="tr">🇹🇷 ${t.tr}</div><p class="sent">${t.s}</p><p class="sentTr">🇹🇷 ${t.st}</p><div class="rate"><button class="rb ${dn?'yes':''}" id="pdone">${dn?'✓ Öğrendim':'Öğrendim olarak işaretle'}</button><button class="rb flag${isFlag(t)?' on':''}" id="pflag">🔁 Tekrar edilecek</button></div></div>`;
  var lo=Math.max(startDay(idx),idx-29),first=true;
  for(var d=idx-1;d>=lo;d--){
    var i=patIdx(d),p=patObj(i);
    if(first){h+='<div class="qh" style="margin-top:20px">Önceki kalıplar</div>';first=false;}
    h+=`<div class="hw"><div class="hl"><b style="font-size:20px">${p.w}</b></div><div class="tr2">${p.tr}</div><div class="hs">${p.s}<span class="hst">🇹🇷 ${p.st}</span></div><div class="rate"><span class="bx">${bx(p)}</span><button class="rb no" data-i="${i}" data-v="0">✗ Bilmiyorum</button><button class="rb yes" data-i="${i}" data-v="1">✓ Biliyorum</button><button class="rb flag${isFlag(p)?' on':''}" data-i="${i}" data-v="2">🔁 Tekrar edilecek</button></div></div>`;
  }
  box.innerHTML=h;
  gid('psay').onclick=function(){speak(t.s);};
  gid('pdone').onclick=function(){var o=gget('dw-pat');if(o[idx])delete o[idx];else o[idx]=1;gset('dw-pat',o);renderPat(idx);};
  gid('pflag').onclick=function(){toggleFlag(t);renderPat(idx);};
  Array.prototype.forEach.call(box.querySelectorAll('.rb[data-i]'),function(b){b.onclick=function(){
    var p=patObj(+b.getAttribute('data-i')),v=b.getAttribute('data-v');
    if(v==='2')toggleFlag(p);else rate(p,v==='1',idx);
    renderPat(idx);progress(idx);
  };});
}
function setTab(n){
  for(var i=1;i<=5;i++){gid('v'+i).hidden=i!==n;gid('t'+i).className=i===n?'on':'';}
  var idx=dayIndex(new Date());
  if(n===1)render();
  if(n===2)renderHistory(idx);
  if(n===3){rvOpen={};renderReview(idx);}
  if(n===5)renderPat(idx);
  if(n===4){var sv=gget('dw-qz');qz=(sv&&sv.d===idx&&sv.qz)?sv.qz:null;renderQuiz(idx);}
}
var EM={
A1:["🍎","📖","🪑","🚪","😊","💧","🤝","🏠","🟢","🐘","🍴","🚶","👨‍👩‍👧","🏫","🌅","🪟","🍽️","🐜","🎵","🐱","🍞","🔴","😴","🛍️","🌧️","🌙","✋","🕒","📱","😄"],
A2:["🤲","🎫","⛅","💌","🧳","📦","🥱","🏘️","🏷️","🌱","📈","🍳","🏖️","🥳","❌","🛎️","💡","📭","🔧","🧹","⭐","👥","🥕","👛","📢","🛬","😔","💵","⚠️","👍"],
B1:["🏆","🗯️","🎁","🧗","🤔","🗣️","💥","🎯","⚡","🌍","👋","🛡️","🧲","🔑","⚖️","⚓","🙅","💭","🧭","💎","🙋","💳","📊","🎩","😤","⏳","🤸","🤗","🙉","😰"]};
var EMW={};Object.keys(BANK).forEach(function(lv){BANK[lv].forEach(function(row,k){EMW[row.split('|')[0]]=EM[lv][k];});});
function emo(x){return EMW[x.w]||(x.lv==='P'?'💬':'📘');}
var IMG={}; /* Buraya {"apple":"data:image/webp;base64,..."} biçiminde kelime resimleri eklenebilir; boşsa emoji gösterilir. */
function pimg(x){return IMG[x.w]?'<img src="'+IMG[x.w]+'" alt="'+x.w+'">':emo(x);}
function dayIndex(now){
  var d=Math.floor(Date.UTC(now.getFullYear(),now.getMonth(),now.getDate())/864e5);
  return now.getHours()<HOUR?d-1:d;
}
function speak(t){
  try{var u=new SpeechSynthesisUtterance(t);u.lang="en-US";u.rate=.85;speechSynthesis.cancel();speechSynthesis.speak(u);}catch(e){}
}
function render(){
  var now=new Date(),idx=dayIndex(now),box=document.getElementById("cards");
  box.innerHTML="";
  var st=load(idx);Object.keys(BANK).forEach(function(lv,n){
    var list=BANK[lv],k=((idx%list.length)+list.length)%list.length,p=list[k].split("|"),tr=TR[lv][k],sT=ST[lv][k];
    var w=p[0],ipa=p[1],s=p[2];
    var c=document.createElement("div");c.className="card "+lv.toLowerCase();
    var top=document.createElement("div");top.className="top";
    var badge=document.createElement("span");badge.className="lvl "+lv.toLowerCase();badge.textContent=lv;
    var btn=document.createElement("button");btn.className="say";btn.textContent="🔊 Dinle";
    btn.onclick=function(){speak(w);setTimeout(function(){speak(s);},900);if(!st[n*3])toggle(idx,n*3,true);};
    top.appendChild(badge);top.appendChild(btn);
    var h=document.createElement("div");h.className="word";h.textContent=w;
    var i=document.createElement("div");i.className="ipa";i.textContent="/"+ipa+"/";
    var sp=document.createElement("p");sp.className="sent";
    var m=s.match(new RegExp("\\b"+w+"\\w*","i"));
    if(m){
      sp.appendChild(document.createTextNode(s.slice(0,m.index)));
      var mk=document.createElement("mark");mk.textContent=m[0];sp.appendChild(mk);
      sp.appendChild(document.createTextNode(s.slice(m.index+m[0].length)));
    }else sp.textContent=s;
    var t=document.createElement("div");t.className="tr";t.textContent="🇹🇷 "+tr;
    var stp=document.createElement("div");stp.className="steps";
    ["Dinle","Sesli tekrar et","Kendi cümleni kur"].forEach(function(txt,j){
      var lb=document.createElement("label"),cb=document.createElement("input");
      cb.type="checkbox";cb.checked=!!st[n*3+j];
      cb.onchange=function(){toggle(idx,n*3+j,cb.checked);};
      lb.appendChild(cb);lb.appendChild(document.createTextNode(txt));stp.appendChild(lb);
    });
    var wh=document.createElement("div");wh.className="wh";var pic=document.createElement("div");pic.className="pic";pic.innerHTML=pimg({w:w});var wt=document.createElement("div");wt.appendChild(h);wt.appendChild(i);wh.appendChild(wt);wh.appendChild(pic);c.appendChild(top);c.appendChild(wh);var sx=document.createElement("p");sx.className="sentTr";sx.textContent="🇹🇷 "+sT;
    c.appendChild(t);c.appendChild(sp);c.appendChild(sx);c.appendChild(stp);box.appendChild(c);
  });
  var next=new Date(now.getFullYear(),now.getMonth(),now.getDate(),HOUR,0,0);
  if(now>=next)next.setDate(next.getDate()+1);
  var mins=Math.round((next-now)/6e4),hh=Math.floor(mins/60),mm=mins%60;
  document.getElementById("date").textContent=now.toLocaleDateString("tr-TR",{weekday:"long",day:"numeric",month:"long",year:"numeric"});
  document.getElementById("status").innerHTML="Yeni kelimeler her gün <b>09:00</b>’da açılır — "+hh+" sa "+mm+" dk sonra.";
  progress(idx);
}
document.getElementById("t1").onclick=function(){setTab(1);};
document.getElementById("t2").onclick=function(){setTab(2);};
document.getElementById("t3").onclick=function(){setTab(3);};
document.getElementById("t4").onclick=function(){setTab(4);};
document.getElementById("t5").onclick=function(){setTab(5);};
var BK=['dw-v1','dw-start','dw-srs','dw-flag','dw-rev','dw-quiz','dw-qz','dw-pat'];
function backupText(){var o={};BK.forEach(function(k){var v=null;try{v=localStorage.getItem(k);}catch(e){}if(v)o[k]=v;});return JSON.stringify(o);}
gid('bk').onclick=function(){var t=backupText(),ta=gid('bkta'),m='Yedek hazır: metni kopyalayıp notlarına yapıştırarak sakla.';ta.hidden=false;ta.value=t;ta.select();
  try{navigator.clipboard.writeText(t).then(function(){gid('bkmsg').textContent='Kopyalandı ✓ Notlarına yapıştırıp sakla.';},function(){gid('bkmsg').textContent=m;});}catch(e){gid('bkmsg').textContent=m;}};
gid('rs').onclick=function(){var ta=gid('bkta');
  if(ta.hidden||!ta.value.trim()){ta.hidden=false;ta.value='';ta.placeholder='Yedek metnini buraya yapıştır, sonra tekrar “Geri yükle”ye bas';gid('bkmsg').textContent='';ta.focus();return;}
  try{var o=JSON.parse(ta.value),n=0;BK.forEach(function(k){if(typeof o[k]==='string'){localStorage.setItem(k,o[k]);n++;}});
    gid('bkmsg').textContent=n?'Geri yüklendi ✓':'Geçersiz yedek';if(n)setTimeout(function(){location.reload();},600);}catch(e){gid('bkmsg').textContent='Geçersiz yedek metni';}};
try{if('serviceWorker' in navigator&&/^https?:/.test(location.protocol))navigator.serviceWorker.register('sw.js').catch(function(){});}catch(e){}
try{if(navigator.storage&&navigator.storage.persist)navigator.storage.persist();}catch(e){}
render();setInterval(render,30000);
</script>
</body>
</html>

```

Kodu tek bir `.html` dosyası olarak ver. İstersen kelime listesini genişletebilir, ama yapıyı bozma.
