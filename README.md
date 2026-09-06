# Tatoeba — sade çeviri arayüzü

Tatoeba'da **Japonca → Türkçe** çeviri yapmak için tek dosyalık arayüz.
Kurulum yok, sunucu yok, bağımlılık yok: `index.html`'e çift tıkla.

Klasörü olduğu gibi taşıyabilirsin, başka hiçbir şeye bağlı değil.

## Neden

Tatoeba'nın kendi arayüzünde iki sorun vardı:

1. İngilizce çeviri ekranda duruyor, göz otomatik ona kayıyor.
2. Her seferinde siteye girip aynı filtreleri yeniden yazmak gerekiyor.

Bu araç ikisini de çözüyor: ekranda **sadece Japonca cümle** var, ve kuyruk
zaten "Türkçesi olmayan" diye filtrelenmiş geliyor.

## Kullanım

Açılışta iki paketten birini seçersin:

| Paket | Çeviri | Ne zaman |
|---|---|---|
| **kısa** | 3 | gün içinde fırsat bulunca |
| **normal** | 30 | oturup çalışmak için |

İkisi de **bitirilebilir** — hedefe varınca "paket bitti" ekranı gelir, sonsuz
akmaz. Bitirme hissi olmadan "kısa paket" anlamsız olurdu.

| Tuş | İş |
|---|---|
| `Enter` | kaydet & sonraki cümle |
| `Tab` | ingilizce ipucunu aç/kapa |
| `Shift+Tab` | furigana (kanji okumaları) aç/kapa |
| `Esc` | bu cümleyi atla |
| `Alt+S` | anadili konuşmacı sesini çal / durdur |
| `Alt+C` | orijinal Japonca cümleyi kopyala |
| `Alt+J` | cümleyi Jisho'da ara |

Üst şeritteki araç düğmeleri:
- **`[ a | A ]`**: Japonca cümlenin yazı boyutunu küçültür / büyütür (seçilen boyut tarayıcıda kalıcı olarak saklanır).
- **`🔊 dinle`**: Anadili Japonca olan konuşmacının ses kaydını çalar.
- **`kopyala`**: Orijinal Japonca cümleyi panoya kopyalar.
- **`Jisho ↗`**: Cümleyi otomatik olarak `jisho.org/search/<cümle>` adresinde açar.
- **`ふりがな`**: Kanji okumalarını açar veya gizler.

### Bağlamsal & Zincirleme Çeviri (Extensive Reading)

Rastgele cümleler yerine öğrenmeyi pekiştiren bir zincir algoritması çalışır:
1. İlk cümle tohum olarak çekilir (`✨ tohum cümle`).
2. Cümlenin içindeki temel kanji bileşikleri ve anlamlı içerik kelimeleri ayrıştırılır (örneğin `車` ve `運転`).
3. Sonraki cümleler bu kelimeler etrafında aranarak getirilir (`🔗 zincir: 車`, `🔗 zincir: 運転`).
4. Gelen cümlelerdeki yeni kelimelerle bağlam doğal olarak genişler; böylece aynı kavramları farklı cümlelerde art arda görerek pekiştirirsin.
5. Açılış ekranından **`[🔊 sesli cümleler]`** filtresini açıp kapatabilirsin.

**Atlamak hedefe saymaz** — `Esc`'lediğin cümlenin yerine yenisi gelir, 3'lük
paket yine 3 çeviriyle biter. Boş `Enter` de saymaz. Bu yüzden hedeften fazla
cümle çekiliyor (`hedef + 7`, en az 12): atladıkça yeni istek atmak gerekmesin.

Yazma alanı klasik terminal komut satırı gibi: `❯` istemi, tek aralıklı yazı tipi
ve yanıp sönen kehribar blok imleç (eski amber CRT terminallerin rengi).

> Blok imleç neden bu kadar kod: tarayıcı `caret-color` ile imlecin *rengini*
> değiştirmeye izin veriyor ama *şeklini* değiştirmeye izin vermiyor. Bu yüzden
> textarea görünmez yapılıp metin `#yansima` içinde kendimiz çiziliyor ve blok,
> imlecin bulunduğu karakterin üstüne konuyor. Ok tuşuyla metnin ortasına
> gittiğinde blok da oraya gider — sona sabitlenmiş bir kare yanlış olurdu.
>
> Yanıp sönme `opacity` ile değil, `background`/`color` ile yapılıyor: `opacity:0`
> altındaki harfi de yok ediyordu, oysa gerçek terminalde imleç sönünce karakter
> normal haliyle görünmeye devam eder.

Çeviriler tarayıcının yerel deposunda birikir. **`gönderim turu →`** düğmesiyle
listeye geçersin; her satırdaki **kopyala & aç**, çeviriyi panoya alıp cümleyi
Tatoeba'da açar → çeviri ikonuna tıkla → `Ctrl+V` → Enter.

Yanlışlıkla "gönderildi" işaretlenen bir satıra tekrar tıklarsan işaret kalkar.

> **Yedek al:** yerel depo tarayıcıya ve dosya yoluna bağlı. Dosyayı başka bir
> yere taşımadan veya tarayıcı verilerini temizlemeden önce **JSON indir**'e bas.

## Ayarlar

`index.html` içinde, script'in en başında:

```js
const KAYNAK_DIL  = "jpn";   // çevirdiğin dil
const HEDEF_DIL   = "tur";   // çevirdiğin dile
const IPUCU_DIL   = "eng";   // takılınca göstereceği yardımcı dil
const KISA_ADET   = 3;       // "kısa" pakette kaç çeviri
const NORMAL_ADET = 30;      // "normal" pakette kaç çeviri
```

Paket sayılarını buradan değiştirirsen açılış ekranındaki etiketler de kendiliğinden
güncellenir — sayı tek yerde duruyor.

## Tatoeba API notları

Hepsi doğrulandı (2026-09-05). Bunlar dokümante edilmemiş, deneyerek bulundu.

**Uç nokta:** `https://api.tatoeba.org/unstable/sentences`

| Parametre | Not |
|---|---|
| `lang=jpn` | kaynak dil |
| `%21trans%3Alang=tur` | `!trans:lang=tur` → **Türkçesi olmayan** cümleler |
| `sort=random` | **zorunlu** — yoksa `400` döner |
| `showtrans=all` | çevirileri de getir (`all` \| `none` \| `matching`) |
| `include=transcriptions` | furigana verisi (hazır `<ruby>` HTML'i ile) |

Havuz büyüklükleri: 249.051 Japonca cümlenin 153.678'inin Türkçesi zaten var,
**95.373'ü çevrilmeyi bekliyor**. Örneklenen 40 cümlenin %100'ünde furigana
verisi vardı ama **%67'si doğrulanmamış** (otomatik üretilmiş) — arayüz bunu
cümlenin üstüne gelince söylüyor.

CORS açık (`access-control-allow-origin: *`, `Origin: null` ile bile), o yüzden
`file://` üzerinden sunucusuz çalışıyor.

### Dikkat: API salt okunur

`POST /unstable/sentences` → **403**. Çeviri gönderimi API'den yapılamaz,
tatoeba.org üzerinden elle yapılmak zorunda.

### Dikkat: `add_translation` diye bir eylem YOK

`tatoeba.org/en/sentences/add_translation/{id}` **çalışmaz** — girişli kullanıcıyı
ana sayfaya atar. Çıkışken login sayfasına yönlenmesi o rotanın var olduğu
anlamına gelmiyor; Tatoeba giriş kontrolünü eylemi çözmeden önce yapıyor.
`SentencesController`'da sadece `show()` ve `save_translation()` var.

**Doğru hedef:** `tatoeba.org/en/sentences/show/{id}`

Çeviri kutusunu doğrudan açan bir URL de yok — `show()` böyle bir parametre
almıyor. Cümle sayfasındaki çeviri ikonuna bir tık gerekiyor, bu kaçınılmaz.

## Güvenlik notu

Furigana, API'den **hazır HTML** olarak geliyor. Dışarıdan gelen HTML doğrudan
`innerHTML`'e basılmıyor; `furiganaDugumu()` onu ayrıştırıp sadece
`ruby/rb/rt/rp` etiketlerini, hiçbir öznitelik almadan yeniden kuruyor.
Geri kalan her şey eleniyor.
