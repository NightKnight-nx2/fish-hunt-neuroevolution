# fish hunt · neuroevrim

Tek dosyalık (`index.html`), tarayıcıda çalışan, mizahi/eğitici bir **nöroevrim** simülasyonu. Küçük bir sinir ağıyla kontrol edilen bir avcı balık, yem balıklarını avlamayı **genetik algoritma** ile — hiçbir insan müdahalesi olmadan — öğreniyor. Sahne, "Level 1/6 · blind" gibi mizahi bir anlatımla, avcının "kör ve amaçsız"dan "apex" (zirve avcı) seviyesine evrilişini bir zaman çizelgesi (time-lapse) olarak gösteriyor.

> Bu proje, Instagram'daki [**@reelgorithm.py**](https://www.instagram.com/reelgorithm.py) hesabından esinlenen, kısa-video formatında algoritma/yapay-zeka anlatımı denemesidir. Format fikri oradan alınmıştır; kod ve içerik bu depoya özgüdür.

## Canlı demo

`index.html`'i doğrudan bir tarayıcıda açmanız yeterli — harici bir derleme adımı veya bağımlılık kurulumu yok.

- **En basit yol:** Dosyaya çift tıklayın / tarayıcıya sürükleyin.
- **Önerilen yol (Web Worker'ın bazı tarayıcılarda `file://` üzerinden kısıtlanmasına karşı):** basit bir yerel sunucu ile açın:
  ```bash
  # proje klasöründe
  python3 -m http.server 8000
  # ardından tarayıcıda: http://localhost:8000
  ```
  veya
  ```bash
  npx serve .
  ```

### Kontroller / URL parametreleri
- `SPACE` → zaman çizelgesini baştan başlat
- `R` → 9:16 ↔ 1:1 en-boy oranı
- `F` → tam ekran
- `?ratio=1x1`, `?seed=7`, `?gens=60` → URL ile oran, rastgele tohum (seed) ve toplam nesil sayısını ayarlayın

## Nasıl çalışıyor (teknik özet)

- **Görselleştirme / döngü:** [p5.js](https://p5js.org) `1.9.4`, `canvas` üzerinde çiziyor; ana döngü (`setup`/`draw`) sahne akışını (seviyeler, zaman çizelgesi, HUD) yönetiyor.
- **Eğitim:** Simülasyon ve sinir ağı mantığı (`#core` script etiketindeki `Core` modülü) hem ana iş parçacığında hem de bir **Web Worker** içinde çalışıyor; eğitim arka planda, kesintisiz sürüyor ve `postMessage` ile nesil sonuçları ana sahneye akıyor (Worker başarısız olursa `setTimeout` tabanlı bir yedek döngüye düşülüyor).
- **Algı:** Avcının "beyni", en yakın `K=3` yem balığının göreli açısı ve yakınlığını girdi alıyor (toplam 6 girdi).
- **Sinir ağı:** Basit bir **ileri beslemeli (feedforward)** ağ — `girdi (6) → gizli katman (8, tanh) → çıktı (2, tanh)`. Çıktılar avcının dönüş açısı ve hızını belirliyor. Ağırlık sayısı ekranda canlı gösteriliyor (`Core.genomeSize`).
- **Genetik algoritma:**
  - Popülasyon: 50 birey, her biri bir genom (ağırlık dizisi)
  - Uygunluk (fitness): yenen yem sayısı × 10 + hayatta kalınan süre (saniye)
  - Seçilim: turnuva seçimi (`tourn=4`)
  - Elitizm: en iyi 3 birey değişmeden sonraki nesle geçiyor
  - Çaprazlama (crossover): nöron-bazlı — her gizli nöronun giriş ağırlıkları tek bir ebeveynden alınıyor
  - Mutasyon: Gauss gürültüsü, `mutRate=0.07`, `mutSigma=0.22`
  - Göçmenler: her nesilde 3 tamamen rastgele birey eklenerek yerel optimumdan kaçış sağlanıyor
  - Durağanlık tespiti: belirli nesillerde (16, 30) ilerleme yetersizse popülasyon sıfırdan başlatılıyor
- **Sahne kurgusu:** Eğitim geçmişi (`hist[]`) üzerinden 6 seviyelik bir anlatı kuruluyor — her seviye, belirli bir uygunluk eşiğini (`thr`) ilk aşan nesli "kahraman" olarak seçip gösteriyor: `blind → twitchy → curious → hungry → locked on → apex`. Seviyeler arasında hızlandırılmış "evrim" montajları (time-lapse) oynatılıyor.

## Ekran görüntüsü / GIF

Depoda şu an bir ekran görüntüsü veya GIF bulunmuyor.

## Kaynakça / References

| Bağımlılık | Amaç | Lisans | Bağlantı |
|---|---|---|---|
| [p5.js](https://p5js.org) `1.9.4` | Canvas çizimi, animasyon döngüsü (`setup`/`draw`), girdi yönetimi | LGPL-2.1 | [Kaynak kodu](https://github.com/processing/p5.js) · [Lisans sayfası](https://archive.p5js.org/copyright.html) |

### Lisans notu (p5.js ↔ bu projenin lisansı)

p5.js, **cdnjs** üzerinden tek bir `<script src="...">` etiketiyle, çalışma zamanında (runtime) ayrı bir kütüphane olarak yükleniyor. Kütüphanenin kaynak kodu bu depoya **gömülmüyor, statik olarak bağlanmıyor ve değiştirilmiyor** — proje kodu, p5.js'in genel/global API'sini (p5 global mode) sadece çağırıyor.

LGPL (Lesser/Library GPL), tam da bu kullanım şeklini — bir kütüphaneyi değiştirmeden, ayrı bir bileşen olarak (dinamik bağlama / ayrı dosya olarak) kullanmayı — istisna tutar: LGPL'in "copyleft" zorunluluğu, kütüphanenin **kendisinde** yapılan değişikliklere ve onu statik olarak içine gömen türev çalışmalara uygulanır; onu çağıran/kullanan ayrı bir programın lisansını dikte etmez. Bu proje p5.js'i değiştirmediği ve kaynağını gömmediği için, **kendi kodumuza (bu depodaki `index.html`'e) MIT lisansı uygulamak LGPL ile çelişmiyor.**

Sonuc: Bu projenin kendi kodu **MIT** lisanslıdır (`LICENSE` dosyasına bakın); p5.js kutuphanesi kendi **LGPL-2.1** lisansı altında, CDN üzerinden ayrı bir bağımlılık olarak kalır.

## Lisans

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

Bu projenin kendi kodu [MIT Lisansı](LICENSE) ile lisanslanmıştır. © 2026 Musa Buğra Demir.
