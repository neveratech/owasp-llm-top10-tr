# LLM02:2026 Sensitive Information Disclosure — Hassas Bilgi İfşası

## Tanım

Hassas bilgi ifşası; LLM entegre bir sistemin gizli, düzenlemeye tabi, ayrıcalıklı veya
mülkiyete konu veriyi, veri sahibinin, veri sorumlusunun veya sistem sahibinin
yetkilendirmediği bir kanaldan açığa çıkarmasıyla oluşur. Kanal yalnızca nihai yanıt
değildir: araç çağrısı argümanları, akıl yürütme izleri, getirilen parçalar (chunk),
multimodal çıktı, günlükler, telemetri, gömmeler (embedding) ve gözlemlenebilir çıkarım
özellikleri (zamanlama, token uzunluğu, log-olasılıkları, güven skoru, önbellek-isabet
davranışı) — hepsi birer ifşa yüzeyidir. Her birini, aynı sınıflandırma ve karartma
kurallarına tabi bir çıktı olarak ele alın.

İfşa, LLM yaşam döngüsünün dört evresinde ortaya çıkar:

1. **Eğitim zamanı.** Bir model, fine-tune veya LoRA adaptörü derlem içeriğini
   ezberler ve sonradan birebir ya da geri kazanılabilir biçimde yeniden üretir.
   Ezberleme; kapasite, tekrar (duplication) ve bağlam uzunluğuyla log-doğrusal ölçeklenir.
   Dar adaptörler nadir örnekleri yüksek sadakatle ezberler — taban modelden ayrı,
   hedefli bir çıkarma (extraction) yüzeyi.
2. **Çıkarım zamanı.** Model canlı bağlamı ifşa eder (sistem komutu, RAG parçaları,
   dosyalar, araç çıktıları, bellek veya başka bir oturumun verisi); çoğu zaman
   özetleme, çeviri veya çıkarma işlemleri istenenden fazlasını — görsel olarak
   karartılmış bölümler dâhil — yüzeye çıkardığı için.
3. **Boru hattı zamanı.** Fine-tuning, damıtma (distillation), sentetik veri üretimi,
   gradyanlar, SDK'lar ve gözlemlenebilirlik araçları hassas veriyi türev artefaktlara
   taşır.
4. **Gözlem zamanı.** Saldırganlar, içerik almadan, dışarıdan ölçülebilir özelliklerden
   (TLS altında token uzunluğu, gecikme, log-olasılıkları, güven skoru, önbellek-isabet
   sinyalleri) olgular çıkarsar.

Korunan bilgi; kişisel veriler (PII), sağlık verileri (PHI), finansal veriler, kimlik
bilgileri, API anahtarları, ticari sırlar, model ağırlıkları, ayrıcalıklı yazışmalar,
gizlilik dereceli veya ihracat kontrolüne tabi malzeme ile biyometrik ve genomik
tanımlayıcıları kapsar. Olayların çoğunu iki yapısal başarısızlık sürükler. Birincisi,
**yukarı akışta aşırı paylaşım**: kapsamı daraltılmamış sürücüler, miras kalan izinler
ve bilgi tabanları RAG'i hassas veriyle besler; model de bu veriyi tasarlandığı gibi
getirir. Çözüm modelde değil veri yüzeyindedir (DSGAI01). İkincisi, **kalıcılık**: veri
bir kez ağırlıkları, gömmeleri veya adaptörleri etkiledi mi, kaynak silindikten sonra da
çıkarılabilir kalır — GDPR 17. Madde ve CCPA §1798.105 silme yükümlülüklerini zorlar.
Açık ağırlıklı dağıtımlar hız sınırlarına bel bağlayamaz; çünkü çıkarma, üyelik çıkarımı
(membership inference) ve tersine çevirme (inversion) çevrimdışı, sınırsız hızda çalışır.

Şiddet, sızıntının doğal dile benzeyip benzemediğine değil, **alıcının ne
öğrenebildiğine** göre belirlenmelidir. Uygulanabilir rejimler arasında AB Yapay Zekâ
Yasası (Regulation (EU) 2024/1689 — yüksek riskli yükümlülükler Ağustos 2026'dan
itibaren), GDPR, HIPAA, CCPA/CPRA, ISO/IEC 42001 ve NIST AI 600-1 bulunur. Bu madde
LLM'i bir uygulama bileşeni olarak kapsar. Özerk bir aktör gibi davrandığı yerlerde
(oturumlar arası bellek, araç seçimi, çok adımlı sızdırma) büyütülmüş risk ASI'ye
aittir; daha derin kontroller DSGAI'dedir.

## Yaygın Risk Örnekleri

1. **Eğitim verisi ezberleme ve çıkarma.** Kasım 2023'teki "şiir" saçılma (divergence)
   saldırısı, yaklaşık 200 USD karşılığında gpt-3.5-turbo'ya 10.000'den fazla benzersiz
   ezberlenmiş örnek yayınlattı (Nasr vd., 2023). Sağlayıcı yamaları defalarca
   atlatıldı. Fine-tune edilmiş modeller ve LoRA adaptörleri, aynı ölçekteki taban
   modellerden daha kolay çıkarılabilir — taban modelden ayrı, hedefli bir çıkarma
   yüzeyi. NYT v. OpenAI ve Getty v. Stability AI davalarındaki deliller birebir
   yeniden üretim örnekleri içerir; Getty davası filigran yeniden üretimi üzerine
   kuruludur.
2. **Çıkarım zamanı bağlam ve çıktı ifşası.** Mart 2023'teki ChatGPT Redis hatası,
   Plus abonelerinin %1,2'sinin ödeme PII'sini açığa çıkardı. 2025'te eksik noindex
   yönergeleri yüzünden 4.500'den fazla paylaşılan konuşma Google tarafından
   indekslendi. Klinik gömme vektör depoları, çoğu ekibin operasyonelleştirmediği
   HIPAA denetim-kontrol kapsamındadır ve getirme katmanı yetkilendirmesi yaygın
   biçimde eksik uygulanır. Akıl yürütme izlerini ve araç argümanlarını hata ayıklama
   kalıntısı değil, çıktı olarak ele alın. İz kanalı ayrıca akıl-yürütme-izi zorlamalı
   model çıkarmayı da mümkün kılar. Regex ve engel listesi filtreleri; diller arası,
   base64 ve hex kodlamalara yenilir. Tek tek izinli kaynakların birleştirilmesi
   (bütçe + işe alım + durum tespiti → bekleyen bir birleşme-satın alma hedefi),
   politika sentezlenmiş sonucu yasaklıyorsa bir ifşadır.
3. **Gömme ve temsil ifşası.** Modern inversion, sızan veya dışa aktarılan
   vektörlerden düz metni yeniden kurar; dolayısıyla "yalnızca gömme" içeren bir yedek,
   kaynak-belge ihlalidir. Kosinüs benzerliği ACL tanımaz. **Getirmeden önce
   yetkilendirin**; çünkü üretim sonrası filtreleme, modele çoktan verilmiş bir parçayı
   geri alamaz. Mekanizmalar LLM09:2026 ve DSGAI13'ün konusudur; bu madde düzenleyici
   sonucun sahibidir.
4. **Multimodal ifşa.** Görü modelleri; ekran görüntülerinden, bildirimlerden ve PDF
   üstverisinden kimlik bilgilerini ve PII'yi OCR ile okur. Üreticiler filigranları ve
   tanınabilir yüzleri yeniden üretir (Getty / Stable Diffusion filigran davası bunun
   klasik örneğidir). Çapraz-modal dönüşüm (metnin görsele render edilmesi, görselin
   OCR ile metne çevrilmesi) tek modaliteli DLP'yi atlatır.
5. **Çıkarım zamanı yan kanallar.** SPV-MIA, fine-tune edilmiş hedeflere karşı üyelik
   çıkarımı AUC'sini 0,9'a çıkardı (Fu vd., 2024) — ismi belirli bir birey hakkında
   düzenleyici ihlal tespitine yetecek düzey. Whisper Leak (McDonald & Bar Or, 2025),
   şifreli trafikten 28 üretim modelinde konuşma konularını %98'in üzerinde AUPRC ile
   sınıflandırdı. Weiss vd. (2024) token uzunluğundan yanıt içeriğinin %29'unu yeniden
   kurdu ve %55'inde konuyu çıkarsadı. Wu vd. (2025) çok kiracılı sunumda KV-önbellek
   paylaşımı üzerinden komut sızıntısı gösterdi. Dong vd. (2025) 4.112 token'lık bir
   tıbbi komutu bir ara katmandan 0,8688 F1 ile (token eşleşmesi) tersine çevirdi.
   Carlini vd. (2024) bir üretim modelinin projeksiyon katmanını logit-bias kanalından
   geri kazandı.
6. **Eğitim boru hattı ifşası.** Kötücül sunucunun gradyan inversion'ı (Boenisch vd.,
   2021), damıtma ve sentetik veri taşınması örnekleri türev modellere aktarır.
   DP korumalı bir modele karşı yinelemeli sorgu saldırıları, LLM üretimi sorguları
   inceltip güven sıçramalarına odaklanarak bireyleri yeniden kimliklendirir;
   dolayısıyla sabit-epsilon DP gereklidir ama hız sınırlama, sorgu-deseni tespiti ve
   kullanıcı başına bütçeler olmadan yeterli değildir.
7. **Platform ve ekosistem ifşası.** Gözlemlenebilirlik platformları (Langfuse,
   LangSmith, Datadog LLM Observability) varsayılan olarak tam komutları,
   tamamlamaları, parçaları ve izleri günlükler. İki temsilî olay: DeepSeek'in Ocak
   2025'te bir milyondan fazla satır günlük ve API anahtarını açığa çıkaran ClickHouse
   ifşası (Wiz, 2025) ve Check Point'in 2026'da açıkladığı, kod-yürütme çalışma
   zamanındaki gizli bir dış kanal üzerinden ChatGPT sızdırması — tek bir kurgulanmış
   komut, görünür yanıt masum kalırken çalışma zamanını sessiz bir DNS kanalına
   çevirdi (Check Point Research, 2026).

## Önleme ve Azaltma Stratejileri

Önlemler, kademeli bir uygulama yolu için DSGAI'nin katmanlı yapısını izler.

### Katman 1: Temel (her dağıtım)

1. **Derlemleri yönetin:** köken, sınıflandırma ve yakın-kopyalar, transliterasyonlar
   ile biçim varyantları genelinde tekilleştirme. PII'yi alım anında temizleyin.
   Tekilleştirme ezberlemeyi azaltır ama ortadan kaldırmaz.
2. **Bağlamı asgariye indirin:** harici sağlayıcılara yalnızca görev için gerekli
   alanları gönderin. Otomatik bağlamı (customer_360, tam kayıt ekleme) şablon başına
   gerekçelendirilmedikçe kapatın.
3. **Getirmeden önce yetkilendirin:** belge ve parça düzeyi yetkilendirmeyi, getirme
   sonrasında uygulama katmanında değil, indeks sorgusunun içinde uygulayın. Yüksek
   hassasiyetli iş yükleri için kiracı başına indeksleri yalıtın.
4. **Sistem komutu hijyeni:** sistem komutlarında asla sır, kimlik bilgisi veya
   düzenlemeye tabi veri tutmayın.
5. **Yalnız regex ile değil, sınıflandırıcılarla arındırın:** desen eşleme + NER +
   eğitilmiş sınıflandırıcılar; çünkü regex, kodlanmış ve diller arası çıktıda başarısız
   olur.
6. **Hassas uç noktalarda kullanıcı ve oturum başına sorgu bütçesi koyun** —
   numaralandırma ve üyelik yoklamasını bozmak için.
7. **Operasyonel hijyen:** günlükleri ve izleri APM alımından önce kısıtlayıp
   temizleyin, aktarımda ve beklemede şifreleyin, no-train/no-retain'i yalnızca
   sözleşme metniyle değil teknik olarak da uygulayın.

### Katman 2: Sıkılaştırma (düzenlemeye tabi / yüksek hassasiyet)

1. **Hassasiyete ve kardinaliteye göre kalibre edilmiş DP-SGD**; aşırı öğrenme,
   ezberleme vekili olarak izlensin. Tespitle eşleştirin; çünkü sabit bütçeler
   uyarlanabilir sorgulama altında aşınır.
2. **Vektör deposu koruması:** şifreleme, belge ACL'lerinden ayrı ACL'ler, kısıtlı
   dışa aktarma API'leri, asgari kapsamlı k-NN ve gömme-uzayı yoklama tespiti.
3. **Üretim uç noktalarında log-olasılıklarını, güven skorlarını ve açıklamaları
   kapılayın.**
4. **Akıl yürütme izlerini birinci sınıf çıktı olarak sınıflandırıp karartın.** Ham
   izleri asla kısıtsız gözlemlenebilirliğe günlüklemeyin.
5. **Yan kanal savunmaları:** akış için rastgele dolgu ve token gruplama, yüksek
   hassasiyetli kiracıların özel önek önbelleklerinde ayrıştırılması ve ortak
   kiracılıkta bölümlenmiş KV önbellekleri.
6. **Yapılandırılmış tanımlayıcılar için biçim koruyan şifreleme**; katı iç/dış
   yönlendirme ayrımı ve dış yolda alan izin listeleriyle.
7. **SIEM'e akan AI-farkındalıklı denetim günlüğü, sürekli DLP ve AI-SPM** ve tek tek
   izinli kaynakların yasak sonuçlarda birleşememesi için zorunlu birleştirme (join)
   politikalı, belgelenmiş bir alan envanteri.

### Katman 3: İleri (düzenlemeye tabi, gizlilik dereceli, yüksek hedef)

1. **Gizli hesaplama** (Intel TDX, AMD SEV-SNP, AWS Nitro Enclaves) veya tehdit modeli
   fayda ve gecikme maliyetini haklı çıkarıyorsa gelişmekte olan gizlilik koruyucu
   çıkarım (AloePri kovaryant-gizleme çerçevesi, Lin vd., 2026).
2. **Doğrulanabilir silme** — ham veri, gömmeler, kontrol noktaları ve adaptörler
   genelinde; unlearning sonrası çıkarma ve üyelik-çıkarımı yoklamalarıyla
   doğrulanmış.
3. **Sürüm kapısı olarak ifşa red team'i:** çıkarma, üyelik çıkarımı, gömme
   inversion'ı, iç-durum inversion'ı, yan kanallar ve LoRA çıkarılabilirliği — nicel
   ölçülmüş ve MITRE ATLAS'a hizalanmış.
4. **Sentetik veriyi çıkarıcılara karşı denetleyin.** Damıtmaya; yoklama tespiti, hız
   sınırları ve filigranla direnin. Toplu analitiği bütçeleyin.
5. **Bir ifşa olay müdahale playbook'u tatbik edin:** veri sınıfına ve etkilenen özneye
   veya oturuma göre kapsam belirleyin; ardından ilgili rejimlerdeki (örneğin GDPR,
   HIPAA ve AB Yapay Zekâ Yasası 73. Madde) ihlal ve ciddi-olay bildirim
   yükümlülüklerini değerlendirip yerine getirin. Devamında unlearning, yeniden eğitim
   veya geri çekme; vektör ve önbellek temizliği; tedarikçi bildirimi ve kalıcı-bellek
   denetimi.

## Örnek Saldırı Senaryoları

**Senaryo #1:** Saçılma komutları, bir üretim modeline ezberlenmiş PII'yi, URL'leri ve
canlı kimlik bilgilerini ölçekli yayınlatır; GDPR 33. Madde bildirimi tetiklenir.

**Senaryo #2:** Paylaşılan çıkarım durumu kusuru, bir kullanıcının tıbbi-mektup
komutunu başka bir kullanıcının akıl yürütme izine sızdırır. HIPAA 60 günlük bildirim
uygulanır.

**Senaryo #3:** Ortak bir APM projesine birebir günlüklenen genişletilmiş-düşünme
izleri, yanıt temiz kalırken getirilen PII'yi yüzlerce mühendise açar.

**Senaryo #4:** Komut enjeksiyonu, bir destek botuna sistem komutunu ve içine gömülü
bir tedarikçi API anahtarını yazdırır.

**Senaryo #5:** Paylaşılan bir hukuk RAG indeksi firma sınırlarını aşar; bir müvekkilin
ayrıcalıklı stratejisini başka bir müvekkilin yanıtına sentezler — avukat-müvekkil
gizliliğinden feragat doğuran bir olay.

**Senaryo #6:** Sızan "yalnızca gömme" içeren bir vektör yedeği, inversion sonrası
kaynak-belge ihlali olarak yeniden sınıflandırılır ve 72 saatlik bildirim saati yeniden
başlar.

**Senaryo #7:** Şifreli akışta Whisper Leak konu çıkarımı, şifre çözmeden tıbbi,
hukuki veya siyasi konular sorgulayan kullanıcıları tespit eder.

**Senaryo #8:** Klinik bir fine-tune'a karşı üyelik çıkarımı, hiçbir kayıt çıkarmadan
eğitim setindeki hastaları yüksek AUC ile tespit eder — HIPAA kapsamında raporlanabilir
bir belirleme.

**Senaryo #9:** Model, değiştirilmemiş metnin üzerine render edilmiş siyah dikdörtgen
PDF karartma katmanının altındaki PII'yi özetler.

**Senaryo #10:** Enjekte edilen bir "tanılama kontrolü", görünür istatistiksel özet
masum kalırken kod çalışma zamanına tablo içeriğini DNS sorgularına kodlatır.

---

*Orijinal: OWASP Top 10 for LLM Applications 2026, OWASP GenAI Security Project,
CC BY-SA 4.0 — https://genai.owasp.org. Bu gayriresmî Türkçe çeviri aynı lisansla
dağıtılır. Kaynakça için orijinal belgenin References bölümüne bakınız.*
