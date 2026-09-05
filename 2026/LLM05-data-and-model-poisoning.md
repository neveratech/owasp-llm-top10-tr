# LLM05:2026 Data and Model Poisoning — Veri ve Model Zehirlenmesi

## Tanım

Veri ve Model Zehirlenmesi; bir saldırganın (veya güvensiz bir sürecin) veri ya da
model artefaktlarını manipüle ederek bir AI sistemine zararlı davranış, yanlılık veya
istismar edilebilir zayıflık gömdüğü bir saldırı ve arıza sınıfını tanımlar. Modern
GenAI ortamlarında zehirlenme, geleneksel anlamdaki "eğitim verisiyle" sınırlı
değildir. Verinin alındığı, dönüştürüldüğü, getirildiği veya yeniden kullanıldığı her
yerde — ön eğitim, fine-tuning, gömme üretimi, getirme destekli üretim (RAG) ve model
dağıtımı sırasında — ortaya çıkabilir. Sonuç, hâlâ işlevsel görünebilen ama güveni,
emniyeti ve güvenliği aşındıracak biçimde davranan bir AI sistemidir.

Veri zehirlenmesi; ön eğitim, fine-tuning veya gömme verisinin zafiyet, arka kapı ya
da yanlılık sokmak üzere kurcalanmasıyla oluşur. Kasıtlı (kötücül zehirleme) veya
kasıtsız (kötü veri hijyeni, kirlenmiş kaynaklar) olabilir. Manipülasyon model
bütünlüğünü bozar: model yanlış örüntüler öğrenir, kötücül korelasyonları içselleştirir
veya hatalı davranmaya koşullanır. Sonuçlar; zararlı çıktılar, zayıflayan yetenekler ve
düşen güvenilirliktir.

Ana fikir: zehirlenme, tek bir çalışma zamanı hatasını değil, modelin "öğrenme
sürecini" hedefler. Kod düzeltmesiyle yamalanabilen tipik yazılım zafiyetlerinden
farklı olarak, zehirlenme; verinin yeniden doğrulanmasını, yeniden eğitimi, model
değişimini veya boru hattının yeniden tasarımını gerektirebilir — pahalı ve
operasyonel açıdan yıkıcı işler.

Zehirlenme, LLM yaşam döngüsünün birden çok evresinde gerçekleşebilir:

- **Ön eğitim:** Kötücül kurgulanmış veya kirlenmiş derlemler; modele zararlı
  örüntüler, güvensiz talimatlar veya çarpık temsiller emdirtir.
- **Fine-tuning:** Manipüle edilmiş veri kümeleri, alana özgü arıza kipleri veya gizli
  tetikleyiciler sokar.
- **Gömme ve vektörleştirme:** Zehirlenme, getirilen içeriği etkilemek için saklanan
  vektörleri hedefler — yönlendirilmiş yanıtlar veya ince yanlış bilgiyle sonuçlanır.
- **Transfer öğrenme / model yeniden kullanımı:** Ele geçirilmiş kaynak modeller, bu
  bozulmayı aşağı akış sistemlerine aktarır.
- **Sürekli öğrenme boru hatları:** Yeterli doğrulama olmadan otomatik alım,
  saldırganların model davranışını kademeli biçimlendirmesine izin verir.

Kuruluşlar harici veri kümelerine, RAG boru hatlarına, paylaşımlı model depolarına ve
ajan tabanlı iş akışlarına giderek daha çok dayandığı için veri zehirlenmesi yüzeyi
genişler. Paylaşımlı depolardan dağıtılan modeller, birlikte paketlenen
ağırlık-dışı artefaktlar üzerinden risk taşıyabilir: kötücül serileştirme çözme
(örneğin pickle dosyaları) ile sohbet şablonlarının, tokenizer yapılandırmalarının,
LoRA/PEFT adaptörlerinin ve nicemleme artefaktlarının kurcalanması — her biri
yüklendiğinde zararlı kod çalıştırabilir veya model davranışını değiştirebilir. Bu tür
arka kapılar, bir tetikleyici davranışı değiştirene dek model davranışını el değmemiş
bırakabilir; modele bir **uyuyan ajan** olma fırsatı yaratır.

Ajan tabanlı dağıtımlarda zehirlenme riskleri araç entegrasyonlarına, kalıcı bellek
depolarına ve RLHF geri bildirim döngülerine uzanır. Bu saldırı yüzeyleri OWASP Top 10
for Agentic Applications'ta derinlemesine ele alınır.

Bu madde, kalıcı verinin veya model davranışının **dayanıklı** bozulmasını kapsar.
Çıkarım anında getirilen içerikle taşınan komut talimatları LLM01:2026 Komut
Enjeksiyonu, gömme geometrisini istismar eden saldırılar LLM09:2026 Vektör ve Gömme
Zafiyetleri kapsamındadır.

## Yaygın Risk Örnekleri

1. **Eğitim ve fine-tuning verisi zehirlenmesi:** Saldırganlar veri kümelerine yanlı
   veya kötücül içerik enjekte eder. Hedefli bir varyant, genel doğruluğu korurken
   reddetme davranışlarını bilinçli aşındırır — bozulma standart değerlendirmeyle
   tespit edilemez hâle gelir.
2. **Finansal model verisi zehirlenmesi:** Saldırganlar, dolandırıcılık tespit
   modellerine dolandırıcılığı meşru diye etiketleyen yanlış etiketli işlem verisi
   enjekte eder. Model gerçek tehditleri görmezden gelmeyi öğrenir; dolandırıcılık
   atlatması mümkün olur ve AI destekli finansal sistemlere güven sarsılır.
3. **Açık kaynak veri kümesi tedarik zinciri zehirlenmesi:** Saldırganlar yaygın
   kullanılan veri kümelerine kötücül veri katar. Paylaşılan bir kümeye sokulan
   tetikleyici ifadeler, onun üzerinde fine-tune edilen sayısız aşağı akış modeline
   yayılabilir; arka kapı keşfedildiğinde maliyetli yeniden eğitim gerekir.
4. **Düşük hacimli, yüksek etkili arka kapı zehirlenmesi:** 250 kadar az zehirli
   belge, veri kümesi boyutundan bağımsız olarak 600M'den 13B parametreye modelleri
   ele geçirir (Souly vd., 2025). Stratejik asgari manipülasyon önemli etki için
   yeterlidir.
5. **AI öneri / bellek zehirlenmesi:** Saldırganlar, AI belleğini veya önerilerini
   fark edilmeden manipüle etmek için web içeriğine gizli talimatlar gömer — ajan
   tabanlı sistemlerde ve kalıcı bellekte risklerin göstergesi.
6. **RAG bilgi tabanı zehirlenmesi:** Hedeflenen sorgu başına enjekte edilen tek bir
   optimize zehirli metin, getirme derlemindeki doğru içeriği bastırabilir; saldırı,
   yeniden ifadeleme, talimatla önleme ve tespit tabanlı savunmalara karşı yüksek
   başarısını korur (Zhang vd., 2025).
7. **Ajan / çoklu sistem zehirlenmesi:** Çok ajanlı iş akışlarındaki zehirli girdiler,
   tek tek modelleri değil, AI destekli ekosistemlerin tamamında davranışı ve veri
   erişimini etkiler.
8. **Sağlık modeli zehirlenmesi:** Tıbbi eğitim verisinin asgari zehirlenmesi,
   standart değerlendirmeleri geçerken model çıktılarını önemli ölçüde değiştirir —
   güvenlik-kritik alanlarda güvensiz öneriler doğurur.
9. **Tedarik zincirinde kötücül AI modelleri:** Saldırganlar, gömülü arka kapılı ele
   geçirilmiş modelleri halka açık depolar üzerinden dağıtır. Bu modelleri indiren
   kuruluşlar, bilmeden gizli tetikleyicileri veya sistem ele geçirmesini devralır.

## Önleme ve Azaltma Stratejileri

1. Veri kümesi ve model soy kütüğünü SBOM/ML-BOM (örneğin CycloneDX) ile izleyin,
   imzalama ve doğrulamayı zorlayın, yaşam döngüsü evreleri boyunca veri bütünlüğünü
   sürekli doğrulayın.
2. Gelen tüm veriler için katı doğrulama kurun, üçüncü taraf tedarikçileri inceleyin
   ve yanlılık ya da düşmanca manipülasyonu erken tespit için çıktıları güvenilir
   kaynaklarla karşılaştırın.
3. RAG sistemlerini; güven sınırları zorlayarak, getirilen içeriği filtreleyerek,
   kaynak puanlaması uygulayarak ve sistem talimatlarını harici veriden yalıtarak
   koruyun.
4. Modelin doğrulanmamış veri, araç veya harici sistemlerle etkileşimini sınırlamak
   için sandbox ve katı yalıtım kontrolleri kullanın.
5. Eğitim, gömme ve çıkarım boru hatlarında istatistiksel ve AI tabanlı anomali
   tespiti uygulayın; ince zehirlenme etkilerini zaman içinde yakalamak için eğitim
   kaybını, çıktıları ve davranışı tanımlı eşiklere karşı kayma ve anomali açısından
   izleyin.
6. Güvenilmeyen veriye ve alanlar arası bulaşmaya maruziyeti azaltmak için
   fine-tuning'de derlenmiş, alana özgü veri kümeleri kullanın.
7. Yetkisiz veri enjeksiyonunu önlemek için en az yetkili erişimi, ağ bölümlemesini
   ve katı veri erişim kontrollerini zorlayın.
8. Veri kümesi değişikliklerini izlemek, sürüm geçmişi tutmak ve zehirlenme tespit
   edildiğinde geri alma ile adli analiz sağlamak için veri sürüm kontrolü (örneğin
   DVC) kullanın.
9. Otomatik yeniden eğitim ve geri bildirim döngülerini; gelen veriyi doğrulayarak,
   insan gözetimi gerektirerek ve manipüle edilmiş tercih sinyalleriyle kademeli
   zehirlenmeye karşı hız sınırları uygulayarak denetim altında tutun.
10. Gizli arka kapıları bulmak için modelleri düşmanca girdiler ve tetikleyici tabanlı
    komutlarla sürekli red team'leyin. Güvenlik hizalamasının arka kapıları
    kaldırdığını varsaymayın; her hizalama döngüsünden sonra özel tetikleyici
    yoklaması gerekir (Hubinger vd., 2024).
11. Getirilen içeriğin çıktıları etkilemeden önce doğrulanmasını sağlayan doğrulama
    katmanlı temellendirme (grounding) teknikleri uygulayın.
12. Çıkarım artefaktlarını — sohbet şablonları, tokenizer yapılandırmaları, LoRA/PEFT
    adaptörleri ve nicemleme artefaktları dâhil — güvenlik açısından önemli kod olarak
    ele alın. Dağıtım öncesi imzalama, hash doğrulama, fark (diff) kontrolleri ve
    statik analizi zorlayın.

## Örnek Saldırı Senaryoları

**Senaryo #1:** Saldırgan, iç bilgi deposuna manipüle edilmiş belgeler ekler. Zehirli
belgeler yanıtlarda yüzeye çıkar; yanlış önerilere, manipüle edilmiş iş kararlarına
veya itibar hasarına yol açar.

**Senaryo #2:** Saldırgan, AI araçlarınca özetlenen web sayfalarına gizli talimatlar
gömer. RAG veya bellek sistemlerine alındığında talimatlar modeli belirli ürünleri
önermeye yönlendirir — finansal manipülasyon ve AI çıktılarına güven kaybı.

**Senaryo #3:** Saldırgan, otomatik yeniden eğitim geri bildirim döngüsüne kurgulanmış
girdiler gönderir. Altyapı erişimi gerekmez; standart kullanıcı arayüzü erişimi yeter.
Sonuç: düşen doğruluğa, yanlı çıktılara veya güvensiz önerilere doğru yavaş model
kayması.

**Senaryo #4:** Kötücül bir iç aktör, eğitim veri kümesine yanlış etiketli işlem
verisi enjekte eder. Model dolandırıcılığı tespit edemez; finansal kayıplar, uyum
ihlalleri ve düzenleyici ihlal doğar.

**Senaryo #5:** Saldırgan, halka açık bir depoya zehirlenmiş ön eğitilmiş ağırlıklar
yükler. Standart güvenlik eğitimi gömülü arka kapıları kaldıramaz (Hubinger vd.,
2024). Kuruluşlar sistem ele geçirmesi, veri sızıntısı veya ölçekli hedefli
manipülasyonla karşılaşır.

**Senaryo #6:** Saldırgan, bir modelin sohbet şablonunu (örneğin bir GGUF paketinde
veya tokenizer yapılandırmasında) tetikleyiciyle etkinleşen koşullu talimatlarla
değiştirir. Halka açık bir hub'dan yeniden dağıtılan model masum girdilerde normal
davranır. 18 model ve 4 çıkarım çalışma zamanında doğrulanmıştır: tetik koşullarında
olgusal doğruluk %90'dan %15'e düşer ve URL yayma %80'in üzerinde başarıya ulaşır
(Fogel vd., 2026).

**Senaryo #7:** Geliştirici, üçüncü taraf bir modeli güvensiz serileştirmeyle
(örneğin pickle) yükler. Gömülü kötücül kod yükleme sırasında çalışır — ana makine ele
geçirme, yanal hareket ve altyapı ihlali.

**Senaryo #8:** Paylaşımlı bir AI ortamında bir kiracı, ortak gömme veya bellek
katmanlarına düşmanca veri enjekte ederek diğer kiracıların yanıtlarını etkiler —
kiracılar arası bulaşma ve gizlilik riski.

**Senaryo #9:** Saldırgan, birden çok oturum boyunca bir AI ajanının kalıcı belleğine
kötücül talimatlar enjekte eder. Ajan, saldırgan kontrolündeki mantığı önceliklendirmeye
başlar — uzun vadeli iş akışı manipülasyonu ve gizli kalıcılık.

---

*Orijinal: OWASP Top 10 for LLM Applications 2026, OWASP GenAI Security Project,
CC BY-SA 4.0 — https://genai.owasp.org. Bu gayriresmî Türkçe çeviri aynı lisansla
dağıtılır. Kaynakça için orijinal belgenin References bölümüne bakınız.*
