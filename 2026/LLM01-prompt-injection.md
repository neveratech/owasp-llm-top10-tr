# LLM01:2026 Prompt Injection — Komut Enjeksiyonu

## Tanım

Komut enjeksiyonu (prompt injection) zafiyeti, bir büyük dil modeline (LLM) giden
girdinin — ister doğrudan kullanıcı girdisi, ister getirilen içerik, araç çıktısı,
görsel, ses veya video içeriği, ara akıl yürütme ya da kalıcı bellek olsun — modelin
davranışını uygulama geliştiricisinin amaçlamadığı biçimde değiştirmesiyle ortaya çıkar.
LLM'ler "talimat" ile "veri" arasında mimari bir ayrım yapmaz (ikisi de aynı akıştaki
token'lardır); bu yüzden parametreli sorguların temiz bir muadili yoktur (NCSC, 2025).
Girdilerin insan tarafından okunabilir olması, doğrudan bir kullanıcıdan gelmesi ya da
render edilen arayüzde görünür olması gerekmez.

Komut enjeksiyonu zafiyetleri, modellerin girdiyi nasıl işlediğinde ve bu girdinin modeli
sistemin diğer parçalarına veri ya da talimatları yanlış aktarmaya nasıl zorlayabildiğinde
yatar. Dağıtım anındaki üç özellik durumu ağırlaştırır. Birincisi, **bağlam penceresi
havuzlaması**: model; sistem komutunu, kullanıcı girdisini, getirilen belgeleri, araç
çıktılarını, konuşma geçmişini ve belleği, aralarında zorunlu bir güven sınırı olmayan
tek bir token akışı olarak ele alır. İkincisi, **bellek kalıcılığı**: uzun süreli belleğe,
bir RAG derlemine, vektör deposuna veya barındırılan bir bellek servisine yazan bir
enjeksiyon, o depodan okuyan sonraki her oturumu kirletir. Üçüncüsü, **ajan tabanlı
yürütme**: modelin çıktısı araç çağrılarını (dosya sistemi, kabuk, e-posta, bulut
API'leri, MCP sunucuları, alt-ajanlar) tetiklediğinde etki yarıçapı sohbet yüzeyinden
ajanın araçlarının erişebildiği her yere uzanır; üstelik araç çıktıları bağlam
penceresine geri girerek zincirleme veya kendi kendini çoğaltan etkilere olanak tanır.

Bir komut enjeksiyonunun anatomisi üç eksende betimlenebilir. **Teslim yüzeyi**, modele
nasıl ulaştığıdır (doğrudan girdi, getirilen içerik, araç çıktısı, araç bağlantı kanalı
veya kalıcı bellek). **Yayılma davranışı**, zaman ve sınırlar boyunca nasıl yayıldığıdır
(tek atımlık, çok adımlı saldırı zinciri, bellek veya RAG üzerinden oturumlar arası, ya
da ajanlar arasında kendi kendini çoğaltan). **Kodlama**, kötücül talimatların token'larda
veya piksellerde nasıl temsil edildiğidir (düz metin, base64 veya başka bir gizleme,
görünmez Unicode, multimodal veya steganografik, düşük kaynaklı dil). Bir senaryoyu bu
eksenlerde ayrıştırmak, hangi önlemlerin uygulanacağını seçmeden önce yararlı bir tehdit
modelleme adımıdır.

Başarılı bir komut enjeksiyonunun şiddeti ve niteliği, modelin içinde çalıştığı iş
bağlamına ve mimarisine tanınan yetki düzeyine göre değişir. Komut enjeksiyonu, bunlarla
sınırlı olmamak üzere şu sonuçlara yol açabilir:

- Hassas bilgilerin, sistem komutu içeriğinin, getirilen özel belgelerin veya altyapı
  ayrıntılarının ifşası.
- Model çıktısının; aşağı akıştaki sistemlerin veya kullanıcıların üzerine eylem kurduğu
  yanlı, zararlı ya da saldırganın seçtiği içerik üretecek şekilde manipüle edilmesi.
- Ajanın çağırmaya yetkili olduğu araçların yetkisiz çağrılması; ajanın kabuk, dosya
  sistemi veya bulut API erişimi varsa keyfî komut yürütmeye ve yıkıcı eylemlere tırmanma.
- Görsel-URL kanalları, render edilen çıktıdaki gizli Unicode karakterler veya örtülü
  araç-günlükleme yan kanalları üzerinden veri sızdırma (exfiltration).
- Bellek veya RAG derlemi zehirlenmesi yoluyla ajan davranışının oturumlar arası kalıcı
  biçimde ele geçirilmesi.

**Not:** Komut enjeksiyonu; modelin çıktıları üzerinden neyi sızdırdığını (akıl yürütme
kanalı içeriği dâhil) ele alan **LLM02:2026 Hassas Bilgi İfşası**'ndan ve model çıktısının
ayrıcalıklı eylemlere ulaşmasının sonuçlarını ele alan **LLM03:2026 Aşırı Yetki**'den
farklıdır. Bu madde, girdi sınırının kendisiyle ilgilidir. Model çıktılarının aşağı akış
bileşenlerine ulaşmadan önce arındırılması ve doğrulanması **LLM10:2026 Hatalı Çıktı
İşleme** kapsamındadır.

## Komut Enjeksiyonu Türleri

### Doğrudan Komut Enjeksiyonu

Bir kullanıcı — veya kullanıcının erişim yolunu ele geçirmiş bir saldırgan — model
davranışını istenmeyen biçimde değiştiren girdi sağlar. Doğrudan enjeksiyon kasıtlı
olabilir (jailbreak kurgulayan kötücül bir kullanıcı) ya da kasıtsız (çelişen talimatlar
içeren bir içeriği yapıştıran meşru bir kullanıcı; veya Senaryo #3'teki gibi, bir LLM'den
yardım alırken girdisini farkında olmadan ilgisiz bir aşağı akış LLM'ine karşı optimize
eden bir kullanıcı).

Jailbreak, saldırganın amacının modele güvenlik protokollerini çiğnetmek olduğu komut
enjeksiyonu alt kümesidir. Uygulama düzeyindeki korumalar onu dizginlemeye yardım eder;
ancak etkili önleme, modelin eğitim ve güvenlik mekanizmalarının sürekli güncellenmesini
gerektirir.

### Dolaylı Komut Enjeksiyonu

Model, harici bir kaynaktan (web sayfası, belge, e-posta, araç yanıtı, getirilen bir RAG
pasajı, görsel, bir MCP sunucusunun çıktısı, veritabanı satırı veya bir issue başlığı)
komut enjeksiyonu işlevi gören veri içeren içerik alır. Kullanıcı bu talimatları ne
sağlamıştır ne de görmüştür. Teslim yüzeyinin güven profili, hangi savunmaların pratik
olduğunu belirler:

- **Güvenilmeyen yüzeyler.** Halka açık web sayfaları, bilinmeyen göndericilerden
  e-postalar, arama sonuçları. Savunucular bu kaynaklardan gelen her şeyi şüpheli
  saymalıdır. Komut enjeksiyonu araştırmalarının çoğu buraya odaklanmıştır.
- **Yarı güvenilir yüzeyler.** Halka açık bir hata takipçisindeki issue başlıkları, paket
  README ve changelog'ları, üçüncü taraf API yanıtları: kullanıcının getirmeyi seçtiği
  ama yazarı olmadığı içerik. Kullanıcı platforma güvenir; tekil katkıcılara ise
  mutlaka güvenmez.
- **Güvenilir yüzeyler.** Geliştiricinin kendi depoları, veritabanları, iç belgeleri ve
  postası. Geliştirici, bir saldırganın buraya — belki halka açık bir hata bildirim formu
  gibi ilgisiz bir yukarı akış vektörüyle — içerik yerleştirdiğini fark etmeyebilir.

Ortak yapı şudur: saldırganın arka ucu doğrudan ele geçirmesi gerekmez. Metni,
geliştiricinin LLM'inin okuyacağı yere bırakır ve geliştiricinin ayrıcalıklarıyla çalışan
LLM işi onun yerine yapar. Yalnızca sohbet yüzeyine odaklanan savunmalar bunu tümüyle
ıskalar.

Dolaylı komut enjeksiyonu, giderek artan biçimde kullanıcının kendi LLM örneğini
kullanıcının kendi arka ucuna karşı silaha dönüştürüyor. Kalıp şu: saldırgan, düşük
ayrıcalıklı bir kanaldan (halka açık bir form, bir müşteri kaydı, bir topluluk PR'ı)
kullanıcının güvendiği bir konuma metin bırakır ve kullanıcının MCP bağlantılı ajanının
ya da geliştirici asistanının bu metni kullanıcının yükseltilmiş kimlik bilgileriyle
çalışırken okumasını bekler. Ayrıcalıklı eylemi saldırgan değil ajan gerçekleştirir
(bkz. Yaygın Örnek #3; Senaryo #9 üretim ortamındaki kavram kanıtlarını adım adım anlatır).

## Yaygın Risk Örnekleri

1. **Doğrudan komut-girdisi geçersiz kılma:** Bir kullanıcı mesajı, sistem komutunun rol
   ve yetenek sınırlarını geçersiz kılarak modele amaçlanan kapsamı dışında ifşa,
   üretim veya eylem yaptırır. Kasıtlı ve kasıtsız girdilerin ikisi de sayılır.
2. **Getirilen içerik üzerinden dolaylı enjeksiyon:** Saldırgan talimatları bir RAG
   pasajında, web sayfasında, belgede veya e-postada taşınır ve içerik bağlam penceresine
   ulaştığında çalışır (örneğin EmailGPT; INCIBE-CERT, 2024).
3. **Güvenilir yüzeyden dolaylı enjeksiyon:** Düşük ayrıcalıklı ama güvenilen bir kanala
   (issue takipçisi, geri bildirim formu, destek kaydı) ekilen metin, kullanıcının
   LLM'ine kendi yükseltilmiş kimlik bilgileriyle eylem yaptırır: depoları sızdırmak,
   veritabanlarını dökmek veya IDE yapılandırmasını değiştirmek — saldırganın doğrudan
   yapamayacağı eylemler (Invariant Labs, 2025; General Analysis, 2025; Rehberger, 2025a).
4. **Multimodal ve steganografik enjeksiyon:** Görsel, ses veya videodaki algı-altı
   bozulmalar kodlayıcı tarafından çıkarılır (Clusmann vd., 2025; bkz. Senaryo #6).
5. **Görünmez karakter enjeksiyonu ve sızdırma:** Tag-block, variation-selector ve sıfır
   genişlikli Unicode; zararsız görünen metnin içinde talimat taşır veya bayt sızdırır.
   Ağustos 2024'teki M365 Copilot ASCII-smuggling kavram kanıtı bir Slack MFA kodunu
   sızdırmıştı (Rehberger, 2024).
6. **Oturumlar arası bellek ve RAG derlemi zehirlenmesi:** Kalıcı bellekteki veya bir RAG
   derlemindeki tek bir kirli kayıt, onu okuyan gelecekteki her oturuma ulaşır
   (W. Zou vd., 2025; bkz. Senaryo #4).
7. **Gradyan kâhini olarak ince ayar arayüzü ("fun-tuning"):** Saldırgan, bir sağlayıcının
   fine-tuning API'sinden örnek başına kayıp (loss) değerini okuyarak payload'ını optimize
   eder (Gemini'ye karşı %65–82 saldırı başarısı) ve kapalı ağırlıklı modellere beyaz
   kutu tarzı optimizasyon getirir (Labunets vd., 2025).
8. **Çok dilli, kodlanmış veya düşük kaynaklı dil payload'ları:** Düşük kaynaklı ve
   kod-karışımlı girdiler saldırı başarısını yükseltir ve o şemayla eğitilmemiş
   sınıflandırıcıları atlatır; Base64, ROT13 veya emoji kodlamaları, o kodlamayı hiç
   görmemiş filtreleri geçer (Hackett vd., 2025).

## Önleme ve Azaltma Stratejileri

Komut enjeksiyonu günümüz üretken yapay zekâsının doğasında vardır: LLM'ler talimat ile
veri arasında mimari ayrım yapmaz ve davranışları stokastiktir; dolayısıyla bugün
güvenilir bir önleme mekanizması yoktur — NIST (2025), NCSC (2025) ve Debenedetti vd.
(2025) ile tutarlı bir tespit. Savunma bu nedenle araya girme değil mimari meselesidir.
Çevreleyen sistemi, modelin talimat sınırının eninde sonunda aşılacağı açık varsayımıyla
tasarlayın; modelin yapmasına izin verilenleri ve çıktılarının ulaşabileceği yerleri
öyle kısıtlayın ki başarılı bir enjeksiyon başarılı bir istismara dönüşmesin.

Kayıtlardaki yüksek etkili komut enjeksiyonu olaylarının çoğu, enjeksiyonun; araçları,
kapsamları veya çıktı-render yetenekleri ele geçirilen modelin kullanıcının ayrıcalık
düzeyinde saldırgan adına eylem yapmasına izin veren bir sistemin içine düşmesi yüzünden
ağırlaştı (bkz. Senaryolar #7–#9). Bu maddenin **LLM03:2026 Aşırı Yetki** ile
operasyonel ilişkisi budur: komut enjeksiyonu girdi tarafındaki ele geçirmedir; aşırı
işlevsellik, izinler veya özerklik ise bu ele geçirmeye sohbet penceresinin dışında sonuç
kazandıran şeydir. Simon Willison'ın "ölümcül üçlüsü" (lethal trifecta, 2025) aynı
yapısal teşhisi dağıtım öncesi bir kontrol olarak yeniden ifade eder: aynı anda özel
veriye erişebilen, güvenilmeyen içerik alabilen ve dışarıyla iletişim kurabilen bir ajan,
yüksek etkili istismarın koşullarını taşır; üç ayaktan herhangi birini kaldırmak bu
koşulları ortadan kaldırır.

Aşağıdaki kontrolleri derinlemesine savunma olarak uygulayın; tek başına hiçbir kontrol
yeterli değildir. Bazıları enjeksiyon başarısını düşürür ve uyarlanabilir saldırganlar
karşısında zayıflaması beklenir. Diğerleri, enjeksiyon başarılı olduktan sonra etki
yarıçapını sınırlar — sistemi yoklayabilen saldırganlara karşı ayakta kalanlar bunlardır.
Ajan tabanlı dağıtımlarda en az yetki ve yetenek bütçeleme kontrolleri (#4, #8) yük
taşıyıcıdır; yetki tarafının tam işlenişi LLM03:2026'dadır.

1. **Modelin rolünü ve yeteneklerini sistem komutunda kısıtlayın.** Açık uçlu yetkiler
   yerine bildirimsel izin/yasak ifadeleri kullanın ("yalnızca X'e yardım et, Y'ye
   erişme, çıktıyı harici adreslere iletme"). Bu yalnızca kısmi bir kontroldür: komutu
   çıkarsayan bir saldırgan onu atlatabilir (Nasr vd., 2025); #4'teki ayrıcalık
   kontrolleriyle eşleştirin.
2. **Katı bir çıktı şeması tanımlayın ve her yanıtı, aşağı akıştaki herhangi bir sistem
   üzerine eylem kurmadan önce güvenilir uygulama kodunda doğrulayın** — ikinci bir LLM
   çağrısıyla değil, yapısal doğrulamayla. Bu, biçim ihlallerini yakalar, anlamsal
   manipülasyonu değil: şemaya uygun bir yanıt yine de kötücül bir SQL sorgusu veya
   sızdırma biçiminde bir e-posta gövdesi taşıyabilir.
3. **Her modalite sınırında filtreleyin** (metin, görsel, ses, yapılandırılmış veri) —
   yalnızca metinde değil. Modaliteye özgü sınıflandırıcılar çalıştırın, görsellere OCR,
   sese transkripsiyon uygulayın; sonra çıkarılan içeriğe metin filtrelerini uygulayın.
   Anlamsal filtreler yeniden ifade veya kodlamayla atlatılabilir; düşük kaynaklı ve
   kod-karışımlı girdiler doğruluklarını düşürür (Hackett vd., 2025).
4. **Kimlik bilgilerini ve durum değiştirme yeteneğini modelde değil uygulama kodunda
   tutun; işlem başına en az yetkiyi verin.** Ayrıcalıklı çağrıları, yürütme anında niyeti
   ve argümanları yeniden doğrulayan deterministik bir politika motorundan geçirin. NIST
   AI 100-2 E2025 ile CISA ve Five Eyes OT ortak rehberi (CISA vd., 2025) bu deterministik
   aracılığı temel bir tedarik beklentisi olarak çerçeveler. Geniş "kolaylık" izinleri ve
   çok-ajanlı sıçramalar riski aşağı akışta yeniden üretir.
5. **Her alım ve render sınırında tag-block (U+E0000–E007F), variation-selector
   (U+FE00–FE0F) ve sıfır genişlikli (U+200B, U+200C, U+200D, U+2060) karakterleri
   ayıklayın.** Bunlar normal render'da görünmezdir ve talimat ya da sızdırma baytları
   kaçırır (bkz. Yaygın Örnek #5); variation-selector varyantları (Rehberger, 2025c)
   keyfî baytları görünmez biçimde taşır. Ayıklama; görünür metin payload'larını veya
   gelecekteki steganografik sınıfları durdurmaz.
6. **Harici içeriği, modelin veriyi talimattan ayırt edebilmesi için yapısal olarak ayrı,
   köken etiketli bir kanaldan geçirin** (S. Chen vd., 2025; Microsoft Research, 2025).
   Bu yalnızca uyarlanabilir olmayan testlerde saldırı başarısını düşürür: işaretleme
   şemasını bilen bir saldırgan onu taklit edebilir; StruQ uyarlanabilir saldırı altında
   atlatılmıştır (Nasr vd., 2025).
7. **Ayrıcalıklı, geri döndürülemez veya dışarıdan görünür her eylemden önce açık insan
   onayı isteyin** ve gözden geçirene özet değil, render edilmiş eylemin tam hâlini
   gösterin. Görünmez karakter kaçakçılığı, gösterilen eylemi yürütülenden
   farklılaştırabilir (#5); onay yorgunluğu ise hacim arttıkça değerlendirme kalitesini
   düşürür.
8. **Ajan yeteneklerini, taban olarak İkili Kuralı (Rule of Two) ile bütçeleyin** (Meta
   AI, 2025). (A) güvenilmeyen girdi, (B) hassas veri ve (C) durum değişikliği veya dış
   iletişime eşzamanlı erişimi yüksek riskli sayın: her [A,B,C] ajanı eylem başına insan
   onayı gerektirir; [A,B] veya [A,C] yapılandırmaları açık bir artık-risk değerlendirmesi
   ister (bkz. Senaryo #8). NIST AI 100-2 E2025 ile CISA, FBI, NSA ve ACSC OT rehberi
   (CISA vd., 2025) kuralı destekler; kural özerklik derinliği konusunda sessizdir (Noma
   Security, 2025).
9. **Ajan bellek yazımlarını ayrıcalıklı işlem olarak ele alın.** Yazıma neden olan
   komutu günlükleyin, yazımları talimat veya rol değiştirme içeriği açısından
   sınıflandırın ve talimat taşıyan bellekler oturumlar arası kalıcı olmadan önce onay
   isteyin. Şubat 2025 tarihli bir Gemini kavram kanıtı (Rehberger, 2025b) belleği
   gecikmeli araç çağrısıyla zehirledi (MITRE, t.y.). Olgusal kayıtlar talimata evrilir
   ve artımlı yazımlar yazım-başına sınıflandırmayı atlatabilir.
10. **Her MCP sunucusunu ve üçüncü taraf araç paketini sabitleyin, imzalayın ve
    doğrulayın**; araç açıklamalarını gizli talimatlar için denetleyin ve araç bileşimini
    izleyin. Bunları bir yazılım tedarik zinciri yüzeyi olarak ele alın: üçüncü taraf
    araç paketleri LLM04:2026 Tedarik Zinciri, MCP sunucuları ve araç kayıtları ASI04
    kapsamındadır (bkz. Senaryo #9). Sabitleme; sabitlenen sürümde gelen bir payload'ı
    veya sürümü değiştirmeyen araç-açıklaması zehirlenmesini durdurmaz.
11. **Dağıtılan savunmayı okumuş uyarlanabilir saldırganlara karşı test edin ve yalnızca
    statik saldırı-başarı iddialarını reddedin.** AgentDojo (Debenedetti vd., 2024) ve
    JailbreakBench (Chao vd., 2024) ile taban çizgisi kurun; ardından tam savunma
    spesifikasyonu test ekibine açıklanmış hâlde red team çalışması yapın. Nasr vd.
    (2025), incelenen 12 güncel savunmanın çoğunda statik saldırı başarısını sıfıra yakın,
    uyarlanabilir saldırı başarısını %90'ın üzerinde buldu (ayrıca bkz. LLMail-Inject,
    Microsoft Security Response Center, 2025).

## Örnek Saldırı Senaryoları

**Senaryo #1 — Doğrudan Enjeksiyon:** Saldırgan, bir müşteri destek sohbet botuna
yönergelerini yok saymasını, özel veri depolarını sorgulamasını ve e-posta göndermesini
söyler; sonuç yetkisiz erişim ve ayrıcalık yükseltmedir. *Anatomi: (a) doğrudan kullanıcı
girdisi, (b) tek atımlık, (c) düz metin.*

**Senaryo #2 — Getirilen Web İçeriğiyle Dolaylı Enjeksiyon:** Kullanıcı, gizli talimatlar
içeren bir web sayfasını özetletmek ister. Model, URL'si özel konuşmayı saldırganın
kontrolündeki bir alan adına sızdıran bir markdown görseli ekler. Kullanıcı yalnızca
render edilen görseli görür; talimatı asla görmez. *Anatomi: (a) getirilen web içeriği
(dolaylı), (b) görsel-URL sızdırmalı tek atımlık, (c) sayfa kaynağında gizli düz metin.*

**Senaryo #3 — Kasıtsız Enjeksiyon:** Bir iş ilanı PDF'ine yapay zekâ tespiti talimatı
gömülüdür. Aday, özgeçmişini farkında olmadan bir LLM ile buna karşı optimize eder; model
talimatı yüzeye çıkarır ve işe alım sistemi adayı işaretler — iki tarafın da kötücül
davranmadığı bir komut enjeksiyonu. *Anatomi: (a) dolaylı (belge/PDF), (b) tek atımlık,
(c) düz metin.*

**Senaryo #4 — RAG Deposu Zehirlenmesi:** Saldırgan, uygulamanın üzerinde arama yaptığı
derleme zehirli belgeler katar. Eşleşen bir sorgu değiştirilmiş içeriği döndürür ve
içindeki talimatlar çıktıyı değiştirir. Milyonlarca metinlik bir bilgi tabanına karşı beş
kadar az zehirli belge yaklaşık %90 saldırı başarısına ulaşmıştır (W. Zou vd., 2025).
*Anatomi: (a) getirilen içerik (RAG derlemi), (b) oturumlar/kullanıcılar arası, (c) düz
metin.*

**Senaryo #5 — Payload Bölme:** Saldırgan, kötücül talimatları birden çok özgeçmiş
alanına (başlık, gövde, ek) böler; böylece alan-başına sınıflandırıcıya hiçbir alan tek
başına kötücül görünmez. LLM bunları değerlendirme sırasında birleştirir ve önerisi
manipüle edilir. *Anatomi: (a) alanlara bölünmüş doğrudan girdi, (b) değerlendirmede
birleşen tek atımlık, (c) parçalanmış düz metin.*

**Senaryo #6 — Multimodal Steganografik Enjeksiyon:** Saldırgan, talimatı insan görsel
eşiğinin altında bir görsele gömer. Multimodal modelin görü kodlayıcısı payload'ı çıkarır,
davranış değişir ve model zararlı çıktı veya yetkisiz araç çağrısı üretir. Bu, onkoloji
görüntülemede dört sınır görsel-dil modeline karşı (Clusmann vd., 2025) ve görsel bozulma
ile metin yönlendirmesinin birleşimiyle genel amaçlı modellere karşı (R. Chen vd., 2025)
gösterilmiştir. *Anatomi: (a) görsel girdi (dolaylı/multimodal), (b) tek atımlık,
(c) steganografik/piksel düzeyi kodlama.*

**Senaryo #7 — Sıfır Tık, Belge Kaynaklı Ajan Sızdırması:** Özel hazırlanmış bir e-posta,
LLM destekli bir üretkenlik asistanının hiçbir kullanıcı etkileşimi olmadan kurumsal veri
sızdırmasını tetikler. Aim Security bunu Microsoft 365 Copilot'a karşı göstermiş (Reddy
& Gujral, 2025), dağıtımdaki komut-enjeksiyonu sınıflandırıcısını da bağlantı-karartma
filtresini de atlatmıştır. *Anatomi: (a) e-posta/belge (dolaylı), (b) araç çağrılı tek
atımlık, (c) görünmez-Unicode sızdırma kanallı düz metin.*

**Senaryo #8 — Ajan Yoluyla Yıkıcı Komut Yürütme:** Temmuz 2025'teki iki olay, aynı etki
sınıfını farklı vektörlerle gösterir. Bir saldırgan, AWS geri almadan önce Amazon Q VS
Code eklentisi deposuna yıkıcı bir sistem komutu commit'ledi; commit'lenen kod bir söz
dizimi hatası nedeniyle çalışmadı (Amazon Web Services, 2025a). Ayrıca bir çalışma
zamanı enjeksiyonu Amazon Q'ya keyfî kod yürüttü (Amazon Web Services, 2025b). Kabuk,
dosya sistemi veya bulut API erişimi olan bir ajan, enjeksiyonu ana makineyi etkileyen
bir olaya büyütür. *Anatomi: (a) tedarik zinciri / ele geçirilmiş sistem komutu veya
çalışma zamanı dolaylı enjeksiyonu, (b) kalıcı oturumlar-arası veya kabuk araçlı tek
atımlık, (c) düz metin.*

**Senaryo #9 — MCP Üzerinden Güvenilir Arka Uca Dolaylı Enjeksiyon:** Saldırgan, düşük
ayrıcalıklı bir kanala (halka açık bir GitHub issue'su, bir destek kaydı veya kötücül bir
npm paketi) metin bırakır ve geliştiricinin LLM'i bunu yükseltilmiş kimlik bilgileriyle
okur. Invariant Labs (2025) zehirli bir GitHub issue'suyla özel depoları sızdırdı; General
Analysis (2025), Cursor'ın service_role ile çalışan ve satır düzeyi güvenliği atlayan
Supabase MCP sunucusu üzerinden bir üretim veritabanını döktü; kötücül postmark-mcp paketi
(Koi Security, 2025; Toulas, 2025) tahminen 300 kuruluşta e-postaları saldırgana BCC'ledi.
*Anatomi: (a) güvenilir yüzeyden dolaylı (MCP kanalı: issue, kayıt, npm paketi), (b) çok
adımlı araç zinciri, (c) düz metin.*

---

*Orijinal: OWASP Top 10 for LLM Applications 2026, OWASP GenAI Security Project,
CC BY-SA 4.0 — https://genai.owasp.org. Bu gayriresmî Türkçe çeviri aynı lisansla
dağıtılır. Kaynakça için orijinal belgenin References bölümüne bakınız.*
