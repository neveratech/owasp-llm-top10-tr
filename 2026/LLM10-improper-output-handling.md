# LLM10:2026 Improper Output Handling — Hatalı Çıktı İşleme

## Tanım

Hatalı Çıktı İşleme; büyük dil modellerinin ürettiği çıktıların, aşağı akıştaki diğer
bileşen ve sistemlere iletilmeden önce yetersiz doğrulanması, arındırılması ve
işlenmesini ifade eder. LLM üretimi içerik komut girdisiyle kontrol edilebildiğinden,
bu davranış kullanıcılara ek işlevselliğe dolaylı erişim vermeye benzer. Hatalı Çıktı
İşleme, model çıktısının aşağı akışa geçirilmeden önceki güvensiz kullanımıyla
ilgilenir; LLM07:2026 Yanlış Bilgi ise hatalı veya yanıltıcı çıktının kendisiyle.
Model girdilerinin doğrulanması ve arındırılması LLM01:2026 Komut Enjeksiyonu
kapsamındadır. Bir Hatalı Çıktı İşleme zafiyetinin başarılı istismarı; web
tarayıcılarında XSS ve CSRF'ye, arka uç sistemlerde SSRF'ye, ayrıcalık yükseltmeye
veya uzaktan kod yürütmeye yol açabilir. Şu koşullar bu zafiyetin etkisini artırır:

- LLM'e tanınan aşırı uygulama ayrıcalıkları — ayrıcalık yükseltme veya uzaktan kod
  yürütmeyi mümkün kılar.
- Saldırgana hedef kullanıcının ortamına ayrıcalıklı erişim verebilecek dolaylı komut
  enjeksiyonuna açıklık.
- Üçüncü taraf araçlardan gelen doğrulanmamış girdiler.
- Bağlama özgü çıktı kodlamasının eksikliği (örneğin HTML, JavaScript, SQL).
- LLM çıktılarının yetersiz izlenmesi ve günlüklenmesi.
- LLM kullanımı için hız sınırlama veya anomali tespitinin yokluğu.
- ANSI kaçış dizileri gibi kontrol karakterlerini etkisizleştirmeden model çıktısını
  render eden terminal, günlük veya IDE hedefleri (sink).
- Model çıktısında referans verilen harici kaynakları (örneğin Markdown görselleri,
  bağlantı ön izlemeleri, iframe'ler) otomatik getiren istemci render'ları (tarayıcı,
  sohbet arayüzü, IDE, terminal) — giden isteklerle bağlam verisinin sızdırılmasına
  olanak tanır.

## Yaygın Risk Örnekleri

1. LLM çıktısı doğrudan bir sistem kabuğuna veya exec/eval benzeri bir fonksiyona
   girilir — sonuç uzaktan kod yürütme.
2. LLM'in ürettiği JavaScript veya Markdown kullanıcıya döndürülür; kod tarayıcıda
   yorumlanır — sonuç XSS.
3. LLM üretimi SQL sorguları düzgün parametrelendirme olmadan çalıştırılır — SQL
   enjeksiyonu.
4. LLM çıktısı, uygun arındırma olmadan dosya yolları kurmakta kullanılır — olası yol
   aşımı (path traversal) zafiyetleri.
5. LLM üretimi içerik, uygun kaçışlama olmadan e-posta şablonlarında kullanılır —
   olası oltalama saldırıları.
6. ANSI kaçış dizileri veya diğer kontrol karakterleri içeren LLM çıktısı; bunları
   yorumlayan bir terminale, günlük görüntüleyiciye veya IDE paneline yazılır —
   görsel sahtecilik, pano kaçırma (örneğin OSC 52) veya bilinen terminal emülatörü
   zafiyetlerinin istismarı mümkün olur (Rehberger, 2024b).
7. Sohbet arayüzü, model çıktısında referans verilen Markdown görsellerini veya
   bağlantı ön izlemelerini otomatik render eder; model bağlamının bir kısmını
   kontrol eden saldırgan, konuşma verisini görsel URL'sinin ana bilgisayar adı veya
   sorgu dizesi üzerinden sızdırabilir (Rehberger, 2024a).

## Önleme ve Azaltma Stratejileri

1. Modeli herhangi bir kullanıcı gibi ele alın: sıfır güven yaklaşımı benimseyin ve
   modelden arka uç fonksiyonlarına gelen yanıtlara uygun girdi doğrulaması uygulayın.
2. Etkili girdi doğrulama ve arındırma için OWASP ASVS (Application Security
   Verification Standard) yönergelerini izleyin (OWASP, t.y.).
3. JavaScript veya Markdown ile istenmeyen kod yürütmeyi azaltmak için model
   çıktısını kullanıcılara dönerken kodlayın. OWASP ASVS, çıktı kodlaması için
   ayrıntılı rehberlik sağlar.
4. LLM çıktısının kullanılacağı yere göre bağlam-farkındalıklı çıktı kodlaması
   uygulayın (örneğin web içeriği için HTML kodlaması, tarayıcı betik bağlamları için
   JavaScript kodlaması).
5. LLM çıktısı içeren tüm veritabanı işlemlerinde parametreli sorgular veya
   hazırlanmış ifadeler (prepared statements) kullanın.
6. LLM üretimi içerikten kaynaklanan XSS riskini azaltmak için katı İçerik Güvenliği
   Politikaları (CSP) uygulayın.
7. LLM çıktılarında istismar girişimine işaret edebilecek olağandışı kalıpları
   yakalamak için günlükleme ve izleme sistemleri kurun.
8. Model çıktısı terminale, günlük dosyalarına veya diğer yorumlayan hedeflere
   yazılmadan önce kontrol karakterlerini (ANSI kaçış dizileri, BEL, OSC, backspace,
   satır başı) ve diğer basılamaz baytları arındırın. Korunmaları gerekiyorsa görünür
   biçimde kodlayın.
9. İstemci render'larında (sohbet arayüzleri, IDE'ler, e-posta istemcileri, mobil
   uygulamalar) model çıktısının saldırgan kontrolündeki uç noktalara otomatik giden
   istekler tetiklemesini önleyin. Markdown görsellerinin, bağlantı ön izlemelerinin,
   iframe'lerin ve benzeri ögelerin otomatik render'ını varsayılan olarak kapatın.
   Render gerekiyorsa getirmeleri açık bir kaynak izin listesiyle kısıtlayın ya da
   veri taşıyan sorgu parametrelerini ayıklayan sunucu tarafı bir getiriciden geçirin.

## Örnek Saldırı Senaryoları

**Senaryo #1:** Bir uygulama, sohbet botu özelliği için yanıt üretmekte bir LLM aracı
kullanır. Araç ayrıca başka bir ayrıcalıklı LLM'in erişebildiği bir dizi yönetim
fonksiyonu sunar. Genel amaçlı LLM, yanıtını uygun çıktı doğrulaması olmadan doğrudan
araca geçirir ve aracın bakım için kapanmasına neden olur.

**Senaryo #2:** Kullanıcı, bir makalenin kısa özetini üretmek için LLM destekli bir
web sitesi özetleme aracı kullanır. Site, LLM'e sitedeki veya kullanıcının
konuşmasındaki hassas içeriği yakalamasını söyleyen bir komut enjeksiyonu içerir.
Oradan LLM, hassas veriyi kodlayıp hiçbir çıktı doğrulaması veya filtresi olmadan
saldırgan kontrolündeki bir sunucuya gönderebilir.

**Senaryo #3:** Bir LLM, kullanıcıların sohbet benzeri bir özellikle arka uç
veritabanı için SQL sorguları kurmasına izin verir. Kullanıcı, tüm veritabanı
tablolarını silen bir sorgu ister. LLM'in kurduğu sorgu denetlenmezse tüm tablolar
silinir.

**Senaryo #4:** Bir web uygulaması, kullanıcı metin komutlarından çıktı arındırması
olmadan içerik üretmek için LLM kullanır. Saldırgan, LLM'e arındırılmamış bir
JavaScript payload'ı döndürten kurgulanmış bir komut gönderebilir — kurbanın
tarayıcısında render edildiğinde XSS. Model çıktısının yetersiz doğrulanması ve
kodlanması bu saldırıyı mümkün kılmıştır.

**Senaryo #5:** Bir LLM, pazarlama kampanyası için dinamik e-posta şablonları
üretmekte kullanılır. Saldırgan, LLM'i e-posta içeriğine kötücül JavaScript katmaya
yönlendirir. Uygulama LLM çıktısını düzgün arındırmazsa, e-postayı zafiyetli
istemcilerde görüntüleyen alıcılarda XSS saldırılarına yol açabilir.

**Senaryo #6:** Bir uygulama, LLM üretimi kodu insan incelemesi veya güvenlik testi
olmadan otomatik derleyip dağıtır. Çıktıya güvenilip doğrulamasız yürütüldüğü için
güvensiz kod üretime ulaşır ve istismar edilir.

---

*Orijinal: OWASP Top 10 for LLM Applications 2026, OWASP GenAI Security Project,
CC BY-SA 4.0 — https://genai.owasp.org. Bu gayriresmî Türkçe çeviri aynı lisansla
dağıtılır. Kaynakça için orijinal belgenin References bölümüne bakınız.*
