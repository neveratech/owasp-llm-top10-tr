# LLM09:2026 Vector and Embedding Weaknesses — Vektör ve Gömme Zafiyetleri

## Tanım

Vektör ve gömme zafiyetleri; metni, görselleri, kodu veya sesi sayısal temsillere
çeviren ve modelin ne göreceğine benzerlik aramasıyla karar veren her LLM
uygulamasında güvenlik riski oluşturur. Getirme Destekli Üretim (RAG) en bilinen
örnektir; ama aynı mekanizma vektör destekli ajan belleğinin, semantik önbelleklerin
ve tekilleştirme boru hatlarının da temelidir. Benzerlik araması bir veri kaynağıyla
komut arasında durduğu her yerde, gömme katmanı uygulamanın güven sınırının parçası
hâline gelir.

Bu zayıflıklar komut enjeksiyonundan farklıdır: modelin talimat-izleme davranışını
değil, gömme uzayının geometrisini ve benzerlik aramasının mekaniğini istismar
ederler. Birçoğu, getirilen içerik hiçbir kötücül talimat taşımasa bile başarılı olur.
Yararlı bir çerçeve: **zehirleme sistemi yanlış yapar, inversion sızdırtır, jamming
susturur, erişim kontrolü başarısızlığı ise ayrım gözetmez kılar.**

Bu madde, başarısı gömme katmanına bağlı saldırıları kapsar. Getirilen içerik
üzerinden dolaylı komut enjeksiyonu LLM01:2026 Komut Enjeksiyonu'nda, gömme modelinin
eğitim zamanı zehirlenmesi LLM05:2026 Veri ve Model Zehirlenmesi'nde, vektör deposu
kütüphanelerindeki serileştirme kusurları LLM04:2026 Tedarik Zinciri'nde ve gömme
geometrisine dayanmayan ajan-belleği saldırıları ASI06:2026 Memory and Context
Poisoning'de (OWASP Top 10 for Agentic Applications) ele alınır. Vektörsüz getirme
sistemleri (yalnız BM25, LLM-yerli ağaç gezinmesi) geometrik olmayan riskleri devralır
ama LLM09 saldırı yüzeyleri yoktur.

## Yaygın Risk Örnekleri

1. **Paylaşımlı benzerlik aramasıyla kiracılar arası sızıntı.** Çok kiracılı
   dağıtımlarda benzerlik araması, erişim kontrolü uygulama katmanında uygulanmadan
   önce sıklıkla tüm indekste çalışır. Saldırgan, indeksi kurgulanmış sorgularla
   yoklayabilir ve belgeleri hiç görmeden; sonuç sayılarından, skor dağılımlarından ve
   zamanlamadan diğer kiracıların belgelerinin varlığını, konusunu ve yaklaşık
   hacmini çıkarsayabilir. Saldırı, her belge doğru etiketlense ve her API çağrısı
   kimlik doğrulasa bile başarılıdır; çünkü erişim kontrolü kararı, gömme-uzayı
   araması çalıştıktan **sonra** verilir. Vektör veritabanı yazılımlarındaki klasik
   kimlik doğrulama kusurları — örneğin CVE-2025-64513 (Milvus, sahte sourceID
   başlığıyla kimlik doğrulama atlatması, CVSS 9.3) ve CVE-2025-69286 (RAGFlow,
   öngörülebilir token türetmeyle hesap ele geçirme, CVSS 9.3) — burada kapsam dışıdır
   ama geometrik riski katmerler: bir vektör deposu sızıntısı inversion'la kaynak
   belgelere geri kazanılabildiğinden (Risk #2), vektör veritabanındaki bir kimlik
   doğrulama hatası, aynı hatanın bir belge veritabanında veya anahtar-değer
   deposunda olduğundan daha ağır sonuç taşır.
2. **Gömme inversion'ı.** Saklanan gömmeler kaynak metni geri kazanacak biçimde
   tersine çevrilebilir. Bildirilen geri kazanım oranları, cümle gömmelerinden
   kelimelerin kabaca %50–70'inden, her kodlayıcı için bir inversion modeli eğiten
   Vec2Text yöntemiyle kısa 32-token'lık girdilerin %92 birebir yeniden kurulumuna
   uzanır (Morris vd., 2023). ZSInvert (C. Zhang vd., 2025) ve Zero2Text (Kim vd.,
   2026) kodlayıcıya özgü eğitim olmadan sıfır-atışlı çalışır, alanlar arası ve kara
   kutu ortamlarda işler ve depolamada eklenen diferansiyel-gizlilik gürültüsüne
   karşı etkili kalır. Operasyonel sonuç: vektör veritabanı yedekleri, üçüncü taraf
   servislere gönderilen gömmeler ve yanlış yapılandırılmış bulut depolamayla açığa
   çıkan gömmeler, alttaki belgelerin sızıntısına eşdeğer sayılmalıdır. GDPR ve
   benzeri rejimlerde ihlal bildirimi, veri öznelerine yönelik riske bağlıdır; modern
   gömmeler tersine çevrilebildiğinden bu risk gerçektir.
3. **Getirme zamanı veri zehirlenmesi.** Derleme yazabilen bir saldırgan — halka
   açık kazıma boru hatları, dosya yüklemeleri, iş ortağı beslemeleri veya ele
   geçirilmiş iç kaynaklar yoluyla — gömmesi hedef sorgunun yakınına düşen içerik
   kurgulayabilir. Kullanıcı o sorguyu gönderdiğinde, saldırganın içeriği getirilir
   ve LLM'e güvenilir bağlam olarak verilir. Yayımlanmış saldırılar, milyonlarca
   belgelik derlemlerde ve kara kutu sistemlere karşı bile bir avuç zehirli belgeyle
   güvenilir biçimde yüksek başarıya ulaşır. Başarılı saldırı iki koşulu aynı anda
   gerektirir: zehirli içerik getirilmelidir (geometrik) ve yanıtı yönlendirmelidir
   (üretim). Savunucular iki katmandan birine müdahale edebilir. MITRE ATLAS bu
   sınıfı Persistence taktiği altında AML.T0070 (RAG Poisoning) olarak kataloglar.
   Gömme modelinin kendisinin eğitim zamanı zehirlenmesi LLM05:2026'dır.
4. **Getirme karıştırması (retrieval jamming).** Saldırganlar, belirli bir sorgu için
   getirilecek ve LLM'e yanıtlamayı reddettirtecek ya da bilgisinin olmadığını iddia
   ettirecek biçimde mühendislenmiş bir "engelleyici" belge sokarak bir RAG sistemini
   devreden çıkarabilir. Zehirlemeden farklı olarak, engelleyici hiçbir kötücül
   talimat taşımaz; getirme mekaniğini ve LLM güvenlik davranışını istismar eder.
   Hedef gömme modeline veya LLM'e erişim olmadan kara kutu optimizasyonla üretilmiş
   tek bir engelleyici belge yeterlidir. Bu, getirme katmanına yönelik bir
   erişilebilirlik saldırısıdır.
5. **Benzerlik aramasıyla üyelik çıkarımı.** Saldırgan, belirli bir belgenin (tıbbi
   kayıt, hukuki dosya, İK şikâyeti) ne söylediğini değil, indekste **var olup
   olmadığını** öğrenmek ister. Uygulama istemciye ham benzerlik skorları veya
   mesafeler döndürüyorsa, indeks LLM hiç işin içine girmeden doğrudan bir üyelik
   kâhinine dönüşür. Yalnızca üretilmiş yanıtlar döndürülüyorsa bile saldırgan, kısmi
   belgeler veya bozulmuş sorgular gönderip yanıtları analiz ederek üyeliği yine
   çıkarsayabilir. İçerik korunsa dahi üyelik bilgisinin kendisi hassas olabilir.
6. **Semantik önbellek ve tekilleştirme zehirlenmesi.** Semantik önbellekler ve
   yakın-kopya tespiti, iki içeriğin "aynı" olduğuna kosinüs-benzerlik eşiğiyle karar
   verir. Bu eşiğin hemen üstüne veya altına düşen içerik kurgulayabilen saldırgan;
   bir önbellek girdisini zehirleyip anlamca eşdeğer tüm sorgulara saldırgan metnini
   servis ettirebilir, zehirli içeriğin yakın-kopyalarıyla tekilleştirmeyi
   atlatabilir veya meşru yeni içeriğin kopya diye sessizce düşürülmesini
   zorlayabilir. Wu vd. (2026), AWS, Azure ve Alibaba dağıtımlarında uçtan uca
   semantik önbellek zehirlemesi gösterir. Z. Zhang vd. (2026), hedef kodlayıcıya
   erişim olmadan eşik-sınırında vektörler mühendislemek için vekil gömme modelleri
   kullanan kara kutu anahtar-çarpışma saldırıları sunar. Üç arıza kipi de gömme-uzayı
   geometrisine bağlıdır ve belge düzeyi kontrollere görünmezdir.
7. **Multimodal gömme zehirlenmesi.** CLIP ve ColPali gibi çapraz-modal
   kodlayıcılar; görselleri, sesi, kodu ve metni tek vektör uzayına eşler. Metin dışı
   içerik katabilen bir saldırgan, gömmesi hassas bir metin sorgusuna yakın duran bir
   görsel kurgulayabilir. Kullanıcı o sorguyu gönderdiğinde görsel güvenilir bağlam
   olarak getirilir. MM-PoisonRAG (Ha vd., 2025) ve Poisoned-MRAG (Liu vd., 2025),
   multimodal RAG boru hatlarında yerel ve küresel zehirlemeyi gösterir. "One Pic is
   All it Takes" çalışması, hedefli ve evrensel VD-RAG zehirlemesi için tek bir
   görselin yettiğini ortaya koyar. İnsan gözden geçirene görsel sıradan görünür;
   payload metin olmadığı için metin tabanlı içerik taraması da yakalamaz.

## Önleme ve Azaltma Stratejileri

1. **İzin ve erişim kontrolü.** Kiracı kapsamını getirme-sonrası filtre olarak değil,
   indeks sorgusunun içinde zorlayın ve sunucu tarafında doğrulayın. İstemcinin
   sağladığı kapsam bir öneridir, kontrol değil. Gömme ve benzerlik-arama uç
   noktalarını kiracı başına hız sınırlı, birinci sınıf API'ler olarak kimlik
   doğrulamasına bağlayın. Yüksek hassasiyetli iş yükleri için kiracı veya güven
   bölgesi başına fiziksel ayrık indeksler kullanın. Erişim kontrolünü parça (chunk)
   düzeyinde uygulayın: çoğunlukla halka açık bir belge gizli bir paragraf
   içerebilir.
2. **Veri doğrulama, kaynak kimlik doğrulaması ve köken.** İçeriği gömmeden önce
   normalize edin: sıfır genişlikli karakterleri, beyaz-üstüne-beyaz metni ve Unicode
   homoglifleri çıkarma aşamasında ayıklayın. Her gömme için köken izleyin (kaynak,
   alım zamanı, güven katmanı, boru hattı sürümü) — ele geçirilmiş partiler geçersiz
   kılınıp denetlenebilsin. Hassas indekslere gidecek dış kaynaklı içeriğe insan
   incelemesi uygulayın. Gömme modelinin kendisini de inceleyin: arka kapılı bir
   kodlayıcı, alınan her şeyin geometrisini bozar.
3. **Güven katmanına göre veri ayrımı.** Karışık güvenli içerik (harici web verisi,
   iç gizli belgeler, iş ortağı verisi) sert yalıtım olmadan aynı indeksi
   paylaşmamalıdır. İndeks düzeyi ayrım, paylaşılan indekste sınıflandırma
   etiketlerini yener; çünkü yanlış yapılandırma yolunu ortadan kaldırır.
4. **Alımda ve getirmede anomali tespiti.** Geniş bir yaygın sorgu yelpazesine
   alışılmadık yakın duran yeni vektörleri işaretleyin — getirme-kaçırma
   zehirlemesinin imzası. Çok fazla yüksek-benzerlikli eşleşme döndüren sorguları,
   gömme uç noktalarında olağandışı hacmi (sorgu tabanlı inversion'ın habercisi) ve
   alım sonrası beklenenden hızlı büyüyen kümeleri izleyin. İstemcilere ham benzerlik
   skorları döndürmeyin, getirme-sıralama katmanında gürültü ve çeşitlendirme
   ekleyin, kâhin gibi sorgulanabilecek uç noktaları hız sınırlayın. Cross-encoder
   yeniden sıralama saldırı maliyetini yükseltir ama köken ve alım kontrollerinin
   yerini tutmaz. Modern saldırılar getirme ile sıralamayı birlikte hedefler.
5. **Depolama yaşam döngüsü kontrolleri.** Kaynak belge silindiğinde gömmeleri
   sınırlı süre içinde silin ve mutabakat denetimleriyle doğrulayın. Vektör
   veritabanı yedeklerini kaynak belgelerle aynı hassasiyet katmanında ele alın.
   Gömmeleri, anahtarları uygulama katmanından ayrı yönetilen şifrelemeyle beklemede
   şifreleyin. Gömme modelini değiştirirken eski ve yeni vektörleri karıştırmak
   yerine derlemi yeniden gömün: heterojen gömmeler, benzerlik davranışında istismar
   edilebilir boşluklar yaratır. Gömme-API anahtarlarını sır olarak ele alın: sızan
   bir anahtar, saldırgana tam kodlayıcınıza sorgu erişimi verir.
6. **İzleme, günlükleme ve olay müdahalesi.** Getirme etkinliğinin değişmez
   günlüklerini tutun (kiracı kapsamı, sorgu, dönen kimlikler, benzerlik skorları).
   Kiracı-filtresi atlatma girişimlerini, kiracılar arası getirme anomalilerini ve
   anormal gömme-API tüketimini izleyin. Olay müdahale playbook'larını, "yalnızca
   gömme" sızıntılarının GDPR 33. Madde ve benzeri rejimlerde ihlal değerlendirmesi
   ve bildirimi açısından kaynak-verisi sızıntısı sayılacağı şekilde güncelleyin.

## Örnek Saldırı Senaryoları

**Senaryo #1 — Halka açık alım boru hattına gömme benzerliği saldırısı:** Bir
şirketin RAG sistemi, halka açık dokümantasyonu ve forum gönderilerini zamanlanmış
biçimde kazır. Saldırgan, gömmeleri "3. çeyrek gelir projeksiyonumuz ne" gibi belirli
iç sorguların yakınına düşecek şekilde mühendislenmiş gönderiler yayımlar. Bir çalışan
o soruyu sorduğunda saldırganın içeriği getirilir ve LLM'e verilir. Aynı metin sohbete
yapıştırılsa hiçbir etkisi olmazdı; saldırı yalnızca saldırganın içeriği gömme
uzayında hedef sorgunun yakınına yerleştirebilmesi sayesinde işler.

**Senaryo #2 — Paylaşımlı vektör indeksinde kiracılar arası çıkarım:** Çok kiracılı
bir SaaS ürünü, kiracı filtrelemesi uygulama katmanında olan tek bir ortak vektör
indeksi kullanır. Kiracı A yoklama sorguları gönderir. Benzerlik araması, filtre
uygulanmadan önce Kiracı B'ninkiler dâhil her gömmede çalışır. A, B'nin belgelerini
hiç görmez; ama zamanlama farkları, sonuç sayıları ve skor-dağılımı boşlukları B'nin
içeriğinin varlığını ve yaklaşık konusunu açığa vurur. Çok sayıda sorguyla A, B'nin
verisinin işe yarar bir haritasını kurar. Bu kategorideki gerçek olaylar arasında,
öngörülebilir token üretiminin yaygın kullanılan açık kaynak bir RAG motorunda
hesaplar arası ele geçirmeye izin verdiği CVE-2025-69286 (RAGFlow) vardır.

**Senaryo #3 — Sızan vektör deposundan gömme inversion'ı:** Bir bulut yanlış
yapılandırması, üretim vektör veritabanının bir yedeğini açığa çıkarır. Alttaki
belgeler — PII içeren müşteri konuşma günlükleri — ayrı şifrelenmiştir ve açığa
çıkmamıştır; olay başta "yalnızca gömmeler sızdı" diye düşük şiddetli sınıflandırılır.
Saldırgan, çalınan gömmelere karşı sıfır-atışlı bir inversion saldırısı çalıştırır ve
özgün kodlayıcıya erişim olmadan kaynak içeriğin — PII dâhil — önemli bir kısmını
yeniden kurar. Olay, kaynak-belge ihlaline eşdeğer diye yeniden sınıflandırılır ve
bildirim yükümlülükleri yeniden değerlendirilir. **"Yalnızca gömme", güvenli liman
sınıflandırması değildir.**

---

*Orijinal: OWASP Top 10 for LLM Applications 2026, OWASP GenAI Security Project,
CC BY-SA 4.0 — https://genai.owasp.org. Bu gayriresmî Türkçe çeviri aynı lisansla
dağıtılır. Kaynakça için orijinal belgenin References bölümüne bakınız.*
