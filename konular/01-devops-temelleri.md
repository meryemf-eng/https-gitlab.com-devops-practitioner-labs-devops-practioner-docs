# DevOps'un temelleri

Bir değişikliğin fikirden kullanıcıya ulaşması ekibinizde kaç gün sürüyor? Bu soruya verilen cevap, bir yazılım organizasyonu hakkında çoğu mimari diyagramdan daha fazla şey anlatır. DevOps, bu süreyi kısaltırken kaliteyi ve kararlılığı da artırmanın yollarını arayan bir çalışma biçimidir. Bu dokümanda DevOps'un nereden geldiğini, ne olduğunu ve ne olmadığını, onu tarif eden çerçeveleri ve nasıl ölçüldüğünü ele alıyoruz.

## Yazılım teslimatının evrimi

Her yazılım geliştirme modeli aynı yolu yürür: fikir, gereksinim, kod, test, paketleme ve dağıtım. Bu yola yazılım geliştirme yaşam döngüsü, kısaca SDLC denir. Modelleri birbirinden ayıran şey adımlar değil, bu yolun ne sıklıkla ve ne büyüklükte paketlerle yürünmesidir.

**Waterfall**, yani şelale modeli bu yolu bir kez ve büyük parçalarla yürür. Her faz bir öncekinin bitmesini bekler; fazın çıktısı bir dokümandır ve bir sonraki ekibe devredilir. Kapsam baştan dondurulur, değişiklik resmi bir değişiklik talebi sürecine girer. Planlama ve bütçe kontrolü kolaydır. Ama kullanıcı ürünü ancak aylar sonra görür ve gereksinim yanlış anlaşıldıysa bu ancak test ya da kabul aşamasında ortaya çıkar. Yazılım mühendisliğinde uzun zamandır bilinen bir gözlem, geç fark edilen hatanın çok daha pahalı olduğudur: canlıda bulunan bir hatanın düzeltme maliyeti, gereksinim aşamasında bulunan bir hatanınkinden kat kat yüksektir. Waterfall'da test sonda olduğu için hatalar tam da en pahalı noktada bulunur.

Waterfall'ın bir başka sonucu, geliştirme ile operasyon arasındaki duvardır. Geliştirme ekibi yeni özellik çıkarmakla, operasyon ekibi sistemi kararlı tutmakla ödüllendirilir. Biri değişiklik ister, diğeri değişiklikten korkar. "Bende çalışıyordu" cümlesi, suçlama kültürü ve uzun release geceleri bu çelişen hedeflerin ürünüdür.

**Agile** bu soruna kısa döngülerle cevap verdi. 2001'de yayımlanan Agile Manifesto dört değer önerir: bireyler ve etkileşimler süreçlerden, çalışan yazılım kapsamlı dokümantasyondan, müşteriyle iş birliği sözleşme pazarlığından, değişime yanıt vermek bir planı takip etmekten daha değerlidir. Manifestonun en çok gözden kaçan cümlesi, sağdaki maddelerin de değerli olduğudur; Agile dokümantasyonu reddetmez, çalışan yazılımı ondan daha değerli bulur.

Agile, iteratif ve artımlı geliştirmeyi birleştirir: bir ile dört hafta süren döngülerin sonunda kullanılabilir bir ürün parçası çıkar, kullanıcı onu erkenden görür, ekip her döngüde kendi sürecini iyileştirir. En yaygın uygulaması olan **Scrum** üç sorumluluk tanımlar: ürünün değerini sahiplenen Product Owner, süreci kolaylaştıran Scrum Master ve çalışan artışı üreten geliştiriciler. **Kanban** ise işi görünür kılmaya ve aynı anda devam eden iş miktarını, yani WIP'i sınırlamaya odaklanır. Bir sütunun WIP limiti dolduğunda yeni iş çekilmez; bu kural darboğazı görünür kılar ve ekibi yeni işe başlamak yerine takılan işi bitirmeye yönlendirir.

**Lean**, yani yalın düşünce, Toyota üretim sisteminden yazılıma taşındı. Beş ilkesi şunlardır: müşterinin neye değer verdiğini tanımla, değerin aktığı adımları haritala, beklemeleri kaldırarak akış yarat, işi itmek yerine talep oldukça çek ve sürekli iyileştir. Lean'in yazılımda tanımladığı yedi israf, ekiplerin çoğuna tanıdık gelir: yarım kalmış iş, kimsenin kullanmadığı özellikler, gereksiz onay süreçleri, ekipten ekibe devir teslim, sürekli görev değiştirme, bekleme ve hatalar. Bunların içinde en büyüğü genellikle beklemedir.

Agile geliştirmeyi hızlandırdı, ama bir sınıra çarptı: geliştirme ekibi iki haftada bir çalışan yazılım üretirken operasyon ekibi canlıya üç ayda bir çıkıyorsa, kod hazırdır ama kullanıcıya ulaşamaz. DevOps bu boşluktan doğdu. Agile doğru ürünü sık üretmeyi çözer; DevOps onu sık ve güvenle kullanıcıya ulaştırmayı.

## Akış verimliliği: zaman nereye gidiyor?

Lean'in en güçlü araçlarından biri, bir işin ne kadar süre aktif olarak çalışıldığını ne kadar süre beklediğiyle karşılaştırmaktır. Bir indirim kuponu özelliğini düşünelim: kodu dört saatte yazılıyor, ama review için bir gün, test ortamı için iki günden fazla, onay için bir gün daha bekliyor. Toplamda 96 saat sonra kullanıcıya ulaşıyor.

**Akış verimliliği**, aktif sürenin toplam süreye oranıdır; bu örnekte dört bölü 96, yani yaklaşık yüzde dört. Bu tür ölçümler, hız kazanmanın yolunun daha hızlı kod yazmak değil beklemeyi azaltmak olduğunu gösterir. Review beklemesini küçük değişiklikler ve ekip içi bir inceleme anlaşması, test ortamı beklemesini otomatik ve geçici ortamlar, onay beklemesini otomatik kalite kapıları azaltır. Bu üç çözüm, sonraki konuların konusudur: CI/CD, altyapının kod olması ve DevSecOps.

## DevOps nedir, ne değildir?

DevOps'un resmi, tek bir tanımı yoktur. Hareketin adı 2009'da Belçika'nın Ghent kentinde düzenlenen ilk DevOpsDays etkinliğinden gelir. Farklı kaynakların tanımlarındaki ortak noktayı şöyle özetleyebiliriz: DevOps, geliştirme ve operasyon ekiplerinin yazılımı kullanıcıya hızlı, güvenli ve sürekli biçimde ulaştırmak için birlikte çalıştığı kültür, pratik ve araçlar bütünüdür.

Bu tanımın üç katmanı vardır ve sıraları önemlidir. **İnsan**: ortak hedef ve ortak sorumluluk. **Süreç**: küçük değişiklikler ve hızlı geri bildirim. **Teknoloji**: otomasyon, ölçüm ve self-service. Önce insan, sonra süreç, en son araç. Sıra tersine döndüğünde, yani dönüşüm bir araç alımıyla başladığında, sonuç genellikle hayal kırıklığıdır.

DevOps hakkında birkaç yaygın yanılgıyı açıkça söylemek gerekir. DevOps bir ürün değildir; satın alınamaz ve kurulamaz. Bir framework değildir; Scrum gibi bir kural seti yoktur. Adım adım bir reçete de değildir. Birkaç "DevOps mühendisi" işe almakla da elde edilmez. Ayrı bir "DevOps ekibi" kurmak çoğu zaman geliştirme ile operasyon arasına üçüncü bir silo eklemekle sonuçlanır. DevOps'un "operasyona gerek yok" anlamına geldiği de doğru değildir; operasyon bilgisi ortadan kalkmaz, herkesin işine yayılır. Son olarak, DevOps sadece hızla ilgili değildir: bu alandaki araştırmaların en önemli bulgularından biri, yüksek performanslı ekiplerin hem daha hızlı hem de daha kararlı olduğudur. Kısacası Jenkins kurmak DevOps değildir; Jenkins'i birlikte kullanan ekiplerin çalışma biçimi DevOps'tur.

## Yaşam döngüsü: sonsuzluk modeli

DevOps genellikle bir sonsuzluk işaretiyle çizilir. Sol halka geliştirmedir: planla, kodla, derle, test et. Sağ halka operasyondur: yayına hazırla, dağıt, işlet, izle. Döngünün en kritik geçişi izlemeden planlamaya dönen oktur: canlıdan gelen bilgi bir sonraki planlamayı besler. Bu geçiş kopuk olan organizasyon, canlıda ne olduğunu bilmeden plan yapar.

Her aşama bir soruya cevap verir ve bir sonrakine somut bir çıktı bırakır. Plan "neyi neden yapıyoruz?" sorusunu cevaplar ve önceliklendirilmiş bir iş listesi bırakır. Code değişikliğin nasıl yazılıp incelendiğini, build kodun çalıştırılabilir bir pakete dönüşüp dönüşmediğini, test değişikliğin doğru ve güvenli olup olmadığını sorar. Release hangi sürümün yayına hazır olduğunu belirler ve versiyonlu, değişmez bir paket bırakır. Deploy sürümü ortama ulaştırır, operate altyapıyı ve çalışan sistemi yönetir, monitor kullanıcının ne yaşadığını gösterir. Bir aşama bir sonrakine somut bir çıktı bırakmıyorsa, orada büyük ihtimalle elle yapılan bir iş ve bir bekleme vardır.

DevOps'ta sık duyulan "continuous" kelimesi, işin küçük parçalarla ve otomatik olarak sürekli aktığını anlatır. **Continuous integration** değişikliklerin günde birkaç kez ana dala birleşmesi ve her birleşmenin otomatik olarak derlenip test edilmesidir. **Continuous delivery** her başarılı build'in yayına hazır bir paket olması, canlıya çıkışın ise bir iş kararı olarak kalmasıdır. **Continuous deployment**, testleri geçen her değişikliğin insan onayı olmadan canlıya çıkmasıdır. Bu üç kavramı CI/CD konusunde ayrıntılı ele alacağız.

## Roller, ekip yapısı ve Conway yasası

DevOps bir unvan değil, değer akışındaki bütün rollerin ortak kültürüdür. Lider güven kültürünü kurar ve otomasyona yatırım yapar. Yazılımcı kodunu test eder ve canlıda çalışacak kalitede teslim eder. Test uzmanları testleri otomatikleştirir. Güvenlik ekibi bir denetçi değil, rehber rolü üstlenir. Platform ve SRE ekipleri herkesin kullanacağı güvenli dağıtım yollarını kurar. Yazılımcı test etmeden kod gönderiyorsa ya da yönetici güvenliği önemsemiyorsa, ekipte kaç DevOps mühendisi olduğunun önemi kalmaz.

Ekip yapısının yazılımı nasıl etkilediğini en iyi anlatan gözlem **Conway yasasıdır**: bir sistemi tasarlayan organizasyonlar, kendi iletişim yapılarının kopyası olan tasarımlar üretir. Birbirleriyle ticket kuyrukları üzerinden konuşan silolar, birbirine sıkı bağlı ve kırılgan servisler üretir; bağımsız, uçtan uca sorumlu ekipler ise bağımsız olarak dağıtılabilen servisler üretir. Bu gözlemden bir strateji doğar: istenen mimariye göre ekipleri yeniden düzenlemek. Mikroservise geçmek isteyip ekipleri silolarda bırakan bir kurum, çoğu zaman dağıtık bir monolit elde eder.

DevOps'ta aranan beceri profili **T-shaped** olarak anılır: T'nin yatay çizgisi Linux, ağ, Git, CI/CD, container, bulut, altyapı kodu, güvenlik ve gözlem gibi geniş bir temeli; dikey çizgisi ise bir alanda derin uzmanlığı temsil eder. Tek alanda derin ama başka hiçbir şeyi bilmeyen bir uzman ekipte darboğaz olabilir; her şeyden biraz bilen biri ise derin bir sorunda tıkanır. İyi bir ekip, birbirini tamamlayan T profillerinden oluşur.

## CALMS: DevOps'un beş boyutu

Bir kurumun DevOps olgunluğunu değerlendirmek için en yaygın kullanılan çerçeve CALMS'tır. Beş harf, beş soruya karşılık gelir.

**Culture**, yani kültür: ortak hedef ve güven var mı? "Geliştiricinin kodu, operasyonun sorunu" ayrımı kalkmış mı? Amazon'un CTO'su Werner Vogels'in sıkça alıntılanan ilkesi "you build it, you run it", kodu yazan ekibin canlıdan da sorumlu olmasını önerir. Kültür yukarıdan ilan edilemez; davranış değiştikçe değişir. Ortak nöbet rotasyonu, ortak postmortem ve ortak panolar bu değişimi başlatan pratiklerdir.

**Automation**, yani otomasyon: tekrarlanan iş otomatik mi? Genel öneri, geri bildirimi en çok hızlandıran yerden başlamaktır; bu da çoğu zaman build ve testtir. Ardından altyapı, dağıtım, güvenlik ve gözlem gelir. Bir uyarıyı da eklemek gerekir: kötü bir süreci otomatikleştirmek, kötü sonucu yalnızca hızlandırır.

**Lean**: bekleme ve israf azalıyor mu? Küçük değişiklikler, düşük WIP ve görünür akış bu boyutun pratikleridir. Kırk yedi dosyalık bir değişikliğin incelenmesi günler sürerken tek bir fikri içeren küçük bir değişiklik saatler içinde incelenir.

**Measurement**, yani ölçüm: sonuçları veriyle görüyor muyuz? Burada aktivite ile sonucu ayırmak gerekir. Yazılan satır sayısı, commit sayısı ya da kişi başına iş puanı gibi metrikler davranışı bozar; insanlar sayıyı optimize eder, sonucu değil. Goodhart yasası bunu şöyle özetler: bir ölçüm hedefe dönüştüğünde iyi bir ölçüm olmaktan çıkar. Teslimat sıklığı, değişikliğin canlıya ulaşma süresi, hata oranı ve toparlanma süresi gibi sonuç metrikleri ise ekip ve sistem düzeyinde ölçülür.

**Sharing**, yani paylaşım: bilgi ekipte çoğalıyor mu? Depoda yaşayan dokümantasyon, kod incelemeleri, ortak kanallarda alınan kararlar ve eşli çalışma bu boyutun araçlarıdır. "Bus factor" kavramı riski iyi anlatır: bir projeyi durdurmak için kaç kişinin aniden ayrılması yeterli? Bu sayı bir ise, sistem tek bir kişinin tatiline bağlıdır.

## Three Ways: üç iyileştirme yolu

DevOps'u tarif eden ikinci çerçeve, Gene Kim'in The Phoenix Project ve The DevOps Handbook kitaplarının temelini oluşturan üç yoldur.

**Birinci yol akıştır**: işin geliştirmeden operasyona, oradan kullanıcıya hızla ve güvenle ilerlemesi. Akışı belirleyen şey en yavaş adımdır. Geliştirme günde beş, test günde iki iş yapabiliyorsa, sistem günde iki iş teslim eder. Eliyahu Goldratt'ın kısıtlar teorisi bunu şöyle formüle eder: darboğaz dışında yapılan iyileştirme bir yanılsamadır. Önce darboğazı bulmak, onu tam kullanmak, diğer her şeyi ona göre ayarlamak ve sonra genişletmek gerekir. Yazılım teslimatında tipik darboğazlar ortam hazırlığı, elle yapılan testler, elle verilen onaylar ve dağıtımın kendisidir.

**İkinci yol geri bildirimdir**: sorunu kaynağında ve erkenden görmek. Geri bildirim döngüleri hızlarına göre sıralanabilir: yazarken uyaran editör ve linter saniyeler içinde, commit öncesi testler dakikalar içinde, CI pipeline'ı on dakika civarında, staging ortamı saatler içinde geri bildirim verir. En pahalı yol ise günler sonra gelen kullanıcı şikâyetidir. Amaç hataların çoğunu en kısa döngülerde yakalamak ve canlıda bir sorun olduğunda kullanıcıdan önce haberdar olmaktır.

**Üçüncü yol sürekli öğrenmedir**: deney yapmak, hatadan öğrenmek ve ustalaşmak. Bu yolun pratikleri planlı arıza tatbikatları, yani game day'ler; kontrollü hata enjeksiyonu, yani chaos engineering; suçlamadan öğrenmeyi amaçlayan postmortem'ler ve kapasitenin bir kısmını teknik borca ayırmaktır. Kriz anında bir ekibin performansı, provalarının seviyesine iner.

Üç yol birbirini besler: hızlı akış olmadan hızlı geri bildirim olmaz, geri bildirim olmadan öğrenme olmaz.

## Value stream mapping

Value stream mapping, yani değer akışı haritalama, fikirden kullanıcıya giden yolu adım adım çizip her adımda ne kadar çalışıldığını ve ne kadar beklendiğini görünür kılar. Dört ölçü kullanılır. **Process time**, bir adımda aktif olarak çalışılan süredir. **Lead time**, işin adıma girişinden çıkışına kadar geçen toplam süredir ve beklemeyi de içerir. **Complete and accurate (%C/A)**, bir adımın çıktısının sonraki adım tarafından düzeltme istenmeden kullanılabilme oranıdır. **Activity ratio** ise toplam process time'ın toplam lead time'a oranı, yani akış verimliliğidir.

Örnek haritada kupon özelliği 96 saatte canlıya çıkıyor, ama bunun yalnızca dört saati aktif çalışma. Bütün adımların %C/A değerlerinin çarpımı ise, işlerin yalnızca üçte birinin hiç geri dönmeden canlıya ulaştığını gösteriyor. Gelecek durumu tasarlarken her darboğaza bir çözüm eşlenir: review kuyruğu için küçük değişiklikler ve inceleme süresi hedefi, test ortamı beklemesi için kodla oluşturulan geçici ortamlar, onay beklemesi için kanıta dayalı otomatik onay, elle testin düşük kalitesi için test otomasyonu. Bu çözümlerle kodlama süresine hiç dokunmadan teslim süresi birkaç saate inebilir.

## Kültür: Westrum modeli ve psikolojik güvenlik

Sosyolog Ron Westrum, organizasyonları bilginin nasıl aktığına göre üç tipe ayırır. **Patolojik**, yani güç odaklı kültürlerde kötü haber getiren cezalandırılır, hata olunca günah keçisi aranır. **Bürokratik**, yani kural odaklı kültürlerde kötü haber görmezden gelinir, sorumluluk dar tanımlanır. **Üretken**, yani performans odaklı kültürlerde kötü haber getirene teşekkür edilir, sorumluluk paylaşılır, hata sorgulanır ve ondan öğrenilir. DORA araştırmaları, üretken kültürün daha yüksek teslimat performansı ve daha az tükenmişlikle ilişkili olduğunu göstermiştir.

Bu kültürün temeli **psikolojik güvenliktir**: ekipte soru sormanın, hata kabul etmenin ve fikir söylemenin güvenli olduğuna dair ortak inanç. Google'ın 2015'te yüzlerce ekibi incelediği Project Aristotle çalışmasında, başarılı ekipleri ayıran en güçlü faktör olarak öne çıkmıştır. Teknik bir dille söylersek: insanlar sorunları saklarsa, hiçbir izleme aracı onları zamanında bulamaz.

Bir olaydan sonra yazılan **blameless postmortem**, yani suçlamasız olay sonrası analiz, bu kültürün somut pratiğidir. "Kim yaptı?" yerine "sistem buna nasıl izin verdi?" diye sorar. Etkiyi, zaman çizelgesini, katkıda bulunan etkenleri ve aksiyonları kayda geçirir. Aksiyonların kalitesi kritiktir: "bir dahaki sefere daha dikkatli olun" hiçbir şeyi değiştirmez; "eksik doğrulamayı pipeline'a ekle, sorumlusu şu kişi, bitiş tarihi şu gün" sistemi değiştirir. İyi niyet bir strateji değildir; sistemi koruyan mekanizmalar inşa etmek gerekir.

## Ölçüm: DORA metrikleri

DORA (DevOps Research and Assessment), yıllardır binlerce ekibin teslimat performansını inceleyen bir araştırma programıdır. Teslimatı iki soruyla ölçer: ne kadar hızlıyız ve ne kadar kararlıyız?

| Metrik | Ne ölçer? |
|---|---|
| Deployment frequency | Canlıya ne sıklıkla dağıtım yapılıyor |
| Change lead time | Bir commit'in canlıya ulaşma süresi |
| Failed deployment recovery time | Başarısız bir dağıtımdan sonra toparlanma süresi |
| Change fail rate | Acil müdahale gerektiren dağıtımların oranı |
| Deployment rework rate | Plansız, düzeltme amaçlı dağıtımların oranı |

En önemli bulgu, hız ile kararlılığın birbirinin alternatifi olmadığıdır; en iyi ekipler ikisinde de iyidir. En iyi performans kümesindeki ekipler ihtiyaç duydukça, günde birçok kez dağıtım yapar, değişikliklerini bir günden kısa sürede canlıya taşır ve bir sorundan bir saatten kısa sürede toparlanır. Bu değerler bir not değil, başlangıç noktasıdır: bir ekip kendisiyle yarışmalıdır.

Bu metrikler mevcut araçlardan hesaplanabilir: commit zamanı Git'ten, dağıtım kayıtları CI/CD aracından, olayların açılış ve kapanışı olay yönetimi aracından gelir. Başlangıç için kısa bir anket yeterlidir; sonra pipeline verisiyle otomatikleştirilebilir. Yine de Goodhart yasasını hatırlamak gerekir: metrikler hedefe dönüştürüldüğünde manipüle edilir.

DORA metrikleri ayda bir bakılan sonuçlardır. Günlük işte bunları **akış metrikleri** tamamlar: işin başlanmasından bitmesine kadar geçen süre (cycle time), aynı anda devam eden iş sayısı (WIP) ve belirli sürede tamamlanan iş sayısı (throughput). Little yasası bu üçü arasındaki ilişkiyi kurar: WIP büyüdükçe iş kuyrukta daha uzun kalır.

## Dönüşüm tuzakları ve nereden başlamalı?

DevOps dönüşümlerinde tekrar eden tuzaklar vardır: ayrı bir DevOps ekibi kurup üçüncü bir silo yaratmak, kültür ve süreç değişmeden araç almak, sistem yöneticisinin unvanını değiştirip işini aynı bırakmak, kötü bir süreci otomatikleştirmek, bireysel metriklerle insanları ölçmek ve güvenliği sona bırakmak.

Nereden başlanacağı sorusunun cevabı ise basittir: büyük bir dönüşüm projesi yerine, küçük ve ölçülebilir bir adım. Önce mevcut akışı ve kullanıcıya etkisini görün. Sonra en önemli sürtünmeyi seçin. Sorumlusu ve ölçütü belli küçük bir değişiklik yapın ve etkisini değerlendirin. Örneğin code review beklemesini azaltmak için küçük değişiklik kuralı ve dört saatlik bir inceleme hedefi koymak, takip ölçütü olarak da inceleme bekleme süresini ve hata oranını izlemek. Aynı anda tek bir değişiklik yapmak önemlidir; aksi hâlde hangi müdahalenin işe yaradığı anlaşılmaz.

> DevOps bir araç ya da unvan değil, insan, süreç ve teknolojiyle kurulan bir değer akışıdır. CALMS ve Three Ways bu akışı tarif eder, value stream haritası beklemeyi görünür kılar, DORA metrikleri de hız ile kararlılığı birlikte ölçer.
