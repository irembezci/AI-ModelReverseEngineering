# AI Reverse Engineering (Tersine Mühendislik) Sürecinde Sorduğumuz Temel Sorular

Bir yapay zeka sistemini reverse engineering yaklaşımıyla incelerken amacımız yalnızca modelin ne olduğunu belirlemek değildir. Aynı zamanda modelin nasıl çalıştığını, hangi verilerle eğitildiğini, hangi karar mekanizmalarını kullandığını ve hangi sınırlamalara sahip olduğunu da anlamaya çalışırız. Bu amaç doğrultusunda sistematik olarak belirli sorular sorarız. Bu sorular, gerçekleştireceğimiz analizlerin temelini oluşturur.

## Mimariye Yönelik Sorular (Architecture Questions)

Bilinmeyen bir yapay zeka modeliyle karşılaştığımızda ilk olarak modelin mimarisini anlamaya çalışırız.

### Soru 1: Bu model hangi model ailesine ait?

Transformer, RNN, CNN, karar ağacı (Decision Tree) veya doğrusal (Linear) bir model mi?

Reverse engineering sürecindeki ilk hedeflerden biri, incelenen sistemin hangi model ailesine ait olduğunu belirlemektir. Her model ailesi kendine özgü davranışsal özelliklere sahiptir. Modelin yanıt sürelerini, ürettiği çıktıları, tokenizasyon davranışını ve diğer davranışsal imzaları (Behavioral Signatures) analiz ederek modelin hangi mimariyi kullandığına dair yüksek doğrulukta çıkarımlar yapmaya çalışırız.

### Soru 2: Modelin ölçeği nedir?

Büyük transformer modelleri ile küçük modeller aynı mimariye sahip olsalar bile farklı davranışlar sergileyebilir. Bu nedenle yalnızca model ailesini belirlemek yeterli değildir; modelin yaklaşık ölçeğini de anlamaya çalışırız. Model büyüdükçe yanıt kalitesi, gecikme süresi ve kaynak kullanımı gibi özellikler de değişebilir. Bu bilgiler olası aday modelleri daraltmamıza yardımcı olur.

Bunun için aşağıdaki dolaylı göstergelerden (Proxy Measures) yararlanabiliriz:

- Yanıt gecikmesi (Response Latency)
- Bellek kullanımı (Memory Usage)
- Üretilen cevapların kalitesi

### Soru 3: Modelin giriş ve çıkış özellikleri (Input/Output Specification) nelerdir?

Model hangi tür girdileri kabul ediyor?

Çıktının biçimi ve boyutu nedir?

Modelin giriş ve çıkış yapısı, kullanılan mimari hakkında önemli ipuçları verir. Özellikle çıktı boyutu, modelin hangi problem için geliştirildiğini veya hangi kelime dağarcığını kullandığını anlamamıza yardımcı olabilir.

Örneğin;

- 1000 sınıflı bir çıktı, büyük olasılıkla ImageNet benzeri bir görüntü sınıflandırma modeline işaret eder.
- 50.257 boyutlu bir çıktı ise GPT-2'nin kullandığı kelime dağarcığına (Vocabulary) işaret edebilir.

### Soru 4: Model hangi tokenizasyon yöntemini kullanıyor?

Dil modellerinde kullanılan tokenizasyon yöntemi model hakkında önemli ipuçları verir. Farklı model aileleri farklı tokenizasyon stratejileri kullanabilir. Bu nedenle üretilen çıktıları dikkatle inceleyerek modelin hangi tokenizasyon yöntemini kullandığını anlamaya çalışırız.

Örneğin model;

- BPE (GPT ailesi)
- WordPiece (BERT ailesi)
- SentencePiece
- Character-level (Karakter tabanlı)

tokenizasyon yöntemlerinden birini kullanıyor olabilir.

Üretilen çıktılarda görülen tokenizasyon izleri (Tokenization Artifacts), modelin kullandığı yöntemin belirlenmesine yardımcı olabilir.


## Eğitim Verisine Yönelik Sorular (Training Data Questions)

İkinci grup sorular modelin hangi verilerden öğrendiğini anlamaya yöneliktir.

### Soru 5: Model hangi alanlarda eğitildi?

Model hangi alanlarda güçlü, hangilerinde zayıf?

Modelin hangi veri kümeleri üzerinde eğitildiğini anlamaya çalışırız. Çünkü eğitim verisi, modelin güçlü ve zayıf olduğu alanları doğrudan etkiler. Örneğin ağırlıklı olarak web verileriyle eğitilmiş bir model ile kaynak kodu veya biyomedikal veriler üzerinde eğitilmiş bir model farklı davranışlar sergileyecektir.

### Soru 6: Modelin bilgi kesim tarihi (Knowledge Cutoff) nedir?

Büyük dil modelleri belirli bir tarihe kadar olan bilgilerle eğitilir.

Modelin hangi tarihe kadar olan bilgilere sahip olduğunu belirlemek, eğitim verisinin zaman aralığını anlamamıza yardımcı olur. Bunun için güncel olaylar, yeni teknolojiler veya yakın tarihte gerçekleşmiş gelişmeler hakkında sorular yönelterek modelin bilgi sınırını tahmin etmeye çalışırız.

### Soru 7: Model hangi metinleri veya veri örneklerini ezberledi?

Membership Inference ve Data Extraction teknikleri kullanılarak modelin eğitim sırasında ezberlediği belirli örnekler ortaya çıkarılabilir.

Bu analizler sayesinde modelin;

- Özel (Private) verileri,
- Telif hakkıyla korunan içerikleri,
- Eğitim verisindeki belirli kayıtları

ezberleyip ezberlemediğini anlamaya çalışırız. Bu aynı zamanda gizlilik ve veri güvenliği açısından önemli bir değerlendirme alanıdır.

### Soru 8: Modelin eğitim dağılımı (Training Distribution) nasıldır?

Eğitim verisinde;

- Hangi konular,
- Hangi diller,
- Hangi yazım stilleri,
- Hangi alanlar

daha fazla veya daha az temsil edilmektedir?

Modelin farklı konulardaki performansını karşılaştırarak eğitim verisinin nasıl bir dağılıma sahip olduğu hakkında çıkarımlar yapmaya çalışırız. Bu analiz aynı zamanda modelde bulunan önyargıları (Bias) anlamamıza da yardımcı olur.


## Karar Mekanizmasına Yönelik Sorular (Decision Logic Questions)

Bu bölümde modelin kararlarını nasıl verdiğini anlamaya çalışırız.

### Soru 9: Model tahmin yaparken hangi özellikleri kullanıyor?

Bir model her kararını girişte bulunan tüm bilgilerden eşit şekilde etkilenerek vermez. Bazı özellikler diğerlerinden daha fazla önem taşır. SHAP, Integrated Gradients ve Attention analizleri gibi attribution yöntemleri sayesinde hangi giriş özelliklerinin model kararını en fazla etkilediğini inceleyebiliriz.

### Soru 10: Model hangi iç temsilleri (Internal Representations) öğrendi?

Derin öğrenme modelleri bilgiyi ara katmanlarda farklı biçimlerde temsil eder. Probing Classifier yöntemleri kullanarak bu katmanlarda hangi bilgilerin öğrenildiğini ve nasıl temsil edildiğini anlamaya çalışırız.

### Soru 11: Karar sınırları (Decision Boundaries) nerede bulunuyor?

Özellikle sınıflandırma modellerinde;

- Sınıflar arasındaki karar sınırları nerede?
- Belirli bir tahmin karar sınırına ne kadar yakın?

gibi soruların cevaplarını ararız. Karar sınırlarının belirlenmesi, modelin hangi durumlarda hata yapabileceğini anlamamıza yardımcı olur.

### Soru 12: Modelin sistematik önyargıları (Systematic Biases) nelerdir?

Model;

- Farklı demografik gruplara farklı davranıyor mu?
- Kültürel önyargılar içeriyor mu?
- Belirli anahtar kelimeler belirli davranışları tetikliyor mu?

Bu sorular sayesinde modelde bulunan sistematik önyargıları ortaya çıkarmaya ve hangi durumlarda beklenmeyen davranışlar sergileyebileceğini anlamaya çalışırız.


## Yetenek ve Sınırlamalara Yönelik Sorular (Capability and Limitation Questions)

Son grup sorular modelin neleri yapabildiğini ve hangi durumlarda başarısız olduğunu anlamaya yöneliktir.

### Soru 13: Modelin belgelenmemiş hangi yetenekleri bulunuyor?

Birçok model, açıkça eğitilmediği halde zaman içerisinde beklenmeyen yeni yetenekler (Emergent Capabilities) geliştirebilir. Sistematik testler uygulayarak modelin belgelenmemiş yeteneklerini ve gerçek kapasitesini ortaya çıkarmaya çalışırız.

### Soru 14: Model hangi durumlarda başarısız oluyor ve neden?

Hiçbir model her durumda doğru sonuç üretmez. Adversarial örnekler, dağılım dışı girdiler (Out-of-Distribution Inputs) ve uç durumlar (Edge Cases) kullanarak modelin başarısız olduğu senaryoları belirlemeye çalışırız. Bu analizler güvenlik değerlendirmelerinin önemli bir parçasını oluşturur.

### Soru 15: Modelin kalibrasyonu (Calibration) nasıldır?

Modelin ifade ettiği güven seviyesi gerçekten doğruluk oranıyla uyumlu mudur?

Bazı modeller yanlış cevaplar verirken bile oldukça yüksek güven ifade edebilir. Bu nedenle modelin güven düzeyi ile gerçek doğruluk oranının ne kadar uyumlu olduğunu analiz ederiz. Kalibrasyonun zayıf olması, eğitim veya eğitim sonrası (Post-Training) süreçlerde çeşitli problemlere işaret edebilir.
