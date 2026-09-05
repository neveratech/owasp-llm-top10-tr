# LLM08:2026 Hidden Context Exposure — Gizli Bağlam İfşası

## Tanım

Gizli Bağlam İfşası; modelin bağlamına yerleştirilen, kullanıcıya dönük olmayan gizli
sistem talimatlarının veya operasyonel bağlamın yetkisiz çıkarılması, çıkarsanması ya
da yeniden kurulmasıdır. Gizli bağlam; sırlar, politika mantığı, araçlar, güven
sınırları, iş akışı ölçütleri, mülkiyete konu davranış veya saldırgan kabiliyetini
kayda değer biçimde artıran diğer hassas uygulama ayrıntılarını içerdiğinde ya da
açığa vurduğunda güvenlik açısından önem kazanır.

Bir LLM uygulamasında gizli bağlam tipik olarak şunları içerir: sistem komutu,
geliştirici talimatları, getirilen politika metni (RAG bilgi tabanlarından,
yapılandırma depolarından veya kullanıcı-profil servislerinden), uygulamanın modele
açtığı araç ve fonksiyon şemaları ile uygulamanın modelin bağlam penceresine
derlediği diğer kurallar, yönergeler ve malzemeler. Ortak nokta: bu gizli bağlamın son
kullanıcılara görünür olması amaçlanmaz ama modele erişilebilirdir.

Uygulayıcılar, **gizli bağlamın keşfedilebilir olduğu** ve bağlamdaki hiçbir içeriğin
sır sayılmaması gerektiği varsayımıyla tasarım yapmalıdır. Uygulama geliştiricileri,
gizli bağlamın ifşasının doğrudan güvenlik etkisinin az veya sıfır olmasını
sağlamalıdır. Kimlik bilgileri, bağlantı dizeleri ve token'lar gibi hassas veriler
gizli bağlama gömülmemeli; gizli bağlama yetkilendirme, ayrıcalık ayrımı, politika
zorlama veya içerik filtreleme için tek güvenlik sınırı olarak da bel bağlanmamalıdır.

Şiddet; gizli bağlama neyin konduğunu ve uygulamanın ona nasıl dayandığını izler.
Bulgular; bilgilendirici düzeyden (sır yok, güvenlikle ilgili mantık yok, gizliliğe
dayanma yok), orta düzeye (saldırgana anlamlı yardım eden ama kritik kararları
kapılamayan iç kurallar, filtreleme ölçütleri, rol tanımları veya iş akışı mantığı),
yükseğe (gömülü kimlik bilgileri ya da token'lar veya yetkilendirme/içerik politikası
için gizli-bağlam gizliliğine dayanma) ve kritiğe (ifşanın bağlı bir sistemde uzaktan
kod yürütmeye, geniş veri sızdırmaya veya ayrıcalık yükseltmeye zincirlendiği durum)
uzanır.

Gizli Bağlam İfşası kendi başına risk getirmenin yanında komşu kategorilerdeki
riskleri de sıkça büyütür:

- İfşa olan kurallar veya mantık, daha hedefli komut enjeksiyonunu mümkün kılar
  (LLM01:2026).
- Gömülü kimlik bilgileri hassas bilgi ifşası oluşturur (LLM02:2026).
- Açığa çıkan araç izinleri ve şemaları, aşırı yetki yüzeyini genişletir (LLM03:2026).
- Sızan çıktı biçimlendirme kuralları, hatalı çıktı işlemeyi kolaylaştırabilir
  (LLM10:2026).

Özetle LLM08, gizli LLM kontrol bağlamının saldırgan kabiliyetini kayda değer biçimde
artıracak şekilde ifşa edilmesi, çıkarsanması veya yeniden kurulması şeklindeki temel
riski kapsar. LLM08 şunları **kapsamaz**:

- Düzenlemeye tabi kullanıcı veya eğitim verisinin sızması (LLM02 Hassas Bilgi İfşası).
- Bu riskin ajan tabanlı büyümeleri — örneğin kalıcı bellek, ajanlar arası kanallar,
  araç yapılandırması kalıcılığı ve çok adımlı ajan ele geçirmesi (OWASP Top 10 for
  Agentic Applications).
- LLM entegre sistemlerin devraldığı genel uygulama güvenliği konuları — örneğin
  sunucu tarafı günlük sızıntısı, istemci tarafı paket incelemesi ve altyapı katmanı
  yan kanalları.

## Yaygın Risk Örnekleri

1. **Hassas işlevselliğin, araç ve fonksiyon şemalarının ifşası.** Uygulamanın sistem
   komutu veya gizli bağlamı, gizli tutulması amaçlanan hassas bilgi ya da
   işlevselliği açığa vurabilir: hassas sistem mimarisi, mevcut araçlar ve
   fonksiyonlar, API anahtarları, veritabanı kimlik bilgileri veya kullanıcı
   token'ları. Bunların ifşası muhtemelen zarar verir; ama asıl risk, hassas kimlik
   bilgilerinin gizli bağlama en baştan konmuş olmasıdır.
2. **Davranışsal kontrol mantığının ifşası.** Uygulamanın bağlamı, gizli kalması
   gereken iç karar süreçlerine dair bilgi içerir. Bu bilgi, saldırganlara uygulamanın
   nasıl işlediğine dair içgörü verir; zayıflıkları istismar etmek veya kontrolleri
   atlatmak için kullanabilirler.
3. **Güvenlik ve reddetme mekanizmalarının tersine mühendisliği.** Sistem komutları,
   modelin hangi koşullarda reddetmesi veya içerik filtrelemesi gerektiğini
   tanımlayabilir. Bu talimatlar sistem komutu sızıntısıyla açığa çıktığında,
   saldırganlar reddetme davranışını yöneten kurallara görünürlük kazanır. Sıradan
   kullanıcı yalnızca "Üzgünüm, bunu yapamam" gibi yanıtlar görürken; sızıntı, o
   karara götüren tetikleyicileri, koşulları ve istisnaları açığa vurur. Saldırganlar
   böylece bilinen reddetme kalıplarından kasten kaçınan veya zorlamadaki boşlukları
   istismar eden girdiler kurgulayabilir — kısıtlı yanıtları elde etme olasılığı
   artar.
4. **İzinlerin ve kullanıcı rollerinin ifşası.** Talimat bağlamı; yetkilendirme ve
   izinlere ilişkin yönergeler veya bilgiler içerebilir. Örneğin iç ağa dönük bir MCP
   sunucusundan sağlanan bir araç açıklaması, aracı kullanmak için kullanıcının
   developer rolüne sahip olması gerektiğini ya da belirli rollü bir kullanıcının
   RAG ile aranacak belge listesine erişebildiğini belirtebilir. Bu bilginin ifşası,
   yönlendirilmiş konuşma ve komut enjeksiyonuyla (LLM01:2026) başka yoklama türlerini
   davet edebilir ve potansiyel olarak ek hassas bilgiyi (LLM02:2026) açığa
   çıkarabilir.
5. **Çıktı yapısının ve biçimlendirme kurallarının ifşası.** Sistem komutları
   yanıtların nasıl yapılandırılacağını sıkça tanımlar: JSON şemaları, şablonlar veya
   doğrulama kısıtları gibi zorunlu biçimler. Bu talimatlar açığa çıktığında,
   saldırganlar çıktıların nasıl kurulduğuna ve aşağı akış sistemlerinin hangi
   varsayımlara dayandığına içgörü kazanır. Bu bilgi, beklenen biçimlere uyarken
   istenmeyen veya manipüle edilmiş değerler gömen yanıtlar üretmekte kullanılabilir —
   hatalı ayrıştırma veya istenmeyen sistem davranışı olasıdır.

## Önleme ve Azaltma Stratejileri

1. **Gizli bağlama hassas veri koymayın.** Kimlik bilgilerini, sırları veya güvenlik
   açısından kritik yapılandırmayı doğrudan sistem komutlarına ya da gizli bağlama
   gömmeyin. LLM'e açık her bağlamın kullanıcılara da açık olabileceğini varsayın.
   Bu bilgileri modelin doğrudan erişmediği sistemlere dışsallaştırın ve hassas veriyi
   modelin kendisinin işlemesinden kaçının.
2. **Doğrulama ve davranış kontrolü için deterministik yöntemler ve korkuluklar
   (guardrail) kullanın.** LLM'ler komut enjeksiyonu gibi saldırılara açık
   olabildiğinden, gizli bağlam model davranışını kontrol etmenin birincil mekanizması
   olarak kullanılmamalıdır. Modelin özel fine-tuning'i veya ek eğitimi ifşa riskini
   azaltabilir; ama tutarlı garanti sağlamaz ve başka istenmeyen sonuçları olabilir.
   Kritik davranışları modelin dışında, bağımsız ve deterministik sistemlerle
   zorlayın. Örneğin zararlı içerik tespiti ve önlenmesi, gizli bağlama gömülü
   talimatlar yerine harici korumalarla yapılmalıdır.
3. **Yetkilendirme ve erişim kontrolünü LLM'den bağımsız zorlayın.** Ayrıcalık
   ayrımı, yetkilendirme sınırı kontrolleri ve benzeri kritik kontroller — sistem
   komutuyla veya başka bir mekanizmayla — LLM'e devredilmemelidir. Bu kontroller,
   LLM'lerin sağlamaya uygun olmadığı deterministik ve denetlenebilir biçimde
   uygulanmalıdır. Görevler farklı erişim düzeyleri gerektiriyorsa bunları
   yetkilendirme bağlamına göre ayırın ve her birine yalnızca gerektirdiği
   ayrıcalıkları verin.

## Örnek Saldırı Senaryoları

**Senaryo #1 — Sistem komutuyla kimlik bilgisi sızıntısı:** LLM'in sistem komutu,
erişim verilen bir araç için kullanılan bir kimlik bilgisi kümesi içerir. Sistem
komutu bir saldırgana sızar; saldırgan bu kimlik bilgilerini başka amaçlarla
kullanabilir.

**Senaryo #2 — Bağlam çıkarmayla araç şeması:** Saldırgan, araç listesini ve parametre
şemalarını içeren gizli bağlamı konuşma yoklamasıyla çıkarır ve bilgiyi uygulamayı
belirli araç çağrılarına yönlendiren girdiler kurgulamakta kullanır. Hiçbir kimlik
bilgisi ifşa olmaz, hiçbir politika açıkça atlatılmaz; ama saldırgan artık sonraki
komut enjeksiyonu denemeleri için somut hedeflere ve aşağı akış eylem zincirlemesi
için keşfe sahiptir.

**Senaryo #3 — Korkuluk ifşasıyla kısıt atlatma:** LLM'in sistem komutu; saldırgan
içerik üretimini, harici bağlantıları ve kod yürütmeyi yasaklar. Saldırgan bu sistem
komutunu çıkarır ve ifşa olan kısıtları, onları atlatan bir komut enjeksiyonu
saldırısı kurgulamakta kullanır.

---

*Orijinal: OWASP Top 10 for LLM Applications 2026, OWASP GenAI Security Project,
CC BY-SA 4.0 — https://genai.owasp.org. Bu gayriresmî Türkçe çeviri aynı lisansla
dağıtılır. Kaynakça için orijinal belgenin References bölümüne bakınız.*
