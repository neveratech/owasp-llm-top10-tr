# LLM06:2026 Unbounded Consumption — Sınırsız Tüketim

## Tanım

Sınırsız Tüketim; bir LLM uygulamasının aşırı ve denetimsiz çıkarımlara izin
vermesiyle oluşur — saldırganların servis erişilebilirliğini bozmasına, sürdürülemez
finansal maliyetler yüklemesine veya model klonlamayla fikrî mülkiyet çalmasına olanak
tanır. Hepsinin ortak zafiyet sınıfı aynıdır: kaynakların nasıl tüketildiğine dair
yeterli kontrolün yokluğu.

LLM'lerin yüksek hesaplama talepleri — özellikle bulut ve token başına ödeme
ortamlarında — onları kaynak istismarına ve yetkisiz kullanıma doğaları gereği açık
kılar. Bu tehdidin belirleyici özelliği **maliyet asimetrisidir**: saldırganlar,
kendilerine ihmal edilebilir maliyetle orantısız pahalı hesaplama tetikleyebilir —
kurgulanmış komutlarla, çalınmış kimlik bilgileriyle veya manipüle edilmiş iş
akışlarıyla.

Bu risk şunlarla katmerlenir: büyük veya yetersiz kısıtlanmış çıktı bütçeli
genişletilmiş-düşünme ve akıl yürütme modellerinin artan benimsenmesi; istek başına
hesaplama maliyetini önemli ölçüde yükselten multimodal modeller; tek bir isteği
zincirleme aşağı akış işlemlerine büyüten ajan mimarileri ve araç kullanım
protokolleri (MCP gibi) ve yeni tedarik zinciri saldırı yüzeyleri getiren paylaşımlı
çıkarım altyapısı. Geleneksel istek-hızı sınırlama tek başına artık yeterli değildir.
Etkili savunma; token farkındalıklı maliyet kontrolleri, sert harcama tavanları, ajan
düzeyi devre kesiciler ve sürekli maliyet-atfetme izlemesi ister.

## Yaygın Risk Örnekleri

1. **Değişken uzunluklu girdi seli ve çıktı patlaması.** Saldırganlar, işleme
   verimsizliklerini istismar ederek LLM'i değişen uzunlukta çok sayıda girdiyle
   boğabilir. Bu, kaynakları tüketip sistemi yanıtsız bırakabilir ve servis
   erişilebilirliğini ciddi etkiler. Fine-tuning zehirlemesiyle çıktı patlaması da
   buna dâhildir: tek bir kötücül eğitim örneği, modelin dizi-sonu davranışını bozar
   ve her istekte çıktıyı azami uzunluğa iter (Gao vd., 2024).
2. **Cüzdan Hizmet Reddi (Denial of Wallet, DoW).** Yüksek hacimli işlem başlatan
   saldırganlar, bulut tabanlı AI servislerinin kullanım başına maliyet modelini
   istismar eder — sağlayıcıya sürdürülemez mali yük bindirir ve finansal yıkım riski
   doğurur.
3. **Büyük bağlam istismarı.** Sınıra yakın tekrarlı istekler, bağlam birikimi ve
   uygulama tarafında yeniden parçalama, orantısız hesaplama ve bellek tüketir. Çoğu
   API, bağlam penceresini aşan girdileri doğrudan reddeder; kalıcı risk, sınırların
   hemen içinde kalıp istek başına maliyeti şişiren isteklerden gelir.
4. **Akıl yürütme döngüsü ve düşünme-token'ı tükenmesi.** Saldırganlar,
   genişletilmiş-düşünme modellerini uzamış veya sonlanmayan akıl yürütme döngülerine
   zorlayarak kaynak tükenmesine yol açan kısa, masum görünüşlü komutlar kurgular —
   girdi boyutu filtrelerini atlatırken devasa düşünme-token bütçeleri tüketir (Li
   vd., 2025). Bu komutlar küçük ve meşru göründüğünden standart girdi doğrulaması
   koruma sağlamaz.
5. **Kaynak aşırı tüketimi için optimize edilmiş düşmanca girdiler.** Saldırganlar,
   hesaplama maliyetini azamileştiren girdiler kurgulamak için optimizasyon teknikleri
   kullanır. Bu, modelden yalnızca kaynak-yoğun bir görev istemekten farklıdır; sünger
   örnekleri (sponge examples) (Shumailov vd., 2020) ve düşmanca görsel bozulmaları
   kapsar. Gradyan tabanlı ve gradyansız tekniklerle girdi optimizasyonunu içerir.
   Akıl yürütme döngüsü saldırılarının aksine, yalnızca komut tasarımı değil, girdi
   uzayında açık optimizasyon gerektirir.
6. **Multimodal girdi ve çıktılar.** Multimodal modeller; görselleri, sesi ve videoyu
   çok sayıda token'a çevirir; tek bir istek, karşılaştırılabilir salt-metin istekten
   epey pahalı olabilir. Kesin ek yük; modele, sağlayıcıya, çözünürlüğe, medya
   süresine ve ön işleme boru hattına göre değişir.
7. **Model çıkarma ve damıtma hırsızlığı.** Saldırganlar, kısmi bir modeli
   kopyalamaya veya işlevsel bir eşdeğeri fine-tune etmeye yetecek çıktıyı toplamak
   için model API'sini kurgulanmış girdilerle sorgular. Logit ve log-olasılıklarının
   açığa çıkması çıkarmayı önemli ölçüde hızlandırır (Carlini vd., 2024). Model
   ağırlıklarının veya mimarisinin zamanlama ya da paylaşımlı altyapı gözlemiyle yan
   kanaldan çıkarılması LLM02:2026 Hassas Bilgi İfşası kapsamındadır.
8. **Model kaynaklarını sele boğan ajan-araç etkileşimleri.** Saldırganlar, LLM
   tabanlı uygulamayı özyinelemeli veya sonsuz araç çağırma döngülerine zorlayarak
   LLM kaynaklarını aşırı kullanan araçlar yayımlayabilir. Görünürde meşru araç
   eylemleri mali yüke veya hizmet kalitesi kaybına dönüşebilir. Tek bir araç çağrısı
   çok daha büyük sayıda eyleme dallandığında, LLM tek görevden doğan yüzlerce çağrıyı
   yönetmek zorunda kalabilir — token aşırı kullanımı tırmanır.
9. **Çıkarım altyapısı istismarı.** Saldırganlar; güvensiz serileştirme çözme
   kusurları, özel-token enjeksiyonu ve enjekte edilen sohbet şablonları yoluyla LLM
   sunum çatılarındaki (vLLM, TensorRT-LLM, SGLang, Triton, Ollama) zafiyetleri
   hedefleyerek servisleri çökertir veya model kaynaklarını tüketir.

## Önleme ve Azaltma Stratejileri

1. **Hız sınırlama ve girdi boyutu doğrulaması.** Tek bir kaynağın belirli sürede
   yapabileceği istek sayısını kısıtlamak için hız sınırları ve kullanıcı kotaları
   uygulayın. Saniye başına istekten öteye geçin: dakika başına token, gün başına
   token ve istek başına tahminî maliyet sınırları zorlayın. İstekleri çıkarım
   başlamadan reddetmek için ön-uçuş token tahmini kullanın. Girdilerin makul boyut
   sınırlarını aşmadığını doğrulamayı da içerir.
2. **Sert harcama tavanları.** API anahtarı, kullanıcı, ekip ve bulut hesabı başına
   geçersiz kılınamaz bütçe tavanları belirleyin. Bunlar, hızla biriken iş yüklerinin
   geride bırakabileceği uyarı eşikleri değil, aşıldığında çıkarımı durduran zorlama
   mekanizmaları olmalıdır. Harcama tavanları modaliteler ve araç protokolleri arası
   maliyet farklarını da hesaba katmalıdır.
3. **Kaynak tahsisi yönetimi.** Tek bir kullanıcının veya isteğin aşırı kaynak
   tüketmesini önlemek için kaynak tahsisini dinamik izleyip yönetin.
4. **Sandbox teknikleri.** LLM'in ağ kaynaklarına, iç servislere ve API'lere erişimini
   kısıtlayın. Uygulamanın ulaşabildiklerini daraltmak, saldırganın çıkarılan model
   bilgisini veya veriyi harici bir hedefe sızdırma yeteneğini sınırlar.
5. **Zarif bozulma (graceful degradation).** Sistemi ağır yük altında tümden çökmek
   yerine kısmi işlevselliği koruyarak zarifçe bozulacak şekilde tasarlayın.
6. **Kuyruğa alınan eylemleri sınırlayın, sağlam ölçekleyin.** Kuyruktaki eylem
   sayısına ve toplam eylemlere kısıt getirin; değişken talebi karşılamak ve tutarlı
   performans için dinamik ölçekleme ile yük dengelemeyi dâhil edin.
7. **Düşmanca bozulmaları tarayın.** Model girdilerini — özellikle LVLM'lere (büyük
   görsel dil modelleri) giden görsel girdileri — kaynak aşırı tüketimine yol
   açabilecek düşmanca bozulma kanıtı için tarayın.
8. **Kaynak-yoğun araç etkileşimlerini tespit edin.** Belirli bir oturumun açık bitiş
   durumu olmayan özyinelemeli veya kaynak-yoğun bir eyleme neden olup olmadığını
   belirlemek için ajan-araç etkileşimlerini izleyin. Bir aracın standart token
   tüketim kalıplarından sapmasını yakalamak için normal araç davranışı taban
   çizgileri kurun.
9. **Ajan devre kesicileri.** Tüm ajan yürütmelerinde adım sınırları, özyineleme
   derinliği sınırları, süre sınırları ve çalıştırma başına maliyet tavanları
   zorlayın. Özyinelemeli döngüleri yakalamak için durum hash'lemesi kullanın.
10. **Çıkarım altyapısı sıkılaştırması.** Sunum çatılarını güncel tutun. Güvensiz
    serileştirme çözmeyi kapatın, özel-token geçişini kısıtlayın ve tüm çıkarım uç
    noktalarında kimlik doğrulamayı zorlayın.

## Örnek Saldırı Senaryoları

**Senaryo #1 — Denetimsiz girdi boyutu:** Saldırgan, metin verisi işleyen bir LLM
uygulamasına alışılmadık büyüklükte girdi gönderir; aşırı bellek kullanımı ve CPU yükü
doğar, sistem çökebilir veya servis ciddi yavaşlar.

**Senaryo #2 — Tekrarlı istekler:** Saldırgan, LLM API'sine yüksek hacimli istek
iletir; hesaplama kaynakları aşırı tüketilir ve servis meşru kullanıcılara kapanır.

**Senaryo #3 — Kaynak-yoğun sorgular:** Saldırgan, LLM'in hesaplama açısından en
pahalı süreçlerini tetikleyecek girdiler kurgular; uzamış GPU kullanımı ve olası
sistem arızası.

**Senaryo #4 — Cüzdan Hizmet Reddi (DoW):** Saldırgan, bulut tabanlı AI servislerinin
kullanım başına ödeme modelini istismar edecek aşırı işlem üretir; sağlayıcı için
sürdürülemez maliyetler.

**Senaryo #5 — İşlevsel model kopyalama:** Saldırgan, LLM API'sini sentetik eğitim
verisi üretmek için kullanır ve başka bir modeli fine-tune eder — işlevsel bir eşdeğer
yaratır ve geleneksel model çıkarma sınırlarını atlatır.

**Senaryo #6 — LVLM görsel girdisinde bozulmalar:** Saldırgan, bir LVLM'in çıktısında
token aşırı tüketimine yol açacak biçimde optimize edilmiş bozulmalar içeren düşmanca
görsel girdiler kurgular (Gao vd., 2025).

**Senaryo #7 — Çok turlu araç çağırma döngüleri ve araç çağrısı dallanması:**
Saldırgan, bir ajana özyinelemeli döngüsel görevler veya çok sayıda araç çağrısı
gerektiren görevler yaptıran kötücül bir araç yayımlayabilir (örneğin açık kaynak bir
depoda bir Claude Skill üzerinden). Bu aracı ajanlarına katan geliştiriciler aşırı
token tüketimi ve servis istikrarsızlığı riskine girer.

**Senaryo #8 — Ajan oturumlarında büyüyen LLM bağlamı:** Saldırgan veya masum bir
kullanıcı açık bir ajan oturumunu sürdürür ve içeriği kademeli enjekte eder; her
çıkarım, birikmiş bağlamın tamamını yeniden işler. Tur başına maliyet bağlam
büyüdükçe tırmanır: ilk turda yaklaşık 0,001 dolardan 100. turda yaklaşık 0,50
dolara. Hiçbir istek tek başına hız sınırlarını tetiklemez — her biri bütçe içinde
kalır — ama çok sayıda eşzamanlı veya uzun ömürlü oturumun toplamı yüzlerce dolara
ulaşır.

---

*Orijinal: OWASP Top 10 for LLM Applications 2026, OWASP GenAI Security Project,
CC BY-SA 4.0 — https://genai.owasp.org. Bu gayriresmî Türkçe çeviri aynı lisansla
dağıtılır. Kaynakça için orijinal belgenin References bölümüne bakınız.*
