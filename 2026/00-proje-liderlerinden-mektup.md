# Proje Liderlerinden Mektup

*(Letter from the Project Leads — Türkçe çeviri)*

Kandırılamayan bir model inşa etmeye çalışmaktan vazgeçin. Sistemi modelin etrafına öyle
kurun ki model kandırıldığında — ki kandırılacak — önemli hiçbir şey kırılmasın. Bu duruş,
buradaki on maddenin tamamına işlemiş durumda. Bu yıl, ilk kez, sizden buna inanmanızı
istemek yerine kanıtı gösterebiliyoruz.

Bu listenin bugüne kadarki her sürümü uzman kanaatiyle inşa edildi. Yüzlerce uygulayıcı
en önemli olanın ne olduğu konusunda görüş bildirdi. Bu oylama hâlâ listenin omurgası ve
orada kalmayı hak ediyor. Ancak bu yıl projenin daha önce hiç yapmadığı bir şeyi yaptık:
oylamayı, gerçekte neyin ters gittiğine dair bir kayıtla sınadık. Kamuya açık zafiyet
veritabanlarından ve bir yapay zekâ zarar veritabanından 7.714 gerçek olaydan oluşan bir
derlem topladık; bu olayları okuyup sınıflandırabilen modeller kurduk ve yeterli ayrıntı
taşıyan 6.639 tanesini kategorilere yerleştirdik. Sonra tek bir açık soru sorduk:
uygulayıcıların korktuğu şeyle olay kaydının gösterdiği şey örtüşüyor mu?

Cevap "hayır, her zaman değil" oldu ve sürpriz de buydu. Oy ile veri, belirli ve öğretici
noktalarda birbirinden ayrıldı; bu ayrılıklar bize uyuşmalardan daha fazlasını öğretti.

Komut enjeksiyonu (prompt injection) bunun en net örneği. Uygulayıcılar onu bir numaralı
risk olarak sıralıyor. Kategorileri ham olay kaydına göre sıralarsanız ise ilk 10'un
tamamen dışına düşüyor. Bu fark bir savunma etkisi: ekipler enjeksiyonla sıkı mücadele
ettiği için kamuya açık veritabanlarına daha az temiz istismar ulaşıyor ve kamusal sayım,
olgun ekiplerin hâlihazırda ciddi kaynak harcayarak dizginlediği riski olduğundan küçük
gösteriyor. Saldırı yüzeyi, modelin güvenilmeyen girdi okuduğu her yerde duruyor — yani
her yerde. Bu yüzden bir numarada kalıyor. Bu yüzeyi kapatamayacağınıza göre, onun size
karşı kullanılacağı gün için inşa edin.

Yanlış bilgi (misinformation) ters yönde ilerliyor ve üzerinde yavaşlamanızı istediğim
madde bu. Oy verenler onu listenin dibine yakın koydu. Olay kaydı ise tepeye yakın
yerleştirdi — gerçekten acıtan yöndeki en geniş makas: oyun düşük, kanıtın yüksek olduğu
yer. Liste, Misinformation'ı yine de ortada tutuyor: oy daha ağır basıyor ama kanıt onu
yukarı çekti. Mesele tam da bu anlaşmazlık. Modelin akıcı, kendinden emin çıktısı bir
kararı veya bir araç çağrısını tetiklediğinde, yanlış bir cevap yanlış bir eyleme dönüşür
ve kayıtlar bu başarısızlığın, oylamanın varsaydığından daha sık gerçekleştiğini
söylüyor.

Topluluk oyu ağırlığın dörtte üçünü, olay verisi kalan dörtte birini taşıyor. Oyu bilerek
ağır tuttuk. Bu liste bir konsensüs ürünü ve tek bir gürültülü veri yılı, işi bizzat yapan
insanların kanaatini devirme hakkına sahip değil. Dörtte birlik ağırlık, inanç ile kanıt
arasındaki makas genişlediğinde bir maddeyi bir kademe oynatmaya yetiyor; kusurlu verinin
listeyi tek başına yeniden yazmasına ise yetmiyor. İlk 10'daki her nihai sırayı bu denge
belirledi.

---

*Orijinal: OWASP Top 10 for LLM Applications 2026, OWASP GenAI Security Project,
CC BY-SA 4.0. Bu gayriresmî Türkçe çeviri aynı lisansla dağıtılır.*
