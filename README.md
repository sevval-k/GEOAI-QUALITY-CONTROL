GEOAI-QUALITY-CONTROL
Uydu Görüntülerinden Bina Alanlarının U-Net ile Tespiti

Makine Öğrenmesi ile Probleme Uygun Çözüm Geliştirme — Proje Raporu

1. Problem Tanımı ve Projenin Amacı

Bu çalışmanın amacı, yüksek çözünürlüklü RGB uydu görüntülerinden bina alanlarını otomatik olarak belirleyen bir derin öğrenme modeli geliştirmektir. Bina ayak izlerinin uydu görüntülerinden elle çıkarılması zaman alabilir ve geniş alanlarda tutarlı sonuç üretmeyi zorlaştırabilir.

Girdi olarak üç bantlı uydu görüntüleri, hedef çıktı olarak ise bina bulunan pikselleri gösteren ikili maskeler kullanılmıştır. Bu problem, görüntü sınıflandırmasından farklı olarak her pikselin “bina” veya “bina dışı” olarak etiketlendiği anlamsal bölütleme (Semantic Segmentation) problemidir.

Çalışma; haritalama, kentsel envanter çıkarımı ve uzaktan algılama iş akışlarında otomatik bina tespitine yönelik bir prototip sunar.

2. Veri Seti ve Veri Keşfi

Projede, SpaceNet 2 veri setinin Las Vegas (AOI 2 – Vegas) bölgesine ait RGB uydu görüntüleri ve bu görüntülere karşılık gelen GeoJSON formatındaki bina geometrileri kullanılmıştır.

Veri setinde yer alan 3.851 GeoJSON dosyası arasından sabit bir rastgelelik değeriyle 100 görüntü kimliği seçilmiştir. Seçilen görüntülere ait bina geometrileri işlenerek, binaları ve bina dışındaki alanları ayırt eden ikili maskeler oluşturulmuştur.

Veri Seti Özellikleri
Özellik	Açıklama
Veri kaynağı	SpaceNet 2
Bölge	Las Vegas (AOI 2 – Vegas)
Seçilen görüntü sayısı	100
Görüntü boyutu	650 × 650 piksel
Renk kanalları	RGB (3 kanal)
Hedef maske	İkili bina maskesi
Bina pikselleri	1
Bina dışı pikseller	0

Modelin eğitimi ve performansının değerlendirilmesi için 100 görüntü, train_test_split() fonksiyonu kullanılarak eğitim ve doğrulama kümelerine ayrılmıştır. test_size=0.20 parametresiyle verilerin %80’i eğitim, %20’si doğrulama amacıyla kullanılmıştır.

Eğitim kümesi: 80 görüntü
Doğrulama kümesi: 20 görüntü
Rastgelelik parametresi: random_state=SEED

Veri bölme işleminin tekrarlanabilir olması için random_state=SEED parametresi kullanılmıştır.

Veri kaynağı: SpaceNet — SpaceNet 2, AOI 2

3. Görsel İnceleme ve Ön İşleme

İlk görsel, seçilen bir uydu görüntüsünün RGB kanallarını gösterir. Görüntü, modelin ayırt etmesi gereken çatı şekilleri, yollar, gölgeler ve çevresel dokular gibi özellikleri içerir.

İkinci görselde gerçek bina maskesi incelenir. Beyaz alanlar bina olarak etiketlenen pikselleri, siyah alanlar ise bina dışı pikselleri gösterir.

Şekil 1. Örnek Uydu Görüntüsü

Görüntü, bina alanlarının çevresindeki kentsel dokuyla birlikte incelenmiştir.

<img width="492" height="509" alt="Örnek uydu görüntüsü" src="https://github.com/user-attachments/assets/a142685c-7b82-4a3c-bb24-f71fd5fe0ee6" />

Şekil 2. Gerçek Bina Maskesi (Ground Truth)

Notebook çıktısındaki örnek maskede bina piksel oranı yaklaşık %26,0’dır. Bu oran yalnızca gösterilen örneğe aittir.

<img width="930" height="320" alt="Gerçek bina maskesi ve maske kontrolü" src="https://github.com/user-attachments/assets/17cdde0e-7769-4913-b5b9-a394796c8b0d" />

Ön işlemede RGB kanalları görselleştirme ve model girdisi için 0–1 aralığına ölçeklenmiş; görüntüler ve maskeler modelin giriş boyutu olan 256 × 256 piksele getirilmiştir. İkili maskeler hedef olarak kullanılmıştır.

Görüntülerin küçültülmesi bellek ve işlem süresini azaltır; ancak küçük binaların ve ince sınırların ayrıntısını kaybettirebilir.

4. Model Tasarımı ve Eğitim

Model olarak U-Net mimarisi seçilmiştir. Encoder bölümü görüntüdeki bağlamsal özellikleri öğrenirken decoder bölümü bu bilgileri yeniden uzamsal çözünürlüğe taşıyarak piksel düzeyinde maske üretir. Atlamalı bağlantılar, encoder katmanlarından gelen konum ve sınır bilgisinin decoder’a aktarılmasına yardımcı olur.

Notebook final özetine göre model 1.928.417 parametreye sahiptir.

Model Yapılandırması
Parametre	Değer
Model mimarisi	U-Net
Girdi boyutu	256 × 256 × 3
Çıktı	Piksel bazlı bina olasılık haritası
Kayıp fonksiyonu	BCE + Dice Loss
Optimizasyon algoritması	Adam
Eğitim süresi	20 epoch
Parametre sayısı	1.928.417

Modelin çıkışında her piksel için bina olasılığı üretilir. BCE piksel düzeyindeki ikili tahmin hatasını ölçerken Dice bileşeni tahmin maskesi ile gerçek bina alanlarının örtüşmesini dikkate alır. Notebook’ta eğitim ve doğrulama kaybı ile IoU eğrileri oluşturulmuştur.

Şekil 3. Eğitim ve Doğrulama Kaybı (Loss) Eğrileri

Eğriler, epoch boyunca optimizasyonun ve doğrulama performansının nasıl değiştiğini izlemek için kullanılır.

<img width="810" height="544" alt="Eğitim ve doğrulama kaybı eğrileri" src="https://github.com/user-attachments/assets/28d4fe72-4877-4ebb-95ba-5ba4f8574b5a" />

Şekil 4. Eğitim ve Doğrulama IoU Eğrileri

Eğitim ve doğrulama eğrileri arasındaki fark, modelin görülmemiş örneklere genelleme davranışını değerlendirmeye yardımcı olur.

<img width="810" height="551" alt="Eğitim ve doğrulama IoU eğrileri" src="https://github.com/user-attachments/assets/baead84b-4be1-41c6-8682-477cf95d9158" />

<img width="975" height="252" alt="IoU eğitim çıktısı" src="https://github.com/user-attachments/assets/88a0eb8a-fb13-4532-9819-44b95f3910e6" />

5. AI Tahmini ve Eşik Değeri Analizi

Modelin olasılık çıktısı, seçilen eşik değeriyle ikili bina maskesine dönüştürülür. Notebook’ta 0,2–0,7 aralığındaki eşikler karşılaştırılmış ve en yüksek ortalama IoU için 0,5 eşiği seçilmiştir.

Şekil 5’te soldan sağa uydu görüntüsü, gerçek bina maskesi, modelin olasılık haritası ve 0,5 eşiğiyle elde edilen ikili tahmin gösterilir. Olasılık haritasındaki parlak bölgeler, modelin bina olma olasılığını yüksek gördüğü alanlardır.

Şekil 5. AI Tahmini Görseli

Gerçek maske ile tahmin maskesi karşılaştırılarak modelin bina alanlarını ne ölçüde yakaladığı görsel olarak incelenebilir.

<img width="780" height="511" alt="AI bina tahmini" src="https://github.com/user-attachments/assets/03b9c17e-d326-4b2c-a742-1d9f58d7b4e2" />

Segmentasyon performansı için IoU (Intersection over Union / Kesişim-Birleşim Oranı) kullanılmıştır. IoU, tahmin edilen bina pikselleri ile gerçek bina maskesinin kesişiminin birleşimine oranıdır; 1’e yaklaştıkça örtüşme artar.

Eğitim sırasında 0,5 eşik değeriyle hesaplanan validation IoU, 20. epoch’ta 0,5386 olarak kaydedilmiştir. Ayrı değerlendirme adımında validation görüntülerinin her biri için 0,2–0,7 arasında farklı olasılık eşikleri denenmiş ve ortalama IoU hesaplanmıştır.

Şekil 6. Eşik Değerine Göre Ortalama IoU

<img width="780" height="511" alt="Eşik değerine göre ortalama IoU" src="https://github.com/user-attachments/assets/42070ee2-d166-4881-98a3-7fea2b092f9d" />

Bu değerlendirmede en yüksek ortalama IoU 0,5 eşik değerinde 0,5287 olarak elde edilmiştir.

Eşik 0,4 ve 0,6 olduğunda sonuçlar birbirine yakındır; 0,2 ve 0,7 eşiklerinde ortalama IoU daha düşüktür. Bu durum, bina piksellerini pozitif olarak sınıflandırmak için kullanılan eşik değerinin sonuçları etkilediğini göstermektedir.

Eşik seçimi yalnızca tek bir görüntüye bakılarak değil, doğrulama kümesindeki ortalama performansa göre yapılmalıdır.

6. Hata Haritası: Model Nerede Yanılıyor?

Hata haritası, modelin doğru ve yanlış tahminlerini uzamsal olarak görünür kılar.

Kodun renk gösteriminde:

Yeşil: Doğru pozitif bina pikselleri (TP)
Kırmızı: Yanlış pozitifler (FP — bina olmadığı hâlde bina tahmini)
Mavi: Yanlış negatifler (FN — gerçek binanın kaçırılan bölümü)

Bu harita, hatanın bina sınırlarında mı, küçük nesnelerde mi yoksa bina dışı alanlarda mı yoğunlaştığını incelemeye yardımcı olur.

Şekil 7. Hata Haritası

<img width="975" height="252" alt="Bina segmentasyonu hata haritası" src="https://github.com/user-attachments/assets/77c42676-134f-4b0d-8072-3c13e2a81cd1" />

Notebook çıktısında gösterilen görüntü için IoU (eşik = 0,5) 0,7709 olarak hesaplanmıştır.

Değerlendirme Değişkeni	Değer
Gerçek bina pikseli	16.772
Tahmin edilen bina pikseli	16.638
Doğru pozitif (TP)	14.544
Yanlış pozitif (FP)	2.094
Yanlış negatif (FN)	2.228
IoU	0,7709

Örnek görüntü, ortalama sonuçtan daha başarılı bir performans göstermektedir.

7. Belirsizlik Haritası (Uncertainty Map)

Notebook’ta Monte Carlo Dropout ile aynı görüntü için 20 tahmin üretilmiş; tahminlerin değişkenliği piksel bazında belirsizlik haritasına dönüştürülmüştür.

Şekil 8’de sol tarafta çoklu tahminlerin ortalaması, sağ tarafta belirsizlik haritası görülür. Belirsizliğin özellikle bina sınırlarında ve bazı çatı bölgelerinde yükselmesi, modelin bu piksellerde daha kararsız tahminler yaptığını düşündürür.

Şekil 8. Monte Carlo Dropout Belirsizlik Haritası

Daha sıcak veya parlak renkler daha yüksek tahmin belirsizliğini temsil eder.

<img width="945" height="434" alt="Monte Carlo Dropout belirsizlik haritası" src="https://github.com/user-attachments/assets/1dc5bef9-72b2-4663-a218-f9d6ceb17e3a" />

Bu örnekte elde edilen değerler:

Belirsizlik Ölçütü	Değer
Tahmin ortalaması	0,2577
Belirsizlik ortalaması	0,0090
Belirsizlik maksimumu	0,1401

Belirsizlik haritası doğrudan doğruluk metriği değildir; modelin hangi bölgelerde daha az kararlı olduğuna dair tamamlayıcı bilgi sağlar. Bu alanlar hata incelemesi ve sonraki iyileştirmeler için önceliklendirilebilir.

8. Sonuç, Sınırlılıklar ve Geliştirme Önerileri

Bu projede, SpaceNet 2 uydu görüntüleri ve bunlara ait bina geometrileri kullanılarak U-Net mimarisi tabanlı bir bina segmentasyonu modeli geliştirilmiştir. Model, 256 × 256 piksel boyutundaki görüntülerle, BCE ve Dice kayıplarının birleştirildiği kayıp fonksiyonu kullanılarak 20 epoch boyunca eğitilmiştir.

Elde edilen sonuçlarda ortalama IoU değeri 0,5287, en uygun eşik değeri ise 0,5 olarak belirlenmiştir. Ayrıca modelin tahminleri; gerçek bina maskeleri, hata haritası ve belirsizlik haritası ile karşılaştırılarak görsel açıdan da değerlendirilmiştir. Bu analizler, modelin doğru tahminlerini, hatalı sınıflandırmalarını ve tahminlerindeki belirsizliğin dağılımını incelemeye olanak sağlamıştır.

Sınırlılıklar

Çalışmanın temel sınırlılıkları şunlardır:

Veri setinin 100 görüntüyle sınırlı olması.
Görüntülerin 256 × 256 piksel boyutuna küçültülmesi nedeniyle bazı küçük binaların ve sınır ayrıntılarının kaybolabilmesi.
Bağımsız bir test kümesine ait performans sonuçlarının sunulmaması.
Geliştirme Önerileri

Modelin performansını geliştirmek için daha fazla görüntü içeren ve farklı coğrafi bölgeleri kapsayan veri setleri kullanılabilir. Modelin daha önce karşılaşmadığı görüntüler üzerindeki başarısı değerlendirilebilir.

Bunun yanında kontrollü veri artırma yöntemleri uygulanabilir ve model performansı IoU metriğinin yanı sıra Dice/F1 skoru, precision ve recall değerleriyle de incelenebilir. Küçük binaların ve bina sınırlarının daha doğru belirlenmesi amacıyla yama temelli eğitim yöntemleri veya daha yüksek çözünürlüklü görüntüler de değerlendirilebilir.

Genel Sonuç

Bu proje, uydu görüntülerinden bina alanlarının otomatik olarak belirlenmesi için derin öğrenme tabanlı bir yaklaşımın uygulanmasını ve model sonuçlarının tahmin, hata ve belirsizlik haritaları aracılığıyla analiz edilmesini sağlamıştır.

Elde edilen bulgular, bina segmentasyonu çalışmalarında model performansının hem sayısal metrikler hem de görsel analizlerle birlikte değerlendirilmesinin önemini ortaya koymaktadır.

Kullanılan Kaynaklar ve Teknolojiler
Veri kaynağı: SpaceNet — SpaceNet 2, AOI 2
Uygulama ortamı: Google Colab
Programlama dili: Python
Temel araçlar: PyTorch, Rasterio, NumPy, Matplotlib ve geometri işleme kütüphaneleri
