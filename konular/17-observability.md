# Observability ve Prometheus

DevOps sonsuzluk döngüsünün son halkası izlemedir: kod yazıldı, paketlendi ve dağıtıldı; şimdi canlıda ne olduğunu görmek gerekir. Bu dokümanda monitoring ile observability farkını, telemetri sinyallerini, neyin ve nasıl ölçüleceğini, OpenTelemetry'yi, Prometheus'u, alarmları, Grafana'yı ve Kubernetes'te izlemeyi ele alıyoruz.

## Monitoring ve observability

İki kavramın farkı iki soruyla anlaşılır. Monitoring şunu sorar: "Hata oranı yükseldi mi? Yanıt süresi hedefi aşıldı mı?" Observability ise şunu sorar: "Hangi kullanıcılarda, hangi sürümde, hangi bağımlılıkta sorun oluştu?" Bir benzetmeyle, monitoring dumanı gösterir, observability yangının kaynağını bulmayı sağlar.

**Monitoring** önceden belirlenmiş metrikleri, eşikleri ve panoları sürekli izler ve sapma olduğunda alarm üretir; "ne bozuk?" sorusunu cevaplar ve bilinen koşullarla ilgilenir. **Observability** ise sistemin ürettiği telemetriden iç durumu çıkarmak ve daha önce düşünülmemiş bir sorunu da araştırabilmektir; "neden bozuk?" sorusunu cevaplar. Monitoring ağırlıklı olarak sabit boyutlu metriklerle çalışır; observability metriklerin yanında log, trace ve kullanıcı, sürüm, bölge gibi zengin bağlam gerektirir. Monitoring observability'nin bir parçasıdır: biri alarmı çaldırır, diğeri nedeni bulur.

Bir olay üzerinden farkı görelim. Monitoring akışında ödeme adımının yüzde doksan beşlik gecikmesi bir eşiği aşar, alarm tetiklenir ve nöbetçi uyarılır; monitoring'in cevabı, ödeme adımının hedefin dışında olduğu ve hata bütçesinin hızla tükendiğidir. Observability akışında trace'e bakılır, bir veritabanı sorgusunun bir saniyeden uzun sürdüğü görülür, sorunun yalnızca belirli bir sürümde olduğu filtrelenir, logda bir index'in eksik olduğu okunur ve geri alma kararı verilir. Bunun için ortak bir trace kimliği, sürüm etiketi, yapılandırılmış log ve zengin bağlam gerekir. Pano yalnızca önceden sorulmuş soruları gösterir; observability yeni bir soruyu sorabilmektir.

İzlemenin tarihi de bu değişimi yansıtır: önce "ayakta mı?" sorusunu soran host ve servis kontrolleri, sonra merkezi metrik ve log sistemleri, ardından etiketli metrikler ve güçlü bir sorgu diliyle Prometheus, son olarak metrik, log ve trace için ortak bir standart olan OpenTelemetry. Değişen iki şey ölçek ve sorudur: tek bir sunucu yerine yüzlerce kısa ömürlü Pod ve "ayakta mı?" yerine "kullanıcı ne yaşıyor ve neden?"

Farklı katmanlar farklı sorulara cevap verir: kullanıcı ve iş katmanında sipariş sayısı ve ödeme başarısı, uygulama katmanında istek hızı, hata oranı ve gecikme, platform katmanında Pod yeniden başlatmaları ve bekleyen Pod'lar, container katmanında bellek ve işlemci kısıtlaması, altyapı katmanında node kaynakları ve dış bağımlılıklarda veritabanı ve kuyruklar. Genel kural, alarmın üstten, yani kullanıcının yaşadığı belirtiden, teşhisin alttan, yani nedenden gelmesidir. İki bakış açısı da gereklidir: **black-box** izleme sistemi kullanıcı gibi dışarıdan dener ve belirtiyi gösterir; **white-box** izleme sistemin kendi ürettiği telemetriye dayanır ve nedeni gösterir.

## Telemetri sinyalleri

Üç sinyal üç farklı soruya cevap verir.

**Metrik** "ne kadar?" sorusunu cevaplar: hata oranı yüzde iki, saniyede yüz yirmi istek. **Log** "ne oldu?" sorusunu cevaplar: bir isteğin neden reddedildiği. **Trace** ise "zaman nerede geçti?" sorusunu cevaplar: API'de kırk, veritabanında iki yüz seksen milisaniye. Kısaca, metrik uyarır, trace yolu gösterir, log teşhis koyar.

Prometheus ve OpenTelemetry'de dört temel metrik tipi vardır. **Counter** yalnızca artar ve yeniden başlatmada sıfırlanır; toplam istek sayısı gibi. Tek başına anlamsızdır, saniyelik artış hızına bakılır. **Gauge** artar ve azalır; bellek kullanımı ya da kuyruk uzunluğu gibi. **Histogram** değerleri kovalara dağıtır; istek süresi gibi değerlerin yüzdeliklerini hesaplamak için kullanılır. **Summary** yüzdeliği istemci tarafında hesaplar, ama birden çok kaynağın summary'si birleştirilemez; bu yüzden histogram tercih edilir. Bir metrik bir ad, etiketler ve bir değerden oluşur; etiketler aynı metriği servis, metot ya da durum koduna göre filtrelemeyi sağlar.

**Yapılandırılmış log** insan için değil makine için yazılır. Düz metin bir satır yerine zaman, seviye, servis, mesaj, sipariş numarası, trace kimliği ve süre gibi alanlar içeren bir JSON kaydı her alanın aranabilmesini ve filtrelenebilmesini sağlar. Container logları standart çıktıya yazılır ve bir toplayıcı tarafından alınır. Log satırında trace kimliğinin bulunması logdan trace'e geçişi sağlar. Hata ayıklama seviyesi üretimde kapalı tutulur ve parolalar, token'lar, kart numaraları ve kişisel veriler loga yazılmaz.

**Distributed tracing**, bir isteğin servisler arasındaki yolculuğunu izler. Bir ödeme isteği web servisinden sepet ve ödeme servislerine, oradan veritabanına ve bir banka API'sine gidebilir. **Trace** isteğin tamamıdır ve bir trace kimliğiyle tanımlanır; **span** başlangıcı, süresi ve durumu olan tek bir işlemdir; **context propagation** ise trace bilgisinin servisler arasında standart bir HTTP başlığıyla taşınmasıdır. Metrik "ödeme yavaş" der; trace hangi çağrının yavaş olduğunu gösterir.

Metriklerde sessiz bir maliyet vardır: **kardinalite**. Prometheus'ta her benzersiz etiket kombinasyonu ayrı bir zaman serisidir. Sınırlı değer kümelerine sahip metot, yol ve durum kodu etiketleri birkaç yüz seri üretir; Pod sayısıyla çarpıldığında on binlere çıkar; kullanıcı kimliği gibi sınırsız bir etiket eklendiğinde milyarlara ulaşır ve Prometheus'un belleği tükenir. Kural şudur: kullanıcı, istek ve sipariş kimlikleri metriklerde değil, log ve trace'lerde bulunur.

## Neyi ölçmeli?

Neyin ölçüleceği için üç yöntem vardır. **RED** yöntemi istek alan servisler içindir: istek hızı (Rate), hata oranı (Errors) ve süre (Duration). **USE** yöntemi kaynaklar içindir: kullanım (Utilization), doygunluk (Saturation) ve hatalar (Errors); örneğin bir veritabanı bağlantı havuzunun doluluğu, bekleyen bağlantılar ve hatalar. **Dört altın sinyal**, yani gecikme, trafik, hatalar ve doygunluk, ise kullanıcıya dönük sistemler içindir.

SRE, platform ve DevSecOps konusundaki hata bütçesi kavramı burada alarm kuralına dönüşür. **Burn rate**, hata bütçesinin normalden kaç kat hızlı tükendiğidir. Yaygın bir yaklaşım çok pencereli alarmlardır: bir saatlik pencerede bütçenin çok hızlı tükenmesi nöbetçiyi hemen uyandırır, altı saatlik pencerede daha yavaş ama sürekli bir tükenme de uyandırır, üç günlük pencerede ise yavaş bir tükenme yalnızca bir iş kaydı açar. İki pencerenin birlikte kullanılmasının sebebi, uzun pencerenin sorunun gerçek olduğunu doğrulaması, kısa pencerenin ise sorun düzelince alarmın çabuk susmasını sağlamasıdır. "İşlemci yüzde sekseni geçti" gibi bir alarmın yerine geçen iyi alarm budur.

İyi bir pano bir soruya cevap verir. Panolar genel bakıştan servise, servisten kaynağa bir hiyerarşi izler; en üstte hizmet hedefleri ve altın sinyaller bulunur. Yönetici, nöbetçi ve geliştirici için ayrı panolar tasarlanır. Panolar kod olarak Git'te tutulur; elle yapılan bir pano bir gün kaybolur. Dağıtım işaretleri, eşik çizgileri ve birimler bağlam sağlar; paneller az ve öz tutulur ve panellerden log ve trace görünümlerine geçiş sağlanır. Panolar bakılmak için, alarmlar uyandırmak için vardır; kimse bir panoyu sürekli izlemez.

## OpenTelemetry

**OpenTelemetry**, üreticiden bağımsız bir API, SDK'lar, bir protokol (OTLP) ve bir toplayıcı (Collector) sunan bir CNCF projesidir.

Uygulama SDK ile telemetri üretir ve OTLP ile Collector'a gönderir; Collector metrikleri Prometheus'a, logları Elasticsearch'e ya da Loki'ye, trace'leri Jaeger'a ya da Tempo'ya iletir. SDK'lar Java, Go, Python ve Node.js gibi diller için otomatik ve manuel enstrümantasyon sağlar. Ortak semantik adlar, örneğin servis adı ve Pod adı, sinyallerin birbirine bağlanmasını sağlar. Asıl kazanç şudur: backend değiştirmek için uygulamayı yeniden enstrümante etmek gerekmez; yalnızca Collector'ın yapılandırması değişir.

Collector bir pipeline olarak yapılandırılır: alıcılar (receivers), işlemciler (processors) ve aktarıcılar (exporters). İki dağıtım modeli vardır: her node'da bir DaemonSet olarak çalışan ve log ile host metriklerini toplayan **agent** ve merkezi işleme, örnekleme ve yönlendirme yapan **gateway**.

Observability'nin asıl değeri sinyallerin birbirine bağlanmasından gelir. Bir alarmdan metrikteki bir örnek trace'e, oradan yavaş çağrıya, oradan ilgili log satırına ve kök nedene gidilebilir. Bunun için her sinyalde aynı kaynak özniteliklerinin bulunması, log satırlarında trace kimliğinin yer alması ve histogram değerlerine örnek trace kimliklerinin, yani exemplar'ların iliştirilmesi gerekir.

Açık kaynak dünyasında her sinyal için birden çok seçenek vardır: metrik için Prometheus ve uzun süreli depolama çözümleri, log için Elasticsearch, OpenSearch ve Loki, trace için Jaeger ve Tempo, görselleştirme için Grafana ve Kibana, toplama için OpenTelemetry Collector ve Fluent Bit. Ticari platformlar aynı sinyalleri yönetilen hizmet olarak sunar.

## Prometheus

Prometheus, Kubernetes izlemede fiilî standarttır: zaman serisi metrikleri için tasarlanmıştır, Pod'ları ve servisleri otomatik keşfeder, etiketlerle çalışır, güçlü bir sorgu dili (PromQL) sunar, Alertmanager ile alarm üretir, dış bir veritabanına ihtiyaç duymaz ve CNCF'in mezun projelerindendir. Prometheus metrikler içindir; loglar ve trace'ler için başka araçlar kullanılır.

Prometheus **pull modeliyle** çalışır: hedeflere kendisi gider ve onların metrik adresini okur.

Kubernetes API'sinden servis keşfi yapar, hedefleri düzenli aralıklarla okur, zaman serisi veritabanında saklar, kuralları değerlendirir ve alarmları Alertmanager'a gönderir. Kendisi metrik sunmayan sistemler için **exporter**'lar kullanılır: node, MySQL ya da Redis exporter'ları. Kısa süren toplu işler için bir istisna olarak Pushgateway bulunur. Veritabanı yerel diskte durur ve uzun süreli saklama için uzak depolamaya yazılabilir. Pull modelinin bir avantajı, Prometheus'un bir hedefin ayakta olup olmadığını kendisinin bilmesidir: okuma başarısız olursa hedefin "up" metriği sıfır olur.

Kubernetes'te dört metrik kaynağı vardır: node kaynakları için node_exporter, container kaynak kullanımı için kubelet'in içindeki cAdvisor, Kubernetes nesnelerinin durumu için kube-state-metrics ve iş ile RED metrikleri için uygulamaların kendisi. Uygulamalara metrik eklemek birkaç satır koddur: bir istemci kütüphanesiyle metot, yol ve durum etiketli bir istek sayacı tanımlanır ve uygulama Prometheus'a RED metrikleri sunmaya başlar.

**PromQL**, Prometheus'un sorgu dilidir. Seçiciler metrikleri ve etiketleri eşleştirir; köşeli parantezle bir zaman aralığındaki örnekler alınır. Temel fonksiyonlar, bir counter'ın saniyelik ortalama artışını veren `rate`, bir aralıktaki toplam artışı veren `increase`, etikete göre toplayan `sum by`, en büyük serileri seçen `topk` ve bir seri yoksa bir döndüren `absent`'tır. Altın kural, counter'a önce rate sonra sum uygulamaktır; önce toplayıp sonra rate almak, bir Pod yeniden başladığında sayacının sıfırlanması nedeniyle hesabı bozar. Sık kullanılan sorgular arasında servis başına saniyedeki istek, hata oranı, histogram ile yüzde doksan beşlik gecikme, node işlemci kullanımı, diskin belirli bir sürede dolup dolmayacağını eğilimden tahmin eden doğrusal öngörü ve son bir saatte çok sayıda yeniden başlayan Pod'lar bulunur.

**Recording rule**'lar pahalı sorguları düzenli olarak önceden hesaplayıp yeni seriler olarak saklar; panolar ve alarmlar bu hazır serileri okur ve hem hızlanır hem okunur hâle gelir. Kurallar da koddur ve CI'da doğrulanıp test edilebilir.

## Alarmlar ve Grafana

Bir alarm kuralı Prometheus'ta değerlendirilir, bildirim ise Alertmanager'dan çıkar.

Kuralın ifadesi bir koşul tanımlar; `for` süresi koşulun belirli bir süre devam etmesini şart koşar, böylece kısa sıçramalarda alarm çalmaz. Etiketler yönlendirme anahtarlarıdır, örneğin önem seviyesi ve ekip; açıklamalar ise insana yönelik bilgidir: özet, müdahale rehberi ve pano bağlantısı. Bir alarm etkin değil, beklemede, tetiklenmiş ve çözülmüş durumlarından geçer.

**Alertmanager** kural değerlendirmez; Prometheus'tan gelen alarmları yönlendirir. Benzer alarmları gruplar, böylece elli Pod çöktüğünde elli değil bir bildirim gelir. Etiketlere göre doğru ekibe ve kanala yönlendirir. Bir node çöktüğünde o node'daki diğer alarmları bastırabilir. Bakım pencerelerinde geçici susturma sağlar.

İyi ve kötü alarm arasındaki fark çoğu zaman belirti ile neden arasındaki farktır. İşlemci kullanımı yerine hizmet hedefini tehdit eden gecikme, tek bir yeniden başlatma yerine kısa sürede tekrarlanan ve Pod'u hazır olmaktan çıkaran yeniden başlatmalar, diskin yüzde doksan dolu olması yerine mevcut hızla birkaç saatte dolacak olması, her ortamda nöbetçiyi uyandırmak yerine üretimde uyandırıp test ortamında iş kaydı açmak iyi alarmlardır. Her alarmın bir müdahale rehberi olmalıdır. Her zaman çalan bir "watchdog" alarmı, alarm zincirinin kendisinin çalıştığını doğrular; gelmezse zincir bozuktur. Haftada kaç kez nöbetçinin uyandırıldığı da ölçülmelidir.

**Grafana**, açık kaynak bir görselleştirme ve analiz platformudur. Prometheus, Loki, Elasticsearch ve SQL veritabanları gibi birçok kaynağı tek panoda birleştirir, değişkenlerle yeniden kullanılabilir panolar sunar, erişimi rollerle yönetir ve pano olmadan keşif yapılabilen bir arayüz sağlar. Grafana veri saklamaz; sorguyu veri kaynağına gönderir ve sonucu çizer. Değişkenlerle bir pano bütün servisler için çalışabilir: üstteki listeden bir servis seçildiğinde bütün paneller güncellenir. Hazır panolar bir başlangıç noktası olarak kullanılıp uyarlanabilir. Grafana'nın kendi alarm sistemi de vardır ve birden çok veri kaynağı üzerinde alarm kurmayı sağlar; ama aynı alarmın iki yerden gelmemesi için bir kaynak seçilmelidir. Grafana'nın varsayılan yönetici parolasıyla bırakılmaması ayrıca önemlidir.

## Kubernetes'te izleme

Kubernetes'te Prometheus'u kurmanın en yaygın yolu tek bir Helm chart'ıdır: kube-prometheus-stack. Bu chart Prometheus Operator, Prometheus, Alertmanager, Grafana, node_exporter, kube-state-metrics ve yüzlerce hazır Kubernetes alarmı ve panosu getirir.

Operatör dünyasında okuma hedefleri ve kurallar da birer Kubernetes kaynağıdır. **ServiceMonitor**, belirli etiketli bir Service'in metrik adresinin hangi aralıkla okunacağını tanımlar. **PrometheusRule** alarm ve kayıt kurallarını tanımlar. Operatör bunları görür ve Prometheus'un yapılandırmasını otomatik günceller. Bir uygulamanın Helm chart'ına bir ServiceMonitor ve PrometheusRule eklemek, izlemenin uygulamayla birlikte sürümlenmesini sağlar.

İzleme yığınında en sık sorunlar şunlardır: hedefin listede görünmemesi genellikle ServiceMonitor'ın seçicisinin ya da namespace'inin eşleşmemesinden, hedefin kapalı görünmesi yanlış port adı, ağ politikası ya da eksik metrik adresinden, Prometheus'un belleğinin tükenmesi kardinalite patlamasından, Grafana'nın boş görünmesi veri kaynağı adresinden ya da zaman aralığından kaynaklanır. Kaçınılması gereken hatalar ise önce sum sonra rate almak, summary ile yüzdelik birleştirmeye çalışmak, for süresi olmayan alarmlar, müdahale rehberi olmayan alarmlar, elle yapılan panolar, varsayılan parolalar, sınırsız saklama süresi ve watchdog alarmının olmamasıdır.

> Monitoring "ne bozuk?", observability "neden bozuk?" sorusunu cevaplar. Metrik uyarır, trace yolu gösterir, log teşhis koyar; RED servisler, USE kaynaklar, hizmet hedefleri kullanıcı deneyimi içindir. Prometheus pull modeliyle metrik toplar, Alertmanager doğru alarmı doğru kişiye bir kez ulaştırır, Grafana görselleştirir.
