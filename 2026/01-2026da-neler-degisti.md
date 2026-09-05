# 2026 İlk 10'unda Neler Yeni?

*(What's New in the 2026 Top 10 — Türkçe çeviri)*

> Orijinal belgedeki Şekil 1, 2025'ten nihai 2026 listesine sıra geçişlerini hareket
> türüne göre (sabit, yükselen, gerileyen, yeniden adlandırılan/kapsamı değişen) renk
> kodlarıyla gösterir. Şekil için [orijinal belgeye](https://genai.owasp.org/resource/owasp-genai-llm-top-10-2026/) bakınız.

Sıralama geçmiş yıllara göre daha fazla hareket etti ve bu hareketler, inanç ile kanıt
arasındaki o makası izliyor. **Excessive Agency (Aşırı Yetki)** üçüncülüğe tırmandı —
listedeki en sonuç doğurucu hamle — çünkü hem oy hem de kayıt, hasarın ajanlaşmış
(agentic) dağıtımlarda gerçekleştiği konusunda hemfikir. **Unbounded Consumption
(Sınırsız Tüketim)** dört basamak yükseldi; kaynak ve maliyet tükenmesini eski sırasından
daha ağır tartan uygulayıcılar taşıdı onu. **Improper Output Handling (Hatalı Çıktı
İşleme)** en sert düşüşü yaşadı: beşincilikten onunculuğa. **Prompt Injection (Komut
Enjeksiyonu)**, oyun gücü ve arkasındaki savunma etkisiyle zirvedeki yerini korudu.
**Sensitive Information Disclosure (Hassas Bilgi İfşası)** ikinci sırada kaldı — listenin
tepesinde inanç ile kanıtın düpedüz uyuştuğu ve güvenimizin en yüksek olduğu tek yer.
Eskiden **System Prompt Leakage** olan madde artık **Hidden Context Exposure (Gizli
Bağlam İfşası)**: erişilmez kalması gereken bilgiye güvenme başarısızlığının aynısı için
daha geniş bir çerçeve.

Bazı maddeler ayrıca genişledi. Daha yeni ve keskin riskleri, onlara zaten sahip olan
maddelerin içine kattık; incecik yeni kategoriler açmak listeyi hiçbir kazanım olmadan
parçalayacaktı. **Prompt Injection** artık çapraz-modal saldırıları — talimatları bir
görselin ya da ses kaydının içine gizleyen türden — kapsıyor. **Supply Chain (Tedarik
Zinciri)** artık, terfi ettirilen bir model artefaktının iddia ettiği şey olmaması
hâlindeki güven başarısızlığını hesaba katıyor. **Data and Model Poisoning (Veri ve Model
Zehirlenmesi)** artık ince ayar (fine-tuning) yoluyla yıkımı bünyesine alıyor. **Improper
Output Handling** artık asistanların ölçekli ürettiği güvensiz kodu da kapsıyor. Bu
risklerin ardındaki olaylar zaten gerçekti; artık ait oldukları yerde sayılıyorlar.

Her yıl daha da önem kazanan bir sınır var. Model, uygulamanızın **içindeki bir bileşen**
olduğunda risk bu listeye aittir. Model bir **aktöre** dönüştüğü anda — çağırabildiği
araçlarla, oturumlar arasında taşıdığı bellekle ve aşağı akışta tetiklediği sonuçlarla —
risk, OWASP Agentic Top 10'a geçer.

---

*Orijinal: OWASP Top 10 for LLM Applications 2026, OWASP GenAI Security Project,
CC BY-SA 4.0. Bu gayriresmî Türkçe çeviri aynı lisansla dağıtılır.*
