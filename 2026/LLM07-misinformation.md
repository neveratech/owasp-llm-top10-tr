# LLM07:2026 Misinformation — Yanlış Bilgi

## Tanım

Yanlış bilgi; bir LLM'in veya LLM destekli uygulamanın, bir insan kararını, otomatik
bir iş akışını veya bir ajan eylemini etkileyecek kadar inandırıcı görünen hatalı,
eksik, desteksiz veya yanıltıcı bilgi üretmesiyle oluşur. Temel risk, hatalı çıktıya
**güvenilmesi ve üzerine eylem kurulmasıdır**.

Modern sistemlerde model çıktıları araç çağrılarını sürer, kod üretir, sistem durumu
çıkarsar, eylemleri yetkilendirir ve ajanlar arası eşgüdüm sağlar. Bu, yanlış bilgiyi
finansal kayba, güvenlik olaylarına, emniyet risklerine veya operasyonel kesintiye yol
açabilen sistem düzeyinde bir başarısızlık yapar.

Ajan tabanlı sistemlerde yanlış bilgi çoğu zaman aşağı akış bileşenlerince tüketilen
hatalı durum, akıl yürütme veya kanıt olarak tezahür eder ve doğrudan istenmeyen
eylemlere götürür.

Yanlış bilgi; halüsinasyondan, eksik veya bayat bağlamdan, zayıf temellendirmeden,
muğlak komutlardan, yanlı ya da bozulmuş veriden, yanıltıcı özetlerden veya
doğrulanmamış araç çıktılarından doğabilir. Saldırganlarca kasıtlı olarak da
tetiklenebilir. Kök neden komut enjeksiyonu, zehirleme veya tedarik zinciri ele
geçirmesi olduğunda o riskler ayrıca referans verilmelidir. Güvensiz üretilen kodun
yürütülmesi ve işlenmesi LLM10:2026 Hatalı Çıktı İşleme, halüsinasyona uğramış paket
adlarının tedarik zinciri vektörü olarak kaydedilmesi LLM04:2026 Tedarik Zinciri
kapsamındadır. Bu madde, sonuçtaki arıza kipine odaklanır: **zararlı bir karar veya
eylemi süren yanlış bir temsil.**

Aşırı güven kilit etken olmayı sürdürüyor. İnsanlar ve sistemler; akıcı, kendinden
emin veya iyi yapılandırılmış çıktıları çoğu zaman yetkili sayar. Ajan mimarilerinde
bu aşırı güven sıklıkla sistem tasarımının içine gömülüdür.

## Yaygın Risk Örnekleri

1. **Desteksiz veya yanlış karar desteği:** Hatalı ya da desteksiz bilgi; iş, hukuk,
   sağlık, finans veya operasyon kararlarını etkiler.
2. **İş akışlarında hatalı durum çıkarımı:** LLM, karşılanmamış bir koşulu karşılanmış
   sayar ve istenmeyen eylemler tetiklenir.
3. **Hatalı veya uydurma kod ve bağımlılıklar:** Model hatalı kod önerileri üretir
   veya var olmayan (halüsinasyona uğramış) paketlere referans verir (Spracklen vd.,
   2025).
4. **Yanıltıcı özetler ve kritik atlamalar:** Özetler; kilit kısıtları, istisnaları,
   zaman damgalarını veya riskleri atlar.
5. **Düşmanca tetiklenen yanlış bilgi:** Saldırganlar, yanlış iddialara veya kritik
   olguların atlanmasına yol açan girdiler kurgular.
6. **Ajanlar arası yanlış bilgi yayılımı:** Hatalı çıktılar ajanlar ve iş akışları
   boyunca yayılır.
7. **Sahte veya yanlış atfedilmiş kanıt:** Uydurma ya da manipüle edilmiş içerik,
   yetkili kanıt olarak sunulur.

## Önleme ve Azaltma Stratejileri

1. **Eylemden önce iddiaları temellendirin:** Çıktıların yetkili ve güncel kaynaklara
   dayandırılmasını zorunlu kılın.
2. **İddia-Doğrula-Eyle (Claim-Check-Act) kalıpları uygulayın:** Üretimi yürütmeden
   ayırın; eyleme geçmeden iddiaları doğrulayın.
3. **Araç çağrılarını doğrulayın:** Yürütmeden önce argümanları, yetkilendirmeyi, ön
   koşulları ve güncel durumu kontrol edin.
4. **Doğrulama sinyalleri kullanın (yalnızca güven skoru değil):** Temellilik ve
   tutarlılık kontrolleri ekleyin.
5. **Yüksek etkili eylemler için çalışma zamanı doğrulaması zorlayın:** Onay iş
   akışları ve sistem kontrolleri getirin.
6. **Atlama arızalarını tespit edip önleyin:** Zorunlu alanlar içeren yapılandırılmış
   çıktılar isteyin.
7. **Etki yarıçapını sınırlayın:** En az yetki, sandbox ve hız sınırları uygulayın.
8. **Yanlış bilgiyi izleyin ve test edin:** İddiaları, kanıtları ve sonuçları
   günlükleyin; düşmanca senaryolarla test edin.
9. **İnsan ve sistem güvenini kalibre edin:** Doğrulanmış olguları varsayımlardan
   ayırt edin.
10. **Düşmanca değerlendirme ve sürekli test:** İş akışlarını yanıltıcı senaryolara
    karşı düzenli test edin.

## Örnek Saldırı Senaryoları

**Senaryo #1 — Halüsinasyonlu bağımlılık önerisi:** Kodlama asistanı, makul görünen
ama var olmayan bir paket önerir; saldırgan bu adı önceden kaydetmiştir. Öneriye
güvenen geliştirici, saldırgan kontrolündeki kodu kurar (Spracklen vd., 2025).

**Senaryo #2 — Ajanın hatalı politika kararı:** Müşteri hizmetleri ajanı bir
politikayı yanlış okur ve şartları ihlal eden bir iadeyi onaylar — finansal kayıp.

**Senaryo #3 — Emniyet-kritik özette atlama:** Klinik bir özet, bir ilaç
kontrendikasyonunu atlar; klinisyen eksik öneriye göre hareket eder.

**Senaryo #4 — Düşmanca tetiklenen yanlış akıl yürütme:** Saldırgan, bir destek
forumunu yanlış çözüm adımlarıyla tohumlar; sorun giderme ajanı bunları getirir ve
güvenilir öneri olarak yineler.

**Senaryo #5 — Yanlış alarm otomatik müdahaleyi tetikler:** Güvenlik ajanı normal
trafiği saldırı diye sınıflandırır ve bir üretim ağı segmentini otomatik engeller —
kesinti.

**Senaryo #6 — Ajanlar arası güven başarısızlığı:** Getirme ajanı bir müşteriyi
kimliği doğrulanmış diye bildirir — oysa değildir; aşağı akıştaki ödeme ajanı bu
duruma güvenip parayı serbest bırakır.

**Senaryo #7 — Uydurma görev tamamlama:** Ajan, hiç çalışmamış gece veritabanı
yedeğinin tamamlandığını bildirir; sonraki geri yükleme, yedek olmadığı için
başarısız olur.

---

*Orijinal: OWASP Top 10 for LLM Applications 2026, OWASP GenAI Security Project,
CC BY-SA 4.0 — https://genai.owasp.org. Bu gayriresmî Türkçe çeviri aynı lisansla
dağıtılır. Kaynakça için orijinal belgenin References bölümüne bakınız.*
