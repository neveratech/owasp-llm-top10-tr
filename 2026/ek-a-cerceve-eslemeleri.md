# Ek A: İlgili Çerçeve Eşlemeleri

*(Appendix A: Related Framework Mappings — Türkçe çeviri, giriş ve kapsam matrisi)*

Bu ek; on OWASP Top 10 for LLM Applications (2026) risk maddesinden dokuz harici
güvenlik çerçevesine ve taksonomisine yapılan eşlemeleri tek yerde toplar. Madde
başına yer alan "İlgili Çerçeveler ve Taksonomiler" bölümlerinin yerini alır; o
bölümler bu tek, sürüm-sabitlenmiş referans lehine kaldırılmıştır.

## Bu ek nasıl okunur

Kapsam matrisi, her riske hangi çerçevelerin eşlendiğini tek bakışta gösterir.
Çerçeveye göre bölümler, belirli öge eşlemelerini ve neden geçerli olduklarını verir.
Eşlemeler her çerçevenin kaba düzeyindedir (taktikler, sütunlar/zayıflıklar, risk
kategorileri, kontrol alanları); her öge, Framework Sources & Versions'ta listelenen
sabitlenmiş çerçeve sürümünden alınmıştır. Her hücre, birincil eşlemeleri ve en
ilgili destekleyicileri gösterir.

**Gösterim:** ● birincil (riskin merkezî bir savunma hattı veya tanımı) · ○
destekleyici (katkı sağlar ama ağırlık merkezi değildir) · — geçerli eşleme yok.

## Kapsam Matrisi

| Risk | ASI | DSGAI | ATLAS | ATT&CK | CWE | 600-1 | RMF | AICM | AIVSS |
|---|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| LLM01 Prompt Injection | ● | ● | ● | ● | ● | ● | ○ | ● | ● |
| LLM02 Sensitive Information Disclosure | ● | ● | ● | ● | ● | ● | ○ | ● | — |
| LLM03 Excessive Agency | ● | ● | ● | ● | ● | ● | ○ | ● | ● |
| LLM04 Supply Chain | ● | ● | ● | ● | ● | ● | ● | ● | ○ |
| LLM05 Data and Model Poisoning | ● | ● | ● | ○ | ● | ● | ○ | ● | ○ |
| LLM06 Unbounded Consumption | ● | ● | ● | ● | ● | ● | ○ | ● | ○ |
| LLM07 Misinformation | ● | ● | ● | ○ | ● | ● | ● | ● | ● |
| LLM08 Hidden Context Exposure | ● | ○ | ● | ○ | ● | ● | ○ | ● | ○ |
| LLM09 Vector and Embedding Weaknesses | ● | ● | ● | ○ | ● | ● | ○ | ● | ○ |
| LLM10 Improper Output Handling | ● | ● | ● | ● | ● | ● | — | ● | ○ |

**Eşlenen çerçeveler ve sabitlenmiş sürümleri:** OWASP Top 10 for Agentic
Applications (ASI) — 2026; OWASP GenAI Data Security 2026 (DSGAI) — v1.0; MITRE
ATLAS — v2026.06; MITRE ATT&CK — v19.1; MITRE CWE — 4.20; NIST AI 600-1 (Generative
AI Profile) — v1.0; NIST AI RMF (AI 100-1) — v1.0; CSA AI Controls Matrix (AICM) —
v1.1; OWASP AIVSS — v0.8.

## Çerçeve bazlı ayrıntılı eşleme tabloları hakkında not

Orijinal belgenin 59–105. sayfalarındaki çerçeve bazlı ayrıntılı eşleme bölümleri;
her risk için ilgili çerçevenin resmî öge kimliklerini (örneğin MITRE ATLAS teknik
kodları, CWE numaraları, NIST kontrol adları) ve kısa gerekçeleri listeler. Bu
kimlikler ve çerçeve terimleri **resmî tanımlayıcılar olduğundan çevrilmemiştir**;
Türkçeye çevrilmeleri, ilgili çerçevelerle çapraz referans kullanımını bozardı.
Ayrıntılı eşleme tabloları için [orijinal belgeye](https://genai.owasp.org/resource/owasp-genai-llm-top-10-2026/)
başvurunuz.

---

*Orijinal: OWASP Top 10 for LLM Applications 2026, OWASP GenAI Security Project,
CC BY-SA 4.0 — https://genai.owasp.org. Bu gayriresmî Türkçe çeviri aynı lisansla
dağıtılır.*
