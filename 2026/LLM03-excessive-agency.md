# LLM03:2026 Excessive Agency — Aşırı Yetki

## Tanım

LLM tabanlı bir sisteme geliştiricisi çoğu zaman bir ölçüde yetki (agency) tanır: bir
komuta yanıt olarak eylem gerçekleştirmek üzere araçlar (farklı sağlayıcılarca
extension, plugin veya skill de denir) aracılığıyla fonksiyon çağırma ya da başka
sistemlerle arayüz kurma yeteneği. Bir LLM ajanı, hangi aracı çağıracağını girdi
komutuna veya önceki LLM çıktısına dayanarak dinamik olarak da seçebilir. Ajan tabanlı
sistemler tipik olarak LLM'e tekrarlı çağrılar yapar; önceki çağrıların çıktısını,
sonraki çağrıları temellendirmek ve yönlendirmek için kullanır.

Aşırı Yetki; LLM'in beklenmedik, muğlak veya manipüle edilmiş çıktılarına yanıt olarak
zarar verici eylemlerin gerçekleştirilmesine olanak tanıyan zafiyettir — LLM'in neden
hatalı çalıştığından bağımsız olarak. Yaygın tetikleyiciler:

- kötü tasarlanmış masum komutların ya da düpedüz düşük performanslı/hizasız bir
  modelin yol açtığı halüsinasyon/konfabülasyon,
- kötücül bir kullanıcıdan, kötücül/ele geçirilmiş bir aracın önceki çağrısından veya
  (çok ajanlı/işbirlikçi sistemlerde) kötücül/ele geçirilmiş bir eş ajandan gelen
  doğrudan/dolaylı komut enjeksiyonu.

Aşırı Yetki'nin kök nedeni tipik olarak şunlardan biri veya birkaçıdır:

- **aşırı işlevsellik**,
- **aşırı izinler**,
- **aşırı özerklik**.

Aşırı Yetki; gizlilik, bütünlük ve erişilebilirlik yelpazesinin tamamında geniş bir
etki aralığına yol açabilir ve LLM tabanlı uygulamanın hangi sistemlerle etkileşime
girebildiğine bağlıdır. Ajan tabanlı sistemler bağlamında Aşırı Yetki; ASI02: Tool
Misuse & Exploitation, ASI03: Identity & Privilege Abuse ve ASI08: Cascading Failures
olarak tezahür edebilir.

**Not:** Aşırı Yetki, LLM çıktılarının yetersiz denetlenmesiyle ilgilenen Hatalı Çıktı
İşleme'den farklıdır. Model girdi ve çıktılarının arındırılması Aşırı Yetki için kök
kontrol değildir; girdiler LLM01:2026 Komut Enjeksiyonu, çıktılar LLM10:2026 Hatalı
Çıktı İşleme kapsamındadır.

## Yaygın Risk Örnekleri

1. **Aşırı işlevsellik.** LLM ajanının erişimindeki araçlar, sistemin amaçlanan
   işleyişi için gerekmeyen fonksiyonlar içerir. Örneğin geliştiricinin, ajana bir
   depodan belge okuma yeteneği vermesi gerekir; ancak seçtiği üçüncü taraf araç,
   belgeleri değiştirme ve silme yeteneğini de barındırır.
2. **Aşırı işlevsellik.** Bir araç geliştirme evresinde denenmiş, daha iyi bir
   alternatif lehine bırakılmıştır; ama özgün araç LLM ajanının erişiminde kalmaya
   devam eder.
3. **Aşırı işlevsellik.** Açık uçlu işlevselliğe sahip bir LLM aracı, girdi
   talimatlarını uygulamanın amaçlanan işleyişi için gerekenin dışındaki komutlara
   karşı düzgün filtrelemez. Örneğin belirli tek bir kabuk komutunu çalıştırmak için
   yazılmış bir araç, diğer kabuk komutlarının yürütülmesini gerektiği gibi
   engelleyemez.
4. **Aşırı izinler.** Bir LLM aracı, aşağı akış sistemlerinde uygulamanın amaçlanan
   işleyişi için gerekmeyen izinlere sahiptir. Örneğin veri okumak için tasarlanmış bir
   araç, veritabanı sunucusuna yalnızca SELECT değil, UPDATE, INSERT ve DELETE
   izinleri de olan bir kimlikle bağlanır.
5. **Aşırı izinler.** Tekil kullanıcı bağlamında işlem yapmak üzere tasarlanmış bir
   LLM aracı, aşağı akış sistemlerine genel, yüksek ayrıcalıklı bir kimlikle erişir.
   Örneğin geçerli kullanıcının belge deposunu okuyacak bir araç, depoya tüm
   kullanıcıların dosyalarına erişimi olan ayrıcalıklı bir hesapla bağlanır.
6. **Aşırı özerklik.** LLM tabanlı bir uygulama veya araç, yüksek etkili eylemleri
   bağımsızca doğrulayıp onaylamaz. Örneğin kullanıcının belgelerinin silinmesine
   olanak tanıyan bir araç, silmeleri kullanıcıdan hiçbir onay almadan gerçekleştirir.

## Önleme ve Azaltma Stratejileri

Aşağıdaki eylemler Aşırı Yetki'yi önleyebilir:

1. **Araçları asgariye indirin.** LLM ajanlarının çağırmasına izin verilen araçları,
   gerekli olan asgariyle sınırlayın. Örneğin LLM tabanlı sistem bir URL'nin içeriğini
   getirme yeteneğine ihtiyaç duymuyorsa, böyle bir araç ajana sunulmamalıdır.
2. **Araç işlevselliğini asgariye indirin.** LLM araçlarında uygulanan fonksiyonları
   gerekli asgariyle sınırlayın. Örneğin e-postaları özetlemek için kullanıcının posta
   kutusuna erişen bir araca yalnızca okuma yeteneği yetebilir; araç, mesaj silme veya
   gönderme gibi başka işlevler içermemelidir.
3. **Açık uçlu araçlardan kaçının.** Mümkün olduğunda açık uçlu araçlar (kabuk komutu
   çalıştır, URL getir vb.) yerine daha ayrıntı düzeyli (granular) işlevselliğe sahip
   araçlar kullanın. Örneğin LLM tabanlı bir uygulamanın bir dosyaya çıktı yazması
   gerekebilir. Bu, kabuk fonksiyonu çalıştıran bir araçla yapılırsa istenmeyen
   eylemlerin kapsamı çok büyür (başka herhangi bir kabuk komutu da çalıştırılabilir).
   Daha güvenli alternatif, yalnızca o işlevi uygulayan özel bir dosya-yazma aracı
   inşa etmektir. Araçlar tüm girdi parametreleri için katı bir şema tanımlamalı ve
   içerikleri kullanmadan önce doğrulamalıdır.
4. **Araç izinlerini asgariye indirin.** İstenmeyen eylemlerin kapsamını sınırlamak
   için LLM araçlarına diğer sistemlerde tanınan izinleri gerekli asgariyle sınırlayın.
   Örneğin müşteriye satın alma önerisi yapmak için ürün veritabanı kullanan bir LLM
   ajanının yalnızca 'products' tablosuna okuma erişimi gerekebilir; başka tablolara
   erişimi de, kayıt ekleme/güncelleme/silme yeteneği de olmamalıdır. Bu, aracın
   veritabanına bağlanırken kullandığı kimliğe uygun veritabanı izinleri uygulanarak
   zorlanmalıdır.
5. **Araçları kullanıcının bağlamında çalıştırın.** Kullanıcı adına yapılan eylemlerin
   aşağı akış sistemlerinde o belirli kullanıcının bağlamında ve gerekli asgari
   ayrıcalıklarla yürütülmesi için kullanıcı yetkilendirmesini ve güvenlik kapsamını
   izleyin. Örneğin kullanıcının kod deposunu okuyan bir LLM aracı, kullanıcının OAuth
   ile ve gereken asgari kapsamla kimlik doğrulamasını gerektirmelidir. Devredilmiş
   veya çok ajanlı iş akışlarında, yalnızca çağıran ajanın veya servis kimliğinin
   izinlerine güvenmek yerine, özgün kullanıcı bağlamını ve yetkilendirme kapsamını
   zincirlenmiş araç/ajan çağrıları boyunca koruyun.
6. **Kullanıcı onayı isteyin.** Yüksek etkili eylemler gerçekleştirilmeden önce bir
   insanın onaylamasını gerektiren human-in-the-loop kontrolü kullanın. Bu, bir aşağı
   akış sisteminde (LLM uygulamasının kapsamı dışında) veya LLM aracının kendisinde
   uygulanabilir. Örneğin kullanıcı adına sosyal medya içeriği oluşturup paylaşan LLM
   tabanlı bir uygulama, 'paylaş' işlemini uygulayan aracın içine bir kullanıcı onay
   rutini koymalıdır.
7. **Tam aracılık (complete mediation).** Bir eylemin izinli olup olmadığına LLM'in
   karar vermesine güvenmek yerine yetkilendirmeyi mantık katmanında uygulayın. Aşağı
   akış sistemlerine yapılan tüm isteklerin güvenlik politikalarına karşı doğrulanması
   için tam aracılık ilkesini zorlayın: araç tarafından, araç ile aşağı akış sistemi
   arasındaki bağımsız bir yürütme-öncesi politika karar noktası tarafından veya aşağı
   akış sisteminin kendisi tarafından. Bu politikalar, ajanın nominal olarak izinli bir
   eyleminin bağlamsal olarak güvensiz olduğu durumları yönetmeye yardım eder.
   Kademeli bir zorlama politikası (denetle, uyar, engelle, yükselt); düşük sonuçlu
   veya kolayca geri döndürülebilir eylemlerin otomatik onayına izin verirken yüksek
   sonuçlu ya da geri döndürülemez olanları insan incelemesine yönlendirir. Örneğin
   bir müşteri hizmetleri sohbet botu, iadeyi mağaza kredisi olarak otomatik
   işleyebilir (geri kazanılabilir); harici bir ödeme gibi geri döndürülemez bir eylem
   ise insan onayına gider.

Aşağıdaki seçenekler Aşırı Yetki'yi önlemez ama verilen hasarın düzeyini sınırlayabilir:

8. **Araç kullanımını izleyin.** İstenmeyen eylemlerin nerede gerçekleştiğini tespit
   etmek için LLM araçlarının ve aşağı akış sistemlerinin etkinliğini günlükleyip
   izleyin ve buna göre müdahale edin.
9. **Hız sınırlama.** Araç çağrıları etrafında eşikler belirleyin ve bu eşikler
   aşıldığında durduran, hız sınırlayan veya insan incelemesine yükselten devre
   kesiciler uygulayın. Basit eşikler çağrı sayısına dayanabilir; bağlam farkındalıklı
   eşikler ise bir araca giden girdi parametresinin kümülatif değerine dayanabilir.

## Örnek Saldırı Senaryoları

**Senaryo #1 — Ele Geçirilen E-posta Asistanı:** LLM tabanlı bir kişisel asistan
uygulamasına, gelen e-postaların içeriğini özetlemek üzere bir araç aracılığıyla
kullanıcının posta kutusuna erişim verilir. Bu işlev için araca yalnızca mesaj okuma
yeteneği gerekir; ama geliştiricinin seçtiği araç mesaj gönderme fonksiyonları da
içerir. Uygulama ayrıca dolaylı komut enjeksiyonuna açıktır: kötücül biçimde
kurgulanmış gelen bir e-posta, LLM'i kandırarak ajana kullanıcının gelen kutusunu
hassas bilgi için taratıp bunları saldırganın e-posta adresine ilettirir. Bu şu
yollarla önlenebilirdi:

- yalnızca posta-okuma yetenekleri uygulayan bir araç kullanarak **aşırı işlevselliği
  ortadan kaldırmak**,
- kullanıcının e-posta servisine salt-okunur kapsamlı bir OAuth oturumuyla kimlik
  doğrulayarak **aşırı izinleri ortadan kaldırmak** ve/veya
- LLM aracının taslağını hazırladığı her postayı kullanıcının elle inceleyip
  'gönder'e basmasını gerektirerek **aşırı özerkliği ortadan kaldırmak**.

Alternatif olarak, posta-gönderme arayüzünde hız sınırlama uygulanarak verilen hasar
azaltılabilirdi.

---

*Orijinal: OWASP Top 10 for LLM Applications 2026, OWASP GenAI Security Project,
CC BY-SA 4.0 — https://genai.owasp.org. Bu gayriresmî Türkçe çeviri aynı lisansla
dağıtılır. Kaynakça için orijinal belgenin References bölümüne bakınız.*
