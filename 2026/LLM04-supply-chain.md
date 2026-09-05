# LLM04:2026 Supply Chain — Tedarik Zinciri

## Tanım

LLM tedarik zincirleri; eğitim verisinin, modellerin, adaptörlerin, dönüştürme boru
hatlarının ve dağıtım platformlarının bütünlüğünü etkileyen zafiyetlere açıktır — sonuç,
yanlı çıktılar, güvenlik ihlalleri veya sistem arızalarıdır. Geleneksel yazılım
zafiyetleri kod kusurlarına ve bağımlılıklara odaklanırken, makine öğrenmesinde riskler
üçüncü taraf önceden eğitilmiş modellere, veri kümelerine ve model artefaktlarına
uzanır; bunlar kurcalama, zehirleme veya kötücül artefakt değiştirme yoluyla manipüle
edilebilir.

LLM üretmek; üçüncü taraf modellere, veri kümelerine ve LoRA (Low-Rank Adaptation) gibi
parametre-verimli ince ayar (PEFT) teknikleriyle üretilip Hugging Face benzeri
platformlarda paylaşılan yeniden kullanılabilir adaptörlere bağımlı, uzmanlık isteyen
bir iştir. Tedarik zinciri artık model artefaktlarını, kökeni (provenance) ve
dönüştürme/birleştirme iş akışlarını birinci sınıf saldırı yüzeyleri olarak içerir;
cihaz-üstü LLM'ler bu yüzeyi daha da genişletir.

Burada ele alınan risklerin bir kısmı LLM05:2026 Veri ve Model Zehirlenmesi'nde de
tartışılır; bu madde onların tedarik zinciri boyutuna odaklanır. MCP sunucuları ve araç
kayıtları dâhil, ajan tabanlı uygulamalara özgü tedarik zinciri riskleri OWASP Top 10
for Agentic Applications içindeki ASI04 Agentic Supply Chain Vulnerabilities
kapsamındadır (OWASP GenAI Security Project, 2026); MITRE ATLAS ise karşılık gelen
saldırgan tekniklerini AML.T0010 AI Supply Chain Compromise altında kataloglar (MITRE,
t.y.).

> Orijinal belgedeki Şekil 3, basit bir LLM tedarik zinciri tehdit modeliyle saldırı
> yüzeylerini gösterir.

## Yaygın Risk Örnekleri

1. **Zafiyetli veya güncelliğini yitirmiş üçüncü taraf bileşenler ve modeller.**
   Paketler, sunum çatıları ve modellerin kendileri dâhil, eskimiş veya kullanımdan
   kaldırılmış bileşenler LLM uygulamalarını ele geçirmek için istismar edilebilir. Bu,
   A06:2021 Vulnerable and Outdated Components'e benzer; bileşenler model geliştirme,
   fine-tuning veya çıkarım sırasında kullanıldığında riskler artar. LLM kodlama
   asistanları yeni bir varyant ekler: makul görünen ama var olmayan paket adlarını
   ölçekli halüsinasyona uğratırlar (Spracklen vd., 2025); saldırganlar bu adları
   önceden kaydeder ("slopsquatting") ve doğrulanmamış AI önerili bağımlılıklar kötücül
   koda çözümlenir.
2. **Lisanslama riskleri.** AI geliştirme; farklı kullanım, dağıtım ve
   ticarileştirme şartları dayatan çeşitli yazılım ve veri kümesi lisansları içerir.
   Düzgün yönetilmezlerse hukuki ve uyum riski doğururlar.
3. **Zafiyetli veya kurcalanmış önceden eğitilmiş modeller.** Model artefaktlarını
   kapsamlıca incelemek zordur ve statik analiz tek başına davranışsal güvenliği
   kanıtlayamaz. Önceden eğitilmiş bir model, zehirlenmiş veri kümeleri veya doğrudan
   kurcalama yoluyla sokulmuş gizli yanlılıklar ya da arka kapılar içerebilir.
   Yüklemede keyfî kod çalıştırabilen Python pickle gibi güvensiz serileştirme
   biçimlerinden uzaklaşmak bu riski azaltır ama ortadan kaldırmaz: bir arka kapı
   doğrudan modelin hesaplama çizgesine (computational graph) gömülebilir ve ONNX gibi
   yaygın biçimde güvenli sayılan biçimlerde varlığını sürdürebilir; kurgulanmış bir
   model dosyası, biçimin yerel ayrıştırıcısındaki bellek bozulması hatalarını da
   istismar edebilir — llama.cpp'nin GGUF ayrıştırmasındaki yığın taşmalarının
   gösterdiği gibi (CVE-2024-23496) (Cisco Talos, 2024).
4. **Zayıf köken ve imzasız model artefaktları.** Yayımlanan modeller güçlü köken
   güvencesi taşımaz: Model Card'lar modeli belgelendirir ama kaynağını kanıtlamaz;
   ele geçirilmiş veya taklit bir tedarikçi hesabı, güvenilen bir ad altında kötücül
   model yayımlayabilir. Modeller, adaptörler, veri kümeleri, fine-tune edilmiş
   kontrol noktaları ve dönüştürme çıktıları imzalanmadığında veya hash ile
   sabitlenmediğinde, saldırgan artefaktları aktarımda, depoda veya otomatik boru
   hatlarının artefaktları güvenilir ortamlara kabul ettiği terfi sınırında
   değiştirebilir ya da sessizce bozabilir — özellikle boru hatları artefaktı
   değişmez bir özet (digest) yerine değişebilir bir referansla (örneğin latest
   etiketi) çözüyorsa.
5. **Zafiyetli adaptörler; ele geçirilmiş dönüştürme, birleştirme ve nicemleme iş
   akışları.** LoRA adaptörleri fine-tuning'i modüler ve verimli kılar; ama kötücül
   bir adaptör, ister işbirlikçi model birleştirme ortamlarında ister adaptörleri
   indirip konuşlandırılmış modele uygulayan çıkarım platformlarında olsun, taban
   modelin bütünlüğünü bozabilir. Model dönüştürme ve birleştirme servisleri, biçimler
   arası dönüşüm sırasında inceleme kontrollerini atlayarak kötücül değişiklikler
   sokabilir. Nicemleme (quantization) ilişkili bir dönüşüm riskidir: model
   ağırlıkları, tam duyarlıklı model masum davranırken nicemlenmiş artefakt saldırganın
   seçtiği davranışı sergileyecek şekilde kurgulanabilir (Egashira vd., 2025);
   dolayısıyla tam duyarlıklı güvenceler, dağıtılan nicemlenmiş artefakta aktarılamaz.
6. **Cihaz-üstü LLM tedarik zinciri zafiyetleri.** Cihazlarla gönderilen LLM'ler;
   ele geçirilmiş üretim süreçlerini, cihaz işletim sistemi veya donanım yazılımı
   (firmware) zafiyetlerinin istismarını ve kurcalanmış modellerle yeniden paketlenmiş
   uygulamaları saldırı yüzeyine ekler — cihaz bütünlüğünü ve firmware güvenini LLM
   tedarik zincirinin parçası yapar.
7. **Belirsiz kullanım şartları ve veri gizliliği politikaları.** Model
   işletmecilerinin belirsiz şartları ve gizlilik politikaları, hassas uygulama
   verisinin model eğitiminde kullanılmasına ve ardından ifşasına yol açabilir; model
   tedarikçisinin sağladığı malzemeden telif riski de doğurabilir.

## Önleme ve Azaltma Stratejileri

1. Veri kaynaklarını ve tedarikçileri — kullanım şartları ve gizlilik politikaları
   dâhil — özenle inceleyin ve yalnızca güvenilir tedarikçiler kullanın. Tedarikçi
   güvenliğini ve erişimini düzenli olarak gözden geçirip denetleyin; güvenlik
   duruşları veya şartları değiştiğinde yeniden değerlendirin.
2. OWASP Top Ten A06:2021'in önlemlerini uygulayın: zafiyet taraması, yönetimi ve
   uygulamayı bileşenlerin, API'lerin ve alttaki modellerin bakımı süren sürümlerinde
   tutan bir yama politikası. Aynı kontrolleri hassas veriye erişimi olan geliştirme
   ortamlarına da uygulayın; AI önerili bağımlılıkların var olduğunu ve amaçlanan
   paket olduğunu benimsemeden önce doğrulayın.
3. Üçüncü taraf model seçerken, kapsamdaki kullanım senaryolarına odaklı AI red team
   çalışması ve değerlendirmeler uygulayın; üretimde MLOps ve LLM boru hatlarında
   kurcalama ve zehirlemeyi tespit için anomali tespiti ve düşmanca dayanıklılık
   testleriyle sürdürün.
4. Yazılım Malzeme Listesi (SBOM) kullanarak bileşenlerin güncel, imzalı bir
   envanterini tutun; bunu AI BOM'lar (AIBOM) ve ML SBOM'larla modellere, adaptörlere
   ve veri kümelerine genişletin — OWASP CycloneDX ML-BOM (OWASP CycloneDX, t.y.) ve
   OWASP AIBOM projesi (OWASP GenAI Security Project, 2025) gibi seçenekleri
   değerlendirin. Lisansları aynı envanterde izleyin ve uyum ile şeffaflık için
   düzenli denetleyin.
5. Yalnızca doğrulanabilir kaynaklardan model kullanın ve zayıf kökeni üçüncü taraf
   bütünlük kontrolleri, imzalama ve dosya hash'leriyle telafi edin. Şeffaflık
   günlüğüyle desteklenen kriptografik model imzalama (örneğin OpenSSF Model Signing
   projesi ve Sigstore), model artefaktını bir imzacı kimliğine bağlar (Open Source
   Security Foundation, 2025). Model eğitiminde yeniden üretilebilir derlemeler
   garanti değildir; Coalition for Secure AI (CoSAI), ML artefaktlarını imzalamak için
   bir olgunluk modeli sunar (OASIS Open, 2025). İmzalama bütünlüğü ve kökeni
   kanıtlar, güvenliği değil (ele geçirilmiş ya da güvenilen-ama-kötücül bir
   tedarikçiden geçerli imzalı bir model yine arka kapılı olabilir); bu yüzden onu
   değişmez artefakt referansları, köken politikası, politika tabanlı sürüm kapıları
   (örneğin SLSA) (Open Source Security Foundation, t.y.), davranışsal değerlendirme
   ve yukarı akış model bütünlüğünün sürekli doğrulamasıyla birleştirin. Benzer
   şekilde, dışarıdan sağlanan kod için kod imzalama kullanın.
6. İşbirlikçi model geliştirme ortamlarını sıkı biçimde izleyip denetleyin; model
   dönüştürme ve birleştirme servislerini yüksek riskli terfi noktaları olarak ele
   alın.
7. Uçta (edge) dağıtılan modelleri bütünlük kontrolleriyle şifreleyin, kurcalanmış
   uygulama ve modelleri önlemek için üretici doğrulama (attestation) API'leri
   kullanın; tanınmayan firmware'i ve güvenilmeyen cihaz durumlarını reddedin.

## Örnek Saldırı Senaryoları

**Senaryo #1 — Ele Geçirilmiş Paketler ve Sunum Çatıları:** Ele geçirilmiş bir
bağımlılık, model geliştirme veya çıkarım ortamına ulaşır — Aralık 2022 PyTorch tedarik
zinciri saldırısında olduğu gibi: PyPI'daki kötücül torchtriton paketi meşru
PyTorch-nightly bağımlılığını gölgeleyip veri sızdırdı (PyTorch Foundation, 2022).
Sunum yığını da aynı saldırı yüzeyinin parçasıdır: ShadowRay saldırıları, tartışmalı
CVE-2023-48022'yi (üretim Ray sunucularında kimlik doğrulamasız panolar) sahada
istismar etti ve sonradan açık kümeler arasında kendi kendine yayılan bir botnet'e
tırmandı (Lumelsky & Elbaz, 2025); Ollama'da ise CVE-2024-37032, bir kayıttan çekilen
kötücül model manifestiyle uzaktan kod yürütmeye izin verdi (Wiz, 2024).

**Senaryo #2 — Hub'a Yayımlanan Kurcalanmış Model:** Saldırgan, güvenilir görünen bir
adla kurcalanmış bir model yayımlar — PoisonGPT kavram kanıtının gösterdiği gibi:
parametreleri cerrahi biçimde değiştirilmiş bir model, standart benchmark
değerlendirmesinden kaçarken yanlış bilgi yaydı (Huynh & Hardouin, 2023). Aynı güven
boşluğu; popüler bir modelin güvenlik özelliklerini söküp masum görevlerdeki
performansını koruyan saldırgan fine-tune'larını (Zhan vd., 2023), doğrulamasız
konuşlandırılan her önceden eğitilmiş modeli ve paylaşımlı AI-as-a-service
platformlarını da kapsar — araştırmacılar kötücül bir model üzerinden çıkarım
konteynerinden kaçıp diğer müşterilerin modellerine ve verisine ulaşmıştır (Tamari &
Tzadik, 2024).

**Senaryo #3 — Ele Geçirilmiş Tedarikçi LoRA Adaptörü:** Saldırgan üçüncü taraf bir
tedarikçiye sızar ve sonradan bir model-birleştirme iş akışıyla konuşlandırılmış LLM'e
katılan bir LoRA adaptörünü ince biçimde değiştirir. Birleştirme sonrası adaptör,
sisteme örtülü bir giriş noktası sağlar.

**Senaryo #4 — Kaçırılan Model Dönüştürme veya Birleştirme Servisi:** Saldırgan,
herkese açık bir modeli ele geçirip kötücül davranış enjekte etmek için bir model
birleştirme veya biçim dönüştürme servisi üzerinden saldırı kurgular — HiddenLayer'ın
Hugging Face'teki Safetensors dönüştürme botunu kaçırma araştırmasının gösterdiği gibi
(HiddenLayer, 2024).

**Senaryo #5 — Model Ad Alanının Yeniden Kullanımı:** Bir kuruluş, halka açık bir
hub'daki modeli yalnızca Author/ModelName tanımlayıcısıyla referans vererek
konuşlandırır. Özgün yazar hesabı siler veya devreder, ad alanı boşalır; saldırgan aynı
adı yeniden kaydedip özgün yol altında kötücül bir model yayımlar. Modeli yalnızca ada
göre çözen boru hatları ve yönetilen model katalogları saldırganın modelini çeker —
sonuç uzaktan kod yürütme (Saraf & Balassiano, 2025).

**Senaryo #6 — Tarayıcı, Güvenli-Yükleyici ve Güvenli-Biçim Atlatması:** Bir kuruluş,
üçüncü taraf modelleri bir kötücül yazılım tarayıcısının ve kod yürütmeye karşı güvenli
diye belgelenmiş bir yükleyici seçeneğinin arkasına alır. Saldırgan ikisini de yener.
Bozulmuş veya sıkıştırmayla sarılmış pickle akışları, tarayıcı bozuk bayta ulaşmadan
payload'larını çalıştırır — Hugging Face'te bulunan nullifAI modellerinde olduğu gibi
(Zanki, 2025). Model tarayıcılarına ve güvenli-yükleyici bayraklarına CVE atanmış
atlatmalar çıkmıştır: PickleScan sıfır-günleri (Cohen, 2025) ve weights_only ile
torch.load (CVE-2025-32434) (GitHub Security Advisories, 2025). ONNX gibi "güvenli"
bir biçimin hesaplama çizgesindeki ShadowLogic tarzı bir arka kapı (Wickens vd., 2024),
serileştirme tarayıcısının işaretleyeceği hiçbir yürütülebilir kod içermez.
Tarayıcıları ve güvenli-yükleyici bayraklarını garanti değil, köken doğrulaması ile
yamalı yükleyici ve ayrıştırıcıların yanında birer derinlemesine savunma katmanı olarak
ele alın.

**Senaryo #7 — Model Artefaktları için Ele Geçirilmiş Derleme Boru Hattı:** Saldırgan,
kuruluşun modelleri fine-tune edip yayımlamak için kullandığı CI/CD boru hattını ele
geçirir — kötücül bir derleme iş akışı bağımlılığı, çalınmış bir artefakt-kaydı kimlik
bilgisi veya önbellek zehirlemesiyle; Ultralytics saldırısında olduğu gibi: GitHub
Actions önbellek enjeksiyonu, amiral gemisi bir AI kütüphanesinin truva atlı PyPI
sürümlerini yayımladı (Python Package Index, 2024) — ele geçirilmiş bir "düzeltme"
sürümü dâhil. Bu, xz-utils arka kapısı ve Codecov ihlali gibi klasik olaylardaki
derleme-zamanı ikamesinin aynısıdır. Arka kapılı artefakt kuruluşun kendi sürüm
altyapısınca derlenip imzalandığı için; aşağı akış köken kontrollerini, iç doğrulamayı
ve yalnızca dış kaynaklı bileşenleri işaretleyen tedarik zinciri tarayıcılarını geçer.

**Senaryo #8 — Tersine Mühendislikle Bozulan Mobil Uygulama:** Saldırgan bir mobil
uygulamayı tersine mühendislikle açar, cihaz-üstü modeli kullanıcıları dolandırıcılık
sitelerine yönlendiren kurcalanmış bir sürümle değiştirir ve yeniden paketlenen
uygulamayı sosyal mühendislikle dağıtır.

---

*Orijinal: OWASP Top 10 for LLM Applications 2026, OWASP GenAI Security Project,
CC BY-SA 4.0 — https://genai.owasp.org. Bu gayriresmî Türkçe çeviri aynı lisansla
dağıtılır. Kaynakça için orijinal belgenin References bölümüne bakınız.*
