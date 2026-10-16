# GEOAI-QUALITY-CONTROL

## Uydu Görüntülerinden Bina Alanlarının U-Net ile Tespiti

**Makine Öğrenmesi ile Probleme Uygun Çözüm Geliştirme — Proje Raporu**

---

## 1. Problem Tanımı ve Projenin Amacı

Bu çalışmanın amacı, yüksek çözünürlüklü RGB uydu görüntülerinden
bina alanlarını otomatik olarak belirleyen bir derin öğrenme modeli geliştirmektir.

## 2. Veri Seti ve Veri Keşfi

| Özellik | Açıklama |
|---|---|
| Veri kaynağı | SpaceNet 2 |
| Bölge | Las Vegas (AOI 2 – Vegas) |
| Görüntü sayısı | 100 |
| Görüntü boyutu | 650 × 650 piksel |
| Renk kanalları | RGB |

## 3. Görsel İnceleme ve Ön İşleme

**Şekil 1. Örnek Uydu Görüntüsü**

<p align="center">
  <img src="https://github.com/user-attachments/assets/a142685c-7b82-4a3c-bb24-f71fd5fe0ee6" width="450" alt="Örnek uydu görüntüsü">
</p>

<p align="center">
  <em>Görüntü, bina alanlarının çevresindeki kentsel dokuyla birlikte incelenmiştir.</em>
</p>
README'yi hazırlarken önerim
GitHub'da depoyu aç.

README.md dosyasına girip kalem simgesine tıkla.

Düzenlenmiş Markdown metnini yapıştır.

Preview sekmesinden görünümü kontrol et.

Commit changes ile kaydet.

Böylece başlıklar, tablolar ve görseller GitHub'ın kendi stilinde görüntülenir.

Not: Gönderdiğin raporun tamamında aynı düzeni korumak için görselleri ortalamak, boyutlarını tutarlı hâle getirmek ve tüm bölümlerin Markdown biçimini standartlaştırmak gerekir.


Şunu dedin:
tamam o zaman az önce hazrıladığın metni tamamen read me için düzenleyip atar mıısn lütfen

ChatGPT:
Aşağıdaki metni GitHub'daki README.md dosyana doğrudan yapıştırabilirsin. Tüm raporu GitHub Markdown biçiminde düzenledim; başlıkları, tabloları, görselleri, görsel açıklamalarını ve bölüm geçişlerini standartlaştırdım. İçerikteki teknik bilgileri korudum.


markdown

# GEOAI-QUALITY-CONTROL
## Uydu Görüntülerinden Bina Alanlarının U-Net ile Tespiti

**Makine Öğrenmesi ile Probleme Uygun Çözüm Geliştirme — Proje Raporu**

---

## İçindekiler

- [1. Problem Tanımı ve Projenin Amacı](#1-problem-tanımı-ve-projenin-amacı)
- [2. Veri Seti ve Veri Keşfi](#2-veri-seti-ve-veri-keşfi)
- [3. Görsel İnceleme ve Ön İşleme](#3-görsel-inceleme-ve-ön-işleme)
- [4. Model Tasarımı ve Eğitim](#4-model-tasarımı-ve-eğitim)
- [5. AI Tahmini ve Eşik Değeri Analizi](#5-ai-tahmini-ve-eşik-değeri-analizi)
- [6. Hata Haritası](#6-hata-haritası)
- [7. Belirsizlik Haritası](#7-belirsizlik-haritası)
- [8. Sonuç, Sınırlılıklar ve Geliştirme Önerileri](#8-sonuç-sınırlılıklar-ve-geliştirme-önerileri)
- [Kullanılan Kaynaklar ve Teknolojiler](#kullanılan-kaynaklar-ve-teknolojiler)

---

## 1. Problem Tanımı ve Projenin Amacı

Bu çalışmanın amacı, yüksek çözünürlüklü RGB uydu görüntülerinden bina alanlarını otomatik olarak belirleyen bir derin öğrenme modeli geliştirmektir. Bina ayak izlerinin uydu görüntülerinden elle çıkarılması zaman alabilir ve geniş alanlarda tutarlı sonuç üretmeyi zorlaştırabilir.

Girdi olarak üç bantlı uydu görüntüleri, hedef çıktı olarak ise bina bulunan pikselleri gösteren ikili maskeler kullanılmıştır. Bu problem, görüntü sınıflandırmasından farklı olarak her pikselin **bina** veya **bina dışı** olarak etiketlendiği anlamsal bölütleme (*Semantic Segmentation*) problemidir.

Çalışma; haritalama, kentsel envanter çıkarımı ve uzaktan algılama iş akışlarında otomatik bina tespitine yönelik bir prototip sunmaktadır.

## 2. Veri Seti ve Veri Keşfi

Projede, SpaceNet 2 veri setinin Las Vegas (AOI 2 – Vegas) bölgesine ait RGB uydu görüntüleri ve bu görüntülere karşılık gelen GeoJSON formatındaki bina geometrileri kullanılmıştır.

Veri setinde yer alan 3.851 GeoJSON dosyası arasından sabit bir rastgelelik değeriyle 100 görüntü kimliği seçilmiştir. Seçilen görüntülere ait bina geometrileri işlenerek binaları ve bina dışındaki alanları ayırt eden ikili maskeler oluşturulmuştur.

### Veri Seti Özellikleri

| Özellik | Açıklama |
|---|---|
| Veri kaynağı | SpaceNet 2 |
| Bölge | Las Vegas (AOI 2 – Vegas) |
| Seçilen görüntü sayısı | 100 |
| Görüntü boyutu | 650 × 650 piksel |
| Renk kanalları | RGB (3 kanal) |
| Hedef maske | İkili bina maskesi |
| Bina pikselleri | 1 |
| Bina dışı pikseller | 0 |

Modelin eğitimi ve performansının değerlendirilmesi için 100 görüntü, `train_test_split()` fonksiyonu kullanılarak eğitim ve doğrulama kümelerine ayrılmıştır. `test_size=0.20` parametresiyle verilerin %80'i eğitim, %20'si doğrulama amacıyla kullanılmıştır.

| Veri kümesi | Görüntü sayısı | Oran |
|---|---:|---:|
| Eğitim | 80 | %80 |
| Doğrulama | 20 | %20 |
| **Toplam** | **100** | **%100** |

Veri bölme işleminin tekrarlanabilir olması için `random_state=SEED` parametresi kullanılmıştır.

**Veri kaynağı:** [SpaceNet](https://spacenet.ai/) — SpaceNet 2, AOI 2

---

## 3. Görsel İnceleme ve Ön İşleme

İlk görsel, seçilen bir uydu görüntüsünün RGB kanallarını göstermektedir. Görüntü; çatı şekilleri, yollar, gölgeler ve çevresel dokular gibi modelin ayırt etmesi gereken özellikleri içermektedir.

İkinci görselde gerçek bina maskesi incelenmektedir. Beyaz alanlar bina olarak etiketlenen pikselleri, siyah alanlar ise bina dışı pikselleri göstermektedir.

### Şekil 1. Örnek Uydu Görüntüsü

<p align="center">
  <img src="https://github.com/user-attachments/assets/a142685c-7b82-4a3c-bb24-f71fd5fe0ee6" width="450" alt="Örnek RGB uydu görüntüsü">
</p>

<p align="center">
  <em>Şekil 1. Bina alanlarının çevresindeki kentsel dokuyla birlikte incelendiği örnek uydu görüntüsü.</em>
</p>

### Şekil 2. Gerçek Bina Maskesi (Ground Truth)

<p align="center">
  <img src="https://github.com/user-attachments/assets/17cdde0e-7769-4913-b5b9-a394796c8b0d" width="750" alt="Gerçek bina maskesi ve maske kontrolü">
</p>

<p align="center">
  <em>Şekil 2. Gerçek bina maskesi. Beyaz alanlar bina piksellerini, siyah alanlar bina dışı pikselleri göstermektedir.</em>
</p>

Notebook çıktısındaki örnek maskede bina piksel oranı yaklaşık %26,0'dır. Bu oran yalnızca gösterilen örneğe aittir.

Ön işlemede RGB kanalları model girdisi için 0–1 aralığına ölçeklenmiş; görüntüler ve maskeler modelin giriş boyutu olan 256 × 256 piksele getirilmiştir. İkili maskeler hedef olarak kullanılmıştır.

Görüntülerin küçültülmesi bellek kullanımını ve işlem süresini azaltmaktadır. Ancak bu işlem küçük binaların ve ince sınırların ayrıntılarının kaybolmasına neden olabilir.

---

## 4. Model Tasarımı ve Eğitim

Model olarak **U-Net** mimarisi seçilmiştir. Encoder bölümü görüntüdeki bağlamsal özellikleri öğrenirken decoder bölümü bu bilgileri yeniden uzamsal çözünürlüğe taşıyarak piksel düzeyinde maske üretmektedir.

Atlamalı bağlantılar (*skip connections*), encoder katmanlarından gelen konum ve sınır bilgisinin decoder bölümüne aktarılmasına yardımcı olmaktadır.

Notebook final özetine göre model 1.928.417 parametreye sahiptir.

### Model Yapılandırması

| Parametre | Değer |
|---|---|
| Model mimarisi | U-Net |
| Girdi boyutu | 256 × 256 × 3 |
| Çıktı | Piksel bazlı bina olasılık haritası |
| Kayıp fonksiyonu | BCE + Dice Loss |
| Optimizasyon algoritması | Adam |
| Eğitim süresi | 20 epoch |
| Parametre sayısı | 1.928.417 |

Modelin çıktısında her piksel için bina olasılığı üretilmektedir. BCE piksel düzeyindeki ikili tahmin hatasını ölçerken Dice bileşeni, tahmin maskesi ile gerçek bina alanlarının örtüşmesini dikkate almaktadır.

Model, Adam optimizasyon algoritması kullanılarak 20 epoch boyunca eğitilmiştir. Eğitim ve doğrulama süreçleri boyunca kayıp ve IoU eğrileri oluşturulmuştur.

### Şekil 3. Eğitim ve Doğrulama Kaybı (Loss) Eğrileri

<p align="center">
  <img src="https://github.com/user-attachments/assets/28d4fe72-4877-4ebb-95ba-5ba4f8574b5a" width="750" alt="Eğitim ve doğrulama kaybı eğrileri">
</p>

<p align="center">
  <em>Şekil 3. Epoch boyunca eğitim ve doğrulama kayıplarının değişimi.</em>
</p>

### Şekil 4. Eğitim ve Doğrulama IoU Eğrileri

<p align="center">
  <img src="https://github.com/user-attachments/assets/baead84b-4be1-41c6-8682-477cf95d9158" width="750" alt="Eğitim ve doğrulama IoU eğrileri">
</p>

<p align="center">
  <img src="https://github.com/user-attachments/assets/88a0eb8a-fb13-4532-9819-44b95f3910e6" width="750" alt="IoU eğitim çıktısı">
</p>

<p align="center">
  <em>Şekil 4. Eğitim ve doğrulama IoU değerleri, modelin öğrenme ve genelleme davranışını incelemek için kullanılmıştır.</em>
</p>

Eğitim ve doğrulama eğrileri arasındaki fark, modelin daha önce görmediği örneklere genelleme davranışını değerlendirmeye yardımcı olmaktadır.

---

## 5. AI Tahmini ve Eşik Değeri Analizi

Modelin olasılık çıktısı, seçilen eşik değeri kullanılarak ikili bina maskesine dönüştürülmektedir. Notebook'ta 0,2–0,7 aralığındaki eşikler karşılaştırılmış ve en yüksek ortalama IoU değerini veren 0,5 eşiği seçilmiştir.

### Şekil 5. AI Tahmini Görseli

<p align="center">
  <img src="https://github.com/user-attachments/assets/03b9c17e-d326-4b2c-a742-1d9f58d7b4e2" width="750" alt="Uydu görüntüsü, gerçek bina maskesi, olasılık haritası ve AI tahmini">
</p>

<p align="center">
  <em>Şekil 5. Uydu görüntüsü, gerçek maske, modelin olasılık haritası ve 0,5 eşik değeriyle elde edilen tahmin maskesi.</em>
</p>

Olasılık haritasındaki parlak bölgeler, modelin bina olma olasılığını yüksek gördüğü alanları göstermektedir. Gerçek maske ile tahmin maskesi karşılaştırılarak modelin bina alanlarını ne ölçüde yakaladığı görsel olarak incelenebilmektedir.

Segmentasyon performansını değerlendirmek için **IoU (Intersection over Union / Kesişim-Birleşim Oranı)** metriği kullanılmıştır. IoU, tahmin edilen bina pikselleri ile gerçek bina maskesinin kesişiminin birleşimine oranıdır. Değer 1'e yaklaştıkça örtüşme artmaktadır.

Eğitim sırasında 0,5 eşik değeriyle hesaplanan doğrulama IoU değeri, 20. epoch'ta 0,5386 olarak kaydedilmiştir. Ayrı değerlendirme adımında doğrulama görüntülerinin her biri için 0,2–0,7 arasında farklı olasılık eşikleri denenmiş ve ortalama IoU hesaplanmıştır.

### Şekil 6. Eşik Değerine Göre Ortalama IoU

<p align="center">
  <img src="https://github.com/user-attachments/assets/42070ee2-d166-4881-98a3-7fea2b092f9d" width="750" alt="Eşik değerine göre ortalama IoU grafiği">
</p>

<p align="center">
  <em>Şekil 6. Farklı eşik değerlerinde elde edilen ortalama IoU sonuçları.</em>
</p>

### Eşik Analizi Sonuçları

| Ölçüt | Sonuç |
|---|---:|
| En iyi eşik değeri | 0,5 |
| En yüksek ortalama IoU | 0,5287 |
| Eğitim sırasında 20. epoch doğrulama IoU | 0,5386 |

Bu değerlendirmede en yüksek ortalama IoU, 0,5 eşik değerinde 0,5287 olarak elde edilmiştir.

Eşik 0,4 ve 0,6 olduğunda sonuçlar birbirine yakındır. 0,2 ve 0,7 eşiklerinde ise ortalama IoU daha düşüktür. Bu durum, bina piksellerini pozitif olarak sınıflandırmak için kullanılan eşik değerinin sonuçları etkilediğini göstermektedir.

Eşik seçimi yalnızca tek bir görüntüye bakılarak değil, doğrulama kümesindeki ortalama performansa göre yapılmalıdır.

---

## 6. Hata Haritası: Model Nerede Yanılıyor?

Hata haritası, modelin doğru ve yanlış tahminlerini uzamsal olarak görünür kılmaktadır. Böylece hataların bina sınırlarında, küçük nesnelerde veya bina dışı alanlarda yoğunlaşıp yoğunlaşmadığı incelenebilmektedir.

### Hata Haritasındaki Renklerin Anlamları

| Renk | Anlamı | Açıklama |
|---|---|---|
| Yeşil | Doğru pozitif (TP) | Gerçek bina piksellerinin doğru tespit edilmesi |
| Kırmızı | Yanlış pozitif (FP) | Bina olmadığı hâlde bina tahmini yapılması |
| Mavi | Yanlış negatif (FN) | Gerçek bina piksellerinin kaçırılması |

### Şekil 7. Hata Haritası

<p align="center">
  <img src="https://github.com/user-attachments/assets/77c42676-134f-4b0d-8072-3c13e2a81cd1" width="750" alt="Bina segmentasyonu hata haritası">
</p>

<p align="center">
  <em>Şekil 7. Modelin doğru pozitif, yanlış pozitif ve yanlış negatif tahminlerinin uzamsal dağılımı.</em>
</p>

Notebook çıktısında gösterilen örnek görüntü için IoU değeri, 0,5 eşik değerinde 0,7709 olarak hesaplanmıştır.

### Örnek Görüntü İçin Değerlendirme Sonuçları

| Değerlendirme Değişkeni | Değer |
|---|---:|
| Gerçek bina pikseli | 16.772 |
| Tahmin edilen bina pikseli | 16.638 |
| Doğru pozitif (TP) | 14.544 |
| Yanlış pozitif (FP) | 2.094 |
| Yanlış negatif (FN) | 2.228 |
| IoU | 0,7709 |

Bu örnek görüntü, 0,5287 olan genel ortalama IoU değerinden daha başarılı bir sonuç göstermektedir. Ancak tek bir görüntüdeki performans, modelin tüm veri setindeki başarısını tek başına temsil etmemektedir.

---

## 7. Belirsizlik Haritası (Uncertainty Map)

Notebook'ta Monte Carlo Dropout yöntemiyle aynı görüntü için 20 tahmin üretilmiş ve tahminlerin değişkenliği piksel bazında belirsizlik haritasına dönüştürülmüştür.

Şekil 8'de sol tarafta çoklu tahminlerin ortalaması, sağ tarafta ise belirsizlik haritası gösterilmektedir. Belirsizliğin özellikle bina sınırlarında ve bazı çatı bölgelerinde yükselmesi, modelin bu piksellerde daha kararsız tahminler yaptığını düşündürmektedir.

### Şekil 8. Monte Carlo Dropout Belirsizlik Haritası

<p align="center">
  <img src="https://github.com/user-attachments/assets/1dc5bef9-72b2-4663-a218-f9d6ceb17e3a" width="750" alt="Monte Carlo Dropout belirsizlik haritası">
</p>

<p align="center">
  <em>Şekil 8. Tahmin ortalaması ve piksel bazında belirsizlik dağılımı. Daha sıcak veya parlak renkler daha yüksek tahmin belirsizliğini temsil etmektedir.</em>
</p>

### Belirsizlik Analizi Sonuçları

| Belirsizlik Ölçütü | Değer |
|---|---:|
| Tahmin ortalaması | 0,2577 |
| Belirsizlik ortalaması | 0,0090 |
| Belirsizlik maksimumu | 0,1401 |

Belirsizlik haritası doğrudan bir doğruluk metriği değildir. Modelin hangi bölgelerde daha az kararlı olduğuna dair tamamlayıcı bilgi sağlamaktadır. Bu alanlar, hata incelemesi ve sonraki iyileştirmeler için önceliklendirilebilir.

---

## 8. Sonuç, Sınırlılıklar ve Geliştirme Önerileri

Bu projede, SpaceNet 2 uydu görüntüleri ve bunlara ait bina geometrileri kullanılarak U-Net mimarisi tabanlı bir bina segmentasyonu modeli geliştirilmiştir. Model, 256 × 256 piksel boyutundaki görüntülerle, BCE ve Dice kayıplarının birleştirildiği kayıp fonksiyonu kullanılarak 20 epoch boyunca eğitilmiştir.

Elde edilen sonuçlarda **ortalama IoU değeri 0,5287**, en uygun eşik değeri ise **0,5** olarak belirlenmiştir. Ayrıca modelin tahminleri; gerçek bina maskeleri, hata haritası ve belirsizlik haritası ile karşılaştırılarak görsel açıdan da değerlendirilmiştir.

Bu analizler, modelin doğru tahminlerini, hatalı sınıflandırmalarını ve tahminlerindeki belirsizliğin dağılımını incelemeye olanak sağlamıştır.

### Sınırlılıklar

Çalışmanın temel sınırlılıkları şunlardır:

- Veri setinin 100 görüntüyle sınırlı olması.
- Görüntülerin 256 × 256 piksel boyutuna küçültülmesi nedeniyle bazı küçük binaların ve sınır ayrıntılarının kaybolabilmesi.
- Bağımsız bir test kümesine ait performans sonuçlarının sunulmaması.

### Geliştirme Önerileri

Modelin performansını geliştirmek için daha fazla görüntü içeren ve farklı coğrafi bölgeleri kapsayan veri setleri kullanılabilir. Modelin daha önce karşılaşmadığı görüntüler üzerindeki başarısı ayrıca değerlendirilebilir.

Bunun yanında:

- Kontrollü veri artırma yöntemleri uygulanabilir.
- Model performansı IoU metriğinin yanı sıra Dice/F1 skoru, precision ve recall değerleriyle de incelenebilir.
- Küçük binaların ve bina sınırlarının daha doğru belirlenmesi amacıyla yama temelli eğitim yöntemleri değerlendirilebilir.
- Daha yüksek çözünürlüklü görüntüler kullanılarak ayrıntı kaybı azaltılabilir.

### Genel Sonuç

Bu proje, uydu görüntülerinden bina alanlarının otomatik olarak belirlenmesi için derin öğrenme tabanlı bir yaklaşımın uygulanmasını ve model sonuçlarının tahmin, hata ve belirsizlik haritaları aracılığıyla analiz edilmesini sağlamıştır.

Elde edilen bulgular, bina segmentasyonu çalışmalarında model performansının hem sayısal metrikler hem de görsel analizlerle birlikte değerlendirilmesinin önemini ortaya koymaktadır.

---

## Kullanılan Kaynaklar ve Teknolojiler

- **Veri kaynağı:** [SpaceNet](https://spacenet.ai/) — SpaceNet 2, AOI 2
- **Uygulama ortamı:** Google Colab
- **Programlama dili:** Python
- **Derin öğrenme kütüphanesi:** PyTorch
- **Diğer araçlar:** Rasterio, NumPy, Matplotlib ve geometri işleme kütüphaneleri
