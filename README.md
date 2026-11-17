# İstanbul Otobüs Doluluk ve Yolculuk Süresi Tahmini

Kamuya açık verilerle İstanbul'daki İETT otobüs hatları için, seçilen sefer ve durak aralığında **ortalama doluluk oranını** ve **yolculuk süresini** tahmin eden açıklanabilir bir model ve web uygulaması.

Bu depo, TÜBİTAK 2209-A Üniversite Öğrencileri Araştırma Projeleri Destekleme Programı'na sunulan araştırma önerisinin ön çalışmasıdır.

- **Proje yürütücüsü:** Mehmet Emir Durmuş, Nişantaşı Üniversitesi, Uygulamalı Bilimler Fakültesi
- **Durum:** Prototip (ön çalışma). Tahminler henüz saha verisiyle doğrulanmamıştır.

> **Uyarı:** Bu uygulama bir araştırma prototipidir; İETT'nin ya da İBB'nin resmî bir ürünü değildir ve bu kurumlarla bağlantılı değildir. Tahminler yolculuk planlaması için kesin bilgi olarak kullanılmamalıdır.

---

## Ne yapıyor?

Kullanıcı hattı yazar, güzergâhı seçer, biniş ve iniş durağını seçer. Uygulama tarih, saat, okul ve üniversite takvimi, resmi tatiller, trafik ve hava koşullarını otomatik olarak dikkate alır ve şunları verir:

- Seçilen durak aralığında beklenen **doluluk oranı** (sapma aralığıyla),
- Seçilen duraklar arasındaki **yolculuk süresi**,
- Her tahminin hangi veriye ve hangi katsayıya dayandığını gösteren **hesaplama dökümü**.

Kapsam: 841 İETT otobüs hattı, 13.427 durak. Metrobüs hatları (34 serisi) biniş verisi istasyon bazlı tutulduğu için kapsam dışıdır.

## Hızlı başlangıç

**Seçenek 1: Tek dosya (internet gerekmez).** `Otobus_Doluluk.html` dosyasını çift tıklayarak tarayıcıda açın. Bütün veri dosyanın içine gömülüdür.

**Seçenek 2: Klasörden çalıştırma.** Tarayıcılar `file://` adresinden ayrı veri dosyalarını okumadığı için yerel bir sunucu gerekir:

```bash
cd uygulama
python -m http.server 8000
# tarayıcıda http://localhost:8000
```

**Kaynaktan derleme.** `uygulama_kaynak/` klasöründe `python build.py` komutu `index.html` dosyasını üretir.

## Klasör yapısı

```
uygulama/            Çalışan uygulama (index.html + data/)
uygulama_kaynak/     Kaynak kod
  model.js           Tahmin modeli
  app.js             Arayüz ve veri yükleme
  template.html      Sayfa iskeleti ve stiller
  proj.html          "Proje ve Yöntem" sekmesi
  build.py           index.html üretir
veri_hazirlama/      Veri işleme betikleri
  build_lines.py         İETT web servisleri + GTFS → hat, durak ve sefer verisi
  build_ridership.py     İBB saatlik biniş verisi → hat profilleri ve okul etkisi
  build_app_data.py      Hepsini + trafik indeksi → uygulama/data
  hat_verisi_hazirla.py  Tek bir hattın verisini ayıklar
Otobus_Doluluk.html  Verisi gömülü, tek dosyalık sürüm
```

## Yöntem (özet)

1. **Saatlik hat talebi:** Hattın gün tipine göre saatlik ortalama binişi İBB biniş verisinden hesaplanır.
2. **Takvim düzeltmesi:** Okulların kapalı olduğu günlerde her saatin talebi, o hattın veriden öğrenilen okul kapalı/açık oranıyla çarpılır. Resmi tatiller ve üniversite dönemi otomatik algılanır.
3. **Sefere paylaştırma:** Saatlik biniş, o saatte yolda olan otobüs-dakikalarına oranla seferlere dağıtılır.
4. **Durak bazlı dağıtım:** Durak çevresindeki okul, üniversite, hastane, aktarma noktası ve iş merkezleri, saate bağlı biniş/iniş ağırlıkları verir; yolculuk uzunluğu dağılımıyla durak durak yük profili çıkarılır.
5. **Doluluk:** Seçilen aralıktaki en yüksek yük, araç kapasitesine bölünür. Araç tipi kurala göre belirlenir.
6. **Yolculuk süresi:** Tarife süresi İBB trafik indeksi ve hava koşuluyla düzeltilir.

Ayrıntılar uygulamadaki **Proje ve Yöntem** sekmesindedir.

## Ön çalışma bulguları

- Okullar açıkken hafta içi toplam otobüs binişi yaklaşık 3,84 milyon, kapalıyken 3,41 milyondur (Eylül 2024, yaklaşık %11 fark).
- 15:00–16:00 arasında hatların ortancasında okullar kapalıyken biniş %37 daha düşüktür. Etki hattan hatta çok değişir (SM6: ×0,19; 500T: ×0,94).
- Hafta içi trafik indeksi 08:00'de ortalama 63, 18:00'de 79'dur; İETT tarifesindeki sefer süreleri ise gün boyu sabittir.
- **Veri kalitesi:** İETT GTFS `stop_times.csv` dosyası 1.048.575 satırda (Excel satır sınırı) kesilmiştir ve ara durak saatleri boştur. Saatlik biniş verisinin Eylül 2024 kaydı yalnızca birkaç gün için eksiksizdir. Bu boşluklar İETT web servisleriyle tamamlanmıştır.

## Planlanan doğrulama

Proje kapsamında model üç katmanda doğrulanacaktır:

| Katman | Veri | Ölçüt |
|---|---|---|
| D1 – Biniş (ağ ölçeği) | İBB biniş verisinin modele verilmeyen günleri | Hat-saat binişi WAPE ≤ %20; hat bazlı okul etkisi |
| D2 – Yolculuk süresi | İETT anlık araç konum servisinden kaydedilen gerçekleşen seferler | MAPE ≤ %15 |
| D3 – Durak aralığı yükü | 41N, SM6 ve 15F hatlarında durak bazlı saha sayımı (en az 24 sefer) | MAPE ≤ %25; doluluk kategorisi doğruluğu ≥ %70 |

Saha sayımlarında yalnızca anonim yolcu sayıları kaydedilir; fotoğraf, kişisel veri veya yolcu görüşmesi yoktur.

## Bilinen sınırlılıklar

- Kamuya açık biniş verisi iniş ve yön bilgisi içermez; durak bazlı yük bir model çıktısıdır.
- Yön dengesizliği henüz modellenmemiştir; yoğun tek yönlü saatlerde doluluk düşük tahmin edilebilir.
- Biniş profilleri Eylül 2024'e, tarife 2026'ya aittir.
- Ara durak süreleri mesafeye göre dağıtılır; araç tipi açık veride bulunmadığı için kurala dayalıdır.
- Hava koşulu katsayıları literatüre dayalı başlangıç değerleridir.

## Veri kaynakları ve atıf

Bu projede kullanılan veriler aşağıdaki kaynaklardan alınmış ve işlenmiştir. `uygulama/data/` klasöründeki dosyalar bu verilerden türetilmiştir; özgün veri setleri değildir.

| Kaynak | Kullanılan veri | Lisans / koşul |
|---|---|---|
| [İBB Açık Veri Portalı](https://data.ibb.gov.tr) | İETT GTFS verisi (Mart 2026), Saatlik Toplu Ulaşım Veri Seti (Eylül 2024), Trafik İndeksi web servisi (20 Ekim – 17 Kasım 2026) | İstanbul Büyükşehir Belediyesi Açık Veri Lisansı |
| İETT web servisleri (api.ibb.gov.tr) | Hat-Durak-Güzergâh, Planlanan Sefer Saati (17 Kasım 2026), Sefer Gerçekleşme / anlık araç konum | İBB Açık Veri Portalı üzerinden yayımlanan servisler |
| [Open-Meteo](https://open-meteo.com) | Geçmiş hava durumu (ERA5) – proje süresinde kullanılacak | CC BY 4.0 |
| Millî Eğitim Bakanlığı | 2026-2027 Eğitim-Öğretim Yılı Çalışma Takvimi | Kamuya açık |

**Kaynak: İBB Açık Veri Portalı.** Veriler İstanbul Büyükşehir Belediyesi tarafından yayımlanmıştır. Bu depodaki işlenmiş veriler ve tahminler İBB'nin ya da İETT'nin görüşlerini yansıtmaz ve bu kurumlar tarafından onaylanmamıştır. Veri kullanımında lisans metninin güncel hâli esas alınmalıdır.

## Yapay zekâ kullanımı beyanı

Bu projenin ön çalışmasında yapay zekâ destekli bir asistandan (Anthropic Claude) yararlanılmıştır. Asistan; uygulama kodunun ve veri işleme betiklerinin önemli bir kısmının yazılmasında, verilerin toplanması ve analizinde ve proje belgelerinin taslaklarının hazırlanmasında kullanılmıştır. Araştırma fikri, problem tanımı, yöntem tercihleri ve tüm sonuçların kontrolü proje yürütücüsüne aittir. Sonuçlar proje yürütücüsü tarafından gözden geçirilmiştir.

## Lisans

- **Kaynak kod:** MIT Lisansı – ayrıntılar için [LICENSE](LICENSE) dosyasına bakınız.
- **Veriler:** `uygulama/data/` içindeki türetilmiş veriler, kaynak verilerin lisans koşullarına (İBB Açık Veri Lisansı) tabidir; kullanırken yukarıdaki atıfı koruyunuz.

## Atıf

Bu çalışmayı kullanırsanız lütfen şöyle atıf yapınız:

> Durmuş, M. E. (2026). *İstanbul Otobüs Doluluk ve Yolculuk Süresi Tahmini* [Yazılım]. Nişantaşı Üniversitesi. GitHub deposu.

## İletişim

Sorular ve öneriler için bu depoda bir *issue* açabilirsiniz.
