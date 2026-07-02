# AI Reverse Engineering (Tersine Mühendislik) Sürecinde Sorduğumuz Temel Sorular

Bir yapay zeka sistemini reverse engineering yaklaşımıyla incelerken amacımız yalnızca modelin ne olduğunu belirlemek değildir. Aynı zamanda modelin nasıl çalıştığını, hangi verilerle eğitildiğini, hangi karar mekanizmalarını kullandığını ve hangi sınırlamalara sahip olduğunu da anlamaya çalışırız. Bu amaç doğrultusunda araştırmacılar sistematik olarak belirli sorular sorar. Bu sorular, gerçekleştirilecek analizlerin temelini oluşturur.

## Mimariye Yönelik Sorular (Architecture Questions)

Bilinmeyen bir yapay zeka modeliyle karşılaştığımızda ilk olarak modelin mimarisini anlamaya çalışırız.

### Soru 1: Bu model hangi model ailesine ait?

Transformer, RNN, CNN, karar ağacı (Decision Tree) veya doğrusal (Linear) bir model mi?

Her model ailesi kendine özgü davranışsal özelliklere sahiptir. Modelin verdiği tepkiler analiz edilerek hangi mimariye ait olduğu hakkında çıkarımlarda bulunulabilir.

### Soru 2: Modelin ölçeği nedir?

Büyük transformer modelleri ile küçük modeller farklı davranışlar sergiler. Modelin ölçeğini anlamak, olası mimarileri daraltmaya yardımcı olur.

Araştırmacılar bunun için aşağıdaki dolaylı göstergelerden (proxy measures) yararlanabilir:

- Yanıt gecikmesi (Response Latency)
- Bellek kullanımı (Memory Usage)
- Üretilen cevapların kalitesi

### Soru 3: Modelin giriş ve çıkış özellikleri (Input/Output Specification) nelerdir?

Model hangi tür girdileri kabul ediyor?

Çıktının biçimi ve boyutu nedir?

Örneğin;

- 1000 sınıflı bir çıktı, büyük olasılıkla ImageNet benzeri bir görüntü sınıflandırma modeline işaret eder.
- 50.257 boyutlu bir çıktı ise GPT-2'nin kullandığı kelime dağarcığına (Vocabulary) işaret edebilir.

### Soru 4: Model hangi tokenizasyon yöntemini kullanıyor?

Dil modellerinde kullanılan tokenizasyon yöntemi model hakkında önemli ipuçları verir.

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

Örneğin ağırlıklı olarak web verileriyle eğitilmiş bir model, kaynak kodu veya biyomedikal veriler üzerinde eğitilmiş bir modele göre farklı güçlü ve zayıf yönlere sahip olacaktır.

### Soru 6: Modelin bilgi kesim tarihi (Knowledge Cutoff) nedir?

Büyük dil modelleri belirli bir tarihe kadar olan bilgilerle eğitilir.

Zamana duyarlı bilgiler kullanılarak yapılan testler sayesinde modelin eğitim verisinin yaklaşık hangi tarihte sona erdiği tahmin edilebilir.

### Soru 7: Model hangi metinleri veya veri örneklerini ezberledi?

Membership Inference ve Data Extraction teknikleri kullanılarak modelin eğitim sırasında ezberlediği belirli örnekler ortaya çıkarılabilir.

Bu örnekler;

- Özel (Private) verileri,
- Telif hakkıyla korunan içerikleri,
- Eğitim verisindeki belirli kayıtları

içerebilir.

### Soru 8: Modelin eğitim dağılımı (Training Distribution) nasıldır?

Eğitim verisinde;

- Hangi konular,
- Hangi diller,
- Hangi yazım stilleri,
- Hangi alanlar

daha fazla veya daha az temsil edilmektedir?

Bu sorunun cevabı model davranışlarındaki önyargıları (Bias) anlamaya yardımcı olur.


## Karar Mekanizmasına Yönelik Sorular (Decision Logic Questions)

Bu bölümde modelin kararlarını nasıl verdiği incelenir.

### Soru 9: Model tahmin yaparken hangi özellikleri kullanıyor?

Attribution yöntemleri (örneğin SHAP, Integrated Gradients ve Attention analizleri), girişteki hangi özelliklerin model kararını en fazla etkilediğini ortaya çıkarabilir.

### Soru 10: Model hangi iç temsilleri (Internal Representations) öğrendi?

Probing Classifier yöntemleri kullanılarak ara katmanlarda hangi bilgilerin temsil edildiği analiz edilebilir.

### Soru 11: Karar sınırları (Decision Boundaries) nerede bulunuyor?

Özellikle sınıflandırma modellerinde;

- Sınıflar arasındaki karar sınırları nerede?
- Belirli bir tahmin karar sınırına ne kadar yakın?

gibi sorular araştırılabilir.

### Soru 12: Modelin sistematik önyargıları (Systematic Biases) nelerdir?

Model;

- Farklı demografik gruplara farklı davranıyor mu?
- Kültürel önyargılar içeriyor mu?
- Belirli anahtar kelimeler belirli davranışları tetikliyor mu?

Bu sorular modelde bulunan sistematik önyargıları ortaya çıkarmaya yardımcı olur.


## Yetenek ve Sınırlamalara Yönelik Sorular (Capability and Limitation Questions)

Son grup sorular modelin neleri yapabildiğini ve hangi durumlarda başarısız olduğunu anlamaya yöneliktir.

### Soru 13: Modelin belgelenmemiş hangi yetenekleri bulunuyor?

Modeller, açıkça eğitilmedikleri halde zamanla ortaya çıkan yeni yetenekler (Emergent Capabilities) sergileyebilir.

Sistematik testler sayesinde modelin tüm yetenek spektrumu ortaya çıkarılabilir.

### Soru 14: Model hangi durumlarda başarısız oluyor ve neden?

Adversarial örnekler, dağılım dışı girdiler (Out-of-Distribution Inputs) ve uç durumlar (Edge Cases), modelin başarısız olduğu senaryoları ortaya çıkarmaya yardımcı olur.

### Soru 15: Modelin kalibrasyonu (Calibration) nasıldır?

Modelin ifade ettiği güven seviyesi gerçekten doğruluk oranıyla uyumlu mudur?

Eğer modelin güven düzeyi ile gerçek performansı arasında önemli farklılıklar bulunuyorsa, bu durum eğitim veya eğitim sonrası süreçlerde (Post-Training) çeşitli problemlere işaret edebilir.
